# Module 06: Templates & Generic Programming

## Unit 6.4: Advanced C++ Concepts (Modern C++ & Resource Semantics)

### 1. Advanced Operator Overloading

Operator overloading gives user-defined types first-class language semantics. While operators can be overloaded either as **member functions** or **non-member (friend) functions**, certain operators enforce strict structural rules.

```
                           Operator Overloading
                                     |
         +---------------------------+---------------------------+
         |                                                       |
Member Functions Only                                Non-Member / Friend Functions
 - Assignment: =                                      - Stream Insertion: <<
 - Subscript:  []                                     - Stream Extraction: >>
 - Invocation: ()                                     - Commutative Math: + - * /
 - Member Access via Pointer: ->                      - Mixed-type Operations (e.g., 2 + obj)

```

#### 1.1 Non-Overloadable Operators (Memorize for CDAC CMCE)

The following six tokens can **never** be overloaded in C++:

1. `::` (Scope Resolution)

2. `.` (Direct Member Access)

3. `.*` (Pointer-to-Member Dereference)

4. `?:` (Ternary Conditional)

5. `sizeof` (Size Determination)

6. `typeid` (RTTI Type Information)

#### 1.2 Overloading Prefix vs Postfix (`++` / `--`)

The compiler distinguishes prefix from postfix by passing a dummy `int` parameter to the postfix overload:

```
class Counter {
private:
    int count;
public:
    Counter(int c = 0) : count(c) {}

    // Prefix: ++c (Increments first, returns by REFERENCE)
    Counter& operator++() {
        ++count;
        return *this; // Fast: zero copies
    }

    // Postfix: c++ (Saves previous state, increments, returns OLD state by VALUE)
    Counter operator++(int) { // 'int' is a dummy parameter used strictly as a tag
        Counter temp = *this; // Incurs copy constructor overhead
        ++count;
        return temp;          // Returns temporary copy by value
    }
};

```

### 2. Value Categories: Lvalues, Rvalues & Rvalue References (`&&`)

C++11 introduced move semantics based on formal value categories:

```
                               Expressions
                                    |
                    +---------------+---------------+
                    |                               |
              glvalue (Generalized)           rvalue (Read value)
                    |                               |
         +----------+----------+         +----------+----------+
         |                     |         |                     |
      lvalue                xvalue     xvalue                prvalue
(Named memory slot)     (Expiring obj) (Expiring obj)    (Pure temporary)

```

- **Lvalue (Locator Value):** An expression identifying a persistent memory location. You can take its address using `&` (e.g., `int x = 10;`, `std::string str;`).

- **Prvalue (Pure Rvalue):** A temporary value not associated with a persistent object (e.g., literal `42`, temporary result `a + b`, function returning by value).

- **Xvalue (eXpiring Value):** An object whose resources can be repurposed/reused (e.g., the result of `std::move(x)`).

- **Rvalue Reference (`Type&&`):** A reference that binds **exclusively to rvalues/temporaries**, enabling functions to steal internal heap resources rather than cloning them.

```
int a = 10;
int& lref = a;       // OK: lvalue reference binds to lvalue
// int&& rref = a;   // ERROR: rvalue reference cannot bind to lvalue
int&& rref = 10;     // OK: rvalue reference binds directly to temporary literal
int&& movedRef = std::move(a); // OK: std::move casts lvalue 'a' to an rvalue (xvalue)

```

### 3. Copy Semantics vs Move Semantics

```
Copy Semantics (Allocation + Bit-for-bit duplication):
Source [ buffer: 0x1000 ] ------------> [ Heap: "Large Data Array" ]
                                            |
                                            v (Deep memory copy via new[])
Target [ buffer: 0x9000 ] ------------> [ Heap: "Large Data Array" ]

Move Semantics (Shallow pointer swap + source nullification):
Source [ buffer: 0x1000 ] ---\
                              +-------> [ Heap: "Large Data Array" ]
Target [ buffer: 0x1000 ] ---/          (No heap allocation, O(1) performance!)
Source [ buffer: nullptr ]              (Source reset to harmless empty state)

```

| Dimension        | Copy Semantics (C++98)                       | Move Semantics (C++11)                                           |
| ---------------- | -------------------------------------------- | ---------------------------------------------------------------- |
| **Mechanics**    | Allocates new memory, clones source elements | Steals existing pointer/resource handle from source              |
| **Performance**  | $\mathcal{O}(N)$ heap allocation overhead    | $\mathcal{O}(1)$ pointer reassignment                            |
| **Source State** | Unchanged and intact                         | Placed into a valid but unspecified (usually empty) state        |
| **Trigger**      | Initializing/assigning from lvalues          | Initializing/assigning from rvalues (temporaries or `std::move`) |

