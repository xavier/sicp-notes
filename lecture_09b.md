# SICP Lecture 9B
# Explicit-control Evaluator

*Harold Abelson & Gerald Jay Sussman*

The magic of building languages:

- Escher Picture language
- Digital logic language
- Query language
- ...

All based on Lisp.

> Lisp is not good for solving any particular problem.  What Lisp is good for is constructing within it the right language to solve the problem you want to solve

What's Lisp based on?

Lisp is based on Lisp: *cfr* the **metacircular evaluator**.

To make the magic disappear: let's implement Lisp in terms of a **register machine architecture**

    +-------------------------+     +------------+     +---------- - -  -   -
    | finite state controller |<--->| data paths |<--->| stack
    +-------------------------+     +------------+     +---------- - -  -   -

Architecture of a Lisp system:

          characters           list struct
    User ------------> Reader ------------> Evaluator ----------> Printer
     ^                   |                      |                   |
     |                   V                      V                   |
     |             List Structure           Primitive               |
     |                Memory                Operators               |
     |                                                              |
     +--------------------------------------------------------------+

We will focus on the **evaluator**, it has 7 registers:

1. `exp`, expression to to be evaluated (pointer to List Structure Memory)
1. `env`, evaluation environment (pointer to List Structure Memory)
1. `fun`, procedure to be applied
1. `argl`, list of evaluated argument
1. `continue`, place to go to next
1. `val`, result of evaluation
1. `unev`, temporary register for expressions

Sample evaluator machine operations:

    (assign val (fetch ecp))

    (branch
      (conditional? (fetch exp))
      ev-cond)

    (assign exp (first-clause (fetch exp)))

    (assign val
      (lookup-variable-value (fetch exp)
                             (fetch env)))

Contract that `eval-dispatch` fulfills:

- the `exp` register holds an expression to be evaluated
- the `env` register holds the environment in which the expression is to be evaluated
- the `continue` register holds a place to go next
- the result will be left in the `val` register
- the contents of all other registers may be destroyed

Contract that `apply-dispatch` fulfills:

- the `argl` register contains a list of arguments
- the `fun` register contains a procedure to be applied
- the *top of the stack* holds a place to go next
- the result will be left in the `val` register
- the stack will be popped
- the contents of all other registers may be destroyed

## Evaluator (Partial)

    eval-dispatch
      (branch (self-evaluating? (fetch exp)) ev-self-eval)
      (branch (variable? (fetch exp)) ev-variable)
      ; ...
      (branch (application? (fetch exp)) ev-application)
      (goto unknown-expression-error)

    ev-self-eval
      (assign val (fetch exp))
      (goto (fetch continue))

    ev-variable
      (assign val (lookup-variable-value (fetch env)))
      (goto (fetch continue))

    ev-application
      (assign unev (operands (fetch exp)))
      (assign exp (operator (fetch exp)))  ; replace the expression by the operation to apply
      (save continue)
      (save env)
      (save unev)
      (assign continue eval-args)
      (goto eval-dispatch)                 ; recursive call

    eval-args
      (restore unev)
      (restore env)
      (assign fun (fetch val))
      (save fun)
      (assign argl '())
      (goto eval-arg-loop)

    eval-arg-loop
      (save argl)
      (assign exp (first-operand (fetch unev)))
      (branch (last-operand? (fetch unev)) eval-last-arg)
      (save env)
      (save unev)
      (assign continue accumulate-arg)
      (goto eval-dispatch)

    accumulate-arg
      (restore unev)
      (restore env)
      (restore argl)
      (assign argl (cons val) (fetch argl))
      (assign unev (rest-operands (fetch unev)))
      (goot eval-arg-loop)

    eval-last-arg
      (assign continue accumulate-last-arg)
      (goto eval-dispatch)

    accumulate-last-arg
      (restore argl)
      (assign argl (cons (fetch val) (fetch argl)))
      (restore fun)
      (goto apply-dispatch)

## Applicator

    apply-dispatch
      (branch (primitive-proc? (fetch fun)) primitive-apply)
      (branch (compound-proc? (fetch fun)) compound-apply)
      (goto unknown-proc-type-error)

    primitive-apply
      (assign val (apply-primitive-proc (fetch fun) (fetch argl)))
      (restore continue)
      (goto (fetch continue))

    compound-apply
      (assign exp (procedure-body (fetch fun)))
      (assign env (make-bindings (fetch fun) (fetch argl)))
      (restore continue)     ; this is where tail recursion happens
      (goto eval-dispatch)

