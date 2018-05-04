## That Constant Coffee Craving {#const}

Up to now, all we've really seen are *anonymous functions*, functions that don't have a name. This feels very different from programming in most other languages, where the focus is on naming functions, methods, and procedures. Naming things is a critical part of programming, but all we've seen so far is how to name arguments.

There are other ways to name things in JavaScript, but before we learn some of those, let's see how to use what we already have to name things. Let's revisit a very simple example:

    (diameter) => diameter * 3.14159265

What is this "3.14159265" number? [PI], obviously. We'd like to name it so that we can write something like:

    (diameter) => diameter * PI

In order to bind `3.14159265` to the name `PI`, we'll need a function with a parameter of `PI` applied to an argument of `3.14159265`. If we put our function expression in parentheses, we can apply it to the argument of `3.14159265`:

    ((PI) => 
      // ????
    )(3.14159265)

What do we put inside our new function that binds `3.14159265` to the name `PI` when evaluated? Our circumference function, of course:

[PI]: https://en.wikipedia.org/wiki/Pi

    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)

This expression, when evaluated, returns a function that calculates circumferences. That sounds bad, but when we think about it, `(diameter) => diameter * 3.14159265` is also an expression, that when evaluated, returns a function that calculates circumferences. All of our "functions" are expressions. This one has a few more moving parts, that's all. But we can use it just like `(diameter) => diameter * 3.14159265`.

Let's test it:

    ((diameter) => diameter * 3.14159265)(2)
      //=> 6.2831853
      
    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)(2)
      //=> 6.2831853

That works! We can bind anything we want in an expression by wrapping it in a function that is immediately invoked with the value we want to bind.[^explain-iife]

[^explain-iife]: JavaScript programmers regularly use the idea of writing an expression that denotes a function and then immediately applying it to arguments. Explaining the pattern, Ben Alman coined the term [Immediately Invoked Function Expression][iife] for it, often abbreviated "IIFE."

### inside-out

There's another way we can make a function that binds `3.14159265` to the name `PI` and then uses that in its expression. We can turn things inside-out by putting the binding inside our diameter calculating function, like this:

    (diameter) =>
      ((PI) =>
        diameter * PI)(3.14159265)

It produces the same result as our previous expressions for a diameter-calculating function:

    ((diameter) => diameter * 3.14159265)(2)
      //=> 6.2831853
      
    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)(2)
      //=> 6.2831853

    ((diameter) =>
      ((PI) =>
        diameter * PI)(3.14159265))(2)
      //=> 6.2831853

Which one is better? Well, the first one seems simplest, but a half-century of experience has taught us that names matter. A "magic literal" like `3.14159265` is anathema to sustainable software development.

The third one is easiest for most people to read. It separates concerns nicely: The "outer" function describes its parameters:

    (diameter) =>
      // ...

Everything else is encapsulated in its body. That's how it should be, naming `PI` is its concern, not ours. The other formulation:

    ((PI) =>
      // ...
    )(3.14159265)

"Exposes" naming `PI` first, and we have to look inside to find out why we care. So, should we always write this?

    (diameter) =>
      ((PI) =>
        diameter * PI)(3.14159265)
      
Well, the wrinkle with this is that typically, invoking functions is considerably more expensive than evaluating expressions. Every time we invoke the outer function, we'll invoke the inner function. We could get around this by writing 
      
    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)
    
But then we've obfuscated our code, and we don't want to do that unless we absolutely have to.

What would be very nice is if the language gave us a way to bind names inside of blocks without incurring the cost of a function invocation. And JavaScript does.

### const

Another way to write our "circumference" function would be to pass `PI` along with the diameter argument, something like this:

    (diameter, PI) => diameter * PI

And we could use it like this:

    ((diameter, PI) => diameter * PI)(2, 3.14159265)
      //=> 6.2831853

This differs from our example above in that there is only one environment, rather than two. We have one binding in the environment representing our regular argument, and another our "constant." That's more efficient, and it's *almost* what we wanted all along: A way to bind `3.14159265` to a readable name.

JavaScript gives us a way to do that, the `const` keyword. We'll learn a lot more about `const` in future chapters, but here's the most important thing we can do with `const`:

    (diameter) => {
      const PI = 3.14159265;

      return diameter * PI
    }

The `const` keyword introduces one or more bindings in the block that encloses it. It doesn't incur the cost of a function invocation. That's great. Even better, it puts the symbol (like `PI`) close to the value (`3.14159265`). That's much better than what we were writing.

We use the `const` keyword in a *const statement*. `const` statements occur inside blocks, we can't use them when we write a fat arrow that has an expression as its body.

It works just as we want. Instead of:

    ((diameter) =>
      ((PI) =>
        diameter * PI)(3.14159265))(2)
        
Or:

    ((diameter, PI) => diameter * PI)(2, 3.14159265)
      //=> 6.2831853
      
