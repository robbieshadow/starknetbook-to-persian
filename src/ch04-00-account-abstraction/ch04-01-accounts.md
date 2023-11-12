# قراردادهای حساب

با درک واضح تری از مفهوم AA، اجازه دهید به کدنویسی آن در Starknet ادامه دهیم.

### رابط قرارداد حساب

قراردادهای حساب، که نوعی قرارداد هوشمند هستند، با روش های خاصی متمایز می شوند. یک قرارداد هوشمند زمانی به یک قرارداد حساب تبدیل می شود که از رابط عمومی که در SNIP-6 (پیشنهاد بهبود Starknet-6: رابط استاندارد حساب) مشخص شده است، پیروی کند. این استاندارد از SRC-6 و SRC-5 الهام گرفته است، مشابه ERCهای اتریوم، که قراردادهای کاربردی و استانداردهای قرارداد را ایجاد می کنند.

```rust
/// @title Represents a call to a target contract
/// @param to The target contract address
/// @param selector The target function selector
/// @param calldata The serialized function parameters
struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Array<felt252>
}

/// @title SRC-6 Standard Account
trait ISRC6 {
    /// @notice Execute a transaction through the account
    /// @param calls The list of calls to execute
    /// @return The list of each call's serialized return value
    fn __execute__(calls: Array<Call>) -> Array<Span<felt252>>;

    /// @notice Assert whether the transaction is valid to be executed
    /// @param calls The list of calls to execute
    /// @return The string 'VALID' represented as felt when is valid
    fn __validate__(calls: Array<Call>) -> felt252;

    /// @notice Assert whether a given signature for a given hash is valid
    /// @param hash The hash of the data
    /// @param signature The signature to validate
    /// @return The string 'VALID' represented as felt when the signature is valid
    fn is_valid_signature(hash: felt252, signature: Array<felt252>) -> felt252;
}

/// @title SRC-5 Standard Interface Detection
trait ISRC5 {
    /// @notice Query if a contract implements an interface
    /// @param interface_id The interface identifier, as specified in SRC-5
    /// @return `true` if the contract implements `interface_id`, `false` otherwise
    fn supports_interface(interface_id: felt252) -> bool;
}

```

از پیشنهاد، یک قرارداد حساب باید دارای روش های **execute**، **validate**، و is\_valid\_signature از ویژگی ISRC6 باشد.

توابع ارائه شده در خدمت این اهداف هستند:

* **validate**: فهرستی از تماس های در نظر گرفته شده برای اجرا را بر اساس قوانین قرارداد تایید می کند. به جای یک بولی، یک رشته کوتاه مانند "VALID" را در یک felt252 برمی گرداند تا نتایج اعتبار سنجی را منتقل کند. در قاهره، این رشته کوتاه نشان دهنده ASCII یک نمد است. اگر تأیید ناموفق باشد، هر نمدی غیر از «VALID» را می توان برگرداند. اغلب، 0 انتخاب می شود.
* is\_valid\_signature: صحت امضای تراکنش را تایید می کند. یک هش داده تراکنش و یک امضا می گیرد و آن را با یک کلید عمومی یا روش دیگری انتخاب شده توسط نویسنده قرارداد مقایسه می کند. نتیجه یک رشته کوتاه 'VALID' در یک فلم252 است.
* **execute**: پس از تایید اعتبار، **execute** یک سری فراخوانی قرارداد (همانطور که Call struct) انجام می دهد. آرایه ای از ساختارهای Span را برمی گرداند و مقادیر بازگشتی آن فراخوان ها را نشان می دهد.

علاوه بر این، ویژگی SNIP-5 (تشخیص رابط استاندارد) باید با تابعی به نام supports\_interface تعریف شود. این تابع بررسی می کند که آیا یک قرارداد از یک رابط خاص پشتیبانی می کند، یک شناسه رابط دریافت می کند و یک بولی را برمی گرداند.

```rust
    trait ISRC5 {
        fn supports_interface(interface_id: felt252) -> bool;
    }
```

