# Cloud architecture

## Designing microservice architecture

1. **Decomposing business problem.** O -> o,o,o,o
2. **Establishing service granularity.** o o o o
3. **Defining the service interfaces.** Web | API | API

### \*Decomposing business problem

O -> o,o,o,o

Break big problem into several small ones === break monolithic domain into MS domain.

The nouns you use becoming domain models.

The verbs you use becoming MS operations.

Look for data relation (strong, weak)


### \*Service granularity

Shouldn't take too much responsability. Shouldn't be too fine grained (CRUD).

1 service - 1 DB and about 5 tables.

**Focus on service-to-service interactions**

### Defining the service interface

Embrace REST

Use URIs to communicate

Use JSON - lightweight

HTTP codes

## When no to use MS

1. Monolith will be faster.
2. Too expensive to mighrate.
3. Too difficult to control.
4. Not suitable for small projects.
5. Transactions matter.
6. Hard to orchestrate containers.
