# Module 04: Inheritance & Polymorphism

## Unit 4.2: Function Overloading, Overriding, and Name Hiding

### 1. Function Overloading

**Function Overloading** is a form of **Compile-Time (Static) Polymorphism** where multiple functions within the **same scope** share the identical identifier but differ in their parameter signatures.

#### 1.1 Overload Resolution Rules

To overload a function, the compiler must see differences in:

1. **Number of parameters** (`f(int)` vs `f(int, int)`)
2. **Types of parameters** (`f(int)` vs `f(double)`)
3. **Order/Sequence of parameter types** (`f(int, double)` vs `f(double, int)`)

#### 1.2 Overloading Disallowed Criteria

- **Return Type Only:** You cannot overload based solely on return type.
  ```cpp
  int calculate(int x);
  double calculate(int x); // COMPILE ERROR: Cannot overload based on return type alone
  ```
- **Top-Level `const` in Parameters:** Top-level `const` on value parameters is ignored during overload resolution.
  ```cpp
  void run(int x);
  void run(const int x); // COMPILE ERROR: Redefinition of run(int)
  ```
  _(Note: Low-level const on references or pointers `void run(int&)` vs `void run(const int&)` **is** valid)._

---

### 2. Function Overriding

**Function Overriding** is a form of **Runtime (Dynamic) Polymorphism** where a derived class provides a concrete, specialized re-implementation of a function declared in its base class.

#### 2.1 Essential Rules for Overriding

1. **Hierarchy Requirement:** Must occur across **different scopes** related by inheritance (Base vs Derived).
2. **Exact Signature Match:** Function name, parameter types, count, and constness **must be identical**.
3. **The `virtual` Keyword:** The base class function must be marked `virtual`.
4. **Return Types (Covariant Return Types Exception):**
   - Usually, return types must be identical.
   - **Exception (Covariant Returns):** An overriding function can return a pointer or reference to a class derived from the base class return type.
     ```cpp
     class Base {
     public:
         virtual Base* clone() { return new Base(); }
     };
     class Derived : public Base {
     public:
         Derived* clone() override { return new Derived(); } // Legal Covariant Return
     };
     ```
5. **Modern C++ `override` & `final` Specifiers (C++11):**
   - `override`: Directs the compiler to verify that the method actually overrides a base virtual method. Throws a compile error if signatures differ.
   - `final`: Prevents a virtual function from being overridden further by down-chain derived classes.

---

### 3. Overloading vs Overriding: The Master Comparison

| Feature / Dimension     | Function Overloading                      | Function Overriding                                                          |
| :---------------------- | :---------------------------------------- | :--------------------------------------------------------------------------- |
| **Polymorphism Form**   | **Compile-Time** (Static / Early Binding) | **Run-Time** (Dynamic / Late Binding)                                        |
| **Scope**               | Occurs within the **same class / scope**  | Occurs across **different scopes** (Base and Derived)                        |
| **Function Signatures** | **Must be different** (parameters differ) | **Must be identical** (same parameters & constness)                          |
| **Return Type**         | Can be identical or different             | Must be identical (except **Covariant Returns**)                             |
| **Keyword Requirement** | No keywords required                      | Requires **`virtual`** in the Base class                                     |
| **Dispatch Mechanism**  | Name mangling resolved at compile time    | Indirect dispatch through **`vptr` and `VTABLE`**                            |
| **Access Control**      | Can have different access specifiers      | Can alter access specifier in Derived (e.g., base private -> derived public) |

---

### 4. Name Hiding (Shadowing across Scope)

In C++, function lookup happens **by name first**, before overload resolution evaluates parameter types.

> ⚠️ **The Hiding Rule (CMCE High-Yield Trap):**  
> If a derived class declares a function with the **same name** as a function in the base class, it **hides all overloads** of that name from the base class, regardless of signature differences!

#### 4.1 Name Hiding Demonstration

```cpp
#include <iostream>

class Base {
public:
    void display() { std::cout << "Base::display()\n"; }
    void display(int a) { std::cout << "Base::display(int): " << a << "\n"; }
};

class Derived : public Base {
public:
    // This declaration hides BOTH display() and display(int) from Base!
    void display(std::string msg) {
        std::cout << "Derived::display(string): " << msg << "\n";
    }
};

int main() {
    Derived d;
    d.display("Hello"); // OK: Calls Derived::display(string)

    // d.display();     // COMPILE ERROR: Base::display() is hidden!
    // d.display(10);   // COMPILE ERROR: Base::display(int) is hidden!

    // Workaround 1: Explicit Scope Resolution
    d.Base::display(10); // OK: Explicitly queries Base scope
    return 0;
}
```

#### 4.2 Unhiding using the `using` Directive

To bring hidden base class overloads back into the derived class's overload set, apply the `using` declaration:

```cpp
class Derived : public Base {
public:
    // Bring all Base::display overloads into Derived's scope
    using Base::display;

    void display(std::string msg) {
        std::cout << "Derived::display(string): " << msg << "\n";
    }
};

int main() {
    Derived d;
    d.display();        // OK: Base::display() unhidden!
    d.display(42);      // OK: Base::display(int) unhidden!
    d.display("CDAC");  // OK: Derived overload
    return 0;
}
```

---

### 5. Compile-Time Binding (Early Binding / Static Dispatch)

