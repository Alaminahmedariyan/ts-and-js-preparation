# System Design Details (বিস্তারিত বোঝার জন্য)

প্রতিটা টপিক সহজ বাংলায়, ডায়াগ্রাম ও কোড উদাহরণসহ। ইন্টারভিউয়ের ছোট ইংরেজি উত্তরের জন্য `system-design-interview.md` দেখো।

**ক্রম:** Tier 1 (সবচেয়ে বেশি আসে) → Tier 2। নম্বর `system-design-interview.md` এর সাথে মেলানো।

---

## আগে এই কাঠামোটা শিখে নাও (যেকোনো system design প্রশ্নে কাজে লাগে)

প্রশ্ন পেলে সরাসরি ডিজাইনে ঝাঁপ দিও না। এই ক্রমে এগোও:

```
১. চাহিদা পরিষ্কার করো   → কী করতে হবে? কতজন ব্যবহারকারী? পড়া বেশি না লেখা বেশি?
২. সংখ্যার আন্দাজ         → প্রতি সেকেন্ডে কত request? কত ডেটা?
৩. সহজ ডিজাইন আঁকো       → Client → API → Database
৪. বোতলের গলা খোঁজো       → কোথায় আটকাবে? (DB? একটা server?)
৫. উন্নত করো              → cache, replica, queue, CDN
৬. ট্রেড-অফ বলো           → "এটা নিলে এটা পাই, এটা হারাই"
৭. ব্যর্থতা ভাবো           → কিছু নষ্ট হলে কী হবে?
```

**ইন্টারভিউয়ে সবচেয়ে বড় ভুল:** প্রশ্ন না করেই "Kafka, microservices, Kubernetes" বলে দেওয়া। সহজ থেকে শুরু করে প্রয়োজনে জটিল করো।

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. Monolith বনাম Microservices ⭐⭐⭐

```
MONOLITH                          MICROSERVICES
┌─────────────────────┐           ┌──────┐  ┌──────┐  ┌──────┐
│  Auth               │           │ Auth │  │Orders│  │ Pay  │
│  Orders             │           └──┬───┘  └──┬───┘  └──┬───┘
│  Payments           │              │         │         │
│  Notifications      │           ┌──▼──┐   ┌──▼──┐   ┌──▼──┐
│  সব এক codebase     │           │ DB  │   │ DB  │   │ DB  │
└──────────┬──────────┘           └─────┘   └─────┘   └─────┘
           │                       প্রতিটা আলাদা deploy, নেটওয়ার্কে কথা
      ┌────▼────┐
      │   DB    │
      └─────────┘
```

| বিষয় | Monolith | Microservices |
|---|---|---|
| Deploy | একসাথে সব | প্রতিটা আলাদা |
| Scaling | পুরোটা একসাথে | যে অংশ চাপে শুধু সেটা |
| ডেভেলপমেন্ট | সহজ, দ্রুত শুরু | জটিল সেটআপ |
| Debug | সহজ (এক জায়গায়) | কঠিন (অনেক সেবা জুড়ে) |
| টেস্ট | সহজ | integration টেস্ট কঠিন |
| ডেটা | একটা DB, transaction সহজ | আলাদা DB, বিতরিত transaction কঠিন |
| টিম | ছোট টিমে ভালো | বড় টিমে আলাদা মালিকানা |
| ব্যর্থতা | একটা bug পুরো অ্যাপ ফেলতে পারে | একটা সেবা নষ্ট, বাকি চলে (ঠিকমতো করলে) |
| বাড়তি খরচ | কম | নেটওয়ার্ক latency, monitoring, CI/CD |

### কখন Monolith

- নতুন প্রোডাক্ট, চাহিদা এখনো অস্পষ্ট
- ছোট টিম (কয়েকজন)
- দ্রুত বাজারে যেতে হবে
- Domain সীমা এখনো বোঝা যায়নি (ভুল জায়গায় ভাগ করলে পরে ঠিক করা কঠিন)

### কখন Microservices

- বড় টিম, আলাদা টিম আলাদা অংশে
- কিছু অংশে সম্পূর্ণ আলাদা scaling দরকার (যেমন ভিডিও প্রসেসিং)
- আলাদা প্রযুক্তি দরকার
- সীমা পরিষ্কার ও স্থির

### মাঝামাঝি পথ: Modular Monolith

একই অ্যাপ, কিন্তু ভেতরে পরিষ্কার module (auth, orders, payments), প্রতিটার সীমা কঠোর। পরে দরকার হলে একটা module কে সেবা হিসেবে আলাদা করা সহজ।

```
src/
├── modules/
│   ├── auth/       ← নিজস্ব service, repository, types
│   ├── orders/
│   └── payments/
└── shared/
```

**ইন্টারভিউয়ের সেরা উত্তর:** "আমি monolith দিয়ে শুরু করব, module গুলো পরিষ্কার রাখব, আর যখন নির্দিষ্ট অংশে scaling বা টিমের সমস্যা সত্যিই দেখা দেবে তখন microservices এ যাব।"

---

## 2. Horizontal বনাম Vertical scaling ⭐⭐⭐

```
VERTICAL (scale up)                HORIZONTAL (scale out)
                                   
  ┌────────┐      ┌────────────┐      ┌───┐ ┌───┐ ┌───┐ ┌───┐
  │ 4 CPU  │  →   │  32 CPU    │      │ S │ │ S │ │ S │ │ S │
  │ 8 GB   │      │  128 GB    │      └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘
  └────────┘      └────────────┘        └─────┴──┬──┴─────┘
  একটাই server কে বড় করা                  Load Balancer
```

