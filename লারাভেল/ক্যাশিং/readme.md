## ওয়েব অ্যাপ্লিকেশনে ক্যাশি
### ক্যাশিং কী?
ক্যাশিং হলো বারবার দরকার হয় এমন ডেটা বা রেজাল্ট একটি দ্রুত স্টোরেজে (সাধারণত RAM) সাময়িকভাবে রেখে দেওয়া, যাতে প্রতিবার ডাটাবেস বা ধীর সোর্স থেকে আনতে না হয়।
এতে অ্যাপ্লিকেশনের পারফরম্যান্স ও দক্ষতা উল্লেখযোগ্যভাবে বাড়ে এবং মূল ডেটা সোর্সের ওপর লোড কমে।

প্রতিবার যদি database থেকে query চালানো হয়:
```
Browser
   ↓
Laravel
   ↓
MySQL
   ↓
Query
   ↓
Result
   ↓
Laravel
   ↓
Browser
```
যদি একই dashboard হাজারবার request পায়, database-কে একই কাজ হাজারবার করতে হবে।
Cache ব্যবহার করলে:
```
Browser
   ↓
Laravel
   ↓
Redis Cache
   ↓
Data পাওয়া গেল
   ↓
Browser
```
Database-এ যাওয়ার প্রয়োজন নেই।
#
### Cache কেন ব্যবহার করব
**Caching-এর প্রধান উদ্দেশ্য:**
- **Application দ্রুত করা:**
Redis memory থেকে data দেয়, তাই database-এর তুলনায় সাধারণত অনেক দ্রুত।
- **Database load কমানো:**
একই query বারবার database-এ চালানোর প্রয়োজন হয় না।
- **Server resource বাঁচানো:**
Database CPU, RAM এবং connection কম ব্যবহার করে।
- **High traffic handle করা:** 
একই data অনেক user চাইলে cache খুব useful।
- **Expensive calculation কমানো:**
যেমন:
  - Dashboard statistics
  - Reports
  - Aggregations
  - API response
  - Large dropdown data
  - Permission data
  - Configuration
#
### ক্যাশিং এর ধরন (প্রকারভেদ)
ক্যাশিং-এর কোনো একটি নির্দিষ্ট সংখ্যা নেই। লেয়ার বা অবস্থান অনুযায়ী সাধারণত নিচের ৮টি ধরন ধরা হয়:
**উদাহরণ**
- **Browser Cache:** ইউজারের ব্রাউজারে (CSS, JS, ছবি)
- **CDN Cache:** বিশ্বব্যাপী এজ সার্ভারে (Cloudflare, CloudFront)
- **Reverse Proxy / Web Server Cache:** সার্ভারের সামনে (Nginx, Varnish)
- **Application Cache (In-Memory):** অ্যাপের নিজের মেমরিতে (Node.js Map, Caffeine (Java))
- **Distributed Cache:** আলাদা ক্যাশ সার্ভারে (Redis, Memcached)
- **Database Cache:** ডাটাবেস ইঞ্জিনে (Query cache, Buffer pool)
- **Object / Query Result Cache:** অ্যাপ লেয়ারে (ORM cache, API response cache)
- **DNS Cache:** OS/ISP/ব্রাউজারে (ডোমেইন → IP)
এছাড়া Page/Fragment Cache (পুরো পেজ বা অংশবিশেষ) এবং Service Worker Cache (PWA অফলাইন সাপোর্ট) ও আছে।
#
### ওয়েব অ্যাপ্লিকেশনে ক্যাশিংয়ের স্তর ও সংখ্যা(deepseek)
ওয়েব অ্যাপ্লিকেশনে ক্যাশিং সাধারণত ৫ থেকে ৬টি স্তরে বিভক্ত। প্রতিটি স্তরে ক্যাশ হিট হলে নিচের স্তরগুলোতে যেতে হয় না, ফলে লেটেন্সি অনেক কমে যায়।

## ক্যাশিং স্তরের তুলনা (Caching Layers Comparison)

