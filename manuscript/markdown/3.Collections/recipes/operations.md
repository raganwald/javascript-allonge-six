## Basic Operations on Iterables

Here are the operations we've defined on Iterables. As discussed, they preserve the collection sementics of the iterable they are given:

### operations that transform one iterable into another

{:lang="js"}
~~~~~~~~
const mapIterableWith = (fn, iterable) =>
  ({
    [Symbol.iterator]: function* () {
      for (let element of iterable) {
        yield fn(element);
      }
    }
  });
  
const mapAllIterableWith = (fn, iterable) =>
  ({
    [Symbol.iterator]: function* () {
      for (let element of iterable) {
        yield* fn(element);
      }
    }
  });
  
const filterIterableWith = (fn, iterable) =>
  ({
    [Symbol.iterator]: function* () {
      for (let element of iterable) {
        if (!!fn(element)) yield element;
      }
    }
  });
  
const compactIterable = (iterable) =>
  ({
    [Symbol.iterator]: function* () {
      for (let element of iterable) {
        if (element != null) yield element;
      }
    }
  });
  
const untilIterableWith = (fn, iterable) =>
  ({
    [Symbol.iterator]: function* () {
      for (let element of iterable) {
        if (fn(element)) break;
        yield fn(element);
      }
    }
  });
  
const restOfIterable = (iterable) => 
  ({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      
      iterator.next();
      return iterator;
    }
  });
  
const take = function* (numberToTake, iterable) {
  let remaining = numberToTake;
  
  for (let value of iterable) {
    if (remaining-- <= 0) break;
    yield value;
  }
}
~~~~~~~~

### operations that compose two or more iterables into an iterable

{:lang="js"}
~~~~~~~~
const zipIterables = (...iterables) =>
  ({
    [Symbol.iterator]: function * () {
      const iterators = iterables.map(i => i[Symbol.iterator]());
      
      while (true) {
        const pairs = iterators.map(j => j.next()),
              dones = pairs.map(p => p.done),
              values = pairs.map(p => p.value);
        
        if (dones.indexOf(true) >= 0) break;
        yield values;
      }
    }
  });
~~~~~~~~

### operations that transform an iterable into a value

{:lang="js"}
~~~~~~~~
const reduceIterableWith = (fn, seed, iterable) => {
    let accumulator = seed;

    for (let element in iterable) {
      accumulator = fn(accumulator, element);
    }
    return accumulator;
  };

const firstOfIterable = (iterable) =>
  iterable[Symbol.iterator]().next().value;
~~~~~~~~