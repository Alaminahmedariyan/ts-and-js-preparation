# Database Interview Answers (Prisma + PostgreSQL + Cross-Database)

ইন্টারভিউতে বলার জন্য ছোট ও সহজ ইংরেজি উত্তর, সাথে বাংলা মানে। বিস্তারিত বোঝার জন্য `database-details.md` পড়ো।

**কীভাবে পড়বে:** আগে Tier 1 (সবচেয়ে বেশি আসে), তারপর Tier 2, শেষে Tier 3।

| চিহ্ন | মানে |
|---|---|
| ⭐⭐⭐ | প্রায় প্রতিটা ইন্টারভিউতে আসে |
| ⭐⭐ | প্রায়ই আসে |
| ⭐ | মাঝে মাঝে আসে |

> **MongoDB/Mongoose সেকশন এই ফাইলে নেই** (তোমার অনুরোধে)। শুধু "কখন PostgreSQL, কখন MongoDB" তুলনার প্রশ্ন আছে, কারণ এটা প্রায়ই জিজ্ঞেস করা হয়।
>
> **Prisma সংস্করণ:** Prisma 7 এ client setup বদলেছে (`prisma-client` generator, driver adapter, `prisma.config.ts`)। ইন্টারভিউয়ের প্রশ্ন ও উত্তর একই, শুধু setup কোড আলাদা। বিস্তারিত `database-details.md` তে।

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. Prisma vs PostgreSQL. What problem does Prisma solve? ⭐⭐⭐

> "PostgreSQL is the database. It stores the data and runs SQL queries. Prisma is an ORM, a tool in my application code that talks to the database for me. Prisma solves three problems: it gives type-safe queries with auto-complete, so many mistakes are caught before running; it describes my tables in one readable schema file; and it manages schema changes with migrations. So they are not alternatives: I use Prisma to work with PostgreSQL."

**বাংলা মানে:** PostgreSQL হলো ডাটাবেস, ডেটা রাখে ও SQL চালায়। Prisma হলো ORM, আমার কোড থেকে ডাটাবেসের সাথে কথা বলার টুল। Prisma তিনটা সমস্যা সমাধান করে: type-safe query, এক জায়গায় পরিষ্কার schema, আর migration দিয়ে schema বদল সামলানো। দুটো বিকল্প না, Prisma দিয়ে PostgreSQL ব্যবহার করি।

---

## 2. Explain one-to-one, one-to-many, and many-to-many in Prisma ⭐⭐⭐

> "One-to-one means one record is linked to exactly one record in another table, like a User and a Profile. In Prisma I make the foreign key `@unique`. One-to-many means one record has many related records, like one User has many Orders. The foreign key is on the 'many' side. Many-to-many means records on both sides can have many of each other, like Posts and Tags. Prisma can create the join table automatically for an implicit relation, or I can define the join model myself when I need extra fields, like a quantity or a date."

**বাংলা মানে:** One-to-one: একটা রেকর্ড অন্য টেবিলের ঠিক একটা রেকর্ডের সাথে (User ও Profile), foreign key তে `@unique` দিই। One-to-many: একটার অনেকগুলো (একজন User এর অনেক Order), foreign key "many" দিকে থাকে। Many-to-many: দুই দিকেই অনেক (Post ও Tag), Prisma নিজে join table বানাতে পারে, বা বাড়তি ফিল্ড (quantity, date) লাগলে আমি নিজে join model লিখি।

---

## 3. How would you model User → Orders → OrderItems → Products? ⭐⭐⭐

> "A User has many Orders. An Order has many OrderItems. A Product can appear in many OrderItems. So OrderItem is the join table between Order and Product, and it also stores the quantity. An important detail is that I save the unit price inside OrderItem, as a snapshot, because the product price can change later and old orders must not change. I use `Decimal` for money, add a unique constraint on `orderId` and `productId`, and add indexes on the foreign keys."

