# Database Details (Prisma + PostgreSQL + Cross-Database)

প্রতিটা টপিক সহজ বাংলায়, কোড উদাহরণসহ। ইন্টারভিউয়ের ছোট ইংরেজি উত্তরের জন্য `database-interview.md` দেখো।

**ক্রম:** Tier 1 (সবচেয়ে বেশি আসে) → Tier 2 → Tier 3। নম্বর `database-interview.md` এর সাথে মেলানো।

> **MongoDB/Mongoose সেকশন এখানে নেই** (তোমার অনুরোধে)। শুধু PostgreSQL বনাম MongoDB তুলনা আছে।

---

## আগে জেনে নাও: Prisma 7 এ কী বদলেছে

Prisma 7 এ client setup আগের (v6) থেকে আলাদা। ইন্টারভিউয়ের প্রশ্ন ও ধারণা একই, কিন্তু নতুন প্রজেক্টে এই বদল জানা ভালো।

| বিষয় | Prisma 6 | Prisma 7 |
|---|---|---|
| Generator | `provider = "prisma-client-js"` | `provider = "prisma-client"` |
| Output path | ঐচ্ছিক (node_modules এ) | **আবশ্যক** (যেমন `../generated/prisma`) |
| Driver adapter | ঐচ্ছিক | **SQL ডাটাবেসে আবশ্যক** (PostgreSQL এর জন্য `@prisma/adapter-pg`) |
| কনফিগ | `.env` + schema | `prisma.config.ts` |
| `.env` লোড | অটো | নিজে (`dotenv`) |
| `$use()` middleware | আছে | সরানো হয়েছে → Client Extensions |
| `Prisma.validator()` | আছে | `satisfies` দিয়ে প্রতিস্থাপন |

```bash
pnpm add @prisma/client @prisma/adapter-pg pg dotenv
pnpm add -D prisma
```

```prisma
// prisma/schema.prisma (v7)
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}
```

```ts
// prisma.config.ts
import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  datasource: {
    url: env("DATABASE_URL"),
  },
});
```

```ts
// lib/prisma.ts (v7)
import { PrismaClient } from "../generated/prisma/client";
import { PrismaPg } from "@prisma/adapter-pg";

const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL });
export const prisma = new PrismaClient({ adapter });
```

