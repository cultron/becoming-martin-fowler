# Refactoring Patterns

## Basic Refactoring Patterns

### Extract Method
```java
void printOwing() {
  printBanner();
  
  // Print details
  System.out.println("name: " + name);
  System.out.println("amount: " + getOutstanding());
}

// Refactored to:
void printOwing() {
  printBanner();
  printDetails();
}

void printDetails() {
  System.out.println("name: " + name);
  System.out.println("amount: " + getOutstanding());
}
```

### Move Method
Moving a method to where it's most used, typically to reduce coupling.

### Replace Temp with Query
```java
double calculateTotal() {
  double basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  return basePrice * 0.98;
}

// Refactored to:
double calculateTotal() {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  return basePrice() * 0.98;
}

double basePrice() {
  return quantity * itemPrice;
}
```

## Object-Oriented Refactoring Patterns

### Replace Type Code with State/Strategy
Replacing type codes with state objects or strategy pattern implementation.

### Replace Conditional with Polymorphism
```java
class Bird {
  double getSpeed() {
    switch (type) {
      case EUROPEAN:
        return getBaseSpeed();
      case AFRICAN:
        return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return isNailed ? 0 : getBaseSpeed(voltage);
    }
    throw new RuntimeException("Should be unreachable");
  }
}

// Refactored to:
abstract class Bird {
  abstract double getSpeed();
}

class European extends Bird {
  double getSpeed() {
    return getBaseSpeed();
  }
}

class African extends Bird {
  double getSpeed() {
    return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
  }
}
```

## Database Refactoring Patterns

### Migrate Method
Moving business logic from database (stored procedures, triggers) to application code.

### Split Table
Breaking a table that serves multiple purposes into focused tables.

## Architecture Refactoring Patterns

### Strangler Fig Pattern
Gradually creating a new system around the edges of the old, letting it grow and eventually strangle the old system.

### Extract Service
Moving functionality into a separate service.

## Testing Patterns for Refactoring

### Characterization Tests
Writing tests that characterize the current behavior of the system before refactoring.

### Breaking Dependencies
Techniques for breaking dependencies to make code testable:
- Extract Interface
- Dependency Injection
- Parameter Objects