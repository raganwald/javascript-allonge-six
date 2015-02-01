## Why JavaScript Allongé, First Edition?

*JavaScript Allongé* is a book about programming with functions, because [JavaScript] is a programming language built on flexible and powerful functions. *JavaScript Allongé* begins at the beginning, with values and expressions, and builds from there to discuss types, identity, functions, closures, scopes, and many more subjects up to working with classes and instances. In each case, *JavaScript Allongé* takes care to explain exactly how things work so that when you encounter a problem, you'll know exactly what is happening and how to fix it.

Second, *JavaScript Allongé* provides recipes for using functions to write software that is simpler, cleaner, and less complicated than alternative approaches that are object-centric or code-centric. JavaScript idioms like function combinators and decorators leverage JavaScript's power to make code easier to read, modify, debug and refactor, thus *avoiding* problems before they happen.

*JavaScript Allongé* teaches you how to handle complex code, and it also teaches you how to simplify code without dumbing it down. As a result, *JavaScript Allongé* is a rich read releasing many of JavaScript's subtleties, much like the Café Allongé beloved by coffee enthusiasts everywhere.

[JavaScript]: https://developer.mozilla.org/en-US/docs/JavaScript

### Why JavaScript Allongé, The "Six" Edition?

The first edition of JavaScript Allongé was written when almost all versions of JavaScript conformed to the ECMAScript-5 standard. ECMAScript-5 makes a nice language, but many JavaScript developers are ployglots: They might write Ruby on a server and JavaScript in the browser, or build systems that have services written with Node/JavaScript and Python.

And lots of JavaScript developers have experimented with compile-to-JavaScript alternatives like CoffeeScript or ClojureScript. The net result is that JavaScript developers have aquired a taste for language features that ther languages have proven to be useful.

The ECMAScript standards committee responded with ECMAScript-6, an update to JavaScript that introduces new features that have been proven useful and popular in other languages. It also introduces a number of features that address common JavaScript concerns.

The first edition of JavaScript Allongé does not use any of the ECMAScript-6 features, and in some cases shows you how to solve a problem with ECMAScript-5 that ECMAScript-6 solves in a natural way.

For example, the first edition notes that in CoffeeScript, you can write:

    callLeft = (fn, args...) =>
      (remainingArgs...) =>
        fn.apply(null, args.concat(remainingArgs))
        
In ECMAScript-5, you do not have "rest arguments," so you can't write `args...`. You also don't have the "arrow notation" for functions, so you have to write `function (...) { ... }`. To get "rest" arguments in ECMAScript-5, the first edition provides a decorator called `variadic`, it looks like this:

    var __slice = Array.prototype.slice;

    function variadic (fn) {
      var fnLength = fn.length;

      if (fnLength < 1) {
        return fn;
      }
      else if (fnLength === 1)  {
        return function () {
          return fn.call(
            this, __slice.call(arguments, 0))
        }
      }
      else {
        return function () {
          var numberOfArgs = arguments.length,
              namedArgs = __slice.call(
                arguments, 0, fnLength - 1),
              numberOfMissingNamedArgs = Math.max(
                fnLength - numberOfArgs - 1, 0),
              argPadding = new Array(numberOfMissingNamedArgs),
              variadicArgs = __slice.call(
                arguments, fn.length - 1);

          return fn.apply(
            this, namedArgs
                  .concat(argPadding)
                  .concat([variadicArgs]));
        }
      }
    };
    
In ECMAScript-5, you could use it like this:

    var callLeft = variadic( function (fn, args) {
      return variadic( function (remainingArgs) {
          return fn.apply(null, args.concat(remainingArgs));
        };
      });
      
But in ECMAScript-6, you can write:

    var callLeft = (fn, ...args) =>
        (...remainingArgs) =>
          fn(...args, ...remainingArgs);

The syntax is very similar to CoffeeScript, and no decorator is required.

JavaScript Allongé, The "Six" Edition packs all the goodness of JavaScript Allongé into a new, updated package that is relevant for programmers working with (or planning to work with) the latest version of JavaScript.

### how the book is organized

*JavaScript Allongé* introduces new aspects of programming with functions in each chapter, explaining exactly how JavaScript works. Code examples within each chapter are small and emphasize exposition rather than serving as patterns for everyday use.

Following each chapter are a series of recipes designed to show the application of the chapters ideas in practical form. While the content of each chapter builds naturally on what was discussed in the previous chapter, the recipes may draw upon any aspect of the JavaScript programming language.