# SICP Lecture 8A
# Logic Programming (Part 1)

*Harold Abelson*

An evaluator for Lisp has two main elements:

1. `eval` takes an **expression** and an **environment** and turns that into a **procedure** and **arguments**
2. `apply` takes the **procedure** and the **arguments** and turns it into another **expression** to be evaluated

This cycle unwinds the **means of combinations** and the **means of abstraction** in the language.

**Metalinguistic abstraction**: ability to gain control of complexity by inventing new languages.

**Declarative knowledge** is based on a set of **facts** unbiased as to what the question is.

Facts:

    son-of adam abel
    son-of adam cain
    son-of cain enoch
    son-of enoch irad

Rules of inference:

    if (son-of ?x ?y) and (son-of ?y ?z) then
      (grand-son ?x ?z)

Example of program:

    (define (merge-sorted-list x y)
      (cond
        ((null? x) y)
        ((null? y) x)
        (else
          (let ((a (car x)) (b (car y)))
            (if (< a b)
              (cons a (merge-sorted-list (cdr x) y))
              (cons b (merge-sorted-list x (cdr y))))))))

Let's extract the logic on which this program is based:

    if (cdr-x and y merge-to-form z) and (a < (car y)) then
      ((cons a cdr-x) and y merge-to-form (cons a z))

    if (x and (cdr-y) merge-to-form z) and (b < (car x)) then
      (x and (cons b cdr-y) merge-to-form (cons b z))

    for all x, (x and () merge-to-form x)

    for all y, (() and y merge-to-form y)

Example of questions:

    (1 3 7) and (2 4 8) merge-to-form ?

    (1 3 7) and ? merge-to-form (1 2 3 4 7 8)

    ? and ? merge-to-form (1 2 3 4 7 8)

## Desiging the Query Language

Definition of facts

    (job (Bitdiddle Ben) (computer wizard))

    (salary (Bitdiddle Ben) 40000)

    (supervisor (Bitdiddle Ben) (Warbucks Oliver))

    (address (Bitdiddle Ben) (Slumervill (Ridge Road) 10))

    (address (Hacker Alyssa P) (Cambridge (Mass Ave) 78))

    (job (Hacker Alyssa P) (computer programmer))

    (salary (Hacker Alyssa P) (35000))

    (supervisor (Hacker Alyssa P) (Bitdiddle Ben))

    (job (Tweakit Lem E) (computer technician))

    ...

The only **primitive** in our language is a **query**.

Example with a single variable:

    (job ?x (computer programmer))

    matches

    (job (Hacker Alyssa P) (computer programmer))

    where x is (Hacker Alyssa P)

Example with a two variables:

    (job ?x (computer (?type)))

    matches

    (job (Bitdiddle Ben) (computer wizard))
    (job (Alyssa P Hacker) (computer programmer))
    (job (Tweakit Lem E) (computer programmer))

**Compounds queries** provide us with **means of combination**

    (and (job ?x (computer . ?y))
         (supervisor ?x ?z))

    => list all people who work in the computer division,
       together with their supervisor

    (and (salary ?p ?a)
          (lisp-value > ?a 30000))

    => List all people whose salary is greater than 30000

    (and
      (job ?x (computer . ?y))
      (not (and (supervisor ?x ?z)
                (job ?z (computer . ?w)))))

    => List all people who work in the computer division,
       who do not have a supervisor who works in the computer division

`lisp-value` allows to invoke any Lisp predicate from the query language.

The addition of *rules* introduce **means of abstraction**:

    ;; (rule <conclusion> <body>)
    ;; if the body is true, the conclusion is true
    (rule
        (bigshot ?x ?dept)
        (and
          (job ?x (?dept . ?y))
          (not (and (supervisor ?x ?z)
                    (job ?z (?dept . ?w))))))

    ;; conversion of the earlier rule into our language
    (rule
        (merge-to-form
            (?a . ?x) (?b . ?y) (?b . ?z))
        (and (merge-to-form (?a . ?x) ?y ?z)
             (lisp-value > ?a ?b)))
