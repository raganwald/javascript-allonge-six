# A Coffeehouse: Symbols {#symbols}

!["Uniqueness" is an important quality in society and in programs.](images/tiny.jpg)

Programmers often spend a lot of time trying to define "sameness:" JavaScript programmers know that `"foo" === "foo"` is always true, but `new String("foo") === new String("foo")` is always false, and how tricky it is to define what we mean when we say that `{ foo: "bar" }` is *semantically equivalent* to `{ foo: "bar" }`.

Programmers don't think about it quite as much, but entities being different from each other is also important. We know that `function () {} !== function () {}`. But having objects that we know to be different from each other can be very useful.

> Any sufficiently complicated C or Fortran program contains an ad hoc, informally-specified, bug-ridden, slow implementation of half of Common Lisp.--[Greenspun's Tenth Rule](https://en.wikipedia.org/wiki/Greenspun's_tenth_rule)

In older versions of JavaScript, programmers would hack together unique objects, using timestamps, GUIDS, counters and other techniques. None of which are individually wrong, but when there are 99 different ways to do the same thing that everybody ends up doing, the important parts of our code become obfuscated under the weight of our ad hoc, informally-specified, bug-ridden, slow implementations of Common Lisp's [gensym](http://www.lispdoc.com/?q=gensym).

So `Symbol` was added to the language. In its simplest form, `Symbol` is a function that returns a unique entity. No two symbols are alike, ever:

~~~~~~~~
Symbol() !=== Symbol()
~~~~~~~~

Symbols have string representations, although they may appear cryptic:[^impl]

[^impl]: The exact representation depends upon the implementation

~~~~~~~~
Symbol().toString()
  //=> Symbol(undefined)_u.mwf0blvw5
Symbol().toString()
  //=> Symbol(undefined)_s.niklxrko8m
Symbol().toString()
  //=> Symbol(undefined)_s.mbsi4nduh
~~~~~~~~

You can add your own text to help make it intelligible:

~~~~~~~~
Symbol("Allongé").toString()
  //=> Symbol(Allongé)_s.52x692eab
Symbol("Allongé").toString()
  //=> Symbol(Allongé)_s.q6hq5lx01p
Symbol("Allongé").toString()
  //=> Symbol(Allongé)_s.jii7eyiyza
~~~~~~~~

There are some ways that JavaScript makes symbols especially handy. Using symbols as property names, for example.

### privacy with symbols {#privacy-with-symbols}

When we use a symbol as a property name, it is automatically unique and non-enumerable. It is still possible to discover its existence and retrieve its value, but it is not possible for accidentally access or overwrite a property that uses a symbol as its key.

Therefore, we can give objects private properties with symbols. Consider this:

~~~~~~~~
const Queue = () =>
  ({
    array: [], 
    head: 0, 
    tail: -1,
    pushTail (value) {
      return this[array][this[tail] += 1] = value
    },
    pullHead () {
      if (this[tail] >= this[head]) {
        let value = this[array][this[head]];
        
        this[array][this[head]] = undefined;
        this[head] += 1;
        return value
      }
    },
    isEmpty () {
      return this[tail] < this[head]
    }
  });

let q = Queue();
q.pushTail('hello');
q.pushTail('symbols');

q.pullHead()
  //=> 'hello'
  
q
  //=> {"array":["hello","symbols"],"head":0,"tail":1}
  
q.tail
  //=> 1
~~~~~~~~

Because we used compact method syntax, the `pushTail`, `pullHead`, and `isEmpty` properties are not "enumerable," so they don't show up in the console. But other code can access them. The `array`, `head`, and `tail` properties are enumerable and accessible.

Let's use symbols for these properties instead:

~~~~~~~~
const array = Symbol(),
      head  = Symbol(),
      tail  = Symbol();
      
const Queue = () =>
  ({
    [array]: [], 
    [head]: 0, 
    [tail]: -1,
    pushTail (value) {
      return this[array][this[tail] += 1] = value
    },
    pullHead () {
      if (this[tail] >= this[head]) {
        let value = this[array][this[head]];
        
        this[array][this[head]] = undefined;
        this[head] += 1;
        return value
      }
    },
    isEmpty () {
      return this[tail] < this[head]
    }
  });

let q = Queue();
q.pushTail('hello');
q.pushTail('symbols');

q.pullHead()
  //=> 'hello'
  
q
  //=> {}
  
q.tail
  //=> undefined
~~~~~~~~

Now the `array`, `head`, and `tail` properties are not enumerable and they aren't accessible by those names because they're actually symbols assigned to the `array`, `head`, and `tail` variables.
