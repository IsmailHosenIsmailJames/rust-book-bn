# আনসেফ রাস্ট (Unsafe Rust)

এখন পর্যন্ত আমরা যে সমস্ত কোড নিয়ে আলোচনা করেছি, কম্পাইল করার সময় সেগুলিতে Rust-এর মেমরি সুরক্ষার গ্যারান্টি প্রয়োগ করা হয়েছে। যাইহোক, Rust-এর ভিতরে একটি লুকানো দ্বিতীয় ভাষা রয়েছে যা এই মেমরি সুরক্ষার গ্যারান্টিগুলি প্রয়োগ করে না: এটিকে _আনসেফ রাস্ট_ বলা হয় এবং এটি সাধারণ Rust-এর মতোই কাজ করে, কিন্তু আমাদের অতিরিক্ত সুপারপাওয়ার দেয়।

আনসেফ রাস্ট বিদ্যমান কারণ, স্বভাবতই, স্ট্যাটিক বিশ্লেষণ রক্ষণশীল। কম্পাইলার যখন নির্ধারণ করার চেষ্টা করে যে কোড গ্যারান্টিগুলি সমর্থন করে কিনা, তখন কিছু অবৈধ প্রোগ্রাম গ্রহণ করার চেয়ে কিছু বৈধ প্রোগ্রাম প্রত্যাখ্যান করা ভাল। যদিও কোডটি ঠিক _হতে পারে_, যদি Rust কম্পাইলারের কাছে আত্মবিশ্বাসী হওয়ার জন্য পর্যাপ্ত তথ্য না থাকে, তাহলে এটি কোডটি প্রত্যাখ্যান করবে। এই ক্ষেত্রগুলিতে, আপনি কম্পাইলারকে বলতে আনসেফ কোড ব্যবহার করতে পারেন, “আমাকে বিশ্বাস করুন, আমি জানি আমি কী করছি।” তবে সতর্ক থাকুন, আপনি নিজের ঝুঁকিতে আনসেফ রাস্ট ব্যবহার করেন: যদি আপনি আনসেফ কোডটি ভুলভাবে ব্যবহার করেন, তাহলে মেমরি অসুরক্ষার কারণে সমস্যা দেখা দিতে পারে, যেমন নাল পয়েন্টার ডিরেফারেন্সিং।

Rust-এর একটি আনসেফ অল্টার ইগো থাকার আরেকটি কারণ হল অন্তর্নিহিত কম্পিউটার হার্ডওয়্যার স্বভাবতই আনসেফ। যদি Rust আপনাকে আনসেফ অপারেশন করতে না দেয়, তাহলে আপনি কিছু কাজ করতে পারবেন না। Rust-এর আপনাকে নিম্ন-স্তরের সিস্টেম প্রোগ্রামিং করার অনুমতি দিতে হবে, যেমন সরাসরি অপারেটিং সিস্টেমের সাথে ইন্টারঅ্যাক্ট করা বা এমনকি আপনার নিজের অপারেটিং সিস্টেম লেখা। নিম্ন-স্তরের সিস্টেম প্রোগ্রামিং নিয়ে কাজ করা এই ভাষার অন্যতম লক্ষ্য। আসুন আমরা আনসেফ রাস্ট দিয়ে কী করতে পারি এবং কীভাবে এটি করতে পারি তা অন্বেষণ করি।

### আনসেফ সুপারপাওয়ার (Unsafe Superpowers)

আনসেফ রাস্টে স্যুইচ করতে, `unsafe` কীওয়ার্ড ব্যবহার করুন এবং তারপর একটি নতুন ব্লক শুরু করুন যা আনসেফ কোড ধারণ করে। আপনি আনসেফ রাস্টে পাঁচটি কাজ করতে পারেন যা আপনি নিরাপদ রাস্টে করতে পারবেন না, যেগুলিকে আমরা _আনসেফ সুপারপাওয়ার_ বলি। সেই সুপারপাওয়ারগুলির মধ্যে এই ক্ষমতাগুলি অন্তর্ভুক্ত রয়েছে:

- একটি র' পয়েন্টার ডিরেফারেন্স করা
- একটি আনসেফ ফাংশন বা মেথড কল করা
- একটি মিউটেবল স্ট্যাটিক ভেরিয়েবল অ্যাক্সেস বা পরিবর্তন করা
- একটি আনসেফ ট্রেইট ইমপ্লিমেন্ট করা।
- একটি `union`-এর ফিল্ড অ্যাক্সেস করা।

এটি বোঝা গুরুত্বপূর্ণ যে `unsafe` বড় হাতের অক্ষর নয় বা Rust-এর অন্য কোনো নিরাপত্তা পরীক্ষাকে অক্ষম করে না: আপনি যদি আনসেফ কোডে একটি রেফারেন্স ব্যবহার করেন, তাহলেও সেটি পরীক্ষা করা হবে। `unsafe` কীওয়ার্ডটি শুধুমাত্র আপনাকে এই পাঁচটি বৈশিষ্ট্যে অ্যাক্সেস দেয় যা তখন কম্পাইলার মেমরি সুরক্ষার জন্য পরীক্ষা করে না। আপনি এখনও একটি আনসেফ ব্লকের ভিতরে কিছুটা নিরাপত্তা পাবেন।

এছাড়াও, `unsafe`-এর মানে এই নয় যে ব্লকের ভিতরের কোডটি অবশ্যই বিপজ্জনক বা এটিতে অবশ্যই মেমরি নিরাপত্তার সমস্যা থাকবে: উদ্দেশ্য হল যে প্রোগ্রামার হিসাবে, আপনি নিশ্চিত করবেন যে একটি `unsafe` ব্লকের ভিতরের কোড মেমরিকে বৈধ উপায়ে অ্যাক্সেস করবে।

মানুষ ত্রুটিপ্রবণ, এবং ভুল ঘটবে, কিন্তু এই পাঁচটি আনসেফ অপারেশনকে `unsafe` দিয়ে চিহ্নিত ব্লকের ভিতরে রাখার প্রয়োজন করে আপনি জানতে পারবেন যে মেমরি নিরাপত্তা সম্পর্কিত কোনও ত্রুটি অবশ্যই একটি `unsafe` ব্লকের মধ্যে থাকবে। `unsafe` ব্লকগুলিকে ছোট রাখুন; আপনি পরে যখন মেমরি বাগগুলি তদন্ত করবেন তখন আপনি কৃতজ্ঞ হবেন।

