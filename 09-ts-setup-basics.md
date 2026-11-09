# فصل ۹ — TypeScript از صفر: نصب، annotate پایه

> 🎯 **هدف بخش دوم:** TS = JS + تایپ. همه‌ی JS فصل ۱-۸ را داری؛ حالا یاد می‌گیری *قبل از اجرا* خطاها را بگیری. نصب، annotate پایه، انواع، توابع — هر مثال JS قبلی، نسخه TS اش را می‌گیرد.

---

## ۹.۱ — چرا TypeScript؟ (با یک مثال دردناک)

```js
// ── JavaScript ──
function getTotal(items) {
  return items.reduce((sum, i) => sum + i.price, 0);
}
getTotal("سلام");        // 💥 ارور در RUNTIME — صفحه کاربر!
getTotal([{ price: "ده" }]);  // "0ده" — باگ خاموش و مخرب!
```

کد بالا تا اجرا ساکت است — و بعد از deploy می‌میرد. TS همین را **قبل از اجرا** می‌گیرد:

```ts
// ── TypeScript ──
type Item = { price: number };

function getTotal(items: Item[]): number {
  return items.reduce((sum, i) => sum + i.price, 0);
}

getTotal("سلام");             // ❌ ارور در EDITOR — قبل از حتی ذخیره!
getTotal([{ price: "ده" }]);  // ❌ string به number نمی‌رود
getTotal([{ price: 50 }]);    // ✅
```

TS هیچ‌چیز به runtime اضافه نمی‌کند — فقط **چک‌کننده‌ی زمان توسعه** است: کد TS کامپایل می‌شود به JS خالص.

> 🔑 جمله کلیدی: «TS فقط در زمان توسعه وجود دارد؛ خروجی نهایی JS خالص است.»

## ۹.۲ — نصب و اجرا

```bash
mkdir ts-playground && cd ts-playground
npm init -y
npm install -D typescript      # نصب محلی (پیشنهادی)
npx tsc --init                 # ساخت tsconfig.json

npx tsc app.ts                 # کامپایل: app.js می‌سازد
npx tsc --watch                # مانیتور: با هر ذخیره دوباره کامپایل
npx tsx app.ts                 # اجرای مستقیم TS (بدون کامپایل — برای تمرین/اسکریپت)
```

## ۹.۳ — Type Annotation: برچسب‌گذاری

```ts
// متغیر:
let name: string = "علی";
let age: number = 25;
let isAdmin: boolean = false;
let ids: number[] = [1, 2, 3];
let pair: [string, number] = ["علی", 25];   // tuple — طول و نوع ثابت

// 💡 نکته مهم: TS «type inference» دارد — اغلب لازم نیست بنویسی:
let city = "تهران";        // TS خودش فهمید: string
city = 5;                  // ❌ ارور! چون از اول string حدس زد
// قاعده: نوع پیش‌فرض را بنویس وقتی inference نمی‌داند (پارامتر توابع!)
```

### نوع‌های ویژه

```ts
let nothing: undefined = undefined;
let missing: null = null;

// union — یکی از چند نوع:
let id: string | number;
id = "abc"; id = 42;       // ✅ هر دو

// literal type — فقط این مقادیر:
let direction: "up" | "down" | "left" | "right";
direction = "up";          // ✅
direction = "diagonal";    // ❌

// any — خاموش کردن TS (دشمن!)
let loose: any = "هرچی";
loose.doAnything();        // ❌ چک نمی‌شود — باگ به runtime می‌رود

// unknown — نسخه امن any: بگیر، ولی قبل مصرف narrow کن
let safe: unknown = getData();
// safe.toUpperCase();     // ❌ اول بگو چه هستی!
if (typeof safe === "string") safe.toUpperCase();  // ✅

// void و never:
function log(msg: string): void { console.log(msg); }   // هیچ برنمی‌گرداند
function fail(): never { throw new Error(); }            // هرگز برنمی‌گردد
```

> 🔑 جمله مصاحبه‌ای: «**any TS را خاموش می‌کند، unknown آن را روشن نگه می‌دارد** — رازها را any نگیر، unknown بگیر و narrow کن.»

## ۹.۴ — تایپ توابع و آبجکت‌ها

```ts
// توابع — پارامترها اجباری، خروجی معمولاً inference:
function greet(name: string, excited: boolean = false): string {
  return excited ? `سلام ${name}!` : `سلام ${name}`;
}

// نوع کل تابع (callback style):
type Mapper = (n: number) => string;
const map: Mapper = (n) => String(n);   // پارامترها خودکار تایپ‌دار!

// آبجکت — annotation مستقیم:
const user: { name: string; age: number; city?: string } = {
  name: "علی",
  age: 25,            // city? اختیاری است — نداشتنش ارور نیست
};

// readonly — تغییر ممنوع:
const config: { readonly port: number } = { port: 3000 };
// config.port = 4000;   // ❌
```

## ۹.۵ — tsconfig: قوانین پروژه

```jsonc
// tsconfig.json — حداقل‌های یک پروژه مدرن:
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,              // ⭐⭐ قوانین سخت — همیشه روشن!
    "noUncheckedIndexedAccess": true,  // آرایه‌ها ممکن است undefined بدهند
    "skipLibCheck": true
  }
}
```

**`strict: true` همیشه** — پروژه‌های مدرن (Next.js) پیش‌فرض روشنش می‌کنند. strict خاموش = نصف TS را بیکار کرده‌ای.

---

## ✅ جمع‌بندی فصل

- TS = چک زمان توسعه؛ خروجی JS خالص؛ ارور قبل از اجرا
- نصب: `npm i -D typescript` + `tsconfig` + `npx tsc --watch` یا `tsx`
- annotation: متغیر/آرایه/tuple/union/literal/any/**unknown**/void/never
- inference: اغلب خودش می‌فهمد؛ پارامترها را تو بگو
- `strict: true` همیشه

## 📝 تمرین فصل ۹

1. playground بساز و سه نسخه بنویس: نسخه JS باگ‌دار (`getTotal`)، بعد همان با TS — و ارورهای editor را ببین.
2. برای این تابع TS بنویس: `function repeat(text, times)` — تایپ کامل با پیش‌فرض.
3. union تمرین: متغیر `status: "idle" | "loading" | "success" | "error"` — مقادیر معتبر/نامعتبر را تست کن و ارورها را بخوان.
4. فرق `any` و `unknown` را عملاً نشان بده: روی هر دو `.toUpperCase()` صدا بزن — کدام ارور می‌دهد؟ چرا این یعنی امن‌تر؟
5. تایپ این آبجکت را کامل کن (شهر اختیاری، تگ‌ها آرایه رشته): `const user = { name: "علی", age: 25, city: "تهران", tags: ["js"] }`

<details><summary>جواب‌ها</summary>

```ts
// ۲
function repeat(text: string, times: number = 1): string {
  return text.repeat(times);
}

// ۴: any بدون ارور اجرا/کامپایل می‌شود و اگر رشته نباشد در runtime می‌شکند؛
//    unknown بلافاصله ارور می‌دهد: «Object is of type 'unknown'» —
//    مجبور می‌شوی اول چک کنی (typeof) — یعنی باگ در editor گرفته می‌شود.

// ۵
const user: {
  name: string;
  age: number;
  city?: string;
  tags: string[];
} = { name: "علی", age: 25, city: "تهران", tags: ["js"] };
```
</details>

➡️ **فصل بعد:** interface/type، union های پیشرفته، narrowing و generics.