**খেয়াল রাখো:** `prisma.config.ts` এর ঠিক গঠন (বিশেষত `datasource` অংশ) সংস্করণ অনুযায়ী একটু আলাদা হতে পারে। তোমার প্রজেক্টের সংস্করণের [Prisma upgrade guide](https://www.prisma.io/docs/orm/more/upgrade-guides/upgrading-to-prisma-7) দেখে নিও। তোমার প্রজেক্ট v6 এ থাকলে নিচের সব query ও schema কোড একই কাজ করবে, শুধু setup অংশ আলাদা।

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. Prisma বনাম PostgreSQL ⭐⭐⭐

| | PostgreSQL | Prisma |
|---|---|---|
| কী | **ডাটাবেস** (ডেটা রাখে) | **ORM** (কোড থেকে ডাটাবেসের সাথে কথা বলে) |
| কোথায় চলে | আলাদা server এ | তোমার অ্যাপের ভেতরে |
| ভাষা | SQL | TypeScript |
| কাজ | ডেটা সংরক্ষণ, query চালানো | query লিখে দেওয়া, type দেওয়া, migration |

**উপমা:** PostgreSQL হলো গুদাম, Prisma হলো গুদামের কর্মচারী যাকে তুমি ভাষায় (TypeScript) অনুরোধ করো, সে গুদামের ভাষায় (SQL) বুঝিয়ে আনে।

**Prisma কোন সমস্যা সমাধান করে:**

1. **Type safety:** ভুল ফিল্ড লিখলে কোড লেখার সময়ই error।
```ts
const user = await prisma.user.findUnique({ where: { id } });
user.nmae;   // ❌ TypeScript সাথে সাথে ধরে
```
2. **এক জায়গায় schema:** `schema.prisma` ফাইলে সব টেবিল পরিষ্কারভাবে।
3. **Migration:** schema বদলকে SQL ফাইলে নথিভুক্ত।
4. **Auto-complete:** কী কী ফিল্ড ও relation আছে editor দেখায়।
5. **SQL injection থেকে নিরাপত্তা:** সাধারণ query তে parameter নিজে নিজে নিরাপদ।

**Prisma তুলনায় যেখানে সীমা:** খুব জটিল SQL এ `$queryRaw` লাগে।

---

## 2. Relations: one-to-one, one-to-many, many-to-many ⭐⭐⭐

### One-to-One (User ↔ Profile)

```prisma
model User {
  id      String   @id @default(cuid())
  email   String   @unique
  profile Profile?          // ঐচ্ছিক: সবার profile নাও থাকতে পারে
}

model Profile {
  id     String @id @default(cuid())
  bio    String?
  userId String @unique     // ← @unique-ই one-to-one বানায়
  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```
`userId` এ `@unique` না দিলে এটা one-to-many হয়ে যায়।

### One-to-Many (User → Orders)

```prisma
model User {
  id     String  @id @default(cuid())
  orders Order[]            // "অনেক" দিক: array
}

model Order {
  id     String @id @default(cuid())
  userId String             // foreign key "অনেক" দিকে
  user   User   @relation(fields: [userId], references: [id])

  @@index([userId])         // foreign key তে index দাও
}
```

### Many-to-Many: implicit (Prisma নিজে join table বানায়)

```prisma
model Post {
  id   String @id @default(cuid())
  tags Tag[]
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  posts Post[]
}
```
```ts
await prisma.post.create({
  data: {
    title: "Hello",
    tags: { connect: [{ id: "tag1" }, { id: "tag2" }] },
  },
});
```

### Many-to-Many: explicit (বাড়তি ফিল্ড লাগলে)

```prisma
model Student {
  id          String       @id @default(cuid())
  enrollments Enrollment[]
}

model Course {
  id          String       @id @default(cuid())
  enrollments Enrollment[]
}

model Enrollment {
  studentId  String
  courseId   String
  enrolledAt DateTime @default(now())    // বাড়তি ফিল্ড
  grade      String?
  student    Student  @relation(fields: [studentId], references: [id])
  course     Course   @relation(fields: [courseId], references: [id])

  @@id([studentId, courseId])            // composite primary key
}
```

**কখন কোনটা:** শুধু সংযোগ দরকার → implicit। সংযোগের সাথে তথ্য (তারিখ, quantity, role) লাগে → explicit।

---

## 3. User → Orders → OrderItems → Products মডেল ⭐⭐⭐

```
User ──< Order ──< OrderItem >── Product
  1:N       1:N          N:1
```

```prisma
enum OrderStatus {
  PENDING
  PAID
  SHIPPED
  CANCELLED
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
  orders    Order[]
}

model Product {
  id         String      @id @default(cuid())
  name       String
  price      Decimal     @db.Decimal(10, 2)
  stock      Int         @default(0)
  orderItems OrderItem[]
}

model Order {
  id        String      @id @default(cuid())
  userId    String
  status    OrderStatus @default(PENDING)
  total     Decimal     @db.Decimal(10, 2)
  createdAt DateTime    @default(now())
  user      User        @relation(fields: [userId], references: [id], onDelete: Restrict)
  items     OrderItem[]

  @@index([userId, status, createdAt])
}

model OrderItem {
  id        String  @id @default(cuid())
  orderId   String
  productId String
  quantity  Int
  unitPrice Decimal @db.Decimal(10, 2)          // দামের snapshot
  order     Order   @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product   Product @relation(fields: [productId], references: [id], onDelete: Restrict)

  @@unique([orderId, productId])                // একই order এ একই পণ্য দুইবার নয়
  @@index([productId])
}
```

**গুরুত্বপূর্ণ সিদ্ধান্তগুলো (ইন্টারভিউয়ে বলার মতো):**

| সিদ্ধান্ত | কেন |
|---|---|
| `unitPrice` OrderItem এ | Product এর দাম পরে বদলালে পুরনো order এর দাম বদলাবে না |
| `Decimal` টাকার জন্য | `Float` এ হিসাবের ভুল হয় (0.1 + 0.2) |
| `onDelete: Cascade` Order→OrderItem | Order মুছলে তার item ও যাবে |
| `onDelete: Restrict` User/Product | order আছে এমন user বা পণ্য মোছা যাবে না |
| `@@unique([orderId, productId])` | duplicate item আটকায় |
| Foreign key তে `@@index` | join ও lookup দ্রুত |
| `enum` status | ভুল মান ঢোকা আটকায় |

**Query:**
```ts
const order = await prisma.order.create({
  data: {
    userId,
    total,
    items: {
      create: [
        { productId: "p1", quantity: 2, unitPrice: 100 },
        { productId: "p2", quantity: 1, unitPrice: 250 },
      ],
    },
  },
  include: { items: { include: { product: true } } },
});
```

---

## 4. Transaction ও ACID ⭐⭐⭐

**Transaction:** কয়েকটা কাজকে এক প্যাকেজে চালানো। সব সফল হলে **commit**, একটাও ব্যর্থ হলে সব **rollback**।

**উদাহরণ:** ব্যাংকে ১০০ টাকা পাঠানো = (১) A থেকে কাটা (২) B তে যোগ। (১) হয়ে (২) ব্যর্থ হলে টাকা হারিয়ে যাবে। Transaction এ দুটোই হবে, নয়তো কোনোটাই না।

### ACID

| অক্ষর | মানে | সহজ ভাষায় |
|---|---|---|
| **A**tomicity | পুরোটা বা কিছুই না | মাঝপথে থেমে গেলে সব আগের অবস্থায় |
| **C**onsistency | নিয়ম সবসময় বজায় | constraint ভাঙা অবস্থা তৈরি হয় না |
| **I**solation | একসাথে চলা transaction একে অপরকে বিরক্ত করে না | প্রত্যেকে যেন একা চলছে |
| **D**urability | commit এর পর ডেটা টেকে | crash হলেও হারায় না |

### Prisma তে দুই ধরনের transaction

**১. Sequential (array):** স্বাধীন কাজ, একটার ফলাফল অন্যটায় লাগে না।
```ts
const [user, post] = await prisma.$transaction([
  prisma.user.create({ data: { email: "a@b.com", name: "A" } }),
  prisma.post.create({ data: { title: "Hi", authorId: "x" } }),
]);
```

**২. Interactive (callback):** মাঝখানে লজিক বা শর্ত লাগলে।
```ts
await prisma.$transaction(async (tx) => {
  // ⚠️ ভেতরে prisma নয়, tx ব্যবহার করো
  const product = await tx.product.findUnique({ where: { id } });
  if (!product || product.stock < qty) throw new Error("স্টক নেই"); // error = rollback

  await tx.product.update({ where: { id }, data: { stock: { decrement: qty } } });
  return tx.order.create({ data: { /* ... */ } });
}, {
  maxWait: 5000,      // transaction শুরুর জন্য সর্বোচ্চ অপেক্ষা (ms)
  timeout: 10000,     // transaction এর সর্বোচ্চ সময় (ms)
  isolationLevel: Prisma.TransactionIsolationLevel.Serializable, // ঐচ্ছিক
});
```

**সতর্কতা:**
- Callback এর ভেতরে `prisma` নয়, `tx` ব্যবহার না করলে ওই কাজ transaction এর বাইরে চলে।
- Transaction ছোট ও দ্রুত রাখো। ভেতরে বাইরের API call (email, payment gateway) করো না, কারণ এতে lock অনেকক্ষণ ধরে থাকে।
- Nested write (`create` এর ভেতরে `items: { create }`) নিজে থেকেই transaction এ চলে।

### Isolation level (সংক্ষেপে)

| Level | মানে |
|---|---|
| Read Committed (PostgreSQL ডিফল্ট) | শুধু commit হওয়া ডেটা দেখে |
| Repeatable Read | transaction চলাকালীন একই ডেটা একই থাকে |
| Serializable | একটার পর একটা চললে যে ফল হতো, সেই ফল। সবচেয়ে নিরাপদ, ধীর, conflict এ retry লাগে |

---

## 5. টাকা কাটা + order তৈরি: একসাথে সফল বা ব্যর্থ ⭐⭐⭐

```ts
import { Prisma } from "../generated/prisma/client"; // v6 এ "@prisma/client"

export async function placeOrder(userId: string, amount: Prisma.Decimal, items: Item[]) {
  return prisma.$transaction(async (tx) => {
    // ১. শর্তসহ atomic কাটা: "balance >= amount হলে তবেই"
    const result = await tx.account.updateMany({
      where: { userId, balance: { gte: amount } },
      data: { balance: { decrement: amount } },
    });

    // কোনো row আপডেট না হলে = টাকা যথেষ্ট না
    if (result.count === 0) {
      throw new Error("INSUFFICIENT_BALANCE");   // → পুরো transaction rollback
    }

    // ২. একই transaction এ order তৈরি
    const order = await tx.order.create({
      data: {
        userId,
        total: amount,
        status: "PAID",
        items: { create: items },
      },
    });

    // ৩. ঐচ্ছিক: লেনদেনের রেকর্ড (audit)
    await tx.ledger.create({
      data: { userId, orderId: order.id, amount: amount.neg(), type: "ORDER_PAYMENT" },
    });

    return order;
  });
}
```

**কেন এটা নিরাপদ:**

| ঝুঁকি | কীভাবে ঠেকানো হয়েছে |
|---|---|
| টাকা কেটে order হলো না | একই transaction, error হলে rollback |
| দুই request এ একই টাকা দুইবার খরচ | `where: { balance: { gte: amount } }` সহ একটাই atomic update, row lock হয় |
| ঋণাত্মক balance | শর্ত + (ভালো হলে) DB `CHECK (balance >= 0)` constraint |
| একই request দুইবার আসা (retry/double click) | **idempotency key**: ইউনিক key দিয়ে duplicate order আটকানো |

**ভুল উদাহরণ (এটা করো না):**
```ts
// ❌ চেক ও আপডেট আলাদা: দুই request একসাথে এলে দুটোই পাস করবে
const account = await tx.account.findUnique({ where: { userId } });
if (account.balance < amount) throw new Error("...");
await tx.account.update({ where: { userId }, data: { balance: account.balance - amount } });
```

**বাইরের payment gateway এর ক্ষেত্রে:** ডাটাবেস transaction বাইরের সিস্টেমকে rollback করতে পারে না। তাই order আগে `PENDING` রেখে, payment সফল হলে `PAID` করা, আর ব্যর্থ হলে `CANCELLED`। Stripe webhook দিয়ে নিশ্চিত করা।

---

## 6. N+1 সমস্যা ⭐⭐⭐

**সমস্যা:**
```ts
// ❌ N+1: ১টা query users এর জন্য + প্রতিটা user এর জন্য ১টা করে
const users = await prisma.user.findMany();           // ১টা query
for (const user of users) {
  const orders = await prisma.order.findMany({        // N টা query
    where: { userId: user.id },
  });
  console.log(user.name, orders.length);
}
// ১০০ জন user = ১০১টা query!
```

**সমাধান ১: `include`**
```ts
// ✅ সম্পর্কিত ডেটা একসাথে
const users = await prisma.user.findMany({
  include: { orders: true },
});
```
Prisma ভেতরে সাধারণত কয়েকটা batched query চালায় (`WHERE userId IN (...)`), প্রতি user এ একটা না। সংস্করণ অনুযায়ী `relationLoadStrategy: "join"` দিয়ে একটা JOIN query ও চালানো যায়।

**সমাধান ২: `select` (আরও ভালো, শুধু দরকারি ফিল্ড)**
```ts
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    orders: { select: { id: true, total: true } },
  },
});
```

**সমাধান ৩: `in` দিয়ে এক query তে সব আনা**
```ts
const users = await prisma.user.findMany();
const ids = users.map((u) => u.id);

const orders = await prisma.order.findMany({ where: { userId: { in: ids } } });

// মেমোরিতে group করো
const byUser = Map.groupBy(orders, (o) => o.userId);
```

**সমাধান ৪: শুধু গণনা দরকার হলে `_count`**
```ts
const users = await prisma.user.findMany({
  select: { name: true, _count: { select: { orders: true } } },
});
```

**GraphQL এ N+1 (তুমি GraphQL জানো, তাই ইন্টারভিউয়ে বললে ভালো):** প্রতিটা resolver আলাদা query চালালে N+1 হয়। সমাধান: **DataLoader** দিয়ে batching। Prisma নিজেও একই tick এর `findUnique` call গুলো স্বয়ংক্রিয়ভাবে batch করে।

**ধরবে কীভাবে:**
```ts
const prisma = new PrismaClient({ adapter, log: ["query"] });
// console এ প্রতিটা SQL দেখো, একই ধাঁচের query বারবার দেখলে N+1
```

---

## 7. Index ⭐⭐⭐

**Index ছাড়া:** পুরো টেবিলের প্রতিটা row দেখতে হয় (**Sequential Scan**, O(n))।
**Index সহ:** বইয়ের সূচিপত্রের মতো সরাসরি জায়গায় যায় (**Index Scan**, O(log n))।

PostgreSQL এর ডিফল্ট index হলো **B-tree**।

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique                // unique index অটো
  name      String
  city      String
  createdAt DateTime @default(now())

  @@index([city])                           // সাধারণ index
  @@index([city, createdAt(sort: Desc)])    // composite index
}
```

### কখন index দেবে

| দাও | দিও না |
|---|---|
| `WHERE` এ ঘনঘন ব্যবহৃত column | ছোট টেবিল (কয়েকশ row) |
| `JOIN` এর foreign key | খুব কম ভিন্ন মান (যেমন `isActive` true/false) |
| `ORDER BY` এর column | ঘনঘন লেখা হয়, কম পড়া হয় এমন টেবিল |
| `UNIQUE` নিশ্চিত করতে | কখনো filter করা হয় না এমন column |

### ধরন

| Index | কখন |
|---|---|
| B-tree (ডিফল্ট) | `=`, `<`, `>`, `BETWEEN`, `ORDER BY` |
| Composite (একাধিক column) | একসাথে একাধিক column এ filter |
| Partial (শর্তসহ) | শুধু কিছু row, যেমন `WHERE deletedAt IS NULL` |
| GIN | JSONB, array, full-text search |
| GIN + `pg_trgm` | `LIKE '%text%'` আংশিক খোঁজ |
| Unique | duplicate আটকানো |

### অনেক index ক্ষতিকর কেন

- প্রতিটা `INSERT`, `UPDATE`, `DELETE` এ **সব index আপডেট** করতে হয়, লেখা ধীর হয়।
- **জায়গা** নেয়।
- Query planner কে বেছে নিতে হয়, কখনো ভুল বেছে নেয়।
- নিয়ম: **আসল query দেখে** index দাও, আন্দাজে না। অব্যবহৃত index খুঁজে বাদ দাও (`pg_stat_user_indexes`)।

### Production এ index বানানো

সাধারণ `CREATE INDEX` টেবিল lock করে। বড় টেবিলে:
```sql
CREATE INDEX CONCURRENTLY idx_orders_user ON "Order" ("userId");
```
Prisma migration ফাইল `--create-only` দিয়ে বানিয়ে এই SQL হাতে বসাও (`CONCURRENTLY` transaction এর ভেতরে চলে না, তাই আলাদা migration এ রাখো)।

---

## 8. Primary key, Unique, Foreign key, Index ⭐⭐⭐

| বিষয় | কাজ | Null? | কয়টা থাকতে পারে | Index অটো? |
|---|---|---|---|---|
| **Primary Key** | row কে অনন্যভাবে চেনায় | না | ১টা | হ্যাঁ |
| **Unique** | duplicate আটকায় | হ্যাঁ (একাধিক null চলে) | অনেক | হ্যাঁ |
| **Foreign Key** | অন্য টেবিলের সাথে সম্পর্ক ঠিক রাখে | হ্যাঁ (ঐচ্ছিক হলে) | অনেক | **না** (নিজে দিতে হয়) |
| **Index** | search দ্রুত করে | — | অনেক | — |

```prisma
model Order {
  id     String @id @default(cuid())     // Primary Key
  code   String @unique                  // Unique constraint
  userId String
  user   User   @relation(fields: [userId], references: [id])  // Foreign Key

  @@index([userId])                      // Index (foreign key তে নিজে দাও!)
}
```

**মনে রাখার ট্রিক:**
- Primary key = **পরিচয়পত্র নম্বর** (প্রত্যেকের একটাই)
- Unique = **ইমেইল** (কারো সাথে মিলবে না, কিন্তু পরিচয়পত্র না)
- Foreign key = **"এই order কার?"** (বাস্তব user এর দিকে ইঙ্গিত)
- Index = **সূচিপত্র** (শুধু খুঁজতে সাহায্য, নিয়ম চাপায় না)

**গুরুত্বপূর্ণ ফাঁদ:** PostgreSQL foreign key তে **অটো index বানায় না**। Prisma ও না বানালে join ধীর হয়, তাই `@@index([userId])` নিজে দাও।

---

## 9. `include` বনাম `select` ⭐⭐⭐

```ts
// include: মডেলের সব ফিল্ড + relation
const user = await prisma.user.findUnique({
  where: { id },
  include: { orders: true },
});
// ফল: { id, email, name, password, createdAt, ..., orders: [...] }
//     ↑ password ও চলে এলো!

// select: শুধু যা বাছলে
const user = await prisma.user.findUnique({
  where: { id },
  select: {
    id: true,
    name: true,
    orders: { select: { id: true, total: true } },
  },
});
// ফল: { id, name, orders: [{ id, total }] }
```

| | `include` | `select` |
|---|---|---|
| ফিল্ড | সব + relation | শুধু বাছাই করা |
| response আকার | বড় | ছোট |
| নিরাপত্তা | `password` ইত্যাদি বেরিয়ে যেতে পারে | নিরাপদ |
| লেখা | সহজ | একটু বেশি |

**নিয়ম:**
- একই স্তরে `include` ও `select` একসাথে ❌।
- `select` এর ভেতরে `include`, বা `include` এর ভেতরে `select` ✅:
```ts
await prisma.user.findMany({
  include: {
    orders: { select: { id: true, total: true } },  // ✅
  },
});
```

**API তে নিয়ম:** `select` দিয়ে শুধু দরকারি ফিল্ড পাঠাও। এটা নিরাপত্তা (সংবেদনশীল ফিল্ড) ও গতি (কম ডেটা) দুটোই দেয়।

---

## 10. শেষ পণ্য দুজন একসাথে কিনছে: overselling, race condition ⭐⭐⭐

**সমস্যা (race condition):**
```
সময়    Request A              Request B            stock
t1      stock পড়ল = 1                               1
t2                             stock পড়ল = 1        1
t3      "আছে" → কিনল                                 
t4                             "আছে" → কিনল           
t5      stock = 0 লিখল                               0
t6                             stock = 0 লিখল         0   ← দুটো বিক্রি, কিন্তু আছিল ১টা!
```

**Race condition:** একই ডেটায় একসাথে কাজ করা একাধিক অপারেশনের ফল তাদের সময়ের ক্রমের উপর নির্ভর করা।

### সমাধান ১: Atomic conditional update (সবচেয়ে সহজ ও ভালো) ✅

```ts
await prisma.$transaction(async (tx) => {
  const result = await tx.product.updateMany({
    where: { id: productId, stock: { gte: qty } },     // শর্ত
    data: { stock: { decrement: qty } },               // atomic কমানো
  });

  if (result.count === 0) {
    throw new Error("OUT_OF_STOCK");                   // কেউ আগে কিনে ফেলেছে
  }

  await tx.order.create({ /* ... */ });
});
```
SQL এ এটা `UPDATE ... SET stock = stock - 1 WHERE id = ? AND stock >= 1`। ডাটাবেস row lock করে, তাই দ্বিতীয় request অপেক্ষা করে ও stock 0 দেখে `count = 0` পায়।

### সমাধান ২: Pessimistic locking (`SELECT ... FOR UPDATE`)

```ts
await prisma.$transaction(async (tx) => {
  // row lock করো: অন্য transaction অপেক্ষা করবে
  const [product] = await tx.$queryRaw<{ id: string; stock: number }[]>`
    SELECT id, stock FROM "Product" WHERE id = ${productId} FOR UPDATE
  `;

  if (product.stock < qty) throw new Error("OUT_OF_STOCK");

  await tx.product.update({
    where: { id: productId },
    data: { stock: { decrement: qty } },
  });
});
```
জটিল লজিক (পড়ে সিদ্ধান্ত নিয়ে লেখা) লাগলে।

### সমাধান ৩: Optimistic locking (version column)

```prisma
model Product {
  id      String @id
  stock   Int
  version Int    @default(0)
}
```
```ts
const p = await prisma.product.findUnique({ where: { id } });

const result = await prisma.product.updateMany({
  where: { id, version: p.version },                     // এখনো একই version?
  data: { stock: p.stock - qty, version: { increment: 1 } },
});

if (result.count === 0) throw new Error("CONFLICT: আবার চেষ্টা করো"); // কেউ আগে বদলেছে
```
conflict কম হলে ভালো। conflict হলে retry করতে হয়।

### সমাধান ৪: Serializable isolation + retry

```ts
await prisma.$transaction(fn, { isolationLevel: Prisma.TransactionIsolationLevel.Serializable });
// conflict এ error (P2034) আসে, retry করতে হয়
```

### সমাধান ৫: Database constraint (শেষ রক্ষা)

```sql
ALTER TABLE "Product" ADD CONSTRAINT stock_non_negative CHECK (stock >= 0);
```
কোডে ভুল থাকলেও ডাটাবেস stock ঋণাত্মক হতে দেবে না।

### তুলনা

| উপায় | সুবিধা | অসুবিধা | কখন |
|---|---|---|---|
| Atomic update | সহজ, দ্রুত | জটিল লজিকে সীমিত | সাধারণ stock/balance |
| `FOR UPDATE` | জটিল লজিক চলে | lock এ অপেক্ষা, deadlock ঝুঁকি | পড়ে-সিদ্ধান্ত-লেখা |
| Optimistic | lock নেই | conflict এ retry | conflict কম |
| Serializable | সবচেয়ে নিরাপদ | ধীর, retry লাগে | জটিল ও গুরুত্বপূর্ণ |

### খুব বেশি traffic (flash sale)

DB তে সব request ঢুকলে চাপ পড়ে। Redis এ stock রেখে atomic `DECR` দিয়ে আগে ছাঁকনি, তারপর queue দিয়ে order তৈরি।

### একই ধারণা: দুজন একই record আপডেট করছে (lost update)

```
A পড়ল name="X"; B পড়ল name="X"; A লিখল "A"; B লিখল "B" → A এর বদল হারিয়ে গেল
```
সমাধান একই: atomic update, version column, বা lock।

---

## 11. PostgreSQL query ধীর: কীভাবে খুঁজবে ⭐⭐⭐

### ধাপ ১: ধীর query টা চেনো

```ts
const prisma = new PrismaClient({ adapter, log: ["query", "warn", "error"] });
```
এবং PostgreSQL এ:
```sql
-- pg_stat_statements extension চালু থাকলে
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