আনসেফ কোডকে যতটা সম্ভব আলাদা করতে, আনসেফ কোডকে একটি নিরাপদ অ্যাবস্ট্রাকশনের মধ্যে আবদ্ধ করা এবং একটি নিরাপদ API সরবরাহ করা ভাল, যা আমরা এই চ্যাপ্টারে পরে আলোচনা করব যখন আমরা আনসেফ ফাংশন এবং মেথডগুলি পরীক্ষা করব। স্ট্যান্ডার্ড লাইব্রেরির অংশগুলি আনসেফ কোডের উপর নিরাপদ অ্যাবস্ট্রাকশন হিসাবে প্রয়োগ করা হয় যা অডিট করা হয়েছে। একটি নিরাপদ অ্যাবস্ট্রাকশনের মধ্যে আনসেফ কোড র‍্যাপ করা `unsafe`-এর ব্যবহারগুলিকে সেই সমস্ত জায়গায় ছড়িয়ে যাওয়া থেকে আটকায় যেখানে আপনি বা আপনার ব্যবহারকারীরা `unsafe` কোড দিয়ে ইমপ্লিমেন্ট করা কার্যকারিতা ব্যবহার করতে চাইতে পারেন, কারণ একটি নিরাপদ অ্যাবস্ট্রাকশন ব্যবহার করা নিরাপদ।

আসুন একে একে পাঁচটি আনসেফ সুপারপাওয়ারের দিকে তাকাই। আমরা কিছু অ্যাবস্ট্রাকশনও দেখব যা আনসেফ কোডের জন্য একটি নিরাপদ ইন্টারফেস সরবরাহ করে।

### একটি র' পয়েন্টার ডিরেফারেন্স করা (Dereferencing a Raw Pointer)

Chapter 4-এর [“Dangling References”][dangling-references]<!-- ignore -->-এ, আমরা উল্লেখ করেছি যে কম্পাইলার নিশ্চিত করে যে রেফারেন্সগুলি সর্বদা বৈধ। আনসেফ রাস্ট-এ _র' পয়েন্টার_ নামে দুটি নতুন টাইপ রয়েছে যা রেফারেন্সের মতো। রেফারেন্সের মতো, র' পয়েন্টারগুলি ইমিউটেবল বা মিউটেবল হতে পারে এবং যথাক্রমে `*const T` এবং `*mut T` হিসাবে লেখা হয়। এখানে তারকাচিহ্নটি ডিরেফারেন্স অপারেটর নয়; এটি টাইপের নামের অংশ। র' পয়েন্টারের প্রসঙ্গে, _ইমিউটেবল_-এর অর্থ হল পয়েন্টারটি ডিরেফারেন্স করার পরে সরাসরি অ্যাসাইন করা যাবে না।

রেফারেন্স এবং স্মার্ট পয়েন্টার থেকে ভিন্ন, র' পয়েন্টার:

- ইমিউটেবল এবং মিউটেবল উভয় পয়েন্টার বা একই লোকেশনে একাধিক মিউটেবল পয়েন্টার রেখে বরোয়িং-এর নিয়ম উপেক্ষা করার অনুমতি দেওয়া হয়।
- বৈধ মেমরির দিকে নির্দেশ করার গ্যারান্টি দেওয়া হয় না
- নাল (null) হওয়ার অনুমতি দেওয়া হয়
- কোনও স্বয়ংক্রিয় ক্লিনিং ইমপ্লিমেন্ট করে না

Rust-কে এই গ্যারান্টিগুলি প্রয়োগ করা থেকে বিরত রেখে, আপনি আরও ভাল পারফরম্যান্স বা অন্য ভাষা বা হার্ডওয়্যারের সাথে ইন্টারফেস করার ক্ষমতার বিনিময়ে গ্যারান্টিযুক্ত নিরাপত্তা ছেড়ে দিতে পারেন যেখানে Rust-এর গ্যারান্টি প্রযোজ্য নয়।

Listing 20-1-এ দেখানো হয়েছে কিভাবে একটি ইমিউটেবল এবং একটি মিউটেবল র' পয়েন্টার তৈরি করা যায়।

<Listing number="20-1" caption="র' বরো অপারেটর দিয়ে র' পয়েন্টার তৈরি করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

লক্ষ্য করুন যে আমরা এই কোডে `unsafe` কীওয়ার্ডটি অন্তর্ভুক্ত করিনি। আমরা নিরাপদ কোডে র' পয়েন্টার তৈরি করতে পারি; আমরা কেবল একটি আনসেফ ব্লকের বাইরে র' পয়েন্টার ডিরেফারেন্স করতে পারি না, যেমনটি আপনি একটু পরেই দেখতে পাবেন।

