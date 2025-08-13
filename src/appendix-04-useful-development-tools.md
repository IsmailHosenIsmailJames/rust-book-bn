## পরিশিষ্ট ঘ - দরকারি ডেভেলপমেন্ট টুল

এই পরিশিষ্টে, আমরা Rust প্রজেক্ট দ্বারা সরবরাহ করা কিছু দরকারি ডেভেলপমেন্ট টুল নিয়ে আলোচনা করব। আমরা স্বয়ংক্রিয় ফরম্যাটিং, ওয়ার্নিংগুলো দ্রুত সমাধান করার উপায়, একটি লিন্টার এবং IDE-এর সাথে ইন্টিগ্রেশন নিয়ে আলোচনা করব।

### `rustfmt` দিয়ে স্বয়ংক্রিয় ফরম্যাটিং

`rustfmt` টুলটি আপনার কোডকে কমিউনিটির কোড স্টাইল অনুযায়ী রিফরম্যাট করে। অনেক সহযোগী প্রজেক্টে `rustfmt` ব্যবহার করা হয় যাতে Rust লেখার সময় কোন স্টাইল ব্যবহার করা হবে তা নিয়ে তর্ক-বিতর্ক এড়ানো যায়: সবাই এই টুলটি ব্যবহার করে তাদের কোড ফরম্যাট করে।

Rust ইনস্টলেশনের সাথে `rustfmt` ডিফল্টভাবে অন্তর্ভুক্ত থাকে, তাই আপনার সিস্টেমে ইতোমধ্যে `rustfmt` এবং `cargo-fmt` প্রোগ্রামগুলো থাকা উচিত। এই দুটি কমান্ড `rustc` এবং `cargo`-এর মতোই, যেখানে `rustfmt` আরও সূক্ষ্ম নিয়ন্ত্রণ দেয় এবং `cargo-fmt` কার্গো ব্যবহারকারী একটি প্রজেক্টের কনভেনশনগুলো বোঝে। যেকোনো কার্গো প্রজেক্ট ফরম্যাট করতে, নিম্নলিখিতটি লিখুন:

```console
$ cargo fmt
```

এই কমান্ডটি চালালে বর্তমান ক্রেটের সমস্ত Rust কোড রিফরম্যাট হয়ে যায়। এটি শুধুমাত্র কোডের স্টাইল পরিবর্তন করবে, কোডের সেমান্টিক্স (অর্থ) নয়। `rustfmt` সম্পর্কে আরও তথ্যের জন্য, [এর ডকুমেন্টেশন][rustfmt] দেখুন।

### `rustfix` দিয়ে আপনার কোড ঠিক করুন

`rustfix` টুলটি Rust ইনস্টলেশনের সাথে অন্তর্ভুক্ত থাকে এবং এটি কম্পাইলারের সেইসব ওয়ার্নিং স্বয়ংক্রিয়ভাবে ঠিক করতে পারে যেগুলোর সমস্যা সমাধানের একটি স্পষ্ট উপায় আছে এবং সম্ভবত আপনি সেটাই করতে চান। আপনি সম্ভবত আগেও কম্পাইলার ওয়ার্নিং দেখেছেন। উদাহরণস্বরূপ, এই কোডটি বিবেচনা করুন:

<span class="filename">ফাইলের নাম: src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

এখানে, আমরা `x` ভেরিয়েবলটিকে মিউটেবল হিসেবে ডিফাইন করছি, কিন্তু আমরা আসলে এটিকে কখনও পরিবর্তন করছি না। Rust আমাদের এই বিষয়ে সতর্ক করে:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

ওয়ার্নিংটি আমাদের `mut` কীওয়ার্ডটি সরিয়ে ফেলার পরামর্শ দিচ্ছে। আমরা `rustfix` টুলটি ব্যবহার করে `cargo fix` কমান্ডটি চালিয়ে স্বয়ংক্রিয়ভাবে সেই পরামর্শটি প্রয়োগ করতে পারি:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

