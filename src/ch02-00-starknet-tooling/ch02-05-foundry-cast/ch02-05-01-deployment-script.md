# نمونه اسکریپت استقرار

این آموزش نحوه راه اندازی یک محیط تست و استقرار برای قراردادهای هوشمند را توضیح می دهد. اسکریپت داده شده حساب ها را مقداردهی اولیه می کند، آزمایش ها را اجرا می کند و چند تماس را انجام می دهد.

سلب مسئولیت: این یک مثال است. از آن به عنوان پایه ای برای کار خود استفاده کنید و در صورت لزوم تنظیم کنید.

### برپایی



### 1.فایل اسکریپت را آماده کنید

* در پوشه ریشه پروژه خود، یک فایل به نام script.sh ایجاد کنید. این اسکریپت را در خود جای خواهد داد.
* مجوزها را تنظیم کنید تا فایل قابل اجرا باشد:

```sh
chmod +x script.sh
```

### 2. اسکریپت را درج کنید

در زیر محتوای script.sh آمده است. به بهترین شیوه ها برای وضوح، مدیریت خطا و پشتیبانی طولانی مدت پایبند است.

نکته امنیتی: استفاده از متغیرهای محیطی امن‌تر از رمزگذاری کلیدهای خصوصی در اسکریپت‌های شما است، اما همچنان برای هر فرآیندی در دستگاه شما قابل دسترسی هستند و ممکن است در گزارش‌ها یا پیام‌های خطا به بیرون درز کنند.

```sh
#!/usr/bin/env bash

# Ensure the script stops on first error
set -e

# Global variables
file_path="$HOME/.starknet_accounts/starknet_open_zeppelin_accounts.json"
CONTRACT_NAME="HelloStarknet"
PROFILE_NAME="account1"
MULTICALL_FILE="multicall.toml"
FAILED_TESTS=false

# Addresses and Private keys as environment variables
ACCOUNT1_ADDRESS=${ACCOUNT1_ADDRESS:-"0x7f61fa3893ad0637b2ff76fed23ebbb91835aacd4f743c2347716f856438429"}
ACCOUNT2_ADDRESS=${ACCOUNT2_ADDRESS:-"0x53c615080d35defd55569488bc48c1a91d82f2d2ce6199463e095b4a4ead551"}
ACCOUNT1_PRIVATE_KEY=${ACCOUNT1_PRIVATE_KEY:-"CHANGE_ME"}
ACCOUNT2_PRIVATE_KEY=${ACCOUNT2_PRIVATE_KEY:-"CHANGE_ME"}

# Utility function to log messages
function log_message() {
    echo -e "\n$1"
}

# Step 1: Clean previous environment
if [ -e "$file_path" ]; then
    log_message "Removing existing accounts file..."
    rm -rf "$file_path"
fi

# Step 2: Define accounts for the smart contract
accounts_json=$(cat <<EOF
[
    {
        "name": "account1",
        "address": "$ACCOUNT1_ADDRESS",
        "private_key": "$ACCOUNT1_PRIVATE_KEY"
    },
    {
        "name": "account2",
        "address": "$ACCOUNT2_ADDRESS",
        "private_key": "$ACCOUNT2_PRIVATE_KEY"
    }
]
EOF
)

# Step 3: Run contract tests
echo -e "\nTesting the contract..."
testing_result=$(snforge 2>&1)
if echo "$testing_result" | grep -q "Failure"; then
    echo -e "Tests failed!\n"
    snforge
    echo -e "\nEnsure that your tests are passing before proceeding.\n"
    FAILED_TESTS=true
fi

if [ "$FAILED_TESTS" != "true" ]; then
    echo "Tests passed successfully."

    # Step 4: Create new account(s)
    echo -e "\nCreating account(s)..."
    for row in $(echo "${accounts_json}" | jq -c '.[]'); do
        name=$(echo "${row}" | jq -r '.name')
        address=$(echo "${row}" | jq -r '.address')
        private_key=$(echo "${row}" | jq -r '.private_key')

        account_creation_result=$(sncast --url http://localhost:5050/rpc account add --name "$name" --address "$address" --private-key "$private_key" --add-profile 2>&1)
        if echo "$account_creation_result" | grep -q "error:"; then
            echo "Account $name already exists."
        else
            echo "Account $name created successfully."
        fi
    done

    # Step 5: Build, declare, and deploy the contract
    echo -e "\nBuilding the contract..."
    scarb build

    echo -e "\nDeclaring the contract..."
    declaration_output=$(sncast --profile "$PROFILE_NAME" --wait declare --contract-name "$CONTRACT_NAME" 2>&1)

    if echo "$declaration_output" | grep -q "error: Class with hash"; then
        echo "Class hash already declared."
        CLASS_HASH=$(echo "$declaration_output" | sed -n 's/.*Class with hash \([^ ]*\).*/\1/p')
    else
        echo "New class hash declaration."
        CLASS_HASH=$(echo "$declaration_output" | grep -o 'class_hash: 0x[^ ]*' | sed 's/class_hash: //')
    fi

    echo "Class Hash: $CLASS_HASH"

    echo -e "\nDeploying the contract..."
    deployment_result=$(sncast --profile "$PROFILE_NAME" deploy --class-hash "$CLASS_HASH")
    CONTRACT_ADDRESS=$(echo "$deployment_result" | grep -o "contract_address: 0x[^ ]*" | awk '{print $2}')
    echo "Contract address: $CONTRACT_ADDRESS"

    # Step 6: Create and execute multicalls
    echo -e "\nSetting up multicall..."
    cat >"$MULTICALL_FILE" <<-EOM
[[call]]
call_type = 'invoke'
contract_address = '$CONTRACT_ADDRESS'
function = 'increase_balance'
inputs = ['0x1']

[[call]]
call_type = 'invoke'
contract_address = '$CONTRACT_ADDRESS'
function = 'increase_balance'
inputs = ['0x2']
EOM

    echo "Executing multicall..."
    sncast --profile "$PROFILE_NAME" multicall run --path "$MULTICALL_FILE"

    # Step 7: Query the contract state
    echo -e "\nChecking balance..."
    sncast --profile "$PROFILE_NAME" call --contract-address "$CONTRACT_ADDRESS" --function get_balance

    # Step 8: Clean up temporary files
    echo -e "\nCleaning up..."
    [ -e "$MULTICALL_FILE" ] && rm "$MULTICALL_FILE"

    echo -e "\nScript completed successfully.\n"
fi
```



