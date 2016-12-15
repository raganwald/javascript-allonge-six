## Flip {#flip}

We wrote [mapWith](#mapWith) like this:

{:lang="js"}
~~~~~~~~
const mapWith = (fn) => (list) => list.map(fn);
~~~~~~~~

Let's consider the case whether we have a `map` function of our own, perhaps from the [allong.es](https://github.com/raganwald/allong.es) library, perhaps from [Underscore](http://underscorejs.org). We could write our function something like this:

    const mapWith = (fn) => (list) => map(list, fn);

Looking at this, we see we're conflating two separate transformations. First, we're reversing the order of arguments. You can see that if we simplify it:

    const mapWith = (fn, list) => map(list, fn);

Second, we're "currying" the function so that instead of defining a function that takes two arguments, it returns a function that takes the first argument and returns a function that takes the second argument and applies them both, like this:

    const mapper = (list) => (fn) => map(list, fn);

Let's return to the implementation of `mapWith` that relies on a `map` function rather than a method:

    const mapWith = (fn) => (list) => map(list, fn);
    
We're going to extract these two operations by refactoring our function to paramaterize `map`. The first step is to give our parameters generic names:

    const mapWith = (first) => (second) => map(second, first);

Then we wrap the entire thing in a function and extract `map`

    const wrapper = (fn) =>
      (first) => (second) => fn(second, first);

What we have now is a function that takes a function and "flips" the order of arguments around, then curries it. So let's call it `flipAndCurry`:

    const flipAndCurry = (fn) =>
      (first) => (second) => fn(second, first);
      
Sometimes you want to flip, but not curry:

    const flip = (fn) =>
      (first, second) => fn(second, first);

This is gold. Consider how we define [mapWith](#mapWith) now:

    var mapWith = flipAndCurry(map);

Much nicer!

### self-currying flip

Sometimes we'll want to flip a function, but retain the flexibility to call it in its curried form (pass one parameter) or non-curried form (pass both). We *could* make that into `flip`:

    const flip = (fn) =>
      function (first, second) {
        if (arguments.length === 2) {
          return fn(second, first);
        }
        else {
          return function (second) {
            return fn(second, first);
          };
        };
      };

Now if we write `mapWith = flip(map)`, we can call `mapWith(fn, list)` or `mapWith(fn)(list)`, our choice.

### flipping methods

When we learn about context and methods, we'll see that `flip` throws the current context away, so it can't be used to flip methods. A small alteration gets the job done:

    const flipAndCurry = (fn) =>
      (first) =>
        function (second) {
          return fn.call(this, second, first);
        }

    const flip = (fn) =>
      function (first, second) {
        return fn.call(this, second, first);
      }

    const flip = (fn) =>
      function (first, second) {
        if (arguments.length === 2) {
          return fn.call(this, second, first);
        }
        else {
          return function (second) {
            return fn.call(this, second, first);
          };
        };
      };
