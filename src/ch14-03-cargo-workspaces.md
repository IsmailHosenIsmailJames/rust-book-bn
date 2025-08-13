## কার্গো ওয়ার্কস্পেস (Cargo Workspaces)

অধ্যায় ১২-এ, আমরা একটি প্যাকেজ তৈরি করেছিলাম যেখানে একটি বাইনারি ক্রেট এবং একটি লাইব্রেরি ক্রেট অন্তর্ভুক্ত ছিল। আপনার প্রজেক্ট বড় হওয়ার সাথে সাথে আপনি দেখতে পারেন যে লাইব্রেরি ক্রেটটি আরও বড় হচ্ছে এবং আপনি আপনার প্যাকেজটিকে একাধিক লাইব্রেরি ক্রেটে ভাগ করতে চান। কার্গো এই ক্ষেত্রে _ওয়ার্কস্পেস_ (workspaces) নামে একটি ফিচার সরবরাহ করে যা একসাথে ডেভেলপ করা একাধিক সম্পর্কিত প্যাকেজ পরিচালনা করতে সাহায্য করতে পারে।

### একটি ওয়ার্কস্পেস তৈরি করা

একটি _ওয়ার্কস্পেস_ হলো এমন一组 প্যাকেজ যা একই _Cargo.lock_ এবং আউটপুট ডিরেক্টরি শেয়ার করে। আসুন আমরা একটি ওয়ার্কস্পেস ব্যবহার করে একটি প্রজেক্ট তৈরি করি—আমরা এখানে সাধারণ কোড ব্যবহার করব যাতে আমরা ওয়ার্কস্পেসের কাঠামোর উপর মনোযোগ দিতে পারি। একটি ওয়ার্কস্পেস গঠন করার বিভিন্ন উপায় আছে, তাই আমরা শুধু একটি সাধারণ উপায় দেখাব। আমাদের একটি ওয়ার্কস্পেস থাকবে যাতে একটি বাইনারি এবং দুটি লাইব্রেরি থাকবে। বাইনারিটি, যা মূল কার্যকারিতা প্রদান করবে, দুটি লাইব্রেরির উপর নির্ভর করবে। একটি লাইব্রেরি `add_one` ফাংশন এবং অন্য লাইব্রেরিটি `add_two` ফাংশন প্রদান করবে। এই তিনটি ক্রেট একই ওয়ার্কস্পেসের অংশ হবে। আমরা ওয়ার্কস্পেসের জন্য একটি নতুন ডিরেক্টরি তৈরি করে শুরু করব:

```console
$ mkdir add
$ cd add
```

এরপর, _add_ ডিরেক্টরিতে, আমরা _Cargo.toml_ ফাইল তৈরি করব যা পুরো ওয়ার্কস্পেস কনফিগার করবে। এই ফাইলে কোনো `[package]` সেকশন থাকবে না। পরিবর্তে, এটি একটি `[workspace]` সেকশন দিয়ে শুরু হবে যা আমাদের ওয়ার্কস্পেসে মেম্বার যোগ করার সুযোগ দেবে। আমরা আমাদের ওয়ার্কস্পেসে কার্গোর সর্বশেষ এবং সর্বশ্রেষ্ঠ রিজলভার অ্যালগরিদম ব্যবহার করার জন্য `resolver`-এর মান `"3"` সেট করব।

<span class="filename">ফাইলের নাম: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

এরপরে, আমরা _add_ ডিরেক্টরির মধ্যে `cargo new` চালিয়ে `adder` বাইনারি ক্রেট তৈরি করব:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

একটি ওয়ার্কস্পেসের ভিতরে `cargo new` চালালে নতুন তৈরি হওয়া প্যাকেজটি স্বয়ংক্রিয়ভাবে ওয়ার্কস্পেসের _Cargo.toml_-এর `[workspace]` ডেফিনিশনের `members` কী-তে যোগ হয়ে যায়, যেমন:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

