Below is the **final Python tutorial I would use if the goal is FAANG+ / staff-level DSA interview performance**, not general Python fluency.

The CS50P transcript is a strong Python foundation, but it teaches Python broadly: functions, variables, conditionals, loops, exceptions, libraries, tests, file I/O, regex, OOP, and advanced topics. For FAANG+ DSA, we compress that into the subset that actually wins coding rounds: **clean functions, correct data structures, algorithmic patterns, complexity analysis, edge cases, and communication.** 

Official interview prep guidance from Amazon explicitly says interviewers are looking for your ability to **apply** computer science fundamentals efficiently, not memorize details; their listed technical topics include programming language, data structures, algorithms, coding, OOD, databases, and distributed computing. ([Amazon.jobs][1]) For senior/staff-like roles, Amazon’s SDE III guidance emphasizes leadership, system-wide thinking, high-performance scalable systems, and exemplary, maintainable, extensible code. ([Amazon.jobs][2]) So this tutorial is written for that bar.

# Python for FAANG+ DSA Interviews

## 0. The Interview Mental Model

In DSA interviews, Python is not the subject. Python is the weapon.

Your real job is:

```text
Understand problem
  -> choose correct pattern
  -> choose correct data structure
  -> prove correctness
  -> write clean code
  -> test edge cases
  -> analyze time and space
  -> discuss tradeoffs
```

Staff-level expectation:

```text
Junior: can code a solution
Mid-level: can code optimal solution
Senior: can explain tradeoffs and edge cases
Staff: can reason like a system owner under ambiguity
```

During a coding interview, your answer should sound like this:

```text
First I’ll clarify constraints.
Then I’ll propose a brute-force solution.
Then I’ll optimize using the key observation.
The core invariant is...
The algorithm is...
The complexity is...
Now I’ll code it.
Finally I’ll test edge cases.
```

That is the framework.

---

# 1. Python You Actually Need

## 1.1 Variables and Types

You need these types constantly:

```python
int
float
str
bool
list
tuple
dict
set
None
```

Interview examples:

```python
x = 10
name = "abc"
seen = set()
freq = {}
arr = [1, 2, 3]
point = (2, 5)
```

Use `None` for missing values:

```python
prev = None
```

Use booleans directly:

```python
if not arr:
    return 0
```

Do not write:

```python
if len(arr) == 0:
    return 0
```

Both work, but the first is idiomatic.

---

## 1.2 Functions Are the Unit of Interview Code

Most platforms expect:

```python
class Solution:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        ...
```

But internally, think in small functions:

```python
def solve(nums: list[int]) -> int:
    ...
```

A good function:

```python
def max_subarray_sum(nums: list[int]) -> int:
    best = curr = nums[0]

    for x in nums[1:]:
        curr = max(x, curr + x)
        best = max(best, curr)

    return best
```

A bad function:

```python
def f(a):
    x = 0
    # unclear logic
```

For FAANG+ interviews, names matter. Use:

```python
left, right
slow, fast
start, end
curr, best
count, freq
parent, rank
graph, indegree
```

---

## 1.3 Loops

Use `for` when you know the collection:

```python
for x in nums:
    ...
```

Use `range` when you need indices:

```python
for i in range(len(nums)):
    ...
```

Use `enumerate` when you need both index and value:

```python
for i, x in enumerate(nums):
    ...
```

Use `while` for pointer movement:

```python
left = 0
right = len(nums) - 1

while left < right:
    ...
```

Use nested loops only when `O(n²)` is acceptable or unavoidable:

```python
for i in range(n):
    for j in range(i + 1, n):
        ...
```

---

## 1.4 Conditions

Write direct conditions:

```python
if x in seen:
    return True
```

Use chained comparisons:

```python
if 0 <= r < rows and 0 <= c < cols:
    ...
```

Use early returns to reduce nesting:

```python
def is_valid(s: str) -> bool:
    if not s:
        return False

    if len(s) < 3:
        return False

    return True
```

Avoid:

```python
def is_valid(s):
    if s:
        if len(s) >= 3:
            return True
        else:
            return False
    else:
        return False
```

---

# 2. Python Data Structures: Interview Cheat Sheet

Python’s `collections` module provides specialized containers like `deque`, `Counter`, and `defaultdict`, which are extremely common in DSA problems. The official docs describe `deque` as supporting fast appends and pops on both ends, `Counter` as a dict subclass for counting hashable objects, and `defaultdict` as a dict subclass that supplies missing values automatically. ([Python documentation][3])

## 2.1 `list`

Use for:

```text
arrays
stacks
dynamic programming tables
adjacency lists
```

Examples:

```python
arr = []
arr.append(10)
arr.pop()
arr[-1]
```

Common operations:

```python
nums[i]          # access
nums.append(x)   # push end
nums.pop()       # pop end
nums[::-1]       # reversed copy
nums.sort()      # sort in place
sorted(nums)     # sorted copy
```

Important:

```python
nums.pop(0)      # bad for queues
nums.insert(0,x) # bad for front insertion
```

Python lists are array-backed; inserting or deleting near the front is expensive because elements must move. The Python time-complexity reference notes that list append and pop-last are constant time on average, while inserts and intermediate pops are linear. ([Python Wiki][4])

Use a list as a stack:

```python
stack = []

stack.append(x)
top = stack[-1]
stack.pop()
```

---

## 2.2 `dict`

Use for:

```text
hash maps
frequency maps
index lookup
memoization
graph adjacency
```

Example:

```python
freq = {}

for x in nums:
    freq[x] = freq.get(x, 0) + 1
```

Membership:

```python
if x in freq:
    ...
```

Index map:

```python
last_seen = {}

for i, x in enumerate(nums):
    last_seen[x] = i
```

---

## 2.3 `set`

Use for:

```text
visited nodes
duplicate detection
O(1) average membership
```

Example:

```python
seen = set()

for x in nums:
    if x in seen:
        return True
    seen.add(x)

return False
```

Graph visited:

```python
visited = set()

def dfs(node):
    if node in visited:
        return
    visited.add(node)
```

---

## 2.4 `tuple`

Use for immutable coordinates or keys:

```python
cell = (r, c)
visited.add((r, c))
```

You cannot put a list inside a set:

```python
visited.add([r, c])  # TypeError
```

Use tuple instead:

```python
visited.add((r, c))
```

---

## 2.5 `deque`

Use for queues and BFS:

```python
from collections import deque

q = deque()
q.append(start)
node = q.popleft()
```

Never use `list.pop(0)` for BFS.

BFS template:

```python
from collections import deque

def bfs(start):
    q = deque([start])
    visited = {start}

    while q:
        node = q.popleft()

        for nei in graph[node]:
            if nei not in visited:
                visited.add(nei)
                q.append(nei)
```

---

## 2.6 `Counter`

Use for frequency counting:

```python
from collections import Counter

freq = Counter(nums)
```

Example:

```python
from collections import Counter

def is_anagram(s: str, t: str) -> bool:
    return Counter(s) == Counter(t)
```

Manual version:

```python
def is_anagram(s: str, t: str) -> bool:
    if len(s) != len(t):
        return False

    freq = {}

    for ch in s:
        freq[ch] = freq.get(ch, 0) + 1

    for ch in t:
        if ch not in freq:
            return False
        freq[ch] -= 1
        if freq[ch] < 0:
            return False

    return True
```

For interviews, both are acceptable. If the interviewer wants fundamentals, explain the manual map.

---

## 2.7 `defaultdict`

Use for graph adjacency and grouping:

```python
from collections import defaultdict

graph = defaultdict(list)

for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)
```

Frequency:

```python
from collections import defaultdict

freq = defaultdict(int)

for x in nums:
    freq[x] += 1
```

Grouping anagrams:

```python
from collections import defaultdict

def group_anagrams(words: list[str]) -> list[list[str]]:
    groups = defaultdict(list)

    for word in words:
        key = tuple(sorted(word))
        groups[key].append(word)

    return list(groups.values())
```

---

## 2.8 `heapq`

Python’s `heapq` module implements a heap / priority queue. The official docs state that Python’s heap is a min-heap where `heap[0]` is the smallest item, and `heapify` transforms a list into a heap in linear time. ([Python documentation][5])

Basic usage:

```python
import heapq

heap = []
heapq.heappush(heap, 5)
heapq.heappush(heap, 2)
heapq.heappush(heap, 9)

smallest = heapq.heappop(heap)  # 2
```

Max heap trick:

```python
heapq.heappush(heap, -x)
largest = -heapq.heappop(heap)
```

Heap with pairs:

```python
heapq.heappush(heap, (distance, node))
```

Used for:

```text
top k
merge k sorted lists
Dijkstra
scheduling
median stream
priority simulation
```

---

## 2.9 `bisect`

The `bisect` module uses binary search to locate insertion points in sorted lists. The official docs describe it as support for maintaining a list in sorted order and locating insertion points. ([Python documentation][6])

Usage:

```python
from bisect import bisect_left, bisect_right

arr = [1, 2, 2, 2, 5]

left = bisect_left(arr, 2)    # 1
right = bisect_right(arr, 2)  # 4
count = right - left          # 3
```

Use when:

```text
array is sorted
you need count <= x
you need first >= x
you need first > x
```

---

## 2.10 `functools.cache`

Use for memoized recursion / top-down DP.

```python
from functools import cache

@cache
def dp(i, j):
    ...
```

The official `functools` docs describe `cache` as a lightweight unbounded function cache, equivalent to `lru_cache(maxsize=None)`. ([Python documentation][7])

Example:

```python
from functools import cache

def climb_stairs(n: int) -> int:
    @cache
    def dp(i: int) -> int:
        if i <= 1:
            return 1
        return dp(i - 1) + dp(i - 2)

    return dp(n)
```

---

# 3. Complexity: What You Must Be Able to Say

Every solution needs:

```text
Time: how many operations?
Space: extra memory beyond input?
```

Common complexities:

```text
O(1)        constant
O(log n)    binary search
O(n)        one pass
O(n log n)  sorting
O(n²)       pair checking / nested loops
O(2^n)      subsets / brute-force recursion
O(n!)       permutations
```

## Python-Specific Complexity You Must Know

```text
list append              O(1) amortized
list pop last            O(1)
list pop front           O(n)
dict lookup              O(1) average
set lookup               O(1) average
sort                     O(n log n)
heap push/pop            O(log n)
BFS/DFS graph            O(V + E)
```

Say complexity precisely:

```text
Let n be the number of elements.
We scan once and use a hash set, so time is O(n), space is O(n).
```

For grids:

```text
Let R be rows and C be columns.
We visit each cell once, so time is O(RC), space is O(RC).
```

For graphs:

```text
Let V be vertices and E be edges.
DFS/BFS is O(V + E).
```

---

# 4. Interview Execution Protocol

Use this exact order.

## Step 1: Clarify

Ask:

```text
Can input be empty?
Can values be negative?
Are duplicates allowed?
Should I return indices or values?
Is the input sorted?
Can I modify the input?
What are the constraints?
```

## Step 2: Give Brute Force

Example:

```text
Brute force checks every pair, O(n²). 
We can optimize by using a hash map to remember complements.
```

## Step 3: State Key Observation

Example:

```text
For each number x, I need target - x. 
If I have seen target - x before, I found the pair.
```

## Step 4: Code Cleanly

Do not rush.

## Step 5: Test

Use:

```text
normal case
empty case
single element
duplicates
negative numbers
large case
```

## Step 6: Analyze Complexity

Always finish with time and space.

---

# 5. Core Pattern 1: Hash Map / Hash Set

## Problem Type

Use when you need:

```text
fast lookup
duplicate detection
frequency counting
complement search
grouping
```

## Template: Seen Set

```python
def contains_duplicate(nums: list[int]) -> bool:
    seen = set()

    for x in nums:
        if x in seen:
            return True
        seen.add(x)

    return False
```

Time: `O(n)`
Space: `O(n)`

## Template: Two Sum

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    seen = {}

    for i, x in enumerate(nums):
        need = target - x

        if need in seen:
            return [seen[need], i]

        seen[x] = i

    return []
```

Key invariant:

```text
Before processing index i, seen contains values from indices < i.
```

## Template: Frequency Map

```python
def first_unique_char(s: str) -> int:
    freq = {}

    for ch in s:
        freq[ch] = freq.get(ch, 0) + 1

    for i, ch in enumerate(s):
        if freq[ch] == 1:
            return i

    return -1
```

---

# 6. Core Pattern 2: Two Pointers

## Problem Type

Use when:

```text
array/string is sorted
you need pair/triplet
you need remove/partition in place
you need compare from both ends
```

## Template: Opposite Ends

```python
def two_sum_sorted(nums: list[int], target: int) -> list[int]:
    left = 0
    right = len(nums) - 1

    while left < right:
        total = nums[left] + nums[right]

        if total == target:
            return [left, right]
        elif total < target:
            left += 1
        else:
            right -= 1

    return []
```

Why it works:

```text
If sum is too small, increasing left is the only useful move.
If sum is too large, decreasing right is the only useful move.
```

## Template: Palindrome

```python
def is_palindrome(s: str) -> bool:
    left = 0
    right = len(s) - 1

    while left < right:
        if s[left] != s[right]:
            return False

        left += 1
        right -= 1

    return True
```

## Template: Remove Duplicates from Sorted Array

```python
def remove_duplicates(nums: list[int]) -> int:
    if not nums:
        return 0

    write = 1

    for read in range(1, len(nums)):
        if nums[read] != nums[write - 1]:
            nums[write] = nums[read]
            write += 1

    return write
```

Invariant:

```text
nums[:write] contains the unique elements seen so far.
```

---

# 7. Core Pattern 3: Sliding Window

## Problem Type

Use when:

```text
contiguous subarray
contiguous substring
longest/shortest window
at most k
exactly k
sum/frequency inside a moving window
```

## Fixed Window

Example: maximum sum of size `k`.

```python
def max_sum_k(nums: list[int], k: int) -> int:
    window = sum(nums[:k])
    best = window

    for right in range(k, len(nums)):
        window += nums[right]
        window -= nums[right - k]
        best = max(best, window)

    return best
