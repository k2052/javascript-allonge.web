# Finish the Cup: Instances and Classes {#methods}

![Other languages call their objects "beans," but serve extra-weak coffee in an attempt to be all things to all people](beans1.jpg)

As discussed in [Rebinding and References](#references) and again in [Encapsulating State](#encapsulation), JavaScript objects are very simple, yet the combination of objects, functions, and closures can create powerful data structures. That being said, there are language features that cannot be implemented with Plain Old JavaScript Objects, functions, and closures[^turing].

[^turing]: Since the JavaScript that we have presented so far is [computationally universal](https://en.wikipedia.org/wiki/Turing_completeness "Computational Universality and Turing Completeness"), it is possible to perform any calculation with its existing feature set, including emulating any other programming language. Therefore, it is not theoretically necessary to have any further language features; If we need macros, continuations, generic functions, static typing, or anything else, we can [greenspun](https://en.wikipedia.org/wiki/Greenspun%27s_Tenth_Rule) them ourselves. In practice, however, this is buggy, inefficient, and presents our fellow developers with serious challenges understanding our code.

One of them is *inheritance*. In JavaScript, inheritance provides a cleaner, simpler mechanism for extending data structures, domain models, and anything else you represent as a bundle of state and operations.

## Prototypes are Simple, it's the Explanations that are Hard To Understand {#prototypes}

As you recall from our code for making objects [extensible](#extensible), we wrote a function that returned a Plain Old JavaScript Object. The colloquial term for this kind of function is a "Factory Function."

Let's strip a function down to the very bare essentials:

    var Ur = function () {};
{: .language-javascript}

This doesn't look like a factory function: It doesn't have an expression that yields a Plain Old JavaScript Object when the function is applied. Yet, there is a way to make an object out of it. Behold the power of the `new` keyword:

    new Ur()
      //=> {}
{: .language-javascript}
      
We got an object back! What can we find out about this object?

    new Ur() === new Ur()
      //=> false
{: .language-javascript}

Every time we call `new` with a function and get an object back, we get a unique object. We could call these "Objects created with the `new` keyword," but this would be cumbersome. So we're going to call them *instances*. Instances of what? Instances of the function that creates them. So given `var i = new Ur()`, we say that `i` is an instance of `Ur`.

For reasons that will be explained after we've discussed prototypes, we also say that `Ur` is the *constructor* of `i`, and that `Ur` is a *constructor function*. Therefore, an instance is an object created by using the `new` keyword on a constructor function, and that function is the instance's constructor.

### prototypes

There's more. Here's something you may not know about functions:

    Ur.prototype
      //=> {}
{: .language-javascript}
    
What's this prototype? Let's run our standard test:

    (function () {}).prototype === (function () {}).prototype
      //=> false
{: .language-javascript}

Every function is initialized with its own unique `prototype`. What does it do? Let's try something:

    Ur.prototype.language = 'JavaScript';
    
    var continent = new Ur();
      //=> {}
    continent.language
      //=> 'JavaScript'
{: .language-javascript}

That's very interesting! Instances seem to behave as if they had the same elements as their constructor's prototype. Let's try a few things:

    continent.language = 'CoffeeScript';
    continent
      //=> {language: 'CoffeeScript'}
    continent.language
      //=> 'CoffeeScript'
    Ur.prototype.language
      'JavaScript'
{: .language-javascript}

You can set elements of an instance, and they "override" the constructor's prototype, but they don't actually change the constructor's prototype. Let's make another instance and try something else.

    var another = new Ur();
      //=> {}
    another.language
      //=> 'JavaScript'
{: .language-javascript}
      
New instances don't acquire any changes made to other instances. Makes sense. And:

    Ur.prototype.language = 'Sumerian'
    another.language
      //=> 'Sumerian'
{: .language-javascript}

Even more interesting: Changing the constructor's prototype changes the behaviour of all of its instances. This strongly implies that there is a dynamic relationship between instances and their constructors, rather than some kind of mechanism that makes objects by copying.[^dynamic]

[^dynamic]: For many programmers, the distinction between a dynamic relationship and a copying mechanism is too fine to worry about. However, it makes many dynamic program modifications possible.

Speaking of prototypes, here's something else that's very interesting:

    continent.constructor
      //=> [Function]
      
    continent.constructor === Ur
      //=> true
{: .language-javascript}

Every instance acquires a `constructor` element that is initialized to their constructor. This is true even for objects we don't create with `new` in our own code:

    {}.constructor
      //=> [Function: Object]
{: .language-javascript}
      
If that's true, what about prototypes? Do they have constructors?

    Ur.prototype.constructor
      //=> [Function]
    Ur.prototype.constructor === Ur
      //=> true
{: .language-javascript}

Very interesting! We will take another look at the `constructor` element when we discuss [class extension](#classextension).

### revisiting `this` idea of queues

Let's rewrite our Queue to use `new` and `.prototype`, using `this` and our `extends` helper from [Composition and Extension](#composition):

    var Queue = function () {
      extend(this, {
        array: [],
        head: 0,
        tail: -1
      })
    };
      
    extend(Queue.prototype, {
      pushTail: function (value) {
        return this.array[this.tail += 1] = value
      },
      pullHead: function () {
        var value;
        
        if (!this.isEmpty()) {
          value = this.array[this.head]
          this.array[this.head] = void 0;
          this.head += 1;
          return value
        }
      },
      isEmpty: function () {
        return this.tail < this.head
      }      
    })
{: .language-javascript}

You recall that when we first looked at `this`, we only covered the case where a function that belongs to an object is invoked. Now we see another case: When a function is invoked by the `new` operator, `this` is set to the new object being created. Thus, our code for `Queue` initializes the queue.

You can see why `this` is so handy in JavaScript: We wouldn't be able to define functions in the prototype that worked on the instance if JavaScript didn't give us an easy way to refer to the instance itself.

### objects everywhere? {#objectseverywhere}

Now that you know about prototypes, it's time to acknowledge something that even small children know: Everything in JavaScript behaves like an object, everything in JavaScript behaves like an instance of a function, and therefore everything in JavaScript behaves as if it inherits some methods from its constructor's prototype and/or has some elements of its own.

For example:

    3.14159265.toPrecision(5)
      //=> '3.1415'
      
    'FORTRAN, SNOBOL, LISP, BASIC'.split(', ')
      //=> [ 'FORTRAN',
      #     'SNOBOL',
      #     'LISP',
      #     'BASIC' ]
      
    [ 'FORTRAN',
      'SNOBOL',
      'LISP',
      'BASIC' ].length
    //=> 4
{: .language-javascript}
    
Functions themselves are instances, and they have methods. For example, every function has a method `call`. `call`'s first argument is a *context*: When you invoke `.call` on a function, it invoked the function, setting `this` to the context. It passes the remainder of the arguments to the function. It seems like objects are everywhere in JavaScript!

### impostors

You may have noticed that we use "weasel words" to describe how everything in JavaScript *behaves like* an instance. Everything *behaves as if* it was created by a function with a prototype.

The full explanation is this: As you know, JavaScript has "value types" like `String`, `Number`, and `Boolean`. As noted in the first chapter, value types are also called *primitives*, and one consequence of the way JavaScript implements primitives is that they aren't objects. Which means they can be identical to other values of the same type with the same contents, but the consequence of certain design decisions is that value types don't actually have methods or constructors. They aren't instances of some constructor.

So. Value types don't have methods or constructors. And yet:

    "Spence Olham".split(' ')
      //=> ["Spence", "Olham"]
{: .language-javascript}

Somehow, when we write `"Spence Olham".split(' ')`, the string `"Spence Olham"` isn't an instance, it doesn't have methods, but it does a damn fine job of impersonating an instance of a `String` constructor. How does `"Spence Olham"` impersonate an instance?

JavaScript pulls some legerdemain. When you do something that treats a value like an object, JavaScript checks to see whether the value actually is an object. If the value is actually a primitive,[^reminder] JavaScript temporarily makes an object that is a kinda-sorta copy of the primitive and that kinda-sorta copy has methods and you are temporarily fooled into thinking that `"Spence Olham"` has a `.split` method.

[^reminder]: Recall that Strings, Numbers, Booleans and so forth are value types and primitives. We're calling them primitives here.

These kinda-sorta copies are called String *instances* as opposed to String *primitives*. And the instances have methods, while the primitives do not. How does JavaScript make an instance out of a primitive? With `new`, of course. Let's try it:

    new String("Spence Olham")
      //=> "Spence Olham"
{: .language-javascript}
      
The string instance looks just like our string primitive. But does it behave like a string primitive? Not entirely:

    new String("Spence Olham") === "Spence Olham"
      //=> false
{: .language-javascript}
      
Aha! It's an object with its own identity, unlike string primitives that behave as if they have a canonical representation. If we didn't care about their identity, that wouldn't be a problem. But if we carelessly used a string instance where we thought we had a string primitive, we could run into a subtle bug:

    if (userName === "Spence Olham") {
      getMarried();
      goCamping()
    }
{: .language-javascript}
      
That code is not going to work as we expect should we accidentally bind `new String("Spence Olham")` to `userName` instead of the primitive `"Spence Olham"`.

This basic issue that instances have unique identities but primitives with the same contents have the same identities--is true of all primitive types, including numbers and booleans: If you create an instance of anything with `new`, it gets its own identity.

There are more pitfalls to beware. Consider the truthiness of string, number and boolean primitives:

    '' ? 'truthy' : 'falsy'
      //=> 'falsy'
    0 ? 'truthy' : 'falsy'
      //=> 'falsy'
    false ? 'truthy' : 'falsy'
      //=> 'falsy'
{: .language-javascript}
      
Compare this to their corresponding instances:

    new String('') ? 'truthy' : 'falsy'
      //=> 'truthy'
    new Number(0) ? 'truthy' : 'falsy'
      //=> 'truthy'
    new Boolean(false) ? 'truthy' : 'falsy'
      //=> 'truthy'
{: .language-javascript}
      
Our notion of "truthiness" and "falsiness" is that all instances are truthy, even string, number, and boolean instances corresponding to primitives that are falsy.

There is one sure cure for "JavaScript Impostor Syndrome." Just as `new PrimitiveType(...)` creates an instance that is an impostor of a primitive, `PrimitiveType(...)` creates an original, canonicalized primitive from a primitive or an instance of a primitive object.

For example:

    String(new String("Spence Olham")) === "Spence Olham"
      //=> true
{: .language-javascript}
      
Getting clever, we can write this:

    var original = function (unknown) {
      return unknown.constructor(unknown)
    }
        
    original(true) === true
      //=> true
    original(new Boolean(true)) === true
      //=> true
{: .language-javascript}
      
Of course, `original` will not work for your own creations unless you take great care to emulate the same behaviour. But it does work for strings, numbers, and booleans.

## Binding Functions to Contexts {#binding}

Recall that in [What Context Applies When We Call a Function?](#context), we adjourned our look at setting the context of a function with a look at a `contextualize` helper function:

    var contextualize = function (fn, context) {
      return function () {
        return fn.apply(context, arguments)
      }
    },
    a = [1,2,3],
    accrete = contextualize(a.concat, a);
        
    accrete([4,5])
      //=> [ 1, 2, 3, 4, 5 ]
{: .language-javascript}
      
How would this help us in a practical way? Consider building an event-driven application. For example, an MVC application would bind certain views to update events when their models change. The [Backbone] framework uses events just like this:

    var someView = ...,
        someModel = ...;

    someModel.on('change', function () {
      someView.render()
    });
{: .language-javascript}
    
This tells `someModel` that when it invoked a `change` event, it should call the anonymous function that in turn invoked `someView`'s `.render` method. Wouldn't it be simpler to simply write:

    someModel.on('change', someView.render);
{: .language-javascript}
    
It would, except that the implementation for `.on` and similar framework methods looks something like this:

    Model.prototype.on = function (eventName, callback) { ... callback() ... }
{: .language-javascript}
    
Although `someView.render()` correctly sets the method's context as `someView`, `callback()` will not. What can we do without wrapping `someView.render()` in a function call as we did above?

### binding methods

Before enumerating approaches, let's describe what we're trying to do. We want to take a method call and treat it as a function. Now, methods are functions in JavaScript, but as we've learned from looking at contexts, method calls involve both invoking a function *and* setting the context of the function call to be the receiver of the method call.

When we write something like:

    var unbound = someObject.someMethod;
{: .language-javascript}
    
We're binding the name `unbound` to the method's function, but we aren't doing anything with the identity of the receiver. In most programming languages, such methods are called "unbound" methods because they aren't associated with, or "bound" to the intended receiver.

So what we're really trying to do is get ahold of a *bound* method, a method that is associated with a specific receiver. We saw an obvious way to do that above, to wrap the method call in another function. Of course, we're responsible for replicating the *arity* of the method being bound. For example:

    var boundSetter = function (value) {
      return someObject.setSomeValue(value);
    };
{: .language-javascript}
    
Now our bound method takes one argument, just like the function it calls. We can use a bound method anywhere:

    someDomField.on('update', boundSetter);
{: .language-javascript}

This pattern is very handy, but it requires keeping track of these bound methods. One thing we can do is bind the method "in place," using the `let` pattern like this:

    someObject.setSomeValue = (function () {
      var unboundMethod = someObject.setSomeValue;
      
      return function (value) {
        return unboundMethod.call(someObject, value);
      }
    })();
{: .language-javascript}
    
Now we know where to find it:

    someDomField.on('update', someObject.setSomeValue);
{: .language-javascript}
    
This is a very popular pattern, so much so that many frameworks provide helper functions to make this easy. [Underscore], for example, provides `_.bind` to return a bound copy of a function and `_.bindAll` to bind methods in place:

    // bind *all* of someObject's methods in place
    _.bindAll(someObject); 
    
    // bind setSomeValue and someMethod in place
    _.bindAll(someObject, 'setSomeValue', 'someMethod');
{: .language-javascript}

There are two considerations to ponder. First, we may be converting an instance method into an object method. Specifically, we're creating an object method that is bound to the object.

Most of the time, the only change this makes is that it uses slightly more memory (we're creating an extra function for each bound method in each object). But if you are a little more dynamic and actually change methods in the prototype, your changes won't "override" the object methods that you created. You'd have to roll your own binding method that refers to the prototype's method dynamically or reorganize your code.

This is one of the realities of "meta-programming." Each technique looks useful and interesting in isolation, but when multiple techniques are used together, they can have unpredictable results. It's not surprising, because most popular languages consider classes and methods to be fairly global, and they handle dynamic changes through side-effects. This is roughly equivalent to programming in 1970s-era BASIC by imperatively changing global variables.

If you aren't working with old JavaScript environments in non-current browsers, you needn't use a framework or roll your own binding functions: JavaScript has a [`.bind`][bind] method defined for functions:

    someObject.someMethod = someObject.someMethod.bind(someObject);
{: .language-javascript}

`.bind` also does some currying for you, you can bind one or more arguments in addition to the context. For example:

    AccountModel.prototype.getBalancePromise(forceRemote) = {
      // if forceRemote is true, always goes to the remote
      // database for the most real-time value, returns
      // a promise.
    };
    
    var account = new AccountModel(...);
    
    var boundGetRemoteBalancePromise = account.
      getBalancePromise.
      bind(account, true);
{: .language-javascript}
    
Very handy, and not just for binding contexts!

[Backbone]: http://backbonejs.org
[Underscore]: http://underscorejs.org
[bind]: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function/bind

T> Getting the context right for methods is essential. The commonplace terminology is that we want bound methods rather than unbound methods. Current flavours of JavaScript provide a `.bind` method to help, and frameworks like Underscore also provide helpers to make binding methods easy.

## Partial Application, Binding, and Currying {#pabc}

Now that we've seen how function contexts work, we can revisit the subject of partial application. Recall our recipe for a generalized left partial application:

    var callLeft = variadic( function (fn, args) {
      return variadic( function (remainingArgs) {
        return fn.apply(this, args.concat(remainingArgs))
      })
    })
{: .language-javascript}
    
`Function.prototype.bind` can sometimes be used to accomplish the same thing, but will be much faster. For example, instead of:

    function add (verb, a, b) { 
      return "The " + verb + " of " + a + ' and ' + b + ' is ' + (a + b) 
    }
    
    var sumFive = callLeft(add, 'sum', 5);
    
    sumFive(6)
      //=> 'The sum of 5 and 6 is 11'
{: .language-javascript}
      
You can write:

    var totalSix = add.bind(null, 'total', 6);
    
    totalSix(5)
      //=> 'The total of 6 and 5 is 11'
{: .language-javascript}

The catch is the first parameter to `.bind`: It sets the context. If you write functions that don't use the context, like our `.add`, You can use `.bind` to do left partial application. But if you want to partially apply a method or other function where the context must be preserved, you can't use `.bind`. You can use the recipes given in *JavaScript Allongé* because they preserve the context properly.

Typically, context matters when you want to perform partial application on methods. So for an extremely simple example, we often use `Array.prototype.slice` to convert `arguments` to an array. So instead of:

    var __slice = Array.prototype.slice;
    
    var array = __slice.call(arguments, 0);
{: .language-javascript}
    
We could write:

    var __copy = callFirst(Array.prototype.slice, 0);
    
    var array = __copy.call(arguments)
{: .language-javascript}

The other catch is that `.bind` only does left partial evaluation. If you want to do right partial application, you'll need `callLast` or `callRight`.

### currying

The terms "partial application" and "currying" are closely related but not synonymous. Currying is the act of taking a function that takes more than one argument and converting it to an equivalent function taking one argument. How can such a function be equivalent? It works provided that it returns a partially applied function.

Code is, as usual, much clearer than words. Recall:

    function add (verb, a, b) { 
      return "The " + verb + " of " + a + ' and ' + b + ' is ' + (a + b) 
    }
    
    add('sum', 5, '6')
      //=> 'The sum of 5 and 6 is 11'
{: .language-javascript}
    
Here is the curried version:

    function addCurried (verb) {
      return function (a) {
        return function (b) {
          return "The " + verb + " of " + a + ' and ' + b + ' is ' + (a + b) 
        }
      }
    }
    
    addCurried('total')(6)(5)
      //=> 'The total of 6 and 5 is 11'
{: .language-javascript}
      
Currying by hand would be an incredible effort, but its close relationship with partial application means that if you have left partial application, you can derive currying. Or if you have currying, you can derive left partial application. Let's derive currying from `callFirst`. [Recall](#simple-partial):

    var __slice = Array.prototype.slice;
    
    function callFirst (fn, larg) {
      return function () {
        var args = __slice.call(arguments, 0);
        
        return fn.apply(this, [larg].concat(args))
      }
    }
{: .language-javascript}

Here's a function that curries any function with two arguments:

    function curryTwo (fn) {
      return function (x) {
        return callFirst(fn, x)
      }
    }
    
    function add2 (a, b) { return a + b }
    
    curryTwo(add)(5)(6)
      //=> 11
{: .language-javascript}

And from there we can curry a function with three arguments:

    function curryThree (fn) {
      return function (x) {
        return curryTwo(callFirst(fn, x))
      }
    }

    function add3 (verb, a, b) { 
      return "The " + verb + " of " + a + ' and ' + b + ' is ' + (a + b) 
    }
    
    curryThree(add3)('sum')(5)(6)
      //=> 'The sum of 5 and 6 is 11'
{: .language-javascript}
      
We'll develop a generalized curry function in the recipes. But to summarize the difference between currying and partial application, currying is an operation that transforms a function taking two or more arguments into a function that takes a single argument and partially applies it to  the function and then curries the rest of the arguments.
    
## A Class By Any Other Name {#class-other-name}

JavaScript has "classes," for some definition of "class." You've met them already, they're constructors that are designed to work with the `new` keyword and have behaviour in their `.prototype` element. You can create one any time you like by:

1. Writing the constructor so that it performs any initialization on `this`, and:
2. Putting all of the method definitions in its prototype.

Let's see it again: Here's a class of todo items:

    function Todo (name) {
      this.name = name || 'Untitled';
      this.done = false;
    };
    
    Todo.prototype.do = function () {
      this.done = true;
    };
    
    Todo.prototype.undo = function () {
      this.done = false;
    };
{: .language-javascript}
    
You can mix other functionality into this class by extending the prototype with an object:

    extend(Todo.prototype, {
      prioritize: function (priority) {
        this.priority = priority;
      };
    });
{: .language-javascript}
    
Naturally, that allows us to define mixins for other classes:

    var ColourCoded = {
      setColourRGB: function (r, g, b) {
        // ...
      },
      getColourRGB: function () {
        // ...
      },
      setColourCSS: function (css) {
        // ...
      },
      getColourCSS: function () {
        // ...
      }
    };
    
    extend(Todo.prototype, ColourCoded);
{: .language-javascript}

This does exactly the same thing as declaring a "class," defining a "method," and adding a "mixin." How does it differ? It doesn't use the words *class*, *method*, *def(ine)* or *mixin*. And it has this `prototype` property that most other popular languages eschew. It also doesn't deal with inheritance, a deal-breaker for programmers who are attached to taxonomies.

For these reasons, many programmers choose to write their own library of functions to mimic the semantics of other programming languages. This has happened so often that most of the popular utility-belt frameworks like [Backbone] have some form of support for defining or extending classes baked in.

[Backbone]: http://backbonejs.org

Nevertheless, JavaScript right out of the box has everything you need for defining classes, methods, mixins, and even inheritance (as we'll see in [Extending Classes with Inheritance]). If we choose to adopt a library with more streamlined syntax, it's vital to understand JavaScript's semantics well enough to know what is happening "under the hood" so that we can work directly with objects, functions, methods, and prototypes when needed.

[Extending Classes with Inheritance]: #classextension

A> One note of caution: A few libraries, such as the vile creation [YouAreDaChef](https://github.com/raganwald/YouAreDaChef#you-are-da-chef), manipulate JavaScript such that ordinary programming such as extending a prototype either don't work at all or break the library's abstraction. Think long and carefully before adopting such a library. The best libraries "Cut with JavaScript's grain."

## Object Methods {#object-methods}

An *instance method* is a function defined in the constructor's prototype. Every instance acquires this behaviour unless otherwise "overridden." Instance methods usually have some interaction with the instance, such as references to `this` or to other methods that interact with the instance. A *constructor method* is a function belonging to the constructor itself.

There is a third kind of method, one that any object (obviously including all instances) can have. An *object method* is a function defined in the object itself. Like instance methods, object methods usually have some interaction with the object, such as references to `this` or to other methods that interact with the object.

Object methods are really easy to create with Plain Old JavaScript Objects, because they're the only kind of method you can use. Recall from [This and That](#this):

    QueueMaker = function () {
      return {
        array: [], 
        head: 0, 
        tail: -1,
        pushTail: function (value) {
          return this.array[this.tail += 1] = value
        },
        pullHead: function () {
          var value;
          
          if (this.tail >= this.head) {
            value = this.array[this.head];
            this.array[this.head] = void 0;
            this.head += 1;
            return value
          }
        },
        isEmpty: function () {
          return this.tail < this.head
        }
      }
    };
{: .language-javascript}
        
`pushTail`, `pullHead`, and `isEmpty` are object methods. Also, from [encapsulation](#hiding-state):

    var stack = (function () {
      var obj = {
        array: [],
        index: -1,
        push: function (value) {
          return obj.array[obj.index += 1] = value
        },
        pop: function () {
          var value = obj.array[obj.index];
          obj.array[obj.index] = void 0;
          if (obj.index >= 0) { 
            obj.index -= 1 
          }
          return value
        },
        isEmpty: function () {
          return obj.index < 0
        }
      };
      
      return obj;
    })();
{: .language-javascript}

Although they don't refer to the object, `push`, `pop`, and `isEmpty` semantically interact with the opaque data structure represented by the object, so they are object methods too.

### object methods within instances

Instances of constructors can have object methods as well. Typically, object methods are added in the constructor. Here's a gratuitous example, a widget model that has a read-only `id`:

    var WidgetModel = function (id, attrs) {
      extend(this, attrs || {});
      this.id = function () { return id }
    }
    
    extend(WidgetModel.prototype, {
      set: function (attr, value) {
        this[attr] = value;
        return this;
      },
      get: function (attr) {
        return this[attr]
      }
    });
{: .language-javascript}

`set` and `get` are instance methods, but `id` is an object method: Each object has its own `id` closure, where `id` is bound to the id of the widget by the argument `id` in the constructor. The advantage of this approach is that instances can have different object methods, or object methods with their own closures as in this case. The disadvantage is that every object has its own methods, which uses up much more memory than instance methods, which are shared amongst all instances.

T> Object methods are defined within the object. So if you have several different "instances" of the same object, there will be an object method for each object. Object methods can be associated with any object, not just those created with the `new` keyword. Instance methods apply  to instances, objects created with the `new` keyword. Instance methods are defined in a  prototype and are shared by all instances.

## Extending Classes with Inheritance {#classextension}

You recall from [Composition and Extension](#extensible) that we extended a Plain Old JavaScript Queue to create a Plain Old JavaScript Deque. But what if we have decided to use JavaScript's prototypes and the `new` keyword instead of Plain Old JavaScript Objects? How do we extend a queue into a deque?

Here's our `Queue`:

    var Queue = function () {
      extend(this, {
        array: [],
        head: 0,
        tail: -1
      })
    };
      
    extend(Queue.prototype, {
      pushTail: function (value) {
        return this.array[this.tail += 1] = value
      },
      pullHead: function () {
        var value;
        
        if (!this.isEmpty()) {
          value = this.array[this.head]
          this.array[this.head] = void 0;
          this.head += 1;
          return value
        }
      },
      isEmpty: function () {
        return this.tail < this.head
      }      
    });
{: .language-javascript}

And here's what our `Deque` would look like before we wire things together:

    var Dequeue = function () {
      Queue.prototype.constructor.call(this)
    };
    
    Dequeue.INCREMENT = 4;
      
    extend(Dequeue.prototype, {
      size: function () {
        return this.tail - this.head + 1
      },
      pullTail: function () {
        var value;
        
        if (!this.isEmpty()) {
          value = this.array[this.tail];
          this.array[this.tail] = void 0;
          this.tail -= 1;
          return value
        }
      },
      pushHead: function (value) {
        var i;
        
        if (this.head === 0) {
          for (i = this.tail; i >= this.head; --i) {
            this.array[i + INCREMENT] = this.array[i]
          }
          this.tail += this.constructor.INCREMENT;
          this.head += this.constructor.INCREMENT
        }
        this.array[this.head -= 1] = value
      }
    });
{: .language-javascript}

A> We obviously want to do all of a `Queue`'s initialization, thus we called `Queue.prototype.constructor.call(this)`. But why not just call `Queue.call(this)`? As we'll see when we wire everything together, this ensures that we're calling the correct constructor even when `Queue` itself is wired to inherit from another constructor function.

So what do we want from dequeues such that we can call all of a `Queue`'s methods as well as a `Dequeue`'s? Should we copy everything from `Queue.prototype` into `Deque.prototype`, like `extend(Deque.prototype, Queue.prototype)`? That would work, except for one thing: If we later modified `Queue`, say by mixing in some new methods into its prototype, those wouldn't be picked up by `Dequeue`.

No, there's a better idea. Prototypes are objects, right? Why must they be Plain Old JavaScript Objects? Can't a prototype be an *instance*?

Yes they can. Imagine that `Deque.prototype` was a proxy for an instance of `Queue`. It would, of course, have all of a queue's behaviour through `Queue.prototype`. We don't want it to be an *actual* instance, mind you. It probably doesn't matter with a queue, but some of the things we might work with might make things awkward if we make random instances. A database connection comes to mind, we may not want to create one just for the convenience of having access to its behaviour.

Here's such a proxy:

    var QueueProxy = function () {}
    
    QueueProxy.prototype = Queue.prototype
{: .language-javascript}
    
Our `QueueProxy` isn't actually a `Queue`, but its `prototype` is an alias of `Queue.prototype`. Thus, it can pick up `Queue`'s behaviour. We want to use it for our `Deque`'s prototype. Let's insert that code in our class definition:

    var Dequeue = function () {
      Queue.prototype.constructor.call(this)
    };
    
    Dequeue.INCREMENT = 4;
    
    Dequeue.prototype = new QueueProxy();
      
    extend(Dequeue.prototype, {
      size: function () {
        return this.tail - this.head + 1
      },
      pullTail: function () {
        var value;
        
        if (!this.isEmpty()) {
          value = this.array[this.tail];
          this.array[this.tail] = void 0;
          this.tail -= 1;
          return value
        }
      },
      pushHead: function (value) {
        var i;
        
        if (this.head === 0) {
          for (i = this.tail; i >= this.head; --i) {
            this.array[i + INCREMENT] = this.array[i]
          }
          this.tail += this.constructor.INCREMENT;
          this.head += this.constructor.INCREMENT
        }
        this.array[this.head -= 1] = value
      }
    });
{: .language-javascript}
      
And it seems to work:

    d = new Dequeue()
    d.pushTail('Hello')
    d.pushTail('JavaScript')
    d.pushTail('!')
    d.pullHead()
      //=> 'Hello'
    d.pullTail()
      //=> '!'
    d.pullHead()
      //=> 'JavaScript'
{: .language-javascript}
      
Wonderful!
      
### getting the constructor element right

How about some of the other things we've come to expect from instances?

    d.constructor == Dequeue
      //=> false
      
Oops! Messing around with Dequeue's prototype broke this important equivalence. Luckily for us, the `constructor` property is mutable for objects we create. So, let's make a small change to `QueueProxy`:

    var QueueProxy = function () {
      this.constructor = Dequeue;
    }
    QueueProxy.prototype = Queue.prototype
{: .language-javascript}
    
Repeat. Now it works:

    d.constructor === Dequeue
      //=> true
{: .language-javascript}

The `QueueProxy` function now sets the `constructor` for every instance of a `QueueProxy` (hopefully just the one we need for the `Dequeue` class). It returns the object being created (it could also return `undefined` and work. But if it carelessly returned something else, that would be assigned to `Dequeue`'s prototype, which would break our code).

### extracting the boilerplate

Let's turn our mechanism into a function:

    var child = function (parent, child) {
      var proxy = function () {
        this.constructor = child
      }
      proxy.prototype = parent.prototype;
      child.prototype = new proxy();
      return child;
    }
{: .language-javascript}

And use it in `Dequeue`:

    var Dequeue = child(Queue, function () {
      Queue.prototype.constructor.call(this)
    });
    
    Dequeue.INCREMENT = 4;
      
    extend(Dequeue.prototype, {
      size: function () {
        return this.tail - this.head + 1
      },
      pullTail: function () {
        var value;
        
        if (!this.isEmpty()) {
          value = this.array[this.tail];
          this.array[this.tail] = void 0;
          this.tail -= 1;
          return value
        }
      },
      pushHead: function (value) {
        var i;
        
        if (this.head === 0) {
          for (i = this.tail; i >= this.head; --i) {
            this.array[i + INCREMENT] = this.array[i]
          }
          this.tail += this.constructor.INCREMENT;
          this.head += this.constructor.INCREMENT
        }
        this.array[this.head -= 1] = value
      }
    });
{: .language-javascript}

### future directions

Some folks just **love** to build their own mechanisms. When all goes well, they become famous as framework creators and open source thought leaders. When all goes badly they create in-house proprietary one-offs that blur the line between application and framework with abstractions everywhere.

If you're keen on learning, you can work on improving the above code to handle extending constructor properties, automatically calling the parent constructor function, and so forth. Or you can decide that doing it by hand isn't that hard so why bother putting a thin wrapper around it?

It's up to you, while JavaScript isn't the tersest language, it isn't so baroque that building inheritence ontologies requires hundreds of lines of inscrutable code.

## Summary

T> ### Instances and Classes
T>
T> * The `new` keyword turns any function into a *constructor* for creating *instances*.
T> * All functions have a `prototype` element.
T> * Instances behave as if the elements of their constructor's prototype are their elements.
T> * Instances can override their constructor's prototype without altering it.
T> * The relationship between instances and their constructor's prototype is dynamic.
T> * `this` works seamlessly with methods defined in prototypes.
T> * Everything behaves like an object.
T> * JavaScript can convert primitives into instances and back into primitives.
T> * Object methods are typically created in the constructor and are private to each object.
T> * Prototypes can be chained to allow extension of instances.
T>
T> And most importantly:
T>
T> * JavaScript has classes and methods, they just aren't formally called classes and methods in the language's syntax.
