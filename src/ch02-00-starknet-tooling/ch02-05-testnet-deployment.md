# استقرار شبکه تست

این فصل توسعه دهندگان را از طریق فرآیند کامپایل، استقرار، و تعامل با قرارداد هوشمند Starknet که در قاهره در شبکه آزمایشی نوشته شده است، راهنمایی می کند. پیش از این، تمرکز بر روی استقرار قراردادها با استفاده از یک گره محلی، Katana بود. این بار، استقرار و تعامل شبکه آزمایشی Starknet را هدف قرار می دهد.

اطمینان حاصل کنید که دستورات زیر با موفقیت بر روی سیستم شما اجرا می شوند. اگر نه، بخش "نصب پایه" را ببینید:

```bash
    scarb --version  # For Cairo code compilation
    starkli --version  # To interact with Starknet
```

### راه اندازی کیف پول هوشمند

یک کیف پول هوشمند شامل یک امضاکننده و یک توصیفگر حساب است. Signer یک قرارداد هوشمند با یک کلید خصوصی برای امضای تراکنش ها است، در حالی که Account Descriptor یک فایل JSON است که آدرس کیف پول و کلید عمومی را نشان می دهد.

برای اینکه یک حساب کاربری به‌عنوان امضاکننده استفاده شود، باید در شبکه مناسب، Starknet Goerli یا mainnet، مستقر شده و تأمین مالی شود. برای این مثال می خواهیم از Goerli Testnet استفاده کنیم. برای استقرار کیف پول خود، از Getting Started دیدن کنید و بخش Smart Wallet Setup را پیدا کنید.

اکنون شما آماده تعامل با قراردادهای هوشمند Starknet هستید.

### ایجاد یک امضاکننده

Signer یک قرارداد هوشمند ضروری است که قادر به امضای تراکنش در Starknet است. برای ایجاد یک کیف پول هوشمند به کلید خصوصی نیاز دارید که کلید عمومی را می توان از آن استخراج کرد.

Starkli ذخیره ایمن کلید خصوصی شما را از طریق یک فایل فروشگاه کلید فعال می کند. این فایل رمزگذاری شده با استفاده از رمز عبور قابل دسترسی است و عموماً در فهرست پیش فرض Starkli ذخیره می شود.

ابتدا دایرکتوری پیش فرض را ایجاد کنید:

```bash
    mkdir ~/.starkli-wallets/deployer -p
```

سپس فایل keystore را تولید کنید. دستور signer شامل دستورات فرعی برای ایجاد یک فایل ذخیره کلید از یک کلید خصوصی یا ایجاد کامل یک فایل جدید است. در این آموزش، از گزینه کلید خصوصی که رایج ترین مورد استفاده است استفاده می کنیم. شما باید مسیر فایل keystore را که می خواهید ایجاد کنید ارائه دهید. شما می توانید هر نامی را به فایل keystore بدهید، احتمالا چندین کیف پول خواهید داشت. در این آموزش از نام my\_keystore\_ 1.json استفاده می کنیم.

```bash
    starkli signer keystore from-key ~/.starkli-wallets/deployer/my_keystore_1.json
    Enter private key:
    Enter password:
```

در اعلان کلید خصوصی، کلید خصوصی کیف پول هوشمند خود را بچسبانید. در اعلان رمز عبور، رمز عبور دلخواه خود را وارد کنید. برای امضای تراکنش ها با استفاده از Starkli به این رمز عبور نیاز دارید.

کلید خصوصی را از کیف پول Braavos یا Argent خود صادر کنید. برای Argent X، می توانید آن را در بخش "تنظیمات" پیدا کنید → حساب خود را انتخاب کنید → "صادر کردن کلید خصوصی". برای Braavos، می توانید آن را در بخش "تنظیمات" → "حریم خصوصی و امنیت" → "صادر کردن کلید خصوصی" پیدا کنید.

