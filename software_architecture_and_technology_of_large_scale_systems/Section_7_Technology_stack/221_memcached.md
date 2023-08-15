# Memcached

- Stores key value pairs
- values can be
  - any blob
  - any size
    - preferred < 1mb
    - Max is configurable
- centralized cache

![Alt text](image-16.png)

![Alt text](image-17.png)

- TTL can be set differently for a each data
- eviction expunges expired data followed by LRU data
- 