# System Design Interview Answers

ইন্টারভিউতে বলার জন্য ছোট ও সহজ ইংরেজি উত্তর, সাথে বাংলা মানে। বিস্তারিত বোঝার জন্য `system-design-details.md` পড়ো।

**কীভাবে পড়বে:** আগে Tier 1 (সবচেয়ে বেশি আসে), তারপর Tier 2।

| চিহ্ন | মানে |
|---|---|
| ⭐⭐⭐ | প্রায় প্রতিটা ইন্টারভিউতে আসে |
| ⭐⭐ | প্রায়ই আসে |
| ⭐ | মাঝে মাঝে আসে |

> **System design প্রশ্নের একটা সাধারণ কৌশল:** সরাসরি উত্তর না দিয়ে আগে ট্রেড-অফ বলো। যেমন "এটা নির্ভর করে... যদি X হয় তাহলে A, যদি Y হয় তাহলে B।" এতে বোঝা যায় তুমি ভেবে সিদ্ধান্ত নাও। তাই প্রতিটা উত্তরে "কখন কোনটা" অংশ রাখা হয়েছে।

---

# Tier 1: সবচেয়ে বেশি আসে

## 1. Monolith vs microservices. When would you choose a monolith? ⭐⭐⭐

> "A monolith is one application where all features are in one codebase and deployed together. Microservices split the application into small independent services, each with its own responsibility, and often its own database, and they talk to each other over the network. A monolith is simpler to build, test, deploy, and debug. Microservices give independent scaling and deployment, and let different teams work separately, but they add complexity: network calls, distributed data, and harder debugging. I would choose a monolith for a new product, a small team, or when the domain is not clear yet. I would move to microservices only when the team or the load really needs it. A good middle path is a modular monolith."

**বাংলা মানে:** Monolith হলো একটাই অ্যাপ, সব ফিচার এক codebase এ, একসাথে deploy। Microservices অ্যাপকে ছোট স্বাধীন সেবায় ভাগ করে, প্রতিটার নিজস্ব দায়িত্ব (প্রায়ই নিজস্ব ডাটাবেস), নেটওয়ার্কে কথা বলে। Monolith বানানো, টেস্ট, deploy, debug সহজ। Microservices এ আলাদা scaling ও deployment, আলাদা টিম কাজ করতে পারে, কিন্তু জটিলতা বাড়ে (নেটওয়ার্ক call, বিতরিত ডেটা, কঠিন debug)। নতুন প্রোডাক্ট, ছোট টিম, বা domain অস্পষ্ট হলে monolith। সত্যিই দরকার হলে তবেই microservices। মাঝামাঝি পথ: modular monolith।

---

## 2. Horizontal vs vertical scaling ⭐⭐⭐

> "Vertical scaling means making one server stronger by adding more CPU, RAM, or disk. It is simple, but it has a limit, it can be expensive, and it is still a single point of failure. Horizontal scaling means adding more servers and sharing the load between them. It has almost no limit and gives better availability, but the application must be stateless, and I need a load balancer. I usually start vertical because it is simple, and go horizontal when I reach limits or need high availability."

**বাংলা মানে:** Vertical scaling মানে একটা server কে শক্তিশালী করা (বেশি CPU, RAM)। সহজ, কিন্তু সীমা আছে, ব্যয়বহুল, আর তবু single point of failure। Horizontal scaling মানে আরও server যোগ করে চাপ ভাগ করা। প্রায় সীমাহীন, availability ভালো, কিন্তু অ্যাপ stateless হতে হয় ও load balancer লাগে। সাধারণত সহজতার জন্য vertical দিয়ে শুরু করি, সীমায় পৌঁছালে বা high availability লাগলে horizontal।

---

## 3. What is a load balancer and why do we need one? ⭐⭐⭐

> "A load balancer sits in front of several servers and distributes incoming requests between them. We need it for three reasons: to share the load so no server is overloaded, to improve availability, because it checks server health and stops sending traffic to a failed server, and to allow horizontal scaling and zero-downtime deployments. Common algorithms are round robin, least connections, and IP hash. Examples are Nginx, HAProxy, and AWS ALB. The load balancer itself must also not be a single point of failure, so it is usually run in a redundant setup."

