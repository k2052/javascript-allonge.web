# Recipes with Objects, Mutations, and State

![The Intestines of an Espresso Machine](intestines.jpg)

### Disclaimer

The recipes are written for practicality, and their implementation may introduce JavaScript features that haven't been discussed in the text to this point, such as methods and/or prototypes. The overall *use* of each recipe will fit within the spirit of the language discussed so far, even if the implementations may not.

## Memoize {#memoize}

Consider that age-old interview quiz, writing a recursive fibonacci function (there are other ways to derive a fibonacci number, of course). Here's an implementation that doesn't use a [named function expression](#named-function-expressions). The reason for that omission will be explained later:

      var fibonacci = function (n) {
        if (n < 2) {
          return n
        }
        else {
          return fibonacci(n-2) + fibonacci(n-1)
        }
      }
{: .language-javascript}

We'll time it:

    s = (new Date()).getTime()
    new Fibonacci(45).toInt()
    ( (new Date()).getTime() - s ) / 1000
      //=> 28.565
{: .language-javascript}
      
Why is it so slow? Well, it has a nasty habit of recalculating the same results over and over and over again. We could rearrange the computation to avoid this, but let's be lazy and trade space for time. What we want to do is use a lookup table. Whenever we want a result, we look it up. If we don't have it, we calculate it and write the result in the table to use in the future. If we do have it, we return the result without recalculating it.

Here's our recipe:

    function memoized (fn, keymaker) {
      var lookupTable = {}, 
          key, 
          value;
        
      keymaker || (keymaker = function (args) {
        return JSON.stringify(args) 
      });
        
      return function () {
        var key = keymaker.call(this, arguments);
      
        return lookupTable[key] || (
          lookupTable[key] = fn.apply(this, arguments)
        )
      }
    }
{: .language-javascript}

We can apply `memoized` to a function and we will get back a new function that "memoizes" its results so that it never has to recalculate the same value twice. It only works for functions that are "idempotent," meaning functions that always return the same result given the same argument(s). Like `fibonacci`:

Let's try it:

    var fastFibonacci = memoized( function (n) {
      if (n < 2) {
        return n
      }
      else {
        return fastFibonacci(n-2) + fastFibonacci(n-1)
      }
    });

    fastFibonacci(45)
      //=> 1134903170
{: .language-javascript}

We get the result back instantly. It works! You can use memoize with all sorts of "idempotent" pure functions. by default, it works with any function that takes arguments which can be transformed into JSON using JavaScript's standard library function for this purpose. If you have another strategy for turning the arguments into a string key, you can supply it as a second parameter.
      
### memoizing recursive functions

We deliberately picked a recursive function to memoize, because it demonstrates a pitfall when combining decorators with named functional expressions. Consider this implementation that uses a named functional expression:

    var fibonacci = function fibonacci (n) {
      if (n < 2) {
        return n
      }
      else {
        return fibonacci(n-2) + fibonacci(n-1)
      }
    }
{: .language-javascript}
    
If we try to memoize it, we don't get the expected speedup:

    var fibonacci = memoized( function fibonacci (n) {
      if (n < 2) {
        return n
      }
      else {
        return fibonacci(n-2) + fibonacci(n-1)
      }
    });
{: .language-javascript}

That's because the function bound to the name `fibonacci` in the outer environment has been memoized, but the named functional expression binds the name `fibonacci` inside the unmemoized function, so none of the recursive calls to fibonacci are *ever* memoized. Therefore we must write:

    var fibonacci = memoized( function (n) {
      if (n < 2) {
        return n
      }
      else {
        return fibonacci(n-2) + fibonacci(n-1)
      }
    });
{: .language-javascript}

If we need to prevent a rebinding from breaking the function, we'll need to use the [module](#modules) pattern.

## getWith {#getWith}

`getWith` is a very simple function. It takes the name of an attribute and returns a function that extracts the value of that attribute from an object:

    function getWith (attr) {
      return function (object) { return object[attr]; }
    }
{: .language-javascript}

You can use it like this:

    var inventory = {
      apples: 0,
      oranges 144,
      eggs: 36
    };
    
    getWith('oranges')(inventory)
      //=> 144
{: .language-javascript}

This isn't much of a recipe yet. But let's combine it with [mapWith](#mapWith):

    var inventories = [
      { apples: 0, oranges: 144, eggs: 36 },
      { apples: 240, oranges: 54, eggs: 12 },
      { apples: 24, oranges: 12, eggs: 42 }
    ];
  
    mapWith(getWith('oranges'))(inventories)
      //=> [ 144, 54, 12 ]
{: .language-javascript}
    
That's nicer than writing things out "longhand:"

    mapWith(function (inventory) { return inventory.oranges })(inventories)
      //=> [ 144, 54, 12 ]
{: .language-javascript}

`getWith` plays nicely with [maybe](#maybe) as well. Consider a sparse array. You can use:

    mapWith(maybe(getWith('oranges')))
{: .language-javascript}
    
To get the orange count from all the non-null inventories in a list.

### what's in a name?

Why is this called `getWith`? Consider this function that is common in languages that have functions and dictionaries but not methods:

    function get (object, attr) {
      return object[attr];
    };
{: .language-javascript}
    
You might ask, "Why use a function instead of just using `[]`?" The answer is, we can manipulate functions in ways that we can't manipulate syntax. For example, do you remember from [filp](#flip) that we can define `mapWith` from `map`?

    var mapWith = flip(map);
{: .language-javascript}
    
We can do the same thing with `getWith`, and that's why it's named in this fashion:

    var getWith = flip(get)
{: .language-javascript}

## pluckWith {#pluck}

This pattern of combining [mapWith](#mapWith) and [getWith](#getWith) is very frequent in JavaScript code. So much so, that we can take it up another level:

    function pluckWith (attr) {
      return mapWith(getWith(attr))
    }
{: .language-javascript}
    
Or even better:

    var pluckWith = compose(mapWith, getWith);
{: .language-javascript}
    
And now we can write:
    
    pluckWith('eggs')(inventories)
      //=> [ 36, 12, 42 ]
{: .language-javascript}
      
Libraries like [Underscore] provide `pluck`, the flipped version of `pluckWith`:

    _.pluck(inventories, 'eggs')
      //=> [ 36, 12, 42 ]
{: .language-javascript}

Our recipe is terser when you want to name a function:

    var eggsByStore = pluck('eggs');
{: .language-javascript}
    
vs.

    function eggsByStore (inventories) {
      return _.pluck(inventories, 'eggs')
    }
{: .language-javascript}
    
And of course, if we have `pluck` we can use [flip](#flip) to derive `pluckWith`:

    var pluckWith = flip(_.pluck);
{: .language-javascript}

[Underscore]: http://underscorejs.org

## Deep Mapping {#deepMapWith}

[mapWith](#mapWith) is an excellent tool, but from time to time you will find yourself working with arrays that represent trees rather than lists. For example, here is a partial list of sales extracted from a report of some kind. It's grouped in some mysterious way, and we need to operate on each item in the report.

    var report = 
      [ [ { price: 1.99, id: 1 },
        { price: 4.99, id: 2 },
        { price: 7.99, id: 3 },
        { price: 1.99, id: 4 },
        { price: 2.99, id: 5 },
        { price: 6.99, id: 6 } ],
      [ { price: 5.99, id: 21 },
        { price: 1.99, id: 22 },
        { price: 1.99, id: 23 },
        { price: 1.99, id: 24 },
        { price: 5.99, id: 25 } ],

      // ...

      [ { price: 7.99, id: 221 },
        { price: 4.99, id: 222 },
        { price: 7.99, id: 223 },
        { price: 10.99, id: 224 },
        { price: 9.99, id: 225 },
        { price: 9.99, id: 226 } ] ];
{: .language-javascript}

We could nest some mapWiths, but we humans are tool users. If we can use a stick to extract tasty ants from a hole to eat, we can automate working with arrays:

    function deepMapWith (fn) {
      return function innerdeepMapWith (tree) {
        return Array.prototype.map.call(tree, function (element) {
          if (Array.isArray(element)) {
            return innerdeepMapWith(element);
          }
          else return fn(element);
        });
      };
    };
{: .language-javascript}

And now we can use `deepMapWith` on a tree the way we use `mapWith` on a flat array:

    deepMapWith(getWith('price'))(report)
      //=>  [ [ 1.99,
                4.99,
                7.99,
                1.99,
                2.99,
                6.99 ],
              [ 5.99,
                1.99,
                1.99,
                1.99,
                5.99 ],
                
              // ...
              
              [ 7.99,
                4.99,
                7.99,
                10.99,
                9.99,
                9.99 ] ]
{: .language-javascript}
