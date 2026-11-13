# Module 06: Templates & Generic Programming

## Unit 6.3: Namespaces, RTTI and Type Casting

### 1. Namespaces

A **namespace** is a declarative region providing a scope to the identifiers (names of types, functions, variables) declared inside it. Namespaces prevent symbol naming collisions in large systems and multi-library integrations.

```
Global Scope
+------------------------------------+------------------------------------+
| namespace NetworkEngine {          | namespace GraphicEngine {          |
|     class Buffer { ... };          |     class Buffer { ... };          |
|     void send() { ... }            |     void render() { ... }          |
| }                                  | }                                  |
+------------------------------------+------------------------------------+
  Accessible via:                      Accessible via:
  NetworkEngine::Buffer                GraphicEngine::Buffer
```

#### 1.1 Defining and Extending Namespaces

- **Discontinuous/Open-Ended:** Namespaces can be declared across multiple files or blocks. The compiler aggregates all blocks with the same namespace identifier into a single unified scope:

```cpp
// File1.h
namespace Utilities {
    void compress();
}

// File2.h
namespace Utilities { // Adds to the existing Utilities namespace
    void encrypt();
}
```

#### 1.2 Accessing Namespaces

1. **Explicit Scope Resolution (Recommended Best Practice):**
   ```cpp
   Utilities::compress();
   ```
2. **`using` Declaration (Selective Import):**
   ```cpp
   using Utilities::compress;
   compress(); // Encrypt still requires Utilities::encrypt()
   ```
3. **`using namespace` Directive (Bulk Import):**
   ```cpp
   using namespace Utilities;
   compress();
   encrypt();
   ```
   > ⚠️ **Code Smell Hazard:** Avoid placing `using namespace std;` inside header files (`.h` / `.hpp`). It injects hundreds of identifiers into the global namespace of any translation unit including that header, causing namespace pollution.

#### 1.3 Nested and Inline Namespaces (C++11 / C++17)

```cpp
// C++17 Compact Nested Namespace Syntax:
namespace Company::Project::Engine {
    void run();
}

// Inline Namespace (API Versioning):
namespace API {
    inline namespace V2 { // Default target
        void endpoint() { std::cout << "v2.0\n"; }
    }
    namespace V1 {
        void endpoint() { std::cout << "v1.0\n"; }
    }
}

int main() {
    API::endpoint();     // Resolves to API::V2::endpoint()
    API::V1::endpoint(); // Explicitly calls legacy v1.0
}
```

#### 1.4 Anonymous (Unnamed) Namespaces vs C-style `static`

When a namespace has no name, identifiers declared within it are accessible directly in that file but have **internal linkage**:

```cpp
namespace {
    int internalTracker = 0; // Accessible only inside this translation unit (.cpp)
}
```

- **CMCE Exam Note:** Modern C++ deprecates `static` global variables for file-scope restriction in favor of **unnamed namespaces**.

---

### 2. Scope Resolution Operator (`::`)

The scope resolution operator (`::`) dictates the exact lookup context of an identifier.

| Usage Pattern             | Syntax                       | Meaning / Target                                                               |
| :------------------------ | :--------------------------- | :----------------------------------------------------------------------------- |
| **Global Scope Access**   | `::identifier`               | Accesses global variable hidden by an identically named local variable         |
| **Namespace Scope**       | `NamespaceName::member`      | Resolves a member within a specific namespace                                  |
| **Class Scope**           | `ClassName::member`          | Accesses static members, nested classes, or defines methods out-of-line        |
| **Base Class Resolution** | `BaseClass::virtualMethod()` | Suppresses dynamic dispatch to invoke the base class implementation explicitly |

---

### 3. Runtime Type Information (RTTI)

**RTTI** provides language-level support to determine the dynamic/exact derived type of an object at **runtime** using pointers or references to polymorphic base classes.

#### 3.1 Architectural Requirement for RTTI:

- RTTI works **only** on classes that are **polymorphic** (i.e., declare or inherit **at least one `virtual` function**).
- **Under the Hood:** The compiler stores a pointer to a `std::type_info` structure inside the class's **`VTABLE`** (usually placed at offset `-1` or directly adjacent to the table pointer). If a class has no virtual functions, it has no `VTABLE`, making runtime type identification impossible.

```
Object Instance in RAM                   VTABLE in .rodata
+------------------------+              +------------------------------+
| _vptr (8 bytes)        | -----------> | [-1]: ptr to std::type_info  |
+------------------------+              +------------------------------+
| member variables       |              | [ 0]: &Derived::virtualFunc  |
+------------------------+              +------------------------------+
```

