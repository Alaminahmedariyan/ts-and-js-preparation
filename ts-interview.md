# TypeScript Interview Answers

ইন্টারভিউতে বলার জন্য ছোট ও সহজ ইংরেজি উত্তর, সাথে বাংলা মানে। বিস্তারিত বোঝার জন্য `ts-details.md` পড়ো।

**কীভাবে পড়বে:** আগে Tier 1, তারপর Tier 2, শেষে Tier 3।

| চিহ্ন | মানে |
|---|---|
| ⭐⭐⭐ | প্রায় প্রতিটা ইন্টারভিউতে আসে |
| ⭐⭐ | প্রায়ই আসে |
| ⭐ | মাঝে মাঝে আসে |

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. Difference between `type` and `interface` ⭐⭐⭐

> "Both describe the shape of an object. An `interface` can be extended with `extends` and supports declaration merging, which means two interfaces with the same name are merged. A `type` is more flexible: it can represent unions, tuples, and primitives, and it uses `&` to combine types. I use `interface` for object shapes and class contracts, and `type` for unions and complex types. Most important is to stay consistent in the team."

**বাংলা মানে:** দুটোই object এর আকার বর্ণনা করে। `interface` এ `extends` ও declaration merging আছে। `type` এ union, tuple, primitive হয়, জোড়া লাগাতে `&`। টিমে একই রকম থাকা সবচেয়ে জরুরি।

---

## 2. Difference between `any`, `unknown`, and `never` ⭐⭐⭐

> "`any` turns off type checking, so it is unsafe and can cause runtime errors. `unknown` can also hold any value, but I must check its type before using it, so it is much safer. `never` represents a value that can never happen, for example the return type of a function that always throws. I also use `never` for exhaustive checks. My rule: avoid `any`, and use `unknown` when I don't know the type, like API data."

**বাংলা মানে:** `any` টাইপ চেক বন্ধ করে, অনিরাপদ। `unknown` ব্যবহারের আগে যাচাই করতে হয়, নিরাপদ। `never` এমন মান যা কখনো ঘটে না। অজানা ডেটায় `unknown`।

---

## 3. What are generics? ⭐⭐⭐

> "Generics let me write reusable code that works with many types while keeping type safety. Instead of `any`, I use a type parameter like `T`. For example, `identity<T>(x: T): T` returns the same type it receives, so if I pass a string, I get a string back."

**বাংলা মানে:** Generics দিয়ে অনেক টাইপের সাথে কাজ করা পুনর্ব্যবহারযোগ্য কোড লেখা যায়, টাইপ নিরাপত্তা বজায় থাকে। `T` হলো টাইপের ভ্যারিয়েবল।

---

## 4. Union vs intersection types ⭐⭐⭐

> "A union type uses `|` and means the value can be one of several types, like `string | number`. An intersection type uses `&` and means the value must have all properties of every type combined. I use union for parameters that accept different types, with narrowing like `typeof` to work safely. I use intersection to combine smaller types into a bigger one."

**বাংলা মানে:** Union `|` মানে "এটা অথবা ওটা"। Intersection `&` মানে "এটা এবং ওটা দুটোই"।

---

## 5. What is a discriminated union? ⭐⭐⭐

> "A discriminated union is a union of object types that all share one common property with a literal value, like `status: 'success'` or `status: 'error'`. TypeScript uses that property to narrow the type, so inside a `switch` it knows exactly which properties are available. I use it for API results and UI states like loading, error, and success. I add a `never` check at the end so TypeScript warns me if I forget a new case."

**বাংলা মানে:** সব সদস্যে একটা common literal property (`status`) থাকে, TS ওটা দেখে টাইপ চেনে। API result ও UI state এ ব্যবহার হয়।

---

## 6. Partial, Required, Pick, Omit, Record ⭐⭐⭐

> "These are built-in utility types that create new types from existing ones. `Partial` makes all properties optional, `Required` makes them all required. `Pick` selects some properties, `Omit` removes some. `Record` creates an object type with specific keys and a value type. For example, I use `Partial<User>` for an update form, and `Omit<User, 'id'>` for creating a new user, because the database generates the id."

