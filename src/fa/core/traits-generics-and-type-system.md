# 3. Traitها، Generics و Type System

---

## trait چیست و چه تفاوتی با interface دارد؟

### پاسخ بلند
`trait` در Rust مجموعه‌ای از قراردادها (متدها، روش‌ها و در صورت نیاز associated types یا consts) است که انواع می‌توانند آن را پیاده‌سازی کنند. تفاوت‌های کلیدی با مفهوم رایج `interface` در زبان‌های شیءگرا:

- **associated types و generics:** Traits می‌توانند associated types و generic parameters داشته باشند که الگوهای قوی‌تری برای abstraction فراهم می‌کند.
- **default method implementations:** Traits می‌توانند پیاده‌سازی پیش‌فرض داشته باشند.
- **coherence / orphan rules:** قوانین پیاده‌سازی در Rust سختگیرانه‌تر است (مثلاً orphan rule) تا تداخل implها و ناسازگاری crateها جلوگیری شود.
- **دو نوع dispatch:** Traits هم می‌توانند به‌صورت static (monomorphization) و هم dynamic (trait objects) استفاده شوند؛ در برخی زبان‌های دارای interface فقط dynamic dispatch مرسوم است.
- **بدون وراثت داده‌ای:** Traits رفتار را توصیف می‌کنند، نه ساختار داده‌ای یا فیلدها.

### پاسخ کوتاه
`trait` قرارداد رفتاری است؛ قوی‌تر از یک interface ساده است (associated types، default impl، انعطاف در dispatch) و قوانین coherence خاص Rust دارد.

---

## تفاوت static dispatch و dynamic dispatch چیست؟

### پاسخ بلند
- **Static dispatch (مونومورفی‌سازی):** کامپایلر برای هر نوعی که trait را پیاده‌سازی کرده، یک نسخهٔ مجزا از کد تولید می‌کند (`generic`/`impl Trait` در پارامترها). نتیجه: سرعت بالا (همانند inline) و بهینه‌سازی کامل در زمان کامپایل، اما اندازهٔ باینری ممکن است افزایش یابد.
- **Dynamic dispatch (trait object):** با استفاده از `dyn Trait` فراخوانی‌ها از طریق یک vtable انجام می‌شود؛ کدِ واحدی وجود دارد و انتخاب متد در زمان اجرا انجام می‌شود. مزیت: امکان نگهداری انواع مختلف در یک کالکشن واحد و کاهش اندازهٔ باینری؛ هزینه: اندک overhead اجرای vtable و از دست رفتن برخی بهینه‌سازی‌های زمان کامپایل.

### پاسخ کوتاه
Static = مونومورفی‌سازی زمان کامپایل (سریع‌تر، بدون vtable). Dynamic = trait objects با vtable در زمان اجرا (انعطاف بیشتر، هزینهٔ اندک runtime).

---

## `dyn Trait` چه زمانی استفاده می‌شود؟

### پاسخ بلند
از `dyn Trait` برای **polymorphism دیرهنگام (runtime polymorphism)** استفاده کنید وقتی نیاز دارید:
- مجموعه‌ای از مقادیر با انواع متفاوت اما رفتار یکسان را در یک کانتینر نگهداری کنید (مثلاً `Vec<Box<dyn Draw>>`).
- نوع دقیق پیاده‌سازی را در زمان کامپایل ندانید یا مخفی نگه دارید (ABI abstraction یا plugin-like architecture).  
  پیش‌نیاز: trait باید **object-safe** باشد (قوانینی مثل نداشتن متدهای generic در سیگنیچر و بازنگرداندن `Self` به‌صورت غیرقابل‌خبری). معمولاً از `Box<dyn Trait>` یا `&dyn Trait` استفاده می‌شود.

### پاسخ کوتاه
وقتی به polymorphism در runtime نیاز دارید — مثلاً heterogeneous collections یا پنهان‌سازی نوع — از `dyn Trait` استفاده کنید و مطمئن شوید trait object-safe است.

---

## تفاوت `impl Trait` در return type و argument چیست؟

### پاسخ بلند
- **`impl Trait` در موقعیت return:** نشان‌دهندهٔ یک نوع **مجهول اما یکتا** است که پیاده‌سازی مشخصی دارد اما برای caller مخفی است. این مناسب است وقتی می‌خواهید نوع دقیق را پنهان کنید ولی guarantee کنید که یک نوع واحد در همهٔ فراخوانی‌ها برگردانده می‌شود. مثال: `fn foo() -> impl Iterator<Item = i32>`.
- **`impl Trait` در پارامتر (argument):** معادل syntactic sugar برای یک generic است؛ یعنی `fn f(x: impl Trait)` معادل `fn f<T: Trait>(x: T)` است. تفاوت عملی: در signature با `impl Trait` شما یک پارامتر generic اما بی‌نام دارید؛ در return موقعیت، `impl Trait` نوع را برای caller مبهم نگه می‌دارد (opaque type).  
  نکته: `impl Trait` در آرگومان نمی‌تواند چندین آرگومان را به یک نوع واحد الزام کند (برای آن باید generic type parameter صریح تعریف کنید).

### پاسخ کوتاه
در return: `impl Trait` یک نوع مخفی و یکتا برمی‌گرداند (opaque). در argument: shorthand برای یک generic parameter (معادل `T: Trait`).

---

## trait bound چیست و چگونه چند trait را constrain می‌کنید؟

### پاسخ بلند
**Trait bound** قیدی است که مشخص می‌کند یک type parameter باید چه traitهایی را پیاده‌سازی کند، مثال: `fn f<T: Read + Send>(t: T) {}`. می‌توانید از syntaxهای زیر استفاده کنید:
- مختصر: `T: Trait1 + Trait2`
- با where clause برای خوانایی و حالات پیچیده:
  ```rust
  fn f<T, U>(t: T, u: U)
  where
      T: Iterator<Item = U>,
      U: Display + Clone,
  { }
