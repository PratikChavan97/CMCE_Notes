# Module 06: Java Collections Framework & Generics

## Unit 6.2: List and Queue Collections

---

### 1. `ArrayList` Architecture & Internal Mechanics

`java.util.ArrayList<E>` is a resizable-array implementation of the `List<E>` interface. It is the most commonly used collection in modern Java applications due to its optimal spatial locality and predictable constant-time random access.

```
                              ArrayList Memory Topography
Stack Frame                                      JVM Heap Memory
┌─────────────────────────┐                      ┌───────────────────────────────────────────────┐
│ ArrayList ref: 0x8100   │ ───────────────────► │ ArrayList Object Instance (0x8100)            │
└─────────────────────────┘                      ├───────────────────────────────────────────────┤
                                                 │ int size = 4                                  │
                                                 │ Object[] elementData: 0x9500 ──────┐          │
                                                 └────────────────────────────────────┼──────────┘
                                                                                      │
                                                 ┌────────────────────────────────────┘
                                                 ▼
                                                 ┌───────────────────────────────────────────────┐
                                                 │ Contiguous Heap Array: Object[10] (0x9500)     │
                                                 ├──────┬──────┬──────┬──────┬──────┬──────┬────┤
                                                 │ [0]  │ [1]  │ [2]  │ [3]  │ [4]  │ [5]  │... │
                                                 ├──────┼──────┼──────┼──────┼──────┼──────┼────┤
                                                 │0x011 │0x022 │0x033 │0x044 │ null │ null │... │
                                                 └──────┴──────┴──────┴──────┴──────┴──────┴────┘
                                                   "A"    "B"    "C"    "D"  ◄── capacity = 10 ──►
```

#### 1.1 Capacity vs. Size

- **`size`:** The actual number of elements currently stored in the list (`list.size()`).
- **`capacity`:** The allocated length of the underlying `elementData[]` buffer on the Heap.

#### 1.2 Lazy Initialization (Java 8+)

When instantiated via the default constructor `new ArrayList<>()`:

1. It does **not** allocate a 10-element array immediately.
2. It assigns `elementData` to a shared static empty array constant: `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`.
3. Upon adding the **first element** (`list.add(e)`), the backing array is lazily expanded to the default initial capacity of **10**.

#### 1.3 Dynamic Growth Formula ($1.5\times$ Expansion)

When an element is appended and `size + 1 > capacity`, the internal array resizes using the standard bitwise shift formula:

$$\text{New Capacity} = \text{Old Capacity} + (\text{Old Capacity} \gg 1) = \text{Old Capacity} \times 1.5$$

```java
// How the JDK resizes the array:
int newCapacity = oldCapacity + (oldCapacity >> 1);
elementData = Arrays.copyOf(elementData, newCapacity); // Emits native System.arraycopy
```

- **Expansion Sequence:** $10 \longrightarrow 15 \longrightarrow 22 \longrightarrow 33 \longrightarrow 49 \dots$
- **Performance Impact:** Dynamic growth requires allocating a brand-new contiguous array on the Heap followed by a native memory block transfer (`System.arraycopy`).
- **Optimization:** Always provide an initial capacity hint (`new ArrayList<>(expectedSize)`) or invoke `ensureCapacity(minCapacity)` if the element volume is known upfront.

---

### 2. `LinkedList` Architecture & Internal Mechanics

`java.util.LinkedList<E>` is a **doubly-linked list** implementation that also implements both `List<E>` and `Deque<E>`.

```
                              LinkedList Memory Topography
              ┌─────────────────────────────────────────────────────────────┐
              │                   LinkedList (Head/Tail)                    │
              │  Node<E> first = 0xAA01              Node<E> last = 0xCC03  │
              └───────────────┬─────────────────────────────▲───────────────┘
                              │                             │
                              ▼                             │
┌─────────────────────────────────┐   ┌─────────────────────────────────┐   ┌─────────────────────────────────┐
│ Node 0xAA01                     │   │ Node 0xBB02                     │   │ Node 0xCC03                     │
├─────────────────────────────────┤   ├─────────────────────────────────┤   ├─────────────────────────────────┤
│ prev: null                      │   │ prev: 0xAA01                    │   │ prev: 0xBB02                    │
│ item: 0x1111 ──► [String: "A"]  │   │ item: 0x2222 ──► [String: "B"]  │   │ item: 0x3333 ──► [String: "C"]  │
│ next: 0xBB02 ───────────────────┼──►│ next: 0xCC03 ───────────────────┼──►│ next: null                      │
└─────────────────────────────────┘   └─────────────────────────────────┘   └─────────────────────────────────┘
```

