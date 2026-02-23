# The Observer Pattern

## CENG431 – Software Engineering | Week 3 Lecture Notes

**Ankara Yıldırım Beyazıt University – Department of Computer Engineering**

---

## 1. Introduction

The Observer Pattern is one of the most frequently used design patterns in software engineering. It establishes a mechanism where an object (known as the **Subject**) maintains a list of its dependents (known as **Observers**) and automatically notifies them of any state changes. This pattern is foundational in event-driven programming, GUI frameworks, and distributed systems.

> **The Observer Pattern** defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically.

The key idea is simple: think of a newspaper subscription. A publisher produces newspapers, subscribers receive them automatically, and anyone can subscribe or unsubscribe at any time. In pattern terminology, the publisher is the **Subject** and the subscribers are the **Observers**.

---

## 2. Motivation: The Weather Monitoring Station

To understand the Observer Pattern, let's walk through a concrete scenario from the classic *Head First Design Patterns* book.

### 2.1 The Problem

Imagine a company called **Weather-O-Rama** that builds weather monitoring stations. The system has three main components:

- **Weather Station** – A physical device with sensors for temperature, humidity, and barometric pressure.
- **WeatherData Object** – A software object that pulls data from the weather station and tracks current conditions.
- **Display Elements** – Multiple screens that show weather information (current conditions, statistics, forecast).

The requirements are:

1. Three initial display elements must be updated whenever new weather data arrives.
2. The system must be **expandable** — other developers should be able to create custom display elements.
3. Display elements can be added or removed at runtime.

### 2.2 The WeatherData Class

The `WeatherData` class provides three getter methods and a callback method:

```java
public class WeatherData {
    // Returns the most recent temperature reading
    public float getTemperature() { ... }

    // Returns the most recent humidity reading
    public float getHumidity() { ... }

    // Returns the most recent barometric pressure reading
    public float getPressure() { ... }

    // Called whenever new measurements are available
    public void measurementsChanged() {
        // Our code goes here
    }
}
```

### 2.3 A Naïve (Wrong) Approach

A first attempt might look like this:

```java
public void measurementsChanged() {
    float temp = getTemperature();
    float humidity = getHumidity();
    float pressure = getPressure();

    currentConditionsDisplay.update(temp, humidity, pressure);
    statisticsDisplay.update(temp, humidity, pressure);
    forecastDisplay.update(temp, humidity, pressure);
}
```

**What's wrong with this implementation?**

- **Coding to concrete implementations** – We are directly calling methods on specific display objects. Adding a new display requires modifying this method.
- **No encapsulation of the varying part** – The area that changes (which displays to update) is not encapsulated.
- **No runtime flexibility** – We cannot add or remove displays at runtime without recompiling.
- **Violates the Open/Closed Principle** – The class is not closed for modification when new displays are added.

The only positive aspect is that all displays share a common `update()` method signature, hinting at a common interface.

---

## 3. Understanding the Observer Pattern

### 3.1 The Newspaper Analogy

The Observer Pattern works exactly like a newspaper subscription service:

1. A **publisher** (Subject) goes into business and starts publishing.
2. You **subscribe** to the publisher, and every new edition is delivered to you.
3. You **unsubscribe** when you no longer want papers, and delivery stops.
4. While the publisher is in business, people constantly **subscribe and unsubscribe**.

In pattern terminology:

| Newspaper Analogy | Observer Pattern |
|---|---|
| Publisher | Subject (Observable) |
| Subscriber | Observer |
| Subscribe | registerObserver() |
| Unsubscribe | removeObserver() |
| Deliver newspaper | notifyObservers() → update() |

### 3.2 A Day in the Life of the Observer Pattern

Let's walk through the lifecycle:

**Step 1: Initial State**
A Subject object holds some data (say, an integer value of `2`). Three observers (Dog, Cat, Mouse) are registered and receive updates.

**Step 2: A New Observer Joins**
A Duck object wants to become an observer. It tells the Subject "register me." The Duck is added to the observer list and will receive future notifications.

**Step 3: State Changes**
The Subject's data changes from `2` to `8`. All four observers (Dog, Cat, Mouse, Duck) are notified and receive the new value.

**Step 4: An Observer Leaves**
The Mouse object decides to unsubscribe. It tells the Subject "remove me." The Subject acknowledges and removes Mouse from the list.

**Step 5: Another State Change**
The Subject's data changes to `14`. Only Dog, Cat, and Duck receive the notification. Mouse is no longer included.

### 3.3 Formal Definition

> **The Observer Pattern** defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically.

Key characteristics:

- **One-to-many relationship** – One Subject, many Observers.
- **Automatic notification** – Observers don't need to poll; they are pushed updates.
- **Loose coupling** – The Subject knows only that observers implement a certain interface.
- **Dynamic subscription** – Observers can register and deregister at runtime.

---

## 4. The Class Diagram

### 4.1 General Structure

```
┌─────────────────────┐         observers         ┌──────────────────┐
│    <<interface>>     │◆───────────────────────▷│   <<interface>>  │
│      Subject         │                          │     Observer     │
├──────────────────────┤                          ├──────────────────┤
│ registerObserver()   │                          │ update()         │
│ removeObserver()     │                          └──────────────────┘
│ notifyObservers()    │                                   △
└─────────────────────┘                                    │
          △                                                │
          │                            ┌───────────────────┤
          │                            │                   │
┌─────────────────────┐   subject  ┌──────────────────┐   │
│  ConcreteSubject     │◁──────────│ ConcreteObserver │   │
├──────────────────────┤           ├──────────────────┤   │
│ registerObserver()   │           │ update()         │   │
│ removeObserver()     │           │ // other methods │   │
│ notifyObservers()    │           └──────────────────┘   │
│ getState()           │                                  │
│ setState()           │           ┌──────────────────┐   │
└─────────────────────┘            │ AnotherObserver  │───┘
                                   ├──────────────────┤
                                   │ update()         │
                                   └──────────────────┘
```

### 4.2 Component Descriptions

**Subject Interface** – Provides methods for observers to register, deregister, and be notified. Any object that wants to be observable implements this interface.

**Observer Interface** – All potential observers implement this interface, which has a single `update()` method called when the Subject's state changes.

**ConcreteSubject** – Implements the Subject interface. Maintains a list of observers and a state. When the state changes, it iterates through the observer list and calls `update()` on each.

**ConcreteObserver** – Implements the Observer interface. Registers with a ConcreteSubject to receive notifications. Can also hold a reference to the Subject to pull additional state if needed.

---

## 5. The Power of Loose Coupling

### 5.1 Design Principle

> **Strive for loosely coupled designs between objects that interact.**

When two objects are loosely coupled, they can interact but have very little knowledge of each other. The Observer Pattern is a perfect example of this principle.

### 5.2 Why is the Observer Pattern Loosely Coupled?

**The Subject only knows that observers implement the Observer interface.** It doesn't know the concrete class of any observer, what it does, or any other details.

**New observers can be added at any time.** Since the Subject depends only on a list of objects that implement the Observer interface, we can add, replace, or remove observers at runtime without modifying the Subject.

**The Subject never needs modification to accommodate new observer types.** A new concrete class just needs to implement the Observer interface and register itself. The Subject doesn't care about the concrete type.

**Subjects and observers can be reused independently.** Because they aren't tightly coupled, either can be reused in different contexts.

**Changes to either side don't affect the other.** As long as both sides honor their interfaces, internal changes are safe.

### 5.3 Benefits Summary

| Benefit | Explanation |
|---|---|
| Extensibility | New observers can be added without modifying the Subject |
| Runtime flexibility | Observers can subscribe/unsubscribe dynamically |
| Reusability | Subjects and observers are independently reusable |
| Separation of concerns | The Subject focuses on state; observers focus on reactions |
| Minimal dependencies | Interaction happens through well-defined interfaces |

---

## 6. Implementation: The Weather Station

### 6.1 Defining the Interfaces

```java
public interface Subject {
    public void registerObserver(Observer o);
    public void removeObserver(Observer o);
    public void notifyObservers();
}

public interface Observer {
    public void update(float temp, float humidity, float pressure);
}

public interface DisplayElement {
    public void display();
}
```

The `Subject` interface defines the three essential methods. The `Observer` interface has a single `update()` method that receives the weather measurements. The `DisplayElement` interface requires all displays to implement a `display()` method.

### 6.2 Implementing the Subject (WeatherData)

```java
import java.util.ArrayList;

public class WeatherData implements Subject {
    private ArrayList<Observer> observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList<>();
    }

    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
    }

    @Override
    public void removeObserver(Observer o) {
        int i = observers.indexOf(o);
        if (i >= 0) {
            observers.remove(i);
        }
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(temperature, humidity, pressure);
        }
    }

    public void measurementsChanged() {
        notifyObservers();
    }

    public void setMeasurements(float temperature, float humidity,
                                 float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }

    public float getTemperature() { return temperature; }
    public float getHumidity() { return humidity; }
    public float getPressure() { return pressure; }
}
```

