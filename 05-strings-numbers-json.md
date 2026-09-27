# فصل ۵ — رشته‌ها، اعداد، Date و JSON

> 🎯 **هدف:** ابزارهای کار با داده: متدهای پرکاربرد String، تله‌های Number (و Intl برای فارسی!)، Date مدرن و JSON — زبان تبادل داده‌ی همه API ها.

---

## ۵.۱ — رشته‌ها (String)

```js
const s = "  Hello JavaScript  ";

// پاکسازی و بررسی:
s.trim();                  // "Hello JavaScript" — حذف فاصله‌های دو طرف
s.trim().length;           // 17
s.toUpperCase();           // همه بزرگ
"ali@dev.ir".includes("@") // true — شامل است؟
s.startsWith("  He");      // true
"report.pdf".endsWith(".pdf"); // true

// برش و جستجو:
"2026-09-13".split("-");      // ["2026","09","13"] — split به آرایه
["a","b"].join(", ");         // "a, b" — برعکس split
"hello world".slice(0, 5);    // "hello" (برش با ایندکس)
"hello".replace("l", "L");    // "heLlo" — فقط اولین!
"hello".replaceAll("l", "L"); // "heLLo" — همه
"a,b,c".replaceAll(",", " | ");

// ترکیب با template (فصل ۱) و تبدیل:
Number("42");       // 42
String(42);         // "42"
parseInt("42px");   // 42 (تا جایی که عدد است)
parseFloat("3.14x");// 3.14
```

> ⚠️ رشته‌ها در JS **immutable** اند: `s.trim()` رشته‌ی جدید می‌دهد؛ `s` بدون تغییر می‌ماند! پس نتیجه را ذخیره کن: `const clean = s.trim();`

## ۵.۲ — اعداد (Number) و Intl: نمایش فارسی ⭐

```js
// دقت اعشاری — تله مشهور!
0.1 + 0.2               // 0.30000000000000004
+(0.1 + 0.2).toFixed(2) // 0.3   ← برای نمایش؛ برای پول واقعی: عدد صحیح نگه دار!

// تبدیل‌ها:
parseInt("42");         // 42
Number("42");           // 42
Number("abc");          // NaN ← Not a Number (نوعش number است!)
Number.isNaN(NaN);      // true — چک NaN فقط با isNaN

// Intl — فرمت‌بندی به زبان/منطقه (جواهر پنهان JS!) ⭐
const price = 1250000;
new Intl.NumberFormat("fa-IR").format(price);        // "۱٬۲۵۰٬۰۰۰"
new Intl.NumberFormat("fa-IR", {
  style: "currency", currency: "IRR",
}).format(price);                                    // "۱٬۲۵۰٬۰۰۰ ریال"

new Intl.DateTimeFormat("fa-IR").format(new Date()); // "۱۴۰۵/۶/۲۲"
new Intl.ListFormat("fa-IR").format(["علی", "سارا"]); // "علی و سارا"
```

(دقیقاً همین `toLocaleString("fa-IR")` را در پروژه Next خودت هم دیدی — Intl زیرش است.)

## ۵.۳ — Date: کار با تاریخ

```js
const now = new Date();               // الان
const bd = new Date("1995-06-15");    // از رشته ISO

now.getFullYear();  now.getMonth();   // ⚠️ 0 تا 11! ژانویه = 0
now.getDate();      now.getDay();     // روز ماه / روز هفته (0=یکشنبه)

// محاسبه فاصله (با میلی‌ثانیه):
const diffMs = now - bd;              // JS تاریخ‌ها را منهای هم می‌کند!
const diffDays = Math.floor(diffMs / 86400000);

// فرمت فارسی:
new Intl.DateTimeFormat("fa-IR", {
  dateStyle: "full",
  timeStyle: "short",
}).format(now);   // "یکشنبه ۲۲ شهریور ۱۴۰۵، ۱۹:۴۵"

// setTimeout/sleep با Date (پایه async فصل ۷):
const start = Date.now();
Date.now() - start;   // چند ms گذشت
```

