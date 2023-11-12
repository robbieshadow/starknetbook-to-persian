# بازیگران Foundry: Starknet CLI Interaction

Starknet Foundry ابزاری است که برای آزمایش و توسعه قراردادهای Starknet طراحی شده است. این یک اقتباس از Ethereum Foundry برای Starknet است که هدف آن تسریع روند توسعه است.

این پروژه از دو جزء اصلی تشکیل شده است:

* Forge: یک ابزار آزمایشی مخصوص قراردادهای قاهره. این ابزار به عنوان یک تست کننده عمل می کند و دارای ویژگی هایی است که برای بهبود فرآیند تست شما طراحی شده اند. آزمون ها مستقیماً در قاهره نوشته می شوند و نیازی به زبان های برنامه نویسی دیگر را از بین می برند. علاوه بر این، پیاده‌سازی Forge از Rust استفاده می‌کند که منعکس‌کننده زبان انتخابی Ethereum Foundry است.
* Cast: این ابزار به عنوان یک ابزار DevOps برای Starknet عمل می کند و در ابتدا از یک سری دستورات برای ارتباط با Starknet پشتیبانی می کند. در آینده، Cast قصد دارد اسکریپت‌های استقرار را برای قراردادها و سایر عملکردهای DevOps ارائه دهد.

### قالب

Cast، Command Line Interface (CLI) را برای starknet فراهم می کند، در حالی که Forge آدرس ها را آزمایش می کند. Cast که در Rust نوشته شده است، از Starknet Rust استفاده می کند و با Scarb ادغام می شود. این ادغام اجازه می دهد تا برای مشخصات آرگومان در Scarb.toml، فرآیند را ساده تر کند.

sncast تعامل با قراردادهای هوشمند را ساده می کند و تعداد دستورات لازم را در مقایسه با استفاده از starkli به تنهایی کاهش می دهد.

در این بخش به sncast می پردازیم.

### مرحله 1: نمونه قرارداد هوشمند

نمونه کد زیر از starknet foundry تهیه شده است. شما می توانید اصل را در اینجا پیدا کنید.

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
        fn increase_balance(ref self: ContractState, amount: felt252) {
            assert(amount != 0, 'amount cannot be 0');
            self.balance.write(self.balance.read() + amount);
        }
        fn get_balance(self: @ContractState) -> felt252 {
            self.balance.read()
        }
    }
}
```

قبل از تعامل با این نمونه قرارداد هوشمند، بسیار مهم است که عملکرد آن را با استفاده از snforge آزمایش کنید تا از یکپارچگی آن اطمینان حاصل کنید.

در اینجا تست های مرتبط آورده شده است:

```rust
#[cfg(test)]
mod tests {
    use learnsncast::IHelloStarknetDispatcherTrait;
    use snforge_std::{declare, ContractClassTrait};
    use super::{IHelloStarknetDispatcher};

