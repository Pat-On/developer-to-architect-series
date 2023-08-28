# Service code caches for redis cache

inside: `services/product`

```java
import java.time.Duration;

@Configuration
@PropertySource(value = { "classpath:config.properties" })
public class CacheConfiguration {

    @Value("${redis.hostname}")
    private String redisHostName;

    @Value("${redis.port}")
    private int redisPort;

    @Bean
    @ConditionalOnProperty(name = "redis.enabled", havingValue = "true")
    JedisConnectionFactory jedisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(redisHostName);
        redisStandaloneConfiguration.setPort(redisPort);
        redisStandaloneConfiguration.setDatabase(0);
        //redisStandaloneConfiguration.setPassword(RedisPassword.of("password"));

        JedisClientConfiguration.JedisClientConfigurationBuilder jedisClientConfiguration = JedisClientConfiguration.builder();
        jedisClientConfiguration.connectTimeout(Duration.ofSeconds(60));// 60s connection timeout

        JedisConnectionFactory jedisConFactory = new JedisConnectionFactory(redisStandaloneConfiguration,
                jedisClientConfiguration.build());

        return jedisConFactory;
    }

    @Bean
    @ConditionalOnProperty(name = "redis.enabled", havingValue = "true")
    RedisTemplate<String, Product> redisProductTemplate() {
        RedisTemplate<String, Product> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(jedisConnectionFactory());
        return redisTemplate;
    }

}

```

## The way of getting products

```java
    public List<Product> getProductsByIds(List<String> ids) {
        List<Product> products = getProductsFromCache(ids); // <----- cache
        if (products == null || products.size() == 0) {
            // Get all from DB
            products = getProductDaoBean().getProducts(ids);
            addProductsToCache(products);
            return products;
        }
        if (products.size() == ids.size()) {
            // All products found in cache
            return products;
        }
        // Find missing products in cache
        Map<String, Product> productMap = new HashMap<>();
        products.forEach(product -> {
            productMap.put(product.getId(), product);
        });
        List<String> missingIds = new LinkedList<>();
        ids.forEach(id -> {
            Product product = productMap.get(id);
            if (product == null) missingIds.add(id);
        });
        // Get missing products from DB
        List<Product> mapProducts = new LinkedList<>();
        products = getProductDaoBean().getProducts(missingIds);
        addProductsToCache(products);
        // Put missing products in map to make it complete
        products.forEach(product -> productMap.put(product.getId(), product));
        // Get all products from map in the order of ids
        ids.forEach(id -> {
            mapProducts.add(productMap.get(id));
        });
        return mapProducts;
    }

```

## Examples of the helper function - to build the cache

```java


    public Product getProduct(String id) {
        Product product = getProductFromCache(id);
        if (product == null) {
            product = getProductDaoBean().getProduct(id);
            addProductToCache(product);
        }
        return product;
    }

    public boolean addProduct(Product product) {
        boolean success = getProductDaoBean().addProduct(product);
        if (success)
            addProductToCache(product);
        return success;
    }

    public Product modifyProduct(Product product) {
        product = getProductDaoBean().modifyProduct(product);
        if (product != null)
            addProductToCache(product);
        return product;
    }

    public boolean removeProduct(String id) {
        removeProductFromCache(id);
        return getProductDaoBean().removeProduct(id);
    }

    public boolean removeProducts() {
        removeProductsFromCache();
        return getProductDaoBean().removeProducts();
    }
```