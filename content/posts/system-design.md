---
title: "System Design Interview Questions"
date: 2026-06-16
lastmod: 2026-08-01
weight: 2
draft: false
tags: ["System Design", "Architecture", "Interview"]
categories: ["System Design"]
---

System design interview questions with detailed explanations. Click on any question to expand details.

<!--more-->

---

<details>
<summary><strong>Design an Order Management System</strong> - <code>System Design</code></summary>

### Problem Statement

Design a scalable Order Management System that handles order creation, processing, and tracking. The system should support multiple services and handle high traffic.

### Key Requirements

- Handle concurrent order creation
- Support order tracking and status updates
- Integrate with payment, inventory, and shipping services
- Ensure consistency and reliability
- Support future scalability

### System Architecture

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
┌──────▼──────────────┐
│  API Gateway        │
│ (Rate Limiting)     │
└──────┬──────────────┘
       │
   ┌───┴────────────────────┬─────────────────────┐
   │                        │                     │
┌──▼──────────────┐  ┌──────▼────────┐  ┌────────▼─────┐
│ Order Service   │  │ Payment       │  │ Inventory    │
│ (REST/gRPC)     │  │ Service       │  │ Service      │
└──┬──────────────┘  └───────────────┘  └──────────────┘
   │
   ├─► Message Queue (RabbitMQ/Kafka)
   ├─► Order DB (PostgreSQL)
   ├─► Cache Layer (Redis)
   └─► Notification Service
