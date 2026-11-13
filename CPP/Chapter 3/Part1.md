# Module 03: Object-Oriented Programming (Core Concepts)

## Unit 3.1: Object-Oriented Programming Fundamentals

### 1. Introduction to Object-Oriented Programming (OOP)

**Object-Oriented Programming (OOP)** is a programming paradigm organized around **data (objects)** and **behaviors (methods)** rather than functions and sequential logic.

In traditional procedural programming, data and procedures are separate entities—data is passed into functions, often leaving it vulnerable to unintended external modifications. OOP re-architects software systems by binding data and functions into cohesive units known as **objects**.

```
Procedural Model:                    Object-Oriented Model:
+---------------------------+        +-----------------------------------+
|        Global Data        |        |               Object              |
+---------------------------+        |  +-------------+  +------------+  |
       ^             ^               |  | Member Data |  | Functions  |  |
       |             |               |  | (State)     |<-| (Behavior) |  |
+-------------+ +-------------+      |  +-------------+  +------------+  |
| Function A  | | Function B  |      +-----------------------------------+
+-------------+ +-------------+              ^                     ^
(Data moves freely; low safety)              | Inter-object        | Message
                                             v Communication       v Passing
```

#### The Four Fundamental Pillars of OOP:

1. **Encapsulation:** Packaging data and methods into a single unit while restricting direct access.
2. **Abstraction:** Exposing only relevant interfaces to the user while hiding internal implementation complexities.
3. **Inheritance:** Creating new classes based on existing ones to foster code reusability and hierarchical relationships.
4. **Polymorphism:** The ability of entities to take on different forms or behaviors depending on invocation context.

---

### 2. Classes and Objects

#### 2.1 The Class: A Blueprint

A **class** is a user-defined data type and template that specifies the attributes (state) and member functions (behavior) that objects instantiated from it will possess.

- Allocates **no memory** for member data when declared (except for static members in the data segment).
- Serves purely as a compile-time schematic.

#### 2.2 The Object: An Instance of a Class

An **object** is a concrete instance of a class occupying physical memory in the Stack, Heap, or Data segment.

- Holds its own independent copy of non-static member variables.
- Shares a single, common copy of class member function machine code in the Code/Text segment with all other instances of that class.

```cpp
#include <iostream>

// Class Definition (Blueprint - No memory allocated for variables yet)
class BankAccount {
private:
    std::string accountNumber; // Attribute / State
    double balance;

public:
    // Member function / Behavior
    void initialize(std::string accNo, double initialBal) {
        accountNumber = accNo;
        balance = initialBal;
    }

    void deposit(double amount) {
        if (amount > 0) balance += amount;
    }

    void display() const {
        std::cout << "Acc: " << accountNumber << " | Balance: $" << balance << "\n";
    }
};

int main() {
    // Instantiation: Memory allocated on the stack for each object
    BankAccount acc1;
    BankAccount acc2;

    acc1.initialize("SB-101", 1500.0);
    acc2.initialize("SB-102", 2800.0);

    acc1.deposit(500.0);

    acc1.display(); // Acc: SB-101 | Balance: $2000
    acc2.display(); // Acc: SB-102 | Balance: $2800

    return 0;
}
```

#### 2.3 Memory Footprint of Classes & Objects

##### Empty Class Size Rule (High-Yield CMCE MCQ):

```cpp
class Empty {};
std::cout << sizeof(Empty); // PRINTS: 1 (in C++)
```

- **Why is an empty class not 0 bytes?**
  To guarantee that two distinct object instances have **unique memory addresses**. If an empty class had a size of 0 bytes, multiple instantiated objects would reside at the exact same memory address, breaking pointer arithmetic and object identity guarantees.
- _Note:_ In pure C, an empty `struct` has an undefined or compiler-dependent size (GCC treats an empty C struct as 0 bytes, but in C++ it is always $\ge 1$ byte).

##### Object Size Composition:

The size of an object is determined by:

