# فصل ۱۲ — TypeScript در پروژه واقعی: API، React و zod

> 🎯 **هدف:** جمع‌بندی عملی — سه سناریویی که هر روز در کار واقعی می‌بینی: تایپ‌کردن پاسخ API، تایپ‌کردن کامپوننت‌ها و هوک‌های React، و اعتبارسنجی داده ورودی با zod. این فصل = پل TS به پروژه‌های React/Next تو.

---

## ۱۲.۱ — تایپ‌کردن پاسخ API

مساله: `fetch` همیشه `any` می‌دهد. سه لایه حل:

```ts
// ۱) تایپ داده — مطابق قرارداد بک‌اند:
interface Product {
  id: number;
  title: string;
  price: number;
  inStock: boolean;
  category: { id: number; name: string };
}

// ۲) تایپ wrapper پاسخ:
interface ApiList<T> {
  data: T[];
  total: number;
  page: number;
}

// ۳) تابع fetch تایپ‌دار (generic):
async function fetchJson<T>(url: string): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json() as Promise<T>;    // ⚠️ as: تو به قرارداد API اعتماد می‌کنی
}

// مصرف — تایپ‌سیف کامل:
const res = await fetchJson<ApiList<Product>>("/api/products");
res.data[0].title;        // ✅ autocomplete و چک کامل
res.data[0].titel;        // ❌ ارور کامپایل — غلط املایی گرفته شد!
```

> ⚠️ نکته صادقانه: `as Promise<T>` یعنی «به API اعتماد می‌کنم که همین شکل را می‌فرستد». اگر API غیرقابل‌اعتماد است یا نسخه‌اش عوض می‌شود → **zod** (بخش ۱۲.۳) داده را در runtime هم چک می‌کند.

**ابزار طلایی:** [GraphQL Code Generator] یا [OpenAPI typescript] تایپ‌ها را از سرویس خود API تولید می‌کنند — صفر دست‌نویسی.

---

## ۱۲.۲ — React با TS: props، events، hooks

```tsx
// props کامپوننت — با interface:
interface ProductCardProps {
  product: Product;
  onSelect?: (id: number) => void;    // callback اختیاری
  highlighted?: boolean;
}

function ProductCard({ product, onSelect, highlighted = false }: ProductCardProps) {
  return (
    <div
      className={highlighted ? "ring-2" : ""}
      onClick={() => onSelect?.(product.id)}    // ?. برای callback اختیاری
    >
      <h3>{product.title}</h3>
      <span>{product.price.toLocaleString("fa-IR")}</span>
    </div>
  );
}
```

```tsx
// رویدادها — تایپ‌های React:
import type { ChangeEvent, FormEvent, KeyboardEvent } from "react";

function SearchBox() {
  const [query, setQuery] = useState("");

  function handleChange(e: ChangeEvent<HTMLInputElement>) {
    setQuery(e.target.value);              // ✅ autocomplete کامل
  }
  function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();
  }
  function onKey(e: KeyboardEvent<HTMLInputElement>) {
    if (e.key === "Enter") submit();
  }
  // ...
}
```

```tsx
// useState تایپ‌دار — صریح وقتی inference نمی‌داند:
const [user, setUser] = useState<User | null>(null);        // کاربر یا هیچ
const [status, setStatus] = useState<"idle" | "loading">("idle");
const [items, setItems] = useState<Product[]>([]);

// ترفند: تایپ را از داده استخراج کن (Single source of truth):
type Product = typeof products[0];      // اگر products از جایی import شده!

// useRef با نوع:
const inputRef = useRef<HTMLInputElement>(null);
```

> 💡 `React.ReactNode` برای children و هر JSX ای؛ `React.ComponentProps<typeof Button>` برای «همه props های یک کامپوننت دیگر» — ترکیب به‌جای تکرار!

---

## ۱۲.۳ — zod: اعتبارسنجی runtime + تایپ خودکار ⭐⭐

مشکل باقی‌مانده: تایپ‌های TS در **runtime وجود ندارند** — داده‌ی API/فرم کاربر را در زمان اجرا باید چک کنی. **zod** هر دو را یک‌جا می‌دهد:

```bash
npm i zod
```

