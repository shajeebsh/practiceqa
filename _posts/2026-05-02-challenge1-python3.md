---
title: "challenge1-python3"
date: 2026-05-02 16:00:29 
author: shajeebhameed
layout: page
slug: challenge1-python3
status: publish
---

## Python 3 Solutions for LeetCode-Style Challenges

## Easy Problems

### 408\. Valid Word Abbreviation

#### Problem Statement

Given a non-empty string `word` and an abbreviation `abbr`, return whether the abbreviation matches the word. A valid abbreviation consists of lowercase letters and numbers representing how many characters to skip. Numbers cannot have leading zeros (except zero itself).

#### Synthetic Test Data
    
    
    # Example 1
    word = "internationalization", abbr = "i12iz4n" → True
    # Example 2
    word = "apple", abbr = "a2e" → False
    # Edge case
    word = "a", abbr = "01" → False  # leading zero

#### Python 3 Solution
    
    
    def validWordAbbreviation(word: str, abbr: str) -> bool:
        i = j = 0
        m, n = len(word), len(abbr)
    
        while i < m and j < n:
            if abbr[j].isdigit():
                # Handle leading zero
                if abbr[j] == '0':
                    return False
    
                # Parse the full number
                num = 0
                while j < n and abbr[j].isdigit():
                    num = num * 10 + int(abbr[j])
                    j += 1
    
                # Skip 'num' characters in word
                i += num
            else:
                # Check character match
                if word[i] != abbr[j]:
                    return False
                i += 1
                j += 1
    
        # Both pointers should reach the end simultaneously
        return i == m and j == n

#### Explanation

**Time Complexity:** O(m + n) - traverse both strings once  
**Space Complexity:** O(1) - constant extra space

We use two pointers to traverse both strings simultaneously. When encountering a number in the abbreviation, we parse the entire number (handling leading zeros), then advance the word pointer by that amount. For letters, we check for exact matches. The process succeeds only if both pointers reach their respective ends at the same time.

* * *

### 680\. Valid Palindrome II

#### Problem Statement

Given a string `s`, return `true` if it can be a palindrome after deleting at most one character from it.

#### Synthetic Test Data
    
    
    # Example 1
    s = "aba" → True
    # Example 2
    s = "abca" → True  # delete 'b' or 'c'
    # Edge case
    s = "abc" → False

#### Python 3 Solution
    
    
    def validPalindrome(s: str) -> bool:
        def is_palindrome_range(left: int, right: int) -> bool:
            """Check if substring s[left:right+1] is palindrome"""
            while left < right:
                if s[left] != s[right]:
                    return False
                left += 1
                right -= 1
            return True
    
        left, right = 0, len(s) - 1
    
        while left < right:
            if s[left] != s[right]:
                # Try skipping left character or right character
                return is_palindrome_range(left + 1, right) or is_palindrome_range(left, right - 1)
            left += 1
            right -= 1
    
        return True

#### Explanation

**Time Complexity:** O(n) - at most two O(n) palindrome checks  
**Space Complexity:** O(1) - constant extra space

The algorithm uses two pointers from both ends. When a mismatch occurs, we have two options: skip the left character or skip the right character. We check if either resulting substring is a palindrome. This works because at most one deletion is allowed.

* * *

### 88\. Merge Sorted Array

#### Problem Statement

You are given two integer arrays `nums1` and `nums2`, sorted in non-decreasing order, and two integers `m` and `n` representing the number of elements in `nums1` and `nums2` respectively. Merge `nums2` into `nums1` in-place such that the result is sorted non-decreasingly.

#### Synthetic Test Data
    
    
    # Example 1
    nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3 → [1,2,2,3,5,6]
    # Example 2
    nums1 = [1], m = 1, nums2 = [], n = 0 → [1]
    # Edge case
    nums1 = [0], m = 0, nums2 = [1], n = 1 → [1]

#### Python 3 Solution
    
    
    def merge(nums1: list[int], m: int, nums2: list[int], n: int) -> None:
        """
        Do not return anything, modify nums1 in-place instead.
        """
        # Start from the end to avoid overwriting elements
        p1, p2, p = m - 1, n - 1, m + n - 1
    
        # Merge from largest to smallest
        while p2 >= 0:
            if p1 >= 0 and nums1[p1] > nums2[p2]:
                nums1[p] = nums1[p1]
                p1 -= 1
            else:
                nums1[p] = nums2[p2]
                p2 -= 1
            p -= 1

#### Explanation

**Time Complexity:** O(m + n) - traverse both arrays once  
**Space Complexity:** O(1) - in-place merge

This approach uses three pointers: one for the end of the valid elements in `nums1`, one for the end of `nums2`, and one for the end of the merged array. By merging from the end, we avoid overwriting elements that haven't been processed yet.

* * *

### 938\. Range Sum of BST

#### Problem Statement

Given the root node of a binary search tree and two integers `low` and `high`, return the sum of values of all nodes with values between `low` and `high` inclusive.

#### Synthetic Test Data
    
    
    # Example 1
    # BST: [10,5,15,3,7,null,18], low=7, high=15 → 32 (10+7+15)
    # Example 2
    # BST: [10,5,15,3,7,13,18,1,null,6], low=6, high=10 → 23 (10+6+7)

#### Python 3 Solution
    
    
    class TreeNode:
        def __init__(self, val=0, left=None, right=None):
            self.val = val
            self.left = left
            self.right = right
    
    def rangeSumBST(root: TreeNode, low: int, high: int) -> int:
        if not root:
            return 0
    
        total = 0
    
        # If current node is within range, add its value
        if low <= root.val <= high:
            total += root.val
    
        # Explore left subtree if it could contain values >= low
        if root.val > low:
            total += rangeSumBST(root.left, low, high)
    
        # Explore right subtree if it could contain values <= high
        if root.val < high:
            total += rangeSumBST(root.right, low, high)
    
        return total

#### Explanation

**Time Complexity:** O(n) in worst case, but O(h + k) where h is height and k is number of nodes in range  
**Space Complexity:** O(h) for recursion stack

We leverage the BST property: if the current node's value is greater than `low`, we need to check the left subtree; if it's less than `high`, we need to check the right subtree. This prunes unnecessary traversals.

* * *

### 1\. Two Sum

#### Problem Statement

Given an array of integers `nums` and an integer `target`, return indices of the two numbers that add up to `target`. You may assume exactly one solution, and you may not use the same element twice.

#### Synthetic Test Data
    
    
    # Example 1
    nums = [2,7,11,15], target = 9 → [0,1]
    # Example 2
    nums = [3,2,4], target = 6 → [1,2]
    # Edge case
    nums = [3,3], target = 6 → [0,1]

#### Python 3 Solution
    
    
    def twoSum(nums: list[int], target: int) -> list[int]:
        seen = {}  # value -> index
    
        for i, num in enumerate(nums):
            complement = target - num
    
            if complement in seen:
                return [seen[complement], i]
    
            seen[num] = i
    
        return []  # Should never reach here given problem constraints

