https://leetcode.com/discuss/post/1879225/Google-L3or-L4-(L3-passed)/

Here is the Java solution to find the longest valid string in the dictionary based on your rules.
### **Algorithm Explanation**
To solve this efficiently, we can use **Depth-First Search (DFS)** combined with **Memoization** (caching results so we don't calculate the same word twice).
 1. **Store Data:** Load all the words from the given array into a HashSet for O(1) fast lookups.
 2. **Sort:** Sort the input dictionary by string length in descending order. This ensures that the first valid string we find is guaranteed to be the longest one. If there's a tie in length, we can sort lexicographically as a tie-breaker.
 3. **Verify Validity (DFS):** * **Base case:** If the string's length is 1, return true (since we already know it's in the dictionary).
   * **Recursive step:** Iterate through the string, removing one character at a time. If the newly formed substring is in the dictionary *and* it is recursively valid, then the current string is valid.
   * **Memoize:** Store whether a string is valid or not in a HashMap to avoid redundant computations.
### **Java Implementation**
```java
import java.util.*;

public class LongestValidString {

    public static String findLongestValidWord(String[] words) {
        // Step 1: Put all words in a HashSet for O(1) lookup
        Set<String> dict = new HashSet<>(Arrays.asList(words));
        
        // Memoization map to store if a string is valid or not
        Map<String, Boolean> memo = new HashMap<>();

        // Step 2: Sort words by length descending. 
        // If lengths are equal, sort lexicographically.
        Arrays.sort(words, (a, b) -> {
            if (a.length() != b.length()) {
                return b.length() - a.length(); // Descending length
            }
            return a.compareTo(b); // Lexicographical tie-breaker
        });

        // Step 3: Check each word starting from the longest
        for (String word : words) {
            if (isValid(word, dict, memo)) {
                return word; // The first valid word found is the longest
            }
        }

        return ""; // Return empty string if no valid word is found
    }

    private static boolean isValid(String word, Set<String> dict, Map<String, Boolean> memo) {
        // Base Case: If we successfully strip it down to 1 character, it's valid
        if (word.length() == 1) {
            return true;
        }
        
        // Check if we've already processed this word
        if (memo.containsKey(word)) {
            return memo.get(word);
        }

        boolean result = false;
        
        // Try removing exactly one character at a time
        for (int i = 0; i < word.length(); i++) {
            // Create the substring by omitting the character at index i
            String nextWord = word.substring(0, i) + word.substring(i + 1);
            
            // If the dictionary contains the smaller word, check its validity
            if (dict.contains(nextWord) && isValid(nextWord, dict, memo)) {
                result = true;
                break; // No need to check other combinations if one path is valid
            }
        }

        // Save the result in memoization map and return
        memo.put(word, result);
        return result;
    }

    public static void main(String[] args) {
        // Your example dictionary
        String[] dictionary = {"string", "sring", "sing", "wording", "ing", "ng", "g"};
        
        String longestWord = findLongestValidWord(dictionary);
        
        System.out.println("Dictionary: " + Arrays.toString(dictionary));
        System.out.println("Longest valid string is: " + longestWord);
    }
}

```
### **Complexity Analysis**
 * **Time Complexity:** O(N \log N \cdot L + N \cdot L^2)
   Sorting the array takes O(N \log N \cdot L) where N is the number of words and L is the maximum length of a word. The DFS checks at most N states, and for each state, it does string concatenations up to L times.
 * **Space Complexity:** O(N \cdot L)
   To store the words in the HashSet and the HashMap cache, along with the memory consumed by the recursive call stack.


---

Here is the formal problem statement based on your description, followed by the solutions in both Python and Java.
### Problem Statement: Number of Distinct Islands
**Description:**
Given an m x n binary matrix grid where 1 represents land and 0 represents water, an island is a group of 1s connected 4-directionally (horizontal or vertical). You may assume all four edges of the grid are surrounded by water.
An island is considered to be the same as another if and only if one island can be translated (moved horizontally and/or vertically) to equal the other. Rotations and reflections do not count as translations; they are considered unique shapes.
Return the number of distinct islands.
**Example Input:**
```text
grid = [
  [1, 1, 1, 1, 0, 0],
  [1, 1, 0, 0, 0, 1],
  [0, 0, 1, 1, 0, 1],
  [1, 1, 0, 0, 0, 0],
  [0, 0, 1, 1, 1, 1],
  [1, 0, 1, 1, 0, 0]
]

```
**Example Output:**
4
**Explanation:**
There are 6 islands in total, but only 4 unique shapes:
 1. One 6-sized shape (appears twice: top-left and bottom-right)
 2. One 2-sized horizontal shape (appears twice: middle and middle-left)
 3. One 2-sized vertical shape (appears once: top-right)
 4. One 1-sized shape (appears once: bottom-left)
