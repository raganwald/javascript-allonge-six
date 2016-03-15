# Colourful Mugs: Symmetry, Colour, and Charm {#symmetry}

![Pantone Coffee Mugs](images/pantone.jpg)

We've seen that functions are *first-class entities*. meaning, we can store them in data structures, pass them to other functions, and return them from functions. An amazing number of very strong programming techniques arise as a consequence of functions-as-first-class-entities.

We've also seen that we can use functions-as-first-class-entities to write decorators like [maybe](#maybe):

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

The power arising from functions-as-first-class-entities is that we have a very flexible way to make functions out of functions, using functions. We are not "multiplying our entities unnecessarily." On the surface, decorators and combinators are made possible by the fact that we can pass functions to functions, and return functions that invoke our original functions.

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
2. Bind an object to a method before invoking it, e.g. `bar.bind(foo)`.
3. Invoke the method with with `.call` or `.apply`, e.g `bar.call(foo, baz)`.

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
  //=> throws "Bad Wolf"
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

The safe thing to do is to write all our higher-order functions in "green" style, so that they work for functions or methods. And that's why we might talk about the simpler, "blue" form when introducing an idea, but we write out the more complete, "green" form when implementing it as a recipe.

### red functions vs. object factories

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

And we certainly can't use a decorator on them:

{:lang="js"}
~~~~~~~~
const CircleRequiringFiniteRadius = requiresFinite(Circle);

const round3 = new CircleRequiringFiniteRadius(3);
  //=> Cannot call a class as a function
~~~~~~~~

Some experienced developers dislike `new` because of this problem: It introduces one more kind of function that doesn't compose neatly with other functions using our existing decorators and combinators.

We could eliminate "red" functions by using prototypes and `Object.create` instead of using the `class` and `new` keywords. A "factory function" is a function that makes new objects. So instead of writing a `Circle` class, we would write a `CirclePrototype` and a `CircleFactory` function:

{:lang="js"}
~~~~~~~~
const CirclePrototype = {
  diameter () {
    return Math.PI * 2 * this.radius;
  },
  scaleBy (factor) {
    return CircleFactory(factor * this.radius);
  }
};

const CircleFactory = (radius) =>
  Object.create(CirclePrototype, {
    radius: { value: radius, enumerable: true }
  })

CircleFactory(2).scaleBy(3).diameter()
  //=> 37.69911184307752
~~~~~~~~

Now we have a "blue" `CircleFactory` function, and we have the benefits of objects and methods, along with the benefits of decorating and composing factories like any other function. For example:

{:lang="js"}
~~~~~~~~
const requiresFinite = (fn) =>
  function (n) {
    if (Number.isFinite(n)){
      return fn.call(this, n);
    }
    else throw "Bad Wolf";
  }

const FiniteCircleFactory = requiresFinite(CircleFactory);

FiniteCircleFactory(2).scaleBy(3).diameter()
  //=> 37.69911184307752

FiniteCircleFactory(null).scaleBy(3).diameter()
  //=> throws "Bad Wolf"
~~~~~~~~

All that being said, programming with factory functions instead of with classes and `new` is not a cure-all. Besides losing some of the convenience and familiarity for other developers, we'd also have to use extreme discipline for fear that accidentally introducing some "red" classes would break our carefully crafted "blue in green" application.

In the end, there's no avoiding the need to know which functions are functions, and which are actually classes. Tooling can help: Some linting applications can enforce a naming convention where classes start with an upper-case letter and functions start with a lower-case letter.

### charmed functions

Consider:

{:lang="js"}
~~~~~~~~
const likesToDrink = (whom) => {
  switch (whom) {
    case 'Bob':
      return 'Ristretto';
    case 'Carol':
      return 'Cappuccino';
    case 'Ted':
      return 'Allongé';
    case 'Alice':
      return 'Cappuccino';
  }
}

likesToDrink('Alice')
  //=> 'Cappuccino'

likesToDrink('Peter')
  //=> undefined;
~~~~~~~~

That's a pretty straightforward function that implements a mapping from `Bob`, `Carol`, `Ted`, and `Alice` to the drinks 'Ristretto', 'Cappuccino', and 'Allongé'. The mapping is encoded implicitly in the code's `switch` statement.

We can use it in combination with other functions. For example, we can find out if the first letter of what someone likes is "c:"

{:lang="js"}
~~~~~~~~
const startsWithC = (something) => !!something.match(/^c/i)

startsWithC(likesToDrink('Alice'))
  //=> true

const likesSomethingStartingWithC =
  compose(startsWithC, likesToDrink);

likesSomethingStartingWithC('Ted')
  //=> false
~~~~~~~~

So far, that's good, clean blue function work. But there's yet another kind of "function call." If you are a mathematician, this is a mapping too:

{:lang="js"}
~~~~~~~~
const personToDrink = {
  Bob: 'Ristretto',
  Carol: 'Cappuccino',
  Ted: 'Allongé',
  Alice: 'Cappuccino'
}

personToDrink['Alice']
  //=> 'Cappuccino'

personToDrink['Ted']
  //=> 'Allongé'
~~~~~~~~

`personToDrink` also maps the names 'Bob', 'Carol', 'Ted', and 'Alice' to the drinks 'Ristretto', 'Cappuccino', and 'Allongé', just like `likesToDrink`. But even though it does the same thing as a function, we can't use it as a function:

{:lang="js"}
~~~~~~~~
const personMapsToSomethingStartingWithC =
  compose(startsWithC, personToDrink);

personMapsToSomethingStartingWithC('Ted')
  //=> undefined is not a function (evaluating 'b.call(this, x)')
~~~~~~~~

