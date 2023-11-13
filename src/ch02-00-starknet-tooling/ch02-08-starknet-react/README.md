# Starknet-React: React Integration

ابزارهای متعددی در اکوسیستم starknet وجود دارد که می‌توانند قسمت جلویی برنامه شما را بسازند. محبوب ترین آنها عبارتند از:

* starknet-react (مستند): مجموعه ای از قلاب های React برای Starknet. این برنامه از wagmi الهام گرفته شده است که توسط starknet.js طراحی شده است.
* starknet.js: یک کتابخانه جاوا اسکریپت برای تعامل با قراردادهای Starknet. این معادل web3.js برای اتریوم خواهد بود.

برای توسعه دهندگان Vue، vue-stark-boil که توسط تیم Don’t Panic DAO ایجاد شده است، گزینه بسیار خوبی است. برای درک عمیق تر از Vue، از وب سایت آنها بازدید کنید. صفحه دیگ بخار vue-stark-boil عملکردهای مختلفی مانند اتصال به کیف پول، گوش دادن به تغییرات حساب و برقراری قرارداد را امکان پذیر می کند.

Starknet React که توسط تیم Apibara نوشته شده است مجموعه ای منبع باز از ارائه دهندگان React و قلاب هایی است که به طور دقیق برای Starknet طراحی شده اند.

برای غوطه ور شدن در برنامه دنیای واقعی Starknet React، توصیه می کنیم نمونه جامع پروژه dApp را در starknet-demo-dapp بررسی کنید.

### ادغام Starknet React

شروع سفر Starknet React شما نیاز به ادغام وابستگی های حیاتی دارد. بیایید با اضافه کردن آنها به پروژه خود شروع کنیم.

```
yarn add @starknet-react/core starknet get-starknet
```

Starknet.js یک SDK ضروری است که تعامل با Starknet را تسهیل می کند. در مقابل، get-starknet بسته ای است که در مدیریت اتصالات کیف پول ماهر است.

با قنداق کردن برنامه خود در مولفه StarknetConfig ادامه دهید. این عمل پوشاننده درجه ای از پیکربندی را ارائه می دهد، در حالی که همزمان یک زمینه React را برای برنامه زیر برای استفاده از داده ها و قلاب های مشترک ارائه می دهد. مؤلفه StarknetConfig یک پایه اتصال دهنده را می پذیرد که امکان تعریف گزینه های اتصال کیف پول را برای کاربر فراهم می کند.

```
const connectors = [
  new InjectedConnector({ options: { id: "braavos" } }),
  new InjectedConnector({ options: { id: "argentX" } }),
];

return (
    <StarknetConfig
      connectors={connectors}
      autoConnect
    >
      <App />
    </StarknetConfig>
)
```

### ایجاد اتصال و مدیریت حساب

هنگامی که کانکتورها در پیکربندی تعریف شدند، مرحله تنظیم می شود تا از یک قلاب برای دسترسی به این کانکتورها استفاده کند و کاربران را قادر می سازد کیف پول خود را به هم متصل کنند:

```
export default function Connect() {
  const { connect, connectors, disconnect } = useConnectors();

  return (
    <div>
      {connectors.map((connector) => (
        <button
          key={connector.id}
          onClick={() => connect(connector)}
          disabled={!connector.available()}
        >
          Connect with {connector.id}
        </button>
      ))}
    </div>
  );
}
```

تابع قطع اتصال را که هنگام فراخوانی اتصال را قطع می کند، مشاهده کنید. اتصال پست، دسترسی به حساب متصل از طریق قلاب useAccount فراهم می‌شود و بینشی از وضعیت فعلی اتصال ارائه می‌دهد:

```
const { address, isConnected, isReconnecting, account } = useAccount();

return (
    <div>
      {isConnected ? (
          <p>Hello, {address}</p>
      ) : (
        <Connect />
      )}
    </div>
);
```

