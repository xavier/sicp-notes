# SICP Lecture 3B
# Symbolic Differentiation; Quotation

*Gerald Jay Sussman*

## Calculating Derivativees

Reduction rules are appropriate for recursion.

We are not calcualting the derivative of a function, we are calculating the derivative of an expression.  We are manipulating syntax.

We dispatch based on the type of

    (define (deriv exp var)
      (cond ((constant? exp var) 0)
            ((same-variable? exp var) 1)
            ((sum? exp var)
              (make-sum
                (deriv (a1 exp) var)
                (deriv (a2 exp) var)))
            ((product? exp var)
              (make-sum
                (make-product
                  (deriv (m1 exp) var)
                  (deriv (m2 exp) var))
               (make-product
                  (deriv (m1 exp) var)
                  (deriv (m2 exp) var))
              ))
            ...
    ))

    (define (constant? exp var)
      (and (atom? exp)
           (not (eq? exp var))))

    (define (same-variable? exp var)
      (and (atom? exp)
           (eq? exp var)))

    (define (sum? exp var)
      (and (not (atom? exp))
           (eq? '+ (car exp))))

    ; Quote = the symbol itself and not what it represents

    (define (make-sum a1 a2)
      (list '+ a1 a2))

    (define a1 cadr)      ; (car (cdr x))
    (define a2 caddr)     ; (car (cdr (cdr x)))

    (define (product? exp var)
      (and (not (atom? exp))
           (eq? '* (car exp))))

    (define (make-product m1 m2)
      (list '* m1 m2))

    (define m1 cadr)      ; (car (cdr x))
    (define m2 caddr)     ; (car (cdr (cdr x)))


Let's derive the quadratic equation `ax^2 + bx + c`:

    (define foo
      (+ (* a (* x x))
         (* b x)
         c))

    (deriv foo 'x)
    ; Expected 2ax + b but got:
    (+ (+ (* a (+ (* x 1) (* 1 X)))
            (* 0 (* x x)))
        (* (* (* b 1) (* 0 x))
        0))

The answer is right but the expression is ugly.

**The form of the process is expanded from the local rules seen in the procedure.  The procedure represent a set of local rules for the expansion of the process**.

Every part of the answer derives from some part of the problem.

      derivative rules
    --------------------------------------------
      constant?       sum?      a1
      same-variable?  make-sum  a2 ...
    --------------------------------------------
      representation of algebraic expressions

Thanks to the abstraction barrier made of interface procedures, we can arbitrarily change the representation without changing the rules.

    (define (make-sum a1 a2)
      (cond ((and (number? a1) (number? a2))
             (+ a1 a2)
            ((and (number a1) (= a1 0))
              a2)
            ((and (number a2) (= a2 0))
              a1)
            (else (list '+ a1 a2)))))

