# SICP Lecture 6B
# Streams (Part 2)

*Harold Abelson*

The key idea behind streams is to offer a mean of decoupling the apparant order of events in our program to the actual order of events in the computer.  We can deal with very long streams and generate elements on demand.

    (define (n-th-stream n s)
      (if (= n 0)
        (head s)
        (n-th-stream (-1+ n) (tail s))))

    (define (print-stream s)
      (cond ((empty-stream? s) "done")
            (else (print (head s))
                  (print-stream (tails s)))))

## Infinite Streams

    (define (integers-from n)
      (cons-stream n (integers-from (+1 n))))

    (define integers (integers-from 1))

    (define (sieve s)
      (cons-stream
        (head s)
        (sieve
            (filter
              (lambda (x)
                (not
                    (divisible? x (head s))))
              (tail s)))))

    (define primes
      (sieves (integers-from 2)))

    (n-th stream 20 primes)
    => 73

    (print-stream primes)
    => 2 3 5 7 11 ...

The sieve is implemented as an **infinitely recursive filter**.

## Recursively Defined Streams

    (define (add-streams s1 s2)
      (cond ((empty-stream? s1) s2)
            ((empty-stream? s2) s1)
            (else
              (cons-stream
                (+ (head s1) (head s2))
                (add-streams (tail s1) (tail s2))))))

    (define (scale-stream c s)
      (map-stream (lambda (x) (* x c)) s))

    ;; An infinite stream of one
    (define ones (cons-stream 1 ones))

    (define integers
      (cons-stream 1 (add-streams integers ones)))

The recursive stream definitions work because of the `delay`.

    (define (integral s initial-value dt)
      (define int
        (cons-stream
          initial-value
          (add-streams (scale-stream dt s) int)))
      int)

    (define fibs
      (cons-stream 0
        (cons-stream 1
          (add-stream fibs (tail fibs)))))

Instead of recursive procedures, we have recursively defined data objets.
There is no difference between procedures and data.

## Problems Streams Are Not Going To Solve

Solving differential equations:

    y'   = y2
    y(0) = 1
    dt   = 0.001

                1
                |
                v
     y'  +------------+  y
    --+->| integrator |---+-->
      |  +------------+   |
      |                   |
      |  +------------+   |
      +--| map square |<--+
         +------------+

    (define y
      (integeral dy 1 .001))

    (define dy
        (map square y))

**Problem!**  Catch 22 in the `y` and `dy` definitions.

    (define (integral delayed-s initial-value dt)
      (define int
        (cons-stream
          initial-value
          (let ((s (force delayed-s)))
            (add-streams (scale-stream dt s) int))))
      int)

    (define y
      (integeral (delay dy) 1 .001))

    (define dy
        (map square y))

It's getting messy, what if all procedures behaved as they were delayed?

Applicative-order evaluation *Vs* normal-order evaluation

Decoupling time and local state do not mix.

    (define x 0)

    (define (id n)
      (set! x n)
      n)

    (define (inc a) (1+ a))

    (define y (inc (id 3)))

    x => 0
    y => 4
    x => 3

Purely Functional Programming => no-side effects
No synchronization problems at the price of giving up assignment

## Limitation of Streams

    (define (make-deposit-account balance deposit-stream)
      (cons-stream
        balance
        (make-deposit-account)
          (+ balance (head deposit-stream))
          (tail deposit-stream)))

What if this is a joint bank account with 2 sources of deposits:

    Bill -----\
               \    +--------------+    +--------------+
                >---| "fair merge" |--->| bank account |
               /    +--------------+    +--------------+
    Dave -----/

We have a problem, we need time: the "fair merge" can't be implemented as a pure function.