---

### 4. The `typeid` Operator & `std::type_info`

Declared in the `<typeinfo>` header, `typeid` queries the type of an expression or object.

```cpp
#include <iostream>
#include <typeinfo>

class Base { public: virtual ~Base() = default; };
class Derived : public Base {};
class NonPoly {};

int main() {
    int a = 42;
    std::cout << typeid(a).name() << "\n"; // Typically prints "i" (implementation-dependent)

    Base* bPtr = new Derived();
    // Polymorphic resolution (queries VTABLE type_info):
    std::cout << typeid(*bPtr).name() << "\n"; // Identifies dynamic type "Derived"

    NonPoly* np = new NonPoly();
    // Non-polymorphic resolution (Static type lookup):
    std::cout << typeid(*np).name() << "\n";   // Identifies static type "NonPoly"

    // Type equality comparison:
    if (typeid(*bPtr) == typeid(Derived)) {
        std::cout << "Pointing to Derived!\n";
    }

    delete bPtr;
    delete np;
    return 0;
}
```

#### 4.1 Key Characteristics of `typeid`:

1. Returns an immutable reference to a `const std::type_info` object.
2. The `std::type_info` class has a private/deleted copy constructor and cannot be copied directly.
3. Member functions: `.name()` (mangled type string), `operator==`, `operator!=`, `.before()`.
4. **`std::bad_typeid` Exception:** If `typeid` is applied to a dereferenced null pointer of a polymorphic type (`typeid(*ptr)` where `ptr == nullptr`), the runtime throws `std::bad_typeid`.

---

### 5. C++ Named Type Casting Operators

C++ provides four explicit, named type casting operators to replace unsafe C-style casts (`(type)expression`):

```
                        C++ Type Casting Operators
                                     |
         +-----------------+---------+---------+-----------------+
         |                 |                   |                 |
    static_cast       dynamic_cast         const_cast     reinterpret_cast
(Compile-time safe)  (Runtime RTTI check) (Add/strip const) (Bitwise raw conversion)
```

#### 5.1 `static_cast`

Performs **compile-time conversions** between compatible types. It checks validity during compilation.

- **Primary Use Cases:**
  1. Standard numeric conversions (e.g., `double` to `int`, `int` to `float`).
  2. Upcasting in class hierarchies (Derived pointer/reference $\rightarrow$ Base pointer/reference).
  3. Downcasting **WITHOUT runtime checks** (programmer guarantees correctness; risky if wrong).
  4. Explicit conversions via user-defined conversion operators or single-arg constructors.
  5. Converting `void*` to a concrete typed pointer (`int*`).

```cpp
double pi = 3.14159;
int intPi = static_cast<int>(pi); // Truncates safely to 3

void* vptr = &intPi;
int* iptr = static_cast<int*>(vptr); // Valid

Base* b = new Derived();
Derived* d = static_cast<Derived*>(b); // Downcast allowed, but unchecked!
```

---

#### 5.2 `dynamic_cast`

Performs **safe runtime downcasting** and cross-casting in polymorphic inheritance hierarchies using RTTI.

- **Requirements:** Base class **must be polymorphic** (contain at least one `virtual` function).
- **Behavior with Pointers:** Returns `nullptr` if the cast fails.
- **Behavior with References:** Throws **`std::bad_cast`** exception if the cast fails (references cannot be null).

```cpp
#include <iostream>
class Base { public: virtual ~Base() = default; };
class Derived : public Base { public: void specialized() {} };
class Unrelated {};

int main() {
    Base* b1 = new Derived();
    Base* b2 = new Base();

    // 1. Pointer Downcasting (Safe check):
    Derived* d1 = dynamic_cast<Derived*>(b1);
    if (d1) {
        std::cout << "d1 cast succeeded!\n"; // Runs
    }

    Derived* d2 = dynamic_cast<Derived*>(b2);
    if (!d2) {
        std::cout << "d2 cast failed! Safely returned nullptr.\n"; // Runs
    }

    // 2. Reference Downcasting:
    try {
        Derived& ref1 = dynamic_cast<Derived&>(*b1); // Succeeds
        Derived& ref2 = dynamic_cast<Derived&>(*b2); // Throws std::bad_cast!
    } catch (const std::bad_cast& e) {
        std::cerr << "Reference cast failed: " << e.what() << "\n";
    }

    delete b1;
    delete b2;
    return 0;
}
```

---

#### 5.3 `const_cast`

The only C++ casting operator that can **add or remove `const` or `volatile`** attributes from a variable.

