## স্কোপ এবং গোপনীয়তা নিয়ন্ত্রণ করতে মডিউল সংজ্ঞায়িত করা (Defining Modules to Control Scope and Privacy)

এই বিভাগে, আমরা মডিউল এবং মডিউল সিস্টেমের অন্যান্য অংশগুলো নিয়ে কথা বলব, যেমন *পাথ (paths)*, যা আপনাকে আইটেমগুলোর নাম দিতে দেয়; `use` কীওয়ার্ড যা একটি পাথকে স্কোপে নিয়ে আসে; এবং `pub` কীওয়ার্ড যা আইটেমগুলোকে পাবলিক করে। আমরা `as` কীওয়ার্ড, এক্সটার্নাল প্যাকেজ এবং গ্লোব অপারেটর নিয়েও আলোচনা করব।

### মডিউল চিট শিট (Modules Cheat Sheet)

মডিউল এবং পাথের বিশদ বিবরণে যাওয়ার আগে, এখানে মডিউল, পাথ, `use` কীওয়ার্ড এবং `pub` কীওয়ার্ড কম্পাইলারে কীভাবে কাজ করে এবং বেশিরভাগ ডেভেলপাররা কীভাবে তাদের কোড সংগঠিত করে তার একটি দ্রুত রেফারেন্স দেওয়া হলো। আমরা এই চ্যাপ্টার জুড়ে এই প্রতিটি নিয়মের উদাহরণ দেখব, তবে মডিউলগুলো কীভাবে কাজ করে তার একটি অনুস্মারক হিসাবে এটি মনে রাখার জন্য একটি দুর্দান্ত জায়গা।

-   **ক্রেট রুট থেকে শুরু করুন**: একটি ক্রেট কম্পাইল করার সময়, কম্পাইলার প্রথমে কোড কম্পাইল করার জন্য ক্রেট রুট ফাইলে (সাধারণত একটি লাইব্রেরি ক্রেটের জন্য _src/lib.rs_ বা একটি বাইনারি ক্রেটের জন্য _src/main.rs_) খোঁজে।
-   **মডিউল ঘোষণা করা**: ক্রেট রুট ফাইলে, আপনি নতুন মডিউল ঘোষণা করতে পারেন; ধরুন আপনি `mod garden;` দিয়ে একটি "গার্ডেন" মডিউল ঘোষণা করছেন। কম্পাইলার এই জায়গাগুলোতে মডিউলের কোড খুঁজবে:
    -   ইনলাইন, কার্লি ব্র্যাকেটের মধ্যে যা `mod garden`-এর পরে সেমিকোলন প্রতিস্থাপন করে
    -   _src/garden.rs_ ফাইলে
    -   _src/garden/mod.rs_ ফাইলে
-   **সাবমডিউল ঘোষণা করা**: ক্রেট রুট ছাড়া অন্য যেকোনো ফাইলে, আপনি সাবমডিউল ঘোষণা করতে পারেন। উদাহরণস্বরূপ, আপনি _src/garden.rs_-এ `mod vegetables;` ঘোষণা করতে পারেন। কম্পাইলার প্যারেন্ট মডিউলের জন্য নির্ধারিত ডিরেক্টরির মধ্যে নিম্নলিখিত জায়গাগুলোতে সাবমডিউলের কোড খুঁজবে:
    -   ইনলাইন, সরাসরি `mod vegetables`-এর পরে, কার্লি ব্র্যাকেটের মধ্যে, সেমিকোলনের পরিবর্তে
    -   _src/garden/vegetables.rs_ ফাইলে
    -   _src/garden/vegetables/mod.rs_ ফাইলে
-   **মডিউলের কোডের পাথ**: একবার একটি মডিউল আপনার ক্রেটের অংশ হয়ে গেলে, আপনি সেই মডিউলের কোডটি সেই একই ক্রেটের অন্য যেকোনো জায়গা থেকে রেফার করতে পারেন, যতক্ষণ গোপনীয়তার নিয়মগুলো অনুমতি দেয়, কোডের পাথ ব্যবহার করে। উদাহরণস্বরূপ, গার্ডেন ভেজিটেবলস মডিউলের একটি `Asparagus` টাইপ `crate::garden::vegetables::Asparagus`-এ পাওয়া যাবে।
-   **প্রাইভেট বনাম পাবলিক**: একটি মডিউলের ভেতরের কোড ডিফল্টরূপে তার প্যারেন্ট মডিউলগুলো থেকে প্রাইভেট থাকে। একটি মডিউলকে পাবলিক করতে, `mod`-এর পরিবর্তে `pub mod` দিয়ে এটি ঘোষণা করুন। একটি পাবলিক মডিউলের ভেতরের আইটেমগুলোকেও পাবলিক করতে, তাদের ঘোষণার আগে `pub` ব্যবহার করুন।
-   **`use` কীওয়ার্ড**: একটি স্কোপের মধ্যে, `use` কীওয়ার্ডটি আইটেমগুলোর শর্টকাট তৈরি করে যাতে লম্বা পথের পুনরাবৃত্তি কমানো যায়। যেকোনো স্কোপে যা `crate::garden::vegetables::Asparagus`-কে রেফার করতে পারে, আপনি `use crate::garden::vegetables::Asparagus;` দিয়ে একটি শর্টকাট তৈরি করতে পারেন এবং তারপর থেকে আপনাকে স্কোপে সেই টাইপটি ব্যবহার করার জন্য শুধুমাত্র `Asparagus` লিখতে হবে।

