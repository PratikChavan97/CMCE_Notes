# Module 06: Java Collections Framework & Generics

## Unit 6.3: Set and Map Collections

### 1. The `Set<E>` Interface Contract

A **`Set<E>`** is a collection that models the mathematical set abstraction: **it contains no duplicate elements**. More formally, sets contain no pair of elements $e_1$ and $e_2$ such that $e_1\text{.equals}(e_2) == \text{true}$.

```
                                  Set Architecture
                                         │
         ┌───────────────────────────────┼───────────────────────────────┐
         ▼                               ▼                               ▼
      HashSet                      LinkedHashSet                      TreeSet
  (Unordered Hash Table)        (Insertion-Ordered Set)        (Sorted Red-Black Tree)
  - Backed by HashMap           - Backed by LinkedHashMap      - Backed by NavigableMap
  - O(1) Search/Insert          - O(1) Search/Insert           - O(log n) Operations
  - Allows 1 null element       - Allows 1 null element        - Prohibits null elements
```

---

### 2. `HashSet` Architecture & Mechanics

`java.util.HashSet<E>` is backed internally by an instance of **`java.util.HashMap<E, Object>`**.

#### 2.1 The Backing Map Invariant

When you create a `HashSet`, elements added to the set are stored as **keys** in an internal backing map. The value associated with each key is a shared, static dummy singleton object: **`PRESENT`**.

```java
// JDK Source Code snippet inside HashSet<E>:
public class HashSet<E> extends AbstractSet<E> implements Set<E> {
    private transient HashMap<E, Object> map;
    private static final Object PRESENT = new Object(); // Shared dummy value

    public HashSet() {
        map = new HashMap<>();
    }

    public boolean add(E e) {
        // If put() returns null, the key was absent -> returns true
        return map.put(e, PRESENT) == null;
    }

    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    public boolean remove(Object o) {
        return map.remove(o) == PRESENT;
    }
}
```

#### 2.2 Key Operational Traits

- **Unordered & Non-Deterministic:** Elements are bucketed according to their `hashCode()`. There is zero guarantee regarding iteration order; the order can even change when the table rehashes.
- **Permits One Null:** Can store at most **one `null` element** (which always maps to bucket index `0`).
- **Performance:** Offers $\mathcal{O}(1)$ constant-time performance for basic operations (`add`, `remove`, `contains`), assuming a uniform hash distribution.

---

### 3. `LinkedHashSet` Architecture

`java.util.LinkedHashSet<E>` extends `HashSet<E>` directly. It is backed by a **`LinkedHashMap`**.

```
                   LinkedHashSet Dual Architecture
    ┌────────────────────────────────────────────────────────┐
    │ Hash Table (Buckets)  ──► Provides fast O(1) lookups   │
    ├────────────────────────────────────────────────────────┤
    │ Doubly-Linked List    ──► Runs through all entries to  │
    │                           preserve insertion order     │
    └────────────────────────────────────────────────────────┘

    Insertion: [ "Cherry" ] -> [ "Apple" ] -> [ "Banana" ]
    Iteration: Always outputs "Cherry", "Apple", "Banana"
```

- **Preserves Insertion Order:** While standard `HashSet` yields elements arbitrarily, `LinkedHashSet` maintains a doubly-linked list running through every node. Iteration visits elements in the exact order they were inserted.
- **Memory Overhead:** Incurs higher memory costs per node than `HashSet` to store the two additional pointer handles: `before` and `after`.

---

### 4. `TreeSet` Architecture & Self-Balancing Trees

`java.util.TreeSet<E>` implements the **`NavigableSet<E>`** interface and is backed internally by a **`java.util.TreeMap<E, Object>`** (a **Red-Black Tree**).

```
                      Red-Black Tree Structure in TreeSet
                                    [40 (B)]
                                   /        \
                            [20 (R)]        [60 (B)]
                            /      \        /      \
                         [10(B)]  [30(B)] [50(R)]  [70(R)]
```

#### 4.1 Red-Black Tree Invariants

A Red-Black tree is a self-balancing binary search tree satisfying:

1. Every node is either **Red** or **Black**.
2. The root node is always **Black**.
3. Red nodes cannot have red children (no two consecutive red nodes on any path).
4. Every path from the root to a leaf (`null`) contains an **identical number of black nodes** (Black-Height).

#### 4.2 Navigational Operations

Because elements are sorted, `TreeSet` provides range and proximity queries:

- `first()` / `last()`: Retrieves smallest / largest elements ($\mathcal{O}(\log n)$ or cached $\mathcal{O}(1)$).
- `lower(e)` / `higher(e)`: Finds the greatest element $< e$ / smallest element $> e$.
- `floor(e)` / `ceiling(e)`: Finds the greatest element $\le e$ / smallest element $\ge e$.
- `subSet(from, to)` / `headSet(to)` / `tailSet(from)`: Generates windowed views over the tree.

