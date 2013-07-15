# SICP Lecture 2B
# Compound Data

*Harold Abelson*

> "We divorce the part of building things from the task of implementing the parts"

abstraction barriers isolate the details at lower layers from the higher layers

### Rational Numbers

    n1/d1 + n2/d2 = (n1*d2 + n2*d1) / d1*d2
    n1/d1 * n2/d2 = (n1*n2) / (d1*d2)

    (define (+rat x y)
      (make-rat
        (+ (* (numer x) (denom y))
           (* (numer y) (denom x)))
        (* (denom x) (denom y))))

    (define (*rat x y)
      (make-rat
        (* (numer x) (numer y))
        (* (denom x) (denom y))))

    (define (make-rat n d) (cons n d))
    (define (numer rat) (car rat))
    (define (denom rat) (cdr rat))

Current implementation is correct but suboptimal as it does not return rationals reduced to their lowest terms:

    (define a (make-rat 1 2))
    (define b (make-rat 1 4))
    (define ans (+rat a b))
    (numer ans) -> 6
    (denom ans) -> 8

Solution:

    (define (make-rat n d)
      (let ((g (gcd n d)))
      (cons (/ n g) (/ d g))))

Use of `let` to create a local context:

    (let <local bindings> expr)

    (define a 5)
    (let ((a 10))
      (+ a a))   -> 20

> "The abstraction layer is made of the constructor and the selectors"

Data abstraction: we separate the way data objects are used from their representation

The data abstraction layer also provide naming, it presents the compound data as a conceptual entity

> "There's a very narrow line between deferring decisions and outright procrastination. If you'd like to make >progress but never be down by the consequences of your decisions, data abstraction is one way of doing this.
We use **wishful thinking**, we **gave a name to decision** and continuing as we had made the decision"

Question about "do all your design before any of your code"

> "Well... that's someone's axiom. And I bet that's the axiom of someone who hasn't implemented very large computer systems very much."

### Vectors and Segments

    ; Vector layer built on top of pairs of numbers
    (define (make-vector x y) (cons x y))
    (define (xcor v) (car v))
    (define (ycor v) (cdr v))

    ; Segments layer built on top of pairs of vectors
    (define (make-seg p q) (cons p q))
    (define (seg-start s) (car s))
    (define (seg-end s) (cdr s))

    (define (midpoint s)
      (let ((a (seg-start s))
            (b (seg-end s)))
        (make-vector
          (average (xcor a) (xcor b))
          (average (ycor a) (ycor b)))))

    (define (length s)
      (let ((dx (- (xcor (seg-end s)) (xcor (seg-start s))))
            (dy (- (ycor (seg-end s)) (ycor (seg-start s)))))
       (sqrt (square dx) (square dy))))

###  Implementation of Pairs

Pair elements are wrapped into a closure.

    (define (cons a b)
      (lambda (pick)
        (cond ((= pick 1) a)
               (= pick 2) b))))
    (define (car x) (x 1))
    (define (cdr x) (x 2))

It's all made of thin air: there's no data objects at all, it's all made of procedures.  This *representation* of `cons`, `car` and `cdr` satisfies the axiom which defines a pair.

We don't need data at all to build this kind of abstraction.  The line between data and procedures is blurry.

> "You really need to believe that procedures are objects."


