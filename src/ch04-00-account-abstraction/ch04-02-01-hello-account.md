# سلام قرارداد حساب جهانی

در این فصل ما یک قرارداد حساب از ابتدا با پیروی از استانداردهای SNIP-6 و SRC-5 ایجاد می کنیم.

### راه اندازی پروژه

برای استقرار یک قرارداد حساب در شبکه آزمایشی یا شبکه اصلی Starknet، مطمئن شوید که از نسخه‌ای از Scarb استفاده می‌کنید که از هدف Sierra 1.3.0 پشتیبانی می‌کند، زیرا هم شبکه آزمایشی و هم شبکه اصلی Starknet در حال حاضر از این نسخه پشتیبانی می‌کنند. برای جزئیات بیشتر به یادداشت های انتشار Starknet مراجعه کنید. از اکتبر 2023، نسخه پیشنهادی Scarb 0.7.0 است.

```bash
$ scarb --version
scarb 0.7.0 (58cc88efb 2023-08-23)
cairo: 2.2.0 (https://crates.io/crates/cairo-lang-compiler/2.2.0)
sierra: 1.3.0

```

برای نصب یا به‌روزرسانی Scarb دستورالعمل‌های اینجا را دنبال کنید.

### راه اندازی یک پروژه Scarb جدید

راه اندازی یک پروژه جدید:

```bash
$ scarb new aa
Created `aa` package.

```

ساختار پیش فرض پروژه را بررسی کنید:

```bash
$ tree .
.
└── aa
    ├── Scarb.toml
    └── src
        └── lib.cairo

```

به‌طور پیش‌فرض، Scarb برای Vanilla Cairo تنظیم می‌شود. برای هدف قرار دادن Starknet، Scarb.toml را تغییر دهید تا افزونه Starknet فعال شود.

```bash
[package]
name = "aa"
version = "0.1.0"

[dependencies]
starknet = "2.2.0"

[[target.starknet-contract]]

```

در فایل src/lib.cairo، کد قاهره را با داربست قرارداد حساب خود جایگزین کنید:

```rust
#[starknet::contract]
mod Account {

}

```

برای تأیید اعتبار امضاها، کلید عمومی مرتبط با کلید خصوصی امضاکننده را ذخیره کنید.

```rust
#[starknet::contract]
mod Account {

    #[storage]
    struct Storage {

        public_key: felt252
    }
}

```

در نهایت، پروژه را برای تأیید تنظیمات کامپایل کنید:

```bash
aa/src$ scarb build
   Compiling aa v0.1.0 (/home/Cyndie/account_abstraction__starknet/aa/Scarb.toml)
    Finished release target(s) in 1 second

```

### پیاده سازی SNIP-6

همانطور که در زیرفصل قبلی توضیح داده شد، برای طبقه بندی قرارداد هوشمند به عنوان یک قرارداد حساب، باید به ویژگی ISRC6 پایبند باشد:

```rust
trait ISRC6 {
  fn __execute__(calls: Array<Call>) -> Array<Span<felt252>>;
  fn __validate__(calls: Array<Call>) -> felt252;
  fn is_valid_signature(hash: felt252, signature: Array<felt252>) -> felt252;
}
```

توابع درون قرارداد حساب مشروح شده با ویژگی #\[external(v0)] دارای انتخابگرهای منحصربه‌فردی هستند که تعامل با قرارداد توسط نهادهای خارجی را تسهیل می‌کنند.

با این حال، در حالی که این توابع به صورت عمومی از طریق انتخابگرها قابل دسترسی هستند، تشخیص کاربران مورد نظر آنها ضروری است. به طور خاص، توابع **execute** و **validate** منحصراً برای پروتکل Starknet محفوظ هستند. در مقابل، is\_valid\_signature به صورت عمومی در دسترس است و به برنامه های web3 برای اعتبارسنجی امضا ارائه می شود.

ویژگی IAccount که با ویژگی #\[starknet::interface] مشخص شده است، توابع در نظر گرفته شده برای تعامل عمومی، مانند is\_valid\_signature را در بر می گیرد. توابع **execute** و **validate**، اگرچه عمومی هستند، به طور غیر مستقیم قابل دسترسی هستند.

