# Module 08: Reflection, Annotations, and JVM Mechanics

## Unit 8.1: Reflection API

### 1. Introduction to Reflection

**Reflection** is a mechanism in Java that allows an executing program to examine, introspect, and modify the runtime behavior, structure, and internal state of classes, interfaces, fields, methods, and constructors loaded into the Java Virtual Machine.

Normally, Java programs exhibit **static typing and early binding**: the compiler checks types, ensures method validity, and binds invocations before producing bytecode. Reflection enables **dynamic runtime inspection and late binding**, allowing code to interact with arbitrary classes whose names or interfaces were completely unknown at compile time.

```
                    Static Compilation vs. Runtime Reflection
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         ▼                                                           ▼
Static Invocation (Early Binding)                           Reflective Invocation (Late Binding)
- Hardcoded class names in source code                      - Class name loaded dynamically as String
- Verified by javac at compile time                         - Verified by JVM at runtime
- Direct machine instruction branch                         - Dynamic method dispatch via Method.invoke()
- Strong compile-time type safety                           - Requires runtime exception handling

```

### 2. Need for Reflection

Without reflection, an application cannot interact with code it was not compiled against. Modern enterprise Java ecosystems rely on reflection to decouple components and build extensible architectures:

```
+--------------------------------------------------------------------------------------------------+
| Architectural Problem Solved by Reflection                                                      |
|                                                                                                  |
| Imagine building Spring Framework or Hibernate ORM:                                             |
| • Spring must instantiate @Component or @Service classes written by users years after Spring     |
|   itself was compiled.                                                                           |
| • Hibernate must read private entity fields to map them to database columns without getters.    |
| • JUnit must find and execute all methods marked with @Test without a hardcoded main() runner.   |
|                                                                                                  |
| Solution: Reflection allows frameworks to inspect bytecode, find annotations, instantiate       |
| objects, and inject dependencies dynamically at runtime.                                         |
+--------------------------------------------------------------------------------------------------+
```

#### Core Benefits of Reflection:

1. **Extensibility & Plugins:** Applications can load third-party extensions or drivers dynamically at runtime (e.g., `Class.forName("org.postgresql.Driver")`).
2. **Framework Plumbing:** Enables Dependency Injection (Spring, Guice), Object-Relational Mapping (Hibernate), and Serialization/Deserialization (Jackson, Gson).
3. **Automated Testing & Tooling:** Test runners (JUnit, TestNG) scan for test methods and execute setup/teardown hooks automatically.
4. **Dynamic Proxies:** Allows runtime interception of method calls for logging, transaction boundaries (`@Transactional`), and security checks.

---

### 3. The `java.lang.Class<T>` Class

Every type loaded into the JVM (classes, interfaces, enums, records, primitive types, and array dimensions) is paired with an immutable descriptor instance of **`java.lang.Class<T>`** allocated on the Heap.

```
Metaspace (JVM Native Process Memory)
┌──────────────────────────────────────────────┐
│ Klass Metadata (JVM internal C++ layout):    │
│ - Bytecode instructions                      │
│ - Virtual Method Table (vtable)              │
│ - Constant Pool entries                      │
└──────────────────────┬───────────────────────┘
                       │ Mirrors / Exposes
                       ▼
JVM Managed Heap
┌──────────────────────────────────────────────┐
│ java.lang.Class<Employee> Object Instance    │ ◄── Object exposed to Java
│ - Class name: "com.cdac.model.Employee"      │     reflection APIs
│ - Fields: [id, name, salary]                 │
│ - Methods: [getId, calculateTax, ...]        │
└──────────────────────────────────────────────┘
```

#### Key Invariants of `Class<T>`:

- **Private Constructor:** `java.lang.Class` has a private constructor (`Class(ClassLoader loader, Class<?> arrayComponentType)`). Only the JVM ClassLoader can instantiate `Class` instances as classes are loaded.
- **Singleton Per Type Per ClassLoader:** For any given loaded class, exactly **one** `Class` object exists per `ClassLoader`.

---

### 4. Class Metadata Inspection

The `Class<T>` descriptor provides rich introspection queries for type hierarchies:

```java
Class<?> clazz = TargetClass.class;

// Package & Identifier Metadata
String simpleName = clazz.getSimpleName();      // "TargetClass"
String canonicalName = clazz.getCanonicalName(); // "com.cdac.core.TargetClass"
Package pkg = clazz.getPackage();                // package com.cdac.core

// Modifiers (bitwise bitmask)
int modifiers = clazz.getModifiers();
boolean isPublic   = Modifier.isPublic(modifiers);
boolean isAbstract = Modifier.isAbstract(modifiers);
boolean isFinal    = Modifier.isFinal(modifiers);

// Type Hierarchy Inspection
Class<?> superClass = clazz.getSuperclass();     // Immediate parent class
Class<?>[] interfaces = clazz.getInterfaces();   // Direct interfaces implemented
```

---

### 5. Obtaining `Class` Objects (The 3 Canonical Techniques)

Java provides three distinct ways to acquire a `Class` object:

```
                            Obtaining a Class Reference
                                         │
         ┌───────────────────────────────┼───────────────────────────────┐
         ▼                               ▼                               ▼
1. Class Literal Syntax         2. Object Instance (.getClass()) 3. Dynamic Name (Class.forName())
   String.class                    obj.getClass()                   Class.forName("java.lang.String")
   - Compile-time reference        - Requires existing instance     - Fully dynamic string path
   - Works on primitives           - Runtime polymorphism           - Throws ClassNotFoundException
   - Zero runtime overhead         - Heap object inspection         - Triggers ClassLoader linking
```

#### 5.1 Technique 1: Class Literal Syntax (`.class`)

Evaluated at compile-time. Safe, fast, and does not require an active object instance. Works on primitives and `void` as well:

```java
Class<String> c1 = String.class;
Class<Integer> c2 = int.class;      // Primitive type class
Class<Void> c3 = void.class;        // Represents return type void
```

#### 5.2 Technique 2: `getClass()` Method

Declared in `java.lang.Object`. Available on any active object reference handle on the Heap:

```java
Object obj = new String("CDAC");
Class<?> c = obj.getClass(); // Evaluates to java.lang.String at runtime
```

#### 5.3 Technique 3: `Class.forName(String className)`

Dynamically loads and links a class using its Fully Qualified Class Name (FQCN). If the class is not found on the classpath, it throws **`ClassNotFoundException`**.

```java
try {
    // Loads and initializes the MySQL JDBC Driver
    Class<?> driverClass = Class.forName("com.mysql.cj.jdbc.Driver");
} catch (ClassNotFoundException e) {
    System.err.println("Driver class not located on current Classpath: " + e.getMessage());
}
```

---

### 6. Inspecting Fields: `Field` API

The **`java.lang.reflect.Field`** class provides metadata and dynamic read/write access to an individual field of a class or interface.

#### 6.1 `getFields()` vs. `getDeclaredFields()` (CRITICAL EXAM DISTINCTION)

```
┌────────────────────────────────────────────────────────────────────────┐
│ Field Reflection Query Methods                                         │
├─────────────────────────┬──────────────────────────────────────────────┤
│ Method                  │ Scope & Visibility                           │
├─────────────────────────┼──────────────────────────────────────────────┤
│ clazz.getFields()       │ Returns ALL public fields (including         │
│                         │ inherited public fields from superclasses).  │
├─────────────────────────┼──────────────────────────────────────────────┤
│ clazz.getDeclaredFields│ Returns ALL fields (public, protected,       │
│                         │ package-private, and private) declared       │
│                         │ STRICTLY within the class. DOES NOT include  │
│                         │ inherited fields!                            │
└─────────────────────────┴──────────────────────────────────────────────┘
```

```java
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;

public class FieldInspector {
    public static void inspect(Class<?> clazz) {
        Field[] fields = clazz.getDeclaredFields();
        for (Field f : fields) {
            System.out.printf("Field: %s | Type: %s | Modifiers: %s%n",
                f.getName(),
                f.getType().getSimpleName(),
                Modifier.toString(f.getModifiers())
            );
        }
    }
}
```

---

### 7. Inspecting Methods: `Method` API

The **`java.lang.reflect.Method`** class models executable instance and static methods.

#### 7.1 `getMethods()` vs. `getDeclaredMethods()`

- **`clazz.getMethods()`:** Returns all **public** methods of the class, including all public methods inherited from superclasses and `java.lang.Object`.
- **`clazz.getDeclaredMethods()`:** Returns **all** methods (private, protected, default, public) declared directly by this class, but **excludes** inherited methods.

