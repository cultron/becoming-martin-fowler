# Microservices Patterns

## Core Patterns

### 1. Database per Service
Each microservice has its own private database, ensuring loose coupling and enabling independent evolution.

### 2. Event-Driven Architecture
Services communicate through events, enabling loose coupling and better scalability.

### 3. API Gateway
Provides a single entry point for clients, handling cross-cutting concerns like authentication and routing.

### 4. Circuit Breaker
Prevents cascading failures by failing fast when a service is unavailable.

### 5. Service Registry
Enables service discovery in a dynamic environment.

## Implementation Patterns

### 1. Strangler Fig Pattern
Gradually migrate from a monolith to microservices by incrementally replacing functionality.

### 2. Sidecar Pattern
Deploy components of an application as separate containers/processes that run alongside the primary service.

### 3. Bulkhead Pattern
Isolate elements of an application into pools so that if one fails, the others will continue to function.

## Testing Patterns

### 1. Consumer-Driven Contract Testing
Ensure services meet their consumers' expectations without requiring end-to-end testing.

### 2. Service Virtualization
Simulate the behavior of components to enable testing in isolation.

## Deployment Patterns

### 1. Blue-Green Deployment
Reduce downtime and risk by running two identical production environments.

### 2. Canary Releases
Test changes on a small subset of users before full deployment.