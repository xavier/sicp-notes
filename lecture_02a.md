# SICP Lecture 2A
# High-order Procedures

*Gerald Jay Sussman*

    (define (sum-int a b)
      (if (> a b)
        0
        (+ a (sum-int (1+ a) b))))

    (define (sum-sq a b)
      (if (> a b)
        0
        (+ (square a) (sum-sq (1+ a) b))))


    (define (sum term a next b)
      (if (> a b)
        0
        (+ (term a) (sum term (next a) next b))))

    (define (sum-int a b)
      (define (identity a) (a))
      (sum identity a 1+ b))

    (define (sum-sq a b)
      (sum square a 1+ b))

    (define (pi-sum a b)
      (sum
        (lambda (i) (/ 1 (* i (+ i 2))))
        a
        (lambda (i) (+ i 4))
        b))

Iterative implementation

    (define (sum term a next b)
      (define (iter j ans)
        (if (> j b)
          ans
          (iter (next j) (+ (term j) and))))
      (iter a 0))

Fixed point: f(x) = x

    (define (sqrt x)
      (fixed-point
        (lambda (y) (average (/ x y) y))
        1))

    (define (fixed-point f start)
      (define tolerance 0.000001)
      (define (close-enough? a b)
        (< (abs (- a b)) tolerance))
      (define (iter old new)
        (if (close-enough? old new)
          new
          (iter new (f new))))
      (iter start (f start))
    )

    (define (sqrt x)
      (fixed-point
        (average-damp (lambda (y) (/ x y))) 1))

    (define average-damp
      (lambda (f)
        (lambda (x)
          (average (f x) x))))

is equivalent to:

    (define (average-damp f)
      (lambda (x)
        (average (f x) x)))


Newton's method:
  - to find a y such that f(y) = 0
  - we start with a guess y0
  - we iterate yn+1 = yn + f(yn) / f'(yn)

    (define (sqrt x)
      (newton (lambda (y) (- x (square y))) 1))

    (define (newton f guess)
      (define df (derivative-of f))
      (fixed-point
        lambda (x) (- x (/ (f x) (df x)))
        guess))

    (define dx 0.00000001)

    (define derivative-of
      (lambda (f)
        (lambda (x)
          (/ (- (f (+ x dx) (f x))
             dx)))))

 The rights and privileges of first-class citizens:

 * to be named by variables
 * to be passed as arguments to procedures
 * to be returned as values of procedures
 * to be incorporated into data structures
