# Next.js Interview Answers

ইন্টারভিউতে বলার জন্য ছোট ও সহজ ইংরেজি উত্তর, সাথে বাংলা মানে। বিস্তারিত বোঝার জন্য `nextjs-details.md` পড়ো।

**কীভাবে পড়বে:** আগে Tier 1 (সবচেয়ে বেশি আসে), তারপর Tier 2, শেষে Tier 3।

| চিহ্ন | মানে |
|---|---|
| ⭐⭐⭐ | প্রায় প্রতিটা ইন্টারভিউতে আসে |
| ⭐⭐ | প্রায়ই আসে |
| ⭐ | মাঝে মাঝে আসে |

> **সংস্করণ সম্পর্কে:** এই উত্তরগুলো Next.js 16 পর্যন্ত আপডেট করা। Next.js 15 এ fetch এর caching ডিফল্টভাবে বন্ধ হয়েছে। Next.js 16 এ `middleware.ts` এর জায়গায় `proxy.ts` এসেছে এবং `"use cache"` দিয়ে caching এখন স্পষ্টভাবে চালু করতে হয়। ইন্টারভিউয়ে এগুলো জানা থাকলে ভালো প্রভাব পড়ে। পুরনো প্রজেক্টে `getServerSideProps` ও `middleware.ts` এখনো দেখা যায়, তাই দুটোই জানা দরকার।

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. What problem does Next.js solve compared to React? ⭐⭐⭐

> "React is only a UI library. It does not give routing, server rendering, or a build setup. Next.js is a full framework built on React. It adds file-based routing, server-side rendering, static generation, API endpoints, image and font optimization, and code splitting out of the box. So I get better performance and SEO, and I don't need to set up everything myself."

**বাংলা মানে:** React শুধু UI লাইব্রেরি। Next.js হলো React এর উপর তৈরি পূর্ণ framework, যাতে routing, server rendering, static generation, API, image/font optimization ও code splitting আগে থেকেই আছে। ফলে performance ও SEO ভালো হয়, সবকিছু নিজে সাজাতে হয় না।

---

## 2. Difference between CSR, SSR, SSG, and ISR ⭐⭐⭐

> "CSR, client-side rendering, builds the page in the browser with JavaScript. It is good for dashboards, but weak for SEO. SSR, server-side rendering, builds the HTML on the server for every request, so data is always fresh. SSG, static site generation, builds the HTML once at build time, so it is very fast but the data can get old. ISR, incremental static regeneration, is SSG that refreshes pages in the background after a set time, so I get speed and fresh data together."

**বাংলা মানে:** CSR এ browser এ JS দিয়ে পেজ বানানো হয় (dashboard এ ভালো, SEO দুর্বল)। SSR এ প্রতি request এ server এ HTML বানানো হয় (ডেটা সবসময় নতুন)। SSG তে build এর সময় একবার HTML বানানো হয় (খুব দ্রুত, ডেটা পুরনো হতে পারে)। ISR হলো SSG যা নির্দিষ্ট সময় পরপর পেছনে নিজে নিজে আপডেট হয়।

---

## 3. App Router vs Pages Router ⭐⭐⭐

> "The Pages Router is the older system, where each file in the `pages` folder is a route, and data is fetched with `getServerSideProps` and `getStaticProps`. The App Router is the newer system in the `app` folder. It uses React Server Components by default, supports nested layouts, streaming, and Server Actions, and fetches data directly with `async/await` in components. New projects should use the App Router, but the Pages Router is still supported, and many old projects use it."

**বাংলা মানে:** Pages Router পুরনো পদ্ধতি (`pages` ফোল্ডার, `getServerSideProps`)। App Router নতুন পদ্ধতি (`app` ফোল্ডার), যেখানে ডিফল্টভাবে Server Component, nested layout, streaming ও Server Actions আছে এবং component এর ভেতরেই `async/await` দিয়ে ডেটা আনা যায়। নতুন প্রজেক্টে App Router।

---

## 4. When should you use Server Components? ⭐⭐⭐

> "In the App Router, components are Server Components by default. I use them for fetching data, accessing the database or secrets, and rendering content that does not need interaction, because their code is not sent to the browser, so the JavaScript bundle is smaller. I add `'use client'` only when I need state, effects, event handlers, or browser APIs. A good pattern is to keep most of the tree on the server and push client components to the small leaves."

**বাংলা মানে:** App Router এ component ডিফল্টভাবে Server Component। ডেটা আনা, ডাটাবেস বা secret ব্যবহার, ইন্টারঅ্যাকশন ছাড়া কনটেন্টের জন্য এটা ব্যবহার করি, কারণ এর কোড browser এ যায় না। state, effect, event handler বা browser API লাগলেই কেবল `'use client'` দিই। বেশিরভাগ অংশ server এ রাখি, client component রাখি ছোট ছোট পাতায়।

---

## 5. How does caching work in the App Router? ⭐⭐⭐

> "Next.js has several caching layers. Request memoization removes duplicate fetch calls during one render. The data cache stores fetch results across requests. The full route cache stores the rendered HTML of static routes. The router cache stores visited pages in the browser. Since Next.js 15, `fetch` is not cached by default, and I opt in with options like `force-cache` or `next: { revalidate }`. In Next.js 16, Cache Components make caching explicit: I mark a component or function with the `'use cache'` directive, and everything else stays dynamic."

**বাংলা মানে:** কয়েকটা স্তর আছে: একই render এ duplicate fetch কমানো (request memoization), request এর মাঝে ডেটা রাখা (data cache), static route এর HTML রাখা (full route cache), browser এ দেখা পেজ রাখা (router cache)। Next.js 15 থেকে `fetch` ডিফল্টে cache হয় না, চাইলে `force-cache` বা `revalidate` দিতে হয়। Next.js 16 এ `'use cache'` দিয়ে স্পষ্টভাবে caching চালু করা হয়।

---

## 6. What is revalidation and how does ISR work internally? ⭐⭐⭐

> "Revalidation means refreshing cached data or pages. With ISR, a page is built once and served from cache. After the revalidate time passes, the next visitor still gets the old page immediately, and Next.js rebuilds the page in the background. After that, new visitors get the fresh page. This is called stale-while-revalidate. I can also refresh on demand with `revalidatePath` or `revalidateTag`, for example after an admin updates a product."

**বাংলা মানে:** Revalidation মানে cache করা ডেটা বা পেজ নতুন করা। ISR এ পেজ একবার বানিয়ে cache থেকে দেওয়া হয়। সময় পার হলে পরের ভিজিটর পুরনো পেজই পায়, আর পেছনে Next.js নতুন পেজ বানায়। এরপর সবাই নতুন পেজ পায়। চাইলে `revalidatePath` বা `revalidateTag` দিয়ে সাথে সাথে refresh করা যায়।

---

## 7. What is middleware (proxy) and how does it differ from API routes? ⭐⭐⭐

> "Middleware runs before a request reaches a page or an API route. I use it for redirects, rewrites, setting headers, and light checks like looking for a session cookie. It should be fast and small. In Next.js 16, `middleware.ts` is renamed to `proxy.ts`, and it runs on the Node.js runtime. API routes, now called Route Handlers, are the backend endpoints themselves: they receive a request, talk to the database, and return a response. So middleware is a gatekeeper in front, and an API route is the actual handler."

**বাংলা মানে:** Middleware request পেজ বা API তে পৌঁছানোর আগে চলে। redirect, rewrite, header বসানো ও হালকা যাচাইয়ে (যেমন session cookie আছে কিনা) ব্যবহার করি। এটা ছোট ও দ্রুত হওয়া উচিত। Next.js 16 এ এটার নাম `proxy.ts`। API route হলো আসল backend endpoint, যেখানে ডাটাবেসের কাজ হয়। Middleware সামনের দারোয়ান, API route আসল কাজের জায়গা।

---

## 8. What causes hydration mismatch errors? ⭐⭐⭐

> "Hydration is when React attaches events to the HTML that came from the server. A mismatch happens when the HTML from the server is different from what React renders on the client. Common causes are using `Date.now()`, `Math.random()`, or `window` and `localStorage` during render, invalid HTML nesting like a `<div>` inside a `<p>`, and browser extensions that change the HTML. To fix it, I move browser-only logic into `useEffect`, keep the first render the same on both sides, and fix the HTML structure."

**বাংলা মানে:** Hydration হলো server এর HTML এ React এর event জুড়ে দেওয়া। Server ও client এর render আলাদা হলে mismatch হয়। কারণ: render এ `Date.now()`, `Math.random()`, `window`, `localStorage` ব্যবহার, ভুল HTML গঠন (`<p>` এর ভেতরে `<div>`), browser extension। সমাধান: browser এর কাজ `useEffect` এ রাখা, দুই দিকে প্রথম render একই রাখা।

