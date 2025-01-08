# Enterprise Integration Patterns

## Core Integration Styles

### 1. File Transfer
- Simple and universal
- Batch processing
- Potential data consistency issues
- Good for large data volumes

### 2. Shared Database
- Strong consistency
- Tight coupling
- Performance bottleneck risk
- Schema evolution challenges

### 3. Remote Procedure Call
- Synchronous communication
- Natural programming model
- Tight coupling to interface
- Network dependency

### 4. Messaging
- Asynchronous communication
- Loose coupling
- Complex error handling
- Event-driven architecture support

## Message Patterns

### Message Construction
1. Command Message
   - Request for action
   - Expects specific outcome

2. Event Message
   - Notification of occurrence
   - No specific expectation

3. Document Message
   - Data transfer
   - No behavioral expectations

### Message Routing
1. Content-Based Router
```java
public class OrderRouter {
    public void route(Order order) {
        if (order.isHighValue()) {
            priorityProcessor.process(order);
        } else if (order.isInternational()) {
            internationalProcessor.process(order);
        } else {
            standardProcessor.process(order);
        }
    }
}
```

2. Message Filter
```java
public class OrderFilter {
    public boolean shouldProcess(Order order) {
        return order.getValue().isGreaterThan(threshold)
            && !order.isTestOrder();
    }
}
```

### Message Transformation
1. Message Translator
```java
public class OrderTranslator {
    public ExternalOrderFormat translate(InternalOrder order) {
        return new ExternalOrderFormat()
            .setOrderNumber(order.getId().toString())
            .setCustomerInfo(translateCustomer(order.getCustomer()))
            .setItems(translateItems(order.getLines()));
    }
}
```

2. Content Enricher
```java
public class OrderEnricher {
    public EnrichedOrder enrich(Order order) {
        CustomerDetails details = customerService.getDetails(order.getCustomerId());
        ShippingRate rate = shippingCalculator.calculateFor(order);
        return new EnrichedOrder(order, details, rate);
    }
}
```

## Integration Patterns

### 1. Gateway
```java
public interface PaymentGateway {
    PaymentResult process(Payment payment);
    PaymentStatus checkStatus(PaymentId id);
    void refund(PaymentId id, Money amount);
}
```

### 2. Adapter
```java
public class LegacySystemAdapter implements OrderSystem {
    private final LegacyOrderSystem legacy;
    
    public Order findOrder(OrderId id) {
        LegacyOrder legacyOrder = legacy.findOrderById(id.toString());
        return convertToModernOrder(legacyOrder);
    }
}
```

### 3. Service Interface
```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {
        Order order = orderService.createOrder(request);
        return ResponseEntity.ok(OrderResponse.from(order));
    }
}
```

## Best Practices

1. Error Handling
```java
public class RetryableOperation {
    public <T> T execute(Supplier<T> operation) {
        int attempts = 0;
        while (attempts < maxAttempts) {
            try {
                return operation.get();
            } catch (RetryableException e) {
                attempts++;
                if (attempts == maxAttempts) {
                    throw new MaxRetriesExceededException(e);
                }
                backoff(attempts);
            }
        }
        throw new IllegalStateException("Should not reach here");
    }
}
```

2. Circuit Breaker
```java
public class CircuitBreaker {
    private final long timeout;
    private final long retryTimeout;
    private final int failureThreshold;
    
    private int failures;
    private long lastFailureTime;
    private State state = State.CLOSED;
    
    public <T> T execute(Supplier<T> operation) {
        if (state == State.OPEN) {
            if (System.currentTimeMillis() - lastFailureTime > retryTimeout) {
                state = State.HALF_OPEN;
            } else {
                throw new CircuitBreakerOpenException();
            }
        }
        
        try {
            T result = operation.get();
            reset();
            return result;
        } catch (Exception e) {
            recordFailure();
            throw e;
        }
    }
}
```

3. Message Reliability
```java
public class ReliableMessageSender {
    public void send(Message message) {
        String messageId = messageStore.store(message);
        try {
            messageBroker.send(message);
            messageStore.markAsSent(messageId);
        } catch (Exception e) {
            messageStore.markAsFailedToSend(messageId);
            throw new MessageDeliveryException(messageId, e);
        }
    }
}
```