### ধাপ ২: `EXPLAIN ANALYZE`

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM "Order" WHERE "userId" = 'abc' AND status = 'PAID' ORDER BY "createdAt" DESC LIMIT 10;
```

**যা খুঁজবে:**

| আউটপুটে | মানে | সমাধান |
|---|---|---|
| `Seq Scan` বড় টেবিলে | পুরো টেবিল ঘাঁটছে | index দাও |
| `Index Scan` / `Index Only Scan` | ভালো | — |
| `Sort` ধীর, disk এ | sort এর জন্য index নেই | `ORDER BY` column index এ রাখো |
| আন্দাজ বনাম আসল row সংখ্যায় বড় ফারাক | statistics পুরনো | `ANALYZE table;` |
| `Nested Loop` লক্ষ বার | join এ index নেই / N+1 | join column এ index |
| `Buffers: read` বেশি | disk থেকে পড়ছে | memory/cache, query ছোট করো |

### ধাপ ৩: সাধারণ কারণ ও সমাধানের তালিকা

| কারণ | সমাধান |
|---|---|
| Index নেই | filter/join/order column এ index |
| ভুল ক্রমের composite index | equality আগে, range/sort পরে |
| `SELECT *` | `select` দিয়ে দরকারি ফিল্ড |
| N+1 | `include`, `in` |
| `OFFSET` অনেক বড় | cursor pagination |
| `LIKE '%text'` | `pg_trgm` GIN index |
| পুরনো statistics | `ANALYZE`, autovacuum ঠিক রাখো |
| Table bloat | `VACUUM` |
| Connection খুব বেশি | PgBouncer / Prisma connection pool |
| ফাংশন column এ (`WHERE lower(email)=...`) | expression index বা আগে normalize |
| অনেক বড় result | pagination |

### ধাপ ৪: বদলের পর আবার মাপো

আগে-পরের `EXPLAIN ANALYZE` তুলনা করো।

---

## 12. PostgreSQL বনাম MongoDB (তুলনা) ⭐⭐⭐

| বিষয় | PostgreSQL | MongoDB |
|---|---|---|
| ডেটা মডেল | টেবিল, row, column (কাঠামোবদ্ধ) | document (JSON-সদৃশ, নমনীয়) |
| Schema | কঠোর | নমনীয় |
| সম্পর্ক | foreign key, `JOIN` | reference বা embed, `$lookup` সীমিত |
| Transaction | পূর্ণ ACID (শক্তিশালী) | আছে, কিন্তু তুলনায় ভারী/কম ব্যবহৃত |
| জটিল query | ভালো (SQL, join, aggregate) | aggregation pipeline |
| Scaling | সাধারণত vertical + replica | সহজে horizontal (sharding) |
| ভালো কাজ | payment, order, inventory, রিপোর্ট | content, catalog, log, দ্রুত বদলানো আকার |

### কখন PostgreSQL

- ডেটার মধ্যে স্পষ্ট সম্পর্ক (user → order → item)
- শক্ত consistency লাগে (টাকা, stock)
- জটিল join ও রিপোর্ট
- ডেটার আকার মোটামুটি স্থির

### কখন MongoDB

- ডেটার আকার ঘনঘন বদলায় বা প্রতিটা item আলাদা (যেমন ভিন্ন ভিন্ন গুণাবলির পণ্য)
- বেশিরভাগ সময় পুরো document একসাথে পড়া হয়
- খুব বড় লেখার চাপ ও সহজ horizontal scaling দরকার

### MongoDB কি সম্পর্ক সামলাতে পারে?

হ্যাঁ, দুই উপায়ে: **embed** (document এর ভেতরে document) বা **reference** (`ObjectId` দিয়ে ইঙ্গিত, `$lookup` দিয়ে join)। কিন্তু **foreign key জোর করে না**, তাই অস্তিত্বহীন রেকর্ডে ইঙ্গিত থেকে যেতে পারে, আর বহু-স্তর join ধীর ও জটিল।

### PostgreSQL কি JSON রাখতে পারে?

হ্যাঁ, `JSONB` টাইপ দিয়ে, এবং GIN index দিয়ে ভেতরে খোঁজাও দ্রুত।

```prisma
model Product {
  id         String @id @default(cuid())
  name       String
  attributes Json                     // PostgreSQL এ JSONB
}
```
```ts
await prisma.product.findMany({
  where: { attributes: { path: ["color"], equals: "red" } },
});
```
```sql
CREATE INDEX idx_attrs ON "Product" USING GIN (attributes);
```
**ফল:** নির্দিষ্ট কিছু নমনীয় ফিল্ডের জন্য আলাদা ডাটাবেস লাগে না। তাই আমার ডিফল্ট PostgreSQL, সম্পর্ক ও নমনীয়তা দুটোই পাওয়া যায়।

**ইন্টারভিউয়ে পরিণত উত্তর:** "কোনটা ভালো" না বলে "ডেটার আকার, সম্পর্ক, consistency এর চাহিদা ও scale অনুযায়ী বেছে নিই" বলো।

---

## 13. Normalization বনাম Denormalization ⭐⭐⭐

### Normalization: প্রতিটা তথ্য একটাই জায়গায়

**❌ Normalize না (duplicate ও ভুলের ঝুঁকি):**

| orderId | customerName | customerEmail | product | price |
|---|---|---|---|---|
| 1 | রহিম | r@a.com | কলম | 10 |
| 2 | রহিম | r@a.com | খাতা | 50 |

রহিমের ইমেইল বদলাতে হলে অনেক row বদলাতে হবে, একটা ভুলে গেলে ডেটা অমিল।

**✅ Normalized:**

```
Customer(id, name, email)
Order(id, customerId)
Product(id, name, price)
OrderItem(orderId, productId, quantity)
```
ইমেইল একটাই জায়গায়, একবার বদলালেই হয়।

### মূল ধাপগুলো (সংক্ষেপে)

| Normal Form | নিয়ম |
|---|---|
| 1NF | প্রতিটা ঘরে একটাই মান (list নয়), প্রতিটা row অনন্য |
| 2NF | 1NF + আংশিক নির্ভরতা নেই (সব column পুরো key এর উপর নির্ভর) |
| 3NF | 2NF + এক non-key column অন্য non-key column এর উপর নির্ভর নয় |

বাস্তবে সাধারণত 3NF পর্যন্ত যথেষ্ট।

### Denormalization: ইচ্ছা করে কপি রাখা

**উদাহরণ:**
```prisma
model Post {
  id           String @id
  commentCount Int    @default(0)   // গণনার জন্য প্রতিবার COUNT না চালিয়ে কপি রাখা
}