| | Vertical | Horizontal |
|---|---|---|
| কীভাবে | বেশি CPU/RAM/disk | বেশি server |
| জটিলতা | সহজ (কোড বদল নেই) | বেশি (LB, stateless দরকার) |
| সীমা | আছে (সবচেয়ে বড় মেশিন) | প্রায় নেই |
| খরচ | বড় মেশিন অনেক ব্যয়বহুল | সস্তা মেশিন অনেকগুলো |
| Availability | ❌ single point of failure | ✅ একটা নষ্ট হলে বাকিরা চলে |
| Downtime | আপগ্রেডে downtime | ছাড়াই (rolling) |
| ডেটাবেসে | সহজ | কঠিন (replica/shard) |

**Horizontal এর শর্ত:** অ্যাপ **stateless** হতে হবে (দেখো টপিক ১২)।

**বাস্তব সিদ্ধান্ত:**
- অ্যাপ server → horizontal (সহজে করা যায়)
- ডাটাবেস → আগে vertical + read replica, পরে দরকারে sharding
- কখন vertical: শুরুতে, সহজতার জন্য
- কখন horizontal: সীমা ছুঁলে, বা high availability লাগলে

**Auto-scaling:** CPU/memory/request সংখ্যা দেখে server সংখ্যা নিজে নিজে বাড়ে-কমে (AWS Auto Scaling, Kubernetes HPA)।

---

## 3. Load Balancer ⭐⭐⭐

```
        Client
          │
   ┌──────▼──────┐
   │ Load Balancer│ ← health check করে: "কে সুস্থ?"
   └──┬────┬────┬─┘
      │    │    ╳ (নষ্ট server এ request যায় না)
   ┌──▼┐ ┌─▼─┐ ┌─▼─┐
   │S1 │ │S2 │ │S3 │
   └───┘ └───┘ └───┘
```

**কেন দরকার:**
1. **চাপ ভাগ:** একটা server ডুবে যায় না
2. **Availability:** নষ্ট server কে বাদ দেয়
3. **Horizontal scaling:** নতুন server যোগ করলে ক্লায়েন্ট টের পায় না
4. **Zero-downtime deploy:** একটা একটা করে server বদলানো
5. **SSL termination:** HTTPS সামলায়, server গুলো হালকা থাকে

### Algorithm

| Algorithm | কীভাবে | কখন |
|---|---|---|
| **Round Robin** | পালা করে S1, S2, S3, S1... | server একই শক্তির, request একই ধরনের |
| **Weighted Round Robin** | শক্তিশালী server বেশি পায় | server এর শক্তি আলাদা |
| **Least Connections** | যার কাছে কম সংযোগ তাকে | request এর সময় আলাদা (লম্বা সংযোগ) |
| **IP Hash** | client IP থেকে সবসময় একই server | sticky session দরকার |

### Layer 4 বনাম Layer 7

| | L4 (Transport) | L7 (Application) |
|---|---|---|
| কী দেখে | IP, port | URL, header, cookie |
| গতি | দ্রুত | একটু ধীর |
| ক্ষমতা | সরল | `/api` এক দিকে, `/images` অন্য দিকে |

### উদাহরণ: Nginx

```nginx
upstream app_servers {
    least_conn;
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
    server 10.0.0.3:3000 backup;     # অন্যরা নষ্ট হলে
}

server {
    listen 80;
    location / {
        proxy_pass http://app_servers;
    }
}
```

**টুল:** Nginx, HAProxy, AWS ALB/NLB, Cloudflare।

**সতর্কতা:** LB নিজেই single point of failure হতে পারে। তাই ক্লাউডের managed LB (যা নিজে redundant) বা একাধিক LB রাখা হয়।

**Sticky session:** একই ব্যবহারকারী সবসময় একই server এ। stateless হলে দরকার নেই। WebSocket এ প্রায়ই লাগে।

---

## 4. Caching ও Redis ⭐⭐⭐

**Caching:** ধীর বা ব্যয়বহুল কাজের ফল দ্রুত জায়গায় রাখা।

### কোথায় cache করা যায় (ক্লায়েন্ট থেকে ডেটা পর্যন্ত)

```
Browser cache  →  CDN  →  API Gateway  →  App memory  →  Redis  →  DB cache
 (সবচেয়ে কাছে)                                                  (সবচেয়ে দূরে)
```

| স্তর | কী রাখে | উদাহরণ |
|---|---|---|
| Browser | ছবি, CSS, JS | `Cache-Control` header |
| CDN | static ফাইল, পেজ | Cloudflare, CloudFront |
| App memory | ছোট, ঘনঘন ব্যবহৃত | `Map`, LRU cache |
| Redis | query ফল, session | shared, সব server দেখে |
| Database | query plan, buffer | PostgreSQL এর নিজস্ব |

### Cache কৌশল (patterns)

**১. Cache-Aside (সবচেয়ে সাধারণ):**
```ts
async function getUser(id: string) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);              // ✅ cache hit

  const user = await prisma.user.findUnique({ where: { id } });   // ❌ miss → DB
  await redis.set(`user:${id}`, JSON.stringify(user), "EX", 300); // ৫ মিনিট
  return user;
}

async function updateUser(id: string, data: UserUpdate) {
  const user = await prisma.user.update({ where: { id }, data });
  await redis.del(`user:${id}`);                       // বদলালে cache মুছো
  return user;
}
```

| Pattern | কীভাবে | কখন |
|---|---|---|
| **Cache-Aside** | অ্যাপ নিজে cache ও DB দেখে | সাধারণ, সবচেয়ে নমনীয় |
| **Write-Through** | লেখার সময় cache ও DB দুটোতেই | পড়া ঘনঘন, সামঞ্জস্য দরকার |
| **Write-Behind** | আগে cache এ, পরে DB তে | লেখা অনেক, কিছু ডেটা হারানোর ঝুঁকি মানা যায় |

### Cache invalidation (সবচেয়ে কঠিন অংশ)

> "কম্পিউটার সায়েন্সে দুটো কঠিন জিনিস: cache invalidation আর নাম দেওয়া।"

| উপায় | কীভাবে |
|---|---|
| **TTL** (মেয়াদ) | নির্দিষ্ট সময় পর নিজে মুছে যায় |
| **সরাসরি মোছা** | ডেটা বদলালে cache মুছো |
| **Versioning** | key তে সংস্করণ (`user:42:v3`) |

### Eviction (জায়গা শেষ হলে কী মুছবে)

LRU (সবচেয়ে কম সম্প্রতি ব্যবহৃত), LFU (সবচেয়ে কম ব্যবহৃত), TTL।

### সাধারণ সমস্যা

| সমস্যা | মানে | সমাধান |
|---|---|---|
| **Cache stampede** | জনপ্রিয় key মেয়াদ শেষ, হাজার request একসাথে DB তে | lock, মেয়াদে random জুড়ে দেওয়া, আগে থেকে refresh |
| **Cache penetration** | যে key আসলে নেই তা বারবার চাওয়া, প্রতিবার DB তে | "নেই" ফলও ছোট TTL এ cache, Bloom filter |
| **পুরনো ডেটা** | DB বদলেছে, cache পুরনো | TTL, বদলে মোছা |

### Redis কী ও কেন

**Redis:** in-memory (RAM এ) key-value store। খুব দ্রুত (মাইক্রোসেকেন্ড)।

| Data type | ব্যবহার |
|---|---|
| String | সাধারণ cache, counter |
| Hash | object (user এর ফিল্ড) |
| List | সহজ queue |
| Set | অনন্য মান (online users) |
| Sorted Set | leaderboard, rate limit |
| Pub/Sub | real-time বার্তা |
| Stream | event log |

**Redis কোথায় ব্যবহার করবে:**
1. Query ফল cache
2. Session সংরক্ষণ
3. Rate limiting
4. Queue (BullMQ)
5. Leaderboard
6. Distributed lock
7. Pub/Sub (Socket.io এর একাধিক server)

**সতর্কতা:** Redis memory তে থাকে, restart এ ডেটা যেতে পারে (persistence চালু না থাকলে)। তাই এটাকে একমাত্র সত্যের উৎস (source of truth) বানিও না, DB ই আসল।

---

## 5. ১ লক্ষ concurrent user সামলানো ⭐⭐⭐

### আগে প্রশ্ন করো

| প্রশ্ন | কেন |
|---|---|
| Concurrent user মানে কী? (একসাথে অনলাইন, নাকি একসাথে request) | ১ লক্ষ online ≠ ১ লক্ষ RPS। বেশিরভাগ বসে থাকে |
| পড়া বেশি না লেখা বেশি? | পড়া বেশি → cache/replica। লেখা বেশি → queue/shard |
| কোন কোন ফিচার? (static, real-time, ভিডিও) | WebSocket এ আলাদা ভাবনা |
| Consistency কতটা জরুরি? | টাকা হলে শক্ত, feed হলে শিথিল |
| ব্যর্থতা সহনীয় কতটা? | 99.9% না 99.99%? |

### সংখ্যার আন্দাজ (উদাহরণ)

ধরো ১ লক্ষ online user, প্রতি ১০ সেকেন্ডে একবার request করে → ≈ **১০,০০০ RPS**। প্রতি server ১,০০০ RPS সামলালে ≈ ১০টা server (+ বাড়তি নিরাপত্তা)।

### স্থাপত্য

```
                    ┌────────────┐
   Users ──────────►│    CDN     │ ← static ফাইল, ছবি, cache করা পেজ
                    └─────┬──────┘
                          │ (বাকিটা)
                    ┌─────▼──────┐
                    │ Load Balancer│ + Rate Limiter / WAF
                    └─────┬──────┘
          ┌───────────────┼───────────────┐
      ┌───▼───┐       ┌───▼───┐       ┌───▼───┐
      │ App 1 │       │ App 2 │  ...  │ App N │   ← stateless, auto-scale
      └───┬───┘       └───┬───┘       └───┬───┘
          └───────┬───────┴───────┬───────┘
             ┌────▼────┐     ┌────▼─────┐
             │  Redis  │     │  Queue   │ → Workers (email, রিপোর্ট)
             │ (cache) │     └──────────┘
             └────┬────┘
        ┌─────────▼─────────┐
        │ PgBouncer (pool)  │
        └─────────┬─────────┘
          ┌───────▼───────┐
          │ Primary (লেখা) │──► Replica 1, Replica 2 (পড়া)
          └───────────────┘
```

### প্রতিটা স্তরের কাজ

| স্তর | কী করে | কেন |
|---|---|---|
| CDN | static ও cacheable কনটেন্ট | অনেক request server ছোঁয়ই না |
| LB + অনেক stateless server | চাপ ভাগ, auto-scale | horizontal scaling |
| Redis cache | গরম ডেটা | DB কম চাপে |
| DB index | query দ্রুত | |
| Connection pooling | PgBouncer | হাজার server connection এ DB ভেঙে পড়ে |
| Read replica | পড়ার চাপ ভাগ | |
| Sharding | শুধু লেখা না সামলালে | শেষ উপায় |
| Queue | ধীর কাজ আলাদা | request দ্রুত ফেরে |
| Rate limiting | অপব্যবহার ঠেকানো | |
| Monitoring | মেট্রিক, log, alert | সমস্যা আগে ধরা |
| Load testing | k6, Artillery | আসলে কতটা সামলায় তা জানা |

### Real-time (WebSocket/Socket.io) হলে

১ লক্ষ খোলা সংযোগ:
- Server গুলো stateful (সংযোগ ধরে রাখে)
- User A server 1 এ, user B server 2 এ। বার্তা পাঠাতে **Redis pub/sub adapter** লাগে
- Sticky session

```ts
// Socket.io + Redis adapter
import { createAdapter } from "@socket.io/redis-adapter";
io.adapter(createAdapter(pubClient, subClient));
```

### ডাটাবেসের চাপ কমানোর ক্রম

```
১. query ও index ঠিক করো
২. cache যোগ করো
৩. connection pooling
৪. read replica
৫. ভারী লেখা queue এ
৬. partitioning
৭. sharding (শেষ উপায়)
```

---

## 6. Authentication ও Access/Refresh token ⭐⭐⭐

### সমস্যা

অনেক server থাকলে "কোন server এ লগইন করেছি" মনে রাখা যায় না। দুটো সমাধান:

| | Server-side session | Token (JWT) |
|---|---|---|
| state কোথায় | server/Redis এ | token এ নিজেই |
| যাচাই | Redis এ খোঁজা | signature যাচাই (shared store ছাড়াই) |
| Revoke (বাতিল) | সহজ (session মোছো) | কঠিন (মেয়াদ শেষ হওয়া পর্যন্ত বৈধ) |
| Scaling | shared store লাগে | সহজ |

### Access + Refresh token

```
লগইন ──► সার্ভার দেয়:
           ├─ Access token  (১৫ মিনিট)   ← প্রতি API call এ
           └─ Refresh token (৭-৩০ দিন)    ← শুধু নতুন access token আনতে

API call  ──► Access token সহ
              ✅ বৈধ → উত্তর
              ❌ মেয়াদ শেষ (401)
                   │
                   ▼
              Refresh token পাঠাও ──► নতুন Access (+ নতুন Refresh)
                   │
              ❌ Refresh ও অবৈধ → আবার লগইন
```

| | Access token | Refresh token |
|---|---|---|
| মেয়াদ | ছোট (৫-১৫ মিনিট) | বড় (দিন/সপ্তাহ) |
| কাজ | API অ্যাক্সেস | নতুন access token আনা |
| কোথায় যায় | প্রতিটা request এ | শুধু refresh endpoint এ |
| চুরি হলে | অল্প সময়ের ক্ষতি | বড় ক্ষতি, তাই বেশি সুরক্ষা |
| DB তে | সাধারণত না | **hash করে** রাখো (revoke এর জন্য) |

### সুরক্ষা কৌশল

**১. Refresh token rotation:** প্রতিবার ব্যবহারে নতুন refresh token দাও, পুরনোটা বাতিল।
```
পুরনো refresh token আবার ব্যবহার হলে → চুরি হয়েছে ধরো
→ ওই ব্যবহারকারীর সব token বাতিল করে আবার লগইন করাও
```

**২. Cookie সেটিং:**
```ts
res.cookie("refreshToken", token, {
  httpOnly: true,     // JS পড়তে পারে না (XSS থেকে সুরক্ষা)
  secure: true,       // শুধু HTTPS
  sameSite: "lax",    // CSRF কমায়
  path: "/api/auth/refresh",
  maxAge: 7 * 24 * 60 * 60 * 1000,
});
```

**৩. Access token কোথায়:** মেমোরিতে (React state) রাখা সবচেয়ে নিরাপদ। `localStorage` এ XSS এ চুরি হতে পারে।

**৪. Token এ কী রাখবে:** `sub` (user id), `role`, `exp`। সংবেদনশীল তথ্য না।

**৫. Logout:** refresh token DB থেকে মুছো, cookie মুছো।

**৬. অতিরিক্ত:**
- Rate limit লগইন endpoint এ
- Password hash: bcrypt/argon2
- MFA
- Device/session তালিকা (ব্যবহারকারী সব ডিভাইস থেকে logout করতে পারে)
- SSO (OIDC/SAML) এন্টারপ্রাইজের জন্য

**টিপ:** নিজে বানানোর বদলে প্রমাণিত লাইব্রেরি (Better Auth, Auth.js ইত্যাদি) ব্যবহার করো। ইন্টারভিউয়ে বলো তুমি ধারণাটা বোঝো।

---

## 7. Rate limiting ⭐⭐⭐

**কেন:** অপব্যবহার, bot, brute-force, DDoS, ওভারলোড ঠেকানো, ন্যায্য ব্যবহার।

### কোন key দিয়ে সীমা

| Key | সুবিধা | অসুবিধা |
|---|---|---|
| IP | লগইনের আগেও কাজ করে | এক অফিস/NAT এর সবাই এক IP |
| User ID | ন্যায্য | লগইনের আগে নেই |
| API key | পরিকল্পনা অনুযায়ী (free/pro) | |
| Endpoint | ভারী endpoint আলাদা সীমা | |

### Algorithm

| Algorithm | কীভাবে | সুবিধা | অসুবিধা |
|---|---|---|---|
| **Fixed Window** | প্রতি মিনিটে ১০০ | সহজ | সীমার ধারে দ্বিগুণ burst (১০০ শেষ সেকেন্ডে + ১০০ পরের প্রথম সেকেন্ডে) |
| **Sliding Window** | গত ৬০ সেকেন্ডে কতগুলো | নিখুঁত | বেশি মেমোরি |
| **Token Bucket** | বালতিতে token জমে, request এ খরচ | ছোট burst অনুমতি, জনপ্রিয় | |
| **Leaky Bucket** | নির্দিষ্ট হারে বের হয় | ট্রাফিক মসৃণ | burst সহ্য করে না |

**Token Bucket:** বালতিতে সর্বোচ্চ ১০টা token, প্রতি সেকেন্ডে ১টা করে নতুন। প্রতি request এ ১টা token যায়। token না থাকলে ব্যর্থ।

### Redis দিয়ে (একাধিক server এ শেয়ার)

