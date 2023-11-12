# حساب های چند امضایی

توجه: این فصل برای انعکاس نحو جدید برای قراردادهای حساب باید به روز شود. لطفاً تا زمانی که این یادداشت حذف نشده است، از این فصل به عنوان مرجع استفاده نکنید.

مشارکت: این فصل فرعی نمونه ای از اعلام، استقرار و تعامل با قرارداد را ندارد. ما دوست داریم مشارکت شما را ببینیم! لطفا یک PR ارسال کنید.

فناوری Multisignature (multisig) بخشی جدایی ناپذیر از چشم انداز بلاک چین مدرن است. این امر امنیت را با نیاز به چندین امضا برای تأیید یک تراکنش افزایش می‌دهد و از این رو خطر معاملات متقلبانه را کاهش می‌دهد و کنترل بر مدیریت دارایی را افزایش می‌دهد.

در استارک‌نت، مفهوم حساب‌های چند علامتی در سطح پروتکل انتزاع می‌شود و به توسعه‌دهندگان اجازه می‌دهد تا قراردادهای حساب سفارشی که این مفهوم را تجسم می‌دهند، پیاده‌سازی کنند. در این فصل، به کارکرد یک حساب multisig می پردازیم و می بینیم که چگونه در Starknet با استفاده از قرارداد حساب ایجاد می شود.

### حساب Multisig چیست؟

حساب مولتی سایگ حسابی است که برای تأیید تراکنش ها به بیش از یک امضا نیاز دارد. این امر به طور قابل توجهی امنیت را افزایش می دهد و به رضایت چندین نهاد برای انجام معاملات وجوه یا انجام اقدامات مهم نیاز دارد.

مشخصات کلیدی یک حساب مولتی سیگ عبارتند از:

* کلیدهای عمومی که حساب را تشکیل می دهند
* تعداد آستانه امضا لازم است

تراکنش امضا شده توسط یک حساب multisig باید به صورت جداگانه توسط کلیدهای مختلف مشخص شده برای حساب امضا شود. اگر تعداد امضاهای مورد نیاز کمتر از حد آستانه وجود داشته باشد، چند امضای حاصل نامعتبر تلقی می شود.

در استارک نت، حساب ها انتزاعی هایی هستند که در سطح پروتکل ارائه می شوند. بنابراین، برای ایجاد یک حساب multisig، باید منطق را در یک قرارداد حساب رمزگذاری کرده و آن را مستقر کنید.

قرارداد زیر به عنوان نمونه ای از قرارداد حساب مولتی سیگ عمل می کند. هنگامی که مستقر می شود، می تواند یک حساب multisig بومی با استفاده از مفهوم انتزاع حساب ایجاد کند. لطفاً توجه داشته باشید که این یک مثال ساده شده است و فاقد بررسی‌ها و اعتبارسنجی‌های جامعی است که در یک قرارداد چندثانی درجه تولید وجود دارد.

### قرارداد حساب Multisig

این کد Rust برای یک قرارداد حساب multisig است:

```rust
    #[account_contract]
    mod MultisigAccount {
        use ecdsa::check_ecdsa_signature;
        use starknet::ContractAddress;
        use zeroable::Zeroable;
        use array::ArrayTrait;
        use starknet::get_caller_address;
        use box::BoxTrait;
        use array::SpanTrait;

        struct Storage {
            index_to_owner: LegacyMap::<u32, felt252>,
            owner_to_index: LegacyMap::<felt252, u32>,
            num_owners: usize,
            threshold: usize,
            curr_tx_index: felt252,
            //Mapping between tx_index and num of confirmations
            tx_confirms: LegacyMap<felt252, usize>,
            //Mapping between tx_index and its execution state
            tx_is_executed: LegacyMap<felt252, bool>,
            //Mapping between a transaction index and its hash
            transactions: LegacyMap<felt252, felt252>,
            has_confirmed: LegacyMap::<(ContractAddress, felt252), bool>,
        }

        #[constructor]
        fn constructor(public_keys: Array::<felt252>, _threshold: usize) {
            assert(public_keys.len() <= 3_usize, 'public_keys.len <= 3');
            num_owners::write(public_keys.len());
            threshold::write(_threshold);
            _set_owners(public_keys.len(), public_keys);
        }

        //GETTERS
        //Get number of confirmations for a given transaction index
        #[view]
        fn get_confirmations(tx_index : felt252) -> usize {
            tx_confirms::read(tx_index)
        }

        //Get the number of owners of this account
        #[view]
        fn get_num_owners() -> usize {
            num_owners::read()
        }


        //Get the public key of the owners
        //TODO - Recursively add the owners into an array and return, maybe wait for loops to be enabled


        //EXTERNAL FUNCTIONS

        #[external]
        fn submit_tx(public_key: felt252) {

            //Need to check if caller is one of the owners.
            let tx_info = starknet::get_tx_info().unbox();
            let signature: Span<felt252> = tx_info.signature;
            let caller = get_caller_address();
            assert(signature.len() == 2_u32, 'INVALID_SIGNATURE_LENGTH');

            //Updating the transaction index
            let tx_index = curr_tx_index::read();

            //`true` if a signature is valid and `false` otherwise.
            assert(
                check_ecdsa_signature(
                    message_hash: tx_info.transaction_hash,
                    public_key: public_key,
                    signature_r: *signature.at(0_u32),
                    signature_s: *signature.at(1_u32),
                ),
                'INVALID_SIGNATURE',
            );

            transactions::write(tx_index, tx_info.transaction_hash);
            curr_tx_index::write(tx_index + 1);

        }

        #[external]
        fn confirm_tx(tx_index: felt252, public_key: felt252) {

            let transaction_hash = transactions::read(tx_index);
            //TBD: Assert that tx_hash is not null

            let num_confirmations = tx_confirms::read(tx_index);
            let executed = tx_is_executed::read(tx_index);

            assert(executed == false, 'TX_ALREADY_EXECUTED');

            let caller = get_caller_address();
            let tx_info = starknet::get_tx_info().unbox();
            let signature: Span<felt252> = tx_info.signature;

             assert(
                check_ecdsa_signature(
                    message_hash: tx_info.transaction_hash,
                    public_key: public_key,
                    signature_r: *signature.at(0_u32),
                    signature_s: *signature.at(1_u32),
                ),
                'INVALID_SIGNATURE',
            );

            let confirmed = has_confirmed::read((caller, tx_index));

            assert (confirmed == false, 'CALLER_ALREADY_CONFIRMED');
            tx_confirms::write(tx_index, num_confirmations+1_usize);
            has_confirmed::write((caller, tx_index), true);


        }

        //An example function to validate that there are at least two signatures
        fn validate_transaction(public_key: felt252) -> felt252 {
            let tx_info = starknet::get_tx_info().unbox();
            let signature: Span<felt252> = tx_info.signature;
            let caller = get_caller_address();
            assert(signature.len() == 2_u32, 'INVALID_SIGNATURE_LENGTH');

            //`true` if a signature is valid and `false` otherwise.
            assert(
                check_ecdsa_signature(
                    message_hash: tx_info.transaction_hash,
                    public_key: public_key,
                    signature_r: *signature.at(0_u32),
                    signature_s: *signature.at(1_u32),
                ),
                'INVALID_SIGNATURE',
            );

            starknet::VALIDATED
        }

        //INTERNAL FUNCTION
        //Function to add the public keys of the multisig in permanent storage
        fn _set_owners(owners_len: usize, public_keys: Array::<felt252>) {
            if owners_len == 0_usize {
            }

            index_to_owner::write(owners_len, *public_keys.at(owners_len - 1_usize));
            owner_to_index::write(*public_keys.at(owners_len - 1_usize), owners_len);
            _set_owners(owners_len - 1_u32, public_keys);
        }


        #[external]
        fn __validate_deploy__(
            class_hash: felt252, contract_address_salt: felt252, public_key_: felt252
        ) -> felt252 {
            validate_transaction(public_key_)
        }

        #[external]
        fn __validate_declare__(class_hash: felt252, public_key_: felt252) -> felt252 {
            validate_transaction(public_key_)
        }

        #[external]
        fn __validate__(
            contract_address: ContractAddress, entry_point_selector: felt252, calldata: Array::<felt252>, public_key_: felt252
        ) -> felt252 {
            validate_transaction(public_key_)
        }

        #[external]
        #[raw_output]
        fn __execute__(
            contract_address: ContractAddress, entry_point_selector: felt252, calldata: Array::<felt252>,
            tx_index: felt252
        ) -> Span::<felt252> {
            // Validate caller.
            assert(starknet::get_caller_address().is_zero(), 'INVALID_CALLER');

            // Check the tx version here, since version 0 transaction skip the __validate__ function.
            let tx_info = starknet::get_tx_info().unbox();
            assert(tx_info.version != 0, 'INVALID_TX_VERSION');

            //Multisig check here
            let num_confirmations = tx_confirms::read(tx_index);
            let owners_len = num_owners::read();
            //Subtracting one for the submitter
            let required_confirmations = threshold::read() - 1_usize;
            assert(num_confirmations >= required_confirmations, 'MINIMUM_50%_CONFIRMATIONS');

            tx_is_executed::write(tx_index, true);

            starknet::call_contract_syscall(
                contract_address, entry_point_selector, calldata.span()
            ).unwrap_syscall()
        }
    }
```

