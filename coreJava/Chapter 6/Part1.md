# Module 06: Java Collections Framework & Generics

## Unit 6.1: Collections Framework Fundamentals

### 1. Introduction to Collections

A **Collection** in Java is an architectural container object that groups multiple individual elements into a single cohesive unit.

The **Java Collections Framework (JCF)**, introduced in **Java 2 (JDK 1.2)** under the `java.util` package, provides a unified, highly optimized architecture for representing, manipulating, sorting, and searching aggregate data structures independent of implementation details.

```
                              Java Collections Framework Architecture
                                                │
         ┌──────────────────────────────────────┴──────────────────────────────────────┐
         ▼                                                                             ▼
Core Interfaces (Contracts)                                                  Standard Implementations
Define abstract data behaviors                                               Pre-built, production-tested
- Collection, List, Set, Queue                                               - ArrayList, LinkedList
- Deque, Map, SortedSet, SortedMap                                           - HashSet, TreeSet, HashMap
```

---

### 2. Need for the Collections Framework

Prior to JDK 1.2, Java lacked a standard collection model. Developers had to rely on distinct, ad-hoc container classes:

- Native Arrays (`Type[]`)
- `java.util.Vector`
- `java.util.Hashtable`
- `java.util.Stack`
- `java.util.Dictionary`

Each legacy container had completely different method names, access contracts, and synchronization semantics (e.g., `Vector.elementAt(i)` vs. `array[i]` vs. `Hashtable.get(k)`). There was no common parent interface linking them.

```
+--------------------------------------------------------------------------------------------------+
| Problem in Legacy Java (Pre-JDK 1.2)                                                              |
|                                                                                                  |
|   Array:        arr[index]             --> Fixed size, fast random access, lacks dynamic growth   |
|   Vector:       v.elementAt(index)     --> Synchronized (slow), dynamic growth                    |
|   Hashtable:    ht.get(key)            --> Key-Value pairs, synchronized, no uniform interface   |
|                                                                                                  |
| Result: Zero interoperability. Code written for Vector could not easily operate on Arrays.        |
+--------------------------------------------------------------------------------------------------+
```

#### Core Benefits of the Unified Collections Framework:

1. **Reduces Programming Effort:** Supplies production-grade data structures (dynamic arrays, trees, hash tables, priority heaps) out-of-the-box.
2. **Increases Performance:** Implementations provide guaranteed algorithmic time and space complexities (e.g., $\mathcal{O}(1)$ hash lookups, $\mathcal{O}(\log n)$ red-black tree operations).
3. **Fosters Interoperability:** Common interfaces allow collections to be passed across disparate APIs cleanly.
4. **Facilitates Code Reuse:** Common algorithms (sorting, shuffling, searching, reversing) work across any collection implementing the base contracts.

---

### 3. The Core Collections Framework Hierarchy

The framework splits into two primary, distinct interface hierarchies:

1. **`java.util.Collection`:** The root hierarchy for individual, element-based collections.
2. **`java.util.Map`:** An independent hierarchy for associative **key-value pair mappings**.

```
                                      java.lang.Iterable<T>
                                                │
                                                ▼
                                    java.util.Collection<E>
                                                │
         ┌──────────────────────────────────────┼──────────────────────────────────────┐
         ▼                                      ▼                                      ▼
   java.util.List<E>                      java.util.Queue<E>                     java.util.Set<E>
   (Ordered, Duplicates)                 (FIFO / Ordering)                      (Unique Elements)
   ├── ArrayList                         ├── PriorityQueue                             │
   ├── LinkedList                        │                                             ▼
   └── Vector                            ▼                                    java.util.SortedSet<E>
       └── Stack                  java.util.Deque<E>                          (Total Element Order)
                                  ├── ArrayDeque                                       │
                                  └── LinkedList                                       ▼
                                                                              java.util.NavigableSet<E>
                                                                                       │
                                                                                       ▼
                                                                                    TreeSet


   [Independent Hierarchy]
                                      java.util.Map<K, V>
                                      (Key-Value Mappings)
                                                │
         ┌──────────────────────────────────────┼──────────────────────────────────────┐
         ▼                                      ▼                                      ▼
      HashMap                             LinkedHashMap                        java.util.SortedMap<K,V>
         │                                (Preserves Insertion Order)                  │
         ▼                                                                             ▼
     Hashtable                                                               java.util.NavigableMap<K,V>
     (Legacy Synchronized)                                                             │
                                                                                       ▼
                                                                                    TreeMap
```

> ⚠️ **Why does `Map` NOT extend `Collection`? (Top Interview Question):**
> A `Collection` models a group of discrete, single elements ($E$). A `Map` models an associative mapping of **Key-Value pairs** ($\langle K, V \rangle$).
> Fundamental `Collection` operations cannot map cleanly to `Map`:
>
> - `collection.add(element)` expects 1 argument; `map.put(key, value)` requires 2.
> - Maps require unique keys, but allow duplicate values.
>   Forcing `Map` to extend `Collection` would break the Liskov Substitution Principle and violate interface segregation.

