## Naming Functions {#named-function-expressions}

Let's get right to it. This code does *not* name a function:

    const repeat = (str) => str + str

It doesn't name the function "repeat" for the same reason that `const answer = 42` doesn't name the number `42`. This syntax binds an anonymous function to a name in an environment, but the function itself remains anonymous.

### the `function` keyword

JavaScript *does* have a syntax for naming a function, we use the `function` keyword. Until ECMAScript 2015 was created, `function` was the usual syntax for writing functions.

Here's our `repeat` function written using a "fat arrow"

    (str) => str + str

And here's (almost) the exact same function written using the `function` keyword:

    function (str) { return str + str }

Let's look at the obvious differences:

1. We introduce a function with the `function` keyword.
1. Something else we're about to discuss is optional.
1. We have arguments in parentheses, just like fat arrow functions.
1. We do not have a fat arrow, we go directly to the body.
1. We always use a block, we cannot write `function (str) str + str`. This means that if we want our functions to return a value, we always need to use the `return` keyword

If we leave out the "something optional" that comes after the `function` keyword, we can translate all of the fat arrow functions that we've seen into `function` keyword functions, e.g.

    (n) => (1.618**n - -1.618**-n) / 2.236

Can be written as:

    function (n) {
      return (1.618**n - -1.618**-n) / 2.236;
    }

This still does not *name* a function, but as we noted above, functions written with the `function` keyword have an optional "something else." Could that "something else" name a function? Yes, of course.[^ofcourse]

[^ofcourse]: "Yes of course?" Well, in chapter of a book dedicated to naming functions, it is not surprising that feature we mention has something to do with naming functions.

Here are our example functions written with names:

    const repeat = function repeat (str) {
      return str + str;
    };

    const fib = function fib (n) {
      return (1.618**n - -1.618**-n) / 2.236;
    };

Placing a name between the `function` keyword and the argument list names the function. Confusingly, the name of the function is not exactly the same thing as the name we may choose to bind to the value of the function. For example, we can write:

    const double = function repeat (str) {
      return str + str;
    }

In this expression, `double` is the name in the environment, but `repeat` is the function's actual name. This is a *named function expression*. That may seem confusing, but think of the binding names as properties of the environment, not of the function. While the name of the function is a property of the function, not of the environment.

And indeed the name *is* a property:

    double.name
      //=> 'repeat'

In this book we are not examining JavaScript's tooling such as debuggers baked into browsers, but we will note that when you are navigating call stacks in all modern tools, the function's binding name is ignored but its actual name is displayed, so naming functions is very useful even if they don't get a formal binding, e.g.

    someBackboneView.on('click', function clickHandler () {
      //...
    });

Now, the function's actual name has no effect on the environment in which it is used. To whit:

    const bindingName = function actualName () {
      //...
    };

    bindingName
      //=> [Function: actualName]

    actualName
      //=> ReferenceError: actualName is not defined

So "actualName" isn't bound in the environment where we use the named function expression. Is it bound anywhere else? Yes it is. Here's a function that determines whether a positive integer is even or not. We'll use it in an IIFE so that we don't have to bind it to a name with `const`:

    (function even (n) {
      if (n === 0) {
        return true
      }
      else return !even(n - 1)
    })(5)
      //=> false

    (function even (n) {
      if (n === 0) {
        return true
      }
      else return !even(n - 1)
    })(2)
      //=> true

Clearly, the name `even` is bound to the function *within the function's body*. Is it bound to the function outside of the function's body?

    even
      //=> Can't find variable: even

`even` is bound within the function itself, but not outside it. This is useful for making recursive functions as we see above, and it speaks to the principle of least privilege: If you don't *need* to name it anywhere else, you needn't.

### function declarations {#function-declarations}

There is another syntax for naming and/or defining a function. It's called a *function declaration statement*, and it looks a lot like a named function expression, only we use it as a statement:

    function someName () {
      // ...
    }

This behaves a *little* like:

    const someName = function someName () {
      // ...
    }

In that it binds a name in the environment to a named function. However, there are two important differences. First, function declarations are *hoisted* to the top of the function in which they occur.

Consider this example where we try to use the variable `fizzbuzz` as a function before we bind a function to it with `const`:

    (function () {
      return fizzbuzz();

      const fizzbuzz = function fizzbuzz () {
        return "Fizz" + "Buzz";
      }
    })()
      //=> undefined is not a function (evaluating 'fizzbuzz()')

We haven't actually bound a function to the name `fizzbuzz` before we try to use it, so we get an error. But a function *declaration* works differently:

    (function () {
      return fizzbuzz();

      function fizzbuzz () {
        return "Fizz" + "Buzz";
      }
    })()
      //=> 'FizzBuzz'

Although `fizzbuzz` is declared later in the function, JavaScript behaves as if we'd written:

    (function () {
      const fizzbuzz = function fizzbuzz () {
        return "Fizz" + "Buzz";
      }

      return fizzbuzz();
    })()

The definition of the `fizzbuzz` is "hoisted" to the top of its enclosing scope (an IIFE in this case). This behaviour is intentional on the part of JavaScript's design to facilitate a certain style of programming where you put the main logic up front, and the "helper functions" at the bottom. It is not necessary to declare functions in this way in JavaScript, but understanding the syntax and its behaviour (especially the way it differs from `const`) is essential for working with production code.

### function declaration caveats[^caveats]

Function declarations are formally only supposed to be made at what we might call the "top level" of a function. Although some JavaScript environments permit the following code, this example is technically illegal and definitely a bad idea:

    (function (camelCase) {
      return fizzbuzz();

      if (camelCase) {
        function fizzbuzz () {
          return "Fizz" + "Buzz";
        }
      }
      else {
        function fizzbuzz () {
          return "Fizz" + "Buzz";
        }
      }
    })(true)
      //=> 'FizzBuzz'? Or ERROR: Can't find variable: fizzbuzz?

Function declarations are not supposed to occur inside of blocks. The big trouble with expressions like this is that they may work just fine in your test environment but work a different way in production. Or it may work one way today and a different way when the JavaScript engine is updated, say with a new optimization.

Another caveat is that a function declaration cannot exist inside of *any* expression, otherwise it's a function expression. So this is a function declaration:

    function trueDat () { return true }

But this is not:

    (function trueDat () { return true })

The parentheses make this an expression, not a function declaration.

[^caveats]: A number of the caveats discussed here were described in Jyrly Zaytsev's excellent article [Named function expressions demystified](http://kangax.github.com/nfe/).
