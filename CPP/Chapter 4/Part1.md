# Module 04: Inheritance & Polymorphism

## Unit 4.1: Inheritance

### 1. Introduction to Inheritance

**Inheritance** is an object-oriented mechanism that allows a new class (the **Derived Class** or Child/Subclass) to inherit attributes and behaviors from an existing class (the **Base Class** or Parent/Superclass).

- **Core Purpose:** Promotes **software reusability**, extensibility, and establishes hierarchical relationships.
- **The "IS-A" Relationship:** Inheritance models an "IS-A" relationship (e.g., a `Dog` _is a_ `Mammal`, a `Manager` _is an_ `Employee`).
  - Contrast with Composition ("HAS-A" relationship: a `Car` _has an_ `Engine`), which should generally be preferred over inheritance unless polymorphism or interface conformity is strictly required.

```
       +------------------------------------+
       |       Base Class (Parent)          |
       |  - Common Attributes (e.g., id)    |
       |  + Common Behaviors  (e.g., login) |
       +------------------------------------+
                         ^
                         |  Inherits ("IS-A")
       +------------------------------------+
       |      Derived Class (Child)         |
       |  + Specialized Attributes          |
       |  + Specialized Behaviors           |
       +------------------------------------+
```

---

### 2. Base Class and Derived Class: Syntax

```cpp
class Base {
    // Base class members
};

// Derived class syntax:
class Derived : access_specifier Base {
    // Derived class members
};
```

- **`access_specifier` (Derivation Mode):** Can be `public`, `protected`, or `private`.
- **Default Derivation Mode:**
  - In a `class`, default inheritance is **`private`**.
  - In a `struct`, default inheritance is **`public`**.

---

### 3. Access Modes in Inheritance

The derivation access specifier dictates how the accessibility of the base class members transforms inside the derived class.

```
Base Class Access           Inheritance Mode           Effective Access in Derived
+-------------------+       +-------------------+      +-------------------------+
| public member     | ----> | public            | ---> | public                  |
| protected member  | ----> | public            | ---> | protected               |
| private member    | ----> | public            | ---> | Inaccessible / Hidden   |
+-------------------+       +-------------------+      +-------------------------+
| public member     | ----> | protected         | ---> | protected               |
| protected member  | ----> | protected         | ---> | protected               |
| private member    | ----> | protected         | ---> | Inaccessible / Hidden   |
+-------------------+       +-------------------+      +-------------------------+
| public member     | ----> | private           | ---> | private                 |
| protected member  | ----> | private           | ---> | private                 |
| private member    | ----> | private           | ---> | Inaccessible / Hidden   |
+-------------------+       +-------------------+      +-------------------------+
```

#### The Master Derivation Matrix (MEMORIZE FOR CMCE):

| Base Class Member Specifier | Inherited `public`        | Inherited `protected`     | Inherited `private`       |
| :-------------------------- | :------------------------ | :------------------------ | :------------------------ |
| **`public`**                | **`public`**              | **`protected`**           | **`private`**             |
| **`protected`**             | **`protected`**           | **`protected`**           | **`private`**             |
| **`private`**               | **Hidden / Inaccessible** | **Hidden / Inaccessible** | **Hidden / Inaccessible** |

> ⚠️ **Key Invariants:**
>
> 1. **Private members of a base class are NEVER directly accessible** in a derived class, regardless of the inheritance mode used! They can only be accessed indirectly via inherited public/protected member functions of the base class.
> 2. Derivation mode can only **restrict** access; it can never expand or elevate member visibility.

---

### 4. Forms of Inheritance

```
1. Single            2. Multilevel         3. Multiple
   [ A ]                [ A ]               [ A ]   [ B ]
     |                    |                   \       /
     v                    v                    \     /
   [ B ]                [ B ]                   v   v
                          |                     [ C ]
                          v
                        [ C ]

4. Hierarchical      5. Hybrid (e.g., Diamond)
       [ A ]                    [ A ]
      /     \                  /     \
     v       v                v       v
   [ B ]   [ C ]            [ B ]   [ C ]
                              \       /
                               v     v
                                [ D ]
```

#### 4.1 Single Inheritance

A derived class inherits directly from only one base class.

```cpp
class Animal { public: void eat(); };
class Dog : public Animal { public: void bark(); };
```

#### 4.2 Multilevel Inheritance

A class is derived from a class that is already derived from another class (inheritance chain).

```cpp
class Grandparent { /* ... */ };
class Parent : public Grandparent { /* ... */ };
class Child : public Parent { /* ... */ };
```

#### 4.3 Multiple Inheritance

A derived class inherits directly from **two or more base classes**.

```cpp
class Printer { public: void print(); };
class Scanner { public: void scan(); };
class MultiFunctionDevice : public Printer, public Scanner { /* ... */ };
```

##### Ambiguity in Multiple Inheritance:

If both base classes contain a member with the identical signature:

```cpp
class A { public: void show() {} };
class B { public: void show() {} };
class C : public A, public B {};

int main() {
    C obj;
    // obj.show(); // COMPILE ERROR: Request for member 'show' is ambiguous!
    obj.A::show(); // RESOLVED: Using scope resolution operator
    obj.B::show(); // RESOLVED
}
```

#### 4.4 Hierarchical Inheritance

Multiple derived classes inherit from a single common base class.

```cpp
class Shape { /* ... */ };
class Circle : public Shape { /* ... */ };
class Rectangle : public Shape { /* ... */ };
```

#### 4.5 Hybrid Inheritance & The Diamond Problem

