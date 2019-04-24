## Reassignment {#reassignment}

Like some imperative programming languages, JavaScript allows you to re-assign the value bound to parameters. We saw this earlier in [rebinding](#rebinding-peek):

By default, JavaScript permits us to *rebind* new values to names bound with a parameter. For example, we can write:

    const evenStevens = (n) => {
      if (n === 0) {
        return true;
      }
      else if (n == 1) {
        return false;
      }
      else {
        n = n - 2;
        return evenStevens(n);
      }
    }

    evenStevens(42)
      //=> true

The line `n = n - 2;` *rebinds* a new value to the name `n`. We will discuss this at much greater length in [Reassignment](#reassignment), but long before we do, let's try a similar thing with a name bound using `const`. We've already bound `evenStevens` using `const`, let's try rebinding it:

    evenStevens = (n) => {
      if (n === 0) {
        return true;
      }
      else if (n == 1) {
        return false;
      }
      else {
        return evenStevens(n - 2);
      }
    }
      //=> ERROR, evenStevens is read-only

JavaScript does not permit us to rebind a name that has been bound with `const`. We can *shadow* it by using `const` to declare a new binding with a new function or block scope, but we cannot rebind a name that was bound with `const` in an existing scope.

Rebinding parameters is usually avoided, but what about rebinding names we declare within a function? What we want is a statement that works like `const`, but permits us to rebind variables. JavaScript has such a thing, it's called `let`:

{:lang="javascript"}
~~~~~~~~
let age = 52;

age = 53;
age
  //=> 53
~~~~~~~~

We took the time to carefully examine what happens with bindings in environments. Let's take the time to explore what happens with reassigning values to variables. The key is to understand that we are rebinding a different value to the same name in the same environment.

So let's consider what happens with a shadowed variable:

    (() => {
      let age = 49;

      if (true) {
        let age = 50;
      }
      return age;
    })()
      //=> 49

Using `let` to bind `50` to age within the block does not change the binding of `age` in the outer environment because the binding of `age` in the block shadows the binding of `age` in the outer environment, just like `const`. We go from:

    {age: 49, '..': global-environment}

To:

    {age: 50, '..': {age: 49, '..': global-environment}}

Then back to:

    {age: 49, '..': global-environment}

However, if we don't shadow `age` with `let`, reassigning within the block changes the original:

    (() => {
      let age = 49;

      if (true) {
        age = 50;
      }
      return age;
    })()
      //=> 50

Like evaluating variable labels, when a binding is rebound, JavaScript searches for the binding in the current environment and then each ancestor in turn until it finds one. It then rebinds the name in that environment.

### mixing `let` and `const`

Some programmers dislike deliberately shadowing variables. The suggestion is that shadowing a variable is confusing code. If you buy that argument, the way that shadowing works in JavaScript exists to protect us from accidentally shadowing a variable when we move code around.

If you dislike deliberately shadowing variables, you'll probably take an even more opprobrious view of mixing `const` and `let` semantics with a shadowed variable:

    (() => {
      let age = 49;

      if (true) {
        const age = 50;
      }
      age = 51;
      return age;
    })()
      //=> 51

Shadowing a `let` with a `const` does not change our ability to rebind the variable in its original scope. And:

    (() => {
      const age = 49;

      if (true) {
        let age = 50;
      }
      age = 52;
      return age;
    })()
      //=> ERROR: age is read-only

Shadowing a `const` with a `let` does not permit it to be rebound in its original scope.

### `var`

JavaScript has one *more* way to bind a name to a value, `var`.[^namecount]

[^namecount]: How many have we seen so far? Well, parameters bind names. Function declarations bind names. Named function expressions bind names. `const` and `let` bind names. So that's five different ways so far. And there are more!

`var` looks a lot like `let`:

{:lang="javascript"}
~~~~~~~~
const factorial = (n) => {
  let x = n;
  if (x === 1) {
    return 1;
  }
  else {
    --x;
    return n * factorial(x);
  }
}

factorial(5)
  //=> 120

const factorial2 = (n) => {
  var x = n;
  if (x === 1) {
    return 1;
  }
  else {
    --x;
    return n * factorial2(x);
  }
}

factorial2(5)
  //=> 120
~~~~~~~~

But of course, it's not exactly like `let`. It's just different enough to present a source of confusion. First, `var` is not block scoped, it's function scoped, just like [function declarations](#function-declarations):

{:lang="javascript"}
~~~~~~~~
(() => {
  var age = 49;

  if (true) {
    var age = 50;
  }
  return age;
})()
  //=> 50
~~~~~~~~

Declaring `age` twice does not cause an error(!), and the inner declaration does not shadow the outer declaration. All `var` declarations behave as if they were hoisted to the top of the function, a little like function declarations.

But, again, it is unwise to expect consistency. A function declaration can appear anywhere within a function, but the declaration *and* the definition are hoisted. Note this example of a function that uses a helper:

{:lang="javascript"}
~~~~~~~~
const factorial = (n) => {

  return innerFactorial(n, 1);

  function innerFactorial (x, y) {
    if (x == 1) {
      return y;
    }
    else {
      return innerFactorial(x-1, x * y);
    }
  }
}

factorial(4)
  //=> 24
~~~~~~~~

JavaScript interprets this code as if we had written:

{:lang="javascript"}
~~~~~~~~
const factorial = (n) => {
  let innerFactorial = function innerFactorial (x, y) {
      if (x == 1) {
        return y;
      }
      else {
        return innerFactorial(x-1, x * y);
      }
    }

  return innerFactorial(n, 1);
}
~~~~~~~~

JavaScript hoists the `let` and the assignment. But not so with `var`:

{:lang="javascript"}
~~~~~~~~
const factorial = (n) => {

  return innerFactorial(n, 1);

  var innerFactorial = function innerFactorial (x, y) {
    if (x == 1) {
      return y;
    }
    else {
      return innerFactorial(x-1, x * y);
    }
  }
}

factorial(4)
  //=> undefined is not a function (evaluating 'innerFactorial(n, 1)')
~~~~~~~~

JavaScript hoists the declaration, but not the assignment. It is as if we'd written:

{:lang="javascript"}
~~~~~~~~
const factorial = (n) => {

  let innerFactorial = undefined;

  return innerFactorial(n, 1);

  innerFactorial = function innerFactorial (x, y) {
    if (x == 1) {
      return y;
    }
    else {
      return innerFactorial(x-1, x * y);
    }
  }
}

factorial(4)
  //=> undefined is not a function (evaluating 'innerFactorial(n, 1)')
~~~~~~~~

In that way, `var` is a little like `const` and `let`, we should always declare and bind names before using them. But it's not like `const` and `let` in that it's function scoped, not block scoped.

### why `const` and `let` were invented

`const` and `let` are recent additions to JavaScript. For nearly twenty years, variables were declared with `var` (not counting parameters and function declarations, of course). However, its functional scope was a problem.

We haven't looked at it yet, but JavaScript provides a `for` loop for your iterating pleasure and convenience. It looks a lot like the `for` loop in C. Here it is with `var`:

    var sum = 0;
    for (var i = 1; i <= 100; i++) {
      sum = sum + i
    }
    sum
      //=> 5050

Hopefully, you can think of a faster way to calculate this sum.[^gauss] And perhaps you have noticed that `var i = 1` is tucked away instead of being at the top as we prefer. But is this ever a problem?

[^gauss]: There is a well known story about Karl Friedrich Gauss when he was in elementary school. His teacher got mad at the class and told them to add the numbers 1 to 100 and give him the answer by the end of the class. About 30 seconds later Gauss gave him the answer. The other kids were adding the numbers like this: `1 + 2 + 3 + . . . . + 99 + 100 = ?` But Gauss rearranged the numbers to add them like this: `(1 + 100) + (2 + 99) + (3 + 98) + . . . . + (50 + 51) = ?` If you notice every pair of numbers adds up to 101. There are 50 pairs of numbers, so the answer is 50*101 = 5050. Of course Gauss came up with the answer about 20 times faster than the other kids.

Yes. Consider this variation:

    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];

    for (var i = 0; i < 3; i++) {
      introductions[i] = "Hello, my name is " + names[i]
    }
    introductions
      //=> [ 'Hello, my name is Karl',
      //     'Hello, my name is Friedrich',
      //     'Hello, my name is Gauss' ]

So far, so good. Hey, remember that functions in JavaScript are values? Let's get fancy!

    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];

    for (var i = 0; i < 3; i++) {
      introductions[i] = (soAndSo) =>
        `Hello, ${soAndSo}, my name is ${names[i]}`
    }
    introductions
      //=> [ [Function],
      //     [Function],
      //     [Function] ]

