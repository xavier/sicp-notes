# SICP Lecture 4B
# Generic Operators

*Harold Abelson*

**Data abstraction separates use from representation**

It's powerful but not sufficient from very complex system: incompatible representations.

data abstraction = horizontal barrier
generic operators = vertical barrier

Allow multiple implementation for the same abstraction: ~> polymorphism

## Arithmetic on Complex Numbers

           Im
           |
         y |....          z = x + iy = re^iA
           |   /.         x = r cos A
           | r/ .         y = r sin A
           | /A .         r = sqrt(x*x + y*y)
     ------|/----- Re     A = arctan(y, x)
           |    x
           |

Complex numbers have two representations:

* Rectangular form: (x, y)
* Polar form: (A, r)

We want to build the following operations on top of data abstraction layer:

    (define (+c z1 z2) ...)
    (define (-c z1 z2) ...)
    (define (*c z1 z2) ...)
    (define (/c z1 z2) ...)

Sum is easy in rectangular form

    Re(z1 + z2) = (Re z1) + (Re z2)
    Im(z1 + z2) = (Im z1) + (Im z2)

Multiplication is easy in polar form:

    Mag(z1 * z2)   = (Mag z1) * (Mag z2)
    Angle(z1 * z2) = (Angle z1) + (Angle z2)

Our data abstraction layer looks like:

    ;; Selectors
    (real-part z)
    (imag-part z)
    (magnitude z)
    (angle z)

    ;; Constructors
    (make-rectangular x y)
    (make-polar r a)

We can implement our operations:

    (define (+c z1 z2)
      (make-rectangular
        (+ (real-part z1) (real-part z2))
        (+ (imag-part z1) (imag-part z2))))

    (define (-c z1 z2)
      (make-rectangular
        (- (real-part z1) (real-part z2))
        (- (imag-part z1) (imag-part z2))))

    (define (*c z1 z2)
      (make-polar
        (* (magnitude z1) (magnitude z2))
        (+ (angle z1) (angle z2))))

    (define (*c z1 z2)
      (make-polar
        (/ (magnitude z1) (magnitude z2))
        (- (angle z1) (angle z2))))

Representation of complex numbers as (real-part, imag-part) pairs

    (define (make-rectangular x y)
      (cons x y))

    (define (real-part z) (car z))

    (define (imag-part z) (cdr z))

    (define (make-polar r a)
      (cons (* r (cos a)) (* r (sin a))))

    (define (magnitude z)
      (sqrt (+ (square (car z))
               (square (cdr z)))))

    (define (angle z)
      (atan (cdr z) (car z)))

Representation of complex numbers as (magnitude, angle) pairs

    (define (make-polar r a) (cons r a))

    (define (magnitude z) (car z))

    (define (angle z) (cdr z))

    (define (make-rectangular x y)
      (cons (sqrt (+ (square x) (square y)))
            (atan y x)))

    (define (real-part z)
      (* (car z) (cos (cdr z))))

    (define (imag-part z)
      (* (car z) (sin (cdr z))))

Both representations have advantages and drawbacks depending on the type of operation.
We can't decide which representation is universally better.

## Typed Data

                  +c   -c   *c   /c
    ------------------------------------------------
       real-part   imag-part   magnitude    angle
    ------------------------------------------------
                          |
        rectangular       |         polar
                          |

We need to introduce **typed data** as a (type, contents) pair:

    ;; Support mechanism for manifest types

    (define (attach-type type contents)
      (cons type contents))

    (define (type datum)
      (car datum))

    (define (contents datum)
      (cdr datum))

