# Module 06: Templates & Generic Programming

## Unit 6.2: Advanced Templates

Advanced templates enable fine-grained compile-time specialization, type-specific algorithmic tuning, and template metaprogramming patterns.

---

### 1. Multiple Template Parameters

Templates are not restricted to a single placeholder. They can accept multiple type and non-type parameters simultaneously.

#### 1.1 Mixed Parameter Categories

```cpp
#include <iostream>
#include <string>

// Two types (K, V) and one non-type parameter (Capacity)
template <typename K, typename V, size_t Capacity = 64>
class KeyValueStore {
private:
    K keys[Capacity];
    V values[Capacity];
    size_t count = 0;

public:
    void insert(const K& k, const V& v) {
        if (count < Capacity) {
            keys[count] = k;
            values[count] = v;
            count++;
        }
    }

    void display() const {
        for (size_t i = 0; i < count; ++i) {
            std::cout << keys[i] << " => " << values[i] << "\n";
        }
    }
};

int main() {
    // Explicitly defining all types and sizes
    KeyValueStore<int, std::string, 10> cache;
    cache.insert(101, "Server_Alpha");

    // Using default non-type capacity (64)
    KeyValueStore<std::string, double> priceMap;
    priceMap.insert("USD_INR", 83.25);

    return 0;
}
```

---

### 2. Template Specialization

While generic templates provide unified algorithms, certain data types require custom optimizations or alternate implementations (e.g., handling raw `const char*` differently than primitives, or optimizing bit vectors).

Specialization provides **alternate implementations** for specific type combinations.

```
                         Template Specialization
                                    |
          +-------------------------+-------------------------+
          |                                                   |
Full (Explicit) Specialization                      Partial Specialization
  - ALL template parameters specified                 - SOME template parameters fixed
  - Applicable to: Functions & Classes                - Applicable to: ONLY Classes!
```

---

### 3. Function Template Overloading vs Specialization

#### 3.1 Function Template Overloading

Function templates can be overloaded with different parameter counts or non-template equivalents.

```cpp
#include <iostream>

// (1) Base Generic Template
template <typename T>
void printData(T val) {
    std::cout << "Generic Template: " << val << "\n";
}

// (2) Overloaded Function Template (different parameter structure)
template <typename T>
void printData(T val, int repetitions) {
    for (int i = 0; i < repetitions; ++i) {
        std::cout << val << " ";
    }
    std::cout << "\n";
}

// (3) Non-Template Overload (Exact Type Match)
void printData(int val) {
    std::cout << "Non-template (int): " << val << "\n";
}
```

#### 3.2 Full (Explicit) Function Template Specialization

To specialize a function template for a concrete type, use the `template <>` empty diamond syntax:

```cpp
#include <cstring>

// Base Primary Template
template <typename T>
bool isEqual(T a, T b) {
    return a == b;
}

// Full Specialization for const char* (C-style strings)
template <>
bool isEqual<const char*>(const char* a, const char* b) {
    std::cout << "[Using C-string Specialization] ";
    return std::strcmp(a, b) == 0; // Deep comparison instead of pointer equality!
}

int main() {
    int x = 10, y = 10;
    std::cout << isEqual(x, y) << "\n"; // 1 (calls primary template)

    const char* s1 = "CDAC";
    const char* s2 = "CDAC";
    // Without specialization, isEqual would just compare pointer addresses!
    std::cout << isEqual(s1, s2) << "\n"; // Calls specialization
    return 0;
}
```

> ⚠️ **CRITICAL RULE (Function Specialization vs. Overloading):**  
> **Function templates CANNOT be partially specialized.** Only full (explicit) specialization is allowed for function templates. If you need partial specialization behavior for a function, **overload** the function instead.

---

### 4. Class Template Specialization

Class templates support **both** Full Specialization and Partial Specialization.

#### 4.1 Full Class Template Specialization

Every template parameter is mapped to a concrete data type:

```cpp
#include <iostream>

// 1. Primary Generic Template
template <typename T>
class Formatter {
public:
    void print(T val) {
        std::cout << "Generic: " << val << "\n";
    }
};

// 2. Full Specialization for bool
template <>
class Formatter<bool> {
public:
    void print(bool val) {
        // Custom representation for booleans
        std::cout << "Bool Specialized: " << (val ? "TRUE" : "FALSE") << "\n";
    }
};

int main() {
    Formatter<double> fDouble;
    fDouble.print(3.14); // Generic: 3.14

    Formatter<bool> fBool;
    fBool.print(true);   // Bool Specialized: TRUE
    return 0;
}
```

#### 4.2 Partial Class Template Specialization

Only a subset of parameters is specialized, or parameters are narrowed to a specific category (e.g., pointer types `T*`, reference types `T&`, or arrays `T[]`).

```cpp
#include <iostream>

// Primary Template (Two types: T1, T2)
template <typename T1, typename T2>
class PairManager {
public:
    void identify() { std::cout << "Primary: Different generic types\n"; }
};

// Case A: Partial Specialization (When both types are identical)
template <typename T>
class PairManager<T, T> {
public:
    void identify() { std::cout << "Partially Specialized: Identical types (T, T)\n"; }
};

// Case B: Partial Specialization (Second type fixed to int)
template <typename T>
class PairManager<T, int> {
public:
    void identify() { std::cout << "Partially Specialized: Second type is fixed to int\n"; }
};

// Case C: Partial Specialization for Pointers (T1*, T2*)
template <typename T1, typename T2>
class PairManager<T1*, T2*> {
public:
    void identify() { std::cout << "Partially Specialized: Both are POINTER types\n"; }
};

int main() {
    PairManager<double, std::string> p1; // Primary
    p1.identify();

    PairManager<int, int> p2;            // Case A
    p2.identify();

    PairManager<std::string, int> p3;    // Case B
    p3.identify();

    PairManager<int*, char*> p4;         // Case C
    p4.identify();

    return 0;
}
```

#### 4.3 Classic Real-World Example: `std::vector<bool>`

In the C++ Standard Library, `std::vector<bool>` is a famous **full specialization** of the `std::vector<T>` class template. Instead of allocating 1 byte per boolean, it packs bits such that each boolean occupies **1 bit** of memory (8 booleans per byte), optimizing space at the cost of returning proxy reference objects instead of genuine `bool&`.

---

### 5. Default Template Arguments

Default template arguments provide fallback types or constant values when client code omits them during instantiation.

#### 5.1 Rules for Default Arguments

1. **Right-to-Left Rule (Classes):** Default template arguments for class templates must be specified from right to left, matching standard function parameter defaults.
2. **Function Templates Exception (Modern C++):** Unlike class templates, function templates allow default arguments anywhere in the list if the trailing types can be deduced from call arguments:
   ```cpp
   template <typename ReturnType = double, typename T>
   ReturnType convert(T val) {
       return static_cast<ReturnType>(val);
   }
   ```
3. **Empty Angle Brackets `<>`:** If all template parameters rely on default arguments, empty brackets `<>` are required when declaring instances of class templates:

   ```cpp
   template <typename T = int, size_t Size = 100>
   class Buffer {};

   Buffer<> bDefault;         // Buffer<int, 100>
   Buffer<char> bCustomType;  // Buffer<char, 100>
   Buffer<char, 256> bCustomAll; // Buffer<char, 256>
   ```

---

### 6. Summary Comparison: Overloading vs Specialization

| Metric                       | Non-Template Overload               | Function Template Specialization        | Class Template Specialization               |
| :--------------------------- | :---------------------------------- | :-------------------------------------- | :------------------------------------------ |
| **Applicable Target**        | Functions                           | Functions                               | Classes                                     |
| **Partial Variant Allowed?** | N/A (Handled via normal overloads)  | ❌ **NO** (Compile error)               | ✅ **YES** (`class Template<T, int>`)       |
| **Syntax Format**            | Standard function signature         | `template <> void func<Type>()`         | `template <> class Class<Type>`             |
| **Resolution Priority**      | **Highest preference** (runs first) | Lower (primary template selected first) | Matches most specific pattern               |
| **Can vary class layout?**   | N/A                                 | No                                      | ✅ **YES** (Can have different member vars) |

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: Can we partially specialize a function template in C++?**
  - _Answer:_ **No.** The C++ standard explicitly disallows partial specialization for function templates (`template <typename T> void foo(T, int);` will not compile if attempting specialization syntax). Instead, simply use standard **function overloading**.

- **Q2: Does class template specialization inherit members of the primary template?**
  - _Answer:_ **No.** A template specialization is a completely separate definition. It does **not** inherit data members, constructors, or member functions from the primary template. You must redefine the entire class interface for the specialized version.

- **Q3: What will this code print?**

  ```cpp
  #include <iostream>
  template <typename T> void f(T) { std::cout << "1"; }
  template <> void f<int*>(int*) { std::cout << "2"; }
  template <typename T> void f(T*) { std::cout << "3"; }

  int main() {
      int* p = nullptr;
      f(p);
      return 0;
  }
  ```

  - _Output:_ `3`
  - _Explanation:_ Overload resolution occurs among **primary templates first**. `f(T*)` is more specialized than `f(T)`. The specialization `f<int*>` belongs to `f(T)`, not to `f(T*)`. Since `f(T*)` is the winning primary overload, the runtime prints `3`. (This is known as the _Sutter Trap_—prefer overloading over function template specialization).

- **Q4: Why does `&vector<bool>::operator[]` not yield a `bool*`?**
  - _Answer:_ Because `std::vector<bool>` is specialized for bit packing, individual bits are not directly addressable on byte-oriented CPU hardware architectures. It returns a temporary proxy object (`std::vector<bool>::reference`), not a standard `bool&`.

- **Q5: Can non-type template arguments use floating-point types (`float`, `double`)?**
  - _Answer:_ In **C++98 through C++17, NO** (only integral, pointers, and enums were allowed). In **C++20, YES**, structural floating-point types became legal as non-type template parameters.
