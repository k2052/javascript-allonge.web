# Recipes with Basic Functions

![Before combining ingredients, begin with implements so clean, they gleam.](mirage.jpg)

Having looked at basic pure functions and closures, we're going to see some practical recipes that focus on the premise of functions that return functions.

### Disclaimer

The recipes are written for practicality, and their implementation may introduce JavaScript features that haven't been discussed in the text to this point, such as methods and/or prototypes. The overall *use* of each recipe will fit within the spirit of the language discussed so far, even if the implementations may not.

## Partial Application {#simple-partial}

In [Building Blocks](#buildingblocks), we discussed partial application, but we didn't write a generalized recipe for it. This is such a common tool that many libraries provide some form of partial application tool. You'll find examples in [Lemonad](https://github.com/fogus/lemonad) from Michael Fogus, [Functional JavaScript](http://osteele.com/sources/javascript/functional/) from Oliver Steele and the terse but handy [node-ap](https://github.com/substack/node-ap) from James Halliday.

These two recipes are for quickly and simply applying a single argument, either the leftmost or rightmost.[^inspired] If you want to bind more than one argument, or you want to leave a "hole" in the argument list, you will need to either use a [generalized partial recipe](#partial), or you will need to repeatedly apply arguments. It is [context](#context)-agnostic.

    var __slice = Array.prototype.slice;
    
    function callFirst (fn, larg) {
      return function () {
        var args = __slice.call(arguments, 0);
        
        return fn.apply(this, [larg].concat(args))
      }
    }
{: .language-javascript}

    function callLast (fn, rarg) {
      return function () {
        var args = __slice.call(arguments, 0);
        
        return fn.apply(this, args.concat([rarg]))
      }
    }
    
    function greet (me, you) {
      return "Hello, " + you + ", my name is " + me
    }
    
    var heliosSaysHello = callFirst(greet, 'Helios');
    
    heliosSaysHello('Eartha')
      //=> 'Hello, Eartha, my name is Helios'
      
    var sayHelloToCeline = callLast(greet, 'Celine');
    
    sayHelloToCeline('Eartha')
      //=> 'Hello, Celine, my name is Eartha'
{: .language-javascript}
      
As noted above, our partial recipe allows us to create functions that are partial applications of functions that are context aware. We'd need a different recipe if we wish to create partial applications of object methods.

[^inspired]: `callFirst` and `callLast` were inspired by Michael Fogus' [Lemonad](https://github.com/fogus/lemonad). Thanks!

## Ellipses and improved Partial Application {#ellipses}

The CoffeeScript programming language has a useful feature: If a parameter of a method is written with trailing ellipses, it collects a list of parameters into an array. It can be used in various ways, and the CoffeeScript transpiler does some pattern matching to sort things out, but 80% of the use is to collect a variable number of arguments without using the `arguments` pseudo-variable, and 19% of the uses are to collect a trailing list of arguments.

Here's what it looks like collecting a variable number of arguments and trailing arguments:

    callLeft = (fn, args...) ->
      (remainingArgs...) ->
        fn.apply(this, args.concat(remainingArgs))
{: .language-coffeescript}

These are very handy features. Here's our bogus, made-up attempt to write our own mapper function:

    mapper = (fn, elements...) ->
      elements.map(fn)

    mapper ((x) -> x*x), 1, 2, 3
      #=> [1, 4, 9]

    squarer = callLeft mapper, (x) -> x*x

    squarer 1, 2, 3
      #=> [1, 4, 9]
{: .language-coffeescript}

JavaScript doesn't support [ellipses](http://en.wikipedia.org/wiki/Ellipsis), those trailing periods CoffeeScript uses to collect arguments into an array. JavaScript is a *functional* language, so here is the recipe for a function that collects trailing arguments into an array for us:

    var __slice = Array.prototype.slice;

    function variadic (fn) {
      var fnLength = fn.length;

      if (fnLength < 1) {
        return fn;
      }
      else if (fnLength === 1)  {
        return function () {
          return fn.call(
            this, __slice.call(arguments, 0))
        }
      }
      else {
        return function () {
          var numberOfArgs = arguments.length,
              namedArgs = __slice.call(
                arguments, 0, fnLength - 1),
              numberOfMissingNamedArgs = Math.max(
                fnLength - numberOfArgs - 1, 0),
              argPadding = new Array(numberOfMissingNamedArgs),
              variadicArgs = __slice.call(
                arguments, fn.length - 1);

          return fn.apply(
            this, namedArgs
                  .concat(argPadding)
                  .concat([variadicArgs]));
        }
      }
    };

    function unary (first) {
      return first
    }

    unary('why', 'hello', 'there')
      //=> 'why'
  
    variadic(unary)('why', 'hello', 'there')
      //=> [ 'why', 'hello', 'there' ]
  
    function binary (first, rest) {
      return [first, rest]
    }

    binary('why', 'hello', 'there')
      //=> [ 'why', 'hello' ]

    variadic(binary)('why', 'hello', 'there')
      //=> [ 'why', [ 'hello', 'there' ] ]
{: .language-javascript}

Here's what we write to create our partial application functions gently:

    var callLeft = variadic( function (fn, args) {
      return variadic( function (remainingArgs) {
        return fn.apply(this, args.concat(remainingArgs))
      })
    })

    // Let's try it!

    var mapper = variadic( function (fn, elements) {
      return elements.map(fn)
    });

    mapper(function (x) { return x * x }, 1, 2, 3)
      //=> [1, 4, 9]

    var squarer = callLeft(mapper, function (x) { return x * x });

    squarer(1, 2, 3)
      //=> [1, 4, 9]
{: .language-javascript}

While we're at it, here's our implementation of `callRight` using the same technique:

    var callRight = variadic( function (fn, args) {
      return variadic( function (precedingArgs) {
        return fn.apply(this, precedingArgs.concat(args))
      })
    })
{: .language-javascript}

Fine print: Of course, `variadic` introduces an extra function call and may not be the best choice in a highly performance-critical piece of code. Then again, using `arguments` is considerably slower than directly accessing argument bindings, so if the performance is that critical, maybe you shouldn't be using a variable number of arguments in that section.

## Unary

In [Ellipses](#ellipses), we saw a function decorator that takes a function with a fixed number of arguments and turns it into a *variadic* function, a function taking any number of arguments. "Unary" is a another function decorator, and it also modifies the number of arguments a function takes: Unary takes any function and turns it into a function taking exactly one argument.

The most common use case is to fix a common problem. JavaScript has a `.map` method for arrays, and many libraries offer a `map` function with the same semantics. Here it is in action:

    ['1', '2', '3'].map(parseFloat)
      //=> [1, 2, 3]
{: .language-javascript}
      
In that example, it looks exactly like the mapping function you'll find in most languages: You pass it a function, and it calls the function with one argument, the element of the array. However, that's not the whole story. JavaScript's `map` actually calls each function with *three* arguments: The element, the index of the element in the array, and the array itself.

Let's try it:

    [1, 2, 3].map(function (element, index, arr) {
      console.log({element: element, index: index, arr: arr})
    })
      //=> { element: 1, index: 0, arr: [ 1, 2, 3 ] }
      //   { element: 2, index: 1, arr: [ 1, 2, 3 ] }
      //   { element: 3, index: 2, arr: [ 1, 2, 3 ] }
{: .language-javascript}
      
If you pass in a function taking only one argument, it simply ignores the additional arguments. But some functions have optional second or even third arguments. For example:

    ['1', '2', '3'].map(parseInt)
      //=> [1, NaN, NaN]
{: .language-javascript}

This doesn't work because `parseInt` is defined as `parseInt(string[, radix])`. It takes an optional radix argument. And when you call `parseInt` with `map`, the index is interpreted as a radix. Not good! What we want is to convert `parseInt` into a function taking only one argument.

We could write `['1', '2', '3'].map(function (s) { return parseInt(s); })`, or we could come up with a decorator to do the job for us:

    function unary (fn) {
      if (fn.length == 1) {
        return fn
      }
      else return function (something) {
        return fn.call(this, something)
      }
    }
{: .language-javascript}

And now we can write:

    ['1', '2', '3'].map(unary(parseInt))
      //=> [1, 2, 3]
{: .language-javascript}
      
Presto!

## Tap {#tap}

One of the most basic combinators is the "K Combinator," nicknamed the "kestrel:"

    function K (x) {
      return function (y) {
        return x
      }
    };
{: .language-javascript}

It has some surprising applications. One is when you want to do something with a value for side-effects, but keep the value around. Behold:

    function tap (value) {
      return function (fn) {
        if (typeof(fn) === 'function') {
          fn(value)
        }
        return value
      }
    }
{: .language-javascript}

`tap` is a traditional name borrowed from various Unix shell commands. It takes a value and returns a function that always returns the value, but if you pass it a function, it executes the function for side-effects. Let's see it in action as a poor-man's debugger:

    var drink = tap('espresso')(function (it) {
      console.log("Our drink is", it) 
    });
    
    // outputs "Our drink is 'espresso'" to the console
{: .language-javascript}

It's easy to turn off:

    var drink = tap('espresso')();
    
    // doesn't output anything to the console
{: .language-javascript}

Libraries like [Underscore] use a version of `tap` that is "uncurried:"

    var drink = _.tap('espresso', function () { 
      console.log("Our drink is", this) 
    });
{: .language-javascript}
    
Let's enhance our recipe so it works both ways:

    function tap (value, fn) {
      if (fn === void 0) {
        return curried
      }
      else return curried(fn);
      
      function curried (fn) {
        if (typeof(fn) === 'function') {
          fn(value)
        }
        return value
      }
    }
{: .language-javascript}

Now you can write:

    var drink = tap('espresso')(function (it) { 
      console.log("Our drink is", it) 
    });
{: .language-javascript}
    
Or:

    var drink = tap('espresso', function (it) { 
      console.log("Our drink is", it) 
    });
{: .language-javascript}
    
And if you wish it to do noting at all, You can write either:

    var drink = tap('espresso')();
{: .language-javascript}

Or:

    var drink = tap('espresso', null);
{: .language-javascript}

`tap` can do more than just act as a debugging aid. It's also useful for working with [object and instance methods](#tap-methods).

[Underscore]: http://underscorejs.org

## Maybe {#maybe}

A common problem in programming is checking for `null` or `undefined` (hereafter called "nothing," while all other values including `0`, `[]` and `false` will be called "something"). Languages like JavaScript do not strongly enforce the notion that a particular variable or particular property be something, so programs are often written to account for values that may be nothing.

This recipe concerns a pattern that is very common: A function `fn` takes a value as a parameter, and its behaviour by design is to do nothing if the parameter is nothing:

    function isSomething (value) {
      return value !== null && value !== void 0;
    }

    function checksForSomething (value) {
      if (isSomething(value)) {
        // function's true logic
      }
    }
{: .language-javascript}

Alternately, the function may be intended to work with any value, but the code calling the function wishes to emulate the behaviour of doing nothing by design when given nothing:

    var something = isSomething(value) ? 
      doesntCheckForSomething(value) : value;
{: .language-javascript}
    
Naturally, there's a recipe for that, borrowed from Haskell's [maybe monad][maybe], Ruby's [andand], and CoffeeScript's existential method invocation:

    function maybe (fn) {
      return function () {
        var i;
        
        if (arguments.length === 0) {
          return
        }
        else {
          for (i = 0; i < arguments.length; ++i) {
            if (arguments[i] == null) return
          }
          return fn.apply(this, arguments)
        }
      }
    }
{: .language-javascript}

`maybe` reduces the logic of checking for nothing to a function call, either:

    var checksForSomething = maybe(function (value) {
      // function's true logic
    });
{: .language-javascript}
    
Or:
    
    var something = maybe(doesntCheckForSomething)(value);
{: .language-javascript}
    
As a bonus, `maybe` plays very nicely with instance methods, we'll discuss those [later](#methods):

    function Model () {};
    
    Model.prototype.setSomething = maybe(function (value) {
      this.something = value;
    });
{: .language-javascript}
    
If some code ever tries to call `model.setSomething` with nothing, the operation will be skipped.

[andand]: https://github.com/raganwald/andand
[maybe]: https://en.wikipedia.org/wiki/Monad_(functional_programming)#The_Maybe_monad
