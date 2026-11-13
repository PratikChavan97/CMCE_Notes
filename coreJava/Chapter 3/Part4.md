# Module 03: Object-Oriented Programming (OOP) Deep-Dive

## Unit 3.4: Access Modifiers and Advanced OOP Concepts

### 1. Java Access Control Architecture

Access modifiers establish access boundaries at the package and class level, enforcing encapsulation and protecting internal invariants.

Java defines **four access levels** using **three explicit keywords**:

```
                       Access Privilege Spectrum
 (Most Restrictive)                                 (Most Permissive)
     private  ───►  Default (Package-Private)  ───►  protected  ───►  public
```

```
┌────────────────────────────────────────────────────────────────────────┐
│ Access Scope Visibility Hierarchy                                     │
├───────────────────┬──────────────┬──────────────┬──────────────┬───────┤
│ Context           │   private    │   default    │  protected   │ public│
├───────────────────┼──────────────┼──────────────┼──────────────┼───────┤
│ Same Class        │      ✅      │      ✅      │      ✅      │   ✅  │
│ Same Package Sub  │      ❌      │      ✅      │      ✅      │   ✅  │
│ Same Package Non  │      ❌      │      ✅      │      ✅      │   ✅  │
│ Diff Package Sub  │      ❌      │      ❌      │    ✅ [*]    │   ✅  │
│ Diff Package Non  │      ❌      │      ❌      │      ❌      │   ✅  │
└───────────────────┴──────────────┴──────────────┴──────────────┴───────┘
[*] Accessible strictly through inheritance via the subclass reference handle.
```

---

### 2. Deep-Dive: The Four Access Modifiers

#### 2.1 `private` Access Modifier

- **Scope:** Accessible **only within the declared class body**.
- **Invisible to Subclasses:** Even direct subclasses cannot access private members by name.
- **Class-Level Restriction:** Top-level outer classes cannot be declared `private` (compilation error: _modifier private not allowed here_). Only nested/inner classes can be `private`.

#### 2.2 Default (Package-Private) Access

- **Scope:** Visible to **all classes within the same package**. Completely invisible outside that package.
- **Absence of Keyword:** Specified by providing **no access modifier**.
- **Package Scope:** A class in package `com.cdac.core` cannot access default members of `com.cdac.core.util`, because sub-packages are treated as distinct packages by the JVM.

#### 2.3 `protected` Access Modifier

- **Scope:** Accessible within the **same package** (by any class) AND by **subclasses located in different packages**.

> ⚠️ **The Critical Cross-Package `protected` Trap (CDAC Classic):**
> A subclass in a different package can access an inherited `protected` member **only through a reference of its own subclass type (or one of its own derived subtypes)**. It **cannot** access that member using a reference to the parent class or a reference to a sibling subclass!

```java
package pkgA;
public class Parent {
    protected int data = 100;
}

package pkgB;
import pkgA.Parent;

public class Child extends Parent {
    public void testAccess() {
        // 1. Valid: Inherited into Child; accessed via 'this' or Child reference
        System.out.println(this.data);
        Child c = new Child();
        System.out.println(c.data);    // VALID

        // 2. COMPILE ERROR: Cannot access protected member using a Parent reference!
        Parent p = new Parent();
        // System.out.println(p.data); // ERROR: data has protected access in pkgA.Parent
    }
}
```

#### 2.4 `public` Access Modifier

- **Scope:** Universally accessible by **any class in any package** across the application (subject to module visibility constraints in Java 9+).

---

### 3. The `final` Keyword

The `final` modifier enforces **immutability and restriction**. It can be applied to variables, methods, and classes.

```
                             The final Keyword
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         ▼                           ▼                           ▼
  final Variables              final Methods               final Classes
  (Cannot be reassigned)       (Cannot be overridden)      (Cannot be extended)
```

#### 3.1 `final` Variables (Compile-Time & Blank Constants)

Once a `final` variable is initialized, its value cannot be changed.

```java
public class FinalVariableEngine {
    // 1. Direct Constant (Inlined by javac into constant pool)
    public static final double TAX_RATE = 0.18;

    // 2. Blank Final Instance Variable (MUST be initialized in EVERY constructor)
    private final int id;

    // 3. Blank Final Static Variable (MUST be initialized in a static block)
    private static final String APP_ENV;

    static {
        APP_ENV = "PRODUCTION";
    }

    public FinalVariableEngine(int id) {
        this.id = id; // Legal: Assigned once
        // this.id = 500; // COMPILE ERROR: variable id might already have been assigned
    }
}
```

> ⚠️ **The Reference Immutability Trap:**
> Marking an object reference variable `final` means the **reference pointer cannot be reassigned** to another heap address. It **does NOT make the underlying object immutable**!

```java
final StringBuilder sb = new StringBuilder("CDAC");
// sb = new StringBuilder("ACTS"); // COMPILE ERROR: Cannot reassign final reference
sb.append(" Pune");                 // FULLY LEGAL: Internal heap state can still mutate!
```