**বাংলা মানে:** User এর অনেক Order, Order এর অনেক OrderItem, একটা Product অনেক OrderItem এ থাকতে পারে। তাই OrderItem হলো Order ও Product এর মাঝের join table, এতে quantity থাকে। গুরুত্বপূর্ণ: OrderItem এ ইউনিট দাম snapshot হিসেবে সংরক্ষণ করি, কারণ পরে Product এর দাম বদলালে পুরনো order বদলানো যাবে না। টাকার জন্য `Decimal`, আর foreign key তে index দিই।

---

## 4. What is a transaction? What does ACID mean? When use `$transaction()`? ⭐⭐⭐

> "A transaction groups several database operations into one unit: either all of them succeed, or all of them are rolled back. ACID describes its guarantees. Atomicity means all or nothing. Consistency means the data always follows the rules, like constraints. Isolation means transactions running at the same time do not break each other. Durability means once committed, the data is saved even after a crash. I use `prisma.$transaction()` whenever several writes must succeed or fail together, like creating an order and reducing stock."

**বাংলা মানে:** Transaction কয়েকটা কাজকে একটা একক হিসেবে চালায়: সব সফল, নয়তো সব বাতিল (rollback)। ACID: Atomicity (হয় সব নয় কিছুই না), Consistency (নিয়ম/constraint সবসময় মানা হয়), Isolation (একসাথে চলা transaction একে অপরকে নষ্ট করে না), Durability (commit এর পর crash হলেও ডেটা থাকে)। একাধিক write একসাথে সফল বা ব্যর্থ হতে হলে `$transaction()` ব্যবহার করি।

---

## 5. Money deducted and order created: how to make both succeed or fail together? ⭐⭐⭐

> "I wrap both operations in one `prisma.$transaction()`. Inside it, I deduct the balance with a conditional update, for example 'update where balance is greater than or equal to the amount'. If no row was updated, I throw an error, so the whole transaction rolls back. Then I create the order in the same transaction. Because the check and the update happen in one atomic statement, two requests cannot spend the same money twice. If anything throws, Prisma rolls everything back."

**বাংলা মানে:** দুটো কাজ একটা `$transaction()` এ রাখি। ভেতরে শর্তসহ update দিয়ে টাকা কাটি ("balance >= amount হলে তবেই")। কোনো row আপডেট না হলে error ছুড়ি, ফলে পুরো transaction rollback হয়। তারপর একই transaction এ order বানাই। চেক ও আপডেট একটাই atomic statement এ হওয়ায় দুটো request একই টাকা দুইবার খরচ করতে পারে না।

---

## 6. What is the N+1 query problem? ⭐⭐⭐

> "The N+1 problem happens when I run one query to get a list of N records, and then run one extra query for each record to get related data. So I make N+1 queries instead of one or two, which is very slow. In Prisma it usually happens when I loop over results and call `findMany` inside the loop. The fix is to load the relations together using `include` or `select`, or to fetch all the related rows in one query with `where: { id: { in: ids } }`."

**বাংলা মানে:** একটা query তে N টা রেকর্ড আনি, তারপর প্রতিটার সম্পর্কিত ডেটার জন্য আলাদা query চালাই। ফলে ১ + N টা query হয়, যা খুব ধীর। Prisma এ সাধারণত লুপের ভেতরে `findMany` ডাকলে হয়। সমাধান: `include`/`select` দিয়ে সম্পর্কিত ডেটা একসাথে আনা, বা `in` দিয়ে সব এক query তে আনা।

---

## 7. What are indexes? When should you create one? Can too many hurt? ⭐⭐⭐

> "An index is a separate data structure, usually a B-tree, that lets the database find rows quickly without scanning the whole table, like the index of a book. I create indexes on columns that I often use in `WHERE`, `JOIN`, and `ORDER BY`, such as foreign keys and search fields. But indexes are not free. Each index takes storage and makes every insert, update, and delete slower, because the index must be updated too. So I add indexes based on real query patterns, and I avoid indexing columns with very few distinct values."

