# SICP Lecture 1B
# Substitution Model

*Gerald Jay Sussman*

    (define (sos x y)
      (+ (sq x) (sq y)))
    (define (sq x)
      (* x x))

kinds of expressions:

* numbers
* symbols
* lambda expressions
* definitions
* conditionals
* combinations

Substitution rule:

1. evaluate the operator to get procedure
2. evaluation the operands to get arguments
3. apply the procedure to the arguments
  * copy the body of the procedure, substituting the argument supplied (reduction)
  * evaluate the new body

> "The key to understanding complicated things is to know where not to look at, what not to compute, what not to think"

(if <predicate> <consequent> <alternative>)

> "one of the thing every sorcerer will tell you: if you have the name of the spirit you have the power over it"

iteration:

    (define (+ x y)
      (if (= x 0)
        y
        (+ (-1+ x) (1+ y))))

-> time = O(x), space = O(1)

linear recursion:

    (define (+ x y)
      (if (= x 0)
        y
        (1+ (+ (-1+ x) y))))

-> time = O(x), space = O(x)


    (define (fib n)
      (if (< n 2)
        n
        (+ fib((-n 1)) fib((-n 2)))))

-> time = O(fib(n)), space = O(n)

    (define (move n from to spare)
      (cond ((n = 0) "done")
        (else
            (move (-1+ n) from spare to)
            (print-move from to)
            (move (-1+ n) spare to from))))