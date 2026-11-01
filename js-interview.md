# JavaScript Interview Answers

ইন্টারভিউতে বলার জন্য ছোট ও সহজ ইংরেজি উত্তর, সাথে বাংলা মানে। বিস্তারিত বোঝার জন্য `js-details.md` পড়ো।

**কীভাবে পড়বে:** আগে Tier 1 (সবচেয়ে বেশি আসে), তারপর Tier 2, শেষে Tier 3।

| চিহ্ন | মানে |
|---|---|
| ⭐⭐⭐ | প্রায় প্রতিটা ইন্টারভিউতে আসে |
| ⭐⭐ | প্রায়ই আসে |
| ⭐ | মাঝে মাঝে আসে |

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. What is a closure? ⭐⭐⭐

> "A closure is a function that remembers the variables from its outer scope, even after the outer function has finished. For example, in a counter function, the inner function keeps the `count` variable. I use closures for private data and in React hooks.
>
> A closure can cause a memory leak when it holds a big object and stays alive for a long time, like an event listener that is never removed."

**বাংলা মানে:** Closure হলো এমন ফাংশন যে বাইরের স্কোপের ভ্যারিয়েবল মনে রাখে, বাইরের ফাংশন শেষ হলেও। আমি এটা private data আর React hooks এ ব্যবহার করি। বড় object ধরে রেখে অনেকদিন বেঁচে থাকলে (যেমন remove না করা event listener) memory leak হতে পারে।

---

## 2. Explain the event loop, call stack, and microtask queue ⭐⭐⭐

> "JavaScript is single-threaded, so it uses the event loop for async work. Normal code runs on the call stack. Async callbacks wait in queues. There are two main queues: the microtask queue for Promises, and the task queue for `setTimeout`. When the call stack is empty, the event loop runs all microtasks first, and then takes one task from the task queue.
>
> That is why a Promise runs before `setTimeout` with zero delay."

**বাংলা মানে:** JavaScript single-threaded, তাই async কাজের জন্য event loop ব্যবহার করে। সাধারণ কোড call stack এ চলে। Stack খালি হলে event loop আগে সব microtask (Promise) চালায়, তারপর task queue থেকে একটা task (setTimeout) নেয়। তাই Promise আগে চলে।

**মনে রাখার ট্রিক:** sync → microtask → task।

---

## 3. How does `this` behave? ⭐⭐⭐

> "The value of `this` depends on how a function is called, not where it is written. In a normal function call, it is `undefined` in strict mode. In an object method, it is that object. Arrow functions do not have their own `this`; they take it from the outer scope. In an event handler with a normal function, `this` is the element that got the event."

**বাংলা মানে:** `this` নির্ভর করে ফাংশন কীভাবে call হয়েছে তার উপর। Object method এ `this` হলো সেই object। Arrow function এর নিজের `this` নেই, বাইরের স্কোপ থেকে নেয়। Normal function দিয়ে event handler হলে `this` হলো যে element এ event হয়েছে সেটা।

---

## 4. Explain hoisting ⭐⭐⭐

> "Hoisting means JavaScript moves declarations to the top of their scope before running the code. A `var` variable is hoisted and its value is `undefined`. `let` and `const` are hoisted too, but not initialized, so using them early gives a ReferenceError. Function declarations are fully hoisted. In modern code, I use `const` by default and avoid `var`."

**বাংলা মানে:** Hoisting মানে কোড চালানোর আগে declaration গুলো স্কোপের উপরে চলে যায়। `var` এর মান `undefined`। `let`/`const` initialize হয় না, তাই আগে ব্যবহার করলে ReferenceError। Function declaration পুরো hoist হয়।

---

## 5. What is the Temporal Dead Zone? ⭐⭐⭐

> "The temporal dead zone is the time between the start of a scope and the line where a `let` or `const` variable is declared. In this time, using the variable gives a ReferenceError. It exists to catch bugs early, because with `var`, a mistake silently gives `undefined`."

**বাংলা মানে:** TDZ হলো স্কোপ শুরু থেকে `let`/`const` declare করার লাইন পর্যন্ত সময়। এই সময়ে ব্যবহার করলে ReferenceError আসে। এটা আছে যাতে বাগ তাড়াতাড়ি ধরা পড়ে।

---

## 6. Difference between Promise.all, allSettled, race, and any ⭐⭐⭐

> "All four run many Promises at the same time, but return results differently. `Promise.all` waits for all to succeed, and fails immediately if one fails. `Promise.allSettled` waits for all to finish and gives the status of each, so it never fails. `Promise.race` returns the first Promise that finishes, success or failure. `Promise.any` returns the first Promise that succeeds, and fails only if all fail.
>
> I use `all` for loading several APIs for a dashboard, and `race` to add a timeout to a request."

**বাংলা মানে:** `all` সবগুলো সফল হতে হবে, একটা fail করলেই fail। `allSettled` সবার শেষ পর্যন্ত অপেক্ষা করে ও প্রতিটার status দেয়। `race` যেটা আগে শেষ হয় তার ফল দেয়। `any` প্রথম সফলটা দেয়।

---

## 7. Difference between async/await and Promises ⭐⭐⭐

> "`async/await` is syntactic sugar over Promises. It makes async code look like normal code. An `async` function always returns a Promise. When the code reaches `await`, it pauses only that function, not the whole program. The rest runs later as a microtask. I use `try/catch` for error handling."

**বাংলা মানে:** `async/await` হলো Promise এর সহজ লেখার ধরন। `async` ফাংশন সবসময় Promise ফেরত দেয়। `await` শুধু ওই ফাংশন থামায়, পুরো প্রোগ্রাম না।

---

## 8. Debounce vs throttle ⭐⭐⭐

> "Debounce runs a function only after the user stops triggering the event for a given time. I use it for a search box. Throttle runs a function at most once in a fixed time, even if the event fires many times. I use it for scroll or resize events. Both reduce unnecessary function calls."

**বাংলা মানে:** Debounce ব্যবহারকারী থামার পর একবার চালায় (search box)। Throttle নির্দিষ্ট সময়ে সর্বোচ্চ একবার চালায় (scroll, resize)।

---

## 9. Explain the prototype chain and property lookup ⭐⭐⭐

> "Every object has a hidden link to another object called its prototype. When I access a property, JavaScript first looks in the object itself, then in its prototype, then in the prototype's prototype, and so on. This is the prototype chain. It ends at `null`, and if the property is not found, the result is `undefined`. This is how methods like `.map()` are shared by all arrays."

**বাংলা মানে:** প্রতিটা object এর একটা prototype থাকে। Property খুঁজলে আগে নিজের কাছে, না পেলে prototype এ, এভাবে উপরে উঠে `null` পর্যন্ত। না পেলে `undefined`।

---

## 10. Shallow copy vs deep copy ⭐⭐⭐

> "A shallow copy copies only the first level. Nested objects are still shared by reference, so changing them changes the original too. A deep copy copies every level, so the new object is fully independent. I use the spread operator for a shallow copy and `structuredClone` for a deep copy. `JSON.parse(JSON.stringify())` loses functions and dates."

**বাংলা মানে:** Shallow copy শুধু প্রথম স্তর কপি করে, ভেতরের object শেয়ার হয়। Deep copy সব স্তর কপি করে। Shallow এর জন্য spread, deep এর জন্য `structuredClone`।

---

## 11. Explain call, apply, and bind ⭐⭐⭐

> "They let me set the value of `this` manually. `call` runs the function immediately, with arguments one by one. `apply` also runs immediately, but with arguments in an array. `bind` does not run the function; it returns a new function with `this` fixed. I use `bind` when a class method loses its `this` as a callback."

**বাংলা মানে:** `call` এখনই চালায় (comma দিয়ে argument), `apply` এখনই চালায় (array দিয়ে argument), `bind` চালায় না, নতুন ফাংশন দেয়।

---

# Tier 2: প্রায়ই আসে

## 12. Promise states and chaining ⭐⭐

> "A Promise has three states: pending, fulfilled, and rejected. Once it is fulfilled or rejected, it never changes. Chaining works because `.then()` always returns a new Promise. The value I return goes to the next `.then()`. If a step fails, the error jumps to the nearest `.catch()`."

**বাংলা মানে:** Promise এর তিন state: pending, fulfilled, rejected। একবার বদলালে আর বদলায় না। `.then()` নতুন Promise ফেরত দেয়, তাই chaining হয়।

---

## 13. What happens if you don't `await` an async function? ⭐⭐

> "I get a Promise object instead of the real value, and the code continues without waiting. This can cause bugs because I might use data before it is ready. If the Promise is rejected and nobody handles it, it becomes an unhandled rejection. So I either use `await` or add `.catch()`."

**বাংলা মানে:** মানের বদলে Promise object পাই, কোড অপেক্ষা না করে এগিয়ে যায়। Reject হলে unhandled rejection হয়।

---

## 14. How would you implement Promise.all manually? ⭐⭐

> "I return a new Promise. I loop through the input, and for each Promise I save the result at its own index, so the order stays the same. I keep a counter. When the counter equals the number of Promises, I resolve with the results array. If any Promise fails, I reject immediately. I wrap each item with `Promise.resolve`, so normal values work too."

**বাংলা মানে:** নতুন Promise ফেরত দিই, প্রতিটার ফল তার নিজের index এ রাখি, counter গুনি, সব শেষ হলে resolve, একটা fail করলে reject।

