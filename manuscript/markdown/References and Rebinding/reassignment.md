## Reassignment and Mutation {#reassignment}

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

![Cupping Grinds](images/cupping.jpg)

### mutation and aliases

Now that we can reassign things, there's another important factor to consider: Some values can *mutate*. Their identities stay the same, but not their structure. Specifically, arrays and objects can mutate. Recall that you can access a value from within an array or an object using `[]`. You can reassign a value using `[]` as well:

    const oneTwoThree = [1, 2, 3];
    oneTwoThree[0] = 'one';
    oneTwoThree
      //=> [ 'one', 2, 3 ]

You can even add a value:

    const oneTwoThree = [1, 2, 3];
    oneTwoThree[3] = 'four';
    oneTwoThree
      //=> [ 1, 2, 3, 'four' ]

You can do the same thing with both syntaxes for accessing objects:

    const name = {firstName: 'Leonard', lastName: 'Braithwaite'};
    name.middleName = 'Austin'
    name
      //=> { firstName: 'Leonard',
      #     lastName: 'Braithwaite',
      #     middleName: 'Austin' }

We have established that JavaScript's semantics allow for two different bindings to refer to the same value. For example:

    const allHallowsEve = [2012, 10, 31]
    const halloween = allHallowsEve;  
      
Both `halloween` and `allHallowsEve` are bound to the same array value within the local environment. And also:

    const allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      // ...
    })(allHallowsEve);

There are two nested environments, and each one binds a name to the exact same array value. In each of these examples, we have created two *aliases* for the same value. Before we could reassign things, the most important point about this is that the identities were the same, because they were the same value.

This is vital. Consider what we already know about shadowing:

    const allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      halloween = [2013, 10, 31];
    })(allHallowsEve);
    allHallowsEve
      //=> [2012, 10, 31]
      
The outer value of `allHallowsEve` was not changed because all we did was rebind the name `halloween` within the inner environment. However, what happens if we *mutate* the value in the inner environment?

    const allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      halloween[0] = 2013;
    })(allHallowsEve);
    allHallowsEve
      //=> [2013, 10, 31]
      
This is different. We haven't rebound the inner name to a different variable, we've mutated the value that both bindings share. Now that we've finished with mutation and aliases, let's have a look at it.
      
T> JavaScript permits the reassignment of new values to existing bindings, as well as the reassignment and assignment of new values to elements of containers such as arrays and objects. Mutating existing objects has special implications when two bindings are aliases of the same value.

T> Note well: Declaring a variable `const` does not prevent us from mutating its value, only from rebinding its name. This is an important distinction.