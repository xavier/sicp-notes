# SICP Lecture 4A
# Pattern Matching and Rule-based Substitution

*Gerald Jay Sussman*

Rules have parts: a left handside and right handside (replacement)

              rule
    Pattern --------> skeleton

* pattern : something that matches
* skeleton : something you susbtitute into an expression in order to get a new expression

Pattern is matched against a source expression

The result of the application of the rule is to produce a target expression by instantiation of a skeleton.

A representation of the rules of calculus in a simple language:

    (define deriv-rules
      '(
          ( (dd (?c c) (? v)) 0 )
          ( (dd (?v v) (? v)) 1 )
          ( (dd (?v u) (? v)) 0 )

          ( (dd (* (? x1) (? x2)) (? v))
            (* (dd (: x1) (: v))
               (dd (: x2) (: v))))

          ( (dd (* (? x1) (? x2)) (?v))
            (+ (* (: x1) (dd (: x2) (: v)))
               (* (dd (: x1) (: v)) (: x2))))
            )
          )

          ...
      )
    )


Pattern match:

    foo     -> matches exactly foo
    (f a b) -> matches any list whose first element is f
                                      second element is a
                                      third element is b
    (? x)   -> matches anything, call it x
    (?c x)  -> matches a constant, call it x
    (?v x)  -> matches a variable, call it x

Skeletons:

    foo       -> instantiates to itself
    (foo a b) -> instantiates to a list of 3 elements which is the result of instantiating each
                 of f, a and b
    (: x)     -> instantiates to the value of x as in the matched pattern

Simplifier:

    (define dsimp (simplifier deriv-rules))

Usage example:

    (dsimp '(dd (+ x y) x))
    => (+ 1 0)

Other example of rules for simplification of algebraic expression:

    (define algebra-rules '(
        ( ((? op) (?c e1) (?c e2))
          (: (op e1 e2))

        ; this one is buggy
        ( ((? op) (? e1) (?c e2))
          ((? op) (: e2) (: e1))

        ( (+ 0 (? e)) (: e))
        ( (* 1 (? e)) (: e))
        ( (* 0 (? e)) 0)
        ...
      )
    )

Each rule is a pair with a pattern and a skeleton

1. The patterns from the rules are fed into a matcher.
2. The matched passes a dictionary (x was matched against a given sub expression) to the instantiator
3. The instantiator receives the skeletons from the rules.
4. The new expressions produced by the instantiator are fed back into the matcher.

The while process is recursive, until nothing changes.

matcher(expresion, pattern, dictionary) -> new dictionary

    (define (match pat exp dict)
      (cond ((eq? dict 'failed) 'failed')
            ((atom? pat)
                *** Atmoic patterns)
            *** Pattern variables clauses
            ((atom? exp) 'failed)
            (else
              (match (cdr pat)
                     (cdr exp)
                     (match (car pat) (car exp dict)))))

Example of expression matching:

    (+ (* (?x) (?y)) (?y))
    (+ (*   3    x)    x)

    => procudes the dictionary {x => 3, y = 'x}


    ((atom? pat)
      (if (atom? exp)
        (if (eq? pat exp)
          dict
          'failed)
      'failed'))


    ((arbitrary-constant? pat)
      (if (constant? exp)
        (extend-dict pat exp dict)
        'failed))

    ((arbitrary-variable? pat)
      (if (variable? exp)
        (extend-dict pat exp dict)
        'failed))

    ((arbitrary-expression? pat)
      (extend-dict pat exp dict))


instantiator(dict, skel) -> expr

    ; recursive tree walk of the skeleton
    (define (instantiate skel dict)
      (define (loop s)
        (cond ((atom? s) s)
              ((skeleton-evaluation? s)
               (evaluate (eval-exp s) dict))
             (else (const (loop (car s) (cdr s))))))
    (loop skel))


    (define (evaluate form dict)
      (if (atom? form)
        (lookup form dict)
        (apply
          (eval (lookup (car form) dict) user-initial-environment)
          (mapcar (lambda (v) (lookup v dict)) (cdr form)))))


Garbage In, Garbage Out (GIGO) simplifier

    ; Returns a procedure for the given set of rules
    (define (simplifier the-rules)
      (define (simplify-exp exp)
        ...)
      (define (simplify-parts exp)
        ...)
      (define (try-rules exp)
        ...)
      simplify-exp
    )

    (define (simplify-exp exp)
      (try-rules (if (compound? exp)
        (simplify-parts exp)
        exp)))

    (define (simplify-parts exp)
      (if (null? exp)
        '()
        (cons (simplify-exp (car exp))
              (simplify-exp (cdr exp)))))


    ; Other idiom
    (define (simplify-exp exp)
      (try-rules
        (if (compound? exp)
          (map simplify-exp exp)
          exp
        )))

    (define (try-rules exp)
      (define (scan rules)
        ...)
      (scan the-rules))

    (define (scan rules)
      (if (null? rules)
        exp
        (let ((dict
                (match (pattern (car rules)) exp (empty-dictionary))))
          (if (eq? dict 'failed)
            (scan (cdr rules))
            (simplify-exp
                (instantiate
                  (skeleton (car rules))
                  dict))))))

    (define (empty-dictionary) '())

    (define (extend-dictionary pat dat dict)
      (let ((name (variable-name pat)))
        (let ((v (assq name dict)))
          (cond ((null? v)
            (cons (list name dat) dict))
            ((eq? (cadr v) dat) dict)
            (else 'failed)))))

    (define (lookup var dict)
      (let ((v (assq var dict)))
        (if (null? v) var (cadr v))))