---

## 9. How would you protect admin routes / run auth before render? ⭐⭐⭐

> "I use layers of protection. First, in `proxy.ts`, I do a quick check for a session cookie and redirect to login if it is missing. This is only a fast, optimistic check. Second, in the server layout or page, I verify the real session and the admin role on the server, and redirect if it fails. Third, I check again inside every Server Action and Route Handler, because they can be called directly. I never trust middleware alone, and I never hide admin UI only on the client."

**বাংলা মানে:** কয়েক স্তরে সুরক্ষা দিই। প্রথমে `proxy.ts` এ দ্রুত session cookie আছে কিনা দেখি, না থাকলে login এ পাঠাই (এটা শুধু দ্রুত প্রাথমিক চেক)। দ্বিতীয়ত server layout/page এ আসল session ও admin role যাচাই করি। তৃতীয়ত প্রতিটা Server Action ও Route Handler এও আবার যাচাই করি, কারণ এগুলো সরাসরি call করা যায়। শুধু middleware বা শুধু client এ UI লুকানোর উপর ভরসা করি না।

---

## 10. Difference between `getServerSideProps` and `getStaticProps` ⭐⭐⭐

> "Both are data fetching functions in the Pages Router. `getStaticProps` runs at build time, so the page is static and very fast, and I can add `revalidate` for ISR. `getServerSideProps` runs on the server for every request, so the data is always fresh but the page is slower. I use `getStaticProps` for blogs and product pages, and `getServerSideProps` for pages that depend on the user or very fresh data. In the App Router, these are replaced by `async` Server Components and fetch options."

**বাংলা মানে:** দুটোই Pages Router এর ডেটা আনার ফাংশন। `getStaticProps` build এর সময় চলে, পেজ static ও দ্রুত (`revalidate` দিয়ে ISR)। `getServerSideProps` প্রতি request এ চলে, ডেটা সবসময় নতুন কিন্তু ধীর। App Router এ এগুলোর জায়গায় `async` Server Component ব্যবহার হয়।

---

## 11. Difference between fetch in server and client components ⭐⭐⭐

> "In a Server Component, I can use `async/await` with `fetch` directly, or call the database. It runs on the server, so there is no CORS problem, secrets stay safe, and I can control caching with fetch options. In a Client Component, I fetch in the browser with `useEffect`, or better with a library like TanStack Query or SWR. The request is visible to the user, secrets cannot be used, and CORS rules apply. I prefer fetching on the server when possible."

**বাংলা মানে:** Server Component এ সরাসরি `async/await` দিয়ে fetch বা ডাটাবেস call করা যায়। CORS সমস্যা নেই, secret নিরাপদ, caching নিয়ন্ত্রণ করা যায়। Client Component এ browser এ `useEffect` বা TanStack Query/SWR দিয়ে fetch হয়; request ব্যবহারকারী দেখতে পারে, secret ব্যবহার করা যায় না, CORS লাগে। সম্ভব হলে server এ fetch করি।

---

## 12. How does Next.js improve SEO by default? ⭐⭐⭐

> "Next.js sends ready HTML from the server, so search engines can read the content without running JavaScript. It also has the Metadata API for titles, descriptions, and Open Graph tags, and it can generate `sitemap.xml` and `robots.txt`. On top of that, image and font optimization and fast loading improve Core Web Vitals, which affect ranking."

**বাংলা মানে:** Server থেকেই তৈরি HTML যায়, তাই search engine JS ছাড়াই কনটেন্ট পড়তে পারে। Metadata API দিয়ে title, description, Open Graph দেওয়া যায়, `sitemap.xml` ও `robots.txt` বানানো যায়। image/font optimization ও দ্রুত লোডিং Core Web Vitals ভালো করে, যা ranking এ প্রভাব ফেলে।

---

## 13. What are streaming and Suspense? ⭐⭐⭐

> "Streaming means the server sends the page in parts, instead of waiting for all data. The user sees the fast parts first, like the layout, while slow parts show a loading state. I use React `Suspense` to wrap a slow component with a fallback, or a `loading.tsx` file for a whole route. This improves perceived speed and Time To First Byte, because one slow query does not block the whole page."