### 3.مسیر Bash را تنظیم کنید

خط #!/usr/bin/env bash مسیر مترجم bash را نشان می دهد. اگر به نسخه یا مکان دیگری از bash نیاز دارید، مسیر آن را با استفاده از:

```sh
which bash
```

سپس #!/usr/bin/env bash را در اسکریپت با مسیر به دست آمده، مانند #!/path/to/your/bash جایگزین کنید.

### اجرا

هنگام اجرای اسکریپت، باید متغیرهای محیطی ACCOUNT1\_PRIVATE\_KEY و ACCOUNT2\_PRIVATE\_KEY را ارائه دهید.

مثال:

```sh
ACCOUNT1_PRIVATE_KEY="0x259f4329e6f4590b" ACCOUNT2_PRIVATE_KEY="0xb4862b21fb97d" ./script.sh
```

### ملاحظات

* دستور set-e در اسکریپت تضمین می کند که در صورت شکست هر فرمانی از آن خارج می شود و قابلیت اطمینان فرآیند استقرار و آزمایش را افزایش می دهد.
* همیشه کلیدهای خصوصی و اطلاعات حساس را ایمن کنید. آنها را از سیاههها و خروجی های قابل مشاهده دور نگه دارید.
* برای انعطاف‌پذیری بیشتر، مقادیر کدگذاری‌شده مانند حساب‌ها یا نام‌های قرارداد را به یک فایل پیکربندی منتقل کنید. این رویکرد به روز رسانی و مدیریت کلی را ساده می کند.
