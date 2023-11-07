# مقدمه ای بر استارکلی، اسکارب و کاتانا

در این فصل، نحوه کامپایل، استقرار و تعامل با قرارداد هوشمند Starknet که در قاهره با استفاده از starkli، scarb و katana نوشته شده است را خواهید آموخت.

ابتدا تأیید کنید که دستورات زیر روی سیستم شما کار می کنند. اگر اینطور نیست، به نصب اولیه در این فصل مراجعه کنید.

```bash
    scarb --version  # For Cairo code compilation
    starkli --version  # To interact with Starknet
    katana --version # To declare and deploy on local development
```

### \[اختیاری] بررسی نسخه های کامپایلر پشتیبانی شده

اگر در طول فرآیند اعلام یا استقرار مشکلاتی پیش آمد، اطمینان حاصل کنید که نسخه کامپایلر Starkli با نسخه کامپایلر Scarb مطابقت دارد.

برای بررسی نسخه های کامپایلر که Starkli پشتیبانی می کند، اجرا کنید:

```bash
starkli declare --help

```

لیستی از نسخه های کامپایلر ممکن را در زیر پرچم --compiler-version مشاهده خواهید کرد.

```bash
    ...
    --compiler-version <COMPILER_VERSION>
              Statically-linked Sierra compiler version [possible values: [COMPILER VERSIONS]]]
    ...
```

توجه داشته باشید: نسخه کامپایلر Scarb ممکن است با Starkli مطابقت نداشته باشد. برای تأیید نسخه Scarb:

```bash
    scarb --version
```

خروجی نسخه‌های scarb، cairo و sierra را نمایش می‌دهد:

```bash
    scarb <SCARB VERSION>
    cairo: <COMPILER VERSION>
    sierra: <SIERRA VERSION>
```

اگر نسخه ها مطابقت ندارند، نسخه ای از Scarb را که با Starkli سازگار است را نصب کنید. مخزن GitHub Scarb را برای نسخه های قبلی مرور کنید.

برای نصب یک نسخه خاص مانند 2.3.0، اجرا کنید:

```bash
    curl --proto '=https' --tlsv1.2 -sSf https://docs.swmansion.com/scarb/install.sh | sh -s -- -v 2.3.0
```

### ساخت یک قرارداد هوشمند Starknet

با شروع یک پروژه Scarb شروع کنید:

```bash
scarb new my_contract
```

### متغیرهای محیط و فایل Scarb.toml را پیکربندی کنید

پروژه my\_contract را مرور کنید. ساختار آن به صورت زیر است:

```bash
    src/
      lib.cairo
    .gitignore
    Scarb.toml
```

فایل Scarb.toml را اصلاح کنید تا وابستگی starknet را یکپارچه کرده و هدف قرارداد starknet را معرفی کنید:

```toml
    [dependencies]
    starknet = ">=2.3.0"

    [[target.starknet-contract]]
```

برای اجرای ساده دستور Starkli، متغیرهای محیطی را ایجاد کنید. دو متغیر اصلی ضروری است:

* یکی برای حساب شما، یک حساب از قبل تامین شده در شبکه توسعه محلی
* دیگری برای تعیین شبکه، به‌ویژه کاتانا محلی محلی

در پوشه src/، یک فایل .env با موارد زیر ایجاد کنید:

```bash
export STARKNET_ACCOUNT=katana-0
export STARKNET_RPC=http://0.0.0.0:5050
```

این تنظیمات عملیات فرمان Starkli را ساده می کند.

### اعلام قراردادهای هوشمند در استارک نت

استقرار یک قرارداد هوشمند Starknet به دو مرحله اولیه نیاز دارد:

* کد قرارداد را اعلام کنید.
* یک نمونه از آن کد اعلام شده را مستقر کنید.

با فایل src/lib.cairo شروع کنید که یک الگوی اساسی ارائه می کند. محتویات آن را بردارید و موارد زیر را وارد کنید:

```rust
#[starknet::contract]
mod hello {
    #[storage]
    struct Storage {
        name: felt252,
    }

    #[constructor]
    fn constructor(ref self: ContractState, name: felt252) {
        self.name.write(name);
    }

    #[external(v0)]
        fn get_name(self: @ContractState) -> felt252 {
            self.name.read()
        }
    #[external(v0)]
        fn set_name(ref self: ContractState, name: felt252) {
            let previous = self.name.read();
            self.name.write(name);
        }
}
```

این قرارداد هوشمند ابتدایی به عنوان نقطه شروع عمل می کند.

قرارداد را با کامپایلر Scarb جمع آوری کنید. اگر Scarb نصب نشده است، به بخش نصب مراجعه کنید.

```bash
scarb build
```

دستور بالا منجر به یک قرارداد کامپایل شده تحت target/dev/ به نام "my\_contract\_hello.contract\_class.json" می شود (برای جزئیات بیشتر، زیرفصل Scarb را بررسی کنید).

پس از جمع آوری قرارداد هوشمند، زمان آن رسیده است که آن را با Starkli و katana اعلام کنید. ابتدا مطمئن شوید که پروژه شما متغیرهای محیطی را تایید می کند:

```bash
source .env
```

بعد، Katana را راه اندازی کنید. در یک ترمینال جداگانه، اجرا کنید (جزئیات بیشتر در فصل فرعی Katan):

```bash
katana
```

برای اعلام قرارداد، موارد زیر را اجرا کنید:

```bash
starkli declare target/dev/my_contract_hello.contract_class.json
```

با «خطا: کلاس قرارداد نامعتبر» مواجه هستید؟ این نشان دهنده عدم تطابق نسخه بین کامپایلر Scarb و Starkli است. برای همگام سازی نسخه ها به مراحل قبلی مراجعه کنید. به طور معمول، Starkli از نسخه های کامپایلر تایید شده توسط mainnet پشتیبانی می کند، حتی اگر جدیدترین نسخه Scarb سازگار نباشد.