```java
import java.lang.reflect.Method;
import java.lang.reflect.Parameter;

public class MethodInspector {
    public static void inspect(Class<?> clazz) {
        for (Method m : clazz.getDeclaredMethods()) {
            System.out.printf("Method: %s (Returns: %s)%n", m.getName(), m.getReturnType().getName());

            for (Parameter p : m.getParameters()) {
                System.out.printf("   Param: %s of type %s%n", p.getName(), p.getType().getName());
            }

            for (Class<?> ex : m.getExceptionTypes()) {
                System.out.println("   Throws: " + ex.getName());
            }
        }
    }
}
```

---

### 8. Inspecting Constructors: `Constructor` API

The **`java.lang.reflect.Constructor<T>`** class represents a constructor declared by a class.

```java
import java.lang.reflect.Constructor;

public class ConstructorInspector {
    public static void inspect(Class<?> clazz) {
        // Inspect all declared constructors (including private ones)
        Constructor<?>[] constructors = clazz.getDeclaredConstructors();
        for (Constructor<?> c : constructors) {
            System.out.println("Constructor: " + c.getName() + " with " +
                               c.getParameterCount() + " parameters.");
        }
    }
}
```

---

### 9. Creating Objects Using Reflection

Historically, Java provided `clazz.newInstance()` to instantiate objects using their no-arg constructor.

> ⚠️ **The Deprecation of `Class.newInstance()` (Java 9+):**
> `clazz.newInstance()` was deprecated because it bypassed checked exception declarations (could throw arbitrary checked exceptions without `throws`) and could not handle parameterized constructors.
> **Modern Invariant:** Always obtain the constructor via **`Constructor.newInstance()`**.

```java
import java.lang.reflect.Constructor;

public class ObjectInstantiationDemo {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Class.forName("com.cdac.model.User");

        // Step 1: Locate the specific constructor by passing parameter types
        Constructor<?> constructor = clazz.getDeclaredConstructor(int.class, String.class);

        // Step 2: Instantiate object by supplying arguments
        Object userInstance = constructor.newInstance(101, "Aarav");

        System.out.println("Created: " + userInstance);
    }
}
```

---

### 10. Invoking Methods Dynamically

A method can be invoked dynamically using **`method.invoke(targetObject, arguments...)`**:

```
                       method.invoke() Dispatch Model
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
Instance Method Invocation                                  Static Method Invocation
method.invoke(targetInstance, arg1, arg2)                   method.invoke(null, arg1, arg2)
- First argument MUST be the object instance                - First argument is ignored (pass null)
- If targetInstance is null -> NullPointerException         - Direct call on class-level method
```

```java
import java.lang.reflect.Method;

public class DynamicInvocationDemo {
    public static void main(String[] args) throws Exception {
        String message = "hello world";

        // Find method: public String toUpperCase()
        Method upperMethod = String.class.getMethod("toUpperCase");

        // Dynamic dispatch: target is 'message', parameters are empty
        Object result = upperMethod.invoke(message);

        System.out.println(result); // "HELLO WORLD"

        // Dynamic dispatch for static method: Math.sqrt(double)
        Method sqrtMethod = Math.class.getMethod("sqrt", double.class);
        Object sqrtResult = sqrtMethod.invoke(null, 25.0); // Pass null as target instance!
        System.out.println(sqrtResult); // 5.0
    }
}
```

#### Exception Wrapping in `Method.invoke()`:

If the underlying method being invoked throws an exception during execution, `Method.invoke()` intercepts it, wraps it, and throws **`java.lang.reflect.InvocationTargetException`**. To retrieve the original root cause, call:

```java
try {
    method.invoke(target, args);
} catch (InvocationTargetException e) {
    Throwable originalCause = e.getCause(); // The genuine application exception!
    System.err.println("Method failed due to: " + originalCause.getMessage());
}
```

---

### 11. Accessing Private Members (`setAccessible`)

Reflection allows code to bypass Java's standard access control boundaries (`private`, `protected`, package-private) using `AccessibleObject.setAccessible(true)`:

```
                  Encapsulation Bypass via setAccessible
┌─────────────────────────────────┐
│ class SecretVault {             │
│    private String pin = "9988"; │ ◄── Inaccessible to normal compiler calls
│ }                               │
└─────────────────────────────────┘
                 ▲
                 │ f.setAccessible(true);
┌────────────────┴────────────────┐
│ Reflection API:                 │
│ f.get(vaultInstance);           │ ──► Reads "9988" directly from Heap memory!
└─────────────────────────────────┘
```