### جریان تراکنش چند رقمی

جریان یک تراکنش چند رقمی شامل مراحل زیر است:

1. ارسال تراکنش: هر یک از مالکان می توانند تراکنش را از حساب ارسال کنند.
2. تایید تراکنش: مالکی که تراکنش ارسال نکرده است می تواند تراکنش را تایید کند.

اگر تعداد تاییدات (از جمله امضای ارسال کننده) بیشتر یا مساوی با تعداد آستانه امضاها باشد، تراکنش با موفقیت انجام می شود، در غیر این صورت شکست می خورد. این مکانیسم تأیید تضمین می‌کند که هیچ یک از طرفین نمی‌توانند به‌طور یک‌جانبه اقدامات حیاتی را انجام دهند، در نتیجه امنیت حساب را افزایش می‌دهند.

### کاوش توابع Multisig

بیایید نگاهی دقیق تر به توابع مختلف مرتبط با عملکرد چند علامتی در قرارداد ارائه شده بیندازیم.

### تابع \_set\_owners

این یک عملکرد داخلی است که برای افزودن کلیدهای عمومی صاحبان حساب به یک حافظه دائمی طراحی شده است. در حالت ایده‌آل، ساختار حساب multisig باید طبق توافق صاحبان حساب اجازه اضافه و حذف مالکان را بدهد. با این حال، هر تغییر باید معامله ای باشد که به تعداد آستانه امضا نیاز دارد.

```rust
    //INTERNAL FUNCTION
    //Function to add the public keys of the multisig in permanent storage
    fn _set_owners(owners_len: usize, public_keys: Array::<felt252>) {
        if owners_len == 0_usize {
        }

        index_to_owner::write(owners_len, *public_keys.at(owners_len - 1_usize));
        owner_to_index::write(*public_keys.at(owners_len - 1_usize), owners_len);
        _set_owners(owners_len - 1_u32, public_keys);
    }
```

### تابع submit\_tx

این عملکرد خارجی به صاحبان حساب اجازه می دهد تا تراکنش ها را ارسال کنند. پس از ارسال، این تابع اعتبار تراکنش را بررسی می‌کند، مطمئن می‌شود که تماس‌گیرنده یکی از صاحبان حساب است و تراکنش را به نقشه تراکنش‌ها اضافه می‌کند. همچنین شاخص تراکنش فعلی را افزایش می دهد.

```rust
    #[external]
    fn submit_tx(public_key: felt252) {

        //Need to check if caller is one of the owners.
        let tx_info = starknet::get_tx_info().unbox();
        let signature: Span<felt252> = tx_info.signature;
        let caller = get_caller_address();
        assert(signature.len() == 2_u32, 'INVALID_SIGNATURE_LENGTH');

        //Updating the transaction index
        let tx_index = curr_tx_index::read();

        //`true` if a signature is valid and `false` otherwise.
        assert(
            check_ecdsa_signature(
                message_hash: tx_info.transaction_hash,
                public_key: public_key,
                signature_r: *signature.at(0_u32),
                signature_s: *signature.at(1_u32),
            ),
            'INVALID_SIGNATURE',
        );

        transactions::write(tx_index, tx_info.transaction_hash);
        curr_tx_index::write(tx_index + 1);

    }
```