```ts
// Fixed window, সহজ সংস্করণ
async function rateLimit(key: string, limit: number, windowSec: number) {
  const redisKey = `rl:${key}`;
  const count = await redis.incr(redisKey);          // atomic বাড়ানো
  if (count === 1) await redis.expire(redisKey, windowSec);   // প্রথমবারে মেয়াদ বসাও
  return { allowed: count <= limit, remaining: Math.max(0, limit - count) };
}

// Express middleware
app.use(async (req, res, next) => {
  const { allowed, remaining } = await rateLimit(req.ip, 100, 60);
  res.setHeader("X-RateLimit-Remaining", remaining);
  if (!allowed) {
    res.setHeader("Retry-After", "60");
    return res.status(429).json({ error: "Too many requests" });
  }
  next();
});
```

> কোডে `incr` ও `expire` আলাদা call। নিখুঁত atomic করতে Lua script বা `@upstash/ratelimit`/`rate-limiter-flexible` এর মতো লাইব্রেরি ব্যবহার করো।

### Response

- **HTTP 429 Too Many Requests**
- `Retry-After` header
- `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`

### কোথায় করবে

```
Client → CDN/WAF → API Gateway/LB → App (middleware)
          ↑ সবচেয়ে ভালো: অ্যাপে পৌঁছানোর আগেই আটকানো
```

### বাড়তি অভ্যাস

- সংবেদনশীল endpoint (login, password reset, OTP) এ কঠোর সীমা
- লগইন ব্যর্থতায় ক্রমবর্ধমান বিলম্ব
- Redis ব্যর্থ হলে কী হবে ঠিক করো (fail-open বনাম fail-closed)
- বিভিন্ন plan এ বিভিন্ন সীমা

---

## 8. Message Queue ⭐⭐⭐

```
Producer ──► [ Queue ] ──► Consumer (Worker)
(API)         বার্তা জমে      পরে প্রসেস করে
```

**সুবিধা:**

| সুবিধা | মানে |
|---|---|
| **Decoupling** | producer ও consumer একে অপরকে জানে না |
| **Async** | ধীর কাজ request এর বাইরে, ব্যবহারকারী দ্রুত উত্তর পায় |
| **Buffering** | spike এ কাজ জমে, worker নিজের গতিতে সামলায় |
| **Retry** | ব্যর্থ কাজ আবার চেষ্টা |
| **Scaling** | worker বাড়িয়ে গতি বাড়ানো |

**কখন ব্যবহার:** email/SMS পাঠানো, ছবি/ভিডিও প্রসেসিং, রিপোর্ট তৈরি, payment webhook প্রসেস, সেবার মধ্যে ইভেন্ট।

**উদাহরণ (BullMQ + Redis):**
```ts
import { Queue, Worker } from "bullmq";

const emailQueue = new Queue("emails", { connection: redis });

// API: দ্রুত queue তে দাও, সাথে সাথে উত্তর
app.post("/signup", async (req, res) => {
  const user = await createUser(req.body);
  await emailQueue.add("welcome", { userId: user.id }, { attempts: 3, backoff: { type: "exponential", delay: 1000 } });
  res.status(201).json(user);             // email এর জন্য অপেক্ষা করতে হয়নি
});

// Worker: আলাদা প্রসেসে
new Worker("emails", async (job) => {
  await sendWelcomeEmail(job.data.userId);
}, { connection: redis });
```

### RabbitMQ বনাম Kafka বনাম SQS

| | RabbitMQ | Kafka | SQS |
|---|---|---|---|
| ধরন | Message broker | বিতরিত event log | Managed queue (AWS) |
| বার্তা | consume হলে মুছে যায় | **থেকে যায়** (replay সম্ভব) | consume হলে মুছে যায় |
| Throughput | মাঝারি-ভালো | **অনেক বেশি** | ভালো |
| Routing | নমনীয় (exchange) | topic/partition | সরল |
| সেটআপ | নিজে চালাতে হয় | জটিল | **কিছু চালাতে হয় না** |
| ক্রম | queue এ | partition এ | FIFO queue এ |
| কখন | task queue, জটিল routing | event streaming, অ্যানালিটিক্স, অনেক consumer, replay | AWS এ সহজ queue |

### গুরুত্বপূর্ণ ধারণা

**ডেলিভারি নিশ্চয়তা:**
- At-most-once: হারাতে পারে, duplicate নেই
- **At-least-once** (সাধারণ): হারায় না, **duplicate হতে পারে**
- Exactly-once: কঠিন ও ব্যয়বহুল

**তাই consumer কে Idempotent হতে হবে:** একই বার্তা দুইবার প্রসেস হলেও ফল একই।
```ts
// ❌ দুইবার চললে দুইবার টাকা কাটবে
await chargeCard(orderId);

// ✅ idempotency key দিয়ে
if (await alreadyProcessed(jobId)) return;
await chargeCard(orderId);
await markProcessed(jobId);
```

**Dead Letter Queue (DLQ):** বারবার ব্যর্থ বার্তা আলাদা queue তে, পরে দেখার জন্য। অন্যথায় একটা খারাপ বার্তা পুরো queue আটকে রাখে।

**Backpressure:** consumer ধীর হলে queue বড় হয়। নজর রাখো (queue দৈর্ঘ্য মেট্রিক)।

---

## 9. একটা সেবা নষ্ট হলে ⭐⭐⭐

**মূলনীতি:** distributed সিস্টেমে ব্যর্থতা **স্বাভাবিক**, ব্যতিক্রম না। ডিজাইন করো ব্যর্থতা ধরে নিয়ে।

### কৌশলগুলো

**১. Timeout (সবচেয়ে মৌলিক)**
```ts
const controller = new AbortController();
const id = setTimeout(() => controller.abort(), 3000);     // ৩ সেকেন্ড
try {
  const res = await fetch(url, { signal: controller.signal });
} finally {
  clearTimeout(id);
}
```
Timeout না থাকলে একটা ধীর সেবা সব thread/connection আটকে দেয় (cascading failure)।

