# SICP Lecture 5B
# Computational Objects

**Gerald Jay Sussman**

Object (with independant local state) tend to be suitable to model the physical world.

## A Language to Describe Digital Circuits

> “Electrical systems are the physicists best objects”

Primitives:

    (define a (make-wire))
    (define b (make-wire))
    (define c (make-wire))
    (define d (make-wire))
    (define e (make-wire))
    (define s (make-wire))

Means of Combination:

    (or-gate a b d)
    (and-gate a b c)
    (inverter c e)
    (and-gate d e s)

Wires represent **abstract signals**: we don't care if it's voltage, current, water pressure, ...

We will build an **embedded language** in Lisp to describe digital circuits.

    ;; Returns the sum of a and b in s, with carry in c
    (define (half-adder a b s c)
      (let ((d (make-wire) (c make-wire)))   ;; internal wires
        (or-gate a b d)
        (and-gate a b c)
        (inverter c e)
        (and-gate d e s)))

> “Lambda is the ultimate glue”

    (define (full-adder a b c-in sum c-out)
      (let ((s (make-wire))
            (c1 (make-wire))
            (c2 (make-wire)))
        (half-adder b c-in s c1)
        (half-adder a s sum c2)
        (or-gate c1 c2 c-out)))

We have now **primitives**, **means of combination** and **means of abstraction**, we can now implement the language:

### Inverter

    (define (inverter in out)
      (define (invert-in)
        (let ((new
                (logical-not (get-signal in))))
            (after-delay inverter-delay
              (lambda () (set-signal! out new)))))
      (add-action! in invert-in))

    (define (logical-not s)
      (cond ((= s 0) 1)
            ((= s 1) 0)
            (else
              (error "Invalid signal" s))))


### And Gate

    (define (and-gate a1 a1 output)
      (define (and-action-procedure)
        (let ((new-value
                (logical-and (get-signal a1)
                             (get-signal a2))))
          (after-delay and-gate-delay
            (lambda ()
              (set-signal! output new-value)))))
      (add-action! a1 and-action-procedure)
      (add-action! a2 and-action-procedure))

`set-signal!` is a derived assignment operator.

### Example of communication

      a
    --------|----\   c      |\   d
      b     |     |---------| O--------
    --------|____/          |/
           and-gate       inverter

Signals have a list of **action procedures**.

### Wires

    (define (make-wire)
      (let ((signal 0) (action-procs '()))
        (define (set-my-signal! new)
          (cond ((= signal new) 'done)
                (else
                  (set! signal new)
                  (call-each-action-procs))))
        (define (accept-action-proc proc)
          (set! action-procs
              (cons proc action-procs))
          (proc))
        (define (dispatch m)
          (cond ((eq? m 'get-signal) signal)
                ((eq? m 'set-signal!) set-my-signal!)
                ((eq? m 'add-action!) accept-action-proc)
                (else
                  (error "Bad message" m))))

        dispatch))

`make-wire` returns the dispatcher to which messages can be sent.

Some helper procedures:

    (define (call-each procedures)
      (cond ((null? procedures) 'done)
             (else
                ((car procedures))
                (call-each (cdr procedures)))))

    (define (get-signal wire)
      (wire 'get-signal))

    ;; Get the method to set the signal from the dispatcher and invoke it with the new value
    (define (set-signal! wire new-value)
      ((wire 'set-signal!) new-value))

    (define (add-action! wire action-proc)

### Delays

We have an Agenda, a structure which organizes time.

    (define (after-delay delay action)
      (add-to-agenda!
        (+ delay (current-time the-agenda))
        action
        the-agenda))
      ((wire 'add-action!) action-proc))

    (define (propagate)
      (cond ((empty-agenda? the-agenda) 'done)
             (else
                ((first-item the-agenda))
                ((remove-first-item! the-agenda)
                (propagate)))))

#### Example

    (define the-agenda (make-agenda))
    (define inverter-delay 2)
    (define and-gate-delay 3)
    (define or-gate-delay 5)

    (define input-1 (make-wire))
    (define input-2 (make-wire))
    (define sum (make-wire))
    (define carry (make-wire))

    (prob 'sum sum)
    >> SUM 0 NEW-VALUE = 0
    ;; SUM <time>

    (prob 'carry carry)
    >> CARRY 0 NEW-VALUE = 0

    (half-adder input-1 input 2 sum carry)
    (set-signal! input-1 1)
    (propagate)
    >> SUM 8 NEW-VALUE = 1
    >> DONE

    (set-signal! input-2 1)
    (propagate)
    >> CARRY 11 NEW-VALUE = 1
    >> SUM 15 NEW-VALUE = 0
    >> DONE

#### Implementing Agendas

Primitives:

    (make-agenda) -> new agenda
    (current-time agenda) -> number (time)
    (empty-agenda? agenda) -> true or false
    (add-to-agenda! time action agenda)
    (first-item agenda) -> action
    (remove-first-item! agenda)

Data structure:

    agenda (header cell)
      -> list of segments (ordered by time)
          -> (time, queue of actions) pairs

Queue primitives

    (make-queue) -> new queue
    (insert-queue! queue item)
    (delete-queue! queue)
    (front-queue queue)
    (empty-queue? queue)

Queue data structure:

    (front, rear)
       |      L__________________
       V                         V
    (item, next) (item, next) (item, )

We need two new primitive to manage the `(front, rear)` pointers

    (set-car! pair new-value)
    (set-cdr! pair new-value)

Original axioms about cons were:

    ∀x,y (car (cons x y)) = x
         (cdr (cons x y)) = y

But with mutation, we have to take **identity** into account.

    (define a (cons 1 2))
    (define b (cons a a))

There's now 3 aliases for the same pair object `a` (`a`, `(car b)` and `(cdr b)`)

    (set-car! (car b) 3)
    (car a) => 3 !!!

Sometimes it is what we want but inadvertent sharing is a liability.
It gives us a lot of power but we pay for it: increase of complexity and bugs.

    (define (cons x y)
      (lambda (m) (m x y)))

    (define (car x)
      (x lambda (a d) a)))

    (define (cdr x)
      (x lambda (a d) d)))

> “Alonzo Church was the greats programmer of the 20th Century although he never saw a computer”

Example:

    ;; Alonzo Church's Hack
    (car (cons 35 47))
    (car (lambda (m) (m 35 47)))
    ((lambda (m) (m 35 47)) (lambda (a d) a))
    (((lambda a d) a) 35 47)
    35

"Lambda Calculus" Mutable Data

    (define (cons x y)
      (lambda (m)
        (m x
           y
           (lambda (n) (set! x n))  ;; Permission to set x to n
           (lambda (n) (set! y n))  ;; Permission to set y to n
           )))

    (define (car x)
      (x (lambda (a d set-a set-d) a)))

    (define (cdr x)
      (x (lambda (a d set-a set-d) d)))

    (define (set-car! x y)
      (x (lambda (a d set-a set-d) (set-a y))))

    (define (set-car! x y)
      (x (lambda (a d set-a set-d) (set-a y))))

    (define (set-cdr! x y)
      (x (lambda (a d set-a set-d) (set-d y))))