    #[test]
    fn call_and_invoke() {
        // Declare and deploy a contract
        let contract = declare('HelloStarknet');
        let contract_address = contract.deploy(@ArrayTrait::new()).unwrap();

        // Create a Dispatcher object for interaction with the deployed contract
        let dispatcher = IHelloStarknetDispatcher { contract_address };

        // Query a contract view function
        let balance = dispatcher.get_balance();
        assert(balance == 0, 'balance == 0');

        // Invoke a contract function to mutate state
        dispatcher.increase_balance(100);

        // Verify the transaction's effect
        let balance = dispatcher.get_balance();
        assert(balance == 100, 'balance == 100');
    }
}
```

در صورت نیاز، قطعه کد ارائه شده را در فایل lib.cairo پروژه جدید scarb خود کپی کنید.

برای اجرای تست ها مراحل زیر را دنبال کنید:

1. اطمینان حاصل کنید که snforge به عنوان یک وابستگی در فایل Scarb.toml شما، در زیر وابستگی starknet قرار گرفته است. بخش وابستگی های شما باید به صورت ظاهر شود (حتما از آخرین نسخه snforge و starknet استفاده کنید):

```txt
starknet = "2.1.0-rc2"
snforge_std = { git = "https://github.com/foundry-rs/starknet-foundry.git", tag = "v0.7.1" }
```

2. دستور را اجرا کنید:

```sh
snforge
```

نکته: برای تست به جای دستور scarb test از snforge استفاده کنید. تست ها برای استفاده از توابع snforge\_std تنظیم شده اند. انجام تست اسکارب باعث خطا می شود.

### مرحله 2: راه اندازی Starknet Devnet

برای این راهنما، تمرکز بر استفاده از starknet-devnet است. اگر از کاتانا استفاده کرده اید، لطفا محتاط باشید زیرا ممکن است ناهماهنگی وجود داشته باشد. اگر devnet را پیکربندی نکرده‌اید، برای راه‌اندازی سریع این راهنما را دنبال کنید.

برای راه اندازی starknet devnet از دستور زیر استفاده کنید:

```sh
starknet-devnet
```

پس از راه اندازی موفقیت آمیز، باید پاسخی مشابه دریافت کنید:

```sh
Predeployed FeeToken
Address: 0x49d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7
Class Hash: 0x6a22bf63c7bc07effa39a25dfbd21523d211db0100a0afd054d172b81840eaf
Symbol: ETH

Account #0:
Address: 0x5fd5ef7f4b0e23a44a3670bd84f802f6cc37983c7766d562a8d4d72bb8360ba
Public key: 0x6bd5d1d46a7f603f1106824a3b276fdb52168f55b595ba7ff6b2ded390161cd
Private key: 0xc12927df61303656b3c066e65eda0acc
...
...
...
 * Listening on http://127.0.0.1:5050/ (Press CTRL+C to quit)
