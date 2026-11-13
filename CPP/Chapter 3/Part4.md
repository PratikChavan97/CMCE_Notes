# Module 03: Object-Oriented Programming (Core Concepts)

## Unit 3.4: Destructors and Object Management

### 1. Introduction to Destructors

A **destructor** is a special member function that cleans up an object when its lifetime ends. Its primary responsibility is to release resources acquired during the object's lifetime (e.g., dynamically allocated heap memory, open file handles, database connections, socket descriptors, mutex locks).

```
+--------------------------------------------------------------+
|                     Destructor Invocations                   |
|                                                              |
| 1. Execute statements inside Destructor Body {}              |
| 2. Call destructors of non-static member objects in reverse  |
|    order of their class declaration                          |
| 3. Call destructors of base classes in reverse order of      |
|    derivation                                                |
| 4. Deallocate raw memory occupied by instance (if heap)      |
+--------------------------------------------------------------+
```

#### 1.1 Structural Properties of Destructors

1. **Name Matching:** Uses the class name prefixed with a tilde (`~`): `~ClassName()`.
2. **No Arguments or Return Types:** A destructor **takes no arguments** and **has no return type** (not even `void`).
3. **No Overloading:** Because it takes no arguments, a class can have **only one destructor**. Destructors **cannot be overloaded**.
4. **Automatic Invocation:** The compiler/runtime automatically calls the destructor when:
   - A stack-allocated automatic object goes out of scope (`}`).
   - A dynamically allocated heap object is explicitly deleted via `delete` or `delete[]`.
   - A static or global object exits at program termination (`exit()` or return from `main()`).
5. **No `const` or `static` Qualifiers:** Destructors cannot be marked `const`, `volatile`, or `static`.
6. **Virtual Destructor Requirement:** If a class contains virtual functions or is intended to be used polymorphically as a base class, its destructor **must be declared `virtual`** to prevent partial destruction leaks.

---

### 2. Constructor and Destructor Execution Order

The order of construction and destruction follows a strict **LIFO (Last-In, First-Out) / Reverse Symmetry** model.

```
Creation Phase (FIFO Order):
Base Constructor  ===>  Member Object Constructors  ===>  Derived Constructor Body
                                                                   |
                                                                   v
Destruction Phase (LIFO / Reverse Order):                          |
Base Destructor   <===  Member Object Destructors  <===  Derived Destructor Body
```

#### 2.1 Stack-Allocated Objects Execution Order

Local variables placed on the call stack are destroyed in the **exact reverse order** of their creation:

```cpp
#include <iostream>

class Tracker {
    int id;
public:
    Tracker(int val) : id(val) { std::cout << "C" << id << " "; }
    ~Tracker() { std::cout << "D" << id << " "; }
};

int main() {
    Tracker t1(1);
    Tracker t2(2);
    Tracker t3(3);
    return 0;
}
```

- **Output:** `C1 C2 C3 D3 D2 D1`
- **Rule:** First constructed is last destroyed.

#### 2.2 Global and Static Objects

- **Global Objects:** Constructed **before** `main()` starts; destroyed **after** `main()` returns.
- **Static Local Objects:** Constructed the **first time** control passes their declaration; destroyed at **program exit** (in reverse order of successful initialization).

```cpp
void func() {
    static Tracker s(99); // Constructed once on first call
}
```

---

### 3. Shallow Copy vs Deep Copy

Understanding memory layouts during object replication is vital for preventing memory corruption bugs.

```
1. Shallow Copy (Bitwise copy of pointer address):
   Object A [ ptr: 0x4000 ] -----\
                                  +-----> [ Heap Chunk: "CDAC" ]
   Object B [ ptr: 0x4000 ] -----/       (Both hold same address!)

   Problem:
   - Destructor of B runs -> Heap Chunk 0x4000 is deallocated!
   - Destructor of A runs -> Deallocates 0x4000 AGAIN! ==> DOUBLE FREE CRASH!

2. Deep Copy (Independent resource allocation):
   Object A [ ptr: 0x4000 ] -------------> [ Heap Chunk: "CDAC" ]
   Object B [ ptr: 0x8000 ] -------------> [ Heap Chunk: "CDAC" ]

   Result: Completely independent lifecycles; safe destruction.
```

#### Key Distinctions

