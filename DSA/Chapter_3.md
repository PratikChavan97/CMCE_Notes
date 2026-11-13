# Chapter 3: Searching and Sorting Algorithms

**Course**: Data Structures and Algorithms with Java
**Preparation**: CDAC C-CAT / CMCE
**Level**: Beginner to Intermediate
**Parts**: 4

---

# PART 1: Searching Algorithms

## 1.1 What is Searching?

Searching is the process of finding a particular element in a collection of data.

The element we want to find is called the target or search key.

Example:

Array:

[10, 20, 30, 40, 50]

Target: 30

Output: Element found at index 2.

**Types of Searching**

1. Linear Search
2. Binary Search

## 1.2 Linear Search

Linear search checks each element one by one until the target is found or the collection ends.

**Algorithm**

1. Start from the first element.
2. Compare the current element with the target.
3. If both match, return the index.
4. Otherwise, move to the next element.
5. If the end is reached, return -1.
6. Java Implementation

```java
class Main {

    static int linearSearch(int[] arr, int target) {

        for (int i = 0; i < arr.length; i++) {

            if (arr[i] == target) {
                return i;
            }
        }

        return -1;
    }

    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50};

        int target = 30;

        int result = linearSearch(arr, target);

        if (result != -1) {
            System.out.println("Found at index: " + result);
        } else {
            System.out.println("Element not found");
        }
    }
}

```

```text
Found at index: 2
```

**Dry Run**
Array: [10, 20, 30, 40, 50]
Target: 30

| Iteration | Index | Element | Comparison  |
| --------- | ----- | ------- | ----------- |
| 1         | 0     | 10      | Not matched |
| 2         | 1     | 20      | Not matched |
| 3         | 2     | 30      | Matched     |

Element found at index 2.

**Complexity Analysis**

| Case         | Time Complexity |
| ------------ | --------------- |
| Best case    | O(1)            |
| Average case | O(n)            |
| Worst case   | O(n)            |

Auxiliary Space: O(1)

**Advantages**

- Easy to implement.
- Works on sorted and unsorted arrays.
- Does not require additional memory.

**Disadvantages**

- Slow for large datasets.
- May require checking every element.

## 1.3 Binary Search

Binary search is an efficient searching algorithm that repeatedly divides the search space into two halves.

**Important**: The array must be sorted.

**Algorithm**

1. Set low = 0.
2. Set high = array length - 1.
3. Calculate middle = low + (high - low) / 2.
4. Compare the middle element with target.
5. If equal, return middle.
6. If target is smaller, search the left half.
7. If target is larger, search the right half.
8. Repeat until found or low becomes greater than high.

**Java Implementation – Iterative**

```java
class Main {

    static int binarySearch(int[] arr, int target) {

        int low = 0;
        int high = arr.length - 1;

        while (low <= high) {

            int mid = low + (high - low) / 2;

            if (arr[mid] == target) {
                return mid;
            }

            else if (arr[mid] < target) {
                low = mid + 1;
            }

            else {
                high = mid - 1;
            }
        }

        return -1;
    }

    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50, 60};

        int target = 40;

        System.out.println(binarySearch(arr, target));
    }
}

```

Output:

```text
3
```

**Dry Run**
Array: [10, 20, 30, 40, 50, 60]

Target: 50

| Iteration | Low | High | Mid | arr[mid] |
| --------- | --- | ---- | --- | -------- |
| 1         | 0   | 5    | 2   | 30       |
| 2         | 3   | 5    | 4   | 50       |

Element found at index 4.

**Complexity Analysis**

| Case         | Time Complexity |
| ------------ | --------------- |
| Best case    | O(1)            |
| Average case | O(log n)        |
| Worst case   | O(log n)        |

Auxiliasy space: O(1) for iterative binary search.

## 1.4 Recursive Binary Search

Binary search can also be implemented using recursion.

**Java Implementation**

```java
class Main {

    static int binarySearch(int[] arr, int low, int high, int target) {

        if (low > high) {
            return -1;
        }

        int mid = low + (high - low) / 2;

        if (arr[mid] == target) {
            return mid;
        }

        if (target < arr[mid]) {
            return binarySearch(arr, low, mid - 1, target);
        }

        return binarySearch(arr, mid + 1, high, target);
    }

    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50};

        int result = binarySearch(arr, 0, arr.length - 1, 40);

        System.out.println(result);
    }
}

```

Time Complexity: O(log n)

Auxiliary Space: O(log n) due to recursive calls.

