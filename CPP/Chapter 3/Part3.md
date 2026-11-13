# Module 03: Object-Oriented Programming (Core Concepts)

## Unit 3.3: Constructors & Object Lifecycle

A **constructor** is a special non-static member function that initializes newly created objects of its class, allocating underlying resources and placing the object into a valid initial state.

---

### 1. Introduction to Constructors

#### 1.1 Core Characteristics

1. **Name Matching:** Shares the exact same name as the enclosing class.
2. **No Return Type:** Has **no return type**, not even `void`. (Returning a value or writing `void ClassName()` is a compile-time error).
3. **Automatic Invocation:** Invoked automatically by the runtime runtime system upon object instantiation; it cannot be called directly like an ordinary member function (except via explicit placement-new syntax).
4. **Placement in Memory:** Primarily declared in the `public` section. If placed in `private`, instantiation from general code is prevented (used in Singleton patterns and factory classes).
5. **No `virtual` Qualifier:** A constructor **cannot be `virtual`** because virtual dispatch requires a valid Virtual Table Pointer (`vptr`), which is only wired up by the constructor during object instantiation.

```
+--------------------------------------------------------------+
|                    Constructor Invocation                    |
|                                                              |
| 1. Allocate raw bytes on Stack or Free Store (operator new)  |
| 2. Base class constructors executed (if inheritance applies) |
| 3. Set up VPTR (if class or base has virtual functions)      |
| 4. Initialize members via Member Initializer List (MIL)      |
| 5. Execute statements inside Constructor Body {}             |
+--------------------------------------------------------------+
```

---

### 2. Default Constructor

A default constructor is a constructor that can be called with **no arguments** (either taking zero parameters or having default values for all parameters).

```cpp
class Device {
private:
    int deviceId;
    bool status;
public:
    // User-defined Default Constructor
    Device() {
        deviceId = 0;
        status = false;
    }
};
```

#### 2.1 Compiler-Synthesized Default Constructor Rules

If a class defines **no constructors at all**, the compiler implicitly generates an inline `public` default constructor (the **defaulted constructor**).

- **Trivial Initialization Trap:** The synthesized default constructor performs **default-initialization**:
  - Primitive built-in types (`int`, `double`, raw pointers) on the stack are left with **indeterminate garbage values**.
  - Member objects of class types have their own default constructors invoked.

> ⚠️ **The Suppression Rule (High-Yield CMCE MCQ):**  
> As soon as you declare **any user-defined constructor** that takes arguments, the compiler **suppresses** its automatic default constructor synthesis.

```cpp
class Sample {
public:
    Sample(int x) {} // Parameterized constructor defined
};

int main() {
    Sample s1(10); // OK
    Sample s2;     // COMPILE ERROR: No matching default constructor exists!
    return 0;
}
```

- **Modern C++ Fix (`= default`):** Re-enable default generation explicitly:
  ```cpp
  Sample() = default; // Tells compiler to synthesize standard default constructor
  ```

---

### 3. Parameterized Constructor

A constructor that accepts one or more arguments to initialize member variables with custom runtime data at the point of instantiation.

```cpp
class Complex {
private:
    double real, imag;
public:
    // Parameterized constructor
    Complex(double r, double i) {
        real = r;
        imag = i;
    }
};

int main() {
    // Invocation Styles:
    Complex c1(3.0, 4.0);         // Functional notation (Implicit call)
    Complex c2 = Complex(1.5, 2.5); // Explicit temporary construction
    Complex c3{5.0, 6.0};         // Uniform brace initialization (C++11)
    return 0;
}
```

#### 3.1 Single-Argument Constructors & the `explicit` Keyword

A constructor callable with a single argument acts as an **implicit type conversion operator** by default:

```cpp
class Entity {
private:
    int age;
public:
    Entity(int a) : age(a) {} // Implicit converting constructor
};

void printEntity(const Entity& e) { /* ... */ }

int main() {
    printEntity(25); // LEGAL: Implicitly converts int '25' to Entity(25)!
}
```

