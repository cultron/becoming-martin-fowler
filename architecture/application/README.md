# Application Architecture

## What is an Application?

Martin Fowler defines applications as a social construction with three key aspects:

1. Code that developers see as a single unit
2. Functionality that business customers see as a single unit
3. An initiative that budget holders see as a single unit

## Key Patterns and Approaches

### GUI Architectures

1. Forms and Controls
   - Suitable for simple flows
   - Breaks down under complexity

2. Model-View-Controller (MVC)
   - Most misunderstood pattern
   - Key principles:
     - Separation of presentation from domain logic
     - State synchronization through events

### Presentation Domain Data Layering

- Three broad layers:
  1. Presentation (UI)
  2. Domain Logic (Business Logic)
  3. Data Access

### Feature Toggles

- Modify system behavior without changing code
- Categories of toggles:
  - Release toggles
  - Experiment toggles
  - Ops toggles
  - Permission toggles

## Modern Approaches

### Serverless Architecture

- Backend as a Service (BaaS)
- Functions as a Service (FaaS)
- Benefits:
  - Reduced operational cost
  - Reduced complexity
  - Faster engineering lead time

### Micro Frontends

- Breaking up frontend monoliths
- Benefits:
  - Team scalability
  - Independent deployment
  - Technology flexibility

## Best Practices

1. Maintain clear boundaries
2. Design for change
3. Keep modules cohesive
4. Practice continuous refactoring
5. Consider scalability early