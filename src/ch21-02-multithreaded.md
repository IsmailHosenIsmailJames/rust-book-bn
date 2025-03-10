## আমাদের সিঙ্গেল-থ্রেডেড সার্ভারকে মাল্টিথ্রেডেড সার্ভারে পরিণত করা

এখন, সার্ভার প্রতিটি অনুরোধ একে একে প্রক্রিয়া করবে, মানে এটি প্রথমটির প্রসেসিং শেষ না হওয়া পর্যন্ত দ্বিতীয় কানেকশনটি প্রক্রিয়া করবে না। যদি সার্ভার আরও বেশি সংখ্যক অনুরোধ গ্রহণ করে, তাহলে এই সিরিয়াল এক্সিকিউশন ক্রমশ কম অপ্টিমাল হবে। যদি সার্ভার এমন একটি অনুরোধ পায় যা প্রক্রিয়া করতে বেশি সময় নেয়, তাহলে পরবর্তী অনুরোধগুলিকে দীর্ঘ অনুরোধটি শেষ না হওয়া পর্যন্ত অপেক্ষা করতে হবে, এমনকি যদি নতুন অনুরোধগুলি দ্রুত প্রক্রিয়া করা যায় তাহলেও। আমাদের এটি ঠিক করতে হবে, কিন্তু প্রথমে আমরা সমস্যাটি কার্যকর অবস্থায় দেখব।

### বর্তমান সার্ভার ইমপ্লিমেন্টেশনে একটি স্লো রিকোয়েস্ট সিমুলেট করা

আমরা দেখব কিভাবে একটি স্লো-প্রসেসিং অনুরোধ আমাদের বর্তমান সার্ভার ইমপ্লিমেন্টেশনে করা অন্যান্য অনুরোধগুলিকে প্রভাবিত করতে পারে। লিস্টিং ২১-১০ _/sleep_-এ একটি অনুরোধ হ্যান্ডেল করা ইমপ্লিমেন্ট করে, যেখানে একটি সিমুলেটেড স্লো রেসপন্স থাকবে যা সার্ভারকে প্রতিক্রিয়া জানানোর আগে পাঁচ সেকেন্ডের জন্য স্লিপিং মোডে রাখবে।

<Listing number="21-10" file-name="src/main.rs" caption="৫ সেকেন্ডের জন্য স্লিপিং মোডে রেখে একটি স্লো অনুরোধ সিমুলেট করা">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

আমরা `if` থেকে `match`-এ পরিবর্তন করেছি, কারণ এখন আমাদের তিনটি কেস রয়েছে। স্ট্রিং লিটারেল ভ্যালুগুলির সাথে প্যাটার্ন ম্যাচ করার জন্য আমাদের `request_line`-এর একটি স্লাইসের উপর স্পষ্টভাবে ম্যাচ করতে হবে; `match` স্বয়ংক্রিয়ভাবে রেফারেন্সিং এবং ডিরেফারেন্সিং করে না, যেমনটি ইকুয়ালিটি মেথড করে।

প্রথম আর্মটি লিস্টিং ২১-৯ এর `if` ব্লকের মতোই। দ্বিতীয় আর্মটি _/sleep_-এর অনুরোধের সাথে মেলে। যখন সেই অনুরোধটি গৃহীত হয়, সার্ভার সফল HTML পেজটি রেন্ডার করার আগে পাঁচ সেকেন্ডের জন্য স্লিপ করবে। তৃতীয় আর্মটি লিস্টিং ২১-৯ এর `else` ব্লকের মতোই।

আপনি দেখতে পাচ্ছেন কিভাবে আমাদের সার্ভারটি প্রাথমিক: প্রকৃত লাইব্রেরিগুলি একাধিক অনুরোধের স্বীকৃতি আরও কম ভারবোস উপায়ে হ্যান্ডেল করবে!

`cargo run` ব্যবহার করে সার্ভার শুরু করুন। তারপর দুটি ব্রাউজার উইন্ডো খুলুন: একটি _http://127.0.0.1:7878/_ এর জন্য এবং অন্যটি _http://127.0.0.1:7878/sleep_ এর জন্য। আপনি যদি আগের মতো কয়েকবার _/_ URI-তে প্রবেশ করেন, তাহলে আপনি দেখতে পাবেন এটি দ্রুত প্রতিক্রিয়া জানাচ্ছে। কিন্তু আপনি যদি _/sleep_-এ প্রবেশ করেন এবং তারপর _/_ লোড করেন, তাহলে আপনি দেখতে পাবেন যে _/_ লোড হওয়ার আগে `sleep` তার পুরো পাঁচ সেকেন্ডের জন্য ঘুমিয়েছে।

একটি স্লো অনুরোধের পিছনে অন্যান্য অনুরোধগুলি আটকে যাওয়া এড়াতে আমরা একাধিক কৌশল ব্যবহার করতে পারি, যার মধ্যে একটি হল async ব্যবহার করা যেমনটি আমরা ১৭ অধ্যায়ে করেছি; আমরা যেটি ইমপ্লিমেন্ট করব সেটি হল একটি থ্রেড পুল।

### থ্রেড পুল দিয়ে থ্রুপুট উন্নত করা

একটি _থ্রেড পুল_ হল স্পন করা থ্রেডগুলির একটি গ্রুপ যা অপেক্ষা করছে এবং একটি কাজ হ্যান্ডেল করার জন্য প্রস্তুত। যখন প্রোগ্রামটি একটি নতুন কাজ পায়, তখন এটি পুলের একটি থ্রেডকে কাজটি অর্পণ করে এবং সেই থ্রেডটি কাজটি প্রক্রিয়া করবে। পুলের অবশিষ্ট থ্রেডগুলি অন্য কোনো কাজ হ্যান্ডেল করার জন্য উপলব্ধ থাকে, যখন প্রথম থ্রেডটি প্রসেসিং করছে। যখন প্রথম থ্রেডটি তার কাজ প্রক্রিয়া করা শেষ করে, তখন সেটি অলস থ্রেডগুলির পুলে ফিরে আসে, একটি নতুন কাজ হ্যান্ডেল করার জন্য প্রস্তুত। একটি থ্রেড পুল আপনাকে কানেকশনগুলি কনকারেন্টলি প্রক্রিয়া করার অনুমতি দেয়, আপনার সার্ভারের থ্রুপুট বৃদ্ধি করে।

DoS অ্যাটাক থেকে আমাদের রক্ষা করার জন্য আমরা পুলের থ্রেডের সংখ্যা একটি ছোট সংখ্যায় সীমাবদ্ধ করব; যদি আমাদের প্রোগ্রাম প্রতিটি অনুরোধ আসার সাথে সাথে একটি নতুন থ্রেড তৈরি করত, তাহলে কেউ আমাদের সার্ভারে ১০ মিলিয়ন অনুরোধ করলে আমাদের সার্ভারের সমস্ত সংস্থান ব্যবহার করে এবং অনুরোধগুলির প্রসেসিং বন্ধ করে দিয়ে বিপর্যয় সৃষ্টি করতে পারত।