در اصل، زمانی که کاربر یک تراکنش فراخوانی را ارسال می کند، پروتکل با فراخوانی متد **validate** آغاز می شود. این اصالت امضاکننده مرتبط را تأیید می کند. به دلایل امنیتی، به ویژه برای محافظت از Sequencer در برابر حملات Denial of Service (DoS) \[1]، محدودیت هایی بر روی عملیات در روش **validate** وجود دارد. اگر امضا تأیید شود، روش مقدار Feel252 'VALID' را به دست می دهد. اگر نه، 0 را برمی گرداند.

پس از اینکه پروتکل امضاکننده را تأیید کرد، تابع **execute** را فراخوانی می‌کند و آرایه‌ای از تمام عملیات‌های مورد نظر را که به آن «تماس‌ها» گفته می‌شود، به عنوان آرگومان ارسال می‌کند. هر یک از این فراخوانی ها یک آدرس قرارداد هوشمند هدف (to)، روشی که باید اجرا شود (انتخاب کننده)، و آرگومان های مورد نیاز این روش (calldata) را مشخص می کند.

```rust
struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Array<felt252>
}

trait ISRC6 {

    ....

    fn __execute__(calls: Array<Call>) -> Array<Span<felt252>>;

    ....

}
```

اجرای یک تماس ممکن است مقدار بازگشتی را از قرارداد هوشمند هدف به دست آورد. پروتکل Starknet خواه یک Feel252، Boolean یا یک ساختار داده پیچیده‌تر مانند ساختار یا آرایه باشد، با استفاده از Span بازگشت را سریال می‌کند. از آنجایی که Span قطعه ای از آرایه \[2] را می گیرد، تابع **execute** آرایه ای از عناصر Span را خروجی می دهد. این آرایه نشان دهنده بازخورد سریالی از هر عملیات در چند تماس است.

روش is\_valid\_signature توسط پروتکل Starknet اجباری یا استفاده نشده است. در عوض، این یک کنوانسیون در جامعه توسعه دهندگان Starknet است. هدف آن تسهیل احراز هویت کاربر در برنامه های web3 است. به عنوان مثال، کاربری را در نظر بگیرید که سعی دارد با استفاده از کیف پول دیجیتال خود وارد بازار NFT شود. برنامه وب از کاربر می خواهد پیامی را امضا کند، سپس از تابع is\_valid\_signature برای تأیید صحت آدرس کیف پول مرتبط استفاده می کند.

برای اطمینان از اینکه سایر قراردادهای هوشمند انطباق یک قرارداد حساب با رابط عمومی SNIP-6 را تشخیص می دهند، توسعه دهندگان باید روش supports\_interface را از ویژگی درون نگری ISRC5 بگنجانند. این روش به شناسه رابط SNIP-6 به عنوان آرگومان نیاز دارد.

```rust
struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Array<felt252>
}

trait ISRC6 {
    // Implementations for __execute__, __validate__, and is_valid_signature go here.
}

trait ISRC5 {
    fn supports_interface(interface_id: felt252) -> bool;
}
```

Interface\_id مطابق با هش انبوه انتخابگرهای صفت است، همانطور که در ERC165 اتریوم \[3] توضیح داده شده است. توسعه دهندگان می توانند شناسه را با استفاده از ابزار src5-rs \[4] محاسبه کنند یا به شناسه از پیش محاسبه شده تکیه کنند:`1270010605630597976495846281167968799381097569185364931397797212080166453709`.

ساختار اساسی برای قرارداد حساب، که با استاندارد رابط SNIP-G همسو می شود، به این صورت است:

```rust
struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Array<felt252>
}

trait ISRC6 {
    fn __execute__(calls: Array<Call>) -> Array<Span<felt252>>;
    fn __validate__(calls: Array<Call>) -> felt252;
    fn is_valid_signature(hash: felt252, signature: Array<felt252>) -> felt252;
}

trait ISRC5 {
    fn supports_interface(interface_id: felt252) -> bool;
}
```

### گسترش رابط

در حالی که مؤلفه‌هایی که قبلاً ذکر شد، پایه و اساس یک قرارداد حساب در راستای استاندارد SNIP-6 را ایجاد می‌کنند، توسعه‌دهندگان می‌توانند ویژگی‌های بیشتری را برای افزایش قابلیت‌های قرارداد معرفی کنند.

