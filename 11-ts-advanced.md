# فصل ۱۱ — TS پیشرفته: Utility Types، Type Guards و satisfies

> 🎯 **هدف:** ابزارهایی که کد TS را از «تایپ‌شده» به «حرفه‌ای» تبدیل می‌کنند: Utility Types آماده، type guard اختصاصی، `satisfies` و چند الگوی واقعی. این فصل = مرز متوسط تا پیشرفته.

---

## ۱۱.۱ — Utility Types: ترنسفورمرهای آماده ⭐

از یک تایپ، تایپ‌های جدید بساز — بدون تکرار:

```ts
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Partial — همه فیلدها اختیاری (فرم آپدیت!):
function updateUser(id: number, changes: Partial<User>) {
  // { name?: string, email?: string, ... } — فقط فیلدهای فرستاده‌شده
}
updateUser(1, { name: "علی جدید" });

// Required — برعکس: همه اجباری
// Pick — فقط بعضی فیلدها:
type UserCard = Pick<User, "id" | "name">;      // { id, name } — کارت محصول!

// Omit — حذف بعضی فیلدها:
type PublicUser = Omit<User, "email">;          // بدون ایمیل (حریم!)

// Record — دیکشنری با شکل مشخص:
const roles: Record<string, string[]> = {
  admin: ["read", "write"],
  viewer: ["read"],
};
const counts: Record<"a" | "b", number> = { a: 1, b: 2 };

// Readonly — همه فقط‌خواندنی:
const frozen: Readonly<User> = { ... };
// frozen.name = "x"   // ❌

// ReturnType / Parameters — تایپ از دل تابع:
function createOrder(input: { productId: number; qty: number }) {
  return { id: 1, ...input, total: input.qty * 100 };
}
type Order = ReturnType<typeof createOrder>;    // { id, productId, qty, total }
type OrderInput = Parameters<typeof createOrder>[0];  // بدون تکرار تایپ ورودی!
```

> 🔑 فلسفه: **Single source of truth تایپ‌ها** — `User` را یک‌بار بنویس و با utility ها مشتق کن؛ تغییر آینده فقط یک‌جا.

---

## ۱۱.۲ — Type Guard اختصاصی: `is`

وقتی TS نمی‌تواند خودش تشخیص دهد (مثل داده‌ی API!)، به آن دست بده:

```ts
interface Cat { meow(): void }
interface Dog { bark(): void }

function isCat(animal: Cat | Dog): animal is Cat {   // ⭐ type predicate
  return "meow" in animal;
}

function speak(animal: Cat | Dog) {
  if (isCat(animal)) animal.meow();   // ✅ TS می‌داند Cat است
  else animal.bark();
}
```

**کاربرد واقعی — پاسخ API:**

```ts
interface ApiUser { id: number; name: string }

function isApiUser(data: unknown): data is ApiUser {
  return (
    typeof data === "object" && data !== null &&
    "id" in data && "name" in data
  );
}

const raw: unknown = await getData();
if (isApiUser(raw)) {
  console.log(raw.name);    // ✅ امن و تایپ‌دار
}
```

(در پروژه‌های جدی، این کار را به zod می‌سپاری که هم validate هم تایپ تولید کند — فصل ۱۲.)

## ۱۱.۳ — satisfies: بهترین هر دو دنیا (TS 4.9+)

مشکل کلاسیک: annotation نوع را **محدود** می‌کند (autocomplete از دست می‌رود)، بدون annotation هم چک نیست. `satisfies` چک می‌کند **بدون از دست دادن نوع دقیق**:

```ts
type Routes = "/home" | "/about" | "/contact";

// ❌ با annotation: مقادیر دقیق گم می‌شوند
const routes1: Record<Routes, { title: string }> = {
  "/home": { title: "خانه" },
  "/about": { title: "درباره" },
  "/contact": { title: "تماس" },
};
// routes1["/home"].title  → string (خوب) اما...

// ✅ با satisfies: چک + حفظ نوع دقیق
const routes2 = {
  "/home": { title: "خانه" },
  "/about": { title: "درباره" },
  "/contact": { title: "تماس" },
} satisfies Record<Routes, { title: string }>;

routes2["/home"].title;    // نوع دقیق حفظ شده — autocomplete کامل!
// و اگر کلیدی جا بیفتد یا اضافه شود → ارور کامپایل
```

