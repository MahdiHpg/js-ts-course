# فصل ۲ — کنترل جریان: شرط‌ها، حلقه‌ها و توابع پایه

> 🎯 **هدف:** مغز برنامه — تصمیم‌گیری (if/switch/ternary)، تکرار (for/while/for-of) و بسته‌بندی منطق در تابع. آخر فصل می‌توانی منطق‌های واقعی بنویسی.

---

## ۲.۱ — if / else if / else

```js
const score = 78;

if (score >= 90) {
  console.log("عالی 🏆");
} else if (score >= 70) {
  console.log("خوب 👍");
} else if (score >= 50) {
  console.log("قبول");
} else {
  console.log("مردود");
}
```

- شرط همیشه به boolean تبدیل می‌شود — و مقادیر «falsy» وجود دارند که false حساب می‌شوند:

```js
// falsy: false, 0, "", null, undefined, NaN
// بقیه همه truthy اند: "0"، "false"، []، {}

if ("") console.log("اجرا نمی‌شود");     // رشته خالی = falsy
if ([])  console.log("اجرا می‌شود");      // آرایه خالی = truthy! ⚠️
```

> ⚠️ تله کلاسیک: `if (value)` برای «وجود دارد» — وقتی value می‌تواند **0** باشد خراب می‌شود (۰ = falsy!). برای «null/undefined» دقیق‌تر: `if (value != null)` یا `if (value !== undefined)`.

## ۲.۲ — ternary و switch

```js
// ternary — شرط یک‌خطی: شرط ? اگر‌درست : اگر‌غلط
const status = score >= 50 ? "قبول" : "مردود";
const label = score >= 90 ? "عالی" : score >= 70 ? "خوب" : "معمولی";  // زنجیره

// switch — وقتی یک مقدار را با چند حالت ثابت مقایسه می‌کنی
const day = "sat";
switch (day) {
  case "sat":
  case "sun":
    console.log("آغاز هفته");
    break;                       // ⚠️ بدون break به case بعدی می‌ریزد (fall-through)!
  case "fri":
    console.log("تعطیل");
    break;
  default:
    console.log("روز عادی");
}
```

قاعده انتخاب: ۲-۳ حالت → ternary/if؛ ۴+ حالت ثابت → switch (یا آبجکت lookup — فصل ۴!).

## ۲.۳ — حلقه‌ها

```js
// for کلاسیک — وقتی شمارنده می‌خواهی
for (let i = 1; i <= 5; i++) {
  console.log(`ردیف ${i}`);
}

// while — وقتی تعداد تکرار نامشخص است
let attempts = 0;
while (attempts < 3) {
  console.log(`تلاش ${attempts + 1}`);
  attempts++;              // ⚠️ بدون این: حلقه بی‌نهایت! (Ctrl+C در ترمینال)
}

// for...of — پیمایش آرایه (مدرن و خوانا) ⭐
const skills = ["JS", "TS", "React"];
for (const skill of skills) {
  console.log(`در حال یادگیری ${skill}`);
}

// break و continue
for (const n of [1, 2, 3, 4, 5]) {
  if (n === 3) continue;   // این دور را رد کن
  if (n === 5) break;      // از حلقه خارج شو
  console.log(n);          // 1 2 4
}
```

> 💡 برای آرایه‌ها معمولاً `for...of` یا متدها (map/filter — فصل ۴) انتخاب‌اند؛ `for` کلاسیک برای شمارنده‌های واقعی (۱ تا ۱۰) یا پیمایش با ایندکس خاص.

## ۲.۴ — توابع: بسته‌بندی منطق

```js
// Function Declaration — با اسم
function greet(name) {
  return `سلام ${name}!`;
}

// فراخوانی:
const msg = greet("علی");     // "سلام علی!"
```

- `return` = خروجی تابع؛ بعد از return اجرای تابع تمام می‌شود
- تابع بدون return → `undefined` برمی‌گرداند
- **پارامتر پیش‌فرض:**

```js
function greet(name = "مهمان") {
  return `سلام ${name}!`;
}
greet();           // "سلام مهمان!"
```

- **تابع به عنوان مقدار:** توابع در JS «اول‌رده» اند — می‌توانند در متغیر، آرگومان و خروجی باشند:

```js
const double = function (n) { return n * 2 };     // function expression
const triple = (n) => n * 3;                      // ⭐ arrow function — فصل ۳ کامل

function applyTwice(fn, value) {
  return fn(fn(value));        // تابع را به عنوان آرگومان پاس می‌دهیم!
}
applyTwice(double, 5);          // 20
```

## ۲.۵ — برنامه واقعی: بازی حدس عدد

```js
// guess.js — چرخه کامل شرط/حلقه/تابع
function checkGuess(guess, secret) {
  if (guess === secret) return "درست بود! 🎉";
  if (guess < secret) return "بزرگ‌تر بگو ⬆️";
  return "کوچک‌تر بگو ⬇️";
}

const secret = Math.floor(Math.random() * 100) + 1;   // ۱ تا ۱۰۰
let guess = 50;

// شبیه‌سازی ۵ تلاش (در REPL خودت بازی کن!)
for (let i = 1; i <= 5; i++) {
  console.log(`تلاش ${i}: ${guess} → ${checkGuess(guess, secret)}`);
  guess = guess < secret ? guess + 7 : guess - 3;      // استراتژی ساده
}
```

---

## ✅ جمع‌بندی فصل

- falsy ها: false/0/""/null/undefined/NaN — و تله‌ی `if (value)` با عدد صفر
- ternary برای یک‌خط، switch برای حالت‌های ثابت (break یادت نره)
- for (شمارنده) / while (تا وقتی) / for...of (پیمایش آرایه) + break/continue
- تابع = بسته‌بندی منطق؛ return خروجی؛ پیش‌فرض پارامتر؛ توابع مقادیرند
- ترکیب‌شان = هر منطقی که در پروژه لازم داری

## 📝 تمرین فصل ۲

1. `fizzbuzz(n)`: اعداد ۱ تا n — مضرب ۳ «Fizz»، مضرب ۵ «Buzz»، مضرب هر دو «FizzBuzz»، بقیه خود عدد. (مسئله‌ی افسانه‌ای مصاحبه!)
2. `max3(a, b, c)`: بزرگ‌ترین سه عدد را برگرداند — دو راه: با if و با ternary.
3. حلقه‌ای بنویس جدول ضرب ۷ را چاپ کند: `7 × 1 = 7` ... `7 × 10 = 70`
4. `isEven(n)` بنویس بدون `%` (راهنما: bitwise `& 1` یا چک روی رقم آخر رشته)
5. چالش: تابع `countVowels(text)` — تعداد حروف صدادار (a,e,i,o,u — فارسی: ا،و،ی) را بشمارد.

<details><summary>جواب‌ها</summary>

```js
// ۱) FizzBuzz
function fizzbuzz(n) {
  for (let i = 1; i <= n; i++) {
    if (i % 15 === 0) console.log("FizzBuzz");
    else if (i % 3 === 0) console.log("Fizz");
    else if (i % 5 === 0) console.log("Buzz");
    else console.log(i);
  }
}
// نکته: اول % 15 (هر دو) — وگرنهFizz فقط می‌گیرد!

// ۲) max3
function max3(a, b, c) {
  if (a >= b && a >= c) return a;
  return b >= c ? b : c;
}
// با Math.max ساده‌تر: Math.max(a, b, c)

// ۳)
for (let i = 1; i <= 10; i++) console.log(`7 × ${i} = ${7 * i}`);

// ۴)
function isEven(n) {
  return (n & 1) === 0;    // بیت آخر صفر = زوج
  // یا: String(n).at(-1) در "02468".includes(...)
}

// ۵)
function countVowels(text) {
  let count = 0;
  for (const ch of text.toLowerCase()) {
    if ("aeiou".includes(ch)) count++;
  }
  return count;
}
```
</details>

➡️ **فصل بعد:** توابع عمیق — scope، closure، arrow و callback.