**বাংলা মানে:** Index হলো আলাদা ডেটা কাঠামো (সাধারণত B-tree), যা পুরো টেবিল না ঘেঁটে দ্রুত row খুঁজে দেয়, বইয়ের সূচিপত্রের মতো। `WHERE`, `JOIN`, `ORDER BY` এ ঘনঘন ব্যবহৃত column এ (foreign key, search ফিল্ড) index দিই। কিন্তু index বিনামূল্যে না: জায়গা নেয় এবং প্রতিটা insert/update/delete ধীর করে। তাই আসল query দেখে index দিই, আর খুব কম ভিন্ন মান ওয়ালা column এ দিই না।

---

## 8. Primary key vs unique constraint vs foreign key vs index ⭐⭐⭐

> "A primary key uniquely identifies each row, cannot be null, and there is only one per table. A unique constraint also stops duplicates, but a table can have many of them, and PostgreSQL allows null values in them. A foreign key links a column to the primary key of another table and keeps the relation valid, so I cannot point to a record that does not exist. An index only makes searches faster. It does not enforce rules, although PostgreSQL creates an index automatically for primary keys and unique constraints."

**বাংলা মানে:** Primary key প্রতিটা row কে অনন্যভাবে চেনায়, null হতে পারে না, টেবিলে একটাই। Unique constraint ও duplicate আটকায়, কিন্তু একাধিক থাকতে পারে ও null অনুমোদিত। Foreign key অন্য টেবিলের primary key এর সাথে সংযোগ রাখে, ফলে অস্তিত্বহীন রেকর্ডে ইঙ্গিত করা যায় না। Index শুধু search দ্রুত করে, কোনো নিয়ম চাপায় না (যদিও primary key ও unique এর জন্য PostgreSQL নিজেই index বানায়)।

---

## 9. `include` vs `select` in Prisma ⭐⭐⭐

> "`select` lets me choose exactly which fields I want, so the response is smaller and safer, because I can leave out sensitive fields like password. `include` returns all the fields of the model plus the relations I add. I cannot use `select` and `include` together at the same level, but I can nest `include` inside a `select` or the other way around. I prefer `select` for API responses, because it returns only what the client needs."

**বাংলা মানে:** `select` দিয়ে ঠিক কোন কোন ফিল্ড চাই তা বাছি, ফলে response ছোট ও নিরাপদ (password এর মতো ফিল্ড বাদ)। `include` মডেলের সব ফিল্ড ও সাথে যোগ করা relation দেয়। একই স্তরে `select` ও `include` একসাথে ব্যবহার করা যায় না। API response এ আমি `select` পছন্দ করি।

---

## 10. Two users buy the last product at the same time. How do you prevent overselling? ⭐⭐⭐

> "This is a race condition: both requests read stock as 1, both think it is available, and both buy it. I solve it by making the check and the decrease one atomic operation in the database. For example, in a transaction I run `updateMany` where `stock` is greater than or equal to the quantity, and I decrement it. If the update count is zero, the stock is gone, so I reject the order. The database locks the row for the update, so the second request sees stock 0. Other options are `SELECT ... FOR UPDATE` row locking, optimistic locking with a version column, and a database `CHECK` constraint so stock can never go below zero."

**বাংলা মানে:** এটা race condition: দুটো request ই stock পড়ে 1, দুজনই ভাবে আছে, দুজনই কেনে। সমাধান: চেক ও কমানো ডাটাবেসে একটাই atomic কাজ করা। Transaction এ `stock >= quantity` শর্তে `updateMany` দিয়ে `decrement` করি। update count 0 হলে stock নেই, order বাতিল। ডাটাবেস row lock করায় দ্বিতীয় request stock 0 দেখে। অন্য উপায়: `SELECT ... FOR UPDATE`, version column দিয়ে optimistic locking, আর stock কখনো ঋণাত্মক না হতে `CHECK` constraint।

---

## 11. A PostgreSQL query is slow. How do you investigate? ⭐⭐⭐