যখন আমরা আবার _src/main.rs_ দেখব, তখন আমরা দেখতে পাব যে `cargo fix` কোডটি পরিবর্তন করেছে:

<span class="filename">ফাইলের নাম: src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}```

`x` ভেরিয়েবলটি এখন ইমিউটেবল, এবং ওয়ার্নিংটি আর দেখা যাচ্ছে না।

আপনি বিভিন্ন Rust এডিশনের মধ্যে আপনার কোড স্থানান্তর করতেও `cargo fix` কমান্ডটি ব্যবহার করতে পারেন। এডিশনগুলো [পরিশিষ্ট ঙ][editions]-এ আলোচনা করা হয়েছে।

### `clippy` দিয়ে আরও লিন্ট

Clippy টুলটি হলো আপনার কোড বিশ্লেষণ করার জন্য লিন্টের একটি সংগ্রহ, যাতে আপনি সাধারণ ভুলগুলো ধরতে পারেন এবং আপনার Rust কোড উন্নত করতে পারেন। Clippy স্ট্যান্ডার্ড Rust ইনস্টলেশনের সাথে অন্তর্ভুক্ত থাকে।

যেকোনো কার্গো প্রজেক্টে Clippy-এর লিন্টগুলো চালাতে, নিম্নলিখিতটি লিখুন:

```console
$ cargo clippy
```

উদাহরণস্বরূপ, ধরুন আপনি একটি প্রোগ্রাম লিখেছেন যা একটি গাণিতিক ধ্রুবক, যেমন পাই (pi)-এর একটি আনুমানিক মান ব্যবহার করে, যেমন এই প্রোগ্রামটি করে:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

এই প্রজেক্টে `cargo clippy` চালালে এই এররটি আসে:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

এই এররটি আপনাকে জানায় যে Rust-এ ইতোমধ্যে আরও নির্ভুল একটি `PI` ধ্রুবক ডিফাইন করা আছে, এবং আপনি যদি সেই ধ্রুবকটি ব্যবহার করেন তবে আপনার প্রোগ্রামটি আরও সঠিক হবে। এরপর আপনি `PI` ধ্রুবকটি ব্যবহার করার জন্য আপনার কোড পরিবর্তন করবেন।

নিম্নলিখিত কোডটি Clippy থেকে কোনো এরর বা ওয়ার্নিং তৈরি করে না:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Clippy সম্পর্কে আরও তথ্যের জন্য, [এর ডকুমেন্টেশন][clippy] দেখুন।

### `rust-analyzer` ব্যবহার করে IDE ইন্টিগ্রেশন

IDE ইন্টিগ্রেশনে সাহায্য করার জন্য, Rust কমিউনিটি [`rust-analyzer`][rust-analyzer] ব্যবহার করার সুপারিশ করে। এই টুলটি হলো কম্পাইলার-কেন্দ্রিক ইউটিলিটিগুলোর একটি সেট যা [Language Server Protocol][lsp] (LSP) ব্যবহার করে কথা বলে, যা IDE এবং প্রোগ্রামিং ল্যাঙ্গুয়েজগুলোর একে অপরের সাথে যোগাযোগের জন্য একটি স্পেসিফিকেশন। বিভিন্ন ক্লায়েন্ট `rust-analyzer` ব্যবহার করতে পারে, যেমন [Visual Studio Code-এর জন্য Rust analyzer প্লাগ-ইন][vscode]।

ইনস্টলেশন নির্দেশাবলীর জন্য `rust-analyzer` প্রজেক্টের [হোম পেজ][rust-analyzer] দেখুন, তারপর আপনার নির্দিষ্ট IDE-তে ল্যাঙ্গুয়েজ সার্ভার সাপোর্ট ইনস্টল করুন। আপনার IDE স্বয়ংক্রিয়-সম্পূর্ণকরণ (autocompletion), সংজ্ঞায় ঝাঁপ (jump to definition), এবং ইনলাইন এরর (inline errors) এর মতো ক্ষমতা অর্জন করবে।

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.md
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer