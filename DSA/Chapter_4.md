# Chapter 4: Arrays and Matrix Operations with Java

**Course:** Data Structures and Algorithms with Java  
**Preparation:** CDAC C-CAT / CMCE  
**Level:** Beginner to Intermediate  
**Parts:** 4

---

# Table of Contents

1. Part 1: Array Fundamentals and Basic Operations
2. Part 2: Important Array Problems and Techniques
3. Part 3: Two-Dimensional Arrays and Matrix Operations
4. Part 4: Advanced Array Problems and Interview Questions
5. Chapter 4 Quick Revision
6. Practice Questions
7. CDAC Important Preparation Points

---

# PART 1: Array Fundamentals and Basic Operations

## 1.1 What is an Array?

An array is a linear data structure that stores multiple elements of the same data type.

Elements are stored in indexed positions.

Example:

```java
int[] arr = {10, 20, 30, 40, 50};
```

Array representation:

| Index | 0   | 1   | 2   | 3   | 4   |
| ----- | --- | --- | --- | --- | --- |
| Value | 10  | 20  | 30  | 40  | 50  |

Important points:

- Array indexing starts from 0.
- Array size is fixed after creation.
- Arrays can store primitive values and object references.
- Array elements are accessed using indexes.
- Java arrays are objects.

## 1.2 Array Declaration and Initialization

**Declaration**

```java
int[] arr;
```

**Memory Allocation**

```java
arr = new int[5];
```

**Declaration and Initialization**

```java
int[] arr = {10, 20, 30, 40, 50};
```

**Using new Keyword**

```java
int[] arr = new int[]{10, 20, 30, 40};
```

## 1.3 Default Values in Java Arrays

When an array is created using new, its elements receive default values.

| Data Type        | Default Value |
| ---------------- | ------------- |
| int              | 0             |
| float            | 0.0f          |
| double           | 0.0           |
| char             | '\u0000'      |
| boolean          | false         |
| Object reference | null          |

Example:

```java
class Main {
    public static void main(String[] args) {

        int[] arr = new int[5];

        for (int x : arr) {
            System.out.println(x);
        }
    }
}
```

Output:

```text
0
0
0
0
0
```

## 1.4 Accessing Array Elements

```java
class Main {
    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50};

        System.out.println(arr[0]);
        System.out.println(arr[2]);
        System.out.println(arr[4]);
    }
}
```

Output:

```text
10
30
50
```

Accessing an invalid index causes:

ArrayIndexOutOfBoundsException

## 1.5 Traversing an Array

Traversal means visiting every element of an array.

**Using for Loop**

```java
int[] arr = {10, 20, 30, 40};

for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}
```

**Using Enhanced for Loop**

```java
int[] arr = {10, 20, 30, 40};

for (int x : arr) {
    System.out.println(x);
}
```

Time Complexity: O(n)

Auxiliary Space: O(1)

## 1.6 Insertion in an Array

Arrays have fixed capacity. Inserting an element into an existing array may require shifting elements.

Example:

Original:

[10, 20, 30, 40, 50]

Insert 25 at index 2.

Result:

[10, 20, 25, 30, 40, 50]

Java implementation using a larger array:

```java
import java.util.Arrays;

class Main {

    static int[] insert(int[] arr, int index, int value) {

        if (index < 0 || index > arr.length) {
            throw new IllegalArgumentException("Invalid index");
        }

        int[] result = new int[arr.length + 1];

        for (int i = 0; i < index; i++) {
            result[i] = arr[i];
        }

        result[index] = value;

        for (int i = index; i < arr.length; i++) {
            result[i + 1] = arr[i];
        }

        return result;
    }

    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50};

        arr = insert(arr, 2, 25);

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[10, 20, 25, 30, 40, 50]
```

Time Complexity: O(n)

## 1.7 Deletion from an Array

Deletion involves removing an element and shifting the remaining elements.

Example:

Original:

[10, 20, 30, 40, 50]

Delete element at index 2.

Result:

[10, 20, 40, 50]

Java implementation:

```java
import java.util.Arrays;

class Main {

    static int[] delete(int[] arr, int index) {

        if (index < 0 || index >= arr.length) {
            throw new IllegalArgumentException("Invalid index");
        }

        int[] result = new int[arr.length - 1];

        for (int i = 0; i < index; i++) {
            result[i] = arr[i];
        }

        for (int i = index + 1; i < arr.length; i++) {
            result[i - 1] = arr[i];
        }

        return result;
    }

    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50};

        arr = delete(arr, 2);

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[10, 20, 40, 50]
```

Time Complexity: O(n)

## 1.8 Searching in an Array

Searching means finding the position of a particular element.

Two common approaches:

1. Linear Search
2. Binary Search

**Linear Search**

```java
class Main {

    static int search(int[] arr, int target) {

        for (int i = 0; i < arr.length; i++) {

            if (arr[i] == target) {
                return i;
            }
        }

        return -1;
    }

    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50};

        System.out.println(search(arr, 30));
    }
}
```

Output:

```text
2
```

Time Complexity: O(n)

**Binary Search**

Binary search requires a sorted array.

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

            if (arr[mid] < target) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }

        return -1;
    }

    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50};

        System.out.println(binarySearch(arr, 40));
    }
}
```

Output:

```text
3
```

Time Complexity: O(log n)

## 1.9 Finding Maximum and Minimum Elements

**Maximum Element**

```java
class Main {

    public static void main(String[] args) {

        int[] arr = {10, 50, 20, 80, 30};

        int max = arr[0];

        for (int x : arr) {

            if (x > max) {
                max = x;
            }
        }

        System.out.println(max);
    }
}
```

Output:

```text
80
```

**Minimum Element**

```java
class Main {

    public static void main(String[] args) {

        int[] arr = {10, 50, 20, 80, 30};

        int min = arr[0];

        for (int x : arr) {

            if (x < min) {
                min = x;
            }
        }

        System.out.println(min);
    }
}
```

Output:

```text
10
```

## 1.10 Finding Sum and Average

```java
class Main {

    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50};

        int sum = 0;

        for (int x : arr) {
            sum += x;
        }

        double average = (double) sum / arr.length;

        System.out.println("Sum: " + sum);

        System.out.println("Average: " + average);
    }
}
```

Output:

```text
Sum : 150
Average: 30.0
```

---

# PART 2: Important Array Problems and Techniques

## 2.1 Reverse an Array

Reversing means changing the order of elements.

Original:

[10, 20, 30, 40, 50]

Reversed:

[50, 40, 30, 20, 10]

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static void reverse(int[] arr) {

        int left = 0;
        int right = arr.length - 1;

        while (left < right) {

            int temp = arr[left];

            arr[left] = arr[right];

            arr[right] = temp;

            left++;
            right--;
        }
    }

    public static void main(String[] args) {

        int[] arr = {10, 20, 30, 40, 50};

        reverse(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[50, 40, 30, 20, 10]
```

Time Complexity: O(n)

Space Complexity: O(1)

## 2.2 Find the Second Largest Element

Find the second largest distinct element without sorting.

Example:

[10, 20, 5, 40, 30]

Output:

30

**Java Implementation**

```java
class Main {

    static int secondLargest(int[] arr) {

        int largest = Integer.MIN_VALUE;
        int second = Integer.MIN_VALUE;

        for (int x : arr) {

            if (x > largest) {

                second = largest;

                largest = x;

            } else if (x > second && x != largest) {

                second = x;
            }
        }

        return second == Integer.MIN_VALUE ? -1 : second;
    }

    public static void main(String[] args) {

        int[] arr = {10, 20, 5, 40, 30};

        System.out.println(secondLargest(arr));
    }
}
```

Output:

```text
30
```

**Note**: For production-quality code, use a separate boolean flag or an optional result if the array can contain Integer.MIN_VALUE.

Time Complexity: O(n)

## 2.3 Find Duplicate Elements

Example:

[1, 2, 3, 2, 4, 1, 5]

Duplicates: 1 and 2.

**Java Implementation**

```java
import java.util.HashSet;

class Main {

    public static void main(String[] args) {

        int[] arr = {1, 2, 3, 2, 4, 1, 5};

        HashSet<Integer> seen = new HashSet<>();

        HashSet<Integer> duplicates = new HashSet<>();

        for (int x : arr) {

            if (!seen.add(x)) {
                duplicates.add(x);
            }
        }

        System.out.println(duplicates);
    }
}
```

Time Complexity: O(n) expected.

Auxiliary Space: O(n).

## 2.4 Remove Duplicates from a Sorted Array

Example:

Original:

[1, 1, 2, 2, 3, 4, 4]

Unique portion:

[1, 2, 3, 4]

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static int removeDuplicates(int[] arr) {

        if (arr.length == 0) {
            return 0;
        }

        int j = 0;

        for (int i = 1; i < arr.length; i++) {

            if (arr[i] != arr[j]) {

                j++;

                arr[j] = arr[i];
            }
        }

        return j + 1;
    }

    public static void main(String[] args) {

        int[] arr = {1, 1, 2, 2, 3, 4, 4};

        int size = removeDuplicates(arr);

        System.out.println(Arrays.toString(Arrays.copyOf(arr, size)));
    }
}
```

Output:

```text
[1,2,3,4]
```

Time Complexity: O(n)

Space Complexity: O(1), excluding the output copy.

## 2.5 Move All Zeros to the End

Example:

Input:

[0, 1, 0, 3, 12]

Output:

[1, 3, 12, 0, 0]

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static void moveZeros(int[] arr) {

        int index = 0;

        for (int x : arr) {

            if (x != 0) {
                arr[index++] = x;
            }
        }

        while (index < arr.length) {
            arr[index++] = 0;
        }
    }

    public static void main(String[] args) {

        int[] arr = {0, 1, 0, 3, 12};

        moveZeros(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 2.6 Rotate an Array

Rotation shifts array elements by a specified number of positions.

**Left Rotation by One Position**

Input:

[1, 2, 3, 4, 5]

Output:

[2, 3, 4, 5, 1]

```java
import java.util.Arrays;

class Main {

    static void leftRotate(int[] arr) {

        if (arr.length == 0) {
            return;
        }

        int first = arr[0];

        for (int i = 0; i < arr.length - 1; i++) {

            arr[i] = arr[i + 1];
        }

        arr[arr.length - 1] = first;
    }

    public static void main(String[] args) {

        int[] arr = {1, 2, 3, 4, 5};

        leftRotate(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

**Right Rotation by One Position**

```java
static void rightRotate(int[] arr) {

    if (arr.length == 0) {
        return;
    }

    int last = arr[arr.length - 1];

    for (int i = arr.length - 1; i > 0; i--) {

        arr[i] = arr[i - 1];
    }

    arr[0] = last;
}
```

Time Complexity: O(n)

## 2.7 Rotate an Array by K Positions

Example:

Input:

[1, 2, 3, 4, 5, 6, 7]

Right rotation by 3:

[5, 6, 7, 1, 2, 3, 4]

**Java Implementation Using Reversal**

```java
import java.util.Arrays;

class Main {

    static void reverse(int[] arr, int start, int end) {

        while (start < end) {

            int temp = arr[start];

            arr[start] = arr[end];

            arr[end] = temp;

            start++;
            end--;
        }
    }

    static void rotate(int[] arr, int k) {

        int n = arr.length;

        if (n == 0) {
            return;
        }

        k = k % n;

        reverse(arr, 0, n - 1);

        reverse(arr, 0, k - 1);

        reverse(arr, k, n - 1);
    }

    public static void main(String[] args) {

        int[] arr = {1, 2, 3, 4, 5, 6, 7};

        rotate(arr, 3);

        System.out.println(Arrays.toString(arr));
    }
}
```

Output:

```text
[5, 6, 7, 1, 2, 3, 4]
```

Time Complexity: O(n)

Space Complexity: O(1)

## 2.8 Two Sum Problem

Find two elements whose sum equals a target.

Example:

Array:

[2, 7, 11, 15]

Target:

9

Output:

[0, 1]

**Java Implementation Using HashMap**

```java
import java.util.HashMap;
import java.util.Arrays;

class Main {

    static int[] twoSum(int[] arr, int target) {

        HashMap<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < arr.length; i++) {

            int complement = target - arr[i];

            if (map.containsKey(complement)) {

                return new int[]{map.get(complement), i};
            }

            map.put(arr[i], i);
        }

        return new int[]{-1, -1};
    }

    public static void main(String[] args) {

        int[] arr = {2, 7, 11, 15};

        System.out.println(Arrays.toString(twoSum(arr, 9)));
    }
}
```

Output:

```text
[0, 1]
```

Time Complexity: O(n) expected.

Space Complexity: O(n).

## 2.9 Prefix Sum Technique

Prefix sum stores cumulative sums of array elements.

Example:

Original:

[2, 4, 6, 8, 10]

Prefix sum:

[2, 6, 12, 20, 30]

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static int[] prefixSum(int[] arr) {

        int[] prefix = new int[arr.length];

        if (arr.length == 0) {
            return prefix;
        }

        prefix[0] = arr[0];

        for (int i = 1; i < arr.length; i++) {

            prefix[i] = prefix[i - 1] + arr[i];
        }

        return prefix;
    }

    public static void main(String[] args) {

        int[] arr = {2, 4, 6, 8, 10};

        System.out.println(Arrays.toString(prefixSum(arr)));
    }
}
```

Output:

```text
[2,6,12,20,30]
```

**Range Sum Query**

To calculate sum from index L to R:

`sum(L, R) = prefix[R] - prefix[L - 1]`

When L = 0:

`sum(0, R) = prefix[R]`

Time Complexity:

- Prefix construction: O(n)
- Each range sum query: O(1)

## 2.10 Kadane's Algorithm – Maximum Subarray Sum

Kadane's algorithm finds the maximum sum of a contiguous subarray.

Example:

`[-2, 1, -3, 4, -1, 2, 1, -5, 4]`

Maximum subarray:

`[4, -1, 2, 1]`

Maximum sum:
`6`

**Java Implementation**

```java
class Main {

    static int maxSubarraySum(int[] arr) {

        int currentSum = arr[0];

        int maxSum = arr[0];

        for (int i = 1; i < arr.length; i++) {

            currentSum = Math.max(arr[i], currentSum + arr[i]);

            maxSum = Math.max(maxSum, currentSum);
        }

        return maxSum;
    }

    public static void main(String[] args) {

        int[] arr = {-2, 1, -3, 4, -1, 2, 1, -5, 4};

        System.out.println(maxSubarraySum(arr));
    }
}
```

Output:

```text
6
```

Time Complexity: O(n)

Space Complexity: O(1)

Important: This implementation handles arrays containing negative numbers, provided the array is non-empty.

## 2.11 Two Pointer Technique

The two-pointer technique uses two indexes to solve array problems efficiently.

Common applications:

- Reversing arrays.
- Finding pairs with a target sum in sorted arrays.
- Removing duplicates.
- Partitioning arrays.

Example: Pair Sum in a Sorted Array.

Input:

`[1, 2, 4, 6, 8, 9]`

Target = 10

Output:

`2 + 8 = 10`

Java Implementation:

```java
class Main {

    static void findPair(int[] arr, int target) {

        int left = 0;

        int right = arr.length - 1;

        while (left < right) {

            int sum = arr[left] + arr[right];

            if (sum == target) {

                System.out.println(
                    arr[left] + " + " + arr[right]
                );

                return;

            } else if (sum < target) {

                left++;

            } else {

                right--;
            }
        }

        System.out.println("Pair not found");
    }

    public static void main(String[] args) {

        int[] arr = {1, 2, 4, 6, 8, 9};

        findPair(arr, 10);
    }
}
```

Time Complexity: O(n).

> Important: The array must be sorted for this two-pointer approach.

## 2.12 Sliding Window Technique

Sliding window is used to solve problems involving contiguous subarrays or substrings.

Example:

Find maximum sum of a subarray of size K.

Input:

`[2, 1, 5, 1, 3, 2]`

K = 3

Subarrays:

- [2, 1, 5] = 8
- [1, 5, 1] = 7
- [5, 1, 3] = 9
- [1, 3, 2] = 6

Maximum sum = 9.

**Java Implementation:**

```java
class Main {

    static int maxSum(int[] arr, int k) {

        if (k <= 0 || k > arr.length) {
            throw new IllegalArgumentException("Invalid window size");
        }

        int sum = 0;

        for (int i = 0; i < k; i++) {
            sum += arr[i];
        }

        int maxSum = sum;

        for (int i = k; i < arr.length; i++) {

            sum += arr[i];

            sum -= arr[i - k];

            maxSum = Math.max(maxSum, sum);
        }

        return maxSum;
    }

    public static void main(String[] args) {

        int[] arr = {2, 1, 5, 1, 3, 2};

        System.out.println(maxSum(arr, 3));
    }
}
```

```text
9
```

Time Complexity: O(n).

Space Complexity: O(1).

# PART 3: Two-Dimensional Arrays and Matrix Operations

## 3.1 What is a Matrix?

A matrix is a rectangular arrangement of elements in rows and columns.

Example:

```text
1 2 3
4 5 6
7 8 9
```

In Java, matrices can be represented using two-dimensional arrays.

```java
int[][] matrix = new int[3][3];
```

## 3.2 Matrix Initialization

```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};
```

## 3.3 matrix Traversal

```java
class Main {

    public static void main(String[] args) {

        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };

        for (int i = 0; i < matrix.length; i++) {

            for (int j = 0; j < matrix[i].length; j++) {

                System.out.print(matrix[i][j] + " ");
            }

            System.out.println();
        }
    }
}
```

Output:

```text
1 2 3
4 5 6
7 8 9
```

Time Complexity: O(rows \* columns)

## 3.4 Matrix Addition

Two matrices can be added if they have the same dimensions.

Example:

```text
A:       B:

1 2      5 6
3 4      7 8
```

Result:

```text
6  8
10 12
```

**Java Implementation**

```java
class Main {

    public static void main(String[] args) {

        int[][] A = {
            {1, 2},
            {3, 4}
        };

        int[][] B = {
            {5, 6},
            {7, 8}
        };

        int rows = A.length;

        int cols = A[0].length;

        int[][] C = new int[rows][cols];

        for (int i = 0; i < rows; i++) {

            for (int j = 0; j < cols; j++) {

                C[i][j] = A[i][j] + B[i][j];
            }
        }

        for (int[] row : C) {

            for (int x : row) {
                System.out.print(x + " ");
            }

            System.out.println();
        }
    }
}
```

Time Complexity: O(rows \* columns)

## 3.5 Matrix Subtraction

Subtraction is possible when both matrices have the same dimensions.

Formula:

`C[i][j] = A[i][j] - B[i][j]`

Example:

```text
A:       B:

5 6      1 2
7 8      3 4
```

Result:

```text
4 4
4 4
```

## 3.6 Matrix Multiplication

Matrix multiplication is possible when:

`Columns of A = Rows of B`

If A is of size m × n and B is of size n × p, then the result is of size m × p.

**Formula**

`C[i][j] = Sum of (A[i][k] \* B[k][j])`

Java Implementation

```java
class Main {

    public static void main(String[] args) {

        int[][] A = {
            {1, 2},
            {3, 4}
        };

        int[][] B = {
            {5, 6},
            {7, 8}
        };

        int rowsA = A.length;

        int colsA = A[0].length;

        int colsB = B[0].length;

        int[][] C = new int[rowsA][colsB];

        for (int i = 0; i < rowsA; i++) {

            for (int j = 0; j < colsB; j++) {

                for (int k = 0; k < colsA; k++) {

                    C[i][j] += A[i][k] * B[k][j];
                }
            }
        }

        for (int[] row : C) {

            for (int x : row) {
                System.out.print(x + " ");
            }

            System.out.println();
        }
    }
}
```

Output:

```text
19 22
43 50
```

Time Complexity: O(m _ n _ p)

For two n × n matrices: O(n³).

## 3.7 Transpose of a Matrix

Transpose converts rows into columns.

Example:

Original:

```text
1 2 3
4 5 6
```

Transpose:

```text
1 4
2 5
3 6
```

**Java Implementation**

```java
class Main {

    public static void main(String[] args) {

        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6}
        };

        int rows = matrix.length;

        int cols = matrix[0].length;

        int[][] transpose = new int[cols][rows];

        for (int i = 0; i < rows; i++) {

            for (int j = 0; j < cols; j++) {

                transpose[j][i] = matrix[i][j];
            }
        }

        for (int[] row : transpose) {

            for (int x : row) {
                System.out.print(x + " ");
            }

            System.out.println();
        }
    }
}
```

Time Complexity: O(rows \* columns)

## 3.8 Primary and Secondary Diagonals

Consider:

```text
1 2 3
4 5 6
7 8 9
```

Primary diagonal:

`1, 5, 9`

Condition:

`i == j`

Secondary diagonal:

`3, 5, 7`

Condition:

`i + j == n - 1`

**Java Implementation**

```java
class Main {

    public static void main(String[] args) {

        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };

        int n = matrix.length;

        int primarySum = 0;

        int secondarySum = 0;

        for (int i = 0; i < n; i++) {

            primarySum += matrix[i][i];

            secondarySum += matrix[i][n - 1 - i];
        }

        System.out.println("Primary: " + primarySum);

        System.out.println("Secondary: " + secondarySum);
    }
}
```

Output:

```text
Primary: 15
Secondary: 15
```

## 3.9 Sum of Each Row and Column

```java
class Main {

    public static void main(String[] args) {

        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };

        int n = matrix.length;

        // Row sum

        for (int i = 0; i < n; i++) {

            int sum = 0;

            for (int j = 0; j < matrix[i].length; j++) {

                sum += matrix[i][j];
            }

            System.out.println("Row " + i + ": " + sum);
        }

        // Column sum

        for (int j = 0; j < matrix[0].length; j++) {

            int sum = 0;

            for (int i = 0; i < n; i++) {

                sum += matrix[i][j];
            }

            System.out.println("Column " + j + ": " + sum);
        }
    }
}
```

Output:

```text
Row 0: 6
Row 1: 15
Row 2: 24

Column 0: 12
Column 1: 15
Column 2: 18
```

## 3.10 Rotate a Matrix by 90 Degrees

For a square matrix, clockwise 90-degree rotation can be achieved by:

1. Transposing the matrix.
2. Reversing each row.

Original:

```text
1 2 3
4 5 6
7 8 9
```

After clockwise rotation:

```text
7 4 1
8 5 2
9 6 3
```

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static void rotate(int[][] matrix) {

        int n = matrix.length;

        // Transpose

        for (int i = 0; i < n; i++) {

            for (int j = i + 1; j < n; j++) {

                int temp = matrix[i][j];

                matrix[i][j] = matrix[j][i];

                matrix[j][i] = temp;
            }
        }

        // Reverse each row

        for (int i = 0; i < n; i++) {

            int left = 0;

            int right = n - 1;

            while (left < right) {

                int temp = matrix[i][left];

                matrix[i][left] = matrix[i][right];

                matrix[i][right] = temp;

                left++;

                right--;
            }
        }
    }

    public static void main(String[] args) {

        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };

        rotate(matrix);

        for (int[] row : matrix) {
            System.out.println(Arrays.toString(row));
        }
    }
}
```

Time Complexity: O(n²)

Auxiliary Space: O(1)

## 3.11 Search in a Row-Wise and Column-Wise Sorted Matrix

Consider:

```text
1  4  7  11
2  5  8  12
3  6  9  16
10 13 14 17
```

Each row and column is sorted in ascending order.

We can search efficiently by starting from the top-right corner.

**Algorithm**

1. Start at the top-right element.
2. If target equals current element, return its position.
3. If target is smaller, move left.
4. If target is greater, move down.
5. Continue until found or outside the matrix.

**Java Implementation**

```java
class Main {

    static boolean search(int[][] matrix, int target) {

        int rows = matrix.length;

        int cols = matrix[0].length;

        int i = 0;

        int j = cols - 1;

        while (i < rows && j >= 0) {

            if (matrix[i][j] == target) {
                return true;
            }

            if (matrix[i][j] > target) {
                j--;
            } else {
                i++;
            }
        }

        return false;
    }

    public static void main(String[] args) {

        int[][] matrix = {
            {1, 4, 7, 11},
            {2, 5, 8, 12},
            {3, 6, 9, 16},
            {10, 13, 14, 17}
        };

```

## 3.12 Upper and Lower Triangular Matrix

Upper Triangular Matrix
All elements below the main diagonal are zero.

```text
1 2 3
0 4 5
0 0 6
```

Condition:

i > j elements must be zero.

Lower Triangular Matrix
All elements above the main diagonal are zero.

```text
1 0 0
2 3 0
4 5 6
```

Condition:

i < j elements must be zero.

## 3.13 Check Identity Matrix

An identity matrix has:

- 1 on the main diagonal.
- 0 everywhere else.

Example:

```text 1 0 0
0 1 0
0
```

## 3.14 Check Whether a Matrix is Symmetric

A square matrix is symmetric if:

A[i][j] = A[j][i]

In simple words, a matrix is symmetric if it is equal to its transpose.

Example:

1 2 3
2 4 5
3 5 6

This matrix is symmetric.

### Java Implementation

class Main {

    static boolean isSymmetric(int[][] matrix) {

        int n = matrix.length;

        for (int i = 0; i < n; i++) {

            for (int j = 0; j < n; j++) {

                if (matrix[i][j] != matrix[j][i]) {
                    return false;
                }
            }
        }

        return true;
    }

    public static void main(String[] args) {

        int[][] matrix = {
            {1, 2, 3},
            {2, 4, 5},
            {3, 5, 6}
        };

        System.out.println(isSymmetric(matrix));
    }

}

Output:

true

Time Complexity: O(n²)
Space Complexity: O(1)

Important: A symmetric matrix must be square.

## 3.16 Find Maximum Element in Each Row

Find the maximum value from every row.

Example:

Input:

1 5 3
8 2 6
4 9 7

Output:

Row 0: 5
Row 1: 8
Row 2: 9

### Java Implementation

class Main {

    public static void main(String[] args) {

        int[][] matrix = {
            {1, 5, 3},
            {8, 2, 6},
            {4, 9, 7}
        };

        for (int i = 0; i < matrix.length; i++) {

            int max = matrix[i][0];

            for (int j = 1; j < matrix[i].length; j++) {

                if (matrix[i][j] > max) {
                    max = matrix[i][j];
                }
            }

            System.out.println("Row " + i + ": " + max);
        }
    }

}

Output:

Row 0: 5
Row 1: 8
Row 2: 9

Time Complexity: O(m \* n)
Space Complexity: O(1)

---

# Chapter 4: Arrays and Matrix Operations with Java

# PART 4: Advanced Array Problems and Techniques

## Table of Contents

1. Kadane's Algorithm – Maximum Subarray Sum
2. Majority Element – Boyer-Moore Voting Algorithm
3. Best Time to Buy and Sell Stock
4. Two Sum Problem
5. Three Sum Problem
6. Sliding Window Technique
7. Trapping Rain Water
8. Merge Two Sorted Arrays
9. Subarray with Given Sum
10. Longest Consecutive Sequence
11. Product of Array Except Self
12. Search in Rotated Sorted Array
13. Important Array Problem Patterns
14. Time and Space Complexity Summary
15. Practice Questions
16. CDAC Preparation Points

---

# 4.1 Kadane's Algorithm – Maximum Subarray Sum

Kadane's Algorithm is used to find the maximum sum of a contiguous subarray.

### Example

Input:

`[-2, 1, -3, 4, -1, 2, 1, -5, 4]`

Maximum Subarray:

`[4, -1, 2, 1]`

Maximum Sum:

`4 + (-1) + 2 + 1 = 6`

### Approach

Maintain two variables:

- currentSum: Maximum sum ending at the current position.
- maxSum: Maximum sum found so far.

At each position:

`currentSum = max(arr[i], currentSum + arr[i])`

`maxSum = max(maxSum, currentSum)`

### Java Implementation

```java
class Main {

    static int maxSubArray(int[] arr) {

        int currentSum = arr[0];
        int maxSum = arr[0];

        for (int i = 1; i < arr.length; i++) {

            currentSum = Math.max(
                arr[i],
                currentSum + arr[i]
            );

            maxSum = Math.max(maxSum, currentSum);
        }

        return maxSum;
    }

    public static void main(String[] args) {

        int[] arr = {-2, 1, -3, 4, -1, 2, 1, -5, 4};

        System.out.println(maxSubArray(arr));
    }
}
```

Output:

```text
6
```

Time Complexity: O(n)

Space Complexity: O(1)

**CDAC Important**: Kadane's Algorithm is a frequently used array problem. Understand how it handles negative numbers.

## 4.2 Majority Element – Boyer-Moore Voting Algorithm

A majority element is an element that appears more than n/2 times.

Example
Input:

`[2, 2, 1, 1, 2, 2, 2]`

Output:

`2`

Frequency of 2 = 5

Array length = 7

Since 5 > 7/2, 2 is the majority element.

**Approach**

Boyer-Moore Voting Algorithm uses:

- candidate
- count

Rules:

1. If count is zero, select the current element as candidate.
2. If current element equals candidate, increment count.
3. Otherwise, decrement count.
4. Verify the candidate because a majority element may not exist.

**Java Implementation**

```java
class Main {

    static int majorityElement(int[] arr) {

        int candidate = 0;
        int count = 0;

        for (int x : arr) {

            if (count == 0) {
                candidate = x;
            }

            if (x == candidate) {
                count++;
            } else {
                count--;
            }
        }

        // Verify candidate

        count = 0;

        for (int x : arr) {

            if (x == candidate) {
                count++;
            }
        }

        if (count > arr.length / 2) {
            return candidate;
        }

        return -1;
    }

    public static void main(String[] args) {

        int[] arr = {2, 2, 1, 1, 2, 2, 2};

        System.out.println(majorityElement(arr));
    }
}
```

Output:

```text
2
```

Time Complexity: O(n)

Space Complexity: O(1)

## 4.3 Best Time to Buy and Sell Stock

Find the maximum profit possible by buying and selling a stock once.

The buying day must come before the selling day.

Example
Input:

`[7, 1, 5, 3, 6, 4]`

Buy at price = 1

Sell at price = 6

Maximum Profit = 6 - 1 = 5

**Approach**

Maintain:

- Minimum price encountered so far.
- Maximum profit encountered so far.

For every price:

`profit = currentPrice - minimumPrice`

**Java Implementation**

```java
class Main {

    static int maxProfit(int[] prices) {

        int minPrice = Integer.MAX_VALUE;

        int maxProfit = 0;

        for (int price : prices) {

            if (price < minPrice) {
                minPrice = price;
            }

            int profit = price - minPrice;

            maxProfit = Math.max(maxProfit, profit);
        }

        return maxProfit;
    }

    public static void main(String[] args) {

        int[] prices = {7, 1, 5, 3, 6, 4};

        System.out.println(maxProfit(prices));
    }
}
```

Output:

```text
5
```

Time Complexity: O(n)

Space Complexity: O(1)

## 4.4 Two Sum Problem

Find two elements whose sum equals the target value.

Example
Input:

`arr = [2, 7, 11, 15]`

Target = 9

Output:

`[0, 1]`

Because:

`arr[0] + arr[1] = 2 + 7 = 9`

**Approach 1: Brute Force**

Check every possible pair.

**Java Implementation**

```java
class Main {

    static void twoSum(int[] arr, int target) {

        for (int i = 0; i < arr.length; i++) {

            for (int j = i + 1; j < arr.length; j++) {

                if (arr[i] + arr[j] == target) {

                    System.out.println(i + " " + j);
                    return;
                }
            }
        }

        System.out.println("Pair not found");
    }

    public static void main(String[] args) {

        int[] arr = {2, 7, 11, 15};

        twoSum(arr, 9);
    }
}
```

Time Complexity: O(n²)

Space Complexity: O(1)

Approach 2: Using HashMap
A HashMap stores previously visited elements and their indices.

For each element:

`complement = target - arr[i]`

Check whether the complement already exists.

**Java Implementation**

```java
import java.util.HashMap;

class Main {

    static void twoSum(int[] arr, int target) {

        HashMap<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < arr.length; i++) {

            int complement = target - arr[i];

            if (map.containsKey(complement)) {

                System.out.println(
                    map.get(complement) + " " + i
                );

                return;
            }

            map.put(arr[i], i);
        }

        System.out.println("Pair not found");
    }

    public static void main(String[] args) {

        int[] arr = {2, 7, 11, 15};

        twoSum(arr, 9);
    }
}
```

Output:

```text
0 1
```

Time Complexity: O(n) average

Space Complexity: O(n)

**Important**: HashMap is useful when searching for complements or counting frequencies.

## 4.5 Three Sum Problem

Find all unique triplets whose sum is equal to zero.

Example
Input:

`[-1, 0, 1, 2, -1, -4]`

Output:

```text
[-1, -1, 2]
[-1, 0, 1]
```

**Approach**

1. Sort the array.
2. Fix one element.
3. Use two pointers for the remaining elements.
4. Move pointers based on the calculated sum.
5. Skip duplicates to avoid repeated triplets.

**Java Implementation**

```java
import java.util.\*;

class Main {

    static List<List<Integer>> threeSum(int[] arr) {

        Arrays.sort(arr);

        List<List<Integer>> result = new ArrayList<>();

        for (int i = 0; i < arr.length - 2; i++) {

            if (i > 0 && arr[i] == arr[i - 1]) {
                continue;
            }

            int left = i + 1;
            int right = arr.length - 1;

            while (left < right) {

                long sum = (long) arr[i] + arr[left] + arr[right];

                if (sum == 0) {

                    result.add(Arrays.asList(
                        arr[i],
                        arr[left],
                        arr[right]
                    ));

                    left++;
                    right--;

                    while (left < right &&
                           arr[left] == arr[left - 1]) {
                        left++;
                    }

                    while (left < right &&
                           arr[right] == arr[right + 1]) {
                        right--;
                    }

                } else if (sum < 0) {

                    left++;

                } else {

                    right--;
                }
            }
        }

        return result;
    }

    public static void main(String[] args) {

        int[] arr = {-1, 0, 1, 2, -1, -4};

        System.out.println(threeSum(arr));
    }

}
```

Output:

```text
[[-1, -1, 2], [-1, 0, 1]]
```

Time Complexity: O(n²)

Space Complexity: O(1) auxiliary space, excluding sorting implementation and output storage.

## 4.6 Sliding Window Technique

Sliding Window is used to solve problems involving contiguous subarrays or substrings.

Instead of repeatedly calculating the sum of every window, we update the existing window.

Example
Find the maximum sum of a subarray of size K.

Input:

`[2, 1, 5, 1, 3, 2]`

K = 3

Possible windows:

| Subarray | Sum |
| -------- | --- |
| 2,1,5    | 8   |
| 1,5,1    | 7   |
| 5,1,3    | 9   |
| 1,3,2    | 6   |

Maximum Sum = 9

**Java Implementation**

```java
class Main {

    static int maxSum(int[] arr, int k) {

        if (k <= 0 || k > arr.length) {
            throw new IllegalArgumentException("Invalid K");
        }

        int sum = 0;

        // Calculate first window

        for (int i = 0; i < k; i++) {
            sum += arr[i];
        }

        int maxSum = sum;

        // Slide the window

        for (int i = k; i < arr.length; i++) {

            sum += arr[i];

            sum -= arr[i - k];

            maxSum = Math.max(maxSum, sum);
        }

        return maxSum;
    }

    public static void main(String[] args) {

        int[] arr = {2, 1, 5, 1, 3, 2};

        System.out.println(maxSum(arr, 3));
    }
}
```

Outpu:

```text
9
```

Time Complexity: O(n)

Space Complexity: O(1)

**Types of Sliding Window**

| Type                                    | Example                                        |
| --------------------------------------- | ---------------------------------------------- |
| Fixed Window                            | Maximum sum of K elements                      |
| Variable Window                         | Longest substring without repeating characters |
| Variable Window                         | Smallest subarray with given sum (+ve value)   |
| Variable Window                         |
| Longest subarray satisfying a condition |

## 4.7 Trapping Rain Water

Find the amount of water trapped between buildings represented by an array of heights.

Example
Input:

`[4, 2, 0, 3, 2, 5]`

Output:

9

**Concept**

Water trapped at an index depends on:

- Maximum height to its left.
- Maximum height to its right.
- Current height.

Formula:

`Water[i] = min(leftMax, rightMax) - height[i]`

The amount is non-negative for valid boundaries.

**Java Implementation – Two Pointer Approach**

```java
class Main {

    static int trap(int[] height) {

        int left = 0;

        int right = height.length - 1;

        int leftMax = 0;

        int rightMax = 0;

        int water = 0;

        while (left < right) {

            if (height[left] <= height[right]) {

                if (height[left] >= leftMax) {

                    leftMax = height[left];

                } else {

                    water += leftMax - height[left];
                }

                left++;

            } else {

                if (height[right] >= rightMax) {

                    rightMax = height[right];

                } else {

                    water += rightMax - height[right];
                }

                right--;
            }
        }

        return water;
    }

    public static void main(String[] args) {

        int[] height = {4, 2, 0, 3, 2, 5};

        System.out.println(trap(height));
    }
}
```

Output:

```text
9
```

Time Complexity: O(n)

Space Complexity: O(1)

## 4.8 Merge Two Sorted Arrays

Merge two sorted arrays into one sorted array.

Example
Array 1:

`[1, 3, 5]`

Array 2:

`[2, 4, 6]`

Output:

`[1, 2, 3, 4, 5, 6]`

Java Implementation

```java
import java.util.Arrays;

class Main {

    static int[] merge(int[] a, int[] b) {

        int n = a.length;

        int m = b.length;

        int[] result = new int[n + m];

        int i = 0;
        int j = 0;
        int k = 0;

        while (i < n && j < m) {

            if (a[i] <= b[j]) {

                result[k++] = a[i++];

            } else {

                result[k++] = b[j++];
            }
        }

        while (i < n) {
            result[k++] = a[i++];
        }

        while (j < m) {
            result[k++] = b[j++];
        }

        return result;
    }

    public static void main(String[] args) {

        int[] a = {1, 3, 5};

        int[] b = {2, 4, 6};

        System.out.println(
            Arrays.toString(merge(a, b))
        );
    }
}
```

Output:

```text
[1, 2, 3, 4, 5, 6]
```

Time Complexity: O(n + m)

Space Complexity: O(n + m)

## 4.9 Subarray with Given Sum

Find a contiguous subarray whose sum equals a given target.

For positive integers, a variable-size sliding window can solve this problem efficiently.

Example
Input:

`[1, 2, 3, 7, 5]`

Target = 12

Output:

`[2, 3, 7]`

**Java Implementation – Positive Integers**

```java
class Main {

    static void subarraySum(int[] arr, int target) {

        int left = 0;

        int sum = 0;

        for (int right = 0; right < arr.length; right++) {

            sum += arr[right];

            while (sum > target && left <= right) {

                sum -= arr[left];

                left++;
            }

            if (sum == target) {

                System.out.println(
                    "Found from index " + left +
                    " to " + right
                );

                return;
            }
        }

        System.out.println("Subarray not found");
    }

    public static void main(String[] args) {

        int[] arr = {1, 2, 3, 7, 5};

        subarraySum(arr, 12);
    }
}
```

Output:

```text
Found from index 1 to 3
```

Time Complexity: O(n)

Space Complexity: O(1)

Important: This sliding-window implementation assumes non-negative elements. For arrays containing negative numbers, use prefix sums with a HashMap.

## 4.10 Longest Consecutive Sequence

Find the length of the longest sequence of consecutive numbers.

Example
Input:

`[100, 4, 200, 1, 3, 2]`

Output:

4

Longest consecutive sequence:

`[1, 2, 3, 4]`

**Approach**

Use a HashSet:

1. Insert all elements into a HashSet.
2. Check whether the previous number exists.
3. Start counting only when the current number is the beginning of a sequence.
4. Track the maximum length.

**Java Implementation**

```java
import java.util.HashSet;

class Main {

    static int longestConsecutive(int[] arr) {

        HashSet<Integer> set = new HashSet<>();

        for (int x : arr) {
            set.add(x);
        }

        int longest = 0;

        for (int x : set) {

            if (x == Integer.MIN_VALUE || !set.contains(x - 1)) {

                int current = x;

                int count = 1;

                while (current != Integer.MAX_VALUE &&
                       set.contains(current + 1)) {

                    current++;

                    count++;
                }

                longest = Math.max(longest, count);
            }
        }

        return longest;
    }

    public static void main(String[] args) {

        int[] arr = {100, 4, 200, 1, 3, 2};

        System.out.println(longestConsecutive(arr));
    }
}
```

Time Complexity: O(n) average

Space Complexity: O(n)

## 4.11 Product of Array Except Self

Return an array where each position contains the product of all other elements.

Division should not be used.

Example
Input:

`[1, 2, 3, 4]`

Output:

`[24, 12, 8, 6]`

Explanation:

| Index | Product                    |
| ----- | -------------------------- |
| 0     | $$\mathcal2 * 3 * 4 = 24$$ |
| 1     | $$\mathcal1 * 3 * 4 = 12$$ |
| 2     | $$\mathcal1 * 2 * 4 = 8$$  |
| 3     | $$\mathcal1 * 2 * 3 = 6$$  |

**Java Implementation**

```java
import java.util.Arrays;

class Main {

    static int[] productExceptSelf(int[] arr) {

        int n = arr.length;

        int[] result = new int[n];

        int prefix = 1;

        for (int i = 0; i < n; i++) {

            result[i] = prefix;

            prefix *= arr[i];
        }

        int suffix = 1;

        for (int i = n - 1; i >= 0; i--) {

            result[i] *= suffix;

            suffix *= arr[i];
        }

        return result;
    }

    public static void main(String[] args) {

        int[] arr = {1, 2, 3, 4};

        System.out.println(
            Arrays.toString(productExceptSelf(arr))
        );
    }
}
```

Output:

```text
[24, 12, 8, 6]
```

Time Complexity: O(n)

Space Complexity: O(1) auxiliary space, excluding the output array.

**Note**: Integer overflow is possible for large products.

