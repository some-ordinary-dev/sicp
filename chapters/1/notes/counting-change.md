# Exercise: Counting Change

Tree-recursive solution

```scheme
(define (count-coins amount)
    (cc amount 5))

(define (cc amount kinds-of-coins)
    (cond ((= amount 0) 1)
          ((or (< amount 0) (= kinds-of-coins 0)) 0)
          (else (+ 
            (cc amount (- kinds-of-coins 1)) 
            (cc (- amount (first-denomination kinds-of-coins)) kinds-of-coins)))))

(define (first-denomination kinds-of-coins)
    (cond ((= kinds-of-coins 1) 1)
          ((= kinds-of-coins 2) 5)
          ((= kinds-of-coins 3) 10)
          ((= kinds-of-coins 4) 25)
          ((= kinds-of-coins 5) 50)))
```

Can we turn this into an iterative process?

If we think about what process this function will actually take, it looks like the following

```scheme
; should be 4 [(10), (5, 5), (5, 1), (1)]
(count-coins 10)
(cc 10 5)
(+ (cc 10 4) (cc -40 5))
(+ (+ (cc 10 3) (cc -15 4)) 0)
(+ (+ (+ (cc 10 2) (cc 0 3)) 0) 0)
(+ (+ (+ (+ (cc 10 1) (cc 5 2)) 1) 0) 0)
(+ (+ (+ (+ (+ (cc 10 0) (cc 9 1)) (+ (cc 5 1) (cc 0 2))) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ (cc 9 0) (cc 8 1))) (+ (+ (cc 5 0) (cc 4 1)) 1)) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ 0 (+ (cc 8 0) (cc 7 1)))) (+ (+ 0 (+ (cc 4 0) (cc 3 1))) 1)) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ 0 (+ 0 (+ (cc 7 0) (cc 6 1))))) (+ (+ 0 (+ 0 (+ (cc 3 0) (cc 2 1)))) 1)) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ (cc 6 0) (cc 5 1)))))) (+ (+ 0 (+ 0 (+ 0 (+ (cc 2 0) (cc 1 1))))) 1)) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ (cc 5 0) (cc 4 1))))))) (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ (cc 1 0) (cc 0 1)))))) 1)) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ (cc 4 0) (cc 3 1)))))))) (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 1))))) 1)) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ (cc 3 0) (cc 2 1))))))))) (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 1))))) 1)) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ (cc 2 0) (cc 1 1)))))))))) (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 1))))) 1)) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ (cc 1 0) (cc 0 1))))))))))) (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 1))))) 1)) 1) 0) 0)
(+ (+ (+ (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 1)))))))))) (+ (+ 0 (+ 0 (+ 0 (+ 0 (+ 0 1))))) 1)) 1) 0) 0)
4
```

If we want to change this to be an iterative process let's first think about other iterative processes.

For example, here is the fibonacci sequence in an iterative and recursive process.

```scheme
; iterative
(define (fib-iter a b count)
    (if (= count 0)
        b
        (fib-iter (+ a b) a (- count 1))))

(define (fib-iterative count)
    (fib-iter 1 0 count))

; recursive
(define (fib-recurse count)
    (cond ((= count 0) 0)
          ((= count 1) 1)
          (else (+ (fib-recurse (- count 1)) (fib-recurse (- count 2))))))
```

After examining both the recursive and iterative processes for the fibonacci sequence we can determine the following characteristics of an iterative process
1. Iterative processes end with a recursive call (tail-recursive).
1. Iterative processes end with a _single_ recursive call.
1. Iterative processes pass a running total as a parameter.

If we compare the recursive fibonacci sequence to the recursive cc procedure we can spot some similarities

```scheme
(define (fib-recurse count)
; defines degenerate cases first
    (cond ((= count 0) 0)
          ((= count 1) 1)
; ends with an addition of two recursive calls
          (else (+ (fib-recurse (- count 1)) (fib-recurse (- count 2))))))

(define (cc amount kinds-of-coins)
; defines degenerate cases first
    (cond ((= amount 0) 1)
          ((or (< amount 0) (= kinds-of-coins 0)) 0)
; ends with an addition of two recursive calls
          (else (+ 
            (cc amount (- kinds-of-coins 1)) 
            (cc (- amount (first-denomination kinds-of-coins)) kinds-of-coins)))))
```

We can then look at the transformation from the recursive fibonacci sequence to the iterative process and attempt to recognize parallels in the cc procedure

```scheme
(define (fib-recurse count)
    (cond ((= count 0) 0)
          ((= count 1) 1)
          (else (+ (fib-recurse (- count 1)) (fib-recurse (- count 2))))))

; instead of passing a single count parameter we now pass two accumulative values, which allows us to remove the addition 
; of the two recursive calls in the recursive process.
(define (fib-iter a b count)
    (if (= count 0)
        b
        (fib-iter (+ a b) a (- count 1))))
```

For clarity and as a refresher let's restate the cc procedure.

> The count of different ways that any amount of change can be made with an amount of denominations
> can be defined as the sum of the ways to make change using the current denomination and the number of ways to make 
> change using anything but the current denomination

For example, the number of ways to make change for 25 cents is equal to the number of ways to make change using a 25 cent coin plus the number of ways to make change using anything but the 25 cent coin,
which is the number of ways that you could make change using a 10 cent coin plus the number of ways to make change using anything but the 10 cent coin, ...