আমরা র' বরো অপারেটর ব্যবহার করে র' পয়েন্টার তৈরি করেছি: `&raw const num` একটি `*const i32` ইমিউটেবল র' পয়েন্টার তৈরি করে এবং `&raw mut num` একটি `*mut
i32` মিউটেবল র' পয়েন্টার তৈরি করে। যেহেতু আমরা সেগুলি সরাসরি একটি লোকাল ভেরিয়েবল থেকে তৈরি করেছি, তাই আমরা জানি যে এই বিশেষ র' পয়েন্টারগুলি বৈধ, কিন্তু আমরা কোনও র' পয়েন্টার সম্পর্কে সেই অনুমান করতে পারি না।

এটি প্রদর্শন করার জন্য, পরবর্তীতে আমরা একটি র' পয়েন্টার তৈরি করব যার বৈধতা সম্পর্কে আমরা এতটা নিশ্চিত হতে পারি না, র' রেফারেন্স অপারেটর ব্যবহার করার পরিবর্তে একটি মান কাস্ট করতে `as` ব্যবহার করে। Listing 20-2 দেখায় কিভাবে মেমরির একটি নির্বিচারে লোকেশনে একটি র' পয়েন্টার তৈরি করতে হয়। নির্বিচারে মেমরি ব্যবহার করার চেষ্টা করা অনির্ধারিত: সেই অ্যাড্রেসে ডেটা থাকতে পারে বা নাও থাকতে পারে, কম্পাইলার কোডটিকে অপ্টিমাইজ করতে পারে যাতে কোনও মেমরি অ্যাক্সেস না থাকে, অথবা প্রোগ্রামটি একটি সেগমেন্টেশন ফল্ট সহ error করতে পারে। সাধারণত, এইরকম কোড লেখার কোনও ভাল কারণ নেই, বিশেষ করে এমন ক্ষেত্রে যেখানে আপনি পরিবর্তে একটি র' বরো অপারেটর ব্যবহার করতে পারেন, তবে এটি সম্ভব।

<Listing number="20-2" caption="একটি নির্বিচারে মেমরি ঠিকানায় একটি র' পয়েন্টার তৈরি করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

মনে রাখবেন যে আমরা নিরাপদ কোডে র' পয়েন্টার তৈরি করতে পারি, কিন্তু আমরা র' পয়েন্টারগুলিকে _ডিরেফারেন্স_ করতে পারি না এবং যে ডেটার দিকে নির্দেশ করা হচ্ছে তা পড়তে পারি না। Listing 20-3-এ, আমরা একটি র' পয়েন্টারে ডিরেফারেন্স অপারেটর `*` ব্যবহার করি যার জন্য একটি `unsafe` ব্লক প্রয়োজন।

<Listing number="20-3" caption="একটি `unsafe` ব্লকের মধ্যে র' পয়েন্টার ডিরেফারেন্স করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

একটি পয়েন্টার তৈরি করা কোনও ক্ষতি করে না; শুধুমাত্র তখনই যখন আমরা এটির নির্দেশিত মানটি অ্যাক্সেস করার চেষ্টা করি তখনই আমরা একটি অবৈধ মানের সম্মুখীন হতে পারি।

এছাড়াও লক্ষ্য করুন যে Listing 20-1 এবং 20-3-এ, আমরা `*const i32` এবং `*mut i32` র' পয়েন্টার তৈরি করেছি যেগুলি উভয়ই একই মেমরি লোকেশনের দিকে নির্দেশ করে, যেখানে `num` সংরক্ষণ করা হয়। যদি আমরা পরিবর্তে `num`-এ একটি ইমিউটেবল এবং একটি মিউটেবল রেফারেন্স তৈরি করার চেষ্টা করতাম, তাহলে কোডটি কম্পাইল হত না কারণ Rust-এর ownership নিয়মগুলি একই সময়ে কোনও ইমিউটেবল রেফারেন্সের সাথে একটি মিউটেবল রেফারেন্সের অনুমতি দেয় না। র' পয়েন্টারগুলির সাহায্যে, আমরা একই লোকেশনে একটি মিউটেবল পয়েন্টার এবং একটি ইমিউটেবল পয়েন্টার তৈরি করতে পারি এবং মিউটেবল পয়েন্টারের মাধ্যমে ডেটা পরিবর্তন করতে পারি, সম্ভাব্যভাবে একটি ডেটা রেস তৈরি করতে পারি। সাবধান!

এই সমস্ত বিপদ থাকা সত্ত্বেও, আপনি কেন কখনও র' পয়েন্টার ব্যবহার করবেন? একটি প্রধান ব্যবহারের ক্ষেত্র হল C কোডের সাথে ইন্টারফেস করার সময়, যেমনটি আপনি পরবর্তী বিভাগে দেখতে পাবেন, [“Calling an Unsafe Function or Method.”](#calling-an-unsafe-function-or-method)<!-- ignore --> আরেকটি ক্ষেত্র হল নিরাপদ অ্যাবস্ট্রাকশন তৈরি করা যা বরো চেকার বুঝতে পারে না। আমরা আনসেফ ফাংশনগুলির পরিচয় দেব এবং তারপরে একটি নিরাপদ অ্যাবস্ট্রাকশনের উদাহরণ দেখব যা আনসেফ কোড ব্যবহার করে।

### একটি আনসেফ ফাংশন বা মেথড কল করা (Calling an Unsafe Function or Method)

আপনি একটি আনসেফ ব্লকে যে দ্বিতীয় ধরনের অপারেশন করতে পারেন তা হল আনসেফ ফাংশন কল করা। আনসেফ ফাংশন এবং মেথডগুলি দেখতে হুবহু রেগুলার ফাংশন এবং মেথডের মতোই, তবে সংজ্ঞার বাকি অংশের আগে তাদের একটি অতিরিক্ত `unsafe` রয়েছে। এই প্রসঙ্গে `unsafe` কীওয়ার্ডটি নির্দেশ করে যে ফাংশনটির প্রয়োজনীয়তা রয়েছে যা আমাদের এই ফাংশনটি কল করার সময় অবশ্যই বজায় রাখতে হবে, কারণ Rust গ্যারান্টি দিতে পারে না যে আমরা এই প্রয়োজনীয়তাগুলি পূরণ করেছি। একটি `unsafe` ব্লকের মধ্যে একটি আনসেফ ফাংশন কল করে, আমরা বলছি যে আমরা এই ফাংশনের ডকুমেন্টেশন পড়েছি এবং ফাংশনের শর্তাবলী বহাল রাখার দায়িত্ব নিচ্ছি।

এখানে `dangerous` নামে একটি আনসেফ ফাংশন রয়েছে যা তার বডিতে কিছুই করে না:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

আমাদের অবশ্যই একটি পৃথক `unsafe` ব্লকের মধ্যে `dangerous` ফাংশনটি কল করতে হবে। যদি আমরা `unsafe` ব্লক ছাড়া `dangerous` কল করার চেষ্টা করি, তাহলে আমরা একটি error পাব:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

`unsafe` ব্লকের মাধ্যমে, আমরা Rust-এর কাছে দাবী করছি যে আমরা ফাংশনের ডকুমেন্টেশন পড়েছি, আমরা কীভাবে এটি সঠিকভাবে ব্যবহার করতে হয় তা বুঝি এবং আমরা যাচাই করেছি যে আমরা ফাংশনের কন্ট্রাক্ট পূরণ করছি।

একটি আনসেফ ফাংশনের বডিতে আনসেফ অপারেশনগুলি সম্পাদন করতে, আপনাকে এখনও একটি রেগুলার ফাংশনের মতোই একটি `unsafe` ব্লক ব্যবহার করতে হবে এবং কম্পাইলার আপনাকে সতর্ক করবে যদি আপনি ভুলে যান। এটি `unsafe` ব্লকগুলিকে যতটা সম্ভব ছোট রাখতে সাহায্য করে, কারণ পুরো ফাংশন বডিতে আনসেফ অপারেশনের প্রয়োজন নাও হতে পারে।

#### আনসেফ কোডের উপর একটি নিরাপদ অ্যাবস্ট্রাকশন তৈরি করা (Creating a Safe Abstraction over Unsafe Code)

শুধুমাত্র একটি ফাংশনে আনসেফ কোড থাকার মানে এই নয় যে আমাদের পুরো ফাংশনটিকে আনসেফ হিসাবে চিহ্নিত করতে হবে। প্রকৃতপক্ষে, একটি নিরাপদ ফাংশনে আনসেফ কোড র‍্যাপ করা একটি সাধারণ অ্যাবস্ট্রাকশন। উদাহরণস্বরূপ, আসুন স্ট্যান্ডার্ড লাইব্রেরি থেকে `split_at_mut` ফাংশনটি অধ্যয়ন করি, যার জন্য কিছু আনসেফ কোড প্রয়োজন। আমরা অন্বেষণ করব কিভাবে আমরা এটি ইমপ্লিমেন্ট করতে পারি। এই নিরাপদ মেথডটি মিউটেবল স্লাইসের উপর সংজ্ঞায়িত করা হয়েছে: এটি একটি স্লাইস নেয় এবং আর্গুমেন্ট হিসাবে দেওয়া ইনডেক্সে স্লাইসটিকে বিভক্ত করে দুটি করে। Listing 20-4 দেখায় কিভাবে `split_at_mut` ব্যবহার করতে হয়।

<Listing number="20-4" caption="নিরাপদ `split_at_mut` ফাংশন ব্যবহার করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-4/src/main.rs:here}}
```

