# Principles of Package Design Reading Notes

## Introduction
The book have two main parts, one for discussing class design principles (*SOLID* principles), and another for discussing packages design principles (mainly *cohesion* and *coupling*).

## PART 1: Class Design

It is harder to be strict about the class design principles. You are not required to always follow them, but in theory you *should*.

The SOLID principles prepare your codebase for future changes, you want these changes to be local and small, not global and big.

### The Single Responsibility Principle

**A class should have one, and only one, reason to changes.**

The goal of applying this principle is to try to minimize the number of responsibilities of each class, ideally reducing it to one. This is usually done by refactoring the class through extracting collaborator classes from the targeted class, leaving it with just one responsibility, while using the other collaborator classes through composition or inheritance.

You will notice that class with single responsibility is easier to test, and maintain.

A sign of a class with many responsibilities is that it has many dependencies which are injected as constructor arguments.

### The Open/Closed Principle

**You should be able to extend a class's behavior without modifying it.**

Extension of a class means that you can influence its behavior from the outside and leave the class untouched. This is usually done through several steps:
- Apply the Single Responsibility principle to extract any collaborating objects
- Inject collaborating objects as constructor arguments (eg. dependency injection)
- Depend on abstractions (eg. interfaces) instead of concrete implementations
- Rely on Factories for creating instances instead of creating them inside the class
- Design the collaborating classes to be easily *decorated* (ie. wrapped) 
- Move any logic related to some of the dependencies to the dependency class instead

There a list of characteristics of a class that violates the open/closed principle:
- It contains conditions to determine a strategy
- Conditions using the same variables or constants are recurring inside the class or related classes
- The class contains hard-coded references to other classes or class names
- Inside the class, objects are being created using the `new` operator
- The class has protected properties or methods, to allow changing its behavior by overriding state or behavior

### The Liskov Substitution Principle

**Derived classes must be substitutable for their base classes.**

A base class can be a concrete class, an abstract class, or an interface. In all cases, a class that implements or inherits a base class, must be a good substitute for it. A good substitute should:
- Provide an implementation for all the methods of the base class
- Return the type of things the base class prescribes (or more specific types)
- Not put extra constraints on arguments for methods
- Not use non-strict typing to break the contract that was provided by the base class

The most common violations for this principle are:
- Improper generalization or *leaky abstraction*, when a single base class exposes more functions that needed for single use case. This is usually solved by splitting the base class or interface into smaller pieces, each suitable for just one specific use case.
- When a derived class and its base class returns different types for the same function. This is usually solved by allowing the wrapping the derived class output by a subtype of the base class output.

### The Interface Segregation Principle

**Make fine-grained interfaces that are client specific**

Fine grained interfaces means interfaces with small amount of functions, and client specific means that that functions should make sense from the point of view of the client. This can be achieved by looking at different clients that use the class or interface, then group the functions that are used together in separate interfaces.

The most common violation for this principle is when a client is forced to depend on a method that it does not use.

Applying this principle leads to smaller interface which reduces the number of reasons to change. Also, it gives us the freedom to add more public functions that are not part of the published interface.

### The Dependency Inversion Principle

**Depend on abstractions, not on concretions.**

Abstractions should not depend upon details, but details should always depend upon abstractions.

Every class has two levels of abstractions: one perceived by the clients that should be more abstract, and another that is going on inside that is more concrete.

Usually this is done through introducing interfaces that decouples the class from any concrete dependency. However, simply depending on an interface is not always enough, sometimes we have to introduce intermediate dependencies that bridges that gap between high-level and low-level classes.

This principles also applies to packages, they should always depend in the direction of abstractness.

Not all third-party code requires you to apply the *Dependency Inversion* principle.

Sometimes it might be better to write your own package instead of depending on a third-party package. When reinventing the wheel, sometimes you end up with a better wheel.

#### When should you publish an explicit interface for a class?
- If not all public methods are meant to be used for regular clients
- If the class uses I/O, you will need an interface for creating test double
- If the class depends on third-party code
- If you want to introduce an abstraction for multiple specific things
- If you foresee that the user want to replace part of the object hierarchy

Other than that, just stick to a final class with no interfaces!

Classes that almost never need an interface are:
- Classes that models some concept from your domain
- Classes that otherwise represents stateful objects
- Classes that represents a particular piece of business logic, or a calculation
- 
