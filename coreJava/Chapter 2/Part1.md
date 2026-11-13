# Module 02: Object-Oriented Programming (OOP) in Java

## Unit 2.1: OOP Fundamentals

### 1. Introduction to Object-Oriented Programming

**Object-Oriented Programming (OOP)** is a design paradigm organized around **data (objects)** rather than functions or logic.

In traditional procedural models (like C), applications are treated as a sequence of procedural instructions operating on passive, decoupled data structures. OOP bridges this divide by binding **state (data fields)** and **behavior (methods)** into a single self-contained entity called an **Object**.

```
Procedural Model (Decoupled):
[ Raw Global / Struct Data ] <==== (Free Functions mutate data directly)

OOP Model (Encapsulated):
┌─────────────────────────────────────────┐
│                 OBJECT                  │
│  ┌───────────────────────────────────┐  │
│  │ State: Private Variables/Fields   │  │
│  └─────────────────▲─────────────────┘  │
│                    │ Controlled Access  │
│  ┌─────────────────┴─────────────────┐  │
│  │ Behavior: Public Methods/APIs     │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

---

### 2. Classes and Objects

- **Class:** A user-defined, blueprint or template from which individual objects are instantiated. It defines the structure, variables, and methods common to all instances. It exists in the **Metaspace** as class metadata.
- **Object:** An identifiable, concrete runtime instance of a class occupying physical memory on the **Heap**. It has:
  1. **State (Properties/Attributes):** Stored inside instance fields.
  2. **Behavior (Actions):** Defined by methods invoked on it.
  3. **Identity:** A distinct location/address in memory (even if states are completely identical).

```
   Class Template (Metaspace): Class Car { String vin; double speed; void drive(); }
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         │ (new Car())                │ (new Car())                │ (new Car())
         ▼                            ▼                            ▼
Heap: Instance 0x1000        Heap: Instance 0x2000        Heap: Instance 0x3000
      vin: "MH12-9901"             vin: "DL04-1122"             vin: "KA01-7788"
      speed: 80.0                  speed: 0.0                   speed: 120.5
```

---

### 3. Class Declaration

A class declaration establishes the class name, access modifiers, type parameters, extended superclasses, implemented interfaces, and the class body:

```java
// Formal Syntax:
[access_modifier] class ClassName [extends SuperClass] [implements Interface1, Interface2] {
    // 1. Static variables (Class state)
    // 2. Instance variables (Object state)
    // 3. Static initializers & Instance initializers
    // 4. Constructors (<init>)
    // 5. Methods (Instance & Static behavior)
    // 6. Nested classes/interfaces
}
```

```java
package com.cdac.banking;

public class BankAccount {
    // Instance Variables (Encapsulated State)
    private final String accountNumber;
    private double balance;

    // Constructor (State Initialization)
    public BankAccount(String accNo, double initialBalance) {
        this.accountNumber = accNo;
        this.balance = initialBalance;
    }

    // Instance Method (Behavior)
    public void deposit(double amount) {
        if (amount > 0) {
            this.balance += amount;
        }
    }

    public double getBalance() {
        return this.balance;
    }
}
```

---

### 4. Object Creation & Memory Allocation

Objects are instantiated at runtime dynamically using the **`new`** keyword.

```java
BankAccount acc = new BankAccount("SB-1002", 5000.0);
```

#### Step-by-Step Execution Sequence of `new`:

1. **Class Loading:** Checks if `BankAccount` is loaded in Metaspace. If not, the ClassLoader loads and initializes it.
2. **Memory Allocation:** Calculates the total byte size of the object (Header + Instance Variables + Alignment Padding) and allocates memory on the **Heap**.
3. **Zero Initialization:** The allocated heap bytes are zero-filled (primitives become `0`/`false`, reference pointers become `null`).
4. **Header Construction:** Sets up the **Mark Word** (GC age, hash code, monitor lock bits) and **Klass Word** (pointer to Metaspace type info).
5. **Constructor Execution (`<init>`):** Invokes the target constructor, running superclass constructors and instance initializer blocks (IIBs).
6. **Reference Assignment:** Returns the 32-bit or 64-bit heap address, which is assigned to the stack variable `acc`.

```
STACK FRAME (main)                               HEAP MEMORY (Runtime Space)
┌────────────────────────────┐                  ┌──────────────────────────────────────────────┐
│ BankAccount acc            │                  │ BankAccount Object (0x8A12F000)              │
│ Address: 0x8A12F000        │ ───────────────► ├──────────────────────────────────────────────┤
└────────────────────────────┘                  │ Mark Word (8 bytes): GC/Lock State           │
                                                │ Klass Word (4 bytes): Pointer to Metaspace   │
                                                ├──────────────────────────────────────────────┤
                                                │ accountNumber: 0x9920 ──► [String: "SB-1002"]│
                                                │ balance: 5000.0 (8 bytes, IEEE 754 double)   │
                                                │ Padding: (4 bytes to align to 8-byte boundary)│
                                                └──────────────────────────────────────────────┘