</Listing>

আমরা শুধুমাত্র নিরাপদ Rust ব্যবহার করে এই ফাংশনটি ইমপ্লিমেন্ট করতে পারি না। একটি প্রচেষ্টা Listing 20-5-এর মতো দেখতে হতে পারে, যা কম্পাইল হবে না। সরলতার জন্য, আমরা `split_at_mut`-কে একটি মেথডের পরিবর্তে একটি ফাংশন হিসাবে এবং শুধুমাত্র `i32` মানের স্লাইসের জন্য ইমপ্লিমেন্ট করব, জেনেরিক টাইপ `T`-এর জন্য নয়।

<Listing number="20-5" caption="শুধুমাত্র নিরাপদ Rust ব্যবহার করে `split_at_mut`-এর একটি সম্ভাব্য ইমপ্লিমেন্টেশন">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

এই ফাংশনটি প্রথমে স্লাইসের মোট দৈর্ঘ্য পায়। তারপর এটি নিশ্চিত করে যে প্যারামিটার হিসাবে দেওয়া ইনডেক্সটি স্লাইসের মধ্যে রয়েছে কিনা তা পরীক্ষা করে যে এটি দৈর্ঘ্যের চেয়ে কম বা সমান কিনা। এই দাবীটির অর্থ হল যে আমরা যদি স্লাইসটিকে বিভক্ত করার জন্য দৈর্ঘ্যের চেয়ে বড় একটি ইনডেক্স পাস করি, তাহলে ফাংশনটি সেই ইনডেক্সটি ব্যবহার করার চেষ্টা করার আগেই প্যানিক করবে।

তারপর আমরা একটি টাপলে দুটি মিউটেবল স্লাইস রিটার্ন করি: একটি মূল স্লাইসের শুরু থেকে `mid` ইনডেক্স পর্যন্ত এবং অন্যটি `mid` থেকে স্লাইসের শেষ পর্যন্ত।

আমরা যখন Listing 20-5-এর কোড কম্পাইল করার চেষ্টা করি, তখন আমরা একটি error পাব।

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Rust-এর বরো চেকার বুঝতে পারে না যে আমরা স্লাইসের বিভিন্ন অংশ বরো করছি; এটি শুধুমাত্র জানে যে আমরা একই স্লাইস থেকে দুবার বরো করছি। একটি স্লাইসের বিভিন্ন অংশ বরো করা মৌলিকভাবে ঠিক আছে কারণ দুটি স্লাইস ওভারল্যাপ করছে না, কিন্তু Rust এটি বোঝার মতো যথেষ্ট স্মার্ট নয়। যখন আমরা জানি কোড ঠিক আছে, কিন্তু Rust জানে না, তখন আনসেফ কোডের কাছে পৌঁছানোর সময়।

Listing 20-6 দেখায় কিভাবে একটি `unsafe` ব্লক, একটি র' পয়েন্টার এবং কিছু আনসেফ ফাংশন কল ব্যবহার করে `split_at_mut`-এর ইমপ্লিমেন্টেশন কাজ করানো যায়।

<Listing number="20-6" caption="`split_at_mut` ফাংশনের ইমপ্লিমেন্টেশনে আনসেফ কোড ব্যবহার করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Chapter 4-এর [“The Slice Type”][the-slice-type]<!-- ignore --> থেকে মনে করুন যে স্লাইসগুলি হল কিছু ডেটার একটি পয়েন্টার এবং স্লাইসের দৈর্ঘ্য। আমরা একটি স্লাইসের দৈর্ঘ্য পেতে `len` মেথড এবং একটি স্লাইসের র' পয়েন্টার অ্যাক্সেস করতে `as_mut_ptr` মেথড ব্যবহার করি। এক্ষেত্রে, যেহেতু আমাদের কাছে `i32` মানের একটি মিউটেবল স্লাইস রয়েছে, তাই `as_mut_ptr` `*mut i32` টাইপের একটি র' পয়েন্টার রিটার্ন করে, যা আমরা `ptr` ভেরিয়েবলে সংরক্ষণ করেছি।

আমরা এই দাবীটি রাখি যে `mid` ইনডেক্সটি স্লাইসের মধ্যে রয়েছে। তারপর আমরা আনসেফ কোডে আসি: `slice::from_raw_parts_mut` ফাংশনটি একটি র' পয়েন্টার এবং একটি দৈর্ঘ্য নেয় এবং এটি একটি স্লাইস তৈরি করে। আমরা এই ফাংশনটি ব্যবহার করে একটি স্লাইস তৈরি করি যা `ptr` থেকে শুরু হয় এবং `mid` সংখ্যক আইটেম দীর্ঘ। তারপর আমরা `ptr`-এ `mid` কে আর্গুমেন্ট হিসাবে দিয়ে `add` মেথড কল করি যাতে একটি র' পয়েন্টার পাওয়া যায় যা `mid` থেকে শুরু হয় এবং আমরা সেই পয়েন্টার এবং `mid`-এর পরের অবশিষ্ট সংখ্যক আইটেম ব্যবহার করে একটি স্লাইস তৈরি করি দৈর্ঘ্যের জন্য।

`slice::from_raw_parts_mut` ফাংশনটি আনসেফ কারণ এটি একটি র' পয়েন্টার নেয় এবং অবশ্যই বিশ্বাস করতে হবে যে এই পয়েন্টারটি বৈধ। র' পয়েন্টারগুলিতে `add` মেথডটিও আনসেফ, কারণ এটিকে অবশ্যই বিশ্বাস করতে হবে যে অফসেট লোকেশনটিও একটি বৈধ পয়েন্টার। অতএব, আমাদের `slice::from_raw_parts_mut` এবং `add`-এর চারপাশে একটি `unsafe` ব্লক রাখতে হয়েছিল যাতে আমরা সেগুলি কল করতে পারি। কোডটি দেখে এবং `mid` অবশ্যই `len`-এর থেকে কম বা সমান হতে হবে এই দাবী যোগ করে, আমরা বলতে পারি যে `unsafe` ব্লকের মধ্যে ব্যবহৃত সমস্ত র' পয়েন্টারগুলি স্লাইসের মধ্যে ডেটার বৈধ পয়েন্টার হবে। এটি `unsafe`-এর একটি গ্রহণযোগ্য এবং উপযুক্ত ব্যবহার।

