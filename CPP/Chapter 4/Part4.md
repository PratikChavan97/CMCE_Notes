# Module 04: Inheritance & Polymorphism

## Unit 4.4: Advanced Inheritance and Polymorphism

### 1. Pure Virtual Functions

A **pure virtual function** (or abstract method) is a virtual function declared in a base class that has **no implementation** within that base class (though an out-of-line body is technically permitted by the standard) and **must be overridden** by any concrete derived class.

#### 1.1 Syntax & Pure Specifier (`= 0`)

The `= 0` syntax is known as the **pure-specifier**:

```cpp
class Shape {
public:
    // Pure virtual function declaration
    virtual double calculateArea() const = 0;
    virtual void render() = 0;
};
```

- **Contractual Mandate:** The presence of `= 0` tells the compiler: _"Any non-abstract class deriving from this class must supply a concrete definition for this function, or compilation will fail."_
- **VTABLE Representation:** In the class's `VTABLE`, the slot corresponding to a pure virtual function points to a runtime error handler (often named `__cxa_pure_virtual`), which aborts the program if triggered directly.

#### 1.2 Can a Pure Virtual Function Have a Body?

**Yes.** A pure virtual function can provide an implementation, but the definition **must be written outside the class declaration**:

```cpp
class Logger {
public:
    virtual void log(const std::string& msg) = 0; // Pure virtual
    virtual ~Logger() = default;
};

// Legal: Out-of-line definition providing default/fallback behavior
void Logger::log(const std::string& msg) {
    std::cout << "[Default Log]: " << msg << "\n";
}

class FileLogger : public Logger {
public:
    void log(const std::string& msg) override {
        Logger::log(msg); // Derived class explicitly reuses base implementation
        // Specific file write logic here...
    }
};
```

---

### 2. Abstract Classes

An **Abstract Class** is any class that contains **at least one pure virtual function**.

#### 2.1 Invariants of Abstract Classes

1. **Cannot Be Instantiated:** Attempting to create an object directly from an abstract class causes a compile-time error:
   ```cpp
   // Shape s; // COMPILE ERROR: Cannot allocate an object of abstract type 'Shape'
   ```
2. **Pointers and References are Allowed:** You cannot instantiate an abstract object, but you **can declare pointers and references** to an abstract class type. This forms the foundation of dynamic interface programming:
   ```cpp
   Shape* ptr = new Circle(5.0); // LEGAL: Polymorphic pointer
   Shape& ref = *ptr;            // LEGAL: Polymorphic reference
   ```
3. **Incomplete Overrides:** If a derived class fails to override _every_ pure virtual function inherited from its base, that derived class **also remains abstract** and cannot be instantiated.

---

### 3. Virtual Destructors: The Complete Mechanical Breakdown

Declaring a **virtual destructor** in any class containing virtual functions is one of the most critical safety idioms in C++.

#### 3.1 The Catastrophic Memory Leak (Non-Virtual Destructor)

Consider deleting a derived class instance via a base class pointer:

```cpp
class Base {
public:
    Base() { std::cout << "Base Constructed\n"; }
    ~Base() { std::cout << "Base Destroyed\n"; } // NON-VIRTUAL!
};

class Derived : public Base {
private:
    int* heapBuffer;
public:
    Derived() : heapBuffer(new int[1000]) { std::cout << "Derived Constructed\n"; }
    ~Derived() {
        delete[] heapBuffer;
        std::cout << "Derived Destroyed\n";
    }
};

int main() {
    Base* ptr = new Derived();
    delete ptr; // UNDEFINED BEHAVIOR / SEVERE RESOURCE LEAK!
    return 0;
}
```

- **Console Output:**
  ```
  Base Constructed
  Derived Constructed
  Base Destroyed
  ```
- **Why did `Derived Destroyed` never print?**
  Because `~Base()` is non-virtual, the compiler performs **Static Binding (Early Binding)** based on the static type of `ptr` (`Base*`). The compiler hardcodes a jump directly to `~Base()`. The `~Derived()` destructor never executes, leaving `heapBuffer` allocated on the heap permanently (**Memory Leak**).

