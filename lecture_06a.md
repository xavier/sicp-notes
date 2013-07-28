# SICP Lecture 6A
# Streams, part 1

*Harold Abelson*

The **consequences of introducing assignment** and state are frightening.  The basic substitution model breaks down, we have to rely on this complex environment-based model.  A variable is not something that stands for a value, it stands for a place holding a value which can change.  Suddenly we have to **think not only about values but about time**.
**Pairs** are not defined by their two values anymore but their **identity**, they are **objects**.
We have to worry about **sharing**.

We ended up with this mess because we were trying to build a **modular system**, decomposed into **natural pieces** (cfr digital circuit).

**Stream processing** is another way of building modular system.  We want our programs to **worry less about time**.

                /\
               /  \
              /    \
             /\    / \
            1  \  13  \
               /\    / \
              2  7  12 14

    (define (sum-odd-squares tree)
      (if (leaf-node? tree)
        (if (odd? tree)
          (square tree)
          0)
        (+ (sum-dd-squares (left-branch tree))
           (sum-dd-squares (right-branch tree)))))


    (define (odd-fibs n)
      (define (next k)
        (if (> k n)
          '()
          (let ((f (fib k)))
            (if (odd? f)
              (cons f (next (1+ k)))
              (next (1+ k))))))
      (next 1))

How can we turn these two programs into streams?

      +------------------+   +-------------+   +------------+   +------------------------------+
      | enumerate leaves |-->| filter odd? |-->| map square |-->| accumulate + starting from 0 |
      +------------------+   +-------------+   +------------+   +------------------------------+

      +---------------+   +---------+   +-------------+   +----------------------------------+
      | enum interval |-->| map fib |-->| filter odd? |-->| accumulate cons starting from () |
      +---------------+   +---------+   +-------------+   +----------------------------------+

Those two programs now look pretty similar.  That commonality was totally obscured in the procedures we initially wrote: the responsabilities of the enumerator and accumulator are spread across the code.

> “In order to control something, you need a name for it.”

We need to build a language to describe streams.

## Streams

    (cons-stream x y)
    (head s)
    (tail s)
    (empty-stream? s)
    the-empty-stream

With the following contract:

    ∀ x,y
      (head (cons-stream x y)) -> x
      (tail (cons-stream x y)) -> y

Those are exactly the axioms for `cons`, `car` and `cdr`.

Let's build the component of the system:

    (define (map-stream proc s)
      (if (empty-stream? s)
        the-empty-stream
        (cons-stream (proc (head s))
                     (map-stream proc (tails s)))))

    (define (filter pred s)
      (cond
        ((empty-stream? s) the-empty-stream)
        ((pred (head s))
          (cons-stream (head s)
                       (filter pred (tail s)))
          (else (filter pred (tail s))))))

    (define (accumulate combiner initial-val s)
      (if (empty-stream? s)
        init-val
        (combiner (head s)
                  (accumulate combiner init-val (tail s)))))

    (define (enumerate-tree tree)
      (if (leaf-node? tree)
        (cons-stream tree the-empty-stream)
        (append-streams
          (enumerate-tree (left-branch tree))
          (enumerate-tree (right-branch tree)))))

    (define (append-streams s1 s2)
      (if (empty-stream? s1)
        s2
        (cons-stream (head s1)
                      (append-streams (tail s1) s2))))

    (define (enum-interval low high)
      (if (> low high)
        the-empty-stream
        (cons-stream
          low
          (enum-intervla (1+ low) high))))

We can now rewrite our initial procedures using these stream processing components:

    (define (sum-odd-squares tree)
      (accumulate
        +
        0
        (map
          square
          (filter odd
                  (enumerate-tree tree)))))

    (define (odd-fibs n)
      (accumulate
        cons
        '()
        (filter
            odd
            (map fib (enum-interval 1 n)))))

We have defined a set of new conventional interfaces and new pieces which we can mix and match.

## Advanced Examples

### Nested Streams

