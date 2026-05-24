A production-style backend system built with **Java 21**, **Spring Boot 4.0.6**, and **Spring Cloud 2025.1.1**, demonstrating a fully decoupled microservices architecture with centralized service discovery and a reactive API Gateway.

---

## Architecture Overview

```
── REQUEST FLOW (per HTTP request) ──────────────────────────────────────────

  ┌──────────────────────────────┐
  │          HTTP Client         │
  │  (Browser / Postman / App)   │
  └──────────────┬───────────────┘
                 │ HTTP Request
  ┌──────────────▼───────────────┐
  │          api-gateway         │  Port: 8080
  │   Spring Cloud Gateway       │  Looks up service address
  │   WebFlux (Non-blocking)     │  from Eureka registry
  └──────┬──────────────┬────────┘
         │ lb://         │ lb://
         │ user-service  │ order-service
  ┌──────▼──────┐  ┌─────▼────────┐
  │ user-service│  │order-service │
  │  Port: 8081 │  │  Port: 8082  │
  └─────────────┘  └──────────────┘

── REGISTRATION FLOW (once at startup, not per request) ─────────────────────

  ┌─────────────┐              ┌────────────────────────────┐
  │ api-gateway │──registers──▶│                            │
  ├─────────────┤              │      discovery-server      │
  │ user-service│──registers──▶│   Netflix Eureka Server    │
  ├─────────────┤              │        Port: 8761          │
  │order-service│──registers──▶│                            │
  └─────────────┘              └────────────────────────────┘
```

> **Key principle:** Eureka is a **Service Registry**, not a request proxy.
> It is **never** on the path of a client HTTP request.
> Services register their address with Eureka once at startup.
> The API Gateway queries Eureka to resolve `lb://` URIs, then routes **directly** to the target service.

---

## Service Port Mapping

| Service            | Module Name        | Port | Role                              |
| :----------------- | :----------------- | :--: | :-------------------------------- |
| Discovery Server   | `discovery-server` | 8761 | Eureka registry & dashboard       |
| API Gateway        | `api-gateway`      | 8080 | Reactive routing & load balancing |
| User Service       | `user-service`     | 8081 | User domain management            |
| Order Service      | `order-service`    | 8082 | Order domain management           |

---

## Prerequisites

| Tool       | Required Version |
| :--------- | :--------------- |
| **JDK**    | 21 or higher     |
| **Gradle** | 8.x (wrapper included — no install needed) |

> All modules use the **Gradle Wrapper** (`gradlew` / `gradlew.bat`). You do **not** need a system-level Gradle installation.

---

## Build All Services

Each service is an independent Gradle project. Build them individually from within each module directory:

```bash
# Build discovery-server
cd discovery-server
./gradlew build -x test

# Build api-gateway
cd ../api-gateway
./gradlew build -x test

# Build user-service
cd ../user-service
./gradlew build -x test

# Build order-service
cd ../order-service
./gradlew build -x test
```

---

## Startup Order

> **Critical**: Services must be started in the following order. Each service depends on Eureka being available for registration.

### Step 1 — Start Discovery Server

```bash
cd discovery-server
./gradlew bootRun
```

Wait until you see `Started DiscoveryServerApplication` in the console.
Verify the Eureka dashboard is accessible at: **http://localhost:8761**

---

### Step 2 — Start API Gateway

```bash
cd api-gateway
./gradlew bootRun
```

Wait until `api-gateway` appears as a registered instance in the Eureka dashboard.

---

### Step 3 — Start Domain Services

Start each domain service in a separate terminal:

```bash
# Terminal A
cd user-service
./gradlew bootRun

# Terminal B
cd order-service
./gradlew bootRun
```

Once all services are registered, verify through the Eureka dashboard that all four instances (`API-GATEWAY`, `USER-SERVICE`, `ORDER-SERVICE`) appear with status **UP**.

---

## API Access via Gateway

All client requests must go through the API Gateway on port `8080`:

| Downstream Service | Direct URL                         | Via Gateway                           |
| :----------------- | :--------------------------------- | :------------------------------------ |
| User Service       | `http://localhost:8081/api/users`  | `http://localhost:8080/api/users`     |
| Order Service      | `http://localhost:8082/api/orders` | `http://localhost:8080/api/orders`    |

---

## Module Documentation

Each module has its own detailed README:

- [`discovery-server/README.md`](./discovery-server/README.md)
- [`api-gateway/README.md`](./api-gateway/README.md)
- [`user-service/README.md`](./user-service/README.md)
- [`order-service/README.md`](./order-service/README.md)

---

## Tech Stack

| Category           | Technology                            |
| :----------------- | :------------------------------------ |
| Language           | Java 21                               |
| Framework          | Spring Boot 4.0.6                     |
| Cloud              | Spring Cloud 2025.1.1                 |
| Service Discovery  | Netflix Eureka Server / Client        |
| API Gateway        | Spring Cloud Gateway Server WebFlux   |
| Persistence        | Spring Data JPA / Hibernate           |
| Database           | H2 In-Memory                          |
| Build Tool         | Gradle (Groovy DSL)                   |
| Boilerplate Reduction | Lombok (Domain services only)       |
