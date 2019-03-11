---
title: Review Outline on Interview
tags: []
categories: []
keywords: []
date: 2019-03-02 11:27:52
mathjax: true   
---
Going through all this before you take the tech interview.
<!--more-->

### Union Find
#### applications
- Percolation.
- Games (Go, Hex).
- Dynamic connectivity.
- Least common ancestor.
- Equivalence of finite state automata.
- Hoshen-Kopelman algorithm in physics.
- Hinley-Milner polymorphic type inference.
- Kruskal's minimum spanning tree algorithm.
- Compiling equivalence statements in Fortran.
- Morphological attribute openings and closings.
- Matlab's bwlabel() function in image processing.

### Stacks and Queues

- LinkedList implementation.
- Array implementation. Array is between 25% and 100% full.(why? think about when to resize array)
- Can always use a stack to replace recursion.
- Dijkstra's 2-stack algorithms to solve arithmetic expression. (It's a compiler!)

### Elementary Sorting

| type | time avg | best | worst | space| comment|
| ------ | ------ | ------ | ---|------ |-- |
| selection sort | $N^2$ |  | | 0 | |
| insertion sort | $N^2$ | $N$ | $~\cfrac{1}{2}N^2$ | 0 |For partially-sorted arrays, insertion sort runs in linear time.Number of exchanges equals the number of inversions.An inversion is a pair of keys that are out of order.   |
| shell sort |  |  | |  | |

#### shuffle sort
Generate a random real number for every array entry, and then sort using those random numbers as the keys.
    - Generate a random real number for each array entry.
    - Sort the array.
Then go better -- Knuth shuffle:
    - In iteration i, pick integer r between 0 and i uniformly at random.
    - Swap a[i] and a[r].
    

#### convex hull
The convex hull of a set of N points is the smallest perimeter fence enclosing the points -- graham scan.

#### binary search
```java
...
    while(l <= r){
        int index = l + (r-l)/2;
        if(value < nums[index]) r = index-1;
        else if (value > nums[index]) l = index+1;
        else return index;
    }
return -1;
```

### Merge Sort

Merge sort use O(n) space!
it's important to **not** create the auxiliary array in the re in the recursive routine because that could lead to extensive cost of extra array creation!

```java
private static void merge(Comparable[] a, Comparable[] aux, int lo, int mid, int hi){
    assert isSorted(a, lo, mid); // precondition: a[lo..mid] sorted
    assert isSorted(a, mid+1, hi); // precondition: a[mid+1..hi] sorted
    for (int k = lo; k <= hi; k++)
        aux[k] = a[k];
    int i = lo, j = mid+1;
    for (int k = lo; k <= hi; k++){
        if (i > mid) a[k] = aux[j++];
        else if (j > hi) a[k] = aux[i++];
        else if (less(aux[j], aux[i])) a[k] = aux[j++];
        else a[k] = aux[i++];
    }
    assert isSorted(a, lo, hi); // postcondition: a[lo..hi] sorted
}
public static void sort(int[] a){
    aux = new int[a.length];
    sort(a, aux, 0, a.length-1);
}
private static void sort(Comparable[] a, Comparable[] aux, int lo, int hi){
    if (hi <= lo) return;
    int mid = lo + (hi - lo) / 2;
    sort(a, aux, lo, mid);
    sort(a, aux, mid+1, hi);
    merge(a, aux, lo, mid, hi);
}
```

#### bottom-up merge sort
> Any compare-based sorting algorithm must use at least $lg ( N ! ) \approx N lg N$ compares in the worst-case.

### Quick Sort
details
- Partitioning in-place. Using an extra array makes partitioning easier(and stable), but is not worth the cost.
- Terminating the loop. Testing whether the pointers cross is a bit trickier than it might seem.
- Staying in bounds. The (j == lo) test is redundant (why?), but the (i == hi) test is not.
- Preserving randomness. Shuffling is needed for performance guarantee.
- Equal keys. When duplicates are present, it is (counter-intuitively) better   to stop on keys equal to the partitioning item's key


```java
private static int partition(int[] a, int lo, int hi){
    int i = lo, j = hi+1;
    while(true){
        while(a[++i] < a[lo])
            if (i == hi) break;
        while(a[lo] < a[--j])
            if (j == lo) break;
        if(i >= j) break;
        exchange(a, i, j);
    }
    exchange(a, lo, j);
    return j;
}


sort(int[]a, int lo, int hi){
    int j = partition(a, lo, hi);
    sort(a, lo, j-1);
    sort(a, j+1, hi);
    
}

```

