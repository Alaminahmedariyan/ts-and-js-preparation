# TypeScript Details (বিস্তারিত বোঝার জন্য)

প্রতিটা টপিক সহজ বাংলায়, কোড উদাহরণসহ। ইন্টারভিউয়ের ছোট ইংরেজি উত্তরের জন্য `ts-interview.md` দেখো।

**ক্রম:** Tier 1 (সবচেয়ে বেশি আসে) → Tier 2 → Tier 3। নম্বর `ts-interview.md` এর সাথে মেলানো।

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. `type` বনাম `interface` ⭐⭐⭐

```ts
interface User { name: string; age: number }
type User = { name: string; age: number };
```

**Extend করা:**
```ts
interface Admin extends User { role: string }   // interface: extends
type Admin = User & { role: string };           // type: &
```

**Declaration merging (শুধু interface এ):**
```ts
interface Car { brand: string }
interface Car { year: number }
// Car = { brand: string; year: number }

type Bike = { brand: string };
type Bike = { year: number }; // ❌ duplicate
```

**`type` দিয়ে বেশি কিছু করা যায়:**
```ts
type ID = string | number;       // union
type Point = [number, number];   // tuple
```

| বিষয় | `interface` | `type` |
|---|---|---|
| Object shape | হ্যাঁ | হ্যাঁ |
| Extend | `extends` | `&` |
| Declaration merging | হ্যাঁ | না |
| Union, tuple, primitive | না | হ্যাঁ |
| Class এ `implements` | হ্যাঁ | হ্যাঁ |

**কখন কোনটা:** object/class এর shape → `interface`; union/tuple/জটিল → `type`; টিমে একই নিয়ম।

---

## 2. `any`, `unknown`, `never` ⭐⭐⭐

**`any`: TypeScript বন্ধ।** বিপজ্জনক।
```ts
let x: any = "hello";
x.foo.bar();   // TS চুপ, কিন্তু runtime এ crash!
```

**`unknown`: টাইপ জানি না, কিন্তু নিরাপদে চাই।** ব্যবহারের আগে যাচাই করতে বাধ্য।
```ts
let y: unknown = "hello";
// y.toUpperCase();   ❌
if (typeof y === "string") {
  y.toUpperCase();    // ✅
}
```

**`never`: এই মান কখনো আসে না।**
```ts
function fail(msg: string): never {
  throw new Error(msg);
}

type Shape = "circle" | "square";
function area(s: Shape) {
  if (s === "circle") return 1;
  if (s === "square") return 2;
  const _check: never = s; // নতুন Shape ভুলে গেলে error
}
```

| | `any` | `unknown` | `never` |
|---|---|---|---|
| কী রাখা যায় | সব | সব | কিছুই না |
| ব্যবহারের আগে যাচাই | লাগে না | লাগে | প্রযোজ্য না |
| নিরাপদ? | না | হ্যাঁ | হ্যাঁ |

অজানা ডেটায় `any` না, **`unknown`**।

---

## 3. Generics ⭐⭐⭐

`any` দিলে টাইপ হারায়:
```ts
function identity(x: any): any { return x; }
const a = identity("hello"); // a: any
```

Generic দিয়ে: "যে টাইপ আসবে, সেই টাইপই ফেরত"।
```ts
function identity<T>(x: T): T { return x; }

const a = identity("hello"); // string ✅
const b = identity(42);      // number ✅
```

`T` হলো **টাইপের ভ্যারিয়েবল**। উপমা: বাক্সের গায়ে লেখা "যা রাখবে, বের করলেও সেটাই পাবে"।

---

## 4. Union বনাম Intersection ⭐⭐⭐

- **Union `|`:** এটা **অথবা** ওটা
- **Intersection `&`:** এটা **এবং** ওটা দুটোই

```ts
type ID = string | number;
let id: ID = "abc";  // ✅
id = 123;            // ✅
id = true;           // ❌

type Person = { name: string };
type Employee = { salary: number };
type Staff = Person & Employee;
const s: Staff = { name: "Alamin", salary: 50000 }; // দুটোই লাগবে
```

**Union এ narrowing:**
```ts
function print(id: string | number) {
  if (typeof id === "string") {
    id.toUpperCase();   // এখানে string
  } else {
    id.toFixed(2);      // এখানে number
  }
}
```

---

## 5. Discriminated union ⭐⭐⭐