```

Time: `O(n)`
Space: `O(1)`

## Variable Window: Longest Substring Without Repeating Characters

```python
def length_of_longest_substring(s: str) -> int:
    seen = set()
    left = 0
    best = 0

    for right, ch in enumerate(s):
        while ch in seen:
            seen.remove(s[left])
            left += 1

        seen.add(ch)
        best = max(best, right - left + 1)

    return best
```

Invariant:

```text
The window s[left:right+1] always contains no duplicate characters.
```

## Variable Window: At Most K Distinct

```python
from collections import defaultdict

def longest_at_most_k_distinct(s: str, k: int) -> int:
    freq = defaultdict(int)
    left = 0
    best = 0

    for right, ch in enumerate(s):
        freq[ch] += 1

        while len(freq) > k:
            left_ch = s[left]
            freq[left_ch] -= 1

            if freq[left_ch] == 0:
                del freq[left_ch]

            left += 1

        best = max(best, right - left + 1)

    return best
```

---

# 8. Core Pattern 4: Prefix Sum

## Problem Type

Use when:

```text
subarray sum
range sum
number of subarrays with property
convert O(n²) range calculation to O(n)
```

## Prefix Sum Basics

```python
prefix = [0]

for x in nums:
    prefix.append(prefix[-1] + x)
```

Range sum `nums[l:r+1]`:

```python
prefix[r + 1] - prefix[l]
```

## Subarray Sum Equals K

```python
from collections import defaultdict

def subarray_sum(nums: list[int], k: int) -> int:
    count = defaultdict(int)
    count[0] = 1

    prefix = 0
    ans = 0

    for x in nums:
        prefix += x
        ans += count[prefix - k]
        count[prefix] += 1

    return ans
```

Key observation:

```text
current_prefix - previous_prefix = k
therefore previous_prefix = current_prefix - k
```

Time: `O(n)`
Space: `O(n)`

---

# 9. Core Pattern 5: Difference Array

## Problem Type

Use when:

```text
many range updates
apply +x from l to r many times
need final array
```

## Template

```python
def apply_range_updates(n: int, updates: list[tuple[int, int, int]]) -> list[int]:
    diff = [0] * (n + 1)

    for left, right, val in updates:
        diff[left] += val
        if right + 1 < n:
            diff[right + 1] -= val

    arr = [0] * n
    curr = 0

    for i in range(n):
        curr += diff[i]
        arr[i] = curr

    return arr
```

Instead of updating every element in `[l, r]`, mark only boundaries.

---

# 10. Core Pattern 6: Binary Search

## Problem Type

Use when:

```text
sorted array
monotonic condition
minimum feasible answer
maximum feasible answer
```

## Basic Binary Search

```python
def binary_search(nums: list[int], target: int) -> int:
    left = 0
    right = len(nums) - 1

    while left <= right:
        mid = (left + right) // 2

        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

## Lower Bound: First Index Where `nums[i] >= target`

```python
def lower_bound(nums: list[int], target: int) -> int:
    left = 0
    right = len(nums)

    while left < right:
        mid = (left + right) // 2

        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid

    return left
```

This is the one you should master.

## Binary Search on Answer

Use when answer space is numeric and feasibility is monotonic.

Example pattern:

```python
def minimize_answer(nums: list[int]) -> int:
    def feasible(x: int) -> bool:
        # return True if answer x works
        ...

    left = min_possible
    right = max_possible

    while left < right:
        mid = (left + right) // 2

        if feasible(mid):
            right = mid
        else:
            left = mid + 1

    return left
```

Examples:

```text
Koko Eating Bananas
Capacity to Ship Packages
Split Array Largest Sum
Minimum Days to Make Bouquets
```

Interview phrase:

```text
The feasibility function is monotonic: if capacity x works, any larger capacity also works.
```

---

# 11. Core Pattern 7: Sorting

## Problem Type

Use sorting when:

```text
order matters
intervals
greedy choice
deduplicate/group
two pointers after sorting
```

Python:

```python
nums.sort()
```

or:

```python
arr = sorted(nums)
```

Sort by key:

```python
intervals.sort(key=lambda x: x[0])
```

Sort descending:

```python
nums.sort(reverse=True)
```

Sort tuple naturally:

```python
pairs.sort()
```

This sorts by first element, then second.

## Merge Intervals

```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    intervals.sort(key=lambda x: x[0])

    merged = []

    for start, end in intervals:
        if not merged or start > merged[-1][1]:
            merged.append([start, end])
        else:
            merged[-1][1] = max(merged[-1][1], end)

    return merged
```

Invariant:

```text
merged contains non-overlapping intervals sorted by start.
```

Time: `O(n log n)` due to sorting.
Space: `O(n)` for output.

---

# 12. Core Pattern 8: Stack

## Problem Type

Use stack when:

```text
matching parentheses
undo/backtracking
monotonic next greater/smaller
path simplification
nested structures
```

## Valid Parentheses

```python
def is_valid_parentheses(s: str) -> bool:
    stack = []
    pairs = {
        ")": "(",
        "]": "[",
        "}": "{",
    }

    for ch in s:
        if ch in pairs.values():
            stack.append(ch)
        else:
            if not stack or stack[-1] != pairs[ch]:
                return False
            stack.pop()

    return not stack
```

Better version avoiding `pairs.values()` each time:

```python
def is_valid_parentheses(s: str) -> bool:
    stack = []
    closing_to_opening = {
        ")": "(",
        "]": "[",
        "}": "{",
    }

    openings = set(closing_to_opening.values())

    for ch in s:
        if ch in openings:
            stack.append(ch)
        else:
            if not stack:
                return False

            if stack.pop() != closing_to_opening[ch]:
                return False

    return not stack
```

---

# 13. Core Pattern 9: Monotonic Stack

## Problem Type

Use when:

```text
next greater element
next smaller element
stock span
daily temperatures
largest rectangle
remove digits
```

## Next Greater Element

```python
def next_greater(nums: list[int]) -> list[int]:
    ans = [-1] * len(nums)
    stack = []  # indices

    for i, x in enumerate(nums):
        while stack and nums[stack[-1]] < x:
            idx = stack.pop()
            ans[idx] = x

        stack.append(i)

    return ans
```

Invariant:

```text
Stack stores indices whose next greater element has not been found yet.
Values on stack are decreasing.
```

## Daily Temperatures

```python
def daily_temperatures(temperatures: list[int]) -> list[int]:
    ans = [0] * len(temperatures)
    stack = []

    for i, temp in enumerate(temperatures):
        while stack and temperatures[stack[-1]] < temp:
            prev = stack.pop()
            ans[prev] = i - prev

        stack.append(i)

    return ans
```

Time: `O(n)` because each index enters and leaves stack once.

---

# 14. Core Pattern 10: Queue / BFS

## Problem Type

Use BFS when:

```text
shortest path in unweighted graph
level order traversal
minimum number of moves
grid spread/infection
```

## Binary Tree Level Order

```python
from collections import deque

def level_order(root):
    if not root:
        return []

    ans = []
    q = deque([root])

    while q:
        level = []

        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)

            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)

        ans.append(level)

    return ans
```

## Grid BFS

```python
from collections import deque

def shortest_path_grid(grid: list[list[int]]) -> int:
    rows = len(grid)
    cols = len(grid[0])

    if grid[0][0] == 1:
        return -1

    q = deque([(0, 0, 0)])  # row, col, distance
    visited = {(0, 0)}

    directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]

    while q:
        r, c, dist = q.popleft()

        if r == rows - 1 and c == cols - 1:
            return dist

        for dr, dc in directions:
            nr = r + dr
            nc = c + dc

            if (
                0 <= nr < rows
                and 0 <= nc < cols
                and grid[nr][nc] == 0
                and (nr, nc) not in visited
            ):
                visited.add((nr, nc))
                q.append((nr, nc, dist + 1))

    return -1
```

For 8-direction movement:

```python
directions = [
    (1, 0), (-1, 0), (0, 1), (0, -1),
    (1, 1), (1, -1), (-1, 1), (-1, -1),
]
```

---

# 15. Core Pattern 11: DFS

## Problem Type

Use DFS when:

```text
connected components
islands
tree recursion
cycle detection
backtracking
topological sort
```

## Recursive DFS Graph

```python
def dfs(node):
    if node in visited:
        return

    visited.add(node)

    for nei in graph[node]:
        dfs(nei)
```

## Iterative DFS

```python
def dfs_iterative(start):
    stack = [start]
    visited = set()

    while stack:
        node = stack.pop()

        if node in visited:
            continue

        visited.add(node)

        for nei in graph[node]:
            if nei not in visited:
                stack.append(nei)
```

## Number of Islands

```python
def num_islands(grid: list[list[str]]) -> int:
    rows = len(grid)
    cols = len(grid[0])
    count = 0

    def dfs(r: int, c: int) -> None:
        if (
            r < 0
            or r >= rows
            or c < 0
            or c >= cols
            or grid[r][c] != "1"
        ):
            return

        grid[r][c] = "0"

        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1":
                count += 1
                dfs(r, c)

    return count
```

Time: `O(RC)`
Space: `O(RC)` worst-case recursion stack.

Python has a recursion limit to prevent infinite recursion from overflowing the interpreter stack; the official `sys` docs explain that `getrecursionlimit()` returns the current limit, and `setrecursionlimit()` can change it but should be used carefully because too-high limits can crash the interpreter. ([Python documentation][8])

In interviews, prefer recursion for clarity unless input size risks recursion depth.

---

# 16. Core Pattern 12: Trees

## Tree Node

Usually provided:

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

## DFS Traversals

Inorder:

```python
def inorder(root):
    if not root:
        return []

    return inorder(root.left) + [root.val] + inorder(root.right)
```

Better for large trees:

```python
def inorder(root):
    ans = []

    def dfs(node):
        if not node:
            return

        dfs(node.left)
        ans.append(node.val)
        dfs(node.right)

    dfs(root)
    return ans
```

Preorder:

```python
def preorder(root):
    ans = []

    def dfs(node):
        if not node:
            return

        ans.append(node.val)
        dfs(node.left)
        dfs(node.right)

    dfs(root)
    return ans
```

Postorder:

```python
def postorder(root):
    ans = []

    def dfs(node):
        if not node:
            return

        dfs(node.left)
        dfs(node.right)
        ans.append(node.val)

    dfs(root)
    return ans
```

## Maximum Depth

```python
def max_depth(root) -> int:
    if not root:
        return 0

    return 1 + max(max_depth(root.left), max_depth(root.right))
```

## Balanced Binary Tree

```python
def is_balanced(root) -> bool:
    def height(node):
        if not node:
            return 0

        left = height(node.left)
        if left == -1:
            return -1

        right = height(node.right)
        if right == -1:
            return -1

        if abs(left - right) > 1:
            return -1

        return 1 + max(left, right)

    return height(root) != -1
```

Staff-level explanation:

```text
Instead of computing height repeatedly, each subtree returns either its height or -1 if already unbalanced.
```

---

# 17. Core Pattern 13: Binary Search Trees

BST property:

```text
left subtree values < node.val < right subtree values
```

## Validate BST

```python
def is_valid_bst(root) -> bool:
    def dfs(node, low, high):
        if not node:
            return True

        if not (low < node.val < high):
            return False

        return (
            dfs(node.left, low, node.val)
            and dfs(node.right, node.val, high)
        )

    return dfs(root, float("-inf"), float("inf"))
```

Do not only compare node with direct children. That misses deep violations.

## Lowest Common Ancestor in BST

```python
def lowest_common_ancestor(root, p, q):
    curr = root

    while curr:
        if p.val < curr.val and q.val < curr.val:
            curr = curr.left
        elif p.val > curr.val and q.val > curr.val:
            curr = curr.right
        else:
            return curr
```

---

# 18. Core Pattern 14: Linked Lists

## Node

Usually provided:

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

## Reverse Linked List

```python
def reverse_list(head):
    prev = None
    curr = head

    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt

    return prev
```

Invariant:

```text
prev is the reversed part.
curr is the remaining unreversed part.
```

## Detect Cycle

```python
def has_cycle(head) -> bool:
    slow = head
    fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow is fast:
            return True

    return False
```

Use `is`, not `==`, because you compare node identity.

## Merge Two Sorted Lists

```python
def merge_two_lists(l1, l2):
    dummy = ListNode()
    tail = dummy

    while l1 and l2:
        if l1.val <= l2.val:
            tail.next = l1
            l1 = l1.next
        else:
            tail.next = l2
            l2 = l2.next

        tail = tail.next

    tail.next = l1 or l2

    return dummy.next
```

Dummy nodes reduce edge-case bugs.

---

# 19. Core Pattern 15: Heap / Priority Queue

## Top K Frequent Elements

```python
from collections import Counter
import heapq

def top_k_frequent(nums: list[int], k: int) -> list[int]:
    freq = Counter(nums)
    heap = []

    for num, count in freq.items():
        heapq.heappush(heap, (count, num))

        if len(heap) > k:
            heapq.heappop(heap)

    return [num for count, num in heap]
```

Time: `O(n log k)`
Space: `O(n)` for frequency map, `O(k)` heap.

## Kth Largest

```python
import heapq

def kth_largest(nums: list[int], k: int) -> int:
    heap = []

    for x in nums:
        heapq.heappush(heap, x)

        if len(heap) > k:
            heapq.heappop(heap)

    return heap[0]
```

Why min-heap?

```text
Keep only k largest elements.
The smallest among those k is the kth largest overall.
```

## Merge K Sorted Lists

```python
import heapq

def merge_k_lists(lists):
    heap = []

    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))

    dummy = ListNode()
    tail = dummy

    while heap:
        _, i, node = heapq.heappop(heap)

        tail.next = node
        tail = tail.next

        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))

    return dummy.next
```

Use `(value, i, node)` so Python does not try to compare `ListNode` objects when values tie.

---

# 20. Core Pattern 16: Greedy

## Problem Type

Use greedy when:

```text
local optimal choice leads to global optimum
sorting enables choice
interval scheduling
jump game
minimum arrows
gas station
```

Greedy needs proof. Do not just say “greedy works.”

## Jump Game