Key points:
- An `ArrayList` stores the registered observers.
- `registerObserver()` adds an observer to the list.
- `removeObserver()` safely removes an observer.
- `notifyObservers()` iterates through the list and calls `update()` on each observer.
- `measurementsChanged()` is called when new data arrives and triggers notification.
- `setMeasurements()` simulates receiving new weather data.

### 6.3 Implementing the Observers (Display Elements)

#### CurrentConditionsDisplay

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
    private float temperature;
    private float humidity;
    private Subject weatherData;

    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    @Override
    public void display() {
        System.out.println("Current conditions: " + temperature
            + "F degrees and " + humidity + "% humidity");
    }
}
```

#### StatisticsDisplay

```java
public class StatisticsDisplay implements Observer, DisplayElement {
    private float maxTemp = 0.0f;
    private float minTemp = 200;
    private float tempSum = 0.0f;
    private int numReadings;
    private Subject weatherData;

    public StatisticsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temperature, float humidity, float pressure) {
        tempSum += temperature;
        numReadings++;
        if (temperature > maxTemp) maxTemp = temperature;
        if (temperature < minTemp) minTemp = temperature;
        display();
    }

    @Override
    public void display() {
        System.out.println("Avg/Max/Min temperature = "
            + (tempSum / numReadings) + "/" + maxTemp + "/" + minTemp);
    }
}
```

#### ForecastDisplay

```java
public class ForecastDisplay implements Observer, DisplayElement {
    private float currentPressure = 29.92f;
    private float lastPressure;
    private Subject weatherData;

    public ForecastDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temperature, float humidity, float pressure) {
        lastPressure = currentPressure;
        currentPressure = pressure;
        display();
    }

    @Override
    public void display() {
        System.out.print("Forecast: ");
        if (currentPressure > lastPressure) {
            System.out.println("Improving weather on the way!");
        } else if (currentPressure == lastPressure) {
            System.out.println("More of the same");
        } else {
            System.out.println("Watch out for cooler, rainy weather");
        }
    }
}
```

### 6.4 The Test Harness

```java
public class WeatherStation {
    public static void main(String[] args) {
        WeatherData weatherData = new WeatherData();

        CurrentConditionsDisplay currentDisplay =
            new CurrentConditionsDisplay(weatherData);
        StatisticsDisplay statisticsDisplay =
            new StatisticsDisplay(weatherData);
        ForecastDisplay forecastDisplay =
            new ForecastDisplay(weatherData);

        weatherData.setMeasurements(80, 65, 30.4f);
        weatherData.setMeasurements(82, 70, 29.2f);
        weatherData.setMeasurements(78, 90, 29.2f);
    }
}
```

### 6.5 Output

```
Current conditions: 80.0F degrees and 65.0% humidity
Avg/Max/Min temperature = 80.0/80.0/80.0
Forecast: Improving weather on the way!
Current conditions: 82.0F degrees and 70.0% humidity
Avg/Max/Min temperature = 81.0/82.0/80.0
Forecast: Watch out for cooler, rainy weather
Current conditions: 78.0F degrees and 90.0% humidity
Avg/Max/Min temperature = 80.0/82.0/78.0
Forecast: More of the same
```

---

## 7. Push vs. Pull Models

The Observer Pattern can be implemented in two ways:

### 7.1 Push Model

The Subject sends all the data to the observers through the `update()` method parameters. This is what we implemented above.

```java
// Subject pushes data
observer.update(temperature, humidity, pressure);
```

**Pros:** Observers immediately have all the data they need.
**Cons:** If the Subject has many state variables, observers receive data they may not need. Changes to the Subject's data require updating the `update()` method signature.

### 7.2 Pull Model

The Subject sends a minimal notification (or a reference to itself), and observers pull the specific data they need using the Subject's getter methods.

```java
// Subject notifies with a reference to itself
observer.update(this);

// Observer pulls what it needs
public void update(Subject subject) {
    if (subject instanceof WeatherData) {
        WeatherData wd = (WeatherData) subject;
        this.temperature = wd.getTemperature();
        this.humidity = wd.getHumidity();
    }
}
```

**Pros:** More flexible — observers request only what they need. Subject's interface doesn't change when new state is added.
**Cons:** Requires the observer to know the Subject's API. Slightly more coupling through getter methods.

> **The pull model is generally considered more "correct"** because it gives observers more control and makes the Subject's notification mechanism more generic.

---

## 8. Java's Built-in Observer Support

### 8.1 java.util.Observable and java.util.Observer

Java provided built-in support for the Observer Pattern through `java.util.Observable` (a class) and `java.util.Observer` (an interface). These were available until Java 9, when they were deprecated.

Using Java's built-in support:

```java
import java.util.Observable;
import java.util.Observer;