লক্ষ্য করুন যে আমাদের ফলস্বরূপ `split_at_mut` ফাংশনটিকে `unsafe` হিসাবে চিহ্নিত করার প্রয়োজন নেই এবং আমরা নিরাপদ Rust থেকে এই ফাংশনটিকে কল করতে পারি। আমরা ফাংশনের একটি ইমপ্লিমেন্টেশন সহ আনসেফ কোডের একটি নিরাপদ অ্যাবস্ট্রাকশন তৈরি করেছি যা নিরাপদ উপায়ে `unsafe` কোড ব্যবহার করে, কারণ এটি শুধুমাত্র সেই ডেটা থেকে বৈধ পয়েন্টার তৈরি করে যাতে এই ফাংশনটির অ্যাক্সেস রয়েছে।

বিপরীতে, Listing 20-7-এ `slice::from_raw_parts_mut`-এর ব্যবহার সম্ভবত ক্র্যাশ করবে যখন স্লাইসটি ব্যবহার করা হবে। এই কোডটি একটি নির্বিচারে মেমরি লোকেশন নেয় এবং 10,000 আইটেম দীর্ঘ একটি স্লাইস তৈরি করে।

<Listing number="20-7" caption="একটি নির্বিচারে মেমরি লোকেশন থেকে একটি স্লাইস তৈরি করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

আমরা এই নির্বিচারে লোকেশনে মেমরির মালিক নই এবং এই কোডটি যে স্লাইস তৈরি করে তাতে বৈধ `i32` মান রয়েছে তার কোনও গ্যারান্টি নেই। `values` ব্যবহার করার চেষ্টা করা যেন এটি একটি বৈধ স্লাইস, এটি অনির্ধারিত আচরণের দিকে পরিচালিত করে।

#### এক্সটার্ন ফাংশন ব্যবহার করে বাহ্যিক কোড কল করা (`Using `extern` Functions to Call External Code)

কখনও কখনও, আপনার Rust কোডের অন্য ভাষায় লেখা কোডের সাথে ইন্টারঅ্যাক্ট করার প্রয়োজন হতে পারে। এর জন্য, Rust-এর `extern` কীওয়ার্ড রয়েছে যা একটি _ফরেন ফাংশন ইন্টারফেস (FFI)_ তৈরি এবং ব্যবহারে সহায়তা করে। একটি FFI হল একটি প্রোগ্রামিং ভাষার জন্য ফাংশন সংজ্ঞায়িত করার এবং একটি ভিন্ন (বিদেশী) প্রোগ্রামিং ভাষাকে সেই ফাংশনগুলিকে কল করার অনুমতি দেওয়ার একটি উপায়।

Listing 20-8 C স্ট্যান্ডার্ড লাইব্রেরি থেকে `abs` ফাংশনের সাথে একটি ইন্টিগ্রেশন কীভাবে সেট আপ করতে হয় তা প্রদর্শন করে। `extern` ব্লকের মধ্যে ঘোষিত ফাংশনগুলি সাধারণত Rust কোড থেকে কল করা অনিরাপদ, তাই তাদের অবশ্যই `unsafe` হিসাবে চিহ্নিত করতে হবে। এর কারণ হল অন্য ভাষাগুলি Rust-এর নিয়ম এবং গ্যারান্টিগুলি প্রয়োগ করে না এবং Rust সেগুলি পরীক্ষা করতে পারে না, তাই নিরাপত্তা নিশ্চিত করার দায়িত্ব প্রোগ্রামারের উপর বর্তায়।

<Listing number="20-8" file-name="src/main.rs" caption="অন্যান্য ভাষায় সংজ্ঞায়িত extern ফাংশন ঘোষণা এবং কল করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

`unsafe extern "C"` ব্লকের মধ্যে, আমরা অন্য ভাষা থেকে যে বাহ্যিক ফাংশনগুলিকে কল করতে চাই তাদের নাম এবং স্বাক্ষর তালিকাভুক্ত করি। `"C"` অংশটি সংজ্ঞায়িত করে যে বাহ্যিক ফাংশনটি কোন _অ্যাপ্লিকেশন বাইনারি ইন্টারফেস (ABI)_ ব্যবহার করে: ABI সংজ্ঞায়িত করে কিভাবে অ্যাসেম্বলি স্তরে ফাংশনটিকে কল করতে হয়। `"C"` ABI হল সবচেয়ে সাধারণ এবং C প্রোগ্রামিং ভাষার ABI অনুসরণ করে।

এই বিশেষ ফাংশনটির কোন মেমরি নিরাপত্তা বিবেচনা নেই। আসলে, আমরা জানি যে `abs`-এ যেকোনো কল সর্বদাই যেকোনো `i32`-এর জন্য নিরাপদ হবে, তাই আমরা `safe` কীওয়ার্ড ব্যবহার করে বলতে পারি যে এই নির্দিষ্ট ফাংশনটি কল করা নিরাপদ, যদিও এটি একটি `unsafe extern` ব্লকে রয়েছে। একবার আমরা সেই পরিবর্তনটি করলে, এটিকে কল করার জন্য আর একটি `unsafe` ব্লকের প্রয়োজন হয় না, যেমনটি Listing 20-9-এ দেখানো হয়েছে।

<Listing number="20-9" file-name="src/main.rs" caption="একটি `unsafe extern` ব্লকের মধ্যে একটি ফাংশনকে স্পষ্টভাবে `safe` হিসাবে চিহ্নিত করা এবং এটিকে নিরাপদে কল করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

একটি ফাংশনকে `safe` হিসাবে চিহ্নিত করা এটিকে সহজাতভাবে নিরাপদ করে না! পরিবর্তে, এটি এমন একটি প্রতিশ্রুতির মতো যা আপনি Rust-কে দিচ্ছেন যে এটি নিরাপদ। সেই প্রতিশ্রুতি রাখা হয়েছে কিনা তা নিশ্চিত করা এখনও আপনার দায়িত্ব!