```python
def can_jump(nums: list[int]) -> bool:
    farthest = 0

    for i, jump in enumerate(nums):
        if i > farthest:
            return False

        farthest = max(farthest, i + jump)

    return True
```

Invariant:

```text
farthest is the farthest index reachable after scanning positions up to i.
```

## Non-overlapping Intervals

```python
def erase_overlap_intervals(intervals: list[list[int]]) -> int:
    intervals.sort(key=lambda x: x[1])

    removed = 0
    prev_end = float("-inf")

    for start, end in intervals:
        if start >= prev_end:
            prev_end = end
        else:
            removed += 1

    return removed
```

Greedy choice:

```text
Keep the interval with the earliest end because it leaves maximum room for future intervals.
```

---

# 21. Core Pattern 17: Backtracking

## Problem Type

Use when:

```text
generate all combinations
generate all permutations
subsets
valid parentheses
word search
N queens
constraint satisfaction
```

## Backtracking Template

```python
def backtrack(path, choices):
    if is_complete(path):
        ans.append(path.copy())
        return

    for choice in choices:
        if not valid(choice):
            continue

        path.append(choice)
        backtrack(path, choices)
        path.pop()
```

The three operations:

```text
choose
explore
unchoose
```

## Subsets

```python
def subsets(nums: list[int]) -> list[list[int]]:
    ans = []
    path = []

    def backtrack(i: int) -> None:
        if i == len(nums):
            ans.append(path.copy())
            return

        path.append(nums[i])
        backtrack(i + 1)
        path.pop()

        backtrack(i + 1)

    backtrack(0)
    return ans
```

Time: `O(n * 2^n)` because there are `2^n` subsets and copying each path can cost `O(n)`.

## Permutations

```python
def permute(nums: list[int]) -> list[list[int]]:
    ans = []
    path = []
    used = [False] * len(nums)

    def backtrack():
        if len(path) == len(nums):
            ans.append(path.copy())
            return

        for i in range(len(nums)):
            if used[i]:
                continue

            used[i] = True
            path.append(nums[i])

            backtrack()

            path.pop()
            used[i] = False

    backtrack()
    return ans
```

## Combination Sum

```python
def combination_sum(candidates: list[int], target: int) -> list[list[int]]:
    ans = []
    path = []

    def backtrack(start: int, remaining: int) -> None:
        if remaining == 0:
            ans.append(path.copy())
            return

        if remaining < 0:
            return

        for i in range(start, len(candidates)):
            path.append(candidates[i])
            backtrack(i, remaining - candidates[i])
            path.pop()

    backtrack(0, target)
    return ans
```

Use `i` again when reuse is allowed. Use `i + 1` when reuse is not allowed.

---

# 22. Core Pattern 18: Dynamic Programming

DP is just caching repeated subproblems.

Use DP when:

```text
optimal substructure
overlapping subproblems
count ways
min/max cost
choose or skip
sequence alignment
grid paths
knapsack-like decisions
```

## DP Thinking Formula

```text
State:
    What variables uniquely define a subproblem?

Transition:
    How does this state depend on smaller states?

Base case:
    What is the smallest known answer?

Order:
    In what order can states be computed?

Answer:
    Which state contains the final answer?
```

## Fibonacci / Climbing Stairs

Top-down:

```python
from functools import cache

def climb_stairs(n: int) -> int:
    @cache
    def dp(i: int) -> int:
        if i <= 1:
            return 1

        return dp(i - 1) + dp(i - 2)

    return dp(n)
```

Bottom-up:

```python
def climb_stairs(n: int) -> int:
    if n <= 1:
        return 1

    prev2 = 1
    prev1 = 1

    for _ in range(2, n + 1):
        curr = prev1 + prev2
        prev2 = prev1
        prev1 = curr

    return prev1
```

## House Robber

State:

```text
dp[i] = max money from houses up to i
```

Transition:

```text
dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])
```

Optimized code:

```python
def rob(nums: list[int]) -> int:
    prev2 = 0
    prev1 = 0

    for x in nums:
        curr = max(prev1, prev2 + x)
        prev2 = prev1
        prev1 = curr

    return prev1
```

## Coin Change

```python
def coin_change(coins: list[int], amount: int) -> int:
    INF = amount + 1
    dp = [INF] * (amount + 1)
    dp[0] = 0

    for total in range(1, amount + 1):
        for coin in coins:
            if total - coin >= 0:
                dp[total] = min(dp[total], dp[total - coin] + 1)

    return dp[amount] if dp[amount] != INF else -1
```

## Longest Increasing Subsequence: `O(n²)`

```python
def length_of_lis(nums: list[int]) -> int:
    dp = [1] * len(nums)

    for i in range(len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)

    return max(dp)
```

## Longest Increasing Subsequence: `O(n log n)`

```python
from bisect import bisect_left

def length_of_lis(nums: list[int]) -> int:
    tails = []

    for x in nums:
        i = bisect_left(tails, x)

        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x

    return len(tails)
```

Interpretation:

```text
tails[i] is the smallest possible tail value of an increasing subsequence of length i + 1.
```

---

# 23. Core Pattern 19: Graphs

## Graph Representations

Edge list:

```python
edges = [[0, 1], [1, 2]]
```

Adjacency list:

```python
from collections import defaultdict

graph = defaultdict(list)

for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)
```

Directed graph:

```python
graph[u].append(v)
```

Weighted graph:

```python
graph[u].append((v, weight))
```

---

## Connected Components

```python
def count_components(n: int, edges: list[list[int]]) -> int:
    graph = [[] for _ in range(n)]

    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    visited = set()

    def dfs(node: int) -> None:
        visited.add(node)

        for nei in graph[node]:
            if nei not in visited:
                dfs(nei)

    count = 0

    for node in range(n):
        if node not in visited:
            count += 1
            dfs(node)

    return count
```

---

## Cycle Detection in Directed Graph

```python
def has_cycle_directed(n: int, edges: list[list[int]]) -> bool:
    graph = [[] for _ in range(n)]

    for u, v in edges:
        graph[u].append(v)

    visiting = set()
    visited = set()

    def dfs(node: int) -> bool:
        if node in visiting:
            return True

        if node in visited:
            return False

        visiting.add(node)

        for nei in graph[node]:
            if dfs(nei):
                return True

        visiting.remove(node)
        visited.add(node)

        return False

    for node in range(n):
        if dfs(node):
            return True

    return False
```

Meaning:

```text
visiting = nodes in current recursion path
visited = fully processed nodes
```

---

## Topological Sort: Kahn’s Algorithm

```python
from collections import deque

def topo_sort(n: int, edges: list[list[int]]) -> list[int]:
    graph = [[] for _ in range(n)]
    indegree = [0] * n

    for u, v in edges:
        graph[u].append(v)
        indegree[v] += 1

    q = deque()

    for node in range(n):
        if indegree[node] == 0:
            q.append(node)

    order = []

    while q:
        node = q.popleft()
        order.append(node)

        for nei in graph[node]:
            indegree[nei] -= 1

            if indegree[nei] == 0:
                q.append(nei)

    return order if len(order) == n else []
```

Use for:

```text
course schedule
build dependencies
task ordering
alien dictionary
```

---

## Dijkstra

Use for shortest path with non-negative edge weights.

```python
import heapq
from collections import defaultdict

def dijkstra(n: int, edges: list[list[int]], source: int) -> list[float]:
    graph = defaultdict(list)

    for u, v, w in edges:
        graph[u].append((v, w))

    dist = [float("inf")] * n
    dist[source] = 0

    heap = [(0, source)]

    while heap:
        curr_dist, node = heapq.heappop(heap)

        if curr_dist > dist[node]:
            continue

        for nei, weight in graph[node]:
            new_dist = curr_dist + weight

            if new_dist < dist[nei]:
                dist[nei] = new_dist
                heapq.heappush(heap, (new_dist, nei))

    return dist
```

Important phrase:

```text
The heap may contain stale entries, so I skip an entry if its distance is greater than the best known distance.
```

---

# 24. Core Pattern 20: Union-Find / DSU

Use for:

```text
connected components
dynamic connectivity
cycle detection in undirected graph
Kruskal MST
accounts merge
number of provinces
```

## Template

```python
class DSU:
    def __init__(self, n: int):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n

    def find(self, x: int) -> int:
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])

        return self.parent[x]

    def union(self, a: int, b: int) -> bool:
        root_a = self.find(a)
        root_b = self.find(b)

        if root_a == root_b:
            return False

        if self.rank[root_a] < self.rank[root_b]:
            root_a, root_b = root_b, root_a

        self.parent[root_b] = root_a

        if self.rank[root_a] == self.rank[root_b]:
            self.rank[root_a] += 1

        self.components -= 1

        return True
```

Use:

```python
def count_components(n: int, edges: list[list[int]]) -> int:
    dsu = DSU(n)

    for u, v in edges:
        dsu.union(u, v)

    return dsu.components
```

Staff-level explanation:

```text
Path compression flattens the tree during find.
Union by rank keeps trees shallow.
```

---

# 25. Core Pattern 21: Trie

Use for:

```text
prefix search
word dictionary
autocomplete
word break variants
search suggestions
```

## Trie Node

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False
```

## Trie

```python
class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root

        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()

            node = node.children[ch]

        node.is_word = True

    def search(self, word: str) -> bool:
        node = self.root

        for ch in word:
            if ch not in node.children:
                return False

            node = node.children[ch]

        return node.is_word

    def starts_with(self, prefix: str) -> bool:
        node = self.root

        for ch in prefix:
            if ch not in node.children:
                return False

            node = node.children[ch]

        return True
```

---

# 26. Core Pattern 22: Intervals

## Merge Intervals

Already shown.

## Insert Interval

```python
def insert_interval(intervals: list[list[int]], new_interval: list[int]) -> list[list[int]]:
    ans = []
    i = 0
    n = len(intervals)

    while i < n and intervals[i][1] < new_interval[0]:
        ans.append(intervals[i])
        i += 1

    while i < n and intervals[i][0] <= new_interval[1]:
        new_interval[0] = min(new_interval[0], intervals[i][0])
        new_interval[1] = max(new_interval[1], intervals[i][1])
        i += 1

    ans.append(new_interval)

    while i < n:
        ans.append(intervals[i])
        i += 1

    return ans
```

## Meeting Rooms

```python
def can_attend_meetings(intervals: list[list[int]]) -> bool:
    intervals.sort()

    for i in range(1, len(intervals)):
        if intervals[i][0] < intervals[i - 1][1]:
            return False

    return True
```

## Minimum Meeting Rooms

```python
import heapq

def min_meeting_rooms(intervals: list[list[int]]) -> int:
    intervals.sort(key=lambda x: x[0])
    heap = []

    for start, end in intervals:
        if heap and heap[0] <= start:
            heapq.heappop(heap)

        heapq.heappush(heap, end)

    return len(heap)
```

---

# 27. Core Pattern 23: Bit Manipulation

Use for:

```text
xor tricks
subsets
flags
single number
power of two
bit masks
```

## XOR Rules

```text
x ^ x = 0
x ^ 0 = x
xor is commutative and associative
```

## Single Number

```python
def single_number(nums: list[int]) -> int:
    ans = 0

    for x in nums:
        ans ^= x

    return ans
```

## Power of Two

```python
def is_power_of_two(n: int) -> bool:
    return n > 0 and (n & (n - 1)) == 0
```

## Generate Subsets with Bitmask

```python
def subsets(nums: list[int]) -> list[list[int]]:
    n = len(nums)
    ans = []

    for mask in range(1 << n):
        subset = []

        for i in range(n):
            if mask & (1 << i):
                subset.append(nums[i])

        ans.append(subset)

    return ans
```

---

# 28. Core Pattern 24: Math

## Modulo

```python
MOD = 10**9 + 7
ans %= MOD
```

## GCD

```python
from math import gcd

g = gcd(a, b)
```

## Ceiling Division

```python
ceil = (a + b - 1) // b
```

For positive integers.

## Prime Check

```python
def is_prime(n: int) -> bool:
    if n < 2:
        return False

    d = 2

    while d * d <= n:
        if n % d == 0:
            return False

        d += 1

    return True
```

## Sieve

```python
def sieve(n: int) -> list[bool]:
    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False

    p = 2

    while p * p <= n:
        if is_prime[p]:
            for multiple in range(p * p, n + 1, p):
                is_prime[multiple] = False

        p += 1

    return is_prime
```

---

# 29. Python Pitfalls That Kill Interviews

## Pitfall 1: Mutable Default Arguments

Bad:

```python
def dfs(node, visited=set()):
    ...
```

Good:

```python
def dfs(node, visited=None):
    if visited is None:
        visited = set()
```

## Pitfall 2: Shallow Copy of 2D List

Bad:

```python
grid = [[0] * cols] * rows
```

This aliases rows.

Good:

```python
grid = [[0] * cols for _ in range(rows)]
```

## Pitfall 3: Modifying List While Iterating

Bad:

```python
for x in nums:
    if x < 0:
        nums.remove(x)
```

Good:

```python
nums = [x for x in nums if x >= 0]
```

## Pitfall 4: `list.pop(0)` for Queue

Bad:

```python
q = []
q.pop(0)
```

Good:

```python
from collections import deque

q = deque()
q.popleft()
```

## Pitfall 5: Recursion Depth

For deep trees or graphs, recursive DFS may fail in Python. Either use iterative DFS or discuss recursion-limit risk. The official Python docs warn that raising the recursion limit should be done carefully because too-high limits can crash the interpreter. ([Python documentation][8])

## Pitfall 6: Sorting Key Bug

Bad:

```python
intervals.sort(key=lambda x: x[1])
```

when the algorithm needs start order.

Good:

```python
intervals.sort(key=lambda x: x[0])
```

Know why you sort.

## Pitfall 7: Returning Too Late

Bad:

```python
for x in nums:
    if x == target:
        found = True

return found
```

If `found` was never initialized, bug.

Good:

```python
for x in nums:
    if x == target:
        return True

return False
```

---

# 30. Staff-Level Code Quality Bar

At staff level, your coding answer should not look like rushed contest code. It should be:

```text
clear
small
well-named
edge-case safe
complexity aware
easy to modify
```

Example: acceptable but weak:

```python
def f(a, k):
    d = {}
    for i in range(len(a)):
        if k - a[i] in d:
            return [d[k - a[i]], i]
        d[a[i]] = i