সব সদস্যে একটা common property, যার মান আলাদা literal। এটা দেখে TS চেনে।

```ts
type Result =
  | { status: "success"; data: string }
  | { status: "error"; message: string }
  | { status: "loading" };

function show(r: Result): string {
  switch (r.status) {
    case "success": return r.data;       // data আছে
    case "error":   return r.message;    // message আছে
    case "loading": return "লোড হচ্ছে...";
    default: {
      const _exhaustive: never = r;      // নতুন case ভুললে error
      return _exhaustive;
    }
  }
}
```

**কোথায়:** API response, React UI state, Redux action, ফর্মের ধাপ।
**কেন ভালো:** অসম্ভব অবস্থা টাইপেই বানানো যায় না।

---

## 6. Utility types ⭐⭐⭐

```ts
interface User {
  id: number;
  name: string;
  email?: string;
}
```

| Utility | কী করে | ফলাফল |
|---|---|---|
| `Partial<T>` | সব **ঐচ্ছিক** | `{ id?; name?; email? }` |
| `Required<T>` | সব **বাধ্যতামূলক** | `{ id; name; email }` |
| `Pick<T, K>` | **বাছাই করা** রাখে | `Pick<User, "id" \| "name">` |
| `Omit<T, K>` | **বাদ** দেয় | `Omit<User, "id">` |
| `Record<K, V>` | key `K`, মান `V` | `Record<string, number>` |

```ts
function updateUser(id: number, changes: Partial<User>) {}
updateUser(1, { name: "Rahim" });

type UserPreview = Pick<User, "id" | "name">;
type NewUser = Omit<User, "id">;

type Roles = Record<"admin" | "user", boolean>;
const roles: Roles = { admin: true, user: false };
```

---

## 7. API response নিরাপদে টাইপ করা ⭐⭐⭐

সবচেয়ে বড় ভুল `as` দিয়ে জোর করে টাইপ বসানো:
```ts
// ❌ বিপজ্জনক
const user = (await res.json()) as User;
// সার্ভার { id: "abc" } পাঠালে TS জানবেই না
```

**নিরাপদ: Zod দিয়ে runtime যাচাই।**
```bash
pnpm add zod
```

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email().optional(),
});

type User = z.infer<typeof UserSchema>;   // schema থেকে টাইপ

async function getUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);

  const json: unknown = await res.json();  // আগে unknown
  return UserSchema.parse(json);           // ভুল হলে এখানেই error
}
```

**ধাপ:** (১) `unknown` ধরো (২) schema দিয়ে যাচাই (৩) তারপর টাইপড ব্যবহার।

```ts
type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; message: string };
```

---

## 8. অজানা backend ডেটা সামলানো ⭐⭐⭐

**স্তর ১:** `any` না, `unknown`।

**স্তর ২: Type guard**
```ts
interface User { id: number; name: string }

function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    typeof (value as any).id === "number" &&
    "name" in value &&
    typeof (value as any).name === "string"
  );
}

const data: unknown = await res.json();
if (isUser(data)) {
  console.log(data.name);   // User ✅
}
```

**স্তর ৩: Zod `safeParse`** (error ছোড়ে না)
```ts
const result = UserSchema.safeParse(data);
if (result.success) {
  console.log(result.data.name);
} else {
  console.log(result.error.issues);
}
```

**অভ্যাস:** ঐচ্ছিক ফিল্ডে `?.` ও `??`; শুধু যা দরকার তাই validate; monorepo তে schema শেয়ার।

---

## 9. Optional chaining `?.` ও Nullish coalescing `??` ⭐⭐⭐

```ts
const user = { name: "Alamin", address: null };

// user.address.city        // ❌ crash
console.log(user.address?.city);   // undefined
console.log(user.getName?.());     // method থাকলে তবেই call
console.log(user.hobbies?.[0]);    // array থাকলে তবেই index

const port = config.port ?? 3000;  // শুধু null/undefined এ default
```

**`??` বনাম `||`:**
```ts
const count = 0;
console.log(count || 10);  // 10  ❌ ০ কে "নেই" ধরল
console.log(count ?? 10);  // 0   ✅
```

```ts
const city = user.address?.city ?? "অজানা";
```

---

## 10. Type inference ⭐⭐⭐

```ts
let name = "Alamin";     // string
let age = 25;            // number
// name = 10;            ❌

const nums = [1, 2, 3];  // number[]