এখানে, আমরা `backyard` নামে একটি বাইনারি ক্রেট তৈরি করি যা এই নিয়মগুলো ব্যাখ্যা করে। ক্রেটের ডিরেক্টরি, যার নামও `backyard`, এই ফাইল এবং ডিরেক্টরিগুলো ধারণ করে:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

এই ক্ষেত্রে ক্রেট রুট ফাইলটি হল _src/main.rs_, এবং এতে রয়েছে:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

`pub mod garden;` লাইনটি কম্পাইলারকে _src/garden.rs_-এ পাওয়া কোড অন্তর্ভুক্ত করতে বলে, যেটি হল:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

এখানে, `pub mod vegetables;` মানে _src/garden/vegetables.rs_-এর কোডও অন্তর্ভুক্ত করা হয়েছে। সেই কোডটি হল:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

এবার চলুন এই নিয়মগুলোর বিস্তারিত বিবরণে যাই এবং সেগুলোকে অ্যাকশনে দেখি!

### মডিউলে সম্পর্কিত কোড গ্রুপ করা (Grouping Related Code in Modules)

*মডিউলগুলো* আমাদের পঠনযোগ্যতা এবং সহজে পুনরায় ব্যবহারের জন্য একটি ক্রেটের মধ্যে কোড সংগঠিত করতে দেয়। মডিউলগুলো আমাদের আইটেমগুলোর *গোপনীয়তা (privacy)* নিয়ন্ত্রণ করতে দেয়, কারণ একটি মডিউলের ভেতরের কোড ডিফল্টরূপে প্রাইভেট থাকে। প্রাইভেট আইটেমগুলো হল অভ্যন্তরীণ ইমপ্লিমেন্টেশনের বিবরণ যা বাইরের ব্যবহারের জন্য উপলব্ধ নয়। আমরা মডিউল এবং সেগুলোর ভেতরের আইটেমগুলোকে পাবলিক করতে পারি, যা সেগুলোকে এক্সটার্নাল কোডের ব্যবহার এবং সেগুলোর উপর নির্ভর করার অনুমতি দেয়।

উদাহরণস্বরূপ, আসুন একটি লাইব্রেরি ক্রেট লিখি যা একটি রেস্তোরাঁর কার্যকারিতা সরবরাহ করে। আমরা ফাংশনগুলোর সিগনেচার সংজ্ঞায়িত করব কিন্তু তাদের বডি খালি রাখব যাতে রেস্তোরাঁর ইমপ্লিমেন্টেশনের পরিবর্তে কোডের সংগঠনের উপর মনোযোগ দেওয়া যায়।

রেস্তোরাঁ শিল্পে, একটি রেস্তোরাঁর কিছু অংশকে *ফ্রন্ট অফ হাউস (front of house)* এবং অন্যগুলোকে *ব্যাক অফ হাউস (back of house)* হিসাবে উল্লেখ করা হয়। ফ্রন্ট অফ হাউস হল যেখানে গ্রাহকরা থাকে; এর মধ্যে রয়েছে যেখানে হোস্টরা গ্রাহকদের বসায়, সার্ভাররা অর্ডার এবং পেমেন্ট নেয় এবং বারটেন্ডাররা পানীয় তৈরি করে। ব্যাক অফ হাউস হল যেখানে শেফ এবং বাবুর্চিরা রান্নাঘরে কাজ করে, ডিশওয়াশাররা পরিষ্কার করে এবং ম্যানেজাররা প্রশাসনিক কাজ করে।

