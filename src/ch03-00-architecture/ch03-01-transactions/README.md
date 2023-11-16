# معاملات

سفر یک تراکنش Starknet، از آغاز تا پایان آن، شامل مراحل خاصی است. هر مرحله انتقال، پردازش و ذخیره سازی دقیق داده ها در شبکه را تضمین می کند. این فصل مسیر تراکنش Starknet را مورد بحث قرار می دهد.

### ایجاد معامله

یک معامله با آماده سازی آن شروع می شود. فرستنده:

1. حساب آنها را یکسره جستجو می کند، که به عنوان یک شناسه منحصر به فرد برای تراکنش عمل می کند.
2. معامله را امضا می کند.
3. آن را به Node خود ارسال می کند.

گره، مشابه یک اداره پست، تراکنش را دریافت می کند و آن را در شبکه Starknet پخش می کند، در درجه اول برای Sequencer. با تکامل شبکه، تراکنش برای چندین Sequencer پخش خواهد شد.

شایان ذکر است که قبل از پخش تراکنش به Sequencer، درگاه‌ها اعتبارسنجی‌هایی را انجام می‌دهند، مانند بررسی اینکه حداکثر کارمزد از حداقل کارمزد بیشتر است و موجودی حساب بیشتر از حداکثر کارمزد است. اگر تابع اعتبارسنجی عبور کند، تراکنش در حافظه ذخیره می شود.

### نقش ترتیب ساز

پس از دریافت تراکنش، Sequencer دریافت آن را تأیید می کند اما هنوز آن را پردازش نکرده است - شبیه به حالت mempool اتریوم.

Sequencer's Process:

1. Receive the transaction.
2. Validate it.
3. Execute it.
4. Update the state.

Remember, Starknet processes transactions sequentially. The nonce won't change until the Sequencer processes the transaction. This can complicate backend application development, potentially causing errors if sending multiple transactions consecutively.

## Acceptance on Layer-2 (L2)

Once the Sequencer validates and executes a transaction, it updates the state without waiting for block creation. The transaction status changes from 'received' to 'accepted on L2' at this stage.

Following the state update, the transaction is included in a block. However, the block isn't emitted immediately. The Sequencer decides the opportune moment to emit the block, either when there are enough transactions to form a block or after a certain time has passed. When the block is emitted, the block becomes available for other Nodes to query.

Transaction Status Transition:

1. Received -> Accepted on L2

If a transaction fails during execution, it will be included in the block with the status 'reverted'.

It's essential to remember that at this stage, no proof has been generated, and the transaction relies on L2 consensus for security against censorship. There remains a slim possibility of transaction reversal if all Sequencers collude. Therefore, these stages should be seen as different layers of transaction finality.

## Acceptance on Layer-1 (L1)

The final step in the transaction's lifecycle is its acceptance on Layer-1 (L1). A Prover receives the block containing the transaction, re-executes the block, generates a proof, and sends it to Ethereum. Specifically, the proof is sent to a smart contract on Ethereum called the Verifier smart contract, which checks the proof's validity. If valid, the transaction's status changes to 'accepted on L1', signifying the transaction's security by Ethereum consensus.

Transaction Status Transition:

1. Accepted on L2 -> Accepted on L1

## \[Optional] Transaction Finality in Starknet

Transaction finality refers to the point at which a transaction is considered irreversible and is no longer susceptible to being reversed or undone. It's the assurance that once a transaction is committed, it can't be altered or rolled back, hence securing the integrity of the transaction and the system as a whole.

Let's dive into the transaction finality in both Starknet and Ethereum, and how they compare.

### Ethereum Transaction Finality

Ethereum operates on a Proof of Stake (PoS) consensus mechanism. A transaction has the finality status when it is part of a block that can't change without a significant amount of ETH getting burned. The number of blocks required to ensure that a transaction won't be rolled back is called 'blocks to finality', and the time to create those blocks is called 'time to finality'.

It is considered to be an average of 6 blocks to reach the finality status; given that a new block is validated each 12 seconds, the average time to finality for a transaction is 75 seconds.

