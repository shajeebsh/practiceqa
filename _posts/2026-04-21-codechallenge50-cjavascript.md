---
title: "CodeChallenge50-c#javascript"
date: 2026-04-21 22:04:44 
author: shajeebhameed
layout: page
slug: codechallenge50-cjavascript
status: publish
---

## 50 Common Coding Challenge Questions — C# & JavaScript  
  
> Covers: Arrays • Strings • Objects/Classes • Recursion • Sorting & Searching • Linked Lists • Trees • Dynamic Programming • Math & Logic • Async/Functional

* * *

* * *

## 🔷 Arrays

* * *

### 1\. Find Duplicates in an Array

**Problem:** Return all elements that appear more than once in an array.

#### ✅ C#
    
    
    using System.Collections.Generic;
    using System.Linq;
    
    public static List<int> FindDuplicates(int[] nums)
    {
        return nums
            .GroupBy(n => n)
            .Where(g => g.Count() > 1)
            .Select(g => g.Key)
            .ToList();
    }
    
    // Usage
    FindDuplicates(new[] { 1, 2, 3, 2, 4, 3 }); // [2, 3]
    
    

#### ✅ JavaScript
    
    
    function findDuplicates(nums) {
      const seen = new Set();
      const duplicates = new Set();
    
      for (const num of nums) {
        if (seen.has(num)) duplicates.add(num);
        else seen.add(num);
      }
    
      return [...duplicates];
    }
    
    // Usage
    findDuplicates([1, 2, 3, 2, 4, 3]); // [2, 3]
    
    

**Key Insight:** Use a `HashSet`/`Set` for O(n) time instead of nested loops O(n²).

* * *

### 2\. Two Sum

**Problem:** Given an array of integers and a target, return the indices of two numbers that add up to the target.

#### ✅ C#
    
    
    public static int[] TwoSum(int[] nums, int target)
    {
        var map = new Dictionary<int, int>();
    
        for (int i = 0; i < nums.Length; i++)
        {
            int complement = target - nums[i];
            if (map.ContainsKey(complement))
                return new[] { map[complement], i };
            map[nums[i]] = i;
        }
    
        return Array.Empty<int>();
    }
    
    // Usage
    TwoSum(new[] { 2, 7, 11, 15 }, 9); // [0, 1]
    
    

#### ✅ JavaScript
    
    
    function twoSum(nums, target) {
      const map = new Map();
    
      for (let i = 0; i < nums.length; i++) {
        const complement = target - nums[i];
        if (map.has(complement)) return [map.get(complement), i];
        map.set(nums[i], i);
      }
    
      return [];
    }
    
    // Usage
    twoSum([2, 7, 11, 15], 9); // [0, 1]
    
    

**Key Insight:** Store `value → index` in a hash map. One pass, O(n) time, O(n) space.

* * *

### 3\. Rotate an Array

**Problem:** Rotate an array to the right by `k` steps.

#### ✅ C#
    
    
    public static int[] RotateArray(int[] nums, int k)
    {
        int n = nums.Length;
        k %= n;
        Reverse(nums, 0, n - 1);
        Reverse(nums, 0, k - 1);
        Reverse(nums, k, n - 1);
        return nums;
    }
    
    private static void Reverse(int[] nums, int left, int right)
    {
        while (left < right)
        {
            (nums[left], nums[right]) = (nums[right], nums[left]);
            left++; right--;
        }
    }
    
    // Usage
    RotateArray(new[] { 1, 2, 3, 4, 5, 6, 7 }, 3); // [5, 6, 7, 1, 2, 3, 4]
    
    

#### ✅ JavaScript
    
    
    function rotateArray(nums, k) {
      k %= nums.length;
      const reverse = (arr, l, r) => {
        while (l < r) [arr[l++], arr[r--]] = [arr[r], arr[l]];
      };
      reverse(nums, 0, nums.length - 1);
      reverse(nums, 0, k - 1);
      reverse(nums, k, nums.length - 1);
      return nums;
    }
    
    // Usage
    rotateArray([1, 2, 3, 4, 5, 6, 7], 3); // [5, 6, 7, 1, 2, 3, 4]
    
    

**Key Insight:** Triple-reverse trick achieves O(n) time, O(1) space.

* * *

### 4\. Maximum Subarray (Kadane's Algorithm)

**Problem:** Find the contiguous subarray with the largest sum.

#### ✅ C#
    
    
    public static int MaxSubArray(int[] nums)
    {
        int maxSum = nums[0];
        int current = nums[0];
    
        for (int i = 1; i < nums.Length; i++)
        {
            current = Math.Max(nums[i], current + nums[i]);
            maxSum  = Math.Max(maxSum, current);
        }
    
        return maxSum;
    }
    
    // Usage
    MaxSubArray(new[] { -2, 1, -3, 4, -1, 2, 1, -5, 4 }); // 6
    
    

#### ✅ JavaScript
    
    
    function maxSubArray(nums) {
      let maxSum = nums[0];
      let current = nums[0];
    
      for (let i = 1; i < nums.length; i++) {
        current = Math.max(nums[i], current + nums[i]);
        maxSum  = Math.max(maxSum, current);
      }
    
      return maxSum;
    }
    
    // Usage
    maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4]); // 6
    
    

**Key Insight:** At each step, decide whether to extend the current subarray or start fresh.

* * *

### 5\. Flatten a Nested Array

**Problem:** Flatten an arbitrarily nested array into a single-level array.

#### ✅ C#
    
    
    using System.Collections;
    
    public static List<int> Flatten(IEnumerable source)
    {
        var result = new List<int>();
        foreach (var item in source)
        {
            if (item is IEnumerable nested && item is not string)
                result.AddRange(Flatten(nested));
            else
                result.Add(Convert.ToInt32(item));
        }
        return result;
    }
    
    // Usage
    Flatten(new object[] { 1, new object[] { 2, new object[] { 3, 4 }, 5 } });
    // [1, 2, 3, 4, 5]
    
    

#### ✅ JavaScript
    
    
    function flatten(arr) {
      return arr.reduce((acc, val) =>
        Array.isArray(val) ? acc.concat(flatten(val)) : acc.concat(val), []);
    }
    
    // Or using the built-in (modern JS):
    // arr.flat(Infinity)
    
    // Usage
    flatten([1, [2, [3, [4]], 5]]); // [1, 2, 3, 4, 5]
    
    

**Key Insight:** Recursive DFS with type check, or leverage native `Array.flat(Infinity)` in JS.

* * *

### 6\. Find the Missing Number

**Problem:** Given an array of `n` integers from 0 to n, find the missing number.

#### ✅ C#
    
    
    public static int MissingNumber(int[] nums)
    {
        int n = nums.Length;
        int expected = n * (n + 1) / 2;
        return expected - nums.Sum();
    }
    
    // Usage
    MissingNumber(new[] { 3, 0, 1 }); // 2
    
    

#### ✅ JavaScript
    
    
    function missingNumber(nums) {
      const n = nums.length;
      const expected = (n * (n + 1)) / 2;
      const actual = nums.reduce((sum, n) => sum + n, 0);
      return expected - actual;
    }
    
    // Usage
    missingNumber([3, 0, 1]); // 2
    
    

