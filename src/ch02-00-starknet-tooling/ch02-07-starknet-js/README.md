# starknet-js

## Starknet-js: Javascript SDK

Starknet.js یک کتابخانه JavaScript/TypeScript است که برای اتصال وب سایت یا برنامه غیرمتمرکز شما (D-App) به Starknet طراحی شده است. هدف آن تقلید از معماری ethers.js است، بنابراین اگر با اترها آشنا هستید، باید کار با Starknet.js را آسان کنید.

![Starknet-js in your dapp](../../img/ch02-starknet-js.png)

Starknet-js در dapp شما

### نصب و راه اندازی

برای نصب Starknet.js مراحل زیر را دنبال کنید:

* برای آخرین نسخه رسمی (شاخه اصلی):

```
npm install starknet
```

* برای استفاده از آخرین ویژگی ها (ادغام در شاخه توسعه):

```
npm install starknet@next
```

### شروع شدن

برای ساخت اپلیکیشنی که کاربران قادر به اتصال به استارک نت و تعامل با آن باشند، توصیه می کنیم کتابخانه get-starknet را اضافه کنید که به شما امکان می دهد اتصالات کیف پول را مدیریت کنید.

با آماده بودن این ابزارها، اساساً 3 مفهوم اصلی وجود دارد که باید بدانید: حساب، ارائه دهنده و قراردادها.

#### حساب

ما به طور کلی می توانیم حساب را به عنوان "کاربر نهایی" یک dapp در نظر بگیریم و برخی از تعاملات کاربر برای دسترسی به آن دخیل خواهد بود.

برنامه‌ای را در نظر بگیرید که در آن کاربر کیف پول افزونه مرورگر خود را (مانند ArgentX یا Braavos) وصل می‌کند - اگر کاربر اتصال را بپذیرد، به ما امکان دسترسی به حساب و امضاکننده را می‌دهد که می‌تواند تراکنش‌ها و پیام‌ها را امضا کند.

برخلاف اتریوم، که در آن حساب‌های کاربری، حساب‌های تحت مالکیت خارجی هستند، حساب‌های Starknet قراردادی هستند. این ممکن است لزوماً تأثیری بر ظاهر dapp شما نداشته باشد، اما قطعاً باید از این تفاوت آگاه باشید.

```ts
async function connectWallet() {
    const starknet = await connect();
    console.log(starknet.account);

    const nonce = await starknet.account.getNonce();
    const message = await starknet.account.signMessage(...)
}
```

قطعه بالا از تابع اتصال ارائه شده توسط get-starknet برای برقراری ارتباط با کیف پول کاربر استفاده می کند. پس از اتصال، می‌توانیم به روش‌های حساب، مانند signMessage یا execute دسترسی پیدا کنیم.

#### ارائه دهنده

ارائه دهنده به شما امکان می دهد با شبکه Starknet تعامل داشته باشید. می توانید آن را به عنوان یک اتصال "خوانده" به بلاک چین در نظر بگیرید، زیرا اجازه امضای تراکنش ها یا پیام ها را نمی دهد. درست مانند اتریوم، می توانید از یک ارائه دهنده پیش فرض استفاده کنید یا از خدماتی مانند Infura یا Alchemy که هر دو از Starknet پشتیبانی می کنند، برای ایجاد یک ارائه دهنده RPC استفاده کنید.

به طور پیش فرض، Provider یک ارائه دهنده ترتیب دهنده است.

```ts
export const provider = new Provider({
  sequencer: {
    network: "goerli-alpha",
  },
  // rpc: {
  //   nodeUrl: INFURA_ENDPOINT
  // }
});

const block = await provider.getBlock("latest"); // <- Get latest block
console.log(block.block_number);
```

### قراردادها

ظاهر شما احتمالاً با قراردادهای مستقر در تعامل است. برای هر قرارداد، باید یک همتا در قسمت جلویی وجود داشته باشد. برای ایجاد این نمونه‌ها، به آدرس قرارداد و ABI و یک ارائه‌دهنده یا امضاکننده نیاز دارید.

```ts
const contract = new Contract(abi_erc20, contractAddress, starknet.account);

const balance = await contract.balanceOf(starknet.account.address);
const transfer = await contract.transfer(recipientAddress, amountFormatted);
//or: const transfer = await contract.invoke("transfer", [to, amountFormatted]);

console.log(`Tx hash: ${transfer.transaction_hash}`);
```

اگر یک نمونه قرارداد با یک ارائه دهنده ایجاد کنید، محدود به فراخوانی توابع خواندن در قرارداد خواهید بود - فقط با امضاکننده می توانید وضعیت بلاک چین را تغییر دهید. با این حال، شما می توانید یک نمونه قراردادی که قبلا ایجاد شده را با یک حساب جدید متصل کنید:

```ts
const contract = new Contract(abi_erc20, contractAddress, provider);

contract.connect(starknet.account);
```

در قطعه بالا، پس از فراخوانی متد اتصال، امکان فراخوانی توابع خواندن در قرارداد وجود دارد، اما نه قبل از آن.

#### واحدها

اگر تجربه قبلی با web3 دارید، می دانید که برخورد با واحدها نیاز به مراقبت دارد و Starknet نیز از این قاعده مستثنی نیست. یک بار دیگر، اسناد در اینجا بسیار مفید هستند، به ویژه این بخش در مورد تبدیل داده ها.

اغلب شما نیاز دارید که ساختارهای Cairo (مانند Uint256) را که از قراردادها برگردانده می شوند به اعداد تبدیل کنید:

```ts
// Uint256 shape:
// {
//    type: 'struct',
//    low: Uint256.low,
//    high: Uint256.high
//
// }
const balance = await contract.balanceOf(address); // <- uint256
const asBN = uint256.uint256ToBN(uint256); // <- uint256 into BN
const asString = asBN.toString(); //<- BN into string
```

و بالعکس:

```ts
const amount = 1;

const amountFormatted = {
  type: "struct",
  ...uint256.bnToUint256(amount),
};
```

علاوه بر bnToUint256 و uint256ToBN، ابزارهای مفید دیگری نیز وجود دارد که توسط Starknet.js ارائه شده است.

ما اکنون یک پایه محکم برای ایجاد یک برنامه Starknet داریم. با این حال، ابزارهای چارچوب خاصی وجود دارد که به ما در ساخت dapp های Starknet کمک می کند، که در فصل 5 پوشش داده شده است.

### منابع اضافی

* Starknet.js GitHub Repository: [https://github.com/0xs34n/starknet.js](https://github.com/0xs34n/starknet.js)
* وب سایت و مستندات رسمی Starknet.js: https://www.starknetjs.com/

منتظر به‌روزرسانی‌های بیشتر در Starknet.js، از جمله راهنماهای دقیق، مثال‌ها و مستندات جامع باشید.

کتاب یک تلاش جامعه محور است که برای جامعه ایجاد شده است.

* اگر چیزی یاد گرفته‌اید یا نه، لطفاً چند لحظه وقت بگذارید و از طریق این نظرسنجی 3 سؤالی بازخورد خود را ارائه دهید.
* در صورت کشف هرگونه خطا یا پیشنهادات اضافی، در باز کردن مشکل در مخزن GitHub ما تردید نکنید.
