# Module 03: Object-Oriented Programming (OOP) Deep-Dive

## Unit 3.2: Polymorphism and Dynamic Method Dispatch

### 1. Introduction to Polymorphism

**Polymorphism** (derived from Greek: _poly_ = many, _morph_ = form) is the capability of a single programming entity—such as a method identifier, an operator, or a reference handle—to exhibit different behaviors based on the invocation context or the underlying runtime object instance.

```
                           Polymorphism in Java
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
Compile-Time Polymorphism                               Runtime Polymorphism
(Static Polymorphism / Early Binding)                   (Dynamic Polymorphism / Late Binding)
├── Method Overloading                                  └── Method Overriding
├── Resolved at compile time by javac                   └── Resolved at runtime by the JVM
└── Bytecode: invokestatic / invokespecial              └── Bytecode: invokevirtual / invokeinterface
```

---

### 2. Compile-Time vs. Runtime Polymorphism: Architectural Contrast

| Dimension                 | Compile-Time Polymorphism                                 | Runtime Polymorphism                                           |
| :------------------------ | :-------------------------------------------------------- | :------------------------------------------------------------- |
| **Mechanism**             | **Method Overloading**                                    | **Method Overriding**                                          |
| **Resolution Phase**      | Static compile-time by `javac`                            | Dynamic runtime by JVM execution engine                        |
| **Binding Mechanism**     | **Early Binding** (Static Binding)                        | **Late Binding** (Dynamic Binding)                             |
| **Performance Overhead**  | **Zero runtime overhead** (direct address call)           | **Slight indirection overhead** (VTABLE lookup)                |
| **Bytecode Instructions** | `invokestatic`, `invokespecial`                           | `invokevirtual`, `invokeinterface`                             |
| **Scope Requirement**     | Can occur within the **same class** or across inheritance | Requires an **inheritance hierarchy** (`extends`/`implements`) |
| **Method Signature**      | Must have **different argument lists**                    | Must have **identical method signatures**                      |

---

### 3. Method Overloading (Compile-Time Polymorphism)

**Method Overloading** occurs when multiple methods within the same class (or inherited into the same scope) share the **exact same method identifier** but differ in their **parameter lists**.

#### 3.1 The Three Mechanisms of Overload Differentiation

1. **Variation in Argument Count:** `compute(int)` vs. `compute(int, int)`
2. **Variation in Argument Types:** `render(int)` vs. `render(String)`
3. **Variation in Argument Sequence:** `map(int, double)` vs. `map(double, int)`

```java
public class Printer {
    public void print(int value)       { System.out.println("Integer: " + value); }
    public void print(String message)  { System.out.println("String: " + message); }
    public void print(int a, double b) { System.out.println("int, double: " + a + ", " + b); }
    public void print(double a, int b) { System.out.println("double, int: " + a + ", " + b); }
}
```

#### 3.2 The Method Signature Invariant

In Java, a **Method Signature** consists strictly of:
$$\text{Method Signature} = \langle \text{Method Identifier}, \text{Parameter Type List} \rangle$$

> ⚠️ **The Return-Type Overloading Fallacy:**  
> Changing **only the return type** or **access modifiers** or the `throws` clause **DOES NOT** constitute a valid overload. The compiler raises a duplicate method error.

```java
public class DuplicateTrap {
    public int calculate(int a)   { return a * 2; }
    // COMPILE ERROR: method calculate(int) is already defined in class DuplicateTrap
    public double calculate(int a) { return a * 2.0; }
}
```

_Why?_ When client code invokes `calculate(10);` as an expression statement ignoring the return value, the compiler cannot deduce which method to bind.

#### 3.3 Overload Resolution Priority Order (CDAC High-Yield)

When multiple overloaded methods can match an argument, `javac` applies a strict 3-phase priority ladder (JLS §15.12.2):

$$\text{Phase 1: Primitive Widening} \longrightarrow \text{Phase 2: Autoboxing} \longrightarrow \text{Phase 3: Varargs}$$