> "First, I find which query is slow, using Prisma query logs or `pg_stat_statements`. Then I run `EXPLAIN ANALYZE` on it to see the execution plan. I look for a sequential scan on a big table, a bad row estimate, or a slow sort. If a filter or join column has no index, I add one. I also check for the N+1 problem, selecting too many columns, missing pagination, outdated statistics, which `ANALYZE` can fix, table bloat, and connection pool limits. After the change, I measure again to confirm the improvement."

**বাংলা মানে:** আগে ধীর query টা খুঁজি (Prisma log বা `pg_stat_statements`)। তারপর `EXPLAIN ANALYZE` দিয়ে execution plan দেখি। বড় টেবিলে sequential scan, ভুল row আন্দাজ, ধীর sort খুঁজি। filter/join column এ index না থাকলে দিই। N+1, অনেক column আনা, pagination না থাকা, পুরনো statistics (`ANALYZE`), table bloat, connection pool সীমাও দেখি। বদলের পর আবার মেপে নিশ্চিত হই।

---

## 12. PostgreSQL vs MongoDB: when to choose which? ⭐⭐⭐

> "I choose PostgreSQL when data is structured and has clear relations, when I need strong consistency and transactions, like for payments, orders, and inventory, and when I need complex queries with joins. I choose MongoDB when the data shape changes often or is document-like, when most reads need the whole document at once, and when I need easy horizontal scaling. MongoDB can handle relationships through references, but it does not enforce foreign keys, and joins are more limited. PostgreSQL can also store JSON with the `JSONB` type and index it, so it can cover many flexible-data needs. My default choice is PostgreSQL, because it is flexible enough for most applications."

**বাংলা মানে:** ডেটা কাঠামোবদ্ধ ও সম্পর্কযুক্ত হলে, শক্ত consistency ও transaction লাগলে (payment, order, inventory), জটিল join query লাগলে PostgreSQL। ডেটার আকার ঘনঘন বদলালে বা document-ধাঁচের হলে, প্রায়ই পুরো document একসাথে পড়লে, সহজ horizontal scaling লাগলে MongoDB। MongoDB reference দিয়ে সম্পর্ক রাখতে পারে, কিন্তু foreign key জোর করে না, join সীমিত। PostgreSQL `JSONB` দিয়ে JSON রেখে index ও করতে পারে। আমার ডিফল্ট PostgreSQL।

---

## 13. Normalization vs denormalization ⭐⭐⭐

> "Normalization means organizing data so that each fact is stored in only one place, by splitting it into related tables. It removes duplicate data and prevents update mistakes. Denormalization means intentionally copying some data to make reads faster, for example storing a product price snapshot in an order, or a `commentCount` in a post. The trade-off is this: normalization gives clean and consistent data but needs more joins, and denormalization gives faster reads but costs more storage and the risk that copies go out of sync. I start normalized, and denormalize only where I have measured a real performance need."

**বাংলা মানে:** Normalization মানে প্রতিটা তথ্য একটাই জায়গায় রাখা, সম্পর্কিত টেবিলে ভাগ করে। এতে duplicate ও আপডেট ভুল কমে। Denormalization মানে পড়া দ্রুত করতে ইচ্ছা করে ডেটার কপি রাখা (order এ দামের snapshot, post এ `commentCount`)। Trade-off: normalized এ ডেটা পরিষ্কার কিন্তু join বেশি; denormalized এ পড়া দ্রুত কিন্তু জায়গা বেশি ও কপি অমিল হওয়ার ঝুঁকি। আমি normalized দিয়ে শুরু করি, মেপে প্রয়োজন দেখলেই denormalize করি।

---

# Tier 2: প্রায়ই আসে

## 14. `findUnique()` vs `findFirst()` vs `findMany()` ⭐⭐

> "`findUnique()` finds one record using a unique field, like `id` or `email`, and returns the record or `null`. It is the fastest, because it uses a unique index. `findFirst()` returns the first record that matches any filter, and I can combine it with `orderBy`. `findMany()` returns an array of all matching records, and an empty array if none match. I use `findUnique` when I know the unique value, and `findMany` with `take` and `skip` for lists."