#### 3.2 `final` Methods

A method marked `final` **cannot be overridden** by any subclass.

- **Why Use It?** To prevent subclasses from altering critical business logic or security validation routines (e.g., authentication handshakes).
- **Compiler Optimization:** In earlier JVM versions, `final` methods allowed direct method inlining. Modern HotSpot JIT compilers inline methods dynamically based on profiling, even if they aren't marked `final`.

#### 3.3 `final` Classes

A class marked `final` **cannot be extended** (`class Sub extends FinalClass` fails compilation).

- **Immutability Invariant:** Core Java library classes such as `java.lang.String`, `java.lang.Integer`, and other primitive wrappers are declared `public final class` to guarantee security and immutability across the runtime.
- **Automatic Method Finality:** All methods inside a `final` class are **implicitly `final`**, because no subclass can ever exist to override them.

---

### 4. Packages and Import Mechanisms

A **package** is a namespace that groups related classes, interfaces, and sub-packages, mapping directly to directory paths on the host filesystem.

#### 4.1 Package Statements & Compilation

- The `package` declaration must be the **first non-comment line** in a source file.
- Class paths match directories: `package com.cdac.acts;` corresponds to `com/cdac/acts/` on disk.

#### 4.2 Standard Imports vs. Static Imports

- **Standard Class Import:** Brings external classes into scope without requiring fully qualified names:

  ```java
  import java.util.ArrayList; // Explicit import
  import java.util.*;         // Wildcard package import
  ```

  _(Note: Wildcard imports do not import sub-packages. `import java.awt._`does NOT import`java.awt.event._`)._

- **Static Import (Java 5+):** Imports `static` members (fields and methods) directly into the file's lexical scope, removing the need to prefix them with the class name:

  ```java
  import static java.lang.Math.PI;
  import static java.lang.Math.sqrt;

  public class Geometry {
      public double getCircleArea(double r) {
          return PI * sqrt(r); // Directly accessible without Math.PI or Math.sqrt()
      }
  }
  ```

---

### 5. Nested Classes Taxonomy

A **nested class** is a class declared inside another enclosing class.

```
                              Nested Classes
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
  Static Nested Class                                      Inner Class
  (Declared with static)                                  (Non-static nested class)
  - Decoupled from enclosing instance                     - Bound to enclosing instance
  - No hidden pointer to outer class                      - Holds hidden pointer to Outer.this
                                                                 │
                                 ┌───────────────────────────────┼───────────────────────────────┐
                                 ▼                               ▼                               ▼
                           Member Inner Class           Method-Local Inner Class       Anonymous Inner Class
                           (Defined at class level)     (Defined inside method/block)  (Inline without a name)
```

---

### 6. Static Nested Classes

A static nested class is declared with the `static` modifier inside an outer class.

- **Coupling:** It behaves like a top-level outer class that has been packaged inside another class for namespace convenience.
- **Memory Isolation:** It **does not hold a reference to an instance of the outer class**.
- **Access Permissions:** It can access `private static` members of the outer class, but **cannot** access outer instance variables or methods without an explicit instance reference.

```java
public class OuterContainer {
    private static String systemCode = "SYS-900";
    private int instanceData = 42;

    public static class StaticNested {
        public void execute() {
            System.out.println("System: " + systemCode); // Valid: static access
            // System.out.println(instanceData);         // COMPILE ERROR: No outer instance!
        }
    }
}

// Instantiation: Does NOT require an Outer instance!
OuterContainer.StaticNested nested = new OuterContainer.StaticNested();
nested.execute();
```

---

### 7. Member Inner Classes

A member inner class is a non-static class declared inside an outer class at the member level.

- **Enclosing Instance Link:** An instance of a member inner class **cannot exist without an instance of the outer class**.
- **The Hidden Pointer:** The compiler automatically passes a hidden reference to the outer instance (`Outer.this`) into every inner class constructor.
- **Direct Access:** It can access **all** members of the outer class directly, including `private` fields and methods.

```java
public class Bank {
    private String bankVaultCode = "VAULT-SECURE-99";

    public class VaultClerk {
        public void accessVault() {
            // Direct access to outer private field!
            System.out.println("Accessing: " + bankVaultCode);
            // Explicit qualification syntax:
            System.out.println("Parent instance: " + Bank.this);
        }
    }
}

// Instantiation Syntax (Requires Outer Instance!):
Bank myBank = new Bank();
Bank.VaultClerk clerk = myBank.new VaultClerk(); // Note: outerRef.new InnerClass()
clerk.accessVault();
```

#### Memory Layout: Outer-to-Inner Pointer

```
STACK FRAME                               HEAP MEMORY
┌──────────────────────────────┐          ┌──────────────────────────────────────────────┐
│ clerk: 0x2200 ───────────────┼─────────►│ VaultClerk Instance (0x2200)                 │
└──────────────────────────────┘          ├──────────────────────────────────────────────┤
                                          │ final Bank this$0 = 0x1100 ──────────┐       │
                                          └──────────────────────────────────────┼───────┘
                                                                                 ▼
                                          ┌──────────────────────────────────────────────┐
                                          │ Bank Instance (0x1100)                       │
                                          ├──────────────────────────────────────────────┤
                                          │ bankVaultCode: "VAULT-SECURE-99"             │
                                          └──────────────────────────────────────────────┘
```