**বাংলা মানে:** Streaming মানে server পেজ টুকরো টুকরো করে পাঠায়, সব ডেটার জন্য অপেক্ষা করে না। ব্যবহারকারী আগে দ্রুত অংশ (layout) দেখে, ধীর অংশে loading দেখায়। ধীর component কে `Suspense` দিয়ে মুড়ি, বা পুরো route এর জন্য `loading.tsx` রাখি। এতে একটা ধীর query পুরো পেজ আটকায় না।

---

## 14. How do you protect server-only logic and secrets? ⭐⭐⭐

> "I keep secrets in environment variables without the `NEXT_PUBLIC_` prefix, because only variables with that prefix are sent to the browser. I put sensitive code, like database access, in server-only modules and import the `server-only` package, so the build fails if a Client Component tries to import it. I also keep data access in one server layer, return only the fields the UI needs, and never pass whole database objects to client components."

**বাংলা মানে:** Secret রাখি `NEXT_PUBLIC_` ছাড়া environment variable এ, কারণ শুধু ওই prefix ওয়ালা variable browser এ যায়। ডাটাবেসের মতো সংবেদনশীল কোড server-only module এ রাখি ও `server-only` প্যাকেজ import করি, যাতে Client Component import করলে build fail করে। ডেটা আনার একটা server layer রাখি, UI এর দরকারি ফিল্ডগুলোই শুধু পাঠাই।

---

## 15. SEO-friendly pages with frequently changing data: which rendering? ⭐⭐⭐

> "I would choose ISR, or Server Components with a short revalidate time. The page is served as fast static HTML, which is good for SEO, and it refreshes in the background every few seconds or minutes. If the data must be exactly fresh on every request, I would use SSR. If only one part changes quickly, like a price or stock count, I keep the page static and fetch that part on the client, or stream it with Suspense."

**বাংলা মানে:** আমি ISR বা ছোট `revalidate` সহ Server Component বেছে নেব। পেজ দ্রুত static HTML হিসেবে যায় (SEO ভালো), আর কিছুক্ষণ পরপর পেছনে নতুন হয়। প্রতি request এ ডেটা সম্পূর্ণ নতুন লাগলে SSR। শুধু একটা অংশ (দাম, stock) দ্রুত বদলালে পেজ static রেখে ওই অংশ client এ বা Suspense দিয়ে আনি।

---

# Tier 2: প্রায়ই আসে

## 16. Explain file-based routing ⭐⭐

> "In Next.js, the folder and file structure creates the routes, so I don't write a router config. In the App Router, a folder inside `app` is a route segment, and a `page.tsx` file makes it public. For example, `app/about/page.tsx` becomes `/about`. Special files like `layout.tsx`, `loading.tsx`, `error.tsx`, and `not-found.tsx` add shared layout, loading state, and error handling for that route."

**বাংলা মানে:** ফোল্ডার ও ফাইলের গঠন থেকেই route তৈরি হয়, আলাদা router config লিখতে হয় না। `app/about/page.tsx` হয় `/about`। `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx` এর মতো বিশেষ ফাইল shared layout, loading ও error সামলায়।

---

## 17. What are dynamic routes? ⭐⭐

> "Dynamic routes handle URLs with changing parts. I put the folder name in square brackets, like `app/blog/[slug]/page.tsx`. The value from the URL, like `slug`, is passed to the page in `params`. In Next.js 15 and later, `params` is a Promise, so I `await` it. For static generation, I use `generateStaticParams` to list the pages to build ahead of time. I can also use `[...slug]` for catch-all routes."

**বাংলা মানে:** URL এর পরিবর্তনশীল অংশের জন্য ফোল্ডারের নাম বর্গ বন্ধনীতে দিই, যেমন `[slug]`। URL এর মান `params` এ আসে। Next.js 15+ এ `params` একটা Promise, তাই `await` করতে হয়। আগে থেকে পেজ বানাতে `generateStaticParams`। সব অংশ ধরতে `[...slug]`।

---

## 18. Purpose of `next/link` and `next/image` ⭐⭐

> "`next/link` does client-side navigation without a full page reload, and it prefetches the linked page when the link is visible, so navigation feels instant. `next/image` optimizes images automatically: it serves modern formats like WebP or AVIF, resizes them for the device, lazy-loads them, and reserves space to avoid layout shift. This improves speed and Core Web Vitals."

