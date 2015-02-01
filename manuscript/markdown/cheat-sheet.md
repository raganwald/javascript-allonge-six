# The Recipe Cheat Sheet {#cheatsheet}

In the recipes, you may see one or more of the following JavaScript constructs being used before being fully explained in the text. Here're some brief explanations to tide you over:

### apply and call

Functions are applied with `()`. But they also have *methods* for applying them to arguments. `.call` and `.apply` are explained when we discuss [function contexts](#context), but here are some examples:

    const plus = (a, b) => a + b;
    
    plus(2, 3) 
      //=> 5
      
    plus.call(this, 2, 3)
      //=> 5
      
    plus.apply(this, [2, 3])
      //=> 5

### slice

Arrays have a `.slice` method. The function can always be found at `Array.prototype.slice`. It works like this:

    [1, 2, 3, 4, 5].slice(0)
      //=> [1, 2, 3, 4, 5]
      
    [1, 2, 3, 4, 5].slice(1)
      //=> [2, 3, 4, 5]
      
    [1, 2, 3, 4, 5].slice(1, 4)
      //=> [2, 3, 4]

Note that `slice` always creates a new array, so `.slice(0)` makes a copy of an array. The [arguments](#arguments-again) pseudo-variable is not an array, but you can use `.slice` with it like this to get an array of all or some of the arguments:

    Array.prototype.slice.call(arguments, 0)
      //=> returns the arguments in an array.
      
    const butFirst = function () {
      return Array.prototype.slice.call(arguments, 1)
    }
    
    butFirst('a', 'b', 'c', 'd')
      //=> [ 'b', 'c', 'd' ]
      
For simplicity and as a small speed improvement, `slice` is usually bound to a local variable:

    const __slice = Array.prototype.slice;
      
    const butFirst = function () {
      return __slice.call(arguments, 1)
    }
    
Or even:
    
    const slice = (list, from, to) => Array.prototype.slice.call(list, from, to);
      
    const butFirst = function () {
      return slice(arguments, 1)
    }
    
### concat

Arrays have another useful method, `.concat`. Concat returns an array created by concatenating the receiver with its argument:

    [1, 2, 3].concat([2, 1])
      //=> [1, 2, 3, 2, 1]
      
### function lengths

Functions have a `.length` property that counts the number of arguments declared:

    ((a, b, c) => a + b + c).length
      //=> 3