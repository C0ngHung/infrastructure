# discovery-server

The central **Service Registry** for the `microservice` system. Built on **Netflix Eureka Server**, it enables all microservices to register themselves and discover each other dynamically — eliminating hardcoded service addresses.

---

## Service Profile

| Property          | Value                                           |
| :---------------- | :---------------------------------------------- |
| **Module**        | `discovery-server`                              |
| **Port**          | `8761`                                          |
| **Role**          | Standalone Eureka Service Registry              |
| **Spring Boot**   | 4.0.6                                           |
| **Spring Cloud**  | 2025.1.1                                        |

---

## Spring Initializr Setup

Here is the Spring Initializr configuration used to bootstrap the Discovery Server:

![Spring Initializr Setup](./images/Spring%20Initializr%20discovery-server.png)

---

## Dependencies

```groovy
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:2025.1.1"
    }
}
```

---

## Configuration

The server runs in **standalone mode**: it does not register itself as a Eureka client and does not attempt to fetch the registry from another peer.

```yaml
server:
  port: 8761

spring:
  application:
    name: discovery-server

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

---

## Application Entry Point

The main class is annotated with `@EnableEurekaServer` to activate the Eureka server auto-configuration:

```java
@EnableEurekaServer
@SpringBootApplication
public class DiscoveryServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServerApplication.class, args);
    }
}
```

---

## Running the Service

```bash
cd discovery-server
./gradlew bootRun
```

Expected log output on successful startup:

```
Started DiscoveryServerApplication in X.XXX seconds
```

---

## Eureka Dashboard

Once running, the Eureka management dashboard is accessible at:

**http://localhost:8761**

The dashboard displays:
- All currently registered service instances
- Instance health status (`UP` / `DOWN`)
- Registration metadata (host, port, service ID)

As other services start, they will appear in the **"Instances currently registered with Eureka"** table on this page.

---

## Startup Dependency

This service has **no upstream dependencies** and must be started **first** before any other module in the system.

| Start Order | Service          |
| :---------: | :--------------- |
| **1st**     | `discovery-server` ← Start here |
| 2nd         | `api-gateway`    |
| 3rd         | `user-service`, `order-service` |
