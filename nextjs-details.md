# Next.js Details (বিস্তারিত বোঝার জন্য)

প্রতিটা টপিক সহজ বাংলায়, কোড উদাহরণসহ। ইন্টারভিউয়ের ছোট ইংরেজি উত্তরের জন্য `nextjs-interview.md` দেখো।

**ক্রম:** Tier 1 (সবচেয়ে বেশি আসে) → Tier 2 → Tier 3। নম্বর `nextjs-interview.md` এর সাথে মেলানো।

> **সংস্করণ সম্পর্কে:** Next.js 16 পর্যন্ত আপডেট করা। কোডে পুরনো ও নতুন দুই পদ্ধতিই দেখানো হয়েছে, কারণ পুরনো প্রজেক্টে `getServerSideProps` ও `middleware.ts` এখনো আছে।
>
> | বদল | কোন সংস্করণে |
> |---|---|
> | `fetch`, GET Route Handler ডিফল্টে cache হয় না | 15 |
> | `params`, `searchParams`, `cookies()`, `headers()` এখন Promise (`await` লাগে) | 15 এ শুরু, 16 এ আবশ্যক |
> | `middleware.ts` → `proxy.ts` (Node.js runtime) | 16 |
> | Cache Components ও `"use cache"` | 16 |
> | Turbopack ডিফল্ট bundler | 16 |
> | Parallel route slot এ `default.tsx` আবশ্যক | 16 |

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. Next.js বনাম React ⭐⭐⭐

React শুধু **UI বানানোর লাইব্রেরি**। এ দিয়ে একটা পূর্ণ প্রজেক্ট বানাতে হলে routing, bundler, server rendering, SEO, optimization সব নিজে জোড়া দিতে হয়।

Next.js হলো React এর উপর তৈরি **পূর্ণ framework**।

| বিষয় | শুধু React (যেমন Vite) | Next.js |
|---|---|---|
| Routing | নিজে সেটআপ (React Router) | ফোল্ডার থেকে নিজে নিজে |
| Rendering | সাধারণত শুধু CSR | CSR, SSR, SSG, ISR সব |
| SEO | দুর্বল (খালি HTML যায়) | ভালো (তৈরি HTML যায়) |
| Backend/API | আলাদা প্রজেক্ট | Route Handler, Server Actions |
| Image/Font | নিজে | অটো optimization |
| Code splitting | নিজে | route অনুযায়ী অটো |

**মনে রাখার উপমা:** React হলো ইঞ্জিন, Next.js হলো ইঞ্জিনসহ পুরো গাড়ি।

**কখন শুধু React:** ভেতরের টুল বা dashboard যেখানে SEO লাগে না।

---

## 2. CSR, SSR, SSG, ISR ⭐⭐⭐

পেজের HTML **কখন ও কোথায়** বানানো হয়, এটাই পার্থক্য।

| ধরন | HTML কখন বানায় | গতি | ডেটা কতটা নতুন | কখন ব্যবহার |
|---|---|---|---|---|
| **CSR** | Browser এ, JS চলার পর | প্রথমে ধীর | সবসময় নতুন | dashboard, লগইনের পরের অংশ |
| **SSR** | প্রতি request এ server এ | মাঝারি | সবসময় নতুন | ব্যবহারকারী-নির্ভর পেজ |
| **SSG** | Build এর সময় একবার | সবচেয়ে দ্রুত | build এর সময়ের | blog, docs, marketing |
| **ISR** | SSG + নির্দিষ্ট সময় পরপর পেছনে আপডেট | দ্রুত | কিছুক্ষণ আগের | product, news |

**App Router এ কীভাবে হয়:**
```tsx
// SSG (static): dynamic কিছু ব্যবহার না করলে নিজে নিজে static
export default async function Page() {
  const res = await fetch("https://api.example.com/posts", {
    cache: "force-cache",              // cache করো
  });
  const posts = await res.json();
  return <PostList posts={posts} />;
}

// ISR: ৬০ সেকেন্ড পর পেছনে নতুন হবে
await fetch(url, { next: { revalidate: 60 } });

// SSR: প্রতি request এ নতুন
await fetch(url, { cache: "no-store" });
// বা cookies()/headers() ব্যবহার করলে পেজ নিজেই dynamic হয়
```

**CSR (Client Component এ):**
```tsx
"use client";
import { useEffect, useState } from "react";

export default function Dashboard() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetch("/api/stats").then((r) => r.json()).then(setData);
  }, []);
  return <div>{data ? data.total : "লোড হচ্ছে..."}</div>;
}
```

---

## 3. App Router বনাম Pages Router ⭐⭐⭐

| বিষয় | Pages Router (পুরনো) | App Router (নতুন) |
|---|---|---|
| ফোল্ডার | `pages/` | `app/` |
| Route ফাইল | `pages/about.tsx` | `app/about/page.tsx` |
| ডেটা আনা | `getServerSideProps`, `getStaticProps` | `async` component, `fetch` |
| Component | ডিফল্ট Client (hydrate হয়) | ডিফল্ট **Server Component** |
| Layout | `_app.tsx`, `_document.tsx` | নেস্টেড `layout.tsx` |
| API | `pages/api/*.ts` | `route.ts` (Route Handler) |
| Loading/Error | নিজে বানাও | `loading.tsx`, `error.tsx` |
| Streaming | সীমিত | পূর্ণ সমর্থন |
| Mutation | API route + fetch | **Server Actions** |

**ফাইলের বিশেষ নাম (App Router):**

| ফাইল | কাজ |
|---|---|
| `page.tsx` | route কে public করে |
| `layout.tsx` | shared layout |
| `loading.tsx` | loading UI (Suspense এর মতো) |
| `error.tsx` | error UI (Client Component হতে হয়) |
| `not-found.tsx` | 404 |
| `route.ts` | API endpoint |

**কোনটা ব্যবহার:** নতুন প্রজেক্টে App Router। পুরনো প্রজেক্ট চালু থাকলে Pages Router ঠিকই চলে, দুটো একসাথেও থাকতে পারে।

---

## 4. Server Components কখন ব্যবহার ⭐⭐⭐

App Router এ **সব component ডিফল্টভাবে Server Component**। এর কোড browser এ যায় না।

| | Server Component | Client Component (`'use client'`) |
|---|---|---|
| কোথায় চলে | Server এ | Server এ (প্রথম HTML) + Browser এ |
| DB/secret ব্যবহার | ✅ | ❌ |
| `useState`, `useEffect` | ❌ | ✅ |
| `onClick` ইত্যাদি event | ❌ | ✅ |
| `window`, `localStorage` | ❌ | ✅ |
| Bundle এ কোড যায়? | না | হ্যাঁ |

