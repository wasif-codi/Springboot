Redis Cache – Practical Notes
What is Cache?

Cache is a temporary high-speed storage layer that stores copies of frequently accessed data to reduce latency and load on primary data sources.

What is Redis?

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
