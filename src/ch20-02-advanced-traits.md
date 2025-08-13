## Advanced Traits

আমরা প্রথমবার চ্যাপ্টার ১০-এর [“Traits: Defining Shared Behavior”][traits-defining-shared-behavior]<!-- ignore -->-এ trait নিয়ে আলোচনা করেছিলাম, কিন্তু তখন আমরা আরও গভীরে যাইনি। এখন যেহেতু আপনি রাস্ট সম্পর্কে আরও কিছু জানেন, আমরা খুঁটিনাটি বিষয়গুলোতে প্রবেশ করতে পারি।

<!-- Old link, do not remove -->
<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>

### Associated Types

_Associated types_ একটি টাইপ প্লেসহোল্ডারকে একটি trait-এর সাথে সংযুক্ত করে, যাতে trait মেথডের সংজ্ঞাগুলো তাদের সিগনেচারে এই প্লেসহোল্ডার টাইপগুলো ব্যবহার করতে পারে। কোনো trait-এর ইমপ্লিমেন্টর নির্দিষ্ট ইমপ্লিমেন্টেশনের জন্য প্লেসহোল্ডার টাইপের পরিবর্তে কোন সুনির্দিষ্ট (concrete) টাইপ ব্যবহার করা হবে তা নির্দিষ্ট করে দেবে। এইভাবে, আমরা এমন একটি trait সংজ্ঞায়িত করতে পারি যা কিছু টাইপ ব্যবহার করে, কিন্তু trait টি ইমপ্লিমেন্ট না করা পর্যন্ত সেই টাইপগুলো ঠিক কী তা জানার প্রয়োজন হয় না।

আমরা এই চ্যাপ্টারের বেশিরভাগ advanced feature-কে এমনভাবে বর্ণনা করেছি যা খুব কমই প্রয়োজন হয়। Associated types মাঝামাঝি অবস্থানে রয়েছে: এগুলো বইয়ের বাকি অংশে ব্যাখ্যা করা feature-গুলোর চেয়ে কম ব্যবহৃত হয়, তবে এই চ্যাপ্টারে আলোচিত অন্যান্য অনেক feature-এর চেয়ে বেশি ব্যবহৃত হয়।

Associated type সহ একটি trait-এর উদাহরণ হলো স্ট্যান্ডার্ড লাইব্রেরির `Iterator` trait। এর associated type-টির নাম `Item` এবং এটি `Iterator` trait ইমপ্লিমেন্ট করা টাইপটি যে মানের উপর ইটারেট করছে তার টাইপের প্রতিনিধিত্ব করে। `Iterator` trait-এর সংজ্ঞাটি লিস্টিং ২০-১৩-এ দেখানো হয়েছে।

<Listing number="20-13" caption="`Iterator` trait-এর সংজ্ঞা, যার একটি associated type `Item` রয়েছে">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

`Item` টাইপটি একটি প্লেসহোল্ডার, এবং `next` মেথডের সংজ্ঞা দেখায় যে এটি `Option<Self::Item>` টাইপের মান রিটার্ন করবে। `Iterator` trait-এর ইমপ্লিমেন্টররা `Item`-এর জন্য সুনির্দিষ্ট টাইপ নির্দিষ্ট করবে এবং `next` মেথডটি সেই সুনির্দিষ্ট টাইপের মানসহ একটি `Option` রিটার্ন করবে।

Associated types-কে generics-এর মতো একটি ধারণা বলে মনে হতে পারে, কারণ generics আমাদের কোনো ফাংশন সংজ্ঞায়িত করার সুযোগ দেয় যেখানে এটি কোন টাইপ পরিচালনা করতে পারে তা নির্দিষ্ট করার প্রয়োজন হয় না। এই দুটি ধারণার মধ্যে পার্থক্য পরীক্ষা করার জন্য, আমরা `Counter` নামক একটি টাইপের উপর `Iterator` trait-এর একটি ইমপ্লিমেন্টেশন দেখব যা নির্দিষ্ট করে যে `Item` টাইপটি হলো `u32`:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

এই সিনট্যাক্সটি generics-এর সিনট্যাক্সের সাথে তুলনীয় বলে মনে হয়। তাহলে কেন `Iterator` trait-টিকে generics দিয়ে সংজ্ঞায়িত করা হলো না, যেমনটি লিস্টিং ২০-১৪-তে দেখানো হয়েছে?

