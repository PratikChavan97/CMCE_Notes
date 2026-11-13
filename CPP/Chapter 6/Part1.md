# Module 06: Templates & Generic Programming

## Unit 6.1: Function and Class Templates

### 1. Introduction to Templates & Generic Programming

**Generic Programming** is a software design approach where algorithms, data structures, and functions are authored independently of the concrete data types they operate on.

In C++, generic programming is implemented through **Templates**.

```
                           Generic Blueprint
                       template <typename T>
                                 |
         +-----------------------+-----------------------+
         | (Instantiation)       | (Instantiation)       | (Instantiation)
         v                       v                       v
    T = int                 T = double              T = std::string
Concrete int function   Concrete double func    Concrete string func
```

#### 1.1 The Shift from Macros (`#define`) and `void*` to Templates

Before templates, type-agnostic programming relied on preprocessor macros or generic `void*` pointers:

- **Macros (`#define MAX(a, b) ((a) > (b) ? (a) : (b))`):**
  - Lack type safety entirely.
  - Prone to dangerous double-evaluation side effects (e.g., `MAX(++x, y)`).
  - Do not respect class scopes or namespaces.
- **C-Style `void*` Pointers:**
  - Require manual and unsafe type casting.
  - Strip away compile-time type verification.
  - Incur runtime pointer indirection overhead and cannot work seamlessly with operator semantics.
- **C++ Templates:**
  - Fully **type-safe**.
  - Evaluated strictly at **compile time** (zero runtime lookup penalty).
  - Generated on demand for the exact types used.

#### 1.2 Under the Hood: Two-Phase Translation & Code Bloat

When the compiler encounters a template, it does **not** generate executable machine instructions immediately. It follows **Two-Phase Translation**:

1. **Phase 1 (Definition Time):** The template definition is parsed for basic syntax errors. No concrete machine code is emitted.
2. **Phase 2 (Instantiation Time):** When client code invokes the template with concrete type arguments (e.g., `MyVector<int>`), the compiler substitutes `T` with `int` and synthesizes a distinct, concrete class or function.

- **Code Bloat (Trade-off):** Because each unique type parameter combination triggers a new function or class instantiation, instantiating `Stack<int>`, `Stack<float>`, and `Stack<double>` results in three complete, separate copies of the class's machine code in the final binary.

---

### 2. Function Templates

A **function template** defines a generic algorithm that works with different types without duplicating code manually.

#### 2.1 Syntax & Basic Example

```cpp
#include <iostream>

// 'typename' and 'class' are completely interchangeable in template headers
template <typename T>
T findMax(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    // 1. Implicit Type Deduction (Compiler infers T from arguments)
    std::cout << findMax(10, 20) << "\n";       // Deduces T = int
    std::cout << findMax(5.5, 2.1) << "\n";     // Deduces T = double

    // 2. Explicit Type Specification (Mandatory if deduction is ambiguous)
    std::cout << findMax<int>(10, 25.5) << "\n";// Forces T = int (25.5 cast to 25)

    // findMax(10, 25.5); // COMPILE ERROR: Deduced conflicting types for T (int vs double)
    return 0;
}
```

#### 2.2 Multiple Type Parameters in Function Templates

When operands might legitimately have differing types, declare multiple type parameters:

```cpp
template <typename T1, typename T2>
auto multiply(T1 a, T2 b) -> decltype(a * b) { // Trailing return type (or auto in C++14)
    return a * b;
}

// In Modern C++14/17:
template <typename T1, typename T2>
auto add(T1 a, T2 b) {
    return a + b;
}
```

#### 2.3 Overloading Function Templates

Function templates can be overloaded with other templates or with non-template functions:

```cpp
#include <iostream>

// 1. Primary Template
template <typename T>
void printVal(T val) {
    std::cout << "Template: " << val << "\n";
}

// 2. Non-Template Overload
void printVal(int val) {
    std::cout << "Non-Template (int): " << val << "\n";
}

int main() {
    printVal(42);       // Calls Non-Template (Exact match preference!)
    printVal<>(42);     // Calls Template explicitly via empty diamond <>
    printVal(3.14);     // Calls Template (T = double)
    return 0;
}
```