#### What does `std::move()` actually do?

`std::move()` does **not** move any bytes or execute any runtime code. It is simply a compile-time cast:

$$
\text{std::move}(x) \equiv \text{static\_cast}<T\&\&>(x)
$$

It tells the compiler: _"Treat this lvalue as an rvalue so its resources can be stolen by move constructors or move assignment operators."_

### 4. Move Constructor & Move Assignment Operator

To implement move semantics, a class must define a move constructor and a move assignment operator taking **rvalue references (`Type&&`)**.

#### 4.1 Implementation of the Dynamic Array Wrapper

```
#include <iostream>
#include <utility>

class DynamicBuffer {
private:
    int* data;
    size_t size;

public:
    // 1. Parameterized Constructor
    DynamicBuffer(size_t s) : size(s), data(new int[s]{}) {
        std::cout << "Constructed size " << size << "\n";
    }

    // 2. Destructor
    ~DynamicBuffer() {
        delete[] data; // Safely deletes nullptr if moved from!
    }

    // 3. Copy Constructor (Deep Copy: O(N))
    DynamicBuffer(const DynamicBuffer& other) : size(other.size), data(new int[other.size]) {
        std::copy(other.data, other.data + size, data);
        std::cout << "Copy Constructor\n";
    }

    // 4. Move Constructor (Resource Theft: O(1) - NOEXCEPT MANDATORY)
    DynamicBuffer(DynamicBuffer&& other) noexcept
        : data(other.data), size(other.size) { // 1. Steal pointer and size
        other.data = nullptr;                 // 2. Nullify source pointer
        other.size = 0;                       // 3. Reset source size
        std::cout << "Move Constructor\n";
    }

    // 5. Copy Assignment Operator (Deep Copy: O(N))
    DynamicBuffer& operator=(const DynamicBuffer& rhs) {
        if (this == &rhs) return *this;
        delete[] data;
        size = rhs.size;
        data = new int[size];
        std::copy(rhs.data, rhs.data + size, data);
        std::cout << "Copy Assignment\n";
        return *this;
    }

    // 6. Move Assignment Operator (Resource Theft: O(1) - NOEXCEPT MANDATORY)
    DynamicBuffer& operator=(DynamicBuffer&& rhs) noexcept {
        if (this == &rhs) return *this; // Self-assignment check

        delete[] data;                  // 1. Release our existing memory
        data = rhs.data;                // 2. Steal resource
        size = rhs.size;

        rhs.data = nullptr;             // 3. Leave source in safe state
        rhs.size = 0;
        std::cout << "Move Assignment\n";
        return *this;
    }
};

```

#### 4.2 Why `noexcept` is Mandatory on Move Operations (CMCE Exam Focus)

Standard containers like `std::vector` resize by moving elements to a newly allocated contiguous buffer. However, to satisfy the **Strong Exception Guarantee** (if an error occurs, the original state is preserved), `std::vector` will **only** call the move constructor if it is marked `noexcept`. If `noexcept` is omitted, `std::vector` silently falls back to the slow, copying copy constructor!

### 5. The Rule of Five

If a class manages raw external resources, implementing any one of these requires implementing **all five**:

1. **Destructor** (`~Class()`)

2. **Copy Constructor** (`Class(const Class&)`)

3. **Copy Assignment Operator** (`Class& operator=(const Class&)`)

4. **Move Constructor** (`Class(Class&&) noexcept`)

5. **Move Assignment Operator** (`Class& operator=(Class&&) noexcept`)

### 6. Smart Pointers (`<memory>`)

Smart pointers are class templates wrapping raw pointers that manage heap memory using **RAII** (Resource Acquisition Is Initialization). When the smart pointer leaves scope, its destructor invokes `delete` or `delete[]` automatically.

```
                         Smart Pointers
                               |
         +---------------------+---------------------+
         |                                           |
std::unique_ptr (C++11)                     std::shared_ptr (C++11)
- Exclusive Ownership                       - Shared Reference Counted
- Non-Copyable, Movable Only                - Heavyweight Control Block
- Zero runtime overhead                              |
                                            std::weak_ptr (C++11)
                                            - Non-Owning Observer
                                            - Breaks Cyclic Memory Leaks

```