اگرچه دانستن کلید خصوصی یک کیف پول هوشمند برای امضای تراکنش ها ضروری است، اما کافی نیست. همچنین باید استارکلی را در مورد مکانیسم امضا استفاده شده توسط کیف پول هوشمند ما که توسط Braavos یا Argent X ایجاد شده است، مطلع کنیم. آیا از منحنی بیضوی استفاده می کند؟ اگر بله، کدام یک؟ این دلیلی است که ما به یک فایل توصیفگر حساب نیاز داریم.

### \[اختیاری] معماری امضاکننده Starknet

این بخش اختیاری است و برای کسانی در نظر گرفته شده است که می خواهند درباره Starknet Signer اطلاعات بیشتری کسب کنند. اگر به جزئیات علاقه ندارید، می توانید از آن صرف نظر کنید.

Starknet Signer نقش مهمی در ایمن سازی تراکنش های شما ایفا می کند. بیایید ابهام زدایی کنیم از آنچه در زیر کاپوت می گذرد.

اجزای کلیدی:

1. کلید خصوصی: یک کلید هگزادسیمال 256 بیتی/32 بایتی/64 کاراکتری (با نادیده گرفتن پیشوند 0x) که سنگ بنای امنیت کیف پول شما است.
2. کلید عمومی: برگرفته از کلید خصوصی، همچنین یک کلید هگزادسیمال 256 بیتی/32 بایتی/64 کاراکتری است.
3. آدرس کیف پول هوشمند: برخلاف اتریوم، آدرس اینجا تحت تأثیر کلید عمومی، هش کلاس و یک نمک قرار می‌گیرد. در Starknet Documentation بیشتر بیاموزید.

برای مشاهده جزئیات فایل keystore ایجاد شده قبلی:

```bash
    cat ~/.starkli-wallets/deployer/my_keystore_1.json
```

آناتومی فایل keystore.json:

```json
{
  "crypto": {
    "cipher": "aes-128-ctr",
    "cipherparams": {
      "iv": "dba5f9a67456b121f3f486aa18e24db7"
    },
    "ciphertext": "b3cda3df39563e3dd61064149d6ed8c9ab5f07fbcd6347625e081fb695ddf36c",
    "kdf": "scrypt",
    "kdfparams": {
      "dklen": 32,
      "n": 8192,
      "p": 1,
      "r": 8,
      "salt": "6dd5b06b1077ba25a7bf511510ea0c608424c6657dd3ab51b93029244537dffb"
    },
    "mac": "55e1616d9ddd052864a1ae4207824baac58a6c88798bf28585167a5986585ce6"
  },
  "id": "afbb9007-8f61-4e62-bf14-e491c30fd09a",
  "version": 3
}
```

* نسخه: نسخه پیاده سازی کیف پول هوشمند.
* id: یک رشته شناسایی که به طور تصادفی تولید می شود.
* crypto: تمام جزئیات رمزگذاری را در خود جای داده است.

داخل کریپتو:

* cipher: الگوریتم رمزگذاری مورد استفاده را مشخص می کند که در این مورد AES-128-CTR است.
  * AES (استاندارد رمزگذاری پیشرفته): یک استاندارد رمزگذاری پذیرفته شده جهانی است.
  * 128: به اندازه کلید در بیت اشاره دارد و آن را به یک کلید 128 بیتی تبدیل می کند.
  * CTR (Counter Mode): حالت خاصی از کار برای رمز AES.
* cipherparams: حاوی یک بردار مقداردهی اولیه (IV) است که تضمین می‌کند رمزگذاری یک متن ساده با همان کلید، متن‌های رمزی متفاوتی را تولید می‌کند.
  * iv (بردار اولیه سازی): یک رشته هگز 16 بایتی که به عنوان یک نقطه شروع تصادفی و منحصر به فرد برای هر عملیات رمزگذاری عمل می کند.