#### 3.2 The Virtual Destructor Solution

```cpp
class Base {
public:
    virtual ~Base() { std::cout << "Base Destroyed\n"; } // VIRTUAL!
};
```

- **Corrected Output:**
  ```
  Base Constructed
  Derived Constructed
  Derived Destroyed
  Base Destroyed
  ```
- **Mechanism:** When marked `virtual`, destructor dispatch looks up the object's `_vptr` and routes through the `VTABLE`. The derived destructor is invoked first, releases its resources, and then automatically cascades up to call the base destructor.

#### 3.3 Pure Virtual Destructor

A class can be rendered abstract even without functional methods by declaring its destructor pure virtual:

```cpp
class AbstractBase {
public:
    virtual ~AbstractBase() = 0; // Pure virtual destructor
};

// MANDATORY: Pure virtual destructors MUST have an out-of-line body!
AbstractBase::~AbstractBase() {}
```

> **Why must a pure virtual destructor have a body?**
> During destruction of a derived object (`delete ptr;`), the derived destructor runs first and _always_ chains upward to call the base class destructor `~AbstractBase()`. If no body exists, the linker fails with an `undefined reference to AbstractBase::~AbstractBase()`.

---

### 4. Multiple Inheritance & Ambiguity Resolution

Multiple inheritance occurs when a derived class inherits simultaneously from two or more base classes.

```
       [ Class A ]        [ Class B ]
            \                  /
             \                /
              v              v
             [ Derived Class C ]
```

#### 4.1 Member Ambiguity Pitfall

If base classes contain identical member signatures, unqualified calls fail:

```cpp
class NetworkPrinter {
public:
    void ping() { std::cout << "Printer Online\n"; }
};

class FileServer {
public:
    void ping() { std::cout << "Server Online\n"; }
};

class OfficeHub : public NetworkPrinter, public FileServer {};

int main() {
    OfficeHub hub;
    // hub.ping(); // COMPILE ERROR: Request for member 'ping' is ambiguous
    hub.NetworkPrinter::ping(); // OK: Explicit resolution using ::
    hub.FileServer::ping();     // OK
}
```

---

### 5. The Diamond Problem & Virtual Base Classes

The **Diamond Problem** arises when two classes inherit from a common base class, and a fourth class inherits from both intermediate classes.

```
                  [ PoweredDevice ]             <-- Base Class (has int powerRating;)
                    /           \
                   /             \
        [ ScannerDevice ]     [ PrinterDevice ]  <-- Intermediate classes
                   \             /
                    \           /
                   [ AllInOneMachine ]          <-- Derived class
```

#### 5.1 The Mechanical Issue Without Virtual Inheritance:

1. `ScannerDevice` receives a subobject of `PoweredDevice`.
2. `PrinterDevice` receives a second, distinct subobject of `PoweredDevice`.
3. `AllInOneMachine` embeds **both independent copies** inside its instance memory layout:
   - **Ambiguity:** Accessing `machine.powerRating` fails because the compiler cannot tell which parent copy is referenced.
   - **Resource Inconsistency:** Updating one copy leaves the other in an outdated state.

#### 5.2 Resolution: Virtual Base Classes (`virtual public`)

Declaring the intermediate base classes with the **`virtual`** keyword ensures that only **one single shared instance** of the root base class is constructed within the final derived class:

```cpp
#include <iostream>

class PoweredDevice {
public:
    int powerRating;
    PoweredDevice(int power) : powerRating(power) {
        std::cout << "PoweredDevice constructed (" << power << "W)\n";
    }
};

// Use virtual inheritance
class ScannerDevice : virtual public PoweredDevice {
public:
    ScannerDevice(int power) : PoweredDevice(power) {}
};

class PrinterDevice : virtual public PoweredDevice {
public:
    PrinterDevice(int power) : PoweredDevice(power) {}
};

// Most derived class
class AllInOneMachine : public ScannerDevice, public PrinterDevice {
public:
    // CRITICAL: The MOST DERIVED class is responsible for initializing the virtual base!
    AllInOneMachine(int power)
        : PoweredDevice(power), ScannerDevice(power), PrinterDevice(power) {
        std::cout << "AllInOneMachine ready\n";
    }
};

int main() {
    AllInOneMachine machine(120);
    machine.powerRating = 150; // NO AMBIGUITY: Only one shared subobject exists!
    return 0;
}
```