> **Resolution Precedence Rule:** When a regular non-template function and an instantiated function template have equally good matches, the **non-template function is always chosen first**.

---

### 3. Class Templates

A **class template** provides a blueprint for generating type-safe, generic classes (e.g., containers like vectors, linked lists, and stacks).

#### 3.1 Syntax and Implementation

```cpp
#include <iostream>
#include <stdexcept>

template <typename T>
class Stack {
private:
    T* elements;
    int topIndex;
    int capacity;

public:
    Stack(int cap = 10);
    ~Stack();

    void push(const T& item);
    T pop();
    bool isEmpty() const { return topIndex == -1; }
};

// Defining member functions outside the class body:
// MUST repeat template header and use ClassName<T>:: scope qualifier!

template <typename T>
Stack<T>::Stack(int cap) : capacity(cap), topIndex(-1) {
    elements = new T[capacity];
}

template <typename T>
Stack<T>::~Stack() {
    delete[] elements;
}

template <typename T>
void Stack<T>::push(const T& item) {
    if (topIndex >= capacity - 1) {
        throw std::overflow_error("Stack full!");
    }
    elements[++topIndex] = item;
}

template <typename T>
T Stack<T>::pop() {
    if (isEmpty()) {
        throw std::underflow_error("Stack empty!");
    }
    return elements[topIndex--];
}
```

#### 3.2 Instantiation and Usage

Unlike function templates prior to C++17, class templates historically **required explicit template arguments**:

```cpp
int main() {
    Stack<int> intStack(5);             // Concrete class Stack<int> instantiated
    intStack.push(10);
    intStack.push(20);

    Stack<std::string> strStack(5);     // Concrete class Stack<std::string> instantiated
    strStack.push("CDAC");
    strStack.push("CMCE");

    return 0;
}
```

_(Note: C++17 introduced **CTAD (Class Template Argument Deduction)**, allowing `std::pair p(10, 20);` without `<int, int>` based on constructor arguments)._

---

### 4. Template Parameters

Templates accept three distinct categories of parameters:

```
                         Template Parameters
                                  |
         +------------------------+------------------------+
         |                        |                        |
   Type Parameters       Non-Type Parameters      Template Template Parameters
 (typename / class T)    (Compile-time constants)    (template <class> class TT)
```

#### 4.1 Type Parameters (`typename` vs `class`)

Inside a template declaration prefix, `typename` and `class` are **100% semantically equivalent**:

```cpp
template <typename T> // Completely identical to:
template <class T>
```

_(Historical context: `class` was the original keyword; `typename` was introduced later to avoid confusion and to resolve dependent type names)._

#### 4.2 Non-Type Template Parameters (NTTP)

A **non-type template parameter** represents a **compile-time constant value** rather than a data type.

- **Permitted Types (up through C++17):**
  - Integral and enumeration types (`int`, `size_t`, `char`, `bool`).
  - Pointers and references to objects or functions with external/static linkage.
  - `std::nullptr_t`.
- **Prohibited Types (prior to C++20):**
  - Floating-point numbers (`float`, `double` were forbidden prior to C++20).
  - Non-static local strings or heap addresses.

##### Implementation Example: Fixed-Size Buffer (like `std::array`)

```cpp
#include <iostream>

template <typename T, size_t Size> // 'Size' is a Non-Type Parameter
class StaticArray {
private:
    T data[Size]; // Statically allocated fixed-size array on the stack
public:
    size_t getSize() const { return Size; }

    T& operator[](size_t index) {
        return data[index];
    }
};

int main() {
    StaticArray<int, 5> smallArr;       // Size = 5 baked into the type
    StaticArray<int, 10> largeArr;      // Size = 10 baked into the type

    // CRITICAL: They are two COMPLETELY DIFFERENT types!
    // smallArr = largeArr; // COMPILE ERROR: Type mismatch!
    return 0;
}
```

#### 4.3 Default Template Arguments

