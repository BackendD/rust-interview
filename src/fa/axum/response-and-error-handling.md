# 3. Response و Error Handling

---

## چه typeهایی می‌توانند response باشند؟

### پاسخ بلند
هر نوعی که `IntoResponse` را پیاده‌سازی کرده باشد می‌تواند از یک handler برگردد. Axum برای انواع رایج (`String`, `&'static str`, `StatusCode`, `http::Response<Body>`, `Json<T>`، tupleهایی مثل `(StatusCode, HeaderMap, body)` و غیره) پیاده‌سازی آماده دارد، بنابراین معمولاً می‌توانید مستقیم یک مقدار ساده یا یک tuple برگردانید و Axum آن را به `Response` تبدیل می‌کند. اگر نوع دلخواه شما این trait را نداشته باشد، می‌توانید آن را خودتان پیاده‌سازی کنید تا مستقیماً از handler قابل برگشت باشد. :contentReference[oaicite:0]{index=0}

### پاسخ کوتاه
هر چیزی که `IntoResponse` را پیاده‌سازی کند — Axum برای بسیاری از انواع رایج پیاده‌سازی آماده دارد. :contentReference[oaicite:1]{index=1}

---

## trait `IntoResponse` چگونه کار می‌کند؟

### پاسخ بلند
`IntoResponse` متدی `into_response(self) -> Response` فراهم می‌کند که نوع حامل را به یک `http::Response<Body>` تبدیل می‌کند. وقتی handler `impl IntoResponse` یا نوعی که `IntoResponse` را پیاده‌سازی کرده برمی‌گرداند، Axum آن را فراخوانی می‌کند تا پاسخ HTTP واقعی ساخته شود. این سازوکار امکان انعطاف‌پذیری بالا (مثلاً بازگرداندن tuple برای تعیین status/headers/body) و یکپارچگی با انواع کاربردی مانند `Json<T>` را فراهم می‌سازد. اغلب لازم نیست خودتان `IntoResponse` را بنویسید مگر برای error typeهای سفارشی یا رفتارهای ویژه. :contentReference[oaicite:2]{index=2}

### پاسخ کوتاه
این trait مقدار را به یک `http::Response` تبدیل می‌کند؛ Axum این تبدیل را برای هر خروجی handler انجام می‌دهد. :contentReference[oaicite:3]{index=3}

---

## چگونه error مرکزی (global error handling) پیاده می‌کنید؟

### پاسخ بلند
الگوهای معمول برای هندلینگ سراسری در Axum:

1. **Middleware / Tower layers** — از `tower::ServiceBuilder` و لِیِرهایی مثل `HandleErrorLayer` برای wrap کردن سرویس استفاده کنید تا خطاهای لایه‌ها (مثل timeout یا fallible middleware) را بگیرید و به `IntoResponse` تبدیل کنید. `HandleErrorLayer` برای تبدیل `BoxError` یا خطاهای لایه به پاسخ HTTP کاربردی است. :contentReference[oaicite:4]{index=4}

2. **خطاهای handler به‌صورت نوعی که `IntoResponse` را پیاده‌سازی می‌کند** — هر error type سفارشی را طوری طراحی کنید که خودش `IntoResponse` را پیاده‌سازی کند؛ سپس هر کجا `Result<T, E>` برگردانید، Axum در صورت `Err(e)` آن را به پاسخ مناسب تبدیل می‌کند. این الگو خطاها را در لایه application متحد می‌سازد (مثلاً تبدیل enum خطاها به JSON+status). :contentReference[oaicite:5]{index=5}

3. **ترکیب هر دو** — middleware برای موارد کلی (logging، تبدیل errorهای لایه‌ای، timeouts) و `IntoResponse` برای mapping دقیق انواع خطای دامنه به status/body.

نکته عملی: middlewareها معمولاً خطاها را به شکل `Result` یا `BoxError` برمی‌گردانند؛ از `HandleErrorLayer` یا یک layer سفارشی استفاده کنید تا log/metric بگیرید و یک `Response` یکنواخت بسازید. :contentReference[oaicite:6]{index=6}

### پاسخ کوتاه
از ترکیب Tower layers (مثلاً `HandleErrorLayer`) برای خطاهای لایه‌ای و پیاده‌سازی `IntoResponse` روی error typeهای دامنه برای mapping دقیق خطا به status/body استفاده کنید. :contentReference[oaicite:7]{index=7}

---

## تفاوت برگرداندن `Result<T, E>` و `impl IntoResponse` چیست؟