```

### Core Components

#### 1. Order Service

```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody CreateOrderRequest request) {
        // Validate request
        // Reserve inventory
        // Process payment
        // Save order
        // Publish event
        return ResponseEntity.ok(orderService.createOrder(request));
    }
    
    @GetMapping("/{orderId}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable String orderId) {
        return ResponseEntity.ok(orderService.getOrder(orderId));
    }
}
```

### Key Points

1. **Event-Driven Architecture**: Orders publish events for downstream services
2. **Idempotency**: All API endpoints are idempotent using unique order IDs
3. **Transaction Management**: ACID compliance for order creation
4. **Service Communication**: Synchronous (REST/gRPC) for critical operations, asynchronous (Kafka) for events
5. **Monitoring & Logging**: All state changes are tracked and logged

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Caching Strategy</strong> - <code>Cache</code></summary>

### Problem Statement

What should be cached in an Order Management System? What TTL should products have?

### What to Cache

#### 1. Product Information

**Why Cache:**
- High read-to-write ratio
- Frequently accessed during order creation
- Reduces database load

**TTL:** 1-2 hours (products change frequently but not real-time)

#### 2. Customer Profile

**Why Cache:**
- Stable data with infrequent updates
- Accessed on every order

**TTL:** 4-6 hours

#### 3. Inventory/Stock Levels

**Why Cache:**
- Critical for order processing
- High read frequency
- Changes frequently

**TTL:** 5-15 minutes (conservative due to frequent updates)

#### 4. Pricing Information

**Why Cache:**
- Used in calculations
- Changes periodically

**TTL:** 30 minutes - 1 hour

#### 5. Shipping Rates

**Why Cache:**
- External service calls can be expensive
- Relatively stable

**TTL:** 24 hours

#### 6. Order Status (Recent Orders)

**Why Cache:**
- Frequently accessed
- Time-bound (recent orders only)

**TTL:** 6 hours

### Caching Patterns

| Resource | TTL | Strategy | Read:Write |
|----------|-----|----------|-----------|
| Product | 1-2 hrs | Cache-Aside | High |
| Customer | 4-6 hrs | Cache-Aside | High |
| Inventory | 5-15 mins | Write-Through | Medium |
| Pricing | 30-60 mins | Cache-Aside | High |
| Orders (Recent) | 6 hrs | Lazy Load | High |
| Shipping Rates | 24 hrs | Cache-Aside | Very High |

### Key Points

1. **Product Cache (1-2 hours)**: Balanced between freshness and performance
2. **Inventory Cache (5-15 minutes)**: Conservative TTL due to frequent changes
3. **Cache Invalidation**: Use event-driven invalidation for consistency
4. **Monitoring**: Track cache hit rates and adjust TTLs accordingly
5. **Fallback**: Always handle cache misses gracefully

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>API Performance Analysis</strong> - <code>API Design</code></summary>

### Problem Statement

What is a good API response time? How would you analyze and improve API performance?

### Good Response Time Benchmarks

| API Type | Target | Acceptable | Poor |
|----------|--------|-----------|------|
| GET (cached) | < 50ms | < 100ms | > 500ms |
| GET (DB) | < 200ms | < 500ms | > 1s |
| POST/PUT | < 300ms | < 1s | > 2s |
| Complex Query | < 1s | < 3s | > 5s |

### Performance Optimization Strategies

#### 1. N+1 Query Prevention

```java
@Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items")
List<Order> findAllWithItems();
```

#### 2. Index Optimization

```sql
CREATE INDEX idx_order_customer_date ON orders(customer_id, created_at DESC);
CREATE INDEX idx_order_status ON orders(status);
```

#### 3. Pagination for Large Result Sets

```java
@GetMapping("/orders")
public ResponseEntity<Page<OrderResponse>> listOrders(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
    
    Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
    Page<Order> orders = orderRepository.findAll(pageable);
    
    return ResponseEntity.ok(orders.map(OrderResponse::from));
}
```

#### 4. Caching Strategy

```java
@Cacheable(value = "orders", key = "#orderId")
public OrderResponse getOrder(String orderId) {
    return OrderResponse.from(
        orderRepository.findById(orderId).orElseThrow()
    );
}
```

### Analysis Tools

- Spring Boot Actuator: Metrics and monitoring
- Prometheus + Grafana: Visualization
- Zipkin: Distributed tracing
- SLA Monitoring: Track percentiles (p50, p95, p99)

### Key Points

1. **Set realistic targets**: Consider resource constraints and business needs
2. **Continuous monitoring**: Use metrics to identify bottlenecks
3. **Database optimization**: Indexes, query optimization, and lazy loading
4. **Caching**: Reduce database hits for frequently accessed data
5. **Distributed tracing**: Understand request flow across services

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Microservices - Extending Features Without Breaking Others</strong> - <code>Microservices</code></summary>

### The Actual Problem

Frontend sends a request with new fields. One backend microservice wants to use them. Other microservices share the same endpoint or DTO — they should ignore the unknown fields silently.

---

### Solution: Use @JsonIgnoreProperties to Ignore Unknown Fields

#### Step 1 — Backend ignores unknown fields by default

```java
@JsonIgnoreProperties(ignoreUnknown = true)
@Data
public class QuantityRequest {
    private Long quantity;              // existing — all services use this
    private Long estimatedQuantity;     // new — only product service reads it
    private Long backupQuantity;        // new — only product service reads it
}
```

Other microservices already using `quantity` will deserialize fine — unknown fields are silently dropped.

---

#### Step 2 — Only the interested service reads new fields

```java
// ProductService — uses new fields
@Service
public class ProductService {
    public void updateInventory(QuantityRequest request) {
        Long estimated = request.getEstimatedQuantity();  // reads it
        Long backup = request.getBackupQuantity();        // reads it
        
        // Process with extended logic
        productRepository.saveWithExtendedFields(estimated, backup);
    }
}

// OrderService — doesn't even reference new fields
@Service
public class OrderService {
    public void processOrder(QuantityRequest request) {
        Long quantity = request.getQuantity();  // just uses quantity as before
        // Order processing continues unchanged — nothing breaks
    }
}
```

---

#### Step 3 — Feature flag if rollout needs to be controlled

```java
@Service
public class ProductService {
    