---

### 5. `HashSet` vs. `TreeSet` Comparison

| Dimension                 | `HashSet`                             | `TreeSet`                                          |
| :------------------------ | :------------------------------------ | :------------------------------------------------- |
| **Backing Structure**     | Hash Table (`HashMap`)                | Self-Balancing Red-Black Tree (`TreeMap`)          |
| **Element Ordering**      | **Unordered / Arbitrary**             | **Sorted** (Natural Order or Custom `Comparator`)  |
| **Time Complexity**       | $\mathcal{O}(1)$ constant time        | $\mathcal{O}(\log n)$ logarithmic time             |
| **Null Elements**         | **Allows 1 null** element             | **Prohibits null** (throws `NullPointerException`) |
| **Comparison Mechanism**  | Relies on `equals()` and `hashCode()` | Relies on `compareTo()` or `compare()`             |
| **Interface Implemented** | `Set<E>`                              | `NavigableSet<E>`, `SortedSet<E>`, `Set<E>`        |

---

### 6. The `Map<K, V>` Interface Contract

A **`Map<K, V>`** maps unique keys to values. A map cannot contain duplicate keys; each key maps to at most one value.

```
                                  Map Architecture
                                         │
         ┌───────────────────────────────┼───────────────────────────────┐
         ▼                               ▼                               ▼
      HashMap                      LinkedHashMap                      TreeMap
  (Unordered Hash Table)        (Insertion-Ordered Map)        (Sorted Red-Black Map)
  - O(1) basic ops              - O(1) basic ops               - O(log n) basic ops
  - 1 null key allowed          - 1 null key allowed           - Null keys prohibited
```

#### The Three Collection Views of a Map:

1. **`keySet()`:** Returns a `Set<K>` view of all keys in the map.
2. **`values()`:** Returns a `Collection<V>` view of all values (can contain duplicates).
3. **`entrySet()`:** Returns a `Set<Map.Entry<K, V>>` representing key-value associations.

```java
Map<String, Integer> stock = new HashMap<>();
stock.put("Apples", 50);
stock.put("Oranges", 30);

// Optimal Iteration via entrySet() - Avoids secondary get() lookups!
for (Map.Entry<String, Integer> entry : stock.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}
```

---

### 7. Internal Working of `HashMap` (JDK 8+ Deep Dive)

The internal architecture of `java.util.HashMap` is one of the most heavily tested interview domains in Java.

```
                           HashMap Node Storage Model
        Node<K, V>[] table
       ┌─────┐
   [0] │     │ ──► Node { hash, key, value, next } ──► null
       ├─────┤
   [1] │     │ ──► null
       ├─────┤
   [2] │     │ ──► Node ──► Node ──► Node ──► Node (Linked List: Collision Chain)
       ├─────┤
       │ ... │
       ├─────┤
   [9] │     │ ──► TreeNode { parent, left, right, red } (Treeified: Red-Black Tree)
       └─────┘
```

#### 7.1 Core Parameters

1. **Initial Capacity:** The number of buckets in the hash table when created. Default is **$16$** ($2^4$). Must always be a **power of two**.
2. **Load Factor ($\alpha$):** A measure of how full the hash table is allowed to get before its capacity is automatically doubled. Default is **$0.75$**.
3. **Threshold:** The entry count that triggers a resize:
   $$\text{Threshold} = \text{Capacity} \times \text{Load Factor} = 16 \times 0.75 = 12$$

#### 7.2 Why Must Capacity Be a Power of Two?

To determine an element's bucket index from its hash, the JVM must perform a modulo operation: $\text{index} = \text{hash} \pmod{\text{capacity}}$.

Modulo arithmetic using the division operator (`%`) consumes considerable CPU cycles. When $\text{capacity} = 2^n$, the modulo operation is mathematically identical to a bitwise **AND (`&`)** with $\text{capacity} - 1$:

$$\text{hash} \pmod{\text{capacity}} \iff \text{hash} \ \& \ (\text{capacity} - 1)$$

Bitwise masking runs in a single CPU clock cycle, drastically speeding up bucket resolution.

#### 7.3 Bit-Shift Hash Perturbation Function

To protect against poorly implemented `hashCode()` methods that only vary in upper bits, `HashMap` mixes the higher bits of the hash into the lower bits before computing the bucket index:

```java
static final int hash(Object key) {
    int h;
    // XORs the high 16 bits with the low 16 bits
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

#### 7.4 Java 8 Treeification Thresholds

In Java 7, collisions within the same bucket formed a linear linked list. If an attacker engineered keys with identical hash codes, lookups degraded to $\mathcal{O}(n)$, making servers vulnerable to Hash-DoS attacks.

Java 8 addresses this by dynamically converting collision bins between linked lists and balanced trees:

- **`TREEIFY_THRESHOLD = 8`:** When a collision chain reaches **8 nodes** AND the total capacity is at least **64**, the bucket converts from a singly-linked list (`Node<K,V>`) into a Red-Black Tree (`TreeNode<K,V>`), reducing lookup time from $\mathcal{O}(n)$ to $\mathcal{O}(\log n)$.
- **`MIN_TREEIFY_CAPACITY = 64`:** If a bucket hits 8 elements but total capacity is $< 64$, the table **resizes** instead of treeifying to prevent premature tree overhead.
- **`UNTREEIFY_THRESHOLD = 6`:** During a resize, if a treeified bucket's node count shrinks to **6 or fewer**, it converts back into a singly-linked list to reduce memory overhead.

```
                      Bucket Structural Transitions
                         Collision Count >= 8
                         (and Capacity >= 64)
          Singly-Linked List ════════════════════► Red-Black Tree
             O(n) search                         O(log n) search
                             ◄════════════════════
                             Node Count Drops <= 6
                              (during resize split)
```

---

### 8. `LinkedHashMap` & LRU Cache Implementation

`LinkedHashMap<K, V>` extends `HashMap<K, V>` by maintaining a doubly-linked list through all its entries.

#### 8.1 Ordering Modes

1. **Insertion-Order (Default):** Iteration yields keys in the order they were inserted.
2. **Access-Order:** Keys are reordered based on when they were last accessed via `get()` or `put()`. The most recently accessed entry moves to the end of the list.

```java
// Constructing an access-order LinkedHashMap:
Map<String, String> map = new LinkedHashMap<>(16, 0.75f, true); // true = access-order
```

#### 8.2 Building a Fixed-Size LRU (Least Recently Used) Cache

By overriding the `removeEldestEntry()` hook method, `LinkedHashMap` can function as an in-memory LRU cache:

```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int maxEntries;

    public LRUCache(int maxEntries) {
        super(maxEntries, 0.75f, true); // Enable access-order
        this.maxEntries = maxEntries;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // Automatically evicts the least recently accessed entry when capacity exceeds limit
        return size() > maxEntries;
    }
}
```

---

### 9. `TreeMap` & Sorted Mappings

`java.util.TreeMap<K, V>` implements `NavigableMap<K, V>` using an underlying Red-Black Tree.

- **Key Invariant:** Keys are ordered either according to their **natural ordering** (implementing `Comparable`) or via an explicit **`Comparator`** passed to the constructor.
- **Prohibits Null Keys:** Attempting to insert a `null` key throws **`NullPointerException`**, because the tree must call `compareTo()` or `compare()` on the key to balance itself.
- **Time Complexity:** Guarantees $\mathcal{O}(\log n)$ time for `containsKey`, `get`, `put`, and `remove`.

---

### 10. Legacy Container: `Hashtable`

Introduced in JDK 1.0, `Hashtable` is an obsolete collection class:

- **Method Synchronization:** Every public read and write method is marked `synchronized` (`public synchronized V get(Object key)`).
- **Null Prohibited:** Does not permit `null` keys or `null` values (throws `NullPointerException`).
- **Superseded:** Replaced by `HashMap` for non-concurrent applications, and by `ConcurrentHashMap` for high-throughput concurrent environments.

---

### 11. Comparison: `HashMap` vs. `Hashtable` vs. `TreeMap`

| Feature                       | `HashMap`                      | `Hashtable`                 | `TreeMap`                          |
| :---------------------------- | :----------------------------- | :-------------------------- | :--------------------------------- |
| **Underlying Data Structure** | Array of Bins (List + Tree)    | Array of Bins (Linked List) | Red-Black Tree                     |
| **Key Ordering**              | None (Unordered)               | None (Unordered)            | **Sorted** (Natural or Comparator) |
| **Time Complexity**           | $\mathcal{O}(1)$ average       | $\mathcal{O}(1)$ average    | $\mathcal{O}(\log n)$ guaranteed   |
| **Null Keys / Values**        | **1 null key**, many null vals | **Zero nulls** allowed      | **No null keys**, many null vals   |
| **Thread Safety**             | ❌ Unsynchronized (Fast)       | ✅ Synchronized (Slow)      | ❌ Unsynchronized                  |
| **Iteration Mechanism**       | Fail-Fast Iterator             | Enumeration & Iterator      | Fail-Fast Iterator                 |

---

### 12. Hashing Fundamentals & Collision Handling

**Hashing** maps variable-length input data into a fixed-size integer value (hash code) using a mathematical algorithm.

#### 12.1 The Pigeonhole Principle & Collisions

Because the domain of possible keys is effectively infinite (e.g., all conceivable `String` objects) while the output range of a 32-bit integer `hashCode()` is finite ($2^{32} \approx 4.29 \times 10^9$), **collisions are mathematically inevitable**:

$$\exists \ x, y \quad \text{such that} \quad x \ne y \ \land \ x\text{.hashCode}() == y\text{.hashCode}()$$

```
Collision Resolution Approaches:
1. Separate Chaining (Used by Java HashMap)
   Buckets maintain secondary collections (linked lists or balanced trees)
   to store colliding entries.