model OrderItem {
  unitPrice Decimal                 // Product এর দামের snapshot
}
```

**কখন করবে:**
- পড়া অনেক বেশি, লেখা কম
- ঘনঘন ব্যবহৃত join/aggregate ধীর
- ইতিহাস সংরক্ষণ (দামের snapshot, ঠিকানার snapshot)
- রিপোর্টিং টেবিল

### Trade-off

| | Normalized | Denormalized |
|---|---|---|
| ডেটার সামঞ্জস্য | ✅ সহজ | ⚠️ কপি অমিল হওয়ার ঝুঁকি |
| লেখা | ✅ সহজ (একটাই জায়গায়) | ⚠️ একাধিক জায়গায় আপডেট |
| পড়া | ⚠️ join বেশি | ✅ দ্রুত |
| জায়গা | ✅ কম | ⚠️ বেশি |

**নিয়ম:** normalized দিয়ে শুরু করো, **মেপে** সমস্যা দেখলে তবেই denormalize করো। কপি সিঙ্কে রাখতে transaction, trigger, বা background job ব্যবহার করো।

---

# Tier 2: প্রায়ই আসে

## 14. `findUnique` বনাম `findFirst` বনাম `findMany` ⭐⭐

```ts
// findUnique: unique ফিল্ড দিয়ে ১টা, না পেলে null
const user = await prisma.user.findUnique({ where: { email: "a@b.com" } });

// findUniqueOrThrow: না পেলে error
const user2 = await prisma.user.findUniqueOrThrow({ where: { id } });

// findFirst: যেকোনো filter এ মেলা প্রথমটা
const latest = await prisma.order.findFirst({
  where: { userId, status: "PAID" },
  orderBy: { createdAt: "desc" },
});

// findMany: সব মেলা, array (না পেলে [])
const orders = await prisma.order.findMany({
  where: { status: "PAID" },
  orderBy: { createdAt: "desc" },
  take: 20,
  skip: 0,
});
```

| | `findUnique` | `findFirst` | `findMany` |
|---|---|---|---|
| `where` এ | শুধু unique ফিল্ড | যেকোনো ফিল্ড | যেকোনো ফিল্ড |
| ফেরত | ১টা বা `null` | ১টা বা `null` | array |
| `orderBy`, `take` | ❌ | ✅ | ✅ |
| গতি | সবচেয়ে দ্রুত (unique index) | মাঝারি | নির্ভর করে |

`*OrThrow` সংস্করণ (`findFirstOrThrow`, `findUniqueOrThrow`) না পেলে error ছোড়ে।

---

## 15. Relations ও nested query ⭐⭐

**পড়া (nested):**
```ts
const user = await prisma.user.findUnique({
  where: { id },
  select: {
    name: true,
    orders: {
      where: { status: "PAID" },              // nested filter
      orderBy: { createdAt: "desc" },
      take: 5,
      select: {
        id: true,
        items: { select: { quantity: true, product: { select: { name: true } } } },
      },
    },
  },
});
```

**Relation এর উপর filter (`some`, `every`, `none`):**
```ts
// অন্তত একটা PAID order আছে এমন user
await prisma.user.findMany({ where: { orders: { some: { status: "PAID" } } } });

// কোনো order নেই এমন user
await prisma.user.findMany({ where: { orders: { none: {} } } });
```

**লেখা (nested write):**
```ts
await prisma.user.create({
  data: {
    email: "a@b.com",
    name: "A",
    profile: { create: { bio: "Hello" } },                        // নতুন বানাও
    orders: { connect: [{ id: "o1" }] },                          // বিদ্যমানের সাথে জোড়ো
    // connectOrCreate: থাকলে জোড়ো, না থাকলে বানাও
  },
});
```
Nested write স্বয়ংক্রিয়ভাবে transaction এ চলে।

---

## 16. রেকর্ড মুছলে সম্পর্কিত রেকর্ডের কী হয় ⭐⭐

```prisma
model Order {
  id     String @id
  userId String
  user   User   @relation(fields: [userId], references: [id], onDelete: Restrict)
}
```

| `onDelete` | কী হয় | উদাহরণ |
|---|---|---|
| `Cascade` | সম্পর্কিত রেকর্ডও মুছে যায় | Order মুছলে OrderItem ও |
| `Restrict` | সম্পর্কিত রেকর্ড থাকলে মোছা আটকায় | order ওয়ালা user মোছা যাবে না |
| `NoAction` | `Restrict` এর মতো, তবে চেক statement এর শেষে | |
| `SetNull` | foreign key `null` হয়, রেকর্ড থাকে | post এর লেখক মুছলে `authorId = null` |
| `SetDefault` | foreign key ডিফল্ট মানে যায় | |

```prisma
model Post {
  id       String  @id
  authorId String?                                   // ← SetNull এর জন্য ঐচ্ছিক হতে হবে
  author   User?   @relation(fields: [authorId], references: [id], onDelete: SetNull)
}
```

**Prisma এর ডিফল্ট:** আবশ্যক relation এ `Restrict`, ঐচ্ছিক relation এ `SetNull`। (`onUpdate` ডিফল্ট `Cascade`)

**কখন কী:**
- নিজস্ব অংশ (order এর item, post এর comment) → `Cascade`
- গুরুত্বপূর্ণ/আর্থিক ডেটা → `Restrict`
- ঐচ্ছিক সংযোগ → `SetNull`

**Soft delete (অনেক অ্যাপে ভালো):**
```prisma
model User {
  id        String    @id
  deletedAt DateTime?       // null = সক্রিয়
}
```
```ts
await prisma.user.update({ where: { id }, data: { deletedAt: new Date() } });
await prisma.user.findMany({ where: { deletedAt: null } });   // সক্রিয় শুধু
```
সুবিধা: ডেটা ফেরত আনা যায়, ইতিহাস থাকে, অডিট। অসুবিধা: সব query তে `deletedAt: null` মনে রাখতে হয় (Client Extension দিয়ে স্বয়ংক্রিয় করা যায়), unique constraint এ সাবধান (partial unique index)।

---

## 17. Migration ও `prisma migrate dev` ⭐⭐

**Migration:** schema বদলের সংস্করণ করা SQL ফাইল।

```
prisma/
├── schema.prisma
└── migrations/
    ├── 20260101_init/migration.sql
    ├── 20260115_add_orders/migration.sql
    └── migration_lock.toml
