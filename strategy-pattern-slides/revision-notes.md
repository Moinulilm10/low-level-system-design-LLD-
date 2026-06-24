# Strategy Design Pattern Revision Notes

## Definition

Strategy is a behavioral design pattern that defines a family of algorithms, encapsulates each algorithm in a separate class or object, and makes them interchangeable through a common contract.

Simple meaning: the main workflow stays the same, but the algorithm can change.

## Key participants

- Context: the class that uses the strategy.
- Strategy: the interface or expected method shape.
- Concrete strategy: a real implementation of the algorithm.
- Client: the code that selects and injects the strategy.

## When to use

Use Strategy when:

- You have multiple ways to do the same action.
- A class has many behavior-based `if`, `else if`, or `switch` branches.
- Algorithms change independently from the main workflow.
- You need to choose behavior at runtime.
- You want easier unit tests for business rules.

Avoid Strategy when:

- There is only one simple algorithm.
- The behavior is stable and unlikely to change.
- Extra classes would make the code harder to understand.

## Where to use

Common business use cases:

- Discount calculation
- Tax calculation
- Payment providers
- Shipping cost calculation
- Report export formats
- Authentication methods
- Validation rules
- Sorting, ranking, and recommendation algorithms
- Commission and pricing policies

## JavaScript example

```js
class VipDiscount {
  calculate(cart) {
    return cart.total() * 0.20;
  }
}

class NoDiscount {
  calculate(cart) {
    return 0;
  }
}

class CheckoutService {
  constructor(discountStrategy) {
    this.discountStrategy = discountStrategy;
  }

  payable(cart) {
    const discount = this.discountStrategy.calculate(cart);
    return cart.total() - discount;
  }
}

const checkout = new CheckoutService(new VipDiscount());
const payableAmount = checkout.payable(cart);
```

## Java example

```java
interface DiscountStrategy {
  BigDecimal calculate(Cart cart);
}

class VipDiscount implements DiscountStrategy {
  public BigDecimal calculate(Cart cart) {
    return cart.total().multiply(new BigDecimal("0.20"));
  }
}

class NoDiscount implements DiscountStrategy {
  public BigDecimal calculate(Cart cart) {
    return BigDecimal.ZERO;
  }
}

class CheckoutService {
  private final DiscountStrategy discountStrategy;

  CheckoutService(DiscountStrategy discountStrategy) {
    this.discountStrategy = discountStrategy;
  }

  BigDecimal payable(Cart cart) {
    return cart.total().subtract(discountStrategy.calculate(cart));
  }
}
```

## Before Strategy

```js
class CheckoutService {
  checkout(cart, customer) {
    if (customer.type === "VIP") {
      return cart.total() * 0.8;
    }

    if (customer.type === "NEW") {
      return cart.total() * 0.9;
    }

    return cart.total();
  }
}
```

Problem:

- Checkout changes whenever a new discount rule appears.
- The class violates Open Closed Principle.
- Testing every branch becomes harder as rules grow.

## After Strategy

```js
class CheckoutService {
  constructor(discountStrategy) {
    this.discountStrategy = discountStrategy;
  }

  checkout(cart, customer) {
    return cart.total() - this.discountStrategy.calculate(cart, customer);
  }
}
```

Benefit:

- Checkout workflow is stable.
- Discount rules change independently.
- New rules can be added as new strategy classes.

## Pros

- Removes large conditional blocks.
- Makes business rules explicit.
- Improves testability.
- Supports Open Closed Principle.
- Allows runtime behavior selection.
- Helps teams own separate policies or algorithms.

## Cons

- More classes or objects.
- More dependency wiring.
- Can be over-engineered for simple behavior.
- Strategy selection logic still needs to live somewhere.
- Poorly named strategies can make navigation harder.

## Business logic efficiency

Strategy is efficient for business logic because it separates stable workflow from volatile rules.

Example: Checkout workflow is usually stable:

1. Calculate total.
2. Apply discount.
3. Charge payment.
4. Save order.
5. Send receipt.

Discount rules may change weekly. If discount logic is inside checkout, checkout becomes risky. With Strategy, only the discount strategy changes.

Business benefits:

- Faster campaign changes.
- Safer payment provider migration.
- Smaller regression test scope.
- Better ownership by feature teams.
- Easier A/B testing and feature flagging.

## Interview question and answer

### Q1. What is the Strategy pattern?

Answer: Strategy is a behavioral design pattern that encapsulates interchangeable algorithms behind a common interface so the client can switch behavior without changing the main workflow.

### Q2. What problem does Strategy solve?

Answer: It avoids large conditional blocks for selecting behavior. Instead of editing the context for every new algorithm, you add a new strategy implementation.

### Q3. Strategy vs State pattern?

Answer: Strategy changes an algorithm selected by the client or configuration. State changes behavior based on the internal state of an object. They look similar structurally, but their intent is different.

### Q4. Strategy vs Template Method?

Answer: Strategy uses composition and can be changed at runtime. Template Method uses inheritance and defines algorithm steps in a base class.

### Q5. Which SOLID principle does Strategy support?

Answer: It strongly supports Open Closed Principle because new behavior can be added without modifying the context. It can also support Dependency Inversion when the context depends on a strategy abstraction.

## Scenario-based question and answer

### Scenario 1

Question: A checkout service contains VIP, new customer, coupon, and festival discount logic in one if/else block. How would you refactor it?

Answer: Create a `DiscountStrategy` contract and separate implementations such as `VipDiscount`, `NewCustomerDiscount`, `CouponDiscount`, and `FestivalDiscount`. Inject the selected strategy into checkout.

### Scenario 2

Question: A report generator uses a switch statement for PDF, CSV, and XLSX export. The business asks for XML export. What should you do?

Answer: Use Strategy. Create an `ExportStrategy` interface and implementations for each format. Add `XmlExportStrategy` without modifying the report generator.

### Scenario 3

Question: A payment service supports card payments today, but the roadmap includes wallet and bank transfer. Is Strategy a good fit?

Answer: Yes. Define a `PaymentStrategy` interface with `pay(amount)`. Add `CardPayment`, `WalletPayment`, and `BankTransferPayment` implementations.

### Scenario 4

Question: A small script always sorts names alphabetically. Should you use Strategy?

Answer: No. There is only one simple and stable algorithm. Strategy would add unnecessary complexity.

## Problem-solving question and answer

### Problem 1

Question: Add a seasonal discount without changing `CheckoutService`.

Answer:

1. Extract a `DiscountStrategy` interface.
2. Move existing discount logic into strategy classes.
3. Add `SeasonalDiscount`.
4. Select the strategy from a factory, configuration, or controller.
5. Inject it into `CheckoutService`.

### Problem 2

Question: How do you unit test checkout without testing real discount rules?

Answer: Inject a fake strategy.

```js
const fakeDiscount = {
  calculate: () => 100
};

const checkout = new CheckoutService(fakeDiscount);
```

Now the checkout test can focus on orchestration and payable amount.

### Problem 3

Question: How do you safely choose a strategy from user input?

Answer: Use a whitelist registry or factory.

```js
const strategies = {
  vip: new VipDiscount(),
  none: new NoDiscount()
};

function chooseStrategy(type) {
  return strategies[type] ?? strategies.none;
}
```

Do not instantiate arbitrary classes from raw user input.

### Problem 4

Question: Two strategies share most of their logic. What should you do?

Answer: Extract shared helper functions or a reusable collaborator. Do not duplicate logic across strategies. Use inheritance only if the relationship is truly stable and clear.

## Fast memory hooks

- Strategy means interchangeable algorithms.
- Context owns workflow.
- Strategy owns variation.
- Use composition, not big conditionals.
- Add behavior by adding a new strategy.
- Avoid it for one simple stable algorithm.

