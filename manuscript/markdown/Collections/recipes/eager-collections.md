## An Eager Collection Mixin

{:lang="js"}
~~~~~~~~
const extend = function (consumer, ...providers) {
  for (let i = 0; i < providers.length; ++i) {
    const provider = providers[i];
    for (let key in provider) {
      if (provider.hasOwnProperty(key)) {
        consumer[key] = provider[key]
      }
    }
  }
  return consumer
};

const EagerCollection = (gatherable) =>
  ({
    map(fn) {
      const  original = this;
      
      return gatherable.from(
        (function* () {
          for (let element of original) {
            yield fn(element);
          }
        })()
      );
    },

    reduce(fn, seed) {
      let accumulator = seed;

      for(let element of this) {
        accumulator = fn(accumulator, element);
      }
      return accumulator;
    },

    filter(fn) {
      const original = this;
    
      return gatherable.from(
        (function* () {
          for (let element of original) {
            if (fn(element)) yield element;
          }
        })()
      );
    },

    find(fn) {
      for (let element of this) {
        if (fn(element)) return element;
      }
    },

    until(fn) {
      const original = this;
    
      return gatherable.from(
        (function* () {
          for (let element of original) {
            if (fn(element)) break;
            yield element;
          }
        })()
      );
    },

    first() {
      return this[Symbol.iterator]().next().value;
    },

    rest() {
      const iteration = this[Symbol.iterator]();
      
      iteration.next();
      return gatherable.from(
        (function* () {
          yield* iteration;
        })()
      );
      return gatherable.from(iterable);
    },

    take(numberToTake) {
      const original = this;
      let numberRemaining = numberToTake;
    
      return gatherable.from(
        (function* () {
          for (let element of original) {
            if (numberRemaining-- <= 0) break;
            yield element;
          }
        })()
      );
    }
  })
~~~~~~~~
  
{:lang="js"}
~~~~~~~~
const EMPTY = {
  isEmpty: () => true
};

const isEmpty = (node) => node === EMPTY;

const Pair = (car, cdr = EMPTY) =>
  extend({
    car,
    cdr,
    isEmpty: () => false,
    [Symbol.iterator]: function () {
      let currentPair = this;
      
      return {
        next: () => {
          if (currentPair.isEmpty()) {
            return {done: true}
          }
          else {
            const value = currentPair.car;
            
            currentPair = currentPair.cdr;
            return {done: false, value}
          }
        }
      }
    }
  }, EagerCollection(Pair));

Pair.from = (iterable) =>
  (function interationToList (iteration) {
    const {done, value} = iteration.next();
    
    return done ? EMPTY : Pair(value, interationToList(iteration));
  })(iterable[Symbol.iterator]());
~~~~~~~~