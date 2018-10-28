## Magic Names {#magic-names}

When a function is applied to arguments (or "called"), JavaScript binds the values of arguments to the function's argument names in an environment created for the function's execution. What we haven't discussed so far is that JavaScript also binds values to some "magic" names in addition to any you put in the argument list.[^read-only]

[^read-only]: You should never attempt to define your own bindings against "magic" names that JavaScript binds for you. It is wise to treat them as read-only at all times.

### the function keyword

There are two separate rules for these "magic" names, one for when you invoke a function using the `function` keyword, and another for functions defined with "fat arrows." We'll begin with how things work for functions defined with the `function` keyword.

The first magic name is  `this`, and it is bound to something called the function's [context](#context). We will explore `this` in more detail when we start discussing objects and classes. The second magic name is very interesting, it's called `arguments`, and the most interesting thing about it is that it contains a list of arguments passed to a function:

{:lang="js"}
~~~~~~~~
const plus = function (a, b) {
  return arguments[0] + arguments[1];
}

plus(2,3)
  //=> 5
~~~~~~~~

Although `arguments` looks like an array, it isn't an array: It's more like an object[^pojo] that happens to bind some values to properties with names that look like integers starting with zero:

{:lang="js"}
~~~~~~~~
const args = function (a, b) {
  return arguments;
}

args(2,3)
  //=> { '0': 2, '1': 3 }
~~~~~~~~

`arguments` always contains all of the arguments passed to a function, regardless of how many are declared. Therefore, we can write `plus` like this:

{:lang="js"}
~~~~~~~~
const plus = function () {
  return arguments[0] + arguments[1];
}

plus(2,3)
  //=> 5
~~~~~~~~

When discussing objects, we'll discuss properties in more depth. Here's something interesting about `arguments`:

{:lang="js"}
~~~~~~~~
const howMany = function () {
  return arguments['length'];
}

howMany()
  //=> 0

howMany('hello')
  //=> 1

howMany('sharks', 'are', 'apex', 'predators')
  //=> 4
~~~~~~~~

The most common use of the `arguments` binding is to build functions that can take a variable number of arguments. We'll see it used in many of the recipes, starting off with [partial application](#simple-partial) and [ellipses](#ellipses).

[^pojo]: We'll look at [arrays](#arrays) and [plain old javascript objects](#pojos) in depth later.

### magic names and fat arrows

The magic names `this` and `arguments` have a different behaviour when you invoke a function that was defined with a fat arrow: Instead of being bound when the function is invoked, the fat arrow function always acquires the bindings for `this` and `arguments` from its enclosing scope, just like any other binding.

For example, when this expression's inner function is defined with `function`, `arguments[0]` refers to its only argument, `"inner"`:

{:lang="js"}
~~~~~~~~
(function () {
  return (function () { return arguments[0]; })('inner');
})('outer')
  //=> "inner"
~~~~~~~~

But if we use a fat arrow, `arguments` will be defined in the outer environment, the one defined with `function`. And thus `arguments[0]` will refer to `"outer"`, not to `"inner"`:

{:lang="js"}
~~~~~~~~
(function () {
  return (() => arguments[0])('inner');
})('outer')
  //=> "outer"
~~~~~~~~

Although it seems quixotic for the two syntaxes to have different semantics, it makes sense when you consider the design goal: Fat arrow functions are designed to be very lightweight and are often used with constructs like mapping or callbacks to emulate syntax.

To give a contrived example, this function takes a number and returns an array representing a row in a hypothetical multiplication table. It uses `mapWith`, which we discussed in [Building Blocks](#buildingblocks).[^mapWith] We'll use `arguments` just to show the difference between using a fat arrow and the function keyword:

[^mapWith]: We can also write the following: `const mapWith = (fn) => array => array.map(fn);`, and trust that it works even though we haven't discussed methods yet.

{:lang="js"}
~~~~~~~~
const row = function () {
  return mapWith((column) => column * arguments[0])(
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
  )
}

row(3)
  //=> [3,6,9,12,15,18,21,24,27,30,33,36
~~~~~~~~

This works just fine, because `arguments[0]` refers to the `3` we passed to the function `row`. Our "fat arrow" function `(column) => column * arguments[0]` doesn't bind `arguments` when it's invoked. But if we rewrite `row` to use the `function` keyword, it stops working:

{:lang="js"}
~~~~~~~~
const row = function () {
  return mapWith(function (column) { return column * arguments[0] })(
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
  )
}

row(3)
  //=> [1,4,9,16,25,36,49,64,81,100,121,144]
~~~~~~~~

Now our inner function binds `arguments[0]` every time it is invoked, so we get the same result as if we'd written
`function (column) { return column * column }`.

Although this example is clearly unrealistic, there is a general design principle that deserves attention. Sometimes, a function is meant to be used as a Big-F function. It has a name, it is called by different pieces of code, it's a first-class entity in the code.

But sometimes, a function is a small-f function. It's a simple representation of an expression to be computed. In our example above, `row` is a Big-F function, but `(column) => column * arguments[0]` is a small-f function, it exists just to give `mapWith` something to apply.

Having magic variables apply to Big-F functions but not to small-f functions makes it much easier to use small-f functions as syntax, treating them as expressions or blocks that can be passed to functions like `mapWith`.