```ts
import { z } from "zod";

// ۱. اسکیما را تعریف کن:
const ProductSchema = z.object({
  id: z.number().int().positive(),
  title: z.string().min(1),
  price: z.number().nonnegative(),
  tags: z.array(z.string()).default([]),
});

// ۲. تایپ را از اسکیما استخراج کن — نوشتن دستی ممنوع!:
type Product = z.infer<typeof ProductSchema>;
// { id: number; title: string; price: number; tags: string[] } — دقیق و همیشه همگام!

// ۳. در runtime اعتبارسنجی کن:
const result = ProductSchema.safeParse(apiResponse);
if (result.success) {
  console.log(result.data.title);    // ✅ تایپ‌دار + واقعاً validated
} else {
  console.error(result.error.issues[0].message);   // "Invalid input..."
}

// فرم با zod (همان الگوی Server Action فصل ۸ مرور Next):
const FormSchema = z.object({
  title: z.string().min(3, "عنوان حداقل ۳ حرف"),
  price: z.coerce.number().positive("قیمت باید مثبت باشد"),  // coerce: "۱۲" → ۱۲
});

const parsed = FormSchema.safeParse(Object.fromEntries(formData));
```

**جریان حرفه‌ای کامل:** zod اسکیما → `z.infer` تایپ → validation در مرز (فرم/API) → بعد از آن همه‌جا تایپ‌سیف مطمئن. (در Next: همین اسکیما در Server Action مصرف می‌شود.)

---

## ۱۲.۴ — چک‌لیست TS در پروژه واقعی

- [ ] `strict: true` — همیشه
- [ ] تایپ‌های دامنه در یک فایل (`types/`) — از API/اسکیمای zod مشتق، نه دستی دوباره‌نویسی
- [ ] `any` ممنوع؛ `unknown` + narrowing یا zod
- [ ] `as` فقط با دلیل مستندشده
- [ ] utility ها به‌جای تکرار تایپ
- [ ] `z.infer` = پل runtime و compile time

---

## ✅ جمع‌بندی فصل — و پایان بخش TS!

- fetch تایپ‌دار با generic: `fetchJson<T>` + تایپ پاسخ — و zod برای اعتماد runtime
- React: interface props، event types، `typeof` برای استخراج
- useState با union literal؛ useRef با نوع DOM
- **zod**: schema → `z.infer` → safeParse — تایپ و validation از یک منبع
- TS واقعی یعنی: داده از مرز (API/کاربر) وارد می‌شود → validate می‌شود → بعدش همه‌چیز تایپ‌سیف

## 📝 تمرین فصل ۱۲

1. برای API https://jsonplaceholder.typicode.com/todos تایپ Todo بساز (id, userId, title, completed) و `fetchJson<Todo[]>` را بنویس و مصرف کن — autocomplete را ببین.
2. کامپوننت `TodoItem` با props تایپ‌دار (todo + onToggle) بنویس — و یک props غلط بده تا ارور کامپایل را ببینی.
3. zod schema برای فرم تماس (نام، ایمیل با `.email()`، پیام min 10) بنویس؛ `z.infer` بگیر و یک داده غلط را safeParse کن — پیام‌های ارور را بخوان.
4. چالش: `fetchJson` را با zod ترکیب کن: `fetchValidated<T>(url, schema: z.ZodType<T>): Promise<T>` — که هم fetch کند هم validate.

<details><summary>جواب‌ها</summary>

```ts
// ۱
interface Todo {
  id: number;
  userId: number;
  title: string;
  completed: boolean;
}
const todos = await fetchJson<Todo[]>("https://jsonplaceholder.typicode.com/todos");
todos.filter((t) => !t.completed).length;   // ✅ همه‌چیز autocomplete

// ۲
interface TodoItemProps {
  todo: Todo;
  onToggle: (id: number) => void;
}
function TodoItem({ todo, onToggle }: TodoItemProps) {
  return (
    <li onClick={() => onToggle(todo.id)}>
      {todo.completed ? "✅" : "⬜"} {todo.title}
    </li>
  );
}
// <TodoItem todo={5} .../>  → ❌ Type 'number' is not assignable to type 'Todo'

// ۳
const ContactSchema = z.object({
  name: z.string().min(1, "نام الزامی است"),
  email: z.string().email("ایمیل نامعتبر"),
  message: z.string().min(10, "پیام حداقل ۱۰ کاراکتر"),
});
type Contact = z.infer<typeof ContactSchema>;
ContactSchema.safeParse({ name: "", email: "bad", message: "hi" });
// success: false — issues: سه پیام فارسی بالا

// ۴
async function fetchValidated<T>(url: string, schema: z.ZodType<T>): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const raw: unknown = await res.json();
  return schema.parse(raw);        // parse یا throw — با پیام دقیق
}
```
</details>

🎓 **TS تمام شد!** فصل آخر: چیت‌شیت دوتایی و نقشه راه.
