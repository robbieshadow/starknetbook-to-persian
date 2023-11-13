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

The app's primary requirements are:

* Wallet connectivity
* Grid for displaying existing tokens
* Cell selection capability
* Multicall function for token approval and minting
* Dropdown to view owned tokens
* On-chain representation of the entire 1 million pixel grid

A significant aspect to consider is the string limitation in Cairo contracts. To store links of varying sizes, they are stored as arrays of `felt252`s. The contract uses the following logic for this purpose:

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

The storage method for links in the contract state is structured as:

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

The OpenZeppelin Cairo Contracts library played a crucial role in speeding up the development of the ERC721 contract for Starknet Homepage. You can find the contract for review [here](https://github.com/dbejarano820/starknet\_homepage/blob/main/cairo\_contracts/src/ERC721.cairo). Once you have installed the library, you can refer to the following example for typical usage:

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

### Component Logic

#### Grid

The Grid component represents a 100x100 matrix, with each cell being 100 pixels. This layout corresponds to the data structure found in the smart contract. To showcase the tokens already minted on the Homepage, the app employs a React Hook from `starknet-react` to invoke the `getAllTokens` function from the contract.

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

Deserialization ensures the data from the Starknet contract is aptly transformed for frontend use. This process involves decoding the array of `felt252`s into extensive strings.

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

Furthermore, the Grid component manages the cell selection process, leading to the minting of a corresponding token. Once an area is chosen, a modal appears displaying the mint details and other necessary inputs for the call data. The intricacies of the multicall will be addressed subsequently.

![Wallets](../../../img/ch02-starknet-homepage-select.jpg)

#### Modals

Modals offer a convenient means to present varied functionalities within the app, such as wallet connection, token minting, and token editing.

![Wallets](../../../img/ch02-starknet-homepage-wallets.jpg)

A recognized best practice is to invoke the React hook for shared information at a top-level, ensuring components like the `WalletBar` remain streamlined and focused.

```typescript
const { address } = useAccount();

return (
    ...
    <WalletBar account={address} />
    ...
)
```

Below, the `WalletConnected` function displays the connected wallet's address, while the `ConnectWallet` function allows users to select and connect their wallet. The `WalletBar` function renders the appropriate modal based on the connection status.

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

#### Token Dropdown

The dropdown component is dedicated to showcasing the tokens associated with the currently connected wallet. To retrieve these tokens, a transaction like the one shown below can be executed. The sole argument for this function is the contract address of the intended owner.

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

### Multicall Contract Interaction

The provided code offers an illustration of a multicall, specifically to approve a transaction for the mint price transfer followed by the actual minting action. Notably, the `shortString` module from `starknet.js` plays a pivotal role; it encodes and segments a lengthy string into an array of `felt252`s, the expected argument type for the contract on Starknet.

The `useContractWrite` is a Hook dedicated to executing a Starknet multicall, which can be employed for a singular transaction or multiple ones.

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

Another crucial aspect to point out is the `calldata` of the approve function for the ether transfer: calldata: `[STARKNET_HOMEPAGE_ERC721_ADDRESS, '${price}', "0"],`. The amount argument is split into two parts because it's a `u256`, which is composed of two separate `felt252` values.

Once the multicall is prepared, the next step is to initiate the function and sign the transaction using the connected wallet.

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

### Conditional Multicall for Token Editing

Another instructive illustration of a conditional multicall setup is the modal used to modify the data associated with a token.

![homepage](../../../img/ch02-starknet-homepage-edit.jpg)

There are scenarios where the user may wish to alter just one attribute of the token, rather than both. Consequently, a conditional multicall configuration becomes necessary. It's essential to recall that the token id in the Cairo contract is defined as a `u256`, implying it comprises two `felt252` values.

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

## Starknet Homepage Overview

* **Grid Component**: Represents a 100x100 matrix, allowing users to select cells and mint corresponding tokens. It fetches existing tokens using the `getAllTokens` function from the contract and displays them.
* **Modals**: Serve as the user interface for actions like wallet connection, token minting, and token editing.
* **Token Dropdown**: Displays tokens associated with a connected wallet. It retrieves these tokens using the `getTokensByOwner` function.
* **Multicall Contract Interaction**: Enables token minting and editing. This process utilizes conditional multicalls based on user preferences, especially for editing token attributes.

Throughout the platform, string limitations in Cairo contracts require encoding lengthy strings into arrays of `felt252`s. The OpenZeppelin Cairo Contracts library significantly expedites the development of the ERC721 contract for the Starknet Homepage.
