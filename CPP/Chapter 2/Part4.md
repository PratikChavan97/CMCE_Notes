# Module 02: Procedural Extensions & Modular Programming

## Unit 2.4: Memory Management

### 1. Process Memory Layout: Stack vs Heap

When a C++ program launches, the operating system's virtual memory manager allocates a process virtual address space organized into distinct segments:

```
+-------------------------------------------------------+ High Memory (0xFFFFFFFF / 0x7FFFFFFFFFFF)
| Kernel Space (Restricted OS space)                   |
+-------------------------------------------------------+
| Stack Segment (Grows downward: High -> Low)           |
| - Local automatic variables, stack frames, ret addrs  |
|                      |                                |
|                      v                                |
|                                                       |
|                      ^                                |
|                      |                                |
| Heap Segment (Grows upward: Low -> High)              |
| - Dynamic allocations (new, malloc)                  |
+-------------------------------------------------------+
| BSS Segment (Block Started by Symbol)                 |
| - Uninitialized global & static variables (zeroed)    |
+-------------------------------------------------------+
| Data Segment (Initialized Data)                       |
| - Explicitly initialized global & static variables    |
+-------------------------------------------------------+
| Text / Code Segment (Read-Only)                       |
| - Binary machine instructions, string literals        |
+-------------------------------------------------------+ Low Memory (0x00000000)
```

#### Detailed Comparison: Stack vs Heap

| Feature / Dimension      | Stack Memory                                                                | Heap Memory (Free Store)                                                                      |
| :----------------------- | :-------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| **Allocation Mechanism** | Automatic by CPU / compiler (moves stack pointer register `ESP`/`RSP`)      | Manual via program requests (`new` / `malloc` interacting with OS runtime heap manager)       |
| **Deallocation**         | Automatic when variable goes out of scope                                   | **Manual** (must call `delete` / `free`) or via smart pointer RAII destructors                |
| **Access Speed**         | **Extremely fast** (L1/L2 cache-friendly, simple pointer offset arithmetic) | **Slower** (requires pointer traversal, heap fragmentation, locks in multi-threaded runtimes) |
| **Size Limit**           | Fixed and small (typically 1 MB–8 MB default per thread; configurable)      | Large (limited only by available physical RAM and virtual memory / swap space)                |
| **Growth Direction**     | Downward (from high memory addresses to lower addresses on x86)             | Upward (from lower memory addresses to higher addresses)                                      |
| **Fragmentation**        | None (strictly LIFO: Last-In, First-Out allocation order)                   | Prone to **internal and external memory fragmentation** over prolonged runtimes               |
| **Variable Resizing**    | Fixed compile-time size (except variable-length arrays in non-standard C)   | Resizable at runtime (e.g., dynamically reallocating or expanding arrays)                     |
| **Failure Modes**        | **Stack Overflow** (deep recursion, huge local arrays)                      | **`std::bad_alloc`** / Out-of-Memory (exhaustion of available RAM)                            |

---

### 2. Static vs Dynamic Memory Allocation

| Parameter            | Static / Compile-Time Allocation                                          | Dynamic / Runtime Allocation                                                                        |
| :------------------- | :------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------- |
| **Timing**           | Size resolved at compile/link time; allocated at start or function entry  | Exact size determined and requested **at runtime**                                                  |
| **Storage Location** | Stack, Data segment, or BSS segment                                       | **Heap (Free Store)**                                                                               |
| **Flexibility**      | Rigid; array sizes must be compile-time constants (e.g., `int arr[100];`) | Highly flexible; sizes can depend on runtime user inputs (e.g., `cin >> n; int* arr = new int[n];`) |
| **Persistence**      | Tied to block scope (stack) or entire program run (data/bss)              | Persists across functions **until explicitly freed**                                                |
| **Efficiency**       | Zero runtime allocation overhead                                          | Incurs memory allocation lookup and book-keeping overhead                                           |

---

### 3. Dynamic Memory Allocation: `new` and `delete`

In C++, `new` and `delete` are **operators**, whereas `malloc()` and `free()` are standard library **functions**.

#### 3.1 Single Object Allocation & Deallocation

```cpp
// Allocate single integer
int* pInt = new int;        // Uninitialized (holds indeterminate/garbage value)
int* pInit = new int(42);   // Direct initialization (holds 42)
int* pZero = new int{};     // Value-initialization (zero-initialized: holds 0)

delete pInt;   // Deallocates heap memory
delete pInit;
delete pZero;

pInt = nullptr; // Defensive programming: Nullify after deletion
pInit = nullptr;
pZero = nullptr;
```

#### 3.2 Array Allocation & Deallocation (`new[]` and `delete[]`)

```cpp
int size = 5;
int* arr = new int[size];       // Allocates contiguous block of 5 integers (uninitialized)
int* arrZero = new int[size](); // Zero-initializes all 5 integers to 0

// CRITICAL: Arrays allocated with new[] MUST be freed using delete[]
delete[] arr;
delete[] arrZero;

arr = nullptr;
arrZero = nullptr;
```

