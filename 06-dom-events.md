# فصل ۶ — DOM و رویدادها: زنده کردن صفحه

> 🎯 **هدف:** JS بدون DOM فقط محاسبه است — با DOM صفحه ساخته می‌شود. انتخاب عناصر، تغییر متن/استایل، ساخت عنصر و مدیریت رویداد. (در React این کارها خودکار می‌شوند — ولی برای فهم React و پروژه‌های vanilla، این پایه‌ست.)

---

## ۶.۱ — انتخاب عناصر

یک فایل `index.html` بساز:

```html
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<body>
  <h1 id="title">سلام!</h1>
  <p class="msg">پیام اول</p>
  <p class="msg">پیام دوم</p>
  <button id="btn">کلیک کن</button>
  <ul id="list"></ul>
  <script src="app.js"></script>   ← آخر body تا عناصر موجود باشند
</body>
</html>
```

```js
// انتخاب:
const title = document.getElementById("title");
const msgs  = document.querySelectorAll(".msg");   // NodeList همه .msg ها
const btn   = document.querySelector("#btn");      // سلکتور CSS — مدرن‌ترین

// خواندن و نوشتن:
title.textContent = "سلام دنیای DOM!";     // متن
title.innerHTML = "سلام <strong>دنیای</strong>!";  // HTML (⚠️ فقط داده معتمد!)

// استایل و کلاس:
title.style.color = "crimson";
title.classList.add("active");
title.classList.toggle("hidden");           // اگر بود بردار، نبود بگذار

// attribute ها:
btn.setAttribute("disabled", "true");
btn.getAttribute("id");
```

> ⚠️ `innerHTML` با داده‌ی کاربر = دروازه XSS (تزریق اسکریپت)! برای متن همیشه `textContent`.

## ۶.۲ — ساخت و حذف عناصر

```js
const list = document.querySelector("#list");

const li = document.createElement("li");
li.textContent = "یادگیری DOM";
li.classList.add("item");
list.appendChild(li);              // به انتها اضافه شو

// حذف:
li.remove();
```

## ۶.۳ — رویدادها (Events) ⭐

```js
const btn = document.querySelector("#btn");

btn.addEventListener("click", (event) => {
  console.log("کلیک شد!", event.target);   // event = اطلاعات رویداد
  title.textContent = "کلیک شد!";
});

// رویدادهای پرکاربرد:
// click، input (تایپ زنده)، change (تغییر مقدار)، submit (فرم!)
// keydown، mouseenter، scroll، load

// فرم — با preventDefault (جلوگیری از رفرش صفحه):
const form = document.querySelector("#contact");
form.addEventListener("submit", (e) => {
  e.preventDefault();                       // بدون این: صفحه refresh می‌شود!
  const name = form.elements.name.value;
  console.log("ارسال:", name);
});

// ورودی زنده:
const search = document.querySelector("#search");
search.addEventListener("input", (e) => {
  console.log("تایپ شد:", e.target.value);  // هر کلید — debounce فصل ۳!
});
```

> 🔗 این دقیقاً همان چیزی است که React برایت **خودکار** می‌کند: `<button onClick={...}>` زیرش همین `addEventListener` است. فهم این فصل = فهم اینکه React چه دردی را حل می‌کند.

## ۶.۴ — پروژه کوچک: لیست کارها (vanilla)

```js
const form  = document.querySelector("#todo-form");
const input = document.querySelector("#todo-input");
const list  = document.querySelector("#todo-list");

form.addEventListener("submit", (e) => {
  e.preventDefault();
  const text = input.value.trim();
  if (!text) return;                       // خالی؟ هیچی

  const li = document.createElement("li");
  li.textContent = text;
  li.addEventListener("click", () => {
    li.classList.toggle("done");           // کلیک روی آیتم = انجام شده
  });

  list.appendChild(li);
  input.value = "";                        // پاک کردن input
});
```

همین TODO را در فصل ۸ مرور Next با React/Server Action دیدی — مقایسه کن: React با state، DOM را خودش sync می‌کند؛ اینجا تو دستی می‌سازی.

---

## ✅ جمع‌بندی فصل

- انتخاب: `querySelector` (سلکتور CSS) + `getElementById`
- `textContent` برای متن (امن)؛ `innerHTML` فقط با داده معتمد
- `createElement` + `appendChild`؛ `classList.toggle`
- `addEventListener` + `preventDefault` در فرم‌ها؛ رویداد input زنده است
- React همین درد را خودکار حل می‌کند — الان می‌فهمی چرا محبوب است!

## 📝 تمرین فصل ۶

1. صفحه‌ای با یک input و یک `<h2>`: هر چه تایپ می‌شود فوری در h2 نمایش داده شود (رویداد input).
2. دکمه‌ای بساز که با هر کلیک یک عدد را یکی زیاد کند و در خودش نشان دهد (closure/state با متغیر بیرونی).
3. لیست رنگ‌ها در JS: با for...of برای هر رنگ یک `<li>` با `background` همان رنگ بساز.
4. فرمی با چک طول متن: زیر ۳ حرف، دکمه submit غیرفعال شود (`disabled`).
5. چالش: دکمه «تم» — با هر کلیک `dark` کلاس روی `<body>` toggle شود و متن دکمه عوض شود.

<details><summary>جواب‌های ۱، ۲ و ۵</summary>

```js
// ۱
const input = document.querySelector("#name-input");
const preview = document.querySelector("#preview");
input.addEventListener("input", (e) => {
  preview.textContent = e.target.value;
});

// ۲
let count = 0;
const btn = document.querySelector("#counter");
btn.addEventListener("click", () => {
  count++;
  btn.textContent = `کلیک‌ها: ${count}`;
});

// ۵
const themeBtn = document.querySelector("#theme-btn");
themeBtn.addEventListener("click", () => {
  const dark = document.body.classList.toggle("dark");
  themeBtn.textContent = dark ? "☀️ روشن" : "🌙 تاریک";
});
```
</details>

➡️ **فصل بعد:** Async — بزرگ‌ترین و مهم‌ترین فصل JS: fetch، Promise و async/await.