Both class and function templates can define default arguments (evaluated from left to right):

```cpp
template <typename T = int, size_t Capacity = 100>
class CircularQueue {
    T buffer[Capacity];
};

CircularQueue<> defaultQueue;       // CircularQueue<int, 100>
CircularQueue<double> doubleQueue;  // CircularQueue<double, 100>
```

---

### 5. The Critical Header Inclusion Rule (Linker Constraint)

> ⚠️ **THE #1 MOST TESTED TEMPLATE PITFALL IN CDAC & INTERVIEWS:**
>
> _Why must template definitions be placed entirely in header files (`.h` / `.hpp`) rather than separated into `.cpp` source files?_

#### The Problem:

In standard C++, classes place declarations in `MyClass.h` and function bodies in `MyClass.cpp`. If you do this with a **template**:

```cpp
// Stack.h
template <typename T>
class Stack {
public:
    void push(T val);
};

// Stack.cpp
#include "Stack.h"
template <typename T>
void Stack<T>::push(T val) { /* implementation */ }

// main.cpp
#include "Stack.h"
int main() {
    Stack<int> s;
    s.push(10); // LINKER ERROR: Undefined reference to Stack<int>::push(int)
}
```

#### The Cause:

1. `main.cpp` compiles independently into `main.o`. It includes only `Stack.h`. It needs the compiler to instantiate `Stack<int>::push(int)`, but the **definition (body) is not in the header**, so the compiler leaves an unresolved external symbol.
2. `Stack.cpp` compiles into `Stack.o`. Because no code inside `Stack.cpp` uses `Stack<int>`, the compiler **never instantiates** machine code for `Stack<int>`.
3. During Phase 4 (Linking), the linker looks for the symbol `Stack<int>::push(int)` and finds it nowhere, throwing an **Undefined Reference Linker Error**.

#### The Solution:

Keep **all** template member definitions directly in the header file (`.h` / `.hpp`), or explicitly include the `.cpp` file at the bottom of the header.

---

### 6. High-Yield CMCE Traps & Interview Questions

- **Q1: Can a template member function be virtual?**
  - _Answer:_
    - A **member function of a class template** can be virtual:
      ```cpp
      template <typename T>
      class Base {
          virtual void process(T val); // 100% LEGAL
      };
      ```
    - A **template member function inside a regular or template class CANNOT be virtual**:
      ```cpp
      class Regular {
          template <typename T>
          virtual void run(); // COMPILE ERROR!
      };
      ```
      _Reason:_ The compiler cannot know in advance how many instantiations of `run<T>()` will exist throughout the program, making it impossible to compute a fixed size for the class's `VTABLE`.

- **Q2: What is the difference between `typename` and `class` when declaring a template?**
  - _Answer:_ In the template declaration header (`template <typename T>` vs `template <class T>`), there is **zero difference**. However, `typename` has an additional mandatory use: disambiguating **dependent types** (nested types dependent on a template parameter, e.g., `typename T::iterator it;`).

- **Q3: What will `sizeof` evaluate to for two instantiations of `StaticArray<int, 5>` and `StaticArray<int, 10>`?**
  - _Answer:_ Assuming standard 4-byte integers:
    - `sizeof(StaticArray<int, 5>)` is $5 \times 4 = 20\text{ bytes}$.
    - `sizeof(StaticArray<int, 10>)` is $10 \times 4 = 40\text{ bytes}$.
      They are treated as completely distinct types with different sizes and binary layouts.

- **Q4: Can static member variables be used in class templates? How are they shared?**
  - _Answer:_ Each **distinct template instantiation gets its own independent static member variable**. `Stack<int>` and `Stack<double>` do _not_ share static members; all `Stack<int>` instances share one copy, and all `Stack<double>` instances share another distinct copy.

- **Q5: Does dynamic binding occur with templates?**
  - _Answer:_ **No.** Templates represent **Parametric Polymorphism**, which is resolved entirely at **compile-time (Static Binding)**. There is no `VPTR`, no `VTABLE`, and no runtime lookup indirection overhead.
