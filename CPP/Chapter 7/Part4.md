# Module 07: Standard Template Library (STL)

## Unit 7.3: Associative and Unordered Containers

Associative and unordered containers manage elements using **key-based retrieval** rather than sequential index positions.

```
                       Key-Based STL Containers
                                  |
         +------------------------+------------------------+
         |                                                 |
Associative Containers                            Unordered Containers
(Ordered / Sorted)                                (Hash Tables / O(1))
- Underlying: Self-Balancing BST (Red-Black Tree)  - Underlying: Bucketed Hash Tables
- Complexity: O(log n)                            - Complexity: O(1) avg, O(n) worst
- Elements sorted by strict weak ordering         - Elements stored in arbitrary order
         |                                                 |
   +-----+-----+                                     +-----+-----+
   |           |                                     |           |
Unique Keys  Duplicate Keys                       Unique Keys  Duplicate Keys
 - std::set   - std::multiset                      - unordered_set   - unordered_multiset
 - std::map   - std::multimap                      - unordered_map   - unordered_multimap
```

---

### 1. Underlying Data Structures & Architectural Contrast

| Dimension                   | Associative Containers (`std::set`, `std::map`, etc.)       | Unordered Containers (`std::unordered_*`)                              |
| :-------------------------- | :---------------------------------------------------------- | :--------------------------------------------------------------------- |
| **Internal Data Structure** | **Self-Balancing Binary Search Tree** (Red-Black Tree)      | **Hash Table** (Buckets with closed addressing/chaining)               |
| **Element Ordering**        | **Strictly Sorted** (Default: `std::less<Key>`)             | **Arbitrary / Unsorted** (Depends on hash function)                    |
| **Search / Insert / Erase** | **Guaranteed $\mathcal{O}(\log n)$** (Worst-case)           | **Average $\mathcal{O}(1)$**, Worst-case $\mathcal{O}(n)$ (collisions) |
| **Key Requirements**        | Needs `operator<` or custom comparator functor              | Needs `std::hash<Key>` and `operator==` (equality)                     |
| **Iterator Category**       | **Bidirectional Iterator**                                  | **Forward Iterator**                                                   |
| **Header File**             | `<set>` and `<map>`                                         | `<unordered_set>` and `<unordered_map>`                                |
| **Memory Overhead**         | 3 pointers (`parent`, `left`, `right`) + 1 color bit / node | Array of bucket pointers + singly-linked node list                     |

---

### 2. Ordered Associative Containers

#### 2.1 `std::set` (Unique Sorted Keys)

- Stores unique keys in sorted order.
- Elements **cannot be modified in-place** once inserted (`*it` is `const Key&`), because changing a key's value would corrupt the search tree invariants. To "modify" a key, you must erase it and insert the new value.

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> s = {30, 10, 20, 10, 50}; // Duplicate '10' is discarded
    s.insert(40);

    for (const auto& elem : s) {
        std::cout << elem << " "; // Output: 10 20 30 40 50 (Sorted automatically)
    }

    // Lookup: O(log n)
    auto it = s.find(20);
    if (it != s.end()) {
        std::cout << "\nFound: " << *it;
    }
    return 0;
}
```

#### 2.2 `std::multiset` (Duplicate Sorted Keys)

- Similar to `std::set`, but allows **multiple equivalent keys**.
- Insertion never fails; duplicates are preserved in balanced tree nodes.

```cpp
std::multiset<int> ms = {10, 20, 10, 30, 20};
// Elements: 10 10 20 20 30
std::cout << "Count of 10: " << ms.count(10) << "\n"; // Output: 2

// Erasing by value removes ALL duplicates:
ms.erase(10); // Removes BOTH instances of 10!

// To erase only ONE instance:
auto it = ms.find(20); // Finds the first instance
if (it != ms.end()) ms.erase(it); // Erases only the targeted node
```

#### 2.3 `std::map` (Sorted Key-Value Pairs with Unique Keys)

- Stores key-value mappings as `std::pair<const Key, Value>`.
- Keys are unique and immutable; mapped values can be altered.

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<int, std::string> empMap;

    // Insertion Method 1: std::pair or make_pair
    empMap.insert({101, "Aarav"});

    // Insertion Method 2: operator[]
    empMap[102] = "Diya";
    empMap[102] = "Ananya"; // Overwrites "Diya"

    for (const auto& [id, name] : empMap) { // C++17 Structured Binding
        std::cout << id << " -> " << name << "\n";
    }
    return 0;
}
```