پس از اجرای موفقیت آمیز دستور، یک هش کلاس قرارداد دریافت خواهید کرد: این هش منحصر به فرد به عنوان شناسه کلاس قرارداد شما در Starknet عمل می کند. مثلا:

```bash
Class hash declared: 0x00bfb49ff80fd7ef5e84662d6d256d49daf75e0c5bd279b20a786f058ca21418
```

این هش را به عنوان آدرس کلاس قرارداد در نظر بگیرید.

اگر می‌خواهید یک کلاس قرارداد از قبل موجود را اعلام کنید، نگران نباشید. فقط ادامه بده ممکن است ببینید:

```bash
Not declaring class as its already declared. Class hash:
0x00bfb49ff80fd7ef5e84662d6d256d49daf75e0c5bd279b20a786f058ca21418
```

### استقرار قراردادهای هوشمند Starknet

برای استقرار یک قرارداد هوشمند در devnet محلی katana، از دستور زیر استفاده کنید. در درجه اول نیاز به:

1. هش کلاس قرارداد شما.
2. سازنده آرگومان های مورد نیاز قرارداد شما (در مثال ما، نامی از نوع felt252).

در اینجا ساختار فرمان آمده است:

```bash
    starkli deploy \
        <CLASS_HASH> \
        <CONSTRUCTOR_INPUTS>
```

توجه داشته باشید که ورودی های سازنده در قالب احساس هستند. بنابراین باید یک رشته کوتاه را به فرمت felt252 تبدیل کنیم. برای این کار می توانیم از دستور to-cairo-string استفاده کنیم:

```bash
    starkli to-cairo-string <STRING>
```

در این مورد، از رشته "starknetbook" به عنوان نام استفاده می کنیم:

```bash
    starkli to-cairo-string starknetbook
```

خروجی:

```bash
    0x737461726b6e6574626f6f6b
```

اکنون با استفاده از هش کلاس و ورودی سازنده آن را مستقر کنید:

```bash
    starkli deploy \
        0x00bfb49ff80fd7ef5e84662d6d256d49daf75e0c5bd279b20a786f058ca21418 \
        0x737461726b6e6574626f6f6b
```

پس از اجرا، انتظار خروجی مشابه زیر را داشته باشید:

```bash
    Deploying class 0x00bfb49ff80fd7ef5e84662d6d256d49daf75e0c5bd279b20a786f058ca21418 with salt 0x054645c0d1e766ddd927b3bde150c0a3dc0081af7fb82160c1582e05f6018794...
    The contract will be deployed at address 0x07cdd583619462c2b14532eddb2b169b8f8d94b63bfb5271dae6090f95147a44
    Contract deployment transaction: 0x00413d9638fecb75eb07593b5c76d13a68e4af7962c368c5c2e810e7a310d54c
    Contract deployed: 0x07cdd583619462c2b14532eddb2b169b8f8d94b63bfb5271dae6090f95147a44
```

تعامل با قراردادهای Starknet

با استفاده از Starkli، می توانید از طریق دو روش اصلی با قراردادهای هوشمند تعامل داشته باشید:

* تماس: برای توابع فقط خواندنی.
* invoke: برای توابعی که حالت را تغییر می دهند.

### خواندن داده ها با تماس

فرمان فراخوانی به شما اجازه می دهد تا عملکردهای قرارداد را بدون انجام تراکنش پرس و جو کنید. به عنوان مثال، اگر می خواهید مالک قرارداد فعلی را با استفاده از تابع get\_name تعیین کنید، که به هیچ آرگومان نیاز ندارد:

```bash
    starkli call \
        <CONTRACT_ADDRESS> \
        get_name
```

آدرس قرارداد خود را جایگزین \<CONTRACT\_ADDRESS> کنید. این فرمان آدرس مالک را که در ابتدا در زمان استقرار قرارداد تنظیم شده بود، برمی گرداند:

```bash
    [
        "0x0000000000000000000000000000000000000000737461726b6e6574626f6f6b"
    ]
```

اما این خروجی طولانی چیست؟ در Starknet، ما از نوع داده felt252 برای نمایش رشته ها استفاده می کنیم. این را می توان در نمایش رشته آن رمزگشایی کرد:

```bash
starkli parse-cairo-string 0x737461726b6e6574626f6f6b
```

نتیجه:

```bash
starknetbook
```

### اصلاح وضعیت قرارداد با فراخوانی

برای تغییر وضعیت قرارداد، از دستور invoke استفاده کنید. به عنوان مثال، اگر می خواهید فیلد نام را در فضای ذخیره سازی به روز کنید، از تابع set\_name استفاده کنید:

```bash
    starkli invoke \
        <CONTRACT_ADDRESS> \
        set_name \
        <felt252>
```

جایی که:

* \<CONTRACT\_ADDRESS> آدرس قرارداد شما است.
* مقدار جدید برای فیلد نام، در قالب felt252 است.

به عنوان مثال، برای به روز رسانی نام به "Omar"، ابتدا رشته "Omar" را به نمایش Fem252 آن تبدیل کنید:

```bash
    starkli to-cairo-string Omar
```

This will return:

```bash
    0x4f6d6172
```

اکنون با دستور invoke ادامه دهید:

```bash
    starkli invoke 0x07cdd583619462c2b14532eddb2b169b8f8d94b63bfb5271dae6090f95147a44 set_name 0x4f6d6172
```

براوو! شما به طرز ماهرانه ای قرارداد Starknet خود را تغییر داده و با آن ارتباط برقرار کرده اید.
