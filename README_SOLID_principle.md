# SOLID Principles

SOLID is a set of five object-oriented design principles that help developers create software that is **maintainable, scalable, testable, and easy to modify**.

SOLID stands for:

- **S** — Single Responsibility Principle
- **O** — Open/Closed Principle
- **L** — Liskov Substitution Principle
- **I** — Interface Segregation Principle
- **D** — Dependency Inversion Principle

---

# 1. Single Responsibility Principle — SRP

> A class should have only one reason to change.

A class should focus on **one responsibility** instead of handling multiple unrelated tasks.

## Bad Example

```java
class Employee {

    public void calculateSalary() {
        // Salary calculation
    }

    public void saveToDatabase() {
        // Database logic
    }

    public void generateReport() {
        // Report generation
    }
}
```

The `Employee` class has multiple responsibilities:

- Salary calculation
- Database operations
- Report generation

Any change to database logic or reporting logic requires modifying the same class.

## Better Example

```java
class Employee {

    public void calculateSalary() {
        // Salary calculation
    }
}

class EmployeeRepository {

    public void save(Employee employee) {
        // Database logic
    }
}

class EmployeeReportGenerator {

    public void generate(Employee employee) {
        // Report generation
    }
}
```

Each class now has a single responsibility.

### Explanation

**SRP means one class should do one job and have one reason to change.**

### Benefits

- Easier maintenance
- Easier testing
- Smaller classes
- Better readability
- Lower coupling

---

# 2. Open/Closed Principle — OCP

> Software entities should be open for extension but closed for modification.

We should be able to add new functionality **without modifying existing working code**.

## Bad Example

```java
class DiscountCalculator {

    public double calculateDiscount(String customerType) {

        if (customerType.equals("REGULAR")) {
            return 10;
        }

        if (customerType.equals("PREMIUM")) {
            return 20;
        }

        return 0;
    }
}
```

Every time we add a new customer type, we must modify `DiscountCalculator`.

## Better Example

```java
interface Discount {

    double calculate();
}

class RegularDiscount implements Discount {

    public double calculate() {
        return 10;
    }
}

class PremiumDiscount implements Discount {

    public double calculate() {
        return 20;
    }
}
```

Now we can add another discount:

```java
class StudentDiscount implements Discount {

    public double calculate() {
        return 15;
    }
}
```

Existing classes do not need to be modified.

### Explanation

**OCP means existing code should remain stable while new behavior is added through extension, usually using interfaces, inheritance, composition, or polymorphism.**

### Benefits

- Reduces risk of breaking existing code
- Makes systems easier to extend
- Encourages polymorphism
- Improves maintainability

---

# 3. Liskov Substitution Principle — LSP

> Objects of a superclass should be replaceable by objects of its subclasses without breaking the program.

A child class must behave in a way that is consistent with the expectations created by the parent class.

## Bad Example

```java
class Bird {

    public void fly() {
        System.out.println("Flying");
    }
}

class Penguin extends Bird {

    @Override
    public void fly() {
        throw new UnsupportedOperationException(
            "Penguins cannot fly"
        );
    }
}
```

The problem is that `Penguin` cannot correctly replace `Bird`.

Code expecting every `Bird` to fly will fail.

## Better Example

```java
interface Bird {

    void eat();
}

interface FlyingBird extends Bird {

    void fly();
}

class Sparrow implements FlyingBird {

    public void eat() {
        System.out.println("Eating");
    }

    public void fly() {
        System.out.println("Flying");
    }
}

class Penguin implements Bird {

    public void eat() {
        System.out.println("Eating");
    }
}
```

Now the hierarchy correctly represents the behavior of each type.

### Explanation

**LSP means a subclass should be usable wherever its parent type is expected without changing the correctness of the program.**

### Easy Way to Remember

If:

```java
Parent object = new Child();
```

causes unexpected behavior, the design may violate LSP.

### Benefits

- Safer inheritance
- Predictable polymorphism
- Better abstraction
- Fewer runtime surprises

---

# 4. Interface Segregation Principle — ISP

> Clients should not be forced to depend on methods they do not use.

Instead of creating one large interface, create **small and focused interfaces**.

## Bad Example

```java
interface Worker {

    void work();

    void eat();

    void sleep();
}
```

Now imagine a robot:

```java
class Robot implements Worker {

    public void work() {
        System.out.println("Working");
    }

    public void eat() {
        // Robot does not eat
    }

    public void sleep() {
        // Robot does not sleep
    }
}
```

The robot is forced to implement methods it does not need.

## Better Example

```java
interface Workable {

    void work();
}

interface Eatable {

    void eat();
}

interface Sleepable {

    void sleep();
}
```

Human:

```java
class Human implements Workable, Eatable, Sleepable {

    public void work() {
        System.out.println("Working");
    }

    public void eat() {
        System.out.println("Eating");
    }

    public void sleep() {
        System.out.println("Sleeping");
    }
}
```

Robot:

```java
class Robot implements Workable {

    public void work() {
        System.out.println("Working");
    }
}
```

Each class implements only what it actually needs.

### Explanation

**ISP means prefer multiple small, specific interfaces instead of one large general-purpose interface.**

### Benefits

- Lower coupling
- Cleaner interfaces
- Easier implementation
- Easier testing
- Fewer unnecessary dependencies

---

# 5. Dependency Inversion Principle — DIP

> High-level modules should not depend on low-level modules. Both should depend on abstractions.

Also:

> Abstractions should not depend on details. Details should depend on abstractions.

## Bad Example

```java
class MySQLDatabase {

    public void save(String data) {
        System.out.println("Saving to MySQL");
    }
}

class UserService {

    private MySQLDatabase database = new MySQLDatabase();

    public void saveUser(String user) {
        database.save(user);
    }
}
```