Again, so far, so good. Let's try one of our functions:

    introductions[1]('Raganwald')
      //=> 'Hello, Raganwald, my name is undefined'

What went wrong? Why didn't it give us 'Hello, Raganwald, my name is Friedrich'? The answer is that pesky `var i`. Remember that `i` is bound in the surrounding environment, so it's as if we wrote:

    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'],
        i = undefined;

    for (i = 0; i < 3; i++) {
      introductions[i] = function (soAndSo) {
        return "Hello, " + soAndSo + ", my name is " + names[i]
      }
    }
    introductions

Now, at the time we created each function, `i` had a sensible value, like `0`, `1`, or `2`. But at the time we *call* one of the functions, `i` has the value `3`, which is why the loop terminated. So when the function is called, JavaScript looks `i` up in its enclosing environment (its  closure, obviously), and gets the value `3`. That's not what we want at all.

The error wouldn't exist at all if we'd used `let` in the first place

    let introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];

    for (let i = 0; i < 3; i++) {
      introductions[i] = (soAndSo) =>
        `Hello, ${soAndSo}, my name is ${names[i]}`
    }
    introductions[1]('Raganwald')
      //=> 'Hello, Raganwald, my name is Friedrich'

This small error was a frequent cause of confusion, and in the days when there was no block-scoped `let`, programmers would need to know how to fake it, usually with an IIFE:

    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];

    for (var i = 0; i < 3; i++) {
      ((i) => {
        introductions[i] = (soAndSo) =>
          `Hello, ${soAndSo}, my name is ${names[i]}`
        }
      })(i)
    }
    introductions[1]('Raganwald')
      //=> 'Hello, Raganwald, my name is Friedrich'

Now we're creating a new inner parameter, `i` and binding it to the value of the outer `i`. This works, but `let` is so much simpler and cleaner that it was added to the language in the ECMAScript 2015 specification.

In this book, we will use function declarations sparingly, and not use `var` at all. That does not mean that you should follow the exact same practice in your own code: The purpose of this book is to illustrate certain principles of programming. The purpose of your own code is to get things done. The two goals are often, but not always, aligned.
