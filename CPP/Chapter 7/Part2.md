# Module 07: Standard Template Library (STL)

## Unit 7.2: Sequence Containers & Container Adapters

Sequence containers store elements in a linear progression, preserving the order of insertion. Container adapters provide specialized, restricted interfaces (LIFO/FIFO) by wrapping underlying sequence containers.

---

### 1. `std::vector` (Dynamic Contiguous Array)

A `std::vector` is a sequence container representing a dynamically resizable array stored in **contiguous heap memory**.

```
vector in Memory (Contiguous Heap Buffer):
+-------+-------+-------+-------+-------+ - - - - - - - - - - - +
|  10   |  20   |  30   |  40   |  50   | Unallocated / Reserved|
+-------+-------+-------+-------+-------+ - - - - - - - - - - - +
  ^                               ^                       ^
  |                               |                       |
[0]                      size() = 5 (Active)    capacity() = 8 (Allocated)
```

#### 1.1 Memory Growth & Capacity Mechanics

- **`size()`**: Number of actual elements currently stored in the vector.
- **`capacity()`**: Total number of elements the vector can hold before needing a reallocation.
- **Growth Factor**: When `size() == capacity()`, calling `push_back()` allocates a new, larger contiguous memory block (typically **$1.5\times$** in MSVC or **$2\times$** in GCC/Clang), moves/copies existing elements, and frees the old block.
- **Amortized Time Complexity**: Individual `push_back()` operations run in $\mathcal{O}(1)$ amortized time, despite occasional $\mathcal{O}(n)$ reallocations.

#### 1.2 `reserve()` vs. `resize()`

| Function         | What it Modifies                       | Allocates Memory?              | Initializes Elements?                    |
| :--------------- | :------------------------------------- | :----------------------------- | :--------------------------------------- |
| **`reserve(n)`** | Only **`capacity()`**                  | Yes (if $n > \text{capacity}$) | **No** (leaves memory raw/uninitialized) |
| **`resize(n)`**  | Both **`size()`** and **`capacity()`** | Yes (if $n > \text{capacity}$) | **Yes** (value-initializes new elements) |

#### 1.3 Element Access: `operator[]` vs. `.at()`

- `vec[i]`: Unchecked random access. High performance, but out-of-bounds access triggers **Undefined Behavior**.
- `vec.at(i)`: Bounds-checked access. Throws **`std::out_of_range`** if $i \ge \text{size()}$.

#### 1.4 Vector Iterator Invalidation Rules (High-Yield MCQ)

- **Reallocation occurs**: **Every single iterator, pointer, and reference** to elements in that vector becomes **invalid** (dangling).
- **No reallocation occurs**: Iterators before the insertion point remain valid; iterators at or after the insertion/deletion point are **invalidated**.
- `push_back()` / `emplace_back()`: If no reallocation occurs, only `end()` is invalidated.

---

### 2. `std::list` (Doubly-Linked List)

`std::list` stores elements in non-contiguous heap nodes, where each node contains pointers to both its predecessor and successor.

```
       +--------------------+      +--------------------+
NULL <-| prev | 10 | next  | <==> | prev | 20 | next  | -> NULL
       +--------------------+      +--------------------+
```

#### 2.1 Key Characteristics

- **Iterator Category**: **Bidirectional Iterator** (supports `++` and `--`, but **no random access** `it + n` or `[]`).
- **Insertion / Deletion Complexity**: $\mathcal{O}(1)$ anywhere, provided an iterator pointing to the location is already available.
- **Cache Locality**: Poor (nodes scattered across heap; high CPU cache miss rate).
- **Memory Overhead**: Significant ($2$ pointers per node: $16\text{ bytes}$ overhead per element on 64-bit systems).

#### 2.2 Splicing: Zero-Copy Node Transfers

`std::list::splice` transfers nodes between lists in $\mathcal{O}(1)$ time by rewiring internal pointers without allocating or copying data:

```cpp
std::list<int> list1 = {1, 2, 3};
std::list<int> list2 = {10, 20};
auto it = list1.begin();
++it; // Points to '2'
list1.splice(it, list2); // list1 becomes: 1, 10, 20, 2, 3; list2 is now empty!
```

---

### 3. `std::deque` (Double-Ended Queue)

`std::deque` (pronounced "deck") allocates memory as fixed-size chunks indexed by a central map of chunk pointers.

```
Central Map of Pointers:
[ ptr0 ] ---> [ Chunk 0: Buffer Buffer Buffer Buffer ]
[ ptr1 ] ---> [ Chunk 1:   10     20     30     40   ]
[ ptr2 ] ---> [ Chunk 2:   50     60    Free   Free  ]
```

#### 3.1 Key Characteristics

- **Iterator Category**: **Random-Access Iterator** (`deque[i]` supported in $\mathcal{O}(1)$ via two pointer lookups).
- **Dual-End Growth**: $\mathcal{O}(1)$ insertions and removals at **both ends** (`push_front`, `push_back`, `pop_front`, `pop_back`).
- **No Reallocation of Existing Elements**: Expanding the front or back allocates a new chunk without copying existing elements.
- **Storage**: Non-contiguous across chunks; elements cannot be passed to APIs expecting contiguous C-arrays (`&deq[0]` is not contiguous across chunk boundaries).

---

### 4. `std::array` (Static Contiguous Array - C++11)

A zero-overhead, thin object-oriented wrapper around raw C-style arrays (`<array>`).

```cpp
template <typename T, size_t N>
class array;
```

#### 4.1 Key Characteristics

- **Storage Location**: Allocated on the **Stack** (or inline within the parent object), not the heap.
- **Fixed Size**: Size $N$ is part of the type signature (`std::array<int, 5>` and `std::array<int, 10>` are different types).
- **Zero Runtime Overhead**: Same size and performance as `T arr[N]`, but provides STL compatibility (`begin()`, `end()`, `size()`, `.at()`).
- **Assignment Support**: Supports assignment and copy-by-value, unlike raw C arrays:
  ```cpp
  std::array<int, 3> a1 = {1, 2, 3};
  std::array<int, 3> a2 = a1; // Fully legal!
  ```

---

### 5. Container Adapters

Container adapters modify and restrict the interfaces of existing sequence containers to implement abstract data types. Adapters **do not provide iterators** (no `begin()` or `end()`).

```
                    Container Adapters Architecture
                                   |
         +-------------------------+-------------------------+
         |                         |                         |
   std::stack<T>             std::queue<T>         std::priority_queue<T>
  (LIFO: push/pop)          (FIFO: push/pop)         (Heap: pop highest)
         |                         |                         |
Default underlying:       Default underlying:       Default underlying:
    std::deque                std::deque                std::vector
```

#### 5.1 `std::stack` (LIFO: Last-In, First-Out)

- **Default Underlying Container**: `std::deque` (can also use `std::vector` or `std::list`).
- **Core API**: `push()`, `emplace()`, `pop()`, `top()`, `empty()`, `size()`.
- **Complexity**: All operations are $\mathcal{O}(1)$.

```cpp
std::stack<int> s; // Uses deque by default
std::stack<int, std::vector<int>> vecStack; // Customized to use vector
```

#### 5.2 `std::queue` (FIFO: First-In, First-Out)

- **Default Underlying Container**: `std::deque` (can also use `std::list`).
- _Cannot use `std::vector`_: `std::queue` requires `pop_front()`, which `vector` does not provide.
- **Core API**: `push()`, `emplace()`, `pop()`, `front()`, `back()`, `empty()`, `size()`.
- **Complexity**: All operations are $\mathcal{O}(1)$.

#### 5.3 `std::priority_queue` (Max-Heap by Default)

- **Default Underlying Container**: `std::vector` (requires random access for binary heap math).
- **Internal Structure**: Maintained as a heap using `std::make_heap`, `std::push_heap`, and `std::pop_heap`.
- **Element Ordering**: Largest element sits at the top (Max-Heap via default `std::less<T>`).
- **Core API**: `push()` $\mathcal{O}(\log n)$, `pop()` $\mathcal{O}(\log n)$, `top()` $\mathcal{O}(1)$.