```

Better:

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    value_to_index = {}

    for i, value in enumerate(nums):
        complement = target - value

        if complement in value_to_index:
            return [value_to_index[complement], i]

        value_to_index[value] = i

    return []
```

A staff-level explanation:

```text
I store previously seen values in a hash map from value to index.
At index i, if target - nums[i] was seen earlier, we found a valid pair.
This avoids checking all pairs.
Time is O(n), and space is O(n).
```

---

# 31. Problem Recognition Map

Use this map during practice.

```text
Need fast lookup?
    hash set / hash map

Need counts?
    Counter / dict / defaultdict(int)

Need contiguous subarray or substring?
    sliding window / prefix sum

Need sorted input and pair?
    two pointers

Need first true / minimum feasible?
    binary search

Need top k / smallest / largest stream?
    heap

Need matching / next greater?
    stack / monotonic stack

Need shortest path unweighted?
    BFS

Need explore all connected cells/nodes?
    DFS

Need dependency order?
    topological sort

Need dynamic connectivity?
    union-find

Need all combinations/permutations?
    backtracking

Need optimize overlapping choices?
    dynamic programming

Need prefix lookup?
    trie

Need range updates?
    difference array

Need intervals?
    sort by start or end, then merge/greedy/heap
```

---

# 32. Must-Know FAANG+ Problems by Pattern

Do not grind randomly. Master patterns.

## Arrays / Hashing

```text
Two Sum
Contains Duplicate
Valid Anagram
Group Anagrams
Top K Frequent Elements
Product of Array Except Self
Longest Consecutive Sequence
```

## Two Pointers

```text
Valid Palindrome
Two Sum II
3Sum
Container With Most Water
Remove Duplicates from Sorted Array
```

## Sliding Window

```text
Best Time to Buy and Sell Stock
Longest Substring Without Repeating Characters
Minimum Window Substring
Permutation in String
Longest Repeating Character Replacement
```

## Stack

```text
Valid Parentheses
Min Stack
Evaluate Reverse Polish Notation
Daily Temperatures
Largest Rectangle in Histogram
```

## Binary Search

```text
Binary Search
Search Rotated Sorted Array
Find Minimum in Rotated Sorted Array
Koko Eating Bananas
Capacity to Ship Packages
Median of Two Sorted Arrays
```

## Trees

```text
Maximum Depth of Binary Tree
Invert Binary Tree
Diameter of Binary Tree
Validate BST
Lowest Common Ancestor
Serialize and Deserialize Binary Tree
```

## Graphs

```text
Number of Islands
Clone Graph
Course Schedule
Pacific Atlantic Water Flow
Word Ladder
Network Delay Time
```

## Dynamic Programming

```text
Climbing Stairs
House Robber
Coin Change
Longest Increasing Subsequence
Word Break
Edit Distance
Regular Expression Matching
```

## Backtracking

```text
Subsets
Permutations
Combination Sum
Word Search
N Queens
Generate Parentheses
```

## Intervals

```text
Merge Intervals
Insert Interval
Meeting Rooms
Meeting Rooms II
Non-overlapping Intervals
Minimum Arrows to Burst Balloons
```

---

# 33. The Final Interview Template

Use this exact skeleton in your head.

```python
def solve(input_data):
    # 1. Handle edge cases
    if not input_data:
        return ...

    # 2. Initialize data structures
    ...

    # 3. Iterate / recurse / search
    for ...:
        ...

    # 4. Return answer
    return ...
```

For graph:

```python
def solve(n, edges):
    graph = [[] for _ in range(n)]

    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    visited = set()

    def dfs(node):
        visited.add(node)

        for nei in graph[node]:
            if nei not in visited:
                dfs(nei)

    ...
```

For DP:

```python
def solve(nums):
    dp = ...

    for i in range(...):
        dp[i] = ...

    return dp[-1]
```

For binary search:

```python
def solve(nums):
    def feasible(x):
        ...

    left = ...
    right = ...

    while left < right:
        mid = (left + right) // 2

        if feasible(mid):
            right = mid
        else:
            left = mid + 1

    return left
```

---

# 34. What to Say Out Loud in a Staff-Level Coding Round

Use this language.

## When starting

```text
I’ll first restate the problem to make sure I understand it.
```

## When giving brute force

```text
The straightforward solution is to check every pair, which is O(n²).
That is likely too slow if n is large.
```

## When optimizing

```text
The repeated work is lookup, so I’ll trade space for time using a hash map.
```

## When coding

```text
I’ll maintain the invariant that the map contains only values seen before the current index.
```

## When testing

```text
Let’s test empty input, a single element, duplicates, and negative values.
```

## When finishing

```text
Time complexity is O(n) because each element is processed once.
Space complexity is O(n) for the hash map.
```

## When stuck

```text
Let me step back and identify the invariant.
```

That sentence is powerful.

---

# 35. 8-Week Study Plan

## Week 1: Python DSA Fluency

Master:

```text
list
dict
set
tuple
deque
Counter
defaultdict
heapq
bisect
cache
```

Implement:

```text
Two Sum
Group Anagrams
Valid Parentheses
Binary Search
Number of Islands
```

## Week 2: Arrays, Strings, Hashing

Focus:

```text
frequency
prefix sum
two pointers
sliding window
```

## Week 3: Stack, Queue, Intervals, Sorting

Focus:

```text
monotonic stack
merge intervals
meeting rooms
greedy after sorting
```

## Week 4: Trees

Focus:

```text
recursive DFS
iterative DFS
BFS level order
BST bounds
tree DP
```

## Week 5: Graphs

Focus:

```text
DFS/BFS
topological sort
union-find
Dijkstra
cycle detection
```

## Week 6: Dynamic Programming

Focus:

```text
1D DP
2D DP
knapsack
sequence DP
memoized recursion
```

## Week 7: Backtracking and Advanced Patterns

Focus:

```text
subsets
permutations
combination sum
word search
tries
bitmasks
```

## Week 8: Mock Interviews

Do:

```text
45-minute timed problems
explain aloud
write tests
give complexity
review mistakes
```

At staff level, also practice:

```text
turning ambiguous requirements into constraints
explaining tradeoffs
handling follow-up changes
discussing production implications
```

---

# 36. Final Python DSA Reference Sheet

```python
from collections import defaultdict, Counter, deque
from functools import cache
from bisect import bisect_left, bisect_right
import heapq
import math
```

## Hash Map

```python
freq = defaultdict(int)
freq[x] += 1
```

## Set

```python
seen = set()
seen.add(x)
x in seen
```

## Queue

```python
q = deque([start])
q.append(x)
q.popleft()
```

## Heap

```python
heap = []
heapq.heappush(heap, x)
heapq.heappop(heap)
```

## Max Heap

```python
heapq.heappush(heap, -x)
largest = -heapq.heappop(heap)
```

## Binary Search

```python
i = bisect_left(arr, x)
j = bisect_right(arr, x)
```

## Memoization

```python
@cache
def dp(i):
    ...
```

## Sort

```python
arr.sort()
arr.sort(key=lambda x: x[0])
arr.sort(reverse=True)
```

## Enumerate

```python
for i, x in enumerate(nums):
    ...
```

## Grid Directions

```python
dirs = [(1, 0), (-1, 0), (0, 1), (0, -1)]
```

## Infinity

```python
INF = float("inf")
```

---

# 37. The Minimum You Must Master

To be interview-ready, you should be able to write these from memory:

```text
two sum
sliding window longest substring
binary search lower bound
merge intervals
valid parentheses
monotonic stack
BFS grid
DFS islands
tree depth
validate BST
reverse linked list
topological sort
Dijkstra
union-find
subsets
permutations
coin change
house robber
LIS
trie
```

If you can write those cleanly, explain them, test them, and adapt them to follow-ups, you are no longer “learning Python.” You are using Python as an interview execution tool.

That is the goal.

[1]: https://www.amazon.jobs/content/en/how-we-hire/interview-prep/software-development-topics "Amazon Software Development Interview Topics"
[2]: https://www.amazon.jobs/content/en/how-we-hire/sde-iii-interview-prep "SDE III Interview Prep"
[3]: https://docs.python.org/3/library/collections.html "collections — Container datatypes — Python 3.14.6 documentation"
[4]: https://wiki.python.org/moin/TimeComplexity "TimeComplexity"
[5]: https://docs.python.org/3/library/heapq.html "heapq — Heap queue algorithm — Python 3.14.6 documentation"
[6]: https://docs.python.org/3/library/bisect.html "bisect — Array bisection algorithm — Python 3.14.6 documentation"
[7]: https://docs.python.org/3/library/functools.html "functools — Higher-order functions and operations on callable objects — Python 3.14.6 documentation"
[8]: https://docs.python.org/3/library/sys.html "sys — System-specific parameters and functions — Python 3.14.6 documentation"


I’m using the uploaded CS50P transcript only as a baseline for Python fundamentals, then refocusing everything toward FAANG+ DS/Algo interview speed, correctness, and Java-to-Python transition. 

# Python for FAANG+ DS/Algo Interviews

## The Java Developer’s End-to-End Python Handbook

This is not a generic Python course. This is the Python you need to solve coding rounds fast, explain tradeoffs clearly, and avoid Python-specific traps that cause wrong answers or TLE.

Assume Python 3.10+ for interviews unless the platform says otherwise. For interview coding, avoid very new version-specific features unless you know the judge supports them. For example, Python’s `heapq` is historically a min-heap API; Python 3.14 adds max-heap helpers, but in interviews you should still usually use negative values for max-heaps because most platforms may not run 3.14. ([Python documentation][1])

---

# 1. The Core Mindset

As a Java developer, your biggest advantage is that you already know programming. Your biggest Python danger is assuming Python has the same performance behavior as Java collections.

In interviews, Python wins because you can express the algorithm faster:

```python
from collections import defaultdict, Counter, deque
from heapq import heappush, heappop
from bisect import bisect_left, bisect_right
from functools import lru_cache
```

But Python loses when you accidentally do hidden O(n) work:

```python
arr.pop(0)       # bad: O(n)
s = s[1:]        # bad in loops: copies string/list
x in list        # O(n), not O(1)
sorted(...)      # O(n log n), not free
```

Your interview goal:

```text
Correct algorithm
  + correct data structure
  + clean Python implementation
  + explicit complexity
  + edge cases handled
```

---

# 2. Python vs Java: Interview Translation Table

| Java                 | Python                    | Interview Notes                                                                           |                    |                    |
| -------------------- | ------------------------- | ----------------------------------------------------------------------------------------- | ------------------ | ------------------ |
| `ArrayList<Integer>` | `list[int]`               | Dynamic array. Append O(1) amortized. Insert/delete middle O(n).                          |                    |                    |
| `int[]`              | `list[int]`               | Python integers are objects, but fine for interviews.                                     |                    |                    |
| `HashMap<K,V>`       | `dict`                    | Average O(1) lookup/insert/delete.                                                        |                    |                    |
| `HashSet<T>`         | `set`                     | Average O(1) membership.                                                                  |                    |                    |
| `PriorityQueue<T>`   | `heapq` on list           | Min-heap only in normal interview Python. Use negative values for max-heap.               |                    |                    |
| `ArrayDeque<T>`      | `collections.deque`       | Use for BFS queue, O(1) `popleft`.                                                        |                    |                    |
| `Stack<T>`           | `list`                    | Use `append` and `pop`.                                                                   |                    |                    |
| `StringBuilder`      | `list` + `"".join(parts)` | Strings are immutable.                                                                    |                    |                    |
| `TreeMap`, `TreeSet` | no built-in equivalent    | Use `bisect` on sorted list, heap, two heaps, or custom structure. Beware insertion O(n). |                    |                    |
| `Comparator`         | `key=` function           | Python sorting uses key extraction, not comparator.                                       |                    |                    |
| `Pair`               | tuple `(a, b)`            | Tuples compare lexicographically. Very useful in heaps/sorting.                           |                    |                    |
| `null`               | `None`                    | Check with `is None`, not `== None`.                                                      |                    |                    |
| `true/false`         | `True/False`              | Capitalized.                                                                              |                    |                    |
| `&&`, `              |                           | `, `!`                                                                                    | `and`, `or`, `not` | Python uses words. |
| `this`               | `self`                    | For class methods.                                                                        |                    |                    |
| `final` constants    | uppercase convention      | Python does not enforce constants.                                                        |                    |                    |

---

# 3. Python Syntax You Actually Need

## Variables

Python is dynamically typed:

```python
x = 10
name = "David"
seen = set()
```

No type declarations needed.

Optional type hints are useful for clarity:

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    ...
```

Type hints do not enforce runtime behavior.

---

## `if`, `elif`, `else`

```python
if x < y:
    return -1
elif x > y:
    return 1
else:
    return 0
```

Python requires indentation. No braces.

---

## Loops

```python
for x in nums:
    print(x)
```

Index loop:

```python
for i in range(len(nums)):
    print(i, nums[i])
```

Better index + value:

```python
for i, x in enumerate(nums):
    print(i, x)
```

Reverse index loop:

```python
for i in range(len(nums) - 1, -1, -1):
    print(nums[i])
```

---

## Functions

```python
def add(a, b):
    return a + b
```

Default parameters:

```python
def dfs(node, parent=None):
    ...
```

Important pitfall: never use mutable default arguments.

Bad:

```python
def f(path=[]):
    path.append(1)
    return path
```

Good:

```python
def f(path=None):
    if path is None:
        path = []
    path.append(1)
    return path
```

---

## Tuple unpacking

```python
a, b = b, a
```

Multiple assignment is heavily used in linked lists:

```python
prev, curr = None, head
```

Returning multiple values:

```python
def min_max(nums):
    return min(nums), max(nums)

lo, hi = min_max(nums)
```

---

## Truthiness

Falsy values:

```python
False
None
0
0.0
""
[]
{}
set()
```

Common use:

```python
if not nums:
    return 0
```

But be careful:

```python
if not node:
    ...