```rust
use starknet::account::Call;

#[starknet::interface]
trait IAccount<T> {
  fn is_valid_signature(self: @T, hash: felt252, signature: Array<felt252>) -> felt252;
}

#[starknet::contract]
mod Account {
  use super::Call;

  #[storage]
  struct Storage {
    public_key: felt252
  }

  #[external(v0)]
  impl AccountImpl of super::IAccount<ContractState> {
    fn is_valid_signature(self: @ContractState, hash: felt252, signature: Array<felt252>) -> felt252 { ... }
  }

  #[external(v0)]
  #[generate_trait]
  impl ProtocolImpl of ProtocolTrait {
    fn __execute__(ref self: ContractState, calls: Array<Call>) -> Array<Span<felt252>> { ... }
    fn __validate__(self: @ContractState, calls: Array<Call>) -> felt252 { ... }
  }
}
```

### محافظت از عملکرد فقط پروتکل

برای امنیت بیشتر حساب، توابع **execute** و **validate** منحصراً توسط پروتکل Starknet قابل فراخوانی هستند، حتی اگر برای عموم قابل دسترسی باشند. پروتکل Starknet هنگام فراخوانی یک تابع از آدرس صفر استفاده می کند. عملکرد خصوصی only\_protocol تضمین می کند که فقط پروتکل Starknet می تواند به این توابع دسترسی داشته باشد

```rust
...

//Starknet uses zero address as caller when calling function
#[starknet::contract]
mod Account {
  use starknet::get_caller_address;
  use zeroable::Zeroable;
  ...

  //protection of protocol-only functions using only_protocol()
  #[external(v0)]
  #[generate_trait]
  impl ProtocolImpl of ProtocolTrait {
    fn __execute__(ref self: ContractState, calls: Array<Call>) -> Array<Span<felt252>> {
      self.only_protocol();
      ...
    }
    fn __validate__(self: @ContractState, calls: Array<Call>) -> felt252 {
      self.only_protocol();
      ...
    }
  }

  //creation of private function only_protocol()
  #[generate_trait]
  impl PrivateImpl of PrivateTrait {
    fn only_protocol(self: @ContractState) {...}
  }
}

```

توجه داشته باشید که تابع is\_valid\_signature توسط تابع only\_protocol محافظت نمی شود زیرا آزادانه استفاده می شود.

### اعتبار سنجی امضا

کلید عمومی مرتبط با امضاکننده قرارداد حساب برای تأیید امضای تراکنش ذخیره می شود. متد سازنده برای گرفتن مقدار کلید عمومی در حین استقرار تعریف شده است. .

```rust
...

#[starknet::contract]
mod Account {
  ...

  #[storage]
  struct Storage {
    public_key: felt252
  }

  #[constructor]
  fn constructor(ref self: ContractState, public_key: felt252) {
    self.public_key.write(public_key);
  }
  ...
}
```

تابع is\_valid\_signature برای یک امضای معتبر VALID و در غیر این صورت 0 را برمی گرداند. یک تابع داخلی، is\_valid\_signature\_bool، یک نتیجه بولی برای اعتبار سنجی امضا ارائه می دهد.

```rust
...

#[starknet::contract]
mod Account {
  ...
  use array::ArrayTrait;
  use ecdsa::check_ecdsa_signature;
  use array::SpanTrait;  //Implements span() method

  ...

  //Implementation of is_valid_signature method
  #[external(v0)]
  impl AccountImpl of super::IAccount<ContractState> {
    fn is_valid_signature(self: @ContractState, hash: felt252, signature: Array<felt252>) -> felt252 {
      //Derive Span from signature passed as Array to be used in is_valid_signature_bool()
      let is_valid = self.is_valid_signature_bool(hash, signature.span());
      if is_valid { 'VALID' } else { 0 }
    }
  }

  ...

  //Implementation of is_valid_signature_bool to return bool
  #[generate_trait]
  impl PrivateImpl of PrivateTrait {
    ...

    //function signature expects a Span instead of an Array
    fn is_valid_signature_bool(self: @ContractState, hash: felt252, signature: Span<felt252>) -> bool {
      let is_valid_length = signature.len() == 2_u32;

      if !is_valid_length {
        return false;
      }

      check_ecdsa_signature(
        hash, self.public_key.read(), *signature.at(0_u32), *signature.at(1_u32)
      )
    }
  }
}

```