* متن رمزی: این کلید خصوصی پس از رمزگذاری است که به طور ایمن ذخیره می شود تا فقط رمز عبور صحیح بتواند آن را آشکار کند.
* kdf و kdfparams: KDF مخفف Key Derivation Function است. این یک لایه امنیتی را با نیاز به کار محاسباتی اضافه می کند و حملات brute-force را سخت تر می کند.
  * dklen: طول (بر حسب بایت) کلید مشتق شده. به طور معمول 32 بایت.
  * n: یک عامل هزینه که نشان دهنده استفاده از CPU/حافظه است. مقدار بالاتر به این معنی است که کار محاسباتی بیشتری مورد نیاز است، بنابراین امنیت افزایش می یابد.
  * p: عامل موازی سازی که بر پیچیدگی محاسباتی تأثیر می گذارد.
  * r: اندازه بلوک برای تابع هش، دوباره بر نیازهای محاسباتی تأثیر می گذارد.
  * salt: یک مقدار تصادفی که با رمز عبور ترکیب می شود تا از حملات فرهنگ لغت جلوگیری کند.
* mac (کد احراز هویت پیام): این یک کد رمزنگاری است که یکپارچگی پیام را تضمین می کند (کلید خصوصی رمزگذاری شده در این مورد). با استفاده از هش متن رمز شده و بخشی از کلید مشتق شده تولید می شود.

### ایجاد یک توصیفگر حساب

یک توصیفگر حساب استارکلی را در مورد ویژگی های منحصر به فرد کیف پول هوشمند شما، مانند مکانیسم امضای آن، مطلع می کند. می توانید این توصیفگر را با استفاده از دستور فرعی واکشی Starkli تحت فرمان حساب ایجاد کنید. فرمان فرعی fetch آدرس کیف پول روی زنجیره شما را به عنوان ورودی می گیرد و فایل توصیفگر حساب را تولید می کند. فایل توصیف حساب یک فایل JSON است که حاوی جزئیات کیف پول هوشمند شما است.

```bash
    starkli account fetch <SMART_WALLET_ADDRESS> --output ~/.starkli-wallets/deployer/my_account_1.json
```

پس از اجرای دستور، پیامی مانند تصویر زیر مشاهده خواهید کرد. ما از کیف پول Braavos به عنوان مثال استفاده می کنیم، اما مراحل برای کیف پول Argent یکسان است.

```bash
    Account contract type identified as: Braavos
    Description: Braavos official proxy account
    Downloaded new account config file: ~/.starkli-wallets/deployer/my_account_1.json
```

در صورتی که با خطای زیر مواجه شدید:

```bash
    Error: code=ContractNotFound, message="Contract with address {SMART_WALLET_ADDRESS} is not deployed."
```

این بدان معناست که احتمالاً به تازگی یک کیف پول جدید ایجاد کرده اید و هنوز راه اندازی نشده است. برای انجام این کار، باید کیف پول خود را با توکن ها تامین کنید و توکن ها را به آدرس کیف پول دیگری انتقال دهید. پس از این فرآیند، آدرس کیف پول خود را در اکسپلورر Starknet جستجو کنید. برای مشاهده جزئیات، به شروع به کار برگردید و بخش Smart Wallet Setup را پیدا کنید.

پس از ایجاد فایل توصیفگر حساب، می توانید جزئیات آن را مشاهده کنید، اجرا کنید:

```bash
    cat ~/.starkli-wallets/deployer/my_account_1.json
```

در اینجا یک توصیفگر معمولی ممکن است به نظر برسد:

```json
{
  "version": 1,
  "variant": {
    "type": "braavos",
    "version": 1,
    "implementation": "0x5dec330eebf36c8672b60db4a718d44762d3ae6d1333e553197acb47ee5a062",
    "multisig": {
      "status": "off"
    },
    "signers": [
      {
        "type": "stark",
        "public_key": "0x49759ed6197d0d385a96f9d8e7af350848b07777e901f5570b3dc2d9744a25e"
      }
    ]
  },
  "deployment": {
    "status": "deployed",
    "class_hash": "0x3131fa018d520a037686ce3efddeab8f28895662f019ca3ca18a626650f7d1e",
    "address": "0x6dcb489c1a93069f469746ef35312d6a3b9e56ccad7f21f0b69eb799d6d2821"
  }
}
```

