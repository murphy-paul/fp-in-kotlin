# Introduction to Functional Programming

## What is functional programming

Functional programming is a [programming paradigm](https://en.wikipedia.org/wiki/Programming_paradigm) -
that is a style of programming that constructs applications from pure functions.

A pure function is one which has no side effect. Side effects can include:

* modifying a variable
* modifying a data structure in place
* setting a field on an object
* read or writing to a file.

This idea of a pure function can be formalized using the concept of [Referential_transparency](https://en.wikipedia.org/wiki/Referential_transparency).
Referential transparency mean that a function (or expression) always evaluates to the same
result in the same context.

Consider the following expression as an example: `2 + 3`. `+` in this case
is a pure function, and the evaluation of this expression results in the same value every time, `5`.

A benefit of referential transparency is that the expression, `2 + 3` in this example, can be replaced by its result without changing the program behaviour.

## A simple program with side effects

The below class `Cafe` implements an impure function, `buyCoffee`.

```
class Cafe {
    fun buyCoffee(cc: CreditCard): Coffee {
        val cup = Coffee(1.14)

        cc.charge(cup.price)

        return cup
    }
}

```

the line `cc.charge(cup.price)` is an example of a side effect. Charging a credit card involves some interaction with the outside world - suppose it requests contacting the credit card company via a web service, authorizing the transaction, charging the card, and if successful persisting some record of the transaction.

Our function merely returns a `Coffee` and these other actions are happening on the side.

## A functional solution: removing the side effects

The functional solution is to elimiate the side effects and have `buyCoffee` return the _charge as a value_ in addition to returning the coffee. The concerns of processing the charge by sending it off to the credit card company can be handled else where.

```
fun buyCoffee(cc: CreditCard): Pair<Coffee, Charge> {
    val coffee = Coffee(4.50)
    return Pair(coffee, Charge(cc, coffee.price))
}
```

`Charge` is a data type that was created for this program, which contains a `CreditCard` and an `amount` and also has a handy function, `combine` for combining charges with the same CreditCard.

```
data class Charge(val cc: CreditCard, val amount: Double) {

    fun combine(other: Charge): Charge {
        if (cc == other.cc) {
            return Charge(cc, amount + other.amount)
        } else {
            throw IllegalArgumentException("Can't combine charges to different cards")
        }
    }
}

```

`Charge` is a _Kotlin_ `Data Type` which we will cover later. For now what we can say is that Charge is an immutable type. The combine function takes another `Charge` as an argument, and if its credit card is equal then it will return a new Charge with the combined total.

With this function in hand we can now implement a batch `buyCoffees` method:

```
fun buyCoffees(cc: CreditCard, num: Int): Pair<List<Coffee>, Charge> {
    val (coffees, charges) = (1..num).map { buyCoffee(cc) }.unzip()

    return Pair(coffees, charges.reduce { x,y -> x.combine(y) })
}
```

This function is quite a bit more complex, especially for those new to Kotlin and functional programming, so lets break it down.
