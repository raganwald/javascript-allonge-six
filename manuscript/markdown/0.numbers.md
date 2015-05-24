# A Rich Aroma: Basic Numbers

![Mathematics and Coffee](images/expressions-title.jpg)

> In computer science, a literal is a notation for representing a fixed value in source code. Almost all programming languages have notations for atomic values such as integers, floating-point numbers, and strings, and usually for booleans and characters; some also have notations for elements of enumerated types and compound values such as arrays, records, and objects. An anonymous function is a literal for the function type.—[Wikipedia](https://en.wikipedia.org/wiki/Literal_(computer_programming))

JavaScript, like most languages, has a collection of literals. We saw that an expression consisting solely of numbers, like `42`, is a literal. It represents the number forty-two, which is 42 base 10. Not all numbers are base ten. If we start a literal with a zero, it is an octal literal. So the literal `042` is 42 base 8, which is actually 34 base 10.

Internally, both `042` and `34` have the same representation, as [double-precision floating point] numbers. A computer's internal representation for numbers is important to understand. The machine's representation of a number almost never lines up perfectly with our understanding of how a number behaves, and thus there will be places where the computer's behaviour surprises us if we don't know a little about what it's doing "under the hood." 

[double-precision floating point]:http://en.wikipedia.org/wiki/Double-precision_floating-point_format

For example, the largest integer JavaScript can safely[^safe] handle is `9007199254740991`, or `2`^`53`^`- 1`. Like most programming languages, JavaScript does not allow us to use commas to separate groups of digits. 

[^safe]: Implementations of JavaScript are free to handle larger numbers. For example, if you type `9007199254740991 + 9007199254740991` into `node.js`, it will happily report that the answer is `18014398509481982`. But code that depends upon numbers larger than `9007199254740991` may not be reliable when moved to other implementations.

### floating

Most programmers never encounter the limit on the magnitude of an integer. But we mentioned that numbers are represented internally as floating point, meaning that they need not be just integers. We can, for example, write `1.5` or `33.33`, and JavaScript represents these literals as floating point numbers.

It's tempting to think we now have everything we need to do things like handle amounts of money, but as the late John Belushi would say, "Nooooooooooooooooooooo." A computer's internal representation for a floating point number is binary, while our literal number was in base ten. This makes no meaningful difference for integers, but it does for fractions, because some fractions base 10 do not have exact representations base 2.

One of the most oft-repeated examples is this:

    1.0
      //=> 1
    1.0 + 1.0
      //=> 2
    1.0 + 1.0 + 1.0
      //=> 3

However:

    0.1
      //=> 0.1
    0.1 + 0.1
      //=> 0.2
    0.1 + 0.1 + 0.1
      //=> 0.30000000000000004
      
This kind of "inexactitude" can be ignored  when performing calculations that have an acceptable deviation. For example, when centering some text on a page, as long as the difference between what you might calculate longhand and JavaScript's calculation is less than a pixel, there is no observable error.

But as a rule, if you need to work with real numbers, you should have more than a nodding acquaintance with the [IEEE Standard for Floating-Point Arithmetic][IEEE754]. Professional programmers almost never use floating point numbers to represent monetary amounts. For example, "$43.21" will nearly always be presented as two numbers: `43` for dollars and `21` for cents, not `43.21`. In this book, we need not think about such details, but outside of this book, we must.

[IEEE754]: https://en.wikipedia.org/wiki/IEEE_floating_point

### operations on numbers

As we've seen, JavaScript has many common arithmetic operators. We can create expressions that look very much like mathematical expressions, for example we can write `1 + 1` or `2 * 3` or `42 - 34` or even `6 / 2`. These can be combined to make more complex expressions, like `2 * 5 + 1`.

In JavaScript, operators have an order of precedence designed to mimic the way humans typically parse written arithmetic. So:

    2 * 5 + 1
      //=> 11
    1 + 5 * 2
      //=> 11
      
JavaScript treats the expressions as if we had written `(2 * 5) + 1` and `1 + (5 * 2)`, because the `*` operator has a *higher precedence* than the `+` operator. JavaScript has many more operators. In a sense, they behave like little functions. If we write `1 + 2`, this is conceptually similar to writing `plus(1, 2)` (assuming we have a function that adds two numbers bound to the name `plus`, of course).

In addition to the common `+`, `-`, `*`, and `/`, JavaScript also supports modulus, `%`, and unary negation, `-`:

    -(457 % 3)
      //=> -1

There are lots and lots more operators that can be used with numbers, including bitwise operators like `|` and `&` that allow you to operate directly on a number's binary representation, and a number of other operators that perform assignment or logical comparison that we will look at later.