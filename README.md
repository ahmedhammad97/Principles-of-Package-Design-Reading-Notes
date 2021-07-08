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



## PART 2: Package Design

Writing packages and classes is much more difficult than writing just code statements.

This part is will discuss six principles related to two main concepts:

#### **Cohesion**
Cohesion is about which classes belongs together. There are different ways of cohesion (eg. *logical cohesion*, *communicational cohesion*, etc.), but the most important is *functional cohesion*, which is achieved when all things in a "module" (eg. package) can be used to perform a single, well-defined task.

SOLID principles automatically enforce the classes to be more cohesive, but there are three more principles to help in that. Applying these principles will lead to smaller packages that are easier to maintain and use.

#### Coupling

Coupling is when a code entity (eg. class) depends on another one. When a class from one package depends on a class from another package, it is called *package coupling*.

The upcoming three principles will prevent your systems from having incompatible dependency versions, circular dependencies, or depending on unstable dependencies at all.
 

### The Release/Reuse Equivalence Principle

**The granule of reuse is the granule of release.**

You can only reuse the code that you actually release, and you should only release as much code as you can reasonably reuse. It makes no sense to invest all the time and energy needed to properly release code if nobody is going to use anyways.

If you decided to release a package, these are the things you need to take care of:
- Keep your package under version control (eg. Git)
- Add Package definition file (eg. *package.json*, *app.csproj*, etc.)
- Use semantic versioning (eg. 1.0.2, 1.5.3, 2.0.0, etc.)
- Design for backward compatibility (eg. provide the same functionality in previous minor versions)
- Do not throw anything away (eg. classes, functions, parameters, constants, etc.)
- When you rename something, add a proxy with the old name that calls the newly named function. Annotate is as deprecated though
- Only add parameters at the end and with a default value
- Functions should not have implicit side effects (as consumers might depend on it)
- Dependency versions should be permissive (ie. not too strict as 2.3.5, but as loose as 2.x.x)
- Use objects instead of primitive values
- Use objects for encapsulation of state and behavior
- Use object factories
- Add metafiles (eg. README, License, etc.)
- Use static analysis tools
- Add tests
- Setup continuous integration

### The Common Reuse Principle

**Classes that are used together are packed together.**

This principle helps us decide which classes should be put together in a package, and which classes should be moved to another package.

A package that adheres to this principle has the following characteristics:
- It is coherent, which means all the classes it contains are about the same thing
- All its dependencies are required (ie. it has not optional dependencies)
- It uses dependency inversion to avoid concrete dependencies
- It is open for extension and closed for modification

Packages that violates the principle usually have some *parallel* features that are not materially related. This is usually achieved through splitting the package or extracting only the problematic parts into another package. At the end, classes that are *always used together* should be together.

Splitting packages have their cost too. The smaller the packages are, the more you will have of them, the more work you have to put into making new releases, managing repositories, issues, etc. Therefore, you need to find the golden middle between too many small packages, and too few large packages.

In practice I tend to just create the class inside the package I am already working on. Afterward, I may decide to move it to another package, based on the following facts:
- If the class introduce a dependency that is optional/suggested
- If the class is useful without the rest of the package

This principle can only be maximized. You cannot always follow it perfectly.

### The Common Closure Principle

**A change that affects a package affects all the classes in that package.**

Following this principles will prevent you from opening a package for all kinds of unrelated reasons, which itself prevents new releases that are irrelevant to most of the consumers.

This is achieved by grouping the classes that changes together in a separate package.

There are still some reasons for which it is okay to open a package:
- If something changes about a dependency (eg. upgrade, replacement, etc.)
- If requirements have changed regarding a piece of business logic
- If part of the infrastructure changes

You should track how many packages need to be released again after each change, and aim to minimize this number by splitting the package.

#### The Tension Triangle of Cohesion Principles

Robert Martin suggests that the three cohesion principles form a triangle (one at each vertex). A package may move anywhere around the triangle in its life cycle, favoring a principle over the other (where moving to a corner means it implements it maximally but neglects the other principles).

### The Acyclic Dependencies Principle

**There must be no cycles in the dependency structure.**

Packages can get coupled together through composition, inheritance, interface implementation, object instantiation, global function usage, and more. However, when two packages depend on each other in a cyclic way it might cause irresolvable problems.

The first two question to ask in the case of a circular dependency are:
- Is it really the entire object that we need? If not, we can just inject the part we need and effectively remove the cycle.
- Is it possible to change the relationship from bidirectional to unidirectional?

The cyclic dependency can be further solved through other techniques:
- Extracting the classes causing the cycle into new separate packages (or may be the same package if they are always used together).
- Use dependency inversion by depending on an abstract entity rather than a concrete class
- Applying any behavioral design pattern (eg. Mediator) to break the communication between the dependent objects
- Applying *Chain of Responsibility* pattern
- Using a combination of both *Mediator* and *Chain of Responsibility* patterns, which turns to be an *event dispatcher*.

### The Stable Dependencies Principle

**A package should only depend upon packages that are more stable than it is.**

Before adding a dependency to your project, you need to consider whether it is likely that the dependency is going to change. How easy for the maintainers to change it? And is it considered *stable* or *unstable*?

A package that needs to change often to accommodate a change in one of its dependencies should be considered unstable. However there is a more accurate way of measuring stability, which is the *I metric*. (ie. Instability metric).

*I metric* equals *C-out* divided by *C-in* plus *C-out*, where *C-in* is the number of outside classes dependent on the package, and *C-out* is the number of outside classes that the package depends on. A highly instable package have an *I-metric* near 1, while stable packages have an *I metric* near 0.

Sometimes it is not an option to modify the dependency packages, however, it is always an option to apply dependency inversion and create an abstraction (eg. interface) to depend on instead of directly depending on the outer package.

### The Stable Abstraction Principle

**The abstraction of a package should be in proportion to its stability.**

It's always better to depend on an abstract package than on a concrete package, and dependencies should have and increasing abstractness.

*A metric* is a way of calculating the packages stability. *A metric* equals *C-abstract* divided by *C-concrete* plus *C-abstract*, where *C-abstract* is number of abstract classes in the package, and *C-concrete* is the number of concrete classes in the package. A high abstract package will have an *A metric* near 1.

It might look that extracting all interfaces from a package to a separate package might enforce the problem, however, it does not. It also violates the *Common Reuse principle*

Applying this principle will increase stability as well.

