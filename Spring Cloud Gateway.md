# Spring Cloud Gateway



Spring Cloud Gateway is a framework for building API gateways on top of Spring Boot and Spring Cloud. It provides features such as route matching, request filtering, and rate limiting to enable you to create a highly customizable and performant API gateway for your microservices architecture.

Spring Cloud Gateway is a web application gateway built on top of the Spring Framework, providing a platform for building modern, cloud-native APIs and microservices. It provides a consistent, reactive programming model for building and deploying web applications, with support for load balancing, rate limiting, routing, and security. Additionally, Spring Cloud Gateway integrates with other Spring Cloud technologies, such as Spring Cloud Config and Spring Cloud Netflix, to provide a complete end-to-end solution for building cloud-native applications.

Ovo je gotovo softversko resenje za gateway koji koriste mikroservisi.

The major benefits, which are delivered by Spring Cloud Gateway: 

- support to reactive programming model: reactive http endpoint, reactive web socket 

- configuring request processing (routes, filters, predicates) by java code or yet another markup language (YAML) 

- dynamic reloading of configuration without restarting the server (integration with Spring Cloud Config Server) 

- support for SSL 

- actuator Api 

- integration gateway to Service Discovery mechanism 

- load-balancing mechanisms 

- rate-limiting (throttling) mechanisms 

- circuit breakers mechanism 

- integration with Oauth2 due to providing security features 

- 

  ![OF-SPRING-CLOUD-GATEWAY-02-1024x764-1](C:\Users\Ilija\Downloads\OF-SPRING-CLOUD-GATEWAY-02-1024x764-1.webp)



The first feature of Spring Cloud Gateway I am going to describe is a configuration of request processing. It can be considered the heart of the gateway. It is one of the major parts and responsibilities. As I mentioned earlier this logic can be created by java code or by YAML files. Below I add an example configuration in YAML and Java code way. Basic building blocks used to create processing logic are: 

- Predicates – match requests based on their feature (path, hostname, headers, cookies, query) 
- Filters – process and modify requests in a variety of ways. Can be divided depending on their purpose: 
- gateway filter – modify the incoming http request or outgoing http response 
- global filter – special filters applying to all routes so long as some conditions are fulfilled 

![spring_cloud_gateway_diagram](C:\Users\Ilija\Downloads\spring_cloud_gateway_diagram.png)

Clients make requests to Spring Cloud Gateway. If the Gateway Handler Mapping determines that a request matches a route, it is sent to the Gateway Web Handler. This handler runs the request through a filter chain that is specific to the request. The reason the filters are divided by the dotted line is that filters can run logic both before and after the proxy request is sent. All “pre” filter logic is executed. Then the proxy request is made. After the proxy request is made, the “post” filter logic is run.



 URIs defined in routes without a port get default port values of 80 and 443 for the HTTP and HTTPS URIs, respectively.

To implement Spring Cloud Gateway, the following steps can be followed:

1. Add the Spring Cloud Gateway dependency to the project's pom.xml file.
2. Create a new Gateway class and annotate it with @SpringBootApplication and @EnableGateway.
3. Define the routes to be used by the Gateway using the @Bean annotation and the RouteLocatorBuilder.
4. Configure the Gateway's load balancing and rate limiting options using the @Bean annotation and the LoadBalancerClient and RateLimiter instances.
5. Add security to the Gateway by configuring it with Spring Security and defining the necessary security policies.
6. Start the Gateway by running the Spring Boot application.
7. Test the Gateway by making requests to the defined routes and verifying the expected responses.



The next feature of Spring Cloud Gateway is the implementation of rate-limiting (throttling) mechanisms. This mechanism was designed to protect gateways from harmful traffic. One of the examples might be distributed denial-of-service (DDoS) attack. It consists of creating an enormous number of requests per second which the system cannot handle.
The filtering of requests may be based on the user principles, special fields in headers, or other rules. In production environments, mostly several gateways instance up and running but for Spring Cloud Gateway framework is not an obstacle, because it uses Redis to store information about the number of requests per key. All instances are connected to one Redis instance so throttling can work correctly in a multi-instances environment.
Due to prove the advantages of this functionality I configured rate-limiting in Gateway in the lab environment and created an end-to-end test, which can be described in the picture below.

The parameters configured for throttling are as follows: DefaultReplenishRate = 4, DefaultBurstCapacity = 8. It means getaways allow 4 Transactions (Request) per second (TPS) for the concrete key. The key in my example is the header value of “Host” field, which means that the first and second clients have a limit of 4TPS separately. If the limit is exceeded, the gateway replies by http response with 429 http code. Because of that, all requests from the first client are passed to the production service, but for the second client only half of the requests are passed to the production service by the gateway, and another half are replied to the client immediately with 429 Http Code. 