> ⚠️ در اپ واقعی (منطقه‌زمانی‌ها و محاسبات پیچیده)، از کتابخانه `date-fns` یا `dayjs` استفاده کن — Date بومی برای پایه کافی است ولی تاریخ در production بغرنج است.

## ۵.۴ — JSON: زبان مشترک API ها

```js
const user = { name: "علی", age: 25, skills: ["JS", "TS"] };

// آبجکت → رشته (برای ارسال/ذخیره):
const json = JSON.stringify(user);
// '{"name":"علی","age":25,"skills":["JS","TS"]}'

// با فرمت خوانا (برای دیباگ):
JSON.stringify(user, null, 2);

// رشته → آبجکت (پاسخ API):
const parsed = JSON.parse(json);
parsed.name;    // "علی"

// ⚠️ JSON.parse روی رشته نامعتبر throw می‌کند — همیشه try/catch:
try {
  const data = JSON.parse(rawString);
} catch {
  console.error("JSON خراب است");
}
```

قوانین JSON: کلیدها حتماً با `"..."` دوتایی؛ کامنت ندارد؛ تابع/undefined نمی‌پذیرد. (چرخه کامل: در دوره‌ی Prisma `JSON.stringify` می‌رفت سمت API و `JSON.parse` برمی‌گشت — حالا زیرش را می‌دانی!)

---

## ✅ جمع‌بندی فصل

- رشته: trim/split/join/replace/include — immutable؛ نتیجه را ذخیره کن
- اعداد: toFixed برای نمایش، Intl برای فارسی (NumberFormat/DateTimeFormat!)
- Date: getMonth از صفر! اختلاف = میلی‌ثانیه؛ پیچیده شد → dayjs/date-fns
- JSON: stringify/parse — زبان مشترک API؛ parse با try/catch

## 📝 تمرین فصل ۵

1. `slugify(title)`: «مقدمه‌ای بر JS!» → رشته لاتین تمیز با `-` (راهنما: trim، lowercase، replaceAll فاصله به `-`).
2. قیمت `1234567` را به سه شکل چاپ کن: با جداکننده فارسی، با «تومان»، و `toFixed(2)`.
3. سن از تاریخ تولد: `getAge("1995-06-15")` — سال کامل حساب‌شده (نه فقط اختلاف سال!).
4. یک آبجکت بساز، stringify کن، دوباره parse کن و تفاوت دو آبجکت را با `JSON.stringify(a) === JSON.stringify(b)` چک کن (و نکته‌اش را بگو: مقایسه عمیق با این روش شکننده است — ترتیب کلیدها مهم است!).
5. چالش: `formatRelative(days)` — «امروز»، «دیروز»، «۳ روز پیش»، «۳۰ روز پیش».

<details><summary>جواب‌ها</summary>

```js
// ۱
function slugify(title) {
  return title.trim().toLowerCase().replaceAll(/\s+/g, "-");
}
// (regex \s+ = یک یا چند فاصله — فصل ۸ آشناتر می‌شود)

// ۲
new Intl.NumberFormat("fa-IR").format(1234567);           // ۱٬۲۳۴٬۵۶۷
new Intl.NumberFormat("fa-IR").format(1234567) + " تومان";
Number(1234567).toFixed(2);                                // "1234567.00"

// ۳
function getAge(birthDateString) {
  const birth = new Date(birthDateString);
  const today = new Date();
  let age = today.getFullYear() - birth.getFullYear();
  const beforeBirthday =
    today.getMonth() < birth.getMonth() ||
    (today.getMonth() === birth.getMonth() && today.getDate() < birth.getDate());
  if (beforeBirthday) age--;
  return age;
}

// ۵
function formatRelative(days) {
  if (days === 0) return "امروز";
  if (days === 1) return "دیروز";
  if (days === -1) return "فردا";
  return days > 0 ? `${days} روز پیش` : `${-days} روز بعد`;
}
```
</details>

➡️ **فصل بعد:** DOM و رویدادها — JS را به صفحه وصل می‌کنیم!
