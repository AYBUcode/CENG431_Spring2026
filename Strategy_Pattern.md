# Strategy Pattern — Detailed Lecture Notes (English)

---

## 0) What this document is

These notes are designed as a **classroom-ready** (lecture + lab) markdown handout for:

- **Design patterns (intro)**
- The **Strategy** design pattern
- Three OO design principles introduced through the **SimUDuck** case study:
  1. **Encapsulate what varies**
  2. **Program to a supertype (interface), not an implementation**
  3. **Favor composition over inheritance**

The running example and diagrams come from the SimUDuck story and the “weapons” design puzzle in the provided PDF (see especially pages **2–25** and **34**). fileciteturn0file0

---

## 1) Learning goals (what students should be able to do)

By the end of the lesson, students should be able to:

1. Explain why **inheritance used for behavior reuse** often causes **maintenance and change** problems.
2. Identify “hot spots” in a design — the parts that **vary frequently** — and **isolate** them.
3. Implement the **Strategy pattern** using:
   - a **Strategy interface** (supertype),
   - multiple **concrete strategies**, and
   - a **context** that delegates to a strategy object.
4. Change behavior **at runtime** (dynamic strategy swapping) without changing the context class.
5. Communicate design intent using a **shared pattern vocabulary** (and avoid “pattern fever”).

---

## 2) Design Patterns in one sentence (and why they matter)

A design pattern is a **named, reusable solution template** for a recurring design problem.

The PDF frames patterns as **experience reuse** rather than code reuse (p.1) and emphasizes that patterns help you handle the one constant in software development: **change** (p.8). fileciteturn0file0

### 2.1 Patterns are not libraries

A library gives you **implementation** you call.  
A pattern gives you a **structure** you implement and adapt to your context (p.29). fileciteturn0file0

### 2.2 Patterns are also a vocabulary

The “diner” and “cubicle” stories show patterns as a **compression language**: you can say “Strategy” or “Observer” and communicate a set of structural expectations, constraints, and tradeoffs (pp.26–28). fileciteturn0file0

---

## 3) Case study setup: SimUDuck

### 3.1 Initial design (works… until requirements change)

In the initial SimUDuck design, there is:

- A superclass `Duck` that implements common behavior like `quack()` and `swim()`.
- Subclasses such as `MallardDuck` and `RedheadDuck` that implement `display()` differently (p.2). fileciteturn0file0

Conceptually:

```text
Duck
 ├─ quack()
 ├─ swim()
 └─ display()   // abstract

MallardDuck extends Duck
 └─ display()

RedheadDuck extends Duck
 └─ display()

... many more ducks ...
```

This is classic OO reuse: shared behavior in the superclass, variation in subclasses.

---

## 4) The “change” event: “Make the ducks fly”

The executives demand flying ducks (p.3). fileciteturn0file0  
Joe (the developer) decides to add `fly()` to the `Duck` superclass so all ducks “inherit it”.

### 4.1 What goes wrong: “non-local side effects”

Not all ducks should fly:

- Rubber ducks (toy/inanimate) do not fly.
- Decoy ducks (wooden) do not fly and may not quack.

When `fly()` is placed in the superclass, *every* duck inherits it.  
The PDF illustrates the demo disaster: rubber ducks flying across the screen (p.4). fileciteturn0file0

**Key lesson:** changing a superclass can produce **unexpected effects** in far-away subclasses.

> In design terms: a local modification caused a **system-wide behavioral change** because behavior reuse was coupled to inheritance.

---

## 5) “Fix” attempt #1: override behavior in special subclasses

Joe considers overriding `fly()` to “do nothing” inside non-flying ducks (p.5). fileciteturn0file0

### 5.1 Why this does not scale

If you add enough duck types, you now have to:

- inspect which ducks need overrides,
- duplicate “do nothing” logic,
- remember to override in new subclasses,
- risk forgetting overrides and introducing bugs.

This is a symptom of **inheritance being used as a behavior configuration mechanism**.

---

## 6) “Fix” attempt #2: use interfaces `Flyable` and `Quackable`

Joe proposes removing `fly()` from `Duck` and instead using `Flyable` / `Quackable` so only relevant ducks implement those (p.6). fileciteturn0file0

### 6.1 What improves

- Non-flying ducks no longer “accidentally fly”.
- Interfaces express capability (“can fly”) explicitly.

### 6.2 What breaks

The PDF points out the new problem: **duplicate code** (p.7). fileciteturn0file0