```

---

### 5. Instance Variables

Instance variables are declared inside a class but outside any method, constructor, or block.

- **Lifecycle:** Born when the object is instantiated via `new`; destroyed when the object is dereferenced and collected by the Garbage Collector.
- **Storage:** Always on the **Heap** inside the object body.
- **Default Values:** Automatically initialized to default values (`0`, `0.0`, `false`, `null`) if not explicitly assigned.
- **Access Modifiers:** Can use any of the four access levels (`public`, `protected`, default/package-private, `private`).

---

### 6. Instance Methods & Bytecode Invocation

Instance methods model the behaviors and operations of an object. They have access to the implicit reference variable **`this`**, which points to the current calling instance.

#### 6.1 Invocation Bytecode: `invokevirtual` vs `invokestatic`

- **`invokevirtual`:** The instruction generated for regular non-static instance methods. It performs dynamic dispatch at runtime by dereferencing the object's `this` pointer and traversing the **vtable** (Virtual Method Table).
- **`invokestatic`:** Used for static methods. It invokes the method directly via static binding, requiring no instance or `this` pointer.

```java
BankAccount acc = new BankAccount("SB-101", 100.0);
acc.deposit(50.0); // Compiled to: invokevirtual #4 (Method deposit:(D)V)
```

---

### 7. The Four Pillars of OOP

```
                               The 4 Pillars of OOP
                                        │
         ┌──────────────────┬───────────┴───────────┬──────────────────┐
         ▼                  ▼                       ▼                  ▼
   Encapsulation       Abstraction             Inheritance        Polymorphism
   (Data Hiding)    (Interface Hiding)      (Code Reusability)   (Dynamic Form)
```

#### 7.1 Encapsulation (Data Hiding & Protection)

- **Concept:** Bundling data (variables) and the methods that mutate them into a single class unit, while restricting direct external modification.
- **Mechanism:**
  1. Declare all fields as **`private`**.
  2. Provide controlled public inspection (**getter**) and mutation (**setter**) methods with built-in validation.
- **Benefits:** Protects object invariants, reduces coupling, and allows internal data structures to change without breaking client code.

```java
public class Employee {
    private double salary; // Encapsulated: direct access blocked

    public void setSalary(double salary) {
        if (salary < 0.0) {
            throw new IllegalArgumentException("Salary cannot be negative");
        }
        this.salary = salary;
    }

    public double getSalary() {
        return this.salary;
    }
}
```

#### 7.2 Abstraction (Hiding Complexity)

- **Concept:** Exposing only essential operational contracts to the user while concealing internal execution mechanics and algorithms.
- **Mechanism:** Implemented through **Abstract Classes** (`abstract class`) and **Interfaces** (`interface`).
- **Real-World Analogy:** A driver interacts with an accelerator pedal and steering wheel (the interface) without needing to monitor fuel injectors or spark-plug timing (the implementation).

```java
// Abstraction contract
public interface PaymentGateway {
    boolean processPayment(double amount);
}

// Concrete hidden implementation
public class UPIPaymentGateway implements PaymentGateway {
    @Override
    public boolean processPayment(double amount) {
        // Complex internal cryptographic handshake, bank routing, network retry
        return true;
    }
}
```

#### 7.3 Inheritance (Code Reuse & Hierarchies)

- **Concept:** A mechanism where a derived sub-type (subclass) inherits state and behaviors from an existing super-type (superclass).
- **Keywords:** `extends` (for classes), `implements` (for interfaces).
- **Relationship:** Represents the **"IS-A"** relationship (e.g., `SavingsAccount` _IS-A_ `BankAccount`).
- **Restriction:** Java supports **single inheritance of state** (a class can extend at most one superclass) to prevent the Diamond Problem.

```java
public class SavingsAccount extends BankAccount {
    private double interestRate;