توجه: اگر از کیف پول Argent استفاده کنید، ساختار متفاوت خواهد بود.

### تنظیم متغیرهای محیطی

برای ساده سازی دستورات Starkli، می توانید متغیرهای محیطی را تنظیم کنید. دو متغیر کلیدی بسیار مهم هستند: یکی برای مکان فایل ذخیره‌کننده کلید Signer و دیگری برای فایل Account Descriptor.

```bash
    export STARKNET_ACCOUNT=~/.starkli-wallets/deployer/my_account_1.json
    export STARKNET_KEYSTORE=~/.starkli-wallets/deployer/my_keystore_1.json
```

تنظیم این متغیرها اجرای دستورات Starkli را آسان تر و کارآمدتر می کند.

### اعلام قراردادهای هوشمند در استارک نت

استقرار یک قرارداد هوشمند در Starknet شامل دو مرحله است:

* کد قرارداد خود را اعلام کنید
* یک نمونه از کد اعلام شده را مستقر کنید.

برای شروع، به دایرکتوری قراردادها در فصل اول مخزن کتاب Starknet بروید. فایل src/lib.cairo حاوی یک قرارداد اساسی برای تمرین است.ابتدا قرارداد را با استفاده از کامپایلر Scarb کامپایل کنید. اگر Scarb را نصب نکرده اید، راهنمای نصب را در بخش تنظیم محیط خود دنبال کنید.

```bash
    scarb build
```

این یک قرارداد کامپایل‌شده در target/dev/ به‌عنوان «contracts\_Ownable.sierra.json» ایجاد می‌کند (در فصل 2 کتاب با جزئیات بیشتری درباره Scarb آشنا خواهیم شد).

با جمع‌آوری قرارداد هوشمند، ما آماده هستیم تا آن را با استفاده از Starkli اعلام کنیم. قبل از اعلام قرارداد خود، در مورد یک ارائه دهنده RPC تصمیم بگیرید.

### انتخاب یک ارائه دهنده RPC

سه گزینه اصلی برای ارائه دهندگان RPC وجود دارد که بر اساس سهولت استفاده مرتب شده اند:

1. Starknet Sequencer's Gateway: سریعترین گزینه و در حال حاضر پیش فرض Starkli است. دروازه ترتیب ساز منسوخ شده است و به زودی توسط StarkWare غیرفعال می شود. اکیداً به شما توصیه می شود از یک ارائه دهنده API شخص ثالث JSON-RPC مانند Infura، Alchemy یا Chainstack استفاده کنید.
2. Infura یا کیمیاگری: یک پله در پیچیدگی. باید یک کلید API تنظیم کنید و یک نقطه پایانی را انتخاب کنید. برای Infura، شبیه https://starknet-goerli.infura.io/v3/\<API\_KEY> است. در مستندات Infura بیشتر بیاموزید.
3. گره خودتان: برای کسانی که خواهان کنترل کامل هستند. این پیچیده ترین است اما بیشترین آزادی را ارائه می دهد. برای راهنمای راه‌اندازی، فصل 4 کتاب Starknet یا Kasar را بررسی کنید.

در این آموزش از کیمیاگری استفاده خواهیم کرد. ما می‌توانیم متغیر محیطی STARKNET\_RPC را برای آسان‌تر کردن فراخوانی دستورات تنظیم کنیم:

```bash
    export STARKNET_RPC="https://starknet-goerli.g.alchemy.com/v2/<API_KEY>"
```

### قرارداد خود را اعلام کنید

این دستور را اجرا کنید تا قرارداد خود را با استفاده از دروازه پیش فرض Starknet Sequencer's اعلام کنید:

```bash
    starkli declare target/dev/contracts_Ownable.sierra.json
```