---

## 15. How do you run async requests with a concurrency limit? ⭐⭐

> "I create a fixed number of workers, for example five. Each worker takes the next task from a shared list, waits for it to finish, and then takes another one. So only five tasks run at the same time. I keep tasks as functions, not Promises, because a Promise starts as soon as it is created. In real projects, I can use `p-limit`."

**বাংলা মানে:** নির্দিষ্ট সংখ্যক worker বানাই, প্রতিটা একটা একটা করে কাজ নেয়। কাজগুলো ফাংশন হিসেবে রাখি।

---

## 16. What causes memory leaks? ⭐⭐

> "A memory leak happens when memory that is no longer needed is not released. Common causes are accidental global variables, timers that are never cleared, event listeners that are never removed, closures that hold big data, and references to deleted DOM elements. In React, I clean up timers and listeners in the `useEffect` return function."

**বাংলা মানে:** অদরকারি মেমোরি ছাড়া না হলে leak হয়। কারণ: global ভ্যারিয়েবল, বন্ধ না করা timer, remove না করা listener, বড় ডেটা ধরা closure, DOM reference।

---

## 17. What is event delegation? ⭐⭐

> "Event delegation means I add one event listener to a parent instead of adding listeners to every child. It works because of event bubbling. I use `event.target` to find which child was clicked. The benefits are lower memory use, and it also works for elements added later."

**বাংলা মানে:** প্রতিটা child এ listener না দিয়ে parent এ একটাই দিই। Event bubbling এর কারণে কাজ করে। পরে যোগ করা element এও কাজ করে।

---

## 18. Explain lexical scoping ⭐⭐

> "Lexical scoping means a variable's scope is decided by where the code is written, not where the function is called. An inner function can access variables of its outer functions, but not the other way around. JavaScript searches from inner to outer scope, which is called the scope chain. Closures are built on this idea."

**বাংলা মানে:** Scope ঠিক হয় কোড কোথায় লেখা হয়েছে তা দিয়ে। ভেতরের ফাংশন বাইরেরটা দেখে, উল্টোটা না। একে scope chain বলে।

---

## 19. Undeclared vs undefined ⭐⭐

> "An undefined variable is declared but has no value yet, so using it gives `undefined`. An undeclared variable was never declared, so using it throws a ReferenceError. One special case is `typeof`: it returns `"undefined"` for an undeclared variable and does not throw."

**বাংলা মানে:** `undefined` মানে declare আছে কিন্তু মান নেই। Undeclared মানে বানানোই হয়নি, ব্যবহার করলে ReferenceError।

---

## 20. How does the execution context work? ⭐⭐

> "An execution context is the environment where JavaScript code runs. There is a global context, and a new function context is created every time a function is called. Each has two phases. In the creation phase, memory is set up for variables and functions, which explains hoisting. In the execution phase, code runs line by line. The call stack manages these contexts."

**বাংলা মানে:** Execution context হলো কোড চলার পরিবেশ। Creation phase এ মেমোরি তৈরি (hoisting এর কারণ), execution phase এ কোড চলে।

---

## 21. Function declaration vs function expression ⭐⭐

> "A function declaration starts with the `function` keyword and has a name. It is fully hoisted, so I can call it before it is defined. A function expression stores a function in a variable. It is not fully hoisted, so calling it early gives an error. I use expressions and arrow functions for callbacks."

**বাংলা মানে:** Declaration পুরো hoist হয়, expression হয় না।

---

## 22. What is currying? ⭐⭐

> "Currying means changing a function that takes many arguments into a chain of functions that take one argument each. So `add(1, 2, 3)` becomes `add(1)(2)(3)`. It is useful because I can fix some arguments early and create a new specialized function, like `double = multiply(2)`."

**বাংলা মানে:** অনেক argument এর ফাংশনকে এক এক argument এর শৃঙ্খলে বদলানো। কিছু argument আগে ঠিক করে নতুন ফাংশন বানানো যায়।

---

## 23. What is immutability and why does it matter? ⭐⭐

> "Immutability means we do not change data directly; we create a new copy with the change. It avoids unexpected side effects and makes bugs easier to find. In React it is required, because React compares references to detect state changes. So I use spread, `map`, and `filter` instead of `push` or direct assignment."

**বাংলা মানে:** ডেটা সরাসরি না বদলে নতুন কপি বানানো। React এ বাধ্যতামূলক, কারণ React reference তুলনা করে।

---

## 24. How does garbage collection work? ⭐⭐

> "JavaScript uses mostly the mark-and-sweep algorithm. It starts from roots like the global object and the call stack, and marks everything reachable. Then it removes everything not marked. So what matters is whether an object is reachable, not whether it is used."

**বাংলা মানে:** Root থেকে যা পৌঁছানো যায় সব চিহ্নিত করে, বাকি মুছে দেয়।

