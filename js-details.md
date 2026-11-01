# JavaScript Details (বিস্তারিত বোঝার জন্য)

প্রতিটা টপিক সহজ বাংলায়, কোড উদাহরণসহ। ইন্টারভিউয়ের ছোট ইংরেজি উত্তরের জন্য `js-interview.md` দেখো।

**ক্রম:** Tier 1 (সবচেয়ে বেশি আসে) → Tier 2 → Tier 3। নম্বর `js-interview.md` এর সাথে মেলানো।

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. Closure ⭐⭐⭐

Closure মানে একটা ফাংশন তার **বাইরের স্কোপের ভ্যারিয়েবল মনে রাখে**, বাইরের ফাংশন শেষ হয়ে গেলেও।

```js
function makeCounter() {
  let count = 0;            // এই ভ্যারিয়েবলটা "আটকে" থাকে
  return function () {
    count++;
    return count;
  };
}

const counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

**কোথায় লাগে:** private ভ্যারিয়েবল, React hooks (`useState`), event handler, `setTimeout`।

**Memory leak কখন:** closure যে ভ্যারিয়েবল ধরে রাখে তা garbage collector মুছতে পারে না। বড় object ধরে থাকলে ও ফাংশন অনেকদিন বেঁচে থাকলে (যেমন remove না করা listener) মেমোরি ভরে যায়।

---

## 2. Event Loop, Call Stack, Microtask Queue ⭐⭐⭐

JavaScript **single-threaded**, এক সময়ে একটা কাজ। তাহলে async হয় কীভাবে?

- **Call Stack:** এখানে কোড চলে, একটা একটা করে।
- **Web APIs:** browser `setTimeout`, `fetch` এর কাজ পেছনে করে।
- **Task Queue (Macrotask):** `setTimeout`, `setInterval`, click callback।
- **Microtask Queue:** `Promise.then`, `await` এর পরের কোড।
- **Event Loop:** দারোয়ান। Stack খালি হলে queue থেকে কাজ তুলে দেয়।

**অগ্রাধিকার:**
1. সব sync কোড শেষ
2. **সব microtask** শেষ
3. **একটা** task
4. আবার microtask, এভাবে চলে

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Output: 1, 4, 3, 2
```

ট্রিক: `1, 4` (sync) → `3` (microtask) → `2` (task)।

---

## 3. `this` কীওয়ার্ড ⭐⭐⭐

`this` নির্ভর করে **ফাংশন কীভাবে call হয়েছে** তার উপর, কোথায় লেখা হয়েছে তার উপর না (arrow function ছাড়া)।

| কোথায় | `this` কী |
|---|---|
| Normal function (সাধারণ call) | `window` বা `undefined` (strict mode) |
| Object এর method | যে object দিয়ে call (`obj.method()` হলে `obj`) |
| Arrow function | নিজের `this` নেই, বাইরের স্কোপের `this` (lexical) |
| Event handler (normal function) | যে element এ event হয়েছে |
| Event handler (arrow function) | বাইরের স্কোপের `this` |

```js
const user = {
  name: "Alamin",
  normal() { console.log(this.name); },       // "Alamin"
  arrow: () => { console.log(this.name); },   // undefined (বাইরের this)
};
user.normal();
user.arrow();

button.addEventListener("click", function () {
  console.log(this); // button element
});
```

---

## 4. Hoisting ⭐⭐⭐

কোড চালানোর আগে JS declaration গুলো "উপরে তুলে রাখে"। কিন্তু আচরণ আলাদা:

| ধরন | Hoist হয়? | আগে ব্যবহার করলে |
|---|---|---|
| `var` | হয়, মান `undefined` | `undefined`, error নেই |
| `let` / `const` | হয়, initialize হয় না | **ReferenceError** |
| Function declaration | পুরো hoist | কাজ করে |
| Function expression | ভ্যারিয়েবলের নিয়ম | `var` হলে `TypeError`, `let/const` হলে `ReferenceError` |

```js
console.log(a); // undefined
var a = 5;

console.log(b); // ReferenceError
let b = 5;

sayHi();        // "Hi"
function sayHi() { console.log("Hi"); }

hello();        // TypeError: hello is not a function
var hello = function () {};
```

---

## 5. Temporal Dead Zone (TDZ) ⭐⭐⭐

স্কোপ শুরু থেকে `let`/`const` লেখার লাইন পর্যন্ত সময়। এই সময়ে ভ্যারিয়েবল আছে, কিন্তু ছোঁয়া যাবে না।

```js
{
  // ---- TDZ শুরু (x এর জন্য) ----
  console.log(x); // ReferenceError
  // ---- TDZ শেষ ----
  let x = 10;
  console.log(x); // 10
}
```

**কেন আছে:** `var` এ ভুল করে আগে ব্যবহার করলে চুপচাপ `undefined` আসত, বাগ ধরা কঠিন ছিল। TDZ সাথে সাথে error দেয়।

---

## 6. Promise.all / allSettled / race / any ⭐⭐⭐

চারটাই একসাথে অনেক Promise চালায়। পার্থক্য ফলাফল দেওয়ায়।

| Method | কখন সফল | কখন ব্যর্থ | ফেরত দেয় |
|---|---|---|---|
| `Promise.all` | **সবগুলো** সফল হলে | **একটা** fail করলেই | সব মানের array |
| `Promise.allSettled` | সবসময় | কখনো না | প্রতিটার status ও মান/error |
| `Promise.race` | যেটা আগে শেষ হয় সেটা সফল হলে | সেটা fail করলে | প্রথম শেষ হওয়াটার ফল |
| `Promise.any` | **প্রথম সফল** | **সবগুলো** fail করলে | প্রথম সফল মান |

**ট্রিক:**
- `all` = "সবাইকে লাগবে, একজন ফেল তো সব ফেল"
- `allSettled` = "সবার ফল দেখাও"
- `race` = "যে আগে শেষ করল (ভালো বা খারাপ)"
- `any` = "যে প্রথম **সফল**"

```js
const ok1 = Promise.resolve("A");
const ok2 = Promise.resolve("B");
const bad = Promise.reject("ভুল");

await Promise.all([ok1, ok2]);        // ["A", "B"]
await Promise.all([ok1, bad]);        // ❌ error: "ভুল"
await Promise.allSettled([ok1, bad]); // [{status:"fulfilled",value:"A"},{status:"rejected",reason:"ভুল"}]
await Promise.race([ok1, bad]);       // "A"
await Promise.any([bad, ok2]);        // "B"
```

**বাস্তবে:** `all` = ড্যাশবোর্ডে ৩টা API, `allSettled` = অনেক ফাইল আপলোডের ফল, `race` = request timeout, `any` = একই ডেটা ৩ সার্ভারে।

---

## 7. async/await বনাম Promise ⭐⭐⭐

`async/await` আলাদা কিছু না, Promise এরই সহজ লেখার ধরন (syntactic sugar)।

```js
// Promise
function getUser() {
  return fetch("/api/user")
    .then((res) => res.json())
    .then((user) => console.log(user));
}

// async/await (একই কাজ)
async function getUser() {
  try {
    const res = await fetch("/api/user");
    const user = await res.json();
    console.log(user);
  } catch (err) {
    console.log(err);
  }
}
```

- `async` ফাংশন **সবসময় Promise ফেরত দেয়**।
- `await` শুধু ওই ফাংশন থামায়, পুরো প্রোগ্রাম না। বাকি অংশ microtask এ চলে।
- Error ধরতে `try/catch`।

**একটা সাধারণ ভুল:** স্বাধীন কাজ একটার পর একটা `await` করলে ধীর হয়।

```js
// ❌ ধীর: ৪ সেকেন্ড
const users = await getUsers();   // ২ সেকেন্ড
const posts = await getPosts();   // ২ সেকেন্ড

// ✅ দ্রুত: ২ সেকেন্ড
const [users, posts] = await Promise.all([getUsers(), getPosts()]);
```

---

## 8. Debounce বনাম Throttle ⭐⭐⭐

- **Debounce:** ব্যবহারকারী **থামার পর** একবার চালাও।
- **Throttle:** নির্দিষ্ট সময়ে **সর্বোচ্চ একবার** চালাও।

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

function throttle(fn, limit) {
  let last = 0;
  return function (...args) {
    const now = Date.now();
    if (now - last >= limit) {
      last = now;
      fn.apply(this, args);
    }
  };
}

input.addEventListener("input", debounce(search, 500));
window.addEventListener("scroll", throttle(onScroll, 200));
```

| | Debounce | Throttle |
|---|---|---|
| কখন চলে | থামার পরে | নির্দিষ্ট বিরতিতে |
| উদাহরণ | search box, form validation | scroll, resize, mouse move |

---

## 9. Prototype chain ও property lookup ⭐⭐⭐

প্রতিটা object এর একটা লুকানো "বাবা" আছে, যাকে **prototype** বলে। Property নিজের কাছে না পেলে JS বাবার কাছে, তারপর দাদার কাছে খোঁজে। শেষে `null` এ পৌঁছালে `undefined`।

```js
const animal = { eat() { console.log("খাচ্ছে"); } };

const dog = Object.create(animal);
dog.bark = function () { console.log("ঘেউ"); };

dog.bark(); // নিজের কাছে আছে
dog.eat();  // prototype (animal) এ পেল
dog.fly();  // কোথাও নেই → TypeError
```

```
dog  →  animal  →  Object.prototype  →  null
```

**কেন কাজের:** `.map()`, `.toString()` প্রতিটা array এ আলাদা কপি হয় না, prototype এ একবারই থাকে। মেমোরি বাঁচে।

---

## 10. Shallow copy বনাম Deep copy ⭐⭐⭐

- **Shallow:** শুধু উপরের স্তর কপি, ভেতরের object শেয়ার হয়।
- **Deep:** সব স্তর নতুন কপি।

```js
const user = { name: "Alamin", address: { city: "Dhaka" } };

const copy = { ...user };
copy.name = "Rahim";             // ✅ মূলটা বদলায়নি
copy.address.city = "Khulna";    // ⚠️ ভেতরের object শেয়ার!
console.log(user.address.city);  // "Khulna" ← মূলটাও বদলাল

const deep = structuredClone(user);
deep.address.city = "Sylhet";
console.log(user.address.city);  // "Khulna" ← অপরিবর্তিত
```

| উপায় | ধরন | সীমাবদ্ধতা |
|---|---|---|
| `{ ...obj }`, `Object.assign` | Shallow | nested শেয়ার |
| `JSON.parse(JSON.stringify(obj))` | Deep | function, `undefined`, `Date`, `Map` হারায় |
| `structuredClone(obj)` | Deep | function কপি হয় না |

**React এ:** nested state বদলাতে নতুন reference বানাতে হয়।

---

## 11. call, apply, bind ⭐⭐⭐

তিনটাই `this` হাতে ঠিক করে দেয়।

```js
const user = { name: "Alamin" };

function greet(city, country) {
  console.log(`${this.name} from ${city}, ${country}`);
}

greet.call(user, "Dhaka", "BD");      // এখনই, argument আলাদা
greet.apply(user, ["Dhaka", "BD"]);   // এখনই, argument array তে
const fn = greet.bind(user, "Dhaka"); // চালায় না, নতুন ফাংশন
fn("BD");
```

| Method | এখনই চালায়? | Argument | ফেরত |
|---|---|---|---|
| `call` | হ্যাঁ | comma দিয়ে | ফাংশনের ফল |
| `apply` | হ্যাঁ | array তে | ফাংশনের ফল |
| `bind` | না | comma (আংশিকও চলে) | নতুন ফাংশন |

**ট্রিক:** `a` = array, `c` = comma, `b` = bind করে পরে চালাও।

```js
class Timer {
  constructor() { this.seconds = 0; }
  start() {
    setTimeout(this.tick.bind(this), 1000); // bind না দিলে this হারায়
  }
  tick() { this.seconds++; }
}
```

---

# Tier 2: প্রায়ই আসে

## 12. Promise state ও chaining ⭐⭐

| State | মানে |
|---|---|
| **pending** | কাজ চলছে |
| **fulfilled** | সফল |
| **rejected** | ব্যর্থ |

একবার fulfilled/rejected হলে আর বদলায় না। `.then()` সবসময় **নতুন Promise** ফেরত দেয়, তাই জোড়া যায়।

```js
fetch("/api/user")
  .then((res) => res.json())
  .then((user) => user.id)
  .then((id) => console.log(id))
  .catch((err) => console.log(err))     // যেকোনো ধাপে error হলে
  .finally(() => console.log("শেষ"));   // সবসময় চলে