    public void updateInventory(QuantityRequest request, String clientId) {
        Long quantity = request.getQuantity();
        
        if (featureToggle.isEnabled("EXTENDED_QUANTITY", clientId)) {
            Long estimated = request.getEstimatedQuantity();
            Long backup = request.getBackupQuantity();
            // process estimatedQuantity and backupQuantity
            productRepository.saveWithExtendedFields(quantity, estimated, backup);
        } else {
            // Legacy behavior
            productRepository.save(quantity);
        }
    }
}
```

---

### Key Takeaway

> `@JsonIgnoreProperties(ignoreUnknown = true)` is the core fix. Frontend can send new fields freely — each microservice only reads what it knows about. No versioning needed. No breaking changes.

### Best Practices

1. **Ignore Unknown Fields**: Always use `@JsonIgnoreProperties(ignoreUnknown = true)` on request/response DTOs
2. **Backward Compatible**: Existing services continue working without any code changes
3. **Optional New Fields**: New fields should be nullable/optional
4. **Feature Flags**: Control which services actually process new fields
5. **No API Versioning Needed**: Same endpoint works for all versions
6. **Independent Adoption**: Services adopt new fields at their own pace
7. **Gradual Rollout**: Use feature flags to enable new fields gradually per client

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Inter-Microservice Communication & Failure Handling</strong> - <code>Microservices</code></summary>

### Problem Statement

How do you communicate between microservices? If one fails, how do you mitigate?

### Communication Patterns

#### 1. Synchronous Communication (REST/gRPC)

```java
@Service
public class OrderService {
    
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    public PaymentResponse processPayment(PaymentRequest request) {
        return restTemplate.postForObject(
            "http://payment-service:8080/api/v1/payments",
            request,
            PaymentResponse.class
        );
    }
    
    public PaymentResponse paymentFallback(PaymentRequest request, Exception e) {
        logger.error("Payment service failed, using fallback", e);
        return PaymentResponse.builder()
            .status("PENDING")
            .message("Payment processing delayed, will retry later")
            .build();
    }
}
```

#### 2. Asynchronous Communication (Event-Driven)

```java
// Service A publishes event
@Service
public class OrderService {
    
    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public void createOrder(CreateOrderRequest request) {
        Order order = new Order();
        orderRepository.save(order);
        
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        kafkaTemplate.send("order-events", event);
    }
}

// Service B subscribes to event
@Service
public class NotificationService {
    
    @KafkaListener(topics = "order-events", groupId = "notification-service")
    public void handleOrderCreated(OrderCreatedEvent event) {
        emailService.sendOrderConfirmation(event.getOrder());
    }
}
```

#### 3. Retry Mechanism

```java
@Service
public class OrderService {
    
