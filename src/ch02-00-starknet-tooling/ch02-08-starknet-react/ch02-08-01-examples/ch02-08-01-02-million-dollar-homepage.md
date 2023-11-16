# صفحه اصلی میلیون دلاری

Starknet Homepage یک برنامه غیرمتمرکز در بلاک چین استارک نت است. این یک فضای مجازی فراهم می کند که در آن کاربران می توانند بخش هایی از یک شبکه 100x100 را که به عنوان "صفحه اصلی Starknet" شناخته می شود، ادعا و شخصی سازی کنند. هر بخش یک منطقه 10x10 پیکسل است. کاربران می‌توانند این بخش‌ها را با برش توکن‌های غیرقابل تعویض (NFT) و سپس شخصی‌سازی آن‌ها با تصاویر و محتوای دیگر به دست آورند.

برنامه زنده را در testnet [اینجا ](https://starknet-homepage-kappa.vercel.app/)مشاهده کنید.

![homepage](../../../img/ch02-starknet-homepage.jpg)

این ابتکار اقتباسی از صفحه اصلی میلیون دلاری مشهور است و در خانه هکرها در Starknet Summit 2023 در پالو آلتو، کالیفرنیا طراحی شده است. در زیر راهنمایی برای درک چگونگی توسعه این پروژه با استفاده از ابزارهای موجود در اکوسیستم ارائه شده است.

### ابزارهای مورد استفاده:

* [Starknet-react](https://github.com/apibara/starknet-react)
* [Starknet.js](https://github.com/0xs34n/starknet.js)
* [OpenZeppelin Cairo Contracts](https://github.com/OpenZeppelin/cairo-contracts)
* [MaterialUI](https://mui.com/material-ui/)

### راه اندازی اولیه

برنامه Starknet-react دستوری برای مقداردهی اولیه برنامه Starknet ارائه می دهد. این دستور ساختار اساسی مورد نیاز برای برنامه NextJS را تنظیم می کند.

```shell
npx create-starknet
```

مؤلفه StarknetConfig یک پایه اتصال دهنده را می پذیرد که گزینه های اتصال کیف پول را برای کاربر تعریف می کند. علاوه بر این، می‌تواند شبکه‌ای را که برنامه باید به‌طور پیش‌فرض به آن متصل شود، به یک پیش‌فرض‌دهنده نیاز دارد.

```javascript
const connectors = [
  new InjectedConnector({ options: { id: "braavos" } }),
  new InjectedConnector({ options: { id: "argentX" } }),
];
const provider = new Provider({
  sequencer: { network: constants.NetworkName.SN_GOERLI },
});
return (
  <StarknetConfig
    autoConnect
    defaultProvider={provider}
    connectors={connectors}
  >
    <CacheProvider value={emotionCache}>
      <ThemeProvider theme={theme}>
        <Component {...pageProps} />
      </ThemeProvider>
    </CacheProvider>
  </StarknetConfig>
);
```

هم CacheProvider و هم ThemeProvider اجزایی هستند که ادغام یکپارچه MaterialUI با NextJS را تسهیل می کنند. برای راهنمای راه اندازی جامع در مورد این قطعات، لطفاً به این لینک مراجعه کنید.

### عملکرد اصلی

عملکرد اصلی صفحه اصلی Starknet حول انتخاب یک منطقه 4 وجهی در یک ماتریس، نشان دهنده سلول های 10x10 مورد نظر، و برش توکن بر اساس آن سلول ها است. مسئولیت قرارداد هوشمند تأیید این است که آیا سلول های انتخاب شده برای ضرب در دسترس هستند یا خیر. اگر کاربر دارای نشانه‌های Starknet Homepage باشد، می‌تواند به یک منوی کشویی برای تغییر محتوای توکن، از جمله تصویر مرتبط و پیوند در شبکه دسترسی داشته باشد.

الزامات اولیه برنامه عبارتند از:

* اتصال کیف پول
* شبکه ای برای نمایش نشانه های موجود
* قابلیت انتخاب سلول
* عملکرد چند تماسی برای تایید و ضرب کردن رمز
* کشویی برای مشاهده توکن های متعلق به
* نمایش زنجیره ای کل شبکه 1 میلیون پیکسلی

یکی از جنبه های قابل توجه محدودیت رشته در قراردادهای قاهره است. برای ذخیره پیوندهایی با اندازه های مختلف، آنها به صورت آرایه هایی از felt252 ذخیره می شوند. قرارداد برای این منظور از منطق زیر استفاده می کند:

```rust
impl StoreFelt252Array of Store<Array<felt252>> {
    fn read(address_domain: u32, base: StorageBaseAddress) -> SyscallResult<Array<felt252>> {
        StoreFelt252Array::read_at_offset(address_domain, base, 0)
    }
    fn write(
        address_domain: u32, base: StorageBaseAddress, value: Array<felt252>
    ) -> SyscallResult<()> {
        StoreFelt252Array::write_at_offset(address_domain, base, 0, value)
    }
    fn read_at_offset(
        address_domain: u32, base: StorageBaseAddress, mut offset: u8
    ) -> SyscallResult<Array<felt252>> {
        let mut arr: Array<felt252> = ArrayTrait::new();
        // Read the stored array's length. If the length is superior to 255, the read will fail.
        let len: u8 = Store::<u8>::read_at_offset(address_domain, base, offset)
            .expect('Storage Span too large');

        offset += 1;

        // Sequentially read all stored elements and append them to the array.
        let exit = len + offset;
        loop {
            if offset >= exit {
                break;
            }
            let value = Store::<felt252>::read_at_offset(address_domain, base, offset).unwrap();
            arr.append(value);
            offset += Store::<felt252>::size();
        };
        Result::Ok(arr)
    }
    fn write_at_offset(
        address_domain: u32, base: StorageBaseAddress, mut offset: u8, mut value: Array<felt252>
    ) -> SyscallResult<()> {
        // // Store the length of the array in the first storage slot. 255 of elements is max
        let len: u8 = value.len().try_into().expect('Storage - Span too large');
        Store::<u8>::write_at_offset(address_domain, base, offset, len);
        offset += 1;
        // Store the array elements sequentially
        loop {
            match value.pop_front() {
                Option::Some(element) => {
                    Store::<felt252>::write_at_offset(address_domain, base, offset, element);
                    offset += Store::<felt252>::size();
                },
                Option::None => {
                    break Result::Ok(());
                }
            };
        }
    }
    fn size() -> u8 {
        255 / Store::<felt252>::size()
    }
}
```

روش ذخیره سازی پیوندها در حالت قرارداد به صورت زیر است:

```rust
struct Cell {
    token_id: u256,
    xpos: u8,
    ypos: u8,
    width: u8,
    height: u8,
    img: Array<felt252>,
    link: Array<felt252>,
}
```

کتابخانه OpenZeppelin Cairo Contracts نقش مهمی در سرعت بخشیدن به توسعه قرارداد ERC721 برای صفحه اصلی Starknet ایفا کرد. می توانید قرارداد را برای بررسی اینجا پیدا کنید. هنگامی که کتابخانه را نصب کردید، می توانید برای استفاده معمولی به مثال زیر مراجعه کنید:

```rust
#[starknet::contract]
mod MyToken {
    use starknet::ContractAddress;
    use openzeppelin::token::erc20::ERC20;
    #[storage]
    struct Storage {}
    #[constructor]
    fn constructor(
        ref self: ContractState,
        initial_supply: u256,
        recipient: ContractAddress
    ) {
        let name = 'MyToken';
        let symbol = 'MTK';
        let mut unsafe_state = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::initializer(ref unsafe_state, name, symbol);
        ERC20::InternalImpl::_mint(ref unsafe_state, recipient, initial_supply);
    }
    #[external(v0)]
    fn name(self: @ContractState) -> felt252 {
        let unsafe_state = ERC20::unsafe_new_contract_state();
        ERC20::ERC20Impl::name(@unsafe_state)
    }
    ...
}
```

### منطق مؤلفه

#### Grid

جزء Grid یک ماتریس 100x100 را نشان می دهد که هر سلول 100 پیکسل است. این طرح با ساختار داده موجود در قرارداد هوشمند مطابقت دارد. برای نمایش توکن هایی که قبلاً در صفحه اصلی ضرب شده اند، برنامه از یک React Hook از starknet-react برای فراخوانی تابع getAllTokens از قرارداد استفاده می کند.

```typescript
const [allNfts, setAllNfts] = useState<any[]>([]);
const { data, isLoading } = useContractRead({
  address: STARKNET_HOMEPAGE_ERC721_ADDRESS,
  functionName: "getAllTokens",
  abi: starknetHomepageABI,
  args: [],
});
useEffect(() => {
  if (!isLoading) {
    const arr = data?.map((nft) => {
      return deserializeTokenObject(nft);
    });
    setAllNfts(arr || []);
  }
}, [data, isLoading]);
```

Deserialization تضمین می کند که داده های قرارداد Starknet به درستی برای استفاده از فرانت اند تغییر شکل داده شده است. این فرآیند شامل رمزگشایی آرایه از felt252s به رشته های گسترده است.

```typescript
import { shortString, num } from "starknet";
const deserializeFeltArray = (arr: any) => {
    return arr
        .map((img: bigint) => {
            return shortString.decodeShortString(num.toHex(img));
        })
        .join("");
};
...
img: deserializeFeltArray(tokenObject.img),
link: deserializeFeltArray(tokenObject.link),
...
```

علاوه بر این، مولفه Grid فرآیند انتخاب سلول را مدیریت می‌کند که منجر به ایجاد یک توکن مربوطه می‌شود. هنگامی که یک منطقه انتخاب می شود، یک مدال ظاهر می شود که جزئیات ضرابخانه و سایر ورودی های لازم برای داده های تماس را نشان می دهد. پیچیدگی های چند تماس متعاقباً مورد بررسی قرار خواهد گرفت.

![Wallets](../../../img/ch02-starknet-homepage-select.jpg)

#### مدال ها

Modals ابزاری مناسب برای ارائه عملکردهای متنوع در برنامه، مانند اتصال کیف پول، برش توکن و ویرایش توکن ارائه می دهد.

![Wallets](../../../img/ch02-starknet-homepage-wallets.jpg)

بهترین روش شناخته شده این است که قلاب React را برای اطلاعات به اشتراک گذاشته شده در سطح بالا فراخوانی کنید تا اطمینان حاصل شود که اجزایی مانند WalletBar ساده و متمرکز باقی می مانند.

```typescript
const { address } = useAccount();

return (
    ...
    <WalletBar account={address} />
    ...
)
```

در زیر، تابع WalletConnected آدرس کیف پول متصل را نشان می دهد، در حالی که عملکرد ConnectWallet به کاربران اجازه می دهد تا کیف پول خود را انتخاب و متصل کنند. تابع WalletBar مودال مناسب را بر اساس وضعیت اتصال ارائه می کند.

```typescript
function WalletConnected({ address }: { address: string }) {
  const { disconnect } = useConnectors();
  const { chain } = useNetwork();
  const shortenedAddress = useMemo(() => {
    if (!address) return "";
    return `${address.slice(0, 6)}...${address.slice(-4)}`;
  }, [address]);

  return (
    <StyledBox>
      <StyledButton color="inherit" onClick={disconnect}>
        {shortenedAddress}
      </StyledButton>
      <span>&nbsp;Connected to {chain && chain.name}</span>
    </StyledBox>
  );
}

function ConnectWallet() {
  const { connectors, connect } = useConnectors();
  const [open, setOpen] = useState(false);
  const theme = useTheme();

  const handleClickOpen = () => {
    setOpen(true);
  };

  const handleClose = () => {
    setOpen(false);
  };

  return (
    <StyledBox>
      <StyledButton color="inherit" onClick={handleClickOpen}>
        Connect Wallet
      </StyledButton>
      <Dialog open={open} onClose={handleClose}>
        <DialogTitle>Connect to a wallet</DialogTitle>
        <DialogContent>
          <DialogContentText>
            <Grid container direction="column" alignItems="flex-start" gap={1}>
              {connectors.map((connector) => (
                <ConnectWalletButton
                  key={connector.id}
                  onClick={() => {
                    connect(connector);
                    handleClose();
                  }}
                  sx={{ margin: theme.spacing(1) }}
                >
                  {connector.id}
                  <Image
                    src={`/${connector.id}-icon.png`}
                    alt={connector.id}
                    width={30}
                    height={30}
                  />
                </ConnectWalletButton>
              ))}
            </Grid>
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleClose} color="inherit">
            Cancel
          </Button>
        </DialogActions>
      </Dialog>
    </StyledBox>
  );
}

export default function WalletBar({
  account,
}: {
  account: string | undefined;
}) {
  return account ? <WalletConnected address={account} /> : <ConnectWallet />;
}
```

### کشویی نشانه

مؤلفه کشویی به نمایش توکن های مرتبط با کیف پول فعلی متصل اختصاص داده شده است. برای بازیابی این توکن ها، تراکنشی مانند شکل زیر می تواند اجرا شود. تنها استدلال برای این تابع آدرس قرارداد مالک مورد نظر است.

```typescript
const readTx = useMemo(() => {
  const tx = {
    address: STARKNET_HOMEPAGE_ERC721_ADDRESS,
    functionName: "getTokensByOwner",
    abi: starknetHomepageABI,
    args: [account || "0x0000000"],
  };
  return tx;
}, [account]);

const { data, isLoading } = useContractRead(readTx);
```

### تعامل قرارداد چند تماسی

کد ارائه شده تصویری از یک تماس چندگانه را ارائه می دهد، به ویژه برای تأیید تراکنش برای انتقال قیمت ضرابخانه و به دنبال آن عمل ضربات واقعی. قابل توجه، ماژول shortString از starknet.js نقش محوری دارد. یک رشته طولانی را به آرایه‌ای از felt252 کدگذاری کرده و بخش‌بندی می‌کند، نوع آرگومان مورد انتظار برای قرارداد در Starknet.

useContractWrite یک هوک است که برای اجرای یک تماس چندگانه Starknet اختصاص داده شده است که می تواند برای یک تراکنش منفرد یا چند تراکنش استفاده شود.

```typescript
const calls = useMemo(() => {
  const splitNewImage: string[] = shortString.splitLongString(newImage);
  const splitNewLink: string[] = shortString.splitLongString(newLink);

  const tx2 = {
    contractAddress: STARKNET_HOMEPAGE_ERC721_ADDRESS,
    entrypoint: "mint",
    calldata: [
      startCell.col,
      startCell.row,
      width,
      height,
      splitNewImage,
      splitNewLink,
    ],
  };

  const price = selectedCells.length * 1000000000000000;

  const tx1 = {
    contractAddress: ERC_20_ADDRESS,
    entrypoint: "approve",
    calldata: [STARKNET_HOMEPAGE_ERC721_ADDRESS, `${price}`, "0"],
  };
  return [tx1, tx2];
}, [startCell, newImage, newLink, width, height, selectedCells.length]);

const { writeAsync: writeMulti } = useContractWrite({ calls });
```

یکی دیگر از جنبه های مهمی که باید به آن اشاره کرد، فراخوانی تابع تایید برای انتقال اتر است: calldata: \[STARKNET\_HOMEPAGE\_ERC721\_ADDRESS، '${price}'، "0"]،. آرگومان مقدار به دو قسمت تقسیم می‌شود، زیرا یک u256 است که از دو مقدار جداگانه felt252 تشکیل شده است.

هنگامی که چند تماس آماده شد، مرحله بعدی شروع عملکرد و امضای تراکنش با استفاده از کیف پول متصل است.

```typescript
const handleMintClick = async (): Promise<void> => {
  setIsMintLoading(true);
  try {
    await writeMulti();
    setIsMintLoading(false);
    setState((prevState) => ({
      ...prevState,
      showPopup: false,
      selectedCells: [],
      mintPrice: undefined,
    }));
  } catch (error) {
    console.error("Error approving transaction:", error);
  }
};
```

### چند فراخوانی شرطی برای ویرایش توکن

یکی دیگر از نمونه های آموزنده تنظیم چند تماس شرطی، حالتی است که برای اصلاح داده های مرتبط با یک نشانه استفاده می شود.

![homepage](../../../img/ch02-starknet-homepage-edit.jpg)

سناریوهایی وجود دارد که در آن کاربر ممکن است بخواهد به جای هر دو، فقط یک ویژگی نشانه را تغییر دهد. در نتیجه، یک پیکربندی چند فراخوانی شرطی ضروری می شود. یادآوری این نکته ضروری است که شناسه توکن در قرارداد قاهره به عنوان u256 تعریف شده است، به این معنی که شامل دو مقدار felt252 است.

```typescript
const calls = useMemo(() => {
  const txs = [];
  const splitNewImage: string[] = shortString.splitLongString(newImage);
  const splitNewLink: string[] = shortString.splitLongString(newLink);

  if (newImage !== "" && nft) {
    const tx1 = {
      contractAddress: STARKNET_HOMEPAGE_ERC721_ADDRESS,
      entrypoint: "setTokenImg",
      calldata: [nft.token_id, 0, splitNewImage],
    };
    txs.push(tx1);
  }

  if (newLink !== "" && nft) {
    const tx2 = {
      contractAddress: STARKNET_HOMEPAGE_ERC721_ADDRESS,
      entrypoint: "setTokenLink",
      calldata: [nft.token_id, 0, splitNewLink],
    };
    txs.push(tx2);
  }

  return txs;
}, [nft, newImage, newLink]);
```

#### نمای کلی صفحه اصلی Starknet

* Grid Component: یک ماتریس 100x100 را نشان می دهد که به کاربران اجازه می دهد سلول ها را انتخاب کرده و توکن های مربوطه را برش دهند. توکن های موجود را با استفاده از تابع getAllTokens از قرارداد دریافت می کند و آنها را نمایش می دهد.
* Modals: به عنوان رابط کاربری برای اقداماتی مانند اتصال کیف پول، استخراج توکن و ویرایش توکن خدمت می کند.
* Token Dropdown: توکن های مرتبط با کیف پول متصل را نمایش می دهد. این توکن ها را با استفاده از تابع getTokensByOwner بازیابی می کند.
* تعامل قرارداد چند تماسی: ضرب و ویرایش توکن را فعال می کند. این فرآیند از تماس‌های چندگانه مشروط بر اساس ترجیحات کاربر، به‌ویژه برای ویرایش ویژگی‌های نشانه استفاده می‌کند.

در سرتاسر پلتفرم، محدودیت‌های رشته در قراردادهای قاهره مستلزم رمزگذاری رشته‌های طولانی در آرایه‌های فلت252 است. کتابخانه OpenZeppelin Cairo Contracts توسعه قرارداد ERC721 را برای صفحه اصلی Starknet به طور قابل توجهی تسریع می کند.
