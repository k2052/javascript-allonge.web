# Recipes with Instances and Classes

![These recipes are being roasted to perfection.](diedrich-roaster.jpg)

### Disclaimer

The recipes are written for practicality, and their implementation may introduce JavaScript features that haven't been discussed in the text to this point, such as methods and/or prototypes. The overall *use* of each recipe will fit within the spirit of the language discussed so far, even if the implementations may not.

## Currying

We discussed currying in [Closures](#closures) and [Partial Application, Binding, and Currying](#pabc). Here is the recipe for a higher-order function that curries its argument function. It works with any function that has a fixed length, and it lets you provide as many arguments as you like.

    var __slice = Array.prototype.slice;
    
    function curry (fn) {
      var arity = fn.length;
      
      return given([]);
      
      function given (argsSoFar) {
        return function helper () {
          var updatedArgsSoFar = argsSoFar.concat(__slice.call(arguments, 0));
          
          if (updatedArgsSoFar.length >= arity) {
            return fn.apply(this, updatedArgsSoFar)
          }
          else return given(updatedArgsSoFar)
        }
      }
      
    }
    
    function sumOfFour (a, b, c, d) { return a + b + c + d }
    
    var curried = curry(sumOfFour);
    
    curried(1)(2)(3)(4)
      //=> 10
    
    curried(1,2)(3,4)
      //=> 10
    
    curried(1,2,3,4)
      //=> 10
{: .language-javascript}

We saw earlier that you can derive a curry function from a partial application function. The reverse is also true:

    function callLeft (fn) {
      return curry(fn).apply(null, __slice.call(arguments, 1))
    }
    
    callLeft(sumOfFour, 1)(2, 3, 4)
      //=> 10
    
    callLeft(sumOfFour, 1, 2)(3, 4)
      //=> 10
{: .language-javascript}
  
(This is a little different from the previous left partial functions in that it returns a *curried* function).

## Bound {#bound}

Earlier, we saw a recipe for [getWith](#getWith) that plays nicely with properties:

    function get (attr) {
      return function (obj) {
        return obj[attr]
      }
    }
{: .language-javascript}

Simple and useful. But now that we've spent some time looking at objects with methods we can see that `get` (and `pluck`) has a failure mode. Specifically, it's not very useful if we ever want to get a *method*, since we'll lose the context. Consider some hypothetical class:

    function InventoryRecord (apples, oranges, eggs) {
      this.record = {
        apples: apples,
        oranges: oranges,
        eggs: eggs
      }
    }
    
    InventoryRecord.prototype.apples = function apples () {
      return this.record.apples
    }
    
    InventoryRecord.prototype.oranges = function oranges () {
      return this.record.oranges
    }
    
    InventoryRecord.prototype.eggs = function eggs () {
      return this.record.eggs
    }
    
    var inventories = [
      new InventoryRecord( 0, 144, 36 ),
      new InventoryRecord( 240, 54, 12 ),
      new InventoryRecord( 24, 12, 42 )
    ];
{: .language-javascript}
    
Now how do we get all the egg counts?

    mapWith(getWith('eggs'))(inventories)
      //=> [ [Function: eggs],
      //     [Function: eggs],
      //     [Function: eggs] ]
{: .language-javascript}

And if we try applying those functions...

    mapWith(getWith('eggs'))(inventories).map(
      function (unboundmethod) { 
        return unboundmethod() 
      }
    )
      //=> TypeError: Cannot read property 'eggs' of undefined
{: .language-javascript}
      
Of course, these are unbound methods we're "getting" from each object. Here's a new version of `get` that plays nicely with methods. It uses [variadic](#ellipses):

    var bound = variadic( function (messageName, args) {
      
      if (args === []) {
        return function (instance) {
          return instance[messageName].bind(instance)
        }
      }
      else {
        return function (instance) {
          return Function.prototype.bind.apply(
            instance[messageName], [instance].concat(args)
          )
        }
      }
    });

    mapWith(bound('eggs'))(inventories).map(
      function (boundmethod) { 
        return boundmethod() 
      }
    )
      //=> [ 36, 12, 42 ]
{: .language-javascript}

`bound` is the recipe for getting a bound method from an object by name. It has other uses, such as callbacks. `bound('render')(aView)` is equivalent to `aView.render.bind(aView)`. There's an option to add a variable number of additional arguments, handled by:

    return function (instance) {
      return Function.prototype.bind.apply(
        instance[messageName], [instance].concat(args)
      )
    }
{: .language-javascript}
        
The exact behaviour will be covered in [Binding Functions to Contexts](#binding). You can use it like this to add arguments to the bound function to be evaluated:

    InventoryRecord.prototype.add = function (item, amount) {
      this.record[item] || (this.record[item] = 0);
      this.record[item] += amount;
      return this;
    }
    
    mapWith(bound('add', 'eggs', 12))(inventories).map(
      function (boundmethod) { 
        return boundmethod() 
      }
    )
      //=> [ { record: 
      //       { apples: 0,
      //         oranges: 144,
      //         eggs: 48 } },
      //     { record: 
      //       { apples: 240,
      //         oranges: 54,
      //         eggs: 24 } },
      //     { record: 
      //       { apples: 24,
      //         oranges: 12,
      //         eggs: 54 } } ]
{: .language-javascript}

## Unbinding

One of the specifications for `Function.prototype.bind` is that it creates a binding that cannot be overridden. In other words:

    function myName () { return this.name }
    
    var harpo   = { name: 'Harpo' },
        chico   = { name: 'Chico' },
        groucho = { name: 'Groucho' };
        
    var fh = myName.bind(harpo);
    fh()
      //=> 'Harpo'
    
    var fc = myName.bind(chico);
    fc()
      //=> 'Chico'
{: .language-javascript}

This looks great. But what happens if we attempt to **re**-bind a bound function, either with `bind` or `.call`?

    var fhg = fh.bind(groucho);
    fhg()
      //=> 'Harpo'
      
    fc.call(groucho)
      //=> 'Chico'
      
    fh.apply(groucho, [])
      //=> 'Harpo'
{: .language-javascript}
      
Bzzt! You cannot override the context of a function that has already been bound, even if you're creating a new function with `.bind`. You also don't want to roll your own `bind` function that allows rebinding, lest you be bitten by someone else's code that expects that a bound function cannot be rebound. (One such case is where bound functions--such as callbacks--are stored in an array. Evaluating `callbacks[index]()` will override the bound context with the array unless the context cannot be overridden.)[^reddit]

### the recipe

Our version of `bind` memoizes the original function so that you can later call `unbind` to restore it for rebinding.

    var unbind = function unbind (fn) {
      return fn.unbound ? unbind(fn.unbound()) : fn
    };
   
    function bind (fn, context, force) {
      var unbound, bound;
      
      if (force) {
        fn = unbind(fn)
      }
      bound = function () {
        return fn.apply(context, arguments)
      };
      bound.unbound = function () {
        return fn;
      };
      
      return bound;
    }

    function myName () { return this.name }
    
    var harpo   = { name: 'Harpo' },
        chico   = { name: 'Chico' },
        groucho = { name: 'Groucho' };
        
    var fh = bind(myName, harpo);
    fh()
      //=> 'Harpo'
    
    var fc = bind(myName, chico);
    fc()
      //=> 'Chico'

    var fhg = bind(fh, groucho);
    fhg()
      //=> 'Harpo'

    var fhug = bind(fh, groucho, true);
    fhug()
      //=> 'Groucho'

    var fhug2 = bind(unbind(fh), groucho);
    fhug()
      //=> 'Groucho'
      
    fc.unbound().call(groucho)
      //=> 'Groucho'
      
    unbind(fh).apply(groucho, [])
      //=> 'Groucho'
{: .language-javascript}
      
[walled garden]: https://github.com/raganwald/homoiconic/blob/master/2012/12/walled-gardens.md#programmings-walled-gardens
[^reddit]: Isnotlupus on Reddit suggested [this line of thinking](http://www.reddit.com/r/javascript/comments/15ix7s/weak_binding_in_javascript/c7n10yd) against "weak binding" functions.

## Send {#send}

Previously, we saw that the recipe [bound](#bound) can be used to get a bound method from an instance. Unfortunately, invoking such methods is a little messy:

    mapWith(bound('eggs'))(inventories).map(
      function (boundmethod) { 
        return boundmethod() 
      }
    )
      //=> [ 36, 12, 42 ]
{: .language-javascript}

As we noted, it's ugly to write

    function (boundmethod) { 
      return boundmethod() 
    }
{: .language-javascript}

So instead, we write a new recipe:

    var send = variadic( function (args) {
      var fn = bound.apply(this, args);
      
      return function (instance) {
        return fn(instance)();
      }
    })

    mapWith(send('apples'))(inventories)
      //=> [ 0, 240, 24 ]
{: .language-javascript}
      
`send('apples')` works very much like `&:apples` in the Ruby programming language. You may ask, why retain `bound`? Well, sometimes we want the function but don't want to evaluate it immediately, such as when creating callbacks. `bound` does that well.

Here's a robust version that doesn't rely on `bound`:

    var send = variadic( function (methodName, leftArguments) {
      return variadic( function (receiver, rightArguments) {
        return receiver[methodName].apply(receiver, leftArguments.concat(rightArguments))
      })
    });
{: .language-javascript}

## Invoke {#invoke}

[Send](#send) is useful when invoking a function that's a member of an object (or of an instance's prototype). But we sometimes want to invoke a function that is designed to be executed within an object's context. This happens most often when we want  to "borrow" a method from one "class" and use it on another object.

It's not an unprecedented use case. The Ruby programming language has a handy feature called [instance_exec]. It lets you execute an arbitrary block of code in the context of any object. Does this sound familiar? JavaScript has this exact feature, we just call it `.apply` (or `.call` as the case may be). We can execute any function in the context of any arbitrary object.

[instance_exec]: http://www.ruby-doc.org/core-1.8.7/Object.html#method-i-instance_exec

The only trouble with `.apply` is that being a method, it doesn't compose nicely with other functions like combinators. So, we create a function that allows us to use it as a combinator:

    var __slice = Array.prototype.slice;

    function invoke (fn) {
      var args = __slice.call(arguments, 1);
      
      return function (instance) {
        return fn.apply(instance, args)
      }
    }
{: .language-javascript}

For example, let's say someone else's code gives you an array of objects that are in part, but not entirely like arrays. Something like:

    var data = [
      { 0: 'zero', 
        1: 'one', 
        2: 'two', 
        foo: 'foo', 
        length: 3 },
      // ...
    ];
{: .language-javascript}

We can use the pattern from [Partial Application, Binding, and Currying](#pabc) to create a context-dependent copy function:

    var __copy = callFirst(Array.prototype.slice, 0);
{: .language-javascript}
    
And now we can compose `mapWith` with `invoke` to convert the data to arrays:
    
    mapWith(invoke(__copy))(data)
      //=> [
      //     [ 'zero', 'one', 'two' ],
      //     // ...
      //   ]
{: .language-javascript}

`invoke` is useful when you have the function and are looking for the instance. It can be written "the other way around," for when you have the instance and are looking for the function:

    function instanceEval (instance) {
      return function (fn) {
        var args = __slice.call(arguments, 1);
        
        return fn.apply(instance, args)
      }
    }
    
    var args = instanceEval(arguments)(__slice, 0);
{: .language-javascript}

## Fluent {#fluent}

Object and instance methods can be bifurcated into two classes: Those that query something, and those that update something. Most design philosophies arrange things such that update methods return the value being updated. For example:

    function Cake () {}
    
    extend(Cake.prototype, {
      setFlavour: function (flavour) { 
        return this.flavour = flavour 
      },
      setLayers: function (layers) { 
        return this.layers = layers 
      },
      bake: function () {
        // do some baking
      }
    });
    
    var cake = new Cake();
    cake.setFlavour('chocolate');
    cake.setLayers(3);
    cake.bake();
{: .language-javascript}

Having methods like `setFlavour` return the value being set mimics the behaviour of assignment, where `cake.flavour = 'chocolate'` is an expression that in addition to setting a property also evaluates to the value `'chocolate'`.

The [fluent] style presumes that most of the time when you perform an update you are more interested in doing other things with the receiver then the values being passed as argument(s), so the rule is to return the receiver unless the method is a query:

    function Cake () {}
    
    extend(Cake.prototype, {
      setFlavour: function (flavour) { 
        this.flavour = flavour;
        return this
      },
      setLayers: function (layers) { 
        this.layers = layers;
        return this
      },
      bake: function () {
        // do some baking
        return this
      }
    });
{: .language-javascript}

The code to work with cakes is now easier to read and less repetitive:

    var cake = new Cake().
      setFlavour('chocolate').
      setLayers(3).
      bake();
{: .language-javascript}

For one-liners like setting a property, this is fine. But some functions are longer, and we want to signal the intent of the method at the top, not buried at the bottom. Normally this is done in the method's name, but fluent interfaces are rarely written to include methods like `setLayersAndReturnThis`.

[fluent]: https://en.wikipedia.org/wiki/Fluent_interface

The `fluent` method decorator solves this problem:

    function fluent (methodBody) {
      return function () {
        methodBody.apply(this, arguments);
        return this;
      }
    }
{: .language-javascript}

Now you can write methods like this:

    Cake.prototype.bake = fluent( function () {
      // do some baking
      // using many lines of code
      // and possibly multiple returns
    });
{: .language-javascript}

It's obvious at a glance that this method is "fluent."

## Once Again {#named-once}

As we noted when we saw the recipe for [once](#once), you do have to be careful that you are calling the function `once` returns multiple times. If you keep calling `once`, you'll get a new function that executes once, so you'll keep calling your function:

    once(function () {
      return 'sure, why not?'
    })()
      //=> 'sure, why not?'

    once(function () {
      return 'sure, why not?'
    })()
      //=> 'sure, why not?'
{: .language-javascript}

This is expected, but sometimes not what we want. Instead of the simple implementation, we can use a *named once*:

    function once (fn) {
      var done = false,
          testAndSet;
          
      if (!!fn.name) {
        testAndSet = function () {
          this["__once__"] || (this["__once__"] = {})
          if (this["__once__"][fn.name]) return true;
          this["__once__"][fn.name] = true;
          return false
        }
      }
      else  {
        testAndSet = function (fn) {
          if (done) return true;
          done = true;
          return false
        }
      }
      
      return function () {
        return testAndSet.call(this) ? void 0 : fn.apply(this, arguments)
      }
    }
{: .language-javascript}

If you call this with just a function, it behaves exactly like our first recipe. But if you supply a named function, you get a different behaviour:

    once(function askedOnDate () {
      return 'sure, why not?'
    })()
      //=> 'sure, why not?'  
        
    once(function askedOnDate () {
      return 'sure, why not?'
    })()
      //=> undefined
{: .language-javascript}

The named once adds a property, `__once__`, to the context where the function is called and uses it to keep track of which named functions have and haven't been run. One very powerful use is for defining object methods that should only be evaluated once, such as an initialization method. Normally this is done in the constructor, but you might write a "fluent" object that lets you call various setters:

    function Widget () {};
    
    Widget.prototype.setVolume = function setVolume (volume) {
      this.volume = volume;
      return this;
    }
    
    Widget.prototype.setDensity = function setDensity (density) {
      this.density = density;
      return this;
    }
    
    Widget.prototype.setLength = function setLength (length) {
      this.length = length;
      return this;
    }
    
    Widget.prototype.initialize = once(function initialize() {
      // do some initialization
      return this;
    });
    
    var w = new Widget()
      .setVolume(...)
      .setDensity)(...)
      .setLength(...)
      .initialize();
{: .language-javascript}
      
If you later call `w.initialize()`, it won't be initialized again. You need a named `once`, because an ordinary `once` would be called once for every instance sharing the same prototype, whereas the named once will keep track of whether it has been run separately for each instance.

Caveat: Every instance will have a `__once__` property. If you later write code that iterates over every property, you'll have to take care not to interact with it.