**বাংলা মানে:** `Partial` সব ঐচ্ছিক, `Required` সব বাধ্যতামূলক, `Pick` বেছে নাও, `Omit` বাদ দাও, `Record` key-value ম্যাপ।

---

## 7. How do you type API responses safely? ⭐⭐⭐

> "I never trust API data blindly, and I avoid using `as` to force a type, because TypeScript does not check it at runtime. Instead, I treat the response as `unknown` and validate it with a schema library like Zod. Zod gives me runtime validation, and I can infer the TypeScript type from the same schema with `z.infer`, so type and validation never go out of sync."

**বাংলা মানে:** `as` দিয়ে জোর করে টাইপ বসাই না। `unknown` ধরে Zod দিয়ে যাচাই করি। একই schema থেকে টাইপও পাই।

---

## 8. How do you handle unknown backend data? ⭐⭐⭐

> "I first type it as `unknown`, not `any`. Then I validate it. I can write a type guard function that returns `value is User`, or better, use Zod's `safeParse`, which returns a success or failure result without throwing. Only after validation do I use the data as a typed value."

**বাংলা মানে:** আগে `unknown`, তারপর type guard বা `safeParse` দিয়ে যাচাই, তারপরই ব্যবহার।

---

## 9. Optional chaining and nullish coalescing ⭐⭐⭐

> "Optional chaining `?.` safely accesses a property. If the value before it is `null` or `undefined`, it stops and returns `undefined` instead of throwing. Nullish coalescing `??` gives a default value only when the left side is `null` or `undefined`. The difference from `||` is that `||` treats `0`, empty string, and `false` as missing, but `??` does not. So `0 ?? 10` gives `0`, but `0 || 10` gives `10`."

**বাংলা মানে:** `?.` null হলে error না দিয়ে `undefined` দেয়। `??` শুধু null/undefined এ default দেয়, `0` কে "নেই" ধরে না।

---

## 10. What is type inference? ⭐⭐⭐

> "Type inference means TypeScript automatically detects the type from the value, so I don't write it everywhere. For example, `let age = 25` is inferred as `number`. I let TypeScript infer local variables, but I write types explicitly for function parameters and for the return types of important or exported functions."

**বাংলা মানে:** TS মান দেখে টাইপ বোঝে। Parameter ও export করা ফাংশনের return টাইপ নিজে লিখি।

---

# Tier 2: প্রায়ই আসে

## 11. How does structural typing work? ⭐⭐

> "TypeScript uses structural typing, so types are compatible based on their shape, not their names. If an object has all the required properties, it can be used as that type, even if never declared as that type. This is called duck typing. One exception is that object literals are checked for extra properties, which helps catch typos."

**বাংলা মানে:** টাইপ মেলে আকার দেখে, নাম দেখে না।

---

## 12. What is `readonly` and how is it enforced? ⭐⭐

> "`readonly` marks a property so it cannot be reassigned after creation. TypeScript enforces it only at compile time, so there is no protection at runtime, because types are removed after compilation. It is also shallow, so nested objects can still be changed. For real runtime protection, I combine it with `Object.freeze()`."

**বাংলা মানে:** শুধু compile time এ কাজ করে, shallow। Runtime এ `Object.freeze`।

---

## 13. Explain `keyof` and `typeof` together ⭐⭐

> "`keyof` takes a type and gives a union of its keys. `typeof` takes a value and gives its type. I use `keyof typeof` when I have an object value and want its keys as a type. For example, with a `colors` object, `keyof typeof colors` gives `'red' | 'green' | 'blue'`. The type updates automatically when I add keys."

**বাংলা মানে:** মান থেকে টাইপ (`typeof`), টাইপ থেকে key এর union (`keyof`)।

---

## 14. Explain mapped types ⭐⭐

