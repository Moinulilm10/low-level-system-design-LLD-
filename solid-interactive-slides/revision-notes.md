# SOLID Principles Revision Notes

## What SOLID means

SOLID is a set of five object-oriented design principles that help keep business code flexible, testable, and maintainable.

Use SOLID when the code is expected to change, when business rules are complex, or when infrastructure dependencies make tests difficult. Do not force SOLID into tiny scripts or stable code where a simple function is clearer.

## S: Single Responsibility Principle

Definition: A class should have one reason to change.

Use when:
- A class mixes business rules, database code, API calls, formatting, and notifications.
- A bug fix in one concern risks breaking another concern.

Where to use:
- Domain services
- Validators
- Formatters
- Repositories
- Notification services

JavaScript example:

```js
class TaxCalculator {
  calculate(order) {
    return order.total * 0.1;
  }
}

class OrderRepository {
  save(order) {
    // Save order
  }
}

class OrderService {
  constructor(taxCalculator, repository) {
    this.taxCalculator = taxCalculator;
    this.repository = repository;
  }

  place(order) {
    order.tax = this.taxCalculator.calculate(order);
    this.repository.save(order);
  }
}
```

Interview smell: "This class does too many unrelated things."

## O: Open Closed Principle

Definition: Software entities should be open for extension but closed for modification.

Use when:
- New variants keep appearing.
- You often edit an old switch or if/else chain.

Where to use:
- Discount strategies
- Payment methods
- Report exporters
- Notification channels
- Tax rules

JavaScript example:

```js
class CardPayment {
  pay(amount) {
    console.log(`Paid ${amount} by card`);
  }
}

class BkashPayment {
  pay(amount) {
    console.log(`Paid ${amount} by bKash`);
  }
}

class PaymentService {
  constructor(paymentMethod) {
    this.paymentMethod = paymentMethod;
  }

  checkout(amount) {
    this.paymentMethod.pay(amount);
  }
}
```

Java example:

```java
interface PaymentMethod {
  void pay(BigDecimal amount);
}

class CardPayment implements PaymentMethod {
  public void pay(BigDecimal amount) {
    // Card payment
  }
}

class PaymentService {
  private final PaymentMethod paymentMethod;

  PaymentService(PaymentMethod paymentMethod) {
    this.paymentMethod = paymentMethod;
  }

  void checkout(BigDecimal amount) {
    paymentMethod.pay(amount);
  }
}
```

Interview smell: "Every new type forces us to edit existing tested code."

## L: Liskov Substitution Principle

Definition: A subtype should be safely usable wherever the parent type is expected.

Use when:
- Inheritance looks convenient but behavior is not compatible.
- A subclass throws unsupported-operation errors.

Where to use:
- Domain inheritance
- Account types
- User roles
- File abstractions
- Shape or geometry models

Bad example:

```js
class Bird {
  fly() {}
}

class Penguin extends Bird {
  fly() {
    throw new Error("Penguins cannot fly");
  }
}
```

Better example:

```js
class FlyingBird {
  fly() {}
}

class Sparrow extends FlyingBird {
  fly() {
    console.log("Flying");
  }
}

class Penguin {
  swim() {
    console.log("Swimming");
  }
}
```

Interview smell: "The child class breaks the parent's promise."

## I: Interface Segregation Principle

Definition: Clients should not be forced to depend on methods they do not use.

Use when:
- Implementations contain empty methods.
- Classes throw "not supported" because an interface is too large.

Where to use:
- Device capabilities
- Repository read/write interfaces
- Permission systems
- Workflow roles

Java example:

```java
interface Printer {
  void print(Document document);
}

interface Scanner {
  Document scan();
}

class SimplePrinter implements Printer {
  public void print(Document document) {
    // Print only
  }
}
```

Interview smell: "This interface is forcing clients to know too much."

## D: Dependency Inversion Principle

Definition: High-level business logic should depend on abstractions, not concrete low-level details.

Use when:
- Business logic creates database clients, HTTP clients, file systems, or payment SDKs directly.
- Tests require real infrastructure.

Where to use:
- Application services
- Controllers
- Domain workflows
- External integration boundaries

JavaScript example:

```js
class OrderService {
  constructor(paymentGateway) {
    this.paymentGateway = paymentGateway;
  }

  place(order) {
    this.paymentGateway.charge(order.total);
  }
}

class StripeGateway {
  charge(amount) {
    // Stripe API call
  }
}
```

Interview smell: "Business code is tightly coupled to infrastructure."

## Pros

- Easier maintenance because changes are localized.
- Better testability because dependencies can be mocked.
- Easier extension through strategies, adapters, and policies.
- Clearer business logic because concepts get explicit names.
- Lower regression risk in complex systems.

## Cons

- Can create too many files and abstractions.
- Can be over-engineered for simple CRUD or throwaway scripts.
- Bad abstractions are expensive to change.
- Developers need discipline to keep interfaces meaningful.
- Navigation can become harder if every tiny behavior is split too early.

## Business logic efficiency

SOLID helps business logic because it separates stable workflow from volatile policy.

Example: In checkout, payment provider, discount rule, tax rule, and notification channel can change independently.

Business benefits:
- Faster rule changes
- Easier testing
- Safer vendor migration
- Cleaner ownership across teams
- Less regression during releases

## Interview answer template

Use this structure:

1. Identify the smell.
2. Name the SOLID principle.
3. Explain why the current design is risky.
4. Propose a small refactor.
5. Mention the tradeoff.

Example answer:

"The checkout class violates SRP and DIP because it calculates discounts, creates a Stripe client, saves to the database, and sends email. I would extract discount policy, payment gateway, repository, and notifier abstractions. Checkout would orchestrate the use case, while implementations handle infrastructure. This improves testability and makes payment provider changes safer, but I would avoid the extra abstraction for a tiny prototype."

## Problem-based interview questions

1. A checkout service calculates tax, charges a card, saves an order, and sends a receipt. Which principles are violated and how would you refactor it?
2. A report generator has a switch statement for PDF, CSV, XLSX, and JSON. Add XML without modifying the generator.
3. A read-only file class extends a writable file class but throws an error from write. What is wrong?
4. A basic printer must implement scan, fax, and staple methods. Which principle applies?
5. A billing service creates a concrete Stripe client inside the method. How would you make it unit-testable?
6. A discount module changes every week and breaks checkout tests. Which design would isolate the business rule?
7. A service depends on five unrelated methods from one interface but only calls one. What refactor would you suggest?
8. A subclass changes method preconditions and accepts fewer valid inputs than the parent. Which principle is violated?
9. You see 20 small interfaces in a simple CRUD app. Is this good SOLID design? Why or why not?
10. How are dependency inversion and dependency injection related but different?

## Fast memory hooks

- SRP: one reason to change.
- OCP: add new behavior by adding code, not editing stable code.
- LSP: child must keep the parent's promise.
- ISP: small interfaces for specific clients.
- DIP: business rules should not depend on tools.