**বাংলা মানে:** `findUnique()` unique ফিল্ড (`id`, `email`) দিয়ে একটা রেকর্ড আনে, পায় রেকর্ড বা `null`। unique index এর কারণে সবচেয়ে দ্রুত। `findFirst()` যেকোনো filter এ মেলা প্রথম রেকর্ড দেয় (`orderBy` সহ)। `findMany()` মেলা সব রেকর্ডের array দেয়, না মিললে খালি array।

---

## 15. How does Prisma handle relations and nested queries? ⭐⭐

> "I define relations in the schema with `@relation`, and Prisma generates typed fields for them. In queries, I use `include` or `select` to load related data in a nested way, and I can filter, sort, and limit inside the nested part. For writes, I can use nested `create`, `connect`, and `connectOrCreate` to create or link related records in one call, which Prisma runs inside a transaction. The result is fully typed, so TypeScript knows the shape of the nested data."

**বাংলা মানে:** Schema তে `@relation` দিয়ে সম্পর্ক লিখি, Prisma typed ফিল্ড বানায়। Query তে `include`/`select` দিয়ে সম্পর্কিত ডেটা nested আনি, ভেতরেও filter, sort, limit করা যায়। Write এ nested `create`, `connect`, `connectOrCreate` দিয়ে এক call এ সম্পর্কিত রেকর্ড বানানো বা জোড়া যায়, যা Prisma transaction এ চালায়। ফলাফল typed।

---

## 16. What happens when you delete a record that has related records? ⭐⭐

> "It depends on the referential action I set with `onDelete`. `Cascade` deletes the related records too. `Restrict` blocks the delete if related records exist. `SetNull` keeps the related records but sets their foreign key to null, so the column must be optional. `NoAction` is similar to `Restrict`, but the check is done at the end of the statement. I use `Cascade` for owned data, like order items of an order, and `Restrict` for important data, like a user with orders. For many apps, I prefer soft delete with a `deletedAt` field."

**বাংলা মানে:** নির্ভর করে `onDelete` এ কী সেট করেছি। `Cascade` সম্পর্কিত রেকর্ডও মুছে দেয়। `Restrict` সম্পর্কিত রেকর্ড থাকলে মোছা আটকায়। `SetNull` সম্পর্কিত রেকর্ড রেখে foreign key null করে (column optional হতে হবে)। `NoAction` প্রায় `Restrict` এর মতো, তবে চেক statement এর শেষে। নিজস্ব ডেটায় (order এর item) `Cascade`, গুরুত্বপূর্ণ ডেটায় (order ওয়ালা user) `Restrict`। অনেক অ্যাপে `deletedAt` দিয়ে soft delete পছন্দ করি।

---

## 17. What is a Prisma migration? What does `prisma migrate dev` do? ⭐⭐

> "A migration is a versioned SQL file that records a change to the database schema. Together, the migration files are the history of my schema. When I run `prisma migrate dev`, Prisma compares my schema with the migration history using a shadow database, creates a new SQL migration file, applies it to my development database, and then regenerates the Prisma Client. It is only for development. In production, I run `prisma migrate deploy`, which applies the existing migration files without creating new ones or resetting anything."

**বাংলা মানে:** Migration হলো সংস্করণ করা SQL ফাইল যা schema বদল নথিভুক্ত করে। সব migration মিলে schema এর ইতিহাস। `prisma migrate dev` চালালে Prisma shadow database দিয়ে schema ও ইতিহাস তুলনা করে, নতুন SQL migration ফাইল বানায়, development ডাটাবেসে প্রয়োগ করে, তারপর Prisma Client নতুন করে generate করে। এটা শুধু development এর জন্য। Production এ `prisma migrate deploy` চালাই, যা শুধু বিদ্যমান migration প্রয়োগ করে।

---