আনলিমিটেড থ্রেড স্পন করার পরিবর্তে, আমাদের পুলে একটি নির্দিষ্ট সংখ্যক থ্রেড অপেক্ষা করবে। আসা অনুরোধগুলি প্রসেসিংয়ের জন্য পুলে পাঠানো হয়। পুলটি আগত অনুরোধগুলির একটি কিউ বজায় রাখবে। পুলের প্রতিটি থ্রেড এই কিউ থেকে একটি অনুরোধ তুলে নেবে, অনুরোধটি হ্যান্ডেল করবে এবং তারপর কিউ-কে অন্য একটি অনুরোধের জন্য জিজ্ঞাসা করবে। এই ডিজাইনের সাহায্যে, আমরা *`N`* পর্যন্ত অনুরোধ কনকারেন্টলি প্রক্রিয়া করতে পারি, যেখানে *`N`* হল থ্রেডের সংখ্যা। যদি প্রতিটি থ্রেড একটি দীর্ঘ-চলমান অনুরোধের প্রতিক্রিয়া জানায়, তাহলে পরবর্তী অনুরোধগুলি এখনও কিউতে ব্যাক আপ করতে পারে, কিন্তু আমরা সেই পয়েন্টে পৌঁছানোর আগে আমরা যে দীর্ঘ-চলমান অনুরোধগুলি হ্যান্ডেল করতে পারি তার সংখ্যা বাড়িয়েছি।

এই কৌশলটি একটি ওয়েব সার্ভারের থ্রুপুট উন্নত করার অনেকগুলি উপায়ের মধ্যে একটি। আপনি যে অন্যান্য অপশনগুলি দেখতে পারেন সেগুলি হল ফর্ক/জয়েন মডেল, সিঙ্গেল-থ্রেডেড অ্যাসিঙ্ক্রোনাস I/O মডেল এবং মাল্টি-থ্রেডেড অ্যাসিঙ্ক্রোনাস I/O মডেল। আপনি যদি এই বিষয়ে আগ্রহী হন, তাহলে আপনি অন্যান্য সমাধান সম্পর্কে আরও পড়তে পারেন এবং সেগুলি ইমপ্লিমেন্ট করার চেষ্টা করতে পারেন; Rust-এর মতো একটি নিম্ন-স্তরের ভাষার সাথে, এই সমস্ত অপশন সম্ভব।

আমরা একটি থ্রেড পুল ইমপ্লিমেন্ট করা শুরু করার আগে, আসুন পুলটি ব্যবহার করা কেমন হওয়া উচিত সে সম্পর্কে কথা বলি। যখন আপনি কোড ডিজাইন করার চেষ্টা করছেন, তখন ক্লায়েন্ট ইন্টারফেসটি প্রথমে লেখা আপনার ডিজাইনকে গাইড করতে সাহায্য করতে পারে। কোডের API এমনভাবে লিখুন যাতে আপনি যেভাবে এটি কল করতে চান সেইভাবে এটি গঠিত হয়; তারপর কার্যকারিতা ইমপ্লিমেন্ট করে এবং তারপর পাবলিক API ডিজাইন করার পরিবর্তে সেই কাঠামোর মধ্যে কার্যকারিতা ইমপ্লিমেন্ট করুন।

১২ অধ্যায়ের প্রোজেক্টে আমরা যেভাবে টেস্ট-চালিত ডেভেলপমেন্ট ব্যবহার করেছি, আমরা এখানে কম্পাইলার-চালিত ডেভেলপমেন্ট ব্যবহার করব। আমরা সেই কোডটি লিখব যা আমাদের ইচ্ছামতো ফাংশনগুলিকে কল করে এবং তারপর কম্পাইলার থেকে এররগুলি দেখে আমরা নির্ধারণ করব যে কোডটি কাজ করার জন্য আমাদের পরবর্তীতে কী পরিবর্তন করা উচিত। তবে, আমরা এটি করার আগে, আমরা সেই কৌশলটি অন্বেষণ করব যা আমরা শুরুর পয়েন্ট হিসাবে ব্যবহার করব না।

<!-- পুরানো শিরোনাম। অপসারণ করবেন না বা লিঙ্কগুলি ভেঙে যেতে পারে। -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### প্রতিটি অনুরোধের জন্য একটি থ্রেড স্পন করা

প্রথমে, আসুন দেখি আমাদের কোডটি কেমন হতে পারে যদি এটি প্রতিটি কানেকশনের জন্য একটি নতুন থ্রেড তৈরি করে। আগেই যেমন উল্লেখ করা হয়েছে, এটি আমাদের চূড়ান্ত প্ল্যান নয় কারণ আনলিমিটেড সংখ্যক থ্রেড স্পন করার সমস্যা রয়েছে, তবে প্রথমে একটি কার্যকরী মাল্টিথ্রেডেড সার্ভার পাওয়ার জন্য এটি একটি শুরুর পয়েন্ট। তারপর আমরা থ্রেড পুলটিকে একটি উন্নতি হিসাবে যুক্ত করব এবং দুটি সমাধানের মধ্যে পার্থক্য করা সহজ হবে। লিস্টিং ২১-১১ `for` লুপের মধ্যে প্রতিটি স্ট্রিম হ্যান্ডেল করার জন্য একটি নতুন থ্রেড স্পন করতে `main`-এ যে পরিবর্তনগুলি করতে হবে তা দেখায়।

<Listing number="21-11" file-name="src/main.rs" caption="প্রতিটি স্ট্রিমের জন্য একটি নতুন থ্রেড স্পন করা">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

আপনি যেমন ১৬ অধ্যায়ে শিখেছেন, `thread::spawn` একটি নতুন থ্রেড তৈরি করবে এবং তারপর নতুন থ্রেডে ক্লোজারের কোডটি চালাবে। আপনি যদি এই কোডটি চালান এবং আপনার ব্রাউজারে _/sleep_ লোড করেন, তারপর আরও দুটি ব্রাউজার ট্যাবে _/_ লোড করেন, তাহলে আপনি দেখবেন যে _/_ এর অনুরোধগুলিকে _/sleep_ শেষ হওয়ার জন্য অপেক্ষা করতে হবে না। তবে, যেমনটি আমরা উল্লেখ করেছি, এটি অবশেষে সিস্টেমটিকে অভিভূত করবে কারণ আপনি কোনো সীমা ছাড়াই নতুন থ্রেড তৈরি করবেন।

আপনি হয়তো ১৭ অধ্যায় থেকেও মনে করতে পারেন যে এটি ঠিক সেই ধরনের পরিস্থিতি যেখানে async এবং await সত্যিই উজ্জ্বল! থ্রেড পুল তৈরি করার সময় এটি মনে রাখবেন এবং চিন্তা করুন যে async-এর সাথে জিনিসগুলি কীভাবে আলাদা বা একই রকম দেখাবে।

<!-- পুরানো শিরোনাম। অপসারণ করবেন না বা লিঙ্কগুলি ভেঙে যেতে পারে। -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### সীমিত সংখ্যক থ্রেড তৈরি করা

আমরা চাই আমাদের থ্রেড পুলটি একই রকম, পরিচিত উপায়ে কাজ করুক যাতে থ্রেড থেকে থ্রেড পুলে পরিবর্তন করার জন্য আমাদের API ব্যবহার করা কোডে বড় পরিবর্তনের প্রয়োজন না হয়। লিস্টিং ২১-১২ একটি `ThreadPool` স্ট্রাক্টের জন্য অনুমানমূলক ইন্টারফেস দেখায় যা আমরা `thread::spawn`-এর পরিবর্তে ব্যবহার করতে চাই।

