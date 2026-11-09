# فصل ۷ — Async کامل: Promise، async/await و fetch

> 🎯 **هدف:** مهم‌ترین فصل JS — ۹۰٪ کار دولوپر مرورگر منتظر ماندن است: fetch از سرور، تایمر، فایل. سه نسل (callback → Promise → async/await)، event loop و الگوهای واقعی.

---

## ۷.۱ — چرا async؟ و Callback

JS single-threaded است — یک کار در آن لحظه. اگر fetch سرور (۲ ثانیه) بلاک می‌کرد، کل صفحه فریز می‌شد! پس عملیات طولانی **ناهمگام** (async) اجرا می‌شوند: «شروع کن، ادامه بده، وقتی حاضر شد مرا صدا بزن».

```js
// نسل ۱: callback (فصل ۳ دیدی)
getUser(1, (user) => {
  console.log(user);
  getOrders(user, (orders) => {
    getDetails(orders[0], (details) => {
      // 💀 callback hell — هرم آبی، مدیریت خطا فاجعه
    });
  });
});
```

## ۷.۲ — Promise: قولِ نتیجه آینده

**Promise** = رسیدی برای نتیجه‌ی عملیات async — سه وضعیت: `pending` (در راه) → `fulfilled` (رسید ✅) یا `rejected` (شکست ❌) — و بعدش قفل می‌شود.

```js
const p = fetch("https://api.example.com/user/1");
// p فوراً یک Promise است — هنوز داده ندارد!

// مصرف:
p.then((response) => response.json())     // then: وقتی موفق شد
 .then((data) => console.log(data))
 .catch((err) => console.error("شکست:", err))   // catch: خطا
 .finally(() => console.log("تمام شد"));        // finally: در هر صورت

// ساخت Promise دستی:
const wait = (ms) => new Promise((resolve) => setTimeout(resolve, ms));
wait(1000).then(() => console.log("یک ثانیه گذشت!"));

// ساخت خطا:
const fail = Promise.reject(new Error("اوپس"));
fail.catch((e) => console.error(e.message));
```

## ۷.۳ — async/await: خوانایی همگام ⭐

شیرین‌سازی Promise ها — کد async که مثل sync خوانده می‌شود:

```js
async function loadUser(id) {
  try {
    const res = await fetch(`https://api.example.com/user/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);   // fetch روی 404 throw نمی‌کند!
    const user = await res.json();
    return user;
  } catch (err) {
    console.error("خطا:", err.message);
    return null;
  } finally {
    console.log("تمام شد");
  }
}
```

قوانین:
- `await` فقط داخل تابع `async` (یا top-level در ماژول‌ها)
- `await` فقط Promise ها را متوقف «منطقی» می‌کند (thread بلاک نمی‌شود!)
- خطای await شده = throw → با try/catch
- تابع async همیشه Promise برمی‌گرداند — حتی بدون await

### Waterfall در برابر موازی ⭐

```js
// ❌ سریالی: ۵۰۰ms + ۳۰۰ms = ۸۰۰ms
const user = await getUser();
const posts = await getPosts();

// ✅ موازی: ۵۰۰ms
const [user, posts] = await Promise.all([getUser(), getPosts()]);

// ✅ بهینه‌تر: همه را می‌خواهیم حتی اگر یکی شکست خورد
const results = await Promise.allSettled([getUser(), getPosts()]);
// [{status:"fulfilled",value:...}, {status:"rejected",reason:...}]

// ✅ timeout دستی با race:
const result = await Promise.race([fetch(url), wait(3000).then(() => { throw new Error("timeout") })]);
```

| ابزار | رفتار |
|---|---|
| `Promise.all` | یکی شکست = همه reject |
| `Promise.allSettled` | همه را گزارش می‌دهد |
| `Promise.race` | اولین settle برنده است |
| `Promise.any` | اولین **موفق** برنده است |

## ۷.۴ — fetch: درخواست واقعی

```js
// GET ساده:
const res = await fetch("https://api.site.com/products");
const data = await res.json();       // ⚠️ دو مرحله: پاسخ → بدنه JSON

// POST با بدنه و هدر:
await fetch("https://api.site.com/orders", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({ productId: 1, qty: 2 }),
});

// چک پاسخ — دو نکته‌ای که همه یادشان می‌رود:
// ۱) fetch روی 404/500 ارور نمی‌دهد! خودت چک کن:
if (!res.ok) throw new Error(`HTTP ${res.status}`);
// ۲) res.json() هم خودش Promise است — await لازم دارد
```

## ۷.۵ — Event Loop: چرا این ترتیب؟ (سوال مصاحبه ⭐)

```js
console.log("1 — sync");
setTimeout(() => console.log("4 — macrotask"), 0);
Promise.resolve().then(() => console.log("3 — microtask"));
console.log("2 — sync");
// خروجی: 1 → 2 → 3 → 4
```

ترتیب حلقه: **کد sync → همه microtask ها (Promise ها) → یک macrotask (setTimeout) → microtask ها → ...**

یعنی `setTimeout(fn, 0)` یعنی «در صف بعدی» نه «الان». و به همین دلیل await ها صفحه را فریز نمی‌کنند — فقط جریان آن تابع منطقاً منتظر می‌ماند.

---

## ✅ جمع‌بندی فصل

- چرا async: single-thread بودن؛ عملیات طولانی نباید بلاک کند
- Promise: pending→fulfilled/rejected؛ then/catch/finally
- **async/await**: خوانا؛ try/catch؛ `Promise.all` برای موازی (ضد waterfall)
- fetch: دو مرحله (پاسخ → json)، `res.ok` را چک کن، POST با headers/body
- event loop: sync → microtask → macrotask

## 📝 تمرین فصل ۷ (با API رایگان https://jsonplaceholder.typicode.com)

1. `getUser(id)`: با fetch کاربر را بگیر و برگردان — با چک `res.ok` و try/catch.
2. دو fetch موازی (users و posts) با Promise.all و خروجی ترکیبی.
3. تابع `withTimeout(promise, ms)`: اگر promise تا X ms جواب نداد، reject با «timeout» — با Promise.race.
4. خروجی را حدس بزن (فصل event loop):
```js
(async () => {
  console.log("A");
  await Promise.resolve();
  console.log("B");
})();
console.log("C");
```
5. چالش واقعی: تابع `retry(fn, times)` بنویس — fetch را تا n بار تلاش کند با فاصله فزاینده (backoff).

<details><summary>جواب‌ها</summary>

```js
// ۱
async function getUser(id) {
  try {
    const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    console.error("خطا:", err.message);
    return null;
  }
}

// ۲
const [users, posts] = await Promise.all([
  fetch(".../users").then(r => r.json()),
  fetch(".../posts").then(r => r.json()),
]);

// ۳
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error("timeout")), ms)
  );
  return Promise.race([promise, timeout]);
}

// ۴: A C B — بعد از await، بقیه تابع به microtask می‌رود؛ C (sync) اول چاپ می‌شود.
// ۵
async function retry(fn, times = 3, delay = 500) {
  try {
    return await fn();
  } catch (err) {
    if (times <= 1) throw err;
    await wait(delay);
    return retry(fn, times - 1, delay * 2);   // backoff فزاینده
  }
}
```
</details>

➡️ **فصل بعد:** ماژول‌ها، مدیریت خطای حرفه‌ای و دیباگ — پایان بخش JS!