```tsx
// Server Component (ডিফল্ট): সরাসরি ডাটাবেস
import { prisma } from "@/lib/prisma";

export default async function UsersPage() {
  const users = await prisma.user.findMany();   // সরাসরি DB, API লাগেনি
  return (
    <ul>
      {users.map((u) => <li key={u.id}>{u.name}</li>)}
    </ul>
  );
}
```

```tsx
// Client Component: ইন্টারঅ্যাকশন লাগলে
"use client";
import { useState } from "react";

export default function LikeButton() {
  const [likes, setLikes] = useState(0);
  return <button onClick={() => setLikes(likes + 1)}>👍 {likes}</button>;
}
```

**সেরা অভ্যাস:** `'use client'` গাছের উপরে না দিয়ে একদম ছোট পাতার component এ দাও।

```tsx
// ✅ ভালো: শুধু বাটনটা client, বাকি পেজ server
export default async function ProductPage() {
  const product = await getProduct();
  return (
    <div>
      <h1>{product.name}</h1>          {/* server */}
      <p>{product.description}</p>     {/* server */}
      <AddToCartButton id={product.id} />  {/* ছোট client component */}
    </div>
  );
}
```

**Server Component কে Client Component এর ভেতরে `children` হিসেবে পাঠানো যায়**, কিন্তু client ফাইলে server component `import` করা যায় না।

---

## 5. App Router এ Caching ⭐⭐⭐

### পুরনো মডেল: ৪টা স্তর

| স্তর | কোথায় | কী রাখে | কতক্ষণ |
|---|---|---|---|
| **Request Memoization** | Server | একই render এ একই `fetch` | একটা request |
| **Data Cache** | Server | `fetch` এর ফল | revalidate না হওয়া পর্যন্ত |
| **Full Route Cache** | Server | static route এর HTML | revalidate/redeploy পর্যন্ত |
| **Router Cache** | Browser | দেখা পেজের অংশ | সেশন, অল্প সময় |

### Next.js 15: ডিফল্ট বদলেছে

Next.js 14 এ `fetch` ডিফল্টে cache হতো, যা বিভ্রান্তি তৈরি করত। **Next.js 15 এ উল্টো: ডিফল্টে cache হয় না।** GET Route Handler ও একই।

```tsx
await fetch(url);                              // ❌ cache হয় না (15+)
await fetch(url, { cache: "force-cache" });    // ✅ cache হয়
await fetch(url, { next: { revalidate: 60 } }); // ✅ ৬০ সেকেন্ড cache
await fetch(url, { next: { tags: ["posts"] } }); // ✅ tag দিয়ে পরে মুছতে পারবে
```

### Next.js 16: Cache Components (`"use cache"`)

Caching এখন **স্পষ্ট ও opt-in**। কনফিগে চালু করো:

```ts
// next.config.ts
const nextConfig = {
  cacheComponents: true,
};
export default nextConfig;
```

```tsx
import { cacheLife, cacheTag } from "next/cache";

// এই ফাংশনের ফল cache হবে
async function getReport() {
  "use cache";
  cacheLife("hours");        // কতক্ষণ
  cacheTag("report");        // পরে মুছতে
  return await generateReport();
}
```

**নিয়ম:** `"use cache"` ছাড়া যা আছে তা dynamic (প্রতি request এ চলে)। ব্যবহারকারী-নির্ভর রিয়েল-টাইম ডেটা cache করো না।

### Cache মোছার API

```ts
"use server";
import { revalidatePath, revalidateTag, updateTag } from "next/cache";

export async function updatePost() {
  // ... DB আপডেট
  revalidatePath("/blog");        // একটা path এর cache মোছো
  revalidateTag("posts", "max");  // tag ওয়ালা সব cache মোছো (16 এ দ্বিতীয় argument দিতে হয়)
  updateTag("posts");             // Server Action এ: সাথে সাথে নতুন ডেটা পাবে (read-your-writes)
}
```

**ইন্টারভিউয়ের উপযোগী লাইন:** "Next.js 15 এ caching ডিফল্টে বন্ধ হয়েছে, আর 16 এ `use cache` দিয়ে সম্পূর্ণ স্পষ্ট করা হয়েছে।"

---

## 6. Revalidation ও ISR ভেতরে কীভাবে কাজ করে ⭐⭐⭐

**Stale-while-revalidate** কৌশল:

```
সময় 0:      পেজ বানানো ও cache এ রাখা        (revalidate = 60)
সময় 30s:    ভিজিটর আসল → cache থেকে সাথে সাথে পেজ (ফ্রেশ)
সময় 90s:    ভিজিটর আসল → পুরনো পেজই সাথে সাথে পায়
             + পেছনে Next.js নতুন করে পেজ বানাতে শুরু করে
সময় 91s:    বানানো শেষ, cache আপডেট
সময় 100s:   নতুন ভিজিটর → নতুন পেজ পায়
```

**ভিজিটরকে কখনো অপেক্ষা করতে হয় না**, শুধু একজন পুরনো পেজ দেখে।

**কোড (App Router):**
```tsx
// সময় ভিত্তিক
await fetch(url, { next: { revalidate: 60 } });

// পুরো পেজের জন্য
export const revalidate = 60;
```

**কোড (Pages Router):**
```tsx
export async function getStaticProps() {
  const posts = await getPosts();
  return {
    props: { posts },
    revalidate: 60,     // ISR: ৬০ সেকেন্ড
  };
}
```

**On-demand (দরকার হলেই সাথে সাথে):**
```ts
// Server Action বা Route Handler এ
revalidatePath("/products/shoes");
revalidateTag("products", "max");
```
যেমন admin পণ্যের দাম বদলালে সাথে সাথে পেজ নতুন করা।

**সতর্কতা:** ISR তে কনটেন্ট কখনো ব্যবহারকারী-নির্ভর (লগইন করা ব্যবহারকারীর নাম ইত্যাদি) রেখো না, কারণ cache সবার জন্য একই।

---

## 7. Middleware (proxy) এবং API route এর পার্থক্য ⭐⭐⭐

### Middleware / Proxy

Request পেজ বা API তে পৌঁছানোর **আগে** চলে। হালকা কাজের জায়গা।

**Next.js 16 (নতুন): `proxy.ts`** (Node.js runtime)
```ts
// proxy.ts (প্রজেক্টের root এ, বা src/ এর ভেতরে)
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function proxy(request: NextRequest) {
  const token = request.cookies.get("session")?.value;

  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*", "/admin/:path*"],   // কোন path এ চলবে
};
```

**Next.js 15 ও আগে: `middleware.ts`** (একই কোড, শুধু নাম ভিন্ন)
```ts
// middleware.ts
export function middleware(request: NextRequest) { /* ... */ }
export const config = { matcher: ["/dashboard/:path*"] };
```

**16 এ migration:** ফাইল `middleware.ts` → `proxy.ts`, ফাংশন `middleware` → `proxy`। যুক্তি একই থাকে। পুরনো `middleware.ts` এখনো চলে কিন্তু deprecated।

**কখন ব্যবহার:** redirect, rewrite, header বসানো, A/B test, locale ঠিক করা, session cookie আছে কিনা দ্রুত দেখা।

**কখন ব্যবহার নয়:** ভারী কাজ, ডাটাবেস query, বড় লজিক। এটা প্রতিটা matching request এ চলে, তাই হালকা রাখো।

### API Route (Route Handler)

আসল backend endpoint।

```ts
// app/api/users/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  const users = await prisma.user.findMany();
  return NextResponse.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();
  const user = await prisma.user.create({ data: body });
  return NextResponse.json(user, { status: 201 });
}
```

| | Middleware / Proxy | API Route |
|---|---|---|
| কখন চলে | request এর **আগে** | request এর **হ্যান্ডলার** হিসেবে |
| কাজ | redirect, header, দ্রুত চেক | ডেটা প্রসেস, DB, response |
| ভারী কাজ | ❌ | ✅ |
| উপমা | গেটের দারোয়ান | ভেতরের অফিস |

---

## 8. Hydration mismatch ⭐⭐⭐

**Hydration:** Server থেকে আসা HTML এ React event ও interactivity জুড়ে দেওয়া। React চায় server ও client এর প্রথম render **হুবহু এক** হবে। না হলে "Hydration failed" error।

### প্রধান কারণ ও সমাধান

**১. প্রতিবার আলাদা মান (`Date`, `Math.random`)**
```tsx
// ❌ server এ এক সময়, client এ আরেক সময়
export default function Clock() {
  return <p>{new Date().toLocaleTimeString()}</p>;
}

// ✅ client এ mount হওয়ার পর বসাও
"use client";
import { useEffect, useState } from "react";

export default function Clock() {
  const [time, setTime] = useState("");
  useEffect(() => setTime(new Date().toLocaleTimeString()), []);
  return <p>{time}</p>;
}
```

**২. Browser-only API render এ (`window`, `localStorage`)**
```tsx
// ❌ server এ window নেই
const theme = localStorage.getItem("theme");

// ✅ useEffect এ
const [theme, setTheme] = useState("light");
useEffect(() => {
  setTheme(localStorage.getItem("theme") ?? "light");
}, []);
```

**৩. ভুল HTML গঠন**
```tsx
// ❌ <p> এর ভেতরে <div> বৈধ নয়
<p><div>text</div></p>

// ✅
<div><p>text</p></div>
```
অন্য উদাহরণ: `<a>` এর ভেতরে `<a>`, `<table>` এ সরাসরি `<div>`।

**৪. Browser extension** (যা HTML বদলে দেয়) বা ভুল ভাবে `<html>`/`<body>` বদলানো।

**৫. যদি সত্যিই আলাদা হওয়া ঠিক থাকে** (যেমন সময় দেখানো):
```tsx
<time suppressHydrationWarning>{new Date().toISOString()}</time>
```
এটা শুধু এক স্তর গভীর কাজ করে, সব সমস্যার সমাধান না।

**৬. যে component শুধু client এ দরকার:**
```tsx
"use client";
import dynamic from "next/dynamic";

const Chart = dynamic(() => import("./Chart"), { ssr: false });
```
`ssr: false` শুধু Client Component এর ভেতরে ব্যবহার করা যায়।

---

## 9. Admin route সুরক্ষা / render এর আগে auth ⭐⭐⭐

**মূলনীতি: Defense in depth, একাধিক স্তর।** শুধু middleware বা শুধু UI লুকানোর উপর ভরসা করা বিপজ্জনক।

> 2025 সালে Next.js middleware এর একটা গুরুতর bypass দুর্বলতা (CVE-2025-29927) ধরা পড়েছিল, যাতে বিশেষ header দিয়ে middleware এড়ানো যেত। শিক্ষা: আসল যাচাই ডেটার কাছেই করো।

### স্তর ১: Proxy তে দ্রুত চেক (প্রাথমিক)
```ts
// proxy.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";
import { getSessionCookie } from "better-auth/cookies";

export function proxy(request: NextRequest) {
  const sessionCookie = getSessionCookie(request);   // শুধু cookie আছে কিনা
  if (!sessionCookie) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
  return NextResponse.next();
}

export const config = { matcher: ["/admin/:path*", "/dashboard/:path*"] };
```
এটা **শুধু cookie আছে কিনা** দেখে, আসল session ঠিক কিনা যাচাই করে না। তাই এটা একমাত্র সুরক্ষা না।

### স্তর ২: Server layout/page এ আসল যাচাই
```tsx
// app/admin/layout.tsx
import { auth } from "@/lib/auth";
import { headers } from "next/headers";
import { redirect } from "next/navigation";

export default async function AdminLayout({ children }: { children: React.ReactNode }) {
  const session = await auth.api.getSession({ headers: await headers() });

  if (!session) redirect("/login");
  if (session.user.role !== "admin") redirect("/");   // role ফিল্ড তোমার schema অনুযায়ী

  return <div>{children}</div>;
}
```
এটা render হওয়ার **আগেই** চলে, তাই অননুমোদিত ব্যবহারকারী কনটেন্ট দেখতেই পায় না।

### স্তর ৩: প্রতিটা Server Action ও Route Handler এ আবার যাচাই
```ts
"use server";

export async function deleteUser(id: string) {
  const session = await auth.api.getSession({ headers: await headers() });
  if (session?.user.role !== "admin") {
    throw new Error("Unauthorized");
  }
  await prisma.user.delete({ where: { id } });
}
```
**কেন:** Server Action ও API সরাসরি call করা যায়। পেজ সুরক্ষিত থাকলেও এগুলো খোলা থাকলে লাভ নেই।

### একটা কেন্দ্রীয় helper (Data Access Layer)
```ts
// lib/dal.ts
import "server-only";
import { cache } from "react";

export const requireAdmin = cache(async () => {
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session) redirect("/login");
  if (session.user.role !== "admin") redirect("/");
  return session;
});
```
এরপর সব জায়গায় `await requireAdmin()` ডাকো।

### সংক্ষেপে

| স্তর | কোথায় | কাজ |
|---|---|---|
| ১ | `proxy.ts` | দ্রুত cookie চেক, redirect |
| ২ | layout/page | আসল session ও role |
| ৩ | Server Action / Route Handler | প্রতিটায় আবার যাচাই |
| ৪ | ডাটাবেস | সম্ভব হলে row-level নিয়ম |