> #### অন্যান্য ভাষা থেকে Rust ফাংশন কল করা (Calling Rust Functions from Other Languages)
>
> আমরা `extern` ব্যবহার করে একটি ইন্টারফেস তৈরি করতে পারি যা অন্যান্য ভাষাগুলিকে Rust ফাংশন কল করার অনুমতি দেয়। একটি সম্পূর্ণ `extern` ব্লক তৈরি করার পরিবর্তে, আমরা `extern` কীওয়ার্ড যোগ করি এবং প্রাসঙ্গিক ফাংশনের জন্য `fn` কীওয়ার্ডের ঠিক আগে ব্যবহার করার জন্য ABI নির্দিষ্ট করি। আমাদের এই ফাংশনের নাম ম্যাংগল না করার জন্য Rust কম্পাইলারকে বলার জন্য একটি `#[unsafe(no_mangle)]` অ্যানোটেশনও যোগ করতে হবে। _ম্যাংলিং_ হল যখন একটি কম্পাইলার আমাদের দেওয়া একটি ফাংশনের নাম পরিবর্তন করে একটি ভিন্ন নামে পরিবর্তন করে যাতে কম্পাইলেশন প্রক্রিয়ার অন্যান্য অংশগুলির জন্য আরও তথ্য থাকে কিন্তু মানুষের পাঠযোগ্যতা কম থাকে। প্রতিটি প্রোগ্রামিং ভাষার কম্পাইলার নামগুলিকে সামান্য ভিন্নভাবে ম্যাংগল করে, তাই অন্য ভাষাগুলির দ্বারা একটি Rust ফাংশনকে নামযোগ্য করার জন্য, আমাদের অবশ্যই Rust কম্পাইলারের নাম ম্যাংলিং অক্ষম করতে হবে। এটি আনসেফ কারণ বিল্ট-ইন ম্যাংলিং ছাড়াই লাইব্রেরি জুড়ে নামের সংঘর্ষ হতে পারে, তাই আমরা যে নামটি এক্সপোর্ট করেছি তা ম্যাংলিং ছাড়াই এক্সপোর্ট করা নিরাপদ কিনা তা নিশ্চিত করা আমাদের দায়িত্ব।
>
>
> নিম্নলিখিত উদাহরণে, আমরা `call_from_c` ফাংশনটিকে C কোড থেকে অ্যাক্সেসযোগ্য করে তুলি, এটিকে একটি শেয়ার্ড লাইব্রেরিতে কম্পাইল করার এবং C থেকে লিঙ্ক করার পরে:
>
> ```rust
> #[unsafe(no_mangle)]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> `extern`-এর এই ব্যবহারের জন্য `unsafe` প্রয়োজন হয় না।

### একটি মিউটেবল স্ট্যাটিক ভেরিয়েবল অ্যাক্সেস বা পরিবর্তন করা (Accessing or Modifying a Mutable Static Variable)

এই বইটিতে, আমরা এখনও _গ্লোবাল ভেরিয়েবল_ সম্পর্কে কথা বলিনি, যা Rust সমর্থন করে কিন্তু Rust-এর ownership নিয়মের সাথে সমস্যাযুক্ত হতে পারে। যদি দুটি থ্রেড একই মিউটেবল গ্লোবাল ভেরিয়েবল অ্যাক্সেস করে, তাহলে এটি একটি ডেটা রেসের কারণ হতে পারে।

Rust-এ, গ্লোবাল ভেরিয়েবলগুলিকে _স্ট্যাটিক_ ভেরিয়েবল বলা হয়। Listing 20-10 একটি স্ট্যাটিক ভেরিয়েবলের উদাহরণ ঘোষণা এবং ব্যবহার দেখায় যার মান একটি স্ট্রিং স্লাইস।

<Listing number="20-10" file-name="src/main.rs" caption="একটি অপরিবর্তনীয় স্ট্যাটিক ভেরিয়েবল সংজ্ঞায়িত করা এবং ব্যবহার করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

স্ট্যাটিক ভেরিয়েবলগুলি কনস্ট্যান্টের মতোই, যা আমরা Chapter 3-এর [“Constants”][differences-between-variables-and-constants]<!-- ignore -->-এ আলোচনা করেছি। স্ট্যাটিক ভেরিয়েবলের নামগুলি রীতি অনুযায়ী `SCREAMING_SNAKE_CASE`-এ থাকে। স্ট্যাটিক ভেরিয়েবলগুলি শুধুমাত্র `'static` লাইফটাইম সহ রেফারেন্স সংরক্ষণ করতে পারে, যার অর্থ হল Rust কম্পাইলার লাইফটাইম বের করতে পারে এবং আমাদের এটিকে স্পষ্টভাবে অ্যানোটেট করার প্রয়োজন নেই। একটি অপরিবর্তনীয় স্ট্যাটিক ভেরিয়েবল অ্যাক্সেস করা নিরাপদ।

কনস্ট্যান্ট এবং অপরিবর্তনীয় স্ট্যাটিক ভেরিয়েবলের মধ্যে একটি সূক্ষ্ম পার্থক্য হল যে একটি স্ট্যাটিক ভেরিয়েবলের মানগুলির মেমরিতে একটি নির্দিষ্ট ঠিকানা থাকে। মান ব্যবহার করা সর্বদা একই ডেটা অ্যাক্সেস করবে। অন্যদিকে, কনস্ট্যান্টগুলিকে যখনই ব্যবহার করা হয় তখনই তাদের ডেটা ডুপ্লিকেট করার অনুমতি দেওয়া হয়। আরেকটি পার্থক্য হল স্ট্যাটিক ভেরিয়েবলগুলি মিউটেবল হতে পারে। মিউটেবল স্ট্যাটিক ভেরিয়েবল অ্যাক্সেস এবং পরিবর্তন করা _আনসেফ_। Listing 20-11 দেখায় কিভাবে `COUNTER` নামে একটি মিউটেবল স্ট্যাটিক ভেরিয়েবল ঘোষণা, অ্যাক্সেস এবং পরিবর্তন করতে হয়।

<Listing number="20-11" file-name="src/main.rs" caption="একটি মিউটেবল স্ট্যাটিক ভেরিয়েবল থেকে পড়া বা লেখা অনিরাপদ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

নিয়মিত ভেরিয়েবলের মতো, আমরা `mut` কীওয়ার্ড ব্যবহার করে মিউটেবিলিটি নির্দিষ্ট করি। যে কোনও কোড যা `COUNTER` থেকে পড়ে বা লেখে তা অবশ্যই একটি `unsafe` ব্লকের মধ্যে থাকতে হবে। Listing 20-11-এর কোডটি কম্পাইল করে এবং আমরা যেমন আশা করি `COUNTER: 3` প্রিন্ট করে কারণ এটি সিঙ্গেল থ্রেডেড। একাধিক থ্রেড `COUNTER` অ্যাক্সেস করলে সম্ভবত ডেটা রেস হবে, তাই এটি অনির্ধারিত আচরণ। অতএব, আমাদের সম্পূর্ণ ফাংশনটিকে `unsafe` হিসাবে চিহ্নিত করতে হবে এবং নিরাপত্তার সীমাবদ্ধতা নথিভুক্ত করতে হবে, যাতে ফাংশনটিকে কল করা যে কেউ জানে যে তারা কী করতে পারে এবং কী করতে পারে না।