---

### 8. Method-Local Inner Classes

A class defined **inside the body of a method, constructor, or block scope**.

- **Scope:** Restricted entirely to the block where it is defined. It cannot be instantiated outside that method.
- **Access to Local Variables:** Can access local variables and parameters of the enclosing method **only if they are `final` or effectively final** (variables whose values are never mutated after initialization).

```java
public class PaymentGateway {
    public void processPayment(final double amount) {
        int transactionFee = 2; // Effectively final

        // Method-Local Inner Class
        class InternalValidator {
            void validate() {
                // Accessing method parameters and local variables:
                if (amount <= 0) throw new IllegalArgumentException();
                System.out.println("Processing: " + (amount + transactionFee));
            }
        }

        InternalValidator validator = new InternalValidator();
        validator.validate();
    }
}
```

> ⚠️ **Why Must Local Variables Be Effectively Final? (Interview Classic):**
> A method-local inner class object may outlive the execution of the method (e.g., if it is passed to a background thread). When the method finishes, its stack frame is destroyed.
> To keep the inner class functional, Java **copies the local variable's value** into a synthetic field inside the inner class. If that local variable were mutable, the copy in the heap object and the variable on the stack would fall out of sync. To prevent this split-state discrepancy, Java requires captured variables to be immutable (`final`).

---

### 9. Enum Types (`java.lang.Enum`)

An `enum` is a specialized reference type introduced in Java 5 that represents a fixed set of named constants.

```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

#### 9.1 Bytecode Lowering of `enum`

Under the hood, the compiler transforms an `enum` into a regular class that extends **`java.lang.Enum<E>`**:

```java
// How javac compiles 'public enum Day':
public final class Day extends java.lang.Enum<Day> {
    public static final Day MONDAY = new Day("MONDAY", 0);
    public static final Day TUESDAY = new Day("TUESDAY", 1);
    // ... remaining constants
    private static final Day[] $VALUES = new Day[]{ MONDAY, TUESDAY, ... };

    public static Day[] values() { return (Day[])$VALUES.clone(); }
    public static Day valueOf(String name) { ... }
    private Day(String name, int ordinal) { super(name, ordinal); }
}
```

#### 9.2 Key Invariants of Enums

1. **Implicitly Final:** Enums extend `java.lang.Enum` and cannot be extended by other classes.
2. **Private Constructors Only:** Enum constructors are implicitly `private`. Calling an enum constructor via `new` from outside causes a compile-time error.
3. **Fields, Methods, and Custom Constructors:** Enums can contain instance fields, constructors, concrete methods, and abstract methods:

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    };

    private final String symbol;

    // Enum constructor (Implicitly private)
    Operation(String symbol) {
        this.symbol = symbol;
    }

    public String getSymbol() { return symbol; }
    public abstract double apply(double x, double y); // Abstract method in enum
}
```

#### 9.3 Core Enum Methods

- `name()`: Returns the exact string identifier as declared (e.g., `"PLUS"`).
- `ordinal()`: Returns the zero-based index of the constant in its declaration order (`0`, `1`, etc.).
- `values()`: Static method returning an array containing all constants in declaration order.
- `valueOf(String)`: Parses a string and returns the corresponding enum constant.

---

### 10. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can an inner class declare `static` members?**
  - _Answer:_
    - Prior to **Java 16**: **No.** Member inner classes could not declare static fields or methods unless they were compile-time constants (`static final`).
    - In **Java 16 and later**: **Yes.** Inner classes are permitted to declare static members and static initializers.

- **Q2: What is the compiler-generated name for inner and anonymous class `.class` files?**
  - _Answer:_
    - Member Inner Class: `Outer$Inner.class`
    - Static Nested Class: `Outer$StaticNested.class`
    - Method-Local Inner Class: `Outer$1LocalClass.class`
    - Anonymous Inner Class: `Outer$1.class`, `Outer$2.class`, etc.

- **Q3: What happens if an enum extends a class?**
  - _Answer:_ **Compilation Error.** Every enum implicitly extends `java.lang.Enum`. Because Java does not support multiple class inheritance, an enum cannot use `extends` to inherit from any other class. It can, however, implement interfaces (`enum Day implements Serializable`).

- **Q4: Can a class be declared `static`?**
  - _Answer:_ **Only if it is a nested class.** A top-level outer class can only use `public`, `abstract`, `final`, or default access. Declaring a top-level outer class `static` causes a compile error: _modifier static not allowed here_.

- **Q5: Can you clone an Enum instance using `.clone()`?**
  - _Answer:_ **No.** The `clone()` method in `java.lang.Enum` is declared `final` and unconditionally throws `CloneNotSupportedException`. This preserves the singleton identity invariant of enum constants.
