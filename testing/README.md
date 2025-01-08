# Software Testing Patterns and Practices

## Test Types

### Unit Tests
- Test individual components in isolation
- Fast execution
- High maintainability
- Guide design through TDD

### Integration Tests
- Test component interactions
- Use real dependencies
- Slower than unit tests
- More complex setup

### End-to-End Tests
- Test complete features
- User perspective
- Most complex setup
- Slowest execution

## Test Organization

### Test Double Patterns

1. Stub
```java
public class PaymentGatewayStub implements PaymentGateway {
    public PaymentResult process(Payment payment) {
        return new PaymentResult(true, "TEST-TRANSACTION-ID");
    }
}
```

2. Mock
```java
public class OrderNotifierMock implements OrderNotifier {
    private List<Order> notifiedOrders = new ArrayList<>();
    
    public void notifyOrderShipped(Order order) {
        notifiedOrders.add(order);
    }
    
    public void verify(Order order) {
        assertTrue(notifiedOrders.contains(order));
    }
}
```

3. Fake
```java
public class InMemoryRepository<T> implements Repository<T> {
    private Map<String, T> storage = new HashMap<>();
    
    public void save(String id, T entity) {
        storage.put(id, entity);
    }
    
    public Optional<T> findById(String id) {
        return Optional.ofNullable(storage.get(id));
    }
}
```

## Testing Patterns

### Given-When-Then
```java
@Test
public void shouldCalculateOrderTotal() {
    // Given
    Order order = new Order();
    order.addLine(new Product("A", Money.of(10)), 2);
    order.addLine(new Product("B", Money.of(15)), 1);
    
    // When
    Money total = order.calculateTotal();
    
    // Then
    assertEquals(Money.of(35), total);
}
```

### Builder Pattern for Test Data
```java
public class OrderBuilder {
    private CustomerId customerId;
    private List<OrderLine> lines = new ArrayList<>();
    private OrderStatus status = OrderStatus.NEW;

    public static OrderBuilder anOrder() {
        return new OrderBuilder();
    }

    public OrderBuilder forCustomer(CustomerId customerId) {
        this.customerId = customerId;
        return this;
    }

    public OrderBuilder withLine(Product product, int quantity) {
        lines.add(new OrderLine(product, quantity));
        return this;
    }

    public OrderBuilder withStatus(OrderStatus status) {
        this.status = status;
        return this;
    }

    public Order build() {
        Order order = new Order(customerId);
        lines.forEach(line -> order.addLine(line.getProduct(), line.getQuantity()));
        order.setStatus(status);
        return order;
    }
}
```

### Test Data Builders
```java
public class TestDataBuilders {
    public static Customer aCustomer() {
        return new Customer(
            new CustomerId(UUID.randomUUID()),
            new CustomerName("John", "Doe"),
            new EmailAddress("john@example.com")
        );
    }

    public static Product aProduct() {
        return new Product(
            new ProductId(UUID.randomUUID()),
            "Test Product",
            Money.of(10),
            ProductCategory.GENERAL
        );
    }
}
```

### Object Mother Pattern
```java
public class OrderMother {
    public static Order simpleOrder() {
        return OrderBuilder.anOrder()
            .forCustomer(CustomerMother.standardCustomer().getId())
            .withLine(ProductMother.standardProduct(), 1)
            .build();
    }

    public static Order complexOrder() {
        return OrderBuilder.anOrder()
            .forCustomer(CustomerMother.premiumCustomer().getId())
            .withLine(ProductMother.expensiveProduct(), 2)
            .withLine(ProductMother.standardProduct(), 3)
            .withStatus(OrderStatus.CONFIRMED)
            .build();
    }
}
```

## Test Categories

### Contract Tests
```java
public abstract class RepositoryContractTest<T extends Repository> {
    protected abstract T createRepository();
    protected abstract Entity createEntity();

    @Test
    public void shouldSaveAndRetrieve() {
        T repository = createRepository();
        Entity entity = createEntity();

        repository.save(entity);
        Optional<Entity> retrieved = repository.findById(entity.getId());

        assertTrue(retrieved.isPresent());
        assertEquals(entity, retrieved.get());
    }
}
```

### Property-Based Tests
```java
@Property
void concatenatingListsShouldPreserveLength(List<Integer> list1, List<Integer> list2) {
    List<Integer> concatenated = Stream.concat(
        list1.stream(),
        list2.stream()
    ).collect(Collectors.toList());

    assertEquals(
        list1.size() + list2.size(),
        concatenated.size()
    );
}
```

### Test Lifecycle
```java
public class OrderServiceTest {
    private OrderService orderService;
    private OrderRepository orderRepository;
    private CustomerRepository customerRepository;

    @BeforeEach
    void setUp() {
        orderRepository = new InMemoryOrderRepository();
        customerRepository = new InMemoryCustomerRepository();
        orderService = new OrderService(orderRepository, customerRepository);
    }

    @Test
    void shouldCreateOrder() {
        Customer customer = CustomerMother.standardCustomer();
        customerRepository.save(customer);

        OrderRequest request = new OrderRequest(customer.getId());
        Order order = orderService.createOrder(request);

        assertNotNull(order.getId());
        assertEquals(customer.getId(), order.getCustomerId());
    }
}
```