    @Retryable(
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2.0)
    )
    public PaymentResponse processPayment(PaymentRequest request) {
        return paymentServiceClient.process(request);
    }
    
    @Recover
    public PaymentResponse recoverPaymentProcessing(
            PaymentException e, PaymentRequest request) {
        logger.error("Payment processing failed after retries", e);
        failedPaymentQueue.add(request);
        return null;
    }
}
```

### Failure Mitigation Strategies

| Scenario | Solution | Implementation |
|----------|----------|-----------------|
| Service Down | Circuit Breaker | Resilience4j, Hystrix |
| Timeout | Timeout + Retry | @Timeout, @Retryable |
| Partial Failure | Saga Pattern | Orchestration, Choreography |
| Message Loss | Dead Letter Queue | Kafka, RabbitMQ |
| Data Inconsistency | Eventual Consistency | Event sourcing |

### Key Points

1. **Circuit Breaker**: Prevents cascading failures
2. **Retry Logic**: Handles transient failures
3. **Timeouts**: Prevents hanging requests
4. **Async Messaging**: Decouples services
5. **Fallbacks**: Graceful degradation
6. **Monitoring**: Alert on failures
7. **Dead Letter Queue**: Handle failed messages

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Microservices vs Monolithic - Real-time Benefits</strong> - <code>Microservices</code></summary>

### Real-time Benefits Comparison

#### 1. Independent Scaling

**Monolithic:**
- Scale entire application even if only one component needs it
- High resource utilization (waste)

**Microservices:**
- Scale only the Payment Service based on demand
- Efficient resource utilization
- Cost-effective

#### 2. Independent Deployment

**Monolithic:**
- Deploy entire app for single feature
- Risk: One bug affects entire system
- Downtime required

**Microservices:**
- Deploy only Order Service without affecting Payment Service
- Lower risk
- Zero downtime deployment

#### 3. Technology Freedom

**Monolithic:**
- Locked to one tech stack (Java, Spring, PostgreSQL)
- Difficult to adopt new technologies

**Microservices:**
- Order Service: Java + Spring
- Payment Service: Node.js + Express
- Notification Service: Python + FastAPI

#### 4. Fault Isolation

**Monolithic:**
- Payment Service bug → Entire system down
- Orders can't be created

**Microservices:**
- Payment Service down → Only payment fails
- Orders cached and queued
- Inventory service works independently

#### 5. Database per Service

**Monolithic:**
- Single database for everything
- Schema changes affect all features

**Microservices:**
- Order Service: PostgreSQL (transactional)
- Cache: Redis (fast lookups)
- Search: Elasticsearch (full-text search)

#### 6. Continuous Deployment

**Monolithic:**
- Deploy 1-2 times per day
- Full regression testing required

**Microservices:**
- Deploy 10+ times per day
- Smaller changes = less risk

### Performance Comparison

| Metric | Monolithic | Microservices |
|--------|-----------|---------------|
| Deployment Time | 30 minutes | 5 minutes |
| Time to Fix Bug | 2 hours | 20 minutes |
| New Feature Time | 3 weeks | 1 week |
| System Availability | 99.5% | 99.99% |

### Key Points

1. **Independent Scaling**: Scale only what needs it
2. **Rapid Deployment**: Smaller services = faster updates
3. **Technology Flexibility**: Choose best tool per service
4. **Fault Tolerance**: Failures isolated
5. **Business Agility**: Faster time to market

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Single-Person Feature Deployment (Canary Deployment)</strong> - <code>Deployment</code></summary>

### Problem Statement

Deploy a feature for a single person or small user group before full rollout.

### Canary Deployment Strategy

#### 1. Feature Flag Implementation

```java
@Service
public class FeatureFlagService {
    
    public boolean isFeatureEnabledForUser(String featureName, String userId) {
        FeatureFlag flag = repository.findByName(featureName);
        
        if (!flag.isEnabled()) {
            return false;
        }
        
        return flag.getCanaryUsers().contains(userId);
    }
}

@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @RequestBody CreateOrderRequest request,
            HttpServletRequest httpRequest) {
        
        String userId = getUserId(httpRequest);
        
        if (featureFlag.isFeatureEnabledForUser("NEW_CHECKOUT_FLOW", userId)) {
            return ResponseEntity.ok(orderService.createOrderV2(request));
        } else {
            return ResponseEntity.ok(orderService.createOrderV1(request));
        }
    }
}
```

#### 2. Gradual Rollout by Percentage

```java
@Service
public class CanaryDeploymentService {
    
    public boolean isFeatureEnabledForUser(String featureName, String userId) {
        CanaryDeployment deployment = canaryRepo.findByFeatureName(featureName);
        
        if (deployment == null) {
            return false;
        }
        
        // Use consistent hashing based on userId
        int userHash = Math.abs(userId.hashCode()) % 100;
        
        // If rollout is at 10%, enable for users with hash 0-9
        return userHash < deployment.getRolloutPercentage();
    }
}
```

#### 3. Kubernetes Canary Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service-v1
spec:
  replicas: 9
  selector:
    matchLabels:
      app: order-service
      version: v1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order-service
      version: v2
```

#### 4. Traffic Splitting with Istio

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
  - order-service
  http:
  - route:
    - destination:
        host: order-service
        subset: v1
      weight: 90
    - destination:
        host: order-service
        subset: v2
      weight: 10