```

This is fine for `None` nodes in linked lists/trees.

---

# 4. Must-Know Python Data Structures

Python’s standard library gives specialized containers such as `deque`, `Counter`, and `defaultdict` in `collections`, which are extremely useful for interview problems. ([Python documentation][2])

## 4.1 List

A Python `list` is a dynamic array.

```python
arr = []
arr.append(10)
arr.append(20)
arr.pop()
```

Complexities:

| Operation          | Complexity     |
| ------------------ | -------------- |
| `arr[i]`           | O(1)           |
| `arr.append(x)`    | O(1) amortized |
| `arr.pop()`        | O(1)           |
| `arr.pop(0)`       | O(n), avoid    |
| `arr.insert(i, x)` | O(n)           |
| `x in arr`         | O(n)           |
| slicing `arr[l:r]` | O(k), copies   |

Interview rules:

```python
stack = []
stack.append(x)
top = stack[-1]
stack.pop()
```

Do not use list as a queue:

```python
q = []
q.pop(0)   # bad
```

Use `deque`.

---

## 4.2 Deque

```python
from collections import deque

q = deque()
q.append(1)
q.append(2)
q.popleft()
```

Use for:

```text
BFS
sliding window max/min
monotonic queues
queue simulation
```

---

## 4.3 Dict

```python
mp = {}
mp["a"] = 1
mp["a"] += 1
```

Safe frequency counting:

```python
freq = {}
for x in nums:
    freq[x] = freq.get(x, 0) + 1
```

Or:

```python
from collections import defaultdict

freq = defaultdict(int)
for x in nums:
    freq[x] += 1
```

Important:

```python
if key in mp:
    ...
```

`key in mp` checks keys, not values.

---

## 4.4 Set

```python
seen = set()
seen.add(x)

if x in seen:
    ...
```

Use for:

```text
visited nodes
duplicate detection
O(1) membership
```

Empty set:

```python
s = set()
```

Not:

```python
s = {}      # this is dict
```

---

## 4.5 Counter

```python
from collections import Counter

freq = Counter(nums)
freq["a"] += 1
freq.most_common(3)
```

Useful for:

```text
anagrams
top-k frequency
frequency comparison
multiset-like logic
```

Pitfall:

```python
freq[x] -= 1
```

This may leave zero or negative counts. Sometimes you must delete:

```python
if freq[x] == 0:
    del freq[x]
```

---

## 4.6 defaultdict

```python
from collections import defaultdict

graph = defaultdict(list)
graph[u].append(v)
```

Common forms:

```python
defaultdict(int)      # frequency
defaultdict(list)     # graph adjacency
defaultdict(set)      # grouping unique values
```

Bad:

```python
defaultdict([])       # wrong
```

You pass a factory, not a value.

---

## 4.7 Heap

Python’s `heapq` operates on lists and gives a min-heap. Use `heappush`, `heappop`, and `heapify`; `heapify` builds a heap in linear time, and push/pop operations are logarithmic. ([Python documentation][1])

```python
from heapq import heappush, heappop, heapify

heap = []
heappush(heap, 5)
heappush(heap, 2)
heappush(heap, 9)

heappop(heap)     # 2
```

Max-heap:

```python
heappush(heap, -x)
largest = -heappop(heap)
```

Heap with tuples:

```python
heappush(heap, (distance, node))
```

Python compares tuples lexicographically:

```python
(1, "a") < (2, "z")    # True
```

Critical pitfall:

```python
heappush(heap, (priority, obj))
```

If two priorities tie, Python compares `obj`. If `obj` is not comparable, crash.

Fix with tie-breaker:

```python
counter = 0
heappush(heap, (priority, counter, obj))
counter += 1
```

---

## 4.8 Bisect

`bisect_left` and `bisect_right` perform binary search on sorted lists. But `insort` still has O(n) insertion because list insertion shifts elements, even though the search part is O(log n). ([Python documentation][3])

```python
from bisect import bisect_left, bisect_right

arr = [1, 2, 2, 2, 5]

bisect_left(arr, 2)    # 1
bisect_right(arr, 2)   # 4
```

Counts of target:

```python
count = bisect_right(arr, x) - bisect_left(arr, x)
```

Insert position:

```python
i = bisect_left(arr, x)
```

But:

```python
arr.insert(i, x)   # O(n)
```

Do not pretend this is `TreeMap`.

---

## 4.9 functools cache

For DP memoization:

```python
from functools import lru_cache

@lru_cache(None)
def dp(i, j):
    ...
```

Python’s `functools.cache` is an unbounded memoization wrapper equivalent to `lru_cache(maxsize=None)`, but `lru_cache(None)` is widely recognized and safe in interviews. ([Python documentation][4])

Arguments must be hashable:

```python
dp(i, tuple(arr))   # okay
dp(i, arr)          # bad if arr is list
```

---

# 5. Python Complexity Cheat Sheet

## List

```text
append: O(1) amortized
pop end: O(1)
pop front: O(n)
insert middle: O(n)
indexing: O(1)
slicing: O(k)
sort: O(n log n)
```

## Dict / Set

```text
lookup: average O(1)
insert: average O(1)
delete: average O(1)
```

Worst case can degrade, but interviews generally use average case.

## Heap

```text
heappush: O(log n)
heappop: O(log n)
heapify: O(n)
peek heap[0]: O(1)
```

## Deque

```text
append: O(1)
appendleft: O(1)
pop: O(1)
popleft: O(1)
```

## Sorting

```text
sorted(arr): returns new sorted list
arr.sort(): sorts in place, returns None
```

---

# 6. Python Pitfalls That Kill Interviews

## Pitfall 1: `list.pop(0)`

Bad BFS:

```python
q = [start]

while q:
    node = q.pop(0)   # O(n)
```

Good BFS:

```python
from collections import deque

q = deque([start])

while q:
    node = q.popleft()
```

---

## Pitfall 2: Aliased 2D arrays

Bad:

```python
grid = [[0] * cols] * rows
```

All rows point to the same list.

Good:

```python
grid = [[0] * cols for _ in range(rows)]
```

---

## Pitfall 3: Mutable default arguments

Bad:

```python
def dfs(node, path=[]):
    ...
```

Good:

```python
def dfs(node, path=None):
    if path is None:
        path = []
```

---

## Pitfall 4: Slicing inside recursion or loops

Bad:

```python
def solve(arr):
    return solve(arr[1:])
```

`arr[1:]` copies O(n) each time.

Good:

```python
def solve(i):
    return solve(i + 1)
```

Pass indices instead.

---

## Pitfall 5: String concatenation in loops

Bad:

```python
s = ""
for ch in chars:
    s += ch
```

Good:

```python
parts = []
for ch in chars:
    parts.append(ch)

s = "".join(parts)
```

---

## Pitfall 6: `sort()` returns `None`

Bad:

```python
arr = arr.sort()
```

Now `arr` is `None`.

Good:

```python
arr.sort()
```

Or:

```python
arr = sorted(arr)
```

---

## Pitfall 7: `is` vs `==`

Use `is` for identity:

```python
if node is None:
    ...
```

Use `==` for value equality:

```python
if s == "abc":
    ...
```

Bad:

```python
if x is 1000:
    ...
```

---

## Pitfall 8: Python division differs from Java

```python
5 // 2     # 2
-5 // 2    # -3
```

Python `//` floors toward negative infinity. Java integer division truncates toward zero.

For LeetCode calculator problems requiring truncation toward zero:

```python
def trunc_div(a, b):
    sign = -1 if (a < 0) ^ (b < 0) else 1
    return sign * (abs(a) // abs(b))
```

Do not use:

```python
int(a / b)
```

It may involve floating-point precision issues for huge numbers.

---

## Pitfall 9: Recursion depth

Python recursion limit is much lower than Java stack expectations.

Recursive DFS may fail on a deep graph/tree.

Option 1:

```python
import sys
sys.setrecursionlimit(10**6)
```

Option 2: safer for interviews, use iterative DFS:

```python
stack = [root]

while stack:
    node = stack.pop()
```

---

## Pitfall 10: Modifying while iterating

Bad:

```python
for x in arr:
    if should_remove(x):
        arr.remove(x)
```

Good:

```python
arr = [x for x in arr if not should_remove(x)]
```

Or iterate over copy:

```python
for x in arr[:]:
    ...
```

But remember slicing copies.

---

## Pitfall 11: Set/list/dict copying

```python
b = a
```

This does not copy. It aliases.

Shallow copy:

```python
b = a[:]
b = list(a)
b = a.copy()
```

For nested structures:

```python
import copy
b = copy.deepcopy(a)
```

In interviews, avoid deep copy in hot loops unless necessary.

---

## Pitfall 12: Tuple comparison in heap

```python
heap = []
heappush(heap, (priority, node))
```

If two priorities tie, Python compares `node`.

If `node` is a custom object, crash.

Fix:

```python
heappush(heap, (priority, counter, node))
counter += 1
```

---

# 7. Interview Input/Output Patterns

On LeetCode-style platforms, you usually write:

```python
class Solution:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        ...
```

No need for `input()`.

For HackerRank / CodeSignal / command-line style:

```python
import sys

data = sys.stdin.read().strip().split()
```

Convert:

```python
nums = list(map(int, data))
```

Line by line:

```python
import sys

for line in sys.stdin:
    ...
```

Fast output:

```python
out = []
out.append(str(ans))
print("\n".join(out))
```

---

# 8. Sorting Mastery

## Basic sort

```python
arr.sort()
```

New sorted list:

```python
b = sorted(arr)
```

Descending:

```python
arr.sort(reverse=True)
```

Sort tuples:

```python
pairs.sort()
```

Lexicographic:

```python
(1, 2) < (1, 3)   # True
(1, 5) < (2, 0)   # True
```

## Sort by key

Java comparator:

```java
Arrays.sort(arr, (a, b) -> a[1] - b[1]);
```

Python:

```python
arr.sort(key=lambda x: x[1])
```

Sort by second ascending, first descending:

```python
arr.sort(key=lambda x: (x[1], -x[0]))
```

Sort intervals:

```python
intervals.sort(key=lambda x: x[0])
```

Sort strings by length then lexicographic:

```python
words.sort(key=lambda w: (len(w), w))
```

## Stable sort

Python sort is stable. This means equal keys preserve original order.

Useful trick:

```python
items.sort(key=lambda x: x.secondary)
items.sort(key=lambda x: x.primary)
```

But usually one tuple key is cleaner.

---

# 9. Pattern 1: Hash Map

## Two Sum

```python
def two_sum(nums, target):
    seen = {}

    for i, x in enumerate(nums):
        need = target - x
        if need in seen:
            return [seen[need], i]
        seen[x] = i

    return []
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

Java instinct: `HashMap<Integer, Integer>`.
Python: `dict`.

---

## Count frequencies

```python
from collections import Counter

freq = Counter(nums)
```

Manual:

```python
freq = {}
for x in nums:
    freq[x] = freq.get(x, 0) + 1
```

---

## Group anagrams

```python
from collections import defaultdict

def group_anagrams(strs):
    groups = defaultdict(list)

    for s in strs:
        key = tuple(sorted(s))
        groups[key].append(s)

    return list(groups.values())
```

Better for lowercase English letters:

```python
def group_anagrams(strs):
    groups = defaultdict(list)

    for s in strs:
        count = [0] * 26
        for ch in s:
            count[ord(ch) - ord("a")] += 1
        groups[tuple(count)].append(s)

    return list(groups.values())
```

Why tuple?

```python
list is not hashable
tuple is hashable
```

---

# 10. Pattern 2: Two Pointers

Use when:

```text
array/string is sorted
you need pair/triplet
you need in-place compression
you need left/right convergence
```

## Valid palindrome

```python
def is_palindrome(s):
    l, r = 0, len(s) - 1

    while l < r:
        while l < r and not s[l].isalnum():
            l += 1
        while l < r and not s[r].isalnum():
            r -= 1

        if s[l].lower() != s[r].lower():
            return False

        l += 1
        r -= 1

    return True
```

---

## Two Sum II sorted

```python
def two_sum_sorted(nums, target):
    l, r = 0, len(nums) - 1

    while l < r:
        total = nums[l] + nums[r]

        if total == target:
            return [l, r]
        elif total < target:
            l += 1
        else:
            r -= 1

    return [-1, -1]
```

---

## 3Sum

```python
def three_sum(nums):
    nums.sort()
    res = []
    n = len(nums)

    for i in range(n):
        if i > 0 and nums[i] == nums[i - 1]:
            continue

        l, r = i + 1, n - 1

        while l < r:
            total = nums[i] + nums[l] + nums[r]

            if total == 0:
                res.append([nums[i], nums[l], nums[r]])
                l += 1
                r -= 1

                while l < r and nums[l] == nums[l - 1]:
                    l += 1
                while l < r and nums[r] == nums[r + 1]:
                    r -= 1

            elif total < 0:
                l += 1
            else:
                r -= 1

    return res
```

Pitfall: duplicate skipping must happen at both `i` and after finding a triplet.

---

# 11. Pattern 3: Sliding Window

Use when:

```text
contiguous subarray/substring
longest/shortest window
at most K
exactly K via atMost(K) - atMost(K-1)
```

## Fixed-size window

Maximum sum of size `k`:

```python
def max_sum_k(nums, k):
    window = sum(nums[:k])
    best = window

    for r in range(k, len(nums)):
        window += nums[r] - nums[r - k]
        best = max(best, window)

    return best
```

---

## Variable-size window: longest substring without repeating

```python
def length_of_longest_substring(s):
    seen = set()
    l = 0
    best = 0

    for r, ch in enumerate(s):
        while ch in seen:
            seen.remove(s[l])
            l += 1

        seen.add(ch)
        best = max(best, r - l + 1)

    return best
```

Alternative faster with last seen index:

```python
def length_of_longest_substring(s):
    last = {}
    l = 0
    best = 0

    for r, ch in enumerate(s):
        if ch in last and last[ch] >= l:
            l = last[ch] + 1

        last[ch] = r
        best = max(best, r - l + 1)

    return best
```

---

## At most K distinct

```python
from collections import defaultdict