---

### 4. The `Iterable<T>` Interface

`java.lang.Iterable<T>` is the top-level parent interface of the entire `Collection` tree. Any class implementing `Iterable` can be the target of the **Enhanced `for-each` loop**.

```java
package java.lang;

public interface Iterable<T> {
    Iterator<T> iterator(); // Returns an iterator over elements of type T

    // Java 8 Default Methods:
    default void forEach(Consumer<? super T> action) { ... }
    default Spliterator<T> spliterator() { ... }
}
```

#### Compiler Lowering of Enhanced `for`:

When you write:

```java
List<String> names = List.of("Aarav", "Bhavna", "Chetan");
for (String name : names) {
    System.out.println(name);
}
```

The Java compiler (`javac`) desugars it into an explicit `Iterator` sequence:

```java
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    System.out.println(name);
}
```

---

### 5. The `Iterator<E>` Interface

`java.util.Iterator<E>` provides a standardized, unidirectional cursor for traversing any `Collection`.

```java
package java.util;

public interface Iterator<E> {
    boolean hasNext(); // Returns true if the iteration has more elements
    E next();          // Returns the next element; throws NoSuchElementException if none

    default void remove() { // Removes the last element returned by next()
        throw new UnsupportedOperationException("remove");
    }
}
```

#### 5.1 Safe Modification: `iterator.remove()`

Modifying a collection directly while traversing it using an enhanced for-loop throws **`ConcurrentModificationException`**. The only safe way to remove elements during traversal is using the iterator's own **`remove()`** method:

```java
List<Integer> numbers = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6));

// WRONG: Triggers java.util.ConcurrentModificationException!
// for (Integer n : numbers) {
//     if (n % 2 == 0) numbers.remove(n);
// }

// CORRECT: Safe in-place removal
Iterator<Integer> it = numbers.iterator();
while (it.hasNext()) {
    Integer n = it.next();
    if (n % 2 == 0) {
        it.remove(); // Safely removes element from the underlying collection
    }
}
```

---

### 6. The `Collection<E>` Interface Contract

`java.util.Collection<E>` defines the foundational methods common to all bag/set/queue data containers:

```java
public interface Collection<E> extends Iterable<E> {
    // Basic Query Operations
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);

    // Modification Operations
    boolean add(E e);
    boolean remove(Object o);

    // Bulk Operations
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c); // Mathematical Set Difference
    boolean retainAll(Collection<?> c); // Mathematical Set Intersection
    void clear();

    // Comparison and Hashing
    boolean equals(Object o);
    int hashCode();
}
```

---

### 7. Overview of Core Collection Sub-Interfaces

```
                         Collection Sub-Interface Matrix
                                        │
         ┌──────────────────────────────┼──────────────────────────────┐
         ▼                              ▼                              ▼
     List<E>                         Set<E>                         Queue<E>
 - Ordered (Index-based)        - Unordered (No index)         - Holding area for processing
 - Positional access (0..n-1)   - Uniqueness guaranteed        - Strict extraction order
 - Allows duplicate elements    - No duplicate elements        - FIFO / Priority ordering
 - Allows multiple nulls        - Allows max 1 null (HashSet)  - Nulls generally prohibited
```

#### 7.1 The `List<E>` Interface

- An **ordered collection** (also known as a _sequence_).
- Elements can be inserted or accessed by their precise zero-based integer index: `get(int index)`, `set(int index, E element)`, `add(int index, E element)`, `remove(int index)`.
- Fully permits duplicate elements ($e_1\text{.equals}(e_2) == \text{true}$) and multiple `null` entries.
- Key implementations: `ArrayList` (dynamic array), `LinkedList` (doubly-linked list), `Vector` (legacy synchronized array).

#### 7.2 The `Set<E>` Interface

- Models the mathematical set abstraction: **contains no duplicate elements**.
- At most one `null` element is permitted in standard hash-based implementations (`HashSet`, `LinkedHashSet`). Sorted sets (`TreeSet`) prohibit `null` entirely.
- Does not offer positional index-based access (`set.get(i)` does not exist).
- Key implementations: `HashSet` (hash table), `LinkedHashSet` (hash table + linked list preserving insertion order), `TreeSet` (Red-Black self-balancing tree).

#### 7.3 The `Queue<E>` Interface

- Designed for holding elements prior to consumption.
- Typically orders elements in a **FIFO (First-In-First-Out)** manner, though priority queues order elements according to natural sorting or an explicit `Comparator`.
- Provides two sets of methods for insert, inspect, and remove operations: one throws exceptions on failure, and the other returns special indicator values (`false` or `null`):

