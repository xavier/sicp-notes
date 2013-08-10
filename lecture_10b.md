# SICP Lecture 10B
# Storage Allocation & Garbage Collection

*Gerald Jay Sussman*

Memory, the glue that data structures are made of.

Gödel came up with a scheme to encode expressions as numbers.

If objects ar represented by numbers then:

* `(cons x y)` could be represented as `2^x 3^y`
* `(car x)` could be represented as the number of factors of 2 in `x`
* `(cdr x)` could be represented as the number of factors of 3 in `x`

This is not very practical.

## Data Structure Representation

Our data structures are hierarchical but **memory is linear**.

                   +-+-+                           +-+-+           +-+-+
    ((1 2) 3 4) -> |.|.|-------------------------->|.|.|---------->|.|/|
                   +-+-+                           +-+-+           +-+-+
                    | 1 (*)                         | 2             | 4
                    V                               V               V
                   +-+-+           +-+-+           +-+             +-+
                   |.|.|---------->|.|/|           |3|             |4|
                   +-+-+           +-+-+           +-+             +-+
                    | 5             | 7
                    V               V
                   +-+             +-+
                   |1|             |2|
                   +-+             +-+

                   (*) indicates the index in memory

We divide the memory into two arrays: `the-cars` and `the-cdrs`.  These arrats store **typed objects**, for instance `pX` is a pair, `nX` is a number, `eX` is an empty list, ...

    +----------+----+----+----+----+----+----+----+----+----+----
    | Index    |  0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 |  8 | ...
    |----------+----+----+----+----+----+----+----+----+----+----
    | the-cars |    | p5 | n3 |    | n4 | n1 |    | n2 |    |
    |----------+----+----+----+----+----+----+----+----+----+----
    | the-cdrs |    | p2 | e0 |    | e0 | p7 |    | e0 |    |
    +----------+----+----+----+----+----+----+----+----+----+----

Machine code instructions like `assign` and `fetch` actually access these arrays.

    ;; Accessors
    (vector-ref  vector index)
    (vector-set! vector index value)

    (assign a (car (fetch b)))
    ====>
    (assign a (vector-ref (fetch the-cars) (fetch b)))

    (assign a (cdr (fetch b)))
    ====>
    (assign a (vector-ref (fetch the-cdrs) (fetch b)))

    (perform (set-car! (fetch a) (fetch b)))
    ====>
    (perform (vector-set! (fetch the-cars) (fetch a) (fetch b)))

    (perform (set-cdr! (fetch a) (fetch b)))
    ====>
    (perform (vector-set! (fetch the-cdrs) (fetch a) (fetch b)))

## Allocation

Freelist allocation scheme: all free cells are linked to the next free cell.

    free -> f6 -> f8 -> f3 -> f0 -> f9 -> ...

    +----------+----+----+----+----+----+----+----+----+----+----
    | Index    |  0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 |  8 | ...
    |----------+----+----+----+----+----+----+----+----+----+----
    | the-cars | e0 | p5 | n3 | e0 | n4 | n1 | e0 | n2 | e0 |
    |----------+----+----+----+----+----+----+----+----+----+----
    | the-cdrs | f9 | p2 | e0 | f0 | e0 | p7 | f8 | e0 | f3 |
    +----------+----+----+----+----+----+----+----+----+----+----