### Starknet Transaction Finality

Starknet, a Layer-2 (L2) solution on Ethereum, has a two-step transaction finality process. The first step is when the transaction gets accepted on Layer-2 (Starknet), and the second step is when the transaction gets accepted on Layer-1 (Ethereum).

پذیرش در L2: هنگامی که یک تراکنش توسط Sequencer پردازش می شود و در یک بلوک در Starknet گنجانده می شود، به L2 نهایی می رسد. با این حال، این قطعیت به اجماع L2 متکی است و با خطر جزئی تبانی بین Sequencer ها که منجر به برگشت تراکنش می شود، همراه است. پذیرش در L1: نهایی شدن مطلق زمانی حاصل می‌شود که بلوک حاوی تراکنش یک اثبات تولید می‌کند، اثبات توسط قرارداد Verifier در اتریوم تأیید می‌شود و وضعیت در اتریوم به‌روزرسانی می‌شود. در این مرحله، تراکنش به همان اندازه ایمن است که اجماع PoW اتریوم می تواند ارائه دهد، به این معنی که تغییر یا معکوس کردن آن از نظر محاسباتی غیرممکن می شود.

### مقایسه

تفاوت اصلی بین نهایی بودن تراکنش اتریوم و استارک نت در مراحل نهایی شدن و اتکای آنها به مکانیسم های اجماع است.

با اضافه شدن بلاک های بیشتر، احتمال معکوس شدن نهایی تراکنش اتریوم به طور فزاینده ای کم می شود. فرآیند نهایی استارک نت دوگانه است. نهایی شدن اولیه (L2) سریعتر است اما بر اجماع L2 متکی است و خطر تبانی کوچکی را به همراه دارد. نهایی شدن نهایی (L1) کندتر است، زیرا شامل تولید و تأیید مدارک و به روز رسانی در اتریوم است. با این حال، پس از رسیدن به آن، همان سطح امنیت تراکنش اتریوم را فراهم می کند.

### رسیدگی به تراکنش های رد شده

در سناریوهای نادر، تراکنش می‌تواند توسط Sequencer رد شود، در صورتی که نامعتبر یا اشتباه باشد، حتی اگر دروازه‌ها قبلاً آن را تأیید کرده باشند.

انتقال وضعیت تراکنش

1. دریافت شد -> رد شد (در صورت نامعتبر یا اشتباه)

### رسیدگی به تراکنش های برگشتی

یک تراکنش می تواند به دلیل اجرای ناموفق برگردانده شود، تراکنش همچنان در یک بلوک قرار می گیرد و حساب برای منابع مصرف شده شارژ می شود.

این یک فرض اعتماد را برای Sequencer به صادق بودن و غیر سانسور بودن اضافه می کند. در نسخه‌های بعدی، یک تغییر سیستم‌عامل وجود خواهد داشت که Sequencer را قادر می‌سازد ثابت کند که یک تراکنش شکست خورده است و مقدار صحیح گاز را برای آن شارژ کند، بنابراین در برابر سانسور با تراکنش‌های شکست‌خورده مقاوم می‌شود.

انتقال وضعیت تراکنش

1. دریافت شد -> برگردانده شد

### خلاصه چرخه حیات تراکنش

موارد زیر مراحل مختلف چرخه عمر تراکنش را تشریح می کند:

![Transaction flow](../../img/ch03-transaction\_flow.png)

### نتیجه

چرخه حیات تراکنش استارک نت یک سفر با دقت تنظیم شده است که پردازش تراکنش کارآمد، ایمن و شفاف را تضمین می کند. این شامل همه چیز از ایجاد تراکنش، پردازش Sequencer، اعتبار سنجی لایه 2 و لایه 1، تا رسیدگی به تراکنش های رد شده و برگردانده شده است. با درک این چرخه حیات، توسعه دهندگان و کاربران می توانند بهتر در اکوسیستم Starknet حرکت کنند و از قابلیت های آن به طور کامل استفاده کنند.
