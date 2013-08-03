# SICP Lecture 7B
# Metacircular Evaluator, Part 2

*Gerald Jay Sussman*

Metacircular Interpreters are defined in terms in such a way the language they interpret contains itself.  They are a convenient medium to experiment for exploring language issues and exchange ideas about language design.

## Adding a New Language Feature

Operations like `+` or `*` accept an indefinite number of arguments:

    (+ (* a x x)
       (* b x)
       c)

In our current implementation of Lisp, the number of arguments is fixed.

First, we define a notation:

E.g. we want to be able to define procedures such as:

    ;; x is required, y is a list of args
    (lambda (x . y)
      (map (lambda (u) (* x u)) y))

    (lambda x x) <=> list

Second, we implement the semantics:

    (define pair-up
      (lambda (vars vals)
        (cond
            ((eq? vars '())
              (cond ((eq? vals '()) '())
                    (else (error "Too Many Arguments"))))
            ((symbol? vars) ;; We simply add the vars to the environment
              (cons (cons vars vals) '()))
            ((eq? vals '()) (error "Too Few Arguments"))
            (else
              (cons (cons (car vars) (car vals))
                    (pair-up (cdr vars) (vdr vals)))))))

## Dynamic Binding of Variables

    (define sum
      (lambda (term a next b)
        (cond ((a > b) 0)
                (+ (term a)
                   (sum term (next a) next b)))))

    (define sum-powers
      (lambda (a b n)
        (sum (lambda (x) (expt x n)) a 1+ b)))

    (define product-powers
      (lambda (a b n)
        (product (lambda (x) (expt x n)) a 1+ b)))

Both `sum-powers` and `product-powers` look exactly the same.
We can abstract out part of the process:

    (define sum-powers
      (lambda (a b n)
        (sum nth-power a 1+ b)))

    (define product-powers
      (lambda (a b n)
        (product nth-power a 1+ b)))

    (define nth-power
      (lambda (x) (expt x n)))

The current environment model cannot give a meaning for the `n` in `nth-power`.

### A famous "bug"

> A **free variable** in a procedure has its value defined in the **chain of callers**, rather than where the procedure is defined.

Easy to implement:

    (define eval
      (lambda (exp env)
        (cond
          ((number? exp) exp)
          ((symbol? exp) (lookup exp env))
          ((eq? (car exp) 'quote) (cadr exp))
          ((eq? (car exp) 'lambda) exp)  ;; Removed closure
          ((eq? (car exp) 'cond) (evcond (cdr exp) env))
          (else
            (apply (eval (car exp) env)
                   (evlist (cdr exp) env) env)))))  ;; Inject caller environment
    (define apply
      (lambda (proc args env)
        (cond ((primitive? proc) (apply-primop proc args))
              ((eq? (car proc) 'lambda)
                ;; Replaced the closure env by the caller env
                (eval (cadadr proc)
                      (bind (caadr proc)
                        args
                        env)))
              (else "error"))))

**BUT**, this introduces **serious problems**.

We have a **modularity crisis**: name interference.  If we renamed `next` to `n` in the `sum` procedure, we break the other programs and we have to back track to the call chain to figure out the actual value of `n`.

Our tentative to abstract out `nth-power` as it is must be abandoned but we can work around this problem by creating a exponentiator generator:

    (define pgen
      (lambda (n)
        (lambda (x (expt x n)))))

    (define sum-powers
      (lambda (a b n)
        (sum (pgen n) a 1+ b)))

    (define product-powers
      (lambda (a b n)
        (product (pgen n) a 1+ b)))

Our ability to change the behaviour of the language by fiddling with `eval` and `apply` has allowed us to experiment with alternative language semantics.

## Named Arguments

    ;; predicate consequent alternative
    (define (unless p c a)
      (cond ((not p) c)
             (else a))

    (unless (= 1 0) 2 (/ 1 0))
    => (cond ((not (= 1 0)) 2)
              (else (/ 1 0)))

Problem: the division by zero will be evaluated before the `cond` has even been evaluated.

    (define (unless p (name c) (name a))
      (cond ((not p) c)
             (else a)))

We add delay into the evaluator and applicator

    (define eval
      (lambda (exp env)
        (cond
          ((number? exp) exp)
          ((symbol? exp) (lookup exp env))
          ((eq? (car exp) 'quote) (cadr exp))
          ((eq? (car exp) 'lambda) (list 'close (cdr exp) env))
          ((eq? (car exp) 'cond) (evcond (cdr exp) env))
          (else
            (apply (undelay
                    (eval (car exp) env)
                    (cdr exp)
                    env))))))

    (define apply
      (lambda (proc ops env)
        (cond ((primitive? proc)
              ;; force values into primitive operations
              (apply-primop proc (evlist ops env))
              ((eq? (car proc) 'closure)
                (eval (cadadr proc)
                      (bind
                        (vnames (caadr proc))
                        ;; wrap delay and force others
                        (gevlist (caadr proc) ops env)
                        (caddr proc))))
              (else "error"))))

    (define evlist
      (lambda (l env)
        (cond
          ((eq? l '()) '())
          (else
            ;; force value
            (cons (undelay (eval (car l) env))
                  (evlist (cdr l) env))))))

    (define gevlist
      (lambda (vars exps env)
        (cond
          ((eq? exps '()) '()))
          ((symbol? (car vars))
            ;; applicative order
            (cons (eval (car exps) env)
                  (gevlist (cdr vars) (cdr exps) env)))
          ((eq? (caar vars) 'name)
            ;; named parameter, delay the evaluation
            (cons (make-delay (car exps) env)
                  (gevlist (cdr vars) (cdr exps) env)))
          (else "error")))

    (define make-delay
      (lambda (exp env)
        (cons 'thunk (cons exp env))))

    (define (undelay v)
      (cond ((pair? v)
              (cond ((eq? (car v) 'thunk)
                      (undelay (eval (cadr v) (caddr v))))
                     (else v)
            (else v)))