**Key Insight:** Gauss formula: expected sum minus actual sum = missing number. O(n) time, O(1) space.

* * *

### 7\. Intersection of Two Arrays

**Problem:** Return elements common to both arrays (unique).

#### ✅ C#
    
    
    public static int[] Intersection(int[] nums1, int[] nums2)
    {
        var set1 = new HashSet<int>(nums1);
        return nums2.Where(set1.Contains).Distinct().ToArray();
    }
    
    // Usage
    Intersection(new[] { 1, 2, 2, 1 }, new[] { 2, 2 }); // [2]
    
    

#### ✅ JavaScript
    
    
    function intersection(nums1, nums2) {
      const set1 = new Set(nums1);
      return [...new Set(nums2.filter(n => set1.has(n)))];
    }
    
    // Usage
    intersection([1, 2, 2, 1], [2, 2]); // [2]
    
    

* * *

### 8\. Move Zeroes to End

**Problem:** Move all zeroes to the end of the array while maintaining relative order of non-zero elements.

#### ✅ C#
    
    
    public static int[] MoveZeroes(int[] nums)
    {
        int insertPos = 0;
        foreach (int num in nums)
            if (num != 0) nums[insertPos++] = num;
    
        while (insertPos < nums.Length)
            nums[insertPos++] = 0;
    
        return nums;
    }
    
    // Usage
    MoveZeroes(new[] { 0, 1, 0, 3, 12 }); // [1, 3, 12, 0, 0]
    
    

#### ✅ JavaScript
    
    
    function moveZeroes(nums) {
      let insertPos = 0;
      for (const num of nums)
        if (num !== 0) nums[insertPos++] = num;
    
      while (insertPos < nums.length)
        nums[insertPos++] = 0;
    
      return nums;
    }
    
    // Usage
    moveZeroes([0, 1, 0, 3, 12]); // [1, 3, 12, 0, 0]
    
    

**Key Insight:** Two-pointer technique — one pointer for the current position, one for the write position.

* * *

## 🔷 Strings

* * *

### 9\. Reverse a String

**Problem:** Reverse a string in-place.

#### ✅ C#
    
    
    public static string ReverseString(string s)
    {
        char[] chars = s.ToCharArray();
        int left = 0, right = chars.Length - 1;
    
        while (left < right)
        {
            (chars[left], chars[right]) = (chars[right], chars[left]);
            left++; right--;
        }
    
        return new string(chars);
    }
    
    // Usage
    ReverseString("hello"); // "olleh"
    
    

#### ✅ JavaScript
    
    
    function reverseString(s) {
      return s.split('').reverse().join('');
    }
    
    // Manual two-pointer version:
    function reverseStringManual(s) {
      const arr = s.split('');
      let left = 0, right = arr.length - 1;
      while (left < right) {
        [arr[left++], arr[right--]] = [arr[right], arr[left]];
      }
      return arr.join('');
    }
    
    // Usage
    reverseString("hello"); // "olleh"
    
    

* * *

### 10\. Check if a String is a Palindrome

**Problem:** Return `true` if the string reads the same forwards and backwards (ignore case and non-alphanumerics).

#### ✅ C#
    
    
    public static bool IsPalindrome(string s)
    {
        string clean = new string(s.ToLower().Where(char.IsLetterOrDigit).ToArray());
        int left = 0, right = clean.Length - 1;
    
        while (left < right)
        {
            if (clean[left++] != clean[right--]) return false;
        }
    
        return true;
    }
    
    // Usage
    IsPalindrome("A man, a plan, a canal: Panama"); // true
    
    

#### ✅ JavaScript
    
    
    function isPalindrome(s) {
      const clean = s.toLowerCase().replace(/[^a-z0-9]/g, '');
      return clean === clean.split('').reverse().join('');
    }
    
    // Usage
    isPalindrome("A man, a plan, a canal: Panama"); // true
    
    

* * *

### 11\. Check if Two Strings are Anagrams

**Problem:** Return `true` if one string is an anagram of another.

#### ✅ C#
    
    
    public static bool IsAnagram(string s, string t)
    {
        if (s.Length != t.Length) return false;
        var count = new Dictionary<char, int>();
    
        foreach (char c in s)
            count[c] = count.GetValueOrDefault(c, 0) + 1;
    
        foreach (char c in t)
        {
            if (!count.ContainsKey(c) || count[c] == 0) return false;
            count[c]--;
        }
    
        return true;
    }
    
    // Usage
    IsAnagram("anagram", "nagaram"); // true
    
    

#### ✅ JavaScript
    
    
    function isAnagram(s, t) {
      if (s.length !== t.length) return false;
      const count = {};
    
      for (const c of s) count[c] = (count[c] || 0) + 1;
      for (const c of t) {
        if (!count[c]) return false;
        count[c]--;
      }
    
      return true;
    }
    
    // Usage
    isAnagram("anagram", "nagaram"); // true
    
    

* * *

### 12\. Count Character Occurrences

**Problem:** Return a map of each character and its frequency in a string.

#### ✅ C#
    
    
    public static Dictionary<char, int> CharCount(string s)
    {
        return s.GroupBy(c => c).ToDictionary(g => g.Key, g => g.Count());
    }
    
    // Usage
    CharCount("hello"); // { h:1, e:1, l:2, o:1 }
    
    

#### ✅ JavaScript
    
    
    function charCount(s) {
      return [...s].reduce((acc, c) => {
        acc[c] = (acc[c] || 0) + 1;
        return acc;
      }, {});
    }
    
    // Usage
    charCount("hello"); // { h:1, e:1, l:2, o:1 }
    
    

* * *

### 13\. String Compression

**Problem:** Compress a string so `"aaabbc"` becomes `"a3b2c1"`. Return the original if the compressed version is not shorter.

#### ✅ C#
    
    
    public static string Compress(string s)
    {
        var sb = new System.Text.StringBuilder();
        int count = 1;
    
        for (int i = 1; i <= s.Length; i++)
        {
            if (i < s.Length && s[i] == s[i - 1])
                count++;
            else
            {
                sb.Append(s[i - 1]);
                sb.Append(count);
                count = 1;
            }
        }
    
        string compressed = sb.ToString();
        return compressed.Length < s.Length ? compressed : s;
    }
    
    // Usage
    Compress("aaabbc"); // "a3b2c1"
    Compress("abc");    // "abc" (not shorter)
    
    

#### ✅ JavaScript
    
    
    function compress(s) {
      let result = '';
      let count = 1;
    
      for (let i = 1; i <= s.length; i++) {
        if (i < s.length && s[i] === s[i - 1]) {
          count++;
        } else {
          result += s[i - 1] + count;
          count = 1;
        }
      }
    
      return result.length < s.length ? result : s;
    }
    
    // Usage
    compress("aaabbc"); // "a3b2c1"
    
    

* * *

### 14\. Valid Brackets / Parentheses

**Problem:** Return `true` if the string has valid, matching bracket pairs.