---

## 10. `getServerSideProps` বনাম `getStaticProps` (Pages Router) ⭐⭐⭐

```tsx
// pages/blog/[slug].tsx

// getStaticProps: build এর সময় চলে
export async function getStaticProps({ params }) {
  const post = await getPost(params.slug);
  return {
    props: { post },
    revalidate: 60,        // ঐচ্ছিক: ISR
  };
}

// getStaticPaths: কোন কোন path আগে থেকে বানাবে
export async function getStaticPaths() {
  const posts = await getAllPosts();
  return {
    paths: posts.map((p) => ({ params: { slug: p.slug } })),
    fallback: "blocking",  // নতুন path এ প্রথমবার server এ বানাবে
  };
}
```

```tsx
// getServerSideProps: প্রতি request এ চলে
export async function getServerSideProps({ req, res }) {
  const user = await getUserFromCookie(req);
  return { props: { user } };
}
```

| | `getStaticProps` | `getServerSideProps` |
|---|---|---|
| কখন চলে | Build এর সময় (+ ISR) | প্রতি request এ |
| গতি | খুব দ্রুত (CDN) | ধীর |
| ডেটা | build/revalidate এর সময়ের | সবসময় নতুন |
| request তথ্য (cookie) | ❌ | ✅ |
| কখন | blog, product, docs | user-নির্ভর, রিয়েল-টাইম |

**App Router এ সমতুল্য:**

| Pages Router | App Router |
|---|---|
| `getStaticProps` | `async` component + `fetch` (cache সহ) |
| `getServerSideProps` | `async` component + `cache: "no-store"` বা `cookies()` |
| `getStaticPaths` | `generateStaticParams` |

---

## 11. Server বনাম Client component এ fetch ⭐⭐⭐

```tsx
// Server Component: সরাসরি async/await
export default async function Posts() {
  const res = await fetch("https://api.example.com/posts", {
    next: { revalidate: 60 },
  });
  const posts = await res.json();
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

```tsx
// Client Component: TanStack Query দিয়ে
"use client";
import { useQuery } from "@tanstack/react-query";

export default function Posts() {
  const { data, isLoading } = useQuery({
    queryKey: ["posts"],
    queryFn: () => fetch("/api/posts").then((r) => r.json()),
  });
  if (isLoading) return <p>লোড হচ্ছে...</p>;
  return <ul>{data.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

| | Server | Client |
|---|---|---|
| কোথায় চলে | Server | Browser |
| Secret/DB | ✅ নিরাপদ | ❌ |
| CORS | লাগে না | লাগে |
| Network দেখা যায়? | না | হ্যাঁ |
| Caching নিয়ন্ত্রণ | `fetch` option, `use cache` | TanStack Query/SWR |
| ইন্টারঅ্যাকটিভ (polling, ফিল্টার) | কঠিন | সহজ |
| SEO | ✅ | ❌ |

**নিয়ম:** SEO লাগে বা secret লাগে → server। ঘনঘন বদলায় বা ব্যবহারকারী ইন্টারঅ্যাকশন করে → client।

**সমান্তরাল fetch (waterfall এড়াতে):**
```tsx
// ❌ waterfall: একটার পর একটা
const user = await getUser();
const posts = await getPosts();

// ✅ একসাথে
const [user, posts] = await Promise.all([getUser(), getPosts()]);
```

---

## 12. SEO ⭐⭐⭐

Next.js এ SEO ভালো হওয়ার কারণ:

1. **তৈরি HTML যায়** (SSR/SSG), তাই crawler JS ছাড়াই কনটেন্ট পড়তে পারে।
2. **Metadata API**
3. **sitemap ও robots** অটো তৈরি
4. **Core Web Vitals** ভালো (image/font optimization)

**স্থির metadata:**
```tsx
// app/about/page.tsx
export const metadata = {
  title: "আমাদের সম্পর্কে",
  description: "আমরা কারা",
  openGraph: {
    title: "আমাদের সম্পর্কে",
    images: ["/og.png"],
  },
};
```

**Dynamic metadata:**
```tsx
export async function generateMetadata({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await getPost(slug);
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: { images: [post.image] },
  };
}
```

**sitemap:**
```ts
// app/sitemap.ts
export default async function sitemap() {
  const posts = await getAllPosts();
  return posts.map((p) => ({
    url: `https://example.com/blog/${p.slug}`,
    lastModified: p.updatedAt,
  }));
}
```

**robots:**
```ts
// app/robots.ts
export default function robots() {
  return {
    rules: { userAgent: "*", allow: "/", disallow: "/admin" },
    sitemap: "https://example.com/sitemap.xml",
  };
}
```

**অতিরিক্ত:** semantic HTML (`<h1>`, `<main>`), canonical URL, structured data (JSON-LD), সুন্দর URL।

---

## 13. Streaming ও Suspense ⭐⭐⭐

**সমস্যা:** একটা ধীর query থাকলে পুরো পেজ আটকে থাকে।
**সমাধান:** দ্রুত অংশ আগে পাঠাও, ধীর অংশ পরে stream করো।

**`loading.tsx` (পুরো route এর জন্য):**
```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <p>লোড হচ্ছে...</p>;
}
```
Next.js নিজে এটাকে `Suspense` এ মুড়ে দেয়।

**`Suspense` (নির্দিষ্ট অংশের জন্য):**
```tsx
import { Suspense } from "react";

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>                      {/* সাথে সাথে দেখা যায় */}
      <Suspense fallback={<p>চার্ট লোড হচ্ছে...</p>}>
        <SlowChart />                         {/* ধীর, পরে আসে */}
      </Suspense>
      <Suspense fallback={<p>অর্ডার লোড হচ্ছে...</p>}>
        <RecentOrders />                      {/* আলাদা, নিজের গতিতে */}
      </Suspense>
    </div>
  );
}

async function SlowChart() {
  const data = await getSlowData();           // ৩ সেকেন্ড
  return <Chart data={data} />;
}
```

**সুবিধা:** TTFB কমে, ব্যবহারকারী দ্রুত কিছু দেখে, একটা ধীর অংশ বাকিগুলো আটকায় না, প্রতিটা অংশ আলাদা loading ও error পায়।

---

## 14. Server-only logic ও secret সুরক্ষা ⭐⭐⭐

**১. `server-only` প্যাকেজ:**
```bash
pnpm add server-only
```
```ts
// lib/db.ts
import "server-only";
import { PrismaClient } from "@prisma/client";

