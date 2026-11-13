# Module 03: Object-Oriented Programming (Core Concepts)

## Unit 3.2: Class Components

Class components form the building blocks of any class definition in C++. They dictate visibility, state encapsulation, behavioral contracts, memory allocation models, and compile-time const correctness.

---

### 1. Access Specifiers

Access specifiers enforce the boundary of **data hiding** and determine from which scopes class members can be accessed.

```
+-----------------------------------------------------------------+
|                          CLASS SCOPE                            |
|                                                                 |
|  [ private ]   --> Accessible ONLY within this class & friends  |
|         ^                                                       |
|  [ protected ] --> Accessible within class, friends & DERIVED   |
|         ^                                                       |
|  [ public ]    --> Accessible from ANYWHERE (World/Clients)     |
+-----------------------------------------------------------------+
```

#### Access Specifier Visibility Matrix:

| Access Specifier | Same Class | Friend Function / Class | Derived Class (Child) | External Code (World) |
| :--------------- | :--------: | :---------------------: | :-------------------: | :-------------------: |
| **`private`**    |    Yes     |           Yes           |         ❌ No         |         ❌ No         |
| **`protected`**  |    Yes     |           Yes           |          Yes          |         ❌ No         |
| **`public`**     |    Yes     |           Yes           |          Yes          |          Yes          |

- **Class vs. Struct Default:**
  - In a `class`, default access for members and base inheritance is **`private`**.
  - In a `struct`, default access for members and base inheritance is **`public`**.
- Access labels can appear multiple times and in any arbitrary order within the class body.

---

### 2. Data Members

Data members define the **state** or attributes of an object.

- **Non-Static Data Members (Instance Variables):**
  - Allocated per object instance inside stack, heap, or global memory.
  - Contribute directly to `sizeof(ClassName)` subject to data alignment and struct padding.
- **In-Class Member Initializers (C++11):**
  ```cpp
  class Account {
      int id = 0;              // Default member initialization
      double balance{100.0};   // Uniform brace initialization
  };
  ```
- **Bit-Fields:** Used in systems/embedded programming to pack data into specific bit widths:
  ```cpp
  struct PackedFlags {
      unsigned int isReady : 1; // 1 bit
      unsigned int mode    : 3; // 3 bits (0-7)
  };
  ```

---

### 3. Member Functions

Member functions define the **behavior** or operations of an object.

#### 3.1 Inside vs. Outside Definition

- **Inside Class Definition:** Automatically treated as an `inline` candidate by the compiler.
- **Outside Class Definition:** Requires the **Scope Resolution Operator (`::`)**:

```cpp
class MathOps {
public:
    int square(int x) { return x * x; } // Implicitly inline
    int cube(int x);                    // Prototype only
};

// Defined outside class scope
int MathOps::cube(int x) {
    return x * x * x;
}
```

---

### 4. Static Data Members

A static data member is a class-level variable shared by **all instances** of that class.

```
       [ Object 1 ]            [ Object 2 ]            [ Object 3 ]
       (non-static)            (non-static)            (non-static)
             \                       |                       /
              \                      |                      /
               v                     v                     v
         +-----------------------------------------------------+
         |      Static Data Member: Account::interestRate      |
         |                 (Stored in .data / .bss)            |
         +-----------------------------------------------------+
```

#### 4.1 Key Characteristics & Lifecycle:

1. **Storage:** Allocated in the **Data Segment / BSS Segment**, not inside object memory. It does **not** contribute to `sizeof(Object)`.
2. **Lifetime:** Entire program runtime (created at program start, destroyed at exit).
3. **Declaration vs. Definition:** Declared inside the class; **must be explicitly defined outside** in global scope (allocates memory).

```cpp
// Account.h / Class declaration
class Account {
private:
    int accNo;                         // Non-static: per-instance
public:
    static double interestRate;        // Declaration only
};

// Account.cpp / File scope
double Account::interestRate = 0.05;   // Mandatory definition and memory allocation!

int main() {
    // Accessible via class name directly without instantiating objects
    Account::interestRate = 0.07;
    return 0;
}
```

- _Modern C++ Note (C++17):_ `inline static double interestRate = 0.05;` allows in-class definition without needing a separate `.cpp` definition.

---

### 5. Static Member Functions

Static member functions operate at the **class level**, independent of any specific object instance.

#### 5.1 Rules for Static Member Functions:

1. **No `this` Pointer:** Static member functions have no instance context; hence, the `this` pointer does not exist within them.
2. **Restricted Member Access:**
   - Can **only** directly access static data members and call other static member functions.
   - **Cannot** access non-static data members or call non-static member functions directly (unless passed an explicit object reference/pointer).
3. **Cannot be marked `const`, `volatile`, or `virtual`:**
   - `const` applies to `*this`, which static functions lack.
   - `virtual` relies on dynamic dispatch via instance `vptr`.

```cpp
class ServerPool {
private:
    int serverId;                      // Non-static
    static int activeConnections;      // Static

public:
    static int getActiveConnections() {
        // serverId = 10;              // COMPILE ERROR: Cannot access non-static member!
        return activeConnections;      // OK
    }
};

int ServerPool::activeConnections = 0;
```

---

### 6. Friend Functions and Friend Classes