---

## 25. Object.create() vs class-based inheritance ⭐⭐

> "Both give inheritance. `Object.create()` creates an object and sets another object as its prototype directly. A `class` is cleaner syntax for the same idea; it is syntactic sugar over prototypes. I use `class` for bigger projects because it is more organized."

**বাংলা মানে:** `class` হলো prototype এরই সহজ লেখা।

---

## 26. Object.freeze() vs Object.seal() ⭐⭐

> "`Object.freeze()` makes the object fully read-only: I cannot add, delete, or change properties. `Object.seal()` stops adding or deleting properties, but I can still change existing values. Both are shallow, so nested objects can still be changed."

**বাংলা মানে:** `freeze` এ মানও বদলানো যায় না, `seal` এ যায়। দুটোই shallow।

---

# Tier 3: মাঝে মাঝে আসে

## 27. What is function composition? ⭐

> "Function composition means combining small functions into a bigger one, where the output of one becomes the input of the next. It keeps code clean and reusable, because each small function does one job and is easy to test."

**বাংলা মানে:** ছোট ফাংশন জুড়ে বড় ফাংশন, একটার output পরেরটার input।

---

## 28. What is callback hell and how do you solve it? ⭐

> "Callback hell happens when callbacks are nested inside callbacks many times. The code becomes deep and hard to read. The modern solution is Promises with flat `.then()` chains, and even better, `async/await` with one `try/catch`."

**বাংলা মানে:** Callback এর ভেতর callback। সমাধান: Promise, তারপর async/await।

---

## 29. What are pure functions? ⭐

> "A pure function always returns the same output for the same input, and has no side effects. It does not change outside variables or call APIs. Pure functions are easy to test. In React, components and Redux reducers should be pure."

**বাংলা মানে:** একই input এ একই output, কোনো side effect নেই।

---

## 30. Monolithic vs modular architecture ⭐

> "A monolithic structure puts all code in one place, which gets hard to read and test as the project grows. A modular structure splits code into small independent files with `import` and `export`. It is easier to reuse, test, and work on in a team."

**বাংলা মানে:** Monolithic এ সব এক জায়গায়, modular এ ছোট ছোট ফাইলে ভাগ করা।

---

## 31. What are Symbols and their use cases? ⭐

> "A Symbol is a unique and immutable value. Two Symbols are never equal, even with the same description. I use them as object keys to avoid name conflicts, and they don't show up in `Object.keys`. Built-in Symbols like `Symbol.iterator` let custom objects work with `for...of`."

**বাংলা মানে:** Symbol অনন্য মান। নাম সংঘাত এড়াতে key হিসেবে ব্যবহার হয়।

---

## 32. How does Proxy work? ⭐

> "A Proxy wraps an object and lets me intercept operations like reading, writing, or deleting a property, using handler functions called traps, such as `get` and `set`. I would use it for validation, logging, or reactivity. For example, Vue 3 uses Proxy to track state changes."

**বাংলা মানে:** Object কে মুড়ে নিয়ে পড়া/লেখা আটকানো যায়। Validation, logging, reactivity তে কাজে লাগে।

---

## 33. How do browsers optimize JavaScript? ⭐

> "Modern engines like V8 use JIT compilation. They compile frequently used code into fast machine code while the program runs. They also use hidden classes and inline caching to access properties faster when objects have the same shape. To help the engine, I keep object shapes consistent and avoid changing a variable's type."

**বাংলা মানে:** JIT compilation, hidden classes, inline caching। আমরা object এর গঠন ও type একই রাখলে সাহায্য হয়।

---

## 34. When would you use Web Workers? ⭐

> "JavaScript runs on one main thread, so heavy calculations can freeze the UI. A Web Worker runs code in a separate thread, so the page stays responsive. I would use it for heavy tasks like processing large data or images. A worker cannot access the DOM, and it talks to the main thread with `postMessage`."

**বাংলা মানে:** ভারী কাজ আলাদা thread এ চলে, UI আটকায় না। Worker DOM ধরতে পারে না।

---

## শেষ মুহূর্তের রিভিশন (এক লাইনে)

- Closure: ফাংশন বাইরের ভ্যারিয়েবল মনে রাখে
- Event loop: sync → microtask → task
- `this`: call এর উপর নির্ভর, arrow বাইরেরটা নেয়
- Hoisting: `var` undefined, `let/const` TDZ error
- `all/allSettled/race/any`: সবাই / সবার ফল / যে আগে / যে আগে সফল
- Debounce: থামার পর, Throttle: নির্দিষ্ট বিরতিতে
- Shallow: nested শেয়ার, Deep: `structuredClone`
- `call/apply/bind`: comma এখনই / array এখনই / পরে
