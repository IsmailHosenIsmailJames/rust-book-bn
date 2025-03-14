## পরিশিষ্ট G - কীভাবে Rust তৈরি হয় এবং "নাইটলি Rust" (How Rust is Made and "Nightly Rust")

এই পরিশিষ্টটি Rust কীভাবে তৈরি হয় এবং এটি আপনাকে একজন Rust ডেভেলপার হিসেবে কীভাবে প্রভাবিত করে সে সম্পর্কে।

### স্থবিরতা ছাড়া স্থিতিশীলতা (Stability Without Stagnation)

একটি ভাষা হিসাবে, Rust আপনার কোডের স্থায়িত্ব সম্পর্কে _অত্যন্ত_ যত্নশীল। আমরা চাই Rust এমন একটি মজবুত ভিত্তি হোক যার উপর আপনি নির্মাণ করতে পারেন, এবং যদি জিনিসগুলি ক্রমাগত পরিবর্তিত হতে থাকে, তবে সেটি অসম্ভব হবে। একই সময়ে, যদি আমরা নতুন ফিচার নিয়ে পরীক্ষা-নিরীক্ষা করতে না পারি, তাহলে আমরা গুরুত্বপূর্ণ ত্রুটিগুলি রিলিজের আগে জানতে পারব না, যখন আমরা আর কোনো পরিবর্তন করতে পারব না।

এই সমস্যার সমাধানে আমরা যাকে বলি "স্থবিরতা ছাড়া স্থিতিশীলতা", এবং আমাদের গাইডলাইন হল: আপনার কখনই স্থিতিশীল Rust-এর একটি নতুন ভার্সনে আপগ্রেড করতে ভয় পাওয়া উচিত নয়। প্রতিটি আপগ্রেড যন্ত্রণাহীন হওয়া উচিত, তবে আপনাকে নতুন ফিচার, কম বাগ এবং দ্রুত কম্পাইল করার সময় এনে দেবে।

### ছু-ছু! রিলিজ চ্যানেল এবং ট্রেনে চড়া (Choo, Choo! Release Channels and Riding the Trains)

Rust ডেভেলপমেন্ট একটি _ট্রেন সময়সূচীতে_ কাজ করে। অর্থাৎ, সমস্ত ডেভেলপমেন্ট Rust রিপোজিটরির `master` ব্রাঞ্চে করা হয়। রিলিজগুলি একটি সফটওয়্যার রিলিজ ট্রেন মডেল অনুসরণ করে, যা Cisco IOS এবং অন্যান্য সফটওয়্যার প্রোজেক্ট দ্বারা ব্যবহৃত হয়েছে। Rust-এর জন্য তিনটি _রিলিজ চ্যানেল_ রয়েছে:

- নাইটলি (Nightly)
- বিটা (Beta)
- স্থিতিশীল (Stable)

বেশিরভাগ Rust ডেভেলপার প্রাথমিকভাবে স্থিতিশীল চ্যানেল ব্যবহার করেন, তবে যারা পরীক্ষামূলক নতুন ফিচারগুলি ব্যবহার করতে চান তারা নাইটলি বা বিটা ব্যবহার করতে পারেন।

ডেভেলপমেন্ট এবং রিলিজ প্রক্রিয়া কীভাবে কাজ করে তার একটি উদাহরণ এখানে দেওয়া হল: ধরা যাক Rust টিম Rust 1.5-এর রিলিজে কাজ করছে। সেই রিলিজটি ২০১৫ সালের ডিসেম্বরে হয়েছিল, তবে এটি আমাদের বাস্তবসম্মত ভার্সন নম্বর সরবরাহ করবে। Rust-এ একটি নতুন ফিচার যোগ করা হয়েছে: `master` ব্রাঞ্চে একটি নতুন কমিট আসে। প্রতি রাতে, Rust-এর একটি নতুন নাইটলি ভার্সন তৈরি করা হয়। প্রতিটি দিন একটি রিলিজের দিন, এবং এই রিলিজগুলি আমাদের রিলিজ পরিকাঠামো দ্বারা স্বয়ংক্রিয়ভাবে তৈরি করা হয়। সুতরাং সময় অতিবাহিত হওয়ার সাথে সাথে, আমাদের রিলিজগুলি প্রতি রাতে এইরকম দেখায়:

```text
nightly: * - - * - - *
```