#### Explanation

**Time Complexity:** O(n) - single pass through the array  
**Space Complexity:** O(n) - hash map stores up to n elements

As we iterate through the array, we store each number and its index in a hash map. For each element, we check if its complement (target - current) exists in the map. This solves the problem in a single pass.

* * *

### 125\. Valid Palindrome

#### Problem Statement

A phrase is a palindrome if, after converting all uppercase letters to lowercase and removing all non-alphanumeric characters, it reads the same forward and backward. Return true if the string is a valid palindrome.

#### Synthetic Test Data
    
    
    # Example 1
    s = "A man, a plan, a canal: Panama" → True
    # Example 2
    s = "race a car" → False
    # Edge case
    s = " " → True

#### Python 3 Solution
    
    
    def isPalindrome(s: str) -> bool:
        left, right = 0, len(s) - 1
    
        while left < right:
            # Skip non-alphanumeric characters
            while left < right and not s[left].isalnum():
                left += 1
            while left < right and not s[right].isalnum():
                right -= 1
    
            # Compare characters (case-insensitive)
            if s[left].lower() != s[right].lower():
                return False
    
            left += 1
            right -= 1
    
        return True

#### Explanation

**Time Complexity:** O(n) - each character visited at most twice  
**Space Complexity:** O(1) - constant extra space

Using two pointers from both ends, we skip non-alphanumeric characters and compare the alphanumeric characters in a case-insensitive manner.

* * *

### 283\. Move Zeroes

#### Problem Statement

Given an integer array `nums`, move all zeroes to the end while maintaining the relative order of non-zero elements. Do this in-place without making a copy of the array.

#### Synthetic Test Data
    
    
    # Example 1
    nums = [0,1,0,3,12] → [1,3,12,0,0]
    # Example 2
    nums = [0] → [0]
    # Edge case
    nums = [0,0,1] → [1,0,0]