**বাংলা মানে:** Load balancer কয়েকটা server এর সামনে বসে আসা request গুলো ভাগ করে দেয়। দরকার তিন কারণে: চাপ ভাগ করা, availability বাড়ানো (health check করে নষ্ট server এ request পাঠায় না), আর horizontal scaling ও zero-downtime deploy সম্ভব করা। সাধারণ algorithm: round robin, least connections, IP hash। উদাহরণ: Nginx, HAProxy, AWS ALB। Load balancer নিজেও single point of failure যেন না হয়, তাই redundant রাখা হয়।

---

## 4. What is caching? Where can it be implemented? What is Redis? ⭐⭐⭐

> "Caching means storing the result of expensive work in a fast place, so the next request can be served faster. It reduces latency and database load. I can cache in many layers: in the browser, in a CDN for static files and pages, in the application memory, in a shared cache like Redis, and in the database itself. Redis is an in-memory key-value store. It is very fast, supports data types like strings, hashes, lists, and sets, and supports expiry time. I use it for caching query results, storing sessions, rate limiting, queues, and leaderboards. The hard part of caching is invalidation: keeping the cache in sync with the data, so I use a TTL and delete or update the cache when data changes."

**বাংলা মানে:** Caching মানে ব্যয়বহুল কাজের ফল দ্রুত জায়গায় রাখা, যাতে পরের request দ্রুত পায়। এতে latency ও ডাটাবেসের চাপ কমে। অনেক স্তরে cache করা যায়: browser, CDN (static ফাইল/পেজ), অ্যাপের memory, shared cache (Redis), এমনকি ডাটাবেসেও। Redis হলো in-memory key-value store, খুব দ্রুত, string/hash/list/set সমর্থন করে, মেয়াদ (TTL) দেওয়া যায়। Query ফল cache, session, rate limiting, queue, leaderboard এ ব্যবহার করি। কঠিন অংশ invalidation: ডেটা বদলালে cache ঠিক রাখা, তাই TTL দিই ও ডেটা বদলালে cache মুছি/আপডেট করি।

---

## 5. How would you design an application to handle 100,000 concurrent users? ⭐⭐⭐

> "I would first clarify the numbers: concurrent users are not the same as requests per second, and the read to write ratio matters. Then I would build in layers. I put a CDN in front for static content. I use a load balancer with many stateless application servers that can auto-scale. I add a cache, like Redis, for hot data. For the database, I add the right indexes, connection pooling, and read replicas, and I use sharding only if one primary cannot handle the writes. I move slow work, like emails and reports, to a message queue. I add rate limiting to protect the system. For real-time features like WebSockets, I need a pub/sub backend so servers can share messages. Finally, I do load testing, and add monitoring and alerts, because I must measure before and after."

**বাংলা মানে:** আগে সংখ্যা পরিষ্কার করি: concurrent user আর requests per second এক না, আর read-write অনুপাত গুরুত্বপূর্ণ। তারপর স্তরে স্তরে বানাই: static কনটেন্টে CDN; load balancer এর পেছনে অনেক stateless server (auto-scale); গরম ডেটার জন্য Redis cache; ডাটাবেসে index, connection pooling, read replica (শুধু একটা primary লেখার চাপ না নিতে পারলে sharding); ধীর কাজ (email, রিপোর্ট) message queue তে; সুরক্ষায় rate limiting; WebSocket এর জন্য pub/sub যাতে server গুলো বার্তা ভাগ করতে পারে। শেষে load test, monitoring ও alert, কারণ আগে-পরে মাপতে হয়।

---

## 6. Authentication for a scalable app. Access token vs refresh token ⭐⭐⭐

> "For a scalable app, I want the servers to be stateless, so I use short-lived access tokens, like JWT, that any server can verify with a signature, without calling a shared store. The access token is valid for a short time, for example 15 minutes, so if it is stolen, the damage is limited. The refresh token lives longer, for example days or weeks. Its only job is to get a new access token. I store the refresh token in an `HttpOnly` secure cookie, keep a hashed copy in the database so I can revoke it, and rotate it on every use, so a stolen token is detected. Another option is server-side sessions stored in Redis, which are easy to revoke but need a shared store. For companies, I add SSO with OIDC."

