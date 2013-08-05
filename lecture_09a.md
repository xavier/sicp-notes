# SICP Lecture 9A
# Register Machines

*Gerald Jay Sussman*

> “In fact the nicest programs I know look more like a pile of languages than like a decomposition of a problem into parts”

## Demystification of the Mechanisms of Execution

Let's turn simple Lisp programs into hardware:

    (define (gcd a b)
      (if (= b 0)
          a
          (gcd b (remainder a b))))

This algorithm has two well defined state variables `a` and `b`.

      a                 b
    +----------+      +----------+      /-------\
    | register |<-----| register |----->| zero? |
    +----------+      +----------+      \-------/
              |        |    ^
              V        V    |
            /------------\  |
            | remainder  |  |
            \------------/  |
                  |         |
               t  V         |
             +----------+   |
             | register |---+
             +----------+

**Data path** of a GCD machine:

* We can move the content of `b` to `a`
* We can calculate the remainder of `a` and `b`
* We can check if `b` is 0.
* We need a temporary storage `t` (since all operations happen one at a time)
* We can store `t` into `b`

**Controller**

    loop:
      IF b = 0
        DONE
      ELSE
        t <- remainder
        a <- b
        b <- t
        GOTO loop

We create a language to represent the machine:

    (define-machine gcd
      (registers a b t)
      (controller
        loop (branch (zero (fetch b)) done)
             (assign t (remainder (fetch a) (fetch b)))
             (assign a (fetch b))
             (assign b (fetch t))
             (goto loop)
        done))

Adding I/O to get the input and display the output:

    (define-machine gcd
      (registers a b t)
      (controller
        main (assign a (read))
             (assign b (read))
        loop (branch (zero (fetch b)) done)
             (assign t (remainder (fetch a) (fetch b)))
             (assign a (fetch b))
             (assign b (fetch t))
             (goto loop)
        done (perform (print (fetch a)))
             (goto main)))

Parts such as the `remainder` look similar to the GCD algorithm and could be implemented as a machine in the same way:

    (define (remainder n d)
      (if (< n d)
          n
          (remainder (- n d) d)))

## Dealing with recursion

Example:

    (define (fact n)
      (if (= n 1)
           1
           (* n (fact (-1+ n)))))

We obviously cannot nest an infinite number of machines, we need an "illusion" to make our machine look as infinite as we need it to be.

    +------------+        +-------------------------+  \
    | data paths |<------>| finite-state controller |   | finite
    +------------+        +-------------------------+  /
          ^
          |
          v                                            \
    +------------+                                      | "infinite"
    |   stack    |                                      |
    |            |                                      |  keeps track of the state
    .            .                                      |  of the "outer machine"
    .            .                                      |
    :            :                                      |

Let's design the machine:

                                                       +-------+    +------+
                                                       | after |    | done |
                                                       +-------+    +------+
                                                              |       |
                                                              V       V
                                                           /------------\
                                                           | controller |
                                                           \------------/
                                                                 ^
                              +------------------------+         | continue
                              |                        |         |
     val                n     |                        V         V
    +----------+      +----------+      /------\     +---------------+
    | register |<-----| register |----->| one? |     |    stack      |
    +----------+      +----------+      \------/     +---------------+
       ^     |         |      | ^                    |               |
       |     V         V      V |                    +---------------+
       |   /------------\   /-------------\          +---------------+
       +---|  multipler |   | decrementer |          +---------------+
           \------------/   \-------------/          .               .
                                                     :               :

Code for the controller:

    (assign continue done)
    loop  (branch (= 1 (fetch n)) base)
          (save continue)    ;; push onto stack
          (save n)
          (assign n (decrement (fetch n)))
          (assign continue after)
          (goto loop)
    after (restore n)
          (restore continue)
          (assign val (* (fetch n) (fetch val)))
          (goto (fetch continue))
    base  (assign val (fetch n))
          (goto (fetch continue))
    done

> “Memory is cheap, people are expensive”

Dealing with multiple recursion:

    (define (fib n)
      (if (< n 2)
          n
          (+ (fib (- n 1))
             (fib (- n 2)))))

Controller:

    (assign continue fib-done)
    fib-loop  ; n contains the argument, continue is the recipient
      (branch (< (fetch n) 2) immediate-answer)
      (save continue)
      (assign continue after-fib-n-1)
      (save n)
      (assign n (- (fetch n) 1))
      (goto fib-loop)
    after-fib-n-1
      (restore n)
      (assign n (- (fetch n) 2))
      (assign continue after-fib-n-2)
      (save val)
      (goto fib-loop)
    after-fib-n-2
      (assign n (fetch val))   ; fib(n-2)
      (restore val)            ; fib(n-1)
      (restore continue)
      (assign val (+ (fetch val) (fetch n)))
      (goto (fetch continue))
    immediate-answer
      (assign val (fetch n))
      (goto (fetch continue))
    fib-done