**২. Retry + Exponential Backoff + Jitter**
```ts
async function withRetry<T>(fn: () => Promise<T>, max = 3) {
  for (let attempt = 0; attempt < max; attempt++) {
    try {
      return await fn();
    } catch (err) {
      if (attempt === max - 1) throw err;
      const delay = 2 ** attempt * 200 + Math.random() * 100;  // ২০০ms, ৪০০ms, ৮০০ms + random
      await new Promise((r) => setTimeout(r, delay));
    }
  }
}
```
- **Backoff:** প্রতিবার বেশি অপেক্ষা (নষ্ট সেবাকে হাঁপ ছাড়তে দেওয়া)।
- **Jitter:** random জুড়ে দেওয়া, যাতে সবাই একসাথে আবার না আসে।
- **শুধু idempotent কাজে retry:** "টাকা কাটো" retry করলে দুইবার কাটতে পারে।

**৩. Circuit Breaker**

```
  CLOSED ──(অনেক ব্যর্থতা)──► OPEN ──(কিছুক্ষণ পর)──► HALF-OPEN
   স্বাভাবিক                  call বন্ধ,              কয়েকটা পরীক্ষামূলক call
     ▲                       সাথে সাথে ব্যর্থ              │
     └────────(সফল)──────────────────────────────────────┘
                              (ব্যর্থ হলে আবার OPEN)
```
নষ্ট সেবায় বারবার call না করে সাথে সাথে ব্যর্থ করে, সেবাকে সেরে উঠতে সময় দেয়। লাইব্রেরি: `opossum` (Node.js)।

**৪. Fallback / Graceful degradation**

| সেবা নষ্ট | বিকল্প |
|---|---|
| Recommendation | জনপ্রিয় পণ্যের স্থির তালিকা |
| Review | পুরনো cache করা review |
| Search | সরল DB query |
| Payment | order "pending" রেখে পরে নিশ্চিত করা |

**৫. Queue দিয়ে বাফার:** জরুরি না এমন কাজ queue এ রাখো, সেবা ফিরলে প্রসেস হবে।

**৬. Bulkhead:** একটা সেবার জন্য আলাদা connection pool, যাতে একটা ডুবলে বাকিরা না ডোবে (জাহাজের আলাদা কক্ষের মতো)।

**৭. Redundancy ও health check:** একাধিক instance, বিভিন্ন availability zone, LB নষ্ট instance বাদ দেয়।

**৮. Monitoring ও alert:** error rate, latency (p95/p99), সেবার স্বাস্থ্য, Sentry, Prometheus/Grafana।

**৯. Idempotency:** retry নিরাপদ করতে।

**ইন্টারভিউয়ে বলার ক্রম:** "প্রথমে timeout, তারপর retry with backoff, circuit breaker, fallback, আর সবকিছুর জন্য monitoring।"

---

# Tier 2: প্রায়ই আসে

## 10. System design কী ও Functional বনাম Non-functional ⭐⭐

**System design:** একটা সিস্টেমের অংশগুলো (সেবা, ডাটাবেস, cache, queue, নেটওয়ার্ক) কীভাবে সাজানো ও জোড়া হবে তার পরিকল্পনা, যাতে চাহিদা পূরণ হয়।

**কেন গুরুত্বপূর্ণ:**
- Scalability (বাড়তি ব্যবহারকারী সামলানো)
- Reliability (ব্যর্থতায় টিকে থাকা)
- Maintainability (পরিবর্তন সহজ)
- খরচ নিয়ন্ত্রণ
- ডিজাইনের ভুল পরে ঠিক করা অনেক ব্যয়বহুল

### Functional বনাম Non-functional

| | Functional | Non-functional |
|---|---|---|
| প্রশ্ন | সিস্টেম **কী** করে? | সিস্টেম **কতটা ভালো** করে? |
| উদাহরণ | ব্যবহারকারী order দিতে পারে, পণ্য খুঁজতে পারে | ২০০ms এর মধ্যে উত্তর, ৯৯.৯% uptime |

**Non-functional এর তালিকা:**

| বৈশিষ্ট্য | মানে |
|---|---|
| Performance | latency, throughput |
| Scalability | বাড়তি লোড সামলানো |
| Availability | কতক্ষণ চালু (99.9% = বছরে ≈ ৮.৭৬ ঘণ্টা বন্ধ) |
| Reliability | ভুল ছাড়া কাজ করা |
| Consistency | সবাই একই ডেটা দেখে |
| Security | অননুমোদিত প্রবেশ ঠেকানো |
| Durability | ডেটা হারায় না |
| Maintainability | বদলানো সহজ |
| Cost | বাজেট |

**মূল কথা:** দুটো সিস্টেমের functional চাহিদা একই হতে পারে, কিন্তু non-functional আলাদা হলে ডিজাইন সম্পূর্ণ আলাদা হবে। (১০০ জনের অ্যাপ বনাম ১ কোটির অ্যাপ।)

### CAP theorem (প্রায়ই জিজ্ঞেস করে)

নেটওয়ার্ক বিভাজন (partition) হলে **Consistency** ও **Availability** এর একটা বেছে নিতে হয়:
- **CP:** ডেটা সঠিক, কিন্তু কিছু অনুরোধ ব্যর্থ হতে পারে (ব্যাংক)
- **AP:** সবসময় উত্তর, কিন্তু ডেটা সামান্য পুরনো হতে পারে (সামাজিক ফিড)

---

## 11. মৌলিক প্রবাহ: Client → API → Business Logic → Database ⭐⭐

```
┌────────┐   HTTP    ┌───────────┐    ┌──────────────┐    ┌──────────────┐    ┌────┐
│ Client │──────────►│ API Layer │───►│Business Logic│───►│  Data Layer  │───►│ DB │
│browser │◄──────────│(controller)│◄──│  (service)   │◄───│ (repository) │◄───│    │
└────────┘  response └───────────┘    └──────────────┘    └──────────────┘    └────┘
```

