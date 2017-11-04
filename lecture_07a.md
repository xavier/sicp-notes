# SICP Lecture 7A
# Metacircular Evaluator, Part 1

*Gerald Jay Sussman*

# A Universal Machine

`eval` is a "universal machine", it takes as input an other machine.

> “There's a certain amount of mysticism that will appear here which may be disturbing and may cause trouble in your minds”

    (define eval
      (lambda (exp env)                            ;; env is a dictionary mapping symbols to values
        (cond
          ;; 3 -> 3
          ((number? exp) exp)
          ;; x -> 3 or car -> [procedure]
          ((symbol? exp) (lookup exp env))
          ;; 'foo == (quote foo) -> foo
          ((eq? (car exp) 'quote) (cadr exp))
          ;; (lambda x) (+ x y) -> (closure ((x) (+ x y)) env)  bound variables + body + env
          ((eq? (car exp) 'lambda) (list 'closure (cdr exp) env))
          ;; (cond (p1 e1) (p2 e2) ...) ->
          ((eq? (car exp) 'cond) (evcond (cdr exp) env))
          ;; (+ x 3) -> default, general application
          (else
            (apply (eval (car exp) env)
                   (evlist (cdr exp) env))))))

we still need lookup, evcond, apply, evlist

    (define apply
      (lambda (proc args)
        (cond ((primitive? proc) (apply-primop proc args))
              ((eq? (car proc) 'closure)
                ;; evaluate the body
                (eval (cadadr proc)
                      (bind (caadr proc)
                        args
                        (caddr proc))))
              (else "error"))))

    (define evlist
      (lambda (l env)
        (cond ((eq? l '()) '())
              (else
                (cons (eval (car l) env)
                      (evlist (cdr l) env))))))

    (define evcond
      (lambda (clauses env)
        (cond ((eq? clauses '()) '())
              ((eq? (caar clauses) 'else) (eval (cadar clauses) env))
              ;; Skip false branch
              ((false? (eval (caar clauses))) (evcond (cdr clauses) env))
              (else
                (eval (cadar clauses) env)))))

    (define bind
      (lambda (vars vals env)
        (cons (pair-up vars vals) env)))

    (define pair-up
      (lambda (vars vals)
        (cond
            ((eq? vars '())
              (cond ((eq? vals '()) '())
                    (else (error "Too Many Arguments"))))
            ((eq? vals '()) (error "Too Few Arguments"))
            (else
              (cons (cons (car vars) (car vals))
                    (pair-up (cdr vars) (vdr vals)))))))

    (define lookup
      (lambda (sym env)
        (cond ((eq? env '()) (error "Unbound Variable"))
              (else
                ((lambda (value-cell)
                  (cond ((eq? value-cell '())
                    (lookpu sys (cdr env)))
                  (else (cdr value-cell))))
                (assq sym (car env)))))))

    (define assq
      (lambda (sym list-of-pairs)
        (cond ((eq? list-of-pairs '()) '())
              ((eq? sym (caar list-of-pairs)) (car list-of-pairs))
              (else
                (assq sym (cdr list-of-pairs))))))

## Stepping through an evaluation

    (eval '(((lambda (x) (lambda y) (+ x y))) 3) 4) e0)

The default environment contains definitions for `+`, `-`, `car`, `cdr`, `cons`, `eq?`, ...
We first go through the special forms.

    (apply (eval '(lambda (x) (lambda y) (+ x y))) 3) e0) (evlist '() e0)
    (apply (eval '(lambda (x) (lambda y) (+ x y))) 3) e0) (cons (eval 4 e0) (evlist '() e0))
    (apply (eval '(lambda (x) (lambda y) (+ x y))) 3) e0) (cons 4 '())
    (apply (eval '(lambda (x) (lambda y) (+ x y))) 3) e0) (4))
    (apply (apply '(lambda (x) (lambda y) (+ x y))) e0) '(3)) '(4))
    (apply (apply '(closure (x) (lambda y) (+ x y))) e0) '(3)) '(4))
    ;; The closure builds a new environment e1 built from e0 + the binding of x to 3
    (apply (eval '(lambda y) (+ x y) e1) '(4))
    (apply (eval '(lambda y) (+ x y) e1) '(4))
    (apply (eval '(closure (y) (+ x y) e1) '(4))
    ;; The closure builds a new environment e2 built from e1 + the binding of y to 4
    (eval (+ x y) e2)
    (apply (eval + e2) (evlist '(x y) e2))
    (apply '+ '(3 4))
    7

`apply` and `eval` feed each others:

      +-----------------------------+
      |         proc, args          |
      |                             V
    EVAL                          APPLY
      ^                             |
      |          exp, env           |
      +-----------------------------+


## Introducing Y

    (define expt
      (lambda (x n)
        (cond ((= n 0) 1)
              (else
                (* x (expt x (- n 1)))))))

    F = (lambda (g)
          (lambda x n)
            (cond ((= n 0) 1)
                  (else
                      (* x
                          (g * (- n 1))))))

`expt` if a **fixed point** of `F`.

    expt = Lim En
           n -> ∞

    expt = (F (F (F (F ... (F ⊥)))))

Infinite loop:

    ((lambda (x) (x x)) (lambda (x) (x x)))

Curry's Paradoxical Combinator :

    Y = (lambda (f)
          ((lambda (x) (f (x x)))
           (lambda (x) (f (x x))))

    (Y F) = ((lambda (x) (F (x x)))
             (lambda (x) (F (x x))))

          = (F ((lambda (x) (F (x x))) (lambda (x) (F x x))))

    (Y F) = (F (Y F))

When applied to a function, Y returns the fixed point of the function.