We have a stream which contains streams:

    {{1, 2, 3, ...}, {10, 20 30, ...}, ... }

We can use our new primitives to flatten this stream of streams:

    (define (flatten st-of-st)
      (accumulate
        append-streams
        the-empty-stream
        st-of-st))

    (define (flatmap f s)
      (flatten (map f s)))

### Integer Pairs with Prime Sum

Given an integer *n*, find all pairs of integer *0 < j < i <= n* such as *i+j* is prime.

    n = 6
    i    j    i+j
    -------------
    2    1    3
    3    2    5
    4    1    5

We want ot obtain a stream of triplets `(i, j, i+j)` satisfying the requiring the primality requirement.

    ;; map -> filter -> flatmap
    (define (prime-sum-pairs n)
      (map
        (lambda (p)
          (list (car p
                (cadr p)
                (+ (car p) (cadr p))))
        (filter
          (lambda (p)
            (prime? (+ (car p) (cadr p))))
          (flatmap
            (lambda (i)
              (map
                (lambda (j) (list i j))
                (enum-interval 1 (-1+ i))))
            (enum-interval 1 n))))))

**Nested loops** are now compositions of `flatmaps`.

We can add some syntactic sugar: `collect` an abbreviation for nested `filter` and `flatmap`.

    (define (prime-sum-pairs n)
      (collect
        (list i j (+i j))                    ;; the result
        ((i (enum-interval 1 n))             ;; generated for i in the given interval
         (j (enum-interval 1 (-1+ i))))      ;;       and for j in the given interval
        (prime? (+ i j))))                   ;; satisfying the given predicate

### 8 Queens Problem

Find a way to put 8 queens on a chess board so that they have no way of attacking each others: they can't be on the same row, the same column or the same diagonal.

    (safe? row col rest-of-positions) -> is it safe to put another Queen in the given position
    (adjoin-position row col rest-queens)

This is a typical case of **backtracking search** but this process implies time and state.  We instead assume that we already know all the positions and filter them.

    (define (queens board-size)
      (define (fill-cols k)
        (if (= k 0)
            (singleton empty-board)
            (collect
              (adjoin-position try-row k rest-queens)
              ((rest-queens (fill-cols (-1+ k)))
               (try-row (enum-interval 1 size)))
              (safe? try-row k rest-queens))))
      (fill-cols board-size))

This is simple but terribly inefficient as it brute forces its way through all possible combinations.

## Efficient Stream Programs

Problem: find the second prime between 10,000 and 1,000,000:

    (head
      (tail
        (filter
            prime?
            (enum-interval 10000 1000000))))

The power of this programming style is its weakness: we are mixing up the enumerating, the testing and the accumulating.

### Streams Are Not Lists

We want streams to be a **data structure which computes itself incrementally**, an on-demand data structure.

    (define (cons-stream x y)
      (cons x (delay y)))
    (define (head s) (car s))
    (define (tail s) (force (cdr s)))

* `delay` takes an expression and produce a promise to compute that expression when you ask for it.
* `force` calls in a promise made by `delay`

For instance the output of `(enum-interval 10000 1000000)` is a pair `(1000, <promise to compute the rest of the integers from 10001 1000000>)`

The enumarator will never generate more items than required by the computation.

There is no magic:

    (define (delay exp)
      (lambda () exp)

    (define (force promise) (promise))

**We have decoupled the apparent order of events in our program from the actual order of events in the computer.**

But what if a recursive program uses a stream like this:

    (tail (tail (tail ...)))

A lot of time will be spent recalculating the tail on each call.  We can refine our implementation:

      (define (delay exp)
        (memo-proc (lambda () exp))

The `memo-proc` procedure **memoizes** the result of the procedure given in parameter.

      (define (memo-proc proc)
        (let ((already-run? nil) (result nil))
          (lambda ()
            (if (not already-run?)
              (sequence
                (set! result (proc))
                (set! already-run? (not nil))
                result)
              result))))

> Any amount of side-effect will mess up everything