#### ✅ C#
    
    
    public static bool IsValidBrackets(string s)
    {
        var stack = new Stack<char>();
        var pairs = new Dictionary<char, char> { [')'] = '(', [']'] = '[', ['}'] = '{' };
    
        foreach (char c in s)
        {
            if ("([{".Contains(c)) stack.Push(c);
            else if (pairs.ContainsKey(c))
            {
                if (stack.Count == 0 || stack.Pop() != pairs[c]) return false;
            }
        }
    
        return stack.Count == 0;
    }
    
    // Usage
    IsValidBrackets("()[]{}"); // true
    IsValidBrackets("(]");     // false
    
    

#### ✅ JavaScript
    
    
    function isValidBrackets(s) {
      const stack = [];
      const pairs = { ')': '(', ']': '[', '}': '{' };
    
      for (const c of s) {
        if ('([{'.includes(c)) stack.push(c);
        else if (pairs[c]) {
          if (stack.pop() !== pairs[c]) return false;
        }
      }
    
      return stack.length === 0;
    }
    
    // Usage
    isValidBrackets("()[]{}"); // true
    isValidBrackets("(]");     // false
    
    

* * *

### 15\. Longest Substring Without Repeating Characters

**Problem:** Return the length of the longest substring with all unique characters.

#### ✅ C#
    
    
    public static int LengthOfLongestSubstring(string s)
    {
        var seen = new Dictionary<char, int>();
        int max = 0, left = 0;
    
        for (int right = 0; right < s.Length; right++)
        {
            if (seen.TryGetValue(s[right], out int prev) && prev >= left)
                left = prev + 1;
    
            seen[s[right]] = right;
            max = Math.Max(max, right - left + 1);
        }
    
        return max;
    }
    
    // Usage
    LengthOfLongestSubstring("abcabcbb"); // 3 ("abc")
    
    

#### ✅ JavaScript
    
    
    function lengthOfLongestSubstring(s) {
      const seen = new Map();
      let max = 0, left = 0;
    
      for (let right = 0; right < s.length; right++) {
        if (seen.has(s[right]) && seen.get(s[right]) >= left)
          left = seen.get(s[right]) + 1;
    
        seen.set(s[right], right);
        max = Math.max(max, right - left + 1);
      }
    
      return max;
    }
    
    // Usage
    lengthOfLongestSubstring("abcabcbb"); // 3
    
    

* * *

### 16\. Caesar Cipher

**Problem:** Encrypt a string by shifting each letter by `k` positions.

#### ✅ C#
    
    
    public static string CaesarCipher(string s, int k)
    {
        return new string(s.Select(c =>
        {
            if (!char.IsLetter(c)) return c;
            char base_ = char.IsUpper(c) ? 'A' : 'a';
            return (char)((c - base_ + k) % 26 + base_);
        }).ToArray());
    }
    
    // Usage
    CaesarCipher("Hello, World!", 3); // "Khoor, Zruog!"
    
    

#### ✅ JavaScript
    
    
    function caesarCipher(s, k) {
      return s.replace(/[a-zA-Z]/g, c => {
        const base = c <= 'Z' ? 65 : 97;
        return String.fromCharCode(((c.charCodeAt(0) - base + k) % 26) + base);
      });
    }
    
    // Usage
    caesarCipher("Hello, World!", 3); // "Khoor, Zruog!"
    
    

* * *

## 🔷 Objects & Classes

* * *

### 17\. Deep Clone an Object

**Problem:** Create a deep copy of a nested object without referencing the original.

#### ✅ C#
    
    
    using System.Text.Json;
    
    public static T DeepClone<T>(T obj)
    {
        var json = JsonSerializer.Serialize(obj);
        return JsonSerializer.Deserialize<T>(json)!;
    }
    
    // Usage
    var original = new { Name = "Alice", Address = new { City = "Dublin" } };
    var clone = DeepClone(original);
    
    

#### ✅ JavaScript
    
    
    // Simple deep clone using JSON (works for plain objects)
    function deepClone(obj) {
      return JSON.parse(JSON.stringify(obj));
    }
    
    // Recursive version (handles more edge cases)
    function deepCloneRecursive(obj) {
      if (obj === null || typeof obj !== 'object') return obj;
      if (Array.isArray(obj)) return obj.map(deepCloneRecursive);
      return Object.fromEntries(
        Object.entries(obj).map(([k, v]) => [k, deepCloneRecursive(v)])
      );
    }
    
    // Usage
    const original = { a: 1, b: { c: 2 } };
    const clone = deepClone(original);
    clone.b.c = 99; // original.b.c remains 2
    
    

* * *

### 18\. Merge Two Objects (Deep Merge)

**Problem:** Deeply merge two objects, with the second object's values overriding the first.

#### ✅ C#
    
    
    using System.Text.Json;
    using System.Text.Json.Nodes;
    
    public static JsonObject DeepMerge(JsonObject target, JsonObject source)
    {
        foreach (var prop in source)
        {
            if (prop.Value is JsonObject srcObj && target[prop.Key] is JsonObject tgtObj)
                target[prop.Key] = DeepMerge(tgtObj, srcObj);
            else
                target[prop.Key] = prop.Value?.DeepClone();
        }
        return target;
    }
    
    

#### ✅ JavaScript
    
    
    function deepMerge(target, source) {
      const output = { ...target };
    
      for (const key of Object.keys(source)) {
        if (source[key] instanceof Object && !Array.isArray(source[key]) && key in target) {
          output[key] = deepMerge(target[key], source[key]);
        } else {
          output[key] = source[key];
        }
      }
    
      return output;
    }
    
    // Usage
    deepMerge({ a: 1, b: { x: 1 } }, { b: { y: 2 }, c: 3 });
    // { a: 1, b: { x: 1, y: 2 }, c: 3 }
    
    

* * *

### 19\. Group By

**Problem:** Group an array of objects by a given property.

#### ✅ C#
    
    
    var people = new[]
    {
        new { Name = "Alice", Dept = "Eng" },
        new { Name = "Bob",   Dept = "HR"  },
        new { Name = "Carol", Dept = "Eng" }
    };
    
    var grouped = people.GroupBy(p => p.Dept)
                        .ToDictionary(g => g.Key, g => g.ToList());
    
    // { "Eng": [Alice, Carol], "HR": [Bob] }
    
    

#### ✅ JavaScript
    
    
    function groupBy(arr, key) {
      return arr.reduce((acc, item) => {
        const group = item[key];
        acc[group] = acc[group] ? [...acc[group], item] : [item];
        return acc;
      }, {});
    }
    
    // Usage
    const people = [
      { name: 'Alice', dept: 'Eng' },
      { name: 'Bob',   dept: 'HR'  },
      { name: 'Carol', dept: 'Eng' }
    ];
    groupBy(people, 'dept');
    // { Eng: [Alice, Carol], HR: [Bob] }
    
    

* * *

### 20\. Debounce Function

**Problem:** Implement debounce — delays a function call until after `delay` ms have elapsed since the last call.

#### ✅ C# (using Timer)
    
    
    public static Action Debounce(Action action, int delayMs)
    {
        System.Timers.Timer? timer = null;
    
        return () =>
        {
            timer?.Stop();
            timer?.Dispose();
            timer = new System.Timers.Timer(delayMs) { AutoReset = false };
            timer.Elapsed += (_, _) => action();
            timer.Start();
        };
    }
    
    // Usage
    var debouncedSave = Debounce(() => Console.WriteLine("Saved!"), 300);
    
    

