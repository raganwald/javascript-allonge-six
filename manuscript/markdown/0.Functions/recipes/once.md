## Once

`once` is an extremely helpful combinator. It ensures that a function can only be called, well, *once*. Here's the recipe:

    const once = (fn) => {
      let done = false;

      return function () {
        return done ? void 0 : ((done = true), fn.apply(this, arguments))
      }
    }

Very simple! You pass it a function, and you get a function back. That function will call your function once, and thereafter will return `undefined` whenever it is called. Let's try it:

    const askedOnBlindDate = once(
      () => "sure, why not?"
    );

    askedOnBlindDate()
      //=> 'sure, why not?'

    askedOnBlindDate()
      //=> undefined

    askedOnBlindDate()
      //=> undefined

It seems some people will only try blind dating once.

(Note: There are some subtleties with decorators like `once` that involve the intersection of state with methods. We'll look at that again in [stateful method decorators](#stateful-method-decorators).)
