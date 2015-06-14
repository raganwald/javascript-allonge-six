## Why? {#y}

This is the [canonical Y Combinator][y]:

    const Y = (f) =>
      ( x => f(v => x(x)(v)) )(
        x => f(v => x(x)(v))
      );

You use it like this:

    const factorial = Y(function (fac) {
      return function (n) {
        return (n == 0 ? 1 : n * fac(n - 1));
      }
    });
 
    factorial(5)
      //=> 120

Why? It enables you to make recursive functions without needing to bind a function to a name in an environment. This has little practical utility in JavaScript, but in combinatory logic it's essential: With fixed-point combinators it's possible to compute everything computable without binding names.

So again, why include the recipe? Well, besides all of the practical applications that combinators provide, there is this little thing called *The joy of working things out.*

There are many explanations of the Y Combinator's mechanism on the internet, but resist the temptation to read any of them: Work it out for yourself. Use it as an excuse to get familiar with your environment's debugging facility.

One tip is to use JavaScript to name things. For example, you could start by writing:

    const Y = (f) => {
      const something = x => f(v => x(x)(v));
      
      return something(something);
    };

What is this `something` and how does it work? Another friendly tip: Change some of the fat arrow functions inside of it into named function expressions to help you decipher stack traces.

Work things out for yourself!

[y]: https://en.wikipedia.org/wiki/Fixed-point_combinator#Example_in_JavaScript "Call-by-value fixed-point combinator in JavaScript"