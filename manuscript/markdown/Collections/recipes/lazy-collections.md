## Lazy Collection Mixin

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

const LazyCollection = {
  map(fn) {
    const original = this;
    
    return extend({
      *[Symbol.iterator] () {
        for (let element of original) {
          yield fn(element);
        }
      }
    }, LazyCollection);
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
    
    return extend({
      *[Symbol.iterator] () {
        for (let element of original) {
          if (fn(element)) yield element;
        }
      }
    }, LazyCollection);
  },

  find(fn) {
    for (let element of this) {
      if (fn(element)) return element;
    }
  },

  until(fn) {
    const original = this;
    
    return extend({
      *[Symbol.iterator] () {
        for (let element of original) {
          if (fn(element)) break;
          yield element;
        }
      }
    }, LazyCollection);
  },

  first() {
    return this[Symbol.iterator]().next().value;
  },

  rest() {
    return extend({
      [Symbol.iterator]: () => {
        const iterator = this[Symbol.iterator]();

        iterator.next();
        return iterator;
      }
    }, LazyCollection);
  },

  take(numberToTake) {
    const original = this;
    let numberRemaining = numberToTake;
    
    return extend({
      *[Symbol.iterator] () {
        for (let element of original) {
          if (numberRemaining-- <= 0) break;
          yield element;
        }
      }
    }, LazyCollection);
  }
}
~~~~~~~~