এইভাবে আমাদের ক্রেটটিকে স্ট্রাকচার করার জন্য, আমরা এর ফাংশনগুলোকে নেস্টেড মডিউলে সংগঠিত করতে পারি। `cargo new restaurant --lib` চালিয়ে `restaurant` নামে একটি নতুন লাইব্রেরি তৈরি করুন। তারপর Listing 7-1-এর কোডটি _src/lib.rs_-এ লিখুন কিছু মডিউল এবং ফাংশন সিগনেচার সংজ্ঞায়িত করতে; এই কোডটি হল ফ্রন্ট অফ হাউস বিভাগ।

<Listing number="7-1" file-name="src/lib.rs" caption="অন্যান্য মডিউল ধারণকারী একটি `front_of_house` মডিউল এবং তারপর ফাংশন ধারণকারী">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

আমরা `mod` কীওয়ার্ড দিয়ে একটি মডিউল সংজ্ঞায়িত করি এবং তারপর মডিউলের নাম দিই (এই ক্ষেত্রে, `front_of_house`)। মডিউলের বডি তারপর কার্লি ব্র্যাকেটের ভিতরে যায়। মডিউলগুলোর ভিতরে, আমরা অন্যান্য মডিউল রাখতে পারি, যেমনটি এই ক্ষেত্রে `hosting` এবং `serving` মডিউলগুলোর সাথে করা হয়েছে। মডিউলগুলো অন্যান্য আইটেমগুলোর জন্যও সংজ্ঞা রাখতে পারে, যেমন স্ট্রাকট, এনাম, কনস্ট্যান্ট, ট্রেইট এবং—Listing 7-1-এর মতো—ফাংশন।

মডিউল ব্যবহার করে, আমরা সম্পর্কিত সংজ্ঞাগুলোকে একসাথে গ্রুপ করতে পারি এবং কেন সেগুলো সম্পর্কিত তার নাম দিতে পারি। এই কোডটি ব্যবহার করা প্রোগ্রামাররা সমস্ত সংজ্ঞা পড়ার পরিবর্তে গ্রুপগুলোর উপর ভিত্তি করে কোডটি নেভিগেট করতে পারে, যা তাদের জন্য প্রাসঙ্গিক সংজ্ঞাগুলো খুঁজে পাওয়া সহজ করে তোলে। এই কোডে নতুন কার্যকারিতা যোগ করা প্রোগ্রামাররা জানবে যে প্রোগ্রামটিকে সংগঠিত রাখতে কোথায় কোড রাখতে হবে।

আগে, আমরা উল্লেখ করেছি যে _src/main.rs_ এবং _src/lib.rs_-কে ক্রেট রুট বলা হয়। তাদের নামের কারণ হল এই দুটি ফাইলের যেকোনো একটির কনটেন্ট ক্রেটের মডিউল কাঠামোর রুটে `crate` নামক একটি মডিউল তৈরি করে, যা *মডিউল ট্রি (module tree)* নামে পরিচিত।

Listing 7-2 Listing 7-1-এর কাঠামোর জন্য মডিউল ট্রি দেখায়।

<Listing number="7-2" caption="Listing 7-1-এর কোডের জন্য মডিউল ট্রি">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

এই ট্রিটি দেখায় কিভাবে কিছু মডিউল অন্য মডিউলের ভিতরে নেস্ট করা হয়; উদাহরণস্বরূপ, `hosting` `front_of_house`-এর ভিতরে নেস্ট করা আছে। ট্রিটি আরও দেখায় যে কিছু মডিউল হল *সিবলিং (siblings)*, মানে সেগুলো একই মডিউলে সংজ্ঞায়িত; `hosting` এবং `serving` হল `front_of_house`-এর মধ্যে সংজ্ঞায়িত সিবলিং। যদি মডিউল A মডিউল B-এর মধ্যে থাকে, তাহলে আমরা বলি যে মডিউল A হল মডিউল B-এর *চাইল্ড (child)* এবং মডিউল B হল মডিউল A-এর *প্যারেন্ট (parent)*। লক্ষ্য করুন যে সম্পূর্ণ মডিউল ট্রিটি `crate` নামক অন্তর্নিহিত মডিউলের অধীনে রয়েছে।

মডিউল ট্রি আপনাকে আপনার কম্পিউটারের ফাইল সিস্টেমের ডিরেক্টরি ট্রির কথা মনে করিয়ে দিতে পারে; এটি একটি খুব উপযুক্ত তুলনা! ফাইল সিস্টেমের ডিরেক্টরিগুলোর মতোই, আপনি আপনার কোড সংগঠিত করতে মডিউল ব্যবহার করেন। এবং একটি ডিরেক্টরির ফাইলগুলোর মতোই, আমাদের মডিউলগুলো খুঁজে বের করার একটি উপায় দরকার।
