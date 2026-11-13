# Module 03: Object-Oriented Programming (OOP) Deep-Dive

## Unit 3.3: Abstract Classes and Interfaces

### 1. Introduction to Abstraction

**Abstraction** is the foundational OOP principle of isolating **what** an entity does from **how** it accomplishes that task. By establishing rigid contracts through abstract types, high-level architectures decouple API consumers from underlying implementation details.

```
                            Contract Abstraction in Java
                                         │
         ┌───────────────────────────────┴───────────────────────────────┐
         ▼                                                               ▼
   Abstract Classes                                                  Interfaces
   (Partial Abstraction: 0% to 100%)                              (Full Contract Abstraction)
   - Models identity & shared state (IS-A)                        - Models capability / behavioral role (CAN-DO)
   - Maintains constructors & instance state                      - Stateless contracts (constant fields only)
   - Single inheritance hierarchy (extends)                       - Multiple interface implementation (implements)
```

---

### 2. Abstract Classes & Abstract Methods

An **abstract class** is a class declared with the `abstract` modifier. It serves as an incomplete base class blueprint that cannot be directly instantiated.

```java
public abstract class DatabaseConnector {
    // 1. Instance state (fields)
    protected String connectionUrl;

    // 2. Constructor (called during subclass instantiation via super())
    public DatabaseConnector(String url) {
        this.connectionUrl = url;
    }

    // 3. Abstract Method (contract without body; must be implemented by concrete subclass)
    public abstract void connect();

    // 4. Concrete Method (shared implementation)
    public void disconnect() {
        System.out.println("Closing socket on: " + connectionUrl);
    }
}
```

#### 2.1 Abstract Class Invariants

1. **Instantiation Prohibited:** Attempting to invoke `new DatabaseConnector("...")` yields a **compile-time error**: _DatabaseConnector is abstract; cannot be instantiated_.
2. **Abstract Methods:** A method marked `abstract` has no method body (`{ }`), terminating directly with a semicolon (`;`). It forces concrete subclasses to provide an override.
3. **Abstract Class Without Abstract Methods:** A class can be declared `abstract` even if it contains **zero abstract methods**. This prevents client code from instantiating the class directly.
4. **Mandatory Abstract Class Declaration:** If a class contains **at least one abstract method**, the class **must be declared `abstract`**. Failure to do so is a compile-time error.
5. **Subclass Obligation:** A concrete subclass extending an abstract superclass **must override and implement all inherited abstract methods**. If it fails to implement even one, the subclass must also be declared `abstract`.

#### 2.2 Prohibited Modifier Combinations for Abstract Methods

The `abstract` keyword requires subclasses to provide an implementation via dynamic dispatch. The following modifier combinations are fundamentally contradictory and cause **compile-time errors**:

| Illegal Combination     | Reason for Compile-Time Failure                                                                                        |
| :---------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| `abstract private`      | `private` methods are invisible to subclasses; an abstract method _must_ be overridden by a subclass.                  |
| `abstract static`       | `static` methods belong to class metadata and are bound at compile time; `abstract` requires runtime dynamic dispatch. |
| `abstract final`        | `final` strictly prohibits method overriding; `abstract` mandates method overriding.                                   |
| `abstract synchronized` | Synchronization locks an object monitor during execution; abstract methods have no execution body.                     |
| `abstract native`       | `native` methods provide implementations in C/C++; abstract methods have no implementation.                            |

---

### 3. Abstract Class vs. Concrete Class

| Architectural Metric      | Abstract Class                        | Concrete Class                                          |
| :------------------------ | :------------------------------------ | :------------------------------------------------------ |
| **Instantiation (`new`)** | ❌ **Prohibited** directly            | ✅ **Permitted** directly                               |
| **Method Types Allowed**  | Abstract methods AND concrete methods | **Concrete methods only** (abstract methods prohibited) |
| **Constructors**          | ✅ Permitted (chained via `super()`)  | ✅ Permitted                                            |
| **Primary Intent**        | Architectural template & code sharing | Complete operational implementation                     |
| **Subclassing**           | Designed explicitly to be extended    | Can be extended (unless marked `final`)                 |

---

### 4. Introduction to Interfaces

An **interface** is an abstract reference type that defines a pure behavioral contract. In traditional Java (prior to Java 8), an interface was a collection of abstract method specifications and constant values (`public static final`).

#### 4.1 Interface Declaration & Implicit Modifiers

```java
public interface RemoteDevice {
    // Implicitly: public static final int MAX_RANGE_METERS = 50;
    int MAX_RANGE_METERS = 50;

    // Implicitly: public abstract void powerOn();
    void powerOn();

    // Implicitly: public abstract void setFrequency(double mhz);
    void setFrequency(double mhz);
}
```

