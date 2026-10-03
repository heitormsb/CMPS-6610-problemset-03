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

  Then, filter keeps A[i] only if i is the first index of A[i].
  filter keeps the order, so the output has the distinct elements in the same order as A.

  Work:
  iterate: each step costs O(1).
  W(n) = W(n-1) + 1 E O(n)
  filter: each test first[A[i]] = i costs O(1), so W = O(n).
  Total: W(n) = O(n) + O(n) = O(n)

  Span:
  iterate: each step needs D from the step before, so nothing runs in parallel.
  S(n) = S(n-1) + 1 E O(n)
  filter: S = O(logn).
  Total: S(n) = O(n) + O(logn) = O(n)





- **2b.**





- **2c.**






- **3b.**





- **3d.**





- **3f.**




