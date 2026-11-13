# Module 03: Object-Oriented Programming (OOP) Deep-Dive

## Unit 3.1: Inheritance

### 1. Introduction to Inheritance

**Inheritance** is a fundamental mechanism of Object-Oriented Programming by which one class derives state (instance variables) and behavior (methods) from another class.

It establishes an **"IS-A" hierarchy**, enabling:

- **Code Reusability:** Subclasses inherit tested implementations from parent classes without code duplication.
- **Polymorphic Substitutability:** Instances of derived classes can be passed to APIs expecting base class references (Liskov Substitution Principle).
- **Extensibility:** Specialized features can be appended to base architectures without modifying or recompiling original base source code.

```
                    Superclass (Base / Parent)
                   ┌───────────────────────────┐
                   │  - Base State             │
                   │  + Base Behavior          │
                   └─────────────▲─────────────┘
                                 │ extends (IS-A)
                   ┌─────────────┴─────────────┐
                   │  + Derived State          │
                   │  + Derived Behavior       │
                   └───────────────────────────┘
                     Subclass (Derived / Child)
```

---

### 2. Base Class vs. Derived Class Terminology

| Concept           | Alternative Names                     | Role & Definition                                                                                                   |
| :---------------- | :------------------------------------ | :------------------------------------------------------------------------------------------------------------------ |
| **Base Class**    | Superclass, Parent Class, Ancestor    | The general class whose properties and methods are inherited. Sits higher in the type hierarchy.                    |
| **Derived Class** | Subclass, Child Class, Extended Class | The specialized class that inherits from the base class. It adds specialized fields or overrides inherited methods. |

- In Java, a subclass inherits **all non-private members** (methods, fields, nested classes) of its superclass.
- **Constructors are NOT members:** Constructors are _never_ inherited by subclasses; they are invoked via chaining (`super(...)`).

---

### 3. Inheritance Taxonomies Supported in Java

```
1. Single Inheritance               2. Multilevel Inheritance           3. Hierarchical Inheritance
   ┌───────────┐                       ┌───────────┐                       ┌───────────┐
   │  Class A  │                       │  Class A  │                       │  Class A  │
   └─────▲─────┘                       └─────▲─────┘                       └──▲─────▲──┘
         │ extends                           │ extends                        │     │ extends
   ┌─────┴─────┐                       ┌─────┴─────┐                    ┌─────┴─┐ ┌─┴─────┐
   │  Class B  │                       │  Class B  │                    │Class B│ │Class C│
   └───────────┘                       └─────▲─────┘                    └───────┘ └───────┘
                                             │ extends
                                       ┌─────┴─────┐
                                       │  Class C  │
                                       └───────────┘
```

#### 3.1 Single Inheritance

A single subclass extends directly from exactly one superclass.

```java
class Vehicle {
    double maxSpeed;
}

class Car extends Vehicle {
    int numberOfDoors;
}
```

#### 3.2 Multilevel Inheritance

A class extends a subclass, forming an inheritance chain. `Class C` inherits members from both `Class B` and `Class A`.

```java
class Account {
    String accountNo;
}

class SavingsAccount extends Account {
    double interestRate;
}

class PremiumSavingsAccount extends SavingsAccount {
    double cashbackLimit;
}
```

#### 3.3 Hierarchical Inheritance

Multiple distinct subclasses extend directly from a single parent superclass.

```java
class Shape { void draw() {} }

class Circle extends Shape { double radius; }
class Rectangle extends Shape { double width, height; }
```

---

### 4. Multiple Inheritance Limitations in Java (The Diamond Problem)

Java **strictly prohibits multiple inheritance of classes** (a single class cannot use `extends` with more than one class).

```java
// COMPILE ERROR: Syntax error on token(s), delete these tokens
class Worker extends Human, RoboticUnit { ... } // ILLEGAL IN JAVA!
```

#### 4.1 The Diamond Problem Explained

Consider a scenario where multiple class inheritance is allowed:

```
                   Class A
               ┌─────────────┐
               │ void ping() │
               └──────▲──────┘
                      │
         ┌────────────┴────────────┐
         │ extends                 │ extends
   ┌─────┴───────┐           ┌─────┴───────┐
   │   Class B   │           │   Class C   │
   │ void ping() │           │ void ping() │ (Overrides ping)
   └─────▲───────┘           └─────▲───────┘
         │                         │
         └────────────┬────────────┘
                      │ extends BOTH (HYPOTHETICAL)
               ┌──────┴──────┐
               │   Class D   │
               └─────────────┘
```

If `Class D` does not override `ping()` and client code executes:

```java
D obj = new D();
obj.ping(); // AMBIGUITY: Does it invoke B's ping() or C's ping()?
```

#### 4.2 State Collision Hazards

Beyond method ambiguity, if `Class A` declares an instance field `int state`, `Class D` would inherit two independent copies of `int state` (one via `B`, one via `C`), creating structural ambiguity in object memory layouts. To eliminate this complexity, the designers of Java disallowed multiple class inheritance.

---

### 5. Hybrid Inheritance Through Interfaces

Java achieves multiple inheritance of **type and contract** while avoiding the diamond problem of state by using **Interfaces**.

A class can extend **one class** while simultaneously implementing **multiple interfaces**:

```java
interface Printable {
    void print();
}

interface Exportable {
    void exportToPDF();
}

class Document {
    String title;
}

// Single class extension, multiple interface implementation
public class Invoice extends Document implements Printable, Exportable {
    @Override
    public void print() {
        System.out.println("Printing invoice: " + title);
    }

    @Override
    public void exportToPDF() {
        System.out.println("Exporting invoice PDF");
    }
}
```

#### 5.1 Handling Java 8 Default Method Collisions

In Java 8+, interfaces can provide `default` method implementations. If a class implements two interfaces that share an identical default method signature, Java resolves ambiguity by forcing a **mandatory compile-time error**. The implementing class **must** explicitly override the colliding method:

```java
interface Alpha {
    default void execute() { System.out.println("Alpha"); }
}

interface Beta {
    default void execute() { System.out.println("Beta"); }
}

class Service implements Alpha, Beta {
    // COMPILER ERROR if not overridden:
    // class Service inherits unrelated defaults for execute() from types Alpha and Beta
    @Override
    public void execute() {
        // Disambiguate explicitly using Interface.super.method() syntax:
        Alpha.super.execute();
    }
}
```

---

### 6. The `extends` Keyword

The `extends` keyword establishes an inheritance relationship between classes or interfaces:

1. **Class-to-Class:** `class Sub extends Super` (Single inheritance only).
2. **Interface-to-Interface:** `interface ChildInterface extends ParentA, ParentB` (An interface **can extend multiple interfaces**!).

```java
interface A {}
interface B {}
interface C extends A, B { } // FULLY LEGAL! Interfaces support multiple inheritance!
```

---

### 7. The `super` Keyword in Inheritance

`super` is an implicit reference variable pointing to the immediate superclass component of the current object.

#### 7.1 Primary Use Cases:

1. **Invoking Superclass Constructors:** `super()` or `super(args...)` (Must be the first line).
2. **Accessing Hidden Superclass Fields:** `super.fieldName` resolves a field in the parent when shadowed by a field with the same name in the child.
3. **Invoking Overridden Superclass Methods:** `super.methodName()` bypasses dynamic virtual dispatch to call the superclass implementation.

```java
class Vehicle {
    int maxSpeed = 120;

    void describe() {
        System.out.println("Standard Vehicle");
    }
}

class SportsCar extends Vehicle {
    int maxSpeed = 300; // Shadows Vehicle.maxSpeed

    @Override
    void describe() {
        super.describe(); // Calls Vehicle.describe()
        System.out.println("SportsCar tuning");
    }

    void printSpeeds() {
        System.out.println("Super maxSpeed: " + super.maxSpeed); // 120
        System.out.println("This maxSpeed:  " + this.maxSpeed);  // 300
    }
}
```

---

### 8. Constructor Execution Lifecycle in Inheritance