```
                Overload Resolution Priority Ladder
                           Input: int (10)
                                  │
                                  ▼
             ┌──────────────────────────────────────────┐
             │ 1. Exact Match: test(int)                │
             └────────────────────┬─────────────────────┘
                                  │ (if absent)
                                  ▼
             ┌──────────────────────────────────────────┐
             │ 2. Primitive Widening: test(long)        │
             └────────────────────┬─────────────────────┘
                                  │ (if absent)
                                  ▼
             ┌──────────────────────────────────────────┐
             │ 3. Autoboxing: test(Integer)             │
             └────────────────────┬─────────────────────┘
                                  │ (if absent)
                                  ▼
             ┌──────────────────────────────────────────┐
             │ 4. Widening to Reference: test(Object)   │
             └────────────────────┬─────────────────────┘
                                  │ (if absent)
                                  ▼
             ┌──────────────────────────────────────────┐
             │ 5. Varargs (Lowest): test(int...)        │
             └──────────────────────────────────────────┘
```

```java
public class OverloadPriorityDemo {
    static void execute(long x)    { System.out.println("1. Widening to long"); }
    static void execute(Integer x) { System.out.println("2. Autoboxing to Integer"); }
    static void execute(Object x)  { System.out.println("3. Widening to Object"); }
    static void execute(int... x)  { System.out.println("4. Varargs"); }

    public static void main(String[] args) {
        int val = 10;
        execute(val); // Prints: "1. Widening to long"
    }
}
```

> ⚠️ **The Illegal Transition: Widening + Autoboxing**  
> `javac` **CANNOT widen a primitive and then autobox it** in a single conversion step!  
> An `int` can widen to `long`, or autobox to `Integer`. An `int` **CANNOT** autobox directly to `Long` because `Integer` and `Long` are sibling types with no inheritance relationship.

---

### 4. Method Overriding (Runtime Polymorphism)

**Method Overriding** occurs when a subclass defines a specific implementation of an instance method that is already declared in one of its superclasses.

```java
class PaymentService {
    public void process(double amount) {
        System.out.println("Base payment processing: $" + amount);
    }
}

class UPIPaymentService extends PaymentService {
    @Override // Optional, but highly recommended for compile-time safety
    public void process(double amount) {
        System.out.println("UPI instant payment processing: $" + amount);
    }
}
```

#### 4.1 The 6 Fundamental Rules of Method Overriding

To constitute a valid override, the subclass method must satisfy:

1. **Identical Signature:** Method name, parameter counts, and parameter types must match exactly.
2. **Return Type Compatibility:** Must be identical or a **Covariant Return Type** (a subtype of the return type declared in the superclass).
3. **Access Privilege Rule:** The overriding method **CANNOT be more restrictive** than the superclass method. It can be equally or more permissive:
   $$\text{private} \longrightarrow \text{package-private (default)} \longrightarrow \text{protected} \longrightarrow \text{public}$$
   _(Example: Overriding a `protected` method as `public` is valid; overriding a `protected` method as `private` fails compilation)._
4. **Exception Handling Rule:** An overriding method:
   - **Cannot declare new or broader checked exceptions** than the superclass method.
   - Can declare **fewer**, **narrower (subclass)** checked exceptions, or **omit** them entirely.
   - Can throw any unchecked exceptions (`RuntimeException` or `Error`) without restriction.
5. **Private Methods Cannot Be Overridden:** `private` methods are bound statically via `invokespecial` and are invisible to subclasses. Declaring an identically named method in a subclass simply creates a new, independent method.
6. **Final & Static Methods Cannot Be Overridden:**
   - `final` methods prohibit overriding by definition.
   - `static` methods undergo **Method Hiding**, not dynamic overriding.

---

### 5. Static Binding vs. Dynamic Binding

```
                      Binding in Java
                             │
     ┌───────────────────────┴───────────────────────┐
     ▼                                               ▼
Static Binding (Early Binding)                 Dynamic Binding (Late Binding)
- Resolved at compile time by javac            - Resolved at runtime by the JVM
- Driven by declared Reference Type            - Driven by actual Object Type on Heap
- Applicable to:                               - Applicable to:
  • private methods                              • public, protected, package instance
  • static methods                                 methods (virtual methods)
  • final methods                                - Dispatched via invokevirtual
  • Constructors
  • Field accesses (all variables)
```