تابع **validate** از is\_valid\_signature\_bool برای اطمینان از اعتبار امضای تراکنش استفاده می کند.

```rust
...

#[starknet::contract]
mod Account {
  ...
  use box::BoxTrait;
  use starknet::get_tx_info;

  ...

  #[external(v0)]
  #[generate_trait]
  impl ProtocolImpl of ProtocolTrait {
    ...

    fn __validate__(self: @ContractState, calls: Array<Call>) -> felt252 {
      self.only_protocol();

      let tx_info = get_tx_info().unbox();
      let tx_hash = tx_info.transaction_hash;
      let signature = tx_info.signature;

      let is_valid = self.is_valid_signature_bool(tx_hash, signature);
      assert(is_valid, 'Account: Incorrect tx signature');  //assert stops transaction execution if signature invalid
      'VALID'
    }
  }

  ...
}

```

### اعتبار سنجی برای توابع Declare and Deploy

تابع **validate\_declare** مسئول تایید امضای تابع declare است. از سوی دیگر، **validate\_deploy** استقرار خلاف واقع را تسهیل می کند، روشی برای استقرار یک قرارداد حساب بدون مرتبط کردن آن با یک آدرس توزیع کننده خاص.

برای ساده‌سازی فرآیند اعتبارسنجی، رفتار سه تابع اعتبارسنجی **validate**،**validate\_declare** و **validate\_deploy** را یکسان می‌کنیم. منطق اصلی از **validate** به تابع خصوصی validate\_transaction انتزاع می شود، که سپس توسط دو تابع اعتبارسنجی دیگر فراخوانی می شود.

```rust
...

#[starknet::contract]
mod Account {
  ...

  #[external(v0)]
  #[generate_trait]
  impl ProtocolImpl of ProtocolTrait {

    //The three validation signatures pooled to function similarly
    fn __validate__(self: @ContractState, calls: Array<Call>) -> felt252 {
      self.only_protocol();
      self.validate_transaction()
    }

    fn __validate_declare__(self: @ContractState, class_hash: felt252) -> felt252 {
      self.only_protocol();
      self.validate_transaction()
    }

    fn __validate_deploy__(self: @ContractState, class_hash: felt252, salt: felt252, public_key: felt252) -> felt252 {
      self.only_protocol();
      self.validate_transaction()
    }
  }

  #[generate_trait]
  impl PrivateImpl of PrivateTrait {
    ...

    //Logic extraction from __validate__ to validate_transaction
    fn validate_transaction(self: @ContractState) -> felt252 {
      let tx_info = get_tx_info().unbox();
      let tx_hash = tx_info.transaction_hash;
      let signature = tx_info.signature;

      let is_valid = self.is_valid_signature_bool(tx_hash, signature);
      assert(is_valid, 'Account: Incorrect tx signature');
      'VALID'
    }
  }
}
```

توجه به این نکته مهم است که تابع **validate\_deploy** کلید عمومی را به عنوان آرگومان دریافت می کند. در حالی که این کلید در مرحله سازنده قبل از فراخوانی این تابع گرفته می شود، ارائه آن هنگام شروع تراکنش بسیار مهم است. متناوبا، کلید عمومی را می توان مستقیماً در تابع **validate\_deploy** با دور زدن سازنده استفاده کرد.

### اجرای تراکنش

تابع **execute** در ماژول Account آرایه ای از ساختارهای Call را می پذیرد که امکان عملکرد چند تماس را فراهم می کند. این ویژگی چندین عملیات کاربر را در یک تراکنش جمع می‌کند و تجربه کاربر را افزایش می‌دهد.