### How the Solution Works
To identify unique shapes, we can iterate through the grid. When we find a 1, we trigger a Depth-First Search (DFS) to explore the entire island.
Because we iterate through the grid from top-to-bottom, left-to-right, we will always discover the same shapes starting from their "top-left-most" block. By recording the coordinates of every cell in the island *relative to this starting block* (subtracting the starting row and column from the current row and column), we generate a translation-invariant signature for the shape. We store these signatures in a hash set to easily filter out duplicates.
### Python Solution
```python
def numDistinctIslands(grid: list[list[int]]) -> int:
    if not grid or not grid[0]:
        return 0

    rows, cols = len(grid), len(grid[0])
    unique_shapes = set()

    def dfs(r, c, base_r, base_c, shape):
        # Out of bounds or water/already visited
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] == 0:
            return
        
        # Mark as visited by sinking the island
        grid[r][c] = 0 
        
        # Add relative coordinates to the shape
        shape.append((r - base_r, c - base_c))
        
        # Explore all 4 directions deterministically
        dfs(r + 1, c, base_r, base_c, shape)
        dfs(r - 1, c, base_r, base_c, shape)
        dfs(r, c + 1, base_r, base_c, shape)
        dfs(r, c - 1, base_r, base_c, shape)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 1:
                current_shape = []
                dfs(r, c, r, c, current_shape)
                # Tuples are hashable, so we can add the list of coordinates to a set
                unique_shapes.add(tuple(current_shape))

    return len(unique_shapes)

# Test with the example input
grid = [
    [1, 1, 1, 1, 0, 0],
    [1, 1, 0, 0, 0, 1],
    [0, 0, 1, 1, 0, 1],
    [1, 1, 0, 0, 0, 0],
    [0, 0, 1, 1, 1, 1],
    [1, 0, 1, 1, 0, 0]
]

print(numDistinctIslands(grid)) # Output: 4

```
### Java Solution
```java
import java.util.HashSet;
import java.util.Set;

public class DistinctIslands {
    
    public int numDistinctIslands(int[][] grid) {
        if (grid == null || grid.length == 0) return 0;
        
        Set<String> uniqueShapes = new HashSet<>();
        int rows = grid.length;
        int cols = grid[0].length;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 1) {
                    StringBuilder shape = new StringBuilder();
                    dfs(grid, r, c, r, c, shape);
                    // Add the encoded string representation to our set
                    uniqueShapes.add(shape.toString());
                }
            }
        }
        
        return uniqueShapes.size();
    }

    private void dfs(int[][] grid, int r, int c, int baseR, int baseC, StringBuilder shape) {
        // Out of bounds or water/already visited
        if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length || grid[r][c] == 0) {
            return;
        }
        
        // Mark as visited
        grid[r][c] = 0; 
        
        // Append relative coordinates to string builder
        shape.append(r - baseR).append(",").append(c - baseC).append(";");
        
        // Explore all 4 directions deterministically
        dfs(grid, r + 1, c, baseR, baseC, shape);
        dfs(grid, r - 1, c, baseR, baseC, shape);
        dfs(grid, r, c + 1, baseR, baseC, shape);
        dfs(grid, r, c - 1, baseR, baseC, shape);
    }

    public static void main(String[] args) {
        DistinctIslands solution = new DistinctIslands();
        int[][] grid = {
            {1, 1, 1, 1, 0, 0},
            {1, 1, 0, 0, 0, 1},
            {0, 0, 1, 1, 0, 1},
            {1, 1, 0, 0, 0, 0},
            {0, 0, 1, 1, 1, 1},
            {1, 0, 1, 1, 0, 0}
        };
        
        System.out.println(solution.numDistinctIslands(grid)); // Output: 4
    }
}

```

---