#### ✅ JavaScript
    
    
    function debounce(fn, delay) {
      let timer;
      return function (...args) {
        clearTimeout(timer);
        timer = setTimeout(() => fn.apply(this, args), delay);
      };
    }
    
    // Usage
    const debouncedSearch = debounce((query) => console.log('Searching:', query), 300);
    input.addEventListener('input', e => debouncedSearch(e.target.value));
    
    

* * *

### 21\. Implement a Stack

**Problem:** Build a Stack class with `push`, `pop`, `peek`, and `isEmpty`.

#### ✅ C#
    
    
    public class Stack<T>
    {
        private readonly List<T> _items = new();
    
        public void Push(T item) => _items.Add(item);
    
        public T Pop()
        {
            if (IsEmpty()) throw new InvalidOperationException("Stack is empty");
            var item = _items[^1];
            _items.RemoveAt(_items.Count - 1);
            return item;
        }
    
        public T Peek() => IsEmpty()
            ? throw new InvalidOperationException("Stack is empty")
            : _items[^1];
    
        public bool IsEmpty() => _items.Count == 0;
    }
    
    

#### ✅ JavaScript
    
    
    class Stack {
      #items = [];
    
      push(item) { this.#items.push(item); }
    
      pop() {
        if (this.isEmpty()) throw new Error('Stack is empty');
        return this.#items.pop();
      }
    
      peek() {
        if (this.isEmpty()) throw new Error('Stack is empty');
        return this.#items[this.#items.length - 1];
      }
    
      isEmpty() { return this.#items.length === 0; }
    }
    
    // Usage
    const stack = new Stack();
    stack.push(1); stack.push(2);
    stack.pop();   // 2
    stack.peek();  // 1
    
    

* * *

### 22\. Implement a Queue

**Problem:** Build a Queue class with `enqueue`, `dequeue`, `front`, and `isEmpty`.

#### ✅ C#
    
    
    public class Queue<T>
    {
        private readonly LinkedList<T> _items = new();
    
        public void Enqueue(T item) => _items.AddLast(item);
    
        public T Dequeue()
        {
            if (IsEmpty()) throw new InvalidOperationException("Queue is empty");
            var item = _items.First!.Value;
            _items.RemoveFirst();
            return item;
        }
    
        public T Front() => IsEmpty()
            ? throw new InvalidOperationException("Queue is empty")
            : _items.First!.Value;
    
        public bool IsEmpty() => _items.Count == 0;
    }
    
    

#### ✅ JavaScript
    
    
    class Queue {
      #items = [];
    
      enqueue(item) { this.#items.push(item); }
    
      dequeue() {
        if (this.isEmpty()) throw new Error('Queue is empty');
        return this.#items.shift();
      }
    
      front() {
        if (this.isEmpty()) throw new Error('Queue is empty');
        return this.#items[0];
      }
    
      isEmpty() { return this.#items.length === 0; }
    }
    
    

* * *

## 🔷 Recursion

* * *

### 23\. Fibonacci Sequence

**Problem:** Return the nth Fibonacci number (memoized for efficiency).

#### ✅ C#
    
    
    public static long Fibonacci(int n, Dictionary<int, long>? memo = null)
    {
        memo ??= new Dictionary<int, long>();
        if (n <= 1) return n;
        if (memo.ContainsKey(n)) return memo[n];
    
        memo[n] = Fibonacci(n - 1, memo) + Fibonacci(n - 2, memo);
        return memo[n];
    }
    
    // Usage
    Fibonacci(10); // 55
    
    

#### ✅ JavaScript
    
    
    function fibonacci(n, memo = {}) {
      if (n <= 1) return n;
      if (memo[n]) return memo[n];
      memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo);
      return memo[n];
    }
    
    // Usage
    fibonacci(10); // 55
    
    

* * *

### 24\. Factorial

**Problem:** Compute n! recursively.

#### ✅ C#
    
    
    public static long Factorial(int n)
    {
        if (n < 0) throw new ArgumentException("n must be non-negative");
        return n <= 1 ? 1 : n * Factorial(n - 1);
    }
    
    // Usage
    Factorial(6); // 720
    
    

#### ✅ JavaScript
    
    
    function factorial(n) {
      if (n < 0) throw new Error('n must be non-negative');
      return n <= 1 ? 1 : n * factorial(n - 1);
    }
    
    // Usage
    factorial(6); // 720
    
    

* * *

### 25\. Flatten a Nested Object Recursively

**Problem:** Flatten `{ a: { b: { c: 1 } } }` into `{ "a.b.c": 1 }`.

#### ✅ C#
    
    
    public static Dictionary<string, object> FlattenObject(
        Dictionary<string, object> obj, string prefix = "")
    {
        var result = new Dictionary<string, object>();
    
        foreach (var (key, value) in obj)
        {
            string fullKey = prefix == "" ? key : $"{prefix}.{key}";
    
            if (value is Dictionary<string, object> nested)
            {
                foreach (var kv in FlattenObject(nested, fullKey))
                    result[kv.Key] = kv.Value;
            }
            else
            {
                result[fullKey] = value;
            }
        }
    
        return result;
    }
    
    

#### ✅ JavaScript
    
    
    function flattenObject(obj, prefix = '') {
      return Object.entries(obj).reduce((acc, [key, val]) => {
        const fullKey = prefix ? `${prefix}.${key}` : key;
        if (typeof val === 'object' && val !== null && !Array.isArray(val)) {
          Object.assign(acc, flattenObject(val, fullKey));
        } else {
          acc[fullKey] = val;
        }
        return acc;
      }, {});
    }
    
    // Usage
    flattenObject({ a: { b: { c: 1 } }, d: 2 });
    // { "a.b.c": 1, "d": 2 }
    
    

* * *

### 26\. Binary Search (Recursive)

**Problem:** Search a sorted array for a target value, return its index or -1.

#### ✅ C#
    
    
    public static int BinarySearchRecursive(int[] arr, int target, int left, int right)
    {c
        if (left > right) return -1;
        int mid = left + (right - left) / 2;
    
        if (arr[mid] == target) return mid;
        if (arr[mid] < target)  return BinarySearchRecursive(arr, target, mid + 1, right);
        return BinarySearchRecursive(arr, target, left, mid - 1);
    }
    
    // Usage
    BinarySearchRecursive(new[] { 1, 3, 5, 7, 9 }, 7, 0, 4); // 3
    
    

#### ✅ JavaScript
    
    
    function binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
      if (left > right) return -1;
      const mid = Math.floor((left + right) / 2);
    
      if (arr[mid] === target) return mid;
      if (arr[mid] < target)   return binarySearchRecursive(arr, target, mid + 1, right);
      return binarySearchRecursive(arr, target, left, mid - 1);
    }
    
    // Usage
    binarySearchRecursive([1, 3, 5, 7, 9], 7); // 3
    
    

* * *

### 27\. Sum All Numbers in a Nested Array

**Problem:** Sum all numbers in an arbitrarily nested array.

#### ✅ C#
    
    
    public static int SumNested(object[] arr)
    {
        int total = 0;
        foreach (var item in arr)
        {
            if (item is int n)        total += n;
            else if (item is object[] nested) total += SumNested(nested);
        }
        return total;
    }
    
    // Usage
    SumNested(new object[] { 1, new object[] { 2, 3 }, new object[] { 4, new object[] { 5 } } });
    // 15
    
    

#### ✅ JavaScript
    
    
    function sumNested(arr) {
      return arr.reduce((sum, val) =>
        sum + (Array.isArray(val) ? sumNested(val) : val), 0);
    }
    
    // Usage
    sumNested([1, [2, 3], [4, [5]]]); // 15
    
    

* * *

## 🔷 Sorting & Searching

* * *

### 28\. Bubble Sort

**Problem:** Implement bubble sort.

#### ✅ C#
    
    
    public static int[] BubbleSort(int[] arr)
    {
        int n = arr.Length;
        for (int i = 0; i < n - 1; i++)
            for (int j = 0; j < n - i - 1; j++)
                if (arr[j] > arr[j + 1])
                    (arr[j], arr[j + 1]) = (arr[j + 1], arr[j]);
        return arr;
    }
    
    // Usage
    BubbleSort(new[] { 64, 34, 25, 12, 22, 11, 90 });
    // [11, 12, 22, 25, 34, 64, 90]
    
    

#### ✅ JavaScript
    
    
    function bubbleSort(arr) {
      const a = [...arr];
      for (let i = 0; i < a.length - 1; i++)
        for (let j = 0; j < a.length - i - 1; j++)
          if (a[j] > a[j + 1]) [a[j], a[j + 1]] = [a[j + 1], a[j]];
      return a;
    }
    
    // Usage
    bubbleSort([64, 34, 25, 12, 22, 11, 90]);
    // [11, 12, 22, 25, 34, 64, 90]
    
    

* * *

### 29\. Merge Sort

**Problem:** Implement merge sort (divide and conquer).

#### ✅ C#
    
    
    public static int[] MergeSort(int[] arr)
    {
        if (arr.Length <= 1) return arr;
    
        int mid = arr.Length / 2;
        var left  = MergeSort(arr[..mid]);
        var right = MergeSort(arr[mid..]);
        return Merge(left, right);
    }
    
    private static int[] Merge(int[] left, int[] right)
    {
        var result = new List<int>();
        int i = 0, j = 0;
    
        while (i < left.Length && j < right.Length)
            result.Add(left[i] <= right[j] ? left[i++] : right[j++]);
    
        result.AddRange(left[i..]);
        result.AddRange(right[j..]);
        return result.ToArray();
    }
    
    // Usage
    MergeSort(new[] { 38, 27, 43, 3, 9 }); // [3, 9, 27, 38, 43]
    
    

#### ✅ JavaScript
    
    
    function mergeSort(arr) {
      if (arr.length <= 1) return arr;
    
      const mid   = Math.floor(arr.length / 2);
      const left  = mergeSort(arr.slice(0, mid));
      const right = mergeSort(arr.slice(mid));
    
      return merge(left, right);
    }
    
    function merge(left, right) {
      const result = [];
      let i = 0, j = 0;
    
      while (i < left.length && j < right.length)
        result.push(left[i] <= right[j] ? left[i++] : right[j++]);
    
      return result.concat(left.slice(i)).concat(right.slice(j));
    }
    
    // Usage
    mergeSort([38, 27, 43, 3, 9]); // [3, 9, 27, 38, 43]
    
    

**Key Insight:** O(n log n) time, O(n) space. Stable sort.

* * *

### 30\. Binary Search (Iterative)

**Problem:** Iterative binary search on a sorted array.

#### ✅ C#
    
    
    public static int BinarySearch(int[] arr, int target)
    {
        int left = 0, right = arr.Length - 1;
    
        while (left <= right)
        {
            int mid = left + (right - left) / 2;
            if (arr[mid] == target) return mid;
            if (arr[mid] < target)  left = mid + 1;
            else                    right = mid - 1;
        }
    
        return -1;
    }
    
    // Usage
    BinarySearch(new[] { 1, 3, 5, 7, 9, 11 }, 7); // 3
    
    

#### ✅ JavaScript
    
    
    function binarySearch(arr, target) {
      let left = 0, right = arr.length - 1;
    
      while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        if (arr[mid] === target) return mid;
        if (arr[mid] < target)   left = mid + 1;
        else                     right = mid - 1;
      }
    
      return -1;
    }
    
    // Usage
    binarySearch([1, 3, 5, 7, 9, 11], 7); // 3
    
    

* * *

### 31\. Find First and Last Position of Element

**Problem:** Return the first and last index of a target in a sorted array.

#### ✅ C#
    
    
    public static int[] FirstAndLast(int[] nums, int target)
    {
        return new[] { FindBound(nums, target, true), FindBound(nums, target, false) };
    }
    
    private static int FindBound(int[] nums, int target, bool findFirst)
    {
        int left = 0, right = nums.Length - 1, result = -1;
    
        while (left <= right)
        {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target)
            {
                result = mid;
                if (findFirst) right = mid - 1;
                else           left  = mid + 1;
            }
            else if (nums[mid] < target) left  = mid + 1;
            else                         right = mid - 1;
        }
    
        return result;
    }
    
    // Usage
    FirstAndLast(new[] { 5, 7, 7, 8, 8, 10 }, 8); // [3, 4]
    
    