| **স্তর** | **সাধারণ লেটেন্সি** | **ধারণক্ষমতা** | **ইনভ্যালিডেশন পদ্ধতি** |
|---|---|---|---|
| ব্রাউজার ক্যাশ (Browser Cache) | ০ ms (তাৎক্ষণিক) | ৫০–১০০ MB | `Cache-Control` হেডার |
| CDN (Content Delivery Network) | ১০–৫০ ms | প্রতি PoP-এ টেরাবাইট পর্যায়ে | API কল বা TTL |
| রিভার্স প্রক্সি (Varnish) | ১–৫ ms | গিগাবাইট পর্যায়ে | ম্যানুয়াল পার্জ বা TTL |
| অ্যাপ্লিকেশন ক্যাশ (Redis/Memcached) | ১–৩ ms | শত শত গিগাবাইট পর্যন্ত | `DEL` কমান্ড বা TTL |
| ডেটাবেস বাফার পুল (Database Buffer Pool) | প্রায় ০.১ ms | উপলব্ধ RAM-নির্ভর | স্বয়ংক্রিয় ব্যবস্থাপনা ও LRU-জাতীয় নীতি |

**নোট:** লেটেন্সি ও ধারণক্ষমতা আনুমানিক। বাস্তব পারফরম্যান্স হার্ডওয়্যার, নেটওয়ার্ক, ডেটার আকার এবং কনফিগারেশনের ওপর নির্ভর করে। ডেটাবেস বাফার পুলের eviction policy ডেটাবেস ইঞ্জিনভেদে আলাদা হতে পারে।

ব্রাউজার থেকে শুরু করে ডেটাবেস পর্যন্ত প্রতিটি স্তরে ক্যাশ থাকতে পারে। যে স্তরে হিট হবে, সেই স্তর থেকেই ডেটা সার্ভ হবে, নিচের স্তরগুলোতে যেতে হবে না
#
### সেরা ক্যাশিং সিস্টেম কোনটি
"সবচেয়ে ভালো" বলে কিছু নেই। এটা আপনার প্রয়োজনের ওপর নির্ভর করে। তবে বাস্তবে সবচেয়ে বেশি ব্যবহৃত ও সেরা বিকল্প হলো Redis।

| **বিষয়** | **Redis** | **Memcached** | **In-Memory (অ্যাপের ভেতর)** | **CDN** |
|---|---|---|---|---|
| ডেটা স্ট্রাকচার | List, Set, Hash, Sorted Set, Stream | শুধু key-value | যেকোনো অবজেক্ট | ফাইল/HTTP রেসপন্স |
| পারসিস্টেন্স | আছে (RDB/AOF) | নেই | নেই | নেই |
| ক্লাস্টারিং | আছে | ক্লায়েন্ট-সাইড| নেই | বিল্ট-ইন |
| Pub/Sub | আছে | নেই | নেই | নেই |
| গতি | অত্যন্ত দ্রুত | অত্যন্ত দ্রুত | সবচেয়ে দ্রুত | লোকেশনের ওপর নির্ভর
|কাজের ক্ষেত্র | সেশন, ক্যাশ, কিউ, লিডারবোর্ড | সাধারণ সিম্পল ক্যাশ | ছোট অ্যাপ | স্ট্যাটিক কনটেন্ট |

**সুপারিশ:**
- ছোট অ্যাপ / একটি সার্ভার: In-memory ক্যাশ যথেষ্ট
- মাঝারি থেকে বড় অ্যাপ: Redis (সেরা অল-রাউন্ডার)
- স্ট্যাটিক ফাইল ও গ্লোবাল ইউজার: CDN (Cloudflare ইত্যাদি)
- বাস্তবে সেরা সেটআপ: CDN + Browser Cache + Redis একসাথে (Multi-layer caching)

**Recommended architecture:**
```
                    ┌───────────────┐
                    │    Browser    │
                    └───────┬───────┘
                            │
                            ↓
                    ┌───────────────┐
                    │    Laravel    │
                    └───────┬───────┘
                            │
                     Cache Check
                            │
                    ┌───────▼───────┐
                    │     Redis     │
                    └───────┬───────┘
                            │
                         Cache Miss
                            │
                    ┌───────▼───────┐
                    │     MySQL     │
                    └───────────────┘
```
#
### ক্যাশিং কীভাবে কাজ করে 
```
ইউজার রিকোয়েস্ট
      │
      ▼
 ক্যাশে ডেটা আছে? ──── হ্যাঁ (Cache Hit) ──► সাথে সাথে রেসপন্স
      │
     না (Cache Miss)
      │
      ▼
 ডাটাবেস থেকে ডেটা আনো
      │
      ▼
 ক্যাশে সেভ করো (TTL সহ)
      │
      ▼
   রেসপন্স দাও
```
#
### প্রধান ক্যাশিং স্ট্র্যাটেজি
- **Cache-Aside (Lazy Loading):** সবচেয়ে জনপ্রিয়। অ্যাপ প্রথমে ক্যাশ দেখে, না পেলে DB থেকে এনে ক্যাশে রাখে।
- **Write-Through:** ডেটা লেখার সময় একসাথে DB ও ক্যাশ দুটোতেই লেখা হয়। ডেটা সবসময় সামঞ্জস্যপূর্ণ থাকে, তবে লেখা একটু ধীর।
- **Write-Behind (Write-Back):** আগে ক্যাশে লেখা হয়, পরে অ্যাসিঙ্ক্রোনাসলি DB-তে যায়। লেখা দ্রুত, তবে ক্র্যাশ হলে ডেটা হারানোর ঝুঁকি আছে।
- **Read-Through:** ক্যাশ লেয়ারই নিজে DB থেকে ডেটা আনে। অ্যাপ শুধু ক্যাশের সাথে কথা বলে।
Eviction Policy (ক্যাশ ভরে গেলে কী মুছবে)
- **LRU (Least Recently Used):** সবচেয়ে পুরনো ব্যবহৃত ডেটা মুছে যায়। সবচেয়ে জনপ্রিয়
- **LFU (Least Frequently Used):** সবচেয়ে কম ব্যবহৃত ডেটা মুছে যায়
- **FIFO:** যেটা আগে ঢুকেছে সেটা আগে বের হয়
- **TTL:** নির্দিষ্ট সময় পর স্বয়ংক্রিয়ভাবে মেয়াদ শেষ হয়
 #
### Redis বনাম Database Cache

Database Cache
CACHE_STORE=database
**Architecture:**
```
Laravel
   ↓
MySQL Cache Table
```

এখানে cache এবং application database একই database infrastructure ব্যবহার করতে পারে।
**সুবিধা**

- Redis install করতে হয় না
- Simple Small application-এর জন্য যথেষ্ট

**অসুবিধা**
Cache-এর উদ্দেশ্যই হলো database-এর উপর load কমানো।
কিন্তু database cache ব্যবহার করলে:
```
Laravel
   ↓
Database Cache
   ↓
MySQL
```
অর্থাৎ cache access-ও database-এর মাধ্যমে হচ্ছে।
#
## কখন Database Cache ব্যবহার করা উচিত

Database cache ব্যবহার যদি:
- Small application
- Low traffic
- Redis available নয়
- Simple deployment
- Single server
আপনার application development/local environment-এ এটি ঠিক আছে।

## কখন Redis ব্যবহার করা উচিত
application এ যদি:

- High traffic
- Large dashboard
- Heavy reports
 - Many users
- Queues
- Broadcasting
- Real-time operations
- Frequent cache reads
- Distributed servers
তাহলে Redis ব্যবহার করা ভালো।

CRM-এর জন্য Redis বিশেষভাবে useful কারণ আপনার application-এ:

- Dashboard
- Lead reports
- Task reports
- Lead lists
- Queues
- Scheduled jobs
- Broadcasting
এসব থাকে।
#
### Cache-কে Database-এর মতো ভাবা উচিত?