2. Open Addressing (Linear/Quadratic Probing)
   When a collision occurs, the algorithm searches for the next available empty
   slot directly within the primary bucket array.
```

---

### 13. High-Yield Concurrency: `ConcurrentHashMap`

In multithreaded systems, using a plain `HashMap` without external locks can cause data corruption or race conditions. Wrapping it via `Collections.synchronizedMap(map)` serializes all access behind a single lock, creating a performance bottleneck.

`java.util.concurrent.ConcurrentHashMap` solves this through concurrent lock management.

```
            Collections.synchronizedMap             ConcurrentHashMap (Java 8+)
              [ Single Global Lock ]             [ CAS + Synchronized Bucket Head ]
            ┌────────────────────────┐              ┌────┐
            │  Entire Map Blocked    │         [0]  │    │ ◄── Lock ONLY Bucket 0
            │  During ANY read/write │              ├────┤
            └────────────────────────┘         [1]  │    │ ◄── Reads proceed LOCK-FREE!
                                                    ├────┤
                                               [2]  │    │ ◄── Lock ONLY Bucket 2
```

#### 13.1 Java 7 vs. Java 8 Concurrency Models

- **Java 7 (Segment-Level Locking):** Used an array of 16 internal `Segment` locks (re-entrant locks). Threads could write concurrently to different segments, but operations within the same segment were blocked.
- **Java 8+ (CAS + Synchronized Bucket Locks):**
  - **Reads are 100% Lock-Free:** `get()` operations involve volatile reads without locks.
  - **Lock-Free Inserts via CAS:** If an array bucket is empty, the first entry is inserted using atomic **Compare-And-Swap (CAS)** with zero locks.
  - **Bucket-Level Synchronization:** If a bucket already contains entries, only the **head node of that specific bucket** is locked using a localized `synchronized(node)` block. Unrelated buckets remain open for concurrent access.

#### 13.2 Null Restrictions in `ConcurrentHashMap`

`ConcurrentHashMap` **prohibits `null` keys and `null` values entirely** (throws `NullPointerException`).

> ⚠️ **Why Nulls Are Prohibited in Concurrent Collections (Doug Lea Invariant):**
> If `map.get(key)` returns `null` in a single-threaded map, you can disambiguate whether the key is missing or explicitly mapped to `null` by calling `map.containsKey(key)`.
> In a concurrent map, another thread could insert or remove the key between those two calls, creating a race condition. To maintain consistency, `null` is prohibited.

---

### 14. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What happens if an object's `hashCode()` method always returns a constant, like `return 1;`?**
  - _Answer:_ The map will still function, but **performance degrades to $\mathcal{O}(n)$ (or $\mathcal{O}(\log n)$ in Java 8)**.
  - _Explanation:_ Every inserted entry maps to bucket index `1`. All entries form a single collision chain, turning the hash table into a linear linked list (or Red-Black tree if treeified), neutralizing the $\mathcal{O}(1)$ lookup benefit of hashing.

- **Q2: Why does `HashSet` evaluate uniqueness using `equals()` instead of `==`?**
  - _Answer:_ `==` checks whether two variables point to the exact same memory address on the Heap. Two distinct object instances (such as `new String("A")` and `new String("A")`) have different heap addresses, but represent identical content. By evaluating `equals()`, `HashSet` enforces logical uniqueness.

- **Q3: What causes an infinite loop in `HashMap.get()` in older Java versions?**
  - _Answer:_ In **Java 7**, when multiple threads concurrently resized a `HashMap`, the collision chain reversal logic could produce a **cyclic circular linked list** ($A \rightarrow B \rightarrow A$). Subsequent calls to `get()` would loop infinitely across this cycle, pegging CPU usage at 100%. Java 8 eliminated this issue by preserving node order during resizes rather than reversing chains.

- **Q4: Can we use a mutable object as a key in a `HashMap`?**
  - _Answer:_ **Technically yes, but it is a major design risk.** If an object's fields change after it is inserted as a key, its `hashCode()` changes with them. When querying `map.get(key)`, the map calculates the updated hash, directing lookup to the wrong bucket and making the original entry unretrievable (causing a memory leak).

- **Q5: Does `ConcurrentHashMap` throw `ConcurrentModificationException` during iteration?**
  - _Answer:_ **No.** The iterator of `ConcurrentHashMap` is **weakly consistent**. It operates on a snapshot of the data structure, can safely reflect concurrent modifications without locking the entire table, and never throws `ConcurrentModificationException`.