#### 2.1 The Internal `Node<E>` Class

Every element added to a `LinkedList` requires instantiating an independent `Node<E>` object on the Heap:

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

#### 2.2 Memory Overhead

- Each `Node` object incurs a 16-byte object header, two 4/8-byte pointers (`prev`, `next`), and a 4/8-byte payload pointer (`item`).
- Allocates roughly **24 to 32 bytes of heap metadata per single element** (excluding the element itself).
- Scatter-allocates nodes across non-contiguous heap regions, causing severe **CPU cache misses** during linear traversal compared to the spatial locality of `ArrayList`.

---

### 3. `Vector` & `Stack` (Legacy Collections)

#### 3.1 `Vector`

- Introduced in **JDK 1.0** (retro-fitted into the Collections Framework in JDK 1.2).
- Internally backed by a contiguous `Object[] elementData` identical to `ArrayList`.
- **Synchronization:** Almost **every public method is marked `synchronized`** (`public synchronized boolean add(E e)`, `public synchronized E get(int index)`).
- **Performance Cost:** Acquires and releases the object monitor lock on every operation, creating heavy synchronization overhead in single-threaded environments.
- **Growth Formula:** Expands by **$2.0\times$ (doubles)** by default if `capacityIncrement <= 0`, consuming significantly more spare memory than `ArrayList`.

#### 3.2 `Stack`

- Extends `Vector` directly to provide LIFO operations (`push()`, `pop()`, `peek()`, `search()`).
- **Architectural Flaw:** Because it inherits from `Vector`, it violates the abstraction of a stack. Client code can invoke arbitrary index-based vector mutations (e.g., `stack.insertElementAt("Hacked", 0)` or `stack.remove(2)`), breaking LIFO invariants.
- **Modern Replacement:** **`ArrayDeque`** is the officially recommended replacement for stack implementations:
  ```java
  Deque<String> stack = new ArrayDeque<>();
  stack.push("A");
  String top = stack.pop();
  ```

---

### 4. Comprehensive Comparison: `ArrayList` vs. `LinkedList` vs. `Vector`

| Characteristic                       | `ArrayList`                                    | `LinkedList`                                         | `Vector`                                       |
| :----------------------------------- | :--------------------------------------------- | :--------------------------------------------------- | :--------------------------------------------- |
| **Backing Structure**                | Contiguous Resizable Array (`Object[]`)        | Doubly-Linked List (`Node<E>`)                       | Contiguous Resizable Array (`Object[]`)        |
| **Random Access (`get(i)`)**         | **$\mathcal{O}(1)$ Instant**                   | $\mathcal{O}(n)$ Pointer Hop ($\approx n/2$)         | **$\mathcal{O}(1)$ Instant**                   |
| **Insert/Delete at End**             | $\mathcal{O}(1)$ Amortized                     | $\mathcal{O}(1)$ Constant                            | $\mathcal{O}(1)$ Amortized                     |
| **Insert/Delete at Head**            | $\mathcal{O}(n)$ (Shifts all elements)         | $\mathcal{O}(1)$ Pointer Update                      | $\mathcal{O}(n)$ (Shifts all elements)         |
| **Insert/Delete at Arbitrary Index** | $\mathcal{O}(n)$ (Array Shift via `arraycopy`) | $\mathcal{O}(n)$ (Traversal) + $\mathcal{O}(1)$ Link | $\mathcal{O}(n)$ (Array Shift via `arraycopy`) |
| **Memory Overhead**                  | Low (unused spare buffer slots only)           | **Extremely High** (3 pointers per Node)             | Low (unused spare buffer slots only)           |
| **CPU Cache Locality**               | **Excellent** (Contiguous RAM blocks)          | **Poor** (Scattered heap references)                 | **Excellent** (Contiguous RAM blocks)          |
| **Thread Safety**                    | ❌ Unsynchronized (Fast)                       | ❌ Unsynchronized (Fast)                             | ✅ **Thread-Safe** (`synchronized`)            |
| **Default Capacity / Growth**        | 10 default; expands by **$1.5\times$**         | Starts empty; grows node by node                     | 10 default; expands by **$2.0\times$**         |

---

### 5. Core List Operations & Algorithmic Patterns

#### 5.1 Insertion & Deletion Mechanics

```java
List<String> list = new ArrayList<>(List.of("A", "B", "D"));

// 1. Positional Insertion (Shifts elements from index to size-1 right by 1)
list.add(2, "C"); // ["A", "B", "C", "D"] - O(n)

// 2. Positional Deletion (Shifts elements left by 1)
list.remove(1);   // Removes "B" -> ["A", "C", "D"] - O(n)

// 3. Object-based Deletion (Linear search via .equals(), then shifts left)
list.remove("C"); // Removes first occurrence -> ["A", "D"] - O(n)
```