export const prisma = new PrismaClient();
```
কোনো Client Component এই ফাইল import করলে **build এ error** আসবে। ভুল করে secret browser এ যাওয়া ঠেকায়।

**২. Environment variable:**
```bash
# .env.local
DATABASE_URL="..."              # ✅ শুধু server এ
STRIPE_SECRET_KEY="sk_..."      # ✅ শুধু server এ
NEXT_PUBLIC_API_URL="https://..."   # ⚠️ browser এ যায়, secret না রাখো
```

**৩. একটা Data Access Layer, শুধু দরকারি ফিল্ড পাঠাও:**
```ts
// lib/dal.ts
import "server-only";

export async function getUserProfile(id: string) {
  const user = await prisma.user.findUnique({ where: { id } });
  return {                      // ✅ শুধু UI এর দরকারি ফিল্ড
    id: user.id,
    name: user.name,
  };                            // ❌ password hash, email পাঠাও না
}
```

**৪. Server Action এ সবসময় input যাচাই ও auth চেক:**
```ts
"use server";
import { z } from "zod";

const schema = z.object({ name: z.string().min(1) });

export async function updateName(formData: FormData) {
  const session = await requireUser();                       // auth
  const { name } = schema.parse({ name: formData.get("name") }); // validation
  await prisma.user.update({ where: { id: session.user.id }, data: { name } });
}
```

**৫. Server Component থেকে Client Component এ props পাঠানোর সময় সতর্ক:** পুরো DB object না পাঠিয়ে শুধু প্রয়োজনীয় ফিল্ড।

---

## 15. SEO-সহ ঘনঘন বদলানো ডেটা: কোন rendering? ⭐⭐⭐

**সিদ্ধান্ত প্রবাহ:**

```
ডেটা ঘনঘন বদলায়, SEO লাগে
        │
        ├─ কয়েক সেকেন্ড/মিনিটের পুরনো ডেটা চলবে?
        │       └─ হ্যাঁ → ISR (revalidate: ৩০-৬০)  ✅ সবচেয়ে ভালো
        │
        ├─ প্রতি request এ হুবহু নতুন লাগবে?
        │       └─ হ্যাঁ → SSR (cache: "no-store")
        │
        └─ শুধু একটা ছোট অংশ ঘনঘন বদলায়? (দাম, stock)
                └─ পেজ static/ISR রাখো
                   + ওই অংশ Suspense/client fetch
```

```tsx
// ISR: SEO ভালো + প্রায় নতুন ডেটা
export default async function ProductPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const product = await fetch(`https://api.example.com/products/${id}`, {
    next: { revalidate: 60 },
  }).then((r) => r.json());

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      {/* শুধু দাম/stock live */}
      <Suspense fallback={<span>...</span>}>
        <LivePrice id={id} />
      </Suspense>
    </div>
  );
}
```

**বাড়তি:** মালিক দাম বদলালে `revalidateTag("products", "max")` দিয়ে সাথে সাথে নতুন করা।

---

# Tier 2: প্রায়ই আসে

## 16. File-based routing ⭐⭐

ফোল্ডার গঠন = URL গঠন।

```
app/
├── page.tsx                  →  /
├── about/
│   └── page.tsx              →  /about
├── blog/
│   ├── page.tsx              →  /blog
│   └── [slug]/
│       └── page.tsx          →  /blog/hello
└── dashboard/
    ├── layout.tsx            →  dashboard এর shared layout
    ├── loading.tsx
    ├── error.tsx
    └── settings/
        └── page.tsx          →  /dashboard/settings
```

**নিয়ম:**
- `page.tsx` থাকলেই route public।
- `page.tsx` ছাড়া ফোল্ডার (যেমন `components/`) route হয় না।
- `_private` দিয়ে শুরু ফোল্ডার route থেকে বাদ।

**Pages Router এ:**
```
pages/
├── index.tsx        →  /
├── about.tsx        →  /about
└── blog/[slug].tsx  →  /blog/hello
```

---

## 17. Dynamic routes ⭐⭐

| ধরন | ফোল্ডার | URL | `params` |
|---|---|---|---|
| একটা অংশ | `[slug]` | `/blog/hello` | `{ slug: "hello" }` |
| সব অংশ | `[...slug]` | `/shop/a/b/c` | `{ slug: ["a","b","c"] }` |
| ঐচ্ছিক সব অংশ | `[[...slug]]` | `/shop` বা `/shop/a/b` | `{ slug?: [...] }` |

**Next.js 15+ এ `params` একটা Promise:**
```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;      // await লাগে
  const post = await getPost(slug);
  return <h1>{post.title}</h1>;
}
```

**`searchParams` ও একই (`/blog?page=2`):**
```tsx
export default async function Page({
  searchParams,
}: {
  searchParams: Promise<{ page?: string }>;
}) {
  const { page = "1" } = await searchParams;
}
```

**আগে থেকে static পেজ বানানো:**
```tsx
export async function generateStaticParams() {
  const posts = await getAllPosts();
  return posts.map((p) => ({ slug: p.slug }));
}
```

---

## 18. `next/link` ও `next/image` ⭐⭐

**Link:**
```tsx
import Link from "next/link";

<Link href="/about">About</Link>
<Link href={`/blog/${post.slug}`}>পড়ো</Link>
<Link href="/dashboard" prefetch={false}>Dashboard</Link>
```
- পুরো পেজ reload হয় না (client-side navigation)।
- Viewport এ দেখা গেলে পেজ আগে থেকে prefetch হয়।
- সাধারণ `<a>` এর বদলে সবসময় `Link`।

**Image:**
```tsx
import Image from "next/image";

// স্থানীয় ছবি (আকার নিজে জানা যায়)
import hero from "./hero.jpg";
<Image src={hero} alt="হিরো ছবি" priority />

// বাইরের ছবি: width/height দিতে হয়
<Image src="https://cdn.example.com/a.jpg" alt="..." width={800} height={600} />

// কন্টেইনার ভরাতে
<div style={{ position: "relative", height: 300 }}>
  <Image src="/banner.jpg" alt="..." fill sizes="100vw" style={{ objectFit: "cover" }} />
</div>
```

**বাইরের domain অনুমতি:**
```ts
// next.config.ts
const nextConfig = {
  images: {
    remotePatterns: [
      { protocol: "https", hostname: "cdn.example.com" },
    ],
  },
};
export default nextConfig;
```

**কী করে:** WebP/AVIF এ বদলায়, ডিভাইস অনুযায়ী ছোট বড় করে, lazy load করে, layout shift ঠেকায়। `priority` শুধু উপরের (above-the-fold) প্রধান ছবিতে।

---

## 19. API Routes / Route Handlers ⭐⭐

**App Router:**
```ts
// app/api/posts/[id]/route.ts
import { NextResponse } from "next/server";