#### find the kth largest
use quick sort to achieve $O(N)$ (it's rather slow)
```java
public static Comparable select(Comparable[] a, int k)
{
    StdRandom.shuffle(a);
    int lo = 0, hi = a.length - 1;
    while (hi > lo){
        int j = partition(a, lo, hi);
        if (j < k) lo = j + 1;
        else if (j > k) hi = j - 1;
        else return a[k];
    }
    return a[k];
}
```

#### duplicate keys
Dijkstra 3-way partitioning demo
- Let v be partitioning item a[lo].
- Scan i from left to right.
    - (a[i] < v): exchange a[lt] with a[i]; increment both lt and i
    - (a[i] > v): exchange a[gt] with a[i]; decrement gt
    - (a[i] == v): increment i

#### sorting summary

|  | inplace? | stable? | worst | average| best|remarks|
| ------ | ------ | ------ | ---|------ |---|---|
| selection | √ |  |$N^2$/2|$N^2$/2|$N^2$/2|N exchanges|
| insertion | √ | √ | $N^2$/2|$N^2$/4 |N |use for small N or partially ordered|
| shell |  | ------ | ---|------ |-- |--|
| merge | | √ |$NlgN$|$NlgN$ |$NlgN$ |N log N guarantee, stable|
| quick | √ |  | $N^2$/2|2NlgN |N |N log N probabilistic guarantee fastest in practice|
| 3-way quick | √ |  |$N^2$/2 |2NlgN |N |improves quicksort in presence of duplicate keys|
| heap | √ |  |2NlgN |2NlgN |NlgN |NlgN guarantee, in-place|

### Priority Queues
#### applications
- Event-driven simulation. [customers in a line, colliding particles]
- Numerical computation. [reducing roundoff error]
- Data compression. [Huffman codes]
- Graph searching. [Dijkstra's algorithm, Prim's algorithm]
- Number theory. [sum of powers]
- Artificial intelligence. [A* search]
- Statistics. [maintain largest M values in a sequence]
- Operating systems. [load balancing, interrupt handling]
- Discrete optimization. [bin packing, scheduling]
- Spam filtering. [Bayesian spam filter]

#### binary heap
Array representation.
- Indices start at 1.
- Take nodes in level order.
- No explicit links needed!
- Parent of node at k is at k/2.
- Children of node at k are at 2k and 2k+1

Cheat sheet.
```java
private void swim(int k)
{
    while (k > 1 && less(k/2, k)){
        exch(k, k/2);
        k = k/2;
    }
}

private void sink(int k)
{
    while (2*k <= N){
        int j = 2*k;
        if (j < N && less(j, j+1)) j++;
        if (!less(k, j)) break;
        exch(k, j);
        k = j;
    }
}

public Key delMax()
{
    Key max = pq[1];
    exch(1, N--);
    sink(1);
     pq[N+1] = null;
    return max;
} 
```

#### heap sort

```java
void sort(int[] a){
    int N = a.length;
    for(int k=N/2;k>=1;k--){
        sink(a, k, N);
    }
    while(N>1){
        exch(a, 1, N);
        sink(a, 1, --N);
    }
}
```

Significance. In-place sorting algorithm with N log N worst-case.
- Mergesort: no, linear extra space.
- Quicksort: no, quadratic time in worst case.
- Heapsort: yes!
Bottom line. Heapsort is optimal for both time and space, but:
- Inner loop longer than quicksort’s.
- Makes poor use of cache memory.
- Not stable.

### Symbol Tables
#### binary search tree
it is a kind of table, so it does not have duplicate keys?

**delete minimum node**
```java
public void deleteMin()
 { root = deleteMin(root); }
 private Node deleteMin(Node x)
 {
     if (x.left == null) return x.right;
     x.left = deleteMin(x.left);
     x.count = 1 + size(x.left) + size(x.right);
     return x;
 }
```

**delete 2-child node** -- find successor
如果下一个节点都从右子数中找，树的整体就会越来越偏左，左部分密集
```java
private Node delete(Node x, Key key) {
     if (x == null) return null;
     int cmp = key.compareTo(x.key);
     if (cmp < 0) x.left = delete(x.left, key);
     else if (cmp > 0) x.right = delete(x.right, key);
     else {
         if (x.right == null) return x.left;
         if (x.left == null) return x.right;
         Node t = x;
         x = min(t.right);
         x.right = deleteMin(t.right);
         x.left = t.left;
     }
     x.count = size(x.left) + size(x.right) + 1;
     return x;
 } 
```

### balanced search tree
2-3 tree, red-black tree, B-tree

#### 2-3 tree
Allow 1 or 2 keys per node.
- 2-node: one key, two children.
- 3-node: two keys, three children.
Symmetric order. Inorder traversal yields keys in ascending order.

- search
- insert into a 2-node at bottom
    - search for key, as usual
    - replace 2-node with 3-node
- insert into a 3-node at bottom
    - add new key to 3-node to create temp 4-node
    - move middle key in 4-node into parent(recursively)(parent will change into a x-node)
    - If you reach the root and it's a 4-node, split it into three 2-nodes.
- construction
    - insert into bottom continuesly
    
**could do it, but there's a better way.**

#### red-black bst
##### Left-leaning red-black BSTs
- No node has two red links connected to it.
- Every path from root to null link has the same number of black links.
- Red links lean left

how to create?
1. Represent 2–3 tree as a BST.
2. Use "internal" left-leaning links as "glue" for 3–nodes

```java
// 中间节点换目标节点，不符合bst的左右换一下
private Node rotateLeft(Node h)
 {
     assert isRed(h.right);
     Node x = h.right;
     h.right = x.left;
     x.left = h;
     x.color = h.color;
     h.color = RED;
     return x;
 }

 private Node rotateRight(Node h)
 {
     assert isRed(h.left);
     Node x = h.left;
     h.left = x.right;
     x.right = h;
     x.color = h.color;
     h.color = RED;
     return x;
 }
 
 flippColor
```

##### B trees
### Geometric Applications of BSTs

#### 1d range search/count
how many numbers between k1 and k2

#### line segments intersection
思路：
{% asset_img 1.png %}

#### 2d range search

{% asset_img 2.png %}

#### interval search trees
{% asset_img 3.png %}
#### rectangle intersection
{% asset_img 4.png %}

#### summary
{% asset_img 5.png %}