## 4.12 Search in Rotated Sorted Array

A sorted array may be rotated at some pivot.

Example:

Original:

`[1, 2, 3, 4, 5, 6, 7]`

Rotated:

`[4, 5, 6, 7, 1, 2, 3]`
Find target = 2.

Output:

5

**Approach**

Use modified binary search.

At every iteration:

1. Find the middle element.
2. Identify which half is sorted.
3. Check whether the target belongs to that half.
4. Eliminate the other half.

**Java Implementation**

```java
class Main {

    static int search(int[] arr, int target) {

        int low = 0;

        int high = arr.length - 1;

        while (low <= high) {

            int mid = low + (high - low) / 2;

            if (arr[mid] == target) {
                return mid;
            }

            // Left half is sorted
            if (arr[low] <= arr[mid]) {

                if (target >= arr[low] &&
                    target < arr[mid]) {

                    high = mid - 1;

                } else {

                    low = mid + 1;
                }

            } else {

                // Right half is sorted
                if (target > arr[mid] &&
                    target <= arr[high]) {

                    low = mid + 1;

                } else {

                    high = mid - 1;
                }
            }
        }

        return -1;
    }

    public static void main(String[] args) {

        int[] arr = {4, 5, 6, 7, 1, 2, 3};

        System.out.println(search(arr, 2));
    }
}
```

Output:

```text
5
```

Time Complexity: O(log n)

Space Complexity: O(1)

Important: This implementation assumes distinct elements.

## 4.13 Important Array Problem Patterns

Understanding problem-solving patterns is more important than memorizing individual programs.

| Pattern            | Important Problems           |
| ------------------ | ---------------------------- |
| Two Pointers       | Pair sum, Reverse Array      |
| Sliding Window     | Maximum Sum of K Elements    |
| Prefix Sum         | Range Sum Queries            |
| Hashing            | Two Sum, Frequency Counting  |
| Sorting            | Three Sum, Duplicate Removal |
| Binary Search      | Rotated Sorted Array         |
| Kadane's Algorihtm | Maximum Subarray Sum         |
| Greedy             | Stock Buy and Sell           |
| Matrix Traversal   | Spiral Matrix                |
| HashSet            | Longest Consecutive Sequence |

## 4.14 Time and Space Complexity Summary

| Algorithm                    | Time                               | Space                                          |
| ---------------------------- | ---------------------------------- | ---------------------------------------------- |
| Kadane's Algorithm           | $$\mathcal{O}(n)$$                 | $$\mathcal{O}(1)$$                             |
| Majority Elements            | $$\mathcal{O}(n)$$                 | $$\mathcal{O}(1)$$                             |
| Stock Buy and Sell           | $$\mathcal{O}(n)$$                 | $$\mathcal{O}(1)$$                             |
| Two Sum- Brute Force         | $$\mathcal{O}(n^2)$$               | $$\mathcal{O}(1)$$                             |
| Two Sum - HashMap            | $$\mathcal{O}(n)$$ average         | $$\mathcal{O}(n)$$                             |
| Three Sum                    | $$\mathcal{O}(n^2)$$ after sorting | $$\mathcal{O}(1)$$ auxiliary, excluding output |
| Sliding Window               | $$\mathcal{O}(n)$$                 | $$\mathcal{O}(1)$$                             |
| Trapping Rain Water          | $$\mathcal{O}(n)$$                 | $$\mathcal{O}(1)$$                             |
| Merger Sorted Array          | $$\mathcal{O}(n + m)$$             | $$\mathcal{O}(n + m)$$                         |
| Subarray Sum - Positive      | $$\mathcal{O}(n)$$                 | $$\mathcal{O}(1)$$                             |
| Longest Consecutive Sequence | $$\mathcal{O}(n)$$ average         | $$\mathcal{O}(n)$$                             |
| Product Except Self          | $$\mathcal{O}(n)$$                 | $$\mathcal{O}(1)$$ auxiliary                   |
| Rotated Array Search         | $$\mathcal{O}(log n)$$             | $$\mathcal{O}(1)$$                             |

## 4.15 Practice Questions

**A. Beginner Level**

1. Find the maximum and minimum element.
2. Find the second largest element.
3. Reverse an array.
4. Check whether an array is sorted.
5. Move all zeros to the end.
6. Find the missing number.
7. Count the frequency of elements.
8. Remove duplicates from a sorted array.

**B. Intermediate Level**

1. Find two numbers whose sum equals a target.
2. Find the majority element.
3. Find the maximum subarray sum.
4. Rotate an array by K positions.
5. Merge two sorted arrays.
6. Find the best time to buy and sell stock.
7. Find a subarray with a given sum.
8. Find the longest consecutive sequence.

**C. Advanced Level**

1. Solve the Three Sum problem.
2. Solve the Trapping Rain Water problem.
3. Find product of array except self.
4. Search in a rotated sorted array.
5. Find the maximum sum of a subarray of size K.
6. Find the longest substring without repeating characters.
7. Find the minimum size subarray with a target sum.
8. Find the maximum product subarray.
9. Find the median of two sorted arrays.

**D. Matrix-Based Practice**

1. Rotate a matrix by 90 degrees.
2. Print matrix in spiral order.
3. Search in a sorted matrix.
4. Set matrix rows and columns to zero.
5. Find the maximum sum submatrix.

## 4.16 CDAC C-CAT / CMCE Preparation Points

Focus on the following concepts for examination preparation:

1. Array declaration, initialization and traversal.
2. Array insertion and deletion.
3. Linear Search and Binary Search.
4. Sorting and searching techniques.
5. Two Pointer technique.
6. Sliding Window technique.
7. Prefix Sum technique.
8. Kadane's Algorithm.
9. Majority Element using Boyer-Moore.
10. Two Sum using HashMap.
11. Three Sum using Sorting and Two Pointers.
12. Stock Buy and Sell problem.
13. Trapping Rain Water.
14. Merge Two Sorted Arrays.
15. Longest Consecutive Sequence.
16. Product of Array Except Self.
17. Search in Rotated Sorted Array.
18. Matrix traversal and rotation.
19. Time and Space Complexity analysis.
20. Dry-run practice for every important algorithm.