```java
import java.lang.reflect.Field;

class SecretVault {
    private String masterKey = "TOP_SECRET_VAULT_KEY";
}

public class PrivateFieldBreaker {
    public static void main(String[] args) throws Exception {
        SecretVault vault = new SecretVault();

        // 1. Locate the private field
        Field keyField = SecretVault.class.getDeclaredField("masterKey");

        // 2. Suppress Java access control checks!
        keyField.setAccessible(true);

        // 3. Read the private state
        String readValue = (String) keyField.get(vault);
        System.out.println("Extracted: " + readValue);

        // 4. Overwrite private state dynamically!
        keyField.set(vault, "MODIFIED_IN_MEMORY");
        System.out.println("Updated: " + keyField.get(vault));
    }
}
```

#### Modern Java Modular Boundary Restrictions (Java 9+ Project Jigsaw):

In modern Java (Java 9 through Java 21+), **Strong Encapsulation** restricts `setAccessible(true)` across Java Module System (`module-info.java`) boundaries:

- If a package inside a module is not explicitly declared `opens <package> to <framework>`, attempting `setAccessible(true)` throws **`java.lang.reflect.InaccessibleObjectException`**.
- The core runtime (`java.base`) restricts deep reflection into internal JDK packages (e.g., `sun.misc.Unsafe`) unless overridden by command-line flags: `--add-opens java.base/java.lang=ALL-UNNAMED`.

---

### 12. Annotations Fundamentals & Runtime Processing

An **Annotation** (`@interface`) is a form of syntactic metadata added to classes, methods, fields, parameters, or packages. Annotations do not directly alter bytecode execution on their own; they must be inspected by tooling or via the Reflection API.

#### 12.1 The Two Critical Meta-Annotations:

1. **`@Retention`:** Defines **how long** the annotation is retained:
   - `RetentionPolicy.SOURCE`: Discarded during compilation (e.g., `@Override`, `@SuppressWarnings`). Invisible to reflection!
   - `RetentionPolicy.CLASS`: Recorded in the `.class` file by `javac`, but discarded by the JVM during class loading. Cannot be read reflectively at runtime.
   - **`RetentionPolicy.RUNTIME`**: Recorded in the `.class` file AND loaded into Metaspace by the JVM. **Mandatory if you want to inspect the annotation via reflection!**

2. **`@Target`:** Defines **where** the annotation can be placed (`ElementType.TYPE`, `METHOD`, `FIELD`, `PARAMETER`, `CONSTRUCTOR`).

#### 12.2 Building and Processing a Custom Annotation:

```java
import java.lang.annotation.*;
import java.lang.reflect.Method;

// Step 1: Declare the Annotation with RUNTIME retention
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface BenchmarkTest {
    String description() default "Standard Benchmark";
    int runs() default 1;
}

// Step 2: Annotate a Service Class
class PaymentService {
    @BenchmarkTest(description = "Payment processing latency", runs = 5)
    public void executePayment() {
        System.out.println("Executing payment settlement...");
    }

    public void regularUtility() {
        System.out.println("Non-benchmarked method");
    }
}

// Step 3: Write the Reflection Processing Engine
public class AnnotationProcessorEngine {
    public static void main(String[] args) throws Exception {
        PaymentService service = new PaymentService();
        Class<?> clazz = service.getClass();

        for (Method m : clazz.getDeclaredMethods()) {
            // Check if annotation is present
            if (m.isAnnotationPresent(BenchmarkTest.class)) {
                BenchmarkTest benchmark = m.getAnnotation(BenchmarkTest.class);
                System.out.println("Found Annotated Method: " + m.getName());
                System.out.println("  Description: " + benchmark.description());
                System.out.println("  Execution count: " + benchmark.runs());

                // Execute according to metadata configuration
                for (int i = 0; i < benchmark.runs(); i++) {
                    m.invoke(service);
                }
            }
        }
    }
}
```

---

### 13. Practical Reflection Use Cases

```
                        Industry Reflection Pipelines
                                      │
     ┌────────────────────────┬───────┴────────┬────────────────────────┐
     ▼                        ▼                ▼                        ▼
Spring Core DI           Hibernate ORM    JSON Serializers         Testing Frameworks
Scans @Autowired,        Maps private     Inspects fields of any   Scans @Test methods,
instantiates beans,      fields to SQL    DTO to dynamically       runs assertions
injects dependencies     result sets      build JSON text          (JUnit 5, TestNG)
```