A combination of two or more forms of inheritance (e.g., hierarchical + multiple).

```
         [ Person ]            <-- Base Class (e.g., has 'int id;')
        /          \
       /            \
  [ Student ]      [ Employee ] <-- Both inherit 'Person'
       \            /
        \          /
       [ TeachingAssistant ]   <-- Inherits both: Contains TWO COPIES of 'Person'!
```

##### The Problem:

`TeachingAssistant` receives two copies of `Person` (one through `Student`, one through `Employee`), causing member access ambiguity (`ta.id` is ambiguous) and memory bloat.

##### The Solution: Virtual Base Classes

Mark the intermediate derived classes with the **`virtual`** keyword during inheritance:

```cpp
class Person {
public:
    int id;
};

// Use 'virtual public' or 'public virtual'
class Student : virtual public Person {};
class Employee : virtual public Person {};

// TA receives only ONE shared instance of Person!
class TeachingAssistant : public Student, public Employee {};

int main() {
    TeachingAssistant ta;
    ta.id = 101; // AMBIGUITY RESOLVED: Only one shared Person subobject exists!
    return 0;
}
```

---

### 5. Constructor and Destructor Execution Order

Inheritance follows a deterministic order for object construction and destruction.

```
Construction (Top-Down):
Base Constructor ===> Derived Constructor Body

Destruction (Bottom-Up):
Derived Destructor Body ===> Base Destructor
```

#### 5.1 Single / Multilevel Order

- **Constructors:** Base classes are constructed first, progressing downward through the hierarchy.
- **Destructors:** Objects are destroyed in the exact reverse order (most derived first).

```cpp
#include <iostream>

class A {
public:
    A()  { std::cout << "A_ctor "; }
    ~A() { std::cout << "A_dtor "; }
};

class B : public A {
public:
    B()  { std::cout << "B_ctor "; }
    ~B() { std::cout << "B_dtor "; }
};

class C : public B {
public:
    C()  { std::cout << "C_ctor "; }
    ~C() { std::cout << "C_dtor "; }
};

int main() {
    C obj;
    return 0;
}
```

- **Output:** `A_ctor B_ctor C_ctor C_dtor B_dtor A_dtor`

#### 5.2 Multiple Inheritance Execution Order Rules

In multiple inheritance, the order of constructor invocation is strictly determined by the **order in which base classes appear in the class declaration list**, **NOT** the order written in the derived constructor initialization list!

```cpp
class Base1 { public: Base1() { std::cout << "B1 "; } };
class Base2 { public: Base2() { std::cout << "B2 "; } };

// Declared: Base2 FIRST, then Base1:
class Derived : public Base2, public Base1 {
public:
    // Even though Base1 is listed first in MIL, Base2 executes first!
    Derived() : Base1(), Base2() { std::cout << "D "; }
};

int main() {
    Derived d;
    return 0;
}
```

- **Output:** `B2 B1 D`

#### 5.3 Virtual Base Class Constructor Rule

If a hierarchy involves **virtual base classes**:

- Virtual base constructors are executed **before any non-virtual base constructors**, regardless of where they appear in the hierarchy.
- The **most derived class** (`TeachingAssistant`) is responsible for directly invoking the virtual base class constructor (`Person()`), bypassing intermediate classes (`Student`, `Employee`).

---

### 6. Passing Arguments to Base Class Constructors

When a base class has parameterized constructors and no default constructor, the derived class constructor **must** explicitly route arguments to the base constructor using the **Member Initializer List (MIL)**:

```cpp
class Base {
protected:
    int baseVal;
public:
    Base(int v) : baseVal(v) {} // No default constructor!
};

class Derived : public Base {
private:
    int derivedVal;
public:
    // Forwarding argument to Base constructor:
    Derived(int b, int d) : Base(b), derivedVal(d) {}
};
```

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: Can a derived class inherit base class constructors and destructors?**
  - _Answer:_ Prior to C++11, constructors and destructors were **never inherited**. In C++11 and later, you can inherit base constructors using the `using` declaration (`using Base::Base;`). Destructors are **never** inherited under any standard.

- **Q2: What is the size of an object inheriting from an empty base class?**
  - _Answer:_ Under **Empty Base Optimization (EBO)**, an empty base class occupies 0 bytes inside the derived class instead of the standard 1 byte, provided the empty base is not the first non-static data member of the same type.
    ```cpp
    class Empty {};
    class Derived : public Empty { int x; }; // sizeof(Derived) == 4 (NOT 5 or 8!)
    ```

- **Q3: What happens if you derive privately (`class D : private Base`)?**
  - _Answer:_ All public and protected members of `Base` become `private` inside `D`. Consequently:
    1. Clients of `D` cannot call base public functions (`d.baseFunc()` fails).
    2. Any class deriving from `D` cannot access those base members either.
    3. The compiler disallows implicit upcasting: `Base* ptr = new D();` will fail with a compile error ("conversion to inaccessible base class").

- **Q4: Why does Multiple Inheritance with Virtual Base Classes introduce memory overhead?**
  - _Answer:_ When virtual inheritance is used, the compiler inserts an internal pointer (commonly called **`vbptr`** or Virtual Base Pointer, or offsets into the VTABLE) inside each derived instance. This allows the runtime to dynamically calculate the memory offset to the single shared virtual base instance, increasing the object's `sizeof` footprint.

- **Q5: Can friend functions be inherited?**
  - _Answer:_ **No.** Friendship is strictly non-inherited, non-transitive, and non-mutual. If function `F` is a friend of `Base`, `F` has no special access privileges to private or protected members of `Derived`.
