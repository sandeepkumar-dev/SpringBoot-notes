# Spring Boot Design Patterns
> From Beginner to Production-Grade — A Complete Reference Guide

---

## Table of Contents

1. [Circuit Breaker Pattern](#1-circuit-breaker-pattern)
2. [Saga Pattern](#2-saga-pattern)
3. [CQRS Pattern](#3-cqrs-pattern)
4. [Transactional Outbox Pattern](#4-transactional-outbox-pattern)
5. [Bulkhead Pattern](#5-bulkhead-pattern)
6. [API Gateway Pattern](#6-api-gateway-pattern)

---

## 1. Circuit Breaker Pattern

**Category:** Resilience
**Tagline:** Stop cascading failures before they spread

### 🌱 Beginner — What is it?

**Analogy:** Think of a home circuit breaker 🏠. When too much current flows, it trips and cuts power to prevent fire. Similarly, a Circuit Breaker pattern monitors calls to a service. If too many fail, it "trips" — stopping further calls to give the failing service time to recover.

**Key Takeaways:**
- Prevents cascading failures in microservices
- Has 3 states: `CLOSED` (normal), `OPEN` (blocking), `HALF-OPEN` (testing)
- Automatically recovers when the service is healthy again

---

### ⚙️ Intermediate — How it works in Spring Boot

**Library:** Resilience4j (recommended) or Spring Cloud Circuit Breaker

**Maven Dependency:**
```xml
<!-- pom.xml -->
<dependency>
  <groupId>io.github.resilience4j</groupId>
  <artifactId>resilience4j-spring-boot3</artifactId>
  <version>2.2.0</version>
</dependency>
```

**Configuration:**
```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowSize: 10          # Last 10 calls evaluated
        failureRateThreshold: 50       # Trip if 50% fail
        waitDurationInOpenState: 10s   # Wait 10s before HALF-OPEN
        permittedCallsInHalfOpenState: 3
```

**Code Example:**
```java
@Service
public class PaymentService {

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    public PaymentResponse processPayment(PaymentRequest request) {
        // calls external payment gateway
        return paymentGateway.charge(request);
    }

    // Called automatically when circuit is OPEN
    public PaymentResponse paymentFallback(PaymentRequest request, Exception ex) {
        log.warn("Payment gateway down, using fallback: {}", ex.getMessage());
        return PaymentResponse.queued("Payment queued for retry");
    }
}
```

---

### 🔥 Experienced — Advanced Patterns & Gotchas

| Topic | Details |
|---|---|
| **Combine with Retry** | `@Retry` + `@CircuitBreaker` — order matters! Apply Retry inside CircuitBreaker so retries count as one logical call. |
| **Bulkhead Pattern** | Use `@Bulkhead` alongside Circuit Breaker to limit concurrent calls (thread pool or semaphore isolation). |
| **Event Publishing** | Subscribe to `CircuitBreakerEvents` for metrics, alerting, and observability via Actuator `/actuator/circuitbreakers`. |
| **Distributed State** | In multi-instance deployments, each instance has its own circuit state. Use Redis-backed state if you need cluster-wide tripping. |

```java
@CircuitBreaker(name = "inventoryService")
@Retry(name = "inventoryService")           // Retry runs first (inner)
@TimeLimiter(name = "inventoryService")     // 2s timeout
@Bulkhead(name = "inventoryService")        // Max 10 concurrent
public CompletableFuture<Inventory> getInventory(String sku) {
    return CompletableFuture.supplyAsync(() -> inventoryClient.get(sku));
}

// Listen to state changes for alerting
@Bean
public CircuitBreakerEventConsumer circuitBreakerMonitor(CircuitBreakerRegistry registry) {
    registry.circuitBreaker("inventoryService")
        .getEventPublisher()
        .onStateTransition(event ->
            alertService.notify("Circuit state: " + event.getStateTransition()));
    return ...;
}
```

---

## 2. Saga Pattern

**Category:** Distributed Transactions
**Tagline:** Manage distributed transactions without 2PC

### 🌱 Beginner — What is it?

**Analogy:** Imagine booking a vacation 🏖️. You book a flight, hotel, and car rental. If the hotel is unavailable, you need to cancel your flight and car too. Saga breaks this into a sequence of local transactions, each with a **compensating transaction** (undo action) if something fails.

**Key Takeaways:**
- Solves the "distributed transaction" problem across microservices
- Each step has a compensating action (rollback equivalent)
- Two flavors: **Choreography** (events) and **Orchestration** (central coordinator)

---

### ⚙️ Intermediate — Choreography vs Orchestration

**Library:** Spring + Kafka/RabbitMQ (Choreography) | Axon Framework (Orchestration)

```xml
<!-- Axon Framework for Orchestration-based Saga -->
<dependency>
  <groupId>org.axonframework</groupId>
  <artifactId>axon-spring-boot-starter</artifactId>
  <version>4.9.3</version>
</dependency>
```

**Choreography vs Orchestration:**
- **Choreography:** Services react to events — no central brain. `OrderService` emits → `InventoryService` listens → `PaymentService` listens
- **Orchestration:** Saga class drives the workflow step by step — tells each service what to do and handles failures

```java
// ORCHESTRATION with Axon
@Saga
public class OrderSaga {

    @Autowired
    private transient CommandGateway commandGateway;

    @StartSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void on(OrderCreatedEvent event) {
        // Step 1: Reserve inventory
        commandGateway.send(new ReserveInventoryCommand(event.orderId(), event.items()));
    }

    @SagaEventHandler(associationProperty = "orderId")
    public void on(InventoryReservedEvent event) {
        // Step 2: Process payment
        commandGateway.send(new ProcessPaymentCommand(event.orderId(), event.amount()));
    }

    @SagaEventHandler(associationProperty = "orderId")
    public void on(PaymentFailedEvent event) {
        // Compensate: Release inventory
        commandGateway.send(new ReleaseInventoryCommand(event.orderId()));
        commandGateway.send(new CancelOrderCommand(event.orderId()));
    }

    @EndSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void on(PaymentSucceededEvent event) {
        commandGateway.send(new ConfirmOrderCommand(event.orderId()));
    }
}
```

---

### 🔥 Experienced — Production Considerations

| Topic | Details |
|---|---|
| **Idempotency** | Each step MUST be idempotent. Messages can be delivered multiple times. Use idempotency keys and database-level unique constraints. |
| **Saga State Persistence** | Axon stores saga state in a saga store (JPA/Mongo). Ensure your saga is serializable. Avoid holding non-serializable beans. |
| **Timeout Handling** | Use `@DeadlineManager` to handle saga timeouts — e.g., cancel an order if payment isn't received in 15 minutes. |
| **Choreography Pitfalls** | Choreography is simpler but harder to debug. Use distributed tracing (Zipkin/Jaeger) and correlation IDs to trace flows. |

```java
@Saga
public class OrderSaga {

    @Autowired
    private transient DeadlineManager deadlineManager;
    private String paymentDeadlineId;

    @SagaEventHandler(associationProperty = "orderId")
    public void on(InventoryReservedEvent event) {
        // Set a 15-minute deadline for payment
        paymentDeadlineId = deadlineManager.schedule(
            Duration.ofMinutes(15), "payment-deadline", event.orderId()
        );
        commandGateway.send(new ProcessPaymentCommand(event.orderId()));
    }

    @DeadlineHandler(deadlineName = "payment-deadline")
    public void onPaymentDeadline(String orderId) {
        // Payment timed out — compensate
        commandGateway.send(new ReleaseInventoryCommand(orderId));
        commandGateway.send(new CancelOrderCommand(orderId, "Payment timeout"));
    }

    @SagaEventHandler(associationProperty = "orderId")
    public void on(PaymentSucceededEvent event) {
        deadlineManager.cancelSchedule("payment-deadline", paymentDeadlineId);
        // ...
    }
}
```

---

## 3. CQRS Pattern

**Category:** Architecture
**Tagline:** Separate reads from writes for ultimate scalability

### 🌱 Beginner — What is it?

**Analogy:** In a restaurant 🍽️, the chef who cooks (writes) and the waiter who serves (reads) are different people. CQRS separates your application into a **Command** side (write/change state) and a **Query** side (read-only). They can even use different data models!

**Key Takeaways:**
- CQRS = **C**ommand **Q**uery **R**esponsibility **S**egregation
- **Commands:** change state (`CreateOrder`, `UpdateUser`)
- **Queries:** read state, never modify (`GetOrder`, `ListUsers`)
- Each side can be independently scaled and optimized

---

### ⚙️ Intermediate — CQRS in Spring Boot

**Library:** Axon Framework or manual implementation

```yaml
# Use separate datasources for read and write
spring:
  datasource:
    write:
      url: jdbc:postgresql://primary-db:5432/orders
    read:
      url: jdbc:postgresql://replica-db:5432/orders
```

```java
// === COMMAND SIDE ===
@RestController
@RequestMapping("/orders")
public class OrderCommandController {

    @PostMapping
    public ResponseEntity<String> createOrder(@RequestBody CreateOrderRequest req) {
        String orderId = orderCommandService.handle(req);
        return ResponseEntity.accepted().body(orderId);
    }
}

@Service
public class OrderCommandService {
    public String handle(CreateOrderRequest req) {
        Order order = Order.create(req);   // domain logic
        orderRepository.save(order);       // write to primary DB
        eventBus.publish(new OrderCreatedEvent(order));
        return order.getId();
    }
}

// === QUERY SIDE ===
@RestController
@RequestMapping("/orders")
public class OrderQueryController {

    @GetMapping("/{id}")
    public OrderDTO getOrder(@PathVariable String id) {
        return orderQueryService.findById(id);  // reads from read-replica/read-model
    }
}

@Service
public class OrderQueryService {
    // Uses a denormalized, query-optimized read model (could be Elasticsearch, Redis, etc.)
    public OrderDTO findById(String id) {
        return orderReadRepository.findById(id);
    }
}
```

---

### 🔥 Experienced — Event Sourcing + CQRS

| Topic | Details |
|---|---|
| **Event Sourcing Combo** | CQRS pairs naturally with Event Sourcing — store events as source of truth, build read models (projections) by replaying events. |
| **Eventual Consistency** | The read model is eventually consistent with the write model. Design your UI to handle this (optimistic UI, polling, WebSocket updates). |
| **Projection Rebuilding** | Since projections are derived, you can rebuild them from events anytime — a powerful debugging and migration tool. |
| **Multiple Read Models** | Have different projections for different use cases: `OrderSummaryProjection` for lists, `OrderDetailProjection` for detail pages, `AnalyticsProjection` for dashboards. |

```java
// Event Sourcing: store state as a stream of events
@Aggregate
public class OrderAggregate {

    @AggregateIdentifier
    private String orderId;
    private OrderStatus status;

    @CommandHandler
    public OrderAggregate(CreateOrderCommand cmd) {
        apply(new OrderCreatedEvent(cmd.orderId(), cmd.items()));
    }

    @EventSourcingHandler
    public void on(OrderCreatedEvent event) {
        this.orderId = event.orderId();
        this.status = OrderStatus.CREATED;
        // Aggregate state rebuilt by replaying events — no DB queries!
    }
}

// Read-side projection: listens to events and builds a queryable view
@Component
@ProcessingGroup("order-summary-projection")
public class OrderSummaryProjection {

    @EventHandler
    public void on(OrderCreatedEvent event, @Timestamp Instant createdAt) {
        orderSummaryRepo.save(new OrderSummaryEntity(
            event.orderId(), "CREATED", createdAt
        ));
    }

    @QueryHandler
    public List<OrderSummary> handle(FindAllOrdersQuery query) {
        return orderSummaryRepo.findAll();
    }
}
```

---

## 4. Transactional Outbox Pattern

**Category:** Data Consistency
**Tagline:** Guarantee event delivery with your database transaction

### 🌱 Beginner — What is it?

**Analogy:** Imagine writing a letter 📨 and dropping it in an outbox on your desk. A mail carrier checks the outbox regularly and delivers letters. Even if you step out, the letter gets delivered. The Outbox Pattern stores messages in your database (the outbox) as part of your business transaction, then a separate process delivers them to the message broker.

**Key Takeaways:**
- Solves the **dual-write problem**: updating DB and publishing event atomically
- Prevents message loss if the broker is down during a transaction
- Uses a dedicated `outbox` table in your database
- A relay/poller process reads the outbox and publishes to Kafka/RabbitMQ

---

### ⚙️ Intermediate — Implementation with Debezium

**Library:** Debezium (CDC) + Kafka | or Spring Scheduler (polling)

```xml
<!-- Option 1: Manual polling with Spring -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Option 2: Debezium for Change Data Capture (production-grade) -->
<dependency>
  <groupId>io.debezium</groupId>
  <artifactId>debezium-api</artifactId>
  <version>2.6.0.Final</version>
</dependency>
```

**Outbox Table (SQL Migration):**
```sql
CREATE TABLE outbox_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  aggregate_type VARCHAR(255),
  aggregate_id VARCHAR(255),
  event_type VARCHAR(255),
  payload JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  processed BOOLEAN DEFAULT FALSE
);
```

```java
@Service
@Transactional   // Single transaction for both DB update + outbox write
public class OrderService {

    public Order createOrder(CreateOrderRequest request) {
        // 1. Save the order
        Order order = orderRepository.save(Order.from(request));

        // 2. Write to outbox (same transaction — atomic!)
        outboxRepository.save(new OutboxEvent(
            "Order", order.getId(), "OrderCreated",
            objectMapper.writeValueAsString(new OrderCreatedPayload(order))
        ));

        return order;  // Broker is NOT called here!
    }
}

// Separate relay: polls outbox and publishes to Kafka
@Component
public class OutboxRelay {

    @Scheduled(fixedDelay = 1000)  // Every 1 second
    @Transactional
    public void publishPendingEvents() {
        List<OutboxEvent> pending = outboxRepository.findUnprocessed();
        for (OutboxEvent event : pending) {
            kafkaTemplate.send(event.getEventType(), event.getPayload());
            event.markProcessed();
        }
    }
}
```

---

### 🔥 Experienced — Production: Debezium CDC

| Topic | Details |
|---|---|
| **Polling vs CDC** | Polling adds DB load and latency. Debezium reads PostgreSQL WAL/MySQL binlog directly — zero polling, near-real-time, no missed events. |
| **At-Least-Once Delivery** | Outbox guarantees at-least-once delivery. Consumers MUST be idempotent — use message IDs to deduplicate. |
| **Ordering Guarantees** | Partition Kafka by `aggregateId` to ensure ordered delivery per aggregate. Don't use random partitioning. |
| **Cleanup** | Archive or delete processed outbox rows to prevent table bloat. Keep 7–30 days for replay/debugging. |

```json
// Debezium Kafka Connect Connector Config
{
  "name": "outbox-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "table.include.list": "public.outbox_events",
    "transforms": "outbox",
    "transforms.outbox.type": "io.debezium.transforms.outbox.EventRouter",
    "transforms.outbox.route.by.field": "event_type",
    "transforms.outbox.table.field.event.key": "aggregate_id"
  }
}
```

```java
// Consumer with idempotency
@KafkaListener(topics = "OrderCreated")
public void handleOrderCreated(OrderCreatedPayload payload,
                                @Header(KafkaHeaders.RECEIVED_KEY) String messageId) {
    if (processedEventRepo.existsById(messageId)) {
        log.info("Duplicate event {}, skipping", messageId);
        return;
    }
    inventoryService.reserve(payload);
    processedEventRepo.save(new ProcessedEvent(messageId));
}
```

---

## 5. Bulkhead Pattern

**Category:** Resilience
**Tagline:** Isolate failures to prevent total system failure

### 🌱 Beginner — What is it?

**Analogy:** Ships have bulkheads — watertight compartments 🚢. If one compartment floods, others stay dry. Bulkhead Pattern isolates different parts of your system into pools. If one pool is exhausted (e.g., slow DB calls flooding threads), other operations still have their own threads to run.

**Key Takeaways:**
- Prevents thread/resource exhaustion from spreading across services
- Two types: **Thread Pool** (separate pools) and **Semaphore** (concurrent call limit)
- Combined with Circuit Breaker for full resilience

---

### ⚙️ Intermediate — Bulkhead with Resilience4j

```xml
<dependency>
  <groupId>io.github.resilience4j</groupId>
  <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
```

```yaml
resilience4j:
  bulkhead:
    instances:
      inventoryService:
        maxConcurrentCalls: 10      # Semaphore: max 10 concurrent
        maxWaitDuration: 10ms       # Wait 10ms before rejecting

  thread-pool-bulkhead:
    instances:
      reportingService:
        maxThreadPoolSize: 4        # Dedicated thread pool
        coreThreadPoolSize: 2
        queueCapacity: 10
```

```java
@Service
public class InventoryService {

    // Semaphore Bulkhead — limits concurrent calls
    @Bulkhead(name = "inventoryService", type = Bulkhead.Type.SEMAPHORE,
              fallbackMethod = "inventoryFallback")
    public Inventory getInventory(String sku) {
        return inventoryClient.fetch(sku);  // max 10 concurrent
    }

    // Thread Pool Bulkhead — for async operations
    @Bulkhead(name = "reportingService", type = Bulkhead.Type.THREADPOOL)
    public CompletableFuture<Report> generateReport(ReportRequest req) {
        return CompletableFuture.supplyAsync(() -> reportEngine.generate(req));
        // Runs in a dedicated pool — won't starve main threads
    }

    public Inventory inventoryFallback(String sku, BulkheadFullException ex) {
        return Inventory.unavailable(sku);
    }
}
```

---

### 🔥 Experienced — Sizing & Tuning Bulkheads

| Topic | Details |
|---|---|
| **Little's Law** | Pool size = Throughput × Latency. If you handle 100 req/s and avg latency is 200ms, you need 100 × 0.2 = **20 threads** minimum. |
| **Semaphore vs Thread Pool** | Semaphore is lightweight, good for fast operations. Thread Pool provides true isolation and timeout capability for slow external calls. |
| **Monitor Rejection Rate** | Watch `bulkhead.rejected` metric in Actuator. High rejection = pool undersized or service degrading. |
| **Virtual Threads (Java 21+)** | With Spring Boot 3.2+ and virtual threads enabled, traditional thread pool bulkheads matter less. Focus on semaphore-style limiting for true backpressure. |

```yaml
# Java 21 + Spring Boot 3.2: Enable virtual threads
spring:
  threads:
    virtual:
      enabled: true
```

```java
// Monitor bulkheads via Actuator
// GET /actuator/metrics/resilience4j.bulkhead.available.concurrent.calls
// GET /actuator/metrics/resilience4j.bulkhead.call.rejected.count

// Custom metrics and alerting
@EventListener
public void onBulkheadEvent(BulkheadOnCallRejectedEvent event) {
    meterRegistry.counter("bulkhead.rejected",
        "name", event.getBulkheadName()).increment();
    if (rejectionRate.get() > 0.1) {  // >10% rejection
        alertService.page("Bulkhead " + event.getBulkheadName() + " under pressure!");
    }
}
```

---

## 6. API Gateway Pattern

**Category:** Architecture
**Tagline:** Single entry point for all your microservices

### 🌱 Beginner — What is it?

**Analogy:** Think of a hotel reception desk 🏨. You don't go directly to the kitchen for food or the basement for laundry. You talk to reception, and they route you. An API Gateway is a **single entry point** that routes requests to the right microservice, handles auth, rate limiting, and more.

**Key Takeaways:**
- Single entry point for all client requests
- Handles cross-cutting concerns: auth, logging, rate limiting, SSL
- Routes requests to appropriate microservices
- Spring Cloud Gateway is the Spring-native solution

---

### ⚙️ Intermediate — Spring Cloud Gateway

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://ORDER-SERVICE       # Load-balanced via Eureka
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20

        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/users/**
            - Header=Authorization, Bearer .+    # Only authenticated
```

```java
// Custom Global Filter (runs for ALL requests)
@Component
public class AuthenticationFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders()
                              .getFirst(HttpHeaders.AUTHORIZATION);

        if (token == null || !jwtService.isValid(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        // Add user info to downstream request headers
        ServerHttpRequest mutatedRequest = exchange.getRequest().mutate()
            .header("X-User-Id", jwtService.getUserId(token))
            .build();

        return chain.filter(exchange.mutate().request(mutatedRequest).build());
    }

    @Override
    public int getOrder() { return -1; }  // Run first
}
```

---

### 🔥 Experienced — Advanced Gateway Patterns

| Topic | Details |
|---|---|
| **Backend For Frontend (BFF)** | Create multiple gateways — one per client type (mobile, web, partner API). Each BFF aggregates and shapes responses for its client's needs. |
| **Request Aggregation** | Gateway can fan-out to multiple services and aggregate responses, reducing client round-trips. Use `WebClient` with `Mono.zip()`. |
| **Circuit Breaker on Gateway** | Add Resilience4j Circuit Breaker filter on routes. If a service is down, return cached/fallback response at the gateway level. |
| **Observability** | Add Micrometer tracing (Zipkin/Jaeger) at the gateway to trace requests across all downstream services with a single correlation ID. |

```yaml
# Route with Circuit Breaker + Fallback + Rate Limiting
spring:
  cloud:
    gateway:
      routes:
        - id: order-service-resilient
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**
          filters:
            - name: CircuitBreaker
              args:
                name: orderServiceCB
                fallbackUri: forward:/fallback/orders
            - name: RequestRateLimiter
              args:
                key-resolver: "#{@userKeyResolver}"
                redis-rate-limiter.replenishRate: 50
```

```java
// Fallback Controller (runs in gateway itself)
@RestController
public class FallbackController {
    @GetMapping("/fallback/orders")
    public ResponseEntity<Map<String, String>> orderFallback() {
        return ResponseEntity.status(503).body(Map.of(
            "message", "Order service temporarily unavailable",
            "retryAfter", "30"
        ));
    }
}

// BFF Pattern: Aggregate two services in one response
@GetMapping("/api/dashboard")
public Mono<Dashboard> getDashboard(@RequestHeader("X-User-Id") String userId) {
    Mono<Orders> orders = orderClient.getRecentOrders(userId);
    Mono<Notifications> notifs = notifClient.getUnread(userId);
    return Mono.zip(orders, notifs, Dashboard::new);  // Parallel fetch!
}
```

---

## Recommended Learning Order

```
1. ⚡ Circuit Breaker  ──┐
                         ├── Resilience fundamentals
2. 🚢 Bulkhead        ──┘

3. 🚪 API Gateway         ── Entry point setup

4. ✂️  CQRS           ──┐
                         ├── Distributed data patterns (build on each other)
5. 📖 Saga            ──┤
                         │
6. 📬 Outbox          ──┘
```

## Quick Reference

| Pattern | Problem Solved | Key Library |
|---|---|---|
| ⚡ Circuit Breaker | Cascading failures | Resilience4j |
| 📖 Saga | Distributed transactions | Axon Framework / Kafka |
| ✂️ CQRS | Read/write scalability | Axon / Spring Data |
| 📬 Transactional Outbox | Guaranteed event delivery | Debezium / Spring Scheduler |
| 🚢 Bulkhead | Resource isolation | Resilience4j |
| 🚪 API Gateway | Single entry point | Spring Cloud Gateway |

---

*Spring Boot 3.x • Java 17+ • Last updated: 2026*
