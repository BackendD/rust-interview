# 1. مبانی Axum

---

## Axum چیست و چه تفاوتی با Actix / Warp دارد؟

### پاسخ بلند
Axum یک وب‌فریم‌ورک مدرن برای Rust است که روی Tokio و Hyper ساخته شده و به‌طور مستقیم از اکوسیستم Tower استفاده می‌کند. تمرکز اصلی Axum بر **type-safety**، **composability** و استفاده‌ی حداکثری از type system Rust است.

در مقایسه:
- **Actix Web** معماری و runtime اختصاصی خود را دارد و middlewareها و مدل اجرایی آن tightly-coupled هستند.
- **Warp** از مدل filter-based استفاده می‌کند که اگرچه type-safe است، اما در APIهای بزرگ و پیچیده به‌سرعت خوانایی و maintainability خود را از دست می‌دهد.
- **Axum** با تکیه بر Router، extractors صریح و Tower middlewareها، ساختار واضح‌تر و قابل‌گسترش‌تری برای APIهای production-grade فراهم می‌کند.

Axum عمداً یک abstraction نازک است و تلاش نمی‌کند همه‌چیز را خودش پیاده‌سازی کند.

### پاسخ کوتاه
Axum فریم‌ورکی type-safe و مدولار روی Hyper/Tokio و Tower است؛ برخلاف Actix runtime اختصاصی ندارد و برخلاف Warp از مدل filter استفاده نمی‌کند.

---

## چرا Axum بر پایه‌ی Tower ساخته شده است؟

### پاسخ بلند
Tower یک abstraction استاندارد برای سرویس‌های async در Rust است که حول trait اصلی `Service` و مفهوم `Layer` (middleware) شکل گرفته است. Axum با ساختن خود بر پایه‌ی Tower:
- از middlewareهای بالغ و battle-tested استفاده می‌کند
- نیاز به reinvent کردن middleware stack را حذف می‌کند
- امکان ترکیب ساده‌ی concerns مثل timeout، retry، rate limit، tracing و auth را فراهم می‌کند

در واقع Axum یک adapter انسانی‌پسند برای HTTP routing روی Tower است.

### پاسخ کوتاه
برای استفاده از abstraction استاندارد `Service` و اکوسیستم آماده‌ی middlewareهای Tower.

---

## نقش `Router` در Axum چیست؟

### پاسخ بلند
`Router` هسته‌ی اصلی تعریف API در Axum است. این ساختار مسئول:
- نگاشت مسیر و HTTP method به handler
- ترکیب چند Router به‌صورت سلسله‌مراتبی
- اعمال middleware در سطوح مختلف
- مدیریت state اشتراکی بین handlerها

Router نه‌تنها مسیرها را نگه می‌دارد، بلکه boundary اصلی طراحی API و modularization در Axum محسوب می‌شود.

### پاسخ کوتاه
Router وظیفه‌ی اتصال مسیرها به handlerها، ترکیب sub-routerها و اعمال middleware و state را دارد.

---

## تفاوت `route` و `nest` چیست؟

### پاسخ بلند
- `route` برای ثبت مستقیم یک مسیر مشخص و اتصال آن به handler یا service استفاده می‌شود.
- `nest` برای mount کردن یک Router دیگر زیر یک prefix مشخص به کار می‌رود.

در `nest`، prefix از مسیر حذف می‌شود و Router داخلی بدون آگاهی از مسیر بالادستی کار می‌کند. این ویژگی برای ماژولار کردن API (مثلاً `/api/v1/users`) حیاتی است.

### پاسخ کوتاه
`route` یک مسیر منفرد را ثبت می‌کند؛ `nest` یک Router کامل را زیر یک prefix قرار می‌دهد.

---

## چرا handlerها در Axum `async` هستند؟

### پاسخ بلند
Axum بر مبنای I/O غیرهمزمان طراحی شده و handlerها مستقیماً در مسیر اجرای async runtime (Tokio) قرار می‌گیرند. `async` بودن handlerها امکان:
- انجام I/O بدون بلوکه کردن thread
- مقیاس‌پذیری بالا با تعداد thread کم
- استفاده‌ی مستقیم از async database clients و HTTP clients

را فراهم می‌کند. این طراحی برای سرویس‌های high-concurrency ضروری است.

### پاسخ کوتاه
چون Axum بر پایه‌ی async I/O ساخته شده و handlerها باید non-blocking و scalable باشند.

---

## چرا امضای handler باید type-safe باشد؟

### پاسخ بلند
Axum از extractors مبتنی بر type system استفاده می‌کند؛ یعنی ورودی‌های handler (path params، query، headers، body، state) همگی type-safe هستند و در زمان کامپایل بررسی می‌شوند.

این رویکرد:
- خطاهای parsing و missing data را به compile-time نزدیک می‌کند
- قرارداد ورودی handler را شفاف و self-documenting می‌سازد
- تست‌پذیری و refactorپذیری را افزایش می‌دهد

Handler در Axum عملاً یک function با وابستگی‌های صریح است، نه یک تابع با دسترسی مبهم به request.

### پاسخ کوتاه
برای اینکه استخراج داده از request در زمان کامپایل بررسی شود و خطاهای runtime حذف شوند.

---