    public SavingsAccount(String accNo, double bal, double rate) {
        super(accNo, bal); // Chain up to superclass constructor
        this.interestRate = rate;
    }
}
```

#### 7.4 Polymorphism (Many Forms)

- **Concept:** The ability of a single interface or reference type to execute distinct underlying behaviors based on the concrete runtime object instance.
- **Forms:**
  1. **Compile-Time Polymorphism (Static Binding):** Method Overloading (same method name, different parameter lists within the same scope).
  2. **Runtime Polymorphism (Dynamic Binding):** Method Overriding (subclass redefines a superclass method with identical signature; resolved at runtime via `invokevirtual`).

```java
// Polymorphic call site:
BankAccount acc1 = new BankAccount("A1", 1000);
BankAccount acc2 = new SavingsAccount("S2", 2000, 4.5);

// The exact method executed depends on the runtime heap instance:
acc1.deposit(100); // Calls BankAccount::deposit
acc2.deposit(100); // Calls SavingsAccount::deposit (if overridden)
```

---

### 8. Procedural Programming vs. Object-Oriented Programming

| Dimension                | Procedural Programming (e.g., C)                           | Object-Oriented Programming (e.g., Java)                     |
| :----------------------- | :--------------------------------------------------------- | :----------------------------------------------------------- |
| **Primary Focus**        | Step-by-step algorithms, tasks, and functions              | Data structures, models, and objects                         |
| **Program Organization** | Modular functions and shared global/local structs          | Classes, hierarchies, and encapsulated objects               |
| **Data Security**        | Low; data moves freely and can be modified by any function | High; data is encapsulated and protected by access modifiers |
| **Execution Approach**   | Top-down structured approach                               | Bottom-up architectural modeling                             |
| **Code Reusability**     | Limited (relies on manual function libraries)              | High (leveraged via Inheritance and Polymorphism)            |
| **Polymorphism**         | None (requires function pointers or manual switch tables)  | Built-in (Method Overloading and Dynamic Overriding)         |
| **Real-World Modeling**  | Difficult to map complex real-world entities               | Maps directly to real-world domain concepts                  |

---

### 9. Real-World Applications of OOP

1. **Banking & Financial Engines:** Modeling accounts, transaction lifecycles, and authorization workflows where strict data encapsulation prevents state corruption.
2. **Enterprise Resource Planning (ERP) & CRM:** Mapping organizational entities (Customers, Invoices, Departments) using hierarchical inheritance and polymorphically dispatched taxation rules.
3. **Game Development Systems:** Representing players, enemies, and interactive objects on a game canvas inheriting from a base `GameObject` with polymorphic `render()` and `update()` ticks.
4. **GUI Frameworks & Web Servers:** Event-driven windowing toolkits (JavaFX/Swing) and servlet engines (Spring Boot/Tomcat) where abstractions manage request-response pipelines.

---

### 10. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can an object exist without its class being loaded into the JVM?**
  - _Answer:_ **No.** An object cannot be instantiated until its class bytecode is loaded, linked, and initialized in the Metaspace. The object header on the Heap holds a mandatory **Klass Word** pointer back to this Metaspace class metadata.

- **Q2: What is the primary difference between Encapsulation and Abstraction?**
  - _Answer:_
    - **Encapsulation** is about **data protection** (packaging data and methods together and hiding internal state via access modifiers like `private`). It focuses on _how_ to protect internal state.
    - **Abstraction** is about **interface simplicity** (hiding operational complexity and exposing only relevant contracts via abstract classes or interfaces). It focuses on _what_ an object does rather than _how_ it does it.

- **Q3: What will be the output of comparing two newly instantiated objects using `==`?**

  ```java
  BankAccount b1 = new BankAccount("101", 500);
  BankAccount b2 = new BankAccount("101", 500);
  System.out.println(b1 == b2);
  ```

  - _Answer:_ `false`
  - _Explanation:_ The `==` operator performs **reference comparison** on objects. Because `new` allocates two distinct objects at different heap addresses, `b1` and `b2` store different references.

- **Q4: Is Java a pure object-oriented language?**
  - _Answer:_ **No.** Java is not considered purely object-oriented because it retains the **8 primitive data types** (`int`, `boolean`, `char`, etc.) for performance reasons, which do not inherit from `java.lang.Object` and have no methods or identity headers.

- **Q5: Does an empty class (e.g., `class Empty {}`) occupy memory on the Heap when instantiated?**
  - _Answer:_ **Yes.** Every object instance in HotSpot JVM requires at least a **12-byte or 16-byte object header** (Mark Word + Klass Word), padded up to the nearest multiple of 8 bytes (yielding a minimum of 16 bytes on modern 64-bit JVMs with compressed OOPs enabled).
