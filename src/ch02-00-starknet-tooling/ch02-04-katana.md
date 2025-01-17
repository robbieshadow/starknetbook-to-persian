# کاتانا: یک گره محلی

کاتانا برای کمک به توسعه محلی طراحی شده است. این ایجاد توسط تیم Dojo شما را قادر می سازد تا تمام فعالیت های مربوط به Starknet را در یک محیط محلی انجام دهید، بنابراین به عنوان یک پلت فرم کارآمد برای توسعه و آزمایش عمل می کند.

پیشنهاد می کنیم از کاتانا یا starknet-devnet برای آزمایش قراردادهای خود استفاده کنید، که مورد دوم در فصل فرعی دیگری مورد بحث قرار گرفته است. starknet-devnet یک شبکه آزمایشی عمومی است که توسط تیم Shard Labs نگهداری می شود. هر دوی این ابزارها یک محیط موثر برای توسعه و آزمایش ارائه می دهند.

برای مثالی از نحوه استفاده از کاتانا برای استقرار و تعامل با یک قرارداد، به فصل فرعی این فصل یا نمونه قرارداد رأی گیری در کتاب قاهره مراجعه کنید.

### آشنایی با RPC در Starknet

تماس رویه از راه دور (RPC) ارتباط بین گره ها را در شبکه Starknet برقرار می کند. اساساً به ما امکان می دهد با یک گره در شبکه Starknet تعامل داشته باشیم. سرور RPC مسئول دریافت این تماس ها است.

RPC را می توان از منابع مختلف به دست آورد: . برای پشتیبانی از تمرکززدایی شبکه، می توانید از گره Starknet محلی خود استفاده کنید. برای سهولت دسترسی، استفاده از ارائه دهنده ای مانند Infura یا Alchemy را برای دریافت مشتری RPC در نظر بگیرید. . برای توسعه و آزمایش، می توان از یک گره محلی موقت مانند کاتانا استفاده کرد.

### شروع کار با کاتانا

برای نصب Katana، از نصب کننده dojoup از خط فرمان استفاده کنید:

```bash
curl -L https://install.dojoengine.org | bash
dojoup
```

پس از راه اندازی مجدد ترمینال، نصب را با استفاده از:

```bash
katana --version
```

برای ارتقای کاتانا، دستور نصب را دوباره اجرا کنید.

برای مقداردهی اولیه یک گره Starknet محلی، دستور زیر را اجرا کنید:

```bash
katana --accounts 3 --seed 0 --gas-price 250
```

پرچم --accounts تعداد حساب‌هایی را که باید ایجاد شود را تعیین می‌کند، در حالی که پرچم -seed پایه کلیدهای خصوصی این حساب‌ها را تعیین می‌کند. این تضمین می کند که مقداردهی اولیه گره با همان seed همیشه همان حساب ها را به همراه خواهد داشت. در نهایت، پرچم --gas-price قیمت گاز معامله را مشخص می کند.

با اجرای دستور خروجی مشابه این تولید می شود:

```
██╗  ██╗ █████╗ ████████╗ █████╗ ███╗   ██╗ █████╗
██║ ██╔╝██╔══██╗╚══██╔══╝██╔══██╗████╗  ██║██╔══██╗
█████╔╝ ███████║   ██║   ███████║██╔██╗ ██║███████║
██╔═██╗ ██╔══██║   ██║   ██╔══██║██║╚██╗██║██╔══██║
██║  ██╗██║  ██║   ██║   ██║  ██║██║ ╚████║██║  ██║
╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝


PREFUNDED ACCOUNTS
==================

| Account address |  0x03ee9e18edc71a6df30ac3aca2e0b02a198fbce19b7480a63a0d71cbd76652e0
| Private key     |  0x0300001800000000300000180000000000030000000000003006001800006600
| Public key      |  0x01b7b37a580d91bc3ad4f9933ed61f3a395e0e51c9dd5553323b8ca3942bb44e

| Account address |  0x033c627a3e5213790e246a917770ce23d7e562baa5b4d2917c23b1be6d91961c
| Private key     |  0x0333803103001800039980190300d206608b0070db0012135bd1fb5f6282170b
| Public key      |  0x04486e2308ef3513531042acb8ead377b887af16bd4cdd8149812dfef1ba924d

| Account address |  0x01d98d835e43b032254ffbef0f150c5606fa9c5c9310b1fae370ab956a7919f5
| Private key     |  0x07ca856005bee0329def368d34a6711b2d95b09ef9740ebf2c7c7e3b16c1ca9c
| Public key      |  0x07006c42b1cfc8bd45710646a0bb3534b182e83c313c7bc88ecf33b53ba4bcbc


ACCOUNTS SEED
=============
0


🚀 JSON-RPC server started: http://0.0.0.0:5050
```

خروجی شامل آدرس ها، کلیدهای خصوصی و کلیدهای عمومی حساب های ایجاد شده است. همچنین حاوی دانه‌هایی است که برای ایجاد حساب‌ها استفاده می‌شود. این seed می تواند برای ایجاد حساب های یکسان در اجراهای بعدی دوباره استفاده شود. علاوه بر این، خروجی URL سرور JSON-RPC را ارائه می دهد. از این URL می توان برای برقراری ارتباط با گره محلی Starknet استفاده کرد.

برای متوقف کردن گره محلی Starknet، به سادگی Ctrl+C را فشار دهید.

گره محلی Starknet داده ها را حفظ نمی کند. بنابراین، پس از توقف، تمام داده ها پاک می شوند.

برای نمایش عملی کاتانا برای استقرار و تعامل با یک قرارداد، به نمونه قرارداد رای گیری فصل 2 مراجعه کنید.