function add(a: number, b: number) {
  return a + b;          // return: number (TS বোঝে)
}
```

```ts
let a = "hello";    // string
const b = "hello";  // "hello" (literal)
```

**নিজে লিখব:** ফাংশনের parameter, মান ছাড়া declare (`let x: string;`), export করা ফাংশনের return টাইপ। বাকিটা inference।

---

# Tier 2: প্রায়ই আসে

## 11. Structural typing ⭐⭐

Java/C# এ টাইপ মেলে **নামে**, TypeScript এ **আকারে**। "Duck typing"।

```ts
interface Person { name: string }

const student = { name: "Alamin", grade: 10 };

function greet(p: Person) { console.log(p.name); }

greet(student); // ✅ name: string আছে, তাই চলে
```

**ব্যতিক্রম (excess property check):**
```ts
greet({ name: "Alamin", grade: 10 }); // ❌ সরাসরি literal এ বাড়তি property
```

---

## 12. `readonly` ⭐⭐

```ts
interface User {
  readonly id: number;
  name: string;
}

const u: User = { id: 1, name: "Alamin" };
u.name = "Rahim";  // ✅
u.id = 2;          // ❌

const nums: readonly number[] = [1, 2, 3];
nums.push(4);      // ❌

type Locked = Readonly<User>;
const config = { mode: "dark" } as const;
```

**সীমাবদ্ধতা:**
1. শুধু **compile time**। Runtime এ টাইপ থাকে না।
2. **Shallow।**
```ts
const data: Readonly<{ tags: string[] }> = { tags: ["a"] };
data.tags.push("b"); // ✅ চলে!
```
Runtime এ আটকাতে `Object.freeze()`।

---

## 13. `keyof` ও `typeof` ⭐⭐

```ts
interface User { name: string; age: number }
type UserKeys = keyof User;      // "name" | "age"

const user = { name: "Alamin", age: 25 };
type UserType = typeof user;     // { name: string; age: number }
```

**একসাথে:**
```ts
const colors = {
  red: "#ff0000",
  green: "#00ff00",
  blue: "#0000ff",
} as const;

type ColorName = keyof typeof colors;  // "red" | "green" | "blue"

function getColor(name: ColorName) { return colors[name]; }
getColor("red");     // ✅
getColor("yellow");  // ❌
```

নতুন color যোগ করলে `ColorName` নিজে আপডেট হয়।

---

## 14. Mapped types ⭐⭐

গঠন `[K in keyof T]`: "T এর প্রতিটা key এর জন্য..."

```ts
interface User { name: string; age: number }

type MyPartial<T> = { [K in keyof T]?: T[K] };
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };
type Flags<T> = { [K in keyof T]: boolean };

type A = MyPartial<User>;  // { name?: string; age?: number }
type B = Flags<User>;      // { name: boolean; age: boolean }

type FormErrors<T> = { [K in keyof T]?: string };
```

---

## 15. Conditional types ⭐⭐

গঠন `T extends U ? X : Y`

```ts
type IsString<T> = T extends string ? "হ্যাঁ" : "না";
type A = IsString<string>;  // "হ্যাঁ"
type B = IsString<number>;  // "না"

type Unwrap<T> = T extends any[] ? T[number] : T;
type C = Unwrap<string[]>;  // string
type D = Unwrap<number>;    // number

type ToArray<T> = T extends any ? T[] : never;
type E = ToArray<string | number>; // string[] | number[] (distributive)
```

`NonNullable`, `Exclude`, `ReturnType` এভাবেই বানানো।

---

## 16. `infer` ⭐⭐

শুধু conditional type এর ভেতরে। "এই জায়গার টাইপ ধরে নাম দাও"।

```ts
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() { return { name: "Alamin", age: 25 }; }
type U = MyReturnType<typeof getUser>; // { name: string; age: number }

type ItemOf<T> = T extends (infer U)[] ? U : never;
type A = ItemOf<string[]>;  // string