### 7. `std::unique_ptr` (Exclusive Ownership)

A `std::unique_ptr` owns and manages another object through a pointer and disposes of that object when the `unique_ptr` goes out of scope.

#### 7.1 Key Invariants

- **Non-Copyable:** The copy constructor and copy assignment operator are explicitly `= delete`.

- **Movable:** Ownership can be transferred to another `unique_ptr` using `std::move()`.

- **Zero Overhead:** Matches raw pointers in size and execution speed (unless customized with stateful deleters).

```
#include <iostream>
#include <memory>

class Task { public: void execute() {} };

int main() {
    // Preferred creation: make_unique (C++14) - Exception-safe & concise
    std::unique_ptr<Task> t1 = std::make_unique<Task>();

    // std::unique_ptr<Task> t2 = t1; // COMPILE ERROR: Cannot copy unique_ptr!

    // Transfer ownership using std::move:
    std::unique_ptr<Task> t2 = std::move(t1); // t1 becomes nullptr; t2 owns Task

    if (!t1) {
        std::cout << "t1 is now null after move.\n";
    }

    t2->execute();
    return 0; // t2 leaves scope: Task is automatically destroyed!
}

```

#### 7.2 Custom Deleters with `unique_ptr`

Unlike `shared_ptr`, the type of custom deleter is part of the `unique_ptr` type signature:

```
auto fileDeleter = [](FILE* fp) { if (fp) fclose(fp); };
std::unique_ptr<FILE, decltype(fileDeleter)> filePtr(fopen("data.txt", "r"), fileDeleter);

```

### 8. `std::shared_ptr` (Shared Ownership)

A `std::shared_ptr` retains shared ownership of an object through an internal pointer. Several `shared_ptr` instances can point to the same object.

#### 8.1 The Control Block Architecture

The managed object is destroyed and its memory deallocated when the **last remaining `shared_ptr`** owning the object goes out of scope or is reassigned.

```
shared_ptr A               shared_ptr B
[ ptr | ctrlBlock* ]      [ ptr | ctrlBlock* ]
     \       /                 \       /
      \     /                   \     /
       v   v                     v   v
    +-----------------------------------------------+
    |            Shared Control Block               |
    | - Strong Reference Count: 2 (shared_ptrs)     |
    | - Weak Reference Count:   0 (weak_ptrs)       |
    | - Custom Deleter / Allocator                  |
    +-----------------------------------------------+
                          |
                          v
               [ Managed Heap Object ]

```

#### 8.2 Usage & `std::make_shared`

```
#include <iostream>
#include <memory>

int main() {
    // make_shared performs 1 single heap allocation for both object & control block
    std::shared_ptr<int> sp1 = std::make_shared<int>(500);
    std::cout << "Use count: " << sp1.use_count() << "\n"; // 1

    {
        std::shared_ptr<int> sp2 = sp1; // Copying increments reference count
        std::cout << "Use count: " << sp1.use_count() << "\n"; // 2
    } // sp2 destroyed here -> reference count drops back to 1

    std::cout << "Use count: " << sp1.use_count() << "\n"; // 1
    return 0; // sp1 destroyed -> ref count reaches 0 -> memory freed!
}

```

### 9. `std::weak_ptr` & The Cyclic Dependency Problem

A `std::weak_ptr` is a non-owning smart pointer that observes an object managed by `std::shared_ptr` without incrementing its strong reference count.

#### 9.1 The Cyclic Dependency Trap (Severe Memory Leak)

```
        Node A                                  Node B
+---------------------+                 +---------------------+
| shared_ptr<Node> next | ------------> | shared_ptr<Node> next |
+---------------------+                 +---------------------+
           ^                                       |
           |                                       |
           +---------------------------------------+

```

When two objects reference each other with `std::shared_ptr`, their reference counts will **never fall below 1**, even after all external pointers leave scope. The destructors will never run, creating a persistent **memory leak**.

#### 9.2 The Solution: Break the Cycle with `std::weak_ptr`

