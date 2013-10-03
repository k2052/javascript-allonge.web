# Recipes with Rebinding and References

![Café Diplomatico in Toronto's Little Italy](diplomatico.jpg)

### Disclaimer

The recipes are written for practicality, and their implementation may introduce JavaScript features that haven't been discussed in the text to this point, such as methods and/or prototypes. The overall *use* of each recipe will fit within the spirit of the language discussed so far, even if the implementations may not.

## Once

`once` is an extremely helpful combinator. It ensures that a function can only be called, well, *once*. Here's the recipe:

    function once (fn) {
      var done = false;
      
      return function () {
        return done ? void 0 : ((done = true), fn.apply(this, arguments))
      }
    }
{: .language-javascript}

Very simple! You pass it a function, and you get a function back. That function will call your function once, and thereafter will return `undefined` whenever it is called. Let's try it:

    var askedOnBlindDate = once(function () {
      return 'sure, why not?'
    });
    
    askedOnBlindDate()
      //=> 'sure, why not?'
      
    askedOnBlindDate()
      //=> undefined
      
    askedOnBlindDate()
      //=> undefined
{: .language-javascript}

It seems some people will only try blind dating once. But you do have to be careful that you are calling the function `once` returns multiple times. If you keep calling `once`, you'll get a new function that executes once, so you'll keep calling your function:

    once(function () {
      return 'sure, why not?'
    })()
      //=> 'sure, why not?'

    once(function () {
      return 'sure, why not?'
    })()
      //=> 'sure, why not?'
{: .language-javascript}

This is expected, but sometimes not what we want. So we must either be careful with our code, or use a variation, the [named once](#named-once) recipe.

## mapWith {#mapWith}

In recent versions of JavaScript, arrays have a `.map` method. Map takes a function as an argument, and applies it to each of the elements of the array, then returns the results in another array. For example:

    [1, 2, 3, 4, 5].map(function (n) { 
      return n*n 
    })
      //=> [1, 4, 9, 16, 25]
{: .language-javascript}
      
We say that `.map` *maps* its arguments over the receiver array's elements. Or if you prefer, that it defines a mapping between its receiver and its result. Libraries like [Underscore] provide a map *function*.[^why] It usually works like this:

    _.map([1, 2, 3, 4, 5], function (n) { 
      return n*n 
    })
      //=> [1, 4, 9, 16, 25]
{: .language-javascript}
      
[^why]: Why provide a map function? well, JavaScript is an evolving language, and when you're writing code that runs in a web browser, you may want to support browsers using older versions of JavaScript that didn't provide the `.map` function. One way to do that is to "shim" the map method into the Array class, the other way is to use a map function. Most library implementations of map will default to the `.map` method if its available.

This recipe isn't for `map`: It's for `mapWith`, a function that wraps around `map` and turns any other function into a mapping. In concept, `mapWith` is very simple:[^mapWith]

    function mapWith (fn) {
      return function (list) {
        return Array.prototype.map.call(list, function (something) {
          return fn.call(this, something);
        });
      };
    };
{: .language-javascript}

Here's the above code written using `mapWith`:

    var squareMap = mapWith(function (n) { 
      return n*n;
    });
    
    squareMap([1, 2, 3, 4, 5])
      //=> [1, 4, 9, 16, 25]
{: .language-javascript}
      
If we didn't use `mapWith`, we'd have written something like this:

    var squareMap = function (array) {
      return Array.prototype.map.call(array, function (n) { 
        return n*n;
      });
    };
{: .language-javascript}
    
And we'd do that every time we wanted to construct a method that maps an array to some result. `mapWith` is a very convenient abstraction for a very common pattern.

*`mapWith` was suggested by [ludicast](http://github.com/ludicast)*
    
[Underscore]: http://underscorejs.org

[^mapWith]: If we were always mapWithting arrays, we could write `list.map(fn)`. However, there are some objects that have a `.length` property and `[]` accessors that can be mapWithted but do not have a `.map` method. `mapWith` works with those objects. This points to a larger issue around the question of whether containers really ought to implement methods like `.map`. In a language like JavaScript, we are free to define objects that know about their own implementations, such as exactly how `[]` and `.length` works and then to define standalone functions that do the rest.

## Flip {#flip}

When we wrote [mapWith](#mapWith), we wrote it like this:

    function mapWith (fn) {
      return function (list) {
        return Array.prototype.map.call(list, function (something) {
          return fn.call(this, something);
        });
      };
    };
{: .language-javascript}

Let's consider the case whether we have a `map` function of our own, perhaps from the [allong.es](http://allong.es) library, perhaps from [Underscore](http://underscorejs.org). We could write our function something like this:

    function mapWith (fn) {
      return function (list) {
        return map.call(list, fn);
      };
    };
{: .language-javascript}

Looking at this, we see we're conflating two separate transformations. First, we're reversing the order of arguments. You can see that if we simplify it:

    function mapWith (fn, list) {
      return map.call(list, fn);
    };
{: .language-javascript}

Second, we're "currying" the function so that instead of defining a function that takes two arguments, it returns a function that takes the first argument and returns a function that takes the second argument and applies them both, like this:

    function mapCurried (list) {
      return function (fn) {
        return map(list, fn);
      };
    };  
{: .language-javascript}
      
Let's return to the implementation that does both:

    function mapWith (fn) {
      return function (list) {
        return map.call(list, fn);
      };
    };
{: .language-javascript}
    
Now let's put a wrapper around it:

    function wrapper () {
      return function (fn) {
        return function (list) {
          return map.call(list, fn);
        };
      };
    };
{: .language-javascript}
    
Abstract the parameter names:

    function wrapper () {
      return function (first) {
        return function (second) {
          return map.call(second, first);
        };
      };
    };
{: .language-javascript}
    
And finally, extract the function as a parameter:

    function wrapper (fn) {
      return function (first) {
        return function (second) {
          return fn.call(second, first);
        };
      };
    };
{: .language-javascript}
    
What we have now is a function that takes a function and "flips" the order of arguments around, then curries it:

    function flip (fn) {
      return function (first) {
        return function (second) {
          return fn.call(second, first);
        };
      };
    };
{: .language-javascript}

This is gold. Consider how we define [mapWith](#mapWith) now:

    var mapWith = flip(map);
{: .language-javascript}
    
Much nicer!

There's one final decoration. Sometimes we'll want to flip a function but retain the flexibility to call it with both parameters at once. No problem:

    function flip (fn) {
      return function (first, second) {
        if (arguments.length === 2) {
          return fn.call(second, first); 
        }
        else {
          return function (second) {
            return fn.call(second, first);
          };
        };
      };
    };
{: .language-javascript}

Now you can call `mapWith(fn, list)` or `mapWith(fn)(list)`, your choice.

## Extend {#extend}

It's very common to want to "extend" an object by adding properties to it:

    var inventory = {
      apples: 12,
      oranges: 12
    };
    
    inventory.bananas = 54;
    inventory.pears = 24;
{: .language-javascript}

It's also common to want to add a [shallow copy] of the properties of one object to another:

[shallow copy]: https://en.wikipedia.org/wiki/Object_copy#Shallow_copy

      for (var fruit in shipment) {
        inventory[fruit] = shipment[fruit]
      }
{: .language-javascript}

Both needs can be met with this recipe for `extend`:

    var extend = variadic( function (consumer, providers) {
      var key,
          i,
          provider;
      
      for (i = 0; i < providers.length; ++i) {
        provider = providers[i];
        for (key in provider) {
          if (provider.hasOwnProperty(key)) {
            consumer[key] = provider[key]
          }
        }
      }
      return consumer
    });
{: .language-javascript}
    
You can copy an object by extending an empty object:

    extend({}, {
      apples: 12,
      oranges: 12
    })
      //=> { apples: 12, oranges: 12 }
{: .language-javascript}

You can extend one object with another:

    var inventory = {
      apples: 12,
      oranges: 12
    };
    
    var shipment = {
      bananas: 54,
      pears: 24
    }
    
    extend(inventory, shipment)
      //=> { apples: 12,
      //     oranges: 12,
      //     bananas: 54,
      //     pears: 24 }
{: .language-javascript}
      
And when we discuss prototypes, we will use `extend` to turn this:

    var Queue = function () {
      this.array = [];
      this.head = 0;
      this.tail = -1
    };
      
    Queue.prototype.pushTail = function (value) {
      // ...
    };
    Queue.prototype.pullHead = function () {
      // ...
    };
    Queue.prototype.isEmpty = function () {
      // ...
    }
{: .language-javascript}

Into this:

    var Queue = function () {
      extend(this, {
        array: [],
        head: 0,
        tail: -1
      })
    };
      
    extend(Queue.prototype, {
      pushTail: function (value) {
        // ...
      },
      pullHead: function () {
        // ...
      },
      isEmpty: function () {
        // ...
      }      
    });
{: .language-javascript}

## Why? {#y}

This is the [canonical Y Combinator][y]:

    function Y (f) {
      return ((function (x) {
        return f(function (v) {
          return x(x)(v);
        });
      })(function (x) {
        return f(function (v) {
          return x(x)(v);
        });
      }));
    }
{: .language-javascript}

You use it like this:

    var factorial = Y(function (fac) {
      return function (n) {
        return (n == 0 ? 1 : n * fac(n - 1));
      }
    });
 
    factorial(5)
      //=> 120
{: .language-javascript}

Why? It enables you to make recursive functions without needing to bind a function to a name in an environment. This has little practical utility in JavaScript, but in combinatory logic it's essential: With fixed-point combinators it's possible to compute everything computable without binding names.

So again, why include the recipe? Well, besides all of the practical applications that combinators provide, there is this little thing called *The joy of working things out.*

There are many explanations of the Y Combinator's mechanism on the internet, but resist the temptation to read any of them: Work it out for yourself. Use it as an excuse to get familiar with your environment's debugging facility. A friendly tip: Name some of the anonymous functions inside it to help you decipher stack traces.

Work things out for yourself. And once you've grokked that recipe, this recipe is for a Y Combinator that is a little more idiomatic. Work it out too:

    function Y (fn) {
      var f = function (f) {
        return function () {
          return fn.apply(f, arguments)
        }
      };
      
      return ((function (x) {
        return f(function (v) {
          return x(x)(v);
        });
      })(function (x) {
        return f(function (v) {
          return x(x)(v);
        });
      }));
    }
{: .language-javascript}

You use this version like this:

    var factorial = Y(function (n) {
      return (n == 0 ? 1 : n * this(n - 1));
    });
 
    factorial(5)
{: .language-javascript}

There are certain cases involving nested recursive functions it cannot handle due to the ambiguity of `this`, and obviously it is useless as a method combination, but it is an interesting alternative to the `let` pattern.

[y]: https://en.wikipedia.org/wiki/Fixed-point_combinator#Example_in_JavaScript "Call-by-value fixed-point combinator in JavaScript"