**বাংলা মানে:** `next/link` পুরো পেজ reload ছাড়া নেভিগেট করে ও লিংক দেখা গেলে পেজ আগে থেকে লোড (prefetch) করে রাখে। `next/image` ছবি অটো অপটিমাইজ করে: WebP/AVIF দেয়, ডিভাইস অনুযায়ী ছোট করে, lazy load করে, জায়গা আগে থেকে রেখে layout shift ঠেকায়।

---

## 19. What are API routes? ⭐⭐

> "API routes let me build backend endpoints inside the Next.js project. In the Pages Router, they are files in `pages/api`. In the App Router, they are called Route Handlers: a `route.ts` file that exports functions named after HTTP methods, like `GET` and `POST`. I use them for webhooks, such as Stripe, for public APIs, and for integrations. For form submissions and mutations from my own UI, I often use Server Actions instead."

**বাংলা মানে:** Next.js প্রজেক্টের ভেতরেই backend endpoint বানানো যায়। Pages Router এ `pages/api`, App Router এ Route Handler (`route.ts`, যেখানে `GET`, `POST` এর মতো ফাংশন export হয়)। Webhook (Stripe), public API ও integration এ ব্যবহার করি। নিজের UI এর form ও mutation এর জন্য প্রায়ই Server Actions ব্যবহার করি।

---

## 20. How does Next.js handle environment variables? ⭐⭐

> "Next.js loads variables from `.env.local` and other `.env` files. By default, they are available only on the server. Only variables with the `NEXT_PUBLIC_` prefix are sent to the browser, and their values are inlined at build time. So a secret must never have that prefix. Also, if I change a `NEXT_PUBLIC_` variable, I need to rebuild. I set different values per environment in my hosting platform."

**বাংলা মানে:** `.env.local` ও অন্য `.env` ফাইল থেকে লোড হয়। ডিফল্টে শুধু server এ পাওয়া যায়। শুধু `NEXT_PUBLIC_` prefix ওয়ালা variable browser এ যায় এবং build এর সময় কোডে বসে যায়। তাই secret এ কখনো এই prefix নয়, আর এটা বদলালে আবার build করতে হয়।

---

## 21. How does code splitting work? ⭐⭐

> "Next.js splits the JavaScript automatically by route, so each page loads only the code it needs. Server Components send no component JavaScript to the browser at all. For heavy client components, like a chart or an editor, I use `next/dynamic` or `React.lazy` to load them only when needed. Next.js also prefetches the code of linked pages in the background."

**বাংলা মানে:** Route অনুযায়ী JS নিজে নিজে ভাগ হয়, প্রতিটা পেজ শুধু নিজের দরকারি কোড লোড করে। Server Component এর JS browser এ যায়ই না। ভারী client component (chart, editor) এর জন্য `next/dynamic` দিয়ে দরকারের সময় লোড করি।

---

## 22. Difference between `useRouter` and `useParams` ⭐⭐

> "`useRouter` is for navigation. It gives me methods like `push`, `replace`, `back`, and `refresh`. `useParams` is for reading. It gives me the dynamic route values from the URL, like the `id` in `/users/[id]`. Both come from `next/navigation` and work only in Client Components. In a Server Component, I get the params from the `params` prop instead."

**বাংলা মানে:** `useRouter` নেভিগেশনের জন্য (`push`, `replace`, `back`, `refresh`)। `useParams` URL থেকে dynamic মান পড়ার জন্য। দুটোই `next/navigation` থেকে আসে আর শুধু Client Component এ চলে। Server Component এ `params` prop ব্যবহার হয়।

---

## 23. How does nested routing work in the App Router? ⭐⭐

> "Nested folders create nested routes. Each folder can have its own `layout.tsx`, and layouts wrap their child routes. When I navigate between pages that share a layout, the layout stays mounted and does not re-render, so its state is preserved. This lets me build things like a dashboard with a shared sidebar and different content on the right."

**বাংলা মানে:** ফোল্ডারের ভেতর ফোল্ডার হলে nested route হয়। প্রতিটা ফোল্ডারে নিজস্ব `layout.tsx` থাকতে পারে, যা child route কে মুড়ে রাখে। একই layout ওয়ালা পেজে গেলে layout আবার render হয় না ও state ঠিক থাকে। যেমন dashboard এ sidebar একই থাকে।

---

## 24. How would you optimize images and fonts? ⭐⭐

> "For images, I use `next/image` with correct `width` and `height` or `fill`, set the `sizes` prop, add `priority` only for the main above-the-fold image, and use modern formats. For fonts, I use `next/font`, which downloads the font at build time and hosts it with my own site, so there is no extra network request and no layout shift. I also remove unused font weights."

