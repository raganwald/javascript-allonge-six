## How to run the examples {#online}

At the time this book was written, EcmaScript-6 was not yet widely available. All of the examples in this book were tested using either [Google Traceur Compiler], [6to5], or both. Traceur and 6to5 are both *transpilers*, they work by parsing EcmaScript-6 code, then emitting valid EcmaScript-5 code that produces teh same semantics.

[Google Traceur Compiler]: https://github.com/google/traceur-compiler
[6to5]: http://6to5.org

For example, this EcmaScript-6 code:

    var before =
      (decoration) =>
        (method) =>
          function () {
            decoration.apply(this, arguments);
            return method.apply(this, arguments)
          };

Is translated into this EcmaScript-5 code:

    var before = function (decoration) {
      return function (method) {
        return function () {
          decoration.apply(this, arguments);
          return method.apply(this, arguments);
        };
      };
    };
    
![The 6to5 "try it out" page](images/6to5.png)
    
If we make it even more idiomatic, we could write:

    var before =
      (decoration) =>
        (method) =>
          function (...args) {
            decoration.apply(this, args);
            return method.apply(this, args)
          };
          
And it wold be "transpiled" into:

    var before = function (decoration) {
      return function (method) {
        return function () {
          for (var _len = arguments.length, args = Array(_len), _key = 0; _key < _len; _key++) {
            args[_key] = arguments[_key];
          }

          decoration.apply(this, args);
          return method.apply(this, args);
        };
      };
    };

Both tools offer an online area where you can type EcmaScript code into a web browser and see the EcmaScript-5 equivalent, and you can run the code as well. To see the result of your expressions, you may have to use the console in your web browser.

So instead of just writing:

    (() => 2 + 2)()
    
And having `4` displayed, you'd need to write:

    console.log(
      (() => 2 + 2)()
    )

And `4` woudl appear in yoru browser's deveopment console.

You can also install the transpilers on your development system and use them with [Node] on [the command line][repl]. The care and feeding of `node` and `npm` are beyond the scope of this book, but both tools offer clear instructions for those who have already insatlled `node`.

[repl]: https://en.wikipedia.org/wiki/REPL "Read–eval–print loop"
[Node]: http://nodejs.org/