We write:

    ((diameter) => {
      const PI = 3.14159265;

      return diameter * PI
    })(2)
      //=> 6.2831853

We can bind any expression. Functions are expressions, so we can bind helper functions:

    (d) => {
      const calc = (diameter) => {
        const PI = 3.14159265;

        return diameter * PI
      };

      return "The circumference is " + calc(d)
    }

Notice `calc(d)`? This underscores what we've said: if we have an expression that evaluates to a function, we apply it with `()`. A name that's bound to a function is a valid expression evaluating to a function.[^namedfn]

[^namedfn]: We're into the second chapter and we've finally named a function. Sheesh.

A> Amazing how such an important idea--naming functions--can be explained *en passant* in just a few words. That emphasizes one of the things JavaScript gets really, really right: Functions as "first class entities." Functions are values that can be bound to names like any other value, passed as arguments, returned from other functions, and so forth.

We can bind more than one name-value pair by separating them with commas. For readability, most people put one binding per line:

    (d) => {
      const PI   = 3.14159265,
          calc = (diameter) => diameter * PI;

      return "The circumference is " + calc(d)
    }

### nested blocks

Up to now, we've only ever seen blocks we use as the body of functions. But there are other kinds of blocks. One of the places you can find blocks is in an `if` statement. In JavaScript, an `if` statement looks like this:

    (n) => {
      const even = (x) => {
        if (x === 0)
          return true;
        else
          return !even(x - 1);
      }
      return even(n)
    }
    
And it works for fairly small numbers:

    ((n) => {
      const even = (x) => {
        if (x === 0)
          return true;
        else
          return !even(x - 1);
      }
      return even(n)
    })(13)
      //=> false

The `if` statement is a statement, not an expression (an unfortunate design choice), and its clauses are statements or blocks. So we could also write something like:

    (n) => {
      const even = (x) => {
        if (x === 0)
          return true;
        else {
          const odd = (y) => !even(y);
          
          return odd(x - 1);
        }
      }
      return even(n)
    }

And this also works:

    ((n) => {
      const even = (x) => {
        if (x === 0)
          return true;
        else {
          const odd = (y) => !even(y);
          
          return odd(x - 1);
        }
      }
      return even(n)
    })(42)
      //=> true
    
We've used a block as the `else` clause, and since it's a block, we've placed a `const` statement inside it.

### const and lexical scope

This seems very straightforward, but alas, there are some semantics of binding names that we need to understand if we're to place `const` anywhere we like. The first thing to ask ourselves is, what happens if we use `const` to bind two different values to the "same" name?

Let's back up and reconsider how closures work. What happens if we use parameters to bind two different values to the same name?

Here's the second formulation of our diameter function, bound to a name using an IIFE:

    ((diameter_fn) =>
      // ...
    )(
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )

It's more than a bit convoluted, but it binds `((PI) => (diameter) => diameter * PI)(3.14159265)` to `diameter_fn` and evaluates the expression that we've elided. We can use any expression in there, and that expression can invoke `diameter_fn`. For example:

    ((diameter_fn) =>
      diameter_fn(2)
    )(
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )
      //=> 6.2831853
      
We know this from the chapter on [closures](#closures), but even though `PI` is not bound when we invoke `diameter_fn` by evaluating `diameter_fn(2)`, `PI` *is* bound when we evaluated `(diameter) => diameter * PI`, and thus the expression `diameter * PI` is able to access values for `PI` and `diameter` when we evaluate `diameter_fn`.

This is called [lexical scoping], because we can discover where a name is bound by looking at the source code for the program. We can see that `PI` is bound in an environment surrounding `(diameter) => diameter * PI`, we don't need to know where `diameter_fn` is invoked.

We can test this by deliberately creating a "conflict:"

    ((diameter_fn) =>
      ((PI) =>
        diameter_fn(2)
      )(3)
    )(
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )
      //=> 6.2831853

Although we have bound `3` to `PI` in the environment surrounding `diameter_fn(2)`, the value that counts is `3.14159265`, the value we bound to `PI` in the environment surrounding (diameter) => diameter * PI.

That much we can carefully work out from the way closures work. Does `const` work the same way? Let's find out:

    ((diameter_fn) => {
      const PI = 3;
      
      return diameter_fn(2)
    })(
      (() => {
        const PI = 3.14159265;
        
        return (diameter) => diameter * PI
      })()
    )
      //=> 6.2831853

Yes. Binding values to names with `const` works just like binding values to names with parameter invocations, it uses lexical scope.

[lexical scoping]: https://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scope_vs._dynamic_scope

### are consts also from a shadowy planet?

We just saw that values bound with `const` use lexical scope, just like values bound with parameters. They are looked up in the environment where they are declared. And we know that functions create environments. Parameters are declared when we create functions, so it makes sense that parameters are bound to environments created when we invoke functions.

But `const` statements can appear inside blocks, and we saw that blocks can appear inside of other blocks, including function bodies. So where are `const` variables bound? In the function environment? Or in an environment corresponding to the block?

We can test this by creating another conflict. But instead of binding two different variables to the same name in two different places, we'll bind two different values to the same name, but one environment will be completely enclosed by the other.

Let's start, as above, by doing this with parameters. We'll start with:

    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)

