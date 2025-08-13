# Rust প্রোগ্রামিং ল্যাঙ্গুয়েজ

![বিল্ড স্ট্যাটাস](https://github.com/rust-lang/book/workflows/CI/badge.svg)

এই রিপোজিটরিতে 'The Rust Programming Language' বইটির সোর্স রয়েছে।

[বইটি No Starch Press থেকে মুদ্রিত আকারে পাওয়া যায়][nostarch]৷

[nostarch]: https://nostarch.com/rust-programming-language-2nd-edition

আপনি বইটি বিনামূল্যে অনলাইনেও পড়তে পারেন। অনুগ্রহ করে লেটেস্ট [stable], [beta], বা [nightly] Rust রিলিজের সাথে সরবরাহ করা বইটি দেখুন। মনে রাখবেন যে সেই সংস্করণগুলোর সমস্যাগুলো হয়তো এই রিপোজিটরিতে ইতোমধ্যে সমাধান করা হয়েছে, কারণ সেই রিলিজগুলো কম ঘন ঘন আপডেট করা হয়।

[stable]: https://doc.rust-lang.org/stable/book/
[beta]: https://doc.rust-lang.org/beta/book/
[nightly]: https://doc.rust-lang.org/nightly/book/

বইটিতে প্রদর্শিত সমস্ত কোড লিস্টিং এর শুধু কোড ডাউনলোড করতে [রিলিজ] দেখুন।

[releases]: https://github.com/rust-lang/book/releases

## প্রয়োজনীয়তা

বইটি তৈরি করার জন্য [mdBook] প্রয়োজন, আদর্শভাবে সেই একই সংস্করণ যা rust-lang/rust [এই ফাইলে][rust-mdbook] ব্যবহার করে। এটি পেতে:

[mdBook]: https://github.com/rust-lang/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/master/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook --locked --version <version_num>
```

বইটি এই রিপোজিটরির দুটি mdbook প্লাগইনও ব্যবহার করে। আপনি যদি সেগুলি ইনস্টল না করেন, তাহলে বিল্ড করার সময় আপনি ওয়ার্নিং দেখতে পাবেন এবং আউটপুটটি ঠিক দেখাবে না, কিন্তু আপনি _তবুও_ বইটি বিল্ড করতে পারবেন। প্লাগইনগুলো ব্যবহার করতে, আপনার চালানো উচিত:

```bash
$ cargo install --locked --path packages/mdbook-trpl --force
```

## তৈরি করা (Building)

বইটি তৈরি করতে, টাইপ করুন:

```bash
$ mdbook build
```

আউটপুটটি `book` সাবডিরেক্টরিতে থাকবে। এটি দেখতে, আপনার ওয়েব ব্রাউজারে এটি খুলুন।

_Firefox:_

```bash
$ firefox book/index.html                       # লিনাক্স
$ open -a "Firefox" book/index.html             # OS X
$ Start-Process "firefox.exe" .\book\index.html # উইন্ডোজ (পাওয়ারশেল)
$ start firefox.exe .\book\index.html           # উইন্ডোজ (Cmd)
```

_Chrome:_

```bash
$ google-chrome book/index.html                 # লিনাক্স
$ open -a "Google Chrome" book/index.html       # OS X
$ Start-Process "chrome.exe" .\book\index.html  # উইন্ডোজ (পাওয়ারশেল)
$ start chrome.exe .\book\index.html            # উইন্ডোজ (Cmd)
```

টেস্ট চালানোর জন্য:

```bash
$ cd packages/trpl
$ mdbook test --library-path packages/trpl/target/debug/deps
```

## অবদান

আমরা আপনার সাহায্য পেলে খুশি হব! আমরা কোন ধরনের অবদান খুঁজছি সে সম্পর্কে জানতে অনুগ্রহ করে [CONTRIBUTING.md][contrib] দেখুন।

[contrib]: https://github.com/rust-lang/book/blob/main/CONTRIBUTING.md

যেহেতু বইটি [মুদ্রিত][nostarch] হয়, এবং যেহেতু আমরা যখন সম্ভব তখন বইটির অনলাইন সংস্করণটি মুদ্রিত সংস্করণের কাছাকাছি রাখতে চাই, তাই আপনার সমস্যা বা পুল রিকোয়েস্ট সমাধান করতে আমাদের স্বাভাবিকের চেয়ে বেশি সময় লাগতে পারে।

এখন পর্যন্ত, আমরা [Rust Editions](https://doc.rust-lang.org/edition-guide/)-এর সাথে মিল রেখে একটি বড় সংশোধন করে আসছি। সেই বড় সংশোধনগুলোর মধ্যে, আমরা শুধুমাত্র ভুল সংশোধন করব। যদি আপনার সমস্যা বা পুল রিকোয়েস্ট কঠোরভাবে একটি ভুল সংশোধন না করে, তবে এটি পরবর্তী বড় সংশোধনের সময় পর্যন্ত বসে থাকতে পারে: মাস বা বছরের ক্রমে আশা করুন। আপনার ধৈর্যের জন্য ধন্যবাদ!

### অনুবাদ

বইটি অনুবাদ করতে আমরা সাহায্য পেলে খুশি হব! বর্তমানে চলমান প্রচেষ্টায় যোগ দিতে [Translations] লেবেলটি দেখুন। একটি নতুন ভাষায় কাজ শুরু করতে একটি নতুন ইস্যু খুলুন! আমরা কোনো অনুবাদ মার্জ করার আগে একাধিক ভাষার জন্য [mdbook support] এর জন্য অপেক্ষা করছি, কিন্তু আপনি শুরু করতে পারেন!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5

## বানান পরীক্ষা

সোর্স ফাইলগুলোতে বানান ভুল স্ক্যান করতে, আপনি `ci` ডিরেক্টরিতে উপলব্ধ `spellcheck.sh` স্ক্রিপ্টটি ব্যবহার করতে পারেন। এর জন্য বৈধ শব্দের একটি অভিধান প্রয়োজন, যা `ci/dictionary.txt`-তে সরবরাহ করা হয়েছে। যদি স্ক্রিপ্টটি একটি ফলস পজিটিভ তৈরি করে (যেমন, আপনি `BTreeMap` শব্দটি ব্যবহার করেছেন যা স্ক্রিপ্টটি অবৈধ মনে করে), আপনাকে এই শব্দটি `ci/dictionary.txt`-তে যোগ করতে হবে (সামঞ্জস্যের জন্য সাজানো ক্রম বজায় রাখুন)।