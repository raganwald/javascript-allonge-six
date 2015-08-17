## Maybe {#maybe}

A common problem in programming is checking for `null` or `undefined` (hereafter called "nothing," while all other values including `0`, `[]` and `false` will be called "something"). Languages like JavaScript do not strongly enforce the notion that a particular variable or particular property be something, so programs are often written to account for values that may be nothing.

This recipe concerns a pattern that is very common: A function `fn` takes a value as a parameter, and its behaviour by design is to do nothing if the parameter is nothing:

    const isSomething = (value) =>
      value !== null && value !== void 0;

    const checksForSomething = (value) => {
      if (isSomething(value)) {
        // function's true logic
      }
    }

Alternately, the function may be intended to work with any value, but the code calling the function wishes to emulate the behaviour of doing nothing by design when given nothing:

    var something =
      isSomething(value)
        ? doesntCheckForSomething(value)
        : value;

Naturally, there's a function decorator recipe for that, borrowed from Haskell's [maybe monad][maybe], Ruby's [andand], and CoffeeScript's existential method invocation:

    const maybe = (fn) =>
      function (...args) {
        if (args.length === 0) {
          return
        }
        else {
          for (let arg of args) {
            if (arg == null) return;
          }
          return fn.apply(this, args)
        }
      }

`maybe` reduces the logic of checking for nothing to a function call:

    maybe((a, b, c) => a + b + c)(1, 2, 3)
      //=> 6

    maybe((a, b, c) => a + b + c)(1, null, 3)
      //=> undefined

As a bonus, `maybe` plays very nicely with instance methods, we'll discuss those [later](#classes):

    function Model () {};

    Model.prototype.setSomething = maybe(function (value) {
      this.something = value;
    });

If some code ever tries to call `model.setSomething` with nothing, the operation will be skipped.

[andand]: https://github.com/raganwald/andand
[maybe]: https://en.wikipedia.org/wiki/Monad_(functional_programming)#The_Maybe_monad
