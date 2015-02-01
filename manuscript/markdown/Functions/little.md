## As Little As Possible About Functions, But No Less

In JavaScript, functions are values, but they are also much more than simple numbers, strings, or even complex data structures like trees or maps. Functions represent computations to be performed. Like numbers, strings, and arrays, they have a representation. Let's start with the second simplest possible function.[^simplest] In JavaScript, it looks like this:

    () => 0
    
This is a function that is applied to no values and returns `0`. Let's verify that our function is a value like all others:

    (() => 0)
      //=> [Function]
      
What!? Why didn't it type back `() => 0` for us? This *seems* to break our rule that if an expression is also a value, JavaScript will give the same value back to us. What's going on? The simplest and easiest answer is that although the JavaScript interpreter does indeed return that value, displaying it on the screen is a slightly different matter. `[Function]` is a choice made by the people who wrote Node.js, the JavaScript environment that hosts the JavaScript REPL. If you try the same thing in a browser, you may see something else.

[^simplest]: The simplest possible function is `() => {}`, we'll see that later.

{pagebreak}

A> I'd prefer something else, but I must accept that what gets typed back to us on the screen is arbitrary, and all that really counts is that it is somewhat useful for a human to read. But we must understand that whether we see `[Function]` or `() => 0`, internally JavaScript has a full and proper function.

### functions and identities

You recall that we have two types of values with respect to identity: Value types and reference types. Value types share the same identity if they have the same contents. Reference types do not.

Which kind are functions? Let's try them out and see. For reasons of appeasing the JavaScript parser, we'll enclose our functions in parentheses:

    (() => 0) === (() => 0)
      //=> false
      
Like arrays, every time you evaluate an expression to produce a function, you get a new function that is not identical to any other function, even if you use the same expression to generate it. "Function" is a reference type.

### applying functions

Let's put functions to work. The way we use functions is to *apply* them to zero or more values called *arguments*. Just as `2 + 2` produces a value (in this case `4`), applying a function to zero or more arguments produces a value as well.

Here's how we apply a function to some values in JavaScript: Let's say that *fn_expr* is an expression that when evaluated, produces a function. Let's call the arguments *args*. Here's how to apply a function to some arguments:

  *fn_expr*`(`*args*`)`
    
Right now, we only know about one such expression: `() => 0`, so let's use it. We'll put it in parentheses[^ambiguous] to keep the parser happy, like we did above: `(() => 0)`. Since we aren't giving it any arguments, we'll simply write `()` after the expression. So we write:

    (() => 0)()
      //=> 0


[^ambiguous]: If you're used to other programming languages, you've probably internalized the idea that sometimes parentheses are used to group operations in an expression like math, and sometimes to apply a function to arguments. If not... Welcome to the [ALGOL] family of programming languages!

[ALGOL]: https://en.wikipedia.org/wiki/ALGOL

### functions that return values and evaluate expressions

We've seen `() => 0`. We know that `(() => 0)()` returns `0`, and this is unsurprising. Likewise, the following all ought to be obvious:

    (() => 1)()
      //=> 1
    (() => "Hello, JavaScript")()
      //=> "Hello, JavaScript"
    (() => Infinity)()
      //=> Infinity

Well, the last one's a doozy, but still, the general idea is this: We can make a function that returns a value by putting the value to the right of the arrow.

In the prelude, we looked at expressions. Values like `0` are expressions, as are things like `40 + 2`. Can we put an expression to the right of the arrow?

    (() => 1 + 1)()
      //=> 2
    (() => "Hello, " + "JavaScript")()
      //=> "Hello, JavaScript"
    (() => Infinity * Infinity)()
      //=> Infinity

Yes we can. We can put any expression to the right of the arrow. For example, `(() => 0)()` is an expression. Can we put it to the right of an arrow, like this: `() => (() => 0)()`?

Let's try it:

    (() => (() => 0)())()
      //=> 0

Yes we can! Functions can return the value of evaluating another function.

When dealing with expressions that have a lot of the same characters (like parentheses), you may find it helpful to format the code to make things stand out. So we can also write:


    (() =>
        (() => 0
          )()
      )()
      //=> 0

It evaluates to the same thing, `0`.

### commas

The comma operator in JavaScript is interesting. It takes two arguments, evaluates them both, and itself evaluates to the value of the right-hand argument. In other words:

    (1, 2)
      //=> 2

    (1 + 1, 2 + 2)
      //=> 4

We can use commas with functions to create functions that evaluate multiple expressions:

    (() => (1 + 1, 2 + 2))()
      //=> 4

This is useful when trying to do things that might involve *side-effects*, but we'll get to that later. In most cases, JavaScript does not care whether things are separated by spaces, tabs, or line breaks. So we can also write:

    () =>
      (1 + 1, 2 + 2)

Or even:

    () => (
        1 + 1,
        2 + 2
      )


### the simplest possible block

There's another thing we can put to the right of an arrow, a *block*. A block has zero or more *statements*, separated by semicolons.[^asi]

[^asi]: Sometimes, you will find JavaScript that has statements that are separated by newlines without semi-colons. This works because JavaScript has a feature that can infer where the semi-colons should be most of the time. We will not take advantage of this feature, but it's helpful to know it exists.

So, this is a valid function:

    () => {}
    
It returns the result of evaluating a block that has no statements. What would that be? Let's try it:

    (() => {})()
      //=> undefined

What is this `undefined`?

### `undefined`

In JavaScript, the absence of a value is written `undefined`, and it means there is no value. It will crop up again. `undefined` is its own type of value, and it acts like a value type:

    undefined
      //=> undefined

Like numbers, booleans and strings, JavaScript can print out the value `undefined`.

    undefined === undefined
      //=> true
    (() => {})() === (() => {})()
      //=> true
    (() => {})() === undefined
      //=> true
      