যখনই আমরা একটি আনসেফ ফাংশন লিখি, তখন `SAFETY` দিয়ে শুরু করে একটি মন্তব্য লেখা এবং কলারকে ফাংশনটি নিরাপদে কল করার জন্য কী করতে হবে তা ব্যাখ্যা করা প্রথাগত। একইভাবে, যখনই আমরা একটি আনসেফ অপারেশন করি, তখন নিরাপত্তার নিয়মগুলি কীভাবে বহাল রাখা হয় তা ব্যাখ্যা করার জন্য `SAFETY` দিয়ে শুরু করে একটি মন্তব্য লেখা প্রথাগত।

অতিরিক্তভাবে, কম্পাইলার আপনাকে একটি মিউটেবল স্ট্যাটিক ভেরিয়েবলের রেফারেন্স তৈরি করার অনুমতি দেবে না। আপনি শুধুমাত্র একটি র' পয়েন্টারের মাধ্যমে এটি অ্যাক্সেস করতে পারেন, যা র' বরো অপারেটরগুলির মধ্যে একটি দিয়ে তৈরি করা হয়। এর মধ্যে এমন ক্ষেত্রগুলিও রয়েছে যেখানে রেফারেন্সটি অদৃশ্যভাবে তৈরি করা হয়, যেমন যখন এটি এই কোড লিস্টিং-এর `println!`-এ ব্যবহৃত হয়। স্ট্যাটিক মিউটেবল ভেরিয়েবলের রেফারেন্স শুধুমাত্র র' পয়েন্টারের মাধ্যমে তৈরি করা যেতে পারে এমন প্রয়োজনীয়তা তাদের ব্যবহারের জন্য নিরাপত্তার প্রয়োজনীয়তাগুলিকে আরও স্পষ্ট করতে সহায়তা করে।

মিউটেবল ডেটা যা বিশ্বব্যাপী অ্যাক্সেসযোগ্য, সেখানে কোনও ডেটা রেস নেই তা নিশ্চিত করা কঠিন, যে কারণে Rust মিউটেবল স্ট্যাটিক ভেরিয়েবলগুলিকে আনসেফ বলে মনে করে। যেখানে সম্ভব, সেখানে কনকারেন্সি কৌশল এবং থ্রেড-নিরাপদ স্মার্ট পয়েন্টারগুলি ব্যবহার করা বাঞ্ছনীয় যা আমরা Chapter 16-এ আলোচনা করেছি যাতে কম্পাইলার পরীক্ষা করে যে বিভিন্ন থ্রেড থেকে অ্যাক্সেস করা ডেটা নিরাপদে করা হয়েছে।

### একটি আনসেফ ট্রেইট ইমপ্লিমেন্ট করা (Implementing an Unsafe Trait)

আমরা একটি আনসেফ ট্রেইট ইমপ্লিমেন্ট করতে `unsafe` ব্যবহার করতে পারি। একটি ট্রেইট আনসেফ হয় যখন এর অন্তত একটি মেথডের কিছু ইনভেরিয়েন্ট থাকে যা কম্পাইলার যাচাই করতে পারে না। আমরা `trait`-এর আগে `unsafe` কীওয়ার্ড যোগ করে এবং ট্রেইটের ইমপ্লিমেন্টেশনকেও `unsafe` হিসাবে চিহ্নিত করে ঘোষণা করি যে একটি ট্রেইট `unsafe`, যেমনটি Listing 20-12-এ দেখানো হয়েছে।