مقادیر حالت، مانند isConnected و isReconnecting، به‌روزرسانی‌های خودکار را دریافت می‌کنند و به‌روزرسانی‌های شرطی UI را ساده می‌کنند. این الگوی مناسب هنگام برخورد با فرآیندهای ناهمزمان می درخشد، زیرا نیازی به مدیریت دستی حالت در اجزای شما را از بین می برد.

پس از برقراری ارتباط، امضای پیام‌ها با استفاده از مقدار حساب بازگردانده شده از قلاب useAccount به سرعت تبدیل می‌شود. برای تجربه‌ای ساده‌تر، قلاب useSignTypedData در اختیار شماست.

```
const { data, signTypedData } = useSignTypedData(typedMessage)

return (
  <>
    <p>
      <button onClick={signTypedData}>Sign</button>
    </p>
    {data && <p>Signed: {JSON.stringify(data)}</p>}
  </>
)
```

Starknet React از امضای آرایه ای از مقادیر BigNumberish یا یک شی پشتیبانی می کند. هنگام امضای یک شی، اطمینان از مطابقت داده ها با نوع EIP712 بسیار مهم است. برای راهنمای جامع تر در مورد امضا، به مستندات Starknet.js مراجعه کنید: اینجا.

### نمایش StarkName

پس از اتصال یک حساب، می توان از قلاب useStarkName برای بازیابی StarkName این حساب متصل استفاده کرد. مربوط به Starknet.id اجازه می دهد تا آدرس کاربر را به روشی کاربر پسندتر نمایش دهد.

```
const { data, isError, isLoading, status } = useStarkName({ address });
// You can track the status of the request with the status variable ('idle' | 'error' | 'loading' | 'success')

if (isLoading) return <p>Loading...</p>
return <p>Account: {isError ? address : data}</p>
```

شما همچنین اطلاعات بیشتری دارید که می توانید از این قلاب دریافت کنید → خطا، isIdle، isFetching، isSuccess، isFetched، isFetchedAfterMount، isRefetching، refetch که می تواند اطلاعات دقیق تری در مورد آنچه اتفاق می افتد به شما بدهد.

### در حال واکشی آدرس از StarkName

همچنین می‌توانید یک آدرس مربوط به StarkName را بازیابی کنید. برای این منظور می توانید از قلاب useAddressFromStarkName استفاده کنید.

```
const { data, isLoading, isError } = useAddressFromStarkName({ name: 'vitalik.stark' })

if (isLoading) return <p>Loading...</p>
if (isError) return <p>Something went wrong</p>
return <p>Address: {data}</p>
```

اگر نام ارائه شده دارای آدرس مرتبط نباشد، "0x0" را برمی گرداند.

### پیمایش در شبکه

علاوه بر مدیریت کیف پول و حساب، Starknet React توسعه دهندگان را به قلاب هایی برای تعاملات شبکه مجهز می کند. به عنوان مثال، useBlock بازیابی آخرین بلوک را فعال می کند:

```
const { data, isError, isFetching } = useBlock({
    refetchInterval: 10_000,
    blockIdentifier: "latest",
});

if (isError) {
  return (
    <p>Something went wrong</p>
  )
}

return (
    <p>Current block: {isFetching ? "Loading..." : data?.block_number}<p>
)
```

در کد فوق، refetchInterval فرکانس واکشی مجدد داده ها را کنترل می کند. در پشت صحنه، Starknet React از react-query برای مدیریت وضعیت و کوئری ها استفاده می کند. علاوه بر useBlock، Starknet React قلاب‌های دیگری مانند useContractRead و useWaitForTransaction را ارائه می‌دهد که می‌توانند برای به‌روزرسانی در فواصل زمانی منظم پیکربندی شوند.

قلاب useStarknet دسترسی مستقیم به ProviderInterface را فراهم می کند:

```
const { library } = useStarknet();

// library.getClassByHash(...)
// library.getTransaction(...)
```

### پیگیری تغییرات کیف پول

