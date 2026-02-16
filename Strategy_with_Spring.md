# Strategy Pattern with Spring (Spring Boot)
## Runtime selection via DI, factory-like registries, and configuration

**Source:** *W2_Strategy.pdf* (Head First Design Patterns, Strategy chapter excerpts).
Throughout these notes, page references like *(p.12)* refer to that PDF.

---

## 1) Start with a real-world example: Payments in an e-commerce checkout

### The situation
You are building a checkout flow. Users can pay with:

- Credit card
- Bank transfer
- Digital wallet (PayPal / Apple Pay-like)

All of these are the **same “business capability”** (“take a payment”), but the **algorithm** differs:
- card → authorization, maybe 3DS
- bank transfer → reference number, delayed confirmation
- wallet → redirect + callback

### The naive implementation (what usually happens)
A single method grows into a big conditional:

```java
void pay(String method, long amount) {
  if (method.equals("CARD")) { ... }
  else if (method.equals("BANK")) { ... }
  else if (method.equals("WALLET")) { ... }
  else throw new IllegalArgumentException("Unknown method");
}
```

This is exactly the kind of change pain the book warns about: software changes constantly *(p.8)*, so designs must be prepared for new requirements.

### Strategy pattern mapping (book vocabulary)
The Strategy pattern:
> “defines a family of algorithms, encapsulates each one, and makes them interchangeable” *(p.24)*.

For payments:

- **Context**: `CheckoutService` (or `PaymentProcessor`)
- **Strategy interface**: `PaymentStrategy`
- **Concrete strategies**: `CreditCardPayment`, `BankTransferPayment`, `WalletPayment`

And you apply these core principles:

- **Encapsulate what varies**: payment algorithms vary; isolate them *(pp.9–10)*.
- **Program to a supertype, not an implementation**: depend on `PaymentStrategy`, not `CreditCardPayment` *(p.12)*.
- **Favor composition over inheritance**: `CheckoutService` **HAS-A** `PaymentStrategy` and delegates *(p.23)*.

---

## 2) Quick recap: what “runtime assignment” means in the book

On *(p.12)*, the book explains that even better than writing:

```java
Animal a = new Dog();
```

is to assign the concrete implementation at runtime:

```java
a = getAnimal();
a.makeSound();
```

Interpretation:
- the caller depends on the **supertype** (`Animal`)
- the concrete type (`Dog` or `Cat`) is chosen **at runtime**
- the decision can come from a factory method, configuration, user input, etc.

**Spring’s DI container is a practical mechanism to achieve that same goal.**

---

## 3) Implementing Strategy with Spring: three practical patterns

Spring helps you avoid hard-coded `new` in your application logic.
This matters because the book explicitly points out that constructing concrete behaviors inside constructors is “not ideal” and hints that later tools/patterns fix it *(p.17)*.

We’ll build from simplest to most flexible.

---

# Pattern A — One strategy wired by Spring (fixed at startup)

Use this when your application has exactly **one** `PaymentStrategy` (or one “default”) and you don’t need switching.

### Step A1: define the strategy interface

```java
public interface PaymentStrategy {
  void pay(long amount);
}
```

### Step A2: implement one strategy as a Spring bean

```java
import org.springframework.stereotype.Service;

@Service
public class CreditCardPayment implements PaymentStrategy {
  @Override
  public void pay(long amount) {
    // card authorization logic
  }
}
```

### Step A3: inject into the context (composition + delegation)

```java
import org.springframework.stereotype.Service;

@Service
public class CheckoutService {
  private final PaymentStrategy paymentStrategy;

  public CheckoutService(PaymentStrategy paymentStrategy) {
    this.paymentStrategy = paymentStrategy;
  }

  public void checkout(long total) {
    paymentStrategy.pay(total); // delegate (like performFly/performQuack) (p.15)
  }
}
```

**What this teaches:**
- Context delegates to the strategy, just like `Duck.performFly()` delegates to `flyBehavior.fly()` *(p.15)*.
- Context depends only on the supertype (interface) *(p.12)*.

**Limitation:** If you later add a second `PaymentStrategy`, Spring won’t know which one to inject (ambiguity).

---

# Pattern B — Multiple strategies, chosen at wiring time (`@Qualifier`, `@Primary`, profiles)

Use this when you have multiple strategies, but you still want **one** selected globally (per environment or per deployment), not per request.

## Option B1: `@Qualifier`

```java
@Service("creditCard")
public class CreditCardPayment implements PaymentStrategy { ... }

@Service("bankTransfer")
public class BankTransferPayment implements PaymentStrategy { ... }
```

```java
@Service
public class CheckoutService {
  private final PaymentStrategy paymentStrategy;

  public CheckoutService(@Qualifier("creditCard") PaymentStrategy paymentStrategy) {
    this.paymentStrategy = paymentStrategy;
  }
}
```

This is compile-time selection (the string is in code).

## Option B2: `@Primary`

```java
@Primary
@Service
public class CreditCardPayment implements PaymentStrategy { ... }
```

Spring uses the primary bean if multiple candidates exist.

## Option B3: `@Profile` (environment-based)

```java
@Profile("prod")
@Service
public class CreditCardPayment implements PaymentStrategy { ... }

@Profile("dev")
@Service
public class FakePayment implements PaymentStrategy { ... }
```

Then select with `spring.profiles.active=prod` etc.