### پاسخ بلند
- `impl IntoResponse`: تابع فقط می‌تواند **یک** نوع بازگشتی را داشته باشد؛ اگر بخواهید در شاخه‌های مختلف انواع متفاوتی برگردانید، `impl Trait` محدودیت دارد و گاهی لازم است به صورت صریح یک `Response` یا یک common enum برگردانید. علاوه بر این، `impl IntoResponse` مناسب وقتی است که موفقیت و خطا در یک نوع واحد نمایش داده شوند. :contentReference[oaicite:8]{index=8}

- `Result<T, E>`: بیان صریحِ احتمال خطاست. Axum برای `Result` نیز پیاده‌سازی دارد، اما هر دو شاخه (`T` و `E`) باید `IntoResponse` را پیاده‌سازی کنند تا تبدیل خودکار به `Response` انجام شود. بنابراین `Result` به شما اجازه می‌دهد خطا را جدا نگه دارید و آن را با یک `IntoResponse` مناسب هندل کنید. اگر `E` انواع مختلفی داشته باشد، معمولاً یک enum خطا می‌سازید و برای آن `IntoResponse` پیاده‌سازی می‌کنید. :contentReference[oaicite:9]{index=9}

### پاسخ کوتاه
`impl IntoResponse` برمی‌گرداند یک نوع قابل تبدیل به پاسخ؛ `Result<T, E>` صراحتاً خطا را نشان می‌دهد — ولی برای تبدیل خودکار، هم `T` و هم `E` باید `IntoResponse` را پیاده‌سازی کنند. :contentReference[oaicite:10]{index=10}

---

## چگونه HTTP status code و body را همزمان کنترل می‌کنید؟

### پاسخ بلند
چند راه معمول:
- برگرداندن یک tuple مثل `(StatusCode, impl IntoResponse)` یا `(StatusCode, HeaderMap, body)` — Axum این ترکیب‌ها را می‌شناسد و آن‌ها را به `Response` تبدیل می‌کند. این ساده‌ترین راه برای تعیین هم‌زمان status و body است. :contentReference[oaicite:11]{index=11}

- ساختن و برگرداندن یک `http::Response<Body>` کامل وقتی نیاز به کنترل دقیق headers، extensions یا body streaming دارید. :contentReference[oaicite:12]{index=12}

- در صورت استفاده از `Result<T, E>`، نوع `E` را طوری پیاده‌سازی کنید که هنگام `Err` یک status مناسب و body توضیحی (مثلاً JSON با فیلدهای خطا) تولید کند (یعنی `IntoResponse` برای `E`). :contentReference[oaicite:13]{index=13}

### پاسخ کوتاه
برای حالت‌های ساده از tuple مثل `(StatusCode, body)` استفاده کنید؛ برای کنترل کامل از `http::Response`؛ یا error type را طوری `IntoResponse` کنید که status+body مناسب تولید کند. :contentReference[oaicite:14]{index=14}

---

## بهترین الگو برای error type در Axum چیست؟

### پاسخ بلند
الگوی رایج و مؤثر:

1. **یک `enum` خطا در سطح application** که تمام خطاهای منطقی را پوشش می‌دهد (مثلاً `enum AppError { NotFound, Validation(ValidationError), Auth(AuthError), Internal(anyhow::Error) }`).
2. از crates مثل `thiserror` برای derive کردن `Error` و متدهای مفید استفاده کنید.
3. برای `AppError` یک `impl IntoResponse` بنویسید که status مناسب را تعیین و body (معمولاً `Json<{ code, message, details }>` ) را برمی‌گرداند. این کار موجب می‌شود هر handler بتواند `Result<T, AppError>` برگرداند و mapping سراسری خطاها ساده و یکنواخت باشد.
4. از logging/observability در نقطهٔ central (مثلاً در `IntoResponse` برای `Internal` یا در یک middleware) استفاده کنید تا stack/context لاگ شود اما پاسخ به کاربر امن بماند.

این الگو ترکیب type-safety، خوانایی و امکان unit/integration testing را فراهم می‌کند و با اکوسیستم Axum/Tower به‌خوبی کار می‌کند. :contentReference[oaicite:15]{index=15}

### پاسخ کوتاه
یک `enum` مرکزی برای خطاها + `thiserror` برای derive + یک `impl IntoResponse` برای mapping به `StatusCode` و JSON body. لاگینگ و تبدیل خطاهای داخلی در middleware یا داخل `IntoResponse` انجام شود. :contentReference[oaicite:16]{index=16}

---