| Operation Type | Throws Exception on Failure | Returns Special Value (`false` / `null`) |
| :------------- | :-------------------------- | :--------------------------------------- |
| **Insert**     | `add(e)`                    | `offer(e)`                               |
| **Remove**     | `remove()`                  | `poll()`                                 |
| **Examine**    | `element()`                 | `peek()`                                 |

#### 7.4 The `Deque<E>` Interface (Double-Ended Queue)

- Supports element insertion and removal at **both endpoints** (Head and Tail).
- Operates simultaneously as a **FIFO Queue** and a **LIFO Stack** (replaces the legacy synchronized `java.util.Stack` class).
- Key implementations: `ArrayDeque` (resizable array, typically faster than `Stack` and `LinkedList`), `LinkedList`.

```
                    ArrayDeque as Double-Ended Buffer
                 Head                                    Tail
          ┌───────▼────────────────────────────────────────▼───────┐
Push/Pop  │ pollFirst()                                 offerLast() │  Offer/Poll
◄═════════│ offerFirst()                                pollLast()  │══════════►
          └────────────────────────────────────────────────────────┘
```

#### 7.5 The `Map<K, V>` Interface

- An object that maps distinct keys to values.
- Keys **must be unique**; values can be duplicated.
- Supports three collection views:
  1. `keySet()`: Returns a `Set<K>` containing all unique keys.
  2. `values()`: Returns a `Collection<V>` containing all values.
  3. `entrySet()`: Returns a `Set<Map.Entry<K, V>>` containing key-value associations.

---

### 8. `Collection` vs. `Collections`

| Dimension         | `java.util.Collection`                            | `java.util.Collections`                                                                            |
| :---------------- | :------------------------------------------------ | :------------------------------------------------------------------------------------------------- |
| **Type**          | Root **Interface**                                | Concrete **Utility Class**                                                                         |
| **Package**       | `java.util`                                       | `java.util`                                                                                        |
| **Instantiation** | Cannot be instantiated directly                   | Cannot be instantiated (private constructor)                                                       |
| **Content**       | Declares abstract method contracts for containers | Consists strictly of **static utility methods** and polymorphic wrappers                           |
| **Typical Use**   | `Collection<String> c = new ArrayList<>();`       | `Collections.sort(list);`<br>`Collections.reverse(list);`<br>`Collections.unmodifiableList(list);` |

---

### 9. Architectural Comparison: Array vs. `ArrayList`

```
  Fixed Native Array: int[4]                      Dynamic ArrayList<Integer>
  ┌────┬────┬────┬────┐                          ┌──────────────────────────────┐
  │ 10 │ 20 │ 30 │ 40 │                          │ ArrayList Object             │
  └────┴────┴────┴────┘                          ├──────────────────────────────┤
  - Fixed capacity = 4                           │ size = 3                     │
  - Direct primitive storage                     │ elementData (Object[]) ──────┼──┐
  - Zero allocation overhead                     └──────────────────────────────┘  │
                                                                                   ▼
                                                 Heap Storage Array (Capacity = 10)
                                                 ┌──────┬──────┬──────┬───┬───┬────┐
                                                 │ 0x01 │ 0x02 │ 0x03 │   │   │    │
                                                 └───┬──────┬──────┬──┴───┴───┴────┘
                                                     ▼      ▼      ▼
                                                  Integer objects on Heap
```

| Characteristic        | Java Native Array (`T[]`)                                                         | `java.util.ArrayList<E>`                                                          |
| :-------------------- | :-------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------- |
| **Sizing**            | **Fixed length**: defined at allocation, immutable size                           | **Dynamic resizing**: grows automatically by $50\%$ when full                     |
| **Data Types**        | Holds **primitives** (`int[]`) AND **references** (`String[]`)                    | Holds **reference types only** (`ArrayList<Integer>`) via Generics                |
| **Performance**       | High; cache-friendly contiguous native memory                                     | Minor overhead due to object wrappers, pointer chasing, and resizing              |
| **Length Property**   | Uses the public final field: `arr.length`                                         | Uses the method call: `list.size()`                                               |
| **Generics Safety**   | **Covariant** (`Integer[]` IS-A `Number[]`); throws runtime `ArrayStoreException` | **Invariant** (`List<Integer>` is NOT a `List<Number>`); enforced at compile-time |
| **Memory Allocation** | Single contiguous memory chunk                                                    | Internal buffer array (`elementData`) plus overhead of heap wrapper objects       |

---

### 10. `Iterator` vs. `ListIterator`

