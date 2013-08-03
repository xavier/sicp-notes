# SICP Lecture 8B
# Logic Programming (Part 2)

*Harold Abelson*

## Query Language Implementation

Sample patterns:

    (a ?x c)
    (job ?x (computer ?y)    ;; one element in ?y
    (job ?x (computer . ?y)  ;; arbitrary number of elements after in ?y
    (a ?x ?x)
    (?x ?y ?y ?x)
    (a . ?x)

Based on **pattern matching** (see 4A):

    ;; Is there any way to match pat against data with bindings in dict?
    (match pat data dict)

The dictionary is initially empty but can be populated by data provided by the user.

Example of input:

* pat = `(?x ?y ?y ?x)`
* data = `(a b b a)`
* dict = `{x: a}`

Ouput:

* extended dictionary = `{x: a, y: b}`

### Primitive Query

    (job ?x (?d . ?y))

Shape:

       2 input streams               1 output stream
                         +--------+
    database ----------->|        |
                         |        |--------> dictionaries
    dictionaries ------->|        |
                         +--------+

For each dictionary in the stream, it find all patterns matching in the database and it outputs zero, one or more dictionaries.

### Combinations

To implement `and`, the output of the first primitve is connected to the input of the second and the facts database is passed along.

#### AND

The inputs and output of the outerbox are the same as the `P` and `Q`'s.

      +----------------------+
      |   +---+      +---+   |
    ----->| P |----->| Q |------>
      |   +---+      +---+   |
      |     ^          ^     |
      |     |          |     |
      |     +-----+----+     |
      +-----------|----------+
                  |

#### OR

      +---------------------------+
      |        +---+      +---+   |
      |  +---->| P |----->| m |   |
      |  |     +---+      | e |   |
    -----+       ^        | r |-------->
      |  |     +---+      | g |   |
         +---->| Q |----->| e |   |
      |        +---+      +---+   |
      |          ^                |
      +----------|----------------+
                 |

#### NOT

Throws away any dictionary for which there is a match

          filter
          +---+
    ----->| P |----->
          +---+
            ^
            |

**Principle of closure**: all these elements have the **same shape**

### Rules

    (rule (boss ?z ?d)
      (and (job ?x (?d . ?y))
           (supervisor ?x ?z)))

A **unifier** is a slight **generalization of the pattern matcher** which takes two patterns and finds what are the most general things which can be substitued to satisfy both of the simultaneously.

Simple example:

    unify (?x ?x)
    with  ((a ?y c) (a b ?z))

        ?x : (a b c)
        ?y : b
        ?z : c

More complicated example:

    unify (?x ?x)
    with  ((?y a ?w) (b ?v ?z))

        ?y: b
        ?v: a
        ?w: ?z
        ?x: (b a ?w)

Even more clever:

    unify (?x ?x)
    with  (?y (a . ?y))

           ?x: ?y
           ?y: (a a a ...)

We need to solve a fixed point equation, which we can't.  There must be a guard to prevent the infinite loop.

Formal similarity with the Metacircular Evaluator:

* **To apply a rule**: evaluate the **rule body** relative to an **environment** formed by **unifying the rule conclusion** with the given **query**.
* **To apply a procedure**: evaluate the **procedure body** relative to an **environment** formed by **binding the procedure parameters** with the given **arguments**.

### Limitations

Our query language does not behave like classical predicate logic, our logic operators are **not commutative**.  Switch the order of statements may trigger an infinite loop.

* All humans are mortal
* All Greeks are human
* Socrates is a Greek
* Therefore, Socrates is mortal

    (Greek Socrates)
    (Greek Plato)
    (Greek Zeus)
    (god Zeus)

    (rule (mortal ?x) (human ?x))
    (rule (fallible ?x) (human ?x))

    (rule (human ?x)
      (and (Greek ?x) (not (god ?x))))

    (rule (address ?x Olympus)
      (and (Greek ?x) (god ?x)))

    (rule (perfect ?x)
      (and (not (mortal ?x)) (not (fallible ?x))))

Now let's find the addreses of all perfect beings:

    (and (addres ?x ?y)
         (perfect ?x))
    => Olympus

    (and (perfect ?x)
         (address ?x ?y))
    => ()

The difference is related to the implelmentation of `not` which is actually a filter.

Bigger problem:

* But which one of these answer is the correct one?
* How does it know that Zeus is infallible?
* How does it know that Zeus is not mortal?

Subtle but big difference: what `not` means in this language is **not deducible from the database**, which is not the same as **being not true**.

**Closed world assumption**: anything we don't know is considered false.  This is a dangerous bias.

This is a very deep problem which cannot be solved by a small algorithm tweak.