| স্তর | দায়িত্ব | উদাহরণ |
|---|---|---|
| **Client** | UI, ব্যবহারকারীর ইনপুট | React/Next.js |
| **API** | routing, authentication, input validation, response আকার | Express route, Next.js Route Handler |
| **Business logic** | অ্যাপের নিয়ম | stock আছে কিনা, ছাড় হিসাব, order এর অবস্থা |
| **Data** | ডাটাবেসের সাথে কথা | Prisma query |
| **Database** | ডেটা সংরক্ষণ | PostgreSQL |

**কোড উদাহরণ:**
```ts
// ১. API layer: গ্রহণ, যাচাই
app.post("/orders", authenticate, async (req, res) => {
  const input = orderSchema.parse(req.body);              // validation
  const order = await orderService.placeOrder(req.user.id, input);
  res.status(201).json(order);
});

// ২. Business logic
export const orderService = {
  async placeOrder(userId: string, input: OrderInput) {
    const total = await calculateTotal(input.items);      // নিয়ম
    return orderRepository.create({ userId, total, items: input.items });
  },
};

// ৩. Data layer
export const orderRepository = {
  create: (data: CreateOrder) => prisma.order.create({ data }),
};
```

**কেন স্তর আলাদা:**
- টেস্ট সহজ (business logic ডাটাবেস ছাড়াই টেস্ট)
- ডাটাবেস বদলানো সহজ
- কোড পুনর্ব্যবহার (একই service REST ও GraphQL দুটোতেই)
- দায়িত্ব পরিষ্কার

**বাস্তবে মাঝখানে যোগ হয়:** CDN, load balancer, API gateway, cache, queue, logging, monitoring।

---

## 12. Stateless বনাম Stateful ⭐⭐

```
STATEFUL (সমস্যা)                    STATELESS (সমাধান)
                                     
User A ──► Server 1 (session মনে)     User A ──► যেকোনো server ──► Redis (session)
User A ──► Server 2 ❌ "কে তুমি?"     User A ──► যেকোনো server ──► Redis ✅
```

| | Stateless | Stateful |
|---|---|---|
| Server কী মনে রাখে | কিছুই না | client এর ডেটা |
| যেকোনো server যেকোনো request সামলায় | ✅ | ❌ |
| Scale | সহজ | কঠিন (sticky session) |
| ব্যর্থতায় পুনরুদ্ধার | সহজ | ডেটা হারাতে পারে |
| Load balancing | সরল | জটিল |

### Server কে stateless রাখার উপায়

| state | কোথায় সরাবে |
|---|---|
| Session | Redis / DB |
| Auth | JWT token (client বহন করে) |
| আপলোড করা ফাইল | S3 / object storage (server এর disk এ না) |
| Cache | Redis |
| Job/কাজ | Queue |

### স্বভাবে stateful যা

- **WebSocket সংযোগ:** সংযোগ নির্দিষ্ট server এ। সমাধান: sticky session + Redis pub/sub
- **ডাটাবেস:** ডেটাই তার কাজ। সমাধান: replication/sharding
- **গেম server**

**মনে রাখো:** state একেবারে মুছে ফেলা যায় না, **সরানো যায়** এমন জায়গায় যা আলাদাভাবে scale হয় (DB, Redis)।

---

## 13. Database replication ⭐⭐

```
                ┌──────────────┐
   লেখা ───────►│   Primary    │
                └──────┬───────┘
              (ডেটা কপি হয়)
          ┌────────────┼────────────┐
     ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
     │Replica 1│  │Replica 2│  │Replica 3│ ◄─── পড়া
     └─────────┘  └─────────┘  └─────────┘
```

**সুবিধা:**
1. **Read scaling:** পড়ার চাপ replica তে ভাগ
2. **High availability:** primary নষ্ট হলে replica কে primary বানানো (failover)
3. **Backup:** replica থেকে নিলে primary তে চাপ পড়ে না
4. **ভৌগোলিক কাছাকাছি:** ব্যবহারকারীর কাছের replica

### Synchronous বনাম Asynchronous

| | Synchronous | Asynchronous |
|---|---|---|
| লেখা সফল কখন | replica তেও পৌঁছালে | primary তে হলেই |
| ডেটা হারানোর ঝুঁকি | নেই | primary ধ্বংস হলে কিছু হারাতে পারে |
| গতি | ধীর | দ্রুত |
| Lag | নেই | আছে |

বেশিরভাগ read replica **asynchronous**।

### Replication lag সমস্যা

```
t0: ব্যবহারকারী প্রোফাইলের নাম বদলাল → Primary এ লেখা হলো
t1: পেজ reload → Replica থেকে পড়ল → এখনো পুরনো নাম! 😕
```

**সমাধান:**
- নিজের লেখার পর কিছুক্ষণ primary থেকে পড়া (read-your-writes)
- জরুরি ডেটা (টাকা, stock) সবসময় primary থেকে
- Replica lag মেট্রিক নজরে রাখো

### Prisma তে read replica

আলাদা client বা `@prisma/extension-read-replicas` এক্সটেনশন দিয়ে পড়া replica তে, লেখা primary তে পাঠানো যায়।

**মনে রাখো:** replication শুধু **পড়া** scale করে। লেখা scale করতে sharding।

---

## 14. Database sharding ⭐⭐

```
Sharding ছাড়া:                  Sharding সহ (user ID দিয়ে):
┌──────────────┐                 ┌────────┐ ┌────────┐ ┌────────┐
│  সব ডেটা     │                 │Shard 1 │ │Shard 2 │ │Shard 3 │
│ (১ কোটি user)│                 │ A-H    │ │ I-P    │ │ Q-Z    │
└──────────────┘                 └────────┘ └────────┘ └────────┘
                                 প্রতিটায় ডেটার অংশ, আলাদা server
```

