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

> **TIP:** A practical pointer that makes your code cleaner or faster to write.

---

## Table of Contents

0. [Quick Reference: Setup and Must-Type Idioms](#0-quick-reference-setup-and-must-type-idioms)
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
19. [Correct vs Pitfall Example Bank](#19-correct-vs-pitfall-example-bank)
20. [The Complete Pitfall Catalog](#20-the-complete-pitfall-catalog)
21. [Staff-Level Python Communication](#21-staff-level-python-communication)
22. [Final Cheat Sheets](#22-final-cheat-sheets)
23. [Official References](#23-official-references)

---

# 0. Quick Reference: Setup and Must-Type Idioms

This page is for the night before and the first two minutes of a round. Everything
here is explained in depth in later sections; this is the muscle-memory layer.

## 0.1 Standard Interview Preamble

This is language setup, not an algorithm template. Paste only what you use.

```python
import sys
from collections import defaultdict, Counter, deque, OrderedDict
from functools import cache, lru_cache, reduce
from itertools import accumulate, product, pairwise, permutations, combinations
import heapq
import bisect
import math

# input = sys.stdin.readline          # faster line input on CLI judges
# sys.setrecursionlimit(10**6)        # only if you knowingly recurse deep
```

> **STAFF NOTE:** Do not paste all of this blindly. Importing names you never use
> reads as noise. Add each import the moment you reach for it.

## 0.2 The Idioms You Will Actually Type

| Goal | Idiom | Cost |
|---|---|---|
| index + value | `for i, x in enumerate(nums):` | O(n) |
| pair adjacent | `for a, b in pairwise(nums):` | O(n) |
| queue | `q = deque(); q.append(x); q.popleft()` | O(1) ends |
| max-heap (numeric) | `heappush(h, -x); top = -heappop(h)` | O(log n) |
| heap tie-break | `heappush(h, (prio, next(counter), obj))` | O(log n) |
| frequency map | `freq = Counter(nums)` | O(n) |
| grouping | `g = defaultdict(list); g[k].append(v)` | O(1) avg |
| group, no auto-create | `d.setdefault(k, []).append(v)` | O(1) avg |
| 2D grid | `grid = [[0]*cols for _ in range(rows)]` | O(rows·cols) |
| index ↔ coordinate | `r, c = divmod(idx, cols)` | O(1) |
| transpose / unzip | `cols = list(zip(*grid))` | O(rows·cols) |
| running totals | `pre = list(accumulate(nums, initial=0))` | O(n) |
| sorted insert point | `i = bisect_left(arr, x)` | O(log n) |
| reverse | `for x in reversed(nums):` | O(n) |
| build a string | `"".join(parts)` | O(total) |
| count predicate | `sum(1 for x in nums if ok(x))` | O(n) |
| keep-while-assigning | `while (line := f.readline()):` | — |
| memoize | `@cache` over a pure, hashable-arg function | — |
| char ↔ small index | `ord(ch) - ord("a")` | O(1) |
| lowest set bit | `x & -x` | O(1) |
| clear lowest set bit | `x & (x - 1)` | O(1) |
| sentinel min/max | `best = inf` / `best = -inf` | — |

## 0.3 Reflexes That Prevent Wrong Answers

```text
queue          -> deque.popleft(), never list.pop(0)
2D grid        -> [[0]*c for _ in range(r)], never [[0]*c]*r
default arg    -> def f(x=None): if x is None: x = []
missingness    -> if x is None, never if not x (0/""/[] are falsy)
identity       -> is only for None / same-object; == for values
int division   -> // floors toward -inf; it does not truncate
float compare  -> math.isclose(a, b), never a == b
heap of tuples -> add a counter tie-breaker before any unorderable payload
cache args     -> must be hashable; pass tuple(...), never a list
dict/set iter  -> iterate over list(d) if you mutate inside the loop
```

## 0.4 Complexity Sanity Targets

```text
n <= 20          exponential / bitmask may pass
n <= 5,000       O(n^2) may pass
n <= 100,000     want O(n log n) or O(n)
n >= 1,000,000   want O(n); avoid hidden O(n) ops (front pop, slice, in-list)
```

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

Example of writing version-safe interview code:

```python
from functools import lru_cache
import heapq

@lru_cache(None)          # Works even when functools.cache is unavailable.
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

max_heap = []
heapq.heappush(max_heap, -10)
heapq.heappush(max_heap, -3)
largest = -heapq.heappop(max_heap)  # 10
```

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

Examples of direct Java-to-Python translations:

```python
# Java: Map<String, Integer> count = new HashMap<>();
count = {}
count["a"] = count.get("a", 0) + 1

# Java: StringBuilder sb = new StringBuilder();
parts = []
for word in words:
    parts.append(word)
text = "".join(parts)

# Java: PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
import heapq

pq = []
heapq.heappush(pq, (distance, node))
distance, node = heapq.heappop(pq)

# Java: for (int i = 0; i < nums.length; i++)
for i, value in enumerate(nums):
    ...
```

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

## 4.9 The Walrus Operator

The assignment expression `:=` (Python 3.8+) binds a value and returns it in the
same expression. It removes a duplicated call or an extra line.

Read-and-test loop:

```python
while (line := f.readline()):
    process(line)
```

Compute once, then branch:

```python
if (n := len(nums)) > k:
    print(f"too many: {n}")
```

Reuse a value inside a comprehension:

```python
results = [y for x in data if (y := transform(x)) is not None]
```

> **TRICK:** Use the walrus to avoid calling an expensive function twice, or to
> capture a loop value you also need to test. Do not overuse it; an extra plain
> line is often clearer.

> **PITFALL:** The parentheses in `while (line := ...)` and `if (n := ...)`
> matter for readability and sometimes precedence. When unsure, parenthesize.

## 4.10 Conditional Expressions and Merge Operators

Python's ternary puts the value first, then the condition:

```python
x = a if cond else b
```

> **JAVA DEV TRAP:** This is not `cond ? a : b`. The order is value, condition,
> value. Read it as "a, if cond, otherwise b."

Unpack iterables and mappings directly into new literals:

```python
merged_list = [*a, *b]        # concatenate without chained +
merged_set  = {*a, *b}        # union into a new set
merged_dict = {**d1, **d2}    # later keys win on conflict
```

Dicts also support union operators (Python 3.9+):

```python
d3 = d1 | d2     # new dict, right side wins on key conflict
d1 |= d2         # in-place update of d1
```

> **TRICK:** `{**defaults, **overrides}` is the clean way to layer config: start
> from defaults, then let the overrides win.

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

## 5.9 The Full `try` Shape: `else`, `finally`, Multi-Catch, and `raise`

The complete structure has four clauses. Each runs at a precise time:

```python
try:
    risky()
except (KeyError, IndexError) as e:   # catch several types; bind the instance
    handle(e)
except Exception as e:                # broader fallback
    raise RuntimeError("wrapped") from e   # chain, preserving the cause
else:
    on_success()                      # runs only if try raised nothing
finally:
    cleanup()                         # runs always: success, exception, return
```

Clause semantics:

```text
except (A, B)   one handler for multiple types; first matching except wins
else            runs only when no exception occurred (keep try bodies small)
finally         runs no matter what, including on return/break/exception
as e            binds the exception object for inspection or re-raise
```

Raising your own:

```python
class ValidationError(Exception):     # subclass Exception, not BaseException
    pass

raise ValidationError("bad input")
```

`raise ... from e` chains exceptions so the original cause is preserved:

```python
try:
    parse(raw)
except ValueError as e:
    raise ValidationError("could not parse record") from e
# the traceback shows both: the new error and "the above was the direct cause"
```

> **JAVA DEV TRAP:** Never write a bare `except:`. It also swallows
> `KeyboardInterrupt` and `SystemExit`, so Ctrl-C and clean shutdown stop
> working. Catch `Exception` (or a specific type), never bare `except:`.

> **PITFALL:** A `return` inside `finally` overrides any return or exception from
> the `try` body, silently discarding it. Do not return from `finally`.

> **STAFF NOTE:** Catch the narrowest exception that expresses intent. Broad
> `except Exception` at a boundary is fine when you log and re-raise; broad
> catches that silently swallow errors are a classic production bug.

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

If mismatched lengths matter, check them, or pad with `zip_longest`:

```python
from itertools import zip_longest

list(zip_longest([1, 2], [3], fillvalue=0))  # [(1, 3), (2, 0)]
```

> **TRICK:** `zip(*rows)` transposes a matrix (it unzips). It is the cleanest way
> to swap rows and columns or to peel parallel columns apart.

```python
grid = [[1, 2, 3], [4, 5, 6]]
cols = list(zip(*grid))  # [(1, 4), (2, 5), (3, 6)]
```

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

> **PITFALL:** Empty-iterable behavior is a classic edge case. `all([])` is
> `True` (vacuous truth) and `any([])` is `False`. A validation like
> `if all(valid(x) for x in items)` silently passes when `items` is empty.

```python
all([])  # True
any([])  # False
```

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

Group consecutive equal keys (run-length style):

```python
from itertools import groupby

for key, group in groupby("aaabbc"):
    count = len(list(group))   # ("a", 3), ("b", 2), ("c", 1)
```

> **PITFALL:** `groupby` only groups *consecutive* equal keys. To group all
> equal keys regardless of position, sort first or use a `defaultdict(list)`.
> Also, each `group` is a lazy sub-iterator: consume it (e.g. `list(group)`)
> before advancing, or it is lost.

Take the first `k` items from any iterable without materializing the rest:

```python
from itertools import islice

first_k = list(islice(gen, k))        # NOT list(gen)[:k], which consumes everything
window  = list(islice(gen, 2, 5))     # start, stop slicing on an iterator
```

> **TIP:** `islice` is the lazy equivalent of slicing. Use it on generators and
> infinite iterators (`count()`, `cycle()`) where `[:k]` is impossible or
> wasteful.

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

Counts set bits in an integer (Python 3.10+). On older runtimes use
`bin(mask).count("1")`.

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

Examples of avoiding convenience when it changes the cost or semantics:

```python
# Avoid repeated slicing in recursive search; pass indexes instead.
def search(nums, lo, hi, target):
    if lo > hi:
        return -1
    mid = (lo + hi) // 2
    if nums[mid] == target:
        return mid
    if nums[mid] < target:
        return search(nums, mid + 1, hi, target)
    return search(nums, lo, mid - 1, target)

# Avoid defaultdict when a read should not mutate the map.
seen = {"a": 1}
if seen.get("b", 0) == 0:
    ...
```

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

Examples by structure:

```python
arr = [10, 20, 30]              # list: indexable dynamic array
stack = []
stack.append(1)
stack.pop()                    # stack: pop from end

point = (2, 3)                 # tuple: immutable compound key
dist = {point: 5}

letters = set("banana")        # set: membership/dedup
exists = "b" in letters

from collections import Counter, defaultdict, deque, OrderedDict
freq = Counter("banana")       # Counter: frequencies
groups = defaultdict(list)
groups["a"].append("alice")    # defaultdict: grouping

q = deque([1, 2, 3])
q.popleft()                    # deque: queue

import heapq
heap = [5, 1, 3]
heapq.heapify(heap)
smallest = heapq.heappop(heap)

import bisect
idx = bisect.bisect_left([10, 20, 30], 20)

lru = OrderedDict()
lru["a"] = 1
lru.move_to_end("a")
```

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

A regular `set` is mutable and therefore unhashable, so it cannot be a dict key
or an element of another set. `frozenset` is the immutable, hashable version:

```python
seen = set()
seen.add(frozenset(state))      # legal: a "set of sets" needs frozenset members
key = frozenset(group)          # canonical key for anagram-style grouping
```

> **TRICK:** When a visited-state, an unordered group, or an edge `{u, v}` must
> be a dict key or live inside a set, reach for `frozenset`. It is the set
> analog of using a `tuple` instead of a `list` as a key.

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

A bounded deque is a ready-made sliding window. With `maxlen` set, appending to
a full deque automatically evicts from the opposite end:

```python
window = deque(maxlen=3)
for x in [1, 2, 3, 4]:
    window.append(x)          # appending the 4th drops the 1st
# window == deque([2, 3, 4], maxlen=3)
```

> **TRICK:** `deque(maxlen=k)` removes all manual eviction bookkeeping for
> last-k / fixed-window problems. Both `append` and the auto-eviction stay O(1).

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

> **TRICK:** `dict.setdefault` gives the same grouping convenience on a plain
> dict, and it only inserts when you intend to. It returns the existing value if
> present, otherwise inserts and returns the default.

```python
d = {}
d.setdefault(key, []).append(value)
```

> **PITFALL:** The default in `setdefault(key, [])` is built on every call, even
> when the key already exists. For a hot loop with expensive defaults,
> `defaultdict` avoids that wasted construction.

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

Lazy k-way merge of already-sorted inputs:

```python
from heapq import merge

merged = list(merge([1, 4, 7], [2, 3, 8], [0, 5]))  # fully sorted, streaming
```

`merge` returns an iterator and does not pull everything into memory, which suits
merging many sorted streams.

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

> **PITFALL:** With the `key=` parameter (Python 3.10+), `key` is applied to the
> list elements but **not** to `x`. You must pass `x` as the already-keyed value.
> For example, to search `records` sorted by `.age`, call
> `bisect_left(records, target_age, key=lambda r: r.age)` — note `target_age`,
> not a record. When in doubt, precompute a separate keys list and bisect that.

> **PITFALL:** The `bisect` functions are not designed for concurrent mutation
> of the same list by multiple threads.

### TreeMap floor / ceiling / lower / higher

This is the idiom that replaces Java's `TreeMap.floorKey` / `ceilingKey` family
on a **sorted, static** list. Memorize the four index formulas:

```text
floor   (largest value <= x):  i = bisect_right(arr, x) - 1   valid if i >= 0
lower   (largest value <  x):  i = bisect_left(arr, x)  - 1   valid if i >= 0
ceiling (smallest value >= x): i = bisect_left(arr, x)        valid if i < len(arr)
higher  (smallest value >  x): i = bisect_right(arr, x)       valid if i < len(arr)
```

```python
from bisect import bisect_left, bisect_right

def floor(arr, x):                       # largest value <= x, else None
    i = bisect_right(arr, x) - 1
    return arr[i] if i >= 0 else None

def ceiling(arr, x):                     # smallest value >= x, else None
    i = bisect_left(arr, x)
    return arr[i] if i < len(arr) else None
```

> **SAY THIS:** "In Java I'd use `TreeMap.floorKey`/`ceilingKey`. In Python, if
> the data is sorted and static, `bisect` gives me the same floor/ceiling
> queries in O(log n). If I also need O(log n) *insertions*, I need
> `sortedcontainers` or a different design, because list insertion is O(n)."

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

If you write comparisons by hand, `functools.total_ordering` fills in the rest
from `__eq__` plus one of `__lt__`, `__le__`, `__gt__`, or `__ge__`:

```python
from functools import total_ordering

@total_ordering
class Version:
    def __init__(self, major, minor):
        self.major, self.minor = major, minor

    def __eq__(self, other):
        return (self.major, self.minor) == (other.major, other.minor)

    def __lt__(self, other):
        return (self.major, self.minor) < (other.major, other.minor)
    # __le__, __gt__, __ge__ are now derived automatically
```

> **STAFF NOTE:** In interviews, tuple entries are often simpler than custom
> ordering classes. Use a class only if it improves clarity.

## 8.8 Mixed-Type Comparison Raises

Python 3 has no universal ordering across unrelated types. Comparing or sorting
mixed types raises `TypeError`:

```python
sorted([1, None])    # TypeError: '<' not supported between 'NoneType' and 'int'
sorted([1, "a"])     # TypeError: '<' not supported between 'str' and 'int'
sorted([1, 2.0, 3])  # OK: int and float are mutually comparable
```

> **JAVA DEV TRAP:** Java often tolerates or defines cross-type ordering; Python
> does not. A single stray `None` from a `dict.get` or a missing field turns a
> `sorted(...)` call into a crash.

Normalize with a key that pushes the odd values to one end:

```python
sorted(xs, key=lambda x: (x is None, x))   # all None values sort last, rest by value
```

> **PITFALL:** `>` / `<` between `str` and `int` also raises. If a column may be
> "mostly numbers but sometimes a string," decide the type before comparing.

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

> **PITFALL:** `bool` is a subclass of `int`. `True == 1` and `False == 0`, and
> `isinstance(True, int)` is `True`. This is occasionally useful and occasionally
> a bug.

```python
sum(x > 0 for x in nums)        # counts positives: each True counts as 1
{1: "a", True: "b"}             # {1: "b"} — True and 1 are the same dict key
[10, 20][True]                  # 20 — indexing with a bool
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

`divmod(a, b)` returns `(a // b, a % b)` in one call. It is the cleanest idiom
for digit extraction and for converting a flat index to grid coordinates.

```python
q, r = divmod(17, 5)            # (3, 2)
row, col = divmod(idx, cols)    # flat index -> (row, col)
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

Combinatorics without hand-rolled formulas (Python 3.8+):

```python
from math import comb, perm

comb(5, 2)   # 10  -> n choose k
perm(5, 2)   # 20  -> ordered arrangements
```

Reduce a binary function across many values:

```python
from functools import reduce
from math import gcd

g = reduce(gcd, [12, 18, 24])     # 6
total_xor = reduce(lambda a, b: a ^ b, nums, 0)
```

> **PITFALL:** `round` uses banker's rounding (round half to even), not the
> round-half-up you may expect. `round(0.5) == 0`, `round(1.5) == 2`, and
> `round(2.5) == 2`. For half-up behavior, use `math.floor(x + 0.5)` or
> `decimal.Decimal` with an explicit rounding mode.

## 9.7 Floating Point

```python
0.1 + 0.2 == 0.3  # False
```

Compare with tolerance:

```python
abs(a - b) < 1e-9
```

The idiomatic 3.5+ form is `math.isclose`, which also handles relative tolerance:

```python
from math import isclose

isclose(0.1 + 0.2, 0.3)                    # True (relative tolerance)
isclose(a, b, abs_tol=1e-9)                # absolute tolerance near zero
```

Infinity:

```python
from math import inf

best = inf
worst = -inf
```

> **STAFF NOTE:** `float("inf")` as a sentinel in an otherwise integer problem
> silently promotes results to `float`. For pure-integer DP or distances where
> type matters, use `sys.maxsize` as the "large" sentinel to stay in `int`.

```python
import sys

dist = [sys.maxsize] * n   # stays int; arithmetic does not become float
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

## 9.10 Bit Manipulation

Python integers are arbitrary precision, so bit operations never overflow. The
core operators are `& | ^ ~ << >>`.

Essential single-bit tricks:

```python
x & -x            # lowest set bit, isolated (e.g. 0b1100 -> 0b0100)
x & (x - 1)       # x with its lowest set bit cleared
x | (1 << k)      # set bit k
x & ~(1 << k)     # clear bit k
x ^ (1 << k)      # toggle bit k
(x >> k) & 1      # read bit k
x.bit_count()     # number of set bits (Python 3.10+)
x.bit_length()    # bits needed to represent x
```

Iterate over all submasks of a bitmask (a bitmask-DP staple):

```python
sub = mask
while sub:
    use(sub)
    sub = (sub - 1) & mask    # walks every nonzero subset of mask, then 0 exits
```

Number-base conversions:

```python
int("ff", 16)     # 255  -> parse hex string
int("1010", 2)    # 10   -> parse binary string
bin(10)           # "0b1010"
hex(255)          # "0xff"
oct(8)            # "0o10"
format(10, "b")   # "1010"  -> binary without the 0b prefix
```

> **PITFALL:** `~x` is `-(x + 1)` in Python (two's-complement on an infinite-width
> integer). For a fixed-width mask, AND with the width mask: `~x & 0xFFFFFFFF`.

> **TRICK:** A useful index pairing: `~i` indexes from the end, so `arr[~i]` is
> `arr[-(i + 1)]`. This is handy for symmetric two-pointer-style indexing.

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

Parse-oriented methods worth knowing:

```python
"key=val=x".partition("=")    # ('key', '=', 'val=x')  split on FIRST sep, always 3-tuple
"a.b.c".rsplit(".", 1)        # ['a.b', 'c']            split from the right, limited
"a,b,,c".split(",")           # ['a', 'b', '', 'c']     keeps empty fields
"unhappy".removeprefix("un")  # "happy"   (3.9+)
"test.py".removesuffix(".py") # "test"    (3.9+)
s.replace("-", "_")           # replace ALL occurrences (or pass a count)
```

> **PITFALL:** `find` returns `-1` when absent, but `index` raises `ValueError`.
> Pick deliberately: `-1` silently becomes a valid-looking index if you forget to
> check it.

> **TRICK:** `partition` / `rpartition` are safer than `split(sep, 1)` for
> "split on first/last delimiter" because they always return three parts even
> when the delimiter is missing, so unpacking never fails.

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

Slice semantics apply to lists and strings alike. Two surprises for a Java dev:

```python
arr[start:stop:step]    # all three optional; arr[::2] every other; arr[::-1] reversed
arr[100:200]            # returns [] — out-of-range SLICES never raise (arr[100] does)
arr[1:2] = [9, 9, 9]    # list slice assignment can CHANGE length: [1,2,3] -> [1,9,9,9,3]
```

> **PITFALL:** Indexing (`arr[i]`) raises `IndexError` out of range, but slicing
> (`arr[i:j]`) clamps silently and returns whatever overlaps. Do not rely on a
> slice to signal an out-of-bounds bug.

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

## 10.6 f-String Format Specs

The doc uses f-strings throughout; this is the format mini-language after the
colon. The single most useful one for interviews is the `=` debug form.

```python
f"{x=}"          # x=42        self-documenting debug print (3.8+)
f"{x:,}"         # 1,234,567   thousands separator
f"{x:.2f}"       # 3.14        fixed decimals
f"{x:.2%}"       # 42.00%      percentage
f"{n:>5}"        # "   42"     right-align in width 5  (<, ^ = left, center)
f"{n:05d}"       # "00042"     zero-pad to width 5
f"{n:b}"         # "101010"    binary  (o = octal, x = hex)
f"{n:08b}"       # "00101010"  zero-padded binary — handy for bitmask debugging
f"{n:#x}"        # "0x2a"      hex WITH the 0x prefix
```

> **TRICK:** `print(f"{lo=}, {hi=}, {mid=}")` prints both the names and values in
> one go. It is the fastest way to instrument a loop without writing label
> strings by hand.

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

> **INTERNAL:** In Python 3, comprehension loop variables do not leak into the
> enclosing scope (they did in Python 2). After `[x for x in nums]`, the name `x`
> is not defined outside the comprehension. This avoids accidental shadowing.

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

## 12.7 `+=` Mutates In Place; `+` Rebinds

On a list, `+=` is `__iadd__` (in-place mutation, like `.extend`), but `a + [...]`
builds a new list and rebinds the name. Same-looking operators, opposite
aliasing:

```python
a = [1, 2]; b = a
a += [3]          # in place: a and b are BOTH [1, 2, 3]
```

```python
a = [1, 2]; b = a
a = a + [3]       # new object: a is [1, 2, 3], b is still [1, 2]
```

> **PITFALL:** `+=` on a mutable target mutates the shared object, so every alias
> (and the caller's variable) sees the change. `+=` on an immutable target
> (`int`, `str`, `tuple`) rebinds instead, because it cannot mutate.

> **PITFALL:** The notorious case: `t = ([1],); t[0] += [2]` raises `TypeError`
> (the tuple slot cannot be rebound) **and yet still mutates the inner list** to
> `[1, 2]`. The `+=` performs the in-place extend, then fails on the rebind.

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

In production, the clean fix is `functools.cached_property` for per-instance
memoized values, or a `weakref.WeakValueDictionary` cache so cached entries do
not keep their objects alive:

```python
from functools import cached_property
import weakref

class Report:
    @cached_property               # computed once per instance, then stored
    def summary(self):
        return expensive(self.data)

_cache: "weakref.WeakValueDictionary[int, Node]" = weakref.WeakValueDictionary()
```

> **STAFF NOTE:** Saying "I would use a weak-value cache so the cache does not
> extend object lifetime" is a strong staff-level memory-management signal.

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

The two dunders you reach for most often are `__repr__` and `__str__`:

```python
class Node:
    def __init__(self, val):
        self.val = val

    def __repr__(self):                 # unambiguous, for developers/debugging
        return f"Node({self.val!r})"    # the !r conversion applies repr() to val

    def __str__(self):                  # readable, for users/print()
        return str(self.val)
```

```text
__repr__  shown in the REPL, debuggers, and inside containers ([Node(1), Node(2)])
__str__   shown by print() and str(); falls back to __repr__ if not defined
!r in an f-string -> use the object's repr instead of str (f"{node!r}")
```

> **TIP:** In interviews, define `__repr__` (not `__str__`) on a debug helper
> class. It is what shows up when you print a list of your objects, so it makes
> "print the queue to see what's happening" actually useful.

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

A mutable default on a dataclass field is the dataclass form of the mutable
default-argument pitfall — and it raises at class-definition time:

```python
@dataclass
class Bad:
    items: list = []                          # ValueError: mutable default not allowed
```

```python
@dataclass
class Good:
    items: list = field(default_factory=list) # fresh list per instance
```

> **PITFALL:** Use `field(default_factory=list)` (or `dict`, `set`, or any
> zero-arg callable) for mutable field defaults. Unlike a plain function, a
> dataclass refuses the bare-list version outright instead of silently sharing
> it — but the fix is the same idea as `param=None`.

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

> **PITFALL:** Defining `__eq__` on a class sets `__hash__` to `None`, making
> instances **unhashable** — they can no longer go in a set or be dict keys. If
> you need both custom equality and hashability, define `__hash__` too, or use
> `@dataclass(frozen=True)` which generates a consistent pair for you.

```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __eq__(self, other):
        return (self.x, self.y) == (other.x, other.y)

    def __hash__(self):                 # required, or Point is unhashable
        return hash((self.x, self.y))
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

Example:

```python
import platform

print(platform.python_implementation())  # Usually "CPython"
print(platform.python_version())         # Example: "3.11.8"
```

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

Bad:

```python
a = "user_id"
b = "".join(["user", "_id"])

if a is b:          # May be False even though values match.
    ...
```

Good:

```python
if a == b:
    ...
```

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
> The limit is a guard against a C-stack overflow (a hard segfault), not just a
> Python-level error. Raising it without also growing the stack can still crash.

> **TRICK:** The competitive-programming fix for genuinely deep recursion is to
> run the work in a thread with a large stack, which sidesteps the segfault:

```python
import sys, threading

sys.setrecursionlimit(1 << 25)
threading.stack_size(256 * 1024 * 1024)   # 256 MB stack
threading.Thread(target=main).start()
```

> **STAFF NOTE:** The cleaner answer for production or adversarial depth is to
> convert the recursion to an explicit stack. Reserve the big-stack thread trick
> for contest settings where rewriting is not worth the time.

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

Example of a reasonable local binding only when the loop is already hot:

```python
result = []
append = result.append

for x in nums:
    if x > 0:
        append(x)
```

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

Examples of convenience hurting:

```python
# Bad: O(n log n) work repeated after every insert.
arr = []
for x in nums:
    arr.append(x)
    arr = sorted(arr)

# Better for a priority-style need.
import heapq

heap = []
for x in nums:
    heapq.heappush(heap, x)
```

```python
# Bad: copies a suffix on every iteration.
while nums:
    nums = nums[1:]

# Better: move an index or use deque for front removal.
i = 0
while i < len(nums):
    i += 1
```

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

Examples:

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# I/O-bound: overlap waiting on network or disk.
with ThreadPoolExecutor(max_workers=8) as pool:
    pages = list(pool.map(fetch_url, urls))

# CPU-bound pure Python: use processes for parallel bytecode execution.
with ProcessPoolExecutor() as pool:
    scores = list(pool.map(score_large_item, items))
```

```python
import asyncio

async def main(urls):
    results = await asyncio.gather(*(fetch_async(url) for url in urls))
    return results
```

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

# 19. Correct vs Pitfall Example Bank

This section is designed for fast review. Each example shows:

```text
bad code
correct code
why it matters
what to say in an interview
```

The examples are Python-focused. They are not full algorithm problem solutions.

## 19.1 Queue: `list.pop(0)` vs `deque.popleft`

Bad:

```python
q = [start]

while q:
    item = q.pop(0)
```

Correct:

```python
from collections import deque

q = deque([start])

while q:
    item = q.popleft()
```

Why:

```text
list.pop(0) shifts all remaining elements, so it is O(n).
deque.popleft() is O(1).
```

> **SAY THIS:** "I am using `deque` because this is queue behavior; list front
> removal would add a hidden O(n) cost per pop."

## 19.2 2D List Initialization

Bad:

```python
grid = [[0] * cols] * rows
grid[0][0] = 1
```

All rows share the same inner list.

Correct:

```python
grid = [[0] * cols for _ in range(rows)]
grid[0][0] = 1
```

Why:

```text
List multiplication copies references, not nested objects.
The comprehension creates a fresh inner list for each row.
```

## 19.3 Mutable Default Argument

Bad:

```python
def add_item(x, items=[]):
    items.append(x)
    return items
```

Correct:

```python
def add_item(x, items=None):
    if items is None:
        items = []
    items.append(x)
    return items
```

Why:

```text
Default arguments are evaluated once at function definition time.
The bad version shares one list across calls.
```

## 19.4 Sentinel Instead of Truthiness

Bad:

```python
value = d.get(key)

if not value:
    value = compute_default()
```

This treats `0`, `False`, and `""` as missing.

Correct:

```python
MISSING = object()

value = d.get(key, MISSING)
if value is MISSING:
    value = compute_default()
```

Why:

```text
Truthiness is not the same as missingness.
A private sentinel preserves valid falsy values.
```

## 19.5 `is` vs `==`

Bad:

```python
if x is 1000:
    ...
```

Correct:

```python
if x == 1000:
    ...
```

Also correct for identity:

```python
if node is None:
    ...
```

Why:

```text
`is` checks object identity.
`==` checks value equality.
CPython small-int caching can make the bad version appear to work.
```

## 19.6 Sorting: `sort()` Return Value

Bad:

```python
nums = nums.sort()
```

Correct:

```python
nums.sort()
```

Or:

```python
nums = sorted(nums)
```

Why:

```text
list.sort() sorts in place and returns None.
sorted() returns a new sorted list.
```

## 19.7 Slicing in a Hot Path

Bad:

```python
while arr:
    head = arr[0]
    arr = arr[1:]
```

Correct:

```python
i = 0

while i < len(arr):
    head = arr[i]
    i += 1
```

Why:

```text
arr[1:] creates a new list each time.
Passing indices or moving a pointer avoids repeated copies.
```

> **SAY THIS:** "I am avoiding slicing here because each slice allocates and
> copies; an index keeps the loop linear."

## 19.8 String Building

Bad:

```python
s = ""

for part in parts:
    s += part
```

Correct:

```python
s = "".join(parts)
```

Or when transforming:

```python
out = []

for part in parts:
    out.append(transform(part))

s = "".join(out)
```

Why:

```text
Strings are immutable.
Repeated concatenation can create many intermediate strings.
```

## 19.9 Heap Tie-Breaker

Bad:

```python
heappush(heap, (priority, obj))
```

Correct:

```python
from itertools import count

counter = count()
heappush(heap, (priority, next(counter), obj))
```

Why:

```text
Tuple comparison moves to the next field when priorities tie.
If obj is not orderable, the heap operation can crash.
The counter makes every heap record comparable without comparing obj.
```

## 19.10 `defaultdict` Auto-Creation

Bad:

```python
from collections import defaultdict

groups = defaultdict(list)

if groups[key]:
    ...
```

This creates `key` even if it was absent.

Correct:

```python
if key in groups:
    ...
```

Or:

```python
items = groups.get(key, [])
```

Why:

```text
Accessing a missing defaultdict key mutates the dictionary.
That can corrupt logic based on key presence or dictionary size.
```

## 19.11 `Counter` Zero Counts

Bad:

```python
freq[x] -= 1

if len(freq) == expected:
    ...
```

Correct:

```python
freq[x] -= 1

if freq[x] == 0:
    del freq[x]
```

Why:

```text
Counter can keep keys with zero or negative counts.
If len(freq) means active distinct keys, delete zero-count entries.
```

## 19.12 Cache With Unhashable State

Bad:

```python
from functools import cache

@cache
def f(state):
    ...

f([1, 2, 3])
```

Correct:

```python
@cache
def f(state):
    ...

f((1, 2, 3))
```

Why:

```text
cache keys are function arguments.
Arguments must be hashable.
Lists are mutable and unhashable; tuples are hashable if their contents are.
```

## 19.13 Method Cache Trap

Risky:

```python
class Service:
    @lru_cache(None)
    def compute(self, key):
        ...
```

Often better:

```python
class Service:
    def compute_all(self, keys):
        @lru_cache(None)
        def compute_one(key):
            ...

        return [compute_one(key) for key in keys]
```

Why:

```text
Decorating a method includes self in the cache key.
That can keep instances alive and create surprising cache lifetime.
Nested cached helpers keep cache lifetime scoped to one operation.
```

## 19.14 Integer Division

Bad when truncation toward zero is required:

```python
result = a // b
```

Correct:

```python
sign = -1 if (a < 0) ^ (b < 0) else 1
result = sign * (abs(a) // abs(b))
```

Why:

```text
Python // floors toward negative infinity.
Java integer division truncates toward zero.
```

## 19.15 `or` Defaults With Valid Zero

Bad:

```python
limit = user_limit or 100
```

Correct:

```python
limit = 100 if user_limit is None else user_limit
```

Why:

```text
0 is falsy but may be a valid configured limit.
Use an explicit None check for missingness.
```

## 19.16 Context Manager Cleanup

Bad:

```python
f = open(path)
data = f.read()
f.close()
```

If `read()` raises, `close()` may not run.

Correct:

```python
with open(path) as f:
    data = f.read()
```

Why:

```text
The context manager guarantees cleanup even when an exception occurs.
```

## 19.17 Custom Context Manager Exception Handling

Risky:

```python
class Manager:
    def __exit__(self, exc_type, exc, tb):
        return True
```

Correct by default:

```python
class Manager:
    def __exit__(self, exc_type, exc, tb):
        cleanup()
        return False
```

Why:

```text
Returning True suppresses exceptions.
Most resource managers should cleanup and let exceptions propagate.
```

## 19.18 Decorator Metadata

Bad:

```python
def traced(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

Correct:

```python
from functools import wraps

def traced(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

Why:

```text
wraps preserves function name, docstring, annotations, and metadata.
This matters for debugging, observability, introspection, and frameworks.
```

## 19.19 Generator Reuse

Bad:

```python
values = (x * x for x in nums)

total = sum(values)
again = list(values)
```

`again` is empty.

Correct:

```python
values = [x * x for x in nums]

total = sum(values)
again = list(values)
```

Or recreate the generator:

```python
def squares(nums):
    for x in nums:
        yield x * x
```

Why:

```text
Generators are one-shot iterators.
Use a list when you need reuse.
Use a generator when you need streaming.
```

## 19.20 Mutable Class Attribute

Bad:

```python
class Registry:
    items = []
```

Correct:

```python
class Registry:
    def __init__(self):
        self.items = []
```

Why:

```text
Class attributes are shared by all instances.
Instance attributes belong to each object.
```

## 19.21 Mutable Object as Hash Key

Bad design:

```python
class User:
    def __init__(self, email):
        self.email = email

    def __hash__(self):
        return hash(self.email)

    def __eq__(self, other):
        return self.email == other.email
```

Then:

```python
u = User("a@example.com")
seen = {u}
u.email = "b@example.com"
```

Correct options:

```python
@dataclass(frozen=True)
class UserKey:
    email: str
```

Or use immutable key directly:

```python
seen = {user.email}
```

Why:

```text
If an object's hash changes after insertion into a set/dict, lookup behavior is
broken.
Hashable keys should be immutable with respect to equality/hash fields.
```

## 19.22 Lock Around Shared State

Bad:

```python
count = 0

def increment():
    global count
    count += 1
```

Correct:

```python
from threading import Lock

count = 0
lock = Lock()

def increment():
    global count
    with lock:
        count += 1
```

Why:

```text
The GIL does not make multi-step shared-state invariants safe.
Use locks or avoid shared mutable state.
```

## 19.23 Producer/Consumer Queue

Risky:

```python
items = []  # shared between threads
```

Correct:

```python
from queue import Queue

q = Queue()
q.put(item)
item = q.get()
q.task_done()
```

Why:

```text
Queue provides thread-safe coordination.
Manual list sharing needs explicit locking and condition signaling.
```

## 19.24 Do Not Hold Locks During Blocking I/O

Bad:

```python
with lock:
    data = fetch_from_network()
    cache[key] = data
```

Better shape:

```python
data = fetch_from_network()

with lock:
    cache[key] = data
```

Why:

```text
Holding locks during slow I/O increases contention and deadlock risk.
Keep critical sections small.
```

## 19.25 Thread Pool Exceptions

Bad:

```python
with ThreadPoolExecutor() as executor:
    for item in items:
        executor.submit(work, item)
```

Exceptions may be hidden if futures are ignored.

Correct:

```python
with ThreadPoolExecutor() as executor:
    futures = [executor.submit(work, item) for item in items]

    for future in as_completed(futures):
        result = future.result()
```

Why:

```text
future.result() re-raises worker exceptions.
Ignoring futures can hide failures.
```

## 19.26 Process Pool Pickling

Bad:

```python
def run(items):
    helper = lambda x: x + 1

    with ProcessPoolExecutor() as executor:
        return list(executor.map(helper, items))
```

Correct:

```python
def helper(x):
    return x + 1

def run(items):
    with ProcessPoolExecutor() as executor:
        return list(executor.map(helper, items))
```

Why:

```text
Process pools send work to child processes.
Functions and arguments generally need to be picklable.
Top-level functions are safer than lambdas or local functions.
```

## 19.27 Async Blocking Call

Bad:

```python
async def handler():
    time.sleep(1)
    return "done"
```

Correct:

```python
async def handler():
    await asyncio.sleep(1)
    return "done"
```

Why:

```text
Blocking calls inside async code block the event loop.
Use async-compatible APIs or move blocking work to an executor.
```

## 19.28 Clean One-Liner vs Too Clever

Acceptable:

```python
names = [user.name for user in users if user.active]
```

Too clever:

```python
result = {k: [f(x) for x in v if g(x)] for k, v in data.items() if h(k, v)}
```

Often clearer:

```python
result = {}

for key, values in data.items():
    if not h(key, values):
        continue

    kept = []
    for value in values:
        if g(value):
            kept.append(f(value))

    result[key] = kept
```

Why:

```text
Comprehensions are great for simple transforms.
Nested business logic should be readable and debuggable.
```

> **STAFF NOTE:** Great Python is not maximal compression. Great Python makes
> cost, ownership, and correctness obvious.

---

# 20. The Complete Pitfall Catalog

## 20.1 Mutable Default Arguments

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

## 20.2 Aliased 2D Lists

Bad:

```python
grid = [[0] * cols] * rows
```

Good:

```python
grid = [[0] * cols for _ in range(rows)]
```

## 20.3 List Used as Queue

Bad:

```python
item = q.pop(0)
```

Good:

```python
from collections import deque

item = q.popleft()
```

## 20.4 Slicing in Hot Paths

Bad:

```python
part = arr[i:]
```

inside a large loop.

Prefer indices or views via boundaries.

## 20.5 String Concatenation in a Loop

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

## 20.6 `sort()` Returns `None`

Bad:

```python
arr = arr.sort()
```

Good:

```python
arr.sort()
```

## 20.7 `is` vs `==`

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

## 20.8 Division with Negatives

```python
-7 // 2  # -4
```

If you need truncation toward zero, implement it explicitly.

## 20.9 `/` Produces Float

```python
6 / 2  # 3.0
```

Use `//` for integer indices.

## 20.10 Modifying While Iterating

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

## 20.11 Dict Mutation During Iteration

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

## 20.12 Assignment Is Aliasing

```python
b = a
```

does not copy.

## 20.13 Heap Tie Comparisons

Bad:

```python
heappush(heap, (priority, obj))
```

Good:

```python
heappush(heap, (priority, next(counter), obj))
```

## 20.14 `Counter` Zero Counts

```python
freq[x] -= 1
```

may leave `x` in the counter with count zero.

Delete when distinct count matters:

```python
if freq[x] == 0:
    del freq[x]
```

## 20.15 `defaultdict` Auto-Creation

```python
d = defaultdict(list)
d[key]
```

creates `key`.

## 20.16 Truthiness of Valid Values

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

## 20.17 Float Equality

Bad:

```python
a == b
```

for computed floats.

Good:

```python
abs(a - b) < 1e-9
```

## 20.18 Integer Overflow Assumptions

Python will not overflow automatically. If bounds matter, check manually.

```python
value = 2**63

if value > 2**31 - 1:
    raise ValueError("outside 32-bit signed integer range")
```

## 20.19 Dict Order Is Not Sorted

Dict preserves insertion order, not key order.

Use:

```python
for key in sorted(d):
    ...
```

## 20.20 Set Order Is Not Stable Sorted Order

Do not use set iteration for sorted output.

```python
nums = {3, 1, 2}

for x in sorted(nums):
    print(x)
```

## 20.21 Late-Binding Closures

Bad:

```python
funcs = [lambda: i for i in range(3)]
```

Good:

```python
funcs = [lambda i=i: i for i in range(3)]
```

## 20.22 `nonlocal` Missing

If you reassign an outer variable inside a nested function, use `nonlocal`.

```python
def counter():
    count = 0

    def inc():
        nonlocal count
        count += 1
        return count

    return inc
```

## 20.23 Generator Exhaustion

```python
it = map(int, ["1", "2"])
list(it)  # [1, 2]
list(it)  # []
```

## 20.24 `zip` Truncation

`zip` stops at the shortest iterable.

```python
names = ["a", "b", "c"]
scores = [10, 20]

list(zip(names, scores))  # [("a", 10), ("b", 20)]
```

## 20.25 `range` Is Not a List

Usually good. Materialize only if needed.

```python
r = range(3)
list(r)  # [0, 1, 2]
```

## 20.26 `sum(list_of_lists, [])`

Bad flattening pattern. Use `chain.from_iterable`.

```python
from itertools import chain

matrix = [[1, 2], [3, 4]]
flat = list(chain.from_iterable(matrix))
```

## 20.27 `bisect.insort` Is O(n)

Do not mistake it for a tree insert.

```python
import bisect

arr = [1, 3, 5]
bisect.insort(arr, 4)  # search is O(log n), insertion shift is O(n)
```

## 20.28 Cache Arguments Unhashable

Lists, dicts, and sets cannot be cache keys.

Convert to tuples when needed.

```python
from functools import lru_cache

@lru_cache(None)
def solve(state):
    return sum(state)

solve(tuple([1, 2, 3]))
```

## 20.29 Cache on Methods Includes `self`

Prefer nested cached helpers in interview code.

```python
from functools import lru_cache

class Solution:
    def countWays(self, n):
        @lru_cache(None)
        def dp(i):
            if i <= 1:
                return 1
            return dp(i - 1) + dp(i - 2)

        return dp(n)
```

## 20.30 Class-Level Mutable State

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

## 20.31 Boolean `or` Defaults

Bad when `0` is valid:

```python
limit = user_limit or 10
```

Good:

```python
limit = 10 if user_limit is None else user_limit
```

## 20.32 NaN

```python
float("nan") != float("nan")
```

## 20.33 Shadowing Built-ins

Avoid:

```python
list = []
dict = {}
sum = 0
```

You lose access to the built-in name.

## 20.34 Leaking State Across Test Cases

On online judges, the same `Solution` instance behavior varies by platform.
Avoid persistent mutable state unless intended.

Prefer local variables inside the method.

Bad:

```python
class Solution:
    seen = set()

    def containsDuplicate(self, nums):
        for x in nums:
            if x in self.seen:
                return True
            self.seen.add(x)
        return False
```

Good:

```python
class Solution:
    def containsDuplicate(self, nums):
        seen = set()
        for x in nums:
            if x in seen:
                return True
            seen.add(x)
        return False
```

## 20.35 Over-Clever One-Liners

Python allows dense code. Interviews reward readable code.

Prefer:

```python
result = []
for x in nums:
    if valid(x):
        result.append(transform(x))
```

Over a deeply nested comprehension that the interviewer cannot inspect quickly.

## 20.36 Vacuous Truth of `all([])`

`all([])` is `True` and `any([])` is `False`. A guard built on `all(...)` over a
possibly empty iterable can pass when you expected it to fail.

```python
all([])  # True
any([])  # False
```

## 20.37 `bool` Is an `int`

`True == 1` and `False == 0`. Booleans collapse with `0`/`1` as dict keys and
count inside `sum`.

```python
{1: "a", True: "b"}  # {1: "b"}
```

## 20.38 Banker's Rounding

`round` rounds half to even, not half up.

```python
round(0.5)  # 0
round(2.5)  # 2
```

## 20.39 `__eq__` Removes `__hash__`

Defining `__eq__` without `__hash__` makes the class unhashable.

```python
class P:
    def __eq__(self, other): ...
# instances can no longer be put in a set or used as dict keys
```

Define `__hash__` too, or use `@dataclass(frozen=True)`.

## 20.40 `float('inf')` Contaminates Integer Math

Using `float("inf")` as a sentinel promotes integer results to `float`. Use
`sys.maxsize` to stay in `int` when type matters.

## 20.41 `groupby` Groups Only Consecutive Keys

`itertools.groupby` starts a new group whenever the key changes. Sort first if
you want all equal keys grouped regardless of position.

---

# 21. Staff-Level Python Communication

## 21.1 When Choosing a Data Structure

> **SAY THIS:** "I am using a dict here because I need average O(1) lookup, and
> the O(n) extra space is the tradeoff."

> **SAY THIS:** "I am using a deque rather than a list because removing from the
> front of a list is O(n)."

> **SAY THIS:** "I am using tuples as keys because lists are mutable and
> unhashable."

> **SAY THIS:** "I am avoiding slicing here because it would allocate a new list
> each time."

Example interview framing:

```text
I need a FIFO queue, so I will use deque.
append and popleft are O(1), while list.pop(0) would shift the remaining items.
```

## 21.2 When Discussing Python vs Java

> **SAY THIS:** "Python lets me express the core logic with fewer lines, but I
> need to be careful about hidden copies, object overhead, recursion depth, and
> list operations that shift elements."

> **SAY THIS:** "Java has built-in tree-backed maps and sets; Python's standard
> library does not, so ordered dynamic operations need a different plan unless
> `sortedcontainers` is allowed."

Example:

```text
In Java I might reach for TreeMap for floor/ceiling queries.
In Python stdlib, I would either sort once and use bisect, use a heap if I only
need min/max priority access, or confirm that sortedcontainers is available.
```

## 21.3 When Asked About Complexity

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

## 21.4 When Asked About Internals

> **SAY THIS:** "A Python list is a dynamic array of references, so append is
> amortized O(1), but inserting or deleting near the front shifts references and
> is O(n)."

> **SAY THIS:** "A dict is a hash table, so lookup is average O(1), assuming
> good hashing and non-adversarial collisions."

> **SAY THIS:** "Strings are immutable, so repeated concatenation can allocate
> many intermediate strings. I would collect parts and join once."

Example:

```text
This list stores references, not inline objects. Keeping many tiny custom
objects can cost more memory than a compact tuple or list representation.
```

## 21.5 When You Need a Non-Standard Library

> **SAY THIS:** "If `sortedcontainers` is allowed, this is straightforward with
> a sorted list or sorted dict. If not, I would avoid assuming it and choose a
> standard-library design."

Example:

```python
# If external libraries are not guaranteed, avoid:
# from sortedcontainers import SortedList

import bisect

arr = []
bisect.insort(arr, value)  # Fine for small n; not a tree replacement.
```

## 21.6 When Recursion Might Fail

> **SAY THIS:** "The recursive version is clean, but Python has a recursion
> limit. If depth can be large, I would use an explicit stack rather than just
> raising the recursion limit."

Example:

```python
stack = [root]
while stack:
    node = stack.pop()
    for child in node.children:
        stack.append(child)
```

---

# 22. Final Cheat Sheets

## 22.1 Imports

```python
from collections import defaultdict, Counter, deque, OrderedDict
from heapq import heappush, heappop, heappushpop, heapreplace, heapify, merge, nlargest, nsmallest
from bisect import bisect_left, bisect_right, insort
from functools import cache, lru_cache, cmp_to_key, reduce, total_ordering
from itertools import count, chain, product, pairwise, accumulate, groupby, zip_longest, permutations, combinations
from operator import itemgetter, attrgetter
from math import inf, gcd, lcm, isqrt, comb, perm, isclose
from threading import Lock, RLock, Event, Semaphore, Condition
from queue import Queue
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed
from contextlib import contextmanager, ExitStack
from dataclasses import dataclass, field
```

## 22.2 Structure Choice

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

Example choices:

```python
from collections import Counter, defaultdict, deque
import heapq

q = deque()
q.append(start)
node = q.popleft()                 # queue

freq = Counter(nums)               # frequency
groups = defaultdict(list)         # grouping
groups[key].append(value)

heap = []
heapq.heappush(heap, (priority, item))
priority, item = heapq.heappop(heap)
```

## 22.3 Complexity Must-Know

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

## 22.4 One-Line Idiom Recall

```python
r, c = divmod(idx, cols)                 # flat index -> grid coordinate
cols = list(zip(*grid))                  # transpose / unzip
pre  = list(accumulate(nums, initial=0)) # prefix sums with leading 0
top  = -heappop(h)                       # max-heap via negation
heappush(h, (prio, next(counter), obj))  # heap with safe tie-break
g.setdefault(k, []).append(v)            # group without defaultdict
while (x := next_value()):               # assign-and-test
x & -x                                   # lowest set bit
x & (x - 1)                              # clear lowest set bit
sub = (sub - 1) & mask                   # enumerate submasks
int("ff", 16); bin(x); format(x, "b")    # base conversions
sum(1 for x in xs if ok(x))              # count by predicate
i = bisect_right(a, x) - 1               # floor index (largest a[i] <= x)
i = bisect_left(a, x)                    # ceiling index (smallest a[i] >= x)
w = deque(maxlen=k)                      # auto-evicting sliding window
sorted(xs, key=lambda v: (v is None, v)) # sort with None pushed to the end
merged = {**d1, **d2}                    # merge dicts (right side wins)
```

## 22.5 Night-Before Pitfall Scan

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
Did all([]) / any([]) on an empty iterable mislead a guard?
Did I forget bool is an int (sum of bools, True as a dict key)?
Did round() surprise me with banker's rounding?
Did I add __eq__ but forget __hash__?
Did float('inf') turn an integer result into a float?
Did groupby only group consecutive keys when I wanted all of them?
Did I compare floats with == instead of math.isclose?
Did I use split(" ") when split() (any-whitespace, no empties) was meant?
Did sorted() hit a TypeError from a stray None or mixed types?
Did += on a list mutate a value the caller still aliases?
Did a bare except: swallow KeyboardInterrupt or SystemExit?
Did a dataclass field use a bare mutable default instead of default_factory?
Did I forget that out-of-range slices return [] instead of raising?
```

## 22.6 Final Rule

```text
Use Python to make the algorithm clear.
Do not use Python cleverness to hide the algorithm.
```

---

# 23. Official References

Authoritative sources for everything in this guide. When an interviewer or
platform disagrees with this doc, these win.

## 23.1 Core Language and Data Model

- The Python Language Reference — https://docs.python.org/3/reference/
- Data model (`__eq__`, `__hash__`, `__lt__`, dunders) — https://docs.python.org/3/reference/datamodel.html
- Assignment expressions (walrus `:=`), PEP 572 — https://peps.python.org/pep-0572/
- Built-in functions — https://docs.python.org/3/library/functions.html
- Built-in types (list, dict, set, str, int, float) — https://docs.python.org/3/library/stdtypes.html

## 23.2 Standard Library Modules Used Here

- `collections` (deque, Counter, defaultdict, OrderedDict) — https://docs.python.org/3/library/collections.html
- `heapq` — https://docs.python.org/3/library/heapq.html
- `bisect` — https://docs.python.org/3/library/bisect.html
- `functools` (cache, lru_cache, reduce, total_ordering) — https://docs.python.org/3/library/functools.html
- `itertools` (accumulate, groupby, product, pairwise, zip_longest) — https://docs.python.org/3/library/itertools.html
- `math` (isqrt, comb, perm, gcd, lcm, isclose, inf) — https://docs.python.org/3/library/math.html
- `dataclasses` — https://docs.python.org/3/library/dataclasses.html
- `operator` (itemgetter, attrgetter) — https://docs.python.org/3/library/operator.html

## 23.3 Concurrency

- `threading` — https://docs.python.org/3/library/threading.html
- `concurrent.futures` — https://docs.python.org/3/library/concurrent.futures.html
- `multiprocessing` — https://docs.python.org/3/library/multiprocessing.html
- `asyncio` — https://docs.python.org/3/library/asyncio.html
- `queue` — https://docs.python.org/3/library/queue.html
- The GIL, from the CPython design FAQ — https://docs.python.org/3/faq/library.html#can-t-we-get-rid-of-the-global-interpreter-lock

## 23.4 Performance and Internals

- TimeComplexity of built-in operations (community wiki) — https://wiki.python.org/moin/TimeComplexity
- Sorting HOW TO (key, stability, Timsort) — https://docs.python.org/3/howto/sorting.html
- `sys` (recursion limit, maxsize, getrefcount) — https://docs.python.org/3/library/sys.html
- `gc` and `weakref` — https://docs.python.org/3/library/gc.html and https://docs.python.org/3/library/weakref.html

## 23.5 Third-Party (Confirm Availability First)

- `sortedcontainers` (SortedList, SortedDict, SortedSet) — https://grantjenks.com/docs/sortedcontainers/

> **STAFF NOTE:** Knowing where the answer lives is itself a senior signal. If
> asked something you are unsure about, naming the authoritative source and your
> reasoning beats guessing confidently.
