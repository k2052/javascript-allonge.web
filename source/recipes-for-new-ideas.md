# Recipes for New Ideas

![For the love of coffee: a collection](new-ideas.jpg)

> "The entire history of Mankind's relationship with coffee is a futile attempt to have the reality of its taste live up to the promise of its aroma."

## Before {#before}

Combinators for functions come in an unlimited panoply of purposes and effects. So do method combinators, but whether from intrinsic utility or custom, certain themes have emerged. One of them that forms a core part of the original [Lisp Flavors][flavors] system and also the [Aspect-Oriented Programming][aop] movement, is decorating a method with some functionality to be performed *before* the method's body is evaluated.

[flavors]: https://en.wikipedia.org/wiki/Flavors_(programming_language)
[aop]: https://en.wikipedia.org/wiki/Aspect-oriented_programming

For example, using our [fluent](#fluent) recipe:

    function Cake () {
      this.ingredients = {}
    }
    
    extend(Cake.prototype, {
      setFlavour: fluent( function (flavour) { 
        this.flavour = flavour
      }),
      setLayers: fluent( function (layers) { 
        this.layers = layers;
      }),
      add: fluent( function (ingredientMap) {
        var ingredient;
        
        for (ingredient in ingredientMap) {
          this.ingredients[ingredient] || 
            (this.ingredients[ingredient] = 0);
          this.ingredients[ingredient] = this.ingredients[ingredient] + 
            ingredientMap[ingredient]
        }
      }),
      mix: fluent( function () {
        // mix ingredients together
      }),
      rise: fluent( function (duration) {
        // let the ingredients rise
      }),
      bake: fluent( function () {
        // do some baking
      })
    });
{: .language-javascript}

This particular example might be better-served as a state machine, but what we want to encode is that we must always mix the ingredients before allowing the batter to rise or baking the cake. The direct way to write that is:

      rise: fluent( function (duration) {
        this.mix();
        // let the ingredients rise
      }),
      bake: fluent( function () {
        this.mix();
        // do some baking
      })
{: .language-javascript}

Nothing wrong with that, however it does clutter the core functionality of rising and baking with a secondary concern, preconditions. There is a similar problem with cross-cutting concerns like logging or checking permissions: You want functions to be smaller and more focused, and decomposing into smaller methods is ugly:

    reallyRise: function (duration) {
      // let the ingredients rise
    },
    rise: fluent (function (duration) {
      this.mix();
      this.reallyRise()
    }),
    reallyBake: function () {
      // do some baking
    },
    bake: fluent( function () {
      this.mix();
      this.reallyBake()
    })
{: .language-javascript}

### the before recipe

This recipe is for a combinator that turns a function into a method decorator. The decorator evaluates the function before evaluating the base method. Here it is:

    function before (decoration) {
      return function (method) {
        return function () {
          decoration.apply(this, arguments);
          return method.apply(this, arguments)
        }
      }
    }
{: .language-javascript}
    
And here we are using it in conjunction with `fluent`, showing the power of composing combinators:

    var mixFirst = before(function () {
      this.mix()
    });
    
    extend(Cake.prototype, {
      
      // Other methods...
      
      mix: fluent( function () {
        // mix ingredients together
      }),
      rise: fluent( mixFirst( function (duration) {
        // let the ingredients rise
      })),
      bake: fluent( mixFirst( function () {
        // do some baking
        return this
      }))
    });
{: .language-javascript}

The decorators act like keywords or annotations, documenting the method's behaviour but clearly separating these secondary concerns from the core logic of the method.

---

([before](#before), [after](#after), and many more combinators for building method decorators can be found in the [method combinators][mc] module.)

[mc]: https://github.com/raganwald/method-combinators/blob/master/README-JS.md#method-combinators

## After {#after}

Combinators for functions come in an unlimited panoply of purposes and effects. So do method combinators, but whether from intrinsic utility or custom, certain themes have emerged. One of them that forms a core part of the original [Lisp Flavors][flavors] system and also the [Aspect-Oriented Programming][aop] movement, is decorating a method with some functionality to be performed *after* the method's body is evaluated.

[flavors]: https://en.wikipedia.org/wiki/Flavors_(programming_language)
[aop]: https://en.wikipedia.org/wiki/Aspect-oriented_programming

For example, consider this "class:"

    Todo = function (name) {
      this.name = name || 'Untitled';
      this.done = false
    }
    
    extend(Todo.prototype, {
      do: fluent( function {
        this.done = true
      }),
      undo: fluent( function {
        this.done = false
      }),
      setName: fluent( maybe( function (name) {
        this.name = name
      }))
    });
{: .language-javascript}

If we're rolling our own model class, we might mix in [Backbone.Events]. Now we can have views listen to our todo items and render themselves when there's a change. Since we've already seen [before](#before), we'll jump right to the recipe for `after`, a combinator that turns a function into a method decorator:

    function after (decoration) {
      return function (method) {
        return function () {
          var value = method.apply(this, arguments);
          decoration.call(this, value);
          return value
        }
      }
    }
{: .language-javascript}

[Backbone.Events]: http://backbonejs.org/#Events

And here it is in use to trigger change events on our `Todo` "class." We're going to be even *more* sophisticated and paramaterize our decorators.

    extend(Todo.prototype, Backbone.Events);
    
    function changes (propertyName) {
      return after(function () {
        this.trigger('changed changed:'+propertyName, this[propertyName])
      })
    }
    
    extend(Todo.prototype, {
      do: fluent( changes('done')( function {
        this.done = true
      })),
      undo: fluent( changes('done')( function {
        this.done = false
      })),
      setName: fluent( changes('name')( maybe( function (name) {
        this.name = name
      })))
    });
{: .language-javascript}
    
The decorators act like keywords or annotations, documenting the method's behaviour but clearly separating these secondary concerns from the core logic of the method.

---

([before](#before), [after](#after), and many more combinators for building method decorators can be found in the [method combinators][mc] module.)

[mc]: https://github.com/raganwald/method-combinators/blob/master/README-JS.md#method-combinators

## Provided and Except {#provided}

Neither the [before](#before) and [after](#after) decorators can actually terminate evaluation without throwing something. Normal execution always results in the base method being evaluated. The `provided` and `excepting` recipes are combinators that produce method decorators that apply a precondition to evaluating the base method body. If the precondition fails, nothing is returned.

The provided combinator turns a function into a method decorator. The function must evaluate to truthy for the base method to be evaluated:

    function provided (predicate) {
      return function(base) {
        return function() {
          if (predicate.apply(this, arguments)) {
            return base.apply(this, arguments);
          }
        };
      };
    };
{: .language-javascript}

`provided` can be used to create named decorators like `maybe`:

    var maybe = provided( function (value) {
      return value != null
    });
  
    SomeModel.prototype.setAttribute = maybe( function (value) {
      this.attribute = value
    });
{: .language-javascript}
    
You can build your own domain-specific decorators:

    var whenNamed = provided( function (record) {
      return record.name && record.name.length > 0
    })
{: .language-javascript}
  
`except` works identically, but with the logic reversed.

    function except (predicate) {
      return function(base) {
        return function() {
          if (!predicate.apply(this, arguments)) {
            return base.apply(this, arguments);
          }
        };
      };
    };
    
    var exceptAdmin = except( function (user) {
      return user.role.isAdmin()
    });
{: .language-javascript}

## A Functional Mixin Factory

[Functional Mixins](#functional-mixins) extend an existing class's prototype. Let's start with:

    function Todo (name) {
      var self = this instanceof Todo
                 ? this
                 : new Todo();
      self.name = name || 'Untitled';
      self.done = false;
    };
    
    Todo.prototype.do = fluent( function () {
      this.done = true;
    });
    
    Todo.prototype.undo = fluent( function () {
      this.done = false;
    });
{: .language-javascript}

We wish to decorate this with:

    ({
      setLocation: fluent( function (location) {
        this.location = location;
      }),
      getLocation: function () { return this.location; }
    });
{: .language-javascript}
    
Instead of writing:

    function becomeLocationAware () {
      this.setLocation = fluent( function (location) {
        this.location = location;
      });
      
      this.getLocation = function () { return this.location; };
      
      return this;
    };
{: .language-javascript}

We'll extract the decoration into a parameter like this:

    function mixin (decoration) {
      extend(this, decoration);
      return this;
    };
{: .language-javascript}

And then "curry" the function manually like this:

    function mixin (decoration) {

      return function () {
        extend(this, decoration);
        return this;
      };
      
    };
{: .language-javascript}
    
We can try it:

    var MixinLocation = mixin({
      setLocation: fluent( function (location) {
        this.location = location;
      }),
      getLocation: function () { return this.location; }
    });
    
    MixinLocation.call(Todo.prototype);
    
    new Todo('Paint Bedroom').setLocation('Home');
      //=> { name: 'Paint Bedroom',
      //     done: false,
      //     location: 'Home'
{: .language-javascript}

Success! Our `mixin` function makes functional mixins. A final refinement is to make it "context-agnostic," so that we can write either `MixinLocation.call(Todo.prototype)` or `MixinLocation(Todo.prototype)`:

    function mixin (decoration) {

      return function decorate () {
        if (arguments[0] !=== void 0) {
          return decorate.call(arguments[0]);
        }
        else {
          extend(this, decoration);
          return this;
        };
      };
      
    };
{: .language-javascript}

## A Class Decorator Factory

As [discussed](#class-decorators), a class decorator creates a new class with some additional decoration. It's lighter weight than subclassing. It's also easy to write a factory function that makes decorators for us. Recall:

    function Todo (name) {
      var self = this instanceof Todo
                 ? this
                 : new Todo();
      self.name = name || 'Untitled';
      self.done = false;
    };
    
    Todo.prototype.do = fluent( function () {
      this.done = true;
    });
    
    Todo.prototype.undo = fluent( function () {
      this.done = false;
    });
{: .language-javascript}

We wish to decorate this with:

    ({
      setColourRGB: fluent( function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
      }),
      getColourRGB: function () {
        return this.colourCode;
      }
    });
{: .language-javascript}
    
Instead of writing:

    function AndColourCoded (clazz) {
      function Decorated  () {
        var self = this instanceof Decorated
                   ? this
                   : new Decorated();
        
        return clazz.apply(self, arguments);
      };
      Decorated.prototype = new clazz();
      
      Decorated.prototype.setColourRGB = fluent( function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
      });
      
      Decorated.prototype.getColourRGB = function () {
        return this.colourCode;
      };
      
      return Decorated;
    };
{: .language-javascript}

We'll extract the decoration into a parameter like this:

    function classDecorator (decoration, clazz) {
      function Decorated  () {
        var self = this instanceof Decorated
                   ? this
                   : new Decorated();
        
        return clazz.apply(self, arguments);
      };
      Decorated.prototype = extend(new clazz(), decoration);
      return Decorated;
    };
{: .language-javascript}

And then "curry" the function manually like this:

    function classDecorator (decoration) {

      return function (clazz) {
        function Decorated  () {
          var self = this instanceof Decorated
                     ? this
                     : new Decorated();
        
          return clazz.apply(self, arguments);
        };
        Decorated.prototype = extend(new clazz(), decoration);
        return Decorated;
      };
      
    };
{: .language-javascript}
    
We can try it:

    var AndColourCoded = classDecorator({
      setColourRGB: fluent( function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
      }),
      getColourRGB: function () {
        return this.colourCode;
      }
    });
    
    var ColourTodo = AndColourCoded(Todo);
    
    new ColourTodo('Use More Decorators').setColourRGB(0, 255, 0);
      //=> { name: 'Use More Decorators',
      //     done: false,
      //     colourCode: { r: 0, g: 255, b: 0 } }
{: .language-javascript}

Success! Our `classDecorator` function makes class decorators.

## Iterator Recipes

### iterators for standard data structures

Note: Despite having capitalized names, iterators are not meant to be used with the `new` keyword.

{:lang="js"}
~~~~~~~~
function FlatArrayIterator (array) {
  var index;
  index = 0;
  return function() {
    return array[index++];
  };
};

function RecursiveArrayIterator (array) {
  var index, myself, state;
  index = 0;
  state = [];
  myself = function() {
    var element, tempState;
    element = array[index++];
    if (element instanceof Array) {
      state.push({
        array: array,
        index: index
      });
      array = element;
      index = 0;
      return myself();
    } else if (element === void 0) {
      if (state.length > 0) {
        tempState = state.pop(), array = tempState.array, index = tempState.index;
        return myself();
      } else {
        return void 0;
      }
    } else {
      return element;
    }
  };
  return myself;
};
~~~~~~~~

### unfolding iterators

{:lang="js"}
~~~~~~~~
function NumberIterator (base) {
  var number;
  if (base == null) {
    base = 0;
  }
  number = base;
  return function() {
    return number++;
  };
};

function FibonacciIterator () {
  var current, previous;
  previous = 0;
  current = 1;
  return function() {
    var value, tempValues;
    value = current;
    tempValues = [current, current + previous], previous = tempValues[0], current = tempValues[1];
    return value;
  };
};
~~~~~~~~

### decorators for slicing iterators

{:lang="js"}
~~~~~~~~
function take (iter, numberToTake) {
  var count = 0;
  return function() {
    if (++count <= numberToTake) {
      return iter();
    } else {
      return void 0;
    }
  };
};

function drop (iter, numberToDrop) {
  while (numberToDrop-- !== 0) {
    iter();
  }
  return iter;
};

function slice (iter, numberToDrop, numberToTake) {
  var count = 0;
  while (numberToDrop-- !== 0) {
    iter();
  }
  if (numberToTake != null) {
    return function() {
      if (++count <= numberToTake) {
        return iter();
      } else {
        return void 0;
      }
    };
  }
  else return iter;
};
~~~~~~~~

(`drop` was suggested by Redditor [skeeto](http://www.reddit.com/user/skeeto). His code also cleaned up an earlier version of `slice`.)

### catamorphic decorator

{:lang="js"}
~~~~~~~~
function fold (iter, binaryFn, seed) {
  var acc, element;
  acc = seed;
  element = iter();
  while (element != null) {
    acc = binaryFn.call(element, acc, element);
    element = iter();
  }
  return acc;
};
~~~~~~~~

### hylomorphic decorators

{:lang="js"}
~~~~~~~~
function map (iter, unaryFn) {
  return function() {
    var element;
    element = iter();
    if (element != null) {
      return unaryFn.call(element, element);
    } else {
      return void 0;
    }
  };
};

function statefulMap (iter, binaryFn, initial) {
  var state = initial;
  return function () {
    element = iter();
    if (element == null) {
      return element;
    }
    else {
      if (state === void 0) {
        return (state = element);
      }
      else return (state = binaryFn.call(element, state, element));
    }
  }
};

function filter (iter, unaryPredicateFn) {
  return function() {
    var element;
    element = iter();
    while (element != null) {
      if (unaryPredicateFn.call(element, element)) {
        return element;
      }
      element = iter();
    }
    return void 0;
  };
};
~~~~~~~~