| Metric               | Shallow Copy                                                       | Deep Copy                                                                |
| :------------------- | :----------------------------------------------------------------- | :----------------------------------------------------------------------- |
| **Mechanics**        | Bitwise member-to-member copy (default copy constructor)           | Allocates dedicated storage and duplicates underlying contents           |
| **Pointer Handling** | Copies the address (both objects reference the same memory)        | Allocates new memory at a new address; copies dereferenced data          |
| **Safety**           | High risk: causes **Dangling Pointers** and **Double-Free errors** | **Completely safe**; independent object state and lifecycle              |
| **Performance**      | $\mathcal{O}(1)$ time; fast                                        | $\mathcal{O}(N)$ time; requires memory allocation and copying            |
| **When to Use**      | When class manages no external raw heap handles/resources          | Mandatory whenever a class manages raw heap pointers or file descriptors |

---

### 4. Assignment Operator & Copy Assignment

It is essential to distinguish between **Copy Initialization** and **Copy Assignment**:

```cpp
MyClass a;
MyClass b = a;  // 1. COPY CONSTRUCTOR (New object 'b' is being created)
MyClass c(a);   // 1. COPY CONSTRUCTOR (Direct form)

MyClass d;
d = a;          // 2. COPY ASSIGNMENT OPERATOR (Object 'd' ALREADY exists)
```

#### 4.1 Signature of Copy Assignment Operator

```cpp
ClassName& operator=(const ClassName& rhs);
```

- Takes a `const ClassName&` to avoid copying the right-hand operand.
- Returns `ClassName&` (a reference to `*this`) to support **operator chaining**:
  ```cpp
  a = b = c; // Evaluates right-to-left: b.operator=(c), then a.operator=(b)
  ```

#### 4.2 The Canonical Copy Assignment Implementation

A manual implementation must address three critical phases:

1. **Self-Assignment Guard:** Prevent an object from deleting its own resources if assigned to itself (`a = a;`).
2. **Deallocate Existing Resources:** Release current memory to prevent memory leaks.
3. **Allocate and Duplicate:** Allocate new storage and copy the incoming data.

```cpp
#include <cstring>
#include <algorithm>

class Buffer {
private:
    char* data;
    size_t size;

public:
    Buffer(const char* str = "") {
        size = std::strlen(str);
        data = new char[size + 1];
        std::strcpy(data, str);
    }

    // 1. Destructor
    ~Buffer() {
        delete[] data;
    }

    // 2. Copy Constructor
    Buffer(const Buffer& other) : size(other.size) {
        data = new char[size + 1];
        std::strcpy(data, other.data);
    }

    // 3. Copy Assignment Operator
    Buffer& operator=(const Buffer& rhs) {
        // Step 1: Self-assignment check
        if (this == &rhs) {
            return *this;
        }

        // Step 2: Free current object's resources
        delete[] data;

        // Step 3: Deep copy incoming data
        size = rhs.size;
        data = new char[size + 1];
        std::strcpy(data, rhs.data);

        // Step 4: Return *this to allow assignment chaining
        return *this;
    }
};
```

#### 4.3 The Copy-and-Swap Idiom (Exception-Safe Modern Pattern)

The basic implementation above is **not exception-safe**: if `new char[...]` throws `std::bad_alloc`, the object has already freed its old `data`, leaving it in a corrupt/dangling state.

The **Copy-and-Swap Idiom** fixes this by reusing the copy constructor and taking the parameter **by value**:

```cpp
class BufferSafe {
    char* data;
    size_t size;
public:
    void swap(BufferSafe& other) noexcept {
        std::swap(data, other.data);
        std::swap(size, other.size);
    }

    // Pass by value invokes copy constructor automatically!
    BufferSafe& operator=(BufferSafe rhs) noexcept {
        this->swap(rhs);
        return *this;
    } // Old data is automatically deleted when temporary 'rhs' dies here!
};
```

---

### 5. The Rule of Three (and Modern Evolution)

The **Rule of Three** is a foundational C++ design guideline:

> **Rule of Three:** If a class requires an explicit, user-defined implementation of **any one** of the following:
>
> 1. **Destructor** (`~Class()`)
> 2. **Copy Constructor** (`Class(const Class&)`)
> 3. **Copy Assignment Operator** (`Class& operator=(const Class&)`)
>
> ...then it almost certainly requires **all three**.

#### Why?

If a class needs a custom destructor, it is managing external resources (like raw heap memory). If you do not provide custom copy operations, the compiler will synthesize shallow-copy operations, leading to dangling pointers and double frees.

```
                    The Resource Management Spectrum

        Legacy C++ (C++98)                    Modern C++ (C++11/17/20)
       +-------------------+                 +------------------------+
       |   Rule of Three   |                 |      Rule of Five      |
       |                   |                 |  1. Destructor         |
       | 1. Destructor     |                 |  2. Copy Constructor   |
       | 2. Copy Ctor      |  --- Evolves -->|  3. Copy Assign        |
       | 3. Copy Assign    |                 |  4. Move Constructor   |
       +-------------------+                 |  5. Move Assign        |
                                             +------------------------+
                                                         |
                                                         v
                                             +------------------------+
                                             |      Rule of Zero      |
                                             | Use std::unique_ptr,   |
                                             | std::vector, etc.      |
                                             | Define NO custom dtor! |
                                             +------------------------+
```