```
#include <iostream>
#include <memory>

class Node {
public:
    std::string name;
    // std::shared_ptr<Node> next; // BUG: Cyclic leak!
    std::weak_ptr<Node> next;      // FIX: Weak non-owning pointer breaks cycle!

    Node(std::string n) : name(n) {}
    ~Node() { std::cout << "Node " << name << " destroyed\n"; }
};

int main() {
    auto a = std::make_shared<Node>("A");
    auto b = std::make_shared<Node>("B");

    a->next = b; // Assigns shared_ptr to weak_ptr (no ref count increment)
    b->next = a;

    return 0; // Both destructors run cleanly!
}

```

#### 9.3 Converting `weak_ptr` to `shared_ptr` (`lock()`)

Because a `weak_ptr` does not keep an object alive, it cannot be dereferenced directly (`operator->` does not exist on `weak_ptr`). You must first verify its validity using `.lock()`:

```
std::weak_ptr<Node> wptr = a;

// Promoting weak_ptr to temporary shared_ptr safely:
if (std::shared_ptr<Node> sptr = wptr.lock()) {
    std::cout << "Node is still alive: " << sptr->name << "\n";
} else {
    std::cout << "Object has already been deleted!\n";
}

```

### 10. Master Smart Pointer Comparison Matrix

| Feature                | `std::unique_ptr`                  | `std::shared_ptr`                                          | `std::weak_ptr`                 |
| ---------------------- | ---------------------------------- | ---------------------------------------------------------- | ------------------------------- |
| **Ownership**          | Exclusive (1 owner)                | Shared ($N$ owners)                                        | Non-owning observer             |
| **Copyable?**          | ❌ **No** (`= delete`)             | ✅ **Yes**                                                 | ✅ **Yes**                      |
| **Movable?**           | ✅ **Yes** (`std::move`)           | ✅ **Yes**                                                 | ✅ **Yes**                      |
| **Memory Footprint**   | Same as raw pointer ($8\text{ B}$) | 2 pointers ($16\text{ B}$: Object ptr + Control block ptr) | 2 pointers ($16\text{ B}$)      |
| **Reference Counting** | None                               | Strong & Weak counts                                       | Observes weak count             |
| **Direct Dereference** | `*p`, `p->`                        | `*p`, `p->`                                                | ❌ **No** (Must call `.lock()`) |
| **Creation Utility**   | `std::make_unique` (C++14)         | `std::make_shared` (C++11)                                 | Initialized from `shared_ptr`   |

### 11. High-Yield CMCE Traps & Interview Questions

- **Q1: Why is `std::make_shared` preferred over `std::shared_ptr<T>(new T())`?**
  - _Answer:_
    1. **Performance:** `new T()` allocates memory twice (one heap allocation for the object, and a second separate allocation for the control block). `std::make_shared` combines both into a **single contiguous heap allocation**.

    2. **Exception Safety:** In expressions like `func(std::shared_ptr<T>(new T()), riskyOp())`, argument evaluation order is unspecified prior to C++17. If `riskyOp()` throws after `new T()` but before the `shared_ptr` constructor wraps it, the memory leaks. `std::make_shared` eliminates this hazard.

- **Q2: What is the output of `sizeof(std::unique_ptr<int>)` vs `sizeof(std::shared_ptr<int>)` on a 64-bit architecture?**
  - _Answer:_
    - `sizeof(unique_ptr)` is **8 bytes** (raw pointer size).

    - `sizeof(shared_ptr)` is **16 bytes** (one 8-byte pointer to the managed object, plus one 8-byte pointer to the shared control block).

- **Q3: What happens if you call `std::move()` on a `const` object?**
  - _Answer:_ It fails to move and silently falls back to a **deep copy**! `std::move(constObj)` casts to `const T&&`. Because move constructors take non-const `T&&`, the compiler cannot bind to the move constructor and instead matches the copy constructor `const T&`.

- **Q4: Can you assign an array to a `std::unique_ptr`?**
  - _Answer:_ **Yes.** `std::unique_ptr<int[]>` has a specialization for dynamic arrays that automatically calls `delete[]` instead of `delete` and provides `operator[]` for indexing:

    ```
    std::unique_ptr<int[]> arr = std::make_unique<int[]>(10);
    arr[0] = 42; // Indexing supported directly

    ```

- **Q5: Can two independent `std::shared_ptr` instances be constructed using the same raw pointer?**
  - _Answer:_ **Never.** Doing `T* raw = new T(); std::shared_ptr<T> p1(raw); std::shared_ptr<T> p2(raw);` creates **two separate control blocks**, each believing it is the sole owner with a reference count of 1. When both go out of scope, they will both call `delete raw;`, triggering an immediate **Double Free crash**.
