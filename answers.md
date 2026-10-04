# CMPS 6610 Problem Set 03
## Answers

**Name:**_________________________


Place all written answers from `problemset-03.md` here for easier grading.




- **1b.**

  isearch calls iterate with f(found, y) = found or y == x.
  Each call of f costs O(1).

  Work:
  iterate looks at each element one time.
  W(n) = W(n-1) + 1 E O(n)

  Span:
  Each step needs the result of the step before, so nothing runs in parallel.
  S(n) = S(n-1) + 1 E O(n)


- **1d.**

  rsearch first does map(y == x) on the list and then reduce with "or".

  map:
  Each comparison costs O(1) and all of them can run in parallel.
  Work = O(n), Span = O(1)

  reduce:
  The two recursive calls of size n/2 run in parallel, and the "or" costs O(1).
  W(n) = 2W(n/2) + 1 E O(n)
  S(n) = S(n/2) + 1 E O(logn)

  Total:
  W(n) = O(n) + O(n) = O(n)
  S(n) = O(1) + O(logn) = O(logn)


- **1e.**

  ureduce splits the list in 1/3 and 2/3 instead of two halves.
  The output is the same, because "or" is associative.

  In main.py, ureduce splits only one time and then calls the normal reduce
  on the two parts. Using the costs of reduce from 1d (W_r and S_r):

  W(n) = W_r(n/3) + W_r(2n/3) + 1 E O(n)
  S(n) = S_r(2n/3) + 1 E O(logn)
  The two calls run in parallel, so the span only waits for the larger part, 2n/3.

  If ureduce called itself at every level:

  Work:
  W(n) = W(n/3) + W(2n/3) + 1
  C(root) = 1
  C(level 1) = 2
  The cost gets larger, so it is leaf dominated.
  The tree has n leaves, one per element.
  W(n) E O(n)

  Span:
  S(n) = S(2n/3) + 1
  Every level costs 1, so it is balanced.
  The tree has log_{3/2}(n) levels.
  S(n) E O(logn)

  With the map from 1d, the total is W(n) = O(n) and S(n) = O(logn)
  in both cases, the same as in 1d.





- **2a.**

  ```
  dedup A =
      let
          addFirst (D, i) =
              if A[i] ∈ D then
                  D
              else
                  D ∪ {A[i] ↦ i}

          first = iterate addFirst {} ⟨0, ..., |A|-1⟩
      in
          ⟨ A[i] : 0 ≤ i < |A| | first[A[i]] = i ⟩
      end
  ```

  e.g. dedup ⟨3, 1, 3, 2, 1⟩ ⇒ ⟨3, 1, 2⟩

  First, iterate goes through the indices of A from left to right and builds
  a hash table first, where first[x] is the index of the first x in A.
  If A[i] is already in D, it is a duplicate, so D does not change.
  D ∪ {A[i] ↦ i} is an insertion in the hash table, so it costs O(1) (expected).
  Each version of D is used only by the next step of iterate, so D can be a normal
  (mutable) hash table and we do not need to copy it.

  Then, filter keeps A[i] only if i is the first index of A[i].
  filter keeps the order, so the output has the distinct elements in the same order as A.

  Work:
  iterate: each step costs O(1) (expected).
  W(n) = W(n-1) + 1 E O(n)
  filter: each test first[A[i]] = i costs O(1), so W = O(n).
  Total: W(n) = O(n) + O(n) = O(n) (expected)

  Span:
  iterate: each step needs D from the step before, so nothing runs in parallel.
  S(n) = S(n-1) + 1 E O(n)
  filter: S = O(logn).
  Total: S(n) = O(n) + O(logn) = O(n) (expected)





- **2b.**

  ```
  multiDedup A =
      let
          B = flatten A
          G = collect compare ⟨ (x, 1) : x ∈ B ⟩
      in
          ⟨ x : (x, c) ∈ G ⟩
      end
  ```

  e.g. multiDedup ⟨⟨3, 1, 3⟩, ⟨2, 1, 5⟩⟩ ⇒ ⟨1, 2, 3, 5⟩

  This is the Map-Reduce idea from the slides (like the word count example).
  flatten joins the lists into one sequence B with mn elements.
  map changes each x into the pair (x, 1).
  collect puts the pairs with the same key x in the same group (compare is the
  normal comparison of two elements and costs O(1)):
  ⟨(1, ⟨1, 1⟩), (2, ⟨1⟩), (3, ⟨1, 1⟩), (5, ⟨1⟩)⟩
  Each distinct element is a key one time in G, so we just take the keys.
  collect sorts the keys, so the output is not in the order of the input,
  but here the order does not matter.

  Work:
  flatten: O(m + mn) = O(mn)
  map (pairs and keys): O(mn)
  collect: O(mn.log(mn))
  Total: W = O(mn.log(mn))

  Span:
  flatten: O(logm)
  map (pairs and keys): O(1)
  collect: O(log^2(mn))
  Total: S = O(log^2(mn))

  Comparing with 2a:
  If we flatten the lists and use dedup from 2a on the mn elements,
  W = O(mn) and S = O(mn), because iterate is sequential.
  multiDedup does a little more work (a log(mn) factor, because collect sorts),
  but the span is much smaller: O(log^2(mn)) instead of O(mn).
  The log factor is expected: using only comparisons (no hash table), finding the
  distinct elements needs Omega(mn.log(mn)) work (element distinctness problem).
  In a network with many machines a small span is more important, so this is better.
  We can do this because the order does not matter here. In 2a we need the order,
  so we cannot just group the elements.


- **2c.**

  Yes, some of them are useful.

  filter (2a):
  After we build the hash table first, each test first[A[i]] = i is independent,
  so filter does all of them in parallel and keeps the order.
  W = O(n), S = O(logn).

  map and tabulate (2a and 2b):
  They do the same O(1) step for each element in parallel, like making the
  pairs (x, 1) in 2b. W = O(n), S = O(1).

  flatten (2b):
  Joins the m lists into one sequence. W = O(mn), S = O(logm).

  collect (2b):
  This is the most useful one. It puts equal elements in the same group, so each
  distinct element appears one time as a key. It is the shuffle step of Map-Reduce,
  it only needs comparisons (no hash table) and it has S = O(log^2(mn)).
  It could also be used in 2a: collect the pairs (A[i], i), take the smallest index of
  each group and sort by this index to get the order back.
  This gives W = O(nlogn) and S = O(log^2 n): more work than 2a, but much less span.

  Less useful:

  iterate works for 2a, but it is sequential, so S = O(n).
  This is why the span of 2a is O(n).

  reduce with ∪ (set union) also gives the distinct elements, because ∪ is
  associative and {} is the identity. But it only helps if ∪ itself is parallel.
  With our operations, ∪ of two sets with n elements in total inserts them one by one,
  so it costs O(n) work and O(n) span:
  W(n) = 2W(n/2) + n, balanced, so W(n) E O(nlogn)
  S(n) = S(n/2) + n, root dominated, so S(n) E O(n)
  So reduce does not make the span smaller, and it also loses the order of 2a.

  scan with ∪ would give the set of elements before each i. But if all elements
  are distinct, these sets have sizes 1, 2, ..., n, so only writing the output
  costs 1 + 2 + ... + n, and the work is Theta(n^2).






- **3b.**





- **3d.**





- **3f.**