As you can see, `[` and `]` are a little like `(` and `)`, because we can pass `Alice` to `personToDrink` and get back `Cappuccino`. But they are just different enough, that we can't write `personToDrink(...)`. Objects (as well as ECMAScript 2015 maps and sets) are "charmed functions."

And you need a different piece of code to go with them. We'd need to write things like this:

{:lang="js"}
~~~~~~~~
const composeblueWithCharm =
  (bluefunction, charmedfunction) =>
    (arg) =>
      bluefunction(charmedfunction[arg]);

const composeCharmWithblue =
  (charmedfunction, bluefunction) =>
    (arg) =>
      charmedfunction[bluefunction(arg)]

// ...
~~~~~~~~

That would get really old, really fast.

### adapting to handle red and charmed functions

We can work our way around some of these cross-colour and charm issues by writing *adaptors*, wrappers that turn red and charmed functions into blue functions. As we saw above, a "factory function" is a function that is called in the normal way, and returns a freshly created object.

If we wanted to create a `CircleFactory`, we could use `Object.create` as we saw above. We could also wrap `new Circle(...)` in a function:

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

const CircleFactory = (radius) =>
  new Circle(radius);

CircleFactory(2).scaleBy(3).diameter()
  //=> 37.69911184307752
~~~~~~~~

With some argument jiggery-pokery, we could abstract `Circle` from `CircleFactory` and make a factory for making factories, a `FactoryFactory`:

We would write a `CircleFactory` function:

{:lang="js"}
~~~~~~~~
const FactoryFactory = (clazz) =>
  (...args) =>
    new clazz(...args);

const CircleFactory = FactoryFactory(Circle);

circleFactory(5).diameter()
  //=> 31.4159265
~~~~~~~~

`FactoryFactory` turns any "red" class into a "blue" function. So we can use it any where we like:

{:lang="js"}
~~~~~~~~
[1, 2, 3, 4, 5].map(FactoryFactory(Circle))
  //=>
    [{"radius":1},{"radius":2},{"radius":3},{"radius":4},{"radius":5}]
~~~~~~~~

Sadly, we still have to remember that `Circle` is a class and be sure to wrap it in `FactoryFactory` when we need to use it as a function, but that does work.

We can do a similar thing with our "charmed" maps (and arrays, for that matter). Here's `Dictionary`, a function that turns objects and arrays (our "charmed" functions) into ordinary ("blue") functions:

{:lang="js"}
~~~~~~~~
const Dictionary = (data) => (key) => data[key];

const personToDrink = {
  Bob: 'Ristretto',
  Carol: 'Cappuccino',
  Ted: 'Allongé',
  Alice: 'Cappuccino'
}

['Bob', 'Ted', 'Carol', 'Alice'].map(Dictionary(personToDrink))
  //=> ["Ristretto","Allongé","Cappuccino","Cappuccino"]
~~~~~~~~

`Dictionary` makes it easier for us to use all of the same tools for combining and manipulating functions on arrays and objects that we do with functions.

### dictionaries as proxies

As [David Nolen](http://swannodette.github.io) has pointed out, languages like Clojure have maps that can be called as functions automatically. This is superior to wrapping a map in a plain function, because the underlying map is still available to be iterated over and otherwise treated as a map. Once we wrap a map in a function, it becomes opaque, useless for anything except calling as a function.

If we wish, we can create a dictionary function that is a partial proxy for the underlying collection object. For example, here is an `IterableDictionary` that turns a collection into a function that is also iterable if its underlying data object is iterable:

{:lang="js"}
~~~~~~~~
const IterableDictionary = (data) => {
  const proxy = (key) => data[key];
  proxy[Symbol.iterator] = function* (...args) {
    yield * data[Symbol.iterator](...args);
  }
  return proxy;
}

const people = IterableDictionary(['Bob', 'Ted', 'Carol', 'Alice']);
const drinks = IterableDictionary(personToDrink);

for (let name of people) {
  console.log(`${name} prefers to drink ${drinks(name)}`)
}
  //=>
    Bob prefers to drink Ristretto
    Ted prefers to drink Allongé
    Carol prefers to drink Cappuccino
    Alice prefers to drink Cappuccino
~~~~~~~~

This technique has limitations. For example, objects in JavaScript are not iterable by default. So we can't write:

{:lang="js"}
~~~~~~~~
for (let [name, drink] of personToDrink) {
  console.log(`${name} prefers to drink ${drink}`)
}
  //=> undefined is not a function (evaluating 'personToDrink[Symbol.iterator]()')
~~~~~~~~

We could write:

{:lang="js"}
~~~~~~~~
for (let [name, drink] of Object.entries(personToDrink)) {
  console.log(`${name} prefers to drink ${drink}`)
}
  //=>
    Bob prefers to drink Ristretto
    Carol prefers to drink Cappuccino
    Ted prefers to drink Allongé
    Alice prefers to drink Cappuccino
~~~~~~~~

It would be an enormous hack to make `Object.entries(IterableDictionary(personToDrink))` work. While we're at it, how would we make `.length` work? Functions implement `.length` as the number of arguments they accept. Arrays implement it as the number of entries they hold. If we wrap an array in a dictionary, what is its `.length`?

Proxying collections, meaning "creating an object that behaves like the collection," works for specific and limited contexts, but it is enormously fragile to attempt to make a universal proxy that also acts as a function.

### summary

JavaScript's elegance comes from having a simple thing, functions, that can be combined in many flexible ways. Exceptions to the ways functions combine, like the `new` keyword, handling `this`, and `[...]`, make combining awkward, but we can work around that by writing adaptors to convert these exceptions to regular function calls.
