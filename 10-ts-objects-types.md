# فصل ۱۰ — interface/type، Union و Generics

> 🎯 **هدف:** ابزارهای اصلی مدل‌سازی داده در TS: تعریف شکل‌ها با interface/type، ترکیب با union/intersect، narrowing امن، و generics. بعد از این فصل می‌توانی هر داده‌ای را تایپ کنی.

---

## ۱۰.۱ — interface در برابر type

```ts
// interface — برای شکل آبجکت‌ها:
interface User {
  id: number;
  name: string;
  email?: string;              // اختیاری — مثل ? در JS
  readonly createdAt: Date;    // فقط‌خواندنی
}

// type — انعطاف بیشتر (union، primitive، utility):
type ID = string | number;
type Status = "active" | "banned";
type Users = Record<string, User>;    // utility (فصل ۱۱)

// هر دو قابل استفاده‌اند:
const u: User = { id: 1, name: "علی", createdAt: new Date() };
const uid: ID = 42;
```

| | interface | type |
|---|---|---|
| آبجکت | ✅ | ✅ |
| union/literal | ❌ | ✅ |
| extends / intersection | `extends` | `&` |
| declaration merging | ✅ (دوباره تعریف = ادغام) | ❌ (ارور تکرار) |

> 💡 قاعده تیمی: برای شکل آبجکت‌ها interface (قابل extends و merge)، برای بقیه type. مهم‌تر: یکدستی در پروژه.

### extends و intersection

```ts
interface Person { name: string }
interface Employee { salary: number }

interface Staff extends Person, Employee {}          // ارث‌بری interface
type StaffT = Person & Employee;                     // intersection type — همان معنا

const s: Staff = { name: "علی", salary: 500 };
```

## ۱۰.۲ — Union و Narrowing: نوع شرطی امن ⭐

```ts
type Result = { ok: true; data: string[] } | { ok: false; error: string };

function show(res: Result) {
  // res.data ❌ — نمی‌دانی کدام شکل است!
  if (res.ok) {
    console.log(res.data);        // ✅ TS فهمید شکل اول است
  } else {
    console.error(res.error);     // ✅ و اینجا شکل دوم
  }
}
```

`if (res.ok)` یک **narrowing** است — TS بر اساس چک، نوع را تنگ‌تر می‌کند. انواع narrowing:

```ts
function process(input: string | number | null) {
  if (typeof input === "string") input.toUpperCase();    // typeof
  if (input !== null) input.valueOf();                   // null چک
  if (Array.isArray(input)) {}                           // آرایه
  if (input instanceof Date) input.getTime();            // کلاس
}
```

**Discriminated union** — الگوی طلایی (این را حفظ کن!):

```ts
type FetchState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; message: string };

function render(state: FetchState<string[]>) {
  switch (state.status) {          // کلید تمایز
    case "idle":   return "شروع نشده";
    case "loading": return "در حال بارگذاری...";
    case "success": return state.data.join(",");   // ✅ data فقط اینجا موجود!
    case "error":  return state.message;
  }
}
```

چرا طلایی؟ چون TS **فراموش‌کردن یک حالت را می‌گیرد** و در هر شاخه فقط فیلدهای همان حالت را می‌دهد — دقیقاً ساختار state واقعی اپ‌ها.

## ۱۰.۳ — Enum و as const

```ts
// enum — لیست ثابت نام‌دار:
enum OrderStatus {
  Pending = "PENDING",
  Paid = "PAID",
}
let s: OrderStatus = OrderStatus.Paid;

// ⭐ مدرن‌تر و سبک‌تر — union + as const:
const ORDER_STATUS = {
  pending: "PENDING",
  paid: "PAID",
} as const;                        // مقادیر literal و فقط‌خواندنی می‌شوند
type OrderStatus2 = (typeof ORDER_STATUS)[keyof typeof ORDER_STATUS];
// "PENDING" | "PAID"
```

(as const و typeof نوع‌سازی پیشرفته‌اند — فعلاً الگو را ببین؛ فصل ۱۱ عمیق می‌شود.)

## ۱۰.۴ — Generics: تایپ پارامتری ⭐

مشکل: `function first(arr)` — ورودی/خروجی چه تایپی؟ `any` می‌گیریم و type safety می‌میرد. جواب:

```ts
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const n = first([1, 2, 3]);        // n: number | undefined — خودش فهمید!
const s = first(["a"]);            // s: string | undefined
```

`<T>` = «پارامتر تایپ». TS از آرگومان حدس می‌زند (inference) — مثل پارامترهای عادی ولی برای نوع:

```ts
// چند generic + محدودیت با extends:
function pick<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const user = { name: "علی", age: 25 };
pick(user, "name");    // ✅ type: string
pick(user, "email");   // ❌ ارور! "email" در keyof T نیست

// generic در interface — جواب API:
interface ApiResponse<T> {
  data: T;
  status: number;
}
const res: ApiResponse<User> = await fetchUser();
res.data.name;         // ✅ تایپ‌سیف کامل!
```

**generic های بومی که هر روز می‌بینی:** `Array<T>`، `Promise<T>`، `Map<K,V>`، `useState<string>(...)`، `useActionState<TState,...>` — همه همین الگو.

---

## ✅ جمع‌بندی فصل

- interface برای شکل آبجکت (+ extends)، type برای union و بقیه
- union + narrowing = امنیت واقعی؛ **discriminated union با switch** الگوی طلایی state
- enum مدرن = union literal + as const
- **generics** = تایپ پارامتری: `first<T>(arr: T[])`، `ApiResponse<T>`، `K extends keyof T`
- همه ابزارهای بومی (Array/Promise/useState) generics اند

## 📝 تمرین فصل ۱۰

1. interface `Product` بساز: id، title، price، category، درست‌کردن یک محصول و یک ارور عمدی (فیلد غلط) — ارور editor را بخوان.
2. discriminated union برای «پرداخت»: حالت‌های idle/loading/success(شناسه تراکنش)/error(پیام) — تابع render با switch بنویس.
3. تابع `wrapInArray<T>(value: T): T[]` بنویس و با سه نوع مختلف تست کن.
4. generic محدود: `function pluck<T, K extends keyof T>(items: T[], key: K): T[K][]` — روی products با "title" کار کند و با کلید غلط ارور بدهد.
5. interface User را با `extends` به AdminUser گسترش بده (permissions: string[]).

<details><summary>جواب‌ها</summary>

```ts
// ۱
interface Product {
  id: number;
  title: string;
  price: number;
  category: string;
}
const p: Product = { id: 1, title: "گوشی", price: 50, cat: "mobile" };
// ❌ 'cat' does not exist in type 'Product'. Did you mean 'category'?

// ۲
type PaymentState =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; txId: string }
  | { status: "error"; message: string };

function renderPay(state: PaymentState): string {
  switch (state.status) {
    case "idle":    return "آماده پرداخت";
    case "loading": return "در حال پردازش...";
    case "success": return `تراکنش ${state.txId} موفق ✅`;
    case "error":   return `خطا: ${state.message}`;
  }
}

// ۳
function wrapInArray<T>(value: T): T[] {
  return [value];
}
wrapInArray(5); wrapInArray("a"); wrapInArray({ x: 1 });

// ۴
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((item) => item[key]);
}
pluck(products, "title");   // string[]
pluck(products, "email");   // ❌ ارور کامپایل!

// ۵
interface AdminUser extends User {
  permissions: string[];
}
```
</details>

➡️ **فصل بعد:** TS پیشرفته — Utility Types، type guards اختصاصی و satisfies.