```
                       Iterator vs. ListIterator Capability
                                         │
         ┌───────────────────────────────┴───────────────────────────────┐
         ▼                                                               ▼
     Iterator<E>                                                 ListIterator<E>
 - Applicable to ANY Collection (List, Set, Queue)            - Applicable ONLY to List implementations
 - Forward traversal ONLY (next())                            - Bidirectional traversal (next() & previous())
 - Operations: hasNext(), next(), remove()                    - Operations: add(), set(), nextIndex(), previousIndex()
 - Single starting position (head)                            - Can start at ANY index: list.listIterator(index)
```

| Feature / Method         | `Iterator<E>`                         | `ListIterator<E>`                                              |
| :----------------------- | :------------------------------------ | :------------------------------------------------------------- |
| **Scope**                | Available on all `Collection` classes | Available **only** on classes implementing `List`              |
| **Traversal Direction**  | **Forward only**                      | **Bidirectional** (Forward and Backward)                       |
| **Backward Traversal**   | ❌ No                                 | ✅ Supported via `hasPrevious()` and `previous()`              |
| **Index Inspection**     | ❌ No                                 | ✅ Supported via `nextIndex()` and `previousIndex()`           |
| **Element Modification** | ❌ Cannot modify elements             | ✅ Supported via `set(E e)` (overwrites last returned element) |
| **Element Insertion**    | ❌ Cannot insert new elements         | ✅ Supported via `add(E e)`                                    |

```java
List<String> framework = new ArrayList<>(List.of("Spring", "Hibernate", "Quarkus"));
ListIterator<String> lit = framework.listIterator();

// Forward traversal:
while (lit.hasNext()) {
    String item = lit.next();
    if (item.equals("Hibernate")) {
        lit.set("JPA / Hibernate"); // Direct replacement!
    }
}

// Backward traversal:
while (lit.hasPrevious()) {
    System.out.println(lit.previousIndex() + ": " + lit.previous());
}
```

---

### 11. Fail-Fast vs. Fail-Safe Iterators

This is one of the most heavily tested concurrency and collections concepts in enterprise technical evaluations:

```
                            Iterator Failure Strategies
                                         │
         ┌───────────────────────────────┴───────────────────────────────┐
         ▼                                                               ▼
Fail-Fast Iterators (Default JCF)                               Fail-Safe / Weakly Consistent
- Direct view over underlying collection                        - Operates on a CLONE or snapshot
- Aborts immediately upon concurrent change                     - Never throws ConcurrentModificationException
- Throws ConcurrentModificationException                        - Higher memory overhead
- Examples: ArrayList, HashSet, HashMap                         - Examples: CopyOnWriteArrayList, ConcurrentHashMap
```

#### 11.1 The Fail-Fast Mechanics (`modCount`)

Standard collections maintain an internal structural modification counter:

```java
protected transient int modCount = 0;
```

- Every time the structure of the collection changes (adding, removing, expanding capacity), `modCount` is incremented.
- When an iterator is created, it captures that value into a local field:
  ```java
  int expectedModCount = modCount;
  ```
- On every subsequent call to `it.next()` or `it.remove()`, the iterator performs an immediate check:
  ```java
  if (modCount != expectedModCount) {
      throw new ConcurrentModificationException();
  }
  ```

---

### 12. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can a `Collection` store primitive data types directly?**
  - _Answer:_ **No.** The Collections Framework operates exclusively on object references (`java.lang.Object`). When writing `list.add(10)`, the compiler uses **Autoboxing** to convert the primitive `int` to `Integer.valueOf(10)`.

- **Q2: Why is `add()` declared in `Collection`, but `get(int index)` is only in `List`?**
  - _Answer:_ Not all collections maintain positional indices. A `Set` represents an unordered bag of unique items, and a `PriorityQueue` arranges elements by dynamic heap order. Positional index access is meaningful only for ordered sequences, so `get(int index)` belongs strictly to `List`.

- **Q3: What happens if an element's state changes while it is stored in a `HashSet`?**
  - _Answer:_ If an element's fields are mutated such that its `hashCode()` changes, the element remains in its original hash bucket. When checking `set.contains(element)`, the set calculates the new hash code, looks in the wrong bucket, and reports that the element does not exist. The original element becomes an unretrievable **memory leak**.

- **Q4: What is the difference between `list.remove(1)` and `list.remove(Integer.valueOf(1))`?**
  - _Answer:_
    - `list.remove(1)` invokes the overloaded method `remove(int index)`, removing the element at **index position 1**.
    - `list.remove(Integer.valueOf(1))` invokes `remove(Object o)`, searching for and removing the **first occurrence of the object representing the value 1**.

- **Q5: Is `ConcurrentModificationException` guaranteed to be thrown only in multi-threaded environments?**
  - _Answer:_ **No.** It is frequently thrown in **single-threaded** code when a collection is modified structurally using its own API (e.g., `list.remove()`) while an active enhanced for-loop or `Iterator` is traversing it.