**Concept link to the book:** these options are “configuration-driven” ways to avoid locking the caller into a concrete type *(p.12)*, while still keeping the system modular.

---

# Pattern C — Runtime selection (the Strategy “registry” pattern)

This is the closest match to the book’s “assign the concrete implementation object at runtime” *(p.12)* **and** to the “change behavior dynamically” goal *(pp.11, 20–21)*.

### The idea
- You have many `PaymentStrategy` beans.
- Spring injects them as a `Map<String, PaymentStrategy>`.
- At runtime, you pick one by key (e.g., from user choice).

### Step C1: define strategies with stable keys

```java
@Service("creditCard")
public class CreditCardPayment implements PaymentStrategy {
  @Override public void pay(long amount) { /* ... */ }
}

@Service("bankTransfer")
public class BankTransferPayment implements PaymentStrategy {
  @Override public void pay(long amount) { /* ... */ }
}

@Service("wallet")
public class WalletPayment implements PaymentStrategy {
  @Override public void pay(long amount) { /* ... */ }
}
```

### Step C2: inject a registry map into the context

```java
@Service
public class CheckoutService {

  private final Map<String, PaymentStrategy> strategies;

  public CheckoutService(Map<String, PaymentStrategy> strategies) {
    this.strategies = strategies;
  }

  public void checkout(String method, long total) {
    PaymentStrategy strategy = strategies.get(method);
    if (strategy == null) {
      throw new IllegalArgumentException("Unsupported payment method: " + method);
    }
    strategy.pay(total);
  }
}
```

### Step C3: an example controller (user-driven runtime choice)

```java
@RestController
@RequestMapping("/checkout")
public class CheckoutController {
  private final CheckoutService checkoutService;

  public CheckoutController(CheckoutService checkoutService) {
    this.checkoutService = checkoutService;
  }

  @PostMapping
  public void checkout(@RequestParam String method, @RequestParam long total) {
    checkoutService.checkout(method, total);
  }
}
```

### Why this is “Strategy” (not just DI)
This matches the Strategy definition *(p.24)*:
- multiple interchangeable algorithms (payment methods)
- each encapsulated separately
- selected at runtime based on context

And it matches the design move in SimUDuck:
- `Duck` **HAS-A** behavior and delegates *(p.23)*
- you can swap the behavior dynamically *(pp.20–21)*

---

## 4) “Factory or config” in Spring terms

The phrase you asked about was:

> “assign the concrete type at runtime via a factory or config” *(p.12)*

In Spring, the **DI container** often replaces the need for hand-written factories.
But you still frequently use configuration to influence which strategy is chosen.

### Example: pick a default method from `application.yml`

```yaml
payment:
  defaultMethod: creditCard
```

```java
@Service
public class CheckoutService {

  private final Map<String, PaymentStrategy> strategies;
  private final String defaultMethod;

  public CheckoutService(
      Map<String, PaymentStrategy> strategies,
      @Value("${payment.defaultMethod}") String defaultMethod
  ) {
    this.strategies = strategies;
    this.defaultMethod = defaultMethod;
  }

  public void checkout(long total) {
    PaymentStrategy strategy = strategies.get(defaultMethod);
    if (strategy == null) throw new IllegalStateException("Bad config: " + defaultMethod);
    strategy.pay(total);
  }
}
```

This is configuration-driven selection (a direct operationalization of *(p.12)*).

---

## 5) Teaching notes: connect back to the SimUDuck refactor

### 5.1 Delegation is the key move
In SimUDuck, the refactor removes `fly()` and `quack()` from `Duck` and replaces them with:

- `performFly()` → delegates to `FlyBehavior.fly()`
- `performQuack()` → delegates to `QuackBehavior.quack()` *(p.15)*

In the payment system, the analogous delegation is:

- `CheckoutService.checkout()` → delegates to `PaymentStrategy.pay()`

### 5.2 Runtime switching: per-request vs setters
The book demonstrates dynamic switching via setters *(pp.20–21)*:

```java
model.setFlyBehavior(new FlyRocketPowered());
```

In a typical Spring web app, you often don’t “mutate the service” with setters.
Instead, you select the strategy **per request** (Pattern C), which is safer in concurrent systems.

**Interpretation:** runtime switching is still happening; it’s just driven by method input rather than mutating global state.

---

## 6) Common pitfalls (what students should watch for)

1. **Using strings everywhere**  
   Prefer an enum for payment methods and map it to bean names to reduce typos.

2. **Stateful strategies shared as singletons**  
   Spring beans are singletons by default. Ensure strategies are stateless or thread-safe.

3. **Overusing strategy for trivial variation**  
   If there are only two stable cases and no growth expected, a simple conditional may be fine.

4. **Selection logic creeping into the context**
   Keep selection logic thin. The main job of the context is to delegate; selection can be:
   - a small resolver component
   - configuration
   - request-driven routing

---

## 7) Summary checklist (what to memorize for exams)

- Strategy = family of algorithms, encapsulated, interchangeable *(p.24)*.
- Use Strategy when an algorithm varies independently from clients *(p.24)*.
- Depend on a supertype, not a concrete class *(p.12)*.
- Prefer HAS-A composition and delegation *(p.23)*.
- Runtime selection can be:
  - factory method (`getAnimal()` idea) *(p.12)*
  - configuration
  - DI container wiring (Spring)
  - per-request registry lookup (Map injection)

---