def longest_at_most_k_distinct(s, k):
    count = defaultdict(int)
    l = 0
    best = 0

    for r, ch in enumerate(s):
        count[ch] += 1

        while len(count) > k:
            count[s[l]] -= 1
            if count[s[l]] == 0:
                del count[s[l]]
            l += 1

        best = max(best, r - l + 1)

    return best
```

Exactly K distinct subarrays:

```python
def subarrays_with_k_distinct(nums, k):
    return at_most(nums, k) - at_most(nums, k - 1)

def at_most(nums, k):
    count = defaultdict(int)
    l = 0
    ans = 0

    for r, x in enumerate(nums):
        count[x] += 1

        while len(count) > k:
            count[nums[l]] -= 1
            if count[nums[l]] == 0:
                del count[nums[l]]
            l += 1

        ans += r - l + 1

    return ans
```

The key insight:

```text
For each r, every subarray ending at r and starting from l..r is valid.
That count is r - l + 1.
```

---

# 12. Pattern 4: Prefix Sum

Use when:

```text
range sum
subarray sum equals target
count subarrays
difference between prefixes
```

## Prefix sum array

```python
def build_prefix(nums):
    prefix = [0]

    for x in nums:
        prefix.append(prefix[-1] + x)

    return prefix
```

Range sum `[l, r]` inclusive:

```python
sum_lr = prefix[r + 1] - prefix[l]
```

---

## Subarray sum equals K

```python
from collections import defaultdict

def subarray_sum(nums, k):
    count = defaultdict(int)
    count[0] = 1

    prefix = 0
    ans = 0

    for x in nums:
        prefix += x
        ans += count[prefix - k]
        count[prefix] += 1

    return ans
```

Why `count[0] = 1`?

```text
A subarray from index 0 to i is valid if prefix == k.
So prefix - k == 0 must already exist once.
```

---

## Longest subarray sum K

```python
def longest_subarray_sum_k(nums, k):
    first = {0: -1}
    prefix = 0
    best = 0

    for i, x in enumerate(nums):
        prefix += x

        if prefix - k in first:
            best = max(best, i - first[prefix - k])

        if prefix not in first:
            first[prefix] = i

    return best
```

Important: store first occurrence, not latest.

---

# 13. Pattern 5: Difference Array

Use when:

```text
many range updates
final array needed
```

Example: add `val` to all indices `[l, r]`.

```python
def apply_updates(n, updates):
    diff = [0] * (n + 1)

    for l, r, val in updates:
        diff[l] += val
        if r + 1 < n:
            diff[r + 1] -= val

    arr = [0] * n
    running = 0

    for i in range(n):
        running += diff[i]
        arr[i] = running

    return arr
```

Complexity:

```text
Range updates: O(1) each
Build final: O(n)
Total: O(n + q)
```

---

# 14. Pattern 6: Intervals

## Merge intervals

```python
def merge(intervals):
    if not intervals:
        return []

    intervals.sort(key=lambda x: x[0])
    res = [intervals[0]]

    for start, end in intervals[1:]:
        last = res[-1]

        if start <= last[1]:
            last[1] = max(last[1], end)
        else:
            res.append([start, end])

    return res
```

---

## Insert interval

```python
def insert(intervals, new_interval):
    res = []
    i = 0
    n = len(intervals)

    while i < n and intervals[i][1] < new_interval[0]:
        res.append(intervals[i])
        i += 1

    while i < n and intervals[i][0] <= new_interval[1]:
        new_interval[0] = min(new_interval[0], intervals[i][0])
        new_interval[1] = max(new_interval[1], intervals[i][1])
        i += 1

    res.append(new_interval)

    while i < n:
        res.append(intervals[i])
        i += 1

    return res
```

---

## Meeting rooms

```python
def can_attend_meetings(intervals):
    intervals.sort()

    for i in range(1, len(intervals)):
        if intervals[i][0] < intervals[i - 1][1]:
            return False

    return True
```

Minimum rooms:

```python
from heapq import heappush, heappop

def min_meeting_rooms(intervals):
    intervals.sort()
    heap = []

    for start, end in intervals:
        if heap and heap[0] <= start:
            heappop(heap)

        heappush(heap, end)

    return len(heap)
```

---

## Sweep line

Use events:

```python
def max_overlapping(intervals):
    events = []

    for start, end in intervals:
        events.append((start, 1))
        events.append((end, -1))

    events.sort()

    cur = 0
    best = 0

    for _, delta in events:
        cur += delta
        best = max(best, cur)

    return best
```

Tie handling matters. For closed intervals `[start, end]`, process starts before ends at same coordinate. For half-open intervals `[start, end)`, process ends before starts.

---

# 15. Pattern 7: Binary Search

Binary search is not just “find target.” It is a framework for eliminating half the search space.

## Standard search

```python
def binary_search(nums, target):
    l, r = 0, len(nums) - 1

    while l <= r:
        mid = (l + r) // 2

        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            l = mid + 1
        else:
            r = mid - 1

    return -1
```

---

## Lower bound

First index where `nums[i] >= target`.

```python
def lower_bound(nums, target):
    l, r = 0, len(nums)

    while l < r:
        mid = (l + r) // 2

        if nums[mid] < target:
            l = mid + 1
        else:
            r = mid

    return l
```

Equivalent:

```python
from bisect import bisect_left

i = bisect_left(nums, target)
```

---

## Upper bound

First index where `nums[i] > target`.

```python
from bisect import bisect_right

i = bisect_right(nums, target)
```

---

## Binary search on answer

Use when answer is numeric and predicate is monotonic:

```text
can(x) == False, False, False, True, True, True
```

Find smallest feasible `x`:

```python
def binary_search_answer(lo, hi):
    while lo < hi:
        mid = (lo + hi) // 2

        if can(mid):
            hi = mid
        else:
            lo = mid + 1

    return lo
```

Example: Koko eating bananas.

```python
import math

def min_eating_speed(piles, h):
    def can(speed):
        hours = 0
        for p in piles:
            hours += (p + speed - 1) // speed
        return hours <= h

    lo, hi = 1, max(piles)

    while lo < hi:
        mid = (lo + hi) // 2

        if can(mid):
            hi = mid
        else:
            lo = mid + 1

    return lo
```

Ceiling division for positive integers:

```python
(a + b - 1) // b
```

---

# 16. Pattern 8: Stack

Use when:

```text
nested structure
previous greater/smaller
monotonic property
undo/reverse
parsing
```

## Valid parentheses

```python
def is_valid(s):
    stack = []
    match = {")": "(", "]": "[", "}": "{"}

    for ch in s:
        if ch in "([{":
            stack.append(ch)
        else:
            if not stack or stack[-1] != match[ch]:
                return False
            stack.pop()

    return not stack
```

---

## Monotonic stack: next greater element

```python
def next_greater(nums):
    res = [-1] * len(nums)
    stack = []  # indices

    for i, x in enumerate(nums):
        while stack and nums[stack[-1]] < x:
            j = stack.pop()
            res[j] = x

        stack.append(i)

    return res
```

---

## Largest rectangle in histogram

```python
def largest_rectangle_area(heights):
    stack = []  # pairs: (start_index, height)
    best = 0

    for i, h in enumerate(heights):
        start = i

        while stack and stack[-1][1] > h:
            idx, height = stack.pop()
            best = max(best, height * (i - idx))
            start = idx

        stack.append((start, h))

    n = len(heights)

    for idx, height in stack:
        best = max(best, height * (n - idx))

    return best
```

---

# 17. Pattern 9: Queue / BFS

## BFS template

```python
from collections import deque

def bfs(start):
    q = deque([start])
    visited = {start}

    while q:
        node = q.popleft()

        for nei in graph[node]:
            if nei not in visited:
                visited.add(nei)
                q.append(nei)
```

---

## Level-order BFS

```python
from collections import deque

def level_order(root):
    if not root:
        return []

    q = deque([root])
    res = []

    while q:
        level = []

        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)

            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)

        res.append(level)

    return res
```

Key idea:

```python
for _ in range(len(q)):
```

This freezes the current level size.

---

## Shortest path in grid

```python
from collections import deque

def shortest_path_grid(grid):
    rows, cols = len(grid), len(grid[0])
    q = deque([(0, 0, 0)])  # row, col, dist
    visited = {(0, 0)}

    dirs = [(1, 0), (-1, 0), (0, 1), (0, -1)]

    while q:
        r, c, d = q.popleft()

        if r == rows - 1 and c == cols - 1:
            return d

        for dr, dc in dirs:
            nr, nc = r + dr, c + dc

            if (
                0 <= nr < rows
                and 0 <= nc < cols
                and grid[nr][nc] == 0
                and (nr, nc) not in visited
            ):
                visited.add((nr, nc))
                q.append((nr, nc, d + 1))

    return -1
```

---

## 0-1 BFS

Use when edge weights are only `0` or `1`.

```python
from collections import deque
from math import inf

def zero_one_bfs(graph, start, n):
    dist = [inf] * n
    dist[start] = 0

    dq = deque([start])

    while dq:
        node = dq.popleft()

        for nei, w in graph[node]:
            nd = dist[node] + w

            if nd < dist[nei]:
                dist[nei] = nd

                if w == 0:
                    dq.appendleft(nei)
                else:
                    dq.append(nei)

    return dist
```

This is better than Dijkstra for 0/1 weights.

---

# 18. Pattern 10: Heap / Priority Queue

## Top K frequent

```python
from collections import Counter
from heapq import heappush, heappop

def top_k_frequent(nums, k):
    freq = Counter(nums)
    heap = []

    for x, count in freq.items():
        heappush(heap, (count, x))

        if len(heap) > k:
            heappop(heap)

    return [x for count, x in heap]
```

Time:

```text
O(n log k)
```

---

## K largest elements

```python
from heapq import heappush, heappop

def k_largest(nums, k):
    heap = []

    for x in nums:
        heappush(heap, x)
        if len(heap) > k:
            heappop(heap)

    return heap
```

The heap contains the k largest, not necessarily sorted.

---

## K-way merge

```python
from heapq import heappush, heappop

def merge_k_sorted_lists(lists):
    heap = []
    counter = 0

    for node in lists:
        if node:
            heappush(heap, (node.val, counter, node))
            counter += 1

    dummy = ListNode(0)
    cur = dummy

    while heap:
        _, _, node = heappop(heap)
        cur.next = node
        cur = cur.next

        if node.next:
            heappush(heap, (node.next.val, counter, node.next))
            counter += 1

    return dummy.next
```

Tie-breaker `counter` prevents comparison errors between `ListNode` objects.

---

## Median finder

```python
from heapq import heappush, heappop

class MedianFinder:
    def __init__(self):
        self.small = []  # max heap via negative values
        self.large = []  # min heap

    def addNum(self, num: int) -> None:
        heappush(self.small, -num)

        if self.small and self.large and -self.small[0] > self.large[0]:
            heappush(self.large, -heappop(self.small))

        if len(self.small) > len(self.large) + 1:
            heappush(self.large, -heappop(self.small))

        if len(self.large) > len(self.small):
            heappush(self.small, -heappop(self.large))

    def findMedian(self) -> float:
        if len(self.small) > len(self.large):
            return -self.small[0]

        return (-self.small[0] + self.large[0]) / 2
```

Invariant:

```text
small contains lower half
large contains upper half
len(small) >= len(large)
max(small) <= min(large)
```

---

# 19. Pattern 11: Linked Lists

Typical LeetCode definition:

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

## Reverse linked list

```python
def reverse_list(head):
    prev = None
    curr = head

    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt

    return prev
```

Interview explanation:

```text
Store next before rewiring current.next.
Move prev and curr forward.
```

---

## Find middle

```python
def middle_node(head):
    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow
```

---

## Detect cycle

```python
def has_cycle(head):
    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow is fast:
            return True

    return False
```

Use `is`, not `==`, because you want same object identity.

---

## Merge two sorted lists

```python
def merge_two_lists(l1, l2):
    dummy = ListNode()
    cur = dummy

    while l1 and l2:
        if l1.val <= l2.val:
            cur.next = l1
            l1 = l1.next
        else:
            cur.next = l2
            l2 = l2.next

        cur = cur.next

    cur.next = l1 or l2
    return dummy.next
```

---

## Remove nth from end

```python
def remove_nth_from_end(head, n):
    dummy = ListNode(0, head)
    fast = slow = dummy

    for _ in range(n):
        fast = fast.next

    while fast.next:
        fast = fast.next
        slow = slow.next

    slow.next = slow.next.next

    return dummy.next
```

Dummy nodes reduce edge-case bugs.

---

# 20. Pattern 12: Trees

Typical LeetCode tree:

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

## Recursive DFS

```python
def inorder(root):
    res = []

    def dfs(node):
        if not node:
            return

        dfs(node.left)
        res.append(node.val)
        dfs(node.right)

    dfs(root)
    return res
```

---

## Iterative inorder

```python
def inorder_iter(root):
    res = []
    stack = []
    curr = root

    while curr or stack:
        while curr:
            stack.append(curr)
            curr = curr.left

        curr = stack.pop()
        res.append(curr.val)
        curr = curr.right

    return res
```

---

## Max depth

```python
def max_depth(root):
    if not root:
        return 0

    return 1 + max(max_depth(root.left), max_depth(root.right))
```

---

## Diameter of binary tree

```python
def diameter_of_binary_tree(root):
    best = 0

    def height(node):
        nonlocal best

        if not node:
            return 0

        left = height(node.left)
        right = height(node.right)

        best = max(best, left + right)

        return 1 + max(left, right)

    height(root)
    return best
```

Use `nonlocal` to modify outer variable.

---

## Lowest common ancestor

For general binary tree:

```python
def lowest_common_ancestor(root, p, q):
    if not root or root is p or root is q:
        return root

    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)

    if left and right:
        return root

    return left or right
```

For BST:

```python
def lca_bst(root, p, q):
    while root:
        if p.val < root.val and q.val < root.val:
            root = root.left
        elif p.val > root.val and q.val > root.val:
            root = root.right
        else:
            return root
```

---

## Validate BST

Wrong approach:

```python
node.left.val < node.val < node.right.val
```

This only checks local children.

Correct:

```python
def is_valid_bst(root):
    def dfs(node, lo, hi):
        if not node:
            return True

        if not (lo < node.val < hi):
            return False

        return dfs(node.left, lo, node.val) and dfs(node.right, node.val, hi)

    return dfs(root, float("-inf"), float("inf"))