type Awaited2<T> = T extends Promise<infer V> ? V : T;
type B = Awaited2<Promise<number>>;  // number
```

---

## 17. বড় codebase এ strict typing ⭐⭐

**১. `tsconfig.json`:**
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

**২. ESLint:** `@typescript-eslint/no-explicit-any`

**৩. CI তে টাইপ চেক:**
```bash
pnpm tsc --noEmit
```

**৪.** বাইরের ডেটা Zod দিয়ে যাচাই।
**৫.** `as` ও `!` কম, code review তে কারণ জিজ্ঞেস করো।
**৬.** টাইপ একটা জায়গায় রাখো (`types/` বা shared package)।
**৭.** পুরনো JS: `allowJs: true` দিয়ে শুরু, ফাইল ধরে `.ts` এ নাও, তারপর `strict`।

---

## 18. Runtime এ টাইপের কী হয় ⭐⭐

**টাইপ runtime এ থাকে না** (**type erasure**)।

```ts
interface User { name: string }

if (x instanceof User) {}        // ❌ Error
if (typeof x === "User") {}      // ❌ typeof শুধু "string", "number" ইত্যাদি
```

**Runtime যাচাই:** `typeof`, `in`, `Array.isArray`, `instanceof` (class), type guard, discriminated union এর `status`, Zod।

**থাকে:** `class`, `enum`। **মুছে যায়:** `interface`, `type`, generics, `: string`।

```ts
class Dog {}
const d = new Dog();
console.log(d instanceof Dog);  // ✅
```

---

## 19. TypeScript কীভাবে JS এ কম্পাইল হয় ⭐⭐

1. **Parse:** `.ts` পড়ে AST বানায়
2. **Type check:** ভুল থাকলে error
3. **Emit:** টাইপ মুছে `.js` বানায়

```ts
// input.ts
const add = (a: number, b: number): number => a + b;
// output.js
const add = (a, b) => a + b;
```

- `target`: কোন JS সংস্করণ (`ES2022` ইত্যাদি)
- `module`: ESM না CommonJS

**গুরুত্বপূর্ণ:** Next.js, Vite, esbuild, SWC শুধু টাইপ মুছে, **চেক করে না**। তাই আলাদা করে:
```bash
pnpm tsc --noEmit
```

---

## 20. TypeScript কীভাবে runtime error ঠেকায় ⭐⭐

```ts
interface User { name: string; age: number }

function greet(user: User) {
  return user.nmae;        // ❌ typo
}

greet({ name: "Alamin" }); // ❌ age নেই
greet("Alamin");           // ❌ string

let email: string | null = null;
email.toLowerCase();       // ❌ null হতে পারে
```

**ধরে:** ভুল বানান, ভুল টাইপের argument, `null`/`undefined` এ property, `switch` এ case ভুলে যাওয়া, ভুল return।

**সীমা:** Runtime এ API থেকে ভুল ডেটা এলে আটকাতে পারে না, তাই validation লাগে।

---

## 21. Generic constraints ⭐⭐

`T` যেকোনো টাইপ হতে পারে, তাই `T` এর property ব্যবহারে TS আপত্তি করে। `extends` দিয়ে সীমা দাও।

```ts
// ❌
function getLength<T>(x: T) { return x.length; }

// ✅
function getLength<T extends { length: number }>(x: T) {
  return x.length;
}

getLength("hello");   // ✅
getLength([1, 2, 3]); // ✅
getLength(123);       // ❌
```

**`keyof` দিয়ে:**
```ts
function getProp<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

const user = { name: "Alamin", age: 25 };
getProp(user, "name");   // ✅
getProp(user, "email");  // ❌
```

---

# Tier 3: মাঝে মাঝে আসে

## 22. Generic inference ⭐

```ts
function first<T>(arr: T[]): T { return arr[0]; }

const a = first([1, 2, 3]);       // T = number
const b = first(["x", "y"]);      // T = string
const c = first<boolean>([true, false]); // নিজে বলা

const d = first([]);            // T = never, ঝামেলা
const e = first<number>([]);    // স্পষ্ট করে বলো
```

---

## 23. `void` বনাম `never` ⭐

- **`void`:** শেষ হয়, কিন্তু কাজের মান ফেরত দেয় না
- **`never`:** কখনো শেষই হয় না

```ts
function logMessage(msg: string): void {
  console.log(msg);
}

function crash(msg: string): never {
  throw new Error(msg);
}

function forever(): never {
  while (true) {}
}
```

---

## 24. DOM typing ⭐

```ts
const box = document.getElementById("box");   // HTMLElement | null
box.textContent = "hi";                       // ❌ null হতে পারে
if (box) box.textContent = "hi";              // ✅

