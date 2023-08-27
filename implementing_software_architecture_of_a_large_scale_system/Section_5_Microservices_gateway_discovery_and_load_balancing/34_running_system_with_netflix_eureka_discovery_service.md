# Running System with Netflix Eureka Discovery Service



### .env of the demo
```
# Web app
SERVICES_HOST=gateway-svc
SERVICES_PORT=8080
REGISTRY_URL=http://eureka:8761/eureka/apps

# Postgres DB
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres

# Services
server.port=8080
database.postgres.host=postgres

# Enable discovery of services
eureka.client.registerWithEureka=true
eureka.client.fetchRegistry=true
eureka.client.serviceUrl.defaultZone=http://eureka:8761/eureka
ribbon.eureka.enabled=true

```

so compare to previous solution we do not have routing to all services, but we have only routing to the discovery service that is going to route everything


```dockerfile

  eureka:
    build:
      context: ../../eureka
      dockerfile: Dockerfile
    image: ntw/eureka
    container_name: eureka
    hostname: eureka
    networks:
      - mynet1
    ports:
      - "8761:8761"
    env_file: .env
    environment:
      - JAVA_OPTIONS=-Xmx512M
      - server.port=8761
    #   this is setup here because we are starting it like any other service
      - eureka.client.registerWithEureka=false
      - eureka.client.fetchRegistry=false
    volumes:
      - app-logs:/usr/local/tomcat/logs


```