No matter how you evaluate `undefined`, you get an identical value back. `undefined` is a value that means "I don't have a value." But it's still a value :-)
      
A> You might think that `undefined` in JavaScript is equivalent to `NULL` in SQL. No. In SQL, two things that are `NULL` are not equal to nor share the same identity, because two unknowns can't be equal. In JavaScript, every `undefined` is identical to every other `undefined`.

### void

We've seen that JavaScript represents an undefined value by typing `undefined`, and we've generated undefined values in two ways:

1. By evaluating a function that doesn't return a value `(() => {})()`, and;
2. By writing `undefined` ourselves.

There's a third way, with JavaScript's `void` operator. Behold:

    void 0
      //=> undefined
    void 1
      //=> undefined
    void (2 + 2)
      //=> undefined
      
`void` is an operator that takes any value and evaluates to `undefined`, always. So, when we deliberately want an undefined value, should we use the first, second, or third form?[^fourth] The answer is, use `void`. By convention, use `void 0`.

The first form works but it's cumbersome. The second form works most of the time, but it is possible to break it by reassigning `undefined` to a different value, something we'll discuss in [Reassignment and Mutation](#reassignment). The third form is guaranteed to always work, so that's what we will use.[^void]

[^fourth]: Experienced JavaScript programmers are aware that there's a fourth way, using a function argument. This was actually the preferred mechanism until `void` became commonplace.

[^void]: As an exercise for the reader, we suggest you ask your friendly neighbourhood programming language designer or human factors subject-matter expert to explain why a keyword called `void` is used to generate an `undefined` value, instead of calling them both `void` or both `undefined`. We have no idea.

### back on the block

Back to our function. We evaluated this:

    (() => {})()
      //=> undefined

We said that the function returns the result of evaluating a *block*, and we said that a block is a (possibly empty) list of JavaScript *statements* separated by semicolons.[^break]

[^break]: You can also separate statements with line breaks. Readers who follow internet flame-fests may be aware of something called [automatic semi-colon insertion](http://lucumr.pocoo.org/2011/2/6/automatic-semicolon-insertion/). Basically, there's a step where JavaScript looks at your code and follows some rules to guess where you meant to put semicolons in should you leave them out. This feature was originally created as a kind of helpful error-correction. Some programmers argue that since it's part of the language's definition, it's fair game to write code that exploits it, so they deliberately omit any semicolon that JavaScript will insert for them.

Something like: `{` statement^1^`;` statement^2^`;` statement^3^`; ... ;` statement^n^ `}`

We haven't discussed these *statements*. What's a statement?

[^todonamed]: TODO: Named functions, probably discussed in a whole new section when we discuss `var` hoisting.

There are many kinds of JavaScript statements, but the first kind is one we've already met. An expression is a JavaScript statement. Although they aren't very practical, these are valid JavaScript functions, and they return `undefined` when applied:

    () => { 2 + 2 }
    () => { 1 + 1; 2 + 2 }
    
As we saw with commas above, we can rearrange these functions onto multiple lines when we feel its more readable that way:

    () => {
        1 + 1;
        2 + 2
      }

But no matter how we arrange them, a block with one or more expressions still evaluates to `undefined`:

    (() => { 2 + 2 })()
      //=> undefined
      
    (() => { 1 + 1; 2 + 2 })()
      //=> undefined
      
    (() => {
        1 + 1;
        2 + 2
      })()
      //=> undefined

As you can see, a block with one expression does not behave like an expression, and a block with more than one expression does not behave like an expression constructed with the comma operator:


    (() => 2 + 2)()
      //=> 4
    (() => { 2 + 2 })()
      //=> undefined
      
    (() => (1 + 1, 2 + 2))()
      //=> 4
    (() => { 1 + 1; 2 + 2 })()
      //=> undefined

So how do we get a function that evaluates a block to return a value when applied? With the `return` keyword and any expression:

    (() => { return 0 })()
      //=> 0
      
    (() => { return 1 })()
      //=> 1
      
    (() => { return 'Hello ' + 'World' })()
      // 'Hello World'
      
The `return` keyword creates a *return statement* that immediately terminates the function application and returns the result of evaluating its expression. For example:
      
    (() => {
        1 + 1;
        return 2 + 2
      })()
      //=> 4

And also:
      
    (() => {
        return 1 + 1;
        2 + 2
      })()
      //=> 2

The return statement is the first statement we've seen, and it behaves differently than an expression. For example, you can't use one as the expression in a simple function, because it isn't an expression:

    (() => return 0)()
      //=> ERROR

Statements belong inside blocks and only inside blocks. Some languages simplify this by making everything an expression, but JavaScript maintains this distinction, so when learning JavaScript we also learn about statements like function declarations, for loops, if statements, and so forth. We'll see a few more of these later.

### functions that evaluate to functions

If an expression that evaluates to a function is, well, an expression, and if a return statement can have any expression on its right side... *Can we put an expression that evaluates to a function on the right side of a function expression?*

Yes:

    () => () => 0
    
That's a function! It's a function that when applied, evaluates to a function that when applied, evaluates to `0`.  So we have *a function, that returns a function, that returns zero*. Likewise:

    () => () => true
    
That's a function, that returns a function, that returns `true`:

    (() => () => true)()()
      //=> true
      
We could, of course, do the same thing with a block if we wanted:

    () => () => { return true; }

But we generally don't.

---

Well. We've been very clever, but so far this all seems very abstract. Diffraction of a crystal is beautiful and interesting in its own right, but you can't blame us for wanting to be shown a practical use for it, like being able to determine the composition of a star millions of light years away. So... In the next chapter, "[I'd Like to Have an Argument, Please](#fargs)," we'll see how to make functions practical.
