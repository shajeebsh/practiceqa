---
title: "challenge1-c#"
date: 2026-05-02 22:48:10 
author: shajeebhameed
layout: page
slug: challenge1-c
status: publish
---

## C# Solutions for LeetCode-Style Easy Problems

## 25 Easy C# Coding Challenges with Solutions

### 1\. Two Sum (Easy)

#### Problem Statement

Given an array of integers `nums` and an integer `target`, return indices of the two numbers that add up to `target`. You may assume exactly one solution, and you may not use the same element twice.

#### Synthetic Test Data
    
    
    // Example 1
    nums = [2,7,11,15], target = 9 → [0,1]
    // Example 2
    nums = [3,2,4], target = 6 → [1,2]
    // Edge case
    nums = [3,3], target = 6 → [0,1]

#### C# Solution
    
    
    using System;
    using System.Collections.Generic;
    
    public class Solution {
        public int[] TwoSum(int[] nums, int target) {
            Dictionary<int, int> seen = new Dictionary<int, int>();
    
            for (int i = 0; i < nums.Length; i++) {
                int complement = target - nums[i];
    
                if (seen.ContainsKey(complement)) {
                    return new int[] { seen[complement], i };
                }
    
                seen[nums[i]] = i;
            }
    
            return new int[0]; // Should never reach here
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the array  
**Space Complexity:** O(n) - hash map stores up to n elements

* * *

### 2\. Valid Palindrome (Easy)

#### Problem Statement

A phrase is a palindrome if, after converting all uppercase letters to lowercase and removing all non-alphanumeric characters, it reads the same forward and backward. Return true if the string is a valid palindrome.

#### Synthetic Test Data
    
    
    // Example 1
    s = "A man, a plan, a canal: Panama" → true
    // Example 2
    s = "race a car" → false
    // Edge case
    s = " " → true

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public bool IsPalindrome(string s) {
            int left = 0;
            int right = s.Length - 1;
    
            while (left < right) {
                // Skip non-alphanumeric characters
                while (left < right && !char.IsLetterOrDigit(s[left])) {
                    left++;
                }
                while (left < right && !char.IsLetterOrDigit(s[right])) {
                    right--;
                }
    
                // Compare characters (case-insensitive)
                if (char.ToLower(s[left]) != char.ToLower(s[right])) {
                    return false;
                }
    
                left++;
                right--;
            }
    
            return true;
        }
    }

#### Explanation

**Time Complexity:** O(n) - each character visited at most twice  
**Space Complexity:** O(1) - constant extra space

* * *

### 3\. Valid Parentheses (Easy)

#### Problem Statement

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['`, and `']'`, determine if the input string is valid.

#### Synthetic Test Data
    
    
    // Example 1
    s = "()" → true
    // Example 2
    s = "()[]{}" → true
    // Example 3
    s = "(]" → false
    // Edge case
    s = "([)]" → false

#### C# Solution
    
    
    using System;
    using System.Collections.Generic;
    
    public class Solution {
        public bool IsValid(string s) {
            Stack<char> stack = new Stack<char>();
            Dictionary<char, char> matching = new Dictionary<char, char> {
                {')', '('},
                {']', '['},
                {'}', '{'}
            };
    
            foreach (char c in s) {
                if (matching.ContainsKey(c)) {
                    // Closing bracket
                    if (stack.Count == 0 || stack.Pop() != matching[c]) {
                        return false;
                    }
                } else {
                    // Opening bracket
                    stack.Push(c);
                }
            }
    
            return stack.Count == 0;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the string  
**Space Complexity:** O(n) - stack can hold up to n characters

* * *

### 4\. Merge Sorted Array (Easy)

#### Problem Statement

You are given two integer arrays `nums1` and `nums2`, sorted in non-decreasing order, and two integers `m` and `n`. Merge `nums2` into `nums1` in-place.

#### Synthetic Test Data
    
    
    // Example 1
    nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3 → [1,2,2,3,5,6]
    // Example 2
    nums1 = [1], m = 1, nums2 = [], n = 0 → [1]

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public void Merge(int[] nums1, int m, int[] nums2, int n) {
            int p1 = m - 1;
            int p2 = n - 1;
            int p = m + n - 1;
    
            while (p2 >= 0) {
                if (p1 >= 0 && nums1[p1] > nums2[p2]) {
                    nums1[p] = nums1[p1];
                    p1--;
                } else {
                    nums1[p] = nums2[p2];
                    p2--;
                }
                p--;
            }
        }
    }

#### Explanation

**Time Complexity:** O(m + n) - traverse both arrays once  
**Space Complexity:** O(1) - in-place merge

* * *

### 5\. Best Time to Buy and Sell Stock (Easy)

#### Problem Statement

You are given an array `prices` where `prices[i]` is the price of a given stock on the ith day. Find the maximum profit you can achieve from one transaction.

#### Synthetic Test Data
    
    
    // Example 1
    prices = [7,1,5,3,6,4] → 5
    // Example 2
    prices = [7,6,4,3,1] → 0

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public int MaxProfit(int[] prices) {
            int minPrice = int.MaxValue;
            int maxProfit = 0;
    
            foreach (int price in prices) {
                if (price < minPrice) {
                    minPrice = price;
                } else if (price - minPrice > maxProfit) {
                    maxProfit = price - minPrice;
                }
            }
    
            return maxProfit;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the array  
**Space Complexity:** O(1) - constant extra space

* * *

### 6\. Valid Anagram (Easy)

#### Problem Statement

Given two strings `s` and `t`, return true if `t` is an anagram of `s`, and false otherwise.

#### Synthetic Test Data
    
    
    // Example 1
    s = "anagram", t = "nagaram" → true
    // Example 2
    s = "rat", t = "car" → false

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public bool IsAnagram(string s, string t) {
            if (s.Length != t.Length) return false;
    
            int[] count = new int[26];
    
            foreach (char c in s) {
                count[c - 'a']++;
            }
    
            foreach (char c in t) {
                count[c - 'a']--;
                if (count[c - 'a'] < 0) return false;
            }
    
            return true;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through both strings  
**Space Complexity:** O(1) - fixed-size array

* * *

### 7\. Maximum Subarray (Easy)

#### Problem Statement

Given an integer array `nums`, find the contiguous subarray with the largest sum and return its sum.

#### Synthetic Test Data
    
    
    // Example 1
    nums = [-2,1,-3,4,-1,2,1,-5,4] → 6
    // Example 2
    nums = [1] → 1

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public int MaxSubArray(int[] nums) {
            int currentSum = nums[0];
            int maxSum = nums[0];
    
            for (int i = 1; i < nums.Length; i++) {
                currentSum = Math.Max(nums[i], currentSum + nums[i]);
                maxSum = Math.Max(maxSum, currentSum);
            }
    
            return maxSum;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the array  
**Space Complexity:** O(1) - constant extra space (Kadane's algorithm)

* * *

### 8\. Climbing Stairs (Easy)

#### Problem Statement

You are climbing a staircase. It takes `n` steps to reach the top. Each time you can either climb 1 or 2 steps. Return the number of distinct ways to climb to the top.

#### Synthetic Test Data
    
    
    // Example 1
    n = 2 → 2
    // Example 2
    n = 3 → 3

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public int ClimbStairs(int n) {
            if (n <= 2) return n;
    
            int prev1 = 2;
            int prev2 = 1;
            int current = 0;
    
            for (int i = 3; i <= n; i++) {
                current = prev1 + prev2;
                prev2 = prev1;
                prev1 = current;
            }
    
            return prev1;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single loop  
**Space Complexity:** O(1) - constant extra space (dynamic programming)

* * *

### 9\. Reverse Linked List (Easy)

#### Problem Statement

Given the head of a singly linked list, reverse the list and return the reversed list.

#### Synthetic Test Data
    
    
    // Example
    head = [1,2,3,4,5] → [5,4,3,2,1]

#### C# Solution
    
    
    using System;
    
    public class ListNode {
        public int val;
        public ListNode next;
        public ListNode(int val=0, ListNode next=null) {
            this.val = val;
            this.next = next;
        }
    }
    
    public class Solution {
        public ListNode ReverseList(ListNode head) {
            ListNode prev = null;
            ListNode current = head;
    
            while (current != null) {
                ListNode nextTemp = current.next;
                current.next = prev;
                prev = current;
                current = nextTemp;
            }
    
            return prev;
        }
    }

#### Explanation

**Time Complexity:** O(n) - traverse the list once  
**Space Complexity:** O(1) - iterative approach

* * *

### 10\. Palindrome Linked List (Easy)

#### Problem Statement

Given the head of a singly linked list, return true if it is a palindrome.

#### Synthetic Test Data
    
    
    // Example 1
    head = [1,2,2,1] → true
    // Example 2
    head = [1,2] → false

#### C# Solution
    
    
    using System;
    using System.Collections.Generic;
    
    public class Solution {
        public bool IsPalindrome(ListNode head) {
            List<int> values = new List<int>();
            ListNode current = head;
    
            while (current != null) {
                values.Add(current.val);
                current = current.next;
            }
    
            int left = 0;
            int right = values.Count - 1;
    
            while (left < right) {
                if (values[left] != values[right]) {
                    return false;
                }
                left++;
                right--;
            }
    
            return true;
        }
    }

#### Explanation

**Time Complexity:** O(n) - traverse the list once  
**Space Complexity:** O(n) - store values in list

* * *

### 11\. Binary Tree Inorder Traversal (Easy)

#### Problem Statement

Given the root of a binary tree, return the inorder traversal of its nodes' values.

#### Synthetic Test Data
    
    
    // Example
    root = [1,null,2,3] → [1,3,2]

#### C# Solution
    
    
    using System;
    using System.Collections.Generic;
    
    public class TreeNode {
        public int val;
        public TreeNode left;
        public TreeNode right;
        public TreeNode(int val=0, TreeNode left=null, TreeNode right=null) {
            this.val = val;
            this.left = left;
            this.right = right;
        }
    }
    
    public class Solution {
        public IList<int> InorderTraversal(TreeNode root) {
            List<int> result = new List<int>();
            Stack<TreeNode> stack = new Stack<TreeNode>();
            TreeNode current = root;
    
            while (current != null || stack.Count > 0) {
                while (current != null) {
                    stack.Push(current);
                    current = current.left;
                }
    
                current = stack.Pop();
                result.Add(current.val);
                current = current.right;
            }
    
            return result;
        }
    }

#### Explanation

**Time Complexity:** O(n) - visit each node once  
**Space Complexity:** O(n) - for the stack

* * *

### 12\. Maximum Depth of Binary Tree (Easy)

#### Problem Statement

Given the root of a binary tree, return its maximum depth.

#### Synthetic Test Data
    
    
    // Example
    root = [3,9,20,null,null,15,7] → 3

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public int MaxDepth(TreeNode root) {
            if (root == null) return 0;
    
            int leftDepth = MaxDepth(root.left);
            int rightDepth = MaxDepth(root.right);
    
            return Math.Max(leftDepth, rightDepth) + 1;
        }
    }

#### Explanation

**Time Complexity:** O(n) - visit each node once  
**Space Complexity:** O(h) - recursion stack where h is height

* * *

### 13\. Contains Duplicate (Easy)

#### Problem Statement

Given an integer array `nums`, return true if any value appears at least twice in the array, and false if every element is distinct.

#### Synthetic Test Data
    
    
    // Example 1
    nums = [1,2,3,1] → true
    // Example 2
    nums = [1,2,3,4] → false

#### C# Solution
    
    
    using System;
    using System.Collections.Generic;
    
    public class Solution {
        public bool ContainsDuplicate(int[] nums) {
            HashSet<int> seen = new HashSet<int>();
    
            foreach (int num in nums) {
                if (seen.Contains(num)) {
                    return true;
                }
                seen.Add(num);
            }
    
            return false;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the array  
**Space Complexity:** O(n) - hash set stores up to n elements

* * *

### 14\. Missing Number (Easy)

#### Problem Statement

Given an array `nums` containing `n` distinct numbers in the range `[0, n]`, return the only number in the range that is missing from the array.

#### Synthetic Test Data
    
    
    // Example 1
    nums = [3,0,1] → 2
    // Example 2
    nums = [0,1] → 2

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public int MissingNumber(int[] nums) {
            int n = nums.Length;
            int expectedSum = n * (n + 1) / 2;
            int actualSum = 0;
    
            foreach (int num in nums) {
                actualSum += num;
            }
    
            return expectedSum - actualSum;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the array  
**Space Complexity:** O(1) - constant extra space

* * *

### 15\. Single Number (Easy)

#### Problem Statement

Given a non-empty array of integers `nums`, every element appears twice except for one. Find that single one.

#### Synthetic Test Data
    
    
    // Example 1
    nums = [2,2,1] → 1
    // Example 2
    nums = [4,1,2,1,2] → 4

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public int SingleNumber(int[] nums) {
            int result = 0;
    
            foreach (int num in nums) {
                result ^= num; // XOR operation
            }
    
            return result;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the array  
**Space Complexity:** O(1) - constant extra space (XOR trick)

* * *

### 16\. Fizz Buzz (Easy)

#### Problem Statement

Given an integer `n`, return a string array where for each `i` from 1 to `n`: answer[i] is "FizzBuzz" if divisible by 3 and 5, "Fizz" if divisible by 3, "Buzz" if divisible by 5, or the number as a string otherwise.

#### Synthetic Test Data
    
    
    // Example
    n = 3 → ["1","2","Fizz"]
    n = 5 → ["1","2","Fizz","4","Buzz"]

#### C# Solution
    
    
    using System;
    using System.Collections.Generic;
    
    public class Solution {
        public IList<string> FizzBuzz(int n) {
            List<string> result = new List<string>();
    
            for (int i = 1; i <= n; i++) {
                if (i % 3 == 0 && i % 5 == 0) {
                    result.Add("FizzBuzz");
                } else if (i % 3 == 0) {
                    result.Add("Fizz");
                } else if (i % 5 == 0) {
                    result.Add("Buzz");
                } else {
                    result.Add(i.ToString());
                }
            }
    
            return result;
        }
    }

#### Explanation

**Time Complexity:** O(n) - iterate from 1 to n  
**Space Complexity:** O(n) - for the result list

* * *

### 17\. Reverse String (Easy)

#### Problem Statement

Write a function that reverses a string. The input string is given as an array of characters `s`. Do it in-place.

#### Synthetic Test Data
    
    
    // Example
    s = ['h','e','l','l','o'] → ['o','l','l','e','h']

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public void ReverseString(char[] s) {
            int left = 0;
            int right = s.Length - 1;
    
            while (left < right) {
                char temp = s[left];
                s[left] = s[right];
                s[right] = temp;
                left++;
                right--;
            }
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass with two pointers  
**Space Complexity:** O(1) - in-place reversal

* * *

### 18\. Power of Two (Easy)

#### Problem Statement

Given an integer `n`, return true if it is a power of two. Otherwise, return false.

#### Synthetic Test Data
    
    
    // Example 1
    n = 1 → true
    // Example 2
    n = 16 → true
    // Example 3
    n = 3 → false

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public bool IsPowerOfTwo(int n) {
            if (n <= 0) return false;
            return (n & (n - 1)) == 0;
        }
    }

#### Explanation

**Time Complexity:** O(1) - constant time operation  
**Space Complexity:** O(1) - constant extra space

* * *

### 19\. Roman to Integer (Easy)

#### Problem Statement

Convert a Roman numeral to an integer. Roman numerals are represented by seven different symbols: I, V, X, L, C, D, M.

#### Synthetic Test Data
    
    
    // Example 1
    s = "III" → 3
    // Example 2
    s = "LVIII" → 58
    // Example 3
    s = "MCMXCIV" → 1994

#### C# Solution
    
    
    using System;
    using System.Collections.Generic;
    
    public class Solution {
        public int RomanToInt(string s) {
            Dictionary<char, int> romanMap = new Dictionary<char, int> {
                {'I', 1}, {'V', 5}, {'X', 10}, {'L', 50},
                {'C', 100}, {'D', 500}, {'M', 1000}
            };
    
            int result = 0;
    
            for (int i = 0; i < s.Length; i++) {
                int current = romanMap[s[i]];
                int next = (i + 1 < s.Length) ? romanMap[s[i + 1]] : 0;
    
                if (current < next) {
                    result -= current;
                } else {
                    result += current;
                }
            }
    
            return result;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the string  
**Space Complexity:** O(1) - constant extra space

* * *

### 20\. Longest Common Prefix (Easy)

#### Problem Statement

Write a function to find the longest common prefix string amongst an array of strings. If there is no common prefix, return an empty string.

#### Synthetic Test Data
    
    
    // Example 1
    strs = ["flower","flow","flight"] → "fl"
    // Example 2
    strs = ["dog","racecar","car"] → ""

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public string LongestCommonPrefix(string[] strs) {
            if (strs.Length == 0) return "";
    
            string prefix = strs[0];
    
            for (int i = 1; i < strs.Length; i++) {
                while (strs[i].IndexOf(prefix) != 0) {
                    prefix = prefix.Substring(0, prefix.Length - 1);
                    if (prefix == "") return "";
                }
            }
    
            return prefix;
        }
    }

#### Explanation

**Time Complexity:** O(n * m) where n is number of strings, m is length of prefix  
**Space Complexity:** O(1) - constant extra space

* * *

### 21\. Remove Duplicates from Sorted Array (Easy)

#### Problem Statement

Given a sorted array `nums`, remove the duplicates in-place such that each element appears only once and return the new length.

#### Synthetic Test Data
    
    
    // Example
    nums = [1,1,2] → 2 (nums becomes [1,2,_])

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public int RemoveDuplicates(int[] nums) {
            if (nums.Length == 0) return 0;
    
            int uniqueIndex = 0;
    
            for (int i = 1; i < nums.Length; i++) {
                if (nums[i] != nums[uniqueIndex]) {
                    uniqueIndex++;
                    nums[uniqueIndex] = nums[i];
                }
            }
    
            return uniqueIndex + 1;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the array  
**Space Complexity:** O(1) - in-place modification

* * *

### 22\. Plus One (Easy)

#### Problem Statement

You are given a large integer represented as an integer array `digits`. Increment the integer by one and return the resulting array of digits.

#### Synthetic Test Data
    
    
    // Example 1
    digits = [1,2,3] → [1,2,4]
    // Example 2
    digits = [4,3,2,1] → [4,3,2,2]
    // Example 3
    digits = [9] → [1,0]

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public int[] PlusOne(int[] digits) {
            for (int i = digits.Length - 1; i >= 0; i--) {
                if (digits[i] < 9) {
                    digits[i]++;
                    return digits;
                }
                digits[i] = 0;
            }
    
            // All digits were 9
            int[] result = new int[digits.Length + 1];
            result[0] = 1;
            return result;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass from right to left  
**Space Complexity:** O(n) - when creating new array for overflow case

* * *

### 23\. Pascal's Triangle (Easy)

#### Problem Statement

Given an integer `numRows`, return the first numRows of Pascal's triangle.

#### Synthetic Test Data
    
    
    // Example
    numRows = 5 → [[1],[1,1],[1,2,1],[1,3,3,1],[1,4,6,4,1]]

#### C# Solution
    
    
    using System;
    using System.Collections.Generic;
    
    public class Solution {
        public IList<IList<int>> Generate(int numRows) {
            IList<IList<int>> triangle = new List<IList<int>>();
    
            if (numRows == 0) return triangle;
    
            triangle.Add(new List<int> { 1 });
    
            for (int i = 1; i < numRows; i++) {
                IList<int> prevRow = triangle[i - 1];
                List<int> newRow = new List<int> { 1 };
    
                for (int j = 1; j < i; j++) {
                    newRow.Add(prevRow[j - 1] + prevRow[j]);
                }
    
                newRow.Add(1);
                triangle.Add(newRow);
            }
    
            return triangle;
        }
    }

#### Explanation

**Time Complexity:** O(numRows²) - building each row  
**Space Complexity:** O(numRows²) - storing the triangle

* * *

### 24\. Majority Element (Easy)

#### Problem Statement

Given an array `nums` of size n, return the majority element (the element that appears more than ⌊n/2⌋ times).

#### Synthetic Test Data
    
    
    // Example 1
    nums = [3,2,3] → 3
    // Example 2
    nums = [2,2,1,1,1,2,2] → 2

#### C# Solution
    
    
    using System;
    
    public class Solution {
        public int MajorityElement(int[] nums) {
            int candidate = nums[0];
            int count = 1;
    
            for (int i = 1; i < nums.Length; i++) {
                if (count == 0) {
                    candidate = nums[i];
                    count = 1;
                } else if (nums[i] == candidate) {
                    count++;
                } else {
                    count--;
                }
            }
    
            return candidate;
        }
    }

#### Explanation

**Time Complexity:** O(n) - single pass through the array  
**Space Complexity:** O(1) - constant extra space (Boyer-Moore Voting Algorithm)

* * *

### 25\. Convert Sorted Array to Binary Search Tree (Easy)

#### Problem Statement

Given an integer array `nums` where the elements are sorted in ascending order, convert it to a height-balanced binary search tree.

#### Synthetic Test Data
    
    
    // Example
    nums = [-10,-3,0,5,9] → [0,-3,9,-10,null,5] (or other valid BST)

#### C# Solution
    
    
    using System;
    
    public class TreeNode {
        public int val;
        public TreeNode left;
        public TreeNode right;
        public TreeNode(int val=0, TreeNode left=null, TreeNode right=null) {
            this.val = val;
            this.left = left;
            this.right = right;
        }
    }
    
    public class Solution {
        public TreeNode SortedArrayToBST(int[] nums) {
            return BuildBST(nums, 0, nums.Length - 1);
        }
    
        private TreeNode BuildBST(int[] nums, int left, int right) {
            if (left > right) return null;
    
            int mid = left + (right - left) / 2;
            TreeNode node = new TreeNode(nums[mid]);
    
            node.left = BuildBST(nums, left, mid - 1);
            node.right = BuildBST(nums, mid + 1, right);
    
            return node;
        }
    }

#### Explanation

**Time Complexity:** O(n) - visit each element once  
**Space Complexity:** O(log n) - recursion stack for balanced tree

* * *

## Summary

This collection includes 25 easy C# coding challenges with solutions. Each solution includes:

  * Problem statement
  * Synthetic test data examples
  * Complete C# code with proper syntax
  * Time and space complexity analysis



All solutions are optimized for readability, efficiency, and follow C# best practices including proper use of collections, algorithms, and data structures.