In the recursive example we realize this by starting at the maximum denomination and working our way down to the lowest denomination, short-circuiting when the denominiation is greater than the amount available.

If we want to do this iteratively, though, we have to start at the bottom and accumulate the number of ways to make change as we go up in denominations.


Another characteristic of the iterative process is that we pass as parameter the current answer and the next answer, using the current answer when needing to terminate
and passing the next answer as the current answer when we can continue iterating.

Thinking about that statement brings to be think that the iterative process for the change count procedure should pass the current number of ways to make change 
as well as the current number of ways to make change plus 1 (which would be the next answer.)

The issue is that I don't see a way around this without backtracking. Which maybe is okay? Or maybe we can get around by persisting running totals per denomination?

For example, if we're counting the ways to make change for 10 cents (the answer is 4) we could do something like this

- Start with amount = 10, base = 0, num-dimes = 0, num-nickels = 0
- Increment amount by 5
- Calculate new values: amount = 10, base = 5, num-dimes = 0, num-nickels = 1
- Increment amount by 5
- Calculate new values: amount = 10, base = 10, num-dimes = 1, num-nickels = 2
- End, because we're at base = amount
- Add 1, to account for all pennies
- End with num-dimes = 1, num-nickels = 2, num-pennies = 1 -> 4

Let's see what's going on here. 

We take a bottom-up approach, starting at 0 and making our way to 10 (the amount), incrementing by 5 each step (5 is our base amount, as incrementing by pennies is redundant.)
At each step we see if any more denominations can fit into the new base amount, using the previous number of denominations multiplied by their value. 
This is where we can determine if we can increment a certain denomination by 1 or not.

To double check our theory let's try counting change for 15 cents

- Start with amount = 15, base = 0, num-dimes = 0, num-nickels = 0
- Increment base by 5
- Calculate new values: amount = 15, base = 5, num-dimes = 0, num-nickels = 1
- Increment base by 5
- Calculate new values: amount = 15, base = 10, num-dimes = 1, num-nickels = 2
- Increment base by 5
- Calculate new values: amount = 15, base = 15, num-dimes = 1, num-nickels = 3
- End, due to base = amount
- Add 1, for pennies
- End with num-dimes = 1, num-nickels = 3, num-pennies = 1 -> 5

Not quite... the actual answer is 6, not 5. I think our issue here is that we're not accounting for the combination of 1 dime and 5 pennies. 
We should be able to solve this by adding another parameter for pennie fill-up that is incremented for each denomination that can't fill up the amount completely.

For example, with 25 this time (answer 13)

1. amount = 25, base = 0, num-quarters = 0, num-dimes = 0, num-nickels = 0
1. base + 5
1. amount = 25, base = 5, num-quarters = 0, num-dimes = 0, num-nickels = 1
1. base + 5
1. amount = 25, base = 10, num-quarters = 0, num-dimes = 1, num-nickels = 2
1. base + 5
1. amount = 25, base = 15, num-quarters = 0, num-dimes = 1, num-nickels = 3
1. base + 5
1. amount = 25, base = 20, num-quarters = 0, num-dimes = 2, num-nickels = 4
1. base + 5
1. amount = 25, base = 25, num-quarters = 1, num-dimes = 2, num-nickels = 5
1. end

```scheme
; amount = 40, original-amount = 40, curr-kinds-of-coins = 5, kinds-of-coins = 5
; amount = 40, original-amount = 40, curr-kinds-of-coins = 4, kinds-of-coins = 4
; amount = 15, original-amount = 40, curr-kinds-of-coins = 4, kinds-of-coins = 4
; amount = 15, original-amount = 40, curr-kinds-of-coins = 4, kinds-of-coins = 4
; amount = 15, original-amount = 40, curr-kinds-of-coins = 4, kinds-of-coins = 4
```

Let's think about the ways to make 40 cents
```scheme
; check if 50 cents can be made into 40 -> false -> decrement denomination
; validate that 10 cents can be made into 40 -> true -> decrement amoun

```

```scheme
; amount = 10
; amount = 10, run = 0, acc = 0, kinds-of-coins = 1 
;    -> add 1 to acc because the (> (- amount (first-denomination 1)) 0)
;    -> increment 


(define (cc amount)
    (cc-iter amount 1))

(define (cc-iter amount amt2 acc kinds-of-coins)
    (if (= kinds-of-coins 6)
        acc
        (cc-iter amount (- amt2 (first-denomination kinds-of-coins)))


    (cc-iter amount (+ acc 1) (+ kinds-of-coins (if (> (- amount (first-denomination kinds-of-coins)) 0) 0 1))
    (cond ((= amount 0) 1)
          ((= kinds-of-coins 6)) 0)
          (else (cc-iter amount (+ acc 1) (

```



Yikes! That's a lot of expansion just for 4...

What if instead of starting from the top denomination we started from the bottom??

In that case we would start counting at the first denomination, then when we reach the next denomination we could count that as another configuration.

For example, if we tried to determine `(count-coins 5)` (should result in 2, [(5), (1)]) we could start counting with 0, incrementing by 1 until we reach 5, which is the next available denomination
that is less than the total amount. We would continue doing this until we reach our original amount. 
We are able to do this because we know that every amount (except 0) will have at least 1 way to count coins: completely with pennies.

With this model we would need 
- the original amount
- a running total, that starts at 0 and ends at the amount, incremented by 1
- the count of denomination combinations that have been counted