```

---

## 13. `await` না দিলে কী হয় ⭐⭐

```js
async function getName() { return "Alamin"; }

async function main() {
  const a = getName();         // await নেই
  console.log(a);              // Promise { 'Alamin' } ← মান না

  const b = await getName();
  console.log(b);              // "Alamin"
}
```

- মান না, Promise object পাবে, কোড অপেক্ষা করবে না।
- Reject হলে কেউ না ধরলে **unhandled rejection** (Node এ crash হতে পারে)।
- ইচ্ছা করে না দিলে (analytics পাঠানো) `.catch()` লাগাও।

---

## 14. Promise.all নিজে বানানো ⭐⭐

```js
function myPromiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;

    if (promises.length === 0) {
      resolve(results);
      return;
    }

    promises.forEach((p, index) => {
      Promise.resolve(p).then((value) => {
        results[index] = value;   // নিজের index এ রাখো
        completed++;
        if (completed === promises.length) resolve(results);
      }, reject);                 // একটা fail করলেই reject
    });
  });
}

myPromiseAll([Promise.resolve(1), Promise.resolve(2)]).then(console.log); // [1, 2]
```

- `Promise.resolve(p)`: সাধারণ মান (`5`) থাকলেও কাজ করে।
- `results[index]`: `push` করলে ক্রম নষ্ট হয়।

---

## 15. Concurrency limit ⭐⭐

১০০টা request একসাথে না পাঠিয়ে একসাথে সর্বোচ্চ ৫টা।

```js
async function runWithLimit(tasks, limit) {
  const results = [];
  let index = 0;

  async function worker() {
    while (index < tasks.length) {
      const current = index++;
      results[current] = await tasks[current]();
    }
  }

  const workers = Array.from({ length: Math.min(limit, tasks.length) }, worker);
  await Promise.all(workers);
  return results;
}

const tasks = urls.map((url) => () => fetch(url).then((r) => r.json()));
const data = await runWithLimit(tasks, 5);
```

**খেয়াল:** `tasks` এ Promise না, **ফাংশন** রাখো। Promise তৈরি হওয়ামাত্র চলে, তখন limit কাজ করে না।

---

## 16. Memory leak এর কারণ ⭐⭐

| কারণ | উদাহরণ |
|---|---|
| ভুলে global ভ্যারিয়েবল | `x = 5` (`let` ছাড়া) |
| বন্ধ না করা timer | `setInterval` চলছেই |
| remove না করা listener | `addEventListener` আছে, `remove` নেই |
| Closure বড় ডেটা ধরে | ফাংশন বেঁচে আছে |
| মুছে ফেলা DOM element এর reference | JS এ ধরা আছে |

```js
// ✅ React এ cleanup
useEffect(() => {
  const id = setInterval(() => console.log("চলছি"), 1000);
  return () => clearInterval(id);
}, []);
```

---

## 17. Event delegation ⭐⭐

১০০টা বাটনে আলাদা listener না দিয়ে parent এ একটাই। Event bubbling এর কারণে কাজ করে।

```js
// ❌ প্রতিটায় আলাদা listener
document.querySelectorAll("li").forEach((li) =>
  li.addEventListener("click", () => console.log("click"))
);

// ✅ Delegation
document.querySelector("ul").addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    console.log("ক্লিক:", e.target.textContent);
  }
});
```

**সুবিধা:** কম মেমোরি, পরে যোগ করা element এও কাজ করে, কোড ছোট।

---

## 18. Lexical scoping ⭐⭐

ভ্যারিয়েবল কোথায় পাওয়া যাবে তা ঠিক হয় **কোড কোথায় লেখা** তা দিয়ে।

```js
const name = "বাইরের";

