---
title: "DSA Interview Questions"
date: 2026-06-16
lastmod: 2026-08-01
weight: 1
draft: false
tags: ["DSA", "Array", "Interview"]
categories: ["Data Structures"]
---

Quick reference for interview questions organized by company and topic. Click on any question to expand details.

<!--more-->

---

<details>
<summary><strong>Move Zeroes to End</strong> - <code>Array</code></summary>

### Problem Statement

Given an integer array `nums`, move all zeroes to the end of it while maintaining the relative order of the non-zero elements.

**Note:** You must do this in-place without making a copy of the array.

**Example 1:**
```
Input: nums = [0,1,0,3,12]
Output: [1,3,12,0,0]
```

**Example 2:**
```
Input: nums = [0]
Output: [0]
```

### Solution - Java

#### Approach: Two-Pointer Technique

The idea is to keep track of the position where the next non-zero element should be placed. We iterate through the array, and whenever we find a non-zero element, we swap it with the element at the position where the next non-zero should go.

```java
class Solution {
    public void moveZeroes(int[] nums) {
        // Position where the next non-zero element should be placed
        int nonZeroIndex = 0;
        
        // First pass: place all non-zero elements at the beginning
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != 0) {
                nums[nonZeroIndex] = nums[i];
                nonZeroIndex++;
            }
        }
        
        // Second pass: fill the remaining positions with zeros
        while (nonZeroIndex < nums.length) {
            nums[nonZeroIndex] = 0;
            nonZeroIndex++;
        }
    }
}
```

#### Optimized Approach: Swap Only When Needed

```java
class Solution {
    public void moveZeroes(int[] nums) {
        int left = 0;
        for(int i = 0; i < nums.length; i++){
            if(nums[i] != 0) {
                int temp = nums[left];
                nums[left] = nums[i];
                nums[i] = temp;
                left++; 
            }
        }
    }
}
```

### Complexity Analysis

- **Time Complexity:** O(n) - We iterate through the array at most twice
- **Space Complexity:** O(1) - In-place solution, only using a pointer variable

### Key Points

1. **In-place modification**: The solution modifies the array without using extra space
2. **Maintains relative order**: Non-zero elements keep their original relative order
3. **Two-pointer technique**: Using one pointer to track where to place elements

### LeetCode Reference

[LeetCode 283 - Move Zeroes](https://leetcode.com/problems/move-zeroes/)

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Flatten Object</strong> - <code>Object</code> | <code>JavaScript</code></summary>

### Problem Statement

Given a nested object, flatten it into a single-level object where nested keys are joined with an underscore. **Arrays are left as-is** — only plain nested objects get flattened.

**Example:**
```javascript
Input:
{
  name: 'John',
  age: 24,
  department: {
    name: 'Consumer Experience',
    section: 'Technical',
    branch: {
      name: 'Bangalore',
      timezone: 'IST'
    }
  },
  company: {
    name: 'SAP',
    customers: ['Nike', 'Adidas']
  },
  skills: ['javascript', 'node.js', 'AWS']
}

Output:
{
  name: 'John',
  age: 24,
  department_name: 'Consumer Experience',
  department_section: 'Technical',
  department_branch_name: 'Bangalore',
  department_branch_timezone: 'IST',
  company_name: 'SAP',
  company_customers: ['Nike', 'Adidas'],
  skills: ['javascript', 'node.js', 'AWS']
}
```

### Solution - JavaScript

#### Approach: Recursive Flattening

```javascript
function flattenObject(obj, prefix = '') {
    const result = {};
    
    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
            const value = obj[key];
            const newKey = prefix ? `${prefix}_${key}` : key;
            
            // If value is an object and not null/array, recurse
            if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
                Object.assign(result, flattenObject(value, newKey));
            } else {
                result[newKey] = value;
            }
        }
    }
    
    return result;
}
```

#### Usage Example

```javascript
const flattened = flattenObject(input);
console.log(flattened);
// { name: 'John', age: 24, department_name: 'Consumer Experience', ... }
```

### Complexity Analysis

- **Time Complexity:** O(n) - Where n is the total number of key-value pairs in the nested object
- **Space Complexity:** O(n) - For the flattened result object

### Key Points

1. **Recursive approach**: Traverses nested structures depth-first
2. **Key construction**: Concatenates parent and child keys with underscore
3. **Arrays are NOT flattened**: `typeof [] === 'object'`, so without the explicit `!Array.isArray(value)` check, arrays would get wrongly recursed into. This is the detail interviewers are actually testing.
4. **Preserves primitive values**: Non-object values (including arrays) are added directly to the result

**Companies:** <code>Nike</code>

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Reverse a String</strong> - <code>String</code> | <code>JavaScript/Java</code></summary>

### Problem Statement

Given a string, reverse it in-place or return a reversed string. The characters should appear in reverse order.

**Example 1:**
```javascript
Input: "abcde"
Output: "edcba"
```

**Example 2:**
```javascript
Input: "hello"
Output: "olleh"
```

### Solution - JavaScript

#### Approach 1: Using Array Methods

```javascript
function reverseString(str) {
    return str.split('').reverse().join('');
}
```

#### Approach 2: Using a Loop

```javascript
function reverseString(str) {
    let reversed = '';
    for (let i = str.length - 1; i >= 0; i--) {
        reversed += str[i];
    }
    return reversed;
}
```

#### Approach 3: Using Recursion

```javascript
function reverseString(str, index = str.length - 1) {
    if (index < 0) {
        return '';
    }
    return str[index] + reverseString(str, index - 1);
}
```

#### Approach 4: Using Spread Operator

```javascript
function reverseString(str) {
    return [...str].reverse().join('');
}
```

### Solution - Java

```java
class Solution {
    public String reverseString(String s) {
        char[] chars = s.toCharArray();
        int left = 0, right = chars.length - 1;
        
        while (left < right) {
            // Swap characters
            char temp = chars[left];
            chars[left] = chars[right];
            chars[right] = temp;
            
            left++;
            right--;
        }
        
        return new String(chars);
    }
}
```

### Complexity Analysis

- **Time Complexity:**
  - Array methods: O(n)
  - Loop: O(n)
  - Recursion: O(n)
  - Spread operator: O(n)

- **Space Complexity:**
  - Array methods: O(n) - Creating new array
  - Loop: O(n) - Creating new string
  - Recursion: O(n) - Call stack
  - Spread operator: O(n) - Creating new array

### Key Points

1. **Multiple approaches**: Different ways to solve depending on requirements
2. **In-place vs. new string**: Consider space constraints
3. **Two-pointer technique**: Efficient for character-level swapping
4. **String immutability**: In Java, strings are immutable, so conversion to char array is needed

### LeetCode References

- [LeetCode 344 - Reverse String](https://leetcode.com/problems/reverse-string/)
- [LeetCode 541 - Reverse String II](https://leetcode.com/problems/reverse-string-ii/)

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Jump Game</strong> - <code>Greedy</code> | <code>Array</code></summary>

### Problem Statement

Given an array `nums` where `nums[i]` is the max jump length from index `i`, determine if you can reach the last index starting from index 0.

**Example:**
```
Input: nums = [2,3,1,1,4]
Output: true
```

### Approach: Greedy — Track Farthest Reachable Index

Keep extending the farthest index you can reach. If your current index ever exceeds the farthest reachable index, you're stuck.

```java
class Solution {
    public boolean canJump(int[] nums) {
        int farthest = 0;
        for (int i = 0; i < nums.length; i++) {
            if (i > farthest) return false;
            farthest = Math.max(farthest, i + nums[i]);
        }
        return true;
    }
}
```

**Complexity:** O(n) time, O(1) space

**Companies:** <code>Other</code>

</details>

---

<details>
<summary><strong>Climbing Stairs</strong> - <code>Dynamic Programming</code></summary>

### Problem Statement

You're climbing a staircase of `n` steps. Each time you can climb 1 or 2 steps. In how many distinct ways can you reach the top?

**Example:**
```
Input: n = 4
Output: 5   // (1+1+1+1, 1+1+2, 1+2+1, 2+1+1, 2+2)
```

### Approach: DP — Same Pattern as Fibonacci

Ways to reach step `n` = ways to reach `n-1` + ways to reach `n-2`.

```java
class Solution {
    public int climbStairs(int n) {
        if (n <= 2) return n;
        int prev2 = 1, prev1 = 2;
        for (int i = 3; i <= n; i++) {
            int curr = prev1 + prev2;
            prev2 = prev1;
            prev1 = curr;
        }
        return prev1;
    }
}
```

**Complexity:** O(n) time, O(1) space

**Companies:** <code>Other</code>

</details>

---

<details>
<summary><strong>Sliding Window Maximum</strong> - <code>Deque</code> | <code>Array</code></summary>

### Problem Statement

Given an array `nums` and window size `k`, return the maximum of each sliding window as it moves left to right.

**Example:**
```
Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
```

### Approach: Monotonic Deque (indices, decreasing values)

Keep indices in the deque whose values are in decreasing order. The front of the deque is always the max for the current window.

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        Deque<Integer> deque = new ArrayDeque<>(); // stores indices
        int[] result = new int[nums.length - k + 1];
        int ri = 0;

        for (int i = 0; i < nums.length; i++) {
            if (!deque.isEmpty() && deque.peekFirst() < i - k + 1) {
                deque.pollFirst();
            }
            while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) {
                deque.pollLast();
            }
            deque.offerLast(i);
            if (i >= k - 1) {
                result[ri++] = nums[deque.peekFirst()];
            }
        }
        return result;
    }
}
```

**Complexity:** O(n) time, O(k) space — each index enters/leaves the deque once

**Companies:** <code>Other</code>

</details>

---

<details>
<summary><strong>Top K Largest Sum Subarrays</strong> - <code>Array</code> | <code>Heap</code></summary>

### Problem Statement

Given an array `nums` and an integer `k`, find the sums of the top `k` contiguous subarrays with the largest sums.

**Example:**
```
Input: nums = [1,2,3], k = 2
Output: [6,5]   // [1,2,3]=6, [2,3]=5
```

### Approach: Min-Heap of Size K

Generate subarray sums and keep only the top `k`, using a min-heap — pop the smallest whenever the heap grows beyond `k`.

```java
class Solution {
    public List<Integer> topKSumSubarrays(int[] nums, int k) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        for (int i = 0; i < nums.length; i++) {
            int sum = 0;
            for (int j = i; j < nums.length; j++) {
                sum += nums[j];
                minHeap.offer(sum);
                if (minHeap.size() > k) minHeap.poll();
            }
        }
        List<Integer> result = new ArrayList<>(minHeap);
        result.sort(Collections.reverseOrder());
        return result;
    }
}
```

**Complexity:** O(n² log k) time, O(k) space — fine to discuss as-is; mention prefix sums as the optimized follow-up if pushed

**Companies:** <code>Other</code>

</details>

---

<details>
<summary><strong>Find the Kth Most Frequent Element</strong> - <code>HashMap</code> | <code>Heap</code></summary>

### Problem Statement

Given an array (or a table of error logs), find the element with the `k`-th highest frequency — e.g., the 3rd most frequent error type.

**Example:**
```
Input: nums = [1,1,1,2,2,3,3,3,3,4], k = 3
Output: 2   // freq: 3→4, 1→3, 2→2, 4→1 → 3rd highest is 2
```

### Approach: Count with HashMap, then Min-Heap of Size K

```java
class Solution {
    public int kthMostFrequent(int[] nums, int k) {
        Map<Integer, Integer> freq = new HashMap<>();
        for (int num : nums) freq.merge(num, 1, Integer::sum);

        PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> a[1] - b[1]);
        for (var entry : freq.entrySet()) {
            minHeap.offer(new int[]{entry.getKey(), entry.getValue()});
            if (minHeap.size() > k) minHeap.poll();
        }
        return minHeap.peek()[0];
    }
}
```

**If asked as SQL** (e.g., "3rd most frequent error in the logs table"):
```sql
SELECT error_type, COUNT(*) AS freq
FROM error_logs
GROUP BY error_type
ORDER BY freq DESC
LIMIT 1 OFFSET 2;   -- 3rd most frequent (0-indexed offset)
```

**Complexity:** O(n log k) time, O(n) space

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Extract Integers from an Array of Strings</strong> - <code>String</code> | <code>Array</code></summary>

**Problem:** Array has a mix of numeric and non-numeric strings. Return only the ones that are integers, as numbers.

```
Input: ["1", "two", "3", "four", "5"]
Output: [1, 3, 5]
```

**Approach:** Filter with a regex, then convert.

```javascript
function extractIntegers(arr) {
    return arr.filter(s => /^-?\d+$/.test(s)).map(Number);
}
extractIntegers(["1", "two", "3", "four", "5"]); // [1, 3, 5]
```

**Companies:** <code>EPAM</code>

</details>

---

<details>
<summary><strong>Find Missing Number in 1 to N</strong> - <code>Array</code> | <code>Math</code></summary>

**Problem:** Array has numbers 1 to n with one missing. Find it.

```
Input: nums = [1,2,4,5], n = 5
Output: 3
```

**Approach:** Expected sum `n*(n+1)/2` minus actual sum.

```java
int findMissing(int[] nums, int n) {
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : nums) actualSum += num;
    return expectedSum - actualSum;
}
```

**Companies:** <code>EPAM</code>

</details>

---

<details>
<summary><strong>Longest Substring Without Repeating Characters</strong> - <code>String</code> | <code>Sliding Window</code></summary>

**Problem:** Find the length of the longest substring with no repeating characters.

```
Input: s = "abcncabca"
Output: 4   // "abcn" or "ncab"
```

**Approach:** Sliding window, track last-seen index of each char; shrink window when a repeat falls inside it.

```java
int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int start = 0, maxLen = 0;
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= start) {
            start = lastSeen.get(c) + 1;
        }
        lastSeen.put(c, i);
        maxLen = Math.max(maxLen, i - start + 1);
    }
    return maxLen;
}
```

**Companies:** <code>EPAM</code>

</details>

---

<details>
<summary><strong>3Sum Closest to Target</strong> - <code>Array</code> | <code>Two Pointers</code></summary>

**Problem:** Find the sum of 3 elements closest to a target value.

```
Input: nums = [-1,2,1,-4], target = 1
Output: 2   // (-1 + 2 + 1)
```

**Approach:** Sort, then fix one element and two-pointer the rest — same skeleton as 3Sum, but track closest diff instead of exact-zero.

```java
int threeSumClosest(int[] nums, int target) {
    Arrays.sort(nums);
    int closestSum = nums[0] + nums[1] + nums[2];
    for (int i = 0; i < nums.length - 2; i++) {
        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (Math.abs(sum - target) < Math.abs(closestSum - target)) {
                closestSum = sum;
            }
            if (sum < target) left++;
            else if (sum > target) right--;
            else return sum; // exact match, can't get closer
        }
    }
    return closestSum;
}
```

**Companies:** <code>EPAM</code>

</details>
