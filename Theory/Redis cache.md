
What is Cache?
============
Cache is a temporary storage layer that stores copies of frequently accessed data to reduce load on primary data base.

What is Redis?
==============

Redis is an in-memory key–value data store commonly used as:

Cache
Session store
Message broker
Leaderboard engine
In most backend systems, Redis is used as a caching layer in front of MySQL.
Why use Redis with MySQL?
MySQL:
Disk-based
Slower
Source of truth

Redis:
RAM-based
Extremely fast
Stores temporary copies
Result:

10x–50x faster response
Reduced DB load

Cache Flow (Cache-Aside Pattern)
Client → Redis → (miss) → MySQL → Redis → Client

Read Flow

Check Redis

If hit → return data

If miss → fetch from DB

Store in Redis with TTL

Return data

Write Flow

Update MySQL

Delete Redis key

Sample Java Implementation
Read
public User getUser(Long id) {
    String key = "user:" + id;

    User user = redisTemplate.opsForValue().get(key);
    if (user != null) {
        return user;
    }

    user = userRepository.findById(id).orElse(null);

    if (user != null) {
        redisTemplate.opsForValue().set(key, user, 10, TimeUnit.MINUTES);
    }

    return user;
}

Update
public User updateUser(User user) {
    User saved = userRepository.save(user);
    redisTemplate.delete("user:" + user.getId());
    return saved;
}

TTL (Time To Live)

Always set expiry on cache:

set(key, value, 5, TimeUnit.MINUTES);


Why?

Prevent stale data

Prevent memory overflow

Automatic cleanup

Cache Key Design

Bad:

"123"


Good:

"user:123"
"order:789"
"product:555"

What should be cached?

Good:

User profiles

Product details

Config data

Sessions

OTPs

Bad:

Payments

Financial data

Highly dynamic data

What if Redis is down?

System should:

Directly hit MySQL

Still work (slower)

This is called graceful degradation.

Common Cache Problems
Cache Stampede

Many requests hit DB when cache expires.

Solution:

Add random TTL

Use locks
=====================
Implementation
Redis Cache Implementation (Short)
Steps

Install Redis

Add Redis dependency in project

Configure Redis connection

On READ → check Redis first

On MISS → fetch from DB and store in Redis

On UPDATE/DELETE → remove cache

Always set TTL

That’s it. This is the full logic.

Code (Spring Boot + Redis + MySQL)
Read
public User getUser(Long id) {
    String key = "user:" + id;

    User user = redisTemplate.opsForValue().get(key);
    if (user != null) return user;

    user = userRepository.findById(id).orElse(null);

    if (user != null) {
        redisTemplate.opsForValue().set(key, user, 10, TimeUnit.MINUTES);
    }

    return user;
}

Update
public User updateUser(User user) {
    User saved = userRepository.save(user);
    redisTemplate.delete("user:" + user.getId());
    return saved;
}

Ultra-short explanation
Redis sits before DB
Reads hit Redis
Miss goes to DB
Writes clear Redis
TTL avoids stale data
Most important to remember (interview)
Annotation	Purpose
@EnableCaching	Activate caching
@Cacheable	Cache reads
@CacheEvict	Remove cache
@CachePut	Update cache
=========================
Where to use each caching annotation
1. @EnableCaching

Where:
On your main Spring Boot class or a config class.

@SpringBootApplication
@EnableCaching
public class Application { }


Use once per project.

2. @Cacheable

Where:
On service layer READ methods.

Example:

@Cacheable(value = "users", key = "#id")
public User getUser(Long id) {
    return userRepository.findById(id).orElse(null);
}


Use when:

Method fetches data

Data is reused

No side effects

Never use on:

Save/update methods

Methods with void return

3. @CacheEvict

Where:
On service layer UPDATE / DELETE methods.

Example:

@CacheEvict(value = "users", key = "#user.id")
public User updateUser(User user) {
    return userRepository.save(user);
}


Use when:

DB data changes

You must remove old cache

4. @CachePut

Where:
On update methods when you want to refresh cache immediately.

Example:

@CachePut(value = "users", key = "#user.id")
public User updateUser(User user) {
    return userRepository.save(user);
}


Use when:

You want cache to always have latest data

Not very common in real projects

Correct layer to use them

Always on:

Service layer

Never on:

Controller

Repository

Entity

Why?
Because service layer controls business logic.

Real-world mapping
Operation	Annotation
GET /user/{id}	@Cacheable
PUT /user	@CacheEvict
DELETE /user/{id}	@CacheEvict
PUT with response reuse	@CachePut
