## Copy on Write {#cow}

![The Coffee Cow](images/coffee-cow.jpg)

We've seen how to build lists with arrays and with linked lists. We've touched on an important difference between them:

* When you take the rest of an array with destructuring (`[first, ...rest]`), you are given a *copy* of the elements of the array.
* When you take the rest of a linked list with its reference, you are given the exact same nodes of the elements of the original list.

The consequence of this is that if you have an array, and you take it's "rest," your "child" array is a copy of the elements of the parent array. And therefore, modifications to the parent do not affect the child, and modifications to the child do not affect the parent.

Whereas if you have a linked list, and you take it's "rest," your "child" list shares its nodes with the "parent" list. And therefore, modifications to the parent also modify the child, and modifications to the child also modify the parent.

Let's confirm our understanding:

{:lang="javascript"}
~~~~~~~~
const parentArray = [1, 2, 3];
const [aFirst, ...childArray] = parentArray;

parentArray[2] = "three";
childArray[0] = "two";

parentArray
  //=> [1,2,"three"]
childArray
  //=> ["two",3]

const EMPTY = { first: {}, rest: {} };
const parentList = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY }}};
const childList = parentList.rest;

parentList.rest.rest.first = "three";
childList.first = "two";

parentList
  //=> {"first":1,"rest":{"first":"two","rest":{"first":"three","rest":{"first":{},"rest":{}}}}}
childList
  //=> {"first":"two","rest":{"first":"three","rest":{"first":{},"rest":{}}}}
~~~~~~~~

This is remarkably unsafe. If we *know* that a list doesn't share any elements with another list, we can safely modify it. But how do we keep track of that? Add a bunch of bookkeeping to track references? We'll end up reinventing reference counting and garbage collection.

### a few utilities

before we go any further, let's write a few naïve list utilities so that we can work at a slightly higher level of abstraction:

{:lang="javascript"}
~~~~~~~~
const copy = (node, head = null, tail = null) => {
  if (node === EMPTY) {
    return head;
  }
  else if (tail === null) {
    const { first, rest } = node;
    const newNode = { first, rest };
    return copy(rest, newNode, newNode);
  }
  else {
    const { first, rest } = node;
    const newNode = { first, rest };
    tail.rest = newNode;
    return copy(node.rest, head, newNode);
  }
}

const first = ({first, rest}) => first;
const rest = ({first, rest}) => rest;

const reverse = (node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : reverse(rest(node), { first: first(node), rest: delayed });

const mapWith = (fn, node, delayed = EMPTY) =>
  node === EMPTY
    ? reverse(delayed)
    : mapWith(fn, rest(node), { first: fn(first(node)), rest: delayed });

const at = (index, list) =>
  index === 0
    ? first(list)
    : at(index - 1, rest(list));
    
const set = (index, value, list, originalList = list) =>
  index === 0
    ? (list.first = value, originalList)
    : set(index - 1, value, rest(list), originalList)
    
const parentList = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY }}};
const childList = rest(parentList);

set(2, "three", parentList);
set(0, "two", childList);

parentList
  //=> {"first":1,"rest":{"first":"two","rest":{"first":"three","rest":{"first":{},"rest":{}}}}}
childList
  //=> {"first":"two","rest":{"first":"three","rest":{"first":{},"rest":{}}}}
~~~~~~~~

Our new `at` and `set` functions behave similarly to `array[index]` and `array[index] = value`. The main difference is that `array[index] = value` evaluates to `value`, while `set(index, value, list)` evaluates to the modified `list`.
  
### copy-on-read

So back to the problem of structure sharing. One strategy for avoiding problems is to be *pessimistic*. Whenever we take the rest of a list, make a copy.

{:lang="javascript"}
~~~~~~~~
const rest = ({first, rest}) => copy(rest);

const parentList = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY }}};
const childList = rest(parentList);

const newParentList = set(2, "three", parentList);
set(0, "two", childList);

parentList
  //=> {"first":1,"rest":{"first":2,"rest":{"first":"three","rest":{"first":{},"rest":{}}}}}
childList
  //=> {"first":"two","rest":{"first":3,"rest":{"first":{},"rest":{}}}}
~~~~~~~~

This strategy is called "copy-on-read", because when we attempt the parent  to "read" the value of a child of the list, we make a copy and read the copy of the child. Thereafter, we can write to the parent or the copy of the child freely.

As we expected, making a copy lets us modify the copy without interfering with the original. This is, however, expensive. Sometimes we don't need to make a copy because we won't be modifying the list. Our `mapWith` function would be very expensive if we make a copy every time we call `rest(node)`.

There's also a bug: What happens when we modify the first element of a list? But before we fix that, let's try being lazy about copying.

### copy-on-write

Why are we copying? In case we modify a child list. Ok, what if we do this: Make the copy when we know we are modifying the list. When do we know that? When we call `set`. We'll restore our original definition for `rest`, but change `set`:

{:lang="javascript"}
~~~~~~~~
const rest = ({first, rest}) => rest;

const set = (index, value, list) =>
  index === 0
    ? { first: value, rest: list.rest }
    : { first: list.first, rest: set(index - 1, value, list.rest) };

const parentList = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY }}};
const childList = rest(parentList);

const newParentList = set(2, "three", parentList);
const newChildList = set(0, "two", childList);
~~~~~~~~

Our original parent and child lists remain unmodified:

{:lang="javascript"}
~~~~~~~~
parentList
  //=> {"first":1,"rest":{"first":2,"rest":{"first":3,"rest":{"first":{},"rest":{}}}}}
childList
  //=> {"first":2,"rest":{"first":3,"rest":{"first":{},"rest":{}}}}
~~~~~~~~

But our new parent and child lists are copies that contain the desired modifications, without interfering with each other:

{:lang="javascript"}
~~~~~~~~
newParentList
  //=> {"first":1,"rest":{"first":2,"rest":{"first":"three","rest":{"first":{},"rest":{}}}}}
newChildList
  //=> {"first":"two","rest":{"first":3,"rest":{"first":{},"rest":{}}}}
~~~~~~~~

And now functions like `mapWith` that make copies without modifying anything, work at full speed.

This strategy of waiting to copy until you are writing is called copy-on-write, or "COW:"

> Copy-on-write is the name given to the policy that whenever a task attempts to make a change to the shared information, it should first create a separate (private) copy of that information to prevent its changes from becoming visible to all the other tasks.—[Wikipedia][Copy-on-write]

[Copy-on-write]: https://en.wikipedia.org/wiki/Copy-on-write

Like all strategies, it makes a tradeoff: It's much cheaper than pessimistically copying structures when you make an infrequent number of small changes, but if you tend to make a lot of changes to some that you aren't sharing, it's more expensive.

Looking at the code again, you see that the `copy` function doesn't copy on write: It follows the pattern that while constructing something, we own it and can be liberal with mutation. Once we're done with it and give it to someone else, we need to be conservative and use a strategy like copy-on-read or copy-on-write.