```rust
...
#[starknet::contract]
mod Account {
  ...
  #[external(v0)]
  #[generate_trait]
  impl ProtocolImpl of ProtocolTrait {
    fn __execute__(ref self: ContractState, calls: Array<Call>) -> Array<Span<felt252>> { ... }
    ...
  }
}
```

ساختار داده تماس شامل اطلاعات لازم برای عملیات یک کاربر است.

```rust
#[derive(Drop, Serde)]
struct Call {
  to: ContractAddress,
  selector: felt252,
  calldata: Array<felt252>
}
```

برای مدیریت تماس های فردی، یک تابع خصوصی execute\_single\_call تعریف شده است. این سیستم از call\_contract\_syscall سطح پایین برای فراخوانی مستقیم تابع قرارداد هوشمند دیگر استفاده می کند.

```rust
...
#[starknet::contract]
mod Account {
  ...
  use starknet::call_contract_syscall;

  #[generate_trait]
  impl PrivateImpl of PrivateTrait {
    ...
    fn execute_single_call(self: @ContractState, call: Call) -> Span<felt252> {
      let Call{to, selector, calldata} = call;
      call_contract_syscall(to, selector, calldata.span()).unwrap_syscall()
    }
  }
}
```

برای رسیدگی به تماس های متعدد، تابع execute\_multiple\_calls روی آرایه Call تکرار می شود و آرایه ای از پاسخ ها را برمی گرداند.

```rust
...
#[starknet::contract]
mod Account {
  ...
  #[generate_trait]
  impl PrivateImpl of PrivateTrait {
    ...
    fn execute_multiple_calls(self: @ContractState, mut calls: Array<Call>) -> Array<Span<felt252>> {
      let mut res = ArrayTrait::new();
      loop {
        match calls.pop_front() {
          Option::Some(call) => {
            let _res = self.execute_single_call(call);
            res.append(_res);
          },
          Option::None(_) => {
            break ();
          },
        };
      };
      res
    }
  }
}
```

در نهایت، تابع **execute** اصلی از این توابع کمکی برای پردازش آرایه ساختارهای فراخوانی استفاده می کند:

```rust
...
#[starknet::contract]
mod Account {
  ...
  #[external(v0)]
  #[generate_trait]
  impl ProtocolImpl of ProtocolTrait {
    fn __execute__(ref self: ContractState, calls: Array<Call>) -> Array<Span<felt252>> {
      self.only_protocol();
      self.execute_multiple_calls(calls)
    }
    ...
  }
  ...
}
```

### پشتیبانی از نسخه تراکنش

برای تطبیق با تکامل Starknet و قابلیت های پیشرفته آن، یک سیستم نسخه سازی برای تراکنش ها معرفی شد. این امر سازگاری با عقب را تضمین می کند و به ساختارهای تراکنش قدیمی و جدید اجازه می دهد تا به طور همزمان عمل کنند.

برای سادگی در این آموزش، قرارداد حساب برای پشتیبانی از آخرین نسخه‌های هر نوع تراکنش طراحی شده است:

* نسخه 1 برای تراکنش های فراخوانی
* نسخه 1 برای تراکنش های deploy\_account
* نسخه 2 برای اعلام تراکنش ها

این نسخه های پشتیبانی شده به طور منطقی در یک ماژول به نام SUPPORTED\_TX\_VERSION گروه بندی می شوند:

```rust
...

mod SUPPORTED_TX_VERSION {
  const DEPLOY_ACCOUNT: felt252 = 1;
  const DECLARE: felt252 = 2;
  const INVOKE: felt252 = 1;
}

#[starknet::contract]
mod Account { ... }
```

برای اطمینان از اینکه فقط آخرین نسخه های تراکنش پردازش می شوند، یک تابع خصوصی only\_supported\_tx\_version معرفی شده است. این تابع نسخه تراکنش ورودی را در برابر نسخه های پشتیبانی شده بررسی می کند. اگر عدم تطابق وجود داشته باشد، اجرای تراکنش با یک خطای ادعا متوقف می شود.

توابع اصلی **execute**، **validate**، **validate\_declare** و **validate\_deploy** از این بررسی نسخه استفاده می کنند تا اطمینان حاصل شود که فقط نسخه های تراکنش پشتیبانی شده پردازش می شوند.

