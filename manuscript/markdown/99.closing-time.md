# Closing Time at the Coffeeshop: Final Remarks

![The Future of Coffee is Black](images/future-of-coffee.jpg)

We began this book with the most basic of basic ideas in programming: What is a value? What is a reference to a value? What is a function? What is applying or invoking a function with values?

We then looked at one of the "big ideas" that JavaScript shares with other powerful languages: The idea that functions are values, and thus that you can invoke a function with another function as an argument, and you can return a function from a function.

This led directly to exploring the idea of composing functions: Creating new functions by putting together functions that represent smaller pieces of behaviour. The idea of function decorators emerges naturally from this approach.

From there we went on to explore objects and methods, but underlying our exploration was the constant rediscovery that we can program with objects using the same approach: Composing behaviour out of smaller pieces of behaviour, such as composing object behaviour using delegation.

### javascript beyond es6/ecmascript 2015

When this edition was written, ECMAScript 2015 had been standardized, and almost all of its features were available via polyfills and transpilation tools like Babel. Some post-2015 features, like method and class decorators, had not yet been scheduled for inclusion in the language, but were available in transpilers.

Others, such as fully private properties, mixins, and traits, have been discussed and/or proposed. By the time you are reading this, they might be available experimentally, formally approved, or even widely available.

You may have noticed that as the book progressed, it delved into implementing programming language ideas like method decorators, mixins, and traits. Some books might implement a to-do list, or a content management system, or a MMRPG to provide an opportunity to write example code. This book chooses to explain JavaScript by implementing ideas that hopefully will become standard features by the time you read this.

Many other computer science textbooks do the same thing: They explain how to make a "toy" operating system, or a compiler, or how to make a Lisp in Lisp. And as you have learned, JavaScript is a language strong enough to implement many of its ideas in itself.

By implementing simple versions of features like decorators, mixins, and traits, we examined how to program in a lightweight fashion, and we also gained a deeper understanding of the semantics of functions, methods, classes, delegation, and behaviour.

That is valuable whether we use those features in production or not. And that is valuable whether those features are added to JavaScript, or not.

![Espresso, Empty](images/espresso-empty.jpg)

### the lightweight way

When creating a new abstraction, (for example, [traits](#traits)), there are two ways to do it: The heavyweight way, and the lightweight way.

The lightweight way, as explained throughout this book, attempts to be as "JavaScript-y" as possible. For example, using functions for protocols and composing them. With the lightweight way, everything is still just a function, or just an object, or just a class with just a prototype. Lightweight code interoperates 100% with code from other libraries. Lightweight approaches can be incrementally added to an existing code base, refactoring a bit here and a bit there.

The heavyweight way would [greenspun] a special class hierarchy with support for traits baked in. The heavyweight way would produce "classes" that don't easily interoperate with other libraries or code, so you can't incrementally make changes: You have to "boil the ocean" and commit 100% to the new approach. Heavyweight approaches often demand new kinds of tooling in the build pipeline.

[greenspun]: https://en.wikipedia.org/wiki/Greenspun%27s_tenth_rule

When we do things the lightweight way, we make very small bets on their benefits. It's easy to change our mind and abandon the approach in favour of something else. because we make small bets along the way, we collect on the small benefits continuously: We don't have to kick off a massive rewrite of our code base to start using lightweight traits, for example. We just start using them as little or as much as we like, and immediately start benefiting from them.

> "A language that doesn't affect the way you think about programming isn't worth learning."—Alan J. Perlis

Every tool affects the way we think about programming. But heavyweight tools force us to think about the heavyweight tooling. That thinking isn't always portable to another tool or another code base.

Whereas lightweight tools are simple things, composed together in simple ways. If we move to a different code base or tool, we can take our experience with the simple things along. With lightweight traits, for example, we are not teaching ourselves how to "program with traits," we're teaching ourselves how to "decompose behaviour," how to "compose functions" and how to "write functions that decorate entities."

These are all fundamental ideas that apply everywhere, even if we don't end up applying them to build a feature like traits. Lightweight thinking is portable and future-proof.

![The End](images/the-end.jpg)