## 1.5 Linear Search vs Binary Search

| Feature              | Linear Search | Binary Search      |
| -------------------- | ------------- | ------------------ |
| Approach             | Sequential    | Divide and conquer |
| Sorted data required | No            | Yes                |
| Best Case            | O(1)          | O(1)               |
| Average Case         | O(n)          | O(log n)           |
| Worst Case           | O(n)          | O(log n)           |
| Implementation       | Simple        | more complex       |

## 1.6 Important Binary Search Variations

These are useful for DSA problem-solving.

**A. First Occurrence of an Element**
Find the first position of a target in a sorted array containing duplicates.

```java
class Main {

    static int firstOccurrence(int[] arr, int target) {

        int low = 0;
        int high = arr.length - 1;
        int answer = -1;

        while (low <= high) {

            int mid = low + (high - low) / 2;

            if (arr[mid] == target) {

                answer = mid;
                high = mid - 1;

            } else if (arr[mid] < target) {

                low = mid + 1;

            } else {

                high = mid - 1;
            }
        }

        return answer;
    }

    public static void main(String[] args) {

        int[] arr = {1, 2, 2, 2, 3, 4};

        System.out.println(firstOccurrence(arr, 2));
    }
}

```

Output: 1

**B. Last Occurrence of an Element**

```java
class Main {

    static int lastOccurrence(int[] arr, int target) {

        int low = 0;
        int high = arr.length - 1;
        int answer = -1;

        while (low <= high) {

            int mid = low + (high - low) / 2;

            if (arr[mid] == target) {

                answer = mid;
                low = mid + 1;

            } else if (arr[mid] < target) {

                low = mid + 1;

            } else {

                high = mid - 1;
            }
        }

        return answer;
    }

    public static void main(String[] args) {

        int[] arr = {1, 2, 2, 2, 3, 4};

        System.out.println(lastOccurrence(arr, 2));
    }
}

```

Output: 3

**C. Count Occurrences**

Count = Last occurrence - First occurrence + 1.

For target 2:

Count = 3 - 1 + 1 = 3.

---

# PART 2: Basic Sorting Algorithms

## 2.1 What is Sorting?

Sorting is the process of arranging elements in a particular order.

Types:

- Ascending order: Smallest to largest.
- Descending order: Largest to smallest.

Example:

Original:

[5, 2, 8, 1, 3]

Ascending:

[1, 2, 3, 5, 8]

Descending:

[8, 5, 3, 2, 1]

## 2.2 Bubble Sort

Bubble sort repeatedly compares adjacent elements and swaps them if they are in the wrong order.

The largest element moves towards the end in each pass.

**Algorithm**