To prevent unexpected and accidental type conversions, prefix single-argument constructors with `explicit`:

```cpp
class Entity {
public:
    explicit Entity(int a) : age(a) {}
};

int main() {
    // printEntity(25);      // COMPILE ERROR: Conversion blocked
    printEntity(Entity(25)); // OK: Explicit conversion required
}
```

---

### 4. Copy Constructor

A copy constructor initializes a new object by copying data members from an existing object of the **same class**.

#### 4.1 Signature & The Pass-By-Reference Invariant

```cpp
ClassName(const ClassName& source);
```

> ⚠️ **The Infinite Recursion Trap (CMCE Interview Classic):**  
> Why **MUST** the parameter be passed by reference (`const ClassName&`) and **NEVER by value (`ClassName`)**?
>
> If passed by value (`ClassName source`), the act of passing an argument by value itself **requires invoking the copy constructor**. Calling the copy constructor would pass by value again, triggering an infinite recursive loop of copy constructor calls until the call stack blows with a **Stack Overflow**. Passing by reference prevents this copy.

#### 4.2 Shallow Copy vs. Deep Copy

```
Shallow Copy (Pointer copied; shared memory):
Object A [ dataPtr: 0x5000 ] -----\
                                   +---> [ Heap Memory: "CDAC" ]
Object B [ dataPtr: 0x5000 ] -----/

Deep Copy (Independent buffer allocated):
Object A [ dataPtr: 0x5000 ] ---------> [ Heap Memory: "CDAC" ]
Object B [ dataPtr: 0x9000 ] ---------> [ Heap Memory: "CDAC" ]
```

- **Shallow Copy (Default behavior):**
  - The compiler-synthesized copy constructor copies bits member-by-member (**bitwise/memberwise copy**).
  - If a class contains raw dynamic pointers (`int* ptr = new int`), both objects end up pointing to the **exact same physical memory address**.
  - **Consequence:** Destructor execution triggers a **Double Free** crash, and mutations through one object silently corrupt the other.

- **Deep Copy (User-defined implementation):**
  - Allocates fresh heap storage for the new object and copies the actual underlying data values.

```cpp
#include <iostream>
#include <cstring>

class StringWrapper {
private:
    char* buffer;
public:
    // Parameterized constructor
    StringWrapper(const char* str) {
        buffer = new char[std::strlen(str) + 1];
        std::strcpy(buffer, str);
    }

    // User-Defined DEEP COPY Constructor
    StringWrapper(const StringWrapper& other) {
        buffer = new char[std::strlen(other.buffer) + 1]; // 1. Allocate own memory
        std::strcpy(buffer, other.buffer);               // 2. Duplicate data
    }

    // Destructor
    ~StringWrapper() {
        delete[] buffer; // Safe: Each instance frees its own buffer
    }
};
```

#### 4.3 Scenarios When Copy Constructor is Invoked:

1. Instantiating an object explicitly initialized from another: `MyClass b = a;` or `MyClass b(a);`
2. Passing an object **by value** to a function: `void process(MyClass obj);`
3. Returning an object **by value** from a function: `MyClass create();` _(Subject to RVO/Copy Elision)_

---

### 5. Constructor Overloading

A class can define multiple constructors having different parameter counts, types, or sequences.

```cpp
class Box {
private:
    double length, breadth, height;
public:
    Box() : length(1), breadth(1), height(1) {}                    // Default
    Box(double side) : length(side), breadth(side), height(side) {} // Single value
    Box(double l, double b, double h) : length(l), breadth(b), height(h) {} // All values
};
```

#### 5.1 Delegating Constructors (C++11)

A constructor can invoke another constructor of the same class in its member initializer list to avoid repetitive logic:

