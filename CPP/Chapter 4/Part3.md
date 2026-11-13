# Module 04: Inheritance & Polymorphism

## Unit 4.3: Polymorphism

### 1. Introduction to Polymorphism

**Polymorphism** (derived from the Greek _poly_ meaning "many" and _morph_ meaning "form") is the ability of a single interface, function, or operator to behave differently based on the type of object it is acting upon.

In C++, polymorphism enables designing generic, extensible systems where high-level modules interact with base class contracts without needing compile-time knowledge of specific derived types.

```
                            Polymorphism in C++
                                     |
         +---------------------------+---------------------------+
         |                                                       |
Compile-Time Polymorphism                               Run-Time Polymorphism
 (Static / Early Binding)                               (Dynamic / Late Binding)
         |                                                       |
   +-----+-----+                                                 |
   |           |                                                 |
Function    Operator     Templates / Generic               Virtual Functions
Overloading Overloading  Programming                         (via VPTR & VTABLE)
```

---

### 2. Compile-Time Polymorphism (Static Binding)

In **compile-time polymorphism**, the compiler resolves which specific function or operator definition to invoke during compilation, hard-coding that target address directly into the binary assembly.

#### 2.1 Mechanisms:

1. **Function Overloading:** Multiple functions sharing identical identifiers but differing in parameter count, type, or ordering.
2. **Operator Overloading:** Giving user-defined types custom implementations of built-in operators (`+`, `-`, `<<`, `[]`).
3. **Templates (Parametric Polymorphism):** Generating type-specific classes or functions at compile time (e.g., `template <typename T>`).

#### 2.2 Key Characteristics:

- **Zero Runtime Overhead:** No lookup indirection at runtime; calls execute at native speed.
- **Inlining Opportunity:** The compiler can easily inline static calls to eliminate function-call stack frame overhead.
- **Rigidity:** Target types and function prototypes must be completely determinable at compile time.

---

### 3. Runtime Polymorphism (Dynamic Binding)

In **runtime polymorphism**, the decision of which specific method to execute is deferred until program **runtime**, depending entirely on the concrete type of the object being pointed to or referenced.

#### 3.1 Prerequisites for Runtime Polymorphism in C++:

Runtime polymorphism requires all three conditions to be met:

1. An **inheritance relationship** between classes (Base $\rightarrow$ Derived).
2. The function in the Base class must be declared with the **`virtual`** keyword and overridden in the Derived class.
3. The function must be invoked through a **Base class pointer (`Base*`) or Base class reference (`Base&`)**.

```cpp
#include <iostream>

class Shape {
public:
    virtual void draw() const { // Virtual function
        std::cout << "Drawing generic Shape\n";
    }
    virtual ~Shape() = default; // Mandatory virtual destructor
};

class Circle : public Shape {
public:
    void draw() const override {
        std::cout << "Drawing Circle\n";
    }
};

class Square : public Shape {
public:
    void draw() const override {
        std::cout << "Drawing Square\n";
    }
};

void render(const Shape& s) {
    s.draw(); // Dynamic dispatch: Executes target based on actual runtime type
}

int main() {
    Circle c;
    Square sq;
    render(c);  // Output: Drawing Circle
    render(sq); // Output: Drawing Square
    return 0;
}
```

---

### 4. Virtual Functions & Internal Mechanics (`VPTR` and `VTABLE`)

Virtual functions are the engine powering dynamic polymorphism in C++. When a class declares or inherits a virtual function, the compiler inserts hidden low-level data structures.

#### 4.1 What is the `VTABLE` (Virtual Table)?

- A static array of function pointers created by the compiler **per class** (one table for `Base`, one table for `Derived`).
- Stored in the **read-only data segment (`.rodata`)**.
- Contains addresses of all virtual member functions accessible by that class.

#### 4.2 What is the `VPTR` (Virtual Pointer)?

- A hidden member pointer (typically named `_vptr`) inserted by the compiler into **every instance** of a class containing virtual functions.
- Allocated inside the object (usually as the first 4 bytes on 32-bit systems, or 8 bytes on 64-bit systems).
- Points to the class's respective `VTABLE`.
- Initialized inside the constructor during object instantiation.

```
Object in Memory (Derived Instance)               VTABLE for Derived (.rodata)
+-----------------------------------+             +----------------------------------+
| _vptr (8 bytes)                   | ----------> | slot 0: &Derived::draw()         |
+-----------------------------------+             | slot 1: &Derived::~Derived()     |
| derivedMemberData                 |             +----------------------------------+
+-----------------------------------+
```