```

### Deployment Steps

1. **Step 1**: Deploy to 1 user - Monitor for errors
2. **Step 2**: Expand to 1% - Continue monitoring
3. **Step 3**: Expand to 5% - Check metrics
4. **Step 4**: Expand to 10% - Broader testing
5. **Step 5**: Expand to 50% - Major traffic
6. **Step 6**: Full rollout (100%) - Complete migration
7. **Step 7**: Cleanup old version - Remove v1 deployment

### Key Points

1. **Feature Flags**: Control rollout without code changes
2. **Consistent Hashing**: Deterministic user assignment
3. **Monitoring**: Track errors and performance
4. **Automated Progression**: Rollout based on health checks
5. **Quick Rollback**: Revert instantly if issues detected
6. **Traffic Splitting**: Mix old and new versions
7. **User Communication**: Alert users of changes

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Monolithic vs Microservices — General Comparison</strong> - <code>Architecture</code></summary>

### Problem Statement

How do monolithic and microservices architectures compare overall? (Broader than just the real-time benefits covered above.)

### Comparison

| Aspect | Monolithic | Microservices |
|---|---|---|
| Codebase | Single, unified | Multiple, independent |
| Deployment | Whole app deployed together | Each service deployed independently |
| Scaling | Scale entire app | Scale individual services |
| Tech Stack | One stack for everything | Different stack per service |
| Data | Single shared database | Database per service |
| Team Structure | One team, shared codebase | Small teams own individual services |
| Communication | In-process function calls | Network calls (REST/gRPC/events) |
| Complexity | Simple to start, hard to maintain at scale | Ops overhead (discovery, tracing) but easier to scale a team |
| Fault Isolation | One bug can crash the app | Failures isolated to a service |

### When to Choose Which

- **Monolith:** small team, early-stage product, need to move fast without ops overhead
- **Microservices:** large team, need independent scaling/deployment, different parts of the system have very different load or tech needs

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Common Microservices Design Patterns</strong> - <code>Microservices</code> | <code>Patterns</code></summary>

### Problem Statement

What design patterns keep microservices reliable, and which one fits a partial-failure scenario across services (e.g., one service's update needs to roll back if a downstream step fails)?

### Key Patterns

| Pattern | Solves | One-liner |
|---|---|---|
| **Saga** | Distributed transactions | Break a transaction into local steps, each with a compensating action if a later step fails. Two flavors: **Choreography** (services react to each other's events) and **Orchestration** (a central coordinator drives the steps) |
| **Circuit Breaker** | Cascading failures | Trip open after N failures, fail fast instead of hammering a dead dependency, retry after a cooldown |
| **Bulkhead** | One overloaded dependency starving others | Isolate resources (thread pools/connections) per dependency |
| **API Gateway** | Clients calling many services directly | Single entry point for routing, auth, rate limiting |
| **CQRS** | Read/write models with different needs | Separate write model from read model, often different stores |
| **Event Sourcing** | Need full audit/history of state changes | Store state as a sequence of events, not just the current row |
| **Strangler Fig** | Migrating a monolith incrementally | Route traffic for a feature to the new service while the rest still hits the monolith |

### Best Fit for Partial-Failure / Distributed-Transaction Issues

**Saga pattern** — the go-to when one microservice's action needs to be safely rolled back if a downstream step fails, without a distributed lock/2PC.

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>What is Rate Limiting?</strong> - <code>API Design</code></summary>

### Problem Statement

What is a rate limiter, and how would you implement one?

### Definition

A rate limiter caps how many requests a client (user/IP/API key) can make in a given time window — protects the system from abuse, traffic spikes, and runaway clients.

### Common Algorithms

| Algorithm | How it Works | Trade-off |
|---|---|---|
| **Token Bucket** | Bucket refills at a fixed rate; each request consumes a token | Allows bursts up to bucket size |
| **Leaky Bucket** | Requests queued, processed at a fixed rate | Smooths bursts but adds latency |
| **Fixed Window Counter** | Count requests per fixed window (e.g., per minute) | Simple, but allows 2x burst at window edges |
| **Sliding Window Log/Counter** | Tracks timestamps in a rolling window | More accurate, slightly more memory |

### Simple Token Bucket

```java
class RateLimiter {
    private final int capacity;
    private double tokens;
    private final double refillRatePerSec;
    private long lastRefillTimestamp;