<Listing number="21-12" file-name="src/main.rs" caption="আমাদের আদর্শ `ThreadPool` ইন্টারফেস">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

আমরা `ThreadPool::new` ব্যবহার করে একটি কনফিগারযোগ্য সংখ্যক থ্রেড সহ একটি নতুন থ্রেড পুল তৈরি করি, এই ক্ষেত্রে চারটি। তারপর, `for` লুপে, `pool.execute`-এর `thread::spawn`-এর মতোই একটি ইন্টারফেস রয়েছে, যেখানে এটি পুলের প্রতিটি স্ট্রিমের জন্য চালানো উচিত এমন একটি ক্লোজার নেয়। আমাদের `pool.execute` ইমপ্লিমেন্ট করতে হবে যাতে এটি ক্লোজারটি নেয় এবং চালানোর জন্য পুলের একটি থ্রেডকে দেয়। এই কোডটি এখনও কম্পাইল হবে না, কিন্তু আমরা চেষ্টা করব যাতে কম্পাইলার আমাদের এটি কীভাবে ঠিক করতে হয় সে সম্পর্কে গাইড করতে পারে।

<!-- পুরানো শিরোনাম। অপসারণ করবেন না বা লিঙ্কগুলি ভেঙে যেতে পারে। -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### কম্পাইলার ড্রাইভেন ডেভেলপমেন্ট ব্যবহার করে `ThreadPool` তৈরি করা

লিস্টিং 21-12-এ _src/main.rs_-এ পরিবর্তন করুন, এবং তারপর `cargo check` থেকে কম্পাইলার এররগুলিকে আমাদের ডেভেলপমেন্ট ড্রাইভ করার জন্য ব্যবহার করি। আমরা যে প্রথম এররটি পাই তা এখানে:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

দারুণ! এই এররটি আমাদের বলে যে আমাদের একটি `ThreadPool` টাইপ বা মডিউল প্রয়োজন, তাই আমরা এখন একটি তৈরি করব। আমাদের `ThreadPool` ইমপ্লিমেন্টেশন আমাদের ওয়েব সার্ভার যে ধরনের কাজ করছে তার থেকে স্বাধীন হবে। তাই আসুন `hello` ক্রেটটিকে একটি বাইনারি ক্রেট থেকে একটি লাইব্রেরি ক্রেটে পরিবর্তন করি যাতে আমাদের `ThreadPool` ইমপ্লিমেন্টেশন থাকে। আমরা একটি লাইব্রেরি ক্রেটে পরিবর্তন করার পরে, আমরা ওয়েব অনুরোধগুলি পরিবেশন করার জন্য নয়, থ্রেড পুল ব্যবহার করে আমরা যে কোনও কাজ করতে চাই তার জন্যও পৃথক থ্রেড পুল লাইব্রেরি ব্যবহার করতে পারি।

একটি _src/lib.rs_ ফাইল তৈরি করুন যাতে নিম্নলিখিতগুলি রয়েছে, যা একটি `ThreadPool` স্ট্রাক্টের সবচেয়ে সহজ সংজ্ঞা যা আমরা আপাতত রাখতে পারি:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>

তারপর লাইব্রেরি ক্রেট থেকে `ThreadPool` কে স্কোপে আনতে _src/main.rs_ ফাইলের শীর্ষে নিম্নলিখিত কোড যোগ করে _main.rs_ ফাইলটি এডিট করুন:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

এই কোডটি এখনও কাজ করবে না, তবে আসুন এটিকে আবার চেক করি যাতে আমাদের পরবর্তী এররটি পাওয়া যায়, যেটিকে আমাদের অ্যাড্রেস করতে হবে:

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

এই এররটি ইঙ্গিত দেয় যে এরপরে আমাদের `ThreadPool`-এর জন্য `new` নামে একটি অ্যাসোসিয়েটেড ফাংশন তৈরি করতে হবে। আমরা জানি যে `new`-এর একটি প্যারামিটার থাকতে হবে যা `4` কে আর্গুমেন্ট হিসাবে গ্রহণ করতে পারে এবং একটি `ThreadPool` ইন্সট্যান্স রিটার্ন করা উচিত। আসুন সবচেয়ে সহজ `new` ফাংশনটি ইমপ্লিমেন্ট করি যার এই বৈশিষ্ট্যগুলি থাকবে:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

আমরা `size` প্যারামিটারের টাইপ হিসাবে `usize` বেছে নিয়েছি কারণ আমরা জানি যে নেগেটিভ সংখ্যক থ্রেডের কোনো অর্থ নেই। আমরা জানি যে আমরা এই `4` কে থ্রেডগুলির একটি কালেকশনের এলিমেন্টের সংখ্যা হিসাবে ব্যবহার করব, যেটি `usize` টাইপের কাজ, যেমনটি তৃতীয় অধ্যায়ের [“পূর্ণসংখ্যার প্রকারভেদ”][integer-types]<!-- ignore --> এ আলোচনা করা হয়েছে।

আসুন আবার কোডটি চেক করি:

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