```

---

# 21. Pattern 13: Graphs

## Build adjacency list

Undirected:

```python
from collections import defaultdict

graph = defaultdict(list)

for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)
```

Directed:

```python
graph = defaultdict(list)

for u, v in edges:
    graph[u].append(v)
```

If nodes are `0..n-1`, list is faster:

```python
graph = [[] for _ in range(n)]

for u, v in edges:
    graph[u].append(v)
```

---

## DFS connected components

```python
def count_components(n, edges):
    graph = [[] for _ in range(n)]

    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    visited = set()

    def dfs(node):
        visited.add(node)

        for nei in graph[node]:
            if nei not in visited:
                dfs(nei)

    count = 0

    for node in range(n):
        if node not in visited:
            count += 1
            dfs(node)

    return count
```

Iterative to avoid recursion depth:

```python
def dfs_iter(start, graph, visited):
    stack = [start]
    visited.add(start)

    while stack:
        node = stack.pop()

        for nei in graph[node]:
            if nei not in visited:
                visited.add(nei)
                stack.append(nei)
```

---

## Topological sort: Kahn’s algorithm

```python
from collections import deque

def can_finish(num_courses, prerequisites):
    graph = [[] for _ in range(num_courses)]
    indegree = [0] * num_courses

    for course, pre in prerequisites:
        graph[pre].append(course)
        indegree[course] += 1

    q = deque([i for i in range(num_courses) if indegree[i] == 0])
    taken = 0

    while q:
        node = q.popleft()
        taken += 1

        for nei in graph[node]:
            indegree[nei] -= 1
            if indegree[nei] == 0:
                q.append(nei)

    return taken == num_courses
```

Use when:

```text
DAG
course schedule
build order
dependency resolution
```

---

## Topological sort with DFS cycle detection

```python
def can_finish(num_courses, prerequisites):
    graph = [[] for _ in range(num_courses)]

    for course, pre in prerequisites:
        graph[pre].append(course)

    state = [0] * num_courses
    # 0 = unvisited, 1 = visiting, 2 = done

    def dfs(node):
        if state[node] == 1:
            return False
        if state[node] == 2:
            return True

        state[node] = 1

        for nei in graph[node]:
            if not dfs(nei):
                return False

        state[node] = 2
        return True

    return all(dfs(i) for i in range(num_courses))
```

---

## Dijkstra

Use when:

```text
weighted graph
non-negative edge weights
shortest path
```

```python
from heapq import heappush, heappop
from math import inf

def dijkstra(n, graph, start):
    dist = [inf] * n
    dist[start] = 0

    heap = [(0, start)]

    while heap:
        d, node = heappop(heap)

        if d != dist[node]:
            continue

        for nei, w in graph[node]:
            nd = d + w

            if nd < dist[nei]:
                dist[nei] = nd
                heappush(heap, (nd, nei))

    return dist
```

Important:

```python
if d != dist[node]:
    continue
```

This skips stale heap entries.

---

## Bellman-Ford

Use when:

```text
negative edges
limited stops
detect negative cycles
```

For “cheapest flight within K stops”:

```python
from math import inf

def find_cheapest_price(n, flights, src, dst, k):
    dist = [inf] * n
    dist[src] = 0

    for _ in range(k + 1):
        new_dist = dist[:]

        for u, v, w in flights:
            if dist[u] != inf and dist[u] + w < new_dist[v]:
                new_dist[v] = dist[u] + w

        dist = new_dist

    return -1 if dist[dst] == inf else dist[dst]
```

Why copy?

```text
Each iteration uses paths with at most that many edges.
Without copy, you accidentally use more edges in same round.
```

---

# 22. Pattern 14: Union-Find / DSU

Use when:

```text
connectivity
components
cycle detection in undirected graph
Kruskal MST
dynamic merging
```

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.count = n

    def find(self, x):
        while x != self.parent[x]:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x

    def union(self, a, b):
        ra = self.find(a)
        rb = self.find(b)

        if ra == rb:
            return False

        if self.rank[ra] < self.rank[rb]:
            ra, rb = rb, ra

        self.parent[rb] = ra

        if self.rank[ra] == self.rank[rb]:
            self.rank[ra] += 1

        self.count -= 1
        return True
```

Cycle detection:

```python
def valid_tree(n, edges):
    if len(edges) != n - 1:
        return False

    dsu = DSU(n)

    for u, v in edges:
        if not dsu.union(u, v):
            return False

    return dsu.count == 1
```

---

# 23. Pattern 15: Trie

Use when:

```text
prefix matching
word search
autocomplete
dictionary of words
```

## Trie with dict nodes

```python
class Trie:
    def __init__(self):
        self.root = {}

    def insert(self, word):
        node = self.root

        for ch in word:
            node = node.setdefault(ch, {})

        node["#"] = True

    def search(self, word):
        node = self.root

        for ch in word:
            if ch not in node:
                return False
            node = node[ch]

        return "#" in node

    def startsWith(self, prefix):
        node = self.root

        for ch in prefix:
            if ch not in node:
                return False
            node = node[ch]

        return True
```

`"#"` marks end of word.

---

# 24. Pattern 16: Backtracking

Use when:

```text
generate all combinations/permutations/subsets
search decision tree
constraint satisfaction
```

## Subsets

```python
def subsets(nums):
    res = []
    path = []

    def backtrack(i):
        if i == len(nums):
            res.append(path[:])
            return

        path.append(nums[i])
        backtrack(i + 1)
        path.pop()

        backtrack(i + 1)

    backtrack(0)
    return res
```

Critical:

```python
res.append(path[:])
```

You must copy the path. Otherwise all results reference same list.

---

## Combinations

```python
def combine(n, k):
    res = []
    path = []

    def backtrack(start):
        if len(path) == k:
            res.append(path[:])
            return

        for x in range(start, n + 1):
            path.append(x)
            backtrack(x + 1)
            path.pop()

    backtrack(1)
    return res
```

Pruning:

```python
remaining_needed = k - len(path)
for x in range(start, n - remaining_needed + 2):
    ...
```

---

## Permutations

```python
def permute(nums):
    res = []
    path = []
    used = [False] * len(nums)

    def backtrack():
        if len(path) == len(nums):
            res.append(path[:])
            return

        for i, x in enumerate(nums):
            if used[i]:
                continue

            used[i] = True
            path.append(x)

            backtrack()

            path.pop()
            used[i] = False

    backtrack()
    return res
```

---

## Word search

```python
def exist(board, word):
    rows, cols = len(board), len(board[0])

    def dfs(r, c, i):
        if i == len(word):
            return True

        if (
            r < 0 or r == rows
            or c < 0 or c == cols
            or board[r][c] != word[i]
        ):
            return False

        temp = board[r][c]
        board[r][c] = "#"

        found = (
            dfs(r + 1, c, i + 1)
            or dfs(r - 1, c, i + 1)
            or dfs(r, c + 1, i + 1)
            or dfs(r, c - 1, i + 1)
        )

        board[r][c] = temp
        return found

    for r in range(rows):
        for c in range(cols):
            if dfs(r, c, 0):
                return True

    return False
```

Important:

```text
Mark visited.
Restore after recursion.
```

---

# 25. Pattern 17: Dynamic Programming

DP is not a syntax problem. It is a state-definition problem.

Always answer:

```text
1. What does dp state mean?
2. What choices/transitions exist?
3. What is the base case?
4. What is the answer?
5. Can space be optimized?
```

---

## Memoized recursion

```python
from functools import lru_cache

def fib(n):
    @lru_cache(None)
    def dp(i):
        if i <= 1:
            return i
        return dp(i - 1) + dp(i - 2)

    return dp(n)
```

---

## Climbing stairs

```python
def climb_stairs(n):
    if n <= 2:
        return n

    a, b = 1, 2

    for _ in range(3, n + 1):
        a, b = b, a + b

    return b
```

---

## House robber

```python
def rob(nums):
    prev2 = 0
    prev1 = 0

    for x in nums:
        cur = max(prev1, prev2 + x)
        prev2 = prev1
        prev1 = cur

    return prev1
```

State:

```text
prev1 = best up to previous house
prev2 = best up to house before previous
```

---

## Coin change minimum coins

```python
def coin_change(coins, amount):
    INF = amount + 1
    dp = [INF] * (amount + 1)
    dp[0] = 0

    for a in range(1, amount + 1):
        for coin in coins:
            if a >= coin:
                dp[a] = min(dp[a], dp[a - coin] + 1)

    return -1 if dp[amount] == INF else dp[amount]
```

---

## 0/1 Knapsack style

Each item used once:

```python
def can_partition(nums):
    total = sum(nums)

    if total % 2:
        return False

    target = total // 2
    dp = [False] * (target + 1)
    dp[0] = True

    for x in nums:
        for s in range(target, x - 1, -1):
            dp[s] = dp[s] or dp[s - x]

    return dp[target]
```

Why reverse loop?

```text
Reverse prevents using same item multiple times.
```

Unbounded knapsack loops forward:

```python
for coin in coins:
    for amount in range(coin, target + 1):
        ...
```

---

## Longest Increasing Subsequence O(n log n)

```python
from bisect import bisect_left

def length_of_lis(nums):
    tails = []

    for x in nums:
        i = bisect_left(tails, x)

        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x

    return len(tails)
```

Meaning:

```text
tails[i] = smallest possible tail of an increasing subsequence of length i + 1
```

---

## Grid DP

Unique paths:

```python
def unique_paths(m, n):
    dp = [1] * n

    for _ in range(1, m):
        for c in range(1, n):
            dp[c] += dp[c - 1]

    return dp[-1]
```

Minimum path sum:

```python
def min_path_sum(grid):
    rows, cols = len(grid), len(grid[0])
    dp = [float("inf")] * cols
    dp[0] = 0

    for r in range(rows):
        dp[0] += grid[r][0]

        for c in range(1, cols):
            dp[c] = min(dp[c], dp[c - 1]) + grid[r][c]

    return dp[-1]
```

---

## Interval DP

Use when:

```text
problem over ranges
merge stones
burst balloons
matrix chain multiplication
palindrome partitioning
```

Burst balloons:

```python
def max_coins(nums):
    arr = [1] + nums + [1]
    n = len(arr)

    dp = [[0] * n for _ in range(n)]

    for length in range(2, n):
        for left in range(0, n - length):
            right = left + length

            for last in range(left + 1, right):
                coins = arr[left] * arr[last] * arr[right]
                dp[left][right] = max(
                    dp[left][right],
                    dp[left][last] + coins + dp[last][right]
                )

    return dp[0][n - 1]
```

Mental model:

```text
Choose the last balloon burst inside interval (left, right).
```

---

# 26. Pattern 18: Bit Manipulation

## Single number

```python
def single_number(nums):
    ans = 0

    for x in nums:
        ans ^= x

    return ans
```

Because:

```text
x ^ x = 0
x ^ 0 = x
```

---

## Check bit

```python
if mask & (1 << i):
    ...
```

Set bit:

```python
mask |= 1 << i
```

Clear bit:

```python
mask &= ~(1 << i)
```

Toggle bit:

```python
mask ^= 1 << i
```

---

## Iterate subsets of mask

```python
sub = mask

while sub:
    # use sub
    sub = (sub - 1) & mask
```

Include empty subset:

```python
sub = mask
while True:
    # use sub
    if sub == 0:
        break
    sub = (sub - 1) & mask
```

---

# 27. Pattern 19: Design Problems

Design problems test API correctness, data structures, and invariants.

## LRU Cache

Use `OrderedDict` for interview speed:

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = OrderedDict()

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1

        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)

        self.cache[key] = value

        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)
```

Staff-level explanation:

```text
Hash map gives O(1) lookup.
Ordered dictionary maintains recency order.
move_to_end marks key as most recently used.
popitem(last=False) evicts least recently used.
```

Manual implementation may be requested. Know the invariant:

```text
dict: key -> node
doubly linked list: least recent near head, most recent near tail
```

---

## RandomizedSet

```python
import random

class RandomizedSet:
    def __init__(self):
        self.arr = []
        self.pos = {}

    def insert(self, val: int) -> bool:
        if val in self.pos:
            return False

        self.pos[val] = len(self.arr)
        self.arr.append(val)
        return True

    def remove(self, val: int) -> bool:
        if val not in self.pos:
            return False

        idx = self.pos[val]
        last = self.arr[-1]

        self.arr[idx] = last
        self.pos[last] = idx

        self.arr.pop()
        del self.pos[val]

        return True

    def getRandom(self) -> int:
        return random.choice(self.arr)
```

Key trick:

```text
Remove in O(1) by swapping with last element.
```

---

## TimeMap

```python
from collections import defaultdict
from bisect import bisect_right

class TimeMap:
    def __init__(self):
        self.store = defaultdict(list)

    def set(self, key: str, value: str, timestamp: int) -> None:
        self.store[key].append((timestamp, value))

    def get(self, key: str, timestamp: int) -> str:
        arr = self.store[key]

        i = bisect_right(arr, (timestamp, chr(255))) - 1

        if i >= 0:
            return arr[i][1]

        return ""
```

Why `(timestamp, chr(255))`?

```text
Tuples compare by first element, then second.
We need the rightmost pair with timestamp <= query timestamp.
```

Alternative clearer binary search:

```python
def get(self, key, timestamp):
    arr = self.store[key]
    l, r = 0, len(arr) - 1
    ans = ""

    while l <= r:
        mid = (l + r) // 2

        if arr[mid][0] <= timestamp:
            ans = arr[mid][1]
            l = mid + 1
        else:
            r = mid - 1

    return ans
```

---

# 28. Advanced Structures You Should Know

## Fenwick Tree / Binary Indexed Tree

Use when:

```text
point updates
prefix sums
range sum queries
count smaller elements
inversion count
```

```python
class Fenwick:
    def __init__(self, n):
        self.n = n
        self.bit = [0] * (n + 1)

    def add(self, i, delta):
        i += 1
        while i <= self.n:
            self.bit[i] += delta
            i += i & -i

    def sum(self, i):
        i += 1
        total = 0
        while i > 0:
            total += self.bit[i]
            i -= i & -i
        return total

    def range_sum(self, l, r):
        return self.sum(r) - (self.sum(l - 1) if l > 0 else 0)
