
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