<Listing number="20-14" caption="Generics ব্যবহার করে `Iterator` trait-এর একটি কাল্পনিক সংজ্ঞা">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

পার্থক্য হলো, যখন generics ব্যবহার করা হয়, যেমন লিস্টিং ২০-১৪-তে, আমাদের প্রতিটি ইমপ্লিমেন্টেশনে টাইপগুলো annotate করতে হবে; কারণ আমরা `Counter`-এর জন্য `Iterator<String>` বা অন্য যেকোনো টাইপও ইমপ্লিমেন্ট করতে পারতাম, ফলে `Counter`-এর জন্য `Iterator`-এর একাধিক ইমপ্লিমেন্টেশন থাকতে পারত। অন্য কথায়, যখন একটি trait-এর একটি জেনেরিক প্যারামিটার থাকে, তখন এটি একটি টাইপের জন্য একাধিকবার ইমপ্লিমেন্ট করা যেতে পারে, প্রতিবার জেনেরিক টাইপ প্যারামিটারের সুনির্দিষ্ট টাইপ পরিবর্তন করে। যখন আমরা `Counter`-এ `next` মেথড ব্যবহার করতাম, তখন আমাদের `Iterator`-এর কোন ইমপ্লিমেন্টেশনটি ব্যবহার করতে চাই তা নির্দেশ করার জন্য টাইপ অ্যানোটেশন সরবরাহ করতে হতো।

Associated types-এর সাথে, আমাদের টাইপ annotate করার প্রয়োজন হয় না কারণ আমরা একটি টাইপের উপর একটি trait একাধিকবার ইমপ্লিমেন্ট করতে পারি না। লিস্টিং ২০-১৩-তে associated types ব্যবহার করা সংজ্ঞার সাথে, আমরা কেবল একবারই `Item`-এর টাইপ কী হবে তা বেছে নিতে পারি কারণ `impl Iterator for Counter` কেবল একটাই থাকতে পারে। আমাদের প্রতিবার `Counter`-এ `next` কল করার সময় নির্দিষ্ট করতে হবে না যে আমরা `u32` মানের একটি iterator চাই।

Associated types trait-এর চুক্তিরও অংশ হয়ে যায়: trait-এর ইমপ্লিমেন্টরদের অবশ্যই associated type প্লেসহোল্ডারের জন্য একটি টাইপ সরবরাহ করতে হবে। Associated types-এর প্রায়শই একটি নাম থাকে যা বর্ণনা করে যে টাইপটি কীভাবে ব্যবহৃত হবে এবং API ডকুমেন্টেশনে associated type-টি নথিভুক্ত করা একটি ভাল অভ্যাস।

### Default Generic Type Parameters এবং Operator Overloading

যখন আমরা জেনেরিক টাইপ প্যারামিটার ব্যবহার করি, তখন আমরা জেনেরিক টাইপের জন্য একটি ডিফল্ট সুনির্দিষ্ট (concrete) টাইপ নির্দিষ্ট করতে পারি। এর ফলে trait-এর ইমপ্লিমেন্টরদের একটি সুনির্দিষ্ট টাইপ নির্দিষ্ট করার প্রয়োজন হয় না যদি ডিফল্ট টাইপটি কাজ করে। আপনি `<PlaceholderType=ConcreteType>` সিনট্যাক্স দিয়ে একটি জেনেরিক টাইপ ঘোষণা করার সময় একটি ডিফল্ট টাইপ নির্দিষ্ট করেন।

এই কৌশলটি যেখানে কার্যকর তার একটি சிறந்த উদাহরণ হলো _operator overloading_, যেখানে আপনি নির্দিষ্ট পরিস্থিতিতে একটি অপারেটরের (যেমন `+`) আচরণ কাস্টমাইজ করেন।

রাস্ট আপনাকে নিজের অপারেটর তৈরি করতে বা ইচ্ছামত অপারেটর ওভারলোড করার অনুমতি দেয় না। কিন্তু আপনি `std::ops`-এ তালিকাভুক্ত অপারেশন এবং সংশ্লিষ্ট trait-গুলো ওভারলোড করতে পারেন অপারেটরের সাথে যুক্ত trait-গুলো ইমপ্লিমেন্ট করে। উদাহরণস্বরূপ, লিস্টিং ২০-১৫-এ আমরা দুটি `Point` ইনস্ট্যান্সকে একসাথে যোগ করার জন্য `+` অপারেটরটিকে ওভারলোড করেছি। আমরা একটি `Point` struct-এ `Add` trait ইমপ্লিমেন্ট করে এটি করি।