## 18. How do you change the schema in production without losing data? ⭐⭐

> "I use small, safe steps, called the expand and contract pattern. For example, to rename a column: first I add the new column, then I deploy code that writes to both, then I copy the old data into the new column, then I switch reads to the new column, and finally I remove the old column in a later release. I never let Prisma drop and recreate a column that has data, so I review the generated SQL with `--create-only`. I take a backup, test the migration on a copy of production data, create indexes with `CONCURRENTLY` on big tables to avoid locking, and run `migrate deploy` from my CI pipeline."

**বাংলা মানে:** ছোট নিরাপদ ধাপে এগোই (expand and contract)। যেমন column এর নাম বদলাতে: নতুন column যোগ → কোড দুটোতেই লিখবে → পুরনো ডেটা কপি → পড়া নতুন column এ সরানো → পরের release এ পুরনো column মোছা। ডেটা ওয়ালা column Prisma কে drop ও recreate করতে দিই না, তাই `--create-only` দিয়ে SQL দেখে নিই। Backup নিই, production ডেটার কপিতে পরীক্ষা করি, বড় টেবিলে `CONCURRENTLY` দিয়ে index বানাই যাতে lock না পড়ে, আর CI থেকে `migrate deploy` চালাই।

---

## 19. API returns users with their latest 10 orders. How would you design the query? ⭐⭐

> "I use `findMany` on users, and inside it I select the `orders` relation with `orderBy: { createdAt: 'desc' }` and `take: 10`. In Prisma, `take` inside a nested relation applies to each user separately, so every user gets their own latest 10 orders. I also use `select` to return only needed fields, add pagination on the users list, and create an index on `(userId, createdAt DESC)` in the orders table, so the nested query is fast."

**বাংলা মানে:** Users এ `findMany` দিই, ভেতরে `orders` relation এ `orderBy: createdAt desc` ও `take: 10`। Prisma তে nested relation এর `take` প্রতিটা user এর জন্য আলাদাভাবে কাজ করে, তাই প্রত্যেকে নিজের সর্বশেষ ১০টা order পায়। `select` দিয়ে দরকারি ফিল্ড, users তালিকায় pagination, আর orders এ `(userId, createdAt DESC)` index দিই।

---

## 20. Orders are filtered by `userId`, `status`, and `createdAt`. Which indexes? ⭐⭐

> "I would create a composite index with the columns in the right order: equality filters first, then the range or sort column. So `(userId, status, createdAt)` works well for queries like 'orders of this user, with this status, sorted by date'. The order matters, because an index can be used from the left side: it also helps queries with only `userId`, or `userId` and `status`, but not with only `status`. If I often filter only by `status`, I may add a separate index. I check with `EXPLAIN ANALYZE` that the index is really used."

**বাংলা মানে:** সঠিক ক্রমে composite index দিই: আগে equality filter, তারপর range বা sort column। তাই `(userId, status, createdAt)`। ক্রম গুরুত্বপূর্ণ, কারণ index বাম দিক থেকে কাজ করে: শুধু `userId`, বা `userId` ও `status` দিয়ে query তেও সাহায্য করে, কিন্তু শুধু `status` দিয়ে করে না। শুধু `status` দিয়ে ঘনঘন filter হলে আলাদা index দিতে পারি। `EXPLAIN ANALYZE` দিয়ে নিশ্চিত হই index সত্যিই ব্যবহার হচ্ছে।

---

## 21. Duplicate records exist even though the app checks before inserting. Why? ⭐⭐

> "This is a race condition. Two requests run at the same time. Both check and find no duplicate, and then both insert. The check and the insert are two separate steps, so the database cannot protect them. The real fix is a unique constraint in the database, because the database guarantees it even with concurrent requests. Then I handle the unique violation error in the code, which in Prisma is error code `P2002`, or I use `upsert`. I also watch for other causes: emails with different cases, which I fix by normalizing to lowercase, and null values, which PostgreSQL treats as different from each other in a unique index."