প্রতি ছয় সপ্তাহে, একটি নতুন রিলিজ প্রস্তুত করার সময়! Rust রিপোজিটরির `beta` ব্রাঞ্চটি নাইটলি দ্বারা ব্যবহৃত `master` ব্রাঞ্চ থেকে শাখা তৈরি করে। এখন, দুটি রিলিজ রয়েছে:

```text
nightly: * - - * - - *
                     |
beta:                *
```

বেশিরভাগ Rust ব্যবহারকারী সক্রিয়ভাবে বিটা রিলিজ ব্যবহার করেন না, তবে সম্ভাব্য রিগ্রেশন আবিষ্কার করতে Rust-কে সাহায্য করার জন্য তাদের CI সিস্টেমে বিটার বিরুদ্ধে পরীক্ষা করেন। ইতিমধ্যে, এখনও প্রতি রাতে একটি নাইটলি রিলিজ রয়েছে:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

ধরা যাক একটি রিগ্রেশন পাওয়া গেছে। ভালো যে রিগ্রেশনটি একটি স্থিতিশীল রিলিজে লুকিয়ে যাওয়ার আগে আমাদের বিটা রিলিজটি পরীক্ষা করার জন্য কিছু সময় ছিল! ফিক্সটি `master`-এ প্রয়োগ করা হয়েছে, যাতে নাইটলি ঠিক করা হয় এবং তারপর ফিক্সটি `beta` ব্রাঞ্চে ব্যাকপোর্ট করা হয় এবং বিটার একটি নতুন রিলিজ তৈরি করা হয়:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

প্রথম বিটা তৈরি হওয়ার ছয় সপ্তাহ পরে, একটি স্থিতিশীল রিলিজের সময়! `stable` ব্রাঞ্চটি `beta` ব্রাঞ্চ থেকে তৈরি করা হয়:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

হুররে! Rust 1.5 সম্পন্ন হয়েছে! যাইহোক, আমরা একটি জিনিস ভুলে গেছি: যেহেতু ছয় সপ্তাহ চলে গেছে, তাই আমাদের Rust-এর _পরবর্তী_ ভার্সন, 1.6-এর একটি নতুন বিটাও প্রয়োজন। তাই `stable` `beta` থেকে শাখা তৈরি করার পরে, `beta`-এর পরবর্তী ভার্সনটি আবার `nightly` থেকে শাখা তৈরি করে:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

এটিকে "ট্রেন মডেল" বলা হয় কারণ প্রতি ছয় সপ্তাহে, একটি রিলিজ "স্টেশন ছেড়ে যায়", কিন্তু একটি স্থিতিশীল রিলিজ হিসাবে পৌঁছানোর আগে এটিকে এখনও বিটা চ্যানেলের মধ্য দিয়ে একটি যাত্রা করতে হয়।

Rust প্রতি ছয় সপ্তাহে ঘড়ির কাঁটার মতো রিলিজ হয়। আপনি যদি একটি Rust রিলিজের তারিখ জানেন, তাহলে আপনি পরবর্তীটির তারিখ জানতে পারবেন: এটি ছয় সপ্তাহ পরে। প্রতি ছয় সপ্তাহে রিলিজ নির্ধারিত হওয়ার একটি সুন্দর দিক হল যে পরবর্তী ট্রেন শীঘ্রই আসছে। যদি কোনো ফিচার কোনো নির্দিষ্ট রিলিজ মিস করে, তাহলে চিন্তার কিছু নেই: অল্প সময়ের মধ্যেই আরেকটি আসছে! এটি রিলিজের সময়সীমার কাছাকাছি সম্ভবত অপালিশ করা ফিচারগুলিকে লুকিয়ে রাখার চাপ কমাতে সাহায্য করে।

এই প্রক্রিয়ার জন্য ধন্যবাদ, আপনি সর্বদা Rust-এর পরবর্তী বিল্ডটি পরীক্ষা করে দেখতে পারেন এবং নিজে যাচাই করতে পারেন যে আপগ্রেড করা সহজ কিনা: যদি একটি বিটা রিলিজ প্রত্যাশা অনুযায়ী কাজ না করে, তাহলে আপনি টিমের কাছে রিপোর্ট করতে পারেন এবং পরবর্তী স্থিতিশীল রিলিজ হওয়ার আগে এটি ঠিক করাতে পারেন! একটি বিটা রিলিজে ভাঙ্গন তুলনামূলকভাবে বিরল, কিন্তু `rustc` এখনও একটি সফটওয়্যারের অংশ, এবং বাগ বিদ্যমান থাকে।