export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const post = await prisma.post.findUnique({ where: { id } });
  if (!post) return NextResponse.json({ error: "Not found" }, { status: 404 });
  return NextResponse.json(post);
}

export async function DELETE(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  await requireAdmin();                    // auth চেক ভুলো না
  await prisma.post.delete({ where: { id } });
  return new Response(null, { status: 204 });
}
```

**Pages Router:**
```ts
// pages/api/hello.ts
import type { NextApiRequest, NextApiResponse } from "next";

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === "GET") return res.status(200).json({ message: "hi" });
  res.status(405).end();
}
```

**Webhook (যেমন Stripe):**
```ts
// app/api/webhooks/stripe/route.ts
export async function POST(request: Request) {
  const body = await request.text();                 // raw body লাগে
  const signature = request.headers.get("stripe-signature")!;
  const event = stripe.webhooks.constructEvent(body, signature, process.env.STRIPE_WEBHOOK_SECRET!);
  // ... event সামলাও
  return new Response("ok");
}
```

**Route Handler বনাম Server Action:**

| | Route Handler | Server Action |
|---|---|---|
| কার জন্য | বাইরের client, webhook, public API | নিজের UI এর form/mutation |
| কীভাবে call | HTTP request | সরাসরি ফাংশন call |
| উদাহরণ | Stripe webhook, মোবাইল অ্যাপের API | "পোস্ট সেভ করো" বাটন |

**মনে রাখো:** 15+ এ `GET` handler ডিফল্টে cache হয় না। cache চাইলে `export const dynamic = "force-static"`।

---

## 20. Environment variables ⭐⭐

**ফাইল:**
```
.env                  # সব পরিবেশ
.env.local            # লোকাল (git এ দিও না)
.env.development      # dev
.env.production       # production
```

**নিয়ম:**
```bash
DATABASE_URL="postgresql://..."          # শুধু server
NEXT_PUBLIC_SITE_URL="https://example.com"   # browser এও যায়
```

```tsx
// server এ ✅
process.env.DATABASE_URL

// client এ
process.env.NEXT_PUBLIC_SITE_URL   // ✅ কাজ করে
process.env.DATABASE_URL           // ❌ undefined
```

**গুরুত্বপূর্ণ বিষয়:**
- `NEXT_PUBLIC_` ওয়ালা মান **build এর সময় কোডে বসে যায়**। পরে বদলালে আবার build লাগে।
- Secret এ কখনো `NEXT_PUBLIC_` দিও না, এটা সবার browser এ দেখা যায়।
- `.env.local` কে `.gitignore` এ রাখো।
- Hosting platform (Vercel ইত্যাদি) এ আলাদা করে সেট করো, ফাইল push হয় না।
- Type-safe করতে: `@t3-oss/env-nextjs` বা Zod দিয়ে চালুর সময় যাচাই।

```ts
// lib/env.ts
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().min(1),
});

export const env = envSchema.parse(process.env);   // না থাকলে শুরুতেই error
```

---

## 21. Code splitting ⭐⭐

**অটো:** প্রতিটা route নিজের JS bundle পায়। `/about` এ গেলে শুধু ওই পেজের কোড লোড হয়।

**Server Component:** এর JS browser এ যায়ই না।

**ভারী component কে দেরিতে লোড:**
```tsx
"use client";
import dynamic from "next/dynamic";

const HeavyChart = dynamic(() => import("./HeavyChart"), {
  loading: () => <p>চার্ট লোড হচ্ছে...</p>,
  ssr: false,          // শুধু browser এ দরকার হলে
});

export default function Page() {
  return <HeavyChart />;
}
```

**শর্তসাপেক্ষ লোড (modal খুললে তবেই):**
```tsx
const [open, setOpen] = useState(false);
const Modal = dynamic(() => import("./Modal"));
return <>{open && <Modal />}</>;
```

**Bundle মাপতে:**
```bash
pnpm add -D @next/bundle-analyzer
```

**`next/link`** viewport এ দেখা গেলে পরের পেজের কোড আগে থেকে নামিয়ে রাখে (prefetch)।

---

## 22. `useRouter` বনাম `useParams` ⭐⭐

সবগুলো `next/navigation` থেকে, শুধু **Client Component** এ।

```tsx
"use client";
import { useRouter, useParams, usePathname, useSearchParams } from "next/navigation";

export default function Example() {
  const router = useRouter();
  const params = useParams<{ id: string }>();     // /users/42 → { id: "42" }
  const pathname = usePathname();                 // "/users/42"
  const searchParams = useSearchParams();         // ?tab=posts
  const tab = searchParams.get("tab");

  return (
    <>
      <p>ID: {params.id}</p>
      <button onClick={() => router.push("/dashboard")}>যাও</button>
      <button onClick={() => router.replace("/login")}>বদলাও (history এ থাকবে না)</button>
      <button onClick={() => router.back()}>পিছনে</button>
      <button onClick={() => router.refresh()}>server ডেটা নতুন করো</button>
    </>
  );
}
```

| Hook | কাজ |
|---|---|
| `useRouter` | **নেভিগেট করা** (`push`, `replace`, `back`, `refresh`, `prefetch`) |
| `useParams` | dynamic মান **পড়া** (`/users/[id]` এর `id`) |
| `usePathname` | বর্তমান path পড়া |
| `useSearchParams` | query string পড়া |

**Server Component এ:** hook নেই, `params` ও `searchParams` prop ব্যবহার করো (দুটোই Promise, `await` করো)।

**Pages Router এ:** `useRouter` from `next/router`, যাতে `router.query` আছে (নাম মিলিও না)।

---

## 23. Nested routing ও layouts ⭐⭐

```
app/
├── layout.tsx                  ← root layout (html, body)
└── dashboard/
    ├── layout.tsx              ← sidebar সহ layout
    ├── page.tsx                ← /dashboard
    └── settings/
        └── page.tsx            ← /dashboard/settings
```

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="bn">
      <body>{children}</body>
    </html>
  );
}

// app/dashboard/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

**`/dashboard/settings` এ কী ঘটে:**
```
RootLayout
  └─ DashboardLayout
       └─ settings/page.tsx
```

**সুবিধা:**
- `/dashboard` থেকে `/dashboard/settings` এ গেলে `DashboardLayout` **আবার render হয় না**, sidebar এর state ঠিক থাকে।
- শুধু `{children}` অংশ বদলায়।
- প্রতিটা স্তরে নিজস্ব `loading.tsx` ও `error.tsx` রাখা যায়।

**`template.tsx`:** layout এর মতো, কিন্তু প্রতি নেভিগেশনে নতুন করে mount হয় (animation বা reset দরকার হলে)।

---

## 24. Image ও font optimization ⭐⭐

**Image:**
```tsx
<Image
  src="/hero.jpg"
  alt="বর্ণনা"
  width={1200}
  height={600}
  sizes="(max-width: 768px) 100vw, 50vw"   // ডিভাইস অনুযায়ী সঠিক আকার
  priority                                   // শুধু উপরের প্রধান ছবিতে
  placeholder="blur"                         // আসার আগে ঝাপসা দেখাবে