```rust
...

#[starknet::contract]
mod Account {
  ...
  use super::SUPPORTED_TX_VERSION;

  ...

  #[external(v0)]
  #[generate_trait]
  impl ProtocolImpl of ProtocolTrait {
    fn __execute__(ref self: ContractState, calls: Array<Call>) -> Array<Span<felt252>> {
      self.only_protocol();
      self.only_supported_tx_version(SUPPORTED_TX_VERSION::INVOKE);
      self.execute_multiple_calls(calls)
    }

    fn __validate__(self: @ContractState, calls: Array<Call>) -> felt252 {
      self.only_protocol();
      self.only_supported_tx_version(SUPPORTED_TX_VERSION::INVOKE);
      self.validate_transaction()
    }

    fn __validate_declare__(self: @ContractState, class_hash: felt252) -> felt252 {
      self.only_protocol();
      self.only_supported_tx_version(SUPPORTED_TX_VERSION::DECLARE);
      self.validate_transaction()
    }

    fn __validate_deploy__(self: @ContractState, class_hash: felt252, salt: felt252, public_key: felt252) -> felt252 {
      self.only_protocol();
      self.only_supported_tx_version(SUPPORTED_TX_VERSION::DEPLOY_ACCOUNT);
      self.validate_transaction()
    }
  }

  #[generate_trait]
  impl PrivateImpl of PrivateTrait {
    ...

    fn only_supported_tx_version(self: @ContractState, supported_tx_version: felt252) {
      let tx_info = get_tx_info().unbox();
      let version = tx_info.version;
      assert(
        version == supported_tx_version,
        'Account: Unsupported tx version'
      );
    }
  }
}
```

### مدیریت تراکنش های شبیه سازی شده

در Starknet، معاملات را می توان برای تخمین گاز بدون اجرای واقعی شبیه سازی کرد. ابزارهایی مانند Starkli برای این منظور یک پرچم فقط تخمینی ارائه می‌کنند که به Sequencer سیگنال می‌دهد تا تراکنش را شبیه‌سازی کند و هزینه تخمینی را برگرداند.

برای تمایز بین تراکنش‌های واقعی و شبیه‌سازی‌شده، نسخه یک تراکنش شبیه‌سازی‌شده با ۲^۱۲۸ از همتای واقعی‌اش جبران می‌شود. به عنوان مثال، یک تراکنش اظهار شبیه سازی شده دارای نسخه 2^128 + 2 است اگر آخرین نسخه تراکنش اعلامی معمولی 2 باشد.

تابع only\_supported\_tx\_version برای تشخیص هر دو نسخه واقعی و شبیه سازی شده تنظیم شده است و از پردازش دقیق برای هر دو نوع اطمینان حاصل می کند.

```rust
...

#[starknet::contract]
mod Account {
  ...
  //Represents simulated transactions
  const SIMULATE_TX_VERSION_OFFSET: felt252 = 340282366920938463463374607431768211456; // 2**128

  ...

  #[generate_trait]
  impl PrivateImpl of PrivateTrait {
    ...
    fn only_supported_tx_version(self: @ContractState, supported_tx_version: felt252) {
      let tx_info = get_tx_info().unbox();
      let version = tx_info.version;
      assert(
        version == supported_tx_version ||
        version == SIMULATE_TX_VERSION_OFFSET + supported_tx_version,
        'Account: Unsupported tx version'
      );
    }
  }
}

```

## Introspection

Introspection allows an account contract to self-identify using the SRC-5 standard.

```rust
trait ISRC5 {
  fn supports_interface(interface_id: felt252) -> bool;
}
```

An account contract should return `true` for the `supports_interface` function when provided with the specific `interface_id` of `1270010605630597976495846281167968799381097569185364931397797212080166453709`. The reason for using this specific identifier is explained in the previous subchapter.

The `supports_interface` function has been added to the public interface of the account contract to facilitate external queries by other smart contracts.