The circuit breaker is a pattern that is used in case of failure connected to a specific microservice. All we need is to define Spring Gateway fallback procedures. Once the connection breaks down, the request will be forwarded to a new route. The circuit breaker offers more possibilities, for example, special action in case of network delays and it can be configured in the gateway.

This project provides an API Gateway for microservices architecture, and is built on top of reactive **Netty** and Project **Reactor**. It is designed to provide a simple, but effective way to route to APIs and address such popular concerns as security, monitoring/metrics, and resiliency.

Spring Cloud Gateway offers you many features and configuration options. Today I’m going to focus on the single one, but very interesting aspect of gateway configuration – **rate limiting**. A rate limiter may be defined as a way to control the rate of traffic sent or received on the network. We can also define a few types of rate limiting. Spring Cloud Gateway currently provides a **Request Rate Limiter**, 

which is responsible for restricting each user to N requests per second.
When using `RequestRateLimiter` with Spring Cloud Gateway we may leverage Redis. Spring Cloud implementation uses [token bucket algorithm](https://en.wikipedia.org/wiki/Token_bucket) to do rate limiting. This algorithm has a centralized bucket host where you take tokens on each request, and slowly drip more tokens into the bucket. If the bucket is empty, it rejects the request.

Spring Cloud Gateway starter is required. For handling rate limiter with Redis we also need to add dependency to `spring-boot-starter-data-redis-reactive` starter. 

Request rate limiting is realized using a Spring Cloud Gateway component called `GatewayFilter`. Each instance of this filter is constructed in a specific factory. Filter is of course responsible for modifying requests and responses before or after sending the downstream request. Currently, there are 30 available built-in gateway filter factories.
The `GatewayFilter` takes an optional `keyResolver` parameter and parameters specific to the rate limiter implementation (in that case an implementation using Redis). Parameter `keyResolver` is a bean that implements the `KeyResolver` interface. It allows you to apply different strategies to derive the key for limiting requests. Parameter `keyResolver` is a bean that implements the `KeyResolver` interface. It allows you to apply different strategies to derive the key for limiting requests. Following Spring Cloud Gateway documentation:

The default implementation of `KeyResolver` is the `PrincipalNameKeyResolver` which retrieves the `Principal` from the `ServerWebExchange` and calls `Principal.getName()`. By default, if the KeyResolver does not find a key, requests will be denied. This behavior can be adjusted with the `spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key` (true or false) and `spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code` properties.

1. Add the Spring Data Redis dependency to the project's pom.xml file.
2. Configure the Redis connection details in the application.properties file.
3. In the Gateway class, use the @Bean annotation to create a RedisConnectionFactory instance and a RedisTemplate instance.
4. Use the RedisTemplate instance to store and retrieve data from Redis.
5. In the route configuration, use the RedisRateLimiter class to apply rate limiting based on the data stored in Redis.
6. Start the Gateway and test the Redis integration by making requests and verifying the expected responses.

```java
@SpringBootApplication
public class GatewayApplication {

   public static void main(String[] args) {
      SpringApplication.run(GatewayApplication.class, args);
   }

   @Bean
   KeyResolver userKeyResolver() {
      return exchange -> Mono.just("1");
   }
}
```

Assuming we have the following configuration and a target application running on port `8091` we may perform some test calls. You may set two properties for customizing the process. The `redis-rate-limiter.replenishRate` decides how many requests per second a user is allowed to send without any dropped requests. This is the rate that the token bucket is filled. The second property `redis-rate-limiter.burstCapacity` is the maximum number of requests a user is allowed to do in a single second. This is the number of tokens the token bucket can hold. Setting this value to zero will block all requests.

```yaml
spring:
  application:
    name: gateway-service
  redis:
    host: 192.168.99.100
    port: 6379
  cloud:
    gateway:
      routes:
      - id: account-service
        uri: http://localhost:8091
        predicates:
        - Path=/account/**
        filters:
        - RewritePath=/account/(?<path>.*), /$\{path}
      - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
```

Now, if you call the endpoint exposed by the gateway you get the following response. It includes some specific headers, which are prefixed by `x-ratelimit`. Header `x-ratelimit-burst-capacity` indicates to `burstCapacity` value, `x-ratelimit-replenish-rate` indicates to `replenishRate` value, and the most important `x-ratelimit-remaining`, which shows you the number of requests you may send in the next second.

in bean style:

```
@Configuration
@EnableRedisWebSession
```

In the case of security, just need (where ServerHttpSecurity is the builder to webflux, can build to your requirements: oauth2.0, etc.):

```
@Bean
public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {...}
```

In the case of rate limiter, just need:

```
@Bean
public RedisRateLimiter redisRateLimiter() {
    return new RedisRateLimiter(5, 7); // replenishRate, burstCapacity
}

@Bean
public RouteLocator appRouteLocator(RouteLocatorBuilder builder, RedisRateLimiter redisRateLimiter) {
  ... .requestRateLimiter(rl -> rl.setRateLimiter(redisRateLimiter)) // see example
}
```

ere is an example of a Spring Cloud Gateway implementation:

@SpringBootApplication @EnableGateway public class GatewayApplication {

```
Copy codepublic static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
}

@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
                  .route(p -> p.path("/users/**")
                               .filters(f -> f.addRequestHeader("X-Request-Foo", "Bar"))
                               .uri("http://localhost:8080/users"))
                  .route(p -> p.path("/products/**")
                               .filters(f -> f.addRequestHeader("X-Request-Foo", "Bar"))
                               .uri("http://localhost:8080/products"))
                  .build();
}
```

}

In this example, the GatewayApplication class is annotated with @SpringBootApplication and @EnableGateway to enable the Gateway functionality. The myRoutes() method uses the RouteLocatorBuilder to define two routes, one for "/users/**" and another for "/products/**". Both routes apply a filter to add a request header and forward the request to the specified target URI. The routes are then registered with the Gateway by returning them as a RouteLocator bean.

@SpringBootApplication @EnableGateway public class GatewayApplication {

```
Copy codepublic static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
}

@Bean
public RedisConnectionFactory redisConnectionFactory() {
    return new JedisConnectionFactory();
}

@Bean
public RedisTemplate<String, String> redisTemplate() {
    RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(redisConnectionFactory());
    return redisTemplate;
}

@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    RedisRateLimiter rateLimiter = new RedisRateLimiter(1, 2);
    return builder.routes()
                  .route(p -> p.path("/users/**")
                               .filters(f -> f.addRequestHeader("X-Request-Foo", "Bar")
                                              .requestRateLimiter(rateLimiter))
                               .uri("http://localhost:8080/users"))
                  .route(p -> p.path("/products/**")
                               .filters(f -> f.addRequestHeader("X-Request-Foo", "Bar")
                                              .requestRateLimiter(rateLimiter))
                               .uri("http://localhost:8080/products"))
                  .build();
}
```

}

In this example, the GatewayApplication class is annotated with @SpringBootApplication and @EnableGateway to enable the Gateway functionality. The redisConnectionFactory() and redisTemplate() methods use the JedisConnectionFactory and RedisTemplate classes to create a Redis connection and template. In the myRoutes() method, a RedisRateLimiter instance is created and applied to the routes using the requestRateLimiter() filter. The Gateway and Redis integration can then be tested by making requests to the defined routes and verifying the expected responses.

@SpringBootApplication @EnableGateway public class GatewayApplication {

```java
Copy codepublic static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
}

@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    converter.setSigningKey("secret");
    converter.setVerifierKey("secret");
    converter.setSigningAlgorithmName("HS256");
    return converter;
}

@Bean
public TokenStore tokenStore() {
    return new JwtTokenStore(accessTokenConverter());
}

@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
                  .route(p -> p.path("/users/**")
                               .filters(f -> f.addRequestHeader("X-Request-Foo", "Bar")
                                              .jwt())
                               .uri("http://localhost:8080/users"))
                  .route(p -> p.path("/products/**")
                               .filters(f -> f.addRequestHeader("X-Request-Foo", "Bar")
                                              .jwt())
                               .uri("http://localhost:8080/products"))
                  .build();
}
```

}

In this example, the GatewayApplication class is annotated with @SpringBootApplication and @EnableGateway to enable the Gateway functionality. The accessTokenConverter() method uses the JwtAccessTokenConverter class to create a JWT token converter and set the signing key, signing algorithm, and verifier key. The tokenStore() method creates a JWT token store using the token converter. In the myRoutes() method, the jwt() filter is applied to the routes to validate the JWT token in incoming requests. The Gateway and JWT validation can then be tested by making requests to the defined routes with a valid JWT token and verifying the expected responses.

Postmam doda sam Bearer 