```

(توجه: مخفف ... فقط یک مکان برای پاسخ دقیق است. در خروجی واقعی خود، جزئیات کامل را خواهید دید.)

اکنون، شما یک قرارداد هوشمند نوشته اید، آن را آزمایش کرده اید، و starknet devnet را با موفقیت راه اندازی کرده اید.

### در sncast شیرجه بزنید

بیایید sncast را باز کنیم.

به عنوان یک ابزار چند منظوره، سریع ترین راه برای کشف قابلیت های آن از طریق دستور زیر است:

```sh
sncast --help
```

در خروجی، دسته بندی های متمایز را مشاهده خواهید کرد: دستورات و گزینه ها. هر گزینه هم یک نوع مختصر (کوتاه) و یک نوع توصیفی (طولانی) ارائه می دهد.

> نکته: در حالی که هر دو نوع گزینه مفید هستند، ما فرم طولانی را در این راهنما اولویت بندی می کنیم. این انتخاب به وضوح کمک می کند، به خصوص هنگام ساخت دستورات پیچیده.

با کاوش عمیق تر، برای درک دستورات خاصی مانند حساب، می توانید اجرا کنید:

```sh
sncast account help
```

هر دستور فرعی حساب مانند افزودن، ایجاد و استقرار را می توان بیشتر مورد بررسی قرار داد. برای مثال:

```sh
sncast account add --help
```

ساختار لایه ای sncast اطلاعات زیادی را درست در اختیار شما قرار می دهد. مثل داشتن مستندات پویا است. کاوش را به یک عادت تبدیل کنید و همیشه در جریان خواهید بود.

### مرحله 3: استفاده از sncast برای مدیریت حساب

بیایید نحوه استفاده از sncast برای تعامل با قرارداد را بررسی کنیم.

به طور پیش فرض، starknet devnet چندین حساب از پیش مستقر شده ارائه می دهد. اینها حساب هایی هستند که قبلاً در گره ثبت شده اند و با توکن های آزمایشی بارگذاری شده اند (برای هزینه های گاز و تراکنش های مختلف). توسعه دهندگان می توانند مستقیماً با هر قراردادی در گره محلی (به عنوان مثال starknet devnet) از آنها استفاده کنند.

### نحوه استفاده از اکانت های از پیش مستقر شده

برای استفاده از یک حساب از پیش مستقر شده با قرارداد هوشمند، دستور add account را مطابق شکل زیر اجرا کنید:

```sh
sncast [SNCAST_MAIN_OPTIONS] account add [SUBCOMMAND_OPTIONS] --name <NAME> --address <ADDRESS> --private-key <PRIVATE_KEY>
```

اگرچه چندین گزینه می‌توانند فرمان افزودن را همراهی کنند (به عنوان مثال، --name، --address، --class-hash، --deployed، --private-key، --public-key، --salt، --add-profile )، ما برای این تصویر روی چند مورد انتخابی تمرکز خواهیم کرد.یک حساب از starknet-devnet انتخاب کنید، برای نمایش، حساب شماره 0 را انتخاب کرده و اجرا می کنیم:

```sh
sncast --url http://localhost:5050/rpc account add  --name account1 --address 0x5f...60ba --private-key 0xc...0acc --add-profile
```

نکاتی که باید به خاطر بسپارید:

1. \-name - فیلد اجباری.
2. \-address - آدرس حساب ضروری.
3. \-private-key - کلید خصوصی حساب.
4. \-add-profile - اگرچه اختیاری است، اما محوری است. با فعال کردن sncast برای گنجاندن حساب در فایل Scarb.toml خود، می‌توانید چندین حساب را مدیریت کنید و هنگام کار با قرارداد هوشمند خود با استفاده از sncast، تراکنش‌های بین آنها را تسهیل کنید.

اکنون که با استفاده از یک حساب از پیش مستقر شده آشنا شدیم، بیایید به افزودن یک حساب جدید ادامه دهیم.

### ایجاد و استقرار یک حساب کاربری جدید در Starknet Devnet

ایجاد یک حساب کاربری جدید شامل چند مرحله بیشتر از استفاده از یک حساب موجود است، اما در صورت شکسته شدن ساده است. در اینجا مراحل انجام می شود:

1. ایجاد حساب کاربری

برای ایجاد یک حساب کاربری جدید، از (می توانید از sncast account create --help برای مشاهده گزینه های موجود استفاده کنید):

```sh
sncast --url http://localhost:5050/rpc account create --name new_account --class-hash  0x19...8dd6 --add-profile
```

آیا نمی دانید هش --class از کجا می آید؟ در خروجی دستور starknet-devnet در قسمت Predeclared Starknet CLI حساب قابل مشاهده است. مثلا:

```sh
Predeclared Starknet CLI account:
Class hash: 0x195c984a44ae2b8ad5d49f48c0aaa0132c42521dcfc66513530203feca48dd6
```

2. تامین مالی حساب

برای تامین مالی حساب جدید، آدرس موجود در دستور زیر را با آدرس جدید خود جایگزین کنید:

```sh
curl -d '{"amount":8646000000000, "address":"0x6e...eadf"}' -H "Content-Type: application/json" -X POST http://127.0.0.1:5050/mint
```

نکته: مقدار در خروجی دستور قبلی مشخص شده است.

اضافه شدن موفقیت آمیز صندوق باز خواهد گشت:

```sh
{"new_balance":8646000000000,"tx_hash":"0x48...1919","unit":"wei"}
```

3. استقرار حساب

برای ثبت آن در زنجیره، حساب را در گره محلی starknet devnet مستقر کنید:

```sh
sncast --url http://localhost:5050/rpc account deploy --name new_account --max-fee 0x64a7168300
```

یک استقرار موفق یک هش تراکنش را فراهم می کند. اگر کار نکرد، مراحل قبلی خود را دوباره مرور کنید.

4. تنظیم یک نمایه پیش فرض

شما می توانید یک نمایه پیش فرض برای اقدامات sncast خود تعریف کنید. برای تنظیم یکی، فایل Scarb.toml را ویرایش کنید. برای اینکه new\_account نمایه پیش فرض باشد، بخش \[tool.sncast.new\_account] را پیدا کرده و آن را به \[tool.sncast] تغییر دهید. این بدان معنی است که sncast به طور پیش فرض از این نمایه استفاده می کند مگر اینکه دستور دیگری داده شود.

### مرحله 4: اعلام و استقرار قرارداد ما

در حال حاضر، ما به مرحله مهم استفاده از sncast برای اعلام و استقرار قراردادهای هوشمند خود رسیده ایم.

#### اعلام قرارداد

به یاد بیاورید که ما پیش نویس قرارداد را در مرحله 1 تهیه و آزمایش کردیم. در اینجا، ما بر دو عمل تمرکز خواهیم کرد: ساخت و اعلام.

1. ساخت قرارداد

برای ساخت قرارداد موارد زیر را اجرا کنید:

```sh
scarb build
```

اگر آزمایشات را با استفاده از snforge با موفقیت انجام داده اید، ساخت اسکارب باید بدون مشکل کار کند. پس از اتمام ساخت، یک پوشه هدف جدید در ریشه پروژه شما ظاهر می شود.

در پوشه هدف، یک زیرپوشه توسعه دهنده حاوی سه فایل: \*.casm.json، \*.sierra.json و \*.starknet\_artifacts.json را خواهید یافت.

اگر این فایل‌ها وجود ندارند، احتمالاً به دلیل عدم وجود تنظیمات در فایل Scarb.toml شماست. برای رفع این مشکل، خطوط زیر را بعد از وابستگی ها اضافه کنید:

```toml
[[target.starknet-contract]]
sierra = true
casm = true
```

این خطوط به کامپایلر دستور می‌دهند که خروجی‌های sierra و casm را تولید کند.

2. اعلام قرارداد

ما از دستور sncast declare برای اعلام قرارداد استفاده خواهیم کرد. این فرمت است:

```shell
sncast declare [OPTIONS] --contract-name <CONTRACT>
```

با توجه به این، دستور صحیح این خواهد بود:

```
sncast --profile account1 declare --contract-name HelloStarknet
```

توجه داشته باشید که ما گزینه --url را حذف کرده ایم. چرا؟ هنگام استفاده از --profile، همانطور که در اینجا با account1 دیده می شود، لازم نیست. به یاد داشته باشید، قبلا در این راهنما، اضافه کردن و ایجاد حساب‌های جدید را مورد بحث قرار دادیم. می توانید از account1 یا new\_account استفاده کنید و به نتیجه دلخواه برسید.

> نکته: می توانید یک نمایه پیش فرض برای اقدامات sncast تعریف کنید. فایل Scarb.toml را برای تنظیم پیش فرض تغییر دهید. برای مثال، برای پیش‌فرض کردن new\_account، \[tool.sncast.new\_account] را پیدا کنید و آن را به \[tool.sncast] تغییر دهید. سپس، نیازی به تعیین نمایه برای هر تماس نیست، و دستور خود را ساده می کند:

```sh
sncast declare --contract-name HelloStarknet
```

خروجی شبیه به:

```sh
command: declare
class_hash: 0x20fe30f3990ecfb673d723944f28202db5acf107a359bfeef861b578c00f2a0
transaction_hash: 0x7fbdcca80e7c666f1b5c4522fdad986ad3b731107001f7d8df5f3cb1ce8fd11
```

حتماً هش \*\* کلاس را یادداشت کنید زیرا در مرحله بعدی ضروری است.

> توجه: اگر با خطایی مواجه شدید مبنی بر اینکه هش کلاس قبلاً اعلام شده است، به سادگی به مرحله بعدی بروید. اعلام مجدد قراردادی که قبلاً اعلام شده است، جایز نیست. از هش کلاس ذکر شده برای استقرار استفاده کنید.

### استقرار قرارداد

با اعلام موفقیت آمیز قرارداد و به دست آمدن هش کلاس، ما آماده هستیم تا به استقرار قرارداد ادامه دهیم. این مرحله سرراست است. را در دستور زیر با هش کلاس به دست آمده خود جایگزین کنید:

```sh
sncast deploy --class-hash 0x20fe30f3990ecfb673d723944f28202db5acf107a359bfeef861b578c00f2a0
```

اجرای این احتمالاً نتیجه خواهد داد:

```sh
command: deploy
contract_address: 0x7e3fc427c2f085e7f8adeaec7501cacdfe6b350daef18d76755ddaa68b3b3f9
transaction_hash: 0x6bdf6cfc8080336d9315f9b4df7bca5fb90135817aba4412ade6f942e9dbe60
```

با این حال، ممکن است با برخی از مشکلات روبرو شوید، مانند:

خطا: آدرس RPC در Scarb.toml پاس نشده و یافت نشد. این نشان دهنده عدم وجود یک نمایه پیش فرض در فایل Scarb.toml است. برای رفع این مشکل:

* گزینه -profile را اضافه کنید و به دنبال آن نام پروفایل مورد نظر را مطابق با مواردی که ایجاد کرده اید اضافه کنید.
* از طرف دیگر، یک نمایه پیش‌فرض را همانطور که قبلاً در بخش «اعلام قرارداد» در زیر «اشاره» یا همانطور که در بخش فرعی «افزودن، ایجاد و استقرار حساب» توضیح داده شده است، تنظیم کنید.

شما با موفقیت قرارداد خود را با sncast اجرا کردید! حالا بیایید نحوه تعامل با آن را بررسی کنیم.

### تعامل با قرارداد

این بخش نحوه خواندن و نوشتن اطلاعات قرارداد را توضیح می دهد.

### فراخوانی توابع قرارداد

برای نوشتن به قرارداد، عملکردهای آن را فراخوانی کنید. در اینجا یک نمای کلی از دستور آمده است:

```sh
Usage: sncast invoke [OPTIONS] --contract-address <CONTRACT_ADDRESS> --function <FUNCTION>

