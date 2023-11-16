# Foundry Forge: تست 🚧

Starknet Foundry ابزاری است که برای آزمایش و توسعه قراردادهای Starknet طراحی شده است. این یک اقتباس از Ethereum Foundry برای Starknet است که هدف آن تسریع روند توسعه است.

این پروژه از دو جزء اصلی تشکیل شده است:

* فورج: ابزار آزمایشی مخصوص قراردادهای قاهره. این ابزار به عنوان یک تست کننده عمل می کند و دارای ویژگی هایی است که برای بهبود فرآیند تست شما طراحی شده اند. آزمون ها مستقیماً در قاهره نوشته می شوند و نیازی به زبان های برنامه نویسی دیگر را از بین می برند. علاوه بر این، پیاده‌سازی Forge از Rust استفاده می‌کند که منعکس‌کننده زبان انتخابی Ethereum Foundry است.
* Cast: این ابزار به عنوان یک ابزار DevOps برای Starknet عمل می کند و در ابتدا از یک سری دستورات برای ارتباط با Starknet پشتیبانی می کند. در آینده، Cast قصد دارد اسکریپت‌های استقرار را برای قراردادها و سایر عملکردهای DevOps ارائه دهد.

### ساختن

صرف استقرار قراردادها پایان بازی نیست. ابزارهای زیادی در گذشته این قابلیت را ارائه کرده اند. Forge خود را با میزبانی یک نمونه ماشین مجازی Cairo متمایز می کند و امکان اجرای متوالی آزمایش ها را فراهم می کند. از Scarb برای تدوین قرارداد استفاده می کند.

برای استفاده از Forge، توابع تست را تعریف کنید و آنها را با ویژگی های تست برچسب بزنید. کاربران می‌توانند عملکردهای مستقل قاهره را آزمایش کنند یا قراردادها، توزیع‌کننده‌ها و تعاملات قراردادی آزمایشی را در زنجیره ادغام کنند.

### استفاده از خط فرمان snForge

این بخش شما را از طریق ابزار خط فرمان Starknet Foundry snforge راهنمایی می کند. نحوه راه اندازی یک پروژه جدید، کامپایل کد و اجرای آزمایش ها را بیاموزید.

برای شروع یک پروژه جدید با Starknet Foundry، از دستور --init استفاده کنید و نام پروژه خود را جایگزین project\_name کنید.

```shell
snforge --init project_name
```

هنگامی که پروژه را راه اندازی کردید، طرح آن را بررسی کنید:

```shell
cd project_name
tree . -L 1
```

ساختار پروژه به شرح زیر است:

```shell
.
├── README.md
├── Scarb.toml
├── src
└── tests
```

* src/ کد منبع قرارداد شما را نگه می دارد.
* tests/ محل فایل های تست شما است.
* Scarb.toml برای پیکربندی پروژه و snforge است.

مطمئن شوید که تولید کد casm در فایل Scarb.toml فعال است:

```shell
# ...
[[target.starknet-contract]]
casm = true
# ...
```

برای اجرای تست ها با استفاده از snforge:

```shell
snforge

Collected 2 test(s) from the `test_name` package
Running 0 test(s) from `src/`
Running 2 test(s) from `tests/`
[PASS] tests::test_contract::test_increase_balance
[PASS] tests::test_contract::test_cannot_increase_balance_with_zero_value
Tests: 2 passed, 0 failed, 0 skipped
```

#### ادغام snforge با پروژه های Scarb موجود

برای کسانی که پروژه Scarb دارند و مایل به ترکیب snforge هستند، مطمئن شوید که بسته snforge\_std به عنوان یک وابستگی اعلام شده است. خط زیر را در بخش \[وابستگی ها] Scarb.toml خود وارد کنید:

```shell
# ...
[dependencies]
snforge_std = { git = "https://github.com/foundry-rs/starknet-foundry.git", tag = "[VERSION]" }
```

