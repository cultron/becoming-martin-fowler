# Microservices Anti-Patterns

## Design Anti-Patterns

### 1. Distributed Monolith
Services that are deployed independently but are tightly coupled, combining the disadvantages of both architectural styles.

Signs:
- Services share databases
- Changes require coordinated deployment
- Strong runtime dependencies

### 2. Wrong Service Boundaries
Services that don't align with business capabilities, leading to tight coupling and complex dependencies.

Signs:
- Services frequently updated together
- Complex dependency graphs
- Data consistency issues

### 3. Shared Libraries Overuse
Too much shared code between services, creating hidden coupling.

## Operational Anti-Patterns

### 1. No API Gateway
Exposing internal services directly, leading to security and maintenance issues.

### 2. No Service Discovery
Hardcoding service locations, making the system brittle and hard to change.

### 3. Inadequate Monitoring
Not having proper observability into the distributed system.

## Development Anti-Patterns

### 1. Starting with Microservices
Building a new system as microservices before understanding the domain.

### 2. No CI/CD Pipeline
Manual deployment processes that can't handle the complexity of microservices.

### 3. Ignoring Conway's Law
Not aligning team structure with the desired architecture.