# نصب و راه اندازی

این فصل شما را در راه اندازی ابزارهای توسعه Starknet خود راهنمایی می کند.

ابزارهای ضروری برای نصب:

1. [Starkli](https://github.com/xJonathanLEI/starkli) -&#x20;
   * یک ابزار CLI برای تعامل با Starknet. ابزارهای بیشتر در فصل 2 مورد بحث قرار گرفته است.
2. [Scarb](https://github.com/software-mansion/scarb) - مدیر بسته قاهره که کد را به Sierra، یک زبان سطح متوسط بین قاهره و CASM، جمع‌آوری می‌کند.
3. [Katana](https://github.com/dojoengine/dojo) -Katana یک گره Starknet است که برای توسعه محلی ساخته شده است.

برای پشتیبانی یا سوالات، به مشکلات GitHub ما مراجعه کنید یا با espejelomar در تلگرام تماس بگیرید.

### نصب استارکلی

با استفاده از Starkliup، نصب کننده ای که از طریق خط فرمان فراخوانی می شود، به راحتی Starkli را نصب کنید.

```bash
curl https://get.starkli.sh | sh
starkliup
```

ترمینال خود را مجددا راه اندازی کنید و نصب را تایید کنید:

```bash
starkli --version
```

برای ارتقای Starkli کافی است مراحل را تکرار کنید.

نصب Scarb Package Manager

بعداً در این فصل به اسکارب عمیق تر خواهیم پرداخت. فعلاً مراحل نصب را مرور خواهیم کرد.

برای macOS و Linux:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://docs.swmansion.com/scarb/install.sh | sh
```

برای ویندوز، تنظیمات دستی را در مستندات Scarb دنبال کنید.

ترمینال را مجددا راه اندازی کنید و اجرا کنید:

```bash
scarb --version
```

برای ارتقاء Scarb، دستور نصب را دوباره اجرا کنید.

### نصب گره کاتانا

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

اکنون می‌توانید در قاهره کدنویسی کنید و در Starknet مستقر شوید.