Because Java interfaces (in the traditional sense) don’t provide shared implementation (ignoring modern default methods), every `Flyable` duck must implement flying itself.

If you have *48 flying duck subclasses*, and you tweak flying behavior, you must modify *48 classes*.

So:  
- inheritance caused *inappropriate sharing*,  
- pure interfaces cause *no sharing* and lead to duplication.

We need a way to:
1. reuse behaviors **without inheriting them**, and
2. let behaviors vary **independently**.

---

## 7) The three design principles introduced by the case study

These principles are explicitly called out in the PDF and are the conceptual bridge to Strategy.

### 7.1 Principle 1 — Encapsulate what varies

> Identify aspects that vary and separate them from what stays the same (p.9). fileciteturn0file0

In SimUDuck, the parts that vary are:

- flying behavior
- quacking behavior

The parts that are stable include:

- `swim()` (all ducks float)
- the “duck-ness” of the base class
- each duck’s `display()` (varies per duck type but is naturally a per-duck responsibility)

So we **extract** the variable behaviors into separate modules.

#### 7.1.1 Consequences

- Fewer unintended consequences from changes.
- Easier extension: add a new behavior without editing old duck classes.

The PDF diagram on p.10 shows “pulling out” `fly` and `quack` into separate behavior class structures. fileciteturn0file0

---

### 7.2 Principle 2 — Program to a supertype, not an implementation

The PDF clarifies “interface” here means **supertype**, not necessarily the Java `interface` keyword (p.12). fileciteturn0file0

**Idea:** the context should depend on *capabilities* and *contracts*, not concrete classes.

Example:

- Bad: `Dog d = new Dog();` (locked to `Dog`)
- Better: `Animal a = new Dog();` (a supertype reference)
- Best: assign the concrete type at runtime via a factory or config.

This enables polymorphism and reduces coupling.

---

### 7.3 Principle 3 — Favor composition over inheritance

The PDF highlights a HAS‑A relationship: a duck **has a** fly behavior and **has a** quack behavior; it **delegates** to them (p.23). fileciteturn0file0

**Inheritance** is an IS‑A relationship (“a MallardDuck is-a Duck”).  
**Composition** is a HAS‑A relationship (“a Duck has-a FlyBehavior”).

Composition allows you to:

- swap parts without changing type hierarchy,
- isolate varying behavior,
- reduce fragile base class issues.

---

## 8) Strategy Pattern: intent and structure

### 8.1 Intent (paraphrased)

The Strategy pattern:

- groups interchangeable algorithms/behaviors behind a common interface,
- encapsulates each algorithm in its own class (or function object),
- lets clients (contexts) choose and swap algorithms without changing client code.

This is introduced explicitly as the “first pattern” applied to SimUDuck (p.24). fileciteturn0file0

### 8.2 Core roles (terminology)

| Role | Responsibility | SimUDuck mapping |
|---|---|---|
| **Context** | Uses a strategy to perform work; delegates the variable part | `Duck` |
| **Strategy** (interface/supertype) | Common contract for a family of algorithms | `FlyBehavior`, `QuackBehavior` |
| **ConcreteStrategy** | Implements a specific algorithm | `FlyWithWings`, `FlyNoWay`, `Quack`, `Squeak`, `MuteQuack`, `FlyRocketPowered` |
| **Client** | Configures the context with a strategy | duck constructors or simulator code |

### 8.3 Minimal UML sketch

```text
           +------------------+
           |     Duck         |   (Context)
           |------------------|
           | flyBehavior      |-----> FlyBehavior (Strategy)
           | quackBehavior    |-----> QuackBehavior (Strategy)
           |------------------|
           | performFly()     |
           | performQuack()   |
           | setFlyBehavior() |
           | setQuackBehavior()|
           +------------------+

FlyBehavior (interface)          QuackBehavior (interface)
   ^        ^                       ^      ^      ^
   |        |                       |      |      |
FlyWithWings FlyNoWay            Quack   Squeak  MuteQuack
   |
FlyRocketPowered
```

This matches the “big picture” diagram of encapsulated behaviors in the PDF (p.22). fileciteturn0file0

---

## 9) Step-by-step: refactoring SimUDuck into Strategy

This section is designed as a guided walkthrough.

### Step 1 — Identify the varying behaviors

- `fly()` varies by duck type and future requirements.
- `quack()` varies by duck type and future requirements.

Everything else stays mostly stable.

(See analysis on pp.9–10.) fileciteturn0file0

---

### Step 2 — Define strategy interfaces