#### Python 3 Solution
    
    
    def moveZeroes(nums: list[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        last_non_zero = 0
    
        # Move all non-zero elements to the front
        for i in range(len(nums)):
            if nums[i] != 0:
                nums[last_non_zero] = nums[i]
                last_non_zero += 1
    
        # Fill the remaining positions with zeroes
        for i in range(last_non_zero, len(nums)):
            nums[i] = 0

#### Explanation

**Time Complexity:** O(n) - two passes through the array  
**Space Complexity:** O(1) - in-place modification

First, we overwrite the array with all non-zero elements in order using a pointer. Then, we fill the remaining positions with zeroes.

* * *

### 543\. Diameter of Binary Tree

#### Problem Statement

Given the root of a binary tree, return the length of the diameter of the tree. The diameter is the longest path between any two nodes, measured by the number of edges.

#### Synthetic Test Data
    
    
    # Example 1
    # Tree: [1,2,3,4,5] → 3 (path: 4-2-1-3 or 5-2-1-3)
    # Example 2
    # Tree: [1,2] → 1

#### Python 3 Solution
    
    
    class TreeNode:
        def __init__(self, val=0, left=None, right=None):
            self.val = val
            self.left = left
            self.right = right
    
    def diameterOfBinaryTree(root: TreeNode) -> int:
        diameter = 0
    
        def depth(node: TreeNode) -> int:
            nonlocal diameter
            if not node:
                return 0
    
            left_depth = depth(node.left)
            right_depth = depth(node.right)
    
            # Update diameter with path through current node
            diameter = max(diameter, left_depth + right_depth)
    
            # Return depth of current subtree
            return max(left_depth, right_depth) + 1
    
        depth(root)
        return diameter

#### Explanation

**Time Complexity:** O(n) - visit each node once  
**Space Complexity:** O(h) - recursion stack where h is tree height

We compute the depth of each node while simultaneously tracking the maximum path length that goes through that node. The diameter is updated as `left_depth + right_depth` at each node.

* * *

### 1768\. Merge Strings Alternately

#### Problem Statement

You are given two strings `word1` and `word2`. Merge the strings by adding letters in alternating order, starting with `word1`. If one string is longer, append the remaining letters at the end.

#### Synthetic Test Data
    
    
    # Example 1
    word1 = "abc", word2 = "pqr" → "apbqcr"
    # Example 2
    word1 = "ab", word2 = "pqrs" → "apbqrs"
    # Edge case
    word1 = "", word2 = "xyz" → "xyz"

#### Python 3 Solution
    
    
    def mergeAlternately(word1: str, word2: str) -> str:
        result = []
        i, j = 0, 0
        len1, len2 = len(word1), len(word2)
    
        while i < len1 and j < len2:
            result.append(word1[i])
            result.append(word2[j])
            i += 1
            j += 1
    
        # Append remaining characters
        if i < len1:
            result.append(word1[i:])
        if j < len2:
            result.append(word2[j:])
    
        return ''.join(result)

#### Explanation

**Time Complexity:** O(m + n) - traverse both strings once  
**Space Complexity:** O(m + n) - for the result string

We use two pointers to iterate through both strings simultaneously, appending characters alternately. After one string is exhausted, we append the remainder of the other string.

* * *

### 20\. Valid Parentheses

#### Problem Statement

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['`, and `']'`, determine if the input string is valid. A valid string must have properly matched and nested brackets.

#### Synthetic Test Data
    
    
    # Example 1
    s = "()" → True
    # Example 2
    s = "()[]{}" → True
    # Example 3
    s = "(]" → False
    # Edge case
    s = "([)]" → False

#### Python 3 Solution
    
    
    def isValid(s: str) -> bool:
        stack = []
        matching = {')': '(', ']': '[', '}': '{'}
    
        for char in s:
            if char in matching:
                # Check if stack is empty or top doesn't match
                if not stack or stack.pop() != matching[char]:
                    return False
            else:
                # Opening bracket
                stack.append(char)
    
        return not stack

#### Explanation

**Time Complexity:** O(n) - single pass through the string  
**Space Complexity:** O(n) - stack can hold up to n characters

We use a stack to keep track of opening brackets. When encountering a closing bracket, we check if it matches the most recent opening bracket.

* * *

## Medium Problems

### 1249\. Minimum Remove to Make Valid Parentheses

#### Problem Statement

Given a string of parentheses and lowercase letters, remove the minimum number of parentheses so that the resulting string is valid. Return the resulting string.

#### Synthetic Test Data
    
    
    # Example 1
    s = "lee(t(c)o)de)" → "lee(t(c)o)de"
    # Example 2
    s = "a)b(c)d" → "ab(c)d"
    # Edge case
    s = "))((" → ""

#### Python 3 Solution
    
    
    def minRemoveToMakeValid(s: str) -> str:
        # Convert to list for mutability
        chars = list(s)
        stack = []
    
        # First pass: mark invalid closing parentheses
        for i, char in enumerate(chars):
            if char == '(':
                stack.append(i)
            elif char == ')':
                if stack:
                    stack.pop()
                else:
                    # Mark for removal
                    chars[i] = ''
    
        # Mark remaining unmatched opening parentheses
        while stack:
            chars[stack.pop()] = ''
    
        # Build result string
        return ''.join(chars)

#### Explanation

**Time Complexity:** O(n) - two passes through the string  
**Space Complexity:** O(n) - for the stack and character list

We use a stack to track indices of unmatched opening parentheses. In the first pass, we mark invalid closing parentheses for removal. In the second pass, we mark remaining unmatched opening parentheses.

* * *

### 215\. Kth Largest Element in an Array

#### Problem Statement

Given an integer array `nums` and an integer `k`, return the kth largest element. Note that it is the kth largest in sorted order, not the kth distinct element.

#### Synthetic Test Data
    
    
    # Example 1
    nums = [3,2,1,5,6,4], k = 2 → 5
    # Example 2
    nums = [3,2,3,1,2,4,5,5,6], k = 4 → 4

#### Python 3 Solution
    
    
    import heapq
    
    def findKthLargest(nums: list[int], k: int) -> int:
        # Use min-heap of size k
        heap = []
    
        for num in nums:
            heapq.heappush(heap, num)
            if len(heap) > k:
                heapq.heappop(heap)
    
        return heap[0]

#### Explanation

**Time Complexity:** O(n log k) - heap operations on size k  
**Space Complexity:** O(k) - for the heap

We maintain a min-heap of size k. By keeping only the k largest elements in the heap, the smallest of these (the root) is the kth largest element.

* * *

### 227\. Basic Calculator II

#### Problem Statement

Implement a basic calculator to evaluate a simple expression string. The expression contains non-negative integers, `+`, `-`, `*`, `/` operators, and spaces. Integer division should truncate toward zero.

#### Synthetic Test Data
    
    
    # Example 1
    s = "3+2*2" → 7
    # Example 2
    s = " 3/2 " → 1
    # Example 3
    s = " 3+5 / 2 " → 5

#### Python 3 Solution
    
    
    def calculate(s: str) -> int:
        if not s:
            return 0
    
        stack = []
        current_num = 0
        last_operator = '+'
        s = s.replace(' ', '')
    
        for i, char in enumerate(s):
            if char.isdigit():
                current_num = current_num * 10 + int(char)
    
            # If current char is operator or last character
            if char in '+-*/' or i == len(s) - 1:
                if last_operator == '+':
                    stack.append(current_num)
                elif last_operator == '-':
                    stack.append(-current_num)
                elif last_operator == '*':
                    stack.append(stack.pop() * current_num)
                elif last_operator == '/':
                    # Integer division truncating toward zero
                    stack.append(int(stack.pop() / current_num))
    
                current_num = 0
                last_operator = char
    
        return sum(stack)

#### Explanation

**Time Complexity:** O(n) - single pass through the string  
**Space Complexity:** O(n) - for the stack

We process the expression by tracking the last operator and current number. Multiplication and division are evaluated immediately by updating the stack, while addition and subtraction are deferred until the end.

* * *

### 56\. Merge Intervals

#### Problem Statement

Given an array of intervals where `intervals[i] = [start_i, end_i]`, merge all overlapping intervals.

#### Synthetic Test Data
    
    
    # Example 1
    intervals = [[1,3],[2,6],[8,10],[15,18]] → [[1,6],[8,10],[15,18]]
    # Example 2
    intervals = [[1,4],[4,5]] → [[1,5]]

#### Python 3 Solution
    
    
    def merge(intervals: list[list[int]]) -> list[list[int]]:
        if not intervals:
            return []
    
        # Sort by start time
        intervals.sort(key=lambda x: x[0])
        merged = [intervals[0]]
    
        for start, end in intervals[1:]:
            last_start, last_end = merged[-1]
    
            # Check for overlap
            if start <= last_end:
                # Merge intervals
                merged[-1] = [last_start, max(last_end, end)]
            else:
                merged.append([start, end])
    
        return merged

#### Explanation

**Time Complexity:** O(n log n) - sorting dominates  
**Space Complexity:** O(n) - for the result list

After sorting intervals by start time, we iterate through them. If the current interval overlaps with the last merged interval, we extend the merged interval. Otherwise, we add it as a new interval.

* * *

### 973\. K Closest Points to Origin

#### Problem Statement

Given an array of points where `points[i] = [x_i, y_i]` and an integer `k`, return the k closest points to the origin (0,0). Return them in any order.

#### Synthetic Test Data
    
    
    # Example 1
    points = [[1,3],[-2,2]], k = 1 → [[-2,2]]
    # Example 2
    points = [[3,3],[5,-1],[-2,4]], k = 2 → [[3,3],[-2,4]]

#### Python 3 Solution
    
    
    import heapq
    
    def kClosest(points: list[list[int]], k: int) -> list[list[int]]:
        # Use max-heap by storing negative distances
        heap = []
    
        for x, y in points:
            dist = -(x*x + y*y)  # Negative for max-heap behavior
            heapq.heappush(heap, (dist, [x, y]))
    
            if len(heap) > k:
                heapq.heappop(heap)
    
        return [point for _, point in heap]

#### Explanation

**Time Complexity:** O(n log k) - heap operations on size k  
**Space Complexity:** O(k) - for the heap

We maintain a max-heap of size k to keep the k closest points. By storing negative distances, Python's min-heap effectively becomes a max-heap.

* * *

### 146\. LRU Cache

#### Problem Statement

Design a data structure that follows the constraints of an LRU (Least Recently Used) cache. Implement `get(key)` and `put(key, value)` functions with O(1) average time complexity.

#### Synthetic Test Data
    
    
    # Example
    lru = LRUCache(2)
    lru.put(1, 1)
    lru.put(2, 2)
    lru.get(1)    # returns 1
    lru.put(3, 3) # evicts key 2
    lru.get(2)    # returns -1

#### Python 3 Solution
    
    
    class Node:
        def __init__(self, key=0, value=0):
            self.key = key
            self.value = value
            self.prev = None
            self.next = None
    
    class LRUCache:
        def __init__(self, capacity: int):
            self.capacity = capacity
            self.cache = {}  # key -> node
            # Dummy head and tail for easy manipulation
            self.head = Node()
            self.tail = Node()
            self.head.next = self.tail
            self.tail.prev = self.head
    
        def _remove_node(self, node: Node) -> None:
            """Remove node from doubly linked list"""
            prev_node = node.prev
            next_node = node.next
            prev_node.next = next_node
            next_node.prev = prev_node
    
        def _add_to_front(self, node: Node) -> None:
            """Add node right after head"""
            node.prev = self.head
            node.next = self.head.next
            self.head.next.prev = node
            self.head.next = node
    
        def _move_to_front(self, node: Node) -> None:
            """Move existing node to front (mark as recently used)"""
            self._remove_node(node)
            self._add_to_front(node)
    
        def _pop_tail(self) -> Node:
            """Remove and return the least recently used node"""
            lru_node = self.tail.prev
            self._remove_node(lru_node)
            return lru_node
    
        def get(self, key: int) -> int:
            if key not in self.cache:
                return -1
    
            node = self.cache[key]
            self._move_to_front(node)
            return node.value
    
        def put(self, key: int, value: int) -> None:
            if key in self.cache:
                node = self.cache[key]
                node.value = value
                self._move_to_front(node)
            else:
                if len(self.cache) >= self.capacity:
                    lru_node = self._pop_tail()
                    del self.cache[lru_node.key]
    
                new_node = Node(key, value)
                self.cache[key] = new_node
                self._add_to_front(new_node)

#### Explanation

**Time Complexity:** O(1) for both get and put operations  
**Space Complexity:** O(capacity) - stores at most capacity nodes

The LRU cache combines a hash map for O(1) lookup with a doubly linked list to maintain order. The head represents the most recently used item, while the tail represents the least recently used.

* * *

### 347\. Top K Frequent Elements

#### Problem Statement

Given an integer array `nums` and an integer `k`, return the k most frequent elements. You may return the answer in any order.

#### Synthetic Test Data
    
    
    # Example 1
    nums = [1,1,1,2,2,3], k = 2 → [1,2]
    # Example 2
    nums = [1], k = 1 → [1]

#### Python 3 Solution
    
    
    import heapq
    from collections import Counter
    
    def topKFrequent(nums: list[int], k: int) -> list[int]:
        # Count frequencies
        freq = Counter(nums)
    
        # Use min-heap of size k
        heap = []
        for num, count in freq.items():
            heapq.heappush(heap, (count, num))
            if len(heap) > k:
                heapq.heappop(heap)
    
        return [num for _, num in heap]

#### Explanation

**Time Complexity:** O(n log k) - n counter operations + k heap operations  
**Space Complexity:** O(n + k) - frequency dictionary and heap

We first count frequencies using `Counter`, then use a min-heap to keep track of the k most frequent elements.

* * *

### 1762\. Buildings With an Ocean View

#### Problem Statement

There are `n` buildings in a line. You are given an integer array `heights` representing the heights of the buildings. Return the indices of buildings that have an ocean view (all buildings to their right are shorter).

#### Synthetic Test Data
    
    
    # Example 1
    heights = [4,2,3,1] → [0,2,3]
    # Example 2
    heights = [4,3,2,1] → [0,1,2,3]
    # Example 3
    heights = [1,3,2,4] → [3]

#### Python 3 Solution
    
    
    def findBuildings(heights: list[int]) -> list[int]:
        result = []
        max_height = 0
    
        # Iterate from right to left
        for i in range(len(heights) - 1, -1, -1):
            if heights[i] > max_height:
                result.append(i)
                max_height = heights[i]
    
        return result[::-1]  # Reverse to maintain original order

#### Explanation

**Time Complexity:** O(n) - single pass from right to left  
**Space Complexity:** O(1) excluding output - constant extra space

By traversing from right to left, we track the maximum height seen so far. A building has an ocean view if its height exceeds this maximum.

* * *

### 986\. Interval List Intersections

#### Problem Statement

You are given two lists of closed intervals, `firstList` and `secondList`, where each interval is `[start, end]`. Return the intersection of these two interval lists.

#### Synthetic Test Data
    
    
    # Example 1
    firstList = [[0,2],[5,10],[13,23],[24,25]]
    secondList = [[1,5],[8,12],[15,24],[25,26]]
    → [[1,2],[5,5],[8,10],[15,23],[24,24],[25,25]]

#### Python 3 Solution
    
    
    def intervalIntersection(firstList: list[list[int]], secondList: list[list[int]]) -> list[list[int]]:
        i = j = 0
        result = []
    
        while i < len(firstList) and j < len(secondList):
            start1, end1 = firstList[i]
            start2, end2 = secondList[j]
    
            # Find intersection
            start = max(start1, start2)
            end = min(end1, end2)
    
            if start <= end:
                result.append([start, end])
    
            # Move pointer for the interval that ends first
            if end1 < end2:
                i += 1
            else:
                j += 1
    
        return result

#### Explanation

**Time Complexity:** O(m + n) - m and n are lengths of the two lists  
**Space Complexity:** O(min(m, n)) - for result list

Using two pointers, we find intersections between intervals. The intersection is valid if the max of starts is ≤ min of ends. We then advance the pointer for the interval that ends earlier.

* * *

### 23\. Merge k Sorted Lists

#### Problem Statement

You are given an array of `k` linked lists, each sorted in ascending order. Merge all linked lists into one sorted linked list and return it.

#### Synthetic Test Data
    
    
    # Example
    lists = [[1,4,5], [1,3,4], [2,6]] → [1,1,2,3,4,4,5,6]

#### Python 3 Solution
    
    
    import heapq
    
    class ListNode:
        def __init__(self, val=0, next=None):
            self.val = val
            self.next = next
    
    def mergeKLists(lists: list[ListNode]) -> ListNode:
        # Min-heap of (value, list_index, node)
        heap = []
        dummy = ListNode(0)
        current = dummy
    
        # Initialize heap with head of each list
        for i, node in enumerate(lists):
            if node:
                heapq.heappush(heap, (node.val, i, node))
    
        while heap:
            val, i, node = heapq.heappop(heap)
            current.next = node
            current = current.next
    
            if node.next:
                heapq.heappush(heap, (node.next.val, i, node.next))
    
        return dummy.next

#### Explanation

**Time Complexity:** O(N log k) where N is total number of nodes and k is number of lists  
**Space Complexity:** O(k) - for the heap

We use a min-heap to always extract the smallest node among all list heads. After extracting, we push the next node from that list into the heap.

* * *

### 426\. Convert Binary Search Tree to Sorted Doubly Linked List

#### Problem Statement

Convert a BST to a sorted circular doubly-linked list in-place. The left pointer should act as `prev` and the right pointer as `next`.

#### Synthetic Test Data
    
    
    # Example
    # BST: [4,2,5,1,3] → 1<->2<->3<->4<->5 (circular)

#### Python 3 Solution
    
    
    class Node:
        def __init__(self, val, left=None, right=None):
            self.val = val
            self.left = left
            self.right = right
    
    def treeToDoublyList(root: Node) -> Node:
        if not root:
            return None
    
        def inorder_convert(node: Node):
            nonlocal first, last
    
            if not node:
                return
    
            # Process left subtree
            inorder_convert(node.left)
    
            # Process current node
            if last:
                last.right = node
                node.left = last
            else:
                first = node
    
            last = node
    
            # Process right subtree
            inorder_convert(node.right)
    
        first = last = None
        inorder_convert(root)
    
        # Make it circular
        if first and last:
            first.left = last
            last.right = first
    
        return first

#### Explanation

**Time Complexity:** O(n) - visit each node once  
**Space Complexity:** O(h) - recursion stack for inorder traversal

We perform an inorder traversal of the BST, which naturally visits nodes in sorted order. As we traverse, we connect adjacent nodes using their left/right pointers.

* * *

### 921\. Minimum Add to Make Parentheses Valid

#### Problem Statement

A parentheses string is valid if it has matching parentheses. Return the minimum number of parentheses to add to make the string valid.

#### Synthetic Test Data
    
    
    # Example 1
    s = "())" → 1
    # Example 2
    s = "(((" → 3
    # Example 3
    s = "()" → 0

#### Python 3 Solution
    
    
    def minAddToMakeValid(s: str) -> int:
        open_needed = 0
        close_needed = 0
    
        for char in s:
            if char == '(':
                open_needed += 1
            else:  # char == ')'
                if open_needed > 0:
                    open_needed -= 1
                else:
                    close_needed += 1
    
        return open_needed + close_needed

#### Explanation

**Time Complexity:** O(n) - single pass through the string  
**Space Complexity:** O(1) - constant extra space

We track unmatched opening parentheses. When we see a closing parenthesis, if there's an unmatched opening, we match them; otherwise, we need an additional opening.

* * *

### 1004\. Max Consecutive Ones III

#### Problem Statement

Given a binary array `nums` and an integer `k`, return the maximum number of consecutive 1's if you can flip at most `k` 0's to 1's.

#### Synthetic Test Data
    
    
    # Example 1
    nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2 → 6
    # Example 2
    nums = [0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1], k = 3 → 10

#### Python 3 Solution
    
    
    def longestOnes(nums: list[int], k: int) -> int:
        left = 0
        max_len = 0
        zero_count = 0
    
        for right in range(len(nums)):
            if nums[right] == 0:
                zero_count += 1
    
            # Shrink window if we have too many zeros
            while zero_count > k:
                if nums[left] == 0:
                    zero_count -= 1
                left += 1
    
            max_len = max(max_len, right - left + 1)
    
        return max_len

#### Explanation

**Time Complexity:** O(n) - each element processed at most twice  
**Space Complexity:** O(1) - constant extra space

We use a sliding window approach. We expand the window to the right, tracking the number of zeros. When the zeros exceed k, we shrink from the left until we have at most k zeros.

* * *

### 346\. Moving Average from Data Stream

#### Problem Statement

Given a stream of integers and a window size, calculate the moving average of all integers in the sliding window.

#### Synthetic Test Data
    
    
    # Example
    movingAverage = MovingAverage(3)
    movingAverage.next(1) → 1.0
    movingAverage.next(10) → 5.5
    movingAverage.next(3) → 4.66667
    movingAverage.next(5) → 6.0

#### Python 3 Solution
    
    
    from collections import deque
    
    class MovingAverage:
        def __init__(self, size: int):
            self.size = size
            self.queue = deque()
            self.window_sum = 0
    
        def next(self, val: int) -> float:
            if len(self.queue) == self.size:
                self.window_sum -= self.queue.popleft()
    
            self.queue.append(val)
            self.window_sum += val
    
            return self.window_sum / len(self.queue)

#### Explanation

**Time Complexity:** O(1) for each next operation  
**Space Complexity:** O(size) - for the queue

We maintain a queue to store window elements and track their sum. When the window exceeds size, we remove the oldest element and adjust the sum accordingly.

* * *

### 129\. Sum Root to Leaf Numbers

#### Problem Statement

You are given the root of a binary tree containing digits 0-9. Each root-to-leaf path represents a number. Return the total sum of all root-to-leaf numbers.

#### Synthetic Test Data
    
    
    # Example 1
    # Tree: [1,2,3] → 25 (12 + 13)
    # Example 2
    # Tree: [4,9,0,5,1] → 1026 (495 + 491 + 40)

#### Python 3 Solution
    
    
    class TreeNode:
        def __init__(self, val=0, left=None, right=None):
            self.val = val
            self.left = left
            self.right = right
    
    def sumNumbers(root: TreeNode) -> int:
        def dfs(node: TreeNode, current_sum: int) -> int:
            if not node:
                return 0
    
            # Update current sum
            current_sum = current_sum * 10 + node.val
    
            # If leaf node, return current sum
            if not node.left and not node.right:
                return current_sum
    
            # Recursively process subtrees
            return dfs(node.left, current_sum) + dfs(node.right, current_sum)
    
        return dfs(root, 0)

#### Explanation

**Time Complexity:** O(n) - visit each node once  
**Space Complexity:** O(h) - recursion stack for DFS

We perform a depth-first search, building numbers as we traverse. When we reach a leaf node, we return the constructed number and sum these values.

* * *

### 791\. Custom Sort String

#### Problem Statement

`order` is a permutation of unique characters. Sort string `s` such that characters appear in the same order as in `order`. Characters not in `order` can be placed anywhere (after sorted characters).

#### Synthetic Test Data
    
    
    # Example 1
    order = "cba", s = "abcd" → "cbad"
    # Example 2
    order = "bcafg", s = "abcd" → "bcad"

#### Python 3 Solution
    
    
    from collections import Counter
    
    def customSortString(order: str, s: str) -> str:
        # Count characters in s
        char_count = Counter(s)
        result = []
    
        # Add characters in order
        for char in order:
            if char in char_count:
                result.append(char * char_count[char])
                del char_count[char]
    
        # Add remaining characters
        for char, count in char_count.items():
            result.append(char * count)
    
        return ''.join(result)

#### Explanation

**Time Complexity:** O(m + n) where m = len(order), n = len(s)  
**Space Complexity:** O(n) - for counters and result

We count frequencies of characters in s, then build the result by adding characters in the specified order first, followed by remaining characters.

* * *

### 670\. Maximum Swap

#### Problem Statement

You are given an integer. You can swap two digits at most once to get the maximum possible number. Return the maximum number.

#### Synthetic Test Data
    
    
    # Example 1
    num = 2736 → 7236
    # Example 2
    num = 9973 → 9973
    # Example 3
    num = 98368 → 98863

#### Python 3 Solution
    
    
    def maximumSwap(num: int) -> int:
        digits = list(str(num))
        last_occurrence = {int(d): i for i, d in enumerate(digits)}
    
        for i, d in enumerate(digits):
            # Try to find a larger digit to swap with
            for larger_digit in range(9, int(d), -1):
                if larger_digit in last_occurrence and last_occurrence[larger_digit] > i:
                    # Swap digits
                    digits[i], digits[last_occurrence[larger_digit]] = digits[last_occurrence[larger_digit]], digits[i]
                    return int(''.join(digits))
    
        return num

#### Explanation

**Time Complexity:** O(n) - single pass with limited digit checks  
**Space Complexity:** O(n) - for digit list

We record the last occurrence of each digit, then scan from left to right looking for a larger digit that appears later to create the maximum number.

* * *

### 987\. Vertical Order Traversal of a Binary Tree

#### Problem Statement

Given the root of a binary tree, calculate the vertical order traversal of the binary tree. For nodes in the same column, sort by row, then by value.

#### Synthetic Test Data
    
    
    # Example 1
    # Tree: [3,9,20,null,null,15,7] → [[9],[3,15],[20],[7]]

#### Python 3 Solution
    
    
    from collections import defaultdict
    import heapq
    
    class TreeNode:
        def __init__(self, val=0, left=None, right=None):
            self.val = val
            self.left = left
            self.right = right
    
    def verticalTraversal(root: TreeNode) -> list[list[int]]:
        if not root:
            return []
    
        # Map column -> list of (row, value)
        col_map = defaultdict(list)
    
        def dfs(node: TreeNode, row: int, col: int):
            if not node:
                return
    
            col_map[col].append((row, node.val))
    
            dfs(node.left, row + 1, col - 1)
            dfs(node.right, row + 1, col + 1)
    
        dfs(root, 0, 0)
    
        # Sort columns and sort within each column
        result = []
        for col in sorted(col_map.keys()):
            # Sort by row then by value
            col_map[col].sort(key=lambda x: (x[0], x[1]))
            result.append([val for _, val in col_map[col]])
    
        return result

#### Explanation

**Time Complexity:** O(n log n) - sorting within columns  
**Space Complexity:** O(n) - for storing all nodes

We assign row and column coordinates to each node. Then group nodes by column, sorting first by row, then by value within each column.

* * *

### 15\. 3Sum

#### Problem Statement

Given an integer array nums, return all the triplets `[nums[i], nums[j], nums[k]]` such that i, j, k are distinct and `nums[i] + nums[j] + nums[k] = 0`.

#### Synthetic Test Data
    
    
    # Example 1
    nums = [-1,0,1,2,-1,-4] → [[-1,-1,2],[-1,0,1]]
    # Example 2
    nums = [0,1,1] → []
    # Example 3
    nums = [0,0,0] → [[0,0,0]]

#### Python 3 Solution
    
    
    def threeSum(nums: list[int]) -> list[list[int]]:
        nums.sort()
        result = []
        n = len(nums)
    
        for i in range(n - 2):
            # Skip duplicates
            if i > 0 and nums[i] == nums[i - 1]:
                continue
    
            left, right = i + 1, n - 1
    
            while left < right:
                total = nums[i] + nums[left] + nums[right]
    
                if total < 0:
                    left += 1
                elif total > 0:
                    right -= 1
                else:
                    result.append([nums[i], nums[left], nums[right]])
    
                    # Skip duplicates
                    while left < right and nums[left] == nums[left + 1]:
                        left += 1
                    while left < right and nums[right] == nums[right - 1]:
                        right -= 1
    
                    left += 1
                    right -= 1
    
        return result

#### Explanation

**Time Complexity:** O(n²) - sorting plus two-pointer traversal  
**Space Complexity:** O(1) excluding output - constant extra space

After sorting, we fix one element and use two pointers to find pairs that sum to the target. We skip duplicates to avoid repeating triplets.

* * *

### 133\. Clone Graph

#### Problem Statement

Given a reference of a node in a connected undirected graph, return a deep copy of the graph.

#### Synthetic Test Data
    
    
    # Example
    # Graph with nodes 1-2-3-4 in a cycle

#### Python 3 Solution
    
    
    class Node:
        def __init__(self, val=0, neighbors=None):
            self.val = val
            self.neighbors = neighbors if neighbors is not None else []
    
    def cloneGraph(node: Node) -> Node:
        if not node:
            return None
    
        from collections import deque
    
        # Map original nodes to their clones
        clones = {}
    
        # BFS to clone graph
        queue = deque([node])
        clones[node] = Node(node.val)
    
        while queue:
            current = queue.popleft()
    
            for neighbor in current.neighbors:
                if neighbor not in clones:
                    clones[neighbor] = Node(neighbor.val)
                    queue.append(neighbor)
    
                # Add connection in cloned graph
                clones[current].neighbors.append(clones[neighbor])
    
        return clones[node]

#### Explanation

**Time Complexity:** O(V + E) - visit each vertex and edge once  
**Space Complexity:** O(V) - for the queue and clone map

We use BFS to traverse the original graph while simultaneously creating clones. A hash map tracks the mapping between original and cloned nodes.

* * *

### 1539\. Kth Missing Positive Number

#### Problem Statement

Given an array `arr` of positive integers sorted in strictly increasing order, and an integer `k`, return the kth missing positive integer.

#### Synthetic Test Data
    
    
    # Example 1
    arr = [2,3,4,7,11], k = 5 → 9
    # Example 2
    arr = [1,2,3,4], k = 2 → 6

#### Python 3 Solution
    
    
    def findKthPositive(arr: list[int], k: int) -> int:
        # Binary search approach
        left, right = 0, len(arr)
    
        while left < right:
            mid = (left + right) // 2
            # Number of missing numbers up to arr[mid]
            missing = arr[mid] - (mid + 1)
    
            if missing < k:
                left = mid + 1
            else:
                right = mid
    
        return left + k

#### Explanation

**Time Complexity:** O(log n) - binary search  
**Space Complexity:** O(1) - constant extra space

We use binary search to find the position where the kth missing number would be. The number of missing numbers up to index i is `arr[i] - (i + 1)`.

* * *

## Hard Problems

### 65\. Valid Number

#### Problem Statement

A valid number can be an integer, decimal, or number with exponent. Validate if a given string is a valid number.

#### Synthetic Test Data
    
    
    # Example valid numbers
    "0", " 0.1 ", "2e10", "-90E3", "6e-1", "53.5e93" → True
    # Example invalid numbers
    "abc", "1a", "1e", "e3", "99e2.5" → False

#### Python 3 Solution
    
    
    def isNumber(s: str) -> bool:
        s = s.strip()
        if not s:
            return False
    
        seen_digit = seen_dot = seen_e = False
        num_count = 0
    
        for i, char in enumerate(s):
            if char.isdigit():
                seen_digit = True
                num_count += 1
            elif char in '+-':
                if i > 0 and s[i-1] not in 'eE':
                    return False
            elif char == '.':
                if seen_dot or seen_e:
                    return False
                seen_dot = True
            elif char in 'eE':
                if seen_e or not seen_digit:
                    return False
                seen_e = True
                seen_digit = False  # Reset for exponent part
            else:
                return False
    
        return seen_digit

#### Explanation

**Time Complexity:** O(n) - single pass  
**Space Complexity:** O(1) - constant extra space

We parse the number following the rules: digits are required, at most one decimal point before exponent, at most one exponent sign, and sign characters only allowed at beginning or after exponent.

* * *

### 76\. Minimum Window Substring

#### Problem Statement

Given two strings `s` and `t`, return the minimum window substring of `s` such that every character in `t` (including duplicates) is included in the window.

#### Synthetic Test Data
    
    
    # Example 1
    s = "ADOBECODEBANC", t = "ABC" → "BANC"
    # Example 2
    s = "a", t = "a" → "a"
    # Example 3
    s = "a", t = "aa" → ""

#### Python 3 Solution
    
    
    from collections import Counter
    
    def minWindow(s: str, t: str) -> str:
        if not s or not t:
            return ""
    
        # Count characters in t
        need = Counter(t)
        have = {}
        required = len(need)
        formed = 0
    
        left = 0
        result = float('inf'), None, None
    
        for right, char in enumerate(s):
            # Expand window to the right
            if char in need:
                have[char] = have.get(char, 0) + 1
                if have[char] == need[char]:
                    formed += 1
    
            # Try to shrink window from left
            while left <= right and formed == required:
                # Update result if current window is smaller
                if right - left + 1 < result[0]:
                    result = (right - left + 1, left, right)
    
                # Remove left character
                left_char = s[left]
                if left_char in need:
                    have[left_char] -= 1
                    if have[left_char] < need[left_char]:
                        formed -= 1
                left += 1
    
        return s[result[1]:result[2] + 1] if result[0] != float('inf') else ""

#### Explanation

**Time Complexity:** O(|s| + |t|) - each character processed at most twice  
**Space Complexity:** O(|s| + |t|) - for counters

We use a sliding window with two pointers. We expand the right pointer until we have a valid window, then shrink from the left to find the minimum.

* * *

### 4\. Median of Two Sorted Arrays

#### Problem Statement

Given two sorted arrays `nums1` and `nums2` of sizes m and n, return the median of the two sorted arrays. The overall run time complexity should be O(log(m+n)).

#### Synthetic Test Data
    
    
    # Example 1
    nums1 = [1,3], nums2 = [2] → 2.0
    # Example 2
    nums1 = [1,2], nums2 = [3,4] → 2.5

#### Python 3 Solution
    
    
    def findMedianSortedArrays(nums1: list[int], nums2: list[int]) -> float:
        # Ensure nums1 is the smaller array
        if len(nums1) > len(nums2):
            nums1, nums2 = nums2, nums1
    
        m, n = len(nums1), len(nums2)
        left, right = 0, m
    
        while left <= right:
            # Partition positions
            partition1 = (left + right) // 2
            partition2 = (m + n + 1) // 2 - partition1
    
            # Handle edge cases
            max_left1 = nums1[partition1 - 1] if partition1 > 0 else float('-inf')
            min_right1 = nums1[partition1] if partition1 < m else float('inf')
            max_left2 = nums2[partition2 - 1] if partition2 > 0 else float('-inf')
            min_right2 = nums2[partition2] if partition2 < n else float('inf')
    
            # Check if partition is correct
            if max_left1 <= min_right2 and max_left2 <= min_right1:
                if (m + n) % 2 == 0:
                    return (max(max_left1, max_left2) + min(min_right1, min_right2)) / 2
                else:
                    return max(max_left1, max_left2)
            elif max_left1 > min_right2:
                right = partition1 - 1
            else:
                left = partition1 + 1
    
        raise ValueError("Input arrays are not sorted")

#### Explanation

**Time Complexity:** O(log(min(m, n))) - binary search on the smaller array  
**Space Complexity:** O(1) - constant extra space

We partition both arrays such that elements on the left are less than or equal to elements on the right. The median is found using the properties of these partitions.

* * *

### 124\. Binary Tree Maximum Path Sum

#### Problem Statement

A path in a binary tree is a sequence of nodes where each pair of adjacent nodes has an edge. A node can appear at most once. Find the maximum path sum.

#### Synthetic Test Data
    
    
    # Example 1
    # Tree: [1,2,3] → 6
    # Example 2
    # Tree: [-10,9,20,null,null,15,7] → 42

#### Python 3 Solution
    
    
    class TreeNode:
        def __init__(self, val=0, left=None, right=None):
            self.val = val
            self.left = left
            self.right = right
    
    def maxPathSum(root: TreeNode) -> int:
        max_sum = float('-inf')
    
        def dfs(node: TreeNode) -> int:
            nonlocal max_sum
            if not node:
                return 0
    
            # Get max sum from left and right subtrees
            left_sum = max(dfs(node.left), 0)
            right_sum = max(dfs(node.right), 0)
    
            # Update global max with path through current node
            max_sum = max(max_sum, node.val + left_sum + right_sum)
    
            # Return max sum of path ending at current node
            return node.val + max(left_sum, right_sum)
    
        dfs(root)
        return max_sum

#### Explanation

**Time Complexity:** O(n) - visit each node once  
**Space Complexity:** O(h) - recursion stack

For each node, we compute the maximum path sum that goes through that node (considering both left and right branches) and update the global maximum. We only include positive contributions from subtrees.

* * *

### 127\. Word Ladder

#### Problem Statement

Given two words `beginWord` and `endWord`, and a dictionary `wordList`, return the number of words in the shortest transformation sequence from `beginWord` to `endWord`, or 0 if no such sequence exists.

#### Synthetic Test Data
    
    
    # Example
    beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"] → 5

#### Python 3 Solution
    
    
    from collections import deque
    
    def ladderLength(beginWord: str, endWord: str, wordList: list[str]) -> int:
        word_set = set(wordList)
        if endWord not in word_set:
            return 0
    
        # BFS from beginWord
        queue = deque([(beginWord, 1)])  # (word, distance)
        visited = {beginWord}
    
        while queue:
            word, distance = queue.popleft()
    
            if word == endWord:
                return distance
    
            # Generate all possible transformations
            for i in range(len(word)):
                for c in 'abcdefghijklmnopqrstuvwxyz':
                    if c == word[i]:
                        continue
    
                    new_word = word[:i] + c + word[i+1:]
                    if new_word in word_set and new_word not in visited:
                        visited.add(new_word)
                        queue.append((new_word, distance + 1))
    
        return 0

#### Explanation

**Time Complexity:** O(L² * N) where L is word length, N is number of words  
**Space Complexity:** O(N) - for queue and visited set

We use BFS to find the shortest path from beginWord to endWord. At each step, we generate all possible one-letter variations and check if they exist in the word list.

* * *

### 329\. Longest Increasing Path in a Matrix

#### Problem Statement

Given an m x n integers matrix, return the length of the longest increasing path. From each cell, you can move to four adjacent cells (up, down, left, right).

#### Synthetic Test Data
    
    
    # Example
    matrix = [[9,9,4],[6,6,8],[2,1,1]] → 4

#### Python 3 Solution
    
    
    def longestIncreasingPath(matrix: list[list[int]]) -> int:
        if not matrix or not matrix[0]:
            return 0
    
        m, n = len(matrix), len(matrix[0])
        memo = [[0] * n for _ in range(m)]
        directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
    
        def dfs(i: int, j: int) -> int:
            if memo[i][j] != 0:
                return memo[i][j]
    
            max_len = 1
            for di, dj in directions:
                ni, nj = i + di, j + dj
                if 0 <= ni < m and 0 <= nj < n and matrix[ni][nj] > matrix[i][j]:
                    max_len = max(max_len, 1 + dfs(ni, nj))
    
            memo[i][j] = max_len
            return max_len
    
        result = 0
        for i in range(m):
            for j in range(n):
                result = max(result, dfs(i, j))
    
        return result

#### Explanation

**Time Complexity:** O(m * n) - each cell processed once with memoization  
**Space Complexity:** O(m * n) - for memoization

We use DFS with memoization. For each cell, we compute the longest increasing path starting from it by exploring all four directions.

* * *

### 273\. Integer to English Words

#### Problem Statement

Convert a non-negative integer to its English words representation.

#### Synthetic Test Data
    
    
    # Example 1
    num = 123 → "One Hundred Twenty Three"
    # Example 2
    num = 12345 → "Twelve Thousand Three Hundred Forty Five"
    # Example 3
    num = 1000000 → "One Million"

#### Python 3 Solution
    
    
    def numberToWords(num: int) -> str:
        if num == 0:
            return "Zero"
    
        below_20 = ["", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine", 
                     "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen", "Sixteen", 
                     "Seventeen", "Eighteen", "Nineteen"]
        tens = ["", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"]
        thousands = ["", "Thousand", "Million", "Billion"]
    
        def helper(n: int) -> str:
            if n == 0:
                return ""
            elif n < 20:
                return below_20[n] + " "
            elif n < 100:
                return tens[n // 10] + " " + helper(n % 10)
            else:
                return below_20[n // 100] + " Hundred " + helper(n % 100)
    
        result = ""
        i = 0
        while num > 0:
            if num % 1000 != 0:
                result = helper(num % 1000) + thousands[i] + " " + result
            num //= 1000
            i += 1
    
        return result.strip()

#### Explanation

**Time Complexity:** O(log n) - number of digits  
**Space Complexity:** O(log n) - recursion stack

We break the number into chunks of three digits and process each chunk separately, adding the appropriate thousand suffix.

* * *

### 10\. Regular Expression Matching

#### Problem Statement

Implement regular expression matching with support for '.' (matches any single character) and '*' (matches zero or more of the preceding element).

#### Synthetic Test Data
    
    
    # Example 1
    s = "aa", p = "a" → False
    # Example 2
    s = "aa", p = "a*" → True
    # Example 3
    s = "ab", p = ".*" → True

#### Python 3 Solution
    
    
    def isMatch(s: str, p: str) -> bool:
        # Memoization cache
        memo = {}
    
        def dp(i: int, j: int) -> bool:
            if (i, j) in memo:
                return memo[(i, j)]
    
            if j >= len(p):
                return i >= len(s)
    
            # Check if current characters match
            first_match = i < len(s) and (p[j] == s[i] or p[j] == '.')
    
            # Handle '*' pattern
            if j + 1 < len(p) and p[j + 1] == '*':
                # Either skip '*' or use it to match current character
                memo[(i, j)] = dp(i, j + 2) or (first_match and dp(i + 1, j))
            else:
                memo[(i, j)] = first_match and dp(i + 1, j + 1)
    
            return memo[(i, j)]
    
        return dp(0, 0)

#### Explanation

**Time Complexity:** O(m * n) where m = len(s), n = len(p)  
**Space Complexity:** O(m * n) - for memoization

We use dynamic programming with memoization. For each state (i, j), we check if s[i:] matches p[j:], handling the '*' operator specially.

* * *

### 295\. Find Median from Data Stream

#### Problem Statement

Design a data structure that supports adding integers and finding the median of all added elements.

#### Synthetic Test Data
    
    
    # Example
    medianFinder = MedianFinder()
    medianFinder.addNum(1)
    medianFinder.addNum(2)
    medianFinder.findMedian() → 1.5
    medianFinder.addNum(3)
    medianFinder.findMedian() → 2.0

#### Python 3 Solution
    
    
    import heapq
    
    class MedianFinder:
        def __init__(self):
            self.max_heap = []  # Store smaller half (negated for max-heap)
            self.min_heap = []  # Store larger half
    
        def addNum(self, num: int) -> None:
            # Add to max_heap first
            heapq.heappush(self.max_heap, -num)
    
            # Balance: move max from max_heap to min_heap
            heapq.heappush(self.min_heap, -heapq.heappop(self.max_heap))
    
            # Ensure max_heap size >= min_heap size
            if len(self.max_heap) < len(self.min_heap):
                heapq.heappush(self.max_heap, -heapq.heappop(self.min_heap))
    
        def findMedian(self) -> float:
            if len(self.max_heap) > len(self.min_heap):
                return -self.max_heap[0]
            else:
                return (-self.max_heap[0] + self.min_heap[0]) / 2

#### Explanation

**Time Complexity:** O(log n) for addNum, O(1) for findMedian  
**Space Complexity:** O(n) - for storing all numbers

We maintain two heaps: a max-heap for the smaller half and a min-heap for the larger half. This allows constant-time median retrieval.

* * *

### 212\. Word Search II

#### Problem Statement

Given an m x n board of characters and a list of words, return all words on the board. Each word must be constructed from adjacent cells (horizontally or vertically).

#### Synthetic Test Data
    
    
    # Example
    board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]]
    words = ["oath","pea","eat","rain"] → ["eat","oath"]

#### Python 3 Solution
    
    
    class TrieNode:
        def __init__(self):
            self.children = {}
            self.word = None
    
    class Solution:
        def findWords(self, board: list[list[str]], words: list[str]) -> list[str]:
            # Build trie
            root = TrieNode()
            for word in words:
                node = root
                for char in word:
                    if char not in node.children:
                        node.children[char] = TrieNode()
                    node = node.children[char]
                node.word = word
    
            m, n = len(board), len(board[0])
            result = []
    
            def dfs(i: int, j: int, node: TrieNode):
                if i < 0 or i >= m or j < 0 or j >= n:
                    return
    
                char = board[i][j]
                if char not in node.children:
                    return
    
                node = node.children[char]
                if node.word:
                    result.append(node.word)
                    node.word = None  # Avoid duplicates
    
                # Mark as visited
                board[i][j] = '#'
    
                # Explore neighbors
                dfs(i + 1, j, node)
                dfs(i - 1, j, node)
                dfs(i, j + 1, node)
                dfs(i, j - 1, node)
    
                # Unmark
                board[i][j] = char
    
            for i in range(m):
                for j in range(n):
                    dfs(i, j, root)
    
            return result

#### Explanation

**Time Complexity:** O(m * n * 4^L) worst case, but optimized by trie  
**Space Complexity:** O(W * L) - for trie where W is number of words, L is max word length

We build a trie from all words, then perform DFS on the board, pruning when the current path doesn't match any prefix in the trie.

* * *

## Notes

This collection includes solutions for all requested LeetCode problems (Easy, Medium, and Hard categories). Each solution is optimized for readability, efficiency, and idiomatic Python 3 code. The implementations handle edge cases and include time/space complexity analysis.
