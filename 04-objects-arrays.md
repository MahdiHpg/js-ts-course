# فصل ۴ — آبجکت‌ها و آرایه‌ها: پرکاربردترین ساختارهای JS

> 🎯 **هدف:** ۹۰٪ داده‌ی هر اپ JS با این دو ساختار جابه‌جا می‌شود (JSON = همین‌ها!). ساخت، دسترسی، destructuring، spread و متدهای آرایه — قلب کار روزمره‌ات.

---

## ۴.۱ — آبجکت: داده ساخت‌یافته

```js
const user = {
  name: "علی",                    // key: value
  age: 25,
  "favorite lang": "JS",          // کلید با فاصله → کوتیشن لازم
  email: "ali@dev.ir",
};

// دسترسی:
user.name            // "علی"      (نقطه‌ای — رایج)
user["favorite lang"] // براکتی — وقتی کلید فاصله/متغیر است

// تغییر و افزودن:
user.age = 26;
user.city = "تهران";              // کلید جدید — همین‌قدر ساده
delete user.email;

// کلید از متغیر:
const key = "name";
user[key];               // "علی"   (براکتی با متغیر)

// چک وجود:
"name" in user               // true
user.phone ?? "ثبت نشده"     // undefined → پیش‌فرض (فصل ۱)

// پیمایش:
for (const key in user) {
  console.log(key, "→", user[key]);
}
Object.keys(user);           // ["name", "age", "city", ...]
Object.values(user);         // مقادیر
Object.entries(user);        // [["name","علی"], ...] — عالی برای map
```

**متد داخل آبجکت:**

```js
const counter = {
  count: 0,
  increment() {               // shorthand متد
    this.count++;             // this = خودِ آبجکت
  },
};
counter.increment();
counter.count;   // 1
```

## ۴.۲ — آرایه: لیست مرتب

```js
const skills = ["JS", "TS", "React"];

skills[0];              // "JS"    (ایندکس از صفر!)
skills.length;          // 3
skills.at(-1);          // "React" ← آخرین عنصر (at با ایندکس منفی!)
skills.push("CSS");     // افزودن آخر
skills.pop();           // حذف آخر
skills.unshift("HTML"); // افزودن اول
skills.shift();         // حذف اول
skills.includes("JS");  // true
skills.indexOf("TS");   // 1 (یا -1 اگر نیست)
skills.slice(1, 3);     // برش بدون تغییر اصلی ["TS","React"]
skills.join(" • ");     // "JS • TS • React"
```

## ۴.۳ — destructuring: باز کردن بسته‌ها ⭐

استخراج تمیز — همه‌جای کد مدرن:

```js
// آبجکت:
const user = { name: "علی", age: 25, city: "تهران" };
const { name, age, city = "نامشخص" } = user;    // با پیش‌فرض!
// name="علی", age=25, city="تهران"

// تغییر نام:
const { name: fullName } = user;   // fullName = "علی"

// تودرتو:
const response = { data: { user: { email: "a@b.c" } } };
const { data: { user: { email } } } = response;   // email = "a@b.c"

// آرایه (بر اساس ترتیب):
const [first, second] = skills;    // "JS", "TS"
const [head, ...rest] = skills;    // rest = بقیه

// در پارامتر تابع (شخصی‌ترین کاربرد):
function printUser({ name, age }) {
  console.log(`${name} — ${age}`);
}
printUser(user);   // فقط آبجکت می‌فرستی
```

## ۴.۴ — spread و rest: `...`

```js
// spread — باز کردن: کپی و ترکیب (immutable!)
const updated = { ...user, age: 27 };           // کپی + تغییر یک فیلد
const merged = { ...defaults, ...overrides };   // بعدی بر قبلی غلبه می‌کند
const all = [...arr1, ...arr2];                 // ترکیب آرایه‌ها

// rest — جمع کردن: پارامترهای نامشخص
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4);   // 10
```

> ⚠️ یادآوری مهم: `{...user}` یک **shallow copy** است — آبجکت‌های تودرتو هنوز مرجع مشترک دارند (فصل ۵ TS و deep copy را هم ببین).

## ۴.۵ — متدهای آرایه: map / filter / find / reduce ⭐⭐