> "A mapped type creates a new type by looping over the keys of an existing type, with the syntax `[K in keyof T]`. For each key, I can change the property type or add modifiers like `?` and `readonly`. Built-in types like `Partial` and `Readonly` are built this way."

**বাংলা মানে:** টাইপের key গুলোর উপর লুপ চালিয়ে নতুন টাইপ বানানো।

---

## 15. What are conditional types? ⭐⭐

> "A conditional type is like an if-else for types. The syntax is `T extends U ? X : Y`: if `T` fits `U`, the result is `X`, otherwise `Y`. Many built-in types, like `Exclude` and `ReturnType`, are built with conditional types."

**বাংলা মানে:** টাইপের জন্য if-else।

---

## 16. How does `infer` work? ⭐⭐

> "`infer` is used inside a conditional type to capture a type and give it a name, so I can use it in the result. For example, `T extends (...args: any[]) => infer R ? R : never` captures the return type of a function as `R`. The built-in `ReturnType` is built this way."

**বাংলা মানে:** Conditional type এর ভেতরে টাইপ ধরে নাম দেওয়া।

---

## 17. How do you enforce strict typing in large codebases? ⭐⭐

> "I enable `strict: true` in `tsconfig`, and add `noUncheckedIndexedAccess`. I use ESLint to forbid `any`, and run `tsc --noEmit` in CI, so code with type errors cannot be merged. I validate external data with Zod, and I avoid `as` and the non-null `!` operator, because they silence the compiler instead of fixing the problem. For old JavaScript projects, I migrate step by step."

**বাংলা মানে:** `strict: true`, ESLint এ `any` নিষিদ্ধ, CI তে `tsc --noEmit`, Zod, `as` ও `!` কম।

---

## 18. What happens to types at runtime? ⭐⭐

> "Types do not exist at runtime. TypeScript removes all types during compilation, which is called type erasure. So `interface` and `type` are gone in the final JavaScript, and I cannot use `instanceof` with an interface. Classes and enums remain, because they are real JavaScript code. For runtime checks, I use `typeof`, `in`, type guards, or Zod."

**বাংলা মানে:** Runtime এ টাইপ থাকে না (type erasure)। Class ও enum থাকে।

---

## 19. How does TypeScript compile to JavaScript? ⭐⭐

> "The TypeScript compiler `tsc` parses the code into an AST, checks the types, and then emits plain JavaScript with all types removed. The `target` option in `tsconfig` decides which JavaScript version it produces. Tools like SWC, esbuild, and Vite only strip the types and don't check them, so I still run `tsc --noEmit` separately."

**বাংলা মানে:** Parse → type check → টাইপ মুছে JS বানানো। Vite/SWC টাইপ চেক করে না।

---

## 20. How does TypeScript help prevent runtime errors? ⭐⭐

> "TypeScript catches errors at compile time, before the code runs. It finds typos, wrong argument types, missing properties, and most importantly, accessing properties on `null` or `undefined`. With `strictNullChecks`, it forces me to handle null cases. But it does not check data at runtime, so for external data like API responses, I also need runtime validation."

**বাংলা মানে:** Compile time এ ভুল ধরে, কিন্তু runtime এ ডেটা যাচাই করে না।

---

## 21. What are generic constraints? ⭐⭐

> "A generic constraint limits what types can be used for a type parameter, using `extends`. For example, `T extends { length: number }` means the type must have a `length` property. This lets me safely use that property inside the function, and TypeScript shows an error if someone passes a type that doesn't fit."

**বাংলা মানে:** `extends` দিয়ে `T` কে সীমিত করা।

---

# Tier 3: মাঝে মাঝে আসে

## 22. How does TypeScript infer generics in functions? ⭐

> "TypeScript infers the generic type from the arguments I pass. For example, with `first([1, 2, 3])`, it understands `T` is `number`. I only write the type explicitly when it cannot infer, like with an empty array."

**বাংলা মানে:** Argument দেখে `T` নিজেই বোঝে।

---

## 23. Difference between `void` and `never` ⭐

