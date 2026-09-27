# فصل ۱ — شروع: اجرای کد، متغیرها و انواع داده

> 🎯 **هدف:** نصب Node، سه راه اجرای کد JS، متغیرها (let/const — و چرا var نه)، ۷ نوع داده پایه، عملگرها و template literal. آخر فصل اولین برنامه‌های واقعی را اجرا کرده‌ای.

---

## ۱.۱ — نصب Node و سه راه اجرا

**Node.js** = موتور JS بیرون از مرورگر (دوره‌های قبلی نصبش کرده‌ای — `node -v`).

سه راه اجرای کد JS:

```bash
node                  # ۱) REPL: تعاملی — بنویس، فوری جواب بگیر
node app.js           # ۲) اجرای فایل
# ۳) کنسول مرورگر (F12 → Console) — برای کدهای مرتبط با صفحه
```

زمین تمرین: `mkdir js-ts-playground && cd js-ts-playground` — فایل‌های هر فصل اینجا.

## ۱.۲ — متغیرها: جعبه‌های داده

```js
const name = "علی";      // const: ثابت — بازتخصیص ممنوع
let score = 20;          // let: قابل تغییر
score = 25;              // ✅
// name = "رضا";         // ❌ TypeError: Assignment to constant variable

var old = "فرار کن";      // مدل قدیمی — فصل ۳ توضیح می‌دهد چرا نمی‌خواهیم
```

قواعد اسم‌گذاری: حروف/عدد/`_`/`$`، شروع با عدد ممنوع، camelCase (`userName`)، **معنادار** — `data1` دشمن آینده‌ی توست.

> 💡 قاعده انتخاب: پیش‌فرض `const`. اگر جایی نیاز به تغییر دیدی، همان خط را به `let` عوض کن — کد را از تغییرهای بی‌دلیل ایمن نگه می‌دارد.

## ۱.۳ — انواع داده پایه (۷ نوع primitive + آبجکت)

```js
// ── ۷ primitive ──
const text    = "سلام";        // String — رشته
const age     = 25;            // Number — عدد (اعشار هم همین)
const ok      = true;          // Boolean — درست/غلط
const nothing = undefined;     // مقدار داده نشده
const empty   = null;          // عمداً خالی
const big     = 9007199254740993n; // BigInt — اعداد خیلی بزرگ
const id      = Symbol("id");  // Symbol — شناسه یکتا (کمتر لازم می‌شود)

// ── آبجکت (همه چیزهای پیچیده از آن ساخته می‌شوند) ──
const user = { name: "علی", age: 25 };   // Object
```

چک نوع با `typeof`:

```js
typeof "سلام"     // "string"
typeof 25         // "number"
typeof true       // "boolean"
typeof undefined  // "undefined"
typeof null       // "object"  ← ⚠️ باگ تاریخی JS! null آبجکت نیست
typeof {}         // "object"
typeof [1,2]      // "object"  ← آرایه هم آبجکت است
typeof (() => {}) // "function"
```

> ⚠️ دو تله کلاسیک: (۱) `typeof null` می‌گوید "object" — باگ ۳۰ ساله که برای سازگاری نگاه داشته شده؛ (۲) آرایه typeof اش object است — تشخیص آرایه با `Array.isArray(x)`.

## ۱.۴ — عملگرها

```js
// حسابی
10 + 3   // 13
10 / 3   // 3.333...   (تفریق: -، ضرب: *)
10 % 3   // 1          ← باقیمانده: تشخیص زوج/فرد (n % 2 === 0)
2 ** 10  // 1024       ← توان

// مقایسه — همیشه === نه ==
5 === 5    // true   (مقدار + نوع)
5 == "5"   // true   ⚠️ تبدیل نوع خودکار — منبع باگ! از == استفاده نکن
5 !== "5"  // true

// منطقی
true && false  // false  (و)
true || false  // true   (یا)
!true          // false  (نقیض)

// nullish — مدرن و مهم:
const port = userInput ?? 3000;   // فقط اگر null/undefined بود ۳۰۰۰
// برخلاف || که 0 و "" را هم می‌گیرد!
```

