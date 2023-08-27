# Netflix Eureka Discovery Service Code and Configuration

check the folder `./services/discovery`

```java


package com.ntw.discovery;

import com.ntw.common.config.EnvConfig;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;

/**
 * Created by anurag on 08/09/20.
 */
@SpringBootApplication
@EnableEurekaServer
@PropertySource(value = { "classpath:config.properties" })
public class WebApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }

    @Bean
    public EnvConfig envConfig(Environment environment) {
        // Added this bean to view env vars on console/log
        EnvConfig envConfigBean = new EnvConfig();
        envConfigBean.setEnvironment(environment);
        return envConfigBean;
    }

}
```


## to configure the server you need to provide config to `resources/config.properties`

```
server.port=8761

spring.application.name=eureka

# set true value for eureka server cluster
eureka.client.registerWithEureka=false
eureka.client.fetchRegistry=false

# used only with eureka server cluster
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka

eureka.server.enableSelfPreservation=false
eureka.server.responseCacheUpdateInvervalMs=30000
eureka.server.evictionIntervalTimerInMs=30000

eureka.instance.preferIpAddress=false
eureka.instance.leaseRenewalIntervalInSeconds=10
eureka.instance.leaseExpirationDurationInSeconds=30
eureka.client.registryFetchIntervalSeconds=10

logging.level.com.netflix.eureka=OFF
logging.level.com.netflix.discovery=OFF
```

## CHanges in the service to let it register

inside the common

```xml
   <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
      <version>2.2.9.RELEASE</version>
      <exclusions>
        <exclusion>
          <groupId>org.springframework.security</groupId>
          <artifactId>spring-security-crypto</artifactId>
        </exclusion>
        <exclusion>
          <groupId>com.google.inject</groupId>
          <artifactId>guice</artifactId>
        </exclusion>
        <exclusion>
          <groupId>xpp3</groupId>
          <artifactId>xpp3_min</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

```

### Order - service
```java
    @Override
    public boolean reserveInventory(InventoryReservation inventoryReservation) throws IOException {

        // finding services - discovery service
        ServiceInstance instance = getLoadBalancer().choose(ServiceID.InventorySvc.toString());
        if (instance == null) {
            logger.error("Unable to reserve inventory, as inventory service is not available");
            throw new IOException("Inventory service is not available");
        }
        StringBuilder url = new StringBuilder()
                .append("http://").append(instance.getHost())
                .append(":").append(instance.getPort())
                .append(AppConfig.INVENTORY_RESOURCE_PATH)
                .append(AppConfig.INVENTORY_RESERVATION_PATH);
                
        // call to inventory service
        String invResJson = (new Gson()).toJson(inventoryReservation);
        RequestBody body = RequestBody.create(invResJson, okhttp3.MediaType.parse("application/json; charset=utf-8"));
        Request.Builder requestBuilder = new Request.Builder()
                .url(url.toString())
                .post(body);
        String authHeader = OrderServiceImpl.getThreadLocal().get();
        requestBuilder.addHeader("Authorization", authHeader);
        Span invRemoteCallSpan = tracer.buildSpan("reserveInventoryRemote").asChildOf(tracer.activeSpan()).start();
        tracer.inject(invRemoteCallSpan.context(),
                    Format.Builtin.HTTP_HEADERS,
                    new RequestBuilderCarrier(requestBuilder));
        Request request = requestBuilder.build();
        try (Response response = client.newCall(request).execute()) {
            if (response.code() == 200) {
                logger.debug("Reserved inventory successfully; context={}", inventoryReservation);
                return true;
            }
            logger.debug("Unable to reserve inventory, got error; errorCode={}, message={} context={}",
                        response.code(), response.message(), inventoryReservation);
            return false;
        } catch (IOException e) {
            logger.error("Error calling InventorySvc while reserving inventory; context={}", inventoryReservation);
            logger.error(e.getMessage(), e);
            throw e;
        }
        finally {
            invRemoteCallSpan.finish();
        }
    }

```