এই মুহূর্তে, আমরা `cargo build` চালিয়ে ওয়ার্কস্পেসটি বিল্ড করতে পারি। আপনার _add_ ডিরেক্টরির ফাইলগুলো এইরকম দেখতে হবে:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

ওয়ার্কস্পেসের টপ লেভেলে একটি _target_ ডিরেক্টরি রয়েছে যেখানে কম্পাইল করা আর্টিফ্যাক্টগুলো রাখা হবে; `adder` প্যাকেজের নিজস্ব কোনো _target_ ডিরেক্টরি নেই। এমনকি যদি আমরা _adder_ ডিরেক্টরির ভেতর থেকে `cargo build` চালাই, কম্পাইল করা আর্টিফ্যাক্টগুলো _add/adder/target_ এর পরিবর্তে _add/target_ এই ডিরেক্টরিতেই জমা হবে। কার্গো একটি ওয়ার্কস্পেসের _target_ ডিরেক্টরি এইভাবে গঠন করে কারণ একটি ওয়ার্কস্পেসের ক্রেটগুলো একে অপরের উপর নির্ভর করার জন্য তৈরি। যদি প্রতিটি ক্রেটের নিজস্ব _target_ ডিরেক্টরি থাকত, তবে প্রতিটি ক্রেটকে ওয়ার্কস্পেসের অন্য ক্রেটগুলোকে পুনরায় কম্পাইল করতে হতো যাতে আর্টিফ্যাক্টগুলো তার নিজস্ব _target_ ডিরেক্টরিতে রাখা যায়। একটি _target_ ডিরেক্টরি শেয়ার করার মাধ্যমে, ক্রেটগুলো অপ্রয়োজনীয় রি-বিল্ডিং এড়াতে পারে।

### ওয়ার্কস্পেসে দ্বিতীয় প্যাকেজ তৈরি করা

এরপরে, আসুন ওয়ার্কস্পেসে আরেকটি মেম্বার প্যাকেজ তৈরি করি এবং এর নাম দিই `add_one`। `add_one` নামে একটি নতুন লাইব্রেরি ক্রেট তৈরি করুন:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

টপ-লেভেল _Cargo.toml_ ফাইলটি এখন `members` তালিকায় _add_one_ পাথ অন্তর্ভুক্ত করবে:

<span class="filename">ফাইলের নাম: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

আপনার _add_ ডিরেক্টরিতে এখন এই ডিরেক্টরি এবং ফাইলগুলো থাকা উচিত:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

_add_one/src/lib.rs_ ফাইলে, আসুন একটি `add_one` ফাংশন যোগ করি:

<span class="filename">ফাইলের নাম: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

এখন আমরা আমাদের বাইনারি সহ `adder` প্যাকেজটিকে আমাদের লাইব্রেরি সহ `add_one` প্যাকেজের উপর নির্ভর করাতে পারি। প্রথমে, আমাদের _adder/Cargo.toml_-এ `add_one`-এর জন্য একটি পাথ ডিপেন্ডেন্সি যোগ করতে হবে।

<span class="filename">ফাইলের নাম: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

কার্গো ধরে নেয় না যে একটি ওয়ার্কস্পেসের ক্রেটগুলো একে অপরের উপর নির্ভর করবে, তাই আমাদের ডিপেন্ডেন্সি সম্পর্কগুলো স্পষ্টভাবে উল্লেখ করতে হবে।

এরপর, আসুন `adder` ক্রেটে (`add_one` ক্রেট থেকে) `add_one` ফাংশনটি ব্যবহার করি। _adder/src/main.rs_ ফাইলটি খুলুন এবং `main` ফাংশনটিকে `add_one` ফাংশন কল করার জন্য পরিবর্তন করুন, যেমনটি তালিকা ১৪-৭ এ দেখানো হয়েছে।

<Listing number="14-7" file-name="adder/src/main.rs" caption="`adder` ক্রেট থেকে `add_one` লাইব্রেরি ক্রেট ব্যবহার করা">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

আসুন টপ-লেভেল _add_ ডিরেক্টরিতে `cargo build` চালিয়ে ওয়ার্কস্পেসটি বিল্ড করি!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