**مهم‌ترین بخش کل دوره** — در هر پروژه React هر روز این‌ها را می‌نویسی:

```js
const products = [
  { id: 1, title: "گوشی",   price: 50,  category: "mobile", inStock: true },
  { id: 2, title: "لپ‌تاپ",  price: 120, category: "laptop", inStock: false },
  { id: 3, title: "ایرپاد", price: 12,  category: "mobile", inStock: true },
];

// map — تبدیل هر عنصر (همیشه هم‌طول)
const titles = products.map((p) => p.title);

// filter — فقط عناصری که شرط دارند
const available = products.filter((p) => p.inStock);

// find — اولین عنصر منطبق (یا undefined)
const phone = products.find((p) => p.id === 1);

// some / every — «حداقل یکی» / «همه»
products.some((p) => p.price > 100);    // true
products.every((p) => p.inStock);       // false

// sort — مرتب‌سازی (⚠️ اعداد به رشته sort می‌شوند مگر مقایسه بدهی!)
[...products].sort((a, b) => a.price - b.price);   // صعودی — روی کپی!

// reduce — تاخوردن به یک مقدار (جمع، گروه‌بندی، شمارش)
const total = products.reduce((sum, p) => sum + p.price, 0);   // 182

// زنجیره‌سازی — قدرت واقعی:
const mobileTitles = products
  .filter((p) => p.category === "mobile")
  .map((p) => `${p.title} (${p.price}M)`);
// ["گوشی (50M)", "ایرپاد (12M)"]
```

**سه قانون طلایی:**
1. `map` برای تبدیل (هم‌طول)، `filter` برای حذف، `reduce` برای تاخوردن — درست انتخاب کن
2. این‌ها **آرایه جدید** می‌سازند — اصل را نمی‌شکنند (immutable فلسفه React!)
3. زنجیره را می‌شود شکست: هر متد، ورودیِ متد بعدی

### groupBy با reduce (سوال مصاحبه!)

```js
const byCategory = products.reduce((acc, p) => {
  (acc[p.category] ??= []).push(p);
  return acc;
}, {});
// { mobile: [...], laptop: [...] }
```

---

## ✅ جمع‌بندی فصل

- آبجکت: نقطه‌ای/براکتی، in/??، Object.keys/values/entries، this در متد
- آرایه: at(-1)، push/pop، slice/join/includes
- destructuring (با پیش‌فرض و تو در تو) + در پارامترها — استایل مدرن
- spread برای کپی/ترکیب، rest برای جمع‌کردن — shallow copy حواست هست!
- **map/filter/find/reduce** = قلب کد فرانت؛ immutable و زنجیره‌پذیر

## 📝 تمرین فصل ۴

1. از آرایه products بالا: (الف) فقط عنوان‌های موجودها (ب) گران‌ترین محصول (ج) میانگین قیمت — هرکدام یک زنجیره.
2. `pluck(items, key)` بنویس: آرایه آبجکت + نام کلید → آرایه مقادیر آن کلید. (`pluck(products, "title")`)
3. destructuring تمرین: از `const meta = { info: { title: "t", tags: ["a","b"] } }` تگ اول و دوم را یک‌خطی استخراج کن.
4. چرا این کد باگ دارد؟ `const copy = products; copy.push(newItem);` — درستش کن.
5. چالش: تابع `sumBy(items, keyFn)` با reduce بنویس — جمع یک فیلد دلخواه.

<details><summary>جواب‌ها</summary>

```js
// ۱
products.filter(p => p.inStock).map(p => p.title);
[...products].sort((a, b) => b.price - a.price)[0];
products.reduce((s, p) => s + p.price, 0) / products.length;

// ۲
function pluck(items, key) {
  return items.map((item) => item[key]);
}

// ۳
const { info: { tags: [firstTag, secondTag] } } = meta;

// ۴: copy همان مرجع products است — push روی هر دو اثر می‌گذارد!
//    درست: const copy = [...products]; copy.push(newItem);

// ۵
function sumBy(items, keyFn) {
  return items.reduce((sum, item) => sum + keyFn(item), 0);
}
sumBy(products, (p) => p.price);
```
</details>

➡️ **فصل بعد:** رشته‌ها، اعداد، Date و JSON — ابزارهای داده.