এখন এররটি ঘটেছে কারণ `ThreadPool`-এ আমাদের কোনো `execute` মেথড নেই। [“সীমাবদ্ধ সংখ্যক থ্রেড তৈরি করা”](#creating-a-finite-number-of-threads)<!-- ignore --> থেকে মনে করুন যে আমরা সিদ্ধান্ত নিয়েছি আমাদের থ্রেড পুলের `thread::spawn`-এর মতোই একটি ইন্টারফেস থাকা উচিত। এছাড়াও, আমরা `execute` ফাংশনটি ইমপ্লিমেন্ট করব যাতে এটি যে ক্লোজারটি পায় সেটি নেয় এবং চালানোর জন্য পুলের একটি অলস থ্রেডকে দেয়।

আমরা `ThreadPool`-এ `execute` মেথডটিকে সংজ্ঞায়িত করব যাতে এটি একটি প্যারামিটার হিসাবে একটি ক্লোজার নেয়। ত্রয়োদশ অধ্যায়ের [“ক্লোজার থেকে ক্যাপচার করা ভ্যালুগুলিকে সরানো এবং `Fn` ট্রেইট”][fn-traits]<!-- ignore --> থেকে মনে করুন যে আমরা তিনটি ভিন্ন ট্রেইট সহ ক্লোজারগুলিকে প্যারামিটার হিসাবে নিতে পারি: `Fn`, `FnMut` এবং `FnOnce`। আমাদের এখানে কোন ধরনের ক্লোজার ব্যবহার করতে হবে তা নির্ধারণ করতে হবে। আমরা জানি যে আমরা শেষ পর্যন্ত স্ট্যান্ডার্ড লাইব্রেরির `thread::spawn` ইমপ্লিমেন্টেশনের মতোই কিছু করব, তাই আমরা দেখতে পারি `thread::spawn`-এর সিগনেচারের তার প্যারামিটারের উপর কী কী বাউন্ড রয়েছে। ডকুমেন্টেশন আমাদের নিম্নলিখিতগুলি দেখায়:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`F` টাইপ প্যারামিটারটি হল যেটি নিয়ে আমরা এখানে চিন্তিত; `T` টাইপ প্যারামিটারটি রিটার্ন ভ্যালুর সাথে সম্পর্কিত এবং আমরা সেটি নিয়ে চিন্তিত নই। আমরা দেখতে পাচ্ছি যে `spawn` `FnOnce` কে `F`-এর উপর ট্রেইট বাউন্ড হিসাবে ব্যবহার করে। এটি সম্ভবত আমরাও চাই, কারণ আমরা অবশেষে `execute`-এ যে আর্গুমেন্টটি পাই সেটি `spawn`-এ পাস করব। আমরা আরও নিশ্চিত হতে পারি যে `FnOnce` হল সেই ট্রেইট যা আমরা ব্যবহার করতে চাই কারণ একটি অনুরোধ চালানোর থ্রেডটি শুধুমাত্র সেই অনুরোধের ক্লোজারটি একবার চালাবে, যা `FnOnce`-এর `Once`-এর সাথে মেলে।

`F` টাইপ প্যারামিটারের `Send` ট্রেইট বাউন্ড এবং `'static` লাইফটাইম বাউন্ডও রয়েছে, যা আমাদের পরিস্থিতিতে দরকারী: আমাদের ক্লোজারটিকে একটি থ্রেড থেকে অন্য থ্রেডে স্থানান্তর করার জন্য `Send` প্রয়োজন এবং `'static` কারণ আমরা জানি না থ্রেডটি এক্সিকিউট হতে কতক্ষণ সময় নেবে। আসুন `ThreadPool`-এ একটি `execute` মেথড তৈরি করি যা এই বাউন্ডগুলি সহ `F` টাইপের একটি জেনেরিক প্যারামিটার নেবে:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

আমরা এখনও `FnOnce`-এর পরে `()` ব্যবহার করি কারণ এই `FnOnce` এমন একটি ক্লোজারকে উপস্থাপন করে যা কোনো প্যারামিটার নেয় না এবং ইউনিট টাইপ `()` রিটার্ন করে। ফাংশন সংজ্ঞার মতোই, রিটার্ন টাইপটি সিগনেচার থেকে বাদ দেওয়া যেতে পারে, কিন্তু আমাদের কোনো প্যারামিটার না থাকলেও, আমাদের এখনও প্যারেন্থেসিস প্রয়োজন।

আবারও, এটি `execute` মেথডের সবচেয়ে সহজ ইমপ্লিমেন্টেশন: এটি কিছুই করে না, তবে আমরা কেবল আমাদের কোড কম্পাইল করার চেষ্টা করছি। আসুন এটি আবার চেক করি:

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

এটি কম্পাইল হয়! তবে মনে রাখবেন যে আপনি যদি `cargo run` চেষ্টা করেন এবং ব্রাউজারে একটি অনুরোধ করেন, তাহলে আপনি ব্রাউজারে সেই এররগুলি দেখতে পাবেন যা আমরা এই অধ্যায়ের শুরুতে দেখেছিলাম। আমাদের লাইব্রেরি এখনও `execute`-এ পাস করা ক্লোজারটিকে কল করছে না!

> দ্রষ্টব্য: কঠোর কম্পাইলার সহ ভাষাগুলি, যেমন হাস্কেল (Haskell) এবং Rust সম্পর্কে আপনি যে উক্তিটি শুনতে পারেন তা হল "যদি কোড কম্পাইল হয়, তাহলে এটি কাজ করে।" কিন্তু এই উক্তিটি সর্বজনীনভাবে সত্য নয়। আমাদের প্রোজেক্ট কম্পাইল হয়, কিন্তু এটি একেবারে কিছুই করে না! যদি আমরা একটি বাস্তব, সম্পূর্ণ প্রোজেক্ট তৈরি করতাম, তাহলে কোডটি কম্পাইল হয় _এবং_ আমাদের ইচ্ছামতো আচরণ করে কিনা তা পরীক্ষা করার জন্য ইউনিট টেস্ট লেখা শুরু করার জন্য এটি একটি ভাল সময় হবে।

বিবেচনা করুন: আমরা যদি ক্লোজারের পরিবর্তে একটি _ফিউচার_ এক্সিকিউট করতাম তবে এখানে কী আলাদা হত?

#### `new`-এ থ্রেডের সংখ্যা ভ্যালিডেট করা

আমরা `new` এবং `execute`-এর প্যারামিটারগুলির সাথে কিছুই করছি না। আসুন এই ফাংশনগুলির বডিগুলিকে আমাদের ইচ্ছামতো আচরণ দিয়ে ইমপ্লিমেন্ট করি। শুরু করতে, আসুন `new` সম্পর্কে চিন্তা করি। এর আগে আমরা `size` প্যারামিটারের জন্য একটি আনসাইনড টাইপ বেছে নিয়েছিলাম কারণ নেগেটিভ সংখ্যক থ্রেড সহ একটি পুলের কোনো মানে হয় না। যাইহোক, শূন্য থ্রেড সহ একটি পুলেরও কোনো মানে হয় না, তবুও শূন্য একটি সম্পূর্ণ বৈধ `usize`। আমরা কোড যোগ করব যাতে `size` শূন্যের চেয়ে বড় কিনা তা পরীক্ষা করার জন্য আমরা একটি `ThreadPool` ইন্সট্যান্স রিটার্ন করার আগে এবং `assert!` ম্যাক্রো ব্যবহার করে শূন্য পেলে প্রোগ্রামটি প্যানিক করে, যেমনটি লিস্টিং ২১-১৩-তে দেখানো হয়েছে।

<Listing number="21-13" file-name="src/lib.rs" caption="`size` শূন্য হলে প্যানিক করার জন্য `ThreadPool::new` ইমপ্লিমেন্ট করা">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>
আমরা ডক কমেন্ট সহ আমাদের `ThreadPool`-এর জন্য কিছু ডকুমেন্টেশনও যুক্ত করেছি। মনে রাখবেন যে আমরা ১৪ অধ্যায়ে আলোচনা করা, ভাল ডকুমেন্টেশন অনুশীলনগুলি অনুসরণ করেছি, যেখানে একটি বিভাগ যুক্ত করা হয়েছে যা আমাদের ফাংশন কোন পরিস্থিতিতে প্যানিক করতে পারে তা উল্লেখ করে। `cargo doc --open` চালানোর চেষ্টা করুন এবং `new`-এর জন্য জেনারেট করা ডক্সগুলি দেখতে `ThreadPool` স্ট্রাক্টে ক্লিক করুন!

আমরা এখানে যেভাবে `assert!` ম্যাক্রো যোগ করেছি, তার পরিবর্তে, আমরা `new` কে `build`-এ পরিবর্তন করতে পারতাম এবং একটি `Result` রিটার্ন করতে পারতাম, যেমনটি আমরা I/O প্রোজেক্টে লিস্টিং ১২-৯-এ `Config::build`-এর সাথে করেছি। কিন্তু আমরা এই ক্ষেত্রে সিদ্ধান্ত নিয়েছি যে কোনো থ্রেড ছাড়াই একটি থ্রেড পুল তৈরি করার চেষ্টা একটি পুনরুদ্ধার অযোগ্য এরর হওয়া উচিত। আপনি যদি উচ্চাকাঙ্ক্ষী হন, তাহলে `new` ফাংশনের সাথে তুলনা করার জন্য নিম্নলিখিত সিগনেচার সহ `build` নামে একটি ফাংশন লেখার চেষ্টা করুন:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### থ্রেড স্টোর করার জন্য জায়গা তৈরি করা

এখন যেহেতু পুলটিতে স্টোর করার জন্য আমাদের কাছে বৈধ সংখ্যক থ্রেড আছে, তাই আমরা সেই থ্রেডগুলি তৈরি করতে পারি এবং স্ট্রাক্টটি রিটার্ন করার আগে `ThreadPool` স্ট্রাক্টে সেগুলি স্টোর করতে পারি। কিন্তু আমরা কীভাবে একটি থ্রেড "স্টোর" করব? আসুন `thread::spawn`-এর সিগনেচারটি আবার দেখি:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`spawn` ফাংশনটি একটি `JoinHandle<T>` রিটার্ন করে, যেখানে `T` হল সেই টাইপ যা ক্লোজারটি রিটার্ন করে। আসুন `JoinHandle` ব্যবহার করার চেষ্টা করি এবং দেখি কী হয়। আমাদের ক্ষেত্রে, থ্রেড পুলে আমরা যে ক্লোজারগুলি পাস করছি সেগুলি কানেকশন হ্যান্ডেল করবে এবং কিছুই রিটার্ন করবে না, তাই `T` হবে ইউনিট টাইপ `()`।

লিস্টিং ২১-১৪-এর কোডটি কম্পাইল হবে কিন্তু এখনও কোনো থ্রেড তৈরি করে না। আমরা `ThreadPool`-এর সংজ্ঞা পরিবর্তন করে `thread::JoinHandle<()>` ইন্সট্যান্সের একটি ভেক্টর রেখেছি, `size` ক্যাপাসিটি সহ ভেক্টরটিকে ইনিশিয়ালাইজ করেছি, একটি `for` লুপ সেট আপ করেছি যা থ্রেড তৈরি করার জন্য কিছু কোড চালাবে এবং সেগুলি ধারণকারী একটি `ThreadPool` ইন্সট্যান্স রিটার্ন করেছি।

<Listing number="21-14" file-name="src/lib.rs" caption="থ্রেড ধারণ করার জন্য `ThreadPool`-এর জন্য একটি ভেক্টর তৈরি করা">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

আমরা লাইব্রেরি ক্রেটে `std::thread` কে স্কোপে এনেছি কারণ আমরা `ThreadPool`-এর ভেক্টরের আইটেমগুলির টাইপ হিসাবে `thread::JoinHandle` ব্যবহার করছি।

একবার একটি বৈধ সাইজ পাওয়া গেলে, আমাদের `ThreadPool` একটি নতুন ভেক্টর তৈরি করে যা `size` সংখ্যক আইটেম ধারণ করতে পারে। `with_capacity` ফাংশনটি `Vec::new`-এর মতোই কাজ করে, কিন্তু একটি গুরুত্বপূর্ণ পার্থক্য সহ: এটি ভেক্টরে আগে থেকে জায়গা বরাদ্দ করে। যেহেতু আমরা জানি যে আমাদের ভেক্টরে `size` সংখ্যক এলিমেন্ট স্টোর করতে হবে, তাই আগে থেকে এই অ্যালোকেশন করা `Vec::new` ব্যবহার করার চেয়ে কিছুটা বেশি কার্যকর, যা এলিমেন্ট যোগ করার সাথে সাথে নিজে থেকেই রিসাইজ হয়।

আপনি যখন আবার `cargo check` চালাবেন, তখন এটি সফল হওয়া উচিত।

#### `ThreadPool` থেকে একটি থ্রেডে কোড পাঠানোর জন্য দায়িত্বশীল একটি `Worker` স্ট্রাক্ট

আমরা লিস্টিং ২১-১৪-তে থ্রেড তৈরির বিষয়ে `for` লুপে একটি কমেন্ট রেখেছিলাম। এখানে, আমরা দেখব কিভাবে আমরা আসলে থ্রেড তৈরি করি। স্ট্যান্ডার্ড লাইব্রেরি থ্রেড তৈরি করার উপায় হিসাবে `thread::spawn` সরবরাহ করে এবং `thread::spawn` আশা করে যে থ্রেডটি তৈরি হওয়ার সাথে সাথেই চালানোর জন্য কিছু কোড পাবে। যাইহোক, আমাদের ক্ষেত্রে, আমরা থ্রেডগুলি তৈরি করতে চাই এবং তাদের _অপেক্ষা_ করাতে চাই সেই কোডের জন্য যা আমরা পরে পাঠাব। থ্রেডের স্ট্যান্ডার্ড লাইব্রেরির ইমপ্লিমেন্টেশনে এটি করার কোনো উপায় নেই; আমাদের এটি ম্যানুয়ালি ইমপ্লিমেন্ট করতে হবে।

আমরা `ThreadPool` এবং থ্রেডগুলির মধ্যে একটি নতুন ডেটা স্ট্রাকচার উপস্থাপন করে এই আচরণটি ইমপ্লিমেন্ট করব যা এই নতুন আচরণটি পরিচালনা করবে। আমরা এই ডেটা স্ট্রাকচারটিকে _Worker_ বলব, যা পুলিং ইমপ্লিমেন্টেশনে একটি সাধারণ শব্দ। `Worker` সেই কোডটি তুলে নেয় যা চালানো দরকার এবং Worker এর থ্রেডে কোডটি চালায়।

একটি রেস্তোরাঁর রান্নাঘরে কাজ করা লোকেদের কথা ভাবুন: কর্মীরা গ্রাহকদের কাছ থেকে অর্ডার আসার জন্য অপেক্ষা করে এবং তারপর সেই অর্ডারগুলি নেওয়া এবং সেগুলি পূরণ করার জন্য তারা দায়ী থাকে।

থ্রেড পুলে `JoinHandle<()>` ইন্সট্যান্সের একটি ভেক্টর সংরক্ষণ করার পরিবর্তে, আমরা `Worker` স্ট্রাক্টের ইন্সট্যান্স সংরক্ষণ করব। প্রতিটি `Worker` একটি একক `JoinHandle<()>` ইন্সট্যান্স সংরক্ষণ করবে। তারপর আমরা `Worker`-এ একটি মেথড ইমপ্লিমেন্ট করব যা চালানোর জন্য কোডের একটি ক্লোজার নেবে এবং এক্সিকিউশনের জন্য ইতিমধ্যে চলমান থ্রেডে পাঠাবে। লগিং বা ডিবাগিং করার সময় আমরা পুলের বিভিন্ন `Worker` ইন্সট্যান্সের মধ্যে পার্থক্য করার জন্য প্রতিটি `Worker`-কে একটি `id` দেব।

এখানে নতুন প্রক্রিয়াটি রয়েছে যা ঘটবে যখন আমরা একটি `ThreadPool` তৈরি করব। আমরা এইভাবে `Worker` সেট আপ করার পরে থ্রেডে ক্লোজার পাঠানোর কোডটি ইমপ্লিমেন্ট করব:

1.  একটি `Worker` স্ট্রাক্ট সংজ্ঞায়িত করুন যা একটি `id` এবং একটি `JoinHandle<()>` ধারণ করে।
2.  `Worker` ইন্সট্যান্সের একটি ভেক্টর ধারণ করতে `ThreadPool` পরিবর্তন করুন।
3.  একটি `Worker::new` ফাংশন সংজ্ঞায়িত করুন যা একটি `id` নম্বর নেয় এবং একটি `Worker` ইন্সট্যান্স রিটার্ন করে যা `id` এবং একটি খালি ক্লোজার দিয়ে স্পন করা থ্রেড ধারণ করে।
4.  `ThreadPool::new`-তে, একটি `id` জেনারেট করতে `for` লুপ কাউন্টার ব্যবহার করুন, সেই `id` দিয়ে একটি নতুন `Worker` তৈরি করুন এবং ভেক্টরে worker-কে স্টোর করুন।

আপনি যদি চ্যালেঞ্জের জন্য প্রস্তুত হন, তাহলে লিস্টিং ২১-১৫-এর কোড দেখার আগে নিজে থেকে এই পরিবর্তনগুলি ইমপ্লিমেন্ট করার চেষ্টা করুন।

প্রস্তুত? পূর্ববর্তী পরিবর্তনগুলি করার একটি উপায় সহ লিস্টিং ২১-১৫ এখানে রয়েছে।

<Listing number="21-15" file-name="src/lib.rs" caption="সরাসরি থ্রেড ধারণ করার পরিবর্তে `Worker` ইন্সট্যান্স ধারণ করতে `ThreadPool` পরিবর্তন করা">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

আমরা `ThreadPool`-এ ফিল্ডের নাম `threads` থেকে `workers`-এ পরিবর্তন করেছি কারণ এটি এখন `JoinHandle<()>` ইন্সট্যান্সের পরিবর্তে `Worker` ইন্সট্যান্স ধারণ করছে। আমরা `Worker::new`-তে আর্গুমেন্ট হিসাবে `for` লুপের কাউন্টার ব্যবহার করি এবং আমরা প্রতিটি নতুন `Worker` কে `workers` নামের ভেক্টরে সংরক্ষণ করি।

বাহ্যিক কোড (যেমন _src/main.rs_-এ আমাদের সার্ভার) `ThreadPool`-এর মধ্যে একটি `Worker` স্ট্রাক্ট ব্যবহার সম্পর্কিত ইমপ্লিমেন্টেশনের বিবরণ জানার প্রয়োজন নেই, তাই আমরা `Worker` স্ট্রাক্ট এবং এর `new` ফাংশনকে প্রাইভেট করি। `Worker::new` ফাংশনটি আমরা যে `id` দিই সেটি ব্যবহার করে এবং একটি খালি ক্লোজার ব্যবহার করে একটি নতুন থ্রেড স্পন করে তৈরি করা একটি `JoinHandle<()>` ইন্সট্যান্স সংরক্ষণ করে।

> দ্রষ্টব্য: যদি অপারেটিং সিস্টেম পর্যাপ্ত সিস্টেম রিসোর্সের অভাবে একটি থ্রেড তৈরি করতে না পারে, তাহলে `thread::spawn` প্যানিক করবে। এটি আমাদের পুরো সার্ভারকে প্যানিক করে তুলবে, যদিও কিছু থ্রেড তৈরি করা সফল হতে পারে। সরলতার খাতিরে, এই আচরণটি ঠিক আছে, কিন্তু একটি প্রোডাকশন থ্রেড পুল ইমপ্লিমেন্টেশনে, আপনি সম্ভবত [`std::thread::Builder`][builder]<!-- ignore --> এবং এর [`spawn`][builder-spawn]<!-- ignore --> মেথড ব্যবহার করতে চাইবেন যা `Result` রিটার্ন করে।

এই কোডটি কম্পাইল হবে এবং `ThreadPool::new`-তে আর্গুমেন্ট হিসাবে নির্দিষ্ট করা `Worker` ইন্সট্যান্সের সংখ্যা স্টোর করবে। কিন্তু আমরা এখনও `execute`-এ পাওয়া ক্লোজারটি প্রক্রিয়া করছি _না_। আসুন দেখি কিভাবে এটি করতে হয়।

#### চ্যানেলগুলির মাধ্যমে থ্রেডগুলিতে অনুরোধ পাঠানো

আমরা যে পরবর্তী সমস্যাটির সমাধান করব তা হল `thread::spawn`-কে দেওয়া ক্লোজারগুলি কিছুই করে না। বর্তমানে, আমরা `execute` মেথডে যে ক্লোজারটি এক্সিকিউট করতে চাই সেটি পাই। কিন্তু `ThreadPool` তৈরির সময় প্রতিটি `Worker` তৈরি করার সময় আমাদের `thread::spawn`-কে চালানোর জন্য একটি ক্লোজার দিতে হবে।

আমরা চাই যে `Worker` স্ট্রাক্টগুলি যা আমরা এইমাত্র তৈরি করেছি সেগুলি `ThreadPool`-এ থাকা একটি কিউ থেকে চালানোর জন্য কোড ফেচ করুক এবং সেই কোডটি চালানোর জন্য তার থ্রেডে পাঠাক।

ষোড়শ অধ্যায়ে আমরা যে চ্যানেলগুলি সম্পর্কে শিখেছি—দুটি থ্রেডের মধ্যে যোগাযোগের একটি সহজ উপায়—এই ব্যবহারের ক্ষেত্রে উপযুক্ত হবে। আমরা কাজের কিউ হিসাবে কাজ করার জন্য একটি চ্যানেল ব্যবহার করব, এবং `execute` `ThreadPool` থেকে `Worker` ইন্সট্যান্সে একটি কাজ পাঠাবে, যা কাজটি তার থ্রেডে পাঠাবে। এখানে পরিকল্পনাটি রয়েছে:

1.  `ThreadPool` একটি চ্যানেল তৈরি করবে এবং সেন্ডারকে ধরে রাখবে।
2.  প্রতিটি `Worker` রিসিভারকে ধরে রাখবে।
3.  আমরা একটি নতুন `Job` স্ট্রাক্ট তৈরি করব যা চ্যানেলটিতে পাঠাতে চাওয়া ক্লোজারগুলিকে ধারণ করবে।
4.  `execute` মেথডটি যে কাজটি এক্সিকিউট করতে চায় সেটি সেন্ডারের মাধ্যমে পাঠাবে।
5.  তার থ্রেডে, `Worker` তার রিসিভারের উপর লুপ করবে এবং প্রাপ্ত যেকোনো কাজের ক্লোজার এক্সিকিউট করবে।

আসুন `ThreadPool::new`-তে একটি চ্যানেল তৈরি করে এবং `ThreadPool` ইন্সট্যান্সে সেন্ডারটিকে ধরে রেখে শুরু করি, যেমনটি লিস্টিং ২১-১৬-তে দেখানো হয়েছে। `Job` স্ট্রাক্ট আপাতত কিছুই ধারণ করে না তবে এটি চ্যানেলে পাঠানো আইটেমটির টাইপ হবে।

<Listing number="21-16" file-name="src/lib.rs" caption="`Job` ইন্সট্যান্স ট্রান্সমিট করে এমন একটি চ্যানেলের সেন্ডার সংরক্ষণ করতে `ThreadPool` পরিবর্তন করা">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

`ThreadPool::new`-তে, আমরা আমাদের নতুন চ্যানেল তৈরি করি এবং পুলটিকে সেন্ডার ধরে রাখতে দিই। এটি সফলভাবে কম্পাইল হবে।

আসুন থ্রেড পুল চ্যানেল তৈরি করার সাথে সাথে প্রতিটি `Worker`-এ চ্যানেলের একটি রিসিভার পাস করার চেষ্টা করি। আমরা জানি যে আমরা `Worker` ইন্সট্যান্সগুলি যে থ্রেড স্পন করে তাতে রিসিভারটি ব্যবহার করতে চাই, তাই আমরা ক্লোজারে `receiver` প্যারামিটারটি রেফারেন্স করব। লিস্টিং ২১-১৭-এর কোডটি এখনও পুরোপুরি কম্পাইল হবে না।

<Listing number="21-17" file-name="src/lib.rs" caption="প্রতিটি `Worker`-কে রিসিভার পাস করা">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

আমরা কিছু ছোট এবং সহজবোধ্য পরিবর্তন করেছি: আমরা `Worker::new`-তে রিসিভারটি পাস করি এবং তারপর আমরা এটি ক্লোজারের ভিতরে ব্যবহার করি।

যখন আমরা এই কোডটি চেক করার চেষ্টা করি, তখন আমরা এই এররটি পাই:

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

কোডটি একাধিক `Worker` ইন্সট্যান্সে `receiver` পাস করার চেষ্টা করছে। এটি কাজ করবে না, যেমনটি আপনি ষোড়শ অধ্যায় থেকে মনে করবেন: Rust যে চ্যানেল ইমপ্লিমেন্টেশন সরবরাহ করে তা হল মাল্টিপল _প্রোডিউসার_, সিঙ্গেল _কনজিউমার_। এর মানে হল আমরা এই কোডটি ঠিক করার জন্য চ্যানেলের কনজিউমিং প্রান্তটি ক্লোন করতে পারি না। আমরা একাধিক কনজিউমারের কাছে একাধিকবার একটি মেসেজ পাঠাতে চাই না; আমরা একাধিক `Worker` ইন্সট্যান্স সহ মেসেজের একটি তালিকা চাই যাতে প্রতিটি মেসেজ একবার প্রক্রিয়া করা হয়।

অতিরিক্তভাবে, চ্যানেল কিউ থেকে একটি কাজ নেওয়া `receiver`-কে মিউটেট করে, তাই থ্রেডগুলির `receiver` শেয়ার এবং মডিফাই করার একটি নিরাপদ উপায় প্রয়োজন; অন্যথায়, আমরা রেস কন্ডিশন পেতে পারি (যেমনটি ষোড়শ অধ্যায়ে আলোচনা করা হয়েছে)।

ষোড়শ অধ্যায়ে আলোচিত থ্রেড-নিরাপদ স্মার্ট পয়েন্টারগুলির কথা মনে করুন: একাধিক থ্রেড জুড়ে ওনারশিপ শেয়ার করতে এবং থ্রেডগুলিকে ভ্যালু মিউটেট করার অনুমতি দেওয়ার জন্য, আমাদের `Arc<Mutex<T>>` ব্যবহার করতে হবে। `Arc` টাইপ একাধিক `Worker` ইন্সট্যান্সকে রিসিভারের মালিক হতে দেবে এবং `Mutex` নিশ্চিত করবে যে একবারে শুধুমাত্র একটি `Worker` রিসিভার থেকে একটি কাজ পায়। লিস্টিং ২১-১৮ আমাদের যে পরিবর্তনগুলি করতে হবে তা দেখায়।

<Listing number="21-18" file-name="src/lib.rs" caption="`Arc` এবং `Mutex` ব্যবহার করে `Worker` ইন্সট্যান্সগুলির মধ্যে রিসিভার শেয়ার করা">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

`ThreadPool::new`-তে, আমরা রিসিভারটিকে একটি `Arc` এবং একটি `Mutex`-এ রাখি। প্রতিটি নতুন `Worker`-এর জন্য, আমরা `Arc` ক্লোন করি যাতে রেফারেন্স কাউন্ট বাড়ে, যাতে `Worker` ইন্সট্যান্সগুলি রিসিভারের ওনারশিপ শেয়ার করতে পারে।

এই পরিবর্তনগুলির সাথে, কোড কম্পাইল হয়! আমরা সেখানে পৌঁছে যাচ্ছি!

#### `execute` মেথড ইমপ্লিমেন্ট করা

আসুন অবশেষে `ThreadPool`-এ `execute` মেথডটি ইমপ্লিমেন্ট করি। আমরা `execute` যে ধরনের ক্লোজার পায় সেটি ধারণ করে এমন একটি ট্রেইট অবজেক্টের জন্য `Job`-কে একটি স্ট্রাক্ট থেকে একটি টাইপ অ্যালিয়াসে পরিবর্তন করব। যেমনটি বিংশ অধ্যায়ের [“টাইপ অ্যালিয়াস সহ টাইপ সিনোনিম তৈরি করা”][creating-type-synonyms-with-type-aliases]<!-- ignore --> তে আলোচনা করা হয়েছে, টাইপ অ্যালিয়াসগুলি আমাদের ব্যবহারের সুবিধার জন্য দীর্ঘ টাইপগুলিকে ছোট করার অনুমতি দেয়। লিস্টিং ২১-১৯ দেখুন।

<Listing number="21-19" file-name="src/lib.rs" caption="প্রতিটি ক্লোজার ধারণ করে এমন একটি `Box`-এর জন্য একটি `Job` টাইপ অ্যালিয়াস তৈরি করা এবং তারপর চ্যানেলে কাজটি পাঠানো">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

`execute`-এ পাওয়া ক্লোজারটি ব্যবহার করে একটি নতুন `Job` ইন্সট্যান্স তৈরি করার পরে, আমরা সেই কাজটি চ্যানেলের পাঠানোর প্রান্তে পাঠাই। পাঠানোর ক্ষেত্রে ব্যর্থ হলে আমরা `send`-এ `unwrap` কল করছি। উদাহরণস্বরূপ, যদি আমরা আমাদের সমস্ত থ্রেড এক্সিকিউট করা বন্ধ করে দিই, তাহলে এটি ঘটতে পারে, যার অর্থ রিসিভিং প্রান্তটি নতুন মেসেজ গ্রহণ করা বন্ধ করে দিয়েছে। এই মুহূর্তে, আমরা আমাদের থ্রেডগুলিকে এক্সিকিউট করা বন্ধ করতে পারি না: পুলটি যতদিন বিদ্যমান থাকে ততদিন আমাদের থ্রেডগুলি এক্সিকিউট করতে থাকে। আমরা `unwrap` ব্যবহার করার কারণ হল আমরা জানি যে ব্যর্থতার ঘটনা ঘটবে না, কিন্তু কম্পাইলার তা জানে না।

কিন্তু আমরা এখনও পুরোপুরি শেষ করিনি! `Worker`-এ, `thread::spawn`-কে পাস করা আমাদের ক্লোজারটি এখনও চ্যানেলের রিসিভিং প্রান্তটিকে _রেফারেন্স_ করে। পরিবর্তে, আমাদের ক্লোজারটিকে চিরতরে লুপ করতে হবে, চ্যানেলের রিসিভিং প্রান্তটিকে একটি কাজের জন্য জিজ্ঞাসা করতে হবে এবং যখন এটি একটি কাজ পাবে তখন সেটি চালাতে হবে। আসুন `Worker::new`-তে লিস্টিং ২১-২০-তে দেখানো পরিবর্তনটি করি।

<Listing number="21-20" file-name="src/lib.rs" caption="`Worker` ইন্সট্যান্সের থ্রেডে কাজগুলি গ্রহণ এবং এক্সিকিউট করা">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

এখানে, আমরা প্রথমে মিউটেক্স অর্জন করতে `receiver`-এ `lock` কল করি এবং তারপর কোনো এরর-এ প্যানিক করার জন্য `unwrap` কল করি। যদি মিউটেক্সটি একটি _পয়জনড_ অবস্থায় থাকে তবে লক অর্জন করা ব্যর্থ হতে পারে, যা ঘটতে পারে যদি অন্য কোনো থ্রেড লকটি রিলিজ করার পরিবর্তে লকটি ধরে রাখার সময় প্যানিক করে। এই পরিস্থিতিতে, এই থ্রেডটিকে প্যানিক করার জন্য `unwrap` কল করা হল সঠিক কাজ। আপনার জন্য অর্থপূর্ণ একটি এরর মেসেজ সহ এই `unwrap` কে একটি `expect`-এ পরিবর্তন করতে পারেন।

যদি আমরা মিউটেক্সের উপর লক পাই, তাহলে আমরা চ্যানেল থেকে একটি `Job` রিসিভ করার জন্য `recv` কল করি। একটি ফাইনাল `unwrap` এখানেও যেকোনো এরর অতিক্রম করে, যা ঘটতে পারে যদি সেন্ডার ধারণ করা থ্রেডটি বন্ধ হয়ে যায়, একইভাবে `send` মেথড `Err` রিটার্ন করে যদি রিসিভার বন্ধ হয়ে যায়।

`recv`-তে কলটি ব্লক করে, তাই যদি এখনও কোনো কাজ না থাকে, তাহলে বর্তমান থ্রেডটি একটি কাজ উপলব্ধ না হওয়া পর্যন্ত অপেক্ষা করবে। `Mutex<T>` নিশ্চিত করে যে একবারে শুধুমাত্র একটি `Worker` থ্রেড একটি কাজের অনুরোধ করার চেষ্টা করছে।

আমাদের থ্রেড পুলটি এখন একটি কার্যকরী অবস্থায় রয়েছে! এটিকে একটি `cargo run` দিন এবং কিছু অনুরোধ করুন:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

সফল! আমাদের এখন একটি থ্রেড পুল রয়েছে যা অ্যাসিঙ্ক্রোনাসভাবে কানেকশন এক্সিকিউট করে। কখনও চারটির বেশি থ্রেড তৈরি হয় না, তাই সার্ভার প্রচুর অনুরোধ পেলেও আমাদের সিস্টেম ওভারলোড হবে না। যদি আমরা _/sleep_-এ একটি অনুরোধ করি, তাহলে সার্ভার অন্য থ্রেড চালিয়ে অন্য অনুরোধগুলি পরিবেশন করতে সক্ষম হবে।

> দ্রষ্টব্য: আপনি যদি একই সাথে একাধিক ব্রাউজার উইন্ডোতে _/sleep_ খোলেন, তাহলে সেগুলি পাঁচ সেকেন্ডের ব্যবধানে একবারে একটি লোড হতে পারে। কিছু ওয়েব ব্রাউজার ক্যাশিংয়ের কারণে একই অনুরোধের একাধিক ইন্সট্যান্স ক্রমানুসারে এক্সিকিউট করে। এই সীমাবদ্ধতা আমাদের ওয়েব সার্ভারের কারণে নয়।

এখানে থামার এবং লিস্টিং 21-18, 21-19 এবং 21-20-এর কোডগুলি কীভাবে আলাদা হত তা বিবেচনা করার জন্য এটি একটি ভাল সময়, যদি আমরা কাজ করার জন্য ক্লোজারের পরিবর্তে ফিউচার ব্যবহার করতাম। কোন টাইপগুলি পরিবর্তন হবে? মেথড স্বাক্ষরগুলি কীভাবে আলাদা হবে, যদি আদৌ হয়? কোডের কোন অংশগুলি একই থাকবে?

17 এবং 18 অধ্যায়ে `while let` লুপ সম্পর্কে জানার পরে, আপনি হয়ত ভাবছেন কেন আমরা লিস্টিং 21-21-এ দেখানো ওয়ার্কার থ্রেড কোডটি লিখিনি।

<Listing number="21-21" file-name="src/lib.rs" caption="`while let` ব্যবহার করে `Worker::new`-এর একটি বিকল্প ইমপ্লিমেন্টেশন">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

এই কোডটি কম্পাইল এবং রান করে কিন্তু কাঙ্ক্ষিত থ্রেডিং আচরণ দেয় না: একটি স্লো অনুরোধ এখনও অন্যান্য অনুরোধগুলিকে প্রসেস হওয়ার জন্য অপেক্ষা করাবে। কারণটি কিছুটা সূক্ষ্ম: `Mutex` স্ট্রাক্টের কোনো পাবলিক `unlock` মেথড নেই কারণ লকের ওনারশিপ `lock` মেথড রিটার্ন করা `LockResult<MutexGuard<T>>`-এর মধ্যে `MutexGuard<T>`-এর লাইফটাইমের উপর ভিত্তি করে। কম্পাইল করার সময়, বরো চেকার তখন এই নিয়মটি প্রয়োগ করতে পারে যে একটি `Mutex` দ্বারা সুরক্ষিত একটি রিসোর্স অ্যাক্সেস করা যাবে না যদি না আমরা লকটি ধরে রাখি। যাইহোক, যদি আমরা `MutexGuard<T>`-এর লাইফটাইম সম্পর্কে সচেতন না হই, তাহলে এই ইমপ্লিমেন্টেশনটি ইচ্ছার চেয়ে বেশি সময় ধরে লক ধরে রাখতে পারে।

লিস্টিং 21-20-এর কোডটি `let job =
receiver.lock().unwrap().recv().unwrap();` ব্যবহার করে কাজ করে কারণ `let`-এর সাথে, সমান চিহ্নের ডান পাশের এক্সপ্রেশনে ব্যবহৃত যেকোনো টেম্পোরারি ভ্যালু `let` স্টেটমেন্ট শেষ হওয়ার সাথে সাথেই ড্রপ হয়ে যায়। যাইহোক, `while
let` (এবং `if let` এবং `match`) অ্যাসোসিয়েটেড ব্লকের শেষ না হওয়া পর্যন্ত টেম্পোরারি ভ্যালু ড্রপ করে না। লিস্টিং 21-21-এ, `job()`-তে কলের সময়কাল ধরে লকটি ধরে রাখা হয়, যার অর্থ অন্য `Worker` ইন্সট্যান্সগুলি কাজ পেতে পারে না।

[creating-type-synonyms-with-type-aliases]: ch20-03-advanced-types.html#creating-type-synonyms-with-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[fn-traits]: ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