اطمینان حاصل کنید که نسخه برچسب با نسخه snforge شما مطابقت دارد. برای تأیید نسخه snforge:

```sh
snforge --version
```

یا با استفاده از دستور scarb این وابستگی را اضافه کنید:

```shell
scarb add snforge_std --git https://github.com/foundry-rs/starknet-foundry.git --tag VERSION
```

با این مراحل، پروژه Scarb موجود شما اکنون آماده snforge است.

### تست با snforge

از دستور snforge Starknet Foundry برای اجرای موثر تست ها استفاده کنید.

### اجرای تست ها

به دایرکتوری بسته بروید و این دستور را برای اجرای تست ها صادر کنید:

```shell
snforge
```

خروجی نمونه ممکن است شبیه به:

```shell
Collected 3 test(s) from `package_name` package
Running 3 test(s) from `src/`
[PASS] package_name::executing
[PASS] package_name::calling
[PASS] package_name::calling_another
Tests: 3 passed, 0 failed, 0 skipped
```

#### تست های فیلتر

تست های خاصی را با استفاده از یک رشته فیلتر پس از دستور snforge اجرا کنید. آزمایش‌هایی که با فیلتر مطابقت دارند بر اساس مسیر درختی ماژول مطلق آن‌ها اجرا می‌شوند:

```shell
$ snforge calling
```

### یک تست خاص را اجرا کنید

از علامت --exact و یک نام آزمون کاملاً واجد شرایط برای اجرای یک آزمون خاص استفاده کنید:

```shell
snforge package_name::calling --exact
```

#### توقف پس از اولین شکست تست

برای توقف پس از اولین شکست تست، پرچم --exit-first را به دستور snforge اضافه کنید:

```shell
snforge --exit-first
```

### مثال پایه

مثال ارائه شده در زیر نحوه آزمایش قرارداد Starknet با استفاده از snforge را نشان می دهد.

```rust
#[starknet::interface]
trait IHelloStarknet<TContractState> {
    fn increase_balance(ref self: TContractState, amount: felt252);
    fn get_balance(self: @TContractState) -> felt252;
}

#[starknet::contract]
mod HelloStarknet {
    #[storage]
    struct Storage {
        balance: felt252,
    }

    #[external(v0)]
    impl HelloStarknetImpl of super::IHelloStarknet<ContractState> {
        // Increases the balance by the specified amount.
        fn increase_balance(ref self: ContractState, amount: felt252) {
            self.balance.write(self.balance.read() + amount);
        }

        // Returns the balance.
        fn get_balance(self: @ContractState) -> felt252 {
            self.balance.read()
        }
    }
}
```

به یاد داشته باشید، شناسه زیر mod نشان دهنده نام قرارداد است. در اینجا، نام قرارداد HelloStarknet است.

#### تست را بسازید

در زیر تستی برای قرارداد HelloStarknet وجود دارد. این تست HelloStarknet را مستقر کرده و با عملکردهای آن تعامل دارد:

```rust
use snforge_std::{ declare, ContractClassTrait };

#[test]
fn call_and_invoke() {
    // Declare and deploy the contract
    let contract = declare('HelloStarknet');
    let contract_address = contract.deploy(@ArrayTrait::new()).unwrap();

    // Instantiate a Dispatcher object for contract interactions
    let dispatcher = IHelloStarknetDispatcher { contract_address };

    // Invoke a contract's view function
    let balance = dispatcher.get_balance();
    assert(balance == 0, 'balance == 0');

    // Invoke another function to modify the storage state
    dispatcher.increase_balance(100);

    // Validate the transaction's effect
    let balance = dispatcher.get_balance();
    assert(balance == 100, 'balance == 100');
}
```

برای اجرای تست، دستور snforge را اجرا کنید. خروجی مورد انتظار عبارت است از:

```shell
Collected 1 test(s) from using_dispatchers package
Running 1 test(s) from src/
[PASS] using_dispatchers::call_and_invoke
Tests: 1 passed, 0 failed, 0 skipped
```