**বাংলা মানে:** এটা race condition। দুটো request একসাথে চলে, দুজনই চেক করে duplicate পায় না, তারপর দুজনই insert করে। চেক ও insert দুটো আলাদা ধাপ, ডাটাবেস সুরক্ষা দিতে পারে না। আসল সমাধান ডাটাবেসে unique constraint, যা একসাথে আসা request এও কাজ করে। কোডে Prisma এর `P2002` error সামলাই, বা `upsert` ব্যবহার করি। অন্য কারণও দেখি: ছোট/বড় হাতের অক্ষর (lowercase এ normalize করি) এবং null (unique index এ PostgreSQL null গুলোকে আলাদা ধরে)।

---

## 22. 10 million users, frequent search by email. How to optimize? ⭐⭐

> "First, I make sure `email` has a unique index. In Prisma, `@unique` creates a B-tree index, and `findUnique` uses it, so the search is fast even with 10 million rows. I normalize the email to lowercase before saving and searching, so one index works for all cases. If I need partial search, like 'starts with' or 'contains', a normal index does not help with a leading wildcard, so I use a `pg_trgm` index or full-text search. I also select only the needed fields, cache hot lookups in Redis, and check `EXPLAIN ANALYZE` to confirm an index scan."

**বাংলা মানে:** আগে নিশ্চিত করি `email` এ unique index আছে। Prisma তে `@unique` B-tree index বানায়, `findUnique` সেটা ব্যবহার করে, তাই ১ কোটি row তেও দ্রুত। সংরক্ষণ ও খোঁজার আগে email lowercase করি, যাতে একটাই index সব ক্ষেত্রে কাজ করে। আংশিক খোঁজ (contains) লাগলে সাধারণ index leading wildcard এ কাজ করে না, তাই `pg_trgm` index বা full-text search। দরকারি ফিল্ডই `select` করি, ঘনঘন খোঁজা ডেটা Redis এ cache করি।

---

# Tier 3: মাঝে মাঝে আসে

## 23. What is eventual consistency? ⭐

> "Eventual consistency means that after a write, different copies of the data may not be the same for a short time, but if there are no new writes, all copies will become the same eventually. It is common in distributed systems, like read replicas or NoSQL databases, where speed and availability are chosen over instant consistency. Strong consistency is the opposite: every read sees the latest write. I use strong consistency for money and stock, and eventual consistency is fine for things like like counts or feeds."

**বাংলা মানে:** Write এর পর কিছুক্ষণ ডেটার বিভিন্ন কপি আলাদা থাকতে পারে, কিন্তু নতুন write না এলে শেষে সব কপি সমান হয়। Distributed system (read replica, NoSQL) এ সাধারণ, যেখানে তাৎক্ষণিক consistency এর বদলে গতি ও availability বেছে নেওয়া হয়। Strong consistency উল্টো: প্রতিটা read সর্বশেষ write দেখে। টাকা ও stock এ strong, like count বা feed এ eventual চলে।

---

## শেষ মুহূর্তের রিভিশন (এক লাইনে)

- **Prisma বনাম PostgreSQL:** টুল বনাম ডাটাবেস
- **Transaction:** সব নয়তো কিছুই না। ACID
- **N+1:** লুপে query। `include`/`in` দিয়ে সমাধান
- **Index:** পড়া দ্রুত, লেখা ধীর। Composite এ equality আগে
- **Overselling:** চেক ও কমানো একটাই atomic update (`stock >= qty`)
- **Duplicate:** app এর চেক যথেষ্ট না, DB তে unique constraint
- **Slow query:** `EXPLAIN ANALYZE`, Seq Scan খোঁজো
- **Production migration:** expand and contract, `migrate deploy`
- **onDelete:** Cascade / Restrict / SetNull
- **PG বনাম Mongo:** সম্পর্ক ও transaction → PG, নমনীয় document → Mongo
- **Normalization:** পরিষ্কার ডেটা। Denormalization: দ্রুত পড়া, অমিলের ঝুঁকি