Options:
  -a, --contract-address <CONTRACT_ADDRESS>  Address of the contract
  -f, --function <FUNCTION>                  Name of the function
  -c, --calldata <CALLDATA>                  Data for the function
  -m, --max-fee <MAX_FEE>                    Maximum transaction fee (auto-estimated if absent)
  -h, --help                                 Show help
```

برای نشان دادن، بیایید روش افزایش\_بالانس قرارداد هوشمند خود را با نمایه پیش‌فرض از پیش تعیین شده فراخوانی کنیم. همیشه هر گزینه ای ضروری نیست. به عنوان مثال، گاهی اوقات، از جمله --max-fee ممکن است ضروری باشد.

```sh
sncast invoke --contract-address 0x7e...b3f9 --function increase_balance --calldata 4
```

در صورت موفقیت آمیز بودن، یک هش تراکنش مانند زیر دریافت خواهید کرد:

```sh
command: invoke
transaction_hash: 0x33248e393d985a28826e9fbb143d2cf0bb3342f1da85483cf253b450973b638
```

### خواندن از قرارداد

برای بازیابی اطلاعات از قرارداد، از دستور تماس sncast استفاده کنید. در اینجا نحوه کار آن آمده است:

```sh
sncast call --help
```

با اجرای دستور نمایش داده می شود:

```sh
Usage: sncast call [OPTIONS] --contract-address <CONTRACT_ADDRESS> --function <FUNCTION>

