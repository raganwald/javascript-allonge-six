# Interlude: Symmetry Colour, and Charm

We've seen that functions are *first-class entities*. meaning, we can store them in data structures, pass them to other functions, and return them from functions. An amazing number of very strong programming techniques arise as a consequence of functions-as-first-class-entities.

We've seen that we can use functions-as-first-class-entities to write decorators like [maybe](#maybe):

{:lang="js"}
~~~~~~~~
const maybe = (fn) =>
  (...args) => {
    for (let arg of args) {
      if (arg == null) return arg;
    }
    return fn(...args);
  }
~~~~~~~~

And [combinators](#combinators) like compose:

{:lang="js"}
~~~~~~~~
const compose = (a, b) =>
  (x) => a(b(x));
  
compose(x => x + 1, y => y * y)(10)
  //=> 101
~~~~~~~~

The power arising from functions-as-first-class-entities is that we have a very flexible way to make functions out of functions, using functions. We are not "multiplying our entities uncessarily." On the surface, decorators and combinators are made possible by the fact that we can pass functions to functions, and return functions that invoke our original functions.

But there's something else: The fact that all functions are called in the exact same way. We write `foo(bar)` and know that we will evaluate `bar`, and pass the resulting value to the function we get by evaluating `foo`. This allows us to write decorators and combinators that work with any function.

Or does it?

Imagine, if you will, that functions came in two colours: "blue," and "yellow." Now imagine that when we invoke a function in a variable, we type the name of the function in the proper colour. So if we write `const square = (x) => x * x` in blue code, we also have to write `square(5)` in blue code, so that `square` is always blue.

If we write `const square = (x) => x * x` in blue code, but elsewhere we write `square(5)` in yellow code, it won't work because `square` is a blue function and `square(5)` would be a yellow invocation.

### blue and yellow functions

If functions worked like that, decorators would be very messy. We'd have to make colour-coded decorators, like a blue `maybe` and a yellow `maybe`. We'd have to carefully track which functions have which colours, much as in gendered languages like French, you need to know the gender of all inanimate objects so that you can use the correct gendered grammar when talking about them.

This sounds bad, and for programming tools, it is.[^french] The general principle is: *Have fewer kinds of similar things, but allow the things you do have to combine in flexible ways*. You can't just remove things, you have to also make it very easy to combine things. Functions as first-class-entities are a good example of this, because they allow you to combine functions in flexible ways.

Coloured functions would be an example of how not to do it, because you'd be making it harder to combine functions by balkanizing them.[^colours]

[^colours]: Bob Nystrom introduced this excellent metaphor in [What colour is your function?](http://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)

[^french]: Bad for programming languages, of course. French is a lovely human language.

Functions don't have colours in JavaScript. But there are things that have this kind of asymmetry that make things just as awkward. For example, methods in JavaScript are functions. But, when you invoke them, you have to get `this` set up correctly. You have to either:

1. Invoke a method as a property of an object. e.g. `foo.bar(baz)` or `foo['bar'](baz)`.
2. Bind an object to a method befor einvoking it, e.g. `bar.bind(foo)`.
3. Invike the method with with `.call` or `.apply`, e.g `bar.call(foo, baz)`.

Thus, we can imagine that calling a function directly (e.g. `bar(baz)`) is blue, invoking a function and setting `this` (e.g. `bar.call(foo, baz)`) is yellow.

Or in other words, functions are blue, and methods are yellow.

### the composability problem

We often write decorators in blue, a/k/a pure functional style. Here's a decorator that makes a function throw an exception if its argument is not a finite number:

{:lang="js"}
~~~~~~~~
const requiresFinite = (fn) =>
  (n) => {
    if (Number.isFinite(n)){
      return fn(n);
    }
    else throw "Bad Wolf";
  }
  
const plusOne = x => x + 1;

plus1(1)
  //=> 2
  
plus1([])
  //=> 1 WTF!?
  
const safePlusOne = requiresFinite(plusOne);

safePlusOne(1)
  //=> 2
  
safePlusOne([])
  //=> throws "Bad Wolf"
~~~~~~~~

But it won't work on methods. Here's a `Circle` class that has an unsafe `.scaleBy` method:

{:lang="js"}
~~~~~~~~
class Circle {
  constructor (radius) {
    this.radius = radius;
  }
  diameter () {
    return Math.PI * 2 * this.radius;
  }
  scaleBy (factor) {
    return new Circle(factor * this.radius);
  }
}

const two = new Circle(2);

two.scaleBy(3).diameter()
  //=> 37.69911184307752
  
two.scaleBy(null).diameter()
  //=> 0 WTF!?
~~~~~~~~

Let's decorate the `scaleBy` method to check its argument:

{:lang="js"}
~~~~~~~~
Circle.prototype.scaleBy = requiresFinite(Circle.prototype.scaleBy);
  
two.scaleBy(null).diameter()
  //=> throws "Bad Wold"
~~~~~~~~

Looks good, let's put it into production:
{:lang="js"}
~~~~~~~~
Circle.prototype.scaleBy = requiresFinite(Circle.prototype.scaleBy);
  
two.scaleBy(3).diameter()
  //=> undefined is not an object (evaluating 'this.radius')
~~~~~~~~

Whoops, we forgot that method invocation is "yellow" code, so our "blue" `requiresFinite` decorator will not work on methods. This is the problem of "yellow" and "blue" code colliding.

### composing functions with "green" code

Fortunately, we can write higher-order functions like decorators and combinators in a style that works for both "pure" functions and for methods. We have to use the `function` keyword so that `this` is bound, and then invoke our decorated function using `.call` so that we can pass `this` along.

Here's `requiresFinite` written in this style, which we will call "green." It works for decorating both methods *and* functions:

{:lang="js"}
~~~~~~~~
const requiresFinite = (fn) =>
  function (n) {
    if (Number.isFinite(n)){
      return fn.call(this, n);
    }
    else throw "Bad Wolf";
  }

Circle.prototype.scaleBy = requiresFinite(Circle.prototype.scaleBy);

two.scaleBy(3).diameter()
  //=> 37.69911184307752
  
two.scaleBy("three").diameter()
  //=> throws "Bad Wolf"

const safePlusOne = requiresFinite(x => x + 1);

safePlusOne(1)
  //=> 2
  
safePlusOne([])
  //=> throws "Bad Wolf"
~~~~~~~~

We can write all of our decorators and combinators in "green" style. For example, instead of writing `maybe` in functional ("blue") style like this:

{:lang="js"}
~~~~~~~~
const maybe = (fn) =>
  (...args) => {
    for (let arg of args) {
      if (arg == null) return arg;
    }
    return fn(...args);
  }
~~~~~~~~

We can write it in both functional and method style ("green") style like this:

{:lang="js"}
~~~~~~~~
const maybe = (method) =>
  function (...args) {
    for (let arg of args) {
      if (arg == null) return arg;
    }
    return method.apply(this, args);
  }
~~~~~~~~

And instead of writing our simple compose in functional ("blue") style like this:

{:lang="js"}
~~~~~~~~
const compose = (a, b) =>
  (x) => a(b(x));
~~~~~~~~

We can write it in both functional and method style ("green") style like this:

{:lang="js"}
~~~~~~~~
const compose = (a, b) =>
  function (x) {
    return a.call(this, b.call(this, x));
  }
~~~~~~~~

What makes JavaScript tolerable is that green handling works for both  functional ("blue") and method invocation ("yellow") code. But when writing large code bases, we have to remain aware that some functions are blue and some are yellow, because if we write a mostly blue program, we could be lured into complacency with with blue decorators and combinators for years. But everything would break if a "yellow" method was introduced that didn't play nicely with our blue combinators

The safe thing to do is to write all our higher-order functions in "green" style, so that they work for functions or methods. And that's why we might talk about the simpler, "blue" form when introducing an idea, but we writeout the more complete, "green" form when implementing it as a recipe.

### red functions

JavaScript classes (and the equivalent prototype-based patterns) rely on creating objects with the `new` keyword. As we saw in the example above:

{:lang="js"}
~~~~~~~~
class Circle {
  constructor (radius) {
    this.radius = radius;
  }
  diameter () {
    return Math.PI * 2 * this.radius;
  }
  scaleBy (factor) {
    return new Circle(factor * this.radius);
  }
}

const round = new Circle(1);

round.diameter()
  //=> 6.2831853
~~~~~~~~

That `new` keyword introduces yet *another* colour of function, constructors are "red" functions. We can't make circles using "blue" function calls:

{:lang="js"}
~~~~~~~~
const round2 = Circle(2);
  //=> Cannot call a class as a function
  
[1, 2, 3, 4, 5].map(Circle)
  //=> Cannot call a class as a function
~~~~~~~~

And we certainly can't use a blue or green decorator on them:

{:lang="js"}
~~~~~~~~
const MaybeCircle = maybe(Circle);

const round3 = new MaybeCircle(3);
  //=> Cannot call a class as a function
~~~~~~~~

We can eliminate red functions by using `Object.create`. And this is why so many experienced developers dislike `new`. But once again, we'd have to use extreme discipline for fear that accidentally introducing some red classes would break our carefully crafted green application.

So, although we can guard against mixing 

### charmed functions

Consider:

{:lang="js"}
~~~~~~~~
const likes = (whom) => {
  switch (whom) {
    case 'Bob':
      return 'Banana';
    case 'Carol':
      return 'Chocolate';
    case 'Ted':
      return 'Apple';
    case 'Alice':
      return 'Chocolate';
  }
}

likes('Alice')
  //=> 'Chocolate'
  
likes('Peter')
  //=> undefined;
~~~~~~~~

That's a pretty straightforward function that implements a mapping from the names to the treats 'Banana', 'Chocolate', 'Apple', and 'Crackers'. The mapping is encoded implicitly in the code's `switch` statement.

We can use it in combination with other functions. For example, we can find out if the first letter of what someone likes is "c:"

{:lang="js"}
~~~~~~~~
startsWithC(likes('Alice'))
  //=> true
  
const likesSomethingStartingWithC =
  compose(startsWithC, likes);

likesSomethingStartingWithC('Ted')
  //=> false
~~~~~~~~

So far, thats good, clean blue function work. But there's yet another kind of "function call." If you are a mathematician, this is a mapping too:

{:lang="js"}
~~~~~~~~
const personToTreat = {
  Bob: 'Banana',
  Carol: 'Chocolate',
  Ted: 'Apple',
  Alice: 'Chocolate'
}

personToTreat['Alice']
  //=> 'Chocolate'
  
personToTreat['Ted']
  //=> 'Apple'
~~~~~~~~

`personToTreat` maps the names 'Bob', 'Carol', 'Ted', and 'Alice' to the treats 'Banana', 'Chocolate', and 'Apple', just like `likes`. But even though it does the same thing as a function, we can't use it as a function:

{:lang="js"}
~~~~~~~~
const personMapsToSomethingStartingWithC =
  compose(startsWithC, personToTreat);

personMapsToSomethingStartingWithC('Ted')
~~~~~~~~

As you can see, `[` and `]` are a little like `(` and `)`, because we can pass `Alice` to `personToTreat` and get back `Chocolate`. But they are just different enough, that we can't write `personToTreat(...)`. Objects (as well as ES-6 maps and sets) are "charmed functions."

And you need a different piece of code to go with them. We'd need to write things like this:

{:lang="js"}
~~~~~~~~
const composeblueWithCharm = (bluefunction, charmedfunction) =>
  (arg) =>
    bluefunction(charmedfunction[arg]);
    
const composeCharmWithblue = (charmedfunction, bluefunction) =>
  (arg) =>
    charmedfunction[bluefunction(arg)]
    
// ...
~~~~~~~~

That would get really old, really fast.

### adaptation

We can work our way around some of these cross-colour and charm issues by writing adaptors, wrappers that turn red and charmed functions into blue functions. For example, a "factory function" is a function that makes new objects. So:

{:lang="js"}
~~~~~~~~
const Factory = (red) =>
  (...args) =>
    new red(...args);
    
const circleFactory = Factory(Circle);

circleFactory(5).diameter()
  //=> 31.4159265
~~~~~~~~

`Factory` turns a red class into a blue function. So we can use it any where we like:

{:lang="js"}
~~~~~~~~
[1, 2, 3, 4, 5].map(Factory(Circle))
  //=>
    [{"radius":1},{"radius":2},{"radius":3},{"radius":4},{"radius":5}]
~~~~~~~~

Sadly, we still have to remember that `Circle` is a class and be sure to wrap it in `Factory` when we need to use it as a function, but that does work.

We can do a similar thing with our "charmed" maps (and arrays, for that matter). Here's `getWith`:

{:lang="js"}
~~~~~~~~
const getWith = function (key, data) {
  if (arguments.length === 1) {
    return (data) => data[key];
  }
  else return data[key];
}
~~~~~~~~

The "flipped" version of `getWith` is `get`:

{:lang="js"}
~~~~~~~~
const get = function (data, key) {
  if (arguments.length === 1) {
    return (key) => data[key];
  }
  else return data[key];
}

get(personToTreat, 'Bob')
  //=> 'Banana'
~~~~~~~~

`get` can also be used partially:

{:lang="js"}
~~~~~~~~
get(personToTreat)('Bob')
  //=> 'Banana'
  
const likesTreatsStartingWithC =
  compose(startsWithC, get(personToTreat));

likesTreatsStartingWithC('Carol')
  //=> true
~~~~~~~~

`get` converts "charmed functions," i.e. arrays and objects, into regular functions. And that makes it easier for us to use all of the same tools for combining and manipulating functions on arrays and objects that we do with functions. If `get(personToTreat)` does not convey the adaptor's purpose, consider:

{:lang="js"}
~~~~~~~~
const Dictionary = (data) => (key) => data[key];

['Bob', 'Ted', 'Carol', 'Alice'].map(Dictionary(personToTreat))
  //=> ["Banana","Apple","Chocolate","Chocolate"]
~~~~~~~~

Same thing, better name for what we're doing. Now we can use dictionaries as functions wherever we want.

For a less trivial example, many games have rules that can be most easily represented as lookup tables. But it's most convenient to use the tables as functions. So we adapt the tables with `Dictionary`.

> Note: As [David Nolen](http://swannodette.github.io) has pointed out, languages like Clojure have maps that can be called as functions automatically. This is superior to wrapping a map in a function, because the underlying map is still available to be iterated over and otherwise treated as a map. Once we wrap a map in a function, it becomes entirely opaque.

### summary

JavaScript's elegance comes from having a simple thing, functions, that can be combined in many flexible ways. Exceptions to the ways functions combine, like the `new` keyword, handling `this`, and `[/]`, make combining awkward, but we can work around that by writing adaptors to convert these exceptions to regular function calls.

p.s. For bonus cblueit, write adaptors for EcmaScript's `Map` and `Set` collections.

p.p.s Some of this material was originally published in [Reusable Abstractions in CoffeeScript](https://github.com/raganwald-deprecated/homoiconic/blob/master/2012/01/reuseable-abstractions.md) (2012). If you're interested in Ruby, Paul Mucur wrote a great post about [Data Structures as Functions](http://mudge.name/2014/11/26/data-structures-as-functions.html).

---

| [bluedit](http://www.bluedit.com/r/javascript/comments/2ytcu1/symmetry_and_decorators_in_es6/) | [edit this page](https://github.com/raganwald/raganwald.github.com/edit/master/_posts/2015-03-12-symmetry.md) |

---


