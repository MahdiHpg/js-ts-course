# فصل ۳ — توابع عمیق: Scope، Closure، Arrow و Callback

> 🎯 **هدف:** مرز جونیور و متوسط در JS همین‌جاست. چهار مفهوم: scope (دید متغیرها)، hoisting، closure (بستار — قلب JS)، و arrow function ها. این فصل را دو بار بخوان — بار دوم همه‌چیز جا می‌افتد!

---

## ۳.۱ — Scope: متغیر کجا «زندگی» می‌کند؟

```js
const global = "همه‌جا دیدنی";          // scope سراسری

function outer() {
  const outerVar = "دیدنی در outer و فرزندانش";

  function inner() {
    const innerVar = "فقط همین‌جا";
    console.log(global);     // ✅ بالا سرش را می‌بیند
    console.log(outerVar);   // ✅ والد
    console.log(innerVar);   // ✅ خودش
  }
  inner();
  // console.log(innerVar);  // ❌ ReferenceError — خارج از scope
}
```

قاعده: **تابع فرزند به متغیرهای والد دسترسی دارد، برعکس نه.** جستجوی متغیر از داخل به بیرون (scope chain) انجام می‌شود.

### Block scope با let/const

```js
{
  const inside = "فقط همین بلوک";
}
// console.log(inside);  // ❌ خارج از بلوک

if (true) {
  let blockLet = "let هم بلوکی است";
}
```

### پس var چه دردسری دارد؟

```js
// var فقط function-scoped است — بلوک را نمی‌شناسد!
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i));   // 3 3 3  ← هر سه بعد از حلقه، همان i نهایی!
}
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j));   // 0 1 2  ✅ هر دور، j خودش
}
```

+ hoisting: `var` با undefined بالا می‌آید، `let/const` تا تعریف TDZ دارند. **نتیجه: همیشه let/const، هرگز var.**

## ۳.۲ — Hoisting: بالا آمدن تعریف‌ها

```js
sayHi();          // ✅ کار می‌کند! — declaration کامل hoist می‌شود
function sayHi() { console.log("hi"); }

console.log(x);   // undefined — var hoist می‌شود ولی مقدار نه
var x = 5;

console.log(y);   // ❌ ReferenceError — let/const در TDZ اند
let y = 5;
```

## ۳.۳ — Closure: بستار (مفهوم طلایی JS) ⭐

**تعریف:** وقتی یک تابع از درون تابع دیگر ساخته می‌شود، به متغیرهای تابع پدر — حتی بعد از پایان اجرای پدر — دسترسی دارد. «تابع + حافظه‌اش»:

```js
function makeCounter() {
  let count = 0;                    // متغیر خصوصی — از بیرون دسترسی نیست!
  return function () {
    count++;                        // closure: count را به یاد دارد
    return count;
  };
}

const counter1 = makeCounter();
const counter2 = makeCounter();     // هر کدام counter مستقل دارند!

counter1();  // 1
counter1();  // 2
counter2();  // 1  ← مستقل!
```

**سه کاربرد واقعی:**

```js
// ۱) داده خصوصی — بدون class
function makeWallet(initial) {
  let balance = initial;
  return {
    deposit: (n) => (balance += n),
    getBalance: () => balance,
  };
}

// ۲) debounce/throttle (در مصاحبه‌ها و پروژه‌ها)
function debounce(fn, delay) {
  let timer;                              // در closure زنده است
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

// ۳) پیکربندی تابع
const multiplier = (factor) => (n) => n * factor;
const byTwo = multiplier(2);
byTwo(5);   // 10
```

> 💡 بگو: «closure یعنی تابع + محیطی که در آن متولد شد» — همین یک جمله در مصاحبه کافی است.

## ۳.۴ — Arrow Function: خلاصه، lexical this