`UserService` is tightly coupled to `MySQLDatabase`.

Changing from MySQL to MongoDB requires modifying `UserService`.

## Better Example

Create an abstraction:

```java
interface Database {

    void save(String data);
}
```

Implement it:

```java
class MySQLDatabase implements Database {

    public void save(String data) {
        System.out.println("Saving to MySQL");
    }
}
```

Another implementation:

```java
class MongoDatabase implements Database {

    public void save(String data) {
        System.out.println("Saving to MongoDB");
    }
}
```

High-level class:

```java
class UserService {

    private Database database;

    public UserService(Database database) {
        this.database = database;
    }

    public void saveUser(String user) {
        database.save(user);
    }
}
```

Usage:

```java
Database database = new MySQLDatabase();

UserService service =
    new UserService(database);

service.saveUser("John");
```

Now we can easily replace the database:

```java
Database database = new MongoDatabase();

UserService service =
    new UserService(database);
```

`UserService` does not need to change.

###  Explanation

**DIP means business logic should depend on abstractions such as interfaces instead of directly depending on concrete implementations.**

Dependency Injection is one common technique used to achieve DIP.

### Benefits

- Loose coupling
- Easier testing
- Easy replacement of implementations
- Better flexibility
- Better maintainability

---

# SOLID Summary

| Principle | Meaning |
|---|---|
| **S — SRP** | One class should have one responsibility |
| **O — OCP** | Extend behavior without modifying existing code |
| **L — LSP** | Subclasses should safely replace parent classes |
| **I — ISP** | Prefer small, focused interfaces |
| **D — DIP** | Depend on abstractions, not concrete implementations |

---

# Easy Way to Remember SOLID

```text
S → One class, one job

O → Add new behavior without changing old code

L → Child objects should work wherever parent objects work

I → Don't force classes to implement unnecessary methods

D → Depend on interfaces, not implementations
```

---

# Common  Questions

## What is SOLID?

SOLID is a collection of five object-oriented design principles used to build maintainable, extensible, loosely coupled, and testable software.

---

## Why are SOLID principles important?

SOLID principles help reduce:

- Tight coupling
- Large classes
- Duplicate code
- Difficult testing
- Difficult maintenance
- Regression bugs

They make software easier to understand and extend.

---

## Is SOLID only for Java?

No.

SOLID principles can be applied to most object-oriented languages, including:

```text
Java
C#
C++
Python
TypeScript
Kotlin
Swift
```

The principles are about **software design**, not a particular programming language.

---

## What is the difference between SRP and ISP?

### SRP

Concerned with **classes**.

A class should have one responsibility.

### ISP

Concerned with **interfaces**.

Interfaces should contain only methods required by their clients.

---

## What is the difference between OCP and LSP?

### OCP

Focuses on extending functionality without modifying existing code.

### LSP

Focuses on ensuring subclasses correctly behave as their parent types.

---

## What is the difference between Dependency Injection and Dependency Inversion?

**Dependency Inversion Principle** is a design principle.

**Dependency Injection** is a technique commonly used to implement that principle.

Example:

```java
class Service {

    private Repository repository;

    public Service(Repository repository) {
        this.repository = repository;
    }
}
```

The dependency is passed from outside rather than created inside the class.

---

# Real-World Example

Consider an e-commerce application.

We may have:

```text
OrderService
PaymentProcessor
NotificationService
OrderRepository
```

Using SOLID:

### SRP

```text
OrderService → Handles order business logic

PaymentProcessor → Handles payments

NotificationService → Sends notifications

OrderRepository → Handles database operations
```

### OCP

New payment methods can be added without changing existing payment logic.

```text
PaymentMethod

├── CreditCardPayment
├── PayPalPayment
├── ApplePayPayment
└── CryptoPayment
```

### LSP

Every payment implementation should work wherever `PaymentMethod` is expected.

### ISP

Instead of:

```text
PaymentInterface
    pay()
    refund()
    generateInvoice()
    sendEmail()
```

create smaller interfaces:

```text
Payable
Refundable
InvoiceGenerator
NotificationSender
```

### DIP

Instead of:

```java
OrderService -> MySQLRepository
```

use:

```text
OrderService
      |
      v
OrderRepository
      ^
      |
-------------------
|                 |
MySQL           MongoDB
```

The business logic depends on an abstraction.

---

#  Cheat Sheet

```text
S — Single Responsibility
    One class = one job.

O — Open/Closed
    Open for extension.
    Closed for modification.

L — Liskov Substitution
    Child should safely replace parent.

I — Interface Segregation
    Small interfaces are better than fat interfaces.

D — Dependency Inversion
    Depend on abstractions, not concrete classes.
```

---


**"What are SOLID principles?"**

You can answer:

> SOLID is a set of five object-oriented design principles used to create maintainable and loosely coupled software. Single Responsibility says a class should have one responsibility. Open/Closed says software should be extendable without modifying existing code. Liskov Substitution says subclasses should safely replace their parent types. Interface Segregation says we should prefer small focused interfaces, and Dependency Inversion says high-level modules should depend on abstractions rather than concrete implementations.

---

# Important Point

SOLID does **not** mean that every program must contain many interfaces and classes.

The goal is to create code that is:

```text
Easy to understand
Easy to test
Easy to modify
Easy to extend
Loosely coupled
Highly cohesive
```

Overusing SOLID can create unnecessary abstraction and complexity.

Use these principles when they improve the design, not simply to increase the number of classes.

---

# Final Revision

```text
SRP → One responsibility.

OCP → Extend, don't modify.

LSP → Child must behave like parent.

ISP → Keep interfaces small.

DIP → Depend on abstractions.
```

## One-Line Memory Trick

**"One job, extend safely, substitute correctly, keep interfaces small, depend on abstractions."**