function outer() {
  const city = "Dhaka";
  function inner() {
    console.log(name); // "বাইরের"
    console.log(city); // "Dhaka"
  }
  inner();
}
outer();
console.log(city); // ❌ ReferenceError
```

JS ভেতর থেকে বাইরে খোঁজে, একে **scope chain** বলে। Closure এর ভিত্তি এটাই।

---

## 19. Undeclared বনাম undefined ⭐⭐

| | `undefined` | Undeclared |
|---|---|---|
| মানে | আছে, মান নেই | বানানোই হয়নি |
| ব্যবহার করলে | `undefined` | **ReferenceError** |

```js
let a;
console.log(a);          // undefined
console.log(b);          // ❌ ReferenceError
console.log(typeof b);   // "undefined" ← typeof error দেয় না
```

---

## 20. Execution context ⭐⭐

কোড চলার "পরিবেশ", যেখানে ভ্যারিয়েবল, `this`, বাইরের scope এর তথ্য থাকে।

**প্রকার:** Global context (একটা), Function context (প্রতিবার call এ নতুন)।

**২ ধাপ:**
1. **Creation phase:** মেমোরি বরাদ্দ। `var` কে `undefined`, function পুরো, `this` ঠিক (hoisting এর আসল কারণ)।
2. **Execution phase:** লাইন ধরে কোড চলে।

Context গুলো call stack এ জমে, শেষ হলে সরে যায়।

---

## 21. Function declaration বনাম expression ⭐⭐

```js
function add(a, b) { return a + b; }           // declaration

const add = function (a, b) { return a + b; }; // expression
```

| বিষয় | Declaration | Expression |
|---|---|---|
| Hoisting | পুরো hoist | ভ্যারিয়েবলের নিয়ম |
| নাম | সবসময় | anonymous হতে পারে |
| ব্যবহার | সাধারণ ফাংশন | callback, arrow function |

---

## 22. Currying ⭐⭐

```js
const add = (a, b, c) => a + b + c;
add(1, 2, 3);

const curriedAdd = (a) => (b) => (c) => a + b + c;
curriedAdd(1)(2)(3);

const multiply = (a) => (b) => a * b;
const double = multiply(2);
const triple = multiply(3);
double(5);  // 10
triple(5);  // 15
```

**বাস্তবে:** একই `baseURL` দিয়ে অনেক API call, একই prefix দিয়ে log।

---

## 23. Immutability ⭐⭐

ডেটা সরাসরি না বদলে নতুন কপি বানানো।

```js
const user = { name: "Alamin", age: 25 };

user.age = 26;                          // ❌ mutation
const updated = { ...user, age: 26 };   // ✅ নতুন object

const nums = [1, 2, 3];
const added = [...nums, 4];                  // push না করে
const removed = nums.filter((n) => n !== 2); // splice না করে
```

**কেন:** React নতুন reference না পেলে বদল ধরে না; বাগ কমে; ডিবাগ সহজ।

---

## 24. Garbage collection ⭐⭐

**Mark-and-sweep:**
1. Roots ঠিক করো (global object, call stack)
2. **Mark:** root থেকে যা পৌঁছানো যায় সব চিহ্নিত
3. **Sweep:** চিহ্নহীন সব মুছো

```js
let user = { name: "Alamin" }; // পৌঁছানো যায়
user = null;                    // কেউ ধরে নেই → মুছবে
```

"ব্যবহার হচ্ছে কিনা" না, "পৌঁছানো যায় কিনা" দেখা হয়।

---

## 25. Object.create() বনাম class ⭐⭐

`class` হলো prototype এরই সহজ লেখা।

```js
const animal = { speak() { console.log("শব্দ করছে"); } };
const dog = Object.create(animal);

class Animal { speak() { console.log("শব্দ করছে"); } }
class Dog extends Animal { bark() { console.log("ঘেউ"); } }
const d = new Dog();
d.speak();
```

| বিষয় | `Object.create()` | `class` |
|---|---|---|
| ধরন | object থেকে object | blueprint থেকে object |
| Constructor | নেই | আছে |
| কখন ভালো | সহজ inheritance | বড় প্রজেক্ট |

---

## 26. Object.freeze() বনাম seal() ⭐⭐

```js
const a = { name: "Alamin" };
Object.freeze(a);
a.name = "Rahim";  // ❌
a.age = 25;        // ❌
delete a.name;     // ❌

const b = { name: "Alamin" };
Object.seal(b);
b.name = "Rahim";  // ✅
b.age = 25;        // ❌
delete b.name;     // ❌
```

| কাজ | `freeze` | `seal` |
|---|---|---|
| নতুন property | না | না |
| মোছা | না | না |
| মান বদলানো | **না** | **হ্যাঁ** |

দুটোই **shallow**। Strict mode এ error, নইলে চুপচাপ ব্যর্থ।

---

# Tier 3: মাঝে মাঝে আসে

## 27. Function composition ⭐

একটার output পরেরটার input।

```js
const trim = (s) => s.trim();
const lower = (s) => s.toLowerCase();
const addDash = (s) => s.replace(/ /g, "-");

