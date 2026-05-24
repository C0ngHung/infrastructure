# api-gateway

The **single entry point** for all external HTTP traffic in the `microservice` system. Built on **Spring Cloud Gateway Server WebFlux** — a fully non-blocking, reactive API Gateway running on Netty. It routes requests to downstream microservices via client-side load balancing using Eureka service discovery.

---

## Service Profile

| Property          | Value                                             |
| :---------------- | :------------------------------------------------ |
| **Module**        | `api-gateway`                                     |
| **Port**          | `8080`                                            |
| **Role**          | Reactive API Gateway & Load Balancer              |
| **Spring Boot**   | 4.0.6                                             |
| **Spring Cloud**  | 2025.1.1                                          |
| **Web Stack**     | Spring WebFlux (Non-blocking, Netty)              |

---

## Spring Initializr Setup

Here is the Spring Initializr configuration used to bootstrap the API Gateway:

![Spring Initializr Setup](./images/Spring%20Initializr%20api-gateway.png)

---

## Dependencies

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.cloud:spring-cloud-starter-gateway-server-webflux'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:2025.1.1"
    }
}
```

> **Note:** `spring-cloud-starter-gateway` (the older artifact) is deprecated as of Spring Cloud 2025.x.
> The `spring-cloud-starter-gateway-server-webflux` starter is the official replacement.
> Do **not** mix this with `spring-boot-starter-webmvc` — the two web stacks are mutually exclusive.

---

## Routing Configuration

Routes are declared using the new **Spring Cloud Gateway 5.0 / 2025.x configuration prefix**:
`spring.cloud.gateway.server.webflux.routes`

> **Migration note:** The legacy prefix `spring.cloud.gateway.routes` will result in an empty route list at runtime in Spring Cloud 2025.x and must not be used.

```yaml
# src/main/resources/application.yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      server:
        webflux:
          routes:
            - id: user-service
              uri: lb://user-service
              predicates:
                - Path=/api/users/**

            - id: order-service
              uri: lb://order-service
              predicates:
                - Path=/api/orders/**

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

### Route Behavior

| Route ID        | Incoming Path        | Forwarded To          | Load Balanced |
| :-------------- | :------------------- | :-------------------- | :-----------: |
| `user-service`  | `/api/users/**`      | `lb://user-service`   | ✅ Yes        |
| `order-service` | `/api/orders/**`     | `lb://order-service`  | ✅ Yes        |

The `lb://` URI scheme instructs Spring Cloud LoadBalancer (integrated with Eureka) to resolve the target address dynamically at request time.

---

## Running the Service

> **Requires:** `discovery-server` must be running and healthy before starting this service.

```bash
cd api-gateway
./gradlew bootRun
```

Verify registration in the Eureka dashboard at **http://localhost:8761**.
The instance `API-GATEWAY` should appear with status **UP**.

---

## Accessing Downstream Services via Gateway

Once all services are running, route all HTTP traffic through the gateway:

```bash
curl http://localhost:8080/api/users

curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'
```

---

## Startup Dependency

| Start Order | Service            |
| :---------: | :----------------- |
| 1st         | `discovery-server` |
| **2nd**     | `api-gateway` ← Start here |
| 3rd         | `user-service`, `order-service` |