And we add **type predicates**:

    (define (rectangular? z)
      (eq? (type z) 'rectangular))

    (define (polar? z)
      (eq? (type z) 'polar))

Now, in the rectangular package:

    ;; Wrap contents into a typed object
    (define (make-rectangular x y)
      (attach-type 'rectangular (const x y)))

    ;; Avoid name clash
    (define (real-part-rectangular z)
      (car z))

    (define (imag-part-rectangular z)
      (cdr z))

And in the polar package:

    ;; Wrap contents into a typed object
    (define (make-polar r a)
      (attach-type 'polar (const r a)))

    ;; Avoid name clash
    (define (real-part-polar z)
      (* (car z) (cos (cdr z))))

    (define (imag-part-polar z)
      (* (car z) (sin (cdr z))))

We can create generic selectors (a "manager"):

    ;;; Dispatch on time
    (define (real-part z)
      (cond ((rectangular? z)
             (real-part-rectangular (contents z)))
            ((polar? z)
             (real-part-polar (contents z)))))

    (define (imag-part z)
      (cond ((rectangular? z) ...)
            ((polar? z) ...)))

But this obviously stinks:

  * Procedures had to be renamed in the packages
  * Duplication
  * The manager is an anemic layer, it doesn't do anything

We need to get rid of the manager which is basically a look-up table.

## Data-directed programming

We can design an interface to manipulate the contents of a table:

    (put key1 key2 value)
    (get key1 key2)

Each package has to install its operations in the table:

    ;;; In the rectangular package
    ;;;   type        op name    procedure
    (put 'rectangular 'real-part real-part-rectangular)
    (put 'rectangular 'imag-part img-part-rectangular)
    (put 'rectangular 'magnitude magnitude-rectangular)
    (put 'rectangular 'angle     angle-rectangular)

    ;;; In the polar package
    (put 'polar 'real-part real-part-polar)
    (put 'polar 'imag-part img-part-polar)
    (put 'polar 'magnitude magnitude-polar)
    (put 'polar 'angle     angle-polar)

Each representation setup a column in the table

The manager is **replaced by a single procedure** called `operate`

    ;;;
    (define (operate op obj)
      (let ((proc (get (type obj) op)))
        (if (not (null? proc))
          (proc (contents obj))
          (error "undefined operator"))))

    (define (real-part obj)
      (operate 'real-part obj))

    (define (imag-part obj)
      (operate 'imag-part obj))

    (define (magnitude obj)
      (operate 'magnitude obj))

    (define (angle obj)
      (operate 'angle obj))

## Higher-level Arithmetic Operations

We have a higher level algebraic operations package with generic operators `add`, `sub`, ...

How can we plug our rational number packages into this system?

    (define (make-rat x y)
      (attach-type 'rational (cons x y)))

    (put 'rational 'add +rat)
    (put 'rational 'sub -rat)
    (put 'rational 'mul *rat)
    (put 'rational 'div /rat)

In the higher-level package

    (define (add x y)
      (operate-2 'add x y))

    (define (operate-2 op arg1 arg2)
      (if (eq? (type arg1) (type arg2))
        (let ((proc (get (type arg1) op)))
          (if (not (null? proc))
            (proc (contents arg1) (contents arg2))
            (error "Op undefined on type")
          ))
       (error "Arg not same type")))

We can also plug our complex number arithmetic into this new system

    (define (make-complex z)
      (attach-type 'complex z))

    (define (+complex z1 z2)
      (make-complex (+c z1 z2)))

    (put 'complex 'add +complex)

    ...

And we can also install ordinary numbers:

    (define (make-number n)
      (attach-type 'number n))

    (define (+number x y)
      (make-number (+ x y)))

    (put 'number 'add +number)

    ...

We end up with **chain of types**

    complex -> rect -> (representation)

## Polynomial Operations

We choose a representation

    ;;; x^15 + 2x^7 + 5

    (polynomial x <term-list>)

    ;;; term-list is a list of (order, coefficient) pairs

    ((15 1) (7 2) (0 5))

And we install it:

    (define (make-polynomial var term-list)
      (attach-type 'polynomlial (cons var term-list)))

    (define (+poly p1 p2)
      (if (same-var? (var p1) (var p2))
        (make-polynomial
            (var p1)
            (+terms (term-list p1)
                    (term-list p2)))
        (error "Polys not in same var")))

    (put 'polynomial 'add +poly)

The heavy lifting is done by `+terms`

    (define (+terms L1 L2)
      (cond ((empty-termlist? L1) L2)
            ((empty-termlist? L2) L1)
            (else
                (let ((t1 (first-term L1))
                      (t2 (first-term L2)))
                  (cond
                    ((> (order t1) (order t2))
                      (adjoin-term
                        t1
                        (+terms (rest-terms L1) L2)))
                    ((< (order t1) (order t2))
                       (adjoin-term
                        t2
                        (+terms L1 (rest-terms L2))))
                    (else
                      (adjoin-term
                        (make-term (order t1)
                                   (ADD (coeff t1)
                                        (coeff t2)))
                        (+terms (rest-terms L1)
                                (rest-terms L2)))))))))

Since we use `ADD`, we can deal with polynomial whose coefficients are rational, complex numbers or even polynomials in a different variable.  We have built a **recursive tower of types** by simply using `ADD` instead of `+`.

If we go back to our rational numbers package and replace the operators by `ADD`, `MUL`, etc we can use rationals whose numerator and denominator are polynomials.

By getting rid of the manager, we have built a system which has a **decentralized control**.

The system doesn't support coercion.  Where do we put that knowledge? The types? The operator?

