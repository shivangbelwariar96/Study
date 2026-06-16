# Python for FAANG+ Staff Interviews

## Java Developer Edition: Python-Only Master Guide

This is the final Python-focused guide for a Java developer using Python in
FAANG+ coding interviews.

This is not a DSA pattern bank. It intentionally avoids algorithm problem
solutions and problem walkthroughs. Its job is narrower and more important:

```text
Make Python itself effortless.
Prevent Python-specific bugs.
Make your code look idiomatic under pressure.
Give you the runtime, complexity, and internals language expected from a senior/staff engineer.
Help you explain Python tradeoffs clearly in interviews.
```

Target interview runtime: Python 3.10+ unless the platform says otherwise.
Avoid brand-new version-specific features unless the judge explicitly supports
them.

---

## How to Read This Guide

Callouts are used consistently:

> **PITFALL:** A Python trap that can cause wrong answers, TLEs, crashes, or bad
> explanations.

> **JAVA DEV TRAP:** A habit that is natural in Java but harmful or noisy in
> Python.

> **TRICK:** A Python idiom that saves time without becoming clever.

> **INTERNAL:** CPython/runtime behavior worth knowing for staff-level
> credibility.

> **STAFF NOTE:** What a senior/staff interviewer is really listening for.

> **SAY THIS:** Interview-ready wording you can use out loud.

---

## Table of Contents