1. **Dependency Injection (Spring / Quarkus):** Frameworks scan the classpath for annotations like `@Component`, read private fields marked `@Autowired`, and inject dependencies using `field.set(beanInstance, dependency)`.
2. **Object-Relational Mapping (Hibernate / JPA):** Hibernate uses reflection to instantiate entity objects without calling user constructors and directly populates private fields from database row sets.
3. **Serialization Libraries (Jackson / Gson):** Jackson traverses all declared fields of an unknown domain object, reads their values reflectively, and constructs matching JSON text.
4. **Dynamic Proxies (`java.lang.reflect.Proxy`):** Creates runtime implementations of interfaces on-the-fly to intercept method calls for transactions, caching, or logging.

---

### 14. Reflection Limitations and Tradeoffs

While reflection provides immense architectural flexibility, it carries significant engineering costs:

| Limitation                                 | Technical Reason                                                                                                                                                                                                                                    |
| :----------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Performance Overhead**                   | Incurs dynamic type resolution, argument boxing/unboxing, security manager verifications, and lookup indirections. Prevents JVM JIT compiler optimizations like inlining. Reflection is roughly **10x to 50x slower** than direct method calls.     |
| **Loss of Compile-Time Safety**            | Method and field names are passed as string literals (e.g., `"calculateTax"`). If a method is renamed during refactoring, compilation succeeds cleanly, but the application crashes at runtime with `NoSuchMethodException`.                        |
| **Encapsulation Breakdown**                | By using `setAccessible(true)`, reflection violates object encapsulation and can break class invariants, making code harder to debug and maintain.                                                                                                  |
| **Security Manager & Module Restrictions** | Environments running with strict Security Managers or Java 9+ Modules can block reflection operations, causing `SecurityException` or `InaccessibleObjectException`.                                                                                |
| **GraalVM Native Image Incompatibility**   | Ahead-Of-Time (AOT) compilers (like GraalVM) compile Java directly into machine code and strip out unused classes. Reflective lookups fail unless every reflective target is manually declared in static `reflect-config.json` configuration files. |

---

### 15. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What is the output of `System.out.println(int.class == Integer.class)`?**
  - _Answer:_ **`false`**.
  - _Explanation:_ `int.class` represents the primitive pseudo-type `int` (equivalent to `Integer.TYPE`), whereas `Integer.class` represents the wrapper class `java.lang.Integer`. They are two completely distinct `Class` instances in the JVM.

- **Q2: Why does `Class.forName("com.mysql.cj.jdbc.Driver")` register a JDBC driver even though no instance is returned?**
  - _Answer:_ `Class.forName(name)` by default **loads, links, and initializes** the class. Class initialization executes all **static initialization blocks (`<clinit>`)**. JDBC drivers contain a static initializer:
    ```java
    static {
        java.sql.DriverManager.registerDriver(new Driver());
    }
    ```
    This registers the driver with `DriverManager` simply as a side-effect of loading the class into Metaspace.

- **Q3: What exception is thrown if `method.invoke()` fails because the target method threw an internal `NullPointerException`?**
  - _Answer:_ **`java.lang.reflect.InvocationTargetException`**.
  - _Explanation:_ The reflection engine intercepts any checked or unchecked exception thrown by the underlying method execution and wraps it inside an `InvocationTargetException`. The actual `NullPointerException` must be accessed via `e.getCause()`.

- **Q4: Can reflection be used to modify a `private static final` field?**
  - _Answer:_ **Technically yes, but with major caveats.**
    - You can use `field.setAccessible(true)` and use reflection to remove the `Modifier.FINAL` bit from the field modifiers.
    - **However:** If the `final` field is a compile-time constant (e.g., `private static final int VALUE = 100;` or a String literal), the `javac` compiler **inlines the constant value directly into all call-sites** in bytecode. Mutating the memory slot via reflection will not affect already-inlined call sites. Furthermore, starting in Java 12+, modifying `final` fields via reflection on core classes throws `IllegalAccessException`.

- **Q5: What is the difference between `clazz.getField("id")` and `clazz.getDeclaredField("id")`?**
  - _Answer:_
    - `getField("id")` searches for a **public** field named `"id"` in the class **or any of its superclasses**. If `"id"` is private, it throws `NoSuchFieldException`.
    - `getDeclaredField("id")` searches for a field named `"id"` with **any access modifier** (`public`, `protected`, `private`, default) declared **strictly within the class body**, but ignores all superclasses.
