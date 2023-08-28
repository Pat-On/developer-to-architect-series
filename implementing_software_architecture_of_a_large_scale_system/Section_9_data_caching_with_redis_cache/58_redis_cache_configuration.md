# Redis Cache Configuration

## .env
```
# Enable redis caching
redis.enabled=true
redis.hostname=redis
redis.port=6379
```

## Docker compose
```yml

  redis:
    image: redis
    container_name: redis
    hostname: redis
    networks:
      - mynet1
    ports:
      - "6379:6379"

 # dedicated for prometheus      
  redis-exporter:
    image: oliver006/redis_exporter
    container_name: redis-exporter
    hostname: redis-exporter
    networks:
      - mynet1
    ports:
      - "9121:9121"
    environment:
      - REDIS_ADDR=redis://redis:6379
    restart: unless-stopped



```