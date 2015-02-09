## Reassignment {#reassignment}

Like some imperative programming languages, JavaScript allows you to re-assign the value bound to parameters. We saw this earlier in [rebinding](#rebinding-peek):

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
    
The line `n = n - 2;` *rebinds* a new value to the name `n`. We will discuss this at much greater length in the chapter on [Rebinding and References](#references), but long before we do, let's try a similar thing with a name bound using `const`. We've already bound `evenStevens` using `const`, let's try rebinding it:

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

Rebinding parameters is usually avoided, but what about rebinding names we declare within a function? What we want is a statement that works like `const`, but permits us to rebind variables. JavaScript has such a thing, it's called `let`:

{:lang="javascript"}
~~~~~~~~
let age = 52;

age = 53;
age
  //=> 53
~~~~~~~~

We took the time to carefully examine what happens with bindings in environments. Let's take the time to explore what happens with reassigning values to variables. The key is to understand that we are rebinding a different value to the same name in the same environment.

So let's consider what happens with a shadowed variable:

    (() => {
      let age = 49;
      
      if (true) {
        let age = 50;
      }
      return age;
    })()
      //=> 49

Using `let` to bind `50` to age within the block does not change the binding of `age` in the outer environment because the binding of `age` in the block shadows the binding of `age` in the outer environment, just like `const`. We go from:

    {age: 49, '..': global-environment}
    
To:

    {age: 50, '..': {age: 49, '..': global-environment}}
    
Then back to:

    {age: 49, '..': global-environment}

However, if we don't shadow `age` with `let`, reassigning within the block changes the original:

    (() => {
      let age = 49;
      
      if (true) {
        age = 50;
      }
      return age;
    })()
      //=> 50

Like evaluating variable labels, when a binding is rebound, JavaScript searches for the binding in the current environment and then each ancestor in turn until it finds one. It then rebinds the name in that environment.

### mixing `let` and `const`

Some programmers dislike deliberately shadowing variables. The suggestion is that shadowing a variable is confusing code. If you buy that argument, the way that shadowing works in JavaScript exists to protect us from accidentally shadowing a variable when we move code around.

If you dislike deliberately shadowing variables, you'll probably take an even more opprobrious view of mixing `const` and `let` semantics with a shadowed variable:

    (() => {
      let age = 49;
      
      if (true) {
        const age = 50;
      }
      age = 51;
      return age;
    })()
      //=> 51
      
Shadowing a `let` with a `const` does not change our ability to rebind the variable in its original scope. And:

    (() => {
      const age = 49;
      
      if (true) {
        let age = 50;
      }
      age = 52;
      return age;
    })()
      //=> ERROR: age is read-only

Shadowing a `const` with a `let` does not permit it to be rebound in its original scope.