```

### `prisma migrate dev` কী করে

```
১. migration ইতিহাস অনুযায়ী shadow database (অস্থায়ী) বানায়
২. তার সাথে তোমার schema.prisma তুলনা করে
৩. পার্থক্য থেকে নতুন SQL migration ফাইল বানায়
৪. development ডাটাবেসে প্রয়োগ করে
৫. Prisma Client নতুন করে generate করে
```
```bash
pnpm prisma migrate dev --name add_orders
```

### কমান্ডগুলোর পার্থক্য

| কমান্ড | কখন | কী করে |
|---|---|---|
| `pnpm prisma migrate dev` | **শুধু development** | migration বানায় ও প্রয়োগ করে |
| `pnpm prisma migrate deploy` | **production/CI** | বিদ্যমান migration প্রয়োগ করে (নতুন বানায় না, reset করে না) |
| `pnpm prisma migrate dev --create-only` | যখন SQL হাতে বদলাতে চাও | ফাইল বানায়, প্রয়োগ করে না |
| `pnpm prisma db push` | দ্রুত prototype | migration ফাইল ছাড়া সরাসরি schema সিঙ্ক |
| `pnpm prisma migrate reset` | development | ডাটাবেস মুছে সব migration আবার চালায় ⚠️ ডেটা যাবে |
| `pnpm prisma generate` | schema বদলের পর | Client নতুন করে বানায় |
| `pnpm prisma migrate status` | যাচাই | কোন migration প্রয়োগ হয়েছে দেখায় |

**Production এ কখনো `migrate dev` বা `migrate reset` চালিও না।**

---

## 18. Production এ ডেটা না হারিয়ে schema বদল ⭐⭐

### Expand and Contract পদ্ধতি

**উদাহরণ: `fullName` column এর নাম বদলে `name` করা।**

Prisma নাম বদলকে "পুরনো column মোছো + নতুন বানাও" হিসেবে বোঝে, **ডেটা হারাবে**। তাই ধাপে ধাপে:

```
ধাপ ১ (Expand):   নতুন `name` column যোগ (ঐচ্ছিক/nullable)
ধাপ ২:            কোড বদলাও: লেখার সময় দুটোতেই লেখো
ধাপ ৩ (Backfill): পুরনো ডেটা কপি
                  UPDATE "User" SET name = "fullName" WHERE name IS NULL;
ধাপ ৪:            পড়া `name` থেকে শুরু করো
ধাপ ৫:            `name` কে NOT NULL করো
ধাপ ৬ (Contract): পরের release এ পুরনো `fullName` মোছো
```

### অন্য সাধারণ ক্ষেত্র

**নতুন আবশ্যক (NOT NULL) column:**
```prisma
// ❌ বিদ্যমান row এ মান নেই, migration ব্যর্থ হবে
phone String

// ✅ default দাও, বা ঐচ্ছিক করে শুরু করে পরে বাধ্যতামূলক করো
phone String @default("")
```

**হাতে SQL বদলানো:**
```bash
pnpm prisma migrate dev --create-only --name rename_fullname
# migration.sql খুলে DROP/ADD এর বদলে লেখো:
#   ALTER TABLE "User" RENAME COLUMN "fullName" TO "name";
pnpm prisma migrate dev     # এবার প্রয়োগ
```

**বড় টেবিলে index:**
```sql
CREATE INDEX CONCURRENTLY idx_orders_created ON "Order" ("createdAt");
```

### নিরাপত্তার তালিকা

| ধাপ | কেন |
|---|---|
| Backup নাও | ভুল হলে ফেরত আনতে |
| Staging এ production এর কপিতে পরীক্ষা | আসল ডেটায় আচরণ দেখতে |
| `--create-only` দিয়ে SQL পড়ো | অজান্তে `DROP` ধরতে |
| ছোট ছোট বদল | ঝুঁকি কম, ফেরত নেওয়া সহজ |
| CI থেকে `migrate deploy` | মানুষের ভুল কমে |
| কোড ও schema পরপর deploy, পুরনো ও নতুন দুই কোডের সাথে মানানসই বদল | downtime ছাড়া |
| বড় টেবিলে lock এর ঝুঁকি দেখো | `ALTER TABLE` এ টেবিল আটকে যেতে পারে |

---

## 19. Users সহ সর্বশেষ ১০টা order ⭐⭐

```ts
const users = await prisma.user.findMany({
  take: 20,                                    // users এর pagination
  skip: 0,
  orderBy: { createdAt: "desc" },
  select: {
    id: true,
    name: true,
    email: true,
    orders: {
      orderBy: { createdAt: "desc" },
      take: 10,                                // ← প্রতিটা user এর জন্য আলাদাভাবে ১০টা
      select: { id: true, total: true, status: true, createdAt: true },
    },
  },
});
```

**মূল কথা:** nested relation এ `take` **প্রতিটা parent এর জন্য আলাদাভাবে** প্রযোজ্য। সব order মিলিয়ে ১০টা নয়।

**Index (জরুরি):**
```prisma
model Order {
  // ...
  @@index([userId, createdAt(sort: Desc)])
}
```
এতে প্রতিটা user এর সর্বশেষ order সরাসরি index থেকে আসে।

**আরও:**
- `select` দিয়ে ফিল্ড কমাও
- users তালিকায় pagination (cursor ভালো)
- ঘনঘন লাগলে Redis এ cache
- `orders` তালিকা বড় ও চাপ বেশি হলে আলাদা endpoint (`/users/:id/orders`)

---

## 20. `userId`, `status`, `createdAt` ফিল্টারে index ⭐⭐

**Query:**
```ts
await prisma.order.findMany({
  where: { userId, status: "PAID" },
  orderBy: { createdAt: "desc" },
  take: 20,
});
```

**Index:**
```prisma
model Order {
  @@index([userId, status, createdAt(sort: Desc)])
}
```

**ক্রমের নিয়ম:** equality (`=`) column আগে, range (`>`, `<`) বা sort column শেষে।

**বাম-থেকে-ডান নিয়ম:** `(userId, status, createdAt)` index এই query গুলোতে কাজ করে:

| Query filter | index ব্যবহার হয়? |
|---|---|
| `userId` | ✅ |
| `userId + status` | ✅ |
| `userId + status + createdAt` | ✅ সবচেয়ে ভালো |
| `userId + createdAt` (status ছাড়া) | ⚠️ আংশিক |
| শুধু `status` | ❌ |
| শুধু `createdAt` | ❌ |

**তাই:**
- শুধু `status` দিয়ে ঘনঘন filter হলে আলাদা index (বা partial index `WHERE status = 'PENDING'`)।
- `EXPLAIN ANALYZE` দিয়ে নিশ্চিত হও index ব্যবহার হচ্ছে।
- সব সম্ভাব্য সংমিশ্রণে index দিও না, লেখা ধীর হবে।

---

## 21. App এ চেক করার পরও duplicate ⭐⭐

**কারণ: check-then-insert race condition**
```
Request A: ইমেইল আছে? → না
Request B: ইমেইল আছে? → না        (A এখনো insert করেনি)
Request A: insert ✅
Request B: insert ✅                ← duplicate!
```

```ts
// ❌ ভুল
const exists = await prisma.user.findUnique({ where: { email } });
if (!exists) await prisma.user.create({ data: { email } });
```

### সমাধান ১: DB তে unique constraint (আসল সমাধান)

```prisma
model User {
  email String @unique
}
```

### সমাধান ২: error সামলানো

```ts
import { Prisma } from "../generated/prisma/client";