**কখন দরকার:** একটা server ডেটার আকার বা **লেখার চাপ** সামলাতে পারছে না (replication তে লেখা scale হয় না)।

### Shard key ও কৌশল

| কৌশল | কীভাবে | সুবিধা | অসুবিধা |
|---|---|---|---|
| **Range-based** | `userId` 1-1M → shard 1 | সহজ, range query ভালো | অসম বণ্টন, hot shard |
| **Hash-based** | `hash(userId) % N` | সমান বণ্টন | range query কঠিন, N বদলালে সব সরাতে হয় |
| **Directory-based** | lookup টেবিল বলে কোথায় | নমনীয় | lookup একটা single point |
| **Geo-based** | অঞ্চল অনুযায়ী | কাছাকাছি, আইন মানা | অসম বণ্টন |

**ভালো shard key:** উচ্চ cardinality (অনেক ভিন্ন মান), সমান বণ্টন, বেশিরভাগ query তে ব্যবহৃত।

### খরচ ও সমস্যা

| সমস্যা | কেন |
|---|---|
| Cross-shard query/join | একাধিক shard এ ঘুরতে হয়, ধীর ও জটিল |
| Cross-shard transaction | খুব জটিল (two-phase commit) |
| Resharding | shard সংখ্যা বাড়ানো কষ্টকর, ডেটা সরাতে হয় |
| Hot shard | খারাপ key এ একটা shard এ সব চাপ (যেমন সেলিব্রিটি user) |
| অপারেশনাল জটিলতা | backup, migration, monitoring সব shard এ |

### আগে এগুলো চেষ্টা করো

```
১. query/index ঠিক  →  ২. cache  →  ৩. read replica  →  ৪. vertical scale
→  ৫. table partitioning (একই server এ)  →  ৬. sharding (শেষ উপায়)
```

**Partitioning বনাম Sharding:** Partitioning একই ডাটাবেসে টেবিলকে ভাগ করা (যেমন তারিখ অনুযায়ী)। Sharding ভাগগুলো আলাদা server এ রাখা।

**সরঞ্জাম:** Citus (PostgreSQL এর জন্য), Vitess (MySQL), MongoDB/Cassandra/DynamoDB এ built-in।

---

## 15. SQL বনাম NoSQL (system design দৃষ্টিকোণ) ⭐⭐

| বিষয় | SQL (PostgreSQL) | NoSQL |
|---|---|---|
| Schema | কঠোর | নমনীয় |
| সম্পর্ক/Join | ✅ শক্তিশালী | সীমিত/নেই |
| Transaction | পূর্ণ ACID | সীমিত (ডাটাবেস অনুযায়ী) |
| Consistency | শক্ত | প্রায়ই eventual (সেটিং অনুযায়ী) |
| Scaling | vertical + read replica; sharding কঠিন | **horizontal সহজ** |
| Query | নমনীয় (SQL), যেকোনো ধরনের | access pattern আগে ঠিক করতে হয় |
| ভালো কাজ | order, payment, inventory, রিপোর্ট | feed, log, session, catalog, IoT, cache |

### NoSQL এর ধরন

| ধরন | উদাহরণ | কখন |
|---|---|---|
| Document | MongoDB | নমনীয় object |
| Key-Value | Redis, DynamoDB | cache, session, দ্রুত lookup |
| Wide-column | Cassandra | অনেক লেখা, time-series |
| Graph | Neo4j | সম্পর্কের জাল (সামাজিক নেটওয়ার্ক) |

### সিদ্ধান্তের প্রশ্ন

1. ডেটা কি সম্পর্কযুক্ত ও কাঠামোবদ্ধ? → SQL
2. শক্ত consistency লাগে (টাকা)? → SQL
3. অসংখ্য লেখা ও সহজ horizontal scaling? → NoSQL
4. Access pattern আগে থেকে জানা ও সরল? → NoSQL ভালো
5. জটিল রিপোর্ট ও ad-hoc query? → SQL

### Polyglot persistence

একই সিস্টেমে একাধিক ডাটাবেস, প্রতিটা তার কাজে সেরা:

```
PostgreSQL  → order, payment, user (সত্যের উৎস)
Redis       → cache, session, rate limit
Elasticsearch/Meilisearch → full-text search
S3          → ফাইল, ছবি
```

**সতর্কতা:** বেশি ডাটাবেস মানে বেশি জটিলতা। প্রয়োজন ছাড়া বাড়িও না। অনেক সিস্টেমে শুধু PostgreSQL + Redis যথেষ্ট।

---

## 16. Database index ও কীভাবে performance বাড়ায় ⭐⭐

সংক্ষেপে (বিস্তারিত `database-details.md` এর টপিক ৭):

```
Index ছাড়া:   ১ কোটি row এর প্রতিটা দেখো         → ধীর (O(n))
Index সহ:      B-tree এ কয়েক ধাপে সরাসরি পৌঁছাও   → দ্রুত (O(log n))
```

**ট্রেড-অফ:**

| সুবিধা | খরচ |
|---|---|
| `WHERE`, `JOIN`, `ORDER BY` দ্রুত | জায়গা লাগে |
| | `INSERT/UPDATE/DELETE` ধীর (index ও আপডেট) |

**নিয়ম:**
- আসল query দেখে index দাও
- foreign key, ঘনঘন filter/sort এর column এ
- composite index এ equality আগে, range পরে
- `EXPLAIN ANALYZE` দিয়ে ব্যবহার হচ্ছে কিনা নিশ্চিত করো
- অব্যবহৃত index বাদ দাও

**System design এ index এর ভূমিকা:** স্কেলের প্রথম ধাপ। সার্ভার বাড়ানোর আগে index ঠিক করো, কারণ এটা সবচেয়ে সস্তা ও কার্যকর।