Implementing `cons` using the freelist method of allocation:

    (assign a (cons (fetch b) (fetch c)))
    ====>
    (assign a (fetch free))                                       ;; head of freelist
    (assign free (vector-ref (fetch the-cdrs) (fetch free)))      ;; freelist = (cdr freelist)
    (perform (vector-set! (fetch (the-cars) (fetch a) (fetch b))) ;; store pair values
    (perform (vector-set! (fetch (the-cdrs) (fetch a) (fetch c)))

Missing from the code above: assigning the type of value before storing the value in the arrays.

## Garbage Collection

The interpretation of programs produces garbage (e.g. short lived frames during procedure evaluation), users produce garbage too:

    ;; reverse a list
    (define (rev-loop x y)
      (if (null? x)
        y
        (rev-loop (cdr x)
                  (cons (car x) y))))

    ;; reverse list onto the empty list and reverse that to the second list
    (define (append u v)
      (rev-loop (rev-loop u '()) v))

The intermediate result (the reversal of the first list) is never going to be used again.  There must be some way to reclaim that garbage.

How to prove that a given data object will never be used again and that it can be discarded without affecting any other computations?

The registers contain pointers to objects in the Lisp Structured Memory.  If any object cannot be reached by walking through the data structure referenced by any of the registers, its memory can be reclaimed.

### Mark & Sweep Strategy

For each of the machine registers, we recursively crawl the data structure they reference and **mark** the memory cells as we access them.  Once this process is completed, all the unmarked cells can be recycled.

Example:

                      root (*)
                        |
                        V
    +-----------+----+----+----+----+----+----+----+----+----
    | Index     |  0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 | ...
    |-----------+----+----+----+----+----+----+----+----+----
    | the-cars  | p3 | p5 | n3 | p0 | p7 | n1 | n4 | n2 |
    |-----------+----+----+----+----+----+----+----+----+----
    | the-cdrs  | p2 | p2 | p4 | p6 | e0 | p7 | n2 | e0 |
    |-----------+----+----+----+----+----+----+----+----+----
    | the-marks |  0 |  0 |  0 |  0 |  0 |  0 |  0 |  0 |
    +-----------+----+----+----+----+----+----+----+----+----

    (*) arbitrarily chosen, could be multiple (all registers)

We walk the structure, depth-first:

      1 -> 5 -> 7
        -> 2 -> 4 (-> 7)

After the marking process:

    +-----------+----+----+----+----+----+----+----+----+----
    | Index     |  0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 | ...
    |-----------+----+----+----+----+----+----+----+----+----
    | the-cars  | p3 | p5 | n3 | p0 | p7 | n1 | n4 | n2 |
    |-----------+----+----+----+----+----+----+----+----+----
    | the-cdrs  | p2 | p2 | p4 | p6 | e0 | p7 | n2 | e0 |
    |-----------+----+----+----+----+----+----+----+----+----
    | the-marks |  0 |  1 |  1 |  0 |  1 |  1 |  0 |  1 |
    +-----------+----+----+----+----+----+----+----+----+----

We scan the memory and add the recycled cell to the free list.

    gc   (assign thing (fetch root))
         (assign continue sweep)

    ;; Marking Process
    mark (branch (not-pair? ((fetch thing) done)))

    pair (assign mark-flag (vector-ref (fetch the-marks) (fetch thing)))
         (branch (= (fetch mark-flag) 1) done)
         (perform (vector-set! (fetch the-marks) (fetch thing) 1))

    ;; Mark car of things
    mcar (push thing)
         (push continue)
         (assign continue mcdr)
         (assign thing (vector-ref (fetch the-cars) (fetch thing)))
         (goto mark)

    ;; Mark cdr of things
    mcdr (pop continue)
         (pop thing)
         (assign thing (vector-ref (fetch the-cdrs) (fetch thing)))
         (goto mark)

    done (goto (fetch continue))

    ;; Sweeping Process
    sweep
        (assign free '())
        (assign scan (-1+ (fetch memtop)))    ;; Start from the top of the memory
    scan-loop
        (branch (negative? (fetch scan)) end)
        (assign mark-flag (vector-ref (fetch the-marks) (fetch scan)))
        (branch (= (fetch mark-flag) 1) unmark)
        (perform (vector-set! (fetch the-cdrs) (fetch scan) (fetch free)))
        (assign free (fetch scan))
        (assign scan (-1+ (fetch scan)))
        (goto scan-loop)
    unmark
        (perform (vector-set! (fetch the-marks) (fetch scan) 0))
        (assign scan (-1+ (fetch scan)))
        (goto scan-loop)
    end

There are serious desadvantages to the Mark & Sweep algorithm.
As the memory gets larger, the scanning process becomes more expensive.  We need to be more selective.

### Minsky-Fenichel-Yochelson Algorithm

Pre-requisite: we have about twice as much address space as we're using.

We start with a mixture of useful data and garbage and we will gradully copy the good stuff into another space.

    from-space
    +------------------------------------------------------------------------+
    |         |         |   |     |                |   |          |          |
    +------------------------------------------------------------------------+
                          ^
                          root
    to-space
    +------------------------------------------------------------------------+
    | | |  | |  | |  |                                                       |
    +------------------------------------------------------------------------+
     ^       ^
     root    free

1. start from root in from-space
2. copy into to-space
3. mark as broken-heart in from-space
4. scan copied data
5. set the root to the scanned data and repeat the process, skipping the broken-hearts

Improvement: interleave the GC process between `cons`.

## Not Everything Can Be Computed

Let's imagine a mathematical function `S` which takes a procedure and its arguments as variables:

    S[P, a] = true  if (P a) will converge to a value without an error
              false if (P a) loops forever or makes an error

### Proof by Contradiction

Suppose we have procedure `safe?` that computes the value of `S`.

    (define diag1
      (lambda (p)
        (if (safe? p p)
          (inf)
          3)))

    (define inf
      (lambda ()
        ((lambda (x) (x x))
          (lambda (x (x x))))))

    (diag1 diag1) => ?

if it's safe ot run `diag1` then it will go into an infinite loop which makes it, by definition, unsafe.

Note: the `diag` name is a nod to [Cantor's diagonal argument](https://en.wikipedia.org/wiki/Cantor's_diagonal_argument).

There's another way to prove it:

    (define diag2
      (lambda (p)
        (if (safe? p p)
          (other-than (p p))
          false)))

    ;; Always produces something else than its argument is
    (define other-than
      lambda (x)
        (if (= x 'A) 'B 'A))

    (diag2 diag2) => (other-than (diag2 diag2)) ?!?

We have proven the [Halting Theorem](https://en.wikipedia.org/wiki/Halting_problem).

`HALT`

---