try {
  await prisma.user.create({ data: { email } });
} catch (e) {
  if (e instanceof Prisma.PrismaClientKnownRequestError && e.code === "P2002") {
    // unique constraint ভেঙেছে → "এই ইমেইল আগে থেকেই আছে"
    throw new Error("EMAIL_EXISTS");
  }
  throw e;
}
```

### সমাধান ৩: `upsert` (থাকলে আপডেট, না থাকলে বানাও)

```ts
await prisma.user.upsert({
  where: { email },
  update: {},
  create: { email, name },
});
```

### সমাধান ৪: bulk insert এ

```ts
await prisma.user.createMany({ data: users, skipDuplicates: true });
```

### অন্য কারণ

| কারণ | সমাধান |
|---|---|
| বড়/ছোট হাতের অক্ষর (`A@b.com` vs `a@b.com`) | সংরক্ষণের আগে `toLowerCase()`, বা `citext` টাইপ |
| `NULL` মান (PostgreSQL unique এ একাধিক NULL চলে) | `NOT NULL` করো, বা partial unique index |
| Soft delete করা রেকর্ডের সাথে সংঘাত | `WHERE deletedAt IS NULL` সহ partial unique index |
| composite uniqueness | `@@unique([a, b])` |
| extra spaces | `trim()` |
| একই request দুইবার আসা (double click, retry) | idempotency key |

**মূলনীতি:** যে নিয়ম ভাঙলে ডেটা নষ্ট হয়, তা **ডাটাবেসে** জোর করো। শুধু অ্যাপ কোডে রাখলে concurrency তে ভাঙবে।

---

## 22. ১ কোটি user, ইমেইল দিয়ে ঘনঘন খোঁজ ⭐⭐

**ধাপ ১: Unique index (সবচেয়ে গুরুত্বপূর্ণ)**
```prisma
model User {
  email String @unique      // B-tree unique index অটো
}
```
```ts
await prisma.user.findUnique({ where: { email } });   // ১ কোটি row এও মিলিসেকেন্ডে
```
B-tree এ খোঁজ O(log n): ১ কোটি row তে মাত্র কয়েক ধাপ।

**ধাপ ২: Normalize করো**
```ts
const email = input.trim().toLowerCase();
await prisma.user.findUnique({ where: { email } });
```
সংরক্ষণ ও খোঁজ দুই জায়গায় একই normalization, তাহলে একটাই index যথেষ্ট।

**ধাপ ৩: আংশিক খোঁজ হলে**

| খোঁজ | index কাজ করে? | সমাধান |
|---|---|---|
| `email = 'a@b.com'` | ✅ | সাধারণ index |
| `email LIKE 'abc%'` (prefix) | ✅ (কিছু ক্ষেত্রে) | `text_pattern_ops` index |
| `email LIKE '%abc%'` (contains) | ❌ | `pg_trgm` + GIN index |
| ফাজি/সম্পূর্ণ লেখা খোঁজ | ❌ | full-text search / Elasticsearch / Meilisearch |

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_user_email_trgm ON "User" USING GIN (email gin_trgm_ops);
```

**ধাপ ৪: অন্যান্য**
- `select` দিয়ে শুধু দরকারি ফিল্ড
- ঘনঘন খোঁজা ডেটা Redis এ cache (login এ)
- `EXPLAIN ANALYZE` দিয়ে Index Scan নিশ্চিত করো
- Connection pooling

---

# Tier 3: মাঝে মাঝে আসে

## 23. Eventual consistency ⭐

**Strong consistency:** write এর পর প্রতিটা read সর্বশেষ মান দেখে।
**Eventual consistency:** write এর পর কিছুক্ষণ আলাদা কপি আলাদা মান দেখাতে পারে, কিন্তু শেষে সব সমান হয়।

**উদাহরণ:** তুমি Facebook এ like দিলে, বন্ধু কয়েক সেকেন্ড পর count বাড়তে দেখে।

```
Primary DB:   লেখা হলো (version 2)
                 │ (কয়েক মিলিসেকেন্ড/সেকেন্ড দেরিতে)
                 ▼
Read replica: এখনো version 1  →  শেষে version 2
```

**কোথায় দেখা যায়:** read replica, NoSQL (বিশেষত বিতরিত), cache (Redis এ পুরনো মান), CDN, ইভেন্ট-ভিত্তিক সিস্টেম।

| | Strong | Eventual |
|---|---|---|
| সঠিকতা | সবসময় সর্বশেষ | কিছুক্ষণ পুরনো হতে পারে |
| গতি/availability | ধীর | দ্রুত, বেশি available |
| কখন | টাকা, stock, auth | like count, feed, analytics, search index |

**Read-your-writes সমস্যা:** ব্যবহারকারী পোস্ট করে সাথে সাথে পেজ reload করলে replica এ পুরনো ডেটা দেখতে পারে। সমাধান: নিজের লেখার পর কিছুক্ষণ primary থেকে পড়া।