#### 4.3 Step-by-Step Dynamic Dispatch at Assembly/Runtime Level:

When executing `ptr->draw();`:

1. The CPU fetches the object's `_vptr` stored at `*ptr`.
2. It offsets into the `VTABLE` to find the slot assigned to `draw()` (resolved at compile time as a fixed index, e.g., slot `0`).
3. It fetches the function pointer stored at `VTABLE[0]`.
4. It issues an indirect call: `call *VTABLE[0]`.

---

### 5. Static Binding vs Dynamic Binding

| Feature / Metric      | Static Binding (Early Binding)                            | Dynamic Binding (Late Binding)                              |
| :-------------------- | :-------------------------------------------------------- | :---------------------------------------------------------- |
| **Resolution Time**   | During **compilation** / link-time                        | During **program execution (runtime)**                      |
| **Target Mechanism**  | Direct function call (`call <address>`)                   | Indirect call via **`VPTR` $\rightarrow$ `VTABLE`**         |
| **Keywords Involved** | Default behavior (no special keyword)                     | Requires **`virtual`** keyword in Base class                |
| **Invocation Type**   | Direct calls on objects (`obj.func()`), non-virtual calls | Via Base pointers (`Base*`) or Base references (`Base&`)    |
| **Performance**       | **Faster** (direct jumps, candidate for inlining)         | **Slightly slower** (double pointer dereference overhead)   |
| **Memory Footprint**  | Standard object size (no hidden pointers)                 | Adds **4 to 8 bytes** (`_vptr`) per object + class `VTABLE` |
| **Flexibility**       | Rigid; predetermined at compile time                      | High; allows runtime extensibility, plugins, and frameworks |

---

### 6. Rules and Constraints of Virtual Functions

1. **Constructors Cannot Be Virtual:** Virtual dispatch relies on `VPTR`, which is set up by the constructor. You cannot use a `VPTR` before the constructor completes its setup.
2. **Destructors CAN (and often SHOULD) Be Virtual:** Base classes intended for polymorphic deletion must declare `virtual ~Base();` to prevent incomplete cleanup.
3. **Static Member Functions Cannot Be Virtual:** Static functions are class-scoped, lack a `this` pointer, and do not belong to an instance carrying a `_vptr`.
4. **Inline Functions vs Virtual:** If invoked via a pointer/reference, the virtual call cannot be inlined because the exact target cannot be deduced statically. If called directly via an object (`obj.display()`), the compiler can inline it.
5. **Private Virtual Functions:** Virtual functions can be `private`. Access checks occur at compile time based on the pointer type, but dynamic dispatch runs the derived override at runtime.

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: What is the output of `sizeof` on an object of a class containing only virtual functions?**
  - _Answer:_ On a 64-bit platform, it prints **8 bytes** (4 bytes on 32-bit). Even if the class has no member variables, the compiler inserts the `_vptr` (a pointer to the `VTABLE`), which accounts for 8 bytes.

- **Q2: What happens if you call a virtual function through an object instance directly (e.g., `Base b; b.show();`)?**
  - _Answer:_ The compiler uses **Static Binding**. The exact type is already known (`b` is definitely of type `Base`), so there is no need for `VTABLE` indirection. Dynamic dispatch only engages when using a **pointer or reference**.

- **Q3: What is "Object Slicing"?**
  - _Answer:_ Slicing happens when a derived class object is assigned to a base class object **by value** (`Base b = derivedObj;`). The base copy constructor copies only the base subobject portion, "slicing off" derived-specific members and resetting the `_vptr` to `Base`'s `VTABLE`. Polymorphism is completely lost.

- **Q4: Can a virtual function be overridden with a different access specifier?**
  - _Answer:_ **Yes.** Access control (private/protected/public) is evaluated statically at compile time using the pointer's static type. If `Base::func()` is `public` and `Derived::func()` is `private`, calling `basePtr->func()` will succeed at compile time and dynamically dispatch to the derived private implementation at runtime.

- **Q5: What happens when a virtual function is called inside a constructor or destructor?**
  - _Answer:_ Dynamic dispatch **does not descend** to derived classes. Inside a `Base` constructor or destructor, the object's `_vptr` points to `Base`'s `VTABLE`, so calls resolve to `Base`'s implementation, never `Derived`. This prevents accessing unconstructed or already-destroyed derived members.