_add_ ডিরেক্টরি থেকে বাইনারি ক্রেটটি চালানোর জন্য, আমরা `-p` আর্গুমেন্ট এবং `cargo run` এর সাথে প্যাকেজের নাম ব্যবহার করে ওয়ার্কস্পেসের কোন প্যাকেজটি চালাতে চাই তা নির্দিষ্ট করতে পারি:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

এটি _adder/src/main.rs_ এর কোডটি চালায়, যা `add_one` ক্রেটের উপর নির্ভর করে।

#### একটি ওয়ার্কস্পেসে একটি এক্সটার্নাল প্যাকেজের উপর নির্ভর করা

লক্ষ্য করুন যে ওয়ার্কস্পেসের টপ লেভেলে শুধুমাত্র একটি _Cargo.lock_ ফাইল আছে, প্রতিটি ক্রেটের ডিরেক্টরিতে একটি করে _Cargo.lock_ না থাকার পরিবর্তে। এটি নিশ্চিত করে যে সমস্ত ক্রেট সমস্ত ডিপেন্ডেন্সির একই সংস্করণ ব্যবহার করছে। যদি আমরা _adder/Cargo.toml_ এবং _add_one/Cargo.toml_ ফাইলে `rand` প্যাকেজ যোগ করি, কার্গো উভয়কেই `rand`-এর একটি সংস্করণে রিজলভ করবে এবং তা একটি _Cargo.lock_-এ রেকর্ড করবে। ওয়ার্কস্পেসের সমস্ত ক্রেটকে একই ডিপেন্ডেন্সি ব্যবহার করতে বাধ্য করার মানে হলো ক্রেটগুলো সবসময় একে অপরের সাথে কম্প্যাটিবল থাকবে। আসুন `add_one/Cargo.toml_ ফাইলের `[dependencies]` সেকশনে `rand` ক্রেট যোগ করি যাতে আমরা `add_one` ক্রেটে `rand` ক্রেট ব্যবহার করতে পারি:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">ফাইলের নাম: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

আমরা এখন _add_one/src/lib.rs_ ফাইলে `use rand;` যোগ করতে পারি, এবং _add_ ডিরেক্টরিতে `cargo build` চালিয়ে পুরো ওয়ার্কস্পেস বিল্ড করলে `rand` ক্রেটটি নিয়ে আসা হবে এবং কম্পাইল করা হবে। আমরা একটি ওয়ার্নিং পাব কারণ আমরা স্কোপে আনা `rand`-কে রেফারেন্স করছি না:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

টপ-লেভেল _Cargo.lock_-এ এখন `add_one`-এর `rand`-এর উপর নির্ভরতার তথ্য রয়েছে। যাইহোক, যদিও `rand` ওয়ার্কস্পেসের কোথাও ব্যবহৃত হচ্ছে, আমরা ওয়ার্কস্পেসের অন্য ক্রেটগুলিতে এটি ব্যবহার করতে পারব না যতক্ষণ না আমরা তাদের _Cargo.toml_ ফাইলে `rand` যোগ করি। উদাহরণস্বরূপ, যদি আমরা `adder` প্যাকেজের জন্য _adder/src/main.rs_ ফাইলে `use rand;` যোগ করি, আমরা একটি এরর পাব:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

এটি ঠিক করার জন্য, `adder` প্যাকেজের জন্য _Cargo.toml_ ফাইলটি এডিট করুন এবং নির্দেশ করুন যে `rand` এটির জন্যও একটি ডিপেন্ডেন্সি। `adder` প্যাকেজ বিল্ড করলে _Cargo.lock_-এ `adder`-এর ডিপেন্ডেন্সি তালিকায় `rand` যোগ হবে, কিন্তু `rand`-এর কোনো অতিরিক্ত কপি ডাউনলোড করা হবে না। কার্গো নিশ্চিত করবে যে ওয়ার্কস্পেসের প্রতিটি প্যাকেজের প্রতিটি ক্রেট, যারা `rand` প্যাকেজ ব্যবহার করছে, তারা একই সংস্করণ ব্যবহার করবে যতক্ষণ তারা `rand`-এর কম্প্যাটিবল সংস্করণ নির্দিষ্ট করে, যা আমাদের স্পেস বাঁচায় এবং নিশ্চিত করে যে ওয়ার্কস্পেসের ক্রেটগুলো একে অপরের সাথে কম্প্যাটিবল হবে।

যদি ওয়ার্কস্পেসের ক্রেটগুলো একই ডিপেন্ডেন্সির ইনকম্প্যাটিবল সংস্করণ নির্দিষ্ট করে, কার্গো সেগুলোর প্রত্যেকটিকে রিজলভ করবে, কিন্তু তারপরও যত কম সম্ভব সংস্করণ রিজলভ করার চেষ্টা করবে।

#### একটি ওয়ার্কস্পেসে একটি টেস্ট যোগ করা

আরেকটি উন্নতির জন্য, আসুন `add_one` ক্রেটের মধ্যে `add_one::add_one` ফাংশনের একটি টেস্ট যোগ করি:

<span class="filename">ফাইলের নাম: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}```