#### 5.3 Constructor Rule for Virtual Bases

- In regular inheritance, classes only initialize their immediate base classes.
- In **virtual inheritance**, the **most derived class** (`AllInOneMachine`) must invoke the constructor of the virtual base (`PoweredDevice`) directly. Calls to the virtual base constructor from intermediate classes (`ScannerDevice`, `PrinterDevice`) are ignored during instantiation.

---

### 6. Interface Design in C++

Unlike Java or C#, C++ does not have a dedicated `interface` keyword. Instead, an **Interface** is represented as a **Pure Abstract Base Class** that satisfies strict architectural criteria.

```
+-------------------------------------------------------------+
|                     C++ Interface Contract                  |
|                                                             |
|  - Contains ONLY pure virtual functions (= 0)               |
|  - Declares NO non-static member variables (stateless)      |
|  - Provides a public virtual destructor (~Interface() = 0)  |
|  - May contain static constants or helper type aliases      |
+-------------------------------------------------------------+
```

#### 6.1 Complete Interface Design Pattern Example

```cpp
#include <iostream>
#include <memory>
#include <string>

// Formal C++ Interface
class IPayable {
public:
    virtual ~IPayable() = default; // Essential virtual destructor
    virtual void processPayment(double amount) = 0;
    virtual std::string getTransactionCurrency() const = 0;
};

// Implementation 1
class CreditCardProcessor : public IPayable {
public:
    void processPayment(double amount) override {
        std::cout << "Processed $" << amount << " via Credit Card Gateway.\n";
    }
    std::string getTransactionCurrency() const override {
        return "USD";
    }
};

// Implementation 2
class CryptoProcessor : public IPayable {
public:
    void processPayment(double amount) override {
        std::cout << "Transferred $" << amount << " equivalent in BTC.\n";
    }
    std::string getTransactionCurrency() const override {
        return "BTC";
    }
};

// Client code depends purely on the interface (Dependency Inversion Principle)
void checkout(IPayable* gateway, double total) {
    gateway->processPayment(total);
}
```

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: Can an abstract class have a constructor?**
  - _Answer:_ **Yes.** Although an abstract class cannot be instantiated directly, its constructors are called by derived class constructors through the Member Initializer List (MIL) to initialize base class member variables.

- **Q2: What is the underlying memory cost of using `virtual` base classes?**
  - _Answer:_ Objects of classes using virtual inheritance include a hidden internal pointer (a **`vbptr`** or virtual base table offset in the `VTABLE`). This allows the runtime to dynamically resolve the memory offset to the shared base subobject, increasing `sizeof(object)` and adding pointer-offset calculation overhead at runtime.

- **Q3: What happens if you define a pure virtual destructor without a body?**
  - _Answer:_ The code compiles, but during linking or execution of a derived object's destructor, a **linker error** (`undefined reference to Base::~Base()`) occurs. This is because derived destructors implicitly call base destructors during object destruction.

- **Q4: What is the difference between an Abstract Class and an Interface in C++?**
  - _Answer:_ An **Abstract Class** can have concrete member functions, instance state (member variables), and non-public access specifiers alongside pure virtual methods. An **Interface** contains _only_ pure virtual functions, has no member variables (stateless), and provides only a public virtual destructor.

- **Q5: What is the output of this code?**

  ```cpp
  #include <iostream>
  class A {
  public:
      virtual void f() = 0;
  };
  void A::f() { std::cout << "A"; }

  class B : public A {
  public:
      void f() override { A::f(); std::cout << "B"; }
  };
  int main() {
      B obj;
      obj.f();
      return 0;
  }
  ```

  - _Output:_ `AB`
  - _Explanation:_ Pure virtual functions can have definitions. Derived classes can invoke the base pure virtual function explicitly using full scope resolution (`A::f()`).
