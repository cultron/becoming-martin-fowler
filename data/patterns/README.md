# Data Patterns and Database Design

## Database Patterns

### Table Module Pattern
```java
public class OrderTable {
    private final Database db;
    
    public List<Order> findByCustomer(CustomerId customerId) {
        return db.query(
            "SELECT * FROM orders WHERE customer_id = ?",
            customerId
        ).map(this::mapToOrder);
    }
    
    public void updateStatus(OrderId orderId, OrderStatus status) {
        db.execute(
            "UPDATE orders SET status = ? WHERE id = ?",
            status, orderId
        );
    }
}
```

### Active Record Pattern
```java
public class Order extends ActiveRecord {
    private OrderStatus status;
    private CustomerId customerId;
    private Money total;
    
    public void save() {
        if (getId() == null) {
            db.execute(
                "INSERT INTO orders (customer_id, status, total) VALUES (?, ?, ?)",
                customerId, status, total
            );
        } else {
            db.execute(
                "UPDATE orders SET customer_id = ?, status = ?, total = ? WHERE id = ?",
                customerId, status, total, getId()
            );
        }
    }
    
    public static Order find(OrderId id) {
        return db.queryOne(
            "SELECT * FROM orders WHERE id = ?",
            id
        ).map(Order::mapFromRow);
    }
}
```

### Data Mapper Pattern
```java
public class OrderMapper {
    private final Database db;
    
    public Order findById(OrderId id) {
        OrderData data = db.queryOne(
            "SELECT * FROM orders WHERE id = ?",
            id
        );
        return mapToOrder(data);
    }
    
    public void save(Order order) {
        OrderData data = mapToData(order);
        if (data.getId() == null) {
            insert(data);
        } else {
            update(data);
        }
    }
    
    private Order mapToOrder(OrderData data) {
        Order order = new Order(new OrderId(data.getId()));
        order.setStatus(OrderStatus.valueOf(data.getStatus()));
        order.setCustomerId(new CustomerId(data.getCustomerId()));
        return order;
    }
}
```

## Database Design Patterns

### Schema Migration Pattern
```java
public class Migration_202401081234_CreateOrders implements Migration {
    public void up() {
        execute("""
            CREATE TABLE orders (
                id UUID PRIMARY KEY,
                customer_id UUID NOT NULL,
                status VARCHAR(50) NOT NULL,
                total_amount DECIMAL(10,2) NOT NULL,
                created_at TIMESTAMP NOT NULL,
                updated_at TIMESTAMP NOT NULL,
                FOREIGN KEY (customer_id) REFERENCES customers(id)
            )
        """);
    }
    
    public void down() {
        execute("DROP TABLE orders");
    }
}
```

### Event Sourcing Pattern
```java
public class OrderEventStore {
    private final Database db;
    
    public void store(OrderEvent event) {
        db.execute("""
            INSERT INTO order_events 
            (order_id, event_type, event_data, occurred_at)
            VALUES (?, ?, ?, ?)
        """, event.getOrderId(),
             event.getType(),
             event.getData(),
             event.getOccurredAt());
    }
    
    public List<OrderEvent> getEvents(OrderId orderId) {
        return db.query("""
            SELECT * FROM order_events
            WHERE order_id = ?
            ORDER BY occurred_at ASC
        """, orderId)
        .map(this::mapToEvent);
    }
}
```

### CQRS Pattern
```java
public interface OrderCommandRepository {
    void save(Order order);
    void delete(OrderId id);
}

public interface OrderQueryRepository {
    List<OrderSummary> findByCustomer(CustomerId customerId);
    OrderDetails findById(OrderId id);
    List<OrderSummary> findRecent(int limit);
}
```

## Data Access Patterns

### Repository Pattern
```java
public interface OrderRepository {
    Optional<Order> findById(OrderId id);
    List<Order> findByCustomer(CustomerId customerId);
    void save(Order order);
    void delete(OrderId id);
}

public class JpaOrderRepository implements OrderRepository {
    private final EntityManager em;
    
    public Optional<Order> findById(OrderId id) {
        return Optional.ofNullable(
            em.find(OrderEntity.class, id)
        ).map(this::mapToDomain);
    }
    
    public void save(Order order) {
        OrderEntity entity = mapToEntity(order);
        if (entity.getId() == null) {
            em.persist(entity);
        } else {
            em.merge(entity);
        }
    }
}
```

### Unit of Work Pattern
```java
public class UnitOfWork implements AutoCloseable {
    private final EntityManager em;
    private final Map<Object, Object> identityMap = new HashMap<>();
    
    public void registerNew(Object entity) {
        em.persist(entity);
        identityMap.put(entity.getId(), entity);
    }
    
    public void registerDirty(Object entity) {
        em.merge(entity);
    }
    
    public void registerDeleted(Object entity) {
        em.remove(entity);
        identityMap.remove(entity.getId());
    }
    
    public void commit() {
        em.getTransaction().commit();
    }
    
    public void rollback() {
        em.getTransaction().rollback();
    }
}
```

### Identity Map Pattern
```java
public class IdentityMap<T> {
    private final Map<Object, T> entities = new HashMap<>();
    
    public void add(Object key, T entity) {
        entities.put(key, entity);
    }
    
    public T get(Object key) {
        return entities.get(key);
    }
    
    public boolean contains(Object key) {
        return entities.containsKey(key);
    }
}
```