##### The `operator[]` vs. `.at()` Trap (High-Yield CMCE MCQ):

- `map[key]`: If `key` exists, returns reference to value. **If `key` does NOT exist, it silently inserts a new default-constructed value!** (Requires `Value` to have a default constructor; cannot be used on a `const std::map`).
- `map.at(key)`: If `key` exists, returns reference to value. If `key` does NOT exist, throws **`std::out_of_range`** exception.

```cpp
std::map<int, int> m;
std::cout << m[5]; // Inserts key '5' with value 0, then prints 0!
// m.size() is now 1!
```

#### 2.4 `std::multimap` (Sorted Key-Value Pairs with Duplicate Keys)

- Associates keys with values while permitting duplicate keys.
- **Does NOT support `operator[]` or `.at()`** because a key does not uniquely identify a single value.
- Use `equal_range()` to retrieve all values associated with a shared key:

```cpp
#include <map>
#include <iostream>

int main() {
    std::multimap<std::string, std::string> dict;
    dict.insert({"bank", "Financial institution"});
    dict.insert({"bank", "River side boundary"});

    auto range = dict.equal_range("bank");
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << it->first << " : " << it->second << "\n";
    }
    return 0;
}
```

---

### 3. Range-Query Operations in Associative Containers

For any tree-based container, binary-search bound operations run in $\mathcal{O}(\log n)$:

```
Given elements in set: [ 10, 20, 30, 40, 50 ]
                                 ^           ^
Target: 30                       |           |
lower_bound(30) -----------------+           | (Points to first element >= 30)
upper_bound(30) -----------------------------+ (Points to first element > 30, i.e., 40)
```

- **`c.lower_bound(k)`**: Returns iterator to the first element that is **$\ge k$** (not less than $k$).
- **`c.upper_bound(k)`**: Returns iterator to the first element that is **$> k$** (strictly greater than $k$).
- **`c.equal_range(k)`**: Returns `std::pair<iterator, iterator>` representing `[lower_bound, upper_bound)`.

---

### 4. Unordered Containers (Hash Tables)

Unordered containers organize elements into **buckets**. The internal bucket for any key is determined by a hash function:

$$\text{Bucket Index} = \text{std::hash<Key>}(key) \pmod{\text{Total Buckets}}$$

```
Bucket Array:
[ 0 ] ---> NULL
[ 1 ] ---> [ Key: "Alpha", Val: 10 ] ---> [ Key: "Delta", Val: 40 ] (Collision chain)
[ 2 ] ---> [ Key: "Beta",  Val: 20 ] ---> NULL
[ 3 ] ---> NULL
```

#### 4.1 Unordered Types:

1. **`std::unordered_set`**: Unique keys in an open-hash table.
2. **`std::unordered_multiset`**: Duplicate keys allowed via hash bucket collision lists.
3. **`std::unordered_map`**: Unique key-value pairs; average $\mathcal{O}(1)$ lookup via `operator[]` or `.at()`.
4. **`std::unordered_multimap`**: Duplicate key-value pairs; no `operator[]`.

#### 4.2 Hash Collisions & Load Factor

- **Load Factor ($\alpha$):**
  $$\alpha = \frac{\text{Current Element Count}}{\text{Total Bucket Count}}$$
- **`max_load_factor()`**: The threshold (default is $1.0$) at which the container automatically performs **rehashing**:
  - Allocates a larger bucket array (usually double).
  - Re-hashes and redistributes all existing elements.
  - **Invalidates all iterators!** (Pointers and references to elements remain valid).

#### 4.3 Custom Types with `std::unordered_map`

To store a user-defined class as a key in an unordered container, you **must provide both**:

1. A **Hash Function Specialization** (`std::hash<T>`).
2. An **Equality Operator** (`operator==`).

```cpp
#include <iostream>
#include <unordered_map>

struct Point {
    int x, y;
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

// Custom Hash Functor
struct PointHash {
    std::size_t operator()(const Point& p) const {
        return std::hash<int>()(p.x) ^ (std::hash<int>()(p.y) << 1);
    }
};

int main() {
    std::unordered_map<Point, std::string, PointHash> grid;
    grid[{1, 2}] = "Start Node";
    std::cout << grid[{1, 2}]; // Output: Start Node
    return 0;
}
```

---

### 5. Master Comparison Matrix: All 8 Key-Based Containers