برای بهبود تجربه کاربری dApp خود، می توانید تغییرات کیف پول کاربر را ردیابی کنید، به خصوص زمانی که کاربر حساب کیف پول را تغییر می دهد (یا وصل/قطع می شود). اما همچنین زمانی که کاربر شبکه را تغییر می دهد. می‌توانید وقتی کاربر حساب را تغییر می‌دهد، موجودی‌های صحیح را دوباره بارگیری کنید، یا وقتی کاربر شبکه را تغییر می‌دهد، وضعیت dApp خود را بازنشانی کنید. برای انجام این کار، می توانید از یک قلاب قبلی که قبلاً به آن نگاه کرده بودیم استفاده کنید: useAccount و یک مورد جدید useNetwork.

قلاب useNetwork می تواند زنجیره شبکه ای را که در حال حاضر استفاده می شود در اختیار شما قرار دهد.

```
const { chain: {id, name} } = useNetwork();

return (
    <>
        <p>Connected chain: {name}</p>
        <p>Connected chain id: {id}</p>
    </>
)
```

شما همچنین اطلاعات بیشتری دارید که می توانید از این قلاب ← blockExplorer، شبکه آزمایشی دریافت کنید که می تواند اطلاعات دقیق تری در مورد شبکه فعلی مورد استفاده به شما بدهد.

پس از دانستن این موضوع، تمام آنچه را که برای ردیابی تعامل کاربر در حساب کاربری و شبکه نیاز دارید دارید. می‌توانید از قلاب useEffect برای انجام برخی تغییرات استفاده کنید.

```
const { chain } = useNetwork();
const { address } = useAccount();

useEffect(() => {
    if(address) {
        // Do some work when the user changes the account on the wallet
        // Like reloading the balances
    }else{
        // Do some work when the user disconnects the wallet
        // Like reseting the state of your dApp
    }
}, [address]);

useEffect(() => {
    // Do some work when the user changes the network on the wallet
    // Like reseting the state of your dApp
}, [chain]);
```

### تعاملات قراردادی

توابع را بخوانید

Starknet React useContractRead را ارائه می کند، یک قلاب تخصصی برای فراخوانی توابع خواندن در قراردادها، شبیه به wagmi. این قلاب مستقل از وضعیت اتصال کاربر عمل می کند، زیرا عملیات خواندن نیازی به امضا کننده ندارد.

```
const { data: balance, isLoading, isError, isSuccess } = useContractRead({
    abi: abi_erc20,
    address: CONTRACT_ADDRESS,
    functionName: "allowance",
    args: [owner, spender],
    // watch: true <- refresh at every block
});
```

برای عملیات ERC20، Starknet React یک قلاب استفاده مناسب برای تعادل ارائه می دهد. این قلاب شما را از گذراندن یک ABI معاف می‌کند و مقدار تعادل با فرمت مناسب را برمی‌گرداند.

```
  const { data, isLoading } = useBalance({
    address,
    token: CONTRACT_ADDRESS, // <- defaults to the ETH token
    // watch: true <- refresh at every block
  });

  return (
    <p>Balance: {data?.formatted} {data?.symbol}</p>
  )
```

### توابع را بنویسید

قلاب useContractWrite که برای عملیات نوشتن طراحی شده است، کمی از wagmi منحرف می شود. معماری منحصر به فرد Starknet تراکنش های چند تماسی را به صورت بومی در سطح حساب تسهیل می کند. این ویژگی تجربه کاربر را هنگام اجرای چندین تراکنش افزایش می دهد و نیازی به تایید هر تراکنش به صورت جداگانه را از بین می برد. Starknet React از طریق قلاب useContractWrite از این قابلیت استفاده می کند. در زیر نمایشی از کاربرد آن است:

```
const calls = useMemo(() => {
    // compile the calldata to send
    const calldata = stark.compileCalldata({
      argName: argValue,
    });

    // return a single object for single transaction,
    // or an array of objects for multicall**
    return {
      contractAddress: CONTRACT_ADDRESS,
      entrypoint: functionName,
      calldata,
    };
}, [argValue]);


// Returns a function to trigger the transaction
// and state of tx after being sent
const { write, isLoading, data } = useContractWrite({
    calls,
});

function execute() {
  // trigger the transaction
  write();
}

return (
  <button type="button" onClick={execute}>
    Make a transaction
  </button>
)
```

