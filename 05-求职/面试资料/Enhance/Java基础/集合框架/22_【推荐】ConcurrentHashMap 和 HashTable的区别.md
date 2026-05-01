1. key和value都不能为null
2. HashTable是全表锁，读操作也会加锁；concurrentHashMap锁的粒度更细，读操作不需要加锁；