> 🚨 سه‌گانه‌ی باگ‌ساز مبتدی‌ها: `=` (انتساب) با `===` (مقایسه) اشتباه گرفتن؛ استفاده از `==`؛ و جمع رشته با عدد: `"5" + 3` می‌شود `"53"`! (JS رشته برنده است.) برای تبدیل واقعی: `Number("5") + 3` → 8.

## ۱.۵ — Template Literal: ساخت رشته مثل حرفه‌ای‌ها

```js
const user = "علی";
const score = 95;

// ❌ قدیمی — پر از + و خطا
console.log("کاربر " + user + " نمره " + score + " گرفت");

// ✅ مدرن — backtick ها (`) + ${}
console.log(`کاربر ${user} نمره ${score} گرفت`);
console.log(`دو در سه برابر: ${2 * 3}`);
console.log(`چند خطی
هم بدون + کار می‌کند`);
```

`${}` هر عبارت JS را می‌گیرد — حتی فراخوانی تابع. از این به بعد همیشه همین.

## ۱.۶ — اولین برنامه واقعی

```js
// grade.js — محاسبه وضعیت نمره
const student = "سارا";
const score = 78;
const passMark = 50;
const passed = score >= passMark;

console.log(`دانشجو: ${student}`);
console.log(`نمره: ${score} — وضعیت: ${passed ? "قبول ✅" : "مردود ❌"}`);
console.log(`درصد: ${Math.round((score / 100) * 100)}٪`);
```

```bash
$ node grade.js
دانشجو: سارا
نمره: 78 — وضعیت: قبول ✅
درصد: 78٪
```

(آن `passed ? "قبول" : "مردود"` = ternary — مقدمه‌ی فصل ۲!)

---

## ✅ جمع‌بندی فصل

- سه راه اجرا: REPL (`node`)، فایل (`node app.js`)، کنسول مرورگر
- متغیر: پیش‌فرض `const`، تغییرپذیر `let`، هرگز `var`
- ۷ primitive + آبجکت؛ `typeof`؛ تله‌های `typeof null` و آرایه
- همیشه `===`؛ `??` برای nullish؛ `"5" + 3` = `"53"`!
- Template literal با backtick و `${}` — استاندارد ساخت رشته

## 📝 تمرین فصل ۱ (در زمین تمرین — هر تمرین یک فایل)

1. `info.js`: نام، سن، شهر خودت را در متغیرها بگذار و با template literal یک جمله معرفی چاپ کن.
2. `calc.js`: دو عدد در متغیر؛ جمع/تفریق/ضرب/باقیمانده را با template چاپ کن.
3. حدس بزن، بعد تست کن: `typeof null`، `typeof [1,2]`، `typeof NaN`، `"5" + 3`، `Number("5") + 3`
4. `swap.js`: دو متغیر a و b را بدون متغیر سوم جابجا کن. (راهنما: destructuring — `[a, b] = [b, a]` — یا با فکر کن!)
5. چالش: تبدیل دقیقه به «ساعت و دقیقه» — `135` → `"2 ساعت و 15 دقیقه"` (با % و Math.floor)

<details><summary>جواب‌ها</summary>

```js
// تمرین ۳:
typeof null    // "object" — باگ تاریخی
typeof [1,2]   // "object"
typeof NaN     // "number"  ← NaN یعنی Not-a-Number ولی نوعش number!
"5" + 3        // "53" (رشته)
Number("5") + 3 // 8

// تمرین ۵:
const minutes = 135;
const h = Math.floor(minutes / 60);   // 2
const m = minutes % 60;               // 15
console.log(`${h} ساعت و ${m} دقیقه`);
```
</details>

➡️ **فصل بعد:** تصمیم‌گیری و تکرار — شرط‌ها، حلقه‌ها و توابع.
