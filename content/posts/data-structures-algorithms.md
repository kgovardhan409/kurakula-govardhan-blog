---
title: "DSA Interview Questions"
date: 2026-06-16
draft: false
tags: ["DSA", "Array", "Interview"]
categories: ["Data Structures"]
---

# DSA Interview Questions Repository

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

</details>

---

<details>
<summary><strong>Flatten Object</strong> - <code>Object</code> | <code>JavaScript</code></summary>

### Problem Statement

Given a nested object, flatten it into a single level object where the keys are constructed by concatenating the parent and child keys with an underscore.

**Example 1:**
```javascript
Input:
{
  department: {
    name: "Consumer Experience",
    branch: {
      name: "Bangalore"
    }
  }
}

Output:
{
  department_name: "Consumer Experience",
  department_branch_name: "Bangalore"
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
const nestedObj = {
  department: {
    name: "Consumer Experience",
    branch: {
      name: "Bangalore"
    }
  }
};

const flattened = flattenObject(nestedObj);
console.log(flattened);
// Output: { department_name: "Consumer Experience", department_branch_name: "Bangalore" }
```

### Complexity Analysis

- **Time Complexity:** O(n) - Where n is the total number of key-value pairs in the nested object
- **Space Complexity:** O(n) - For the flattened result object

### Key Points

1. **Recursive approach**: Traverses nested structures depth-first
2. **Key construction**: Concatenates parent and child keys with underscore
3. **Type checking**: Handles objects, primitives, and arrays appropriately
4. **Preserves primitive values**: Non-object values are added directly to result

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

</details>
