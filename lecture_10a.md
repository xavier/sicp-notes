# SICP Lecture 10A
# Compilation

*Harold Abelson*

## A Special Purpose Machine

### Interpreters

> We are writing an interpreter to raise the machine to the level of the programs we want to write.

         +-----------------------------------+
         |         Lisp Interpreter          |
         | +-------------------------------+ |
         | | Register Language Interpreter | |
      5  | +-------------------------------+ | 120
    ---->|         ^                ^        |----->
         | /–---------------\  /---------\   |
         | | (assign val    |  | Library |   |
         | |   (fetch exp)) |  \---------/   |
         | \----------------/                |
         +-----------------------------------+
                           ^
                 /------------------\
                 | (define (fact n) |
                 |   (if ...))      |
                 \------------------/


### Compilers

We lower our language down to the level of the machine in order to make our code run more efficiently.

    /------------------\  source code   +----------+
    | (define (fact n) |--------------->| Compiler |
    |   (if ...))      |                +----------+
    \------------------/                     |   translates into
                                object code  |   register language
                                             |
                                             V
                                /–---------------\
               +--------+       | (assign val    |
               |        |<------|   (fetch exp)) |
               | Linker |       \----------------/
               |   /
               | Loader |       /---------\
               |        |<------| Library |
               +--------+       \---------/
                   |
                   | load module
                   |
                   V
          +-------------------+
      5   | Register Language | 120
    ----->|    Interpreter    |----->
          +-------------------+

To build a very simple compiler, we can simply capture the output of the evaluator instead of interpreting it.

Example: register operations in interpreting `(f x)`

    (assign unev (operands (fetch exp)))
    (assign exp (operator (fetch exp)))
    (save continue)
    (save env)
    (save unev)
    (assign continue eval-args)
    (assign val (lookup-var-val (fetch exp (fetch env))))
    (restore unev)
    (restore env)
    (assign fun (fetch val))
    (save fun)
    (assign argl '())
    (save argl)
    ...

Dealing with conditionals in a 0th order compiler:

    (if p a b)

      <code for p - result in val>
      branch if val is true goto label1
      <code for b>
      goto next
    label1:
      <code for a>
    next:
      ...

## Optimizations

In the previous registeration operations example, some irrelevant operations were emitted because they deal with an invariant.  The 19 initial operations can be optimized into a sequence of only 10 instructions:

    (save env)
    (assign val (lookup-var-val 'f (fetch env)))
    (restore env)
    (assign fun (fetch val))
    (assign arg '())
    (save argl)
    (assign val (lookup-var-val 'x (fetch env)))
    (restor argl)
    (assign argl (cons (fetch val) (fetch argl)))
    (restore fun)

Now if we get rid of the useless stack operations, we are down to this:

    (assign fun (lookup-var-val 'f (fetch env)))
    (assign val (lookup-var-val 'x (fetch env)))
    (assign argl (cons (fetch val) (fetch argl)))

    ;; computation proceeds at apply-dispatch

Using constant structure of code:

         (   f          x     )
    env  | v |  | ^     |  |
    unev |  v|  |^      |  |
    cont |v  |  |       |  |  ^
    func |   |  |  v| | |  | ^
    argl |   |  |   |v| |  |v


Append `seq1` to `seq2` preserving some registers:

    if seq2 needs reg and seq1 modifies reg
      (save reg)
      <compile seq1>
      (restore reg)
      <compile seq2>
    else
      <compile seq1>
      <compile seq2>

Where the compiler has the preserving **notes**, the interpreter is more pessimistic and always saves the state.

To the compiler an **annotated code sequence** contains:

* a **sequence of instrutions**
* set of **registers modified**
* set of **registers needed**

Example:

* `(assign r1 (fetch r2))`
* {`r1`}
* {`r2`}

The combination of `<s1,m1,n1>` with `<s2,m2,n2>` produces

* `s1` follwed by `s2`
* `m1` ∪ `m2`
* `n1` ∪ [`n2` - `m1`]

**Register optimizations** are based what registers are needed and modifier by code fragments.