<Listing number="20-15" file-name="src/main.rs" caption="`Point` ইনস্ট্যান্সের জন্য `+` অপারেটর ওভারলোড করতে `Add` trait ইমপ্লিমেন্ট করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

`add` মেথডটি দুটি `Point` ইনস্ট্যান্সের `x` মান এবং `y` মান যোগ করে একটি নতুন `Point` তৈরি করে। `Add` trait-টির `Output` নামে একটি associated type রয়েছে যা `add` মেথড থেকে রিটার্ন করা টাইপ নির্ধারণ করে।

এই কোডে ডিফল্ট জেনেরিক টাইপটি `Add` trait-এর মধ্যে রয়েছে। এখানে এর সংজ্ঞা:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

এই কোডটি সাধারণভাবে পরিচিত মনে হওয়া উচিত: একটি মেথড এবং একটি associated type সহ একটি trait। নতুন অংশটি হলো `Rhs=Self`: এই সিনট্যাক্সটিকে _default type parameters_ বলা হয়। `Rhs` জেনেরিক টাইপ প্যারামিটার ("right-hand side"-এর সংক্ষিপ্ত রূপ) `add` মেথডে `rhs` প্যারামিটারের টাইপ সংজ্ঞায়িত করে। যদি আমরা `Add` trait ইমপ্লিমেন্ট করার সময় `Rhs`-এর জন্য একটি সুনির্দিষ্ট টাইপ নির্দিষ্ট না করি, তাহলে `Rhs`-এর টাইপ ডিফল্ট হিসেবে `Self` হবে, যা হবে সেই টাইপ যার উপর আমরা `Add` ইমপ্লিমেন্ট করছি।

যখন আমরা `Point`-এর জন্য `Add` ইমপ্লিমেন্ট করেছিলাম, আমরা `Rhs`-এর জন্য ডিফল্ট ব্যবহার করেছি কারণ আমরা দুটি `Point` ইনস্ট্যান্স যোগ করতে চেয়েছিলাম। আসুন `Add` trait ইমপ্লিমেন্ট করার এমন একটি উদাহরণ দেখি যেখানে আমরা ডিফল্ট ব্যবহার না করে `Rhs` টাইপটি কাস্টমাইজ করতে চাই।

আমাদের দুটি struct আছে, `Millimeters` এবং `Meters`, যা বিভিন্ন এককে মান ধারণ করে। একটি বিদ্যমান টাইপকে অন্য একটি struct-এ এভাবে মোড়ানোর প্রক্রিয়াকে _newtype pattern_ বলা হয়, যা আমরা [“Using the Newtype Pattern to Implement External Traits”][newtype]<!-- ignore --> বিভাগে আরও বিস্তারিতভাবে বর্ণনা করব। আমরা মিলিমিটারের মানকে মিটারের মানের সাথে যোগ করতে চাই এবং `Add`-এর ইমপ্লিমেন্টেশনটি যেন সঠিকভাবে রূপান্তরটি করে। আমরা `Millimeters`-এর জন্য `Add` ইমপ্লিমেন্ট করতে পারি `Meters`-কে `Rhs` হিসেবে ব্যবহার করে, যেমনটি লিস্টিং ২০-১৬-তে দেখানো হয়েছে।

<Listing number="20-16" file-name="src/lib.rs" caption="`Millimeters` এবং `Meters` যোগ করার জন্য `Millimeters`-এ `Add` trait ইমপ্লিমেন্ট করা">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

`Millimeters` এবং `Meters` যোগ করার জন্য, আমরা `Rhs` টাইপ প্যারামিটারের মান সেট করতে `impl Add<Meters>` নির্দিষ্ট করি, ডিফল্ট `Self` ব্যবহার করার পরিবর্তে।

আপনি দুটি প্রধান উপায়ে ডিফল্ট টাইপ প্যারামিটার ব্যবহার করবেন:

১. বিদ্যমান কোড না ভেঙে একটি টাইপ প্রসারিত করতে।
২. নির্দিষ্ট ক্ষেত্রে কাস্টমাইজেশনের অনুমতি দিতে যা বেশিরভাগ ব্যবহারকারীর প্রয়োজন হবে না।

