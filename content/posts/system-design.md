---
title: "System Design Interview Questions"
date: 2026-06-16
draft: false
tags: ["System Design", "Architecture", "Interview"]
categories: ["System Design"]
---

# System Design Interview Questions Repository

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

</details>

---

<details>
<summary><strong>Microservices - Extending Features Without Breaking Others</strong> - <code>Microservices</code></summary>

### Problem Statement

One microservice needs new fields (estimated quantity, backup quantity) but other microservices depend on quantity. How do you extend without breaking existing services?

### Solution: Schema Evolution with Backward Compatibility

#### 1. Add New Fields Without Breaking Existing API

```java
@Data
@Builder
public class QuantityResponse {
    private Long quantity;
    private LocalDateTime lastUpdated;
    
    // New fields - optional with default values
    @JsonInclude(JsonInclude.Include.NON_NULL)
    private Long estimatedQuantity;
    
    @JsonInclude(JsonInclude.Include.NON_NULL)
    private Long backupQuantity;
}
```

#### 2. API Versioning

```java
// Keep old API intact
@RestController
@RequestMapping("/api/v1/products/{productId}")
public class QuantityControllerV1 {
    
    @GetMapping("/quantity")
    public ResponseEntity<QuantityResponseV1> getQuantity(@PathVariable String productId) {
        return ResponseEntity.ok(quantityService.getQuantityV1(productId));
    }
}

// New API with extended features
@RestController
@RequestMapping("/api/v2/products/{productId}")
public class QuantityControllerV2 {
    
    @GetMapping("/quantity")
    public ResponseEntity<QuantityResponseV2> getQuantity(@PathVariable String productId) {
        return ResponseEntity.ok(quantityService.getQuantityV2(productId));
    }
}
```

#### 3. Database Schema Evolution

```sql
ALTER TABLE product_quantity ADD COLUMN estimated_quantity BIGINT NULL;
ALTER TABLE product_quantity ADD COLUMN backup_quantity BIGINT NULL;
```

#### 4. Feature Toggle for Gradual Rollout

```java
@Service
public class QuantityService {
    
    public QuantityResponse getQuantity(String productId, String clientId) {
        ProductQuantity data = repository.findByProductId(productId);
        
        QuantityResponse response = QuantityResponse.builder()
            .quantity(data.getQuantity())
            .lastUpdated(data.getUpdatedAt())
            .build();
        
        if (featureToggle.isEnabled("EXTENDED_QUANTITY_FIELDS", clientId)) {
            response.setEstimatedQuantity(data.getEstimatedQuantity());
            response.setBackupQuantity(data.getBackupQuantity());
        }
        
        return response;
    }
}
```

### Best Practices

1. **Backward Compatibility**: Always support old API versions
2. **Nullable Fields**: New fields should be nullable or have defaults
3. **API Versioning**: Maintain multiple API versions simultaneously
4. **Feature Flags**: Use toggles to control feature rollout
5. **Gradual Migration**: Roll out to small percentage first
6. **Documentation**: Document API changes clearly
7. **Client Adaptation**: Frontend should handle optional fields

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

</details>
