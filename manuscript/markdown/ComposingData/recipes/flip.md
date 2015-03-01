## Flip {#flip}

When we first wrote [mapWith](#mapping), we wrote it like this:

{:lang="js"}
~~~~~~~~
const mapWith = (fn, [first, ...rest]) =>
  first === undefined
    ? []
    : [fn(first), ...mapWith(fn, rest)];
~~~~~~~~

Let's consider the case whether we have a `map` function of our own, perhaps from the [allong.es](http://allong.es) library, perhaps from [Underscore](http://underscorejs.org). We could write our function something like this:

{:lang="js"}
~~~~~~~~
const mapWith = (fn, list) => map(list, fn);
~~~~~~~~

We're flip:

    function mapCurried (list) {
      return function (fn) {
        return map(list, fn);
      };
    };

Let's return to the implementation that does both:

    function mapWith (fn) {
      return function (list) {
        return map.call(list, fn);
      };
    };

Now let's put a wrapper around it:

    function wrapper () {
      return function (fn) {
        return function (list) {
          return map.call(list, fn);
        };
      };
    };

Abstract the parameter names:

    function wrapper () {
      return function (first) {
        return function (second) {
          return map.call(second, first);
        };
      };
    };

And finally, extract the function as a parameter:

    function wrapper (fn) {
      return function (first) {
        return function (second) {
          return fn.call(second, first);
        };
      };
    };

What we have now is a function that takes a function and "flips" the order of arguments around, then curries it:

    function flip (fn) {
      return function (first) {
        return function (second) {
          return fn.call(this, second, first);
        };
      };
    };

This is gold. Consider how we define `mapWith` now:

    var mapWith = flip(map);

Much nicer!

There's one final decoration. Sometimes we'll want to flip a function but retain the flexibility to call it with both parameters at once. No problem:

    function flip (fn) {
      return function (first, second) {
        if (arguments.length === 2) {
          return fn.call(this, second, first);
        }
        else {
          return function (second) {
            return fn.call(this, second, first);
          };
        };
      };
    };

Now you can call `mapWith(fn, list)` or `mapWith(fn)(list)`, your choice.