##### Constructing a Min-Heap:

```cpp
#include <queue>
#include <vector>
#include <functional>

// Syntax: priority_queue<Type, Container, Comparator>
std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
```

---

### 6. Master Comparison Matrix of Sequence Containers

| Dimension                    | `std::vector`              | `std::deque`          | `std::list`                  | `std::forward_list`          | `std::array`           |
| :--------------------------- | :------------------------- | :-------------------- | :--------------------------- | :--------------------------- | :--------------------- |
| **Underlying Structure**     | Contiguous dynamic array   | Array of fixed chunks | Doubly-linked list           | Singly-linked list           | Contiguous stack array |
| **Iterator Category**        | Random-Access              | Random-Access         | Bidirectional                | Forward                      | Random-Access          |
| **Random Access (`[]`)**     | $\mathcal{O}(1)$           | $\mathcal{O}(1)$      | ❌ Not supported             | ❌ Not supported             | $\mathcal{O}(1)$       |
| **Insert / Erase at Back**   | $\mathcal{O}(1)$ amortized | $\mathcal{O}(1)$      | $\mathcal{O}(1)$             | ❌ No direct back            | ❌ Fixed size          |
| **Insert / Erase at Front**  | $\mathcal{O}(n)$           | $\mathcal{O}(1)$      | $\mathcal{O}(1)$             | $\mathcal{O}(1)$             | ❌ Fixed size          |
| **Insert / Erase in Middle** | $\mathcal{O}(n)$           | $\mathcal{O}(n)$      | $\mathcal{O}(1)$ (with iter) | $\mathcal{O}(1)$ (with iter) | ❌ Fixed size          |
| **Memory Allocation**        | Heap                       | Heap                  | Heap                         | Heap                         | Stack / Inline         |
| **Memory Overhead**          | Unused capacity            | Map pointers + chunks | 2 pointers / node            | 1 pointer / node             | Zero                   |
| **Contiguous Storage?**      | **Yes**                    | No                    | No                           | No                           | **Yes**                |

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: Why is `std::vector` preferred over `std::list` even when many insertions occur in the middle?**
  - _Answer:_ **Hardware Cache Locality.** `std::vector` stores elements contiguously, enabling CPU cache lines to prefetch data efficiently. In contrast, `std::list` nodes are scattered randomly across the heap, triggering frequent CPU cache misses. Unless element sizes are large or cheap node splicing is required, `std::vector` consistently outperforms `std::list` in practice.

- **Q2: Why does `std::vector<bool>` behave differently from other vector types?**
  - _Answer:_ `std::vector<bool>` is a template specialization optimized for memory footprint. It packs booleans into **individual bits** (1 bit per bool rather than 1 byte). Consequently, individual bits cannot be directly referenced, so `operator[]` returns a temporary proxy object (`std::vector<bool>::reference`), not a genuine `bool&`.

- **Q3: What happens to `std::deque` iterators when elements are inserted at the front or back?**
  - _Answer:_ Inserting at either end invalidates all existing **iterators**, but all existing **references and pointers** to elements remain completely valid (because existing data chunks are not moved).

- **Q4: Why can't `std::vector` be used as the underlying container for `std::queue`?**
  - _Answer:_ `std::queue` requires `pop_front()`. `std::vector` does not implement `pop_front()` because removing an element from the front requires shifting every subsequent element down by one position, an $\mathcal{O}(n)$ operation that violates the adapter's $\mathcal{O}(1)$ efficiency contract.

- **Q5: What is the difference between `shrink_to_fit()` and `clear()` on a vector?**
  - _Answer:_ `vec.clear()` reduces `size()` to $0$ by destructing elements, but leaves `capacity()` unchanged (retains allocated memory). `vec.shrink_to_fit()` requests the runtime to deallocate unused memory, reducing `capacity()` to match `size()`.
