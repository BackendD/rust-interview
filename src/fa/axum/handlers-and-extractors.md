# 2. Handlerها و Extractorها

---

## extractor چیست و چگونه کار می‌کند؟

### پاسخ بلند  
در Axum، **extractor** نوعی است که از یک درخواست HTTP مقدار/اطلاعات خاصی را استخراج می‌کند و آن را به‌عنوان آرگومان به handler تحویل می‌دهد. یک extractor با یکی از traitهای `FromRequestParts` (برای بخش‌های درخواست مثل headers, path, query, extensions) یا `FromRequest` (برای استخراج‌هایی که باید body را مصرف کنند) پیاده‌سازی می‌شود. قبل از فراخوانی handler، Axum هر extractor را اجرا می‌کند و نتیجهٔ موفق را به عنوان آرگومان به handler می‌دهد؛ اگر extraction شکست بخورد، handler اجرا نخواهد شد و یک rejection که قابل تبدیل به `Response` است بازگردانده می‌شود. :contentReference[oaicite:0]{index=0}

### پاسخ کوتاه  
Extractor نوعی است که بخش‌هایی از `Request` را می‌خواند/پارسه می‌کند و به handler به‌صورت پارامتر تحویل می‌دهد؛ پیاده‌سازی آن معمولاً از `FromRequestParts` یا `FromRequest` است. :contentReference[oaicite:1]{index=1}

---

## تفاوت `Json<T>`، `Form<T>`، `Path<T>`، `Query<T>` چیست؟

### پاسخ بلند  
- `Json<T>`: بدنهٔ درخواست را به‌عنوان JSON خوانده و با `serde::Deserialize` به `T` تبدیل می‌کند — **مصرف‌کنندهٔ body** است (پس باید آخرین extractor باشد اگر body مصرفی دیگری نیست).  
- `Form<T>`: بدنهٔ فرم x-www-form-urlencoded را پارس می‌کند و به `T` تبدیل می‌کند (نیز body-consuming).  
- `Path<T>`: پارامترهای مسیر (path params) را از URI استخراج و با `serde` دی‌سریالایز می‌کند؛ معمولاً برای مقادیر شناسه/پارامتر مسیر استفاده می‌شود.  
- `Query<T>`: query string را پارس کرده و به `T` تبدیل می‌کند (برای پارامترهای optional یا pagination و غیره مناسب است).  
هر یک از این‌ها نوعی extractor هستند و رفتار خاص خود را در صورت خطا (مثلاً بدفرمت بودن JSON یا فقدان پارامتر لازم) با یک rejection بازمی‌گردانند. :contentReference[oaicite:2]{index=2}

### پاسخ کوتاه  
`Json`/`Form` بدنه را مصرف و پارس می‌کنند؛ `Path` پارامترهای مسیر و `Query` پارامترهای query string را استخراج می‌کند. :contentReference[oaicite:3]{index=3}

---

## ترتیب اجرای extractorها چگونه است؟

### پاسخ بلند  
Extractorها **همیشه به ترتیب پارامترهای تابع (از چپ به راست)** اجرا می‌شوند. دلیل مهم این قاعده این است که بدنهٔ درخواست یک استریم async است که فقط یک‌بار قابل مصرف است؛ بنابراین extractorهایی که body را مصرف می‌کنند باید آخرین پارامتر باشند و Axum این محدودیت را از طریق ترتیب اجرا و نوع traitها اعمال می‌کند. به‌طور خلاصه: ترتیب پارامترها اهمیت دارد و extractorهای مصرف‌کنندهٔ body را آخر قرار دهید. :contentReference[oaicite:4]{index=4}

### پاسخ کوتاه  
Axum extractors را از چپ به راست (ترتیب پارامترهای handler) اجرا می‌کند؛ extractorهایی که body را مصرف می‌کنند باید آخر باشند. :contentReference[oaicite:5]{index=5}

---

## اگر یک extractor fail شود چه اتفاقی می‌افتد؟

### پاسخ بلند  
وقتی یک extractor شکست می‌خورد، آن extractor یک نوع **rejection** برمی‌گرداند (نوع مرتبط با extractor که `IntoResponse` را پیاده‌سازی می‌کند). در این حالت:
- اجرای سایر extractorها/handler متوقف می‌شود.  
- Router پاسخ HTTP را با تبدیل rejection به `Response` ارسال می‌کند (مثلاً 400 برای JSON نامعتبر یا 404 برای مسیر نامناسب، بسته به نوع rejection).  
برای دسترسی به خطای درون یک extractor یا سفارشی‌سازی پاسخ خطا می‌توان extractor را به صورت `Result<T, T::Rejection>` در آرگومان handler گرفت یا از ابزارهایی مانند `WithRejection` / تعریف extractor سفارشی استفاده کرد. :contentReference[oaicite:6]{index=6}

### پاسخ کوتاه  
در صورت شکست، extractor یک rejection بازمی‌گرداند و handler اجرا نمی‌شود — آن rejection به `Response` تبدیل و به کاربر برگردانده می‌شود. :contentReference[oaicite:7]{index=7}

---

## چگونه extractor سفارشی می‌نویسید؟

### پاسخ بلند  
دو مسیر اصلی وجود دارد:
1. اگر نیاز فقط به خواندن بخش‌های request (headers, path, query, extensions) دارید، `FromRequestParts<S>` را پیاده‌سازی کنید.  
2. اگر باید body را مصرف کنید، `FromRequest<S>` را پیاده‌سازی کنید.  
هر پیاده‌سازی باید یک نوع `Rejection` تعیین کند (که `IntoResponse` را پیاده‌سازی کند). معمول‌ترین الگو این است که در `from_request_parts`/`from_request` منطق استخراج را انجام دهید، در صورت موفقیت مقدار موردنظر را بسازید و در صورت شکست یک rejection مناسب برگردانید. برای سفارشی‌سازی خطاها می‌توانید از `WithRejection` یا از آرگومان `Result<T, T::Rejection>` در handler استفاده کنید. مثال کوتاه (اسکلت):

```rust
use axum::extract::{FromRequestParts, RequestParts};
use http::request::Parts;

struct MyExtractor(/* fields */);

#[axum::async_trait]
impl<S> FromRequestParts<S> for MyExtractor
where
    S: Send + Sync,
{
    type Rejection = MyRejection;

    async fn from_request_parts(parts: &mut Parts, state: &S) -> Result<Self, Self::Rejection> {
        // خواندن header/extension و ساخت extractor یا بازگرداندن MyRejection
    }
}
