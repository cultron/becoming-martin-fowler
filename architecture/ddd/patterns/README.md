# Domain-Driven Design Patterns

## Strategic Patterns

### Context Mapping Patterns

#### Anti-corruption Layer
```java
// External legacy system interface
class LegacyCustomerSystem {
    LegacyCustomer findCustomerById(String customerId);
    void updateCustomer(LegacyCustomer customer);
}

// Anti-corruption layer
class CustomerAdapter {
    private LegacyCustomerSystem legacySystem;
    
    Customer findCustomer(CustomerId id) {
        LegacyCustomer legacyCustomer = legacySystem.findCustomerById(id.toString());
        return translateFromLegacy(legacyCustomer);
    }
    
    private Customer translateFromLegacy(LegacyCustomer legacy) {
        return new Customer(
            new CustomerId(legacy.getId()),
            new CustomerName(legacy.getFirstName(), legacy.getLastName()),
            new EmailAddress(legacy.getEmail())
        );
    }
}
```

#### Published Language
```java
// Shared format for integration
public class CustomerData {
    private final String id;
    private final String fullName;
    private final String email;
    private final CustomerStatus status;
    
    // Value object with validation
    public static CustomerData fromJson(String json) {
        // Parse and validate JSON format
    }
    
    public String toJson() {
        // Convert to agreed JSON format
    }
}
```

### Domain Patterns

#### Entity
```java
public class Order {
    private final OrderId id;
    private Customer customer;
    private List<OrderLine> orderLines;
    private OrderStatus status;
    
    public void addProduct(Product product, int quantity) {
        // Business logic for adding products
    }
    
    public Money calculateTotal() {
        // Business logic for total calculation
    }
    
    public void submit() {
        // Business rules for order submission
    }
}
```

#### Value Object
```java
public class Money {
    private final BigDecimal amount;
    private final Currency currency;
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(amount.add(other.amount), currency);
    }
    
    public Money multiply(int multiplier) {
        return new Money(amount.multiply(BigDecimal.valueOf(multiplier)), currency);
    }
}
```

#### Aggregate Root
```java
public class Order {
    private final OrderId id;
    private final List<OrderLine> orderLines;
    
    public void addProduct(Product product, int quantity) {
        validateProductAvailability(product, quantity);
        orderLines.add(new OrderLine(product, quantity));
    }
    
    // Only the aggregate root can modify its children
    private void validateProductAvailability(Product product, int quantity) {
        // Business rules for product availability
    }
}
```

#### Domain Event
```java
public class OrderPlaced extends DomainEvent {
    private final OrderId orderId;
    private final CustomerId customerId;
    private final Money totalAmount;
    private final LocalDateTime placedAt;
    
    // Events are immutable and capture what happened
    public OrderPlaced(OrderId orderId, CustomerId customerId, Money totalAmount) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.totalAmount = totalAmount;
        this.placedAt = LocalDateTime.now();
    }
}
```

#### Factory
```java
public class OrderFactory {
    private final ProductRepository products;
    private final CustomerRepository customers;
    
    public Order createOrder(CustomerId customerId, List<OrderLineDto> orderLines) {
        Customer customer = customers.findById(customerId)
            .orElseThrow(() -> new CustomerNotFoundException(customerId));
            
        Order order = new Order(new OrderId(), customer);
        
        for (OrderLineDto line : orderLines) {
            Product product = products.findById(line.getProductId())
                .orElseThrow(() -> new ProductNotFoundException(line.getProductId()));
            order.addProduct(product, line.getQuantity());
        }
        
        return order;
    }
}
```

#### Repository
```java
public interface OrderRepository {
    Optional<Order> findById(OrderId id);
    void save(Order order);
    void delete(OrderId id);
    List<Order> findByCustomer(CustomerId customerId);
}

// Implementation with JPA
@Repository
public class JpaOrderRepository implements OrderRepository {
    private final JpaOrderEntityRepository jpaRepository;
    private final OrderMapper mapper;
    
    @Override
    public Optional<Order> findById(OrderId id) {
        return jpaRepository.findById(id.toString())
            .map(mapper::toDomain);
    }
    
    @Override
    public void save(Order order) {
        OrderEntity entity = mapper.toEntity(order);
        jpaRepository.save(entity);
    }
}
```

#### Specification
```java
public interface OrderSpecification {
    boolean isSatisfiedBy(Order order);
}

public class HighValueOrderSpecification implements OrderSpecification {
    private final Money threshold;
    
    public boolean isSatisfiedBy(Order order) {
        return order.calculateTotal().isGreaterThan(threshold);
    }
}

public class RushOrderSpecification implements OrderSpecification {
    public boolean isSatisfiedBy(Order order) {
        return order.getDeliveryDate().isBefore(LocalDate.now().plusDays(2));
    }
}
```