/>
```
- `sizes` ঠিক দিলে মোবাইলে অকারণে বড় ছবি নামে না।
- নিচের ছবিতে `priority` দিও না (lazy load হতে দাও)।
- সবসময় `alt` দাও।

**Font:**
```tsx
// app/layout.tsx
import { Inter } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  weight: ["400", "700"],     // শুধু যা লাগে
  display: "swap",
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```
- Build এর সময় ফন্ট নামিয়ে নিজের সাইটের সাথে দেয়, Google এ আলাদা request যায় না।
- Layout shift ঠেকায়।
- বাংলা ফন্টের জন্য `subsets: ["bengali"]` (যেমন `Noto_Sans_Bengali`)।

---

## 25. TTFB কমানো ⭐⭐

**TTFB** = Time To First Byte, request পাঠানো থেকে প্রথম বাইট আসা পর্যন্ত সময়।

| কৌশল | কীভাবে |
|---|---|
| Static/ISR | cache ও CDN থেকে দাও, server কাজ করে না |
| Waterfall এড়াও | `Promise.all` দিয়ে একসাথে আনো |
| Streaming | `Suspense`, ধীর অংশ পরে আসুক |
| DB ঠিক করো | index, ধীর query, connection pooling |
| Server কাছে রাখো | DB ও ব্যবহারকারীর কাছাকাছি region |
| Cache | `"use cache"`, `revalidate`, Redis |
| Proxy হালকা রাখো | ভারী কাজ নয় |
| Compression, HTTP/2 | hosting এ সাধারণত অটো |

```tsx
// ❌ waterfall
const user = await getUser();
const orders = await getOrders(user.id);   // user এর উপর নির্ভর, ঠিক আছে
const stats = await getStats();            // স্বাধীন, তবু অপেক্ষা করছে!

// ✅
const user = await getUser();
const [orders, stats] = await Promise.all([getOrders(user.id), getStats()]);
```

**মাপতে:** Lighthouse, Chrome DevTools Network, Vercel Analytics/Speed Insights।

---

## 26. JWT যাচাই কোথায় ⭐⭐

**নিয়ম: সবসময় server এ। Client এ decode করা যায়, কিন্তু বিশ্বাস করা যায় না।**

**Token কোথায় রাখবে:**
```ts
// ✅ HttpOnly cookie: JS পড়তে পারে না, XSS থেকে নিরাপদ
cookies().set("token", jwt, {
  httpOnly: true,
  secure: true,
  sameSite: "lax",
  path: "/",
  maxAge: 60 * 60 * 24 * 7,
});

// ❌ localStorage: XSS এ চুরি হতে পারে
localStorage.setItem("token", jwt);
```

**যাচাই (jose লাইব্রেরি):**
```bash
pnpm add jose
```
```ts
// lib/session.ts
import "server-only";
import { jwtVerify } from "jose";
import { cookies } from "next/headers";

const secret = new TextEncoder().encode(process.env.JWT_SECRET);

export async function getSession() {
  const token = (await cookies()).get("token")?.value;
  if (!token) return null;
  try {
    const { payload } = await jwtVerify(token, secret);   // signature ও মেয়াদ যাচাই
    return payload;
  } catch {
    return null;
  }
}
```

**কোথায় কোথায় যাচাই:**

| জায়গা | ভূমিকা |
|---|---|
| Server Component / layout | আসল যাচাই, render এর আগে |
| Server Action | প্রতিটায় |
| Route Handler | প্রতিটায় |
| `proxy.ts` | দ্রুত চেক (ঐচ্ছিক), একমাত্র সুরক্ষা না |

**যা যাচাই করবে:** signature, মেয়াদ (`exp`), role/permission।

**ভালো অভ্যাস:** access token ছোট মেয়াদের, আলাদা refresh token, প্রতিটা অনুরোধে ডেটা-লেভেল permission চেক।

---

## 27. SSG তেও পেজ ধীর: কী করবে ⭐⭐

**মূল কথা:** SSG মানে HTML তৈরি। ধীরতা সম্ভবত **browser এ**।

**ধাপ ১: মাপো**
```bash
pnpm add -D @next/bundle-analyzer
```
```ts
// next.config.ts
import withBundleAnalyzer from "@next/bundle-analyzer";

const nextConfig = {};
export default withBundleAnalyzer({ enabled: process.env.ANALYZE === "true" })(nextConfig);
```
```bash
ANALYZE=true pnpm build
```
এবং Lighthouse চালাও। LCP, CLS, TBT দেখো।

**ধাপ ২: সম্ভাব্য কারণ ও সমাধান**

| সমস্যা | সমাধান |
|---|---|
| বড় JS bundle | `next/dynamic`, অপ্রয়োজনীয় লাইব্রেরি সরাও (যেমন পুরো `lodash` এর বদলে নির্দিষ্ট ফাংশন) |
| অনেক Client Component | যতটা সম্ভব Server Component এ নাও |
| ভারী third-party script | `next/script` এ `strategy="lazyOnload"` |
| বড় ছবি | `next/image`, `sizes`, WebP/AVIF |
| ধীর ফন্ট | `next/font`, কম weight |
| Layout shift | ছবি/ফন্টে জায়গা ঠিক রাখো |
| CDN cache হচ্ছে না | response header ও hosting সেটিং দেখো |
| Hydration ভারী | client এ কম কাজ, বড় তালিকায় virtualization |

```tsx
import Script from "next/script";