با توجه به آدرس اینترنتی STARKNET\_RPC، starkli می تواند شبکه بلاک چین هدف، در این مورد "goerli" را تشخیص دهد، بنابراین لازم نیست به طور صریح آن را مشخص کنید.

مگر اینکه با شبکه‌های سفارشی کار می‌کنید که شناسایی نسخه کامپایلر مناسب برای استارکلی غیرممکن است، نیازی به انتخاب دستی نسخه‌ای با --network و --compiler-نسخه ندارید.

اگر با «خطا: کلاس قرارداد نامعتبر» مواجه شدید، احتمالاً به این معنی است که نسخه کامپایلر Scarb شما با Starkli ناسازگار است. مراحل بالا را برای تراز کردن نسخه ها دنبال کنید. Starkli معمولاً از نسخه های کامپایلر پذیرفته شده توسط شبکه اصلی پشتیبانی می کند، حتی اگر آخرین نسخه Scarb هنوز سازگار نباشد.

پس از اجرای دستور، یک هش کلاس قرارداد دریافت خواهید کرد. این هش منحصر به فرد به عنوان شناسه کلاس قرارداد شما در Starknet عمل می کند. مثلا:

```bash
    Class hash declared: 0x04c70a75f0246e572aa2e1e1ec4fffbe95fa196c60db8d5677a5c3a3b5b6a1a8
```

می توانید این هش را به عنوان آدرس کلاس قرارداد در نظر بگیرید. از یک بلاک کاوشگر مانند StarkScan برای تأیید این هش در بلاک چین استفاده کنید.

اگر کلاس قراردادی که می‌خواهید اعلام کنید از قبل وجود داشته باشد، اشکالی ندارد که می‌توانیم ادامه دهیم. پیامی مانند زیر دریافت خواهید کرد:

```bash
    Not declaring class as its already declared. Class hash:
    0x04c70a75f0246e572aa2e1e1ec4fffbe95fa196c60db8d5677a5c3a3b5b6a1a8
```

### استقرار قراردادهای هوشمند در Starknet

برای استقرار یک قرارداد هوشمند، باید آن را در شبکه آزمایشی Starknet نمونه سازی کنید. این فرآیند شامل اجرای دستوری است که به دو جزء اصلی نیاز دارد:

1. هش کلاس قرارداد هوشمند شما.
2. هر استدلال سازنده ای که قرارداد انتظار دارد.