```java
public interface FlyBehavior {
    void fly();
}

public interface QuackBehavior {
    void quack();
}
```

Key point: `Duck` will hold references **typed by these interfaces**, not by concrete classes.

---

### Step 3 — Create concrete strategies (behavior implementations)

Examples (names can vary; these are conventional):

```java
public class FlyWithWings implements FlyBehavior {
    @Override public void fly() {
        System.out.println("I'm flying with wings!");
    }
}

public class FlyNoWay implements FlyBehavior {
    @Override public void fly() {
        System.out.println("I can't fly.");
    }
}

public class FlyRocketPowered implements FlyBehavior {
    @Override public void fly() {
        System.out.println("I'm flying with a rocket!");
    }
}
```

Quack strategies:

```java
public class Quack implements QuackBehavior {
    @Override public void quack() {
        System.out.println("Quack!");
    }
}

public class Squeak implements QuackBehavior {
    @Override public void quack() {
        System.out.println("Squeak!");
    }
}

public class MuteQuack implements QuackBehavior {
    @Override public void quack() {
        System.out.println("<< silence >>");
    }
}
```

The PDF shows this split into “families” of behaviors (p.13) and later explicitly calls them a “family of algorithms” (p.22). fileciteturn0file0

---

### Step 4 — Redesign `Duck` to use composition + delegation

Now `Duck` is a context that **delegates**:

```java
public abstract class Duck {

    protected FlyBehavior flyBehavior;
    protected QuackBehavior quackBehavior;

    public abstract void display();

    public void performFly() {
        flyBehavior.fly();     // delegation
    }

    public void performQuack() {
        quackBehavior.quack(); // delegation
    }

    public void swim() {
        System.out.println("All ducks float (even decoys).");
    }
}
```

This corresponds to the “integrating behavior” explanation and the `performQuack()` delegation on p.15. fileciteturn0file0

---

### Step 5 — Configure strategies (initially via constructors)

Example: Mallard ducks fly with wings and quack normally.

```java
public class MallardDuck extends Duck {

    public MallardDuck() {
        this.flyBehavior = new FlyWithWings();
        this.quackBehavior = new Quack();
    }

    @Override
    public void display() {
        System.out.println("I'm a real Mallard duck.");
    }
}
```

The PDF walks through this idea using `MallardDuck` construction (p.16). fileciteturn0file0

> Teaching note: this still instantiates concrete strategies in the constructor.  
> The PDF explicitly calls out that this is not ideal and hints that later patterns (e.g., factories/DI) can improve strategy creation (p.17). fileciteturn0file0

---

### Step 6 — Add runtime swapping (dynamic behavior)

To enable behavior changes while the object is alive, add setters to the context:

```java
public void setFlyBehavior(FlyBehavior fb) {
    this.flyBehavior = fb;
}

public void setQuackBehavior(QuackBehavior qb) {
    this.quackBehavior = qb;
}
```

This is demonstrated via `ModelDuck` and “rocket-powered flying” in the PDF (pp.20–21). fileciteturn0file0

#### Example context subtype: a model duck starts grounded

```java
public class ModelDuck extends Duck {

    public ModelDuck() {
        this.flyBehavior = new FlyNoWay();
        this.quackBehavior = new Quack();
    }

    @Override
    public void display() {
        System.out.println("I'm a model duck.");
    }
}
```

#### Demo / test harness

```java
public class MiniDuckSimulator {

    public static void main(String[] args) {
        Duck mallard = new MallardDuck();
        mallard.performQuack();
        mallard.performFly();

        Duck model = new ModelDuck();
        model.performFly();
        model.setFlyBehavior(new FlyRocketPowered());
        model.performFly();
    }
}
```

Expected behavior: the model duck initially cannot fly, then it gains rocket flight after swapping strategies.

---

## 10) Why Strategy works here (and what you gained)

### 10.1 Separation of concerns

- Duck types focus on **identity / presentation** (`display()`).
- Behaviors focus on **algorithms** (ways of flying/quacking).
- The context (`Duck`) coordinates and delegates.

### 10.2 Reuse without inheritance baggage

A single `FlyWithWings` can be reused by any “winged” duck without putting code in the superclass.

The PDF emphasizes you get reuse “without the baggage” of inheritance (p.13). fileciteturn0file0

### 10.3 Open-Closed Principle (informal)

You can add new behaviors (e.g., rocket flying) by **adding a new class**, with minimal or no edits to old code.

### 10.4 Runtime flexibility

The setter-based approach creates a plug-in behavior model:

- configure once (constructor),
- or adapt at runtime (setters).

The PDF explicitly highlights runtime switching (pp.11 and 20–21). fileciteturn0file0

---

## 11) Tradeoffs and pitfalls (important for teaching)

Strategy is not “free”. It introduces costs and design choices.

### 11.1 More classes (and more moving parts)

You trade:
- fewer conditionals and less duplication  
for
- more types (`FlyNoWay`, `FlyWithWings`, …)

This is usually a good trade when:
- behaviors change frequently,
- you need runtime switching,
- you want isolated testing of behaviors.

### 11.2 Strategy explosion

If you create a strategy for every micro-variation, you may end up with too many small classes.

Mitigations:
- Use parameterized strategies (e.g., `FlyWithSpeed(double mps)`).
- Use functional strategies (lambdas) in languages that support it.
- Merge truly equivalent strategies.

### 11.3 Who owns strategy creation?

Constructors like `new FlyWithWings()` in `MallardDuck` couple a duck subtype to a concrete strategy class (the PDF calls this out on p.17). fileciteturn0file0

Better alternatives (advanced, but worth mentioning):
- dependency injection (DI),
- factories,
- configuration-driven strategy selection,
- registries.

### 11.4 Mutability and safety

If strategies are mutable and shared across multiple contexts, you can create surprising coupling.

Rule of thumb:
- Make strategies **stateless** and reusable when possible, or
- if strategies hold state, treat them as **per-context instances**.

### 11.5 Strategy vs “just use an `if`”

Sometimes an `if` is simpler.

Use Strategy when:
- you have multiple algorithms and expect more,
- algorithms vary independently of the context,
- you want to unit-test algorithms in isolation,
- you want runtime swapping.

Avoid Strategy when:
- there are only 2 cases and they are stable,
- performance and allocation costs matter and variation is trivial,
- the behavior is not truly independent.

---

## 12) Strategy Pattern in another domain (short examples)

These are useful in class to generalize beyond ducks.

### 12.1 Pricing / discount policies

- `PricingStrategy` interface: `Money price(Order order)`
- strategies: `NoDiscount`, `StudentDiscount`, `BlackFridayDiscount`, `SubscriptionDiscount`

### 12.2 Sorting

- `SortStrategy<T>`: `void sort(List<T> xs)`
- strategies: `QuickSort`, `MergeSort`, `HeapSort`
- context: `Sorter` (or just pass strategy into method)

### 12.3 Authentication providers

- `AuthStrategy`: `User authenticate(Credentials c)`
- strategies: `PasswordAuth`, `OAuthAuth`, `SAMLAuth`

---

## 13) In-class exercise: “Weapons” design puzzle (Strategy again)

The PDF includes a second scenario: an action adventure game with characters that can change weapons dynamically (p.25). fileciteturn0file0

### 13.1 Problem statement (paraphrased)

- There are character types: `King`, `Queen`, `Knight`, `Troll`.
- Each character uses **one weapon at a time**.
- A character can **switch weapons during the game**.
- Weapon behaviors include: sword, knife, axe, bow-and-arrow.

### 13.2 What students should draw / implement

1. An abstract `Character` class (context)
2. A `WeaponBehavior` interface (strategy)
3. Concrete weapon behaviors (concrete strategies)
4. A HAS‑A relationship: each character *has a* weapon behavior
5. A `setWeapon(WeaponBehavior w)` method in the right place (the context)

### 13.3 Expected structure

```text
Character (abstract)  (Context)
  - weapon : WeaponBehavior
  + fight()
  + setWeapon(w)

WeaponBehavior (interface) (Strategy)
  + useWeapon()

Concrete strategies:
  SwordBehavior, KnifeBehavior, AxeBehavior, BowAndArrowBehavior

Concrete contexts:
  King, Queen, Knight, Troll  (extend Character)
```

The provided solution in the PDF confirms this structure and that weapon can be swapped via `setWeapon` defined on the abstract `Character` superclass (p.34). fileciteturn0file0

### 13.4 Skeleton code for the lab

```java
public interface WeaponBehavior {
    void useWeapon();
}

public abstract class Character {
    protected WeaponBehavior weapon;

    public void setWeapon(WeaponBehavior w) {
        this.weapon = w;
    }

    public void fight() {
        if (weapon == null) throw new IllegalStateException("No weapon equipped");
        weapon.useWeapon();
    }
}
```

Concrete strategies:

```java
public class SwordBehavior implements WeaponBehavior {
    @Override public void useWeapon() { System.out.println("Swing sword"); }
}

public class BowAndArrowBehavior implements WeaponBehavior {
    @Override public void useWeapon() { System.out.println("Shoot arrow"); }
}
```

Concrete contexts:

```java
public class King extends Character {
    public King() { this.weapon = new SwordBehavior(); }
}
```

> Teaching point: the PDF notes that **any object** could implement `WeaponBehavior`, not only “weapons” in a literal sense (p.34). fileciteturn0file0  
> This is a good moment to emphasize: strategies are “algorithms”, not “things”.

---

## 14) Discussion prompts (ready-to-use)

### 14.1 Why did inheritance fail for flying?

Use the page 4 demo problem (rubber ducks flying) to discuss:

- Why “reuse by inheritance” can be fragile.
- How superclass changes propagate globally.
- Why “override everywhere” becomes hard to maintain.

### 14.2 Identify “what varies” in a system you know

Prompt (based on p.8): list real drivers of change, for example:

- new user requirements,
- vendor changes (DB/provider),
- platform / protocol updates,
- refactoring from lessons learned.

Have students list **their own** examples and map them to “varying modules”.

### 14.3 Pattern vocabulary: when is it useful?

Based on the diner/cubicle stories (pp.26–28):

- What information is implicitly communicated when someone says “Strategy”?
- When does “pattern talk” become unhelpful or performative (“pattern fever”)?

---

## 15) Quick quiz (with answer key)

### Q1
In SimUDuck, which two behaviors were extracted into strategy families?

- A) display and swim  
- B) fly and quack  
- C) fly and display  
- D) quack and swim

**Answer:** B. (pp.9–10) fileciteturn0file0

### Q2
What relationship best describes the Strategy pattern between context and strategy?

- A) Context IS-A Strategy  
- B) Context HAS-A Strategy  
- C) Strategy HAS-A Context  
- D) Context IMPLEMENTS Strategy

**Answer:** B. (pp.22–23) fileciteturn0file0

### Q3
Why can “interfaces only” lead to duplication in this case?

**Answer:** because each implementing class must provide its own code for the behavior; updating a behavior requires edits across many classes (p.7). fileciteturn0file0

### Q4
True/False: The phrase “program to an interface” means you must use the `interface` keyword in Java.

**Answer:** False. It means “program to a supertype” (p.12). fileciteturn0file0

---

## 16) Cheat sheet (for students)

### When to reach for Strategy

Use Strategy if:

- you have multiple ways to do the same task (a “family” of algorithms),
- you expect new variants over time,
- you want to avoid big conditional blocks,
- you want runtime switching,
- you want to unit test each algorithm independently.

### Implementation checklist

1. Name the **Context** and identify what varies.
2. Create a **Strategy interface** for the varying part.
3. Implement each algorithm as a **ConcreteStrategy**.
4. Give the context a **field of the strategy supertype**.
5. Delegate work via the strategy interface.
6. Decide configuration:
   - constructor injection,
   - setter injection (for runtime swapping),
   - factory/DI for creation.

---

## 17) Appendix: “Sharpen your pencil” — inheritance disadvantages

The PDF includes a multiple-choice reflection (p.5) asking which disadvantages can occur when subclassing is used to provide behavior. fileciteturn0file0

A good summary answer (instructor phrasing) is:

- **Duplication** appears when you override the same behavior in many subclasses.
- **Runtime swapping** is difficult because behavior is tied to class type.
- Adding new features (e.g., “make ducks dance”) can require edits across many subclasses.
- Changes in shared code can have **unintended side effects** in subclasses.

(An explicit checkbox solution is shown in the PDF’s “Solutions” page (p.35). fileciteturn0file0)

---

## 18) Appendix: Suggested 75–90 minute lecture flow

1. **Motivation (10 min)**: change is constant; patterns are experience reuse; patterns as vocabulary (pp.1, 8, 26–28). fileciteturn0file0
2. **SimUDuck failure modes (15 min)**: inheritance pitfalls (pp.3–5). fileciteturn0file0
3. **Principles (15 min)**: encapsulate what varies; program to a supertype; favor composition (pp.9–13, 23). fileciteturn0file0
4. **Strategy implementation (25 min)**: code walkthrough + UML (pp.15–22). fileciteturn0file0
5. **Weapons puzzle (15 min)**: group activity (p.25), then show solution discussion (p.34). fileciteturn0file0
6. **Wrap-up (5 min)**: tradeoffs + cheat sheet.

---

## 19) Source

W2_Strategy.pdf fileciteturn0file0
