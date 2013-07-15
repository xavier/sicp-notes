# SICP Lecture 3A
# Henderson Escher Example

*Harold Abelson*

## More on Vectors Representation

    (define (+vect v1 v2)
      (make-vector
        (+ (xcor v1) (xcor v2))
        (+ (ycor v1) (ycor v2))))

    (define (scale s v)
      (make-vector
        (* s (xcor v))
        (* s (ycor v))))


Actually no need for formal parameters, procedures can be aliased

    (define make-vector cons)
    (define xcor car)
    (define ycor cdr)

    (define make-segment cons)
    (define seg-start car)
    (define seg-end cdr)

Closures allow to build complexity.

## Lists

A list is **convention** to reprensent a sequence using a **chain of pairs**.

    ; [1, 2, 3, 4]
    (cons 1 (cons 2 (cons 3 (cons 4 nil))))

`nil` marks the end of the list

Syntactic sugar:

    (define 1-to-4 (list 1 2 3 4))

    (car (cdr 1-to-4)) -> 2
    (car (cdr (cdr 1-to-4))) -> 3
    (cdr (cdr 1-to-4)) -> (3, 4)
    (cdr (cdr (cdr (cdr 1-to-4)))) -> ()

    ; Scaling all items of a list
    (define (scale-list s l)
      (if (null? l)
        nil
        (cons (* s (car l)) (scale-list s (cdr l)))))

### Map

    ; Generalization
    (define (map p l)
      (if (null? l)
        nil
        (cons (p (car l)) (map p (cdr l)))))

    ; Redefined using map
    (define (scale-list s l)
      (map (lambda (x) (* s x)) l))

    ; Other examples
    (map square 1-to-4)
    (map (lambda (x) (+ x 10)) 1-to-4)

### For Each

Similar to map but does not return a new list

    (define (for-each proc list)
      (cond ((null? list) *done*)
            (else (proc (car list))
                  (for-each proc (cdr list)))))


## Henderson Escher Example

Primitive in the language: picture (image in a rectangle)
Means of combination:

* rotate
* flip
* beside and above (fits 2 pictures in a rectangle)

A rectangle is specified by 3 vectors:
* an origin
* the horizontal part of the rectangle
* the vertical part of the rectangle

    (define (make-rect o h v) (...))
    (define (horiz rect) (...))
    (define (vert rect) (...))
    (define (origin rect) (...))

A rectangle defines transformation from a unit square.

    (x,y) -> (x',y') = orig + x*horz.x + y*vert.y

    ; Returns a procedure which transform a point in the rectangle's referential
    (define (coord-map rect)
      (lambda (point)
        (+vect
          (+vect
            (scale (xcor point) (horiz rect))
            (scale (ycor point) (vert rect)))
          (origin rect))))

Constructing primitive pictures from lists of segments:

    (define (make-picture seglist)
      (lambda (rect)
        (for-each
          (lambda (seg)
            (drawline
              ((coord-map rect) (seg-start seg))
              ((coord-map rect) (seg-end seg))))
          seglist)))

How to use this:

    (define r (make-rect ...))
    (define g (make-picture ...))

    ; Draw picture g in rectangle r
    (g r)

With these abstractions in place, the combination possibilities fall out:

    (define (beside p1 p2 a)
      (lambda (rect)
        (p1 (make-rect
              (origin rect)
              (scale a (horiz rect))
              (vert rect)))
        (p2 (make-rect
          (+vect (origin rect) (scale a (horiz rect)))
          (scale (- 1 a) (horiz rect))
          (vert rect))))))

    (define p1-next-to-p1 (beside p1 p2 golden-ratio))
    (p1-next-to-p2 rect)



    (define (rotate90 pict)
      (lambda (rect)
        (pict (make-rect
          (+vect (origin rect) (horiz rect))
          (vert rect)
          (scale -1 (horiz rect))))))

We use the representation of pictures as procedure to benefit from the closure property.

The language is **embedded in Lisp** and we can thus benefit from all its constructs.

Example: build a procedure which puts 4 pictures in rectangle:

    (define (make-4-picture p1 p2 p3 p4)
      (lambda (rect)
        (beside
          (above p1 p3 0.5)
          (above p2 p4 0.5)
        0.5))))

Example of recursive operation

    (define (right-push p n a)
      (if (= n 0)
        p
        (beside p (right-push p (-1 n) a))))

    ; Generalization using a provided combinator
    (define (push comb)
      (lambda (pict n a)
        ((repeated
          (lambda (i) (comb pict i a))
          n)
          pict)))

    (define right-push (push beside))


There is no difference between data and procedure, the system is both and neither.

A sequence of layers of language:

  * language of schemes of combinations: (push, ...)
  * language of geometric positions
  * language of primitive (picts)

Design as **levels of languages** vs hierarchy of tasks