```java
class SuperClass {
    int data = 100; // Variables are statically bound
    public void render() { System.out.println("Super render"); }
}

class SubClass extends SuperClass {
    int data = 200; // Shadows SuperClass.data
    @Override
    public void render() { System.out.println("Sub render"); }
}

public class BindingDemo {
    public static void main(String[] args) {
        SuperClass ref = new SubClass();
        System.out.println(ref.data); // 100  <-- Static Binding: checks declared type (SuperClass)
        ref.render();                 // Sub render <-- Dynamic Binding: checks Heap instance (SubClass)
    }
}
```

---

### 6. Dynamic Method Dispatch & The Virtual Method Table (VTABLE)

**Dynamic Method Dispatch** is the JVM execution runtime mechanism that resolves calls to overridden instance methods at runtime based on the actual object instance on the Heap.

#### 6.1 Bytecode Perspective: `invokevirtual`

When Java source code invokes `ref.render()`, `javac` emits:

```text
invokevirtual #4 // Method SuperClass.render:()V
```

`invokevirtual` does not contain a hardcoded memory offset to machine instructions. It instructs the JVM to inspect the target object's header at runtime.

#### 6.2 The Virtual Method Table (VTABLE)

Every class loaded into Metaspace containing virtual methods is assigned a **VTABLE (Virtual Method Table)**. The VTABLE is an indexed array of direct function pointers pointing to the executable bytecode of methods available to that class.

```
METASPACE MEMORY
┌──────────────────────────────────────────────┐
│ VTABLE: SuperClass                           │
├──────────────────────────────────────────────┤
│ Slot 0: &SuperClass::render                  │
│ Slot 1: &Object::toString                    │
│ Slot 2: &Object::hashCode                    │
└──────────────────────────────────────────────┘

┌──────────────────────────────────────────────┐
│ VTABLE: SubClass                             │
├──────────────────────────────────────────────┤
│ Slot 0: &SubClass::render    <───────────────┼── Override overwrites base pointer!
│ Slot 1: &Object::toString                    │
│ Slot 2: &Object::hashCode                    │
└──────────────────────────────────────────────┘
```

#### Step-by-Step Dispatch Walkthrough:

1. The JVM fetches the object reference from the operand stack (`ref`).
2. It reads the object's **Klass Word** in the object header on the Heap to locate the class metadata in Metaspace.
3. It accesses the class's **VTABLE**.
4. It reads the direct memory address stored at the method's predefined slot offset (`Slot 0`).
5. Execution branches to `SubClass::render`.

---

### 7. Upcasting vs. Downcasting

```
                  Inheritance Hierarchy Cast Tree
                            ┌────────────┐
                            │   Animal   │
                            └─────▲──────┘
                                  │
         ▲ Upcasting              │  ▼ Downcasting
         │ (Always Safe)          │  │ (Explicit / Risky)
         │ Implicit               │  │ Explicit (Dog)
                                  │
                            ┌─────┴──────┐
                            │    Dog     │
                            └────────────┘
```

#### 7.1 Upcasting (Subclass Reference $\rightarrow$ Superclass Reference)

- Assigning a subclass instance to a superclass reference variable.
- **Implicit & Guaranteed Safe:** Since every `Dog` is an `Animal`, upcasting requires no explicit cast syntax.
- **Access Limitation:** Once upcast, the reference variable can **only** access members declared in the superclass interface (though overridden methods still dispatch dynamically to the subclass implementation).

```java
Animal a = new Dog(); // Upcasting (Implicit, safe)
a.eat();              // Valid: declared in Animal
// a.bark();          // COMPILE ERROR: Animal has no bark() method!
```

#### 7.2 Downcasting (Superclass Reference $\rightarrow$ Subclass Reference)

- Converting a superclass reference back down to a more specialized subclass type.
- **Explicit & Risky:** Requires explicit casting syntax: `(TargetSubclass)`.
- **Runtime Verification:** If the heap object is **not** an instance of the target subclass, the JVM throws **`java.lang.ClassCastException`**.

```java
Animal a1 = new Dog();
Dog d1 = (Dog) a1; // Downcasting: OK, heap object is genuinely a Dog
d1.bark();         // Valid!

Animal a2 = new Animal();
Dog d2 = (Dog) a2; // RUNTIME ERROR: java.lang.ClassCastException!
                   // The object on the heap is Animal, NOT Dog!
```

---

### 8. Defensive Type-Checking: The `instanceof` Operator

To prevent `ClassCastException`, downcasting must always be preceded by an **`instanceof`** inspection:

```java
public void processAnimal(Animal a) {
    // 1. Classical instanceof + Explicit Downcast
    if (a instanceof Dog) {
        Dog d = (Dog) a;
        d.bark();
    }

    // 2. Modern Pattern Matching for instanceof (Java 16+)
    if (a instanceof Dog d) {
        d.bark(); // 'd' is automatically downcast and scoped in-place!
    }
}
```

---

### 9. Covariant Return Types (Java 5+)

Historically (Java 1.4 and earlier), an overriding method was required to match the superclass method's return type **identically**.  
**Covariant Return Types** allow an overriding method in a subclass to return a **subtype (narrower reference type)** of the return type declared in the superclass.

```java
class Asset {}
class RealEstate extends Asset {}

class Investor {
    public Asset acquire() {
        return new Asset();
    }
}

class PropertyInvestor extends Investor {
    @Override
    public RealEstate acquire() { // RealEstate IS-A Asset (Covariant Return Type)
        return new RealEstate();
    }
}
```

#### 9.1 Advantages of Covariant Returns

Eliminates tedious downcasting on the caller side when working with specialized factories:

```java
PropertyInvestor investor = new PropertyInvestor();
// Without covariant returns, caller had to write: (RealEstate) investor.acquire();
RealEstate house = investor.acquire(); // Direct, type-safe assignment!
```

#### 9.2 Bytecode Mechanics: Bridge Methods

At the JVM bytecode level, method identity includes the return type descriptor. To support covariant returns without breaking backward compatibility, `javac` automatically synthesizes a **Bridge Method**:

```text
// Decompiled synthetic bridge method in PropertyInvestor.class:
public Asset acquire();
  Code:
     0: aload_0
     1: invokevirtual #3 // Method acquire:()LRealEstate;
     4: areturn
```

---

### 10. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can we overload a method by changing only the return type?**
  - _Answer:_ **No.** The method signature consists only of the name and the parameter type list. If two methods in the same class have identical names and arguments, changing the return type results in a compile-time error: _method is already defined_.

- **Q2: What is the output of the following code?**

  ```java
  class Base {
      public static void display() { System.out.println("Base Static"); }
      public void show()            { System.out.println("Base Virtual"); }
  }
  class Derived extends Base {
      public static void display() { System.out.println("Derived Static"); }
      public void show()            { System.out.println("Derived Virtual"); }
  }
  public class Test {
      public static void main(String[] args) {
          Base b = new Derived();
          b.display();
          b.show();
      }
  }
  ```

  - _Answer:_
    ```
    Base Static
    Derived Virtual
    ```
  - _Explanation:_
    - `b.display()` is a call to a `static` method. Static methods do not undergo dynamic dispatch; they are **statically bound** at compile time based on the reference type (`Base`), which prints `"Base Static"` (Method Hiding).
    - `b.show()` is a non-static virtual method. It undergoes **Dynamic Method Dispatch** via `invokevirtual`, resolving to the actual runtime heap instance (`Derived`), which prints `"Derived Virtual"`.

- **Q3: What happens when calling `test(null)` with these overloads?**

  ```java
  class AmbiguityDemo {
      static void test(String s) { System.out.println("String"); }
      static void test(Integer i) { System.out.println("Integer"); }
      public static void main(String[] args) {
          test(null);
      }
  }
  ```

  - _Answer:_ **Compilation Error:** _reference to test is ambiguous_.
  - _Explanation:_ Both `String` and `Integer` are reference types that can accept `null`. Because neither `String` nor `Integer` is a subtype of the other, the compiler cannot determine which one is "more specific" and throws an ambiguity compilation error. (If one overload took `Object` and the other took `String`, `test(null)` would cleanly choose `String` because `String` is more specific than `Object`).

- **Q4: Can an overriding method declare broader checked exceptions than the superclass method?**
  - _Answer:_ **No.** A subclass cannot expand the checked exception contract of a superclass method. If the base method throws `IOException`, the derived method can throw `FileNotFoundException` (a subclass of `IOException`) or no exception at all, but it cannot throw `Exception` or `Throwable`.

- **Q5: Can you downcast an object without an explicit cast operator?**
  - _Answer:_ **No.** Downcasting moves down the hierarchy from a general type to a specific type, which introduces type-safety risks. The compiler mandates an explicit cast operator: `(TargetType)`.