1. The sum of its non-static data members.
2. Compiler memory alignment and structure padding.
3. A virtual table pointer (`vptr`), if the class declares or inherits virtual functions (adds 4 bytes on 32-bit, 8 bytes on 64-bit systems).
4. _Member functions do NOT increase the size of the object_ (they reside in the `.text` segment).

---

### 3. Encapsulation

**Encapsulation** is the programming mechanism that binds together code (methods) and the data it manipulates (variables), keeping both safe from outside interference and misuse.

```
+-------------------------------------------------------+
|                       CLASS                           |
|                                                       |
|  [ PRIVATE SECTION ]                                  |
|  - double balance;   <-- Inaccessible directly from   |
|  - string pin;           outside the class            |
|         ^                                             |
|         | Controlled Access                           |
|         v                                             |
|  [ PUBLIC SECTION ]                                   |
|  + deposit(amount)   <-- Exposed operational API      |
|  + getBalance()          (Validates inputs & rules)   |
|                                                       |
+-------------------------------------------------------+
```

#### How Encapsulation is Enforced in C++:

- Encapsulation is implemented through **classes** and **access specifiers**:
  - `private`: Accessible only by member functions and `friend` functions/classes.
  - `protected`: Accessible by member functions, friends, and derived (child) classes.
  - `public`: Accessible from any scope where the object is visible.

#### Advantages of Encapsulation:

1. **Data Integrity / Validation:** Prevents invalid state transitions (e.g., preventing a bank balance from becoming negative without overdraft approval).
2. **Controlled Mutability:** Enables fields to be made read-only (getter only) or write-only (setter only).
3. **Decoupling / Maintainability:** Internal variable representations can change (e.g., switching an internal date format from `string` to epoch `time_t`) without breaking external consumer code relying on public methods.

---

### 4. Data Hiding

While often used interchangeably with encapsulation in casual conversation, **Data Hiding** is specifically the _security technique_ of concealing internal object data to restrict direct manipulation from outside the class boundary.

- **Encapsulation is the wrapper (packaging).**
- **Data Hiding is the restriction (insulation).**

```cpp
class Employee {
private:
    // Data Hiding: Direct access strictly denied
    double salary;

public:
    // Controlled Interface: Read access
    double getSalary() const {
        return salary;
    }

    // Controlled Interface: Write access with business logic rules
    void setSalary(double s) {
        if (s >= 0) {
            salary = s;
        } else {
            std::cerr << "Invalid salary amount!\n";
        }
    }
};
```

---

### 5. Abstraction

**Data Abstraction** is the principle of representing essential features of an entity without including background implementation details or complex internal mechanics.

- It provides an **interface-first perspective**: Focuses on **what** an object does rather than **how** it does it.
- **Real-World Analogy:** Driving a car requires interacting with the steering wheel, accelerator, and brake pedals (the abstract interface). You do not need to understand valve timings, ECU fuel injection maps, or hydraulic line pressures (internal complexity) to drive.

#### Implementing Abstraction in C++:

1. **Class-Level Abstraction:** Designing intuitive public member function interfaces while keeping utility and calculation routines hidden in `private` methods.
2. **Interface / Pure Abstract Base Classes:** Using **Abstract Classes** containing **Pure Virtual Functions** (`virtual void draw() = 0;`) to define high-level contracts that derived classes must fulfill.

```cpp
#include <iostream>

// Abstract Class: Pure Abstraction Contract
class DatabaseConnection {
public:
    virtual void connect() = 0;        // Pure virtual function
    virtual void executeQuery(std::string query) = 0;
    virtual void disconnect() = 0;
    virtual ~DatabaseConnection() {}   // Virtual destructor
};

// Concrete Implementation
class PostgreSQLConnection : public DatabaseConnection {
public:
    void connect() override {
        std::cout << "Connected to PostgreSQL via port 5432.\n";
    }
    void executeQuery(std::string query) override {
        std::cout << "Executing SQL query: " << query << "\n";
    }
    void disconnect() override {
        std::cout << "Disconnected from PostgreSQL.\n";
    }
};
```

#### Encapsulation vs Abstraction: The Architectural Contrast