```
                     Implicit Interface Member Qualifiers
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         ▼                                                           ▼
     All Fields                                                 All Methods
  (Implicitly by javac)                                     (Implicitly by javac)
  • public                                                  • public
  • static                                                  • abstract (unless default/static/private)
  • final
```

> ⚠️ **The Public Visibility Invariant:**
> All methods declared in an interface are implicitly **`public`**. When an implementing class overrides these methods, it **must explicitly use the `public` modifier**. Omitting `public` in the class causes a compiler error: _attempting to assign weaker access privileges; was public_.

---

### 5. Implementation & Multiple Interface Inheritance

A class implements an interface using the **`implements`** keyword. Java enables multiple inheritance of type/interface contracts by allowing a class to implement an arbitrary number of interfaces.

```java
public interface Playable { void play(); }
public interface Recordable { void record(); }

// Concrete class implementing multiple interfaces
public class MediaStation implements Playable, Recordable {
    @Override
    public void play() {
        System.out.println("Streaming media stream");
    }

    @Override
    public void record() {
        System.out.println("Writing audio bytes to disk");
    }
}
```

#### 5.1 Interface-to-Interface Inheritance (`extends`)

Interfaces support **multiple inheritance** directly. An interface can extend multiple parent interfaces using `extends`:

```java
public interface SuperA { void ping(); }
public interface SuperB { void pong(); }

// An interface extending multiple interfaces:
public interface CompositeService extends SuperA, SuperB {
    void execute(); // Inherits ping(), pong(), adds execute()
}
```

---

### 6. Interface vs. Abstract Class (Core Comparison Matrix)

| Dimension                   | Abstract Class                                           | Interface (Modern Java 8+)                             |
| :-------------------------- | :------------------------------------------------------- | :----------------------------------------------------- |
| **Inheritance Model**       | Single inheritance (`class A extends B`)                 | Multiple implementation (`class A implements B, C, D`) |
| **Instance State (Fields)** | Any modifiers (`private`, `protected`, `volatile`, etc.) | **Only `public static final`** constants               |
| **Constructors**            | ✅ **Permitted** (initialized via subclass `super()`)    | ❌ **Prohibited** (cannot declare constructors)        |
| **Methods Supported**       | Abstract, concrete, `final`, `static`                    | Abstract, `default`, `static`, `private` (Java 9+)     |
| **Relationship Model**      | **IS-A** (Identity & class specialization)               | **CAN-DO / Role** (Capability specification)           |
| **Memory Allocation**       | Fields occupy memory inside heap object                  | Constants reside in class mirror; no instance state    |
| **Speed / Dispatch**        | Fast (`invokevirtual` via class vtable)                  | Slightly slower (`invokeinterface` via itable)         |

---

### 7. Modern Interface Evolution (Java 8 & Java 9)

To support functional programming and backward compatibility (allowing new methods to be added to existing standard library interfaces like `java.util.Collection` without breaking millions of legacy implementations), Java evolved the interface model significantly.

```
                               Interface Methods Roadmap
                                           │
         ┌───────────────────┬─────────────┴─────────────┬───────────────────┐
         ▼                   ▼                           ▼                   ▼
  Abstract Methods     Default Methods             Static Methods      Private Methods
  (All Java versions)  (Introduced in Java 8)      (Java 8)            (Introduced in Java 9)
  • No method body     • Concrete 'default' body   • Utility methods   • Helper methods
  • Must be overridden • Inherited / Overridable   • Not inherited     • Internal code reuse
```

#### 7.1 Default Methods (Defender Methods - Java 8)

Marked with the `default` keyword; provides a concrete implementation inside the interface. Implementing classes inherit this implementation unless they choose to override it.

```java
public interface SecurityModule {
    void authenticateUser(String username, String hash);

    // Default method providing non-breaking extension
    default void logSecurityAudit(String event) {
        System.out.println("[AUDIT " + System.currentTimeMillis() + "]: " + event);
    }
}
```

#### 7.2 The Diamond Problem with Default Methods

If a class implements two interfaces that declare identical default method signatures, the compiler **fails with an ambiguity error**:

```java
interface EngineA {
    default void start() { System.out.println("EngineA starting"); }
}

interface EngineB {
    default void start() { System.out.println("EngineB starting"); }
}

public class HybridVehicle implements EngineA, EngineB {
    // COMPILE ERROR: class HybridVehicle inherits unrelated defaults for start() from types EngineA and EngineB

    // MANDATORY RESOLUTION: Subclass must override the conflicting method!
    @Override
    public void start() {
        // Disambiguate using InterfaceName.super.methodName()
        EngineA.super.start();
        System.out.println("Hybrid vehicle systems engaged.");
    }
}
```

#### 7.3 Static Interface Methods (Java 8)

- Declared with the `static` keyword and must include a concrete implementation body.
- Act as pure utility methods associated with the interface namespace.
- **Not inherited by implementing classes!** They can **only** be invoked using the interface name:

```java
public interface CryptoValidator {
    static boolean isSecure(String algorithm) {
        return "SHA-256".equalsIgnoreCase(algorithm);
    }
}

// In client code:
boolean safe = CryptoValidator.isSecure("SHA-256"); // Valid!
// implementingClassInstance.isSecure("...");       // COMPILE ERROR!
```

#### 7.4 Private & Private Static Interface Methods (Java 9)

Introduced in Java 9 to promote code reuse and encapsulation within interfaces, preventing multiple default or static methods from duplicating shared logic.

- Cannot be accessed or overridden by implementing classes or outside code.
- `private`: Accessible only by default methods and other private methods in the same interface.
- `private static`: Accessible by static methods, default methods, and private methods in the same interface.

```java
public interface TransactionProcessor {
    default void processCredit(double amount) {
        validateTransaction(amount);
        System.out.println("Crediting: " + amount);
    }

    default void processDebit(double amount) {
        validateTransaction(amount);
        System.out.println("Debiting: " + amount);
    }

    // Java 9 Private Method: Encapsulates shared internal helper logic
    private void validateTransaction(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Invalid transaction amount: " + amount);
        }
    }
}
```

---

### 8. Functional Interfaces (SAM - Single Abstract Method)

A **Functional Interface** is an interface that contains **exactly one abstract method**. It acts as the structural target type for Lambda Expressions and Method References.

#### 8.1 The `@FunctionalInterface` Annotation

An optional informative annotation. If applied, the compiler verifies that the interface contains **exactly one abstract method**. If it contains zero or more than one abstract method, compilation fails.

```java
@FunctionalInterface
public interface Transformer<T, R> {
    // The Single Abstract Method (SAM)
    R transform(T input);

    // default and static methods DO NOT count against the single abstract method rule!
    default void logTransformation(T input) {
        System.out.println("Processing input: " + input);
    }

    // Methods inherited from java.lang.Object DO NOT count!
    @Override
    boolean equals(Object obj);
}
```

#### 8.2 Standard Built-In Functional Interfaces (`java.util.function`)

| Interface                 | Method Signature    | Functionality / Purpose                                                          |
| :------------------------ | :------------------ | :------------------------------------------------------------------------------- |
| **`Predicate<T>`**        | `boolean test(T t)` | Evaluates a boolean condition on input `t`.                                      |
| **`Function<T, R>`**      | `R apply(T t)`      | Maps/transforms input of type `T` to output of type `R`.                         |
| **`Consumer<T>`**         | `void accept(T t)`  | Consumes input `t` and performs side effects; returns void.                      |
| **`Supplier<T>`**         | `T get()`           | Factory pattern; produces output of type `T` without input.                      |
| **`UnaryOperator<T>`**    | `T apply(T t)`      | Specialization of `Function<T, T>` where operand and result are identical types. |
| **`BiFunction<T, U, R>`** | `R apply(T t, U u)` | Maps two distinct inputs to an output result.                                    |

---

### 9. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can an abstract class have a constructor if it cannot be instantiated?**
  - _Answer:_ **Yes.** An abstract class can (and often does) have constructors. These constructors execute during the object initialization sequence when a concrete subclass is instantiated (`super()`), setting up base instance fields and enforcing base invariants.

- **Q2: What is the output of the following code?**

  ```java
  interface Alpha {
      int VAL = 10;
  }
  class Beta implements Alpha {
      public static void main(String[] args) {
          // Alpha.VAL = 20;
          System.out.println(VAL);
      }
  }
  ```

  - _Answer:_ `10`
  - _Explanation:_ Fields in interfaces are implicitly **`public static final`**. Un-commenting `Alpha.VAL = 20;` triggers a compile error (_cannot assign a value to final variable VAL_). The variable `VAL` is accessible directly inside `Beta` via inheritance.

- **Q3: Does an interface method that overrides a method from `java.lang.Object` count as an abstract method in a Functional Interface?**
  - _Answer:_ **No.** Declaring an abstract method matching a public method of `java.lang.Object` (such as `boolean equals(Object)` or `int hashCode()`) does **not** increase the abstract method count of a functional interface, because every implementing class is guaranteed to inherit an implementation from `java.lang.Object`.

- **Q4: Why can't a `default` method in an interface override `equals()`, `hashCode()`, or `toString()` from `Object`?**
  - _Answer:_ Java enforces the **"Classes Win Rule"** for method resolution. Any implementation from a superclass (including `java.lang.Object`) always takes precedence over an interface default method. Permitting default overrides of `Object` methods would lead to ambiguity and violate the fundamental object model.

- **Q5: Can an interface be declared `final`?**
  - _Answer:_ **No.** An interface is designed specifically to be implemented by classes and extended by other interfaces. Marking an interface `final` causes an immediate **compile-time error**: _illegal combination of modifiers: interface and final_.