    public synchronized boolean allowRequest() {
        refill();
        if (tokens >= 1) {
            tokens -= 1;
            return true;
        }
        return false;
    }

    private void refill() {
        long now = System.currentTimeMillis();
        double secondsElapsed = (now - lastRefillTimestamp) / 1000.0;
        tokens = Math.min(capacity, tokens + secondsElapsed * refillRatePerSec);
        lastRefillTimestamp = now;
    }
}
```

At scale, this is usually centralized with **Redis** (`INCR` + `EXPIRE`, or a sorted set for sliding window) so all app instances share the same counter.

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Describe a Recent Task You Enjoyed Working On</strong> - <code>Behavioral</code></summary>

### Problem Statement

Interviewers often ask you to walk through a recent piece of work you liked — they're checking depth of ownership and technical decision-making, not just "what" you built.

### How to Structure the Answer (STAR)

- **Situation:** 1-2 lines of context — what was the project/problem
- **Task:** What was *your* specific responsibility
- **Action:** The technical decisions you made and why — this is what they're really listening for (trade-offs, alternatives considered)
- **Result:** Measurable outcome — performance gain, bug reduction, time saved, business impact

### Template to Fill In Before Your Next Interview

> "Recently I worked on **[feature/system]**, where the challenge was **[problem]**. I chose to **[approach]** over **[alternative]** because **[reasoning/trade-off]**. It resulted in **[measurable outcome]**, and what I liked about it was **[the specific technical challenge you found interesting]**."

> This one needs your own specifics — swap in an actual recent task before the interview.

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Order Service — API Design & Versioning</strong> - <code>API Design</code> | <code>System Design</code></summary>

### Problem Statement

Design the `Create Order` API for an e-commerce Order Service — the service split, DB choice, request/response contract, and how you'd version the API as it evolves.

### Service Breakdown

- **OrderService** — owns order creation and tracking
- **PaymentService** — handles payment processing
- **NotificationService** — sends order confirmations

### Data Store: SQL (PostgreSQL), not NoSQL

Orders and payments need **ACID transactions** and relational integrity between an order and its payment — a natural fit for a relational model over a document store. Core tables: `OrderTable`, `PaymentHistory`.

### API Contract

**`POST /createOrder`**

Request:
```json
{
  "quantity": 2,
  "productId": "P12345",
  "userId": 1001
}
```
`createdTimestamp` is set server-side — never trust a client-supplied timestamp.

Response:
```json
{
  "orderStatus": "ORDER_SUCCESS | PAYMENT_FAILED | PAYMENT_PROCESSING | FAILED",
  "orderDateTime": "2026-08-01T10:00:00Z",
  "quantity": 2,
  "price": {
    "discount": 0,
    "tax": 0,
    "price": 0,
    "total": 0
  },
  "productDetails": { "productId": "P12345", "name": "..." }
}
```

### API Versioning Options

| Approach | How | Trade-off |
|---|---|---|
| **URL Versioning** | `/createOrder/v2` | Explicit, easy to route and monitor, but clients must actively migrate |
| **Feature Flag** | Same endpoint, gate new fields/behavior per user or cohort | No client migration needed, gradual rollout — but adds branching logic inside the service |

### Caching

Cache the read-heavy, slow-changing inputs to order creation — product details, pricing — not the order itself, which is write-heavy and must stay strongly consistent. See the Caching Strategy question above for per-resource TTLs.

**Companies:** <code>Nike</code>

</details>
