# learning-path

**Interview preparation**:

- ### Microservices Integration
- **31. How do you implement inter-service communication?**   REST (RestTemplate/WebClient), gRPC, Kafka. Prefer Feign for declarative REST. 
- **32. What is the difference between synchronous and asynchronous calls?**   Sync waits for response; async uses callbacks or messaging (Kafka/RabbitMQ). 
- **33. How do you implement retry, fallback, and circuit breaker?**   Use Resilience4j or Spring Cloud Circuit Breaker with fallback methods. 
- **34. How do you ensure backward compatibility between services?**   Versioned endpoints (`/v1/api`), contract testing (Pact), schema evolution tools. 
- **35. What’s the role of API Gateway in microservice architecture?**   Routes requests, applies auth, rate limits, transforms payloads.

### Security & Resilience

- **36. How do you secure REST endpoints?**   JWT with Spring Security, OAuth2 for user-based access.
- **37. Difference between OAuth2 and JWT?**   OAuth2 is a protocol. JWT is a token format. OAuth2 may use JWT for tokens.
- **38. How do you implement RBAC in Spring Security?**   Map roles to endpoints using annotations or config DSL.
- **39. How do you protect microservices against DDOS or abuse?**   Rate limiting, circuit breakers, Web Application Firewall (WAF).
- **40. What is rate limiting and how do you implement it?**   Restricts request volume. Use Bucket4j, Redis-based counters, API Gateway filters.