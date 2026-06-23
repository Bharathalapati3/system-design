
Cache - 1ms
DB - 100ms

query once in DB and write in cache.

most common case: 
  - on write to db, delete cache key
  - on read again, write to cache. best way for cache invalidation

Strategy
 - LRU - Least recently used - common
 - LFU - Least frequently used
 - Set TTL always


Cache stampede - 
    - Very comon used key expires
    - 1000 requests hit db at once
    - Solution is Mutex lock
    - 1 requests fetcs from DB , while others wait

Cache penetration - 
    - When some one tries to look for the details that doesnot exist in both cache and DB
    - Bloom filter common used

Cache avalanche - 
    - When more keys expire at the same time, all the queries hits the DB at the same time
    - Set different ttl for keys, so no heavy load at once.
    - cache warming, circuit breaker - other solutions