> ⚠️ **The `new[]` vs `delete` Mismatch Trap (CMCE MCQ Classic):**  
> If an array allocated with `new[]` is released using a plain `delete arr;` instead of `delete[] arr;`:
>
> - For primitive types (`int`, `char`), it causes **undefined behavior** and potential heap corruption.
> - For user-defined class objects, the compiler executes the destructor for **only the very first element**, leaving remaining object instances undestroyed and causing immediate resource leaks.

#### 3.3 What `new` and `delete` Do Under the Hood

When executing `MyClass* obj = new MyClass(arg);`, C++ performs two discrete phases:

1. **Memory Allocation:** Calls `operator new(sizeof(MyClass))` to reserve raw heap bytes (internally wraps `malloc` or an OS kernel allocator).
2. **Object Construction:** Invokes the constructor `MyClass::MyClass(arg)` on that allocated memory block, passing the address as the `this` pointer.

Conversely, executing `delete obj;`:

1. **Object Destruction:** Invokes the destructor `MyClass::~MyClass()` to release internal resources.
2. **Memory Deallocation:** Calls `operator delete(obj)` to release the raw bytes back to the heap pool.

#### 3.4 `new` / `delete` vs `malloc()` / `free()`

| Feature                      | `new` / `delete`                                                    | `malloc()` / `free()`                                                   |
| :--------------------------- | :------------------------------------------------------------------ | :---------------------------------------------------------------------- |
| **Entity Type**              | Language **Operators** (Keyword)                                    | Standard C Library **Functions** (`<cstdlib>`)                          |
| **Constructor / Destructor** | **Calls constructor** on creation; **calls destructor** on deletion | **Never calls constructors or destructors**; treats memory as raw bytes |
| **Return Type**              | Returns exact typed pointer (`MyClass*`); **type-safe**             | Returns generic `void*`; requires explicit type casting                 |
| **Size Calculation**         | **Automatic**: `new MyClass` computes size implicitly               | **Manual**: Must calculate using `sizeof` (`malloc(sizeof(MyClass))`)   |
| **Failure Handling**         | Throws **`std::bad_alloc`** exception by default                    | Returns **`NULL`**                                                      |
| **Reallocation**             | Not natively supported (must allocate new, copy, and delete)        | Supported natively via `realloc()`                                      |
| **Overloadable?**            | Can be overloaded globally or per-class                             | Cannot be overloaded                                                    |

#### 3.5 Handling Allocation Failure: Exceptions vs `std::nothrow`

By default, modern C++ does **not** return `nullptr` if heap allocation fails; it throws a `std::bad_alloc` exception:

```cpp
#include <iostream>
#include <new> // Required for std::bad_alloc and std::nothrow

// Approach 1: Exception Handling (Modern C++ Standard)
try {
    long long* hugeBlock = new long long[10000000000ULL];
} catch (const std::bad_alloc& e) {
    std::cerr << "Allocation failed: " << e.what() << '\n';
}

// Approach 2: nothrow version (Emulates C-style NULL return)
int* ptr = new (std::nothrow) int[10000000000ULL];
if (ptr == nullptr) {
    std::cerr << "Failed to allocate memory without throwing!\n";
}
```

---

### 4. Memory Leaks

A **memory leak** occurs when dynamically allocated heap memory loses all accessible pointer references without first being returned to the system via `delete` or `free`.

```
Step 1: Allocation                       Step 2: Pointer Overwritten (LEAK!)
+---------+      +---------------+       +---------+      +---------------+
|   ptr   | ---> | Heap Block A  |       |   ptr   | -X-> | Heap Block A  | (Orphaned / Unreachable)
+---------+      +---------------+       +---------+      +---------------+
                                              |
                                              v           +---------------+
                                                          | Heap Block B  |
                                                          +---------------+
```

#### Common Causes of Memory Leaks:

1. **Reassigning pointers before deallocation:**
   ```cpp
   int* p = new int(10);
   p = new int(20); // LEAK: The memory holding '10' is orphaned forever!
   ```
2. **Missing `delete` before function returns:**
   ```cpp
   void processData() {
       int* buffer = new int[256];
       if (someErrorCondition) {
           return; // LEAK: buffer is lost when stack frame pops!
       }
       delete[] buffer;
   }
   ```
3. **Non-virtual destructors in polymorphic base classes:**
   ```cpp
   Base* b = new Derived();
   delete b; // If Base's destructor is not virtual, Derived cleanup is skipped!
   ```

#### Consequences & Detection:

- Leads to progressive RAM bloat, system swapping, slowdowns, and eventual process termination by the OS OOM (Out Of Memory) killer.
- **Detection Tools:** **Valgrind** (Memcheck on Linux: `valgrind --leak-check=full ./a.out`), AddressSanitizer (`g++ -fsanitize=address -g main.cpp`).

---

### 5. Dangling Pointers

A **dangling pointer** is a pointer that continues referencing a memory address whose underlying storage has already been deallocated or freed.

#### Scenarios Creating Dangling Pointers:

1. **Deallocating heap memory without clearing the pointer:**

   ```cpp
   int* ptr = new int(50);
   delete ptr; // Memory freed, but ptr still holds the old address!
   // *ptr = 100; // CRITICAL: Writing to a dangling pointer (Undefined Behavior / Crash)
   ```

2. **Returning the address of an automatic stack variable:**

   ```cpp
   int* getStackAddress() {
       int localVal = 42;
       return &localVal; // BUG: localVal is destroyed upon returning!
   } // Returned pointer is dangling immediately
   ```

3. **Variable going out of block scope:**
   ```cpp
   int* p = nullptr;
   {
       int blockVar = 10;
       p = &blockVar;
   } // blockVar destroyed here
   // 'p' is now a dangling pointer
   ```

_Prevention:_ Assign pointers to `nullptr` immediately after calling `delete`, and never return pointers/references to local stack variables.

---

### 6. Null Pointers

A **null pointer** is a pointer explicitly assigned a reserved sentinel value indicating that it points to no valid memory object.

- **Dereferencing `nullptr`:**
  ```cpp
  int* p = nullptr;
  *p = 10; // CRASH: Operating system triggers SIGSEGV (Segmentation Fault)
  ```
- **Deleting a Null Pointer is Safe:**
  In C++, calling `delete` or `delete[]` on a `nullptr` is **completely safe and guaranteed to do nothing (no-op)**:
  ```cpp
  int* p = nullptr;
  delete p;   // Completely safe, no crash
  delete[] p; // Completely safe
  ```

---

### 7. Memory Management Best Practices

#### 7.1 Follow RAII (Resource Acquisition Is Initialization)

Bind the lifecycle of dynamic resources directly to object lifetimes. Acquire resources in constructors; release them in destructors:

```cpp
class ResourceWrapper {
private:
    int* data;
public:
    ResourceWrapper(size_t size) : data(new int[size]) {} // Acquire
    ~ResourceWrapper() { delete[] data; }                 // Release automatically
    // Delete copy operations or implement Rule of Five!
};
```

#### 7.2 Prefer Modern Smart Pointers (`<memory>`) over Raw Pointers

Raw pointers should only be used as non-owning observers. Dynamic resource management should use standard smart pointers:

1. **`std::unique_ptr<T>` (Exclusive Ownership):**
   - Automatically deletes managed memory when the pointer goes out of scope.
   - Zero runtime overhead compared to a raw pointer.

   ```cpp
   #include <memory>
   std::unique_ptr<int> uptr = std::make_unique<int>(100);
   // No explicit delete required; freed automatically upon scope exit
   ```

2. **`std::shared_ptr<T>` (Shared Ownership):**
   - Uses an internal thread-safe reference count.
   - Memory is deallocated when the final owner goes out of scope.

3. **`std::weak_ptr<T>` (Non-Owning Reference):**
   - Points to `shared_ptr` objects without incrementing the reference count; breaks cyclic memory dependency leaks.

#### 7.3 Rule of Three / Rule of Five

If a class explicitly manages raw dynamic memory, it **must** explicitly define:

1. **Destructor** (`~Class()`)
2. **Copy Constructor** (`Class(const Class&)`)
3. **Copy Assignment Operator** (`Class& operator=(const Class&)`)
   _(In Modern C++, add Move Constructor and Move Assignment Operator for the **Rule of Five**)._

---

### 8. High-Yield CMCE Traps & Interview Questions

- **Q1: What is the difference between `delete` and `delete[]`?**
  - _Answer:_ `delete` calls the destructor of a single object and deallocates its memory. `delete[]` reads the hidden allocation overhead metadata (stored just before the array block, indicating the number of elements), invokes the destructor for **every element** in the array in reverse order, and frees the entire block. Using `delete` on a pointer allocated with `new[]` results in undefined behavior.

- **Q2: Does `free()` call the destructor of an object?**
  - _Answer:_ **No.** `free()` only knows the address of the raw heap block and marks that chunk as free in the runtime heap manager's table. It does not invoke any C++ destructors.

- **Q3: What happens if `new` cannot find enough contiguous heap memory?**
  - _Answer:_ By default, the global allocation function throws a `std::bad_alloc` exception. If the non-throwing placement variant `new (std::nothrow)` is specified, it suppresses the exception and returns `nullptr`.

- **Q4: Why does deleting a pointer twice (Double Free) crash the program?**
  - _Answer:_ Double free corrupts the internal free-list data structures maintained by the heap allocator, allowing attackers to manipulate heap metadata or causing immediate termination (`SIGABRT` / `heap-use-after-free`). Setting pointers to `nullptr` right after `delete` prevents this, because subsequent `delete nullptr` calls are harmless no-ops.

- **Q5: Can `sizeof` determine the size of dynamically allocated heap memory?**
  - _Answer:_ **No.** `sizeof` is a compile-time operator. Applying `sizeof` to a dynamically allocated pointer (e.g., `int* arr = new int[50]; sizeof(arr);`) yields only the size of the pointer itself (4 or 8 bytes), not the total capacity of the allocated heap chunk.