**বাংলা মানে:** ছবির জন্য `next/image`, সঠিক `width/height`, `sizes`, শুধু প্রধান উপরের ছবিতে `priority`। ফন্টের জন্য `next/font`, যা build এর সময় ফন্ট নামিয়ে নিজের সাইটের সাথে দেয়, তাই বাড়তি network request ও layout shift নেই। অব্যবহৃত font weight বাদ দিই।

---

## 25. How do you reduce TTFB? ⭐⭐

> "I serve pages from the cache or a CDN with static generation or ISR, so the server does no work per request. For dynamic pages, I fetch independent data in parallel with `Promise.all`, avoid request waterfalls, and add indexes or caching to slow database queries. I use streaming with Suspense so a slow part does not block the first byte. I also deploy the server close to the database and the users, and keep middleware light."

**বাংলা মানে:** Static/ISR দিয়ে CDN বা cache থেকে পেজ দিই। Dynamic পেজে স্বাধীন ডেটা `Promise.all` দিয়ে একসাথে আনি, waterfall এড়াই, ধীর query তে index/cache দিই। Suspense দিয়ে streaming করি। Server ডাটাবেস ও ব্যবহারকারীর কাছাকাছি রাখি, middleware হালকা রাখি।

---

## 26. Where should JWT verification happen? ⭐⭐

> "JWT must be verified on the server, never trusted from the client. I do the real verification where the data is accessed: in Server Components, Server Actions, and Route Handlers, using a shared helper function. In `proxy.ts`, I can do a quick check for fast redirects, but it should not be the only protection. I store the token in an `HttpOnly` secure cookie so JavaScript cannot read it, and I check the signature, expiry, and role."

**বাংলা মানে:** JWT সবসময় server এ যাচাই করতে হয়, client কে বিশ্বাস করা যায় না। আসল যাচাই ডেটা ধরার জায়গায়: Server Component, Server Action, Route Handler এ (একটা shared helper দিয়ে)। `proxy.ts` এ দ্রুত চেক চলে, কিন্তু একমাত্র সুরক্ষা হিসেবে না। Token রাখি `HttpOnly` secure cookie তে। signature, মেয়াদ ও role যাচাই করি।

---

## 27. Page loads slowly even with SSG. How do you optimize? ⭐⭐

> "SSG only means the HTML is ready, so the slowness is probably in the browser. I would measure first with Lighthouse and the bundle analyzer. Then I check: large JavaScript bundles, too many Client Components, heavy third-party scripts, unoptimized images, and slow fonts. I fix them by using dynamic imports, moving logic to Server Components, loading scripts with `next/script` and a lazy strategy, using `next/image` and `next/font`, and checking that the CDN is caching the page."

**বাংলা মানে:** SSG তে HTML তৈরি, তাই ধীরতা সম্ভবত browser এ। আগে Lighthouse ও bundle analyzer দিয়ে মাপি। দেখি: বড় JS bundle, অনেক Client Component, ভারী third-party script, অপটিমাইজ না করা ছবি, ধীর ফন্ট। সমাধান: dynamic import, Server Component, `next/script` এ lazy strategy, `next/image`/`next/font`, আর CDN ঠিকমতো cache করছে কিনা।

---

## 28. A page works locally but fails after deployment. How do you debug? ⭐⭐

> "First, I read the build and runtime logs on the hosting platform. Then I run `next build` and `next start` locally, because development mode behaves differently, especially for caching. Common causes are missing environment variables, `NEXT_PUBLIC_` variables not set at build time, file name case differences on Linux, the database not allowing the server's IP, different Node versions, hardcoded `localhost` URLs, and missing image domains. I fix the cause and test again."

**বাংলা মানে:** আগে hosting এর build ও runtime log পড়ি। তারপর লোকালে `next build` ও `next start` চালাই, কারণ dev mode আলাদা আচরণ করে (বিশেষত caching)। সাধারণ কারণ: environment variable না থাকা, `NEXT_PUBLIC_` variable build এর সময় সেট না থাকা, Linux এ ফাইলের নামের বড়/ছোট হাতের অক্ষর, ডাটাবেস server এর IP অনুমতি না দেওয়া, Node সংস্করণ আলাদা, `localhost` URL, image domain অনুমতি না থাকা।

---

# Tier 3: মাঝে মাঝে আসে

## 29. What are route groups and why use them? ⭐