1. [The Goal](#1-the-goal)
2. [Python Version and Platform Reality](#2-python-version-and-platform-reality)
3. [Java to Python Translation](#3-java-to-python-translation)
4. [Core Language Semantics](#4-core-language-semantics)
5. [Functions, Scope, and Closures](#5-functions-scope-and-closures)
6. [Core Built-ins and Idioms](#6-core-built-ins-and-idioms)
7. [Data Structures and Complexity](#7-data-structures-and-complexity)
8. [Sorting, Comparison, and Ordering](#8-sorting-comparison-and-ordering)
9. [Numbers, Math, and Precision](#9-numbers-math-and-precision)
10. [Strings and Unicode](#10-strings-and-unicode)
11. [Iteration, Generators, and Comprehensions](#11-iteration-generators-and-comprehensions)
12. [Copying, Aliasing, and Mutability](#12-copying-aliasing-and-mutability)
13. [Caching and Memoization Mechanics](#13-caching-and-memoization-mechanics)
14. [Classes, Dataclasses, and Custom Objects](#14-classes-dataclasses-and-custom-objects)
15. [CPython Internals That Matter](#15-cpython-internals-that-matter)
16. [Performance and Memory Rules](#16-performance-and-memory-rules)
17. [Production Python: OOP, Context Managers, Decorators, and Generators](#17-production-python-oop-context-managers-decorators-and-generators)
18. [Concurrency: Threads, Locks, Processes, and Asyncio](#18-concurrency-threads-locks-processes-and-asyncio)
19. [The Complete Pitfall Catalog](#19-the-complete-pitfall-catalog)
20. [Staff-Level Python Communication](#20-staff-level-python-communication)
21. [Final Cheat Sheets](#21-final-cheat-sheets)
22. [Official References](#22-official-references)

---

# 1. The Goal

In the coding round, Python should disappear. The interviewer should see your
problem-solving, not your struggle with syntax, copies, hashability, heaps,
division, or recursion.

Your Python code should be:

```text
short but not cryptic
idiomatic but not clever
explicit about invariants
safe on edge cases
accurate in complexity
easy for a human to review
```

## 1.1 Staff-Level Python Bar

At staff level, the bar is not "can write Python syntax." The bar is:

```text
Can you choose the right Python data structure?
Can you explain its cost model?
Can you avoid hidden O(n) operations?
Can you reason about mutability and aliasing?
Can you communicate tradeoffs when constraints change?
Can your code be read, debugged, and modified quickly?
```

> **STAFF NOTE:** In a staff coding round, clean and correct beats clever and
> dense. Python gives you expressive tools. Use them to reduce bug surface, not
> to show off.

## 1.2 What This Guide Does Not Include

This guide intentionally does not include:

```text
algorithm templates
prewritten coding-problem answers
DSA pattern walkthroughs
problem lists
DP recurrence libraries
graph traversal solution code
```

Those are separate study materials. This document is your Python language,
runtime, standard-library, and interview-execution reference.

## 1.3 Complexity by Input Size

This is not a problem-solving section, but you should know how interviewers use
constraints to evaluate your Python choices:

```text
n <= 12        factorial or exhaustive search may be acceptable
n <= 20        2^n style state spaces may be acceptable
n <= 500       O(n^3) may be acceptable
n <= 5,000     O(n^2) may be acceptable
n <= 100,000   O(n log n) or O(n) is usually expected
n >= 1,000,000 O(n) or better is usually needed
```

> **SAY THIS:** "Given n is around 100,000, I want to avoid Python-level O(n^2)
> work and also avoid hidden O(n) operations like list front pops or repeated
> slicing."

---

# 2. Python Version and Platform Reality

## 2.1 Safe Baseline

Assume Python 3.10+ for interviews unless told otherwise.

Comfortable features:

```python
def solve(nums: list[int]) -> int:
    ...
```

```python
from functools import lru_cache
```

```python
match value:
    case 0:
        ...
```

That said, pattern matching is rarely necessary in coding interviews.

> **TIP:** Use Python 3.10 type hints like `list[int]` if the platform supports
> them. If uncertain, omit type hints or use `List[int]` from `typing`.

## 2.2 Version-Sensitive Features

| Feature | Version note | Interview guidance |
|---|---:|---|
| `list[int]` generic syntax | Python 3.9+ | Usually fine on modern judges |
| `functools.cache` | Python 3.9+ | Use `lru_cache(None)` if unsure |
| `match/case` | Python 3.10+ | Avoid unless it genuinely clarifies |
| `bisect` `key=` | Python 3.10+ | Know behavior, but avoid if platform older |
| `math.lcm` | Python 3.9+ | Easy to replace if unavailable |
| `heapq` max-heap helpers | Python 3.14+ | Avoid in interviews unless confirmed |

Python 3.14 added `heapq.heapify_max`, `heappush_max`,
`heappop_max`, `heappushpop_max`, and `heapreplace_max`. Most interview
platforms may not run 3.14 yet, so the safe max-heap idiom remains pushing
negative numeric priorities.

## 2.3 Third-Party Libraries

Python's standard library does not include a `TreeMap`, `TreeSet`, or
`SortedList`.

Some platforms include:

```python
from sortedcontainers import SortedList, SortedDict, SortedSet
```

But availability is not universal.

> **SAY THIS:** "In Java I would use `TreeMap`. In Python, if
> `sortedcontainers` is available, `SortedList` or `SortedDict` is the closest
> equivalent. If not, I would redesign around heaps, offline sorting, coordinate
> compression, or another structure depending on the operation mix."

> **PITFALL:** Do not assume `sortedcontainers` is available unless the
> interviewer or platform confirms it.

---

# 3. Java to Python Translation

## 3.1 Direct Translation Table

| Java | Python | Interview note |
|---|---|---|
| `int`, `long` | `int` | Arbitrary precision, no overflow |
| `double` | `float` | IEEE-style binary float, precision caveats |
| `boolean` | `bool` | `True` / `False`, capitalized |
| `null` | `None` | Compare with `is None` |
| `String` | `str` | Immutable Unicode string |
| `StringBuilder` | `list` + `"".join(parts)` | Avoid repeated `+=` |
| `int[]` | `list[int]` | Dynamic array of object references |
| `ArrayList<T>` | `list` | Append/pop-end are fast |
| `HashMap<K,V>` | `dict` | Average O(1), insertion-ordered |
| `HashSet<T>` | `set` | Average O(1) membership |
| `ArrayDeque<T>` | `collections.deque` | O(1) both ends |
| `PriorityQueue<T>` | `heapq` over `list` | Min-heap by default |
| `LinkedHashMap` | `dict` / `OrderedDict` | Use `OrderedDict` for recency ops |
| `TreeMap`, `TreeSet` | no stdlib equivalent | Confirm `sortedcontainers` or redesign |
| `Pair<A,B>` | tuple `(a, b)` | Hashable if contents hashable |
| `Comparator<T>` | `key=` or `cmp_to_key` | Prefer `key=` |
| `this` | `self` | Explicit first method parameter |
| `static final` constant | `UPPER_CASE` convention | Not enforced |
| `a / b` integer division | `a // b` | But floors, does not truncate |
| `&&`, `||`, `!` | `and`, `or`, `not` | Boolean operators are words |
| `for (T x : xs)` | `for x in xs:` | Use `enumerate` for index |

## 3.2 Java Habits to Drop

> **JAVA DEV TRAP:** Do not create tiny classes for pairs. Use tuples unless the
> object truly has behavior or named fields matter.

```python
cell = (row, col)
row, col = cell
```

> **JAVA DEV TRAP:** Do not use index loops by default.

Java-shaped:

```python
for i in range(len(nums)):
    x = nums[i]
```

More Pythonic when index is unnecessary:

```python
for x in nums:
    ...
```

When both are needed:

```python
for i, x in enumerate(nums):
    ...
```

> **JAVA DEV TRAP:** Do not simulate `TreeMap` by maintaining a sorted list with
> `bisect.insort` unless constraints are small. Insertion into a Python list is
> O(n) because elements shift.

> **JAVA DEV TRAP:** Do not assume recursion depth behaves like Java. CPython's
> default recursion limit is around 1000.

## 3.3 What Python Gives You That Java Does Not

Python makes these lightweight:

```text
tuple packing/unpacking
multiple assignment
dictionary and set literals
comprehensions
first-class functions
nested helper functions
lexicographic tuple sorting
arbitrary precision integers
standard-library counters, deques, heaps, bisection, caching
```

Use these to write less code with fewer moving parts.

---

# 4. Core Language Semantics

## 4.1 Names, Objects, and References

Python variables are names bound to objects.

```python
a = [1, 2]
b = a
b.append(3)
print(a)  # [1, 2, 3]
```

`b = a` does not copy. It creates another name for the same list.

> **INTERNAL:** In CPython, objects have identity, type, reference count, and
> value. Names live in namespaces and point to objects.

## 4.2 Mutability

Common immutable objects:

```text
int
float
bool
str
tuple, if all contained values are immutable
frozenset
None
```

Common mutable objects:

```text
list
dict
set
deque
bytearray
most custom objects
```

Immutable example:

```python
x = 10
y = x
y += 1
print(x)  # 10
```

Mutable example:

```python
a = []
b = a
b.append(1)
print(a)  # [1]
```

## 4.3 Equality vs Identity

Use `==` for value equality:

```python
if value == target:
    ...
```

Use `is` for identity:

```python
if node is None:
    ...
```

Use identity when you need the exact same object:

```python
if a is b:
    ...
```

> **PITFALL:** Never use `is` for numeric or string equality. CPython caches
> some small integers and may intern some strings, so `is` may appear to work
> until it does not.

## 4.4 Truthiness

Falsy values:

```text
False
None
0
0.0
""
[]
{}
set()
()
```

Idiomatic empty check:

```python
if not nums:
    return 0
```

But be precise when `0` or `""` is valid:

```python
if x is None:
    ...
```

> **PITFALL:** `if not x` means "x is falsy", not "x is missing."

## 4.5 Boolean Operators Return Operands

Python's `and` and `or` return one of the original operands, not necessarily a
boolean.

```python
result = "" or "default"   # "default"
result = "abc" and 123     # 123
```

This is useful:

```python
name = user_input or "anonymous"
```

But dangerous:

```python
count = user_count or 10
```

If `user_count` is `0`, this incorrectly chooses `10`.

## 4.6 Chained Comparisons

```python
if 0 <= i < n:
    ...
```

Equivalent to:

```python
if 0 <= i and i < n:
    ...
```

The middle expression is evaluated once.

## 4.7 Multiple Assignment

```python
a, b = b, a
```

```python
prev, curr = curr, curr.next
```

> **TRICK:** The right-hand side is evaluated before assignment. This makes
> swaps and state transitions clean.

## 4.8 Unpacking

```python
row, col = cell
```

```python
first, *middle, last = arr
```

```python
for key, value in d.items():
    ...
```

> **TIP:** Unpacking makes tuple-heavy Python code readable. Use descriptive
> names rather than indexing into tuples repeatedly.

---

# 5. Functions, Scope, and Closures

## 5.1 Function Basics

```python
def solve(nums: list[int], target: int) -> int:
    ...
```

Type hints improve readability but are not enforced at runtime.

```python
def f(x: int) -> int:
    return x

f("abc")  # runs unless your code fails inside
```

## 5.2 Default Arguments

Good:

```python
def f(limit: int = 10) -> int:
    return limit
```

Bad with mutable objects:

```python
def f(path=[]):
    path.append(1)
    return path
```

The list is created once, shared across calls.

Correct pattern:

```python
def f(path=None):
    if path is None:
        path = []
    path.append(1)
    return path
```

> **PITFALL:** Default argument expressions are evaluated once when the function
> is defined, not once per call.

## 5.3 Positional, Keyword, and Keyword-Only Arguments

```python
def connect(host, port, *, timeout=5):
    ...
```

`timeout` must be passed by name:

```python
connect("localhost", 8080, timeout=10)
```

This is useful in real code, but interview functions are usually simple.

## 5.4 Nested Helpers

Nested helpers are idiomatic in coding interviews because they keep helper state
close to the main function.

```python
def outer(nums):
    total = sum(nums)

    def helper(i):
        return nums[i] + total

    return helper(0)
```

## 5.5 `nonlocal` vs `global`

Use `nonlocal` to reassign a variable from the nearest enclosing function scope.

```python
def outer():
    best = 0

    def update(x):
        nonlocal best
        best = max(best, x)

    update(5)
    return best
```

Without `nonlocal`, assigning `best` inside `update` creates a new local
variable.

Use `global` only for module-level globals. Avoid it in interviews.

> **TIP:** If a helper only needs to mutate a list or dict, you do not need
> `nonlocal` because you are mutating the object, not rebinding the name.

## 5.6 Closure Late Binding

Closures capture variables, not their values at the time the function is
created.

Bad:

```python
funcs = []
for i in range(3):
    funcs.append(lambda: i)

print([f() for f in funcs])  # [2, 2, 2]
```

Good:

```python
funcs = []
for i in range(3):
    funcs.append(lambda i=i: i)
```

Rare in DSA code, but important Python knowledge.

## 5.7 Lambdas

Use lambdas for small key functions:

```python
items.sort(key=lambda x: x[1])
```

Avoid complex lambdas. Use a named helper when logic is non-trivial.

## 5.8 Exceptions: EAFP vs LBYL

Python often uses EAFP:

```text
Easier to Ask Forgiveness than Permission
```

Example:

```python
try:
    value = d[key]
except KeyError:
    value = 0
```

LBYL:

```python
if key in d:
    value = d[key]
else:
    value = 0
```

In interviews, LBYL is often clearer unless exception handling is central to
the task.

---

# 6. Core Built-ins and Idioms

## 6.1 Imports Worth Knowing

```python
from collections import defaultdict, Counter, deque, OrderedDict
from heapq import heappush, heappop, heappushpop, heapreplace, heapify, nlargest, nsmallest
from bisect import bisect_left, bisect_right, insort
from functools import cache, lru_cache, cmp_to_key
from math import inf, gcd, lcm, isqrt
from itertools import count, chain, product, pairwise, accumulate, combinations, permutations
from operator import itemgetter, attrgetter
```

Do not paste all imports blindly. Know what they are for.

## 6.2 Essential Built-ins

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

## 6.3 `enumerate`

```python
for i, x in enumerate(nums):
    ...
```

With custom start:

```python
for rank, item in enumerate(items, start=1):
    ...
```

## 6.4 `zip`

```python
for a, b in zip(left, right):
    ...
```

> **PITFALL:** `zip` stops at the shortest iterable.

```python
list(zip([1, 2], [3]))  # [(1, 3)]
```

If mismatched lengths matter, check them.

## 6.5 `any` and `all`

```python
if any(x < 0 for x in nums):
    ...
```

```python
if all(x >= 0 for x in nums):
    ...
```

They short-circuit.

> **TRICK:** Prefer generator expressions inside `any` and `all`; do not build
> a list unless you need it.

Good:

```python
any(x > 10 for x in nums)
```

Wasteful:

```python
any([x > 10 for x in nums])
```

## 6.6 `min` and `max`

```python
best = max(nums)
```

With key:

```python
longest = max(words, key=len)
```

Default for empty iterables:

```python
best = max(nums, default=0)
```

## 6.7 `sum`

Good for numbers:

```python
total = sum(nums)
```

Bad for list concatenation:

```python
flat = sum(list_of_lists, [])  # O(n^2)-ish behavior
```

Better:

```python
from itertools import chain

flat = list(chain.from_iterable(list_of_lists))
```

## 6.8 `reversed`

```python
for x in reversed(nums):
    ...
```

`reversed(nums)` returns an iterator, not a list.

Materialize if needed:

```python
rev = list(reversed(nums))
```

String reverse:

```python
s[::-1]
```

This copies.

## 6.9 `map` and `filter`

Common for parsing:

```python
nums = list(map(int, input().split()))
```

For readability, list comprehensions are often better:

```python
nums = [int(x) for x in input().split()]
```

## 6.10 `itertools`

Useful tools:

```python
from itertools import count, chain, product, pairwise, accumulate, combinations, permutations
```

Tie-break counter for heaps:

```python
from itertools import count

counter = count()
next(counter)
```

Cartesian product:

```python
for r, c in product(range(rows), range(cols)):
    ...
```

Flatten:

```python
flat = list(chain.from_iterable(matrix))
```

> **TIP:** Use `itertools` when it makes intent clearer. Avoid using it to hide
> simple logic from the interviewer.

## 6.11 `operator`

For sorting by tuple/object fields:

```python
from operator import itemgetter, attrgetter

pairs.sort(key=itemgetter(1))
users.sort(key=attrgetter("age"))
```

Equivalent lambdas are fine:

```python
pairs.sort(key=lambda x: x[1])
```

## 6.12 Coding-Round Python Tricks, Without Problem Solutions

This section is not an algorithm template library. It is the small Python
surface area that repeatedly appears while implementing algorithms.

### Tuple States

Use tuples for immutable compound state:

```python
state = (row, col, mask)
seen.add(state)
```

Why:

```text
tuples are lightweight
tuples are hashable if contents are hashable
tuples unpack cleanly
tuples compare lexicographically for sorting/heaps
```

> **PITFALL:** A tuple containing a list is still unhashable:

```python
hash((1, [2, 3]))  # TypeError
```

### Sentinel Values

Use explicit sentinels when `None` may be a valid value:

```python
MISSING = object()

value = d.get(key, MISSING)
if value is MISSING:
    ...
```

Use infinities for numeric best/worst:

```python
best = float("inf")
worst = float("-inf")
```

> **STAFF NOTE:** Sentinels make edge cases explicit. They avoid bugs where
> `0`, `""`, `False`, or `None` are valid data.

### `enumerate`, `zip`, and `pairwise`

```python
for i, x in enumerate(nums):
    ...
```

```python
for a, b in zip(left, right):
    ...
```

Python 3.10+:

```python
from itertools import pairwise

for prev, curr in pairwise(nums):
    ...
```

> **PITFALL:** `zip` truncates to the shortest input. `pairwise` yields nothing
> for fewer than two elements.

### Prefix-Like Running Values With `accumulate`

```python
from itertools import accumulate

running = list(accumulate(nums))
```

With an initial value:

```python
running = list(accumulate(nums, initial=0))
```

> **TIP:** `accumulate` is clean for simple running totals. For custom logic or
> interview clarity, a normal loop can be better.

### Heap Push-Pop Operations

```python
from heapq import heappushpop, heapreplace
```

`heappushpop(heap, x)` pushes `x`, then pops and returns the smallest item. It
is often faster than separate push then pop.

`heapreplace(heap, x)` pops the smallest item first, then pushes `x`. The heap
must be non-empty.

> **PITFALL:** `heappushpop` and `heapreplace` are not interchangeable. If the
> new item may be smaller than the current heap minimum, they return different
> results.

### `bisect_left` and `bisect_right`

```python
from bisect import bisect_left, bisect_right

lo = bisect_left(arr, x)
hi = bisect_right(arr, x)
```

Use this wording:

```text
bisect_left: first index with value >= x
bisect_right: first index with value > x
```

### `@cache`

```python
from functools import cache

@cache
def f(i, state):
    ...
```

Use only with hashable arguments.

> **PITFALL:** If `state` is a list, convert it to a tuple first.

### `math.isqrt`

```python
from math import isqrt

r = isqrt(n)
```

Use for exact integer square roots. It avoids float precision issues.

### `int.bit_count`

```python
ones = mask.bit_count()
```

Counts set bits in an integer.

### Bit Length

```python
bits = n.bit_length()
```

Returns the number of bits needed to represent a non-negative integer in
binary, excluding sign and leading zeros.

### Fixed-Size Count Arrays

When keys are known small integers or lowercase English letters, a list can be
faster and simpler than a dict:

```python
count = [0] * 26
idx = ord(ch) - ord("a")
count[idx] += 1
```

> **PITFALL:** This is only correct when the character set is guaranteed. For
> arbitrary Unicode or arbitrary keys, use a dict or `Counter`.

### When Not to Use Python Convenience

| Convenience | Avoid when |
|---|---|
| slicing | inside large loops/recursion where copy cost matters |
| recursion | depth can exceed the recursion limit |
| `defaultdict` | reading missing keys must not mutate the map |
| `Counter` | zero-count keys break distinct-count logic |
| `sortedcontainers` | platform availability is not confirmed |
| `bisect.insort` | you need true O(log n) ordered insertion |
| `deepcopy` | in a hot path |
| clever comprehensions | interviewer cannot inspect the logic quickly |

> **STAFF NOTE:** Staff signal is not using every Python feature. Staff signal
> is knowing exactly when a convenience turns into a hidden cost.

---

# 7. Data Structures and Complexity

## 7.1 Summary Table

| Structure | Use for | Key costs |
|---|---|---|
| `list` | array, stack, table | index O(1), append amortized O(1), pop-end O(1) |
| `tuple` | immutable record, key | index O(1), hashable if contents hashable |
| `str` | text | index O(1), slice O(k), concat copies |
| `dict` | hash map | avg lookup/insert/delete O(1) |
| `set` | membership, dedup | avg add/remove/in O(1) |
| `deque` | queue, both-end ops | append/pop both ends O(1) |
| `Counter` | frequency/multiset | dict-like |
| `defaultdict` | grouping/counting | dict-like, auto-creates on missing access |
| `heapq` | priority queue | push/pop O(log n), peek O(1), heapify O(n) |
| `bisect` | binary search sorted list | search O(log n), insertion O(n) |
| `OrderedDict` | recency order | O(1) move-to-end/pop front |

## 7.2 `list`

Python `list` is a dynamic array of object references.

Fast:

```python
arr[i]
arr.append(x)
arr.pop()
arr[-1]
```

Potentially expensive:

```python
arr.pop(0)
arr.insert(0, x)
x in arr
arr[a:b]
```

Costs:

| Operation | Cost |
|---|---:|
| index read/write | O(1) |
| append | amortized O(1) |
| pop end | O(1) |
| pop front | O(n) |
| insert/delete middle | O(n) |
| membership | O(n) |
| slice length k | O(k) |
| sort | O(n log n) |

> **INTERNAL:** CPython lists over-allocate capacity, so repeated append is
> amortized O(1). Front or middle operations still shift references.

## 7.3 `tuple`

Tuples are immutable sequences.

Use them for:

```text
coordinates
compound dictionary keys
heap records
multiple return values
lightweight records
```

```python
point = (row, col)
```

Hashable only if all contents are hashable:

```python
hash((1, "a"))      # ok
hash((1, []))       # TypeError
```

Tuple comparison is lexicographic:

```python
(1, 2) < (1, 3)  # True
```

## 7.4 `dict`

Python `dict` is a hash table.

```python
d = {}
d[key] = value
value = d[key]
```

Safe lookup:

```python
value = d.get(key, default)
```

Membership tests keys:

```python
if key in d:
    ...
```

Iteration:

```python
for key in d:
    ...

for key, value in d.items():
    ...
```

> **INTERNAL:** Modern Python dicts preserve insertion order as a language
> guarantee. They are still not sorted by key.

> **PITFALL:** Mutating a dict while iterating over it can raise an error or
> produce bad logic. Iterate over `list(d)` if you need to delete keys.

## 7.5 `set`

```python
seen = set()
seen.add(x)
if x in seen:
    ...
```

Empty set:

```python
seen = set()
```

Not:

```python
seen = {}  # dict
```

Set operations:

```python
a & b   # intersection
a | b   # union
a - b   # difference
a ^ b   # symmetric difference
```

Remove:

```python
seen.remove(x)   # KeyError if absent
seen.discard(x)  # no error if absent
```

> **PITFALL:** Set iteration order is not a sorted order. Do not rely on it for
> deterministic sorted output.

## 7.6 `deque`

```python
from collections import deque

q = deque()
q.append(x)
q.appendleft(x)
q.pop()
q.popleft()
```

All four end operations are O(1).

Use `deque` for queues and both-end workloads.

> **PITFALL:** `deque` is not a replacement for random-access arrays. Indexing
> into the middle is not what it is optimized for.

## 7.7 `Counter`

```python
from collections import Counter

freq = Counter(items)
```

Useful operations:

```python
freq[x]
freq.most_common(k)
freq.update(items)
freq.subtract(items)
```

Important behavior:

```python
freq = Counter("a")
freq["a"] -= 1
print(freq)  # Counter({'a': 0})
```

Zero counts may remain.

> **PITFALL:** If you use `len(freq)` to mean number of positive-count keys,
> delete keys when count reaches zero.

Counter arithmetic drops non-positive counts:

```python
Counter(a=2) - Counter(a=2)  # Counter()
```

This differs from direct assignment/subtraction.

## 7.8 `defaultdict`

```python
from collections import defaultdict

groups = defaultdict(list)
groups[key].append(value)

count = defaultdict(int)
count[x] += 1
```

> **PITFALL:** Pass the factory, not a value.

Correct:

```python
defaultdict(list)
```

Wrong:

```python
defaultdict([])
```

> **PITFALL:** Reading a missing key creates it.

```python
d = defaultdict(list)
d["missing"]
print(d)  # now has "missing"
```

Use `key in d` or `d.get(key)` if you do not want mutation.

## 7.9 `OrderedDict`

Modern dicts preserve insertion order, but `OrderedDict` has explicit recency
operations.

```python
from collections import OrderedDict

d = OrderedDict()
d.move_to_end(key)
d.popitem(last=False)
```

Use when the operation is about moving keys to the end or popping the oldest.

## 7.10 `heapq`

`heapq` implements a min-heap over a list.

```python
from heapq import heappush, heappop, heapify

heap = []
heappush(heap, item)
item = heappop(heap)
smallest = heap[0]
```

Costs:

| Operation | Cost |
|---|---:|
| push | O(log n) |
| pop | O(log n) |
| peek | O(1) |
| heapify | O(n) |

Max-heap for numeric priorities:

```python
heappush(heap, -x)
x = -heappop(heap)
```

> **PITFALL:** Tuple heap entries compare field by field. If priorities tie,
> Python compares the next field.

Risky:

```python
heappush(heap, (priority, obj))
```

Safe:

```python
from itertools import count

counter = count()
heappush(heap, (priority, next(counter), obj))
```

> **TIP:** `heapq.nlargest(k, items)` and `heapq.nsmallest(k, items)` are clean
> for small `k`. For large `k`, sorting may be simpler and faster.

> **PITFALL:** `heapq` has no built-in efficient delete or decrease-key. Use
> lazy deletion with an auxiliary dict/set when needed.

## 7.11 `bisect`

```python
from bisect import bisect_left, bisect_right

i = bisect_left(arr, x)   # first index where arr[i] >= x
j = bisect_right(arr, x)  # first index where arr[i] > x
```

Count equal values:

```python
count_x = bisect_right(arr, x) - bisect_left(arr, x)
```

`bisect` works on sorted lists.

> **PITFALL:** `insort` is O(n), not O(log n), because list insertion shifts
> elements. The binary search part is O(log n), but insertion dominates.

> **PITFALL:** The `key=` parameter added in Python 3.10 is applied to array
> elements for searching, but not to the raw `x` value in the same way you may
> expect. For complex cases, precompute keys or be explicit.

> **PITFALL:** The `bisect` functions are not designed for concurrent mutation
> of the same list by multiple threads.

## 7.12 `sortedcontainers`

If available:

```python
from sortedcontainers import SortedList, SortedDict, SortedSet
```

They fill the Java `TreeMap` / `TreeSet` gap.

Use only after confirming availability.

> **SAY THIS:** "Python's standard library does not have a sorted map. If
> `sortedcontainers` is allowed, I can use it. Otherwise I need a different
> design."

---

# 8. Sorting, Comparison, and Ordering

## 8.1 `sort` vs `sorted`

In-place:

```python
arr.sort()
```

New list:

```python
b = sorted(arr)
```

> **PITFALL:** `arr.sort()` returns `None`.

Bad:

```python
arr = arr.sort()
```

Good:

```python
arr.sort()
```

## 8.2 `key=`

```python
items.sort(key=lambda x: x.score)
```

Tuples for multiple keys:

```python
items.sort(key=lambda x: (x.primary, x.secondary))
```

Descending numeric secondary:

```python
items.sort(key=lambda x: (x.primary, -x.score))
```

## 8.3 Stable Sort

Python sort is stable: equal keys keep their original relative order.

This allows multi-pass sorting:

```python
items.sort(key=lambda x: x.secondary)
items.sort(key=lambda x: x.primary)
```

After the second sort, items are sorted by primary, with secondary order
preserved inside equal primary groups.

> **INTERNAL:** Python uses Timsort, which is stable and adaptive. It can take
> advantage of existing order in the data.

## 8.4 Tuple Ordering

Tuples compare lexicographically:

```python
(1, 2, 9) < (1, 3, 0)  # True
```

This is useful in sorting and heaps.

> **PITFALL:** Tuple comparison can accidentally compare objects you did not
> intend to compare. Add a numeric tie-breaker when needed.

## 8.5 `cmp_to_key`

Python sorting prefers `key=`, not comparator functions. If true pairwise
comparison is needed:

```python
from functools import cmp_to_key

def cmp(a, b):
    if a < b:
        return -1
    if a > b:
        return 1
    return 0

items.sort(key=cmp_to_key(cmp))
```

> **TIP:** Use `cmp_to_key` rarely. Most interview sorting is cleaner with
> `key=`.

## 8.6 `operator.itemgetter` and `attrgetter`

```python
from operator import itemgetter, attrgetter

pairs.sort(key=itemgetter(1))
users.sort(key=attrgetter("age"))
```

These are clean alternatives to simple lambdas.

## 8.7 Custom Object Ordering

If objects need ordering:

```python
from dataclasses import dataclass

@dataclass(order=True)
class Item:
    priority: int
    name: str
```

For heap entries where only priority should compare:

```python
from dataclasses import dataclass, field
from typing import Any

@dataclass(order=True)
class PrioritizedItem:
    priority: int
    item: Any = field(compare=False)
```

> **STAFF NOTE:** In interviews, tuple entries are often simpler than custom
> ordering classes. Use a class only if it improves clarity.

---

# 9. Numbers, Math, and Precision

## 9.1 Integers

Python `int` has arbitrary precision.

```python
10 ** 100
```

No overflow:

```python
2**63
```

> **JAVA DEV TRAP:** If a problem expects 32-bit overflow behavior, Python will
> not overflow for you. You must check bounds manually.

Common bounds:

```python
INT_MIN = -2**31
INT_MAX = 2**31 - 1
```

## 9.2 Division

```python
7 / 2    # 3.5
7 // 2   # 3
-7 // 2  # -4
```

Python `//` floors toward negative infinity.

Java integer division truncates toward zero.

For truncation toward zero:

```python
def trunc_div(a: int, b: int) -> int:
    sign = -1 if (a < 0) ^ (b < 0) else 1
    return sign * (abs(a) // abs(b))
```

> **PITFALL:** Do not use `int(a / b)` for large integers. `/` creates a float
> and can lose precision.

## 9.3 Modulo

```python
7 % 3    # 1
-7 % 3   # 2
```

Python modulo result has the sign of the divisor.

This is often convenient for normalization:

```python
i = (i + delta) % n
```

## 9.4 Ceiling Division

For positive integers:

```python
ceil = (a + b - 1) // b
```

General integer ceiling division:

```python
ceil = -(-a // b)
```

## 9.5 `pow`

```python
pow(a, b)
```

Modular exponentiation:

```python
pow(a, b, mod)
```

This is efficient and avoids huge intermediate values.

## 9.6 `math`

```python
from math import inf, gcd, lcm, isqrt
```

Integer square root:

```python
r = isqrt(n)
```

> **PITFALL:** Prefer `math.isqrt` over `int(math.sqrt(n))` for integer
> arithmetic. It avoids floating-point precision issues.

## 9.7 Floating Point

```python
0.1 + 0.2 == 0.3  # False
```

Compare with tolerance:

```python
abs(a - b) < 1e-9
```

Infinity:

```python
from math import inf

best = inf
worst = -inf
```

## 9.8 NaN

```python
nan = float("nan")
nan == nan  # False
```

> **PITFALL:** NaN is not equal to itself. If floats are involved, know whether
> NaN can appear.

## 9.9 Decimal and Fraction

For exact decimal arithmetic:

```python
from decimal import Decimal
```

For exact rational arithmetic:

```python
from fractions import Fraction
```

Rare in coding interviews, but useful to know.

---

# 10. Strings and Unicode

## 10.1 Strings Are Immutable

Bad for repeated building:

```python
s = ""
for ch in chars:
    s += ch
```

Better:

```python
parts = []
for ch in chars:
    parts.append(ch)

s = "".join(parts)
```

> **PITFALL:** Repeated string concatenation can create many intermediate
> strings. Use list accumulation and join for large builds.

## 10.2 Common String Methods

```python
s.lower()
s.upper()
s.casefold()
s.strip()
s.split()
s.split(",")
s.join(parts)
s.startswith(prefix)
s.endswith(suffix)
s.find(sub)
s.count(sub)
s.isalpha()
s.isdigit()
s.isalnum()
```

`find` returns `-1` when absent.

## 10.3 `ord` and `chr`

```python
ord("a")  # 97
chr(97)   # "a"
```

Lowercase English index:

```python
i = ord(ch) - ord("a")
```

> **PITFALL:** This only works for known lowercase English letters. Ask about
> character set if the prompt is ambiguous.

## 10.4 Slicing

```python
s[::-1]
s[l:r]
```

Slicing creates a new string.

> **PITFALL:** Slicing inside loops or recursion can silently change complexity.

## 10.5 Unicode

Python strings are Unicode.

```python
len("a")  # 1
```

But user-perceived characters can be more complex than code points.

Case-insensitive comparison:

```python
s.casefold()
```

`casefold` is more aggressive than `lower` for Unicode.

> **STAFF NOTE:** Most coding interviews use ASCII/lowercase assumptions. If
> not stated, ask. Unicode correctness can change the design.

---

# 11. Iteration, Generators, and Comprehensions

## 11.1 Iterables vs Iterators

An iterable can produce an iterator:

```python
for x in nums:
    ...
```

An iterator is consumed as you use it:

```python
it = iter([1, 2, 3])
list(it)  # [1, 2, 3]
list(it)  # []
```

> **PITFALL:** `map`, `filter`, `zip`, `enumerate`, and many `itertools`
> objects are lazy iterators. Once consumed, they are empty.

## 11.2 `range`

```python
range(10)
```

`range` is lazy and memory-efficient.

Materialize only if needed:

```python
list(range(10))
```

## 11.3 List Comprehensions

```python
squares = [x * x for x in nums]
positives = [x for x in nums if x > 0]
```

2D initialization:

```python
grid = [[0] * cols for _ in range(rows)]
```

> **PITFALL:** Do not write `[[0] * cols] * rows`; it aliases rows.

## 11.4 Set and Dict Comprehensions

```python
seen = {x for x in nums}
index = {value: i for i, value in enumerate(nums)}
```

## 11.5 Generator Expressions

```python
total = sum(x * x for x in nums)
```

No list is built.

Good:

```python
any(x < 0 for x in nums)
```

Wasteful:

```python
any([x < 0 for x in nums])
```

## 11.6 Nested Comprehensions

Flatten:

```python
flat = [x for row in matrix for x in row]
```

Read order is the same as nested loops:

```python
for row in matrix:
    for x in row:
        ...
```

> **TIP:** If a comprehension needs more than two clauses or complex
> conditions, a normal loop is often clearer in interviews.

---

# 12. Copying, Aliasing, and Mutability

## 12.1 Assignment Does Not Copy

```python
b = a
```

This aliases the same object.

## 12.2 Shallow Copies

Lists:

```python
b = a[:]
b = list(a)
b = a.copy()
```

Dicts:

```python
d2 = d.copy()
```

Sets:

```python
s2 = s.copy()
```

## 12.3 Deep Copies

```python
import copy

b = copy.deepcopy(a)
```

> **PITFALL:** Deep copy is expensive. Avoid it in hot loops unless absolutely
> necessary.

## 12.4 Nested Structures

```python
a = [[1], [2]]
b = a.copy()
b[0].append(99)
print(a)  # [[1, 99], [2]]
```

The outer list was copied. Inner lists were shared.

## 12.5 Repetition

Safe:

```python
arr = [0] * n
```

Dangerous:

```python
grid = [[]] * n
```

All rows point to the same list.

Correct:

```python
grid = [[] for _ in range(n)]
```

## 12.6 Chained Assignment

Dangerous with mutables:

```python
a = b = []
a.append(1)
print(b)  # [1]
```

Use separate objects:

```python
a = []
b = []
```

---

# 13. Caching and Memoization Mechanics

## 13.1 `cache` and `lru_cache`

```python
from functools import cache, lru_cache

@cache
def f(x):
    ...

@lru_cache(maxsize=None)
def g(x):
    ...
```

`cache` is equivalent to an unbounded cache.

Use `lru_cache(None)` if older platform compatibility matters.

## 13.2 Arguments Must Be Hashable

Good:

```python
@cache
def f(i: int, state: tuple[int, ...]):
    ...
```

Bad:

```python
@cache
def f(state: list[int]):
    ...
```

Lists are unhashable.

## 13.3 Cache Lifetime

The cache persists as long as the function object persists.

For module-level functions:

```python
f.cache_clear()
```

Use nested functions when cache should belong to one call:

```python
def solve(nums):
    @cache
    def helper(i):
        ...
```

## 13.4 Method Cache Trap

```python
class Solution:
    @lru_cache(None)
    def helper(self, i):
        ...
```

Here `self` is part of the cache key.

> **PITFALL:** Caching instance methods can keep instances alive and create
> surprising memory behavior. In interviews, prefer nested cached helpers
> inside the method.

## 13.5 Cache Is for Pure Computation

Do not cache functions that:

```text
depend on time
depend on randomness
mutate state
return fresh mutable objects that callers modify
perform I/O
```

> **STAFF NOTE:** Memoization is a correctness decision as well as a performance
> decision. It assumes same arguments imply same result.

---

# 14. Classes, Dataclasses, and Custom Objects

## 14.1 Basic Class Syntax

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None
```

`self` is explicit.

## 14.2 Instance vs Class Attributes

Class attribute:

```python
class Bad:
    items = []
```

All instances share `items`.

Instance attribute:

```python
class Good:
    def __init__(self):
        self.items = []
```

> **PITFALL:** Mutable class attributes are shared across instances. This is the
> class-level cousin of mutable default arguments.

## 14.3 Dataclasses

```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
```

With ordering:

```python
@dataclass(order=True)
class Item:
    priority: int
    name: str
```

Exclude a field from comparison:

```python
from dataclasses import field
from typing import Any

@dataclass(order=True)
class Entry:
    priority: int
    item: Any = field(compare=False)
```

## 14.4 NamedTuple

```python
from typing import NamedTuple

class Point(NamedTuple):
    row: int
    col: int
```

Named tuples are immutable, tuple-like, and hashable if fields are hashable.

## 14.5 `__eq__` and `__hash__`

Objects used as dict keys or set elements must be hashable.

Rule:

```text
If a == b, then hash(a) must equal hash(b).
```

> **PITFALL:** If you define custom equality on a mutable object, making it
> hashable can be dangerous. Mutating a key after insertion can corrupt lookup
> logic.

## 14.6 `__lt__` and Ordering

Heaps and sorting may need `<`.

Prefer:

```python
items.sort(key=lambda x: x.priority)
```

Over defining custom comparison unless needed.

## 14.7 `__slots__`

```python
class Node:
    __slots__ = ("value", "next")

    def __init__(self, value):
        self.value = value
        self.next = None
```

`__slots__` can reduce per-instance memory by avoiding an instance `__dict__`.

Rarely necessary in interviews, but useful staff-level Python knowledge.

## 14.8 OOP Concepts in Python

Python supports object-oriented programming, but it is more dynamic than Java.

Core ideas:

```text
encapsulation by convention, not access modifiers
inheritance when there is a real is-a relationship
composition when behavior should be assembled from smaller objects
polymorphism through duck typing
interfaces through abstract base classes or protocols
```

## 14.9 Encapsulation

Python does not have Java-style `private`.

Conventions:

```python
class Service:
    def __init__(self):
        self.public = 1
        self._internal = 2
        self.__name_mangled = 3
```

Meaning:

```text
public: normal public attribute
_internal: intended for internal use by convention
__name_mangled: name-mangled to reduce subclass collisions, not true privacy
```

> **STAFF NOTE:** Python privacy is cultural and API-based. Design clear public
> interfaces rather than relying on enforced access control.

## 14.10 Composition vs Inheritance

Prefer composition when behavior is optional or replaceable:

```python
class ReportService:
    def __init__(self, storage, renderer):
        self.storage = storage
        self.renderer = renderer
```

Use inheritance when subclasses truly share an is-a relationship and a stable
contract.

> **JAVA DEV TRAP:** Do not create deep inheritance hierarchies just because
> Java would. Python code often stays cleaner with small objects and
> composition.

## 14.11 Polymorphism and Duck Typing

Python cares more about behavior than declared type.

```python
def write_all(writer, rows):
    for row in rows:
        writer.write(row)
```

Any object with a compatible `write` method works.

> **SAY THIS:** "In Python I usually depend on behavior, not concrete classes.
> If this were production code, I would document the expected protocol or use a
> `Protocol` type for static checking."

## 14.12 Abstract Base Classes and Protocols

Abstract base class:

```python
from abc import ABC, abstractmethod

class Store(ABC):
    @abstractmethod
    def get(self, key: str) -> str | None:
        ...
```

Protocol:

```python
from typing import Protocol

class Writer(Protocol):
    def write(self, value: str) -> None:
        ...
```

Difference:

```text
ABC: nominal interface; class explicitly inherits
Protocol: structural interface; object matches by having required methods
```

## 14.13 `@property`

```python
class Account:
    def __init__(self, balance: int):
        self._balance = balance

    @property
    def balance(self) -> int:
        return self._balance
```

Use properties when attribute access should remain simple but computation or
validation is needed.

> **PITFALL:** Avoid surprising expensive work in properties. Attribute access
> should feel cheap unless clearly documented.

## 14.14 `@classmethod` and `@staticmethod`

Class method receives the class:

```python
class User:
    def __init__(self, name: str):
        self.name = name

    @classmethod
    def from_email(cls, email: str):
        return cls(email.split("@")[0])
```

Static method receives neither `self` nor `cls`:

```python
class Math:
    @staticmethod
    def clamp(x, lo, hi):
        return max(lo, min(hi, x))
```

Rule of thumb:

```text
instance method: needs object state
class method: alternate constructor or class-level polymorphism
static method: namespacing helper with no object/class state
```

## 14.15 MRO and `super`

Python uses method resolution order (MRO), especially important with multiple
inheritance.

```python
class Child(Parent):
    def __init__(self):
        super().__init__()
```

View MRO:

```python
Child.mro()
```

> **STAFF NOTE:** Multiple inheritance can be powerful, but production Python
> usually favors simple inheritance trees, mixins with narrow behavior, and
> composition.

---

# 15. CPython Internals That Matter

## 15.1 CPython vs Python

Python is the language. CPython is the most common implementation.

Most interview platforms use CPython.

## 15.2 Object Model

Every Python object has:

```text
identity
type
value
reference count, in CPython
```

This explains:

```text
assignment aliases
mutability bugs
object overhead
identity vs equality
```

## 15.3 Reference Counting and Garbage Collection

CPython primarily uses reference counting. When an object's reference count
drops to zero, it can be deallocated.

Cycles are handled by a cyclic garbage collector.

> **STAFF NOTE:** You almost never need to mention this in coding interviews,
> but it explains why holding references in caches or globals can keep objects
> alive.

More precise staff-level model:

```text
Reference counting handles most cleanup immediately.
The cyclic GC handles reference cycles that refcounting alone cannot reclaim.
Objects with finalizers can make cycle cleanup more subtle.
Long-lived caches, globals, closures, and thread locals can keep objects alive.
```

Example cycle:

```python
a = []
a.append(a)
```

`a` references itself. Reference counting alone cannot free it while the cycle
exists; the cyclic garbage collector is responsible for detecting it.

Useful modules:

```python
import gc
import sys

sys.getrefcount(obj)
gc.collect()
```

> **PITFALL:** `sys.getrefcount(obj)` itself temporarily adds a reference while
> measuring, so the number is usually one higher than you expect.

## 15.4 Small Integer Caching

CPython caches small integers, commonly `-5` through `256`.

```python
a = 256
b = 256
a is b  # may be True in CPython
```

Do not rely on this.

```python
a == b  # value equality
```

## 15.5 String Interning

Some strings may be interned, especially identifier-like strings.

This can make `is` appear to work for strings.

Do not rely on it.

## 15.6 List Internals

Python lists are arrays of pointers.

Consequences:

```text
indexing is O(1)
append is amortized O(1)
front/middle insertion and deletion shift elements
list stores references, not inline primitive values
```

CPython list growth is over-allocated. The exact formula is an implementation
detail and can change, but the important behavior is stable:

```text
append is amortized O(1)
occasional resize copies references to a larger backing array
front and middle insert/delete shift references and stay O(n)
```

> **STAFF NOTE:** Say "amortized O(1)" for append, not "always O(1)." That
> small precision sounds senior.

## 15.7 Dict and Set Internals

Dicts and sets are hash tables.

Average:

```text
lookup O(1)
insert O(1)
delete O(1)
```

Worst-case collisions can degrade, but interviews normally use average-case
unless discussing adversarial input.

Keys must be hashable.

CPython dicts are compact, insertion-ordered hash tables. They resize as they
fill to preserve efficient probing. The exact load threshold is an
implementation detail, but the practical model is:

```text
more entries -> occasional resize
resize rehashes/repositions table entries
individual insert is usually O(1)
over many inserts, cost is amortized O(1)
```

Hash collisions:

```text
equal objects must have equal hashes
unequal objects may collide
collisions require probing/equality checks
pathological collisions can degrade performance
```

> **PITFALL:** If a custom object's hash depends on mutable fields and those
> fields change after insertion into a set/dict, lookup behavior becomes broken.

## 15.8 Hash Randomization

Python randomizes string hashing between processes for security.

Consequence:

```text
Do not rely on set iteration order.
Dict insertion order is stable by insertion, not by hash order.
```

`PYTHONHASHSEED` controls hash randomization:

```bash
PYTHONHASHSEED=0 python script.py
```

This can make some hash-based behavior reproducible for debugging, but
interview code should not depend on hash iteration order.

## 15.9 GIL

CPython has a Global Interpreter Lock.

Staff-level summary:

```text
Threads do not speed up CPU-bound Python bytecode in standard CPython.
Threads can still help I/O-bound workloads.
For CPU-bound parallelism, use multiprocessing or native extensions.
```

Python 3.13+ has optional free-threaded builds that can disable the GIL, but
that is not the default assumption for interviews or most production Python
deployments today.

Why the GIL matters:

```text
It protects CPython interpreter internals.
It does not make your multi-step business logic automatically thread-safe.
It limits CPU-bound parallelism in normal CPython threads.
It still allows useful concurrency for I/O-bound workloads.
```

Coding interviews are almost always single-threaded, but this is useful in
follow-up discussion.

## 15.10 Recursion and the C Stack

Python function calls consume stack frames. CPython protects itself with a
recursion limit.

```python
import sys

sys.getrecursionlimit()
sys.setrecursionlimit(10**6)
```

> **PITFALL:** Raising the recursion limit too high can crash the interpreter.
> Prefer iterative code for genuinely deep structures.

---

# 16. Performance and Memory Rules

## 16.1 Hidden O(n) Operations

These are the usual Python performance killers:

```text
list.pop(0)
list.insert(0, x)
x in list
list slicing in loops
string += in loops
bisect.insort into list
deepcopy in hot loops
repeated sorting
recomputing expensive key functions
```

## 16.2 Prefer C-Implemented Built-ins When Clear

Often faster and clearer:

```python
sum(nums)
min(nums)
max(nums)
sorted(nums)
"".join(parts)
Counter(items)
```

But do not hide logic just to use a built-in.

## 16.3 Big-O Still Wins

Python constant factors are larger than Java for tight loops.

This means:

```text
An O(n^2) solution that barely passes in Java may TLE in Python.
Hidden copies matter more.
Data-structure choice matters more.
Pushing work into built-ins can help.
```

## 16.4 Memory Awareness

Python objects are heavy compared with Java primitives.

Prefer:

```text
list of ints over many tiny custom objects
tuple coordinates over Pair objects
list adjacency for dense 0..n-1 ids
dict only when keys are sparse or arbitrary
```

## 16.5 Local Variables

Local variable access is faster than global lookup, but this is rarely the
first optimization.

Do not contort interview code for micro-optimizations. Fix algorithmic and
data-structure costs first.

## 16.6 Input/Output

For command-line judges:

```python
import sys

data = sys.stdin.buffer.read().split()
```

For line-based input:

```python
import sys

line = sys.stdin.readline()
```

Class-method online judges usually do not need manual input parsing.

## 16.7 When Python Convenience Hurts

| Feature | Great for | Avoid when |
|---|---|---|
| slicing | small, clear copies | repeated hot-path copies |
| recursion | shallow tree-like logic | input can be deep or adversarial |
| `defaultdict` | grouping/counting | missing-key reads must not mutate |
| `Counter` | quick frequency logic | zero-count keys affect correctness |
| `sorted` | one-time ordering | repeated re-sorting in a loop |
| `bisect.insort` | small sorted lists | large dynamic ordered sets |
| `deepcopy` | rare full clone | performance-sensitive loops |
| comprehensions | simple transforms | complex branching or side effects |
| decorators | cross-cutting concerns | hiding control flow or exceptions |
| generators | streaming large data | result must be reused many times |
| threads | I/O-bound overlap | CPU-bound pure Python speedup |
| async | high-concurrency I/O | blocking libraries or CPU-heavy work |

> **STAFF NOTE:** "Pythonic" does not mean "use the shortest feature." It means
> use the construct whose semantics and cost match the problem.

---

# 17. Production Python: OOP, Context Managers, Decorators, and Generators

This section is for staff+ fluency. These topics may not appear in a classic
coding round, but they matter in production engineering interviews and Python
fluency follow-ups.

## 17.1 Production OOP Checklist

Good Python OOP usually means:

```text
small classes with clear responsibilities
composition over deep inheritance
explicit dependencies passed into constructors
stable public methods
limited mutable shared state
dunder methods only when they make the object behave naturally
type hints that clarify contracts without over-engineering
```

Ask yourself:

```text
Does this class own state?
Does it enforce an invariant?
Is inheritance truly needed?
Could a function be clearer?
Could a dataclass be enough?
Will equality/hash/ordering be surprising?
```

> **STAFF NOTE:** Python does not force everything into classes. In Python,
> using a function instead of a class can be the more senior design choice when
> no persistent state or polymorphic contract is needed.

## 17.2 Context Managers and `with`

Context managers guarantee setup and cleanup around a block.

Common examples:

```python
with open(path) as f:
    data = f.read()
```

```python
with lock:
    shared_state += 1
```

The protocol:

```python
class Managed:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc, tb):
        return False
```

`__exit__` runs whether the block succeeds or raises. Returning `True`
suppresses the exception; returning `False` lets it propagate.

> **PITFALL:** Do not accidentally suppress exceptions from `__exit__` unless
> that is the intended API.

## 17.3 `contextlib`

Function-based context manager:

```python
from contextlib import contextmanager

@contextmanager
def managed_resource():
    resource = acquire()
    try:
        yield resource
    finally:
        release(resource)
```

Multiple dynamic context managers:

```python
from contextlib import ExitStack

with ExitStack() as stack:
    resources = [stack.enter_context(open(path)) for path in paths]
```

> **STAFF NOTE:** In production Python, `with` is the standard shape for files,
> locks, transactions, temporary overrides, metrics scopes, and cleanup.

## 17.4 Decorators

A decorator wraps a function or class.

Basic function decorator:

```python
from functools import wraps

def traced(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

Use:

```python
@traced
def work(x):
    return x + 1
```

> **PITFALL:** Always use `functools.wraps` in production decorators. It
> preserves metadata like `__name__`, `__doc__`, and annotations.

Decorator with arguments:

```python
from functools import wraps

def retry(times: int):
    def outer(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_error = None
            for _ in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as exc:
                    last_error = exc
            raise last_error
        return wrapper
    return outer
```

> **PITFALL:** A real production retry decorator should handle exception types,
> backoff, jitter, timeouts, idempotency, and cancellation. The example above is
> only the shape.

## 17.5 Generators and `yield`

A generator produces values lazily.

```python
def numbers(n):
    for i in range(n):
        yield i
```

Use:

```python
for x in numbers(3):
    ...
```

Generator benefits:

```text
lazy evaluation
lower memory usage
pipeline-style processing
natural stream representation
```

> **PITFALL:** Generators are one-shot iterators. Once consumed, they are empty.

## 17.6 `yield from`

Delegate to another iterable:

```python
def flatten(rows):
    for row in rows:
        yield from row
```

Equivalent to yielding each item in each row.

## 17.7 Iterator Protocol

An iterator implements:

```python
__iter__()
__next__()
```

Example:

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value
```

Most interview code does not need custom iterators, but production Python APIs
often expose iterable objects.

## 17.8 Generator Cleanup

If a generator manages resources, use `try/finally` or a context manager.

```python
def stream_rows(path):
    f = open(path)
    try:
        for line in f:
            yield line
    finally:
        f.close()
```

Usually better:

```python
def stream_rows(path):
    with open(path) as f:
        for line in f:
            yield line
```

## 17.9 Production Design Rules

```text
Use context managers for ownership and cleanup.
Use decorators for cross-cutting behavior, but keep them transparent.
Use generators for streaming and large data.
Use classes when state/invariants matter.
Use functions when behavior is stateless and direct.
Prefer explicit dependencies over hidden globals.
```

---

# 18. Concurrency: Threads, Locks, Processes, and Asyncio

Python concurrency is a production topic. It is also a staff-level interview
topic because it tests whether you understand execution, shared state, and
failure modes.

## 18.1 The Short Answer

| Need | Use |
|---|---|
| I/O-bound concurrency with blocking libraries | `threading` or `ThreadPoolExecutor` |
| CPU-bound parallelism in normal CPython | `multiprocessing` or `ProcessPoolExecutor` |
| many concurrent network operations with async libraries | `asyncio` |
| protect shared in-process state | `Lock`, `RLock`, `Condition`, `Semaphore`, `Queue` |
| communicate safely between threads | `queue.Queue` |
| periodic/background work | thread/process/task plus explicit shutdown |

> **SAY THIS:** "In standard CPython, threads are useful for I/O-bound
> concurrency, but not for speeding up CPU-bound Python bytecode because of the
> GIL. For CPU-bound parallelism I would use processes or native code."

## 18.2 GIL: Correct Staff-Level Answer

The Global Interpreter Lock means that in standard CPython only one thread
executes Python bytecode at a time.

Implications:

```text
CPU-bound Python threads do not scale across cores.
I/O-bound threads can still improve throughput because threads wait on I/O.
C extensions may release the GIL during heavy native work.
Multiprocessing bypasses the GIL by using separate processes.
The GIL does not make your program logically thread-safe.
```

Python 3.13+ has optional free-threaded builds that can disable the GIL, but
that is not the default production or interview assumption.

## 18.3 Threads

```python
from threading import Thread

def work(item):
    ...

t = Thread(target=work, args=(item,))
t.start()
t.join()
```

Production guidance:

```text
name important threads
avoid daemon threads for critical work
provide shutdown signals
join threads during shutdown
handle exceptions inside worker functions or via futures
```

> **PITFALL:** Daemon threads can be stopped abruptly during interpreter
> shutdown. They may not release files, locks, or transactions cleanly.

## 18.4 Locks

Use a lock to protect shared mutable state.

```python
from threading import Lock

lock = Lock()
count = 0

def increment():
    global count
    with lock:
        count += 1
```

Use locks as context managers:

```python
with lock:
    critical_section()
```

> **PITFALL:** The GIL does not make compound operations like "check then act"
> safe at the business-logic level. Use locks around shared invariants.

## 18.5 `Lock` vs `RLock`

`Lock` is a primitive lock. A thread cannot acquire it twice without deadlock.

`RLock` is re-entrant. The same thread can acquire it multiple times and must
release it the same number of times.

```python
from threading import RLock

lock = RLock()
```

Use `RLock` when recursive or layered code may re-enter the same lock. Prefer
plain `Lock` when re-entrancy is not needed.

## 18.6 Events, Conditions, Semaphores, and Queues

`Event`: one thread signals others.

```python
from threading import Event

stop = Event()
stop.set()
stop.is_set()
```

`Condition`: wait until a predicate/state changes.

`Semaphore`: limit concurrent access to a resource.

`queue.Queue`: thread-safe producer/consumer queue.

```python
from queue import Queue

q = Queue()
q.put(item)
item = q.get()
q.task_done()
```

> **STAFF NOTE:** Prefer `Queue` over manually sharing a list between threads.
> It gives you synchronization and clearer producer/consumer semantics.

## 18.7 Deadlocks and Lock Ordering

Deadlock pattern:

```text
Thread A holds lock1 and waits for lock2.
Thread B holds lock2 and waits for lock1.
```

Avoid with:

```text
consistent global lock ordering
small critical sections
timeouts where appropriate
no blocking I/O while holding locks
careful callbacks while locked
```

> **SAY THIS:** "I would define a lock ordering and avoid calling external code
> while holding the lock."

## 18.8 Thread Pools

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

with ThreadPoolExecutor(max_workers=8) as executor:
    futures = [executor.submit(fetch, url) for url in urls]
    for future in as_completed(futures):
        result = future.result()
```

Good for blocking I/O tasks.

> **PITFALL:** `future.result()` re-raises exceptions from worker threads. Do
> not ignore it.

## 18.9 Processes

Use processes for CPU-bound parallel work in standard CPython.

```python
from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor() as executor:
    results = list(executor.map(cpu_heavy, items))
```

Tradeoffs:

```text
separate memory space
data must be pickled between processes
higher startup and communication overhead
better CPU parallelism for pure Python work
```

> **PITFALL:** Functions and arguments sent to worker processes must generally
> be picklable. Lambdas, local functions, open file handles, and live network
> connections often do not serialize cleanly.

## 18.10 Asyncio

`asyncio` is cooperative concurrency using an event loop.

```python
import asyncio

async def fetch_one(client, url):
    return await client.get(url)

async def main(urls):
    tasks = [fetch_one(client, url) for url in urls]
    return await asyncio.gather(*tasks)
```

Use for:

```text
many concurrent network calls
async database clients
async web servers
high-concurrency I/O workloads
```

Do not use for:

```text
CPU-bound speedup
blocking libraries unless moved to a thread/process
simple scripts where synchronous code is clearer
```

> **PITFALL:** Calling blocking code inside an async function blocks the event
> loop. Use async-compatible libraries or run blocking work in an executor.

## 18.11 Asyncio Concepts

```text
coroutine: async function result that can be awaited
task: scheduled coroutine
event loop: runs tasks cooperatively
await: yield control until awaited operation is ready
gather: wait for multiple awaitables
timeout: prevent waiting forever
cancel: request task cancellation
```

Timeout:

```python
result = await asyncio.wait_for(operation(), timeout=5)
```

Python 3.11+ structured concurrency:

```python
async with asyncio.TaskGroup() as tg:
    tg.create_task(work_one())
    tg.create_task(work_two())
```

## 18.12 Production Concurrency Checklist

```text
What is shared mutable state?
Who owns shutdown?
How are exceptions surfaced?
Are timeouts enforced?
Is there backpressure?
Are retries idempotent?
Are locks held during I/O?
Can tasks be cancelled safely?
Is observability present for queue depth, latency, failures, and saturation?
```

> **STAFF NOTE:** Concurrency bugs are usually ownership bugs. Be explicit
> about who owns state, who signals shutdown, and how failure propagates.

---

# 19. The Complete Pitfall Catalog

## 19.1 Mutable Default Arguments

Bad:

```python
def f(path=[]):
    ...
```

Good:

```python
def f(path=None):
    if path is None:
        path = []
```

## 19.2 Aliased 2D Lists

Bad:

```python
grid = [[0] * cols] * rows
```

Good:

```python
grid = [[0] * cols for _ in range(rows)]
```

## 19.3 List Used as Queue

Bad:

```python
item = q.pop(0)
```

Good:

```python
from collections import deque

item = q.popleft()
```

## 19.4 Slicing in Hot Paths

Bad:

```python
part = arr[i:]
```

inside a large loop.

Prefer indices or views via boundaries.

## 19.5 String Concatenation in a Loop

Bad:

```python
s += ch
```

Better:

```python
parts.append(ch)
```

then:

```python
s = "".join(parts)
```

## 19.6 `sort()` Returns `None`

Bad:

```python
arr = arr.sort()
```

Good:

```python
arr.sort()
```

## 19.7 `is` vs `==`

Bad:

```python
if x is 1000:
    ...
```

Good:

```python
if x == 1000:
    ...
```

## 19.8 Division with Negatives

```python
-7 // 2  # -4
```

If you need truncation toward zero, implement it explicitly.

## 19.9 `/` Produces Float

```python
6 / 2  # 3.0
```

Use `//` for integer indices.

## 19.10 Modifying While Iterating

Bad:

```python
for x in arr:
    if bad(x):
        arr.remove(x)
```

Good:

```python
arr = [x for x in arr if not bad(x)]
```

## 19.11 Dict Mutation During Iteration

Bad:

```python
for key in d:
    del d[key]
```

Good:

```python
for key in list(d):
    del d[key]
```

## 19.12 Assignment Is Aliasing

```python
b = a
```

does not copy.

## 19.13 Heap Tie Comparisons

Bad:

```python
heappush(heap, (priority, obj))
```

Good:

```python
heappush(heap, (priority, next(counter), obj))
```

## 19.14 `Counter` Zero Counts

```python
freq[x] -= 1
```

may leave `x` in the counter with count zero.

Delete when distinct count matters:

```python
if freq[x] == 0:
    del freq[x]
```

## 19.15 `defaultdict` Auto-Creation

```python
d = defaultdict(list)
d[key]
```

creates `key`.

## 19.16 Truthiness of Valid Values

```python
if not index:
    ...
```

This treats `0` as missing.

Use:

```python
if index is None:
    ...
```

## 19.17 Float Equality

Bad:

```python
a == b
```

for computed floats.

Good:

```python
abs(a - b) < 1e-9
```

## 19.18 Integer Overflow Assumptions

Python will not overflow automatically. If bounds matter, check manually.

## 19.19 Dict Order Is Not Sorted

Dict preserves insertion order, not key order.

Use:

```python
for key in sorted(d):
    ...
```

## 19.20 Set Order Is Not Stable Sorted Order

Do not use set iteration for sorted output.

## 19.21 Late-Binding Closures

Bad:

```python
funcs = [lambda: i for i in range(3)]
```

Good:

```python
funcs = [lambda i=i: i for i in range(3)]
```

## 19.22 `nonlocal` Missing

If you reassign an outer variable inside a nested function, use `nonlocal`.

## 19.23 Generator Exhaustion

```python
it = map(int, ["1", "2"])
list(it)  # [1, 2]
list(it)  # []
```

## 19.24 `zip` Truncation

`zip` stops at the shortest iterable.

## 19.25 `range` Is Not a List

Usually good. Materialize only if needed.

## 19.26 `sum(list_of_lists, [])`

Bad flattening pattern. Use `chain.from_iterable`.

## 19.27 `bisect.insort` Is O(n)

Do not mistake it for a tree insert.

## 19.28 Cache Arguments Unhashable

Lists, dicts, and sets cannot be cache keys.

Convert to tuples when needed.

## 19.29 Cache on Methods Includes `self`

Prefer nested cached helpers in interview code.

## 19.30 Class-Level Mutable State

Bad:

```python
class C:
    items = []
```

Good:

```python
class C:
    def __init__(self):
        self.items = []
```

## 19.31 Boolean `or` Defaults

Bad when `0` is valid:

```python
limit = user_limit or 10
```

Good:

```python
limit = 10 if user_limit is None else user_limit
```

## 19.32 NaN

```python
float("nan") != float("nan")
```

## 19.33 Shadowing Built-ins

Avoid:

```python
list = []
dict = {}
sum = 0
```

You lose access to the built-in name.

## 19.34 Leaking State Across Test Cases

On online judges, the same `Solution` instance behavior varies by platform.
Avoid persistent mutable state unless intended.

Prefer local variables inside the method.

## 19.35 Over-Clever One-Liners

Python allows dense code. Interviews reward readable code.

Prefer:

```python
result = []
for x in nums:
    if valid(x):
        result.append(transform(x))
```

Over a deeply nested comprehension that the interviewer cannot inspect quickly.

---

# 20. Staff-Level Python Communication

## 20.1 When Choosing a Data Structure

> **SAY THIS:** "I am using a dict here because I need average O(1) lookup, and
> the O(n) extra space is the tradeoff."

> **SAY THIS:** "I am using a deque rather than a list because removing from the
> front of a list is O(n)."

> **SAY THIS:** "I am using tuples as keys because lists are mutable and
> unhashable."

> **SAY THIS:** "I am avoiding slicing here because it would allocate a new list
> each time."

## 20.2 When Discussing Python vs Java

> **SAY THIS:** "Python lets me express the core logic with fewer lines, but I
> need to be careful about hidden copies, object overhead, recursion depth, and
> list operations that shift elements."

> **SAY THIS:** "Java has built-in tree-backed maps and sets; Python's standard
> library does not, so ordered dynamic operations need a different plan unless
> `sortedcontainers` is allowed."

## 20.3 When Asked About Complexity

Be specific:

```text
dict lookup is average O(1)
list membership is O(n)
heap push/pop is O(log n)
deque popleft is O(1)
slice of length k is O(k)
sort is O(n log n)
```

Do not say:

```text
Python handles it
```

## 20.4 When Asked About Internals

> **SAY THIS:** "A Python list is a dynamic array of references, so append is
> amortized O(1), but inserting or deleting near the front shifts references and
> is O(n)."

> **SAY THIS:** "A dict is a hash table, so lookup is average O(1), assuming
> good hashing and non-adversarial collisions."

> **SAY THIS:** "Strings are immutable, so repeated concatenation can allocate
> many intermediate strings. I would collect parts and join once."

## 20.5 When You Need a Non-Standard Library

> **SAY THIS:** "If `sortedcontainers` is allowed, this is straightforward with
> a sorted list or sorted dict. If not, I would avoid assuming it and choose a
> standard-library design."

## 20.6 When Recursion Might Fail

> **SAY THIS:** "The recursive version is clean, but Python has a recursion
> limit. If depth can be large, I would use an explicit stack rather than just
> raising the recursion limit."

---

# 21. Final Cheat Sheets

## 21.1 Imports

```python
from collections import defaultdict, Counter, deque, OrderedDict
from heapq import heappush, heappop, heappushpop, heapreplace, heapify, nlargest, nsmallest
from bisect import bisect_left, bisect_right, insort
from functools import cache, lru_cache, cmp_to_key
from itertools import count, chain, product, pairwise, accumulate
from operator import itemgetter, attrgetter
from math import inf, gcd, lcm, isqrt
from threading import Lock, RLock, Event, Semaphore, Condition
from queue import Queue
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed
from contextlib import contextmanager, ExitStack
from dataclasses import dataclass, field
```

## 21.2 Structure Choice

| Need | Use |
|---|---|
| dynamic array | `list` |
| stack | `list` |
| queue | `deque` |
| fast membership | `set` |
| key-value lookup | `dict` |
| frequency | `Counter` / `defaultdict(int)` |
| grouping | `defaultdict(list)` |
| priority queue | `heapq` |
| sorted array search | `bisect_left`, `bisect_right` |
| recency operations | `OrderedDict` |
| immutable compound key | `tuple` |
| ordered map/set | confirm `sortedcontainers` or redesign |
| resource cleanup | `with` / context manager |
| cross-cutting wrapper | decorator with `functools.wraps` |
| lazy stream | generator / iterator |
| I/O-bound concurrency | threads / `ThreadPoolExecutor` |
| CPU-bound parallelism | processes / `ProcessPoolExecutor` |
| async network concurrency | `asyncio` |
| shared thread state | `Lock` / `Queue` |

## 21.3 Complexity Must-Know

```text
list index/read/write           O(1)
list append/pop end             amortized O(1) / O(1)
list pop front/insert front      O(n)
list membership                  O(n)
list slice length k              O(k)
dict/set lookup/insert/delete    average O(1)
deque append/pop both ends       O(1)
heap push/pop                    O(log n)
heap peek                        O(1)
heapify                          O(n)
bisect search                    O(log n)
bisect insort                    O(n)
sort                             O(n log n)
string slice length k            O(k)
string join total length n       O(n)
```

## 21.4 Night-Before Pitfall Scan

```text
Did I use list.pop(0)?
Did I use list.insert(0, x)?
Did I slice in a loop or recursion?
Did I build strings with += in a loop?
Did I create [[0] * cols] * rows?
Did I use a mutable default argument?
Did I compare values with is?
Did I assume // truncates toward zero?
Did I mutate a dict/list while iterating?
Did I leave zero-count Counter keys?
Did defaultdict create keys by reading?
Did heap tuples risk comparing objects?
Did I use bisect.insort assuming O(log n)?
Did I rely on recursion depth being large?
Did I pass an unhashable list/dict/set into cache?
Did I rely on dict order being sorted?
Did I accidentally shadow list, dict, set, sum, or max?
Did a context manager accidentally suppress exceptions?
Did a decorator forget functools.wraps?
Did I consume a generator twice?
Did threads share mutable state without a lock or Queue?
Did I hold a lock while doing blocking I/O?
Did I use threads for CPU-bound speedup in normal CPython?
Did process-pool work require unpicklable functions or arguments?
Did async code call blocking functions on the event loop?
```

## 21.5 Final Rule

```text
Use Python to make the algorithm clear.
Do not use Python cleverness to hide the algorithm.
```