Constructors **do not get inherited**. When a subclass object is instantiated, constructors are executed up the hierarchy from root to leaf via **Constructor Chaining**.

```
                   Constructor Execution Timeline
                                 │
     ┌───────────────────────────┴───────────────────────────┐
     │ 1. Child constructor invoked                          │
     │ 2. super() passes execution to Parent constructor      │
     │ 3. super() passes execution to Object() constructor   │
     │                                                       │
     │ 4. java.lang.Object.<init>() finishes                 │
     │ 5. Parent fields initialized & Parent body runs       │
     │ 6. Child fields initialized & Child body runs         │
     └───────────────────────────────────────────────────────┘
```

#### 8.1 The Compiler’s Silent `super()` Injection

If the first statement of a constructor is not an explicit `this(...)` or `super(...)`, the compiler injects an implicit **`super();`**:

```java
class GrandParent {
    GrandParent() { System.out.println("1. GrandParent"); }
}

class Parent extends GrandParent {
    Parent() {
        // super(); is injected here implicitly
        System.out.println("2. Parent");
    }
}

class Child extends Parent {
    Child() {
        // super(); is injected here implicitly
        System.out.println("3. Child");
    }

    public static void main(String[] args) {
        new Child();
        // Console Output:
        // 1. GrandParent
        // 2. Parent
        // 3. Child
    }
}
```

#### 8.2 The Missing No-Arg Constructor Trap (CDAC Classic)

If a base class defines a parameterized constructor and **does not** provide an explicit no-arg constructor, subclasses **fail to compile** unless they explicitly call the parameterized constructor:

```java
class Base {
    Base(int x) { } // Only parameterized constructor exists
}

class Derived extends Base {
    // COMPILE ERROR: There is no default constructor available in 'Base'
    Derived() {
        // Compiler injects super(); -> Fails because Base() does not exist!
    }

    // FIX: Provide explicit super(val)
    Derived(int x) {
        super(x); // Legal: matches Base(int x)
    }
}
```

---

### 9. Memory Layout of Inherited Objects on the Heap

When a subclass object is created, the JVM does **not** create two separate objects on the Heap. It allocates **one contiguous memory block** that holds the superclass fields alongside the subclass fields:

```java
class Employee {
    int empId;       // 4 bytes
    double salary;   // 8 bytes
}

class Developer extends Employee {
    int teamId;      // 4 bytes
}

Developer dev = new Developer();
```

```
STACK MEMORY (Frame)                               HEAP MEMORY (0x5500)
┌───────────────────────┐                  ┌───────────────────────────────────────────────┐
│ dev: 0x5500 ──────────┼─────────────────►│ Developer Object Instance                     │
└───────────────────────┘                  ├───────────────────────────────────────────────┤
                                           │ Mark Word (8 bytes)  - Lock/GC metadata       │
                                           │ Klass Word (4 bytes) - Class metadata pointer │
                                           ├───────────────────────────────────────────────┤
                                           │ Base Class Fields:                            │
                                           │ - empId: 0   (4 bytes)                        │
                                           │ - salary: 0.0 (8 bytes)                       │
                                           ├───────────────────────────────────────────────┤
                                           │ Subclass Fields:                              │
                                           │ - teamId: 0  (4 bytes)                        │
                                           │ Padding (8-byte alignment)                    │
                                           └───────────────────────────────────────────────┘
```

---

### 10. Method Inheritance & Member Accessibility Matrix

What a subclass can access from its parent is governed by access modifiers:

| Superclass Member Modifier    | Accessible to Child in Same Package? | Accessible to Child in Different Package? |            Inherited by Child?             |
| :---------------------------- | :----------------------------------: | :---------------------------------------: | :----------------------------------------: |
| **`public`**                  |                ✅ Yes                |                  ✅ Yes                   |                   ✅ Yes                   |
| **`protected`**               |                ✅ Yes                |       ✅ Yes (via inheritance only)       |                   ✅ Yes                   |
| **Default (Package-Private)** |                ✅ Yes                |                   ❌ No                   |         ✅ Only if in same package         |
| **`private`**                 |                ❌ No                 |                   ❌ No                   | ❌ No (Allocated in RAM, but inaccessible) |