#### 5.2 Searching & Positional Invariants

- `indexOf(Object o)`: Returns first zero-based index matching via `(o == null ? get(i) == null : o.equals(get(i)))`, or `-1` if absent ($\mathcal{O}(n)$).
- `lastIndexOf(Object o)`: Scans backwards from `size - 1` to `0`.
- `subList(int fromIndex, int toIndex)`: Returns a **structural view** of the specified range $[from, to)$.
  > ⚠️ **The `subList` Mutation Trap:** The sublist is **backed directly by the original list**. Modifying the sublist mutates the master list. Conversely, structurally modifying the master list directly (adding/removing elements) invalidates the sublist view, causing subsequent sublist operations to throw **`ConcurrentModificationException`**.

---

### 6. Searching, Sorting, and the Dual-Pivot Quicksort / TimSort Pipeline

Java collections rely on modern hybrid sorting algorithms implemented in `java.util.Collections.sort()` and `List.sort()`:

```
                            Java Sorting Engine Architecture
                                           │
         ┌─────────────────────────────────┴─────────────────────────────────┐
         ▼                                                                   ▼
Primitive Arrays (Arrays.sort(int[]))                      Object Lists & Arrays (List.sort())
Dual-Pivot Quicksort (Vladimir Yaroslavskiy)               TimSort (Tim Peters Algorithm)
- Worst: O(n^2), Average: O(n log n)                       - Worst: O(n log n), Best: O(n)
- In-place (O(1) memory), NOT STABLE                       - Adaptive, requires O(n) memory, STABLE
```

#### 6.1 TimSort Mechanics

- **Adaptive:** Identifies natural runs of already sorted data in the input list.
- **Hybrid:** Merges natural runs using an optimized **Merge Sort**, switching to **Binary Insertion Sort** when run sizes drop below 32 elements.
- **Stable:** Preserves the relative spatial order of elements that evaluate as equal under comparison.

#### 6.2 Binary Search (`Collections.binarySearch`)

- Requires the input `List` to be **strictly sorted** in ascending order according to the passed comparator/natural order.
- Evaluates in $\mathcal{O}(\log n)$ time for random access lists (`ArrayList`), but degrades to $\mathcal{O}(n)$ for sequential access lists (`LinkedList`) due to link-hopping.
- **Return Value Contract:**
  - Returns the $\ge 0$ zero-based index if the target is found.
  - If absent, returns a negative value: **$\text{Insertion Point} = -(\text{return value}) - 1$**.

---

### 7. Iteration Paradigms: `Iterator` vs. `ListIterator`

```
                                Cursor Navigation
                 Iterator: Forward Only ──────────────►
        ┌───────┬───────┬───────┬───────┬───────┬───────┐
        │  [0]  │  [1]  │  [2]  │  [3]  │  [4]  │  [5]  │
        └───────┴───────┴───────┴───────┴───────┴───────┘
        ◄───────────── ListIterator: Bidirectional ────────────►
```

| Trait                    | `Iterator<E>`                                | `ListIterator<E>`                                                           |
| :----------------------- | :------------------------------------------- | :-------------------------------------------------------------------------- |
| **Applicable Targets**   | Any `Collection<E>` (`List`, `Set`, `Queue`) | **`List<E>` implementations only**                                          |
| **Directionality**       | **Forward only** (`hasNext()`, `next()`)     | **Bidirectional** (`hasPrevious()`, `previous()`)                           |
| **Index Inspection**     | ❌ Cannot inspect indices                    | ✅ Inspects indices via `nextIndex()`, `previousIndex()`                    |
| **Element Modification** | Supports deletion only (`remove()`)          | Supports deletion (`remove()`), update (`set(e)`), and insertion (`add(e)`) |

```java
List<String> tokens = new ArrayList<>(List.of("alpha", "beta", "gamma"));
ListIterator<String> lit = tokens.listIterator();

while (lit.hasNext()) {
    String val = lit.next();
    if (val.equals("beta")) {
        lit.set("BETA-MODIFIED"); // In-place replacement
        lit.add("INSERTED-DELTA"); // Inserts immediately before next element
    }
}
```

---

### 8. The `Queue<E>` Architecture

A `Queue` holds elements temporarily prior to processing. It enforces distinct operational contracts depending on how unexpected states (full or empty conditions) are handled:

```
                            Queue Operation Contracts
                                        │
         ┌──────────────────────────────┴──────────────────────────────┐
         ▼                                                             ▼
Throws Exception on Failure                                   Returns Special Value on Failure
- add(e)   : Throws IllegalStateException (if full)           - offer(e) : Returns false (if full)
- remove() : Throws NoSuchElementException (if empty)         - poll()   : Returns null (if empty)
- element(): Throws NoSuchElementException (if empty)         - peek()   : Returns null (if empty)
```

---

### 9. `PriorityQueue` Architecture & Heap Mechanics

`java.util.PriorityQueue<E>` is an unbounded priority queue based on a **Priority Min-Heap** (represented internally as a contiguous, balanced binary tree in an array: `Object[] queue`).

```
                    Logical Min-Heap Binary Tree
                                [10]
                               /    \
                            [20]    [15]
                            /  \    /
                         [40] [50] [30]

               Corresponding Physical Array Representation
             ┌────┬────┬────┬────┬────┬────┬───────┐
  Index:     │ 0  │ 1  │ 2  │ 3  │ 4  │ 5  │ 6 ... │
  Value:     │ 10 │ 20 │ 15 │ 40 │ 50 │ 30 │ null  │
             └────┴────┴────┴────┴────┴────┴───────┘
```

#### 9.1 Array-to-Tree Pointer Mathematics

For any element positioned at array index $i$:

- **Left Child Index:** $2i + 1$
- **Right Child Index:** $2i + 2$
- **Parent Index:** $\lfloor (i - 1) / 2 \rfloor$

#### 9.2 Key Invariants of `PriorityQueue`

1. **Ordering:** The head of the queue is always the **least element** with respect to the specified ordering (Natural Ordering via `Comparable` or explicit `Comparator`).
2. **Time Complexity:**
   - Enqueue (`offer(e)`): $\mathcal{O}(\log n)$ via upward heapify (_sift-up_).
   - Dequeue (`poll()`): $\mathcal{O}(\log n)$ via downward heapify (_sift-down_).
   - Inspection (`peek()`): **$\mathcal{O}(1)$ instant access** (reads `queue[0]`).
3. **Null Prohibited:** Does **not** permit `null` elements (throws `NullPointerException` because it must compare elements to build the heap).
4. **Non-Thread-Safe:** For thread-safe concurrent variants, use `java.util.concurrent.PriorityBlockingQueue`.

---

### 10. `Deque` & `ArrayDeque` Architecture

The `Deque<E>` (Double-Ended Queue) interface supports element insertions and removals at both ends.

`java.util.ArrayDeque<E>` is a resizable, **circular ring-buffer array** implementation:

```
                            ArrayDeque Circular Buffer
                          head = 6, tail = 2, capacity = 8
                                  ┌───┬───┐
                       tail ────► │ 2 │   │ ◄── head (elements wrap around!)
                                  ├───┼───┤
                                  │ 1 │ 7 │
                                  ├───┼───┤
                                  │ 0 │ 6 │
                                  └───┴───┘
```

#### 10.1 Circular Index Mathematics

ArrayDeque wraps head and tail pointers around the end of the backing array using bitwise masking:
$$\text{head} = (\text{head} - 1) \ \& \ (\text{elements.length} - 1)$$
$$\text{tail} = (\text{tail} + 1) \ \& \ (\text{elements.length} - 1)$$

#### 10.2 Architectural Dominance of `ArrayDeque`

- **Zero Allocation on Push:** Unlike `LinkedList`, which must allocate a `Node` object on the Heap for every addition, `ArrayDeque` writes primitive object pointers into an existing pre-allocated array.
- **Stack Operations:** Outperforms `java.util.Stack` because it is unsynchronized.
- **Queue Operations:** Outperforms `LinkedList` because it incurs zero garbage collection churn from temporary node wrappers and exhibits spatial cache locality.

---

### 11. Object Ordering: `Comparable` vs. `Comparator`

```
                          Object Ordering Architectures
                                        │
         ┌──────────────────────────────┴──────────────────────────────┐
         ▼                                                             ▼
java.lang.Comparable<T>                                      java.util.Comparator<T>
(Natural / Default Ordering)                                 (Custom / External Ordering)
• Package: java.lang                                         • Package: java.util
• Modified target class itself (intrusive)                   • Separate ordering class/lambda (non-intrusive)
• Method: int compareTo(T o)                                 • Method: int compare(T o1, T o2)
• Single defined sort sequence                               • Unlimited multiple sort strategies
```

#### 11.1 Implementing `Comparable<T>` (Natural Order)

Used when a domain class has an obvious, single standard sort order:

```java
public class Student implements Comparable<Student> {
    private final int rollNumber;
    private final String name;

    public Student(int rollNumber, String name) {
        this.rollNumber = rollNumber;
        this.name = name;
    }

    @Override
    public int compareTo(Student other) {
        // Ascending sort by rollNumber
        return Integer.compare(this.rollNumber, other.rollNumber);
    }
}
```

#### 11.2 Implementing `Comparator<T>` (Custom Strategy Pattern)

Used when sorting requirements vary (e.g., sort by name, then sort by salary descending):

```java
import java.util.Comparator;

public class StudentComparators {
    // Strategy 1: Classical Comparator Implementation
    public static final Comparator<Student> BY_NAME = new Comparator<Student>() {
        @Override
        public int compare(Student s1, Student s2) {
            return s1.getName().compareTo(s2.getName());
        }
    };

    // Strategy 2: Modern Java 8+ Fluent Comparator Composition
    public static final Comparator<Student> BY_NAME_THEN_ROLL =
        Comparator.comparing(Student::getName)
                  .thenComparingInt(Student::getRollNumber)
                  .reversed(); // Inverts overall sort sequence
}
```

#### 11.3 Master Comparison Matrix

| Metric                   | `Comparable<T>`                               | `Comparator<T>`                                                 |
| :----------------------- | :-------------------------------------------- | :-------------------------------------------------------------- |
| **Package**              | `java.lang`                                   | `java.util`                                                     |
| **Method Name**          | `int compareTo(T o)`                          | `int compare(T o1, T o2)`                                       |
| **Number of Arguments**  | **1 parameter** (compares `this` against `o`) | **2 parameters** (compares `o1` against `o2`)                   |
| **Modifications Needed** | Requires modifying the target source code     | Requires **zero changes** to target domain entity               |
| **Sort Count**           | Only **one** natural order                    | **Multiple** sorting strategies can be defined                  |
| **Invocation Syntax**    | `Collections.sort(list)`                      | `Collections.sort(list, comparator)` or `list.sort(comparator)` |

---

### 12. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What is the output of the following removal operation?**

  ```java
  List<Integer> list = new ArrayList<>(List.of(10, 20, 30, 40));
  list.remove(2);
  System.out.println(list);
  ```

  - _Answer:_ `[10, 20, 40]`
  - _Explanation:_ The `List` interface exposes two overloaded removal methods: `remove(int index)` and `remove(Object o)`. When passing a literal integer `2`, the compiler resolves to the primitive parameter `remove(int index)` due to primitive widening rules. It removes the element at **index 2** (which is `30`), **not** the object `2`. To remove by object value, cast it explicitly: `list.remove(Integer.valueOf(2))`.

- **Q2: Why does iterating over an `ArrayList` using an index loop perform well, while doing the same on a `LinkedList` results in catastrophic performance?**
  - _Answer:_
    - `ArrayList.get(i)` is an $\mathcal{O}(1)$ operation that accesses contiguous RAM directly via array indexing: $\text{Base} + (i \times \text{scale})$. A loop of size $n$ runs in linear time $\mathcal{O}(n)$.
    - `LinkedList.get(i)` is an $\mathcal{O}(n)$ operation that must traverse node pointers sequentially from the head or tail. Evaluating `list.get(i)` inside an $n$-iteration loop results in **quadratic time complexity: $\mathcal{O}(n^2)$**.

- **Q3: What happens if an element inserted into a `PriorityQueue` does not implement `Comparable` and no `Comparator` is provided?**
  - _Answer:_ Throws a **`java.lang.ClassCastException`** at runtime during the first insertion attempt when the queue attempts to cast the element to `Comparable`.

- **Q4: Can `LinkedList` store `null` elements? What about `PriorityQueue`?**
  - _Answer:_
    - `LinkedList` **can store arbitrary `null` elements**, as it simply sets `node.item = null`.
    - `PriorityQueue` **prohibits `null` elements** and throws `NullPointerException` immediately, because it must execute comparison checks (`compareTo()` or `compare()`) during sift-up and sift-down heap reorganizations.

- **Q5: What is the difference between `Arrays.asList(array)` and `List.of(elements)`?**
  - _Answer:_
    - `Arrays.asList()` returns a **fixed-size, mutable-element view** backed directly by the original array. Elements can be updated (`set()`), which modifies the original array, but elements cannot be added or removed (`add()` or `remove()` throws `UnsupportedOperationException`). It permits `null` values.
    - `List.of()` (Java 9+) returns a **truly unmodifiable, immutable list**. Any mutation attempt (`set()`, `add()`, `remove()`) throws `UnsupportedOperationException`. It **prohibits `null` elements** entirely (throws `NullPointerException` upon creation).
