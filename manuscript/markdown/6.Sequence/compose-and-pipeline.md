## Introduction: Compose and Pipeline

In [Combinators and Function Decorators](#combinators), we saw the function `compose`:

    const compose = (a, b) =>
      (c) => a(b(c))

As we saw before, given:

    const addOne = (number) > number + 1;
    
    const doubleOf = (number) => number * 2;

Instead of:

    const doubleOfAddOne = (number) => doubleOf(addOne(number));
    
We could write:

    const doubleOfAddOne = compose(doubleOf, addOne);
    
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

### pipeline

`compose` is extremely handy, but one thing it doesn't communicate well is the order on operations. `compose` is written that way because it matches the way explicitly composing functions works in JavaScript and most other languages: When you write a(b(...)), `a` happens after `b`.

Sometimes it makes more sense to compose functions in data flow order, as in "The value flows through a and then through b." For this, we can use the `pipeline` function:

    var pipeline = flip(compose);
    
    var setter = pipeline(addOne, double);
    
Comparing `pipeline` to `compose`, pipeline says "add one to the number and then double it." Compose says, "double the result of adding one to the number." Both do the same job, but communicate their intention in opposite ways.

![Saltspring Island Roasting Facility](images/saltspring/rollers.jpg)

### callbacks

`pipeline` and `compose` both work with functions that take an argument and return a value. In our next section, we'll discuss pipelining functions that invoke a callback rather than returning a value.