// Subject extends Observable (a class, not an interface)
public class WeatherData extends Observable {
    private float temperature;
    private float humidity;
    private float pressure;

    public void measurementsChanged() {
        setChanged();           // Must call this first!
        notifyObservers();      // Then notify
    }

    public void setMeasurements(float t, float h, float p) {
        this.temperature = t;
        this.humidity = h;
        this.pressure = p;
        measurementsChanged();
    }

    public float getTemperature() { return temperature; }
    public float getHumidity() { return humidity; }
    public float getPressure() { return pressure; }
}
```

```java
// Observer implements java.util.Observer
public class CurrentConditionsDisplay implements Observer, DisplayElement {
    private float temperature;
    private float humidity;
    Observable observable;

    public CurrentConditionsDisplay(Observable observable) {
        this.observable = observable;
        observable.addObserver(this);
    }

    public void update(Observable obs, Object arg) {
        if (obs instanceof WeatherData) {
            WeatherData weatherData = (WeatherData) obs;
            this.temperature = weatherData.getTemperature();
            this.humidity = weatherData.getHumidity();
            display();
        }
    }

    public void display() {
        System.out.println("Current conditions: " + temperature
            + "F degrees and " + humidity + "% humidity");
    }
}
```

### 8.2 The setChanged() Method

The `setChanged()` method adds a layer of control over notifications:

```
setChanged() {
    changed = true
}

notifyObservers(Object arg) {
    if (changed) {
        for every observer on the list {
            call update(this, arg)
        }
        changed = false
    }
}
```

This is useful for optimizing notifications. For example, if temperature fluctuates by tiny amounts, you might only call `setChanged()` when the change exceeds a meaningful threshold (e.g., half a degree).

### 8.3 Problems with java.util.Observable

There are significant design issues with Java's built-in implementation:

**Observable is a class, not an interface.**
- You must subclass it, which prevents adding Observable behavior to a class that already extends another class (Java doesn't support multiple inheritance of classes).
- This severely limits reuse potential.

**The setChanged() method is protected.**
- You can't call `setChanged()` unless you subclass Observable.
- You can't compose Observable behavior — you can't create an instance and delegate to it.
- This violates the "favor composition over inheritance" principle.

**No Observable interface exists.**
- You can't create your own implementation that is compatible with Java's Observer API.
- You can't swap implementations at runtime.

**Notification order is not guaranteed.**
- The order in which observers are notified may differ between implementations.
- Never write code that depends on a specific notification order.

> **Recommendation:** Because of these limitations, it is often better to roll your own Observer Pattern implementation using interfaces, as we did in Section 6. Java deprecated `Observable` and `Observer` in Java 9.

---

## 9. Observer Pattern in the JDK and Frameworks

### 9.1 Swing Event Listeners

The Swing GUI framework makes heavy use of the Observer Pattern. Components like `JButton` use listeners (observers) to respond to events.

```java
public class SwingObserverExample {
    JFrame frame;

    public static void main(String[] args) {
        SwingObserverExample example = new SwingObserverExample();
        example.go();
    }

    public void go() {
        frame = new JFrame();
        JButton button = new JButton("Should I do it?");

        // Register observers (listeners)
        button.addActionListener(new AngelListener());
        button.addActionListener(new DevilListener());

        frame.getContentPane().add(BorderLayout.CENTER, button);
        frame.setSize(300, 200);
        frame.setVisible(true);
    }

    class AngelListener implements ActionListener {
        public void actionPerformed(ActionEvent event) {
            System.out.println("Don't do it, you might regret it!");
        }
    }