#### ✅ JavaScript
    
    
    function firstAndLast(nums, target) {
      const findBound = (findFirst) => {
        let left = 0, right = nums.length - 1, result = -1;
        while (left <= right) {
          const mid = Math.floor((left + right) / 2);
          if (nums[mid] === target) {
            result = mid;
            if (findFirst) right = mid - 1;
            else           left  = mid + 1;
          } else if (nums[mid] < target) left  = mid + 1;
          else                           right = mid - 1;
        }
        return result;
      };
      return [findBound(true), findBound(false)];
    }
    
    // Usage
    firstAndLast([5, 7, 7, 8, 8, 10], 8); // [3, 4]
    
    

* * *

## 🔷 Linked Lists

* * *

### 32\. Reverse a Linked List

**Problem:** Reverse a singly linked list in place.

#### ✅ C#
    
    
    public class ListNode
    {
        public int Val;
        public ListNode? Next;
        public ListNode(int val) { Val = val; }
    }
    
    public static ListNode? ReverseList(ListNode? head)
    {
        ListNode? prev = null;
        var curr = head;
    
        while (curr != null)
        {
            var next = curr.Next;
            curr.Next = prev;
            prev = curr;
            curr = next;
        }
    
        return prev;
    }
    
    

#### ✅ JavaScript
    
    
    class ListNode {
      constructor(val) {
        this.val = val;
        this.next = null;
      }
    }
    
    function reverseList(head) {
      let prev = null;
      let curr = head;
    
      while (curr) {
        const next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
      }
    
      return prev;
    }
    
    

* * *

### 33\. Detect a Cycle in a Linked List