### রক্ষণাবেক্ষণের সময়

Rust প্রোজেক্ট সবচেয়ে সাম্প্রতিক স্থিতিশীল ভার্সনটিকে সমর্থন করে। যখন একটি নতুন স্থিতিশীল ভার্সন প্রকাশিত হয়, তখন পুরানো ভার্সনটি তার জীবনকালের শেষে (EOL) পৌঁছে যায়। এর মানে প্রতিটি ভার্সন ছয় সপ্তাহের জন্য সমর্থিত।

### অস্থির ফিচার (Unstable Features)

এই রিলিজ মডেলের সাথে আরও একটি বিষয় রয়েছে: অস্থির ফিচার। Rust একটি "ফিচার ফ্ল্যাগ" নামক কৌশল ব্যবহার করে, যা নির্ধারণ করে যে একটি প্রদত্ত রিলিজে কোন ফিচারগুলি সক্রিয় করা হয়েছে। যদি একটি নতুন ফিচার সক্রিয় ডেভেলপমেন্টের অধীনে থাকে, তবে সেটি `master`-এ আসে এবং সেইজন্য, নাইটলিতে, কিন্তু একটি _ফিচার ফ্ল্যাগের_ পিছনে। আপনি যদি একজন ব্যবহারকারী হিসাবে, কাজের অগ্রগতিতে থাকা ফিচারটি ব্যবহার করতে চান, তাহলে আপনি তা করতে পারেন, কিন্তু আপনাকে অবশ্যই Rust-এর একটি নাইটলি রিলিজ ব্যবহার করতে হবে এবং অপ্ট ইন করার জন্য উপযুক্ত ফ্ল্যাগ দিয়ে আপনার সোর্স কোড অ্যানোটেট করতে হবে।

আপনি যদি Rust-এর বিটা বা স্থিতিশীল রিলিজ ব্যবহার করেন, তাহলে আপনি কোনো ফিচার ফ্ল্যাগ ব্যবহার করতে পারবেন না। এটি হল মূল বিষয় যা আমাদের নতুন ফিচারগুলিকে চিরতরে স্থিতিশীল ঘোষণা করার আগে ব্যবহারিক ব্যবহারের সুযোগ দেয়। যারা ব্লিডিং এজ-এ থাকতে চান তারা তা করতে পারেন, এবং যারা একটি রক-সলিড অভিজ্ঞতা চান তারা স্থিতিশীল থাকতে পারেন এবং জানতে পারেন যে তাদের কোড ভাঙবে না। স্থবিরতা ছাড়া স্থিতিশীলতা।

এই বইটিতে শুধুমাত্র স্থিতিশীল ফিচার সম্পর্কে তথ্য রয়েছে, কারণ অগ্রগতির ফিচারগুলি এখনও পরিবর্তনশীল, এবং নিশ্চিতভাবেই এই বইটি লেখার সময় এবং স্থিতিশীল বিল্ডগুলিতে সক্রিয় হওয়ার মধ্যে সেগুলি আলাদা হবে। আপনি অনলাইন-এ নাইটলি-অনলি ফিচারগুলির জন্য ডকুমেন্টেশন খুঁজে পেতে পারেন।

### Rustup এবং Rust Nightly-র ভূমিকা

Rustup গ্লোবাল বা প্রোজেক্ট-ভিত্তিক ভিত্তিতে Rust-এর বিভিন্ন রিলিজ চ্যানেলের মধ্যে পরিবর্তন করা সহজ করে তোলে। ডিফল্টরূপে, আপনার স্থিতিশীল Rust ইনস্টল করা থাকবে। উদাহরণস্বরূপ, নাইটলি ইনস্টল করতে:

```console
$ rustup toolchain install nightly
```

আপনি `rustup` দিয়ে আপনার ইনস্টল করা সমস্ত _টুলচেইন_ (Rust-এর রিলিজ এবং সংশ্লিষ্ট কম্পোনেন্ট) দেখতে পারেন। আপনার একজন লেখকের উইন্ডোজ কম্পিউটারে এখানে একটি উদাহরণ রয়েছে:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