স্ট্যান্ডার্ড লাইব্রেরির `Add` trait দ্বিতীয় উদ্দেশ্যের একটি উদাহরণ: সাধারণত, আপনি দুটি একই ধরনের টাইপ যোগ করবেন, কিন্তু `Add` trait এর বাইরেও কাস্টমাইজ করার ক্ষমতা প্রদান করে। `Add` trait-এর সংজ্ঞায় একটি ডিফল্ট টাইপ প্যারামিটার ব্যবহার করার অর্থ হলো বেশিরভাগ সময় আপনাকে অতিরিক্ত প্যারামিটার নির্দিষ্ট করতে হবে না। অন্য কথায়, সামান্য ইমপ্লিমেন্টেশন বয়লারপ্লেটের প্রয়োজন হয় না, যা trait-টি ব্যবহার করা সহজ করে তোলে।

প্রথম উদ্দেশ্যটি দ্বিতীয়টির মতোই কিন্তু বিপরীত: যদি আপনি একটি বিদ্যমান trait-এ একটি টাইপ প্যারামিটার যোগ করতে চান, আপনি trait-এর কার্যকারিতা প্রসারিত করার জন্য এটিকে একটি ডিফল্ট দিতে পারেন বিদ্যমান ইমপ্লিমেন্টেশন কোড না ভেঙে।

<!-- Old link, do not remove -->
<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>

### একই নামের মেথডগুলোর মধ্যে পার্থক্য করা

রাস্টে এমন কোনো নিয়ম নেই যা একটি trait-কে অন্য একটি trait-এর মেথডের সাথে একই নামের মেথড থাকা থেকে বিরত রাখে, বা রাস্ট আপনাকে একটি টাইপের উপর উভয় trait ইমপ্লিমেন্ট করা থেকে বিরত রাখে না। trait-এর মেথডের মতো একই নামের একটি মেথড সরাসরি টাইপের উপর ইমপ্লিমেন্ট করাও সম্ভব।

একই নামের মেথড কল করার সময়, আপনাকে রাস্টকে বলতে হবে আপনি কোনটি ব্যবহার করতে চান। লিস্টিং ২০-১৭-এর কোডটি বিবেচনা করুন যেখানে আমরা দুটি trait, `Pilot` এবং `Wizard`, সংজ্ঞায়িত করেছি যে দুটিরই `fly` নামে একটি মেথড আছে। তারপর আমরা `Human` নামক একটি টাইপের উপর উভয় trait ইমপ্লিমেন্ট করি, যার উপর ইতিমধ্যে `fly` নামের একটি মেথড ইমপ্লিমেন্ট করা আছে। প্রতিটি `fly` মেথড ভিন্ন কিছু করে।

<Listing number="20-17" file-name="src/main.rs" caption="দুটি trait-কে `fly` মেথড সহ সংজ্ঞায়িত করা হয়েছে এবং `Human` টাইপের উপর ইমপ্লিমেন্ট করা হয়েছে, এবং একটি `fly` মেথড সরাসরি `Human`-এর উপর ইমপ্লিমেন্ট করা হয়েছে।">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

যখন আমরা `Human`-এর একটি ইনস্ট্যান্সের উপর `fly` কল করি, কম্পাইলার ডিফল্ট হিসেবে সেই মেথডটি কল করে যা সরাসরি টাইপের উপর ইমপ্লিমেন্ট করা হয়েছে, যেমনটি লিস্টিং ২০-১৮-তে দেখানো হয়েছে।

<Listing number="20-18" file-name="src/main.rs" caption="`Human`-এর একটি ইনস্ট্যান্সের উপর `fly` কল করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

এই কোডটি চালালে `*waving arms furiously*` প্রিন্ট হবে, যা দেখায় যে রাস্ট সরাসরি `Human`-এর উপর ইমপ্লিমেন্ট করা `fly` মেথডটি কল করেছে।

`Pilot` trait বা `Wizard` trait থেকে `fly` মেথডগুলো কল করতে, আমাদের কোন `fly` মেথডটি বোঝাতে চাই তা নির্দিষ্ট করার জন্য আরও স্পষ্ট সিনট্যাক্স ব্যবহার করতে হবে। লিস্টিং ২০-১৯ এই সিনট্যাক্সটি প্রদর্শন করে।

