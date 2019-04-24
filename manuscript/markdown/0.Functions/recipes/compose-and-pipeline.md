## Compose and Pipeline

Here is the B Combinator, or `compose` that we saw in [Combinators and Decorators](#combinators):

    const compose = (a, b) =>
      (c) => a(b(c))

As we saw before, given:

    const addOne = (number) => number + 1;

    const doubleOf = (number) => number * 2;

Instead of:

    const doubleOfAddOne = (number) => doubleOf(addOne(number));

We could write:

    const doubleOfAddOne = compose(doubleOf, addOne);

### variadic compose and recursion

If we wanted to implement a `compose3`, we could write:

    const compose3 = (a, b, c) => (d) =>
      a(b(c(d)))

Or observe that it is really:

    const compose3 = (a, b, c) => compose(a, (compose(b, c)))

Once we get to `compose4`, we ask ourselves if there is a better way. For example, if we had a *variadic* compose, we could write `compose(a, b)`, `compose(a, b, c)`, or `compose(a, b, c, d)`.

We can implement a variadic `compose` recursively. The easiest way to reason about writing a recursive `compose` is to start with the smallest or *degenerate* case. If `compose` only took one argument, it would look like this:

    const compose = (a) => a

The next thing is to have a way of breaking a piece off the problem. We can do this with a variadic function:

    const compose = (a, ...rest) =>
      "to be determined"

We can test whether we have the degenerate case:

    const compose = (a, ...rest) =>
      rest.length === 0
        ? a
        : "to be determined"

If it is not the degenerate case, we need to combine what we have with the solution for the rest. In other words, we need to combine `fn` with `compose(...rest)`. How do we do that? Well, consider `compose(a, b)`. We know that `compose(b)` is the degenerate case, it's just `b`. And we know that `compose(a, b)` is `(c) => a(b(c))`.

So let's substitute `compose(b)` for `b`:

    compose(a, compose(b)) === (c) => a(compose(b)(c))

Now substitute `...rest` for `b`:

    compose(a, compose(...rest)) === (c) => a(compose(...rest)(c))

This is our solution:

    const compose = (a, ...rest) =>
      rest.length === 0
        ? a
        : (c) => a(compose(...rest)(c))

There are others, of course. `compose` can be implemented with iteration or with `.reduce`, like this:

    const compose = (...fns) =>
      (value) =>
        fns.reverse().reduce((acc, fn) => fn(acc), value);

But the principle behaviour is the same: To compose a series of functions together, creating a new one. And the value is the same: We can write smaller, single purpose functions and put them together in different ways.

### the semantics of compose

With `compose`, we're usually making a new function. Although it works perfectly well, we don't need to write things like `compose(double, addOne)(3)` inline to get the result `8`. It's easier and clearer to write `double(addOne(3))`.

On the other hand, when working with something like method decorators, it can help to write:

    const setter = compose(fluent, maybe);

    // ...

    SomeClass.prototype.setUser = setter(function (user) {
      this.user = user;
    });

    SomeClass.prototype.setPrivileges = setter(function (privileges) {
      this.privileges = privileges;
    });

This makes it clear that `setter` adds the behaviour of both `fluent` and `maybe` to each method it decorates, and it's sometimes easier to read `const setter = compose(fluent, maybe);` than:

    const setter = (fn) => fluent(maybe(fn));

The take-away is that `compose` is helpful when we are defining a new function that combines the effects of existing functions.

### pipeline {#pipeline}

`compose` is extremely handy, but one thing it doesn't communicate well is the order on operations. `compose` is written that way because it matches the way explicitly composing functions works in JavaScript and most other languages: When you write a(b(...)), `a` happens after `b`.

Sometimes it makes more sense to compose functions in data flow order, as in "The value flows through a and then through b." For this, we can use the `pipeline` function:

    const pipeline = (...fns) =>
      (value) =>
        fns.reduce((acc, fn) => fn(acc), value);

    const setter = pipeline(addOne, double);

Comparing `pipeline` to `compose`, pipeline says "add one to the number and then double it." Compose says, "double the result of adding one to the number." Both do the same job, but communicate their intention in opposite ways.

![Saltspring Island Roasting Facility](images/saltspring/rollers.jpg)
