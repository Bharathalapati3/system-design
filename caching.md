# Caching

## Overview

Caching is a technique for storing frequently accessed data in a fast, temporary storage layer to reduce latency and database load. It's essential for building scalable systems.

### Performance Impact
- **Cache**: ~1ms latency
- **Database**: ~100ms latency
- **Network**: Can range from 10ms to 1000ms+

Caching can reduce response times by 100x and significantly decrease database pressure.

---

## Caching Strategies

### Cache-Aside (Lazy Loading)
- **How it works**: 
  - Check cache first; if miss, fetch from DB
  - Write data to cache for future requests
- **Best for**: Read-heavy workloads, intermittent data access
- **Pros**: Simple, only cache accessed data
- **Cons**: Cache misses cause DB queries, stale data possible

### Write-Through
- **How it works**: 
  - Write to cache AND database simultaneously
  - Read from cache
- **Best for**: Consistency-critical applications
- **Pros**: Data always consistent, no stale data
- **Cons**: Higher latency on writes, double writes

### Write-Behind (Write-Back)
- **How it works**: 
  - Write to cache immediately
  - Asynchronously persist to database
- **Best for**: Write-heavy workloads, high throughput systems
- **Pros**: Fast writes, reduced DB load
- **Cons**: Data loss risk if cache fails, complexity in consistency

### Write-Around
- **How it works**: 
  - Write directly to database, bypass cache
  - Only cache on read misses
- **Best for**: Large writes that won't be read soon
- **Pros**: Prevents cache pollution, reduces write overhead
- **Cons**: Read misses until data accessed

---

## Eviction Policies

### LRU (Least Recently Used)
- **How it works**: Evict the least recently accessed item when cache is full
- **Best for**: General-purpose caching, time-based locality
- **Pros**: Simple, effective for most workloads
- **Cons**: Doesn't consider access frequency

### LFU (Least Frequently Used)
- **How it works**: Evict the least frequently accessed item
- **Best for**: Long-term performance optimization
- **Pros**: Keeps hot data in cache
- **Cons**: Higher computational overhead, harder to implement

### FIFO (First In First Out)
- **How it works**: Evict the oldest entry regardless of access
- **Best for**: Simple use cases, time-series data
- **Pros**: Minimal overhead
- **Cons**: May evict still-useful data

### Random
- **How it works**: Randomly evict an entry
- **Best for**: When eviction policy doesn't matter much
- **Pros**: No overhead
- **Cons**: Unpredictable performance

---

## Cache Invalidation Patterns

### TTL (Time-To-Live)
- Automatically expire cache entries after a set duration
- **Best for**: Data that changes predictably
- **Trade-offs**: 
  - Short TTL = more DB hits, fresher data
  - Long TTL = stale data risk, less DB load

### Active Invalidation
- **Best approach for consistency**:
  1. On write to database, delete cache key
  2. On next read, cache miss → fetch from DB → repopulate cache
- **Pros**: Ensures data freshness, prevents stale reads
- **Cons**: Brief period of cache misses after writes

### Event-Based Invalidation
- Use pub/sub or message queues to trigger cache invalidation across services
- **Best for**: Microservices, distributed systems
- **Pros**: Coordinated invalidation across multiple services
- **Cons**: Added complexity, eventual consistency

---

## Cache Problems & Solutions

### Cache Stampede (Thundering Herd)
**Problem:**
- Very commonly used key expires
- 1000+ requests hit the database simultaneously
- Causes DB overload and spike in latency

**Solutions:**
1. **Mutex Lock / Probabilistic Early Expiration**
   - Only 1 request fetches from DB while others wait
   - Others receive cached value or retry
2. **Cache Warming**: Pre-populate cache before expiration
3. **Delayed Expiration**: Set staggered TTLs for related keys

### Cache Penetration
**Problem:**
- Attacker/user repeatedly requests data that doesn't exist in cache or database
- Every request bypasses cache and hits database
- Can exhaust database resources

**Solutions:**
1. **Bloom Filters** (most common)
   - Efficiently track non-existent keys
   - Prevent unnecessary DB lookups for non-existent data
2. **Cache Negative Results**: Store empty/null responses with shorter TTL
3. **Rate Limiting**: Limit requests from same IP/user
4. **Input Validation**: Reject obviously invalid requests early

### Cache Avalanche
**Problem:**
- Many cache keys expire at the same time
- All queries hit the database simultaneously
- Causes sudden DB load spike and system slowdown

**Solutions:**
1. **Randomized TTLs**: Set different expiration times for similar keys
   - E.g., TTL = base_ttl + random(0, variance)
2. **Cache Warming**: Pre-populate cache before predicted expiration
3. **Circuit Breaker**: Stop queries when DB is overwhelmed, return graceful errors
4. **Load Shedding**: Temporarily increase cache TTL or reject requests

---

## Cache Layers & Tools

### In-Memory Caching
- **Redis**: Most popular, supports data structures, pub/sub, persistence
- **Memcached**: Simple key-value, extremely fast, no persistence
- **Use when**: Need fast access, acceptable data loss on restart

### Browser/Client Caching
- HTTP cache headers (Cache-Control, ETag, Last-Modified)
- **Use when**: Static assets, API responses

### Database Query Caching
- Query result caching at database level
- **Use when**: Expensive queries, read-heavy workloads

### CDN Caching (Content Delivery Network)
- **Examples**: CloudFlare, Akamai, AWS CloudFront
- **Use when**: Geographic distribution, static content, global scale

---

## Best Practices

### Set TTL Always
- Every cache entry should have an expiration time
- Prevents stale data indefinitely consuming resources
- Choose TTL based on data freshness requirements

### Monitor Cache Metrics
- **Hit Ratio**: (Hits / Total Requests) - Target: >80%
- **Miss Ratio**: (Misses / Total Requests)
- **Eviction Rate**: How often items are evicted
- **Memory Usage**: Monitor cache size and growth

### Use Consistent Cache Keys
- Include version info, user ID, parameters in key
- Example: `user:{userId}:profile:v1`
- Prevents collisions and makes debugging easier

### Avoid Cache Stampede with Locks
```
function getUser(userId):
  data = cache.get("user:" + userId)
  if data exists:
    return data
  
  // Try to acquire lock
  if cache.setNX("lock:user:" + userId, true, 10s):
    // Got lock - fetch from DB
    data = database.getUser(userId)
    cache.set("user:" + userId, data, 3600s)
    cache.del("lock:user:" + userId)
    return data
  else:
    // Wait for lock to be released, then retry
    sleep(100ms)
    return getUser(userId)
```

### Document Invalidation Strategy
- Clearly document when and how cache is invalidated
- Make it explicit in code comments
- This prevents bugs from inconsistent invalidation

---

## Common Mistakes

- ❌ Caching without TTL (data grows infinitely)
- ❌ Using cache for data that changes frequently (high miss rate)
- ❌ Not monitoring cache metrics (can't detect problems)
- ❌ Complex invalidation logic without documentation
- ❌ Ignoring cache stampede problems in high-traffic systems