> ⚠️ **Field Shadowing vs. Method Overriding:**
>
> - **Methods** are resolved dynamically at runtime using the **runtime object type** (Dynamic Dispatch via `invokevirtual`).
> - **Variables** are resolved at compile time based on the **declared reference type** (Static Binding). **Variables cannot be overridden; they are shadowed!**

```java
class Parent {
    int value = 10;
}

class Child extends Parent {
    int value = 20; // Shadows Parent.value
}

public class FieldTrap {
    public static void main(String[] args) {
        Parent ref = new Child();
        System.out.println(ref.value); // PRINTS 10! (Bound to reference type Parent)
    }
}
```

---

### 11. The "IS-A" Relationship Verification

The **IS-A** relationship represents structural inheritance:

- An object of a subclass is an instance of its parent class.
- Verified at compile time and runtime using the **`instanceof`** operator:

```java
Developer dev = new Developer();
System.out.println(dev instanceof Employee); // true  (Developer IS-A Employee)
System.out.println(dev instanceof Object);   // true  (Developer IS-A Object)
```

---

### 12. The `java.lang.Object` Root Class

Every class in Java is a direct or indirect descendant of **`java.lang.Object`**. If a class does not specify `extends`, `javac` automatically attaches `extends Object`.

```
                    java.lang.Object (Root of all types)
                                     ▲
                    ┌────────────────┴────────────────┐
                    │                                 │
                 Vehicle                           String
                    ▲
             ┌──────┴──────┐
            Car          Truck
```

#### The 11 Core Methods of `java.lang.Object`:

1. `public boolean equals(Object obj)`: Compares memory addresses (`this == obj`) by default.
2. `public int hashCode()`: Generates an internal memory-based hash value.
3. `public String toString()`: Returns `getClass().getName() + "@" + Integer.toHexString(hashCode())`.
4. `protected Object clone() throws CloneNotSupportedException`: Performs a field-by-field shallow copy.
5. `public final Class<?> getClass()`: Returns the runtime Metaspace reflection descriptor.
6. `protected void finalize() throws Throwable`: (Deprecated in Java 9, removed in newer versions) GC reclamation hook.
7. `public final void wait()`: Puts the current thread to sleep until notified; releases the monitor lock.
8. `public final void wait(long timeoutMillis)`: Timed monitor wait.
9. `public final void wait(long timeoutMillis, int nanos)`: Precision timed monitor wait.
10. `public final void notify()`: Wakes up a single arbitrary thread waiting on this object's monitor.
11. `public final void notifyAll()`: Wakes up all threads waiting on this object's monitor.

---

### 13. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can a class extend its own self? (e.g., `class A extends A`?)**
  - _Answer:_ **No.** Causes a compile-time error: _Cyclic inheritance involving 'A'_.

- **Q2: Does a subclass inherit the private members of its superclass?**
  - _Answer:_ **They are allocated in memory, but they are not directly accessible.** The memory for private fields is physically allocated inside the subclass object on the Heap, but those fields cannot be accessed directly by name within the subclass code. They can only be accessed or mutated through inherited public/protected getter and setter methods.

- **Q3: What happens if `super()` is placed on the second line of a constructor?**
  - _Answer:_ **Compilation Error:** _Constructor call must be the first statement in a constructor_.

- **Q4: Why does an interface allow multiple inheritance (`interface C extends A, B`), but a class does not?**
  - _Answer:_ Interfaces do not maintain instance state (no instance fields). Therefore, inheriting from multiple interfaces presents **no state collision hazard**. Even with default methods, Java forces the implementing class to resolve method collisions explicitly at compile time, eliminating ambiguity.

- **Q5: Can we use `super.super.method()` to call a grand-parent's method?**
  - _Answer:_ **No.** Java strictly prohibits bypassing the immediate parent (`super.super` is a syntax error). Allowing it would break encapsulation by letting a subclass ignore its immediate parent's overridden behavior.