| Container                     | Underlying Structure | Keys           | Allows Duplicates? | Subscript `[]`?        | Typical Search Complexity                      | Iterator      |
| :---------------------------- | :------------------- | :------------- | :----------------- | :--------------------- | :--------------------------------------------- | :------------ |
| **`std::set`**                | Red-Black Tree       | Key only       | ❌ No              | ❌ No                  | $\mathcal{O}(\log n)$ guaranteed               | Bidirectional |
| **`std::multiset`**           | Red-Black Tree       | Key only       | ✅ Yes             | ❌ No                  | $\mathcal{O}(\log n)$ guaranteed               | Bidirectional |
| **`std::map`**                | Red-Black Tree       | Key-Value Pair | ❌ No              | ✅ Yes (`[]`, `.at()`) | $\mathcal{O}(\log n)$ guaranteed               | Bidirectional |
| **`std::multimap`**           | Red-Black Tree       | Key-Value Pair | ✅ Yes             | ❌ No                  | $\mathcal{O}(\log n)$ guaranteed               | Bidirectional |
| **`std::unordered_set`**      | Hash Table           | Key only       | ❌ No              | ❌ No                  | Avg: $\mathcal{O}(1)$, Worst: $\mathcal{O}(n)$ | Forward       |
| **`std::unordered_multiset`** | Hash Table           | Key only       | ✅ Yes             | ❌ No                  | Avg: $\mathcal{O}(1)$, Worst: $\mathcal{O}(n)$ | Forward       |
| **`std::unordered_map`**      | Hash Table           | Key-Value Pair | ❌ No              | ✅ Yes (`[]`, `.at()`) | Avg: $\mathcal{O}(1)$, Worst: $\mathcal{O}(n)$ | Forward       |
| **`std::unordered_multimap`** | Hash Table           | Key-Value Pair | ✅ Yes             | ❌ No                  | Avg: $\mathcal{O}(1)$, Worst: $\mathcal{O}(n)$ | Forward       |

---

### 6. Iterator Invalidation Rules

- **Ordered Associative Containers (`set`, `map`, `multiset`, `multimap`):**
  - **Node-based Stability:** Inserting elements **never invalidates** any existing iterators, pointers, or references.
  - Erasing an element invalidates **only the iterator pointing directly to the erased element**. All other iterators remain valid.
- **Unordered Containers (`unordered_*`):**
  - If an insertion triggers a **rehash** (load factor exceeds max load factor), **all iterators are invalidated** (references and pointers remain valid).
  - If no rehash occurs, iterators remain valid.
  - Erasing an element invalidates only iterators to that specific erased element.

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: What happens if you search for a non-existent key using `myMap[k]` versus `myMap.find(k)`?**
  - _Answer:_ `myMap[k]` will implicitly allocate and insert a new node containing key `k` with a default-initialized value, altering container size (`size()` increases). `myMap.find(k)` is a non-mutating read-only inspection that returns `myMap.end()` without altering the map.

- **Q2: Why are keys in `std::set` and `std::map` immutable?**
  - _Answer:_ The internal Red-Black tree structure relies on strict ordering invariants (`left < root < right`). If the key were modifiable in-place through an iterator, changing its value would break tree invariants, causing searches, insertions, and deletions to fail or cause undefined behavior.

- **Q3: When would you prefer `std::map` over `std::unordered_map` despite the slower $\mathcal{O}(\log n)$ lookup?**
  - _Answer:_
    1. When elements **must be traversed in sorted order** (e.g., printing an alphabetical dictionary or ranking leaderboard).
    2. When **order-based range queries** are required (e.g., using `lower_bound()` / `upper_bound()`).
    3. When **guaranteed $\mathcal{O}(\log n)$ worst-case performance** is mandatory (real-time/safety-critical systems where an $\mathcal{O}(n)$ worst-case hash collision storm is unacceptable).
    4. When types have an `operator<` defined, but writing a quality hash function is difficult.

- **Q4: What is the return type of `std::set::insert()`?**
  - _Answer:_ It returns a pair: `std::pair<iterator, bool>`.
    - `pair.first` is an iterator pointing to the inserted element (or the existing element preventing insertion).
    - `pair.second` is a boolean: `true` if the item was successfully inserted, `false` if it was rejected as a duplicate.

- **Q5: What is the difference between `std::multiset::erase(val)` and `std::multiset::erase(iterator)`?**
  - _Answer:_ `erase(val)` searches for and deletes **all occurrences** matching `val`, returning the count of deleted elements. `erase(iterator)` removes **only the single specific node** pointed to by the iterator, leaving identical keys intact.
