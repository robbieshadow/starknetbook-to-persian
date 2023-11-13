# تایید کننده استحکام

قبل از غواصی در این فصل، برای درک جریان اساسی معماری، به فصل معماری Starknet مراجعه کنید. ما فرض می کنیم که شما درک اولیه ای از مفاهیم زیر دارید: Sequencers، Provers، SHARP و Sharp Jobs.

در دنیای جمع‌آوری‌ها، تأییدکنندگان کلید اطمینان از اعتماد و شفافیت هستند. این نقش تأیید کننده استحکام استارک نت است که صحت تراکنش ها و قراردادهای هوشمند را بررسی می کند.

### بررسی SHARP و Sharp Jobs

توجه: برای توضیح بیشتر در مورد SHARP و Sharp Jobs، به فصل فرعی Provers در فصل Starknet Architecture مراجعه کنید. این یک بررسی مختصر است.

SHARP یا Shared Prover در Starknet، برنامه‌های مختلف قاهره را از کاربران مجزا جمع‌آوری می‌کند. این برنامه ها که هر کدام منطق منحصر به فردی دارند، با هم اجرا می شوند و یک اثبات مشترک برای همه ایجاد می کنند و هزینه و کارایی را بهینه می کنند.

![](https://hackmd.io/\_uploads/HJ7UiFLfa.png)

علاوه بر این، SHARP از ترکیب چندین اثبات در یک مورد پشتیبانی می‌کند و کارایی آن را با اجازه دادن به پردازش و تأیید اثبات موازی افزایش می‌دهد.

SHARP بسیاری از تراکنش‌های Starknet مانند نقل و انتقالات، معاملات و به‌روزرسانی‌های ایالتی را تأیید می‌کند. همچنین اجرای قراردادهای هوشمند را تایید می کند.

برای نشان دادن SHARP: به رفت و آمد با اتوبوس فکر کنید. راننده اتوبوس، پروور، مسافران را جابجا می کند، برنامه های قاهره. راننده فقط بلیط های مسافرانی را که در ایستگاه آینده پیاده می شوند، بررسی می کند، دقیقاً مانند SHARP. Prover یک اثبات واحد برای همه برنامه‌های قاهره به صورت دسته‌ای تشکیل می‌دهد، اما فقط اثبات‌های برنامه‌هایی را که در بلوک بعدی اجرا می‌شوند تأیید می‌کند.

شغل های شارپ Sharp Jobs که به عنوان Shared Prover Jobs شناخته می شود، به چندین کاربر اجازه می دهد برنامه های Cairo خود را برای اجرای ترکیبی ارائه دهند و هزینه تولید اثبات را توزیع کنند. این رویکرد مشترک، Starknet را برای کاربران مقرون به صرفه‌تر می‌کند و آنها را قادر می‌سازد تا به مشاغل مداوم بپیوندند و از صرفه‌جویی در مقیاس استفاده کنند.

### تایید کننده های استحکام

تأیید کننده Solidity یک قرارداد هوشمند L1 است که در Solidity ساخته شده است و برای تأیید اثبات STARK از SHARP (Shared Prover) طراحی شده است.

### معماری قبلی: تایید کننده یکپارچه

از لحاظ تاریخی، تأیید کننده استحکام یک قرارداد یکپارچه بود، که هم توسط یک قرارداد آغاز و هم اجرا می‌شد. برای مثال، اپراتور تابع وضعیت به روز رسانی را در قرارداد اصلی فراخوانی می کند و وضعیت را برای اصلاح و تأیید اعتبار آن ارائه می کند. متعاقباً، قرارداد اصلی مدرک را به تأیید کننده و کمیته اعتبار ارائه می کند. هنگامی که آنها مدرک را تأیید کردند، ایالت در قرارداد اصلی به روز می شود.

![](https://hackmd.io/\_uploads/BJNEAKIzT.png)

However, this architecture faced several constraints:

* Batching transactions frequently surpassed the original geth32kb transaction size limit (later adjusted to 128kb) due to accumulating excessive transactions.
* The gas required often outstripped the block size (e.g., 8 Mgas), as the block couldn't accommodate a complete batch of proof.
* A prospective constraint was that the verifier wouldn't support proof bundling, which is fundamental for SHARP.

### Current Architecture: Multiple Smart Contracts

The current verifier utilizes multiple smart contracts rather than being a singular, monolithic structure.

Here are some key smart contracts associated with the verifier:

* [`GpsStatementVerifier`](https://etherscan.io/address/0x47312450b3ac8b5b8e247a6bb6d523e7605bdb60): This is the primary contract of the Sharp verifier. It verifies a proof and then registers the related facts using `verifyProofAndRegister`. It acts as an umbrella for various layouts, each named `CpuFrilessVerifier`. Every layout has a unique combination of built-in resources.

![](https://hackmd.io/\_uploads/SyqKDqLzT.png)

The system routes each proof to its relevant layout.

* [`MemoryPageFactRegistry`](https://etherscan.io/address/0xfd14567eaf9ba941cb8c8a94eec14831ca7fd1b4): This registry maintains facts for memory pages, primarily used to register outputs for data availability in rollup mode. The Fact Registry is a separate smart contract ensuring the verification and validity of attestations or facts. The verifier function is separated from the main contract to ensure each segment works optimally within its limits. The main proof segment relies on other parts, but these parts operate independently.
* [`MerkleStatementContract`](https://etherscan.io/address/0x5899efea757e0dbd6d114b3375c23d7540f65fa4): This contract verifies merkle paths.
* [`FriStatementContract`](https://etherscan.io/address/0x3e6118da317f7a433031f03bb71ab870d87dd2dd): It focuses on verifying the FRI layers.

### Sharp Verifier Contract Map

![](https://hackmd.io/\_uploads/r1Re\_qUG6.png) ![](https://hackmd.io/\_uploads/HkkOOc8M6.png)

### پارامترهای سازنده قراردادهای کلیدی

CpuFrilessVerifiers و GpsStatementVerifier قراردادهایی هستند که پارامترهای سازنده را می پذیرند. در اینجا پارامترهایی هستند که به سازنده آنها ارسال می شود:

![](https://hackmd.io/\_uploads/rJgPt5UMp.png)

### جریان تأیید صریح

![](https://hackmd.io/\_uploads/ByPO5qUMa.png)

1. توزیع کننده شارپ تمام تراکنش های ضروری را برای تأیید ارسال می کند، از جمله: الف. MemoryPages (معمولاً تعداد زیادی). ب MerkleStatements (معمولاً بین 3 تا 5). ج. FriStatements (به طور کلی از 5 تا 15 متغیر است).
2. سپس توزیع کننده شارپ با استفاده از verifyProofAndRegister، اثبات را ارسال می کند.
3. برنامه هایی مانند مانیتور Starknet وضعیت را تأیید می کنند. پس از تکمیل تأیید، آنها یک معامله UpdateState ارسال می کنند.

### نتیجه

Starknet تأیید کننده Solidity را از یک واحد به یک سیستم انعطاف پذیر و چند قراردادی تبدیل کرد و تمرکز آن بر مقیاس پذیری و کارایی را برجسته کرد. Starknet با استفاده از SHARP و مراحل تأیید صحت، اطمینان حاصل می کند که Solidity Verifier سنگ بنای قوی در راه اندازی خود باقی می ماند.
