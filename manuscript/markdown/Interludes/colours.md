# Interlude: Symmetry and Colourful Functions

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

Imagine, if you will, that functions came in two colours: <span style="color: red;">red</span> and <span style="color: blue;">blue</span>. Now imagine that when we invoke a function in a variable, we type the name of the function in the proper colour. So if we write <code>const <span style="color: red;">square</span> = (x) => x * x</code>, we later have to write <code>[1, 2, 3, 4, 5].map(<span style="color: red;">square</span>)</code>. If we write <code>[1, 2, 3, 4, 5].map(<span style="color: blue;">square</span>)</code>, it won't work because `square` is a <span style="color: red;">red</span> function.

### working with coloured functions

If functions worked like that, decorators would be very messy. We'd have to make colour-coded decorators, like <code><span style="color: red;">maybe</span></code> and <code><span style="color: blue;">maybe</span></code>. We'd have to carefully track which functions have which colours, much as in gendered languages like French, you need to know the gender of all inanimate objects so that you can use the correct gendered grammar when talking about them.

This sounds bad, and it is.[^french] The general principle is: *Have fewer kinds of similar things, but allow the things you do have to combine in flexible ways*. You can't just remove things, you have to also make it very easy to combine things. Functions as first-class-entities are a good example of this, because they allow you to combine functions in flexible ways.

Coloured functions would be an example of how not to do it, because you'd be making it harder to combine functions by balkanizing them.[^colours]

[^colours]: Bob Nystrom introduced this excellent metaphor in [What colour is your function?](http://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)

[^french]: Bad for programming languages, of course. French is a lovely human language.

### does javascript have coloured functions?

Functions don't have colours in JavaScript. But there are things that have this kind of asymmetry that make things just as awkward. For example, methods in JavaScript are functions. But, when you invoke them, you have to get `this` set up correctly. You have to either:

1. Invoke a method as a property of an object. e.g. `foo.bar(baz)` or `foo['bar'](baz)`.
2. Bind an object to a method befor einvoking it, e.g. `bar.bind(foo)`.
3. Invike the method with with `.call` or `.apply`, e.g `bar.call(foo, baz)`.

So while `maybe` for functions looks like this:

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

`maybe` for methods looks like this:

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

In JavaScript, ordinary function invocation with `(` and `)` is <span style="color: red;">red</span>, but method invocation is <span style="color: purple;">purple</span>. Browser event handling is also purple, but what makes this somewhat tolerable is that <span style="color: purple;">purple</span> handling also works for <span style="color: red;">red</span> functions. But you have to be aware that some functions are <span style="color: purple;">purple</span>, because if you write a <span style="color: red;">red</span> program, you could be happy with <span style="color: red;">red</span> decorators for years and then suddenly something breaks when a <span style="color: purple;">purple</span> function or method is introduced.

### yellow functions

EcmaScript-6 classes (and the equivalent EcmaScript-5 patterns) rely on creating objects with the `new` keyword:

{:lang="js"}
~~~~~~~~
class Circle {
  constructor (radius) {
    this.radius = radius;
  }
  diameter () {
    return 3.14159265 * 2 * this.radius;
  }
}

const round = new Circle(1);

round.diameter()
  //=> 6.2831853
~~~~~~~~

That `new` keyword introduces another colour, constructors are <span style="color: yellow;">yellow</span> functions. We can't make circles using <span style="color: red;">red</span> function calls:

{:lang="js"}
~~~~~~~~
const round2 = Circle(2);
  //=> Cannot call a class as a function
  
[1, 2, 3, 4, 5].map(Circle)
  //=> Cannot call a class as a function
~~~~~~~~

And we certainly can't use a red or purple decorator on them:

{:lang="js"}
~~~~~~~~
const MaybeCircle = maybe(Circle);

const round3 = new MaybeCircle(3);
  //=> Cannot call a class as a function
~~~~~~~~

We can eliminate <span style="color: yellow;">yellow</span> functions by using `Object.create`. And this is why so many experienced developers dislike `new`. But once again, we'd have to use extreme discipline for fear that accidentally introducing some <span style="color: yellow;">yellow</span> would break our carefully crafted <span style="color: purple;">purple</span> application.

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

So far, thats good, clean <span style="color: red;">red</span> function work. But there's yet another kind of "function call." If you are a mathematician, this is a mapping too:

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
const composeRedWithCharm = (redfunction, charmedfunction) =>
  (arg) =>
    redfunction(charmedfunction[arg]);
    
const composeCharmWithRed = (charmedfunction, redfunction) =>
  (arg) =>
    charmedfunction[redfunction(arg)]
    
// ...
~~~~~~~~

That would get really old, really fast.

### adaptation

We can work our way around some of these cross-colour and charm issues by writing adaptors, wrappers that turn yellow and charmed functions into red functions. For example, a "factory function" is a function that makes new objects. So:

{:lang="js"}
~~~~~~~~
const Factory = (yellow) =>
  (...args) =>
    new yellow(...args);
    
const circleFactory = Factory(Circle);

circleFactory(5).diameter()
  //=> 31.4159265
~~~~~~~~

`Factory` turns a <span style="color: yellow;">yellow</span> class into a <span style="color: red;">red</span> function. So we can use it any where we like:

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

p.s. For bonus credit, write adaptors for EcmaScript's `Map` and `Set` collections.

p.p.s Some of this material was originally published in [Reusable Abstractions in CoffeeScript](https://github.com/raganwald-deprecated/homoiconic/blob/master/2012/01/reuseable-abstractions.md) (2012). If you're interested in Ruby, Paul Mucur wrote a great post about [Data Structures as Functions](http://mudge.name/2014/11/26/data-structures-as-functions.html).

---

| [reddit](http://www.reddit.com/r/javascript/comments/2ytcu1/symmetry_and_decorators_in_es6/) | [edit this page](https://github.com/raganwald/raganwald.github.com/edit/master/_posts/2015-03-12-symmetry.md) |

---