const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);

const slugify = compose(addDash, lower, trim);
slugify("  Hello World  "); // "hello-world"
```

---

## 28. Callback hell ⭐

```js
// ❌ Callback hell
getUser(1, (user) => {
  getOrders(user.id, (orders) => {
    getPayment(orders[0].id, (payment) => {
      console.log(payment);
    });
  });
});

// ✅ async/await
async function main() {
  try {
    const user = await getUser(1);
    const orders = await getOrders(user.id);
    const payment = await getPayment(orders[0].id);
    console.log(payment);
  } catch (err) {
    console.log(err);
  }
}
```

ক্রম: callback → Promise → async/await।

---

## 29. Pure functions ⭐

শর্ত: (১) একই input → একই output (২) কোনো side effect নেই।

```js
const add = (a, b) => a + b;                      // ✅ Pure

let total = 0;
const addToTotal = (n) => { total += n; };        // ❌ বাইরে বদলায়

const getTime = () => Date.now();                 // ❌ প্রতিবার আলাদা
```

React component ও Redux reducer pure হওয়া উচিত।

---

## 30. Monolithic বনাম Modular ⭐

```js
// math.js
export const add = (a, b) => a + b;

// app.js
import { add } from "./math.js";
```

| | Monolithic | Modular |
|---|---|---|
| গঠন | সব এক জায়গায় | ভাগ করা ফাইল |
| পুনর্ব্যবহার | কঠিন | সহজ |
| টিমে কাজ | conflict বেশি | সহজ |
| টেস্ট | কঠিন | সহজ |

---

## 31. Symbol ⭐

```js
const a = Symbol("id");
const b = Symbol("id");
console.log(a === b); // false

const ID = Symbol("id");
const user = { name: "Alamin", [ID]: 101 };
console.log(Object.keys(user)); // ["name"]
console.log(user[ID]);          // 101
```

**ব্যবহার:** নাম সংঘাত এড়ানো, আধা-লুকানো property, `Symbol.iterator` দিয়ে `for...of`।

---

## 32. Proxy ⭐

```js
const user = { name: "Alamin", age: 25 };

const validated = new Proxy(user, {
  get(target, prop) {
    console.log(`পড়া হচ্ছে: ${prop}`);
    return target[prop];
  },
  set(target, prop, value) {
    if (prop === "age" && typeof value !== "number") {
      throw new Error("age অবশ্যই number হতে হবে");
    }
    target[prop] = value;
    return true; // সফল বোঝাতে অবশ্যই true
  },
});

validated.age = 26;      // ✅
validated.age = "বিশ";   // ❌ Error
```

**ব্যবহার:** validation, logging, default মান, reactivity (Vue 3), read-only।

---

## 33. Browser optimization ⭐

- **JIT compilation:** কোড চলতে চলতে মেশিন কোডে বদলায়, hot code বিশেষভাবে অপটিমাইজ।
- **Hidden classes ও inline caching:** একই গঠনের object এর property খোঁজা দ্রুত।
- **Generational GC:** নতুন object ঘনঘন পরিষ্কার।

```js
// ✅ গঠন একই → দ্রুত
const a = { x: 1, y: 2 };
const b = { x: 3, y: 4 };

// ❌ গঠন আলাদা
const c = { x: 1 };
c.y = 2;
```

---

## 34. Web Workers ⭐

ভারী হিসাব main thread এ চললে পেজ আটকে যায়। Worker আলাদা thread এ চলে।

```js
// main.js
const worker = new Worker("worker.js");
worker.postMessage(1000000);
worker.onmessage = (e) => console.log(e.data);

// worker.js
self.onmessage = (e) => {
  let sum = 0;
  for (let i = 0; i < e.data; i++) sum += i;
  self.postMessage(sum);
};
```

**ব্যবহার:** বড় ডেটা, ছবি/ভিডিও প্রসেসিং, জটিল হিসাব। **সীমা:** DOM ধরতে পারে না।