- **The Rule of Zero:** Prefer classes that manage resources using standard wrappers (`std::string`, `std::vector`, `std::unique_ptr`). In such classes, you do **not** declare a destructor, copy constructor, or copy assignment operator at all—the compiler defaults will manage everything cleanly and safely.

---

### 6. The Complete Object Lifecycle

Every C++ object passes through distinct phases from creation to deallocation:

```
+-------------------------------------------------------------------------+
|                         Object Lifecycle Stages                         |
+-------------------------------------------------------------------------+
| 1. Storage Allocation:                                                  |
|    - Memory reserved on Stack (SP shift), Heap (operator new), or BSS   |
+-------------------------------------------------------------------------+
                                    |
                                    v
+-------------------------------------------------------------------------+
| 2. Construction:                                                        |
|    - Base class initialization                                          |
|    - VPTR established and bound to VTABLE                               |
|    - Members initialized via MIL                                        |
|    - Constructor body {} executed                                       |
+-------------------------------------------------------------------------+
                                    |
                                    v
+-------------------------------------------------------------------------+
| 3. Operational State (Lifetime):                                        |
|    - Member function calls, modifications, passing by ref/val           |
|    - Copy assignment or move assignment if mutated                      |
+-------------------------------------------------------------------------+
                                    |
                                    v
+-------------------------------------------------------------------------+
| 4. Destruction:                                                         |
|    - Triggered by scope exit, explicit delete, or program termination   |
|    - Destructor body {} executed                                        |
|    - Member objects destroyed in reverse order                          |
|    - Base class destructors invoked                                     |
+-------------------------------------------------------------------------+
                                    |
                                    v
+-------------------------------------------------------------------------+
| 5. Storage Deallocation:                                                |
|    - Memory freed back to OS/Heap pool (operator delete / stack unwind) |
+-------------------------------------------------------------------------+
```

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: What is the output of the following program?**

  ```cpp
  #include <iostream>
  class Sample {
  public:
      Sample() { std::cout << "1"; }
      Sample(const Sample&) { std::cout << "2"; }
      Sample& operator=(const Sample&) { std::cout << "3"; return *this; }
  };
  int main() {
      Sample a;
      Sample b = a;
      Sample c;
      c = a;
      return 0;
  }
  ```

  - _Output:_ `1213`
  - _Explanation:_
    - `Sample a;` invokes the default constructor (`1`).
    - `Sample b = a;` uses initialization syntax, invoking the **copy constructor** (`2`), _not_ the assignment operator.
    - `Sample c;` invokes the default constructor (`1`).
    - `c = a;` assigns to an existing object, invoking the **copy assignment operator** (`3`).

- **Q2: Why must a base class destructor be declared `virtual`?**
  - _Answer:_ If a derived class object is deleted via a pointer to its base class (`Base* ptr = new Derived(); delete ptr;`), and the base destructor is **non-virtual**, the compiler performs static binding. Only the `~Base()` destructor will execute. The derived class destructor `~Derived()` will **never run**, causing undefined behavior and resource leaks for derived members. Declaring `virtual ~Base()` enables dynamic dispatch to invoke the full destruction chain from derived to base.

- **Q3: Can a destructor throw an exception in C++?**
  - _Answer:_ **Never.** Destructors default to `noexcept(true)` in modern C++. If a destructor throws an exception while another exception is already active (during stack unwinding), the runtime calls `std::terminate()`, immediately aborting the entire process. Any internal exception within a destructor must be caught and handled locally.

- **Q4: Can we call a destructor explicitly?**
  - _Answer:_ Yes, using the syntax `obj.~ClassName();`. However, this should almost never be done for standard objects because the compiler will call it again when the object leaves scope, leading to a double-destruction bug. The only legitimate use case is in low-level systems code when using **placement-new** (`new (buffer) MyClass()`), where memory is managed manually in a pre-allocated buffer.

- **Q5: What is the difference between `delete p;` and `delete[] p;` under the hood?**
  - _Answer:_ `delete p;` destructs a single object and returns the memory block to the heap. `delete[] p;` reads an allocation cookie/size prefix stored by the compiler directly adjacent to the array on the heap, extracts the element count $N$, invokes destructors for all $N$ objects in reverse order, and frees the entire block. Calling `delete` on an array leads to undefined behavior because only the first element's destructor runs.