**Problem:** Return `true` if the linked list has a cycle (Floyd's algorithm).

#### ✅ C#
    
    
    public static bool HasCycle(ListNode? head)
    {
        var slow = head;
        var fast = head;
    
        while (fast?.Next != null)
        {
            slow = slow!.Next;
            fast = fast.Next.Next;
            if (slow == fast) return true;
        }
    
        return false;
    }
    
    

#### ✅ JavaScript
    
    
    function hasCycle(head) {
      let slow = head;
      let fast = head;
    
      while (fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow === fast) return true;
      }
    
      return false;
    }
    
    

**Key Insight:** Floyd's Tortoise and Hare — if there's a cycle, fast will lap slow.

* * *

### 34\. Merge Two Sorted Linked Lists

**Problem:** Merge two sorted linked lists into one sorted list.

#### ✅ C#
    
    
    public static ListNode? MergeTwoLists(ListNode? l1, ListNode? l2)
    {
        var dummy = new ListNode(0);
        var curr  = dummy;
    
        while (l1 != null && l2 != null)
        {
            if (l1.Val <= l2.Val) { curr.Next = l1; l1 = l1.Next; }
            else                  { curr.Next = l2; l2 = l2.Next; }
            curr = curr.Next;
        }
    
        curr.Next = l1 ?? l2;
        return dummy.Next;
    }
    
    

#### ✅ JavaScript
    
    
    function mergeTwoLists(l1, l2) {
      const dummy = new ListNode(0);
      let curr = dummy;
    
      while (l1 && l2) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else                  { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
      }
    
      curr.next = l1 || l2;
      return dummy.next;
    }
    
    

* * *

### 35\. Remove Nth Node From End

**Problem:** Remove the nth node from the end of the list in one pass.

#### ✅ C#
    
    
    public static ListNode? RemoveNthFromEnd(ListNode? head, int n)
    {
        var dummy = new ListNode(0) { Next = head };
        var fast  = dummy;
        var slow  = dummy;
    
        for (int i = 0; i <= n; i++)
            fast = fast.Next!;
    
        while (fast != null)
        {
            fast = fast.Next;
            slow = slow.Next!;
        }
    
        slow.Next = slow.Next?.Next;
        return dummy.Next;
    }
    
    

#### ✅ JavaScript
    
    
    function removeNthFromEnd(head, n) {
      const dummy = new ListNode(0);
      dummy.next = head;
      let fast = dummy, slow = dummy;
    
      for (let i = 0; i <= n; i++) fast = fast.next;
    
      while (fast) {
        fast = fast.next;
        slow = slow.next;
      }
    
      slow.next = slow.next.next;
      return dummy.next;
    }
    
    

* * *

## 🔷 Trees

* * *

### 36\. Inorder Traversal (Iterative)

**Problem:** Return inorder traversal of a binary tree without recursion.

#### ✅ C#
    
    
    public class TreeNode
    {
        public int Val;
        public TreeNode? Left, Right;
        public TreeNode(int val) { Val = val; }
    }
    
    public static IList<int> InorderTraversal(TreeNode? root)
    {
        var result = new List<int>();
        var stack  = new Stack<TreeNode>();
        var curr   = root;
    
        while (curr != null || stack.Count > 0)
        {
            while (curr != null) { stack.Push(curr); curr = curr.Left; }
            curr = stack.Pop();
            result.Add(curr.Val);
            curr = curr.Right;
        }
    
        return result;
    }
    
    

#### ✅ JavaScript
    
    
    function inorderTraversal(root) {
      const result = [], stack = [];
      let curr = root;
    
      while (curr || stack.length) {
        while (curr) { stack.push(curr); curr = curr.left; }
        curr = stack.pop();
        result.push(curr.val);
        curr = curr.right;
      }
    
      return result;
    }
    
    

* * *

### 37\. Maximum Depth of Binary Tree

**Problem:** Return the height (max depth) of a binary tree.

#### ✅ C#
    
    
    public static int MaxDepth(TreeNode? root)
    {
        if (root == null) return 0;
        return 1 + Math.Max(MaxDepth(root.Left), MaxDepth(root.Right));
    }
    
    // Usage — tree with 3 levels returns 3
    
    

#### ✅ JavaScript
    
    
    function maxDepth(root) {
      if (!root) return 0;
      return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
    }
    
    

* * *

### 38\. Level Order Traversal (BFS)

**Problem:** Return nodes level by level (breadth-first).

#### ✅ C#
    
    
    public static IList<IList<int>> LevelOrder(TreeNode? root)
    {
        var result = new List<IList<int>>();
        if (root == null) return result;
    
        var queue = new Queue<TreeNode>();
        queue.Enqueue(root);
    
        while (queue.Count > 0)
        {
            var level = new List<int>();
            int size  = queue.Count;
    
            for (int i = 0; i < size; i++)
            {
                var node = queue.Dequeue();
                level.Add(node.Val);
                if (node.Left  != null) queue.Enqueue(node.Left);
                if (node.Right != null) queue.Enqueue(node.Right);
            }
    
            result.Add(level);
        }
    
        return result;
    }
    
    

#### ✅ JavaScript
    
    
    function levelOrder(root) {
      if (!root) return [];
      const result = [], queue = [root];
    
      while (queue.length) {
        const level = [], size = queue.length;
        for (let i = 0; i < size; i++) {
          const node = queue.shift();
          level.push(node.val);
          if (node.left)  queue.push(node.left);
          if (node.right) queue.push(node.right);
        }
        result.push(level);
      }
    
      return result;
    }
    
    

* * *

### 39\. Lowest Common Ancestor

**Problem:** Find the lowest common ancestor of two nodes in a BST.

#### ✅ C#
    
    
    public static TreeNode? LowestCommonAncestor(TreeNode? root, TreeNode p, TreeNode q)
    {
        if (root == null) return null;
        if (p.Val < root.Val && q.Val < root.Val) return LowestCommonAncestor(root.Left, p, q);
        if (p.Val > root.Val && q.Val > root.Val) return LowestCommonAncestor(root.Right, p, q);
        return root;
    }
    
    

#### ✅ JavaScript
    
    
    function lowestCommonAncestor(root, p, q) {
      if (!root) return null;
      if (p.val < root.val && q.val < root.val) return lowestCommonAncestor(root.left, p, q);
      if (p.val > root.val && q.val > root.val) return lowestCommonAncestor(root.right, p, q);
      return root;
    }
    
    

* * *

## 🔷 Dynamic Programming

* * *

### 40\. Climbing Stairs

**Problem:** You can climb 1 or 2 steps at a time. How many distinct ways to reach step `n`?

#### ✅ C#
    
    
    public static int ClimbStairs(int n)
    {
        if (n <= 2) return n;
        int prev2 = 1, prev1 = 2;
    
        for (int i = 3; i <= n; i++)
        {
            int curr = prev1 + prev2;
            prev2 = prev1;
            prev1 = curr;
        }
    
        return prev1;
    }
    
    // Usage
    ClimbStairs(5); // 8
    
    

