Ref:- https://medium.com/@ankithahjpgowda/api-gateway-in-spring-boot-3ea804003021

=> API Gateway — receives incoming requests, performs authentication (if enabled) and forwards requests to actual microservice. On getting the response, return it to the consumer.

=> An API Gateway acts as an entry point for client requests in a microservices architecture. It sits between the client and a set of backend services, handling all the external API requests.

- The key functions of an API Gateway include:
    1. Request Routing: The gateway routes client requests to the appropriate microservice.
    2. Security: It can handle authentication, authorization, and rate limiting.
    3. Load Balancing: It can distribute traffic across multiple instances of services.
    4. Caching: The gateway can cache responses to reduce load on backend services.
    5. Transformation: It can modify the request or response, for example, converting from XML to JSON.
    6. Monitoring and Logging: Tracks the requests coming through, generating logs and metrics.

- Popular API Gateway solutions include:
    1. AWS API Gateway (for serverless and cloud-native applications)
    2. Kong
    4. Nginx
    5. Zuul (developed by Netflix)
    6. Apigee (by Google)
    7. In a Spring Boot or microservices context, the API Gateway (like Spring Cloud Gateway or Zuul) can be used to manage requests to your services, ensuring seamless communication and enforcing security policies across distributed systems.

*********************************Implementing API Gateway************************************************
Step1:-> Create a spring project with following dependencies.
    spring-cloud-starter-gateway, spring-boot-starter-actuator
Step2:-> Add properties in application.yaml file which helps us to route the requests comming to the gateway.

Spring:
  cloud:
    gateway:
      routes:
        - id: ORDER-SERVICE
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/order/**
        - id: PRODUCT-SERVICE
          uri: lb://PRODUCT-SERVICE
          predicates:
            - Path=/product/**
        - id: PAYMENT-SERVICE
          uri: lb://PAYMENT-SERVICE
          predicates:
            - Path=/payment/**

Now our requests comming to the gateway will be routed to the following -Paths defined as a predicates.


=> CiruitBrakers for application
    - A Circuit Breaker is a design pattern used in microservices architecture to prevent service failures from cascading across the system, which could result in service degradation or system unavailability. The circuit breaker works by monitoring calls to external services and "tripping" (opening) if the service fails too frequently, preventing further calls until the service is restored.
    - In Spring Cloud, you can implement a Circuit Breaker using Resilience4j, a lightweight fault tolerance library. Resilience4j integrates well with Spring Boot and Feign Clients.

**Implemnting circuitBraker.
    Step1:-> Include circuitbraker dependency of resilence4J.
        spring-cloud-starter-circuitbreaker-reactor-resilience4j
    Stpe2:-> Setup the filter property to our application routes in application.yaml file.

    Our application.yaml file finally looks like.
spring:
    cloud:
        gateway:
          routes:
            - id: ORDER-SERVICE
              uri: lb://ORDER-SERVICE
              predicates:
                - Path=/order/**
              filters:
                - name: CircuitBreaker
                  args:
                    name: ORDER-SERVICE
                    fallbackuri: forward:/orderServiceFallBack
            - id: PRODUCT-SERVICE
              uri: lb://PRODUCT-SERVICE
              predicates:
                - Path=/product/**
              filters:
                - name: CircuitBreaker
                  args:
                    name: PRODUCT-SERVICE
                    fallbackuri: forward:/productServiceFallBack
            - id: PAYMENT-SERVICE
              uri: lb://PAYMENT-SERVICE
              predicates:
                - Path=/payment/**
              filters:
                - name: CircuitBreaker
                  args:
                    name: PAYMENT-SERVICE
                    fallbackuri: forward:/paymentServiceFallBack
    
Step3:-> We have to create a FallBackController in APIGateway project which returns the specified response back to the caller during downtime of an application

@RestController
public class FallBackController {
    @RequestMapping("/orderServiceFallBack")
    public String orderServiceFallBack(){
        return "OrderService unavailable";
    }
    @RequestMapping("/productServiceFallBack")
    public String productServiceFallBack(){
        return "ProductService unavailable";
    }@RequestMapping("/paymentServiceFallBack")
    public String paymentServiceFallBack(){
        return "PaymentService unavailable";
    }
}

=> Configure the behavior of the circuit breaker to specify the conditions under which it should open and trigger the fallback. We can also configure timeouts, failure thresholds, etc.

resilience4j.circuitbreaker:
  instances:
    service1CircuitBreaker:
      slidingWindowSize: 5
      failureRateThreshold: 50
      waitDurationInOpenState: 10000


- slidingWindowSize: 5 means the circuit breaker will evaluate 5 requests before determining the failure rate.
- failureRateThreshold: 50 means if 50% of the requests fail, the circuit breaker will open.
- waitDurationInOpenState: 10000 means the circuit breaker will stay open for 10 seconds before trying again.

=> Configuring Rate Limiter
  -  Spring Boot, rate limiting is a technique used to control the number of requests a client can make to an API or service over a given time period. It helps in managing traffic and preventing abuse. Spring Cloud Gateway, a component of the Spring Cloud ecosystem, provides built-in support for rate limiting using Redis and in-memory solutions.

1. Rate Limiting with Redis
To implement rate limiting with Redis in a Spring Cloud Gateway setup, you can use the spring-boot-starter-data-redis-reactive dependency. Here's an example setup:

Step1: Add Redis Dependency to your pom.xml:
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
  </dependency>

Step2: Configure the Rate Limiting Filter in your application.yml:
spring:
  redis:
    host: localhost
    port: 6379

cloud:
  gateway:
    routes:
      - id: rate_limit_route
        uri: http://localhost:8080
        predicates:
          - Path=/api/**
        filters:
          - name: RequestRateLimiter
            args:
              redis-rate-limiter.replenishRate: 10 # Number of requests per second
              redis-rate-limiter.burstCapacity: 20 # Max burst capacity

Note: 
replenishRate: Number of requests per second.
burstCapacity: Maximum number of requests allowed in a burst.

Step3: Enable KeyResolver to specify which key to use for rate limiting (like the client IP):
@Bean
public KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
}

Step4: Add the KeyResolver in application.yml:

cloud:
  gateway:
    default-filters:
      - name: RequestRateLimiter
        args:
          key-resolver: "#{@userKeyResolver}"

2. In-Memory Rate Limiting
  - In-memory rate limiting can also be used when you don't want external storage like Redis. However, this is less scalable because the limits apply only to the instance memory.

Step1:-> Configuration:
  spring:
  cloud:
    gateway:
      routes:
        - id: in_memory_rate_limit_route
          uri: http://localhost:8080
          filters:
            - name: RequestRateLimiter
              args:
                replenishRate: 5 # 5 requests per second
                burstCapacity: 10

Step2:-> Enable KeyResolver to specify which key to use for rate limiting (like the client IP):
@Bean
public KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
}

Step3:-> Add the KeyResolver in application.yml:

cloud:
  gateway:
    default-filters:
      - name: RequestRateLimiter
        args:
          key-resolver: "#{@userKeyResolver}"