আপনি দেখতে পাচ্ছেন, স্থিতিশীল টুলচেইন হল ডিফল্ট। বেশিরভাগ Rust ব্যবহারকারী বেশিরভাগ সময় স্থিতিশীল ব্যবহার করেন। আপনি হয়তো বেশিরভাগ সময় স্থিতিশীল ব্যবহার করতে চাইতে পারেন, কিন্তু একটি নির্দিষ্ট প্রোজেক্টে নাইটলি ব্যবহার করতে পারেন, কারণ আপনি একটি কাটিং-এজ ফিচার সম্পর্কে আগ্রহী। এটি করার জন্য, আপনি সেই প্রোজেক্টের ডিরেক্টরিতে `rustup override` ব্যবহার করে নাইটলি টুলচেইন সেট করতে পারেন, যেটি `rustup` ব্যবহার করবে যখন আপনি সেই ডিরেক্টরিতে থাকবেন:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

এখন, প্রতিবার আপনি _~/projects/needs-nightly_-এর ভিতরে `rustc` বা `cargo` কল করবেন, `rustup` নিশ্চিত করবে যে আপনি আপনার স্থিতিশীল Rust-এর ডিফল্টের পরিবর্তে নাইটলি Rust ব্যবহার করছেন। আপনার যখন অনেক Rust প্রোজেক্ট থাকে তখন এটি কাজে আসে!

### RFC প্রক্রিয়া এবং টিম (The RFC Process and Teams)

তাহলে আপনি এই নতুন ফিচারগুলি সম্পর্কে কীভাবে জানবেন? Rust-এর ডেভেলপমেন্ট মডেল একটি _রিকোয়েস্ট ফর কমেন্টস (RFC) প্রক্রিয়া_ অনুসরণ করে। আপনি যদি Rust-এ কোনো উন্নতি চান, তাহলে আপনি একটি প্রস্তাব লিখতে পারেন, যাকে RFC বলা হয়।

যেকোনো ব্যক্তি Rust উন্নত করার জন্য RFC লিখতে পারেন, এবং প্রস্তাবগুলি Rust টিম দ্বারা পর্যালোচনা ও আলোচনা করা হয়, যা অনেক বিষয়ভিত্তিক সাবটিম নিয়ে গঠিত। [Rust-এর ওয়েবসাইটে](https://www.rust-lang.org/governance) টিমগুলির একটি সম্পূর্ণ তালিকা রয়েছে, যার মধ্যে প্রোজেক্টের প্রতিটি ক্ষেত্রের জন্য টিম রয়েছে: ভাষা ডিজাইন, কম্পাইলার ইমপ্লিমেন্টেশন, পরিকাঠামো, ডকুমেন্টেশন এবং আরও অনেক কিছু। উপযুক্ত টিম প্রস্তাব এবং কমেন্টগুলি পড়ে, তাদের নিজস্ব কিছু কমেন্ট লেখে এবং অবশেষে, ফিচারটি গ্রহণ বা প্রত্যাখ্যান করার জন্য একটি ঐক্যমত হয়।

যদি ফিচারটি গৃহীত হয়, তাহলে Rust রিপোজিটরিতে একটি ইস্যু খোলা হয় এবং কেউ এটি ইমপ্লিমেন্ট করতে পারে। যিনি এটি ইমপ্লিমেন্ট করেন তিনি খুব সম্ভবত সেই ব্যক্তি নাও হতে পারেন যিনি প্রথমে ফিচারটির প্রস্তাব করেছিলেন! যখন ইমপ্লিমেন্টেশন প্রস্তুত হয়, তখন এটি একটি ফিচার গেটের পিছনে `master` ব্রাঞ্চে আসে, যেমনটি আমরা [“অস্থির ফিচার”](#unstable-features)<!-- ignore --> বিভাগে আলোচনা করেছি।

কিছু সময় পরে, যখন নাইটলি রিলিজ ব্যবহারকারী Rust ডেভেলপাররা নতুন ফিচারটি ব্যবহার করার সুযোগ পান, তখন টিমের সদস্যরা ফিচারটি নিয়ে আলোচনা করবেন, এটি নাইটলিতে কীভাবে কাজ করেছে এবং এটি স্থিতিশীল Rust-এ যাওয়া উচিত কিনা তা নিয়ে সিদ্ধান্ত নেবেন। যদি সিদ্ধান্ত হয় এগিয়ে যাওয়ার, তাহলে ফিচার গেটটি সরিয়ে ফেলা হয় এবং ফিচারটি এখন স্থিতিশীল বলে বিবেচিত হয়! এটি ট্রেনে চড়ে Rust-এর একটি নতুন স্থিতিশীল রিলিজে যায়।
