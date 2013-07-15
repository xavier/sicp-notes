# SICP Lecture 1A
# Overview and Introduction ot Lisp

*Harold Abelson*

Techniques for controlling complexity

* black box abstractions
* conventional interfaces
* metalinguistic abstractions

LISP

* primitive elements
* means of abstraction
* means of combination

means of combination

combination = (operator operand1 operand2 ... operandn)
operands can be combinations
prefix and fully parenthesized notation: cannot leave or put extra parentheses
combination is a tree
parentheses = way to represent the tree as a string

means of abstraction

    (define A (* 5 5))
    (define B (+ A (* 5 A)))
    (define (square x)(* x x)) <=> (define square (lambda (x) (* x x))
    (square 10)
    (define (average x y) (/ (+ x y) 2))
    (define (mean-square x y) (average (square x) (square y)))

No difference between built-ins and compound

    (define (abs x)
      (cond
          ((< x 0) (- x))
          ((= x 0) 0)
          ((> x 0) x)))

    (define (abs x)
      (if (< x 0)
        (-x)
        x))

Approximating sqrt:

    (define (try guess x)
      (if (good-enough? guess x)
        guess
        (try (improve guess x) x)))
    (define (improve guess x) (average guess (/ x guess)))
    (define (good-enough? guess x)
      (< (abs (- (square guess) x)) .001))
    (define (sqrt x) (try 1 x))

Block structure to package internals inside a definition:

    (define (sqrt x)
      (define (good-enough? guess) ...)
      (define (improve guess) ...)
      (define (try guess) ...)
      (try 1))


    (define a (* 5 5))     => evals to 25
    (define (b) (* 5 5))   => evals to compound procedure