And gratuitously wrap it in another IIFE so that we can bind `PI` to something else:

    ((PI) =>
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )(3)
    
This still evaluates to a function that calculates diameters:

    ((PI) =>
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )(3)(2)
      //=> 6.2831853
      
And we can see that our `diameter * PI` expression uses the binding for `PI` in the closest parent environment.  but one question: Did binding `3.14159265` to `PI` somehow change the binding in the "outer" environment? Let's rewrite things slightly differently:

    ((PI) => {
      ((PI) => {})(3);
      
      return (diameter) => diameter * PI;
    })(3.14159265)
    
Now we bind `3` to `PI` in an otherwise empty IIFE inside of our IIFE that binds `3.14159265` to `PI`. Does that binding "overwrite" the outer one? Will our function return `6` or `6.2831853`? This is a book, you've already scanned ahead, so you know that the answer is **no**, the inner binding does not overwrite the outer binding:

    ((PI) => {
      ((PI) => {})(3);
      
      return (diameter) => diameter * PI;
    })(3.14159265)(2)
      //=> 6.2831853
      
We say that when we bind a variable using a parameter inside another binding, the inner binding *shadows* the outer binding. It has effect inside its own scope, but does not affect the binding in the enclosing scope.

So what about `const`. Does it work the same way?

    ((diameter) => {
      const PI = 3.14159265;
      
      (() => {
        const PI = 3;
      })();
      
      return diameter * PI;
    })(2)
      //=> 6.2831853

Yes, names bound with `const` shadow enclosing bindings just like parameters. But wait! There's more!!!

Parameters are only bound when we invoke a function. That's why we made all these IIFEs. But `const` statements can appear inside blocks. What happens when we use a `const` inside of a block?

We'll need a gratuitous block. We've seen `if` statements, what could be more gratuitous than:

    if (true) {
      // an immediately invoked block statement (IIBS)
    }
    
Let's try it:

    ((diameter) => {
      const PI = 3;
      
      if (true) {
        const PI = 3.14159265;
      
        return diameter * PI;
      }
    })(2)
      //=> 6.2831853

    ((diameter) => {
      const PI = 3.14159265;
      
      if (true) {
        const PI = 3;
      }
      return diameter * PI;
    })(2)
      //=> 6.2831853
      
Ah! `const` statements don't just shadow values bound within the environments created by functions, they shadow values bound within environments created by blocks!

This is enormously important. Consider the alternative: What if `const` could be declared inside of a block, but it always bound the name in the function's scope. In that case, we'd see things like this:

    ((diameter) => {
      const PI = 3.14159265;
      
      if (true) {
        const PI = 3;
      }
      return diameter * PI;
    })(2)
      //=> would return 6 if const had function scope
      
If `const` always bound its value to the name defined in the function's environment, placing a `const` statement inside of a block would merely rebind the existing name, overwriting its old contents. That would be super-confusing. And this code would "work:"

    ((diameter) => {
      if (true) {
        const PI = 3.14159265;
      }
      return diameter * PI;
    })(2)
      //=> would return 6.2831853 if const had function scope

Again, confusing. Typically, we want to bind our names as close to where we need them as possible. This design rule is called the [Principle of Least Privilege][plp], and it has both quality and security implications. Being able to bind a name inside of a block means that if the name is only needed in the block, we are not "leaking" its binding to other parts of the code that do not need to interact with it.

[plp]: https://en.wikipedia.org/wiki/Principle_of_least_privilege
    
### rebinding {#rebinding-peek}

By default, JavaScript permits us to *rebind* new values to names bound with a parameter. For example, we can write:

    const evenStevens = (n) => {
      if (n === 0) {
        return true;
      }
      else if (n == 1) {
        return false;
      }
      else {
        n = n - 2;
        return evenStevens(n);
      }
    }
    
    evenStevens(42)
      //=> true
    
The line `n = n - 2;` *rebinds* a new value to the name `n`. We will discuss this at much greater length in [Reassignment](#reassignment), but long before we do, let's try a similar thing with a name bound using `const`. We've already bound `evenStevens` using `const`, let's try rebinding it:

    evenStevens = (n) => {
      if (n === 0) {
        return true;
      }
      else if (n == 1) {
        return false;
      }
      else {
        return evenStevens(n - 2);
      }
    }
      //=> ERROR, evenStevens is read-only
      
JavaScript does not permit us to rebind a name that has been bound with `const`. We can *shadow* it by using `const` to declare a new binding with a new function or block scope, but we cannot rebind a name that was bound with `const` in an existing scope.

This is valuable, as it greatly simplifies the analysis of programs to see at a glance that when something is bound with `const`, we need never worry that its value may change.