```rust
...

#[starknet::interface]
trait IAccount<T> {
  ...
  fn supports_interface(self: @T, interface_id: felt252) -> bool;
}

#[starknet::contract]
mod Account {
  ...
  const SRC6_TRAIT_ID: felt252 = 1270010605630597976495846281167968799381097569185364931397797212080166453709;

  ...

  #[external(v0)]
  impl AccountImpl of super::IAccount<ContractState> {
    ...
    fn supports_interface(self: @ContractState, interface_id: felt252) -> bool {
      interface_id == SRC6_TRAIT_ID
    }
  }
  ...
}
```

## Public Key Accessibility

For enhanced transparency and debugging purposes, it's recommended to make the public key of the account contract's signer accessible. This allows users to verify the correct deployment of the account contract by comparing the stored public key with the signer's public key offline.

```rust
...

#[starknet::contract]
mod Account {
  ...

  #[external(v0)]
  impl AccountImpl of IAccount<ContractState> {
    ...
    fn public_key(self: @ContractState) -> felt252 {
      self.public_key.read()
    }
  }
}
```

## Final Implementation

We now have a fully functional account contract. Here's the final implementation;

```rust
use starknet::account::Call;

mod SUPPORTED_TX_VERSION {
  const DEPLOY_ACCOUNT: felt252 = 1;
  const DECLARE: felt252 = 2;
  const INVOKE: felt252 = 1;
}

#[starknet::interface]
trait IAccount<T> {
  fn is_valid_signature(self: @T, hash: felt252, signature: Array<felt252>) -> felt252;
  fn supports_interface(self: @T, interface_id: felt252) -> bool;
  fn public_key(self: @T) -> felt252;
}

#[starknet::contract]
mod Account {
  use super::{Call, IAccount, SUPPORTED_TX_VERSION};
  use starknet::{get_caller_address, call_contract_syscall, get_tx_info, VALIDATED};
  use zeroable::Zeroable;
  use array::{ArrayTrait, SpanTrait};
  use ecdsa::check_ecdsa_signature;
  use box::BoxTrait;

  const SIMULATE_TX_VERSION_OFFSET: felt252 = 340282366920938463463374607431768211456; // 2**128
  const SRC6_TRAIT_ID: felt252 = 1270010605630597976495846281167968799381097569185364931397797212080166453709; // hash of SNIP-6 trait

  #[storage]
  struct Storage {
    public_key: felt252
  }

  #[constructor]
  fn constructor(ref self: ContractState, public_key: felt252) {
    self.public_key.write(public_key);
  }

  #[external(v0)]
  impl AccountImpl of IAccount<ContractState> {
    fn is_valid_signature(self: @ContractState, hash: felt252, signature: Array<felt252>) -> felt252 {
      let is_valid = self.is_valid_signature_bool(hash, signature.span());
      if is_valid { VALIDATED } else { 0 }
    }

    fn supports_interface(self: @ContractState, interface_id: felt252) -> bool {
      interface_id == SRC6_TRAIT_ID
    }

    fn public_key(self: @ContractState) -> felt252 {
      self.public_key.read()
    }
  }

  #[external(v0)]
  #[generate_trait]
  impl ProtocolImpl of ProtocolTrait {
    fn __execute__(ref self: ContractState, calls: Array<Call>) -> Array<Span<felt252>> {
      self.only_protocol();
      self.only_supported_tx_version(SUPPORTED_TX_VERSION::INVOKE);
      self.execute_multiple_calls(calls)
    }

    fn __validate__(self: @ContractState, calls: Array<Call>) -> felt252 {
      self.only_protocol();
      self.only_supported_tx_version(SUPPORTED_TX_VERSION::INVOKE);
      self.validate_transaction()
    }

    fn __validate_declare__(self: @ContractState, class_hash: felt252) -> felt252 {
      self.only_protocol();
      self.only_supported_tx_version(SUPPORTED_TX_VERSION::DECLARE);
      self.validate_transaction()
    }

    fn __validate_deploy__(self: @ContractState, class_hash: felt252, salt: felt252, public_key: felt252) -> felt252 {
      self.only_protocol();
      self.only_supported_tx_version(SUPPORTED_TX_VERSION::DEPLOY_ACCOUNT);
      self.validate_transaction()
    }
  }

  #[generate_trait]
  impl PrivateImpl of PrivateTrait {
    fn only_protocol(self: @ContractState) {
      let sender = get_caller_address();
      assert(sender.is_zero(), 'Account: invalid caller');
    }

    fn is_valid_signature_bool(self: @ContractState, hash: felt252, signature: Span<felt252>) -> bool {
      let is_valid_length = signature.len() == 2_u32;

      if !is_valid_length {
        return false;
      }

      check_ecdsa_signature(
        hash, self.public_key.read(), *signature.at(0_u32), *signature.at(1_u32)
      )
    }

    fn validate_transaction(self: @ContractState) -> felt252 {
      let tx_info = get_tx_info().unbox();
      let tx_hash = tx_info.transaction_hash;
      let signature = tx_info.signature;

      let is_valid = self.is_valid_signature_bool(tx_hash, signature);
      assert(is_valid, 'Account: Incorrect tx signature');
      VALIDATED
    }

    fn execute_single_call(self: @ContractState, call: Call) -> Span<felt252> {
      let Call{to, selector, calldata} = call;
      call_contract_syscall(to, selector, calldata.span()).unwrap()
    }

    fn execute_multiple_calls(self: @ContractState, mut calls: Array<Call>) -> Array<Span<felt252>> {
      let mut res = ArrayTrait::new();
      loop {
        match calls.pop_front() {
          Option::Some(call) => {
            let _res = self.execute_single_call(call);
            res.append(_res);
          },
          Option::None(_) => {
            break ();
          },
        };
      };
      res
    }

    fn only_supported_tx_version(self: @ContractState, supported_tx_version: felt252) {
      let tx_info = get_tx_info().unbox();
      let version = tx_info.version;
      assert(
        version == supported_tx_version ||
        version == SIMULATE_TX_VERSION_OFFSET + supported_tx_version,
        'Account: Unsupported tx version'
      );
    }
  }
}
```

