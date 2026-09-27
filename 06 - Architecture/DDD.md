**Domain-Driven Design (DDD)** is a software-design approach where the system’s structure and terminology are based on the **business domain** and its rules.

It is especially useful for applications with complex business logic, such as banking, insurance, logistics, healthcare, or e-commerce.

> DDD is not primarily about databases. It is about modeling business concepts and behavior in software.

## Core idea

Developers and domain experts—such as product owners, accountants, or logistics specialists—build a shared model and vocabulary.

For example, in an e-commerce domain, terms might include:

- Order
- Customer
- Product
- Payment
- Shipment
- Refund

These terms should have consistent meanings in conversations, requirements, code, and documentation. This is called **ubiquitous language**.

## Strategic DDD concepts

### 1. Domain and subdomains

A large business domain is divided into smaller subdomains.

For an e-commerce platform:

- **Ordering**
- **Payments**
- **Inventory**
- **Shipping**
- **Customer support**

Subdomains are often classified as:

- **Core domain:** The primary business differentiator.
- **Supporting subdomain:** Necessary but not the main competitive advantage.
- **Generic subdomain:** A standard capability that can often use an existing product, such as authentication.

### 2. Bounded context

A **bounded context** is a boundary within which a domain model and its terminology have a precise meaning.

For example, “customer” might mean:

- A buyer with a shipping address in the Ordering context.
- An account with payment risk in the Billing context.
- A ticket requester in the Support context.

These contexts can use separate models instead of forcing one large universal `Customer` model.

### 3. Context map

A context map describes how bounded contexts interact—for example, whether they exchange events, call APIs, or translate one model into another.

## Tactical DDD building blocks

### Entity

An object identified by its identity rather than only by its attributes.

```text
Order(id=123)
```

An order remains the same entity even if its status or shipping address changes.

### Value object

An immutable object defined by its values and without an independent identity.

```text
Money(amount=100, currency="USD")
Address(city="London", postalCode="SW1A")
```

Two `Money(100, "USD")` values are considered equivalent.

### Aggregate

A group of related entities and value objects treated as one consistency boundary.

For example:

```text
Order aggregate
├── Order
├── OrderLine
└── ShippingAddress
```

The **aggregate root** is the only external entry point—in this case, `Order`.

A business rule could be:

> An order cannot be modified after it has shipped.

The aggregate enforces this rule instead of allowing arbitrary updates to its internal objects.

### Repository

Provides access to aggregates while hiding persistence details.

```text
orderRepository.findById(orderId)
orderRepository.save(order)
```

A repository is usually defined in the domain/application layer and implemented using PostgreSQL, an ORM, or another persistence mechanism.

### Domain service

Contains domain logic that does not naturally belong to one entity or value object.

Example:

```text
PricingService.calculateDiscount(customer, order)
```

### Domain event

Represents a business event that has already occurred.

Examples:

```text
OrderPlaced
PaymentReceived
OrderShipped
```

Other bounded contexts can react to these events.

## Simple example

Instead of writing procedural database-oriented code:

```text
UPDATE orders SET status = 'shipped' WHERE id = 123;
```

DDD-style application code might express the business operation:

```text
order = orderRepository.findById(123)
order.ship()
orderRepository.save(order)
```

The `ship()` method can enforce rules:

```text
- Payment must be completed.
- The order must contain at least one item.
- A cancelled order cannot be shipped.
```

The emphasis is on modeling **business behavior**, not merely updating data.

## DDD and microservices

DDD and microservices are related, but they are not the same.

- A **bounded context** is a logical business/model boundary.
- A **microservice** is a deployment boundary.
- A bounded context may become one service, but this is not mandatory.
- DDD can be used in a modular monolith.

A common mistake is to create one microservice per entity or database table. Services should generally align with business capabilities, not individual tables.

## When to use DDD

DDD is helpful when:

- Business rules are complex and frequently change.
- Different departments use terms differently.
- The application has many workflows and invariants.
- Domain experts can collaborate with developers.

It may be unnecessary for:

- Simple CRUD applications
- Basic reporting tools
- Small applications with little business logic

DDD introduces modeling and architectural overhead, so it should be applied where domain complexity justifies it.