Options:
  -a, --contract-address <CONTRACT_ADDRESS>  Address of the contract (hex format)
  -f, --function <FUNCTION>                  Name of the function to call
  -c, --calldata <CALLDATA>                  Function arguments (list of hex values)
  -b, --block-id <BLOCK_ID>                  Block identifier for the call. Accepts: pending, latest, block hash (with a 0x prefix), or block number (u64). Default is 'pending'.
  -h, --help                                 Show help
```

برای مثال:

```sh
sncast call --contract-address 0x7e...b3f9 --function get_balance
```

در حالی که همه گزینه‌ها در مثال استفاده نمی‌شوند، ممکن است لازم باشد گزینه‌هایی مانند --calldata را اضافه کنید و آن را به عنوان یک لیست یا آرایه مشخص کنید.

تماس موفق برمی گردد:

```sh
command: call
response: [0x4]
```

این نشان دهنده موفقیت آمیز عملیات خواندن و نوشتن در قرارداد است.

### راهنمای چند تماس sncast

از sncast multicall برای خواندن و نوشتن همزمان قرارداد استفاده کنید. بیایید نحوه استفاده موثر از این ویژگی را بررسی کنیم.

ابتدا کاربرد اصلی آن را درک کنید:

```sh
sncast multicall --help
```

این دستور نمایش می دهد:

```sh
Execute multiple calls