**বাংলা মানে:** Scalable অ্যাপে server গুলো stateless রাখতে চাই, তাই স্বল্পমেয়াদী access token (যেমন JWT) ব্যবহার করি, যা যেকোনো server signature দিয়ে যাচাই করতে পারে, shared store ছাড়াই। Access token ছোট মেয়াদের (যেমন ১৫ মিনিট), চুরি হলে ক্ষতি সীমিত। Refresh token দীর্ঘ মেয়াদের (দিন/সপ্তাহ), শুধু নতুন access token আনার কাজে। Refresh token `HttpOnly` secure cookie তে রাখি, ডাটাবেসে hash করা কপি রাখি যাতে revoke করা যায়, প্রতিবার ব্যবহারে বদলে দিই (rotation), ফলে চুরি ধরা পড়ে। বিকল্প: Redis এ server-side session (revoke সহজ, কিন্তু shared store লাগে)।

---

## 7. How would you handle rate limiting for an API? ⭐⭐⭐

> "Rate limiting controls how many requests a client can make in a time window. It protects the system from abuse, bots, and overload, and keeps usage fair. I decide the key to limit by: IP address, user ID, or API key. Common algorithms are fixed window, sliding window, and token bucket, which allows short bursts. For multiple servers, the counters must be shared, so I store them in Redis, using atomic operations. When the limit is exceeded, I return HTTP 429 with a `Retry-After` header. I apply stricter limits on sensitive endpoints, like login and password reset, and I can do it at the API gateway or the load balancer, before the request reaches my application."

**বাংলা মানে:** Rate limiting একটা নির্দিষ্ট সময়ে একজন client কতগুলো request করতে পারবে তা নিয়ন্ত্রণ করে। এটা অপব্যবহার, bot ও ওভারলোড থেকে রক্ষা করে। কোন key দিয়ে সীমা দেব ঠিক করি (IP, user ID, API key)। Algorithm: fixed window, sliding window, token bucket (ছোট burst অনুমোদন করে)। একাধিক server এ counter শেয়ার করতে Redis এ atomic অপারেশনে রাখি। সীমা পেরোলে HTTP 429 ও `Retry-After` header দিই। login ও password reset এর মতো সংবেদনশীল endpoint এ কঠোর সীমা, আর API gateway বা load balancer এ করলে request অ্যাপে পৌঁছানোর আগেই আটকানো যায়।

---

## 8. What is a message queue? When would you use RabbitMQ/Kafka/SQS? ⭐⭐⭐

> "A message queue lets one part of the system, the producer, send a message, and another part, the consumer, process it later. This decouples the services and makes the system asynchronous. I use it to move slow work out of the request, like sending emails, processing images, and generating reports, to handle traffic spikes by buffering the work, and to retry failed jobs safely. RabbitMQ is a classic message broker, good for task queues and flexible routing. Kafka is a distributed event log, good for very high throughput, event streaming, and replaying events. SQS is a managed queue from AWS, simple and needs no servers to manage. Because delivery is usually at-least-once, my consumers must be idempotent, and I use a dead letter queue for messages that keep failing."

**বাংলা মানে:** Message queue এ একটা অংশ (producer) বার্তা পাঠায়, অন্য অংশ (consumer) পরে প্রসেস করে। এতে সেবাগুলো আলাদা হয় ও সিস্টেম asynchronous হয়। ধীর কাজ request থেকে সরাতে (email, ছবি প্রসেস, রিপোর্ট), ট্রাফিক spike এ কাজ জমিয়ে রাখতে, ব্যর্থ কাজ নিরাপদে retry করতে ব্যবহার করি। RabbitMQ ক্লাসিক broker (task queue, নমনীয় routing)। Kafka বিতরিত event log (খুব বেশি throughput, event streaming, event replay)। SQS হলো AWS এর managed queue (সহজ, server চালাতে হয় না)। সাধারণত at-least-once ডেলিভারি, তাই consumer idempotent হতে হয়, আর বারবার ব্যর্থ বার্তার জন্য dead letter queue।

---

## 9. What happens when one service in your system becomes unavailable? ⭐⭐⭐

> "I design for failure, because in a distributed system something will always fail. First, I set timeouts on every call, so a slow service does not block everything. I retry with exponential backoff and jitter, but only for safe, idempotent operations. I use a circuit breaker: after many failures, it stops calling the broken service for a while, so it can recover and my service does not waste resources. I provide a fallback or graceful degradation, for example showing cached data or hiding one feature, instead of failing the whole page. For non-urgent work, I put messages in a queue, so they are processed when the service comes back. I also run multiple instances with health checks, and use monitoring and alerts to detect the problem quickly."

**বাংলা মানে:** আমি ব্যর্থতা ধরে নিয়েই ডিজাইন করি, কারণ distributed সিস্টেমে কিছু না কিছু ব্যর্থ হবেই। প্রতিটা call এ timeout দিই, যাতে ধীর সেবা সবকিছু আটকে না দেয়। exponential backoff ও jitter সহ retry করি (শুধু নিরাপদ, idempotent কাজে)। Circuit breaker রাখি: অনেকবার ব্যর্থ হলে কিছুক্ষণ ওই সেবা call বন্ধ রাখে, সেবাটা সেরে উঠতে পারে ও আমার সম্পদ নষ্ট হয় না। Fallback বা graceful degradation দিই (cache থেকে দেখানো, একটা ফিচার লুকানো), পুরো পেজ ব্যর্থ না করে। জরুরি না এমন কাজ queue তে রাখি। একাধিক instance ও health check রাখি, monitoring ও alert দিয়ে দ্রুত ধরি।

---

# Tier 2: প্রায়ই আসে

## 10. What is system design? Functional vs non-functional requirements ⭐⭐

> "System design is the process of planning how the parts of a system, like services, databases, caches, and queues, work together to meet the requirements. It matters because a good design makes the system scalable, reliable, maintainable, and cost-effective, and it is much cheaper to fix design mistakes early. Functional requirements describe what the system does, like 'a user can place an order'. Non-functional requirements describe how well it does it, like performance, scalability, availability, security, and consistency. In an interview, I first ask questions to clarify both, because the non-functional requirements usually drive the design."

**বাংলা মানে:** System design হলো পরিকল্পনা করা যে সিস্টেমের অংশগুলো (সেবা, ডাটাবেস, cache, queue) কীভাবে একসাথে চাহিদা পূরণ করবে। এটা গুরুত্বপূর্ণ কারণ ভালো ডিজাইন সিস্টেমকে scalable, নির্ভরযোগ্য, রক্ষণাবেক্ষণযোগ্য ও সাশ্রয়ী করে, আর ডিজাইনের ভুল আগে ঠিক করা অনেক সস্তা। Functional requirement বলে সিস্টেম কী করে ("ব্যবহারকারী order দিতে পারে")। Non-functional বলে কতটা ভালোভাবে করে (গতি, scalability, availability, নিরাপত্তা, consistency)। ইন্টারভিউয়ে আগে দুটোই প্রশ্ন করে পরিষ্কার করি, কারণ non-functional চাহিদাই সাধারণত ডিজাইন ঠিক করে।

---

## 11. Explain the basic flow: Client → API → Business Logic → Database ⭐⭐

> "The client, like a browser or a mobile app, sends an HTTP request. The API layer receives it, handles routing, authentication, and input validation, and passes it on. The business logic layer, often called the service layer, applies the rules of the application, like checking stock and calculating prices. The data layer then reads or writes the database. The response goes back the same way. Keeping these layers separate makes the code easier to test, change, and scale. In a real system, I would add a CDN, a load balancer, and a cache between these parts."

**বাংলা মানে:** Client (browser/মোবাইল অ্যাপ) HTTP request পাঠায়। API স্তর সেটা নেয়, routing, authentication ও input validation করে, তারপর পাঠায়। Business logic স্তর (service layer) অ্যাপের নিয়ম প্রয়োগ করে (stock দেখা, দাম হিসাব)। Data স্তর ডাটাবেস পড়ে বা লেখে। Response একই পথে ফেরে। স্তর আলাদা রাখলে কোড টেস্ট, বদল ও scale করা সহজ। বাস্তবে এর মাঝে CDN, load balancer ও cache যোগ করি।

---

## 12. Stateless vs stateful architecture ⭐⭐

> "In a stateless architecture, the server does not remember anything about the client between requests. Every request carries all the information needed, like a token. Any server can handle any request, so it is easy to scale horizontally and to recover from a failure. In a stateful architecture, the server keeps client data, like a session, in its own memory, so the same client must go to the same server, which makes scaling and failover harder. To keep servers stateless, I move the state out to a shared store, like Redis or the database, or into a token. Some things, like WebSocket connections, are naturally stateful, so I use sticky sessions or a pub/sub layer."

**বাংলা মানে:** Stateless এ server request এর মাঝে client সম্পর্কে কিছু মনে রাখে না। প্রতিটা request এ দরকারি সব তথ্য থাকে (যেমন token)। যেকোনো server যেকোনো request সামলাতে পারে, তাই horizontal scale ও ব্যর্থতা থেকে পুনরুদ্ধার সহজ। Stateful এ server নিজের memory তে client এর ডেটা (session) রাখে, তাই একই client কে একই server এ যেতে হয়, scale ও failover কঠিন। Server stateless রাখতে state shared store (Redis/ডাটাবেস) বা token এ সরাই। WebSocket স্বভাবে stateful, তাই sticky session বা pub/sub স্তর ব্যবহার করি।

---

## 13. What is database replication? ⭐⭐

> "Replication means keeping copies of the same data on several database servers. Usually there is one primary that accepts writes, and one or more replicas that copy the data and serve reads. It gives three benefits: read scaling, because I can send reads to replicas, high availability, because a replica can be promoted if the primary fails, and backups and safety. The trade-off is replication lag: replicas can be a little behind, so a read right after a write may return old data. This is eventual consistency. I handle it by reading from the primary for data the user just changed, or when I need the latest value."

**বাংলা মানে:** Replication মানে একই ডেটার কপি কয়েকটা ডাটাবেস server এ রাখা। সাধারণত একটা primary (লেখা নেয়) ও এক বা একাধিক replica (ডেটা কপি করে ও পড়ার request সামলায়)। তিন সুবিধা: পড়ার scale (read replica তে পাঠানো), high availability (primary নষ্ট হলে replica কে primary বানানো), নিরাপত্তা/backup। Trade-off হলো replication lag: replica একটু পিছিয়ে থাকতে পারে, তাই write এর পরপরই read এ পুরনো ডেটা আসতে পারে (eventual consistency)। ব্যবহারকারী যে ডেটা এইমাত্র বদলেছে বা সর্বশেষ মান লাগলে primary থেকে পড়ি।

---

## 14. What is database sharding? ⭐⭐

> "Sharding means splitting a large database into smaller parts called shards, and spreading them across several servers, so that each server holds only part of the data. It is horizontal scaling for the database, and it helps when one server cannot handle the data size or the write load, because replication only scales reads. The shard key decides where each row goes, for example the user ID. A good shard key spreads the data evenly. Sharding has real costs: cross-shard queries and joins are hard, transactions across shards are complex, resharding is painful, and a bad key creates a hot shard. So I try other options first: indexes, caching, read replicas, and partitioning, and I shard only when really needed."

**বাংলা মানে:** Sharding মানে বড় ডাটাবেসকে ছোট অংশে (shard) ভেঙে কয়েকটা server এ ছড়িয়ে দেওয়া, প্রতিটা server ডেটার একটা অংশ রাখে। এটা ডাটাবেসের horizontal scaling। যখন একটা server ডেটার আকার বা লেখার চাপ সামলাতে পারে না তখন দরকার, কারণ replication শুধু পড়া scale করে। Shard key ঠিক করে কোন row কোথায় যাবে (যেমন user ID), ভালো key ডেটা সমানভাবে ছড়ায়। খরচও আছে: shard এর মধ্যে query/join কঠিন, transaction জটিল, resharding কষ্টকর, খারাপ key এ একটা shard অতিরিক্ত চাপে পড়ে। তাই আগে index, cache, read replica, partitioning চেষ্টা করি, সত্যিই দরকার হলে তবেই shard।

---

## 15. SQL vs NoSQL from a system-design perspective ⭐⭐

> "SQL databases, like PostgreSQL, have a fixed schema, support joins and strong ACID transactions, and are the best choice for structured data with relations and strong consistency, like orders and payments. They usually scale vertically and with read replicas, and sharding is harder. NoSQL databases, like MongoDB, Cassandra, and DynamoDB, have flexible schemas and are built for horizontal scaling and very high write volume. They often give up some joins or strict consistency, which I accept for things like feeds, logs, and sessions. The choice depends on the access patterns, consistency needs, and scale. Many systems use both: this is called polyglot persistence."

**বাংলা মানে:** SQL ডাটাবেসে (PostgreSQL) schema নির্দিষ্ট, join ও শক্তিশালী ACID transaction আছে, সম্পর্কযুক্ত ও শক্ত consistency লাগা ডেটার (order, payment) জন্য সেরা। সাধারণত vertical ও read replica দিয়ে scale হয়, sharding কঠিন। NoSQL (MongoDB, Cassandra, DynamoDB) এ schema নমনীয়, horizontal scaling ও খুব বেশি লেখার জন্য তৈরি। এগুলো প্রায়ই কিছু join বা কঠোর consistency ছাড়ে, যা feed, log, session এর জন্য মানা যায়। বেছে নেওয়া নির্ভর করে access pattern, consistency চাহিদা ও scale এর উপর। অনেক সিস্টেম দুটোই ব্যবহার করে (polyglot persistence)।

---

## 16. What is a database index and how does it improve performance? ⭐⭐

> "An index is a data structure, usually a B-tree, that lets the database find rows without scanning the whole table, like the index of a book. Instead of checking every row, it goes straight to the right place, so a search becomes much faster, especially on large tables. The cost is that indexes use storage and slow down writes, because they must be updated too. So I create indexes on the columns I use in filters, joins, and sorting, based on real query patterns, and I check with `EXPLAIN` that they are used."

**বাংলা মানে:** Index হলো ডেটা কাঠামো (সাধারণত B-tree), যা পুরো টেবিল না ঘেঁটে row খুঁজে দেয়, বইয়ের সূচিপত্রের মতো। প্রতিটা row না দেখে সরাসরি সঠিক জায়গায় যায়, তাই বড় টেবিলে খোঁজ অনেক দ্রুত। খরচ: জায়গা লাগে ও লেখা ধীর হয়। তাই আসল query দেখে filter, join ও sort এর column এ index দিই, আর `EXPLAIN` দিয়ে ব্যবহার হচ্ছে কিনা দেখি।

---

## শেষ মুহূর্তের রিভিশন (এক লাইনে)

- **Monolith বনাম microservices:** সহজ বনাম স্বাধীন। শুরুতে monolith / modular monolith
- **Vertical বনাম horizontal:** শক্তিশালী server বনাম আরও server। Horizontal এ stateless দরকার
- **Load balancer:** চাপ ভাগ, health check, availability
- **Cache:** দ্রুত জায়গায় ফল রাখা। Redis। কঠিন অংশ: invalidation
- **১ লক্ষ user:** CDN + LB + stateless server + cache + replica + queue + rate limit + load test
- **Auth:** ছোট access token + দীর্ঘ refresh token (HttpOnly cookie, rotation)
- **Rate limit:** token bucket/sliding window, Redis, HTTP 429
- **Queue:** ধীর কাজ আলাদা, idempotent consumer, DLQ
- **Service নষ্ট:** timeout, retry+backoff, circuit breaker, fallback
- **Replication:** পড়া scale + availability, lag এর ঝুঁকি
- **Sharding:** লেখা scale, জটিল, শেষ উপায়
- **ইন্টারভিউয়ে:** আগে প্রশ্ন করে চাহিদা পরিষ্কার, তারপর ট্রেড-অফ সহ উত্তর