به عنوان مثال، اگر قرارداد سایر قراردادها را اعلام می کند و هزینه های گاز مربوطه را مدیریت می کند، تابع **validate\_declare** را ادغام کنید. این روشی برای تأیید اعتبار اظهارنامه قرارداد ارائه می دهد. برای کسانی که مشتاق استقرار قرارداد هوشمند خلاف واقع هستند، تابع **validate\_deploy** را می توان گنجاند.

استقرار خلاف واقع به توسعه دهندگان این امکان را می دهد تا یک قرارداد حساب بدون وابستگی به قرارداد حساب دیگری برای هزینه گاز تنظیم کنند. این روش زمانی ارزشمند است که تمایلی به پیوند قرارداد حساب جدید با آدرس استقرار آن وجود نداشته باشد و شروعی تازه را تضمین کند.

این رویکرد شامل:

1. تعیین آدرس بالقوه قرارداد حساب ما بدون استقرار واقعی، با ابزار Starkli \[5] امکان پذیر است.
2. انتقال ETH کافی به آدرس پیش بینی شده برای پوشش هزینه های استقرار.
3. ارسال یک تراکنش deploy\_account به Starknet حاوی کد کامپایل شده قرارداد ما. سپس ترتیب‌دهنده قرارداد حساب را در آدرس تخمینی فعال می‌کند و هزینه‌های گاز خود را از ETH منتقل شده جبران می‌کند. هیچ اقدام اعلامی از قبل لازم نیست.

برای سازگاری بهتر با ابزارهایی مانند Starkli بعداً، کلید عمومی امضاکننده را از طریق یک تابع view در رابط عمومی در معرض دید قرار دهید. در زیر رابط قرارداد حساب افزوده شده است:

```rust
/// @title IAccountAddon - Extended account contract interface
trait IAccountAddon {
    /// @notice Validates if a declare transaction can proceed
    /// @param class_hash Hash of the smart contract under declaration
    /// @return 'VALID' string as felt, if valid
    fn __validate_declare__(class_hash: felt252) -> felt252;

    /// @notice Validates if counterfactual deployment can proceed
    /// @param class_hash Hash of the account contract under deployment
    /// @param salt Modifier for account address
    /// @param public_key Account signer's public key
    /// @return 'VALID' string as felt, if valid
    fn __validate_deploy__(class_hash: felt252, salt: felt252, public_key: felt252) -> felt252;

    /// @notice Fetches the signer's public key
    /// @return Public key
    fn public_key() -> felt252;
}
```

در نتیجه، یک قرارداد حساب جامع شامل رابط های SNIP-5، SNIP-6 و Addon است.

```rust
// Cheat sheet

struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Array<felt252>
}

trait ISRC6 {
    fn __execute__(calls: Array<Call>) -> Array<Span<felt252>>;
    fn __validate__(calls: Array<Call>) -> felt252;
    fn is_valid_signature(hash: felt252, signature: Array<felt252>) -> felt252;
}

trait ISRC5 {
    fn supports_interface(interface_id: felt252) -> bool;
}

trait IAccountAddon {
    fn __validate_declare__(class_hash: felt252) -> felt252;
    fn __validate_deploy__(class_hash: felt252, salt: felt252, public_key: felt252) -> felt252;
    fn public_key() -> felt252;
}

```

#### خلاصه

ما تمایزات بین قراردادهای حساب و قراردادهای هوشمند اولیه را تجزیه کرده ایم، به ویژه با تمرکز بر روش های ارائه شده در SNIP-6.

* ویژگی ISRC6 را معرفی کرد و عملکردهای اساسی را برجسته کرد:
  * **validate**: تراکنش ها را تایید می کند.
  * is\_valid\_signature: امضاها را تایید می کند.
  * **execute**: تماس های قراردادی را اجرا می کند.
* ویژگی ISRC5 را مورد بحث قرار داد و اهمیت تابع supports\_interface در تأیید پشتیبانی رابط را برجسته کرد.
* ساختار Call را برای نشان دادن یک فراخوان قرارداد منفرد توضیح داد و اجزای آن را توضیح داد: به، انتخابگر، و calldata.
* ویژگی‌های پیشرفته قراردادهای حساب، مانند توابع **validate\_declare** و **validate\_deploy** را لمس کرد.

در آینده، ما یک قرارداد حساب پایه ایجاد می کنیم و آن را در Starknet مستقر می کنیم و بینش عملی در مورد عملکرد و تعاملات آنها ارائه می دهیم.