<Script src="https://example.com/analytics.js" strategy="lazyOnload" />
```

**ধাপ ৩:** আবার মাপো ও তুলনা করো।

---

## 28. লোকালে চলে, deploy এ ফেল: debug ⭐⭐

**ধাপ ১: log পড়ো** (hosting এর build log ও runtime log)।

**ধাপ ২: লোকালে production এর মতো চালাও**
```bash
pnpm build
pnpm start
```
Dev mode ও production আলাদা আচরণ করে (বিশেষত caching, error, optimization)। অনেক বাগ শুধু এখানেই ধরা পড়ে।

**ধাপ ৩: সাধারণ কারণের তালিকা**

| কারণ | কীভাবে ধরবে/ঠিক করবে |
|---|---|
| Env variable নেই | hosting এ সেট করা আছে কিনা |
| `NEXT_PUBLIC_` build এর সময় সেট নেই | সেট করে **আবার build** |
| ফাইলের নামের বড়/ছোট হাতের অক্ষর | `Header.tsx` কে `header` import (Mac/Windows এ চলে, Linux এ না) |
| DB তে অনুমতি নেই | DB এর IP allowlist, SSL সেটিং |
| Node সংস্করণ আলাদা | `engines` / hosting এ সংস্করণ মেলাও |
| `localhost` URL hardcode | আসল domain / env variable |
| Image domain অনুমতি নেই | `remotePatterns` |
| Prisma client তৈরি হয়নি | build এ `prisma generate` চালাও |
| Dev dependency এ দরকারি প্যাকেজ | `dependencies` এ নাও |
| Edge/Node runtime অমিল | কোন runtime এ চলছে দেখো |
| Caching আচরণ | prod এ ডেটা পুরনো কিনা, `revalidate` |
| Hydration error | browser console |
| CORS | production domain অনুমতি দাও |

**Prisma এর জন্য পরামর্শ:**
```json
{
  "scripts": {
    "postinstall": "prisma generate",
    "build": "prisma generate && next build"
  }
}
```

**ধাপ ৪:** কারণ ঠিক করো, আবার deploy, নিশ্চিত করো।

---

# Tier 3: মাঝে মাঝে আসে

## 29. Route groups ⭐

ফোল্ডারের নাম `( )` তে দিলে **URL এ আসে না**।

```
app/
├── (marketing)/
│   ├── layout.tsx         ← marketing layout (নেভবার, ফুটার)
│   ├── page.tsx           →  /
│   └── about/page.tsx     →  /about
└── (app)/
    ├── layout.tsx         ← dashboard layout (sidebar)
    └── dashboard/page.tsx →  /dashboard
```

`(marketing)` ও `(app)` URL এ নেই।

**ব্যবহার:**
- আলাদা অংশে আলাদা layout
- প্রজেক্ট গোছানো (auth, admin, public)
- একই স্তরে একাধিক root layout

```
app/
├── (auth)/login/page.tsx        → /login
├── (auth)/register/page.tsx     → /register
└── (admin)/admin/page.tsx       → /admin
```

---

## 30. Parallel routes ⭐

`@` দিয়ে শুরু ফোল্ডার = **slot**। Layout এ একসাথে একাধিক পেজ দেখানো যায়।

```
app/dashboard/
├── layout.tsx
├── page.tsx
├── @analytics/
│   ├── page.tsx
│   └── default.tsx        ← Next.js 16 এ আবশ্যক
└── @team/
    ├── page.tsx
    └── default.tsx
```

```tsx
// app/dashboard/layout.tsx
export default function Layout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
}) {
  return (
    <div>
      {children}
      <div className="grid grid-cols-2">
        {analytics}
        {team}
      </div>
    </div>
  );
}
```

```tsx
// default.tsx (slot এর সাথে মিলে এমন route না থাকলে)
export default function Default() {
  return null;
}
```

**সুবিধা:**
- প্রতিটা slot আলাদাভাবে লোড, error, stream হয়।
- শর্তসাপেক্ষ দেখানো যায় (role অনুযায়ী ভিন্ন slot)।
- **Intercepting routes** এর সাথে মিলে modal বানানো যায় (যেমন ছবিতে ক্লিক করলে modal, সরাসরি URL এ পূর্ণ পেজ)।

---

## 31. SSO ⭐

**SSO** = একবার লগইন করে একাধিক অ্যাপে প্রবেশ (Google Workspace, Okta, Azure AD ইত্যাদি)।

**প্রোটোকল:** OIDC (আধুনিক, সহজ) বা SAML (পুরনো, এন্টারপ্রাইজে বেশি)।

**প্রবাহ (Authorization Code + PKCE):**
```
1. ব্যবহারকারী "SSO দিয়ে লগইন" চাপে
2. তোমার অ্যাপ তাকে Identity Provider (IdP) এ redirect করে
3. IdP তে সে লগইন করে
4. IdP তাকে তোমার callback URL এ ফেরত পাঠায় (সাথে একটা code)
5. তোমার server code দিয়ে token নেয় (server-to-server)
6. Token যাচাই করে (signature, issuer, audience, মেয়াদ)
7. ব্যবহারকারী তৈরি/খুঁজে session বানায় (HttpOnly cookie)
8. এরপর সব অনুরোধে server এ session যাচাই
```

**বাস্তবায়ন:** নিজে বানিও না। এগুলো ব্যবহার করো:
- Auth.js, Better Auth (OIDC/OAuth সমর্থন)
- Auth0, WorkOS, Clerk (এন্টারপ্রাইজ SSO/SAML সহজ করে)

**মাল্টি-টেন্যান্ট SaaS এ:** প্রতিটা organization এর নিজস্ব IdP কনফিগ ডাটাবেসে রাখতে হয়, আর ইমেইল domain দেখে কোন IdP তে পাঠাতে হবে ঠিক করা হয়। এ ক্ষেত্রে WorkOS এর মতো সেবা সময় বাঁচায়।

**সতর্কতা:** `state` ও `nonce` যাচাই করো (CSRF ও replay আটকাতে), callback URL কঠোরভাবে allowlist করো, session ঠিকমতো শেষ করো (logout)।

---

## 32. Traffic spike সামলানো ⭐

| স্তর | কৌশল |
|---|---|
| **কম কাজ করা** | SSG/ISR + CDN, যাতে বেশিরভাগ request server ছোঁয় না |
| **Cache** | `use cache`, `revalidate`, Redis, CDN cache header |
| **ডাটাবেস** | connection pooling (PgBouncer, Prisma Accelerate), index, read replica |
| **Scale** | স্বয়ংক্রিয় scale হয় এমন platform (serverless/container autoscaling) |
| **সুরক্ষা** | rate limiting, bot protection, WAF |
| **ধীর কাজ সরাও** | email, রিপোর্ট queue তে (BullMQ, Inngest, ইত্যাদি) |
| **Graceful degradation** | ভারী অংশ বন্ধ রেখে মূল অংশ চালু রাখা |
| **প্রস্তুতি** | ইভেন্টের আগে load test (k6, Artillery) |
| **নজরদারি** | monitoring, alert, error tracking (Sentry) |

**Serverless এ ডাটাবেসের সতর্কতা:** প্রতিটা ফাংশন নতুন connection খুলতে পারে, তাই হাজার request এ DB এর connection শেষ হয়ে যেতে পারে। Pooling আবশ্যক।

```ts
// lib/prisma.ts: dev এ বারবার নতুন client এড়াতে
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };
export const prisma = globalForPrisma.prisma ?? new PrismaClient();
if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```