### تابع confirm\_tx

به طور مشابه، تابع confirm\_tx راهی برای ثبت تاییدیه برای هر تراکنش فراهم می کند. صاحب حسابی که تراکنش را ارسال نکرده است، می‌تواند آن را تأیید کند و تعداد تأیید آن را افزایش دهد.

```rust
        #[external]
        fn confirm_tx(tx_index: felt252, public_key: felt252) {

            let transaction_hash = transactions::read(tx_index);
            //TBD: Assert that tx_hash is not null

            let num_confirmations = tx_confirms::read(tx_index);
            let executed = tx_is_executed::read(tx_index);

            assert(executed == false, 'TX_ALREADY_EXECUTED');

            let caller = get_caller_address();
            let tx_info = starknet::get_tx_info().unbox();
            let signature: Span<felt252> = tx_info.signature;

             assert(
                check_ecdsa_signature(
                    message_hash: tx_info.transaction_hash,
                    public_key: public_key,
                    signature_r: *signature.at(0_u32),
                    signature_s: *signature.at(1_u32),
                ),
                'INVALID_SIGNATURE',
            );

            let confirmed = has_confirmed::read((caller, tx_index));

            assert (confirmed == false, 'CALLER_ALREADY_CONFIRMED');
            tx_confirms::write(tx_index, num_confirmations+1_usize);
            has_confirmed::write((caller, tx_index), true);
        }
```

### اجرای تابع

تابع execute به عنوان آخرین مرحله در فرآیند تراکنش عمل می کند. اعتبار تراکنش را بررسی می کند، اینکه آیا قبلاً انجام شده است یا خیر، و اینکه آیا تعداد آستانه امضاها رسیده است یا خیر. معامله در صورتی انجام می شود که تمام چک ها پاس شوند.

```rust
    #[external]
        #[raw_output]
        fn __execute__(
            contract_address: ContractAddress, entry_point_selector: felt252, calldata: Array::<felt252>,
            tx_index: felt252
        ) -> Span::<felt252> {
            // Validate caller.
            assert(starknet::get_caller_address().is_zero(), 'INVALID_CALLER');

            // Check the tx version here, since version 0 transaction skip the __validate__ function.
            let tx_info = starknet::get_tx_info().unbox();
            assert(tx_info.version != 0, 'INVALID_TX_VERSION');

            //Multisig check here
            let num_confirmations = tx_confirms::read(tx_index);
            let owners_len = num_owners::read();
            //Subtracting one for the submitter
            let required_confirmations = threshold::read() - 1_usize;
            assert(num_confirmations >= required_confirmations, 'MINIMUM_50%_CONFIRMATIONS');

            tx_is_executed::write(tx_index, true);

            starknet::call_contract_syscall(
                contract_address, entry_point_selector, calldata.span()
            ).unwrap_syscall()
        }
```

### افکار بسته

این فصل شما را با مفهوم حساب‌های چند علامتی در Starknet آشنا کرده و نحوه پیاده‌سازی آن‌ها را با استفاده از قرارداد حساب نشان می‌دهد. با این حال، مهم است که توجه داشته باشید که این یک مثال ساده شده است، و یک قرارداد چند علامتی درجه تولید باید شامل بررسی‌ها و تاییدیه‌های اضافی برای استحکام و امنیت باشد.

کتاب یک تلاش جامعه محور است که برای جامعه ایجاد شده است.

* اگر چیزی یاد گرفته‌اید یا نه، لطفاً چند لحظه وقت بگذارید و از طریق این نظرسنجی 3 سؤالی بازخورد خود را ارائه دهید.
* در صورت کشف هرگونه خطا یا پیشنهادات اضافی، در باز کردن مشکل در مخزن GitHub ما تردید نکنید.