- **Type Invariant:** The target type must be identical to the source type except for the `const`/`volatile` qualifiers. You cannot use `const_cast` to convert an `int` to a `double`.
- **Undefined Behavior Danger:** Modifying an object that was **originally declared as `const`** via a casting pointer leads to immediate **Undefined Behavior** (the operating system often places original const variables in read-only `.rodata` memory).

```cpp
void legacyAPI(char* str); // Takes non-const pointer, but does not modify it

void safeWrapper(const char* readOnlyStr) {
    // Strips const attribute to satisfy legacy interface signature
    legacyAPI(const_cast<char*>(readOnlyStr));
}

int main() {
    const int originalConst = 100;
    int* p = const_cast<int*>(&originalConst);
    *p = 200; // UNDEFINED BEHAVIOR! originalConst was fundamentally defined const.
}
```

---

#### 5.4 `reinterpret_cast`

Performs purely low-level bit reinterpretation without altering data representations. It converts between unrelated pointer types or between pointers and integer addresses.

- **Compile-Time Only:** Generates no machine-code conversion instructions; it simply instructs the compiler to treat the bit sequence as a different type.
- **Primary Use Cases:**
  1. Low-level driver/embedded development (e.g., mapping raw peripheral memory addresses: `reinterpret_cast<UART_Reg*>(0x40003000)`).
  2. Pointer-to-integer and integer-to-pointer conversions (`uintptr_t`).
  3. Reading raw binary byte buffers into struct representations.

```cpp
struct DataPacket {
    int id;
    char code;
};

char buffer[sizeof(DataPacket)];
DataPacket* packet = reinterpret_cast<DataPacket*>(buffer); // Direct overlay
```

---

### 6. Master Casting Comparison Matrix

| Cast Operator          | Timing       | Safety Mechanism               | Target Transformations                                     | Failure Result                            |
| :--------------------- | :----------- | :----------------------------- | :--------------------------------------------------------- | :---------------------------------------- |
| **`static_cast`**      | Compile-Time | Type compatibility checks      | Numeric, conversions, checked upcasts, `void*` conversions | Compile Error                             |
| **`dynamic_cast`**     | **Run-Time** | **RTTI / VTABLE verification** | Downcasts & cross-casts in polymorphic hierarchies         | `nullptr` (ptrs) / `std::bad_cast` (refs) |
| **`const_cast`**       | Compile-Time | Strips/Adds CV qualifiers      | Adding or removing `const` / `volatile`                    | Compile Error                             |
| **`reinterpret_cast`** | Compile-Time | **None** (Blind bit-mapping)   | Unrelated pointers, pointer $\leftrightarrow$ integer      | Compile Error (if types incompatible)     |

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: Why is C++ named casting preferred over C-style casting `(Type)expression`?**
  - _Answer:_
    1. **Clarity:** Explicitly identifies developer intent (`static_cast` vs `const_cast` vs `reinterpret_cast`).
    2. **Compiler Safety:** C-style casting silently falls back through `const_cast`, `static_cast`, and `reinterpret_cast`, masking dangerous unintended conversions.
    3. **Searchability:** Named casts (`static_cast<...>()`) are trivial to locate using search tools (`grep`) across large enterprise codebases.

- **Q2: Can `dynamic_cast` downcast in a non-polymorphic class hierarchy?**
  - _Answer:_ **No.** The compiler will raise a compile-time error (`source type is not polymorphic`). Dynamic casting relies entirely on querying the `type_info` pointer embedded inside the `VTABLE`.

- **Q3: What happens if you call `typeid` on an uninitialized pointer vs a dereferenced null pointer?**
  - _Answer:_
    - `typeid(ptr)`: Evaluates the static type of the pointer itself (e.g., `Base*`), which is safe.
    - `typeid(*ptr)` (where `ptr == nullptr` and points to a polymorphic class): The runtime throws a **`std::bad_typeid`** exception.

- **Q4: What is the purpose of `uintptr_t` with `reinterpret_cast`?**
  - _Answer:_ Defined in `<cstdint>`, `uintptr_t` is an unsigned integer type guaranteed to be wide enough to hold a machine pointer without truncation. Converting a pointer to `uintptr_t` via `reinterpret_cast` allows safe bitwise address math before casting back to a pointer.

- **Q5: Can `static_cast` remove `const` attributes?**
  - _Answer:_ **No.** Only **`const_cast`** is legally permitted to alter `const` or `volatile` qualifiers. Attempting to strip `const` via `static_cast` produces a compile error.