- **Definition:** The compiler determines the exact address of the target function to call during compilation and hard-codes that immediate jump address into the resulting binary assembly.
- **When does it occur?**
  - All non-virtual member functions.
  - Standalone global functions.
  - Overloaded functions / operator overloading.
  - Virtual functions called directly via an **object instance** (`Obj.foo()`), rather than via a pointer or reference.
- **Advantage:** Maximum execution performance with zero runtime lookup penalty; compiler can easily inline the code.

---

### 6. Ambiguity Resolution in Function Calls

Ambiguity occurs when the compiler identifies multiple potential candidates in an overload set but cannot determine a unique, highest-ranking conversion sequence according to standard ranking rules.

```
                  Standard Conversion Ranking Tiers
+-------------------------------------------------------------------+
| 1. Exact Match        : No conversion / Trivial conversions       |
|                         (e.g., array to pointer, lvalue to rvalue)|
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| 2. Promotion          : Integral promotions (bool/char/short -> int)
|                         Floating-point promotions (float -> double)
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| 3. Standard Conversion: Numeric conversions (int -> double,       |
|                         double -> int, derived* to base*)         |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| 4. User-Defined Conv. : Converting constructors / cast operators  |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| 5. Ellipsis Match     : Catch-all variadic (...)                  |
+-------------------------------------------------------------------+
```

#### Common Ambiguity Examples (CMCE Objective Favorites):

1. **Numeric Promotion Ambiguity:**

   ```cpp
   void print(float f);
   void print(double d);

   print(5); // COMPILE ERROR: Ambiguous call!
             // int 5 converts to both float and double with equal ranking (Tier 3 Standard Conversion).
   ```

2. **Default Argument Ambiguity:**

   ```cpp
   void calculate(int x);
   void calculate(int x, int y = 0);

   calculate(10); // COMPILE ERROR: Ambiguous call!
   ```

3. **Reference vs Value Ambiguity:**

   ```cpp
   void step(int x);
   void step(int& x);

   int a = 5;
   step(a); // COMPILE ERROR: Ambiguous call! Compiler cannot decide between copy and alias.
   ```

---

### 7. Inheritance-Related Function Resolution Steps

When the compiler resolves a function call `obj.func(args)` or `ptr->func(args)`, it executes three deterministic phases in order:

```
  Step 1: Name Lookup
  Search begins in the target static scope.
  Does the name exist?
        |
   YES  v            NO
  +----------+    -------> Move up to enclosing Base Class scope
  | Found!   |             (If reached global scope without finding: ERROR)
  +----------+
        |
        v
  Step 2: Overload Resolution
  Search STOPPING immediately upon finding the name in that scope!
  All functions with that name in that SINGLE scope form the candidate set.
  Select best candidate matching parameters.
        |
        v
  Step 3: Access Validation & Binding
  Is the chosen function accessible (public/protected/private)?
  Is it virtual?
    -> YES (and called via ptr/ref) : Dynamic Binding (via VTABLE)
    -> NO                           : Static Binding (Call address hard-coded)
```

> **The Separation of Access and Resolution:**  
> Access control (`public` vs `private`) is checked **after** overload resolution. If a `private` function is determined to be the best match, compilation fails with an access error; the compiler will **not** silently fall back to an accessible `public` function with a worse signature!

---

### 8. High-Yield CMCE Traps & Interview Questions

- **Q1: Can virtual functions be overloaded in the same class?**
  - _Answer:_ **Yes.** A class can have multiple virtual functions sharing the same name provided their parameter signatures differ (`virtual void log(int);` and `virtual void log(double);`).
- **Q2: What is the difference between Function Overriding and Function Hiding?**
  - _Answer:_
    - **Overriding** occurs when a derived class re-implements a `virtual` base function with the exact same signature, enabling dynamic runtime dispatch via `vptr`/`VTABLE`.
    - **Hiding** occurs when a derived class declares a function with the same name as a base function (virtual or non-virtual) with different or matching parameters without dynamic intention, completely masking base function overloads from standard lookup.
- **Q3: What is the output of this snippet?**
  ```cpp
  #include <iostream>
  class Base {
  public:
      virtual void print(int x = 10) { std::cout << "Base: " << x << "\n"; }
  };
  class Derived : public Base {
  public:
      void print(int x = 20) override { std::cout << "Derived: " << x << "\n"; }
  };
  int main() {
      Base* b = new Derived();
      b->print();
      return 0;
  }
  ```

  - _Output:_ `Derived: 10`
  - _Explanation (CLASSIC INTERVIEW TRAP):_ Virtual functions are bound **dynamically at runtime**, but default arguments are bound **statically at compile-time** based on the static type of the pointer (`Base*`). The derived implementation runs, but it receives the base class's default argument value (`10`)!
- **Q4: Can a static member function override a virtual function?**
  - _Answer:_ **No.** A static member function cannot be virtual, nor can it override a base virtual function, because static member functions lack an implicit `this` pointer and do not participate in `vptr`/`VTABLE` dispatch.
- **Q5: Can you change the access specifier of an overridden function in the derived class?**
  - _Answer:_ **Yes.** A function declared `private` in the base class can be overridden and exposed as `public` in the derived class, or vice versa. If called through a `Base*` pointer, the base class access specifiers govern compile-time checks, even though the derived implementation executes at runtime.