A `friend` function is a non-member function granted access to the `private` and `protected` members of a class.

```cpp
class Box {
private:
    double width;

public:
    Box(double w) : width(w) {}

    // Friend declaration (can be in private or public section without change in meaning)
    friend void printWidth(const Box& b);
    friend class BoxInspector; // Entire class can access private members
};

// Standalone function definition (Notice: NO Box:: scope qualifier)
void printWidth(const Box& b) {
    std::cout << "Width: " << b.width << "\n"; // Legal access to private field
}
```

#### 6.1 Essential Rules of Friendship (CMCE High-Frequency Topic):

1. **Friendship is NOT Mutual (Asymmetric):** If Class `A` is a friend of Class `B`, `B` is not automatically a friend of `A`.
2. **Friendship is NOT Transitive:** If `A` is a friend of `B`, and `B` is a friend of `C`, `A` is not automatically a friend of `C`.
3. **Friendship is NOT Inherited:** Derived classes do not inherit friend status from their base classes.

---

### 7. The `this` Pointer

The `this` pointer is an **implicit, constant pointer** passed as an invisible first argument to all non-static member functions. It points to the object on which the function was invoked.

```
Call:  obj.deposit(500.0);
Transforms to compiler equivalent:
       BankAccount::deposit(&obj, 500.0);
```

#### 7.1 Type Signature of `this`:

- In a regular member function of `ClassType`:
  $$\text{ClassType* const this}$$
  _(A const pointer to non-const object — cannot change address, can modify members)._
- In a `const` member function of `ClassType`:
  $$\text{const ClassType* const this}$$
  _(A const pointer to a const object — cannot change address, cannot modify members)._

#### 7.2 Primary Use Cases:

1. **Disambiguating Shadowed Member Variables:**
   ```cpp
   class Node {
       int data;
   public:
       void setData(int data) {
           this->data = data; // Disambiguates parameter 'data' from member 'data'
       }
   };
   ```
2. **Method Chaining (Fluent Interface):** Returning `*this` by reference enables chained calls:

   ```cpp
   class Builder {
   public:
       Builder& setStepA() { /* work */ return *this; }
       Builder& setStepB() { /* work */ return *this; }
   };

   // Invocation:
   Builder b;
   b.setStepA().setStepB();
   ```

---

### 8. `const` Member Functions & `mutable`

A `const` member function promises **not to modify the state** of the object on which it is invoked.

#### 8.1 Syntax & Const-Correctness Rules:

```cpp
class Tracker {
private:
    int count;
    mutable int accessLogCount{0}; // mutable: exempt from const constraints

public:
    Tracker(int c) : count(c) {}

    // Const member function
    int getCount() const {
        // count++;                // COMPILE ERROR: Attempting mutation in const function
        accessLogCount++;          // OK: 'mutable' members can be changed
        return count;
    }
};
```

#### 8.2 Invocability Matrix:

| Calling Object Type                   | Calling Non-const Function (`void foo();`) | Calling Const Function (`void foo() const;`) |
| :------------------------------------ | :----------------------------------------: | :------------------------------------------: |
| **Non-Const Object** (`Tracker t;`)   |                    Yes                     |                     Yes                      |
| **Const Object** (`const Tracker t;`) |            ❌ **COMPILE ERROR**            |                     Yes                      |

- **Const Overloading:** You can overload a member function solely based on `const`:
  ```cpp
  class Array {
  public:
      int& operator[](size_t i);       // For non-const instances (returns modifiable ref)
      const int& operator[](size_t i) const; // For const instances (read-only ref)
  };
  ```

---

### 9. High-Yield CMCE Traps & Interview Questions

- **Q1: Can a `static` member function be declared `virtual` or `const`?**
  - _Answer:_ **No.** Virtual dispatch requires a `vptr` found on an object instance, and `const` qualifies the implicit `this` pointer. Since static methods have no instance context and no `this` pointer, both qualifiers are illegal.

- **Q2: Does declaring a `friend` function in the `private` section hide it from callers?**
  - _Answer:_ **No.** The location of a `friend` declaration inside `private`, `protected`, or `public` sections has no effect. A friend function is not a member of the class, so class access specifiers do not apply to it.

- **Q3: What is the output of the following snippet?**

  ```cpp
  #include <iostream>
  class Test {
      static int val;
  public:
      void show() { std::cout << ++val; }
  };
  int Test::val = 10;
  int main() {
      Test t1, t2;
      t1.show();
      t2.show();
      return 0;
  }
  ```

  - _Output:_ `1112`
  - _Explanation:_ `val` is static and shared between `t1` and `t2`. `t1.show()` increments `val` to 11 and prints it. `t2.show()` increments the same shared variable to 12 and prints it.

- **Q4: What happens if you forget to define a `static` data member outside the class?**
  - _Answer:_ The code compiles during Phase 2 (syntactic/assembly compilation) because the member is declared, but fails in Phase 4 during **linking** with an `undefined reference to ClassName::member` error.

- **Q5: Can `delete this` ever be executed safely inside a member function?**
  - _Answer:_ Yes, but **only** under strict conditions:
    1. The object must have been allocated on the heap via `new` (never stack or static).
    2. The member function must not access any member variables or call other member functions after `delete this;`.
    3. The caller must never access that object instance again.