#### ✅ JavaScript
    
    
    function climbStairs(n) {
      if (n <= 2) return n;
      let prev2 = 1, prev1 = 2;
    
      for (let i = 3; i <= n; i++) {
        [prev2, prev1] = [prev1, prev1 + prev2];
      }
    
      return prev1;
    }
    
    // Usage
    climbStairs(5); // 8
    
    

**Key Insight:** This is Fibonacci in disguise. O(n) time, O(1) space with two variables.

* * *

### 41\. Longest Common Subsequence

**Problem:** Find the length of the longest common subsequence of two strings.

#### ✅ C#
    
    
    public static int LongestCommonSubsequence(string text1, string text2)
    {
        int m = text1.Length, n = text2.Length;
        var dp = new int[m + 1, n + 1];
    
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                dp[i, j] = text1[i - 1] == text2[j - 1]
                    ? dp[i - 1, j - 1] + 1
                    : Math.Max(dp[i - 1, j], dp[i, j - 1]);
    
        return dp[m, n];
    }
    
    // Usage
    LongestCommonSubsequence("abcde", "ace"); // 3
    
    

#### ✅ JavaScript
    
    
    function longestCommonSubsequence(text1, text2) {
      const m = text1.length, n = text2.length;
      const dp = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));
    
      for (let i = 1; i <= m; i++)
        for (let j = 1; j <= n; j++)
          dp[i][j] = text1[i - 1] === text2[j - 1]
            ? dp[i - 1][j - 1] + 1
            : Math.max(dp[i - 1][j], dp[i][j - 1]);
    
      return dp[m][n];
    }
    
    // Usage
    longestCommonSubsequence("abcde", "ace"); // 3
    
    

* * *

### 42\. Coin Change

**Problem:** Find the minimum number of coins needed to make up a given amount.

#### ✅ C#
    
    
    public static int CoinChange(int[] coins, int amount)
    {
        var dp = new int[amount + 1];
        Array.Fill(dp, amount + 1);
        dp[0] = 0;
    
        for (int i = 1; i <= amount; i++)
            foreach (int coin in coins)
                if (coin <= i)
                    dp[i] = Math.Min(dp[i], dp[i - coin] + 1);
    
        return dp[amount] > amount ? -1 : dp[amount];
    }
    
    // Usage
    CoinChange(new[] { 1, 5, 6, 9 }, 11); // 2  (coins: 5+6)
    
    

#### ✅ JavaScript
    
    
    function coinChange(coins, amount) {
      const dp = new Array(amount + 1).fill(amount + 1);
      dp[0] = 0;
    
      for (let i = 1; i <= amount; i++)
        for (const coin of coins)
          if (coin <= i)
            dp[i] = Math.min(dp[i], dp[i - coin] + 1);
    
      return dp[amount] > amount ? -1 : dp[amount];
    }
    
    // Usage
    coinChange([1, 5, 6, 9], 11); // 2
    
    

* * *

### 43\. 0-1 Knapsack

**Problem:** Maximize value without exceeding weight capacity; each item can be taken once.

#### ✅ C#
    
    
    public static int Knapsack(int[] weights, int[] values, int capacity)
    {
        int n = weights.Length;
        var dp = new int[n + 1, capacity + 1];
    
        for (int i = 1; i <= n; i++)
            for (int w = 0; w <= capacity; w++)
            {
                dp[i, w] = dp[i - 1, w];
                if (weights[i - 1] <= w)
                    dp[i, w] = Math.Max(dp[i, w], dp[i - 1, w - weights[i - 1]] + values[i - 1]);
            }
    
        return dp[n, capacity];
    }
    
    // Usage
    Knapsack(new[] { 1, 3, 4, 5 }, new[] { 1, 4, 5, 7 }, 7); // 9
    
    

#### ✅ JavaScript
    
    
    function knapsack(weights, values, capacity) {
      const n = weights.length;
      const dp = Array.from({ length: n + 1 }, () => new Array(capacity + 1).fill(0));
    
      for (let i = 1; i <= n; i++)
        for (let w = 0; w <= capacity; w++) {
          dp[i][w] = dp[i - 1][w];
          if (weights[i - 1] <= w)
            dp[i][w] = Math.max(dp[i][w], dp[i - 1][w - weights[i - 1]] + values[i - 1]);
        }
    
      return dp[n][capacity];
    }
    
    // Usage
    knapsack([1, 3, 4, 5], [1, 4, 5, 7], 7); // 9
    
    

* * *

## 🔷 Math & Logic

* * *

### 44\. FizzBuzz

**Problem:** Print numbers 1–n; "Fizz" for multiples of 3, "Buzz" for 5, "FizzBuzz" for both.

#### ✅ C#
    
    
    public static IEnumerable<string> FizzBuzz(int n)
    {
        for (int i = 1; i <= n; i++)
        {
            if (i % 15 == 0) yield return "FizzBuzz";
            else if (i % 3 == 0) yield return "Fizz";
            else if (i % 5 == 0) yield return "Buzz";
            else yield return i.ToString();
        }
    }
    
    // Usage
    FizzBuzz(15).ToList();
    
    

#### ✅ JavaScript
    
    
    function fizzBuzz(n) {
      return Array.from({ length: n }, (_, i) => {
        i += 1;
        if (i % 15 === 0) return 'FizzBuzz';
        if (i % 3  === 0) return 'Fizz';
        if (i % 5  === 0) return 'Buzz';
        return String(i);
      });
    }
    
    // Usage
    fizzBuzz(15);
    
    

* * *

### 45\. Power of Two

**Problem:** Return `true` if a number is a power of two.

#### ✅ C#
    
    
    public static bool IsPowerOfTwo(int n)
    {
        return n > 0 && (n & (n - 1)) == 0;
    }c
    
    // Usage
    IsPowerOfTwo(16); // true
    IsPowerOfTwo(18); // false
    
    

#### ✅ JavaScript
    
    
    function isPowerOfTwo(n) {
      return n > 0 && (n & (n - 1)) === 0;
    }
    
    // Usage
    isPowerOfTwo(16); // true
    isPowerOfTwo(18); // false
    
    

**Key Insight:** A power of two in binary has exactly one `1` bit. `n & (n-1)` clears the lowest set bit.

* * *

### 46\. Check if a Number is Prime

**Problem:** Return `true` if a number is prime.

#### ✅ C#
    
    
    public static bool IsPrime(int n)
    {
        if (n < 2) return false;
        if (n == 2) return true;
        if (n % 2 == 0) return false;
    
        for (int i = 3; i <= Math.Sqrt(n); i += 2)
            if (n % i == 0) return false;
    
        return true;
    }
    
    // Usage
    IsPrime(17); // true
    IsPrime(18); // false
    
    

#### ✅ JavaScript
    
    
    function isPrime(n) {
      if (n < 2) return false;
      if (n === 2) return true;
      if (n % 2 === 0) return false;
    
      for (let i = 3; i <= Math.sqrt(n); i += 2)
        if (n % i === 0) return false;
    
      return true;
    }
    
    // Usage
    isPrime(17); // true
    
    

**Key Insight:** Only check divisors up to √n. Skip even numbers after checking 2.

* * *

### 47\. Roman Numerals to Integer

**Problem:** Convert a Roman numeral string to an integer.

