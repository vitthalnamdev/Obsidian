### ⚔️ CLASSIC USE CASES

#### 1. **Range Sum / Min / Max / GCD / LCM / Product Query**

- Each node stores the sum/min/max/gcd/lcm/product of a range.
    
- Standard Segment Tree.
    

---

### 📊 ORDER-BASED QUERIES

#### 2. **Counting Inversions**

- **Idea**: Build a segment tree on a compressed array or frequency array.
    
- **Use**: While iterating from right to left, for each `a[i]`, count how many elements `< a[i]` (query sum in [1, a[i]-1]) and then insert `a[i]`.
    
- **Build**: 0s initially; update as you go.
    

#### 3. **K-th Order Statistic in a Subarray**

- Use a **Merge Sort Tree** or **Wavelet Tree**.
    
- At each node, store sorted values. Perform binary search to find the k-th element in range `[l, r]`.
    

#### 4. **Count of Elements < / ≤ / > / ≥ k in a Range**

- Same setup as merge sort tree.
    
- Binary search in each segment to count.
    

---

### 📍POSITIONAL QUERIES

#### 5. **Rightmost Element ≥/≤ x from Index i**

- Store a segment tree with max/min in nodes.
    
- Perform a **custom binary search** on the segment tree.
    
- Works similarly to `lower_bound` but on a segment tree.
    

#### 6. **Leftmost Element ≥/≤ x from Index i**

- Same as above but binary search goes to the left.
    

---

### ⚙️ FREQUENCY / OCCURRENCE QUERIES

#### 7. **Frequency of an Element in a Range**

- Use segment tree of maps or use Merge Sort Tree with binary search.
    

#### 8. **Majority Element in a Range**

- Use **Boyer-Moore Voting** with segment tree to track candidates.
    
- Use second pass to verify if it appears > (r - l + 1)/2 times.
    

---

### 🧮 DYNAMIC STRUCTURES

#### 9. **Dynamic Segment Tree**

- For sparse data or large coordinate range (e.g., up to 1e9).
    
- Nodes created only when needed.
    
- Perfect for competitive problems with updates and queries on large ranges.
    

#### 10. **Persistent Segment Tree**

- Keep versions of the tree after updates.
    
- Use case: Historical queries, rollback, versioning, range k-th query.
    

---

### 💥 ADVANCED AND CUSTOM USE CASES

#### 11. **Range Count of a Specific Value**

- Augment each node with a hashmap or array counting occurrences.
    

#### 12. **Sum of Element-wise Min(a[i], b[i]) over range**

- If both `a` and `b` are static: preprocess.
    
- If one is updatable, segment tree over one array and query accordingly.
    

#### 13. **Sum of GCDs over All Subarrays / Ranges**

- Use segment tree with GCD nodes.
    
- Combine with two-pointers or stack for optimization.
    

---

### ⚡BITMASK / LAZY / MULTIDIMENSIONAL

#### 14. **Lazy Propagation Segment Tree**

- For range updates and range queries.
    
- Crucial when dealing with frequent range modifications.
    

#### 15. **Bitmask Segment Tree**

- Store bitmasks in nodes (e.g., track which elements exist in a range).
    
- Useful for problems like finding how many distinct elements exist.
    

#### 16. **2D Segment Tree**

- Segment tree on both rows and columns.
    
- Useful in matrix queries: sum/min/max in a rectangular region.
    

---

### 🧠 OTHER INTERESTING IDEAS

#### 17. **Binary Search on Segment Tree**

- Used to find leftmost/rightmost element satisfying a condition.
    
- Trick: `query(l, r) >= x` → binary search on answer range using segtree.
    

#### 18. **Range Minimum Position (not value)**

- Store value and index in the node. On tie, compare index.
    

#### 19. **Euler Tour + Segment Tree for LCA**

- Preprocess Euler tour + RMQ on depths → lowest common ancestor in O(log N).
    

#### 20. **Difference Array + Segment Tree**

- Maintain prefix sums on the difference array for range increment problems.
    

---

### 🛠 Examples You Mentioned

#### ✅ Finding Inversions

- Count of elements less than `a[i]` to its right → Fenwick Tree or Segment Tree with frequencies.
    

#### ✅ Finding Rightmost Smallest / Largest from index i

- Use max/min segment tree + custom binary search over the segment tree.