1. Start from index 0.
2. Compare adjacent elements.
3. Swap if the left element is greater than the right element.
4. Continue until the end.
5. Repeat for remaining unsorted elements.

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static void bubbleSort(int[] arr) {

        int n = arr.length;

        for (int i = 0; i < n - 1; i++) {

            boolean swapped = false;

            for (int j = 0; j < n - 1 - i; j++) {

                if (arr[j] > arr[j + 1]) {

                    int temp = arr[j];

                    arr[j] = arr[j + 1];

                    arr[j + 1] = temp;

                    swapped = true;
                }
            }

            if (!swapped) {
                break;
            }
        }
    }

    public static void main(String[] args) {

        int[] arr = {5, 2, 8, 1, 3};

        bubbleSort(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

```text
[1, 2, 3, 5, 8]
```

**Dry Run**

Original array:

[5, 2, 8, 1]

Pass 1:

- Compare 5 and 2 → Swap.
- Compare 5 and 8 → No swap.
- Compare 8 and 1 → Swap.

Result: [2, 5, 1, 8]

Pass 2:

- Compare 2 and 5 → No swap.
- Compare 5 and 1 → Swap.
  Result: [2, 1, 5, 8]

Pass 3:

- Compare 2 and 1 → Swap.
  Result: [1, 2, 5, 8]

**Complexity**

| Case    | Time                      |
| ------- | ------------------------- |
| Best    | O(n), with early stopping |
| Average | $$\mathcal{O}(n^2)$$      |
| Worst   | $$\mathcal{O}(n^2)$$      |

Auxiliary Space: O(1)

Stable: Yes.

In-place: Yes.

## 2.3 Selection Sort

Selection sort repeatedly finds the smallest element from the unsorted portion and places it in the correct position.

**Algorithm**

1. Assume the first unsorted element is minimum.
2. Find the actual minimum in the remaining array.
3. Swap it with the first unsorted element.
4. Move the sorted boundary one position ahead.
5. Repeat.

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static void selectionSort(int[] arr) {

        int n = arr.length;

        for (int i = 0; i < n - 1; i++) {

            int minIndex = i;

            for (int j = i + 1; j < n; j++) {

                if (arr[j] < arr[minIndex]) {
                    minIndex = j;
                }
            }

            int temp = arr[i];

            arr[i] = arr[minIndex];

            arr[minIndex] = temp;
        }
    }

    public static void main(String[] args) {

        int[] arr = {64, 25, 12, 22, 11};

        selectionSort(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

```text
[11, 12, 22, 25, 64]
```

**Complexity**

| Case    | Time                 |
| ------- | -------------------- |
| Best    | $$\mathcal{O}(n^2)$$ |
| Average | $$\mathcal{O}(n^2)$$ |
| Worst   | $$\mathcal{O}(n^2)$$ |

Auxiliary Space: O(1)

Stable: Generally no.

In-place: Yes.

> Important: Selection sort performs at most n-1 swaps in its usual implementation.

## 2.4 Insertion Sort

Insertion sort builds a sorted portion one element at a time.

Each new element is inserted into its correct position.

**Algorithm**

1. Assume the first element is sorted.
2. Select the next element as key.
3. Compare the key with previous elements.
4. Shift larger elements one position right.
5. Insert the key into its correct position.
6. Repeat.

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static void insertionSort(int[] arr) {

        int n = arr.length;

        for (int i = 1; i < n; i++) {

            int key = arr[i];

            int j = i - 1;

            while (j >= 0 && arr[j] > key) {

                arr[j + 1] = arr[j];

                j--;
            }

            arr[j + 1] = key;
        }
    }

    public static void main(String[] args) {

        int[] arr = {5, 3, 4, 1, 2};

        insertionSort(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

```text
[1, 2, 3, 4, 5]
```

**Complexity**

| Case    | Time                 |
| ------- | -------------------- |
| Best    | $$\mathcal{O}(n)$$   |
| Average | $$\mathcal{O}(n^2)$$ |
| Worst   | $$\mathcal{O}(n^2)$$ |

Auxiliary Space: O(1)

Stable: Yes.

In-place: Yes.

## 2.5 Comparison of Basic Sorting Algorithms

| Algorithm | Best                 | Average              | Worst                | Stable        |
| --------- | -------------------- | -------------------- | -------------------- | ------------- |
| Bubble    | $$\mathcal{O}(n)$$   | $$\mathcal{O}(n^2)$$ | $$\mathcal{O}(n^2)$$ | Yes           |
| Selection | $$\mathcal{O}(n^2)$$ | $$\mathcal{O}(n^2)$$ | $$\mathcal{O}(n^2)$$ | No, generally |
| Insertion | $$\mathcal{O}(n)$$   | $$\mathcal{O}(n^2)$$ | $$\mathcal{O}(n^2)$$ | Yes           |

---

# PART 3: Advanced Sorting Algorithms

## 3.1 Merge Sort

Merge sort is a divide-and-conquer sorting algorithm.

It divides the array into smaller halves, sorts them and merges the sorted halves.

**Working**

1. Divide the array into two halves.
2. Recursively sort the left half.
3. Recursively sort the right half.
4. Merge both sorted halves.
5. Continue until the complete array is sorted.

**Example**

Original:

[8, 4, 2, 6]

Divide:

[8, 4] and [2, 6]

Further divide:

[8] [4] [2] [6]

Merge:

[4, 8] and [2, 6]

Final merge:

[2, 4, 6, 8]

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static void mergeSort(int[] arr, int left, int right) {

        if (left < right) {

            int mid = left + (right - left) / 2;

            mergeSort(arr, left, mid);

            mergeSort(arr, mid + 1, right);

            merge(arr, left, mid, right);
        }
    }

    static void merge(int[] arr, int left, int mid, int right) {

        int n1 = mid - left + 1;

        int n2 = right - mid;

        int[] L = new int[n1];

        int[] R = new int[n2];

        for (int i = 0; i < n1; i++) {
            L[i] = arr[left + i];
        }

        for (int j = 0; j < n2; j++) {
            R[j] = arr[mid + 1 + j];
        }

        int i = 0;
        int j = 0;
        int k = left;

        while (i < n1 && j < n2) {

            if (L[i] <= R[j]) {
                arr[k++] = L[i++];
            } else {
                arr[k++] = R[j++];
            }
        }

        while (i < n1) {
            arr[k++] = L[i++];
        }

        while (j < n2) {
            arr[k++] = R[j++];
        }
    }

    public static void main(String[] args) {

        int[] arr = {8, 4, 2, 6, 1, 9};

        mergeSort(arr, 0, arr.length - 1);

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[1, 2, 4, 6, 8, 9]
```

**Complexity Analysis**

| Case    | Time                     |
| ------- | ------------------------ |
| Best    | $$\mathcal{O}(n log n)$$ |
| Average | $$\mathcal{O}(n log n)$$ |
| Worst   | $$\mathcal{O}(n log n)$$ |

Auxiliary Space: O(n)

Stable: Yes, with the merge implementation shown.

In-place: No, in this implementation.

**Advantages**

- Predictable performance.
- Good for large datasets.
- Stable sorting.
- Useful for linked lists and external sorting.

**Disadvantages**

- Requires additional memory.
- Recursive calls introduce overhead.

## 3.2 Quick Sort

Quick sort is a divide-and-conquer algorithm.

It selects a pivot and partitions the array into elements smaller than or equal to the pivot and elements greater than the pivot.

**Working**

1. Select a pivot.
2. Partition the array around the pivot.
3. Place the pivot in its final sorted position.
4. Recursively sort the left partition.
5. Recursively sort the right partition.

**Java Implementation – Lomuto Partition**

```java
import java.util.Arrays;

class Main {

    static void quickSort(int[] arr, int low, int high) {

        if (low < high) {

            int pivotIndex = partition(arr, low, high);

            quickSort(arr, low, pivotIndex - 1);

            quickSort(arr, pivotIndex + 1, high);
        }
    }

    static int partition(int[] arr, int low, int high) {

        int pivot = arr[high];

        int i = low - 1;

        for (int j = low; j < high; j++) {

            if (arr[j] <= pivot) {

                i++;

                int temp = arr[i];

                arr[i] = arr[j];

                arr[j] = temp;
            }
        }

        int temp = arr[i + 1];

        arr[i + 1] = arr[high];

        arr[high] = temp;

        return i + 1;
    }

    public static void main(String[] args) {

        int[] arr = {10, 7, 8, 9, 1, 5};

        quickSort(arr, 0, arr.length - 1);

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[1, 5, 7, 8, 9, 10]
```

**Complexity Analysis**

| Case    | Time                     |
| ------- | ------------------------ |
| Best    | $$\mathcal{O}(n log n)$$ |
| Average | $$\mathcal{O}(n log n)$$ |
| Worst   | $$\mathcal{O}(n^2)$$     |

Auxiliary Space:

- Average recursion space: O(log n).
- Worst recursion space: O(n).

Stable: No, in this implementation.

In-place: Yes, apart from recursion stack.

**Why Does Quick Sort Have O(n²) Worst Case?**

If the pivot repeatedly divides the array into an empty partition and a partition of size n-1, the recursion becomes highly unbalanced.

This can happen with already sorted input when choosing the last element as pivot.

**Merge Sort vs Quick Sort**

| Feature         | Merge Sort                 | Quick Sort                      |
| --------------- | -------------------------- | ------------------------------- |
| Approach        | Divede and conquer         | Divide and conquer              |
| Best Time       | $$\mathcal{O}(n log n)$$   | $$\mathcal{O}(n log n)$$        |
| Average Time    | $$\mathcal{O}(n log n)$$   | $$\mathcal{O}(n log n)$$        |
| Worst Time      | $$\mathcal{O}(n log n)$$   | $$\mathcal{O}(n ^2)$$           |
| Auxiliary space | $$\mathcal{O}(n)$$         | $$\mathcal{O}(log n)$$ average  |
| Stability       | Yes                        | No                              |
| In-place        | No, standard array version | Generally, yes apart from stack |

---

# PART 4: Counting Sort, Heap Sort and Java Sorting

## 4.1 Counting Sort

Counting sort is a non-comparison-based sorting algorithm.

It counts how many times each value occurs and uses these frequencies to construct the sorted output.

It works especially well when the range of integer values is relatively small.

**Example**

Original array:

[4, 2, 2, 8, 3, 3, 1]

Frequency:

| Number | Frequency |
| ------ | --------- |
| 1      | 1         |
| 2      | 2         |
| 3      | 2         |
| 4      | 1         |
| 5      | 1         |

Sorted array:

[1, 2, 2, 3, 3, 4, 8]

**Java Implementation**

This implementation supports non-negative integers.

```java
import java.util.Arrays;

class Main {

    static void countingSort(int[] arr) {

        if (arr.length == 0) {
            return;
        }

        int max = arr[0];

        for (int x : arr) {
            max = Math.max(max, x);
        }

        int[] count = new int[max + 1];

        for (int x : arr) {
            count[x]++;
        }

        int index = 0;

        for (int i = 0; i < count.length; i++) {

            while (count[i] > 0) {

                arr[index++] = i;

                count[i]--;
            }
        }
    }

    public static void main(String[] args) {

        int[] arr = {4, 2, 2, 8, 3, 3, 1};

        countingSort(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[1, 2, 2, 3, 3, 4, 8]
```

**Complexity Analysis**

Let k be the range of values.

Time Complexity: O(n + k)

Auxiliary Space: O(k) for this implementation.

Stable: The implementation shown is not a general stable output-array implementation.

Important: This implementation assumes non-negative input. Negative values require an offset or another approach.

## 4.2 Heap Sort

Heap sort uses a binary heap to sort elements.

A max heap is used for ascending order.

**What is a Heap?**

A heap is a complete binary tree that satisfies the heap property.

**Max Heap:** Parent is greater than or equal to its children.

**Min Heap:** Parent is less than or equal to its children.

**Array Representation of a Heap**

For an element at index i in a zero-based array:

- Left child = 2\*i + 1
- Right child = 2\*i + 2
- Parent = (i - 1) / 2

**Heap Sort Algorithm**

1. Build a max heap.
2. Swap root with the last element.
3. Reduce the heap size.
4. Heapify the root.
5. Repeat until sorted.

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static void heapSort(int[] arr) {

        int n = arr.length;

        // Build max heap

        for (int i = n / 2 - 1; i >= 0; i--) {
            heapify(arr, n, i);
        }

        // Extract elements one by one

        for (int i = n - 1; i > 0; i--) {

            int temp = arr[0];

            arr[0] = arr[i];

            arr[i] = temp;

            heapify(arr, i, 0);
        }
    }

    static void heapify(int[] arr, int n, int i) {

        int largest = i;

        int left = 2 * i + 1;

        int right = 2 * i + 2;

        if (left < n && arr[left] > arr[largest]) {
            largest = left;
        }

        if (right < n && arr[right] > arr[largest]) {
            largest = right;
        }

        if (largest != i) {

            int temp = arr[i];

            arr[i] = arr[largest];

            arr[largest] = temp;

            heapify(arr, n, largest);
        }
    }

    public static void main(String[] args) {

        int[] arr = {12, 11, 13, 5, 6, 7};

        heapSort(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[5, 6, 7, 11, 12, 13]
```

**Complexity Analysis**

| Case    | Time                     |
| ------- | ------------------------ |
| Best    | $$\mathcal{O}(n log n)$$ |
| Average | $$\mathcal{O}(n log n)$$ |
| Worst   | $$\mathcal{O}(n log n)$$ |

Auxiliary Space: O(log n) for the recursive heapify stack in this implementation.

Stable: No.

In-place: Yes, apart from recursion stack.

## 4.3 Java Built-in Sorting

Java provides built-in sorting methods.

**A. Arrays.sort()**

Used for sorting arrays.

```java
import java.util.Arrays;

class Main {

    public static void main(String[] args) {

        int[] arr = {5, 2, 8, 1, 3};

        Arrays.sort(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[1, 2, 3, 5, 8]
```

**B. Sorting in Descending Order**

For objects, use a Comparator.

```java
import java.util.Arrays;
import java.util.Collections;

class Main {

    public static void main(String[] args) {

        Integer[] arr = {5, 2, 8, 1, 3};

        Arrays.sort(arr, Collections.reverseOrder());

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[8, 5, 3, 2, 1]
```

> Note: Collections.reverseOrder() cannot be used directly with primitive int[].

**C. Collections.sort()**

Used for sorting lists.

```java
import java.util.*;

class Main {

    public static void main(String[] args) {

        ArrayList<Integer> list =
            new ArrayList<>(Arrays.asList(5, 2, 8, 1, 3));

        Collections.sort(list);

        System.out.println(list);
    }
}
```

Output:

```text
[1, 2, 3, 5, 8]
```

## 4.4 Important Comparison of Sorting Algorithms

| Algorithm      | Best                     | Average                  | Worst                    | Stable                    |
| -------------- | ------------------------ | ------------------------ | ------------------------ | ------------------------- |
| Bubble Sort    | $$\mathcal{O}(n)$$       | $$\mathcal{O}(n^2)$$     | $$\mathcal{O}(n^2)$$     | Yes                       |
| Selection Sort | $$\mathcal{O}(n^2)$$     | $$\mathcal{O}(n^2)$$     | $$\mathcal{O}(n^2)$$     | No                        |
| Insertion Sort | $$\mathcal{O}(n)$$       | $$\mathcal{O}(n^2)$$     | $$\mathcal{O}(n^2)$$     | Yes                       |
| Merge Sort     | $$\mathcal{O}(n log n)$$ | $$\mathcal{O}(n log n)$$ | $$\mathcal{O}(n log n)$$ | Yes                       |
| Quick Sort     | $$\mathcal{O}(n log n)$$ | $$\mathcal{O}(n log n)$$ | $$\mathcal{O}(n^2)$$     | No                        |
| Counting Sort  | $$\mathcal{O}(n + k)$$   | $$\mathcal{O}(n + k)$$   | $$\mathcal{O}(n + k)$$   | Depends on implementation |
| Heap Sort      | $$\mathcal{O}(log n)$$   | $$\mathcal{O}(log n)$$   | $$\mathcal{O}(log n)$$   | No                        |

## 4.5 Important Sorting Terminology

**Stable Sorting**

A stable sorting algorithm preserves the relative order of elements with equal keys.

**In-place Sorting**

An in-place sorting algorithm uses very little additional memory, typically O(1) auxiliary storage.

**Adaptive Sorting**

An adaptive algorithm can perform better when the input is already or nearly sorted.

Example: Insertion sort.

**Comparison-Based Sorting**

Sorts elements using comparisons.

Examples:

- Bubble sort
- Selection sort
- Insertion sort
- Merge sort
- Quick sort
- Heap sort

**Non-Comparison-Based Sorting**
Uses properties of keys rather than only pairwise comparisons.

Examples:

- Counting sort
- Radix sort
- Bucket sort

---

# Chapter 3: Practice Questions

## A. Theory and MCQs

1. What is searching?
2. What is the difference between linear and binary search?
3. Why must binary search use sorted data?
4. What is the time complexity of binary search?
5. What is the difference between iterative and recursive binary search?
6. Explain bubble sort with an example.
7. How does selection sort work?
8. Why is insertion sort efficient for nearly sorted arrays?
9. What is the divide-and-conquer technique?
10. Explain merge sort.
11. What is a pivot in quick sort?
12. Why can quick sort take O(n²)?
13. What is a stable sorting algorithm?
14. What is an in-place algorithm?
15. How does counting sort work?
16. What is a max heap?
17. Explainheap sort.
18. Difference between merge sort and quick sort.
19. Difference between Arrays.sort() and Collections.sort().
20. Which sorting algorithms are stable?

## B. Java Coding Practice

**Beginner Level**

1. Implement linear search.
2. Implement binary search.
3. Find the first occurrence of an element.
4. Find the last occurrence of an element.
5. Count occurrences of an element.
6. Implement bubble sort.
7. Implement selection sort.
8. Implement insertion sort.
9. Sort an array in ascending order.
10. Sort an array in descending order.

**Intermediate Level**

1. Implement recursive binary search.
2. Find the insertion position of a target.
3. Find the square root using binary search.
4. Implement merge sort.
5. Implement quick sort.
6. Implement counting sort.
7. Implement heap sort.
8. Merge two sorted arrays.
9. kth smallest element.
10. Find kth largest element.

**Advanced Practice**

1. Search in a rotated sorted array.
2. Find peak element.
3. Find minimum element in a rotated sorted array.
4. Find first and last position of a target.
5. Find median of two sorted arrays.

---

# CDAC Important Preparation Points

1. Memorize best, average and worst time complexities.
2. Understand why binary search is O(log n).
3. Practice tracing binary search manually.
4. Remember that binary search requires sorted data.
5. Understand the number of passes in bubble and selection sort.
6. Learn stable and unstable sorting.
7. Understand the divide-and-conquer approach.
8. Remember merge sort has O(n log n) worst-case time.
9. Remember quick sort has O(n²) worst-case time.
10. Understand pivot selection and partitioning.
11. Memorize heap index formulas.
12. Understand the difference between comparison and non-comparison sorting.
13. Practice Java Arrays.sort() and Collections.sort().
    Be comfortable calculating complexity from nested loops.

---