<Listing number="20-19" file-name="src/main.rs" caption="আমরা কোন trait-এর `fly` মেথডটি কল করতে চাই তা নির্দিষ্ট করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

মেথডের নামের আগে trait-এর নাম নির্দিষ্ট করা রাস্টকে স্পষ্ট করে দেয় যে আমরা `fly`-এর কোন ইমপ্লিমেন্টেশনটি কল করতে চাই। আমরা `Human::fly(&person)`-ও লিখতে পারতাম, যা লিস্টিং ২০-১৯-এ ব্যবহৃত `person.fly()`-এর সমতুল্য, কিন্তু যদি আমাদের দ্ব্যর্থতা নিরসনের প্রয়োজন না হয় তবে এটি লিখতে একটু দীর্ঘ।

এই কোডটি চালালে নিম্নলিখিতটি প্রিন্ট হয়:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

যেহেতু `fly` মেথডটি একটি `self` প্যারামিটার নেয়, যদি আমাদের দুটি _টাইপ_ থাকত যা উভয়ই একটি _trait_ ইমপ্লিমেন্ট করত, রাস্ট `self`-এর টাইপের উপর ভিত্তি করে একটি trait-এর কোন ইমপ্লিমেন্টেশনটি ব্যবহার করতে হবে তা বের করতে পারত।

তবে, যে associated function-গুলো মেথড নয়, তাদের `self` প্যারামিটার থাকে না। যখন একাধিক টাইপ বা trait থাকে যা একই ফাংশন নামের non-method ফাংশন সংজ্ঞায়িত করে, রাস্ট সবসময় জানে না আপনি কোন টাইপটি বোঝাতে চান যদি না আপনি fully qualified syntax ব্যবহার করেন। উদাহরণস্বরূপ, লিস্টিং ২০-২০-এ আমরা একটি পশু আশ্রয়কেন্দ্রের জন্য একটি trait তৈরি করি যা সব বাচ্চা কুকুরের নাম Spot রাখতে চায়। আমরা `baby_name` নামে একটি associated non-method ফাংশন সহ একটি `Animal` trait তৈরি করি। `Animal` trait টি `Dog` struct-এর জন্য ইমপ্লিমেন্ট করা হয়েছে, যার উপর আমরা সরাসরি `baby_name` নামে একটি associated non-method ফাংশনও সরবরাহ করি।