#### ✅ C#
    
    
    public static int RomanToInt(string s)
    {
        var map = new Dictionary<char, int>
        {
            ['I'] = 1,   ['V'] = 5,   ['X'] = 10,
            ['L'] = 50,  ['C'] = 100, ['D'] = 500, ['M'] = 1000
        };
    
        int result = 0;
        for (int i = 0; i < s.Length; i++)
        {
            int curr = map[s[i]];
            int next = i + 1 < s.Length ? map[s[i + 1]] : 0;
            result += curr < next ? -curr : curr;
        }
    
        return result;
    }
    
    // Usage
    RomanToInt("MCMXCIV"); // 1994
    
    

#### ✅ JavaScript
    
    
    function romanToInt(s) {
      const map = { I: 1, V: 5, X: 10, L: 50, C: 100, D: 500, M: 1000 };
      let result = 0;
    
      for (let i = 0; i < s.length; i++) {
        const curr = map[s[i]];
        const next = map[s[i + 1]] || 0;
        result += curr < next ? -curr : curr;
      }
    
      return result;
    }
    
    // Usage
    romanToInt("MCMXCIV"); // 1994
    
    

* * *

## 🔷 Async & Functional (JS) / LINQ (C#)

* * *

### 48\. Chaining Promises / async-await

**Problem:** Fetch user data, then fetch their posts, and return combined results.

#### ✅ C# (HttpClient + async/await)
    
    
    public async Task<(User user, IEnumerable<Post> posts)> GetUserWithPosts(int userId)
    {
        using var client = new HttpClient();
        client.BaseAddress = new Uri("https://jsonplaceholder.typicode.com");
    
        var userJson  = await client.GetStringAsync($"/users/{userId}");
        var user      = JsonSerializer.Deserialize<User>(userJson)!;
    
        var postsJson = await client.GetStringAsync($"/posts?userId={userId}");
        var posts     = JsonSerializer.Deserialize<IEnumerable<Post>>(postsJson)!;
    
        return (user, posts);
    }
    
    

#### ✅ JavaScript (Promise chaining + async/await)
    
    
    // Promise chain version
    function getUserWithPosts(userId) {
      return fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
        .then(res => res.json())
        .then(user =>
          fetch(`https://jsonplaceholder.typicode.com/posts?userId=${userId}`)
            .then(res => res.json())
            .then(posts => ({ user, posts }))
        );
    }
    
    // async/await version (cleaner)
    async function getUserWithPostsAsync(userId) {
      const user  = await fetch(`/users/${userId}`).then(r => r.json());
      const posts = await fetch(`/posts?userId=${userId}`).then(r => r.json());
      return { user, posts };
    }
    
    

* * *

### 49\. Retry with Exponential Backoff

**Problem:** Retry a failing async operation up to `n` times with exponential delay.

#### ✅ C#
    
    
    public static async Task<T> RetryWithBackoff<T>(
        Func<Task<T>> operation, int maxRetries = 3)
    {
        int delay = 100;
    
        for (int attempt = 0; attempt <= maxRetries; attempt++)
        {
            try
            {
                return await operation();
            }
            catch when (attempt < maxRetries)
            {
                await Task.Delay(delay);
                delay *= 2;
            }
        }
    
        throw new Exception("All retries exhausted");
    }
    
    // Usage
    var result = await RetryWithBackoff(() => FetchDataAsync(), maxRetries: 3);
    
    

#### ✅ JavaScript
    
    
    async function retryWithBackoff(fn, maxRetries = 3, delay = 100) {
      for (let attempt = 0; attempt <= maxRetries; attempt++) {
        try {
          return await fn();
        } catch (err) {
          if (attempt === maxRetries) throw err;
          await new Promise(res => setTimeout(res, delay * 2 ** attempt));
        }
      }
    }
    
    // Usage
    const data = await retryWithBackoff(() => fetch('/api/data').then(r => r.json()));
    
    

* * *

### 50\. LINQ / Functional Array Pipelines

**Problem:** From a list of orders, find the top 3 customers by total spend (over $100), sorted descending.

#### ✅ C#
    
    
    var orders = new[]
    {
        new { Customer = "Alice", Amount = 120.0 },
        new { Customer = "Bob",   Amount = 85.0  },
        new { Customer = "Alice", Amount = 60.0  },
        new { Customer = "Carol", Amount = 200.0 },
        new { Customer = "Bob",   Amount = 130.0 }
    };
    
    var top3 = orders
        .GroupBy(o => o.Customer)
        .Select(g => new { Customer = g.Key, Total = g.Sum(o => o.Amount) })
        .Where(x => x.Total > 100)
        .OrderByDescending(x => x.Total)
        .Take(3)
        .ToList();
    
    // [{ Carol, 200 }, { Bob, 215 }, { Alice, 180 }]
    
    

#### ✅ JavaScript
    
    
    const orders = [
      { customer: 'Alice', amount: 120 },
      { customer: 'Bob',   amount: 85  },
      { customer: 'Alice', amount: 60  },
      { customer: 'Carol', amount: 200 },
      { customer: 'Bob',   amount: 130 }
    ];
    
    const top3 = Object.entries(
      orders.reduce((acc, { customer, amount }) => {
        acc[customer] = (acc[customer] || 0) + amount;
        return acc;
      }, {})
    )
      .map(([customer, total]) => ({ customer, total }))
      .filter(x => x.total > 100)
      .sort((a, b) => b.total - a.total)
      .slice(0, 3);
    
    // [{ Bob, 215 }, { Alice, 180 }, { Carol, 200 }]
    
    

* * *

## ⚡ Quick Reference: Complexity Cheat Sheet

Algorithm| Time (Best)| Time (Avg)| Time (Worst)| Space  
---|---|---|---|---  
Bubble Sort| O(n)| O(n²)| O(n²)| O(1)  
Merge Sort| O(n log n)| O(n log n)| O(n log n)| O(n)  
Binary Search| O(1)| O(log n)| O(log n)| O(1)  
Hash Map Lookup| O(1)| O(1)| O(n)| O(n)  
DFS / BFS| O(V+E)| O(V+E)| O(V+E)| O(V)  
DP (Knapsack)| —| O(n×W)| O(n×W)| O(n×W)  
Fibonacci (memo)| —| O(n)| O(n)| O(n)  
  
* * *

## 🏷️ Category Summary

Category| Questions| Core Concepts  
---|---|---  
Arrays| 1–8| HashSet, Two-pointer, Kadane, Sliding window  
Strings| 9–16| Hash map, Stack, Sliding window, Two-pointer  
Objects & Classes| 17–22| Deep clone, Recursion, Encapsulation  
Recursion| 23–27| Memoization, Base cases, Divide & conquer  
Sorting & Searching| 28–31| Comparison, Divide & conquer, Binary search  
Linked Lists| 32–35| Two-pointer, Dummy node, Floyd's cycle  
Trees| 36–39| DFS, BFS, Recursion  
Dynamic Programming| 40–43| Bottom-up DP, Optimal substructure  
Math & Logic| 44–47| Bit manipulation, Math tricks  
Async & Functional| 48–50| Promises, LINQ, Pipelines
