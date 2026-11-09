# فصل ۸ — ماژول‌ها، مدیریت خطا و دیباگ (پایان بخش JS)

> 🎯 **هدف:** کد را از یک فایل به پروژه‌ی واقعی برسانیم: ماژول‌های ESM (import/export)، استراتژی مدیریت خطا، و ابزارهای دیباگ حرفه‌ای. آخر این فصل، آماده‌ی ورود به TypeScript هستی.

---

## ۸.۱ — ماژول‌ها (ESM): تقسیم کد به فایل‌ها

هر فایل = یک ماژول با scope خودش. `export` = عرضه، `import` = مصرف:

```js
// ── math.js ──
export function add(a, b) { return a + b; }        // export نام‌دار
export const PI = 3.14159;

export default function calculate(x, y) {          // export پیش‌فرض — یکی در هر فایل
  return { sum: add(x, y), area: PI * x * y };
}

// ── app.js ──
import calculate, { add, PI } from "./math.js";    // default بدون {}، نام‌دار با {}
import { add as sumFn } from "./math.js";          // تغییر نام
import * as math from "./math.js";                 // همه به صورت namespace

add(2, 3);
math.PI;
```

- مسیر نسبی: `./` هم‌پوشه، `../` والد — و پسوند `.js` را بنویس (در Node ESM)
- HTML: `<script type="module" src="app.js"></script>`
- یک `default` در هر فایل (چیز اصلی) + هر تعداد export نام‌دار

> 💡 در Next.js همین است — فقط مسیرها با alias (`@/lib/...`) خواناتر می‌شوند و import ها در build مدیریت می‌شوند.

## ۸.۲ — مدیریت خطا: استراتژی حرفه‌ای

```js
// throw — پرتاب خطای سفارشی:
function divide(a, b) {
  if (b === 0) throw new Error("تقسیم بر صفر ممنوع");
  return a / b;
}

// try/catch/finally:
try {
  const r = divide(10, 0);
} catch (err) {
  console.error("خطا:", err.message);
} finally {
  console.log("همیشه اجرا می‌شود");   // پاکسازی
}

// خطاهای سفارشی با کلاس (برای تفکیک نوع خطا):
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}
```

**استراتژی سه‌لایه:**

| لایه | کار |
|---|---|
| **جلوگیری** | validation ورودی (فصل‌های بعدی: zod در TS) |
| **گرفتن محل** | try/catch دقیق در مرزها (fetch، فرم) |
| **گزارش** | پیام تمیز به کاربر + لاگ کامل برای تو |

> ⚠️ هرگز `catch (e) {}` خالی — خطا را می‌بلعی و باگ نامرئی می‌شود. حداقل لاگ کن؛ بهتر: دوباره throw یا پیام معنادار.

## ۸.۳ — دیباگ: ابزارهای واقعی

```js
// console خانواده کامل:
console.log("عادی");
console.warn("هشدار");
console.error("خطا");
console.table(users);            // آرایه آبجکت‌ها به شکل جدول! ⭐
console.group("جزئیات"); console.log("..."); console.groupEnd();
console.time("حلقه"); ...; console.timeEnd("حلقه");   // زمان‌گیری

// debugger — نقطه توقف در DevTools:
function tricky(data) {
  debugger;         // اجرا اینجا متوقف می‌شود (فقط وقتی DevTools باز است)
  return data.map(process);
}

// خطاها را جدی بگیر: استک‌ترِیس را از بالا به پایین بخوان —
// اولین فایلِ خودِ تو = محل شروع تحقیق.
```

در DevTools (F12): تب **Sources** → breakpoint با کلیک روی شماره خط → Watch متغیرها → Step (F10/F11). با `node --inspect app.js` همین ابزار در VS Code.

---

## ✅ جمع‌بندی فصل — پایان بخش JavaScript!

- ESM: `export/default` + `import` — هر فایل یک ماژول
- خطا: throw سفارشی + try/catch/finally + کلاس خطای اختصاصی + هرگز catch خالی
- دیباگ: console.table/time/debugger/DevTools Sources + خواندن stack trace
- تو حالا **JS کامل** بلدی: متغیر تا async، DOM تا ماژول — TS فقط لایه‌ی امنیتی روی همین است!

## 📝 تمرین فصل ۸

1. پروژه‌ات را سه فایلی کن: `math.js` (توابع فصل ۲)، `format.js` (فصل ۵) و `app.js` که import کند و استفاده کند.
2. کلاس `HttpError extends Error` با فیلد `status` بنویس؛ تابع fetchی بنویس که روی پاسخ‌های بد آن را throw کند و مصرف‌کننده بر اساس `instanceof HttpError` پیام متفاوت بدهد.
3. در DevTools مرورگر: `console.table` را روی آرایه‌ای از ۳ آبجکت امتحان کن و یک breakpoint واقعی بگذار و متغیر را Watch کن.
4. باگ‌گیری: این کد چرا همیشه «خطا» می‌دهد؟
```js
try {
  const data = await fetch(url).then(r => r.json());
  console.log(data);
} catch { console.log("خطا") }
```
(راهنما: `await` داخل try بدون تابع async؟)

<details><summary>جواب تمرین ۴</summary>

`await` فقط در تابع async مجاز است — این کد SyntaxError می‌دهد (نه «خطا» را چاپ می‌کند). درست: تابع را async کن:

```js
async function load(url) {
  try {
    const data = await fetch(url).then(r => r.json());
    console.log(data);
  } catch (err) {
    console.log("خطا:", err.message);
  }
}
```
</details>

🎓 **JavaScript تمام شد — وارد دنیای TypeScript می‌شویم:** همان JS، ولی با سپر تایپ!