<Listing number="20-20" file-name="src/main.rs" caption="একটি associated function সহ একটি trait এবং একই নামের একটি associated function সহ একটি টাইপ যা trait-টিও ইমপ্লিমেন্ট করে">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}```

</Listing>

আমরা `Dog`-এর উপর সংজ্ঞায়িত `baby_name` associated function-এ সব কুকুরছানার নাম Spot রাখার কোডটি ইমপ্লিমেন্ট করি। `Dog` টাইপটি `Animal` trait-ও ইমপ্লিমেন্ট করে, যা সব প্রাণীর বৈশিষ্ট্য বর্ণনা করে। বাচ্চা কুকুরকে puppy বলা হয়, এবং এটি `Animal` trait-এর সাথে যুক্ত `baby_name` ফাংশনে `Dog`-এর উপর `Animal` trait-এর ইমপ্লিমেন্টেশনে প্রকাশ করা হয়েছে।

`main`-এ, আমরা `Dog::baby_name` ফাংশনটি কল করি, যা সরাসরি `Dog`-এর উপর সংজ্ঞায়িত associated function-টিকে কল করে। এই কোডটি নিম্নলিখিতটি প্রিন্ট করে:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

এই আউটপুটটি আমরা যা চেয়েছিলাম তা নয়। আমরা `Dog`-এর উপর ইমপ্লিমেন্ট করা `Animal` trait-এর অংশ `baby_name` ফাংশনটি কল করতে চাই যাতে কোডটি `A baby dog is called a puppy` প্রিন্ট করে। লিস্টিং ২০-১৯-এ ব্যবহৃত trait-এর নাম নির্দিষ্ট করার কৌশলটি এখানে সাহায্য করবে না; যদি আমরা `main`-কে লিস্টিং ২০-২১-এর কোডে পরিবর্তন করি, আমরা একটি কম্পাইলেশন এরর পাব।

<Listing number="20-21" file-name="src/main.rs" caption="`Animal` trait থেকে `baby_name` ফাংশনটি কল করার চেষ্টা, কিন্তু রাস্ট জানে না কোন ইমপ্লিমেন্টেশনটি ব্যবহার করতে হবে">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

যেহেতু `Animal::baby_name`-এর কোনো `self` প্যারামিটার নেই, এবং এমন অন্যান্য টাইপ থাকতে পারে যা `Animal` trait ইমপ্লিমেন্ট করে, রাস্ট বের করতে পারে না আমরা `Animal::baby_name`-এর কোন ইমপ্লিমেন্টেশনটি চাই। আমরা এই কম্পাইলার এররটি পাব:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

দ্ব্যর্থতা নিরসন করতে এবং রাস্টকে বলতে যে আমরা অন্য কোনো টাইপের `Animal` ইমপ্লিমেন্টেশনের পরিবর্তে `Dog`-এর জন্য `Animal`-এর ইমপ্লিমেন্টেশন ব্যবহার করতে চাই, আমাদের fully qualified syntax ব্যবহার করতে হবে। লিস্টিং ২০-২২ fully qualified syntax কীভাবে ব্যবহার করতে হয় তা প্রদর্শন করে।

<Listing number="20-22" file-name="src/main.rs" caption="Fully qualified syntax ব্যবহার করে নির্দিষ্ট করা যে আমরা `Dog`-এর উপর ইমপ্লিমেন্ট করা `Animal` trait থেকে `baby_name` ফাংশনটি কল করতে চাই">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

আমরা অ্যাঙ্গেল ব্র্যাকেটের মধ্যে রাস্টকে একটি টাইপ অ্যানোটেশন প্রদান করছি, যা নির্দেশ করে যে আমরা `Animal` trait থেকে `baby_name` মেথডটি কল করতে চাই যা `Dog`-এর উপর ইমপ্লিমেন্ট করা হয়েছে, এই ফাংশন কলের জন্য `Dog` টাইপটিকে একটি `Animal` হিসাবে বিবেচনা করতে বলে। এই কোডটি এখন আমরা যা চাই তা প্রিন্ট করবে:

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

সাধারণভাবে, fully qualified syntax নিম্নরূপ সংজ্ঞায়িত করা হয়:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

যেসব associated function মেথড নয়, তাদের জন্য কোনো `receiver` থাকবে না: কেবল অন্যান্য আর্গুমেন্টের তালিকা থাকবে। আপনি ফাংশন বা মেথড কল করার সব জায়গায় fully qualified syntax ব্যবহার করতে পারতেন। তবে, এই সিনট্যাক্সের যেকোনো অংশ যা রাস্ট প্রোগ্রামের অন্যান্য তথ্য থেকে বের করতে পারে তা বাদ দেওয়ার অনুমতি আপনার আছে। আপনাকে কেবল সেইসব ক্ষেত্রে এই দীর্ঘ সিনট্যাক্স ব্যবহার করতে হবে যেখানে একই নাম ব্যবহার করে একাধিক ইমপ্লিমেন্টেশন রয়েছে এবং রাস্টকে সনাক্ত করতে সাহায্যের প্রয়োজন হয় আপনি কোন ইমপ্লিমেন্টেশনটি কল করতে চান।

<!-- Old link, do not remove -->
<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### Supertraits ব্যবহার করা

কখনও কখনও আপনি এমন একটি trait সংজ্ঞা লিখতে পারেন যা অন্য একটি trait-এর উপর নির্ভরশীল: একটি টাইপের জন্য প্রথম trait-টি ইমপ্লিমেন্ট করার জন্য, আপনি চাইতে পারেন যে সেই টাইপটি দ্বিতীয় trait-টিও ইমপ্লিমেন্ট করুক। আপনি এটি করবেন যাতে আপনার trait সংজ্ঞাটি দ্বিতীয় trait-এর associated item-গুলো ব্যবহার করতে পারে। যে trait-এর উপর আপনার trait সংজ্ঞাটি নির্ভর করছে তাকে আপনার trait-এর একটি _supertrait_ বলা হয়।

উদাহরণস্বরূপ, ধরা যাক আমরা একটি `OutlinePrint` trait তৈরি করতে চাই যার `outline_print` মেথডটি একটি প্রদত্ত মানকে এমনভাবে ফরম্যাট করে প্রিন্ট করবে যাতে এটি তারকাচিহ্ন দ্বারা ফ্রেম করা থাকে। অর্থাৎ, একটি `Point` struct যা স্ট্যান্ডার্ড লাইব্রেরি trait `Display` ইমপ্লিমেন্ট করে `(x, y)` ফলাফল দেয়, যখন আমরা `x`-এর জন্য `1` এবং `y`-এর জন্য `3` সহ একটি `Point` ইনস্ট্যান্সের উপর `outline_print` কল করি, তখন এটি নিম্নলিখিতটি প্রিন্ট করা উচিত:

```text
**********
*        *
* (1, 3) *
*        *
**********```

`outline_print` মেথডের ইমপ্লিমেন্টেশনে, আমরা `Display` trait-এর কার্যকারিতা ব্যবহার করতে চাই। অতএব, আমাদের নির্দিষ্ট করতে হবে যে `OutlinePrint` trait-টি কেবল সেইসব টাইপের জন্য কাজ করবে যা `Display`-ও ইমপ্লিমেন্ট করে এবং `OutlinePrint`-এর প্রয়োজনীয় কার্যকারিতা সরবরাহ করে। আমরা trait সংজ্ঞায় `OutlinePrint: Display` নির্দিষ্ট করে তা করতে পারি। এই কৌশলটি trait-এ একটি trait bound যোগ করার মতো। লিস্টিং ২০-২৩ `OutlinePrint` trait-এর একটি ইমপ্লিমেন্টেশন দেখায়।

<Listing number="20-23" file-name="src/main.rs" caption="`OutlinePrint` trait ইমপ্লিমেন্ট করা যা `Display` থেকে কার্যকারিতা প্রয়োজন">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

যেহেতু আমরা নির্দিষ্ট করেছি যে `OutlinePrint`-এর জন্য `Display` trait প্রয়োজন, আমরা `to_string` ফাংশনটি ব্যবহার করতে পারি যা `Display` ইমপ্লিমেন্ট করে এমন যেকোনো টাইপের জন্য স্বয়ংক্রিয়ভাবে ইমপ্লিমেন্ট করা হয়। যদি আমরা trait নামের পরে একটি কোলন যোগ না করে এবং `Display` trait নির্দিষ্ট না করে `to_string` ব্যবহার করার চেষ্টা করতাম, আমরা একটি এরর পেতাম যে বর্তমান স্কোপে `&Self` টাইপের জন্য `to_string` নামের কোনো মেথড পাওয়া যায়নি।

আসুন দেখি কী ঘটে যখন আমরা `Display` ইমপ্লিমেন্ট করে না এমন একটি টাইপের উপর `OutlinePrint` ইমপ্লিমেন্ট করার চেষ্টা করি, যেমন `Point` struct:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

আমরা একটি এরর পাব যে `Display` প্রয়োজন কিন্তু ইমপ্লিমেন্ট করা হয়নি:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

এটি ঠিক করার জন্য, আমরা `Point`-এর উপর `Display` ইমপ্লিমেন্ট করি এবং `OutlinePrint`-এর প্রয়োজনীয় শর্ত পূরণ করি, এইভাবে:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

তারপর, `Point`-এর উপর `OutlinePrint` trait ইমপ্লিমেন্ট করা সফলভাবে কম্পাইল হবে, এবং আমরা একটি `Point` ইনস্ট্যান্সের উপর `outline_print` কল করে এটিকে তারকাচিহ্নের একটি আউটলাইনের মধ্যে প্রদর্শন করতে পারি।

<!-- Old link, do not remove -->
<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>

### External Types-এর উপর External Traits ইমপ্লিমেন্ট করার জন্য Newtype Pattern ব্যবহার করা

চ্যাপ্টার ১০-এর [“Implementing a Trait on a Type”][implementing-a-trait-on-a-type]<!-- ignore -->-এ, আমরা orphan rule-এর কথা উল্লেখ করেছি যা বলে যে আমরা কেবল একটি টাইপের উপর একটি trait ইমপ্লিমেন্ট করতে পারি যদি trait বা টাইপ, অথবা উভয়ই, আমাদের crate-এর জন্য লোকাল হয়। _newtype pattern_ ব্যবহার করে এই সীমাবদ্ধতা এড়ানো সম্ভব, যা একটি tuple struct-এ একটি নতুন টাইপ তৈরি করা জড়িত। (আমরা চ্যাপ্টার ৫-এর [“Using Tuple Structs Without Named Fields to Create Different Types”][tuple-structs]<!-- ignore -->-এ tuple struct কভার করেছি।) tuple struct-এর একটি ফিল্ড থাকবে এবং এটি সেই টাইপের একটি পাতলা র‍্যাপার (thin wrapper) হবে যার জন্য আমরা একটি trait ইমপ্লিমেন্ট করতে চাই। তারপর র‍্যাপার টাইপটি আমাদের crate-এর জন্য লোকাল হয়, এবং আমরা র‍্যাপারের উপর trait-টি ইমপ্লিমেন্ট করতে পারি। _Newtype_ শব্দটি Haskell প্রোগ্রামিং ভাষা থেকে উদ্ভূত হয়েছে। এই প্যাটার্নটি ব্যবহার করার জন্য কোনো রানটাইম পারফরম্যান্স পেনাল্টি নেই, এবং র‍্যাপার টাইপটি কম্পাইল টাইমে বাদ দেওয়া হয়।

উদাহরণস্বরূপ, ধরা যাক আমরা `Vec<T>`-এর উপর `Display` ইমপ্লিমেন্ট করতে চাই, যা orphan rule আমাদের সরাসরি করতে বাধা দেয় কারণ `Display` trait এবং `Vec<T>` টাইপ উভয়ই আমাদের crate-এর বাইরে সংজ্ঞায়িত। আমরা একটি `Wrapper` struct তৈরি করতে পারি যা `Vec<T>`-এর একটি ইনস্ট্যান্স ধারণ করে; তারপর আমরা `Wrapper`-এর উপর `Display` ইমপ্লিমেন্ট করতে পারি এবং `Vec<T>` মানটি ব্যবহার করতে পারি, যেমনটি লিস্টিং ২০-২৪-এ দেখানো হয়েছে।

<Listing number="20-24" file-name="src/main.rs" caption="`Display` ইমপ্লিমেন্ট করার জন্য `Vec<String>`-এর চারপাশে একটি `Wrapper` টাইপ তৈরি করা">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

`Display`-এর ইমপ্লিমেন্টেশনটি ভিতরের `Vec<T>` অ্যাক্সেস করার জন্য `self.0` ব্যবহার করে কারণ `Wrapper` একটি tuple struct এবং `Vec<T>` tuple-এর ০ ইনডেক্সের আইটেম। তারপর আমরা `Wrapper`-এর উপর `Display` trait-এর কার্যকারিতা ব্যবহার করতে পারি।

এই কৌশলটি ব্যবহার করার অসুবিধা হলো `Wrapper` একটি নতুন টাইপ, তাই এর মধ্যে থাকা মানের মেথডগুলো এর নেই। আমাদের `Vec<T>`-এর সমস্ত মেথড সরাসরি `Wrapper`-এর উপর ইমপ্লিমেন্ট করতে হবে যাতে মেথডগুলো `self.0`-কে ডে্লিগেট করে, যা আমাদের `Wrapper`-কে ঠিক একটি `Vec<T>`-এর মতো ব্যবহার করার অনুমতি দেবে। যদি আমরা চাইতাম যে নতুন টাইপটির ভিতরের টাইপের সমস্ত মেথড থাকুক, তাহলে `Wrapper`-এর উপর `Deref` trait ইমপ্লিমেন্ট করে ভিতরের টাইপটি রিটার্ন করা একটি সমাধান হবে (আমরা চ্যাপ্টার ১৫-এর [“Treating Smart Pointers Like Regular References with `Deref`”][smart-pointer-deref]<!-- ignore -->-এ `Deref` trait ইমপ্লিমেন্ট করার বিষয়ে আলোচনা করেছি)। যদি আমরা না চাইতাম যে `Wrapper` টাইপটির ভিতরের টাইপের সমস্ত মেথড থাকুক—উদাহরণস্বরূপ, `Wrapper` টাইপের আচরণ সীমিত করার জন্য—আমাদের কেবল সেই মেথডগুলো ম্যানুয়ালি ইমপ্লিমেন্ট করতে হতো যা আমরা চাই।

এই newtype pattern-টি তখনও কার্যকর যখন কোনো trait জড়িত থাকে না। আসুন ফোকাস পরিবর্তন করি এবং রাস্টের টাইপ সিস্টেমের সাথে ইন্টারঅ্যাক্ট করার কিছু advanced উপায় দেখি।

[newtype]: ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits-defining-shared-behavior]: ch10-02-traits.html#traits-defining-shared-behavior
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types