```

---

## Segment Tree

Use when:

```text
range queries + updates
min/max/sum/gcd over range
```

Iterative segment tree for range sum:

```python
class SegmentTree:
    def __init__(self, nums):
        self.n = len(nums)
        self.tree = [0] * (2 * self.n)

        for i in range(self.n):
            self.tree[self.n + i] = nums[i]

        for i in range(self.n - 1, 0, -1):
            self.tree[i] = self.tree[2 * i] + self.tree[2 * i + 1]

    def update(self, idx, val):
        i = idx + self.n
        self.tree[i] = val

        i //= 2
        while i:
            self.tree[i] = self.tree[2 * i] + self.tree[2 * i + 1]
            i //= 2

    def query(self, l, r):
        # inclusive [l, r]
        l += self.n
        r += self.n
        total = 0

        while l <= r:
            if l % 2 == 1:
                total += self.tree[l]
                l += 1

            if r % 2 == 0:
                total += self.tree[r]
                r -= 1

            l //= 2
            r //= 2

        return total
```

---

# 29. Python Class Patterns for LeetCode

## Simple class

```python
class Solution:
    def solve(self, nums: list[int]) -> int:
        ...
```

## Nested helper function

```python
class Solution:
    def maxDepth(self, root):
        def dfs(node):
            if not node:
                return 0
            return 1 + max(dfs(node.left), dfs(node.right))

        return dfs(root)
```

## Use `self` only when needed

```python
class Solution:
    def diameterOfBinaryTree(self, root):
        self.best = 0

        def height(node):
            if not node:
                return 0

            left = height(node.left)
            right = height(node.right)

            self.best = max(self.best, left + right)
            return 1 + max(left, right)

        height(root)
        return self.best
```

Alternative:

```python
best = 0

def height(node):
    nonlocal best
    ...
```

Use `nonlocal` for enclosing function variables. Use `self` for object fields.

---

# 30. The Staff-Level Coding Interview Bar

At staff level, they are not just checking whether you memorized templates. They are checking whether you can drive ambiguity to a clean model.

Your answer should sound like this:

```text
Let me restate the problem.
I’ll clarify constraints and edge cases.
The brute force is O(...).
The bottleneck is ...
I can improve it with ...
The invariant is ...
I’ll implement the simpler correct version first.
Then I’ll test edge cases.
```

## What to say before coding

Example:

```text
We need longest subarray with sum k.
Brute force checks all O(n^2) subarrays.
Because subarray sums can be represented as prefix sums, 
if prefix[j] - prefix[i] = k, then prefix[i] = prefix[j] - k.
So I’ll scan once and store earliest index of each prefix sum.
That gives O(n) time and O(n) space.
```

This is staff-level communication: model, invariant, complexity.

---

# 31. Problem Recognition Map

| Problem Clue                          | Likely Pattern                             |
| ------------------------------------- | ------------------------------------------ |
| “subarray”, “substring”, “contiguous” | sliding window or prefix sum               |
| “longest substring with at most…”     | sliding window                             |
| “sum equals k” with negatives         | prefix sum hashmap                         |
| “sorted array”                        | binary search / two pointers               |
| “k largest / smallest”                | heap                                       |
| “top k frequent”                      | Counter + heap or bucket sort              |
| “merge k sorted”                      | heap                                       |
| “intervals”                           | sort + merge / sweep line                  |
| “minimum number of rooms/resources”   | heap or sweep line                         |
| “shortest path unweighted”            | BFS                                        |
| “shortest path weighted nonnegative”  | Dijkstra                                   |
| “dependencies / course schedule”      | topological sort                           |
| “connect components dynamically”      | DSU                                        |
| “prefix search / dictionary words”    | trie                                       |
| “all combinations/permutations”       | backtracking                               |
| “optimal choice over sequence”        | DP                                         |
| “range update many times”             | difference array                           |
| “range query with updates”            | Fenwick/segment tree                       |
| “next greater/smaller”                | monotonic stack                            |
| “sliding max/min”                     | monotonic deque                            |
| “palindrome / pair in sorted”         | two pointers                               |
| “cycle in linked list”                | fast/slow pointers                         |
| “copy random linked list”             | hashmap or interweaving                    |
| “least recently used”                 | hashmap + doubly linked list / OrderedDict |

---

# 32. Edge Cases Checklist

Before finalizing, test mentally:

```text
empty input
one element
two elements
all same values
duplicates
negative numbers
zero
very large values
already sorted
reverse sorted
disconnected graph
cycle graph
single-node tree
skewed tree
target absent
k = 0
k = n
overlapping intervals at boundary
```

For Python-specific testing:

```text
Does slicing copy too much?
Am I using pop(0)?
Did I mutate shared rows in a 2D list?
Did I forget to copy path in backtracking?
Did I use // with negative numbers?
Did I rely on recursion too deeply?
Did heap tie-breaking compare non-comparable objects?
```

---

# 33. Python One-Liners That Are Actually Useful

Use only when readable.

```python
nums = list(map(int, input().split()))
```

```python
freq = Counter(nums)
```

```python
graph = [[] for _ in range(n)]
```

```python
grid = [[0] * cols for _ in range(rows)]
```

```python
pairs.sort(key=lambda x: (x[0], -x[1]))
```

```python
return all(x > 0 for x in nums)
```

```python
return any(word in s for word in words)
```

```python
res = [x * x for x in nums if x > 0]
```

But avoid clever unreadable one-liners in interviews. Clear code beats flashy code.

---

# 34. Python Built-ins You Must Know Cold

```python
len(x)
range(n)
enumerate(arr)
zip(a, b)
sum(arr)
min(arr)
max(arr)
sorted(arr)
reversed(arr)
any(iterable)
all(iterable)
abs(x)
ord(ch)
chr(num)
```

## `enumerate`

```python
for i, x in enumerate(nums):
    ...
```

## `zip`

```python
for a, b in zip(arr1, arr2):
    ...
```

## Reverse

```python
for x in reversed(nums):
    ...
```

`reversed(nums)` returns an iterator.

To get list:

```python
list(reversed(nums))
```

For strings:

```python
s[::-1]
```

But slicing copies.

---

# 35. Python Math Tools

```python
import math

math.gcd(a, b)
math.lcm(a, b)
math.inf
math.sqrt(x)
math.isqrt(x)
```

Integer square root:

```python
math.isqrt(10)    # 3
```

Useful for primality:

```python
def is_prime(n):
    if n < 2:
        return False

    d = 2
    while d * d <= n:
        if n % d == 0:
            return False
        d += 1

    return True
```

---

# 36. Java Habits to Drop Immediately

## Do not write verbose classes for everything

Java instinct:

```java
class Pair {
    int x;
    int y;
}
```

Python interview style:

```python
(x, y)
```

Or:

```python
row, col, dist = item
```

---

## Do not manually count if `Counter` is clearer

Manual is fine, but this is faster to write:

```python
freq = Counter(nums)
```

---

## Do not simulate Java `TreeMap` with sorted list blindly

This:

```python
from bisect import insort

insort(arr, x)
```

is O(n) insertion, not O(log n), because Python list insertion shifts elements. The official `bisect` docs explicitly note that the O(log n) search is dominated by the O(n) insertion step for `insort`. ([Python documentation][3])

For ordered-map problems, consider:

```text
heap
two heaps
monotonic deque
offline sorting
Fenwick tree
segment tree
coordinate compression
```

---

## Do not overuse recursion

Java recursion may pass where Python fails due to recursion limit. For graph DFS, iterative is often safer.

---

# 37. Full Template Library

## Graph adjacency

```python
def build_graph(n, edges, directed=False):
    graph = [[] for _ in range(n)]

    for u, v in edges:
        graph[u].append(v)
        if not directed:
            graph[v].append(u)

    return graph
```

---

## Grid directions

```python
DIRS4 = [(1, 0), (-1, 0), (0, 1), (0, -1)]
DIRS8 = [
    (1, 0), (-1, 0), (0, 1), (0, -1),
    (1, 1), (1, -1), (-1, 1), (-1, -1)
]
```

---

## Bounds check

```python
def inside(r, c, rows, cols):
    return 0 <= r < rows and 0 <= c < cols
```

---

## Backtracking skeleton

```python
def backtrack(state):
    if is_solution(state):
        res.append(build_answer(state))
        return

    for choice in choices(state):
        if invalid(choice):
            continue

        make(choice)
        backtrack(state)
        undo(choice)
```

---

## Memoized DP skeleton

```python
from functools import lru_cache

@lru_cache(None)
def dp(i, state):
    if base_case:
        return value

    ans = initial

    for choice in choices:
        ans = best(ans, transition)

    return ans
```

---

## Binary search answer skeleton

```python
def solve():
    lo, hi = min_possible, max_possible

    while lo < hi:
        mid = (lo + hi) // 2

        if feasible(mid):
            hi = mid
        else:
            lo = mid + 1

    return lo
```

---

## Dijkstra skeleton

```python
from heapq import heappush, heappop
from math import inf

def dijkstra(graph, start, n):
    dist = [inf] * n
    dist[start] = 0
    heap = [(0, start)]

    while heap:
        d, node = heappop(heap)

        if d != dist[node]:
            continue

        for nei, w in graph[node]:
            nd = d + w

            if nd < dist[nei]:
                dist[nei] = nd
                heappush(heap, (nd, nei))

    return dist
```

---

# 38. Mini Interview Walkthrough Example

Problem:

```text
Given an array nums and integer k, return the number of subarrays with sum k.
```

Staff-level reasoning:

```text
Brute force is O(n^2): check all subarrays.
A subarray sum from i+1 to j equals prefix[j] - prefix[i].
We need prefix[i] = prefix[j] - k.
So while scanning, count how many previous prefix sums equal current_prefix - k.
Use hashmap frequency.
```

Code:

```python
from collections import defaultdict

def subarray_sum(nums, k):
    count = defaultdict(int)
    count[0] = 1

    prefix = 0
    ans = 0

    for x in nums:
        prefix += x

        ans += count[prefix - k]

        count[prefix] += 1

    return ans
```

Test:

```python
nums = [1, 1, 1], k = 2
prefix scan:
0 seen once
prefix 1: need -1 -> 0
prefix 2: need 0 -> +1
prefix 3: need 1 -> +1
answer = 2
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

This is exactly how you should present solutions.

---

# 39. The Final Python Interview Cheat Sheet

## Imports

```python
from collections import defaultdict, Counter, deque, OrderedDict
from heapq import heappush, heappop, heapify
from bisect import bisect_left, bisect_right
from functools import lru_cache
from math import inf, gcd, isqrt
```

## Data structure choices

```text
Need stack                 -> list
Need queue                 -> deque
Need frequency             -> Counter / defaultdict(int)
Need grouping              -> defaultdict(list)
Need visited               -> set
Need priority queue        -> heapq
Need ordered binary search -> bisect, but insertion is O(n)
Need prefix matching       -> trie
Need connectivity          -> DSU
Need range sum updates     -> Fenwick / segment tree
Need memoized recursion    -> lru_cache(None)
```

## Complexity traps

```text
pop(0) is O(n)
insert into list is O(n)
slicing copies
string += in loop is bad
sort is O(n log n)
bisect insertion is O(n)
recursion may hit limit
heap ties may compare bad objects
```

## Most common patterns

```text
Hash map
Two pointers
Sliding window
Prefix sum
Binary search
Stack
Monotonic stack
Deque BFS
Heap
Intervals
Graph DFS/BFS
Topological sort
Dijkstra
Union-Find
Trie
Backtracking
Dynamic programming
Fenwick/segment tree
```

---

# 40. What You Should Practice in Order

For fastest FAANG+ readiness as a Java dev, do this order:

```text
1. Python syntax + built-ins
2. Arrays/hash maps/two pointers/sliding window
3. Binary search + prefix sums
4. Stack/monotonic stack/heap
5. Linked list/tree basics
6. Graph BFS/DFS/toposort/DSU/Dijkstra
7. Backtracking
8. DP
9. Design problems
10. Advanced structures: Fenwick, segment tree, trie
```

Problem types to master:

```text
Two Sum
Group Anagrams
Longest Substring Without Repeating
Minimum Window Substring
Subarray Sum Equals K
Merge Intervals
Meeting Rooms II
Binary Search on Answer
Kth Largest
Top K Frequent
Merge K Lists
LRU Cache
RandomizedSet
Reverse Linked List
Cycle Linked List
Tree Diameter
LCA
Validate BST
Number of Islands
Course Schedule
Dijkstra shortest path
Union-Find components
Word Search
Combination Sum
House Robber
Coin Change
LIS
Partition Equal Subset Sum
Burst Balloons
```

---

# 41. Final Rulebook for Interview Python

Write code like this:

```text
simple
explicit
correct
asymptotically clean
easy to explain
```

Avoid code like this:

```text
too clever
too compressed
dependent on obscure Python behavior
full of hidden copies
using wrong data structure
```

When stuck, ask:

```text
Can I sort?
Can I hash?
Can I use two pointers?
Is this a sliding window?
Is this prefix sum?
Is this binary search on answer?
Is this graph traversal?
Is this DP state?
Can I maintain a heap?
Can I process events in sorted order?
```

That question list alone solves a huge percentage of coding rounds.

The goal is not to “write Python.” The goal is to express the right algorithm with fewer bugs than your Java implementation would have under time pressure.

[1]: https://docs.python.org/3/library/heapq.html?utm_source=chatgpt.com "heapq — Heap queue algorithm — Python 3.14.6 documentation"
[2]: https://docs.python.org/3/library/collections.html?utm_source=chatgpt.com "collections — Container datatypes — Python 3.14.5 documentation"
[3]: https://docs.python.org/3/library/bisect.html?utm_source=chatgpt.com "bisect — Array bisection algorithm — Python 3.14.6 documentation"
[4]: https://docs.python.org/3/library/functools.html?utm_source=chatgpt.com "functools — Higher-order functions and operations on ... - Python"
