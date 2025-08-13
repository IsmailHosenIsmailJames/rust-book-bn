আপনার নির্দেশিকা অনুসারে, আমি বিষয়বস্তুর সারণীটি (table of contents) অনুবাদ করেছি। সমস্ত ফাইলের লিঙ্ক এবং প্রযুক্তিগত পরিভাষা অপরিবর্তিত রাখা হয়েছে।

# The Rust Programming Language

[Rust প্রোগ্রামিং ল্যাঙ্গুয়েজ](title-page.md)
[মুখবন্ধ](foreword.md)
[ভূমিকা](ch00-00-introduction.md)

## শুরু করা যাক

- [শুরু করা](ch01-00-getting-started.md)
  - [ইনস্টলেশন](ch01-01-installation.md)
  - [হ্যালো, ওয়ার্ল্ড!](ch01-02-hello-world.md)
  - [হ্যালো, কার্গো!](ch01-03-hello-cargo.md)

- [একটি অনুমান করার গেম প্রোগ্রামিং](ch02-00-guessing-game-tutorial.md)

- [সাধারণ প্রোগ্রামিং ধারণা](ch03-00-common-programming-concepts.md)
  - [ভেরিয়েবল এবং মিউটেবিলিটি](ch03-01-variables-and-mutability.md)
  - [ডেটা টাইপ](ch03-02-data-types.md)
  - [ফাংশন](ch03-03-how-functions-work.md)
  - [কমেন্ট](ch03-04-comments.md)
  - [কন্ট্রোল ফ্লো](ch03-05-control-flow.md)

- [মালিকানা (Ownership) বোঝা](ch04-00-understanding-ownership.md)
  - [মালিকানা (Ownership) কী?](ch04-01-what-is-ownership.md)
  - [রেফারেন্স এবং বরোয়িং](ch04-02-references-and-borrowing.md)
  - [স্লাইস টাইপ](ch04-03-slices.md)

- [সম্পর্কিত ডেটা গঠন করতে Struct ব্যবহার](ch05-00-structs.md)
  - [Structs ডিফাইন এবং ইনস্ট্যানশিয়েট করা](ch05-01-defining-structs.md)
  - [Structs ব্যবহার করে একটি উদাহরণ প্রোগ্রাম](ch05-02-example-structs.md)
  - [মেথড সিনট্যাক্স](ch05-03-method-syntax.md)

- [Enums এবং প্যাটার্ন ম্যাচিং](ch06-00-enums.md)
  - [একটি Enum ডিফাইন করা](ch06-01-defining-an-enum.md)
  - [`match` কন্ট্রোল ফ্লো কনস্ট্রাক্ট](ch06-02-match.md)
  - [`if let` এবং `let else` দিয়ে সংক্ষিপ্ত কন্ট্রোল ফ্লো](ch06-03-if-let.md)

## Rust এর প্রাথমিক জ্ঞান

- [প্যাকেজ, ক্রেট এবং মডিউল দিয়ে বড় প্রজেক্ট পরিচালনা](ch07-00-managing-growing-projects-with-packages-crates-and-modules.md)
  - [প্যাকেজ এবং ক্রেট](ch07-01-packages-and-crates.md)
  - [স্কোপ এবং প্রাইভেসি নিয়ন্ত্রণের জন্য মডিউল ডিফাইন করা](ch07-02-defining-modules-to-control-scope-and-privacy.md)
  - [মডিউল ট্রি-তে আইটেম রেফার করার জন্য পাথ](ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md)
  - [`use` কীওয়ার্ড দিয়ে স্কোপে পাথ নিয়ে আসা](ch07-04-bringing-paths-into-scope-with-the-use-keyword.md)
  - [মডিউলগুলোকে বিভিন্ন ফাইলে আলাদা করা](ch07-05-separating-modules-into-different-files.md)

- [সাধারণ কালেকশন](ch08-00-common-collections.md)
  - [ভেক্টর দিয়ে ভ্যালুর তালিকা সংরক্ষণ](ch08-01-vectors.md)
  - [স্ট্রিং দিয়ে UTF-8 এনকোডেড টেক্সট সংরক্ষণ](ch08-02-strings.md)
  - [হ্যাশ ম্যাপে সংশ্লিষ্ট ভ্যালুসহ কী (key) সংরক্ষণ](ch08-03-hash-maps.md)

- [এরর হ্যান্ডলিং](ch09-00-error-handling.md)
  - [`panic!` দিয়ে অপরিবর্তনীয় এরর](ch09-01-unrecoverable-errors-with-panic.md)
  - [`Result` দিয়ে পুনরুদ্ধারযোগ্য এরর](ch09-02-recoverable-errors-with-result.md)
  - [`panic!` করা বা না করা](ch09-03-to-panic-or-not-to-panic.md)