## ۱۱.۴ — الگوهای پیشرفته‌ی پرکاربرد

```ts
// keyof — کلیدهای یک تایپ:
type UserKeys = keyof User;         // "id" | "name" | "email" | "age"

// lookup type — نوع یک فیلد:
type UserEmail = User["email"];     // string

// تایپ آرایه از مقدار:
const colors = ["red", "green", "blue"] as const;
type Color = (typeof colors)[number];   // "red" | "green" | "blue"

// exclude از union:
type EditableKeys = Exclude<keyof User, "id">;   // بدون id!
```

## ۱۱.۵ — strict mode و نکات دیباگ TS

```ts
// strict: true یعنی این‌ها فعال‌اند:
// strictNullChecks: null/undefined را جدی بگیر (مهم‌ترین!)
// noImplicitAny: هرجا TS نفهمید، ارور بده (any ضمنی ممنوع)

// ارورهای TS را چطور بخوانی:
// "Type 'string' is not assignable to type 'number'"
// = «رشته به عدد نمی‌رود» — خطا را از انتها به ابتدا بخوان

// type assertion — وقتی «تو بهتر می‌دانی» (کمتر استفاده کن!):
const el = document.getElementById("input") as HTMLInputElement;
el.value = "...";
// ⚠️ as یعنی «TS ساکت شو» — مسوولیت باگ با توست
// (non-null assertion: x! یعنی «قطعاً null نیست» — با چک پشتش!)
```

---

## ✅ جمع‌بندی فصل

- Utility ها: Partial (آپدیت) / Pick / Omit / Record / Readonly / ReturnType
- **Single source of truth**: تایپ پایه + مشتقات
- type guard با `is` برای داده unknown (API)
- `satisfies` = چک بدون از دست دادن دقت
- keyof / lookup / as const / Exclude — ترکیب‌ها قدرت واقعی‌اند
- strict: true همیشه؛ `as` و `!` را کم و آگاهانه استفاده کن

## 📝 تمرین فصل ۱۱

1. از `interface Invoice { id, customerId, total, createdAt }`: تایپ‌های `InvoiceDraft` (بدون id و createdAt)، `InvoiceSummary` (فقط id و total) و دیکشنری `InvoicesById` بساز — با utility ها.
2. type guard بنویس: `isStringArray(data: unknown): data is string[]` — و در تابع مصرفی استفاده‌اش کن.
3. `satisfies` را روی یک آبجکت config با کلیدهای union امتحان کن — یک کلید غلط اضافه کن و ارور را ببین.
4. با `as const` و lookup، تایپ `Theme = "light" | "dark"` را از یک آرایه استخراج کن (بدون نوشتن union دستی).
5. چالش: تابعی بنویس که فقط کلیدهای string دارِ یک آبجکت را بپذیرد: `K extends keyof T` + شرط `T[K] extends string`.

<details><summary>جواب‌ها</summary>

```ts
// ۱
type InvoiceDraft = Omit<Invoice, "id" | "createdAt">;
type InvoiceSummary = Pick<Invoice, "id" | "total">;
type InvoicesById = Record<string, Invoice>;

// ۲
function isStringArray(data: unknown): data is string[] {
  return Array.isArray(data) && data.every((x) => typeof x === "string");
}

// ۳
type Page = "home" | "about";
const titles = {
  home: "خانه",
  about: "درباره",
  extra: "❌"        // ارور: Object literal may only specify known keys
} satisfies Record<Page, string>;

// ۴
const themes = ["light", "dark"] as const;
type Theme = (typeof themes)[number];   // "light" | "dark"

// ۵
function pickStrings<T, K extends keyof T>(obj: T, key: K): T[K] | undefined
  // راه تمیز با قید:
function stringProps<T extends Record<string, unknown>, K extends keyof T>(
  obj: T, key: K
): T[K] {
  return obj[key];
}
```
</details>

➡️ **فصل بعد:** TS در پروژه واقعی — تایپ API، کامپوننت‌های React و zod!