Usage: sncast multicall <COMMAND>

Commands:
  run   Execute multicall using a .toml file
  new   Create a template for the multicall .toml file
  help  Display help for subcommand(s)

Options:
  -h, --help  Show help
```

برای کاوش عمیق تر، دستور فرعی جدید را راه اندازی کنید:

```sh
Generate a template for the multicall .toml file

Usage: sncast multicall new [OPTIONS]

Options:
  -p, --output-path <OUTPUT_PATH>  File path for saving the template
  -o, --overwrite                  Overwrite file if it already exists at specified path
  -h, --help                       Display help
```

الگویی به نام call1.toml ایجاد کنید:

```sh
sncast multicall new --output-path ./call1.toml --overwrite
```

این یک الگوی اساسی ارائه می دهد:

```toml
[[call]]
call_type = "deploy"
class_hash = ""
inputs = []
id = ""
unique = false

[[call]]
call_type = "invoke"
contract_address = ""
function = ""
inputs = []
```

تغییر call1.toml به:

```toml
[[call]]
call_type = "invoke"
contract_address = "0x7e3fc427c2f085e7f8adeaec7501cacdfe6b350daef18d76755ddaa68b3b3f9"
function = "increase_balance"
inputs = ['0x4']

[[call]]
call_type = "invoke"
contract_address = "0x7e3fc427c2f085e7f8adeaec7501cacdfe6b350daef18d76755ddaa68b3b3f9"
function = "increase_balance"
inputs = ['0x1']
```

در چند تماس، فقط اعمال Deploy و Invoke مجاز هستند. برای راهنمایی دقیق در مورد آنها، به بخش قبلی مراجعه کنید.

> توجه: اطمینان حاصل کنید که ورودی ها در قالب هگزادسیمال هستند. رشته ها به طور معمول کار می کنند، اما اعداد برای نتایج دقیق به این قالب نیاز دارند.

برای اجرای چند فراخوانی از:

```sh
sncast multicall run --path call1.toml
```

پس از موفقیت:

```sh
command: multicall run
transaction_hash: 0x1ae4122266f99a5ede495ff50fdbd927c31db27ec601eb9f3eaa938273d4d61
```

تعادل را بررسی کنید:

```sh
sncast call --contract-address 0x7e...b3f9 --function get_balance
```

پاسخ:

```shell
command: call
response: [0x9]
```

موجودی مورد انتظار، 0x9، تایید شده است.

### نتیجه

در این راهنما استفاده از sncast، یک ابزار خط فرمان قوی که برای قراردادهای هوشمند starknet طراحی شده است، توضیح داده شده است. هدف آن این است که تعامل با قراردادهای هوشمند starknet را بدون دردسر انجام دهد. عملکردهای کلیدی شامل استقرار قرارداد، فراخوانی عملکرد و فراخوانی عملکرد است.