قطعه کد با کامپایل کردن calldata با استفاده از ابزار compileCalldata ارائه شده توسط Starknet.js آغاز می شود. این فراخوانی، همراه با آدرس قرارداد و نقطه ورود، به قلاب useContractWrite منتقل می شود. هوک یک تابع نوشتن را برمی گرداند که متعاقباً برای اجرای تراکنش استفاده می شود. هوک همچنین هش و وضعیت تراکنش را ارائه می دهد.

### یک نمونه قرارداد واحد

در موارد استفاده خاص، کار با یک نمونه قرارداد ممکن است به تعیین آدرس قرارداد و ABI در هر هوک ترجیح داده شود. Starknet React این نیاز را با قلاب useContract برآورده می کند:

```
const { contract } = useContract({
    address: CONTRACT_ADDRESS,
    abi: abi_erc20,
});

// Call functions directly on contract
// contract.transfer(...);
// contract.balanceOf(...);
```

### ردیابی تراکنش ها

قلاب useTransaction امکان ردیابی وضعیت های تراکنش را با توجه به هش تراکنش فراهم می کند. این قلاب یک کش از تمام تراکنش ها را حفظ می کند و در نتیجه درخواست های اضافی شبکه را به حداقل می رساند.

```
const { data, isLoading, error } = useTransaction({ hash: txHash });

return (
  <pre>
    {JSON.stringify(data?.calldata)}
  </pre>
)
```

آرایه کامل قلاب‌های موجود را می‌توانید در مستندات Starknet React، در اینجا پیدا کنید: https://apibara.github.io/starknet-react/.

## Conclusion

کتابخانه Starknet React مجموعه جامعی از قلاب‌ها و ارائه‌دهندگان React را ارائه می‌دهد که به‌طور هدفمند برای Starknet و Starknet.js SDK ساخته شده‌اند. با بهره‌گیری از این ابزارهای خوش ساخت، توسعه‌دهندگان می‌توانند برنامه‌های غیرمتمرکز قوی بسازند که از قدرت شبکه Starknet بهره می‌برد.

Starknet React از طریق کار مجدانه توسعه دهندگان و مشارکت کنندگان اختصاصی به تکامل خود ادامه می دهد. ویژگی‌ها و بهینه‌سازی‌های جدید مرتباً اضافه می‌شوند و اکوسیستم پویا و رو به رشدی از برنامه‌های غیرمتمرکز را تقویت می‌کنند.

این یک سفر جذاب، پر از فناوری نوآورانه، فرصت های بی پایان، و جامعه رو به رشدی از افراد پرشور است. به‌عنوان یک توسعه‌دهنده، شما نه تنها برنامه‌های کاربردی می‌سازید، بلکه به پیشرفت یک شبکه جهانی و غیرمتمرکز کمک می‌کنید.

سوال دارید یا به کمک احتیاج دارید؟ انجمن Starknet همیشه آماده کمک است. به Starknet Discord بپیوندید یا مخزن GitHub StarknetBook را برای منابع و پشتیبانی کاوش کنید.

### بیشتر خواندن

* [Starknet.js](https://starknet.js.org)
* [Starknet React Docs](https://www.apibara.com/starknet-react-docs)
* [Mastering Ethereum](https://github.com/ethereumbook/ethereumbook)
* [Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook)

کتاب یک تلاش جامعه محور است که برای جامعه ایجاد شده است.

* اگر چیزی یاد گرفته‌اید یا نه، لطفاً چند لحظه وقت بگذارید و از طریق این نظرسنجی 3 سؤالی بازخورد خود را ارائه دهید.
* در صورت کشف هرگونه خطا یا پیشنهادات اضافی، در باز کردن مشکل در مخزن GitHub ما تردید نکنید.