    class DevilListener implements ActionListener {
        public void actionPerformed(ActionEvent event) {
            System.out.println("Come on, do it!");
        }
    }
}
```

In this example, the `JButton` is the Subject, and the `ActionListener` implementations are the Observers. When the button is clicked, all registered listeners are notified via their `actionPerformed()` method.

### 9.2 JavaBeans PropertyChangeListener

JavaBeans uses the `PropertyChangeListener` interface, which is another implementation of the Observer Pattern. Components fire `PropertyChangeEvent` objects when their properties change, and listeners respond accordingly.

---

## 10. Design Principles Recap

The Observer Pattern connects to several important OO design principles:

| Principle | How It Applies |
|---|---|
| **Encapsulate what varies** | The varying part (which observers exist) is encapsulated in a dynamic list |
| **Program to interfaces, not implementations** | Both Subject and Observer are defined as interfaces |
| **Favor composition over inheritance** | Observers are composed with the Subject via registration, not inheritance |
| **Strive for loosely coupled designs** | Subject and observers interact with minimal knowledge of each other |
| **Open/Closed Principle** | The system is open for extension (new observers) without modifying the Subject |

---

## 11. Real-World Examples of the Observer Pattern

### 11.1 Social Media Notifications

When you follow someone on Twitter/X or Instagram, you become an observer. The person you follow is the subject. When they post new content, all followers receive notifications. You can unfollow (unsubscribe) at any time.

```
Subject: @elonmusk (Twitter account)
Observers: All followers
Event: New tweet posted
Notification: Push notification to all followers
```

### 11.2 Email Newsletter Subscriptions

Websites like Medium or Substack work exactly like the newspaper analogy. You subscribe to a newsletter (register as observer), receive new articles automatically (update notification), and can unsubscribe at any time.

### 11.3 Stock Market Ticker

A stock exchange broadcasts price changes. Trading applications, portfolio dashboards, and alert systems all subscribe to specific stock symbols. When the price changes, all subscribers are updated in real time.

```java
// Conceptual example
public interface StockObserver {
    void onPriceChange(String symbol, double newPrice);
}

public class StockExchange implements StockSubject {
    private Map<String, List<StockObserver>> observers = new HashMap<>();

    public void subscribe(String symbol, StockObserver observer) {
        observers.computeIfAbsent(symbol, k -> new ArrayList<>()).add(observer);
    }

    public void priceChanged(String symbol, double newPrice) {
        for (StockObserver observer : observers.getOrDefault(symbol, List.of())) {
            observer.onPriceChange(symbol, newPrice);
        }
    }
}
```

### 11.4 MVC (Model-View-Controller) Architecture

The MVC pattern heavily relies on Observer. The **Model** is the Subject, and the **Views** are Observers. When the Model's data changes (e.g., a database update), all Views are notified and refresh themselves. This is the foundation of many web and desktop frameworks.

```
Model (Subject) ──notifies──▷ View 1 (Observer): Desktop UI
                ──notifies──▷ View 2 (Observer): Mobile UI
                ──notifies──▷ View 3 (Observer): Web Dashboard
```

### 11.5 Event Bus / Message Queues

Systems like Apache Kafka, RabbitMQ, and Google Pub/Sub are large-scale implementations of the Observer Pattern. Producers publish messages to topics (subjects), and consumers subscribe to topics and process messages as they arrive.

### 11.6 Smart Home Systems

A smart home hub acts as a subject. When a sensor detects motion, all subscribed devices respond: lights turn on, cameras start recording, the phone app sends an alert. Each device is an independent observer.

### 11.7 Auction Systems

In an online auction, the auction item is the subject. All bidders who are "watching" the item are observers. When a new bid is placed, all watchers are notified of the new price.

### 11.8 Chat Applications

In a group chat, the chat room is the subject. All participants are observers. When someone sends a message, all participants receive it. When someone leaves the group, they are removed from the observer list.

---

## 12. Key Takeaways

1. The Observer Pattern defines a one-to-many relationship between objects.
2. Subjects (Observables) update Observers using a common interface.
3. Observers are loosely coupled — the Subject knows nothing about them other than that they implement the Observer interface.
4. You can push or pull data from the Observable (pull is generally preferred).
5. Don't depend on a specific order of observer notification.
6. Java's built-in `java.util.Observable` has significant limitations (it's a class, not an interface) and was deprecated in Java 9.
7. Don't be afraid to roll your own implementation when the built-in one doesn't fit.
8. The pattern is ubiquitous: Swing listeners, JavaBeans, MVC, message queues, social media notifications, and countless other systems use it.

---

## 13. Exercises

### Exercise 1: Implement a Heat Index Display
Create a new display element called `HeatIndexDisplay` that computes and displays the heat index (a combination of temperature and humidity). Register it as an observer of `WeatherData` without modifying the `WeatherData` class.

### Exercise 2: Add Unsubscribe Functionality
Modify the `CurrentConditionsDisplay` so it has a method `unsubscribe()` that removes itself from the WeatherData's observer list. Test that it no longer receives updates after unsubscribing.

### Exercise 3: Implement the Pull Model
Refactor the Weather Station implementation to use the pull model. The `update()` method should receive a reference to the Subject, and each observer should pull only the data it needs.

### Exercise 4: Event System
Design an event system for a simple game where a `GameEventManager` (Subject) can notify different systems (ScoreDisplay, SoundSystem, AchievementTracker) when events like "enemy defeated" or "level completed" occur. Use the Observer Pattern.

---

*Reference: Freeman, E. & Robson, E. (2004). Head First Design Patterns. O'Reilly Media. Chapter 2: The Observer Pattern.*