const input = document.querySelector<HTMLInputElement>("#name");
if (input) console.log(input.value);          // ✅
```

**Event:**
```ts
btn?.addEventListener("click", (e: MouseEvent) => console.log(e.clientX));
```

**React:**
```tsx
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);
};
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
};

const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();
```

`as HTMLInputElement` বা `!` দিয়ে null ঢাকলে element না থাকলে runtime এ crash।

---

## 25. ESM বনাম CommonJS ⭐

```js
// CommonJS
const express = require("express");
module.exports = { add };

// ESM
import express from "express";
export const add = (a, b) => a + b;
```

| বিষয় | CommonJS | ESM |
|---|---|---|
| Syntax | `require` / `module.exports` | `import` / `export` |
| লোড | runtime | আগে বিশ্লেষণ (static) |
| Tree shaking | কঠিন | সহজ |
| Top-level `await` | না | হ্যাঁ |
| Browser এ | সরাসরি না | সরাসরি |

**TS এ:**
```json
{
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "esModuleInterop": true
  }
}
```
- `"module": "CommonJS"` দিলে `import` → `require`
- `esModuleInterop: true` দিলে CJS লাইব্রেরি `import x from "x"` দিয়ে আনা যায়
- Node এ ESM চালালে import এ `.js` লিখতে হয় (ফাইল `.ts` হলেও):
```ts
import { add } from "./math.js";
```
- Next.js/Vite এ bundler সামলায়।

---

## 26. Function overloads ⭐

```ts
function format(x: string): string;
function format(x: number): number;

function format(x: string | number) {
  if (typeof x === "string") return x.toUpperCase();
  return x * 2;
}

const a = format("hi");  // string
const b = format(5);     // number
```

শেষের implementation signature বাইরে থেকে call করা যায় না। সম্ভব হলে generic/union দিয়ে সমাধান করো।

---

## 27. Default generic type ⭐

```ts
interface ApiResponse<T = string> {
  data: T;
  status: number;
}

const a: ApiResponse = { data: "ok", status: 200 };
const b: ApiResponse<number> = { data: 42, status: 200 };
const c: ApiResponse<{ id: number }> = { data: { id: 1 }, status: 200 };
```

---

## 28. Template literal types ⭐

```ts
type Greeting = `Hello, ${string}`;
const a: Greeting = "Hello, Alamin";  // ✅
const b: Greeting = "Hi, Alamin";     // ❌

type Color = "red" | "blue";
type Size = "sm" | "lg";
type ClassName = `${Color}-${Size}`;
// "red-sm" | "red-lg" | "blue-sm" | "blue-lg"

type Event = "click" | "focus";
type Handler = `on${Capitalize<Event>}`;  // "onClick" | "onFocus"
```

---

## 29. Index signature ⭐

```ts
interface Scores {
  [subject: string]: number;
}

const scores: Scores = { math: 90, english: 85 };
scores.science = 70;     // ✅
scores.history = "A";    // ❌
```

**সীমাবদ্ধতা:**

১. সব property কে index signature এর টাইপ মানতে হয়:
```ts
interface Bad {
  [key: string]: number;
  name: string;  // ❌
}
```

২. না থাকা key এও TS `number` বলে:
```ts
const x = scores.physics;  // TS: number, আসলে undefined
```
সমাধান: `number | undefined` দাও বা `noUncheckedIndexedAccess` চালু করো।

৩. key জানা থাকলে `Record<"math" | "english", number>` ব্যবহার করো।

---

## 30. টাইপহীন থার্ড-পার্টি লাইব্রেরি ⭐

"Could not find a declaration file for module" error এর সমাধান:

**ধাপ ১:** `@types` প্যাকেজ আছে কিনা দেখো
```bash
pnpm add -D @types/lodash
```

**ধাপ ২:** দ্রুত সমাধান (সব `any`), `src/types/untyped-lib.d.ts`:
```ts
declare module "untyped-lib";
```

**ধাপ ৩:** শুধু ব্যবহৃত অংশের টাইপ লেখো (সবচেয়ে ভালো):
```ts
declare module "untyped-lib" {
  export function parse(input: string): { id: number; name: string };
  export const version: string;
}
```

```ts
import { parse } from "untyped-lib";
const result = parse("...");   // { id: number; name: string }
```

**খেয়াল:** `.d.ts` ফাইল `tsconfig.json` এর `include` এ থাকতে হবে। ডকুমেন্টেশন দেখে সঠিক টাইপ লেখো।