- [জেনেরিক টাইপ, ট্রেইট এবং লাইফটাইম](ch10-00-generics.md)
  - [জেনেরিক ডেটা টাইপ](ch10-01-syntax.md)
  - [ট্রেইট: শেয়ার্ড আচরণ ডিফাইন করা](ch10-02-traits.md)
  - [লাইফটাইম দিয়ে রেফারেন্স যাচাই করা](ch10-03-lifetime-syntax.md)

- [স্বয়ংক্রিয় টেস্ট লেখা](ch11-00-testing.md)
  - [কীভাবে টেস্ট লিখতে হয়](ch11-01-writing-tests.md)
  - [টেস্ট কীভাবে চালানো হবে তা নিয়ন্ত্রণ করা](ch11-02-running-tests.md)
  - [টেস্ট অর্গানাইজেশন](ch11-03-test-organization.md)

- [একটি I/O প্রজেক্ট: একটি কমান্ড লাইন প্রোগ্রাম তৈরি](ch12-00-an-io-project.md)
  - [কমান্ড লাইন আর্গুমেন্ট গ্রহণ করা](ch12-01-accepting-command-line-arguments.md)
  - [একটি ফাইল পড়া](ch12-02-reading-a-file.md)
  - [মডুলারিটি এবং এরর হ্যান্ডলিং উন্নত করতে রিফ্যাক্টরিং](ch12-03-improving-error-handling-and-modularity.md)
  - [টেস্ট ড্রিভেন ডেভেলপমেন্টের মাধ্যমে লাইব্রেরির কার্যকারিতা তৈরি](ch12-04-testing-the-librarys-functionality.md)
  - [এনভায়রনমেন্ট ভেরিয়েবল নিয়ে কাজ করা](ch12-05-working-with-environment-variables.md)
  - [স্ট্যান্ডার্ড আউটপুটের পরিবর্তে স্ট্যান্ডার্ড এরর-এ এরর মেসেজ লেখা](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Rust এর মতো করে ভাবা

- [ফাংশনাল ল্যাঙ্গুয়েজের বৈশিষ্ট্য: ইটারেটর এবং ক্লোজার](ch13-00-functional-features.md)
  - [ক্লোজার: অ্যানোনিমাস ফাংশন যা তাদের এনভায়রনমেন্ট ক্যাপচার করে](ch13-01-closures.md)
  - [ইটারেটর দিয়ে আইটেমের একটি সিরিজ প্রসেস করা](ch13-02-iterators.md)
  - [আমাদের I/O প্রজেক্টের উন্নতি](ch13-03-improving-our-io-project.md)
  - [পারফরম্যান্স তুলনা: লুপ বনাম ইটারেটর](ch13-04-performance.md)

- [কার্গো এবং Crates.io সম্পর্কে আরও](ch14-00-more-about-cargo.md)
  - [রিলিজ প্রোফাইল দিয়ে বিল্ড কাস্টমাইজ করা](ch14-01-release-profiles.md)
  - [Crates.io-তে একটি ক্রেট প্রকাশ করা](ch14-02-publishing-to-crates-io.md)
  - [কার্গো ওয়ার্কস্পেস](ch14-03-cargo-workspaces.md)
  - [`cargo install` দিয়ে Crates.io থেকে বাইনারি ইনস্টল করা](ch14-04-installing-binaries.md)
  - [কাস্টম কমান্ড দিয়ে কার্গো এক্সটেন্ড করা](ch14-05-extending-cargo.md)

- [স্মার্ট পয়েন্টার](ch15-00-smart-pointers.md)
  - [হিপের ডেটাতে পয়েন্ট করতে `Box<T>` ব্যবহার](ch15-01-box.md)
  - [`Deref` দিয়ে স্মার্ট পয়েন্টারকে রেগুলার রেফারেন্সের মতো ব্যবহার](ch15-02-deref.md)
  - [`Drop` ট্রেইট দিয়ে ক্লিনআপের সময় কোড চালানো](ch15-03-drop.md)
  - [`Rc<T>`, রেফারেন্স কাউন্টেড স্মার্ট পয়েন্টার](ch15-04-rc.md)
  - [`RefCell<T>` এবং ইন্টেরিয়র মিউটেবিলিটি প্যাটার্ন](ch15-05-interior-mutability.md)
  - [রেফারেন্স সাইকেল মেমোরি লিক করতে পারে](ch15-06-reference-cycles.md)

- [নির্ভীক কনকারেন্সি](ch16-00-concurrency.md)
  - [একই সাথে কোড চালানোর জন্য থ্রেড ব্যবহার](ch16-01-threads.md)
  - [থ্রেডগুলির মধ্যে ডেটা স্থানান্তরের জন্য মেসেজ পাসিং ব্যবহার](ch16-02-message-passing.md)
  - [শেয়ার্ড-স্টেট কনকারেন্সি](ch16-03-shared-state.md)
  - [`Send` এবং `Sync` ট্রেইট দিয়ে এক্সটেনসিবল কনকারেন্সি](ch16-04-extensible-concurrency-sync-and-send.md)

- [অ্যাসিঙ্ক্রোনাস প্রোগ্রামিংয়ের মৌলিক বিষয়: Async, Await, Futures, এবং Streams](ch17-00-async-await.md)
  - [ফিউচার এবং অ্যাসিঙ্ক সিনট্যাক্স](ch17-01-futures-and-syntax.md)
  - [অ্যাসিঙ্ক দিয়ে কনকারেন্সি প্রয়োগ](ch17-02-concurrency-with-async.md)
  - [যেকোনো সংখ্যক ফিউচার নিয়ে কাজ করা](ch17-03-more-futures.md)
  - [স্ট্রিম: ক্রমানুসারে ফিউচার](ch17-04-streams.md)
  - [অ্যাসিঙ্কের জন্য ট্রেইটগুলোর দিকে একটি গভীর দৃষ্টি](ch17-05-traits-for-async.md)
  - [ফিউচার, টাস্ক, এবং থ্রেড](ch17-06-futures-tasks-threads.md)

- [Rust এর অবজেক্ট ওরিয়েন্টেড প্রোগ্রামিং বৈশিষ্ট্য](ch18-00-oop.md)
  - [অবজেক্ট-ওরিয়েন্টেড ল্যাঙ্গুয়েজের বৈশিষ্ট্য](ch18-01-what-is-oo.md)
  - [বিভিন্ন টাইপের ভ্যালুর জন্য ট্রেইট অবজেক্ট ব্যবহার](ch18-02-trait-objects.md)
  - [একটি অবজেক্ট-ওরিয়েন্টেড ডিজাইন প্যাটার্ন ইমপ্লিমেন্ট করা](ch18-03-oo-design-patterns.md)

## অ্যাডভান্সড টপিক

- [প্যাটার্ন এবং ম্যাচিং](ch19-00-patterns.md)
  - [যেসব জায়গায় প্যাটার্ন ব্যবহার করা যেতে পারে](ch19-01-all-the-places-for-patterns.md)
  - [রিফিউটেবিলিটি: কোনো প্যাটার্ন ম্যাচ করতে ব্যর্থ হতে পারে কিনা](ch19-02-refutability.md)
  - [প্যাটার্ন সিনট্যাক্স](ch19-03-pattern-syntax.md)

- [অ্যাডভান্সড ফিচার](ch20-00-advanced-features.md)
  - [আনসেফ Rust](ch20-01-unsafe-rust.md)
  - [অ্যাডভান্সড ট্রেইট](ch20-02-advanced-traits.md)
  - [অ্যাডভান্সড টাইপ](ch20-03-advanced-types.md)
  - [অ্যাডভান্সড ফাংশন এবং ক্লোজার](ch20-04-advanced-functions-and-closures.md)
  - [ম্যাক্রো](ch20-05-macros.md)

- [চূড়ান্ত প্রজেক্ট: একটি মাল্টিথ্রেডেড ওয়েব সার্ভার তৈরি](ch21-00-final-project-a-web-server.md)
  - [একটি সিঙ্গেল-থ্রেডেড ওয়েব সার্ভার তৈরি](ch21-01-single-threaded.md)
  - [আমাদের সিঙ্গেল-থ্রেডেড সার্ভারকে মাল্টিথ্রেডেড সার্ভারে রূপান্তরিত করা](ch21-02-multithreaded.md)
  - [সুন্দরভাবে শাটডাউন এবং পরিচ্ছন্নতা](ch21-03-graceful-shutdown-and-cleanup.md)

- [পরিশিষ্ট](appendix-00.md)
  - [A - কীওয়ার্ড](appendix-01-keywords.md)
  - [B - অপারেটর এবং প্রতীক](appendix-02-operators.md)
  - [C - ডিরাইভেবল ট্রেইট](appendix-03-derivable-traits.md)
  - [D - দরকারি ডেভেলপমেন্ট টুল](appendix-04-useful-development-tools.md)
  - [E - এডিশন](appendix-05-editions.md)
  - [F - বইটির অনুবাদসমূহ](appendix-06-translation.md)
  - [G - Rust কীভাবে তৈরি হয় এবং “নাইটলি Rust”](appendix-07-nightly-rust.md)