```js
// از طولانی به کوتاه:
const add = (a, b) => { return a + b; };
const add = (a, b) => a + b;             // بدنه تک‌عبارتی: return ضمنی!
const square = n => n * n;               // تک‌پارامتر: بدون پرانتز
const zero = () => 0;

// خواندن کد دیگران (callback های آرایه):
const names = users.map(u => u.name).filter(n => n.length > 3);
```

**فرق اصلی با function معمولی:** arrow **`this` خودش را ندارد** — از محیط اطراف می‌گیرد (lexical this). در کلاس‌ها و event handler های قدیمی حیاتی بود؛ در function components مدرن فرقش کمتر حس می‌شود ولی استایل استاندارد کد مدرن است.

## ۳.۵ — Callback: تابع به عنوان آرگومان

```js
function processUser(user, onSuccess, onError) {
  if (user.age >= 18) onSuccess(user);
  else onError(new Error("زیر ۱۸"));
}

processUser(
  { name: "علی", age: 20 },
  (u) => console.log(`خوش آمدی ${u.name}`),
  (err) => console.error(err.message)
);
```

callback پایه‌ی همه‌چیز است: متدهای آرایه (map/filter — فصل ۴)، event listener ها، و حتی Promise ها (فصل ۷ — که callback hell را هم حل کردند!).

> ⚠️ **Callback Hell** — قبل از Promise ها، async های تو در تو مثل هرم آبی می‌شدند:

```js
getUser(id, (user) => {
  getOrders(user, (orders) => {
    getDetails(orders[0], (details) => {
      // ... تودرتو و غیرقابل مدیریت — «هرم آبی» 💀
    });
  });
});
```

علاجش async/await است — فصل ۷.

---

## ✅ جمع‌بندی فصل

- scope: فرزند والد را می‌بیند، برعکس نه؛ let/const بلوکی، var نه (و باگ setTimeout)
- hoisting: function declaration کامل، var بی‌مقدار، let/const در TDZ
- **closure = تابع + حافظه‌اش**: counter، داده خصوصی، debounce
- arrow: خلاصه + return ضمنی + lexical this
- callback = تابع به عنوان آرگومان — پایه‌ی async و متدهای فصل ۴

## 📝 تمرین فصل ۳

1. خروجی را حدس بزن (قبل از اجرا!):
```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log("var:", i));
}
```
2. `makePasswordChecker(minLength)` بنویس که تابعی برگرداند: رشته بگیرد و true/false بدهد — با closure (هر checker حد خودش را دارد).
3. خروجی؟
```js
const obj = {
  name: "علی",
  say: function () { console.log(this.name); },
  sayArrow: () => console.log(this.name),
};
obj.say();      // ?
obj.sayArrow(); // ?
```
4. با callback ها: تابع `repeat(n, action)` بنویس که action را n بار صدا بزند. تست: `repeat(3, (i) => console.log("دور", i))`
5. چالش: با closure یک `makeCache()` بنویس که `get(key)` و `set(key, value)` بدهد و داده را بین فراخوانی‌ها حفظ کند.

<details><summary>جواب‌ها</summary>

1. سه بار «var: 3» — var بلوکی نیست؛ سه callback بعد از پایان حلقه اجرا می‌شوند و همه همان i نهایی (3) را می‌بینند. با let: 0،1،2.
2. 
```js
function makePasswordChecker(minLength) {
  return (password) => password.length >= minLength;
}
const check8 = makePasswordChecker(8);
check8("abc");  // false
```
3. اولی «علی» (this = obj)؛ دومی undefined (یا global — چون arrow this ندارد و از scope بیرونی می‌گیرد که obj نیست!). سوال محبوب مصاحبه.
4. 
```js
function repeat(n, action) {
  for (let i = 0; i < n; i++) action(i);
}
```
5. 
```js
function makeCache() {
  const store = new Map();     // در closure زنده می‌ماند
  return {
    get: (key) => store.get(key),
    set: (key, value) => store.set(key, value),
  };
}
```
</details>

➡️ **فصل بعد:** آبجکت‌ها و آرایه‌ها — پرکاربردترین ساختارها و متدهایشان.
