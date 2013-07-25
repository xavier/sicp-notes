# SICP Lecture 5A
# Assignment, State, and Side-effects

*Gerald Jay Sussman*

## Assignments

So far we haven't used any assignment statement, we have written **functional programs** which are **encoded mathematical truths**.
Processes evolved by such programs can be understood by **substitution**:

    (fact 3)
    (* 3 (fact 2))
    (* 3 (* 2 (fact 1)))
    (* 3 (* 2 1))
    (* 3 2)
    6

**An assignment creates a moment in time.**

    <before>
    (set! <var> <value>)
    <after>


    (define count 1)
    (define (demo x)
      (set! count (1+ count))
      (+ x count))

The same expression leads to two different answer dependent on time:

    (demo 3) => 5
    (demo 3) => 6

This kills the substitution model dead.  We have truths that change.  We have lost our model of computation.

Functional version:

    (define (fact n)
      (define (iter m i)
        (cond ((> i n)) m)
              (else (iter (* i m))))
      (iter 1 1))

Imperative version:

    (define (fact n)
      (let ((i 1) (m 1)))
        (define (loop)
          (cond ((> i n) m))
                (else
                  (set! m (*i m))
                  (set! i (+1 i))
                  (loop)))
        (loop)))

Dependency between statements introduces potential new bugs.

Both `define` and `let` happen only once, you cannot redefine a symbol.

    (let ((var1 e1) (var2 e2)) e3)

    <=>

    ((lambda (var1 var2) e3) e1 e2)


## The Environment Model

### Bound Variables

A variable `V` is **bound in an expresison** `E` if the meaning of `E` is **unchanged** by the uniform replacement of a variable `W`  not occurring in `E` for every occurence of `V` in `E`

    ∀x ∃y P(x, y)

In this example, x and y are bound variables: if they are changed to u and v, the meaning of the expression remains unchanged.

    (lambda (y) ((lambda (x) (* x y)) 3))

    <=>

    (lambda (y) ((lambda (z) (* z y)) 3))

### Free Variables

A variable `V` is **free in an expresison** `E` if the meaning of `E` is **changed** by the uniform replacement of a variable `W`  not occurring in `E` for every occurence of `V` in `E`

    (lambda (x) (* x y))

In this example the variable y is free (or unbound).

    (lambda (y) (lambda (x) (* x y)) 3))

In this expression the variable `*` is free.

### Scope

If `X` is a bound variable in `E` then there is a lambda expression where it is bound.  We call the list of formal parameters of the lambda expression the **"bound variable list"** and we sya that the lambda expression **"binds"** the variable **"declared"** in its bound variable list.  In addition, these parts of the expression where a variable has a value defined by the lambda expression which binds it is called the **"scope"** of the variable.

### Environments

An environment is a way to perform substitution virtually.  An environment is a function or a "table", it's made of frames (pieces of environment chained together).

A procedure is a composite object made of a (pointer to a) piece of code and a (pointer to an) environment.

    (define make-counter
      (lambda (n)
        (lambda()
          (set! n (1+ n))
          n)))

The `define` adds `make-counter` to the global environment.

    (define c1 (make-counter 0))

We evaluate make-counter in the global environment.  We construct a frame with a value for n = 0.  The lambda is linked to this frame.

    (define c2 (make-counter 10))

The new counter is linked to another frame with a value for n = 10.

We have created **computational objects** with their own **local state**.

By introducing **assignments** and **objects**, we have open a can of worms.

## Actions and Identity

We say that an action `A` has an effect on an object `X` (or equivalently that `X` has changed `A`) if some property `P` which was tru of `X` before `A` became false of `X` after `A`.

We say that two objects, `X` and `Y` are the same if any action which has an effect on `X` has the same effect on `Y`.

### Cesaro's method for estimating Pi

Prob(gcd(n1, n2) = 1) = 6/(Pi*Pi)

    (define (estimate-pi n)
      (sqrt (/ 6 (monte-carlo n cesaro))))

    (define (cesaro)
      (= (gcd (rand) (rand)) 1))

    (define (monte-carlo trials experiment)
      (define (iter remaining passed)
        (cond ((= remaining 0))
          (/ passed trials))
          ((experiment) (iter (-1+ remaining) (1+ passed)))
          (else
              (iter (-1+ remaining) passed))))
      (iter trials 0))

    (define rand
      (let ((x random-init))
        (lambda ()
          (set! x (rand-update x))
        x)))