| Dimension          | Encapsulation                                                                         | Abstraction                                                                                   |
| :----------------- | :------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------- |
| **Core Goal**      | **Information Hiding:** Protects internal state from unauthorized external tampering. | **Complexity Management:** Hides system complexity by presenting a simple, unified interface. |
| **Focus**          | Focuses on **HOW** an object stores and protects its data.                            | Focuses on **WHAT** an object does from the consumer's viewpoint.                             |
| **Implementation** | Achieved using `private`, `protected`, and `public` access specifiers.                | Achieved using **Abstract Classes**, **Interfaces**, and **Header files**.                    |
| **Analogy**        | A medical capsule containing active medicinal granules sealed inside.                 | The car dashboard and pedals that abstract the internal combustion mechanics.                 |

---

### 6. Procedural Programming vs Object-Oriented Programming (POP vs OOP)

| Feature / Metric         | Procedural Programming (POP)                                                | Object-Oriented Programming (OOP)                                                    |
| :----------------------- | :-------------------------------------------------------------------------- | :----------------------------------------------------------------------------------- |
| **Core Paradigm**        | Focuses on step-by-step procedures, algorithms, and function calls.         | Focuses on autonomous objects combining both state and behavior.                     |
| **Data Security**        | **Low:** Data moves globally between functions; vulnerable to side-effects. | **High:** Data hiding and access specifiers insulate state from unauthorized access. |
| **Design Approach**      | **Top-Down** design approach (Problem broken down into sub-routines).       | **Bottom-Up** design approach (Base entities and domain objects built first).        |
| **Code Reusability**     | Limited (primarily function calls and libraries).                           | Extensive (achieved via **Inheritance** and **Composition**).                        |
| **Polymorphism**         | No direct concept of dynamic polymorphism.                                  | Supports dynamic dispatch, operator/function overloading, and templates.             |
| **Program Partitioning** | Programs are partitioned into **Functions** / Modules.                      | Programs are partitioned into **Classes** and **Objects**.                           |
| **Data Coupling**        | High coupling between functions and data structures.                        | Low coupling; data is tightly bound only to its owning class.                        |
| **Typical Languages**    | C, Pascal, FORTRAN, BASIC.                                                  | C++, Java, C#, Python.                                                               |

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: What is the size of an empty class in C++, and what is the size of a derived class inheriting from an empty class?**
  - _Answer:_
    - An empty class has a size of **1 byte** to ensure unique memory identity:
      ```cpp
      class Empty {}; // sizeof(Empty) == 1
      ```
    - Under **Empty Base Optimization (EBO)**, if a class inherits from an empty base class, the compiler is allowed to allocate 0 bytes for the empty base, meaning the derived class will not pay the 1-byte penalty:
      ```cpp
      class Derived : public Empty { int x; }; // sizeof(Derived) == 4 (NOT 5 or 8!)
      ```

- **Q2: Does defining member functions inside a class declaration impact the size of instantiated objects?**
  - _Answer:_ **No.** Functions reside in the `.text` (Code) segment of memory, completely independent of instance storage. Instantiated objects store only their non-static data members (plus alignment padding and a `vptr` if virtual functions are present).

- **Q3: What is the default access specifier for a `class` versus a `struct` in C++?**
  - _Answer:_
    - In a `class`, members and base class inheritance default to **`private`**.
    - In a `struct`, members and base class inheritance default to **`public`**.

- **Q4: Can a class achieve Data Hiding without Abstraction?**
  - _Answer:_ Yes. You can mark all data members `private` to achieve data hiding, yet provide trivial getters and setters for every variable without designing high-level functional abstractions. Abstraction requires deliberate modeling of behaviors rather than exposing raw variables.

- **Q5: Why is Object-Oriented Programming considered a "Bottom-Up" approach?**
  - _Answer:_ In OOP, designers identify the fundamental real-world business entities, model them as autonomous classes with their own data and behaviors first, and then compose these independent units together to construct larger, complex application workflows. Procedural code begins with the overarching algorithm (top) and decomposes it downward into subroutines.
