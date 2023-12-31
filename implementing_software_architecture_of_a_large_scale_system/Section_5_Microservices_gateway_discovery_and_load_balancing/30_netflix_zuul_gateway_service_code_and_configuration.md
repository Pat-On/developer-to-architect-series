# Netflix Zuul gateway service code and configuration


The main file that we have to configure. 
for now it is going to be static because we do not have discovery service.

```
server.port=8080
server.threadPool.threads.minimum=1
server.threadPool.threads.maximum=10
server.threadPool.threads.idleTime=60000

# Application Config
spring.application.name=GatewaySvc

# Eureka Config
eureka.instance.hostname=localhost
#eureka.client.eureka-server-port=9090
eureka.client.registerWithEureka=false
eureka.client.fetchRegistry=false
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka
eureka.instance.preferIpAddress=false
eureka.instance.leaseRenewalIntervalInSeconds=10
eureka.instance.leaseExpirationDurationInSeconds=30
eureka.client.registryFetchIntervalSeconds=10

# Service mapping
zuul.routes.authsvc.path=/auth/**
zuul.routes.authsvc.serviceId=AuthSvc
zuul.routes.authsvc.stripPrefix=false

zuul.routes.productsvc.path=/products/**
zuul.routes.productsvc.serviceId=ProductSvc
zuul.routes.productsvc.stripPrefix=false

zuul.routes.cartsvc.path=/carts/**
zuul.routes.cartsvc.serviceId=OrderSvc
zuul.routes.cartsvc.stripPrefix=false

zuul.routes.ordersvc.path=/orders/**
zuul.routes.ordersvc.serviceId=OrderSvc
zuul.routes.ordersvc.stripPrefix=false

zuul.routes.inventorysvc.path=/inventory/**
zuul.routes.inventorysvc.serviceId=InventorySvc
zuul.routes.inventorysvc.stripPrefix=false

zuul.routes.userprofilesvc.path=/users-profile/**
zuul.routes.userprofilesvc.serviceId=AuthSvc
zuul.routes.userprofilesvc.stripPrefix=false

zuul.routes.adminsvc.path=/admin/**
zuul.routes.adminsvc.serviceId=AdminSvc
zuul.routes.adminsvc.stripPrefix=false

zuul.routes.gatewaysvc.path=/**
zuul.routes.gatewaysvc.url=forward:/

spring.jmx.enabled=false
zuul.sensitiveHeaders=Cookie,Set-Cookie

ribbon.ReadTimeout=60000
ribbon.ConnectTimeout=10000
#hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=60000

ribbon.MaxTotalConnections=10
ribbon.MaxConnectionsPerHost=10

#####################################################

ribbon.eureka.enabled=false

AdminSvc.ribbon.listOfServers=http://localhost:8081
AdminSvc.ribbon.ServerListRefreshInterval=10000
AuthSvc.ribbon.listOfServers=http://localhost:8082
AuthSvc.ribbon.ServerListRefreshInterval=10000
ProductSvc.ribbon.listOfServers=http://localhost:8083
ProductSvc.ribbon.ServerListRefreshInterval=10000
OrderSvc.ribbon.listOfServers=http://localhost:8084
OrderSvc.ribbon.ServerListRefreshInterval=10000
InventorySvc.ribbon.listOfServers=http://localhost:8085
InventorySvc.ribbon.ServerListRefreshInterval=10000

opentracing.jaeger.enabled=false
opentracing.jaeger.udp-sender.host=localhost
opentracing.jaeger.udp-sender.port=6831

management.endpoints.web.exposure.include=health,info,prometheus


```