> "A route group is a folder name in parentheses, like `(marketing)`. It organizes routes without changing the URL, because the name is not part of the path. I use route groups to give different layouts to different sections, for example one layout for public pages and another for the dashboard, or just to keep the project organized."

**বাংলা মানে:** ফোল্ডারের নাম ব্র্যাকেটে দিলে (`(marketing)`) সেটা URL এ আসে না। এটা দিয়ে আলাদা আলাদা অংশে আলাদা layout দিই (public পেজের layout এক, dashboard এর আরেক), বা প্রজেক্ট গোছানো রাখি।

---

## 30. How do parallel routes work? ⭐

> "Parallel routes let me render more than one page in the same layout at the same time. I create folders that start with `@`, called slots, like `@analytics` and `@team`. The layout receives each slot as a prop and places it where it wants. Each slot can load, fail, and stream on its own. I use them for dashboards and modals. In Next.js 16, each slot needs a `default.tsx` file for unmatched routes."

**বাংলা মানে:** একই layout এ একসাথে একাধিক পেজ দেখানো যায়। `@` দিয়ে শুরু হওয়া ফোল্ডার (slot) বানাই, layout সেগুলোকে prop হিসেবে পেয়ে যেখানে চায় বসায়। প্রতিটা slot আলাদাভাবে load, error ও stream হতে পারে। Dashboard ও modal এ কাজে লাগে। Next.js 16 এ প্রতিটা slot এ `default.tsx` লাগে।

---

## 31. How would you implement SSO in Next.js? ⭐

> "I would not build SSO from scratch. I would use an auth library or service, like Auth.js, Better Auth, Auth0, or WorkOS, that supports OIDC or SAML. The flow is: the user clicks login, is redirected to the identity provider, signs in there, and comes back to my callback route with a code. The server exchanges the code for tokens, verifies them, and creates a session in an `HttpOnly` cookie. Then I protect routes by checking the session on the server."

**বাংলা মানে:** SSO নিজে বানাই না। OIDC বা SAML সমর্থন করা লাইব্রেরি/সেবা (Auth.js, Better Auth, Auth0, WorkOS) ব্যবহার করি। ব্যবহারকারী login চাপে → identity provider এ যায় → সেখানে sign in করে → code সহ আমার callback route এ ফেরে। Server code দিয়ে token নেয়, যাচাই করে, `HttpOnly` cookie তে session বানায়। তারপর server এ session দেখে route সুরক্ষিত করি।

---

## 32. How do you handle large traffic spikes? ⭐

> "I make as much as possible static or cached with SSG or ISR and a CDN, so most requests never reach the server. For dynamic parts, I cache data, use database connection pooling, and add rate limiting. I deploy on a platform that scales automatically, and I move slow work, like emails, to background queues. Finally, I do load testing before the event and set up monitoring and alerts."

**বাংলা মানে:** যতটা সম্ভব SSG/ISR ও CDN দিয়ে static/cache করি, যাতে বেশিরভাগ request server এ না পৌঁছায়। Dynamic অংশে data cache, database connection pooling, rate limiting রাখি। স্বয়ংক্রিয় scale হয় এমন platform এ deploy করি, ধীর কাজ (email) queue তে পাঠাই। ইভেন্টের আগে load test ও monitoring রাখি।

---

## শেষ মুহূর্তের রিভিশন (এক লাইনে)

- **CSR / SSR / SSG / ISR:** browser এ / প্রতি request এ / build এ / build + পেছনে আপডেট
- **App Router:** ডিফল্ট Server Component, `'use client'` শুধু দরকারে
- **Caching:** Next 15 এ `fetch` ডিফল্টে cache হয় না। Next 16 এ `'use cache'`
- **ISR:** stale পেজ দেয়, পেছনে নতুন বানায় (stale-while-revalidate)
- **Middleware:** Next 16 এ `proxy.ts`। দ্রুত চেক, একমাত্র সুরক্ষা না
- **Auth:** proxy এ দ্রুত চেক + server এ আসল চেক + প্রতিটা Action/Handler এ চেক
- **Secret:** `NEXT_PUBLIC_` ছাড়া, `server-only`
- **Hydration mismatch:** server ও client এর প্রথম render আলাদা। browser কাজ `useEffect` এ
- **Streaming:** `Suspense` / `loading.tsx`
- **Deploy সমস্যা:** `next build && next start` লোকালে চালাও, env variable দেখো