```cpp
class Window {
private:
    int width, height;
public:
    // Target constructor
    Window(int w, int h) : width(w), height(h) { /* complex setup */ }

    // Delegating default constructor
    Window() : Window(800, 600) {}
};
```

---

### 6. Member Initializer Lists (MIL)

The Member Initializer List is placed between the constructor argument list and the body block `{}`:

```cpp
ClassName(args) : member1(arg1), member2(arg2) {
    // Body
}
```

```
Assignment in Body (Two Steps):           Member Initializer List (One Step):
1. Default-construct member               1. Direct-construct member with target value
2. Invoke operator= to assign value          (Zero redundant work)
```

#### 6.1 The Four Mandatory Cases for Using MIL

You **must** use a Member Initializer List in these scenarios (attempting to initialize them inside `{}` is a compile-time error):

1. **Initializing `const` Data Members:** Const values cannot be assigned after construction.
2. **Initializing Reference Data Members (`&`):** References must be bound when initialized.
3. **Base Class Parameterized Constructor Calls:** Passing arguments to a parent class lacking a default constructor.
4. **Member Objects Lacking a Default Constructor:** Initializing member class instances that require parameters.

```cpp
class Engine {
public:
    Engine(int cylinders) {} // No default constructor
};

class Car {
private:
    const int manufacturingYear; // 1. const member
    int& vinReference;          // 2. reference member
    Engine engine;              // 3. member object without default constructor

public:
    Car(int year, int& vin, int cyl)
        : manufacturingYear(year),   // OK
          vinReference(vin),         // OK
          engine(cyl)                // OK
    {
        // manufacturingYear = year; // COMPILE ERROR!
        // vinReference = vin;       // COMPILE ERROR!
    }
};
```

#### 6.2 Initialization Order Invariant (High-Yield CMCE Trap)

> ⚠️ **CRITICAL RULE:**  
> Data members are initialized strictly in the **order of their declaration inside the class**, **NOT** the order written in the Member Initializer List!

```cpp
class OrderTest {
public:
    int a;
    int b;

    // MIL writes b first, but 'a' is declared first above!
    OrderTest(int val) : b(val), a(b + 10) {}
};

int main() {
    OrderTest obj(5);
    std::cout << obj.a << " " << obj.b;
}
```

- **Result:** `obj.a` will contain **garbage/undefined data**, because `a` is initialized first using the uninitialized contents of `b`!

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: What is the "Most Vexing Parse" in C++?**
  - _Answer:_ An ambiguity resolution rule where statements that can be parsed as a function declaration are parsed that way instead of as an object instantiation:
    ```cpp
    class Worker { public: Worker() {} };
    // Looks like object creation, but compiler treats it as a function declaration!
    Worker w(); // Declares a function 'w' returning Worker with zero arguments!
    Worker w;   // Correct object instantiation
    Worker w{}; // Correct (Modern C++ brace syntax)
    ```

- **Q2: Can constructors be overloaded? Can destructors be overloaded?**
  - _Answer:_ **Constructors can be overloaded** (by varying signatures). **Destructors CANNOT be overloaded** because they take no arguments (`~ClassName()`).

- **Q3: What happens if an exception is thrown inside a constructor?**
  - _Answer:_ The object is considered **not fully constructed**. The destructor for that object will **not** be called! However, destructors for base classes and data members that were already successfully initialized before the exception _will_ run as the stack unwinds.

- **Q4: Can a constructor be private?**
  - _Answer:_ **Yes.** Marking constructors `private` prevents external direct instantiation. This is used in the **Singleton Pattern** (restricting to one shared instance) and the **Named Constructor Idiom / Factory Pattern**.

- **Q5: What is Copy Elision / Return Value Optimization (RVO)?**
  - _Answer:_ A compiler optimization that eliminates unnecessary copying of temporary objects returned by value. Since C++17, copy elision for temporaries is **guaranteed by the standard**, bypassing the copy/move constructor altogether even if it has side effects.