در مثال ما، سازنده انتظار یک آدرس مالک را دارد. می‌توانید در \[فصل 12 کتاب قاهره] درباره سازنده‌ها بیشتر بیاموزید (https://book.cairo-lang.org/ch99-01-03-02-contract-functions.html?highlight=constructor#1-constructors) .

دستور به شکل زیر خواهد بود:

```bash
    starkli deploy \
        <CLASS_HASH> \
        <CONSTRUCTOR_INPUTS>
```

در اینجا یک مثال خاص با هش کلاس واقعی و ورودی های سازنده آورده شده است (به عنوان آدرس مالک از آدرس کیف پول هوشمند شما استفاده می کند تا بتوانید بعداً تابع transfer\_ownership را فراخوانی کنید):

```bash
    starkli deploy \
        0x04c70a75f0246e572aa2e1e1ec4fffbe95fa196c60db8d5677a5c3a3b5b6a1a8 \
        0x02cdAb749380950e7a7c0deFf5ea8eDD716fEb3a2952aDd4E5659655077B8510
```

پس از اجرای دستور و وارد کردن رمز عبور، باید خروجی زیر را مشاهده کنید:

```bash
    Deploying class 0x04c70a75f0246e572aa2e1e1ec4fffbe95fa196c60db8d5677a5c3a3b5b6a1a8 with salt 0x065034b27a199cbb2a5b97b78a8a6a6c6edd027c7e398b18e5c0e5c0c65246b7...
    The contract will be deployed at address 0x02a83c32d4b417d3c22f665acbc10e9a1062033b9ab5b2c3358952541bc6c012
    Contract deployment transaction: 0x0743de1e233d38c4f3e9fb13f1794276f7d4bf44af9eac66e22944ad1fa85f14
    Contract deployed:
    0x02a83c32d4b417d3c22f665acbc10e9a1062033b9ab5b2c3358952541bc6c012
```

این قرارداد در حال حاضر در شبکه آزمایشی Starknet فعال است. می‌توانید وضعیت آن را با استفاده از یک Block Explorer مانند StarkScan تأیید کنید. در برگه «خواندن/نوشتن قرارداد»، عملکردهای خارجی قرارداد را مشاهده خواهید کرد.

### تعامل با قرارداد Starknet

Starkli تعامل با قراردادهای هوشمند را از طریق دو روش اصلی امکان پذیر می کند: فراخوانی توابع فقط خواندنی و فراخوانی برای توابع نوشتن که حالت را تغییر می دهند.

### فراخوانی یک تابع خواندن

فرمان فراخوانی شما را قادر می سازد تا عملکرد قرارداد هوشمند را بدون ارسال تراکنش پرس و جو کنید. به عنوان مثال، برای اینکه بدانید مالک فعلی قرارداد کیست، می‌توانید از تابع get\_owner استفاده کنید که نیازی به آرگومان ندارد.

```bash
    starkli call \
        <CONTRACT_ADDRESS> \
        get_owner
```

آدرس قرارداد خود را جایگزین \<CONTRACT\_ADDRESS> کنید. این فرمان آدرس مالک را که در ابتدا در زمان استقرار قرارداد تنظیم شده بود، برمی گرداند:

```bash
    [
        "0x02cdab749380950e7a7c0deff5ea8edd716feb3a2952add4e5659655077b8510"
    ]
```

### فراخوانی یک تابع Write

می توانید وضعیت قرارداد را با استفاده از دستور invoke تغییر دهید. برای مثال، اجازه دهید مالکیت قرارداد را با تابع transfer\_ownership منتقل کنیم.

```bash
    starkli invoke \
        <CONTRACT_ADDRESS> \
        transfer_ownership \
        <NEW_OWNER_ADDRESS>
```

\<CONTRACT\_ADDRESS> را با آدرس قرارداد و \<NEW\_OWNER\_ADDRESS> را با آدرسی که می‌خواهید مالکیت را به آن منتقل کنید جایگزین کنید. اگر کیف پول هوشمندی که استفاده می کنید مالک قرارداد نباشد، خطایی ظاهر می شود. توجه داشته باشید که مالک اولیه هنگام استقرار قرارداد تنظیم شده است:

```bash
    Execution was reverted; failure reason: [0x43616c6c6572206973206e6f7420746865206f776e6572].
```

دلیل شکست به صورت فلت کدگذاری می شود. o آن را رمزگشایی کنید، از دستور parse-cairo-string starkli استفاده کنید.

```bash
    starkli parse-cairo-string <ENCODED_ERROR>
```

به عنوان مثال، اگر 0x43616c6c6572206973206e6f7420746865206f776e6572 را مشاهده کردید، رمزگشایی آن "تماس گیرنده مالک نیست" به دست می آید.

پس از یک تراکنش موفق در L2، از یک بلاک کاوشگر مانند StarkScan یا Voyager برای تأیید وضعیت تراکنش با استفاده از هش ارائه شده توسط دستور invoke استفاده کنید.

برای تأیید اینکه مالکیت با موفقیت منتقل شده است، می توانید دوباره تابع get\_owner را فراخوانی کنید:

```bash
    starkli call \
        <CONTRACT_ADDRESS> \
        get_owner
```

اگر تابع آدرس مالک جدید را برگرداند، انتقال موفقیت آمیز بوده است.

تبریک می گویم! شما با موفقیت قرارداد Starknet را مستقر کرده و با آن تعامل برقرار کرده اید.