## Recap

Account Contract Creation Recap:

* SNIP-6 Implementation
  * The account contract is designed to adhere to the `ISRC6` trait, which dictates the structure of an account contract.
* Protecting Protocol-Only Functions
  * `__validate__` and `__execute__` functions were restricted to be accessed only by the Starknet protocol.
  * `is_valid_signature` was exposed for external interactions.
  * Private function `only_protocol` was introduced to enforce this restriction.
* Signature Validation
  * Public key associated with signer of account contract was stored to facilitate transaction signature validation.
  * `constructor` method was defined to capture the public key's value during deployment.
  * The `is_valid_signature` function was implemented to validate signatures, returning `VALID` or `0`.
  * The helper function `is_valid_signature_bool` was introduced to return a boolean result.
* Validation of Declare and Deploy Functions
  * The `__validate_declare__` function was setup to validate signature of `declare` function.
  * The `__validate_deploy__` function was designed for counterfactual deployment.
  * The core validation logic was abstracted to a private function named `validate_transaction`.
* Transaction Execution
  * Introduced multicall functionality via the `__execute__` function.
  * Implemented `execute_single_call` and `execute_multiple_calls` to manage individual and multiple calls respectively.
* Transaction Version Support
  * Implemented a versioning system to ensure backward compatibility with evolving Starknet functionalities.
  * Created `SUPPORTED_TX_VERSION` module to define supported versions for various transaction types.
  * Introduced `only_supported_tx_version` to validate transaction versions.
* Handling Simulated Transactions
  * Adjusted the `only_supported_tx_version` function to recognize both actual and simulated transaction versions.
* Introspection
  * Enabled the account contract to self-identify using the SRC-5 standard.
  * The `supports_interface` function was added to the public interface for external queries about the contract's capabilities.
* Public Key Accessibility
  * Enhanced transparency by making the public key of the account contract's signer accessible.
* Final Implementation
  * Final Implementation of the account contract.

Coming up, we'll use Starkli to deploy to testnet the account created, and use it to interact with other smart contracts.
