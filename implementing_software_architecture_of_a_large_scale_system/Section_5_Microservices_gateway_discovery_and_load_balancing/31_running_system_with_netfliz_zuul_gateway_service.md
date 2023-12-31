# Running System with Netflix Zuul Gateway Service

```yml
  gateway-svc:
    image: ntw/services
    container_name: gateway-svc
    hostname: gateway-svc
    networks:
      - mynet1
    ports:
      - "8080:8080"
    env_file: .env
    volumes:
      - app-logs:/var/log/oms
    command: gateway
```


Example of configuration
```
# Web app
SERVICES_HOST=gateway-svc
SERVICES_PORT=8080

# Postgres DB
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres

# Services
server.port=8080
database.postgres.host=postgres

# Services -> Routing information
GatewaySvc.ribbon.listOfServers=http://gateway-svc:8080
AdminSvc.ribbon.listOfServers=http://admin-svc:8080
AuthSvc.ribbon.listOfServers=http://auth-svc:8080
UserProfileSvc.ribbon.listOfServers=http://user-profile-svc:8080
ProductSvc.ribbon.listOfServers=http://product-svc:8080
OrderSvc.ribbon.listOfServers=http://order-svc:8080
InventorySvc.ribbon.listOfServers=http://inventory-svc:8080

```