একটি গুরুত্বপূর্ণ principle:

MySQL = Source of Truth

Redis = Temporary Fast Copy
অর্থাৎ:
```
MySQL
  ↓
Truth
Redis
  ↓
Performance layer
```
Redis-এর data হারিয়ে গেলেও application ideally database থেকে আবার data তৈরি করতে পারবে।
#
### Multi-Level Cache Architecture
**Advanced application:**
```
               Request
                   ↓
             Browser Cache
                   ↓
              CDN Cache
                   ↓
          Application Memo Cache
                   ↓
                Redis
                   ↓
                MySQL
```
সব application-এর জন্য সব layer প্রয়োজন হয় না।
CRM-এর জন্য:
```
Browser
   ↓
Laravel
   ↓
Redis
   ↓
MySQL
```
দিয়ে শুরু করাই যথেষ্ট।
#
### Cache::flush() সাবধানে ব্যবহার করা

**Cache::flush();**
এটি configured prefix-কে respect নাও করতে পারে এবং shared cache store-এ অন্য application-এর cache-ও remove করতে পারে। Laravel documentation এই বিষয়ে explicit warning দেয়।
তাই production-এ:
Cache::flush();
এর পরিবর্তে targeted invalidation ভালো।
#
### Cache Strategy

একটি ভালো strategy:
```
                    ┌─────────────┐
                    │   Request   │
                    └──────┬──────┘
                           ↓
                    Generate Cache Key
                           ↓
                    ┌──────▼──────┐
                    │    Redis    │
                    └──────┬──────┘
                           │
                  ┌────────┴────────┐
                  │                 │
                 HIT              MISS
                  │                 │
                  ↓                 ↓
               Return           MySQL Query
                                    │
                                    ↓
                              Save to Redis
                                    │
                                    ↓
                                  Return
```
#
### Write Flow
Database write-এর সময়:
```
User Update
    ↓
Laravel
    ↓
MySQL UPDATE
    ↓
Cache::forget()
    ↓
Redis cache removed
পরের read:
Redis MISS
    ↓
  MySQL
    ↓
Redis SET
```
এটাই সাধারণ এবং safe cache-aside approach।
#
### Redis Down হলে কী হবে-

Application ideally completely crash করা উচিত নয় শুধু cache-এর জন্য।
**Architecture:**
```
Redis available
    ↓
Use Cache
Redis unavailable হলে:
Fallback
    ↓
Database
```
Laravel configuration এবং application design অনুযায়ী fallback strategy রাখা যায়।
তবে Redis যদি queue/broadcasting-এরও dependency হয়, তখন Redis outage-এর impact আলাদা হবে।
#### Cache Monitoring
Production-এ শুধু cache implement করলেই হবে না। Monitor করতে হবে:
- Cache hit rate
- Cache miss rate
- Memory usage
- Evictions
- TTL
- Key count
- Slow queries
- Redis CPU
- Redis connections
- Database load
#
### Cache Hit Ratio
ধরুন: 

1000 requests এর মধ্যে:

900 cache hit

100 cache miss তাহলে:

Cache Hit Ratio = 90% এটি ভালো।
কিন্তু সব application-এর জন্য নির্দিষ্ট "ideal" percentage নেই।
যদি cache miss-এর কারণে expensive query হয়, hit rate যত বেশি হবে সাধারণত তত ভালো।
#
### Cache Monitoring-এর সময় যে প্রশ্নগুলো করতে হবে-

প্রতিটি cache-এর জন্য:

- Why are we caching?
- What is the TTL?
- What is the key?
- Who owns invalidation?
- What happens on miss?
- What happens when Redis is down?
- Can stale data be tolerated?
- How large is the cached value?
- How frequently is it accessed?
#
### Caching-এর Golden Rules
- Never cache blindly.
- Always design cache key carefully.
- Always think about invalidation.
- User-specific data must have user context.
- Permission-sensitive data must have permission context.
- Don't cache huge datasets.
- Use TTL.
- MySQL remains source of truth.
- Use Redis for high-performance application caching.
- Measure before and after caching.
#
### এক লাইনে পুরো Caching Concept
Cache = Frequently used data-এর একটি দ্রুত temporary copy,যা database-এর load কমিয়ে application-এর response দ্রুত করে।
আর production architecture:
```
                READ
                  │
                  ↓
               Redis
              /     \
            HIT     MISS
             │        │
             ↓        ↓
          Return    MySQL
                       │
                       ↓
                    Redis
                       │
                       ↓
                    Return


                 WRITE
                   │
                   ↓
                 MySQL
                   │
                   ↓
              Invalidate
                   │
                   ↓
                 Redis
```
#
### Cache Invalidation (সবচেয়ে কঠিন অংশ)-

পুরনো ডেটা (stale data) যেন ইউজার না দেখে, সেজন্য:

- TTL দেওয়া: মেয়াদ শেষে নিজে মুছে যায়
- Manual invalidation: ডেটা বদলালে redis.del(key)
- Versioned keys: user:v2:5 (ভার্সন বদলালে নতুন key)
- Event-based: Pub/Sub দিয়ে সব সার্ভারকে জানানো
#
### সাধারণ সমস্যা ও সমাধান

| **সমস্যা** | **বর্ণনা** | **সমাধান** |
|---|---|---|
| Cache Stampede | একটি জনপ্রিয় key মেয়াদ শেষ হলে হাজারো রিকোয়েস্ট একসাথে DB-তে যায় | Lock/mutex, TTL-এ random jitter যোগ |
| Cache Penetration | যে ডেটা নেই তা বারবার চাওয়া হয়, সবসময় DB-তে যায় | খালি রেজাল্টও ক্যাশ করা, Bloom Filter |
| Stale Data | পুরনো ডেটা দেখানো | সঠিক invalidation ও ছোট TTL |
| Hot Key | একটি key-তে অতিরিক্ত লোড | লোকাল ক্যাশ + Redis একসাথে |
#
### Best Practices

- কী ক্যাশ করব: বেশি পড়া হয় কিন্তু কম বদলায় এমন ডেটা (প্রোডাক্ট লিস্ট, কনফিগ, প্রোফাইল)
- কী ক্যাশ করব না: ব্যক্তিগত সংবেদনশীল ডেটা (যদি shared cache হয়), রিয়েল-টাইম ব্যালেন্স
- সবসময় TTL সেট করুন
- Key naming convention মানুন: entity:id:field
- ক্যাশ হিট রেট মনিটর করুন (Redis: INFO stats)
- Redis ডাউন হলে অ্যাপ যেন DB থেকে চলতে পারে (graceful fallback)
- Redis-এ maxmemory ও maxmemory-policy allkeys-lru সেট করা
আগে মেজার করে, তারপর ক্যাশ করা উচিত। অকারণে ক্যাশ জটিলতা বাড়ায়
#
### সাজেস্টেড আর্কিটেকচার (প্রোডাকশন)
```
ইউজার
  │
  ▼
Browser Cache
  │
  ▼
CDN (Cloudflare) ── স্ট্যাটিক ফাইল
  │
  ▼
Nginx (Reverse Proxy Cache)
  │
  ▼
Application Server ── In-memory cache (খুব হট ডেটা)
  │
  ▼
Redis (Distributed Cache)
  │
  ▼
Database (PostgreSQL/MySQL)
```
#
### সারসংক্ষেপ

**ক্যাশিং মূলত ৮টি লেয়ারে হয়:** Browser, CDN, Proxy, App, Distributed, DB, Object, DNS
Redis সবচেয়ে বহুমুখী ও জনপ্রিয় ক্যাশ সিস্টেম
সবচেয়ে প্রচলিত প্যাটার্ন হলো Cache-Aside + TTL + Invalidation
সেরা পারফরম্যান্সের জন্য একাধিক লেয়ার একসাথে ব্যবহার করুন
