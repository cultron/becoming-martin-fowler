# Architectural Quality and Technical Debt

## Understanding Technical Debt

Technical debt is a metaphor developed by Ward Cunningham to help think about the refactoring work that needs to be done in a codebase. Like financial debt, technical debt accumulates interest over time and needs to be actively managed.

### Types of Technical Debt

1. Deliberate Technical Debt
   - Conscious decisions to optimize for the present
   - Should be documented and tracked
   - Needs a repayment plan

2. Inadvertent Technical Debt
   - Results from poor choices
   - Often due to inexperience
   - Discovered after the fact

3. Cruft
   - Gradual decay of software quality
   - Results from changes without refactoring
   - Makes code harder to understand and modify

## Measuring Quality

### Code Quality Metrics
```java
public class CodeMetrics {
    public static class ClassMetrics {
        private final int linesOfCode;
        private final int methodCount;
        private final double cyclomaticComplexity;
        private final double efferentCoupling;
        private final double afferentCoupling;
        
        public double calculateInstability() {
            return efferentCoupling / (efferentCoupling + afferentCoupling);
        }
        
        public boolean exceedsComplexityThreshold() {
            return cyclomaticComplexity > 10 ||
                   methodCount > 20 ||
                   linesOfCode > 500;
        }
    }
}
```

### Architecture Metrics
```java
public class ArchitectureMetrics {
    public static class ComponentMetrics {
        private final int abstractClasses;
        private final int concreteClasses;
        private final double efferentCoupling;
        private final double afferentCoupling;
        
        public double calculateAbstractness() {
            return abstractClasses / (double)(abstractClasses + concreteClasses);
        }
        
        public double calculateInstability() {
            return efferentCoupling / (efferentCoupling + afferentCoupling);
        }
        
        public double calculateDistance() {
            return Math.abs(calculateAbstractness() + calculateInstability() - 1);
        }
    }
}
```

## Quality Patterns

### Clean Architecture
```java
public interface UseCase<I extends UseCase.InputPort, O extends UseCase.OutputPort> {
    void execute(I input, O output);
    
    interface InputPort {}
    interface OutputPort {}
}

public class CreateOrderUseCase implements UseCase<CreateOrderUseCase.Input, CreateOrderUseCase.Output> {
    private final OrderRepository orderRepository;
    private final CustomerRepository customerRepository;
    
    @Override
    public void execute(Input input, Output output) {
        Customer customer = customerRepository.findById(input.getCustomerId())
            .orElseThrow(() -> new CustomerNotFoundException(input.getCustomerId()));
            
        Order order = new Order(customer);
        input.getItems().forEach(item -> 
            order.addItem(item.getProductId(), item.getQuantity()));
            
        orderRepository.save(order);
        output.orderCreated(order.getId());
    }
    
    public static class Input implements UseCase.InputPort {
        private final CustomerId customerId;
        private final List<OrderItem> items;
    }
    
    public interface Output extends UseCase.OutputPort {
        void orderCreated(OrderId orderId);
    }
}
```

### Ports and Adapters
```java
public interface OrderPort {
    void submitOrder(Order order);
    OrderStatus checkStatus(OrderId orderId);
}

public class RestOrderAdapter implements OrderPort {
    private final RestTemplate restTemplate;
    private final String baseUrl;
    
    @Override
    public void submitOrder(Order order) {
        OrderDto dto = OrderDto.from(order);
        restTemplate.postForEntity(baseUrl + "/orders", dto, Void.class);
    }
}

public class JmsOrderAdapter implements OrderPort {
    private final JmsTemplate jmsTemplate;
    private final String queueName;
    
    @Override
    public void submitOrder(Order order) {
        OrderMessage message = OrderMessage.from(order);
        jmsTemplate.convertAndSend(queueName, message);
    }
}
```

## Dealing with Technical Debt

### Refactoring Strategy
```java
public class RefactoringPlan {
    private final List<RefactoringItem> items;
    private final Set<String> affectedModules;
    
    public double estimateEffort() {
        return items.stream()
            .mapToDouble(RefactoringItem::getEstimatedEffort)
            .sum();
    }
    
    public double calculateRiskScore() {
        return items.stream()
            .mapToDouble(item -> 
                item.getComplexity() * item.getImpact())
            .average()
            .orElse(0.0);
    }
    
    public List<RefactoringItem> prioritize() {
        return items.stream()
            .sorted(Comparator.comparing(
                item -> item.getValue() / item.getEstimatedEffort(),
                Comparator.reverseOrder()))
            .collect(Collectors.toList());
    }
}
```

### Quality Gates
```java
public interface QualityGate {
    boolean check(Codebase codebase);
    List<Violation> getViolations();
}

public class ComplexityGate implements QualityGate {
    private final int maxMethodComplexity;
    private final int maxClassComplexity;
    
    @Override
    public boolean check(Codebase codebase) {
        violations = codebase.getMethods().stream()
            .filter(method -> method.getComplexity() > maxMethodComplexity)
            .map(method -> new Violation("Method too complex", method))
            .collect(Collectors.toList());
            
        violations.addAll(codebase.getClasses().stream()
            .filter(clazz -> clazz.getComplexity() > maxClassComplexity)
            .map(clazz -> new Violation("Class too complex", clazz))
            .collect(Collectors.toList()));
            
        return violations.isEmpty();
    }
}
```