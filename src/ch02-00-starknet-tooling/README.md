# ابزار Starknet

برای استفاده بیشتر از این فصل، درک اولیه زبان برنامه نویسی قاهره توصیه می شود. پیشنهاد می‌کنیم فصل‌های 1-6 کتاب قاهره را بخوانید که موضوعاتی از شروع تا Enums و تطبیق الگو را پوشش می‌دهد. این را با مطالعه فصل قراردادهای هوشمند Starknet در همان کتاب دنبال کنید. با این پیشینه، برای درک مثال های ارائه شده در اینجا به خوبی مجهز خواهید بود.

امروزه، Starknet همه ابزارهای ضروری را برای ساخت برنامه های غیرمتمرکز (dApps)، سازگار با چندین زبان مانند جاوا اسکریپت، Rust و Python فراهم می کند. می توانید از Starknet SDK برای توسعه استفاده کنید. توسعه‌دهندگان فرانت‌اند می‌توانند از Starknet.js با React استفاده کنند، در حالی که Rust و Python برای کارهای بک‌اند به خوبی کار می‌کنند.

ما از مشارکت کنندگان برای بهبود ابزارهای موجود یا توسعه راه حل های جدید استقبال می کنیم.

در این فصل، شما بررسی خواهید کرد:

* چارچوب ها: با استفاده از Starknet-Foundry بسازید
* SDK: پشتیبانی چند زبانه را از طریق Starknet.js، Starknet-rs، Starknet\_py و Caigo کشف کنید.
* توسعه Front-end: از Starknet.js و React استفاده کنید
* تست: روش های تست را با Starknet-Foundry و Devnet درک کنید

تا پایان فصل، درک کاملی از مجموعه ابزار Starknet خواهید داشت که امکان توسعه کارآمد dApp را فراهم می کند.

در اینجا خلاصه‌ای از ابزارهایی است که می‌توان برای توسعه Starknet استفاده کرد و در این فصل به آن‌ها خواهیم پرداخت:

1. Scarb: یک مدیر بسته که قراردادهای شما را جمع آوری می کند.
2. Starkli: یک ابزار CLI برای تعامل با شبکه Starknet.
3. Starknet Foundry: برای آزمایش قرارداد.
4. Katana: یک گره تست محلی ایجاد می کند.
5. SDK ها: starknet.js، Starknet.py و starknet.rs با استفاده از زبان های برنامه نویسی رایج با Starknet ارتباط برقرار می کنند.
6. Starknet-react: با استفاده از React، اپلیکیشن های فرانت اند را می سازد.