> "A `void` function finishes normally but does not return a useful value, like a function that only logs. A `never` function never finishes, because it always throws or runs forever."

**বাংলা মানে:** `void` শেষ হয় কিন্তু কিছু দেয় না, `never` কখনো শেষই হয় না।

---

## 24. How does TypeScript handle DOM typing? ⭐

> "TypeScript has specific types for DOM elements, like `HTMLInputElement`. Methods like `getElementById` return `HTMLElement | null`, so I must check for null first. I use `querySelector<HTMLInputElement>` to get the right type. In React, I use `React.ChangeEvent<HTMLInputElement>` for events and `useRef<HTMLInputElement>(null)` for refs."

**বাংলা মানে:** Null যাচাই করা জরুরি, generic দিয়ে সঠিক element টাইপ।

---

## 25. ESM vs CommonJS in TypeScript projects ⭐

> "CommonJS uses `require` and `module.exports` and loads at runtime. ESM uses `import` and `export`, is the official standard, and is analyzed statically, so tools can do tree shaking. In TypeScript, the `module` option in `tsconfig` decides the output format. I use ESM for new projects and enable `esModuleInterop` to import older CommonJS packages."

**বাংলা মানে:** `tsconfig` এর `module` আউটপুট ঠিক করে। নতুন প্রজেক্টে ESM।

---

## 26. What are function overloads? ⭐

> "Function overloads let me define multiple signatures for one function, so TypeScript knows exactly which return type belongs to which input type. I write overload signatures first, then one implementation. I use overloads only when a simple union or generic cannot describe the input-output relationship."

**বাংলা মানে:** একই ফাংশনের একাধিক signature, input অনুযায়ী ঠিক output টাইপ।

---

## 27. What is a default generic type? ⭐

> "A default generic type is a fallback type used when I don't provide one, like `interface ApiResponse<T = string>`. It works like default parameter values and makes generic types easier to use."

**বাংলা মানে:** Generic এর default মান।

---

## 28. What are template literal types? ⭐

> "Template literal types use the same syntax as JavaScript template strings, but at the type level. I can combine string types, and with unions TypeScript creates all combinations. For example, `${Color}-${Size}` creates every color and size pair. It is useful for event names like `onClick`."

**বাংলা মানে:** টাইপের স্তরে string জোড়া লাগানো।

---

## 29. What is an index signature and its limitations? ⭐

> "An index signature describes an object where I don't know key names in advance, like `{ [key: string]: number }`. Limitations: all other properties must match its type, and TypeScript assumes every key exists, so a missing key gives `number` but is actually `undefined`. When I know the keys, I prefer `Record` with a union of keys, or I enable `noUncheckedIndexedAccess`."

**বাংলা মানে:** অজানা key এর জন্য। সীমা: সব property একই টাইপ মানতে হয়, না থাকা key `undefined` হলেও TS `number` বলে।

---

## 30. How do you type third-party libraries without types? ⭐

> "First, I check if there is an `@types` package, for example `pnpm add -D @types/lodash`. If not, I create a `.d.ts` file. The quick way is `declare module 'lib-name'`, which makes everything `any`. A better way is to write types for only the functions I actually use inside `declare module`."

**বাংলা মানে:** `@types` → `declare module` → নিজে প্রয়োজনীয় অংশের টাইপ লেখা।

---

## শেষ মুহূর্তের রিভিশন (এক লাইনে)

- `type` বনাম `interface`: interface এ merging, type এ union/tuple
- `any/unknown/never`: চেক বন্ধ / যাচাই বাধ্যতামূলক / কখনো ঘটে না
- Union `|` = অথবা, Intersection `&` = এবং
- Discriminated union: common literal ফিল্ড + `never` চেক
- `Partial/Required/Pick/Omit/Record`: ঐচ্ছিক / বাধ্যতামূলক / বাছো / বাদ / ম্যাপ
- API ডেটা: `unknown` + Zod
- `0 ?? 10` = `0`, `0 || 10` = `10`
- Runtime এ টাইপ থাকে না