<Listing number="20-12" caption="একটি আনসেফ ট্রেইট সংজ্ঞায়িত করা এবং ইমপ্লিমেন্ট করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs}}
```

</Listing>

`unsafe impl` ব্যবহার করে, আমরা প্রতিশ্রুতি দিচ্ছি যে আমরা সেই ইনভেরিয়েন্টগুলি বহাল রাখব যা কম্পাইলার যাচাই করতে পারে না।

একটি উদাহরণ হিসাবে, [“Extensible Concurrency with the `Sync` and `Send` Traits”][extensible-concurrency-with-the-sync-and-send-traits]<!-- ignore --> in Chapter 16-এ আমরা যে `Sync` এবং `Send` মার্কার ট্রেইটগুলি নিয়ে আলোচনা করেছি তা স্মরণ করুন: কম্পাইলার এই ট্রেইটগুলি স্বয়ংক্রিয়ভাবে ইমপ্লিমেন্ট করে যদি আমাদের টাইপগুলি সম্পূর্ণরূপে `Send` এবং `Sync` টাইপ দ্বারা গঠিত হয়। যদি আমরা এমন একটি টাইপ ইমপ্লিমেন্ট করি যাতে এমন একটি টাইপ থাকে যা `Send` বা `Sync` নয়, যেমন র' পয়েন্টার, এবং আমরা সেই টাইপটিকে `Send` বা `Sync` হিসাবে চিহ্নিত করতে চাই, তাহলে আমাদের অবশ্যই `unsafe` ব্যবহার করতে হবে। Rust যাচাই করতে পারে না যে আমাদের টাইপটি এই গ্যারান্টিগুলি বহাল রাখে যে এটি নিরাপদে থ্রেড জুড়ে পাঠানো যেতে পারে বা একাধিক থ্রেড থেকে অ্যাক্সেস করা যেতে পারে; অতএব, আমাদের সেই পরীক্ষাগুলি ম্যানুয়ালি করতে হবে এবং সেই অনুযায়ী `unsafe` দিয়ে নির্দেশ করতে হবে।

### একটি ইউনিয়নের ক্ষেত্রগুলি অ্যাক্সেস করা (Accessing Fields of a Union)

`unsafe` দিয়ে কাজ করে এমন চূড়ান্ত অ্যাকশন হল একটি _ইউনিয়ন_-এর ক্ষেত্রগুলি অ্যাক্সেস করা। একটি `union` একটি `struct`-এর মতোই, কিন্তু একটি নির্দিষ্ট দৃষ্টান্তে একবারে শুধুমাত্র একটি ঘোষিত ক্ষেত্র ব্যবহার করা হয়। ইউনিয়নগুলি প্রাথমিকভাবে C কোডের ইউনিয়নগুলির সাথে ইন্টারফেস করতে ব্যবহৃত হয়। ইউনিয়ন ক্ষেত্রগুলি অ্যাক্সেস করা অনিরাপদ কারণ Rust বর্তমানে ইউনিয়ন ইনস্ট্যান্সে সংরক্ষিত ডেটার টাইপ গ্যারান্টি দিতে পারে না। আপনি [Rust Reference][reference]-এ ইউনিয়ন সম্পর্কে আরও জানতে পারেন।

### আনসেফ কোড পরীক্ষা করার জন্য Miri ব্যবহার করা (Using Miri to check unsafe code)

আনসেফ কোড লেখার সময়, আপনি পরীক্ষা করতে চাইতে পারেন যে আপনি যা লিখেছেন তা আসলে নিরাপদ এবং সঠিক কিনা। এটি করার অন্যতম সেরা উপায় হল [Miri][miri] ব্যবহার করা, যা অনির্ধারিত আচরণ সনাক্ত করার জন্য একটি অফিশিয়াল Rust টুল। যেখানে বরো চেকার একটি _স্ট্যাটিক_ টুল যা কম্পাইল করার সময় কাজ করে, Miri হল একটি _ডায়নামিক_ টুল যা রানটাইমে কাজ করে। এটি আপনার প্রোগ্রাম বা এর টেস্ট স্যুট চালানোর মাধ্যমে আপনার কোড পরীক্ষা করে এবং আপনি যখন Rust কীভাবে কাজ করা উচিত সে সম্পর্কে এটি যে নিয়মগুলি বোঝে তা লঙ্ঘন করেন তখন সনাক্ত করে।

Miri ব্যবহার করার জন্য Rust-এর একটি নাইটলি বিল্ড প্রয়োজন (যা আমরা [Appendix G: How Rust is Made and “Nightly Rust”][nightly])-এ আরও আলোচনা করব)। আপনি `rustup +nightly component add miri` টাইপ করে Rust-এর একটি নাইটলি ভার্সন এবং Miri টুল উভয়ই ইনস্টল করতে পারেন। এটি আপনার প্রোজেক্ট যে Rust ভার্সন ব্যবহার করে তা পরিবর্তন করে না; এটি শুধুমাত্র আপনার সিস্টেমে টুলটি যোগ করে যাতে আপনি যখন চান তখন এটি ব্যবহার করতে পারেন। আপনি `cargo +nightly miri run` বা `cargo +nightly miri test` টাইপ করে একটি প্রোজেক্টে Miri চালাতে পারেন।

এটি কতটা সহায়ক হতে পারে তার একটি উদাহরণ হিসাবে, Listing 20-11-এর বিরুদ্ধে আমরা যখন এটি চালাই তখন কী ঘটে তা বিবেচনা করুন:

```console
{{#include ../listings/ch20-advanced-features/listing-20-11/output.txt}}
```

এটি সহায়কভাবে এবং সঠিকভাবে লক্ষ্য করে যে আমাদের কাছে মিউটেবল ডেটার শেয়ার্ড রেফারেন্স রয়েছে এবং এটি সম্পর্কে সতর্ক করে। এক্ষেত্রে, এটি আমাদের বলে না কিভাবে সমস্যাটি ঠিক করতে হয়, তবে এর মানে হল যে আমরা জানি যে একটি সম্ভাব্য সমস্যা রয়েছে এবং কীভাবে এটি নিরাপদ তা নিশ্চিত করতে হবে সে সম্পর্কে ভাবতে পারি। অন্যান্য ক্ষেত্রে, এটি আসলে আমাদের বলতে পারে যে কিছু কোড _অবশ্যই_ ভুল এবং এটি কীভাবে ঠিক করতে হবে সে সম্পর্কে সুপারিশ করতে পারে।

Miri আনসেফ কোড লেখার সময় আপনি যা ভুল করতে পারেন তার _সবকিছু_ ধরে না। একটি জিনিসের জন্য, যেহেতু এটি একটি ডায়নামিক চেক, এটি শুধুমাত্র সেই কোডের সাথে সমস্যাগুলি ধরে যা আসলে চালানো হয়। এর মানে হল যে আপনি যে আনসেফ কোড লিখেছেন সে সম্পর্কে আপনার আত্মবিশ্বাস বাড়ানোর জন্য আপনাকে এটিকে ভাল পরীক্ষার কৌশলগুলির সাথে ব্যবহার করতে হবে। অন্য একটি জিনিসের জন্য, এটি আপনার কোডটি আনসাউন্ড হতে পারে এমন প্রতিটি সম্ভাব্য উপায় কভার করে না। যদি Miri একটি সমস্যা _ধরে_, তাহলে আপনি জানেন যে একটি বাগ আছে, কিন্তু শুধু এই কারণে যে Miri একটি বাগ _ধরে না_ তার মানে এই নয় যে কোনও সমস্যা নেই। Miri অনেক কিছু ধরতে পারে, যদিও। এই চ্যাপ্টারের আনসেফ কোডের অন্যান্য উদাহরণগুলিতে এটি চালানোর চেষ্টা করুন এবং দেখুন এটি কী বলে!

### কখন আনসেফ কোড ব্যবহার করবেন (When to Use Unsafe Code)

এইমাত্র আলোচিত পাঁচটি অ্যাকশন (সুপারপাওয়ার) নেওয়ার জন্য `unsafe` ব্যবহার করা ভুল বা এমনকি ভ্রুকুটি করাও নয়। তবে কম্পাইলার মেমরি নিরাপত্তা বহাল রাখতে সাহায্য করতে পারে না বলে `unsafe` কোড সঠিক করা আরও কঠিন। যখন আপনার `unsafe` কোড ব্যবহার করার একটি কারণ থাকে, তখন আপনি তা করতে পারেন এবং একটি স্পষ্ট `unsafe` অ্যানোটেশন থাকা সমস্যাগুলি দেখা দিলে সমস্যার উৎস ট্র্যাক করা সহজ করে তোলে। আপনি যখনই আনসেফ কোড লেখেন, আপনি Miri ব্যবহার করে আরও আত্মবিশ্বাসী হতে পারেন যে আপনি যে কোডটি লিখেছেন সেটি Rust-এর নিয়মগুলি বহাল রাখে।

আনসেফ রাস্ট-এর সাথে কার্যকরভাবে কীভাবে কাজ করতে হয় তার আরও গভীর অনুসন্ধানের জন্য, Rust-এর এই বিষয়ের উপর অফিশিয়াল গাইড, [Rustonomicon][nomicon] পড়ুন।

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[differences-between-variables-and-constants]: ch03-01-variables-and-mutability.html#constants
[extensible-concurrency-with-the-sync-and-send-traits]: ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits
[the-slice-type]: ch04-03-slices.html#the-slice-type
[reference]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