এখন টপ-লেভেল _add_ ডিরেক্টরিতে `cargo test` চালান। এই ধরনের গঠনযুক্ত একটি ওয়ার্কস্পেসে `cargo test` চালালে ওয়ার্কস্পেসের সমস্ত ক্রেটের জন্য টেস্টগুলো চলবে:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

আউটপুটের প্রথম সেকশনটি দেখায় যে `add_one` ক্রেটের `it_works` টেস্টটি পাস করেছে। পরবর্তী সেকশনটি দেখায় যে `adder` ক্রেটে শূন্যটি টেস্ট পাওয়া গেছে, এবং তারপর শেষ সেকশনটি দেখায় যে `add_one` ক্রেটে শূন্যটি ডকুমেন্টেশন টেস্ট পাওয়া গেছে।

আমরা টপ-লেভেল ডিরেক্টরি থেকে একটি ওয়ার্কস্পেসের একটি নির্দিষ্ট ক্রেটের জন্য টেস্টও চালাতে পারি `-p` ফ্ল্যাগ ব্যবহার করে এবং যে ক্রেটটি আমরা টেস্ট করতে চাই তার নাম উল্লেখ করে:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

এই আউটপুটটি দেখায় যে `cargo test` শুধুমাত্র `add_one` ক্রেটের জন্য টেস্ট চালিয়েছে এবং `adder` ক্রেটের টেস্ট চালায়নি।

আপনি যদি ওয়ার্কস্পেসের ক্রেটগুলো [crates.io](https://crates.io/)-তে পাবলিশ করেন, ওয়ার্কস্পেসের প্রতিটি ক্রেটকে আলাদাভাবে পাবলিশ করতে হবে। `cargo test`-এর মতো, আমরা `-p` ফ্ল্যাগ ব্যবহার করে এবং যে ক্রেটটি আমরা পাবলিশ করতে চাই তার নাম উল্লেখ করে আমাদের ওয়ার্কস্পেসের একটি নির্দিষ্ট ক্রেট পাবলিশ করতে পারি।

অতিরিক্ত অনুশীলনের জন্য, `add_one` ক্রেটের মতোই এই ওয়ার্কস্পেসে একটি `add_two` ক্রেট যোগ করুন!

আপনার প্রজেক্ট বড় হওয়ার সাথে সাথে, একটি ওয়ার্কস্পেস ব্যবহার করার কথা বিবেচনা করুন: এটি আপনাকে একটি বড় কোডের ব্লবের চেয়ে ছোট, সহজে বোঝা যায় এমন কম্পোনেন্ট নিয়ে কাজ করতে সক্ষম করে। উপরন্তু, ক্রেটগুলোকে একটি ওয়ার্কস্পেসে রাখা ক্রেটগুলোর মধ্যে সমন্বয় সহজ করতে পারে যদি সেগুলো প্রায়শই একই সময়ে পরিবর্তন করা হয়।