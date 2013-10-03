# Stir the Allongé: Objects, Mutation, and State {#mutable}

![Life measured out by coffee spoons](coffee-spoons.jpg)

So far, we have discussed what many call "pure functional" programming, where every expression is necessarily [idempotent], because we have no way of changing state within a program using the tools we have examined.

[idempotent]: https://en.wikipedia.org/wiki/Idempotence

It's time to change *everything*.

## Encapsulating State with Closures {#encapsulation}

> OOP to me means only messaging, local retention and protection and hiding of state-process, and extreme late-binding of all things.--[Alan Kay][oop]

[oop]: http://userpage.fu-berlin.de/~ram/pub/pub_jf47ht81Ht/doc_kay_oop_en

We're going to look at encapsulation using JavaScript's functions and objects. We're not going to call it object-oriented programming, mind you, because that would start a long debate. This is just plain encapsulation,[^encapsulation] with a dash of information-hiding.

[^encapsulation]: "A language construct that facilitates the bundling of data with the methods (or other functions) operating on that data."--[Wikipedia]

[Wikipedia]: https://en.wikipedia.org/wiki/Encapsulation_(object-oriented_programming)

### what is hiding of state-process, and why does it matter?

> In computer science, information hiding is the principle of segregation of the design decisions in a computer program that are most likely to change, thus protecting other parts of the program from extensive modification if the design decision is changed. The protection involves providing a stable interface which protects the remainder of the program from the implementation (the details that are most likely to change).

> Written another way, information hiding is the ability to prevent certain aspects of a class or software component from being accessible to its clients, using either programming language features (like private variables) or an explicit exporting policy.

> --[Wikipedia][ih]

[ih]:https://en.wikipedia.org/wiki/Information_hiding "Information hiding"

Consider a [stack] data structure. There are three basic operations: Pushing a value onto the top (`push`), popping a value off the top (`pop`), and testing to see whether the stack is empty or not (`isEmpty`). These three operations are the stable interface.

[stack]: https://en.wikipedia.org/wiki/Stack_(data_structure)

Many stacks have an array for holding the contents of the stack. This is relatively stable. You could substitute a linked list, but in JavaScript, the array is highly efficient. You might need an index, you might not. You could grow and shrink the array, or you could allocate a fixed size and use an index to keep track of how much of the array is in use. The design choices for keeping track of the head of the list are often driven by performance considerations.

If you expose the implementation detail such as whether there is an index, sooner or later some programmer is going to find an advantage in using the index directly. For example, she may need to know the size of a stack. The ideal choice would be to add a `size` function that continues to hide the implementation. But she's in a hurry, so she reads the `index` directly. Now her code is coupled to the existence of an index, so if we wish to change the implementation to grow and shrink the array, we will break her code.

The way to avoid this is to hide the array and index from other code and only expose the operations we have deemed stable. If and when someone needs to know the size of the stack, we'll add a `size` function and expose it as well.

Hiding information (or "state") is the design principle that allows us to limit the coupling between components of software.

### how do we hide state using javascript? {#hiding-state}

We've been introduced to JavaScript's objects, and it's fairly easy to see that objects can be used to model what other programming languages call (variously) records, structs, frames, or what-have-you. And given that their elements are mutable, they can clearly model state.

Given an object that holds our state (an array and an index[^length]), we can easily implement our three operations as functions. Bundling the functions with the state does not require any special "magic" features. JavaScript objects can have elements of any type, including functions:

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

    stack.isEmpty()
      //=> true
    stack.push('hello')
      //=> 'hello'
    stack.push('JavaScript')
     //=> 'JavaScript'
    stack.isEmpty()
      //=> false
    stack.pop()
     //=> 'JavaScript'
    stack.pop()
     //=> 'hello'
    stack.isEmpty()
      //=> true
{: .language-javascript}     

### method-ology

In this text, we lurch from talking about "functions that belong to an object" to "methods." Other languages may separate methods from functions very strictly, but in JavaScript every method is a function but not all functions are methods.

The view taken in this book is that a function is a method of an object if it belongs to that object and interacts with that object in some way. So the functions implementing the operations on the stack are all absolutely methods of the stack.

But these two wouldn't be methods. Although they "belong" to an object, they don't interact with it:

    {
      min: function (x, y) {
        if (x < y) {
          return x
        }
        else {
          return y
        }
      } 
      max: function (x, y) {
        if (x > y) {
          return x
        }
        else {
          return y
        }
      } 
    }
{: .language-javascript}     

### hiding state

Our stack does bundle functions with data, but it doesn't hide its state. "Foreign" code could interfere with its array or index. So how do we hide these? We already have a closure, let's use it:

    var stack = (function () {
      var array = [],
          index = -1;
          
      return {
        push: function (value) {
          array[index += 1] = value
        },
        pop: function () {
          var value = array[index];
          if (index >= 0) {
            index -= 1
          }
          return value
        },
        isEmpty: function () {
          return index < 0
        }
      }
    })();

    stack.isEmpty()
      //=> true
    stack.push('hello')
      //=> 'hello'
    stack.push('JavaScript')
     //=> 'JavaScript'
    stack.isEmpty()
      //=> false
    stack.pop()
     //=> 'JavaScript'
    stack.pop()
     //=> 'hello'
    stack.isEmpty()
      //=> true
{: .language-javascript}     
      
![Coffee DOES grow on trees](coffee-trees-1200.jpg)

We don't want to repeat this code every time we want a stack, so let's make ourselves a "stack maker." The temptation is to wrap what we have above in a function:

    var StackMaker = function () {
      return (function () {
        var array = [],
            index = -1;
          
        return {
          push: function (value) {
            array[index += 1] = value
          },
          pop: function () {
            var value = array[index];
            if (index >= 0) {
              index -= 1
            }
            return value
          },
          isEmpty: function () {
            return index < 0
          }
        }
      })() 
    }
{: .language-javascript}     

But there's an easier way :-)

    var StackMaker = function () {
      var array = [],
          index = -1;
          
      return {
        push: function (value) {
          array[index += 1] = value
        },
        pop: function () {
          var value = array[index];
          if (index >= 0) {
            index -= 1
          }
          return value
        },
        isEmpty: function () {
          return index < 0
        }
      }
    };

    stack = StackMaker()
{: .language-javascript}     

Now we can make stacks freely, and we've hidden their internal data elements. We have methods and encapsulation, and we've built them out of JavaScript's fundamental functions and objects. In [Instances and Classes](#methods), we'll look at JavaScript's support for class-oriented programming and some of the idioms that functions bring to the party.

A> ### is encapsulation "object-oriented?"
A>
A> We've built something with hidden internal state and "methods," all without needing special `def` or `private` keywords. Mind you, we haven't included all sorts of complicated mechanisms to support inheritance, mixins, and other opportunities for debating the nature of the One True Object-Oriented Style on the Internet.
A>
A> Then again, the key lesson experienced programmers repeat--although it often falls on deaf ears--is [Composition instead of Inheritance](http://www.c2.com/cgi/wiki?CompositionInsteadOfInheritance). So maybe we aren't missing much.

[^length]: Yes, there's another way to track the size of the array, but we don't need it to demonstrate encapsulation and hiding of state.

## Composition and Extension {#composition}

### composition

A deeply fundamental practice is to build components out of smaller components. The choice of how to divide a component into smaller components is called *factoring*, after the operation in number theory [^refactoring]. 

[^refactoring]: And when you take an already factored component and rearrange things so that it is factored into a different set of subcomponents without altering its behaviour, you are *refactoring*.

The simplest and easiest way to build components out of smaller components in JavaScript is also the most obvious: Each component is a value, and the components can be put together into a single object or encapsulated with a closure.

Here's an abstract "model" that supports undo and redo composed from a pair of stacks (see [Encapsulating State](#encapsulation)) and a Plain Old JavaScript Object:

    // helper function
    //
    // For production use, consider what to do about
    // deep copies and own keys
    var shallowCopy = function (source) {
      var dest = {},
          key;
          
      for (key in source) {
        dest[key] = source[key]
      }
      return dest
    };

    // our model maker
    var ModelMaker = function (initialAttributes) {
      var attributes = shallowCopy(initialAttributes || {}), 
          undoStack = StackMaker(), 
          redoStack = StackMaker(),
          obj = {
            set: function (attrsToSet) {
              var key;
              
              undoStack.push(shallowCopy(attributes));
              if (!redoStack.isEmpty()) {
                redoStack = StackMaker()
              }
              for (key in (attrsToSet || {})) {
                attributes[key] = attrsToSet[key]
              }
              return obj
            },
            undo: function () {
              if (!undoStack.isEmpty()) {
                redoStack.push(shallowCopy(attributes));
                attributes = undoStack.pop()
              }
              return obj
            },
            redo: function () {
              if (!redoStack.isEmpty()) {
                undoStack.push(shallowCopy(attributes));
                attributes = redoStack.pop()
              }
              return obj
            },
            get: function (key) {
              return attributes(key)
            },
            has: function (key) {
              return attributes.hasOwnProperty(key)
            },
            attributes: function {
              shallowCopy(attributes)
            }
          };
        return obj
      };
{: .language-javascript}     
      
The techniques used for encapsulation work well with composition. In this case, we have a "model" that hides its attribute store as well as its implementation that is composed of an undo stack and redo stack.

### extension {#extensible}

Another practice that many people consider fundamental is to *extend* an implementation. Meaning, they wish to define a new data structure in terms of adding new operations and semantics to an existing data structure.

Consider a [queue]:

    var QueueMaker = function () {
      var array = [], 
          head = 0, 
          tail = -1;
      return {
        pushTail: function (value) {
          return array[tail += 1] = value
        },
        pullHead: function () {
          var value;
          
          if tail >= head {
            value = array[head];
            array[head] = void 0;
            head += 1;
            return value
          }
        },
        isEmpty: function () {
          return tail < head
        }
      }
    };
{: .language-javascript}     

Now we wish to create a [deque] by adding `pullTail` and `pushHead` operations to our queue.[^wasa] Unfortunately, encapsulation prevents us from adding operations that interact with the hidden data structures.

[queue]: http://duckduckgo.com/Queue_(data_structure)
[deque]: https://en.wikipedia.org/wiki/Double-ended_queue "Double-ended queue"
[^wasa]: Before you start wondering whether a deque is-a queue, we said nothing about types and classes. This relationship is called was-a, or "implemented in terms of a."

This isn't really surprising: The entire point of encapsulation is to create an opaque data structure that can only be manipulated through its public interface. The design goals of encapsulation and extension are always going to exist in tension.

Let's "de-encapsulate" our queue:

    var QueueMaker = function () {
      var queue = {
        array: [], 
        head: 0, 
        tail: -1,
        pushTail: function (value) {
          return queue.array[queue.tail += 1] = value
        },
        pullHead: function () {
          var value;
          
          if (queue.tail >= queue.head) {
            value = queue.array[queue.head];
            queue.array[queue.head] = void 0;
            queue.head += 1;
            return value
          }
        },
        isEmpty: function () {
          return queue.tail < queue.head
        }
      };
      return queue
    };
{: .language-javascript}     

Now we can extend a queue into a deque:

    var DequeMaker = function () {
      var deque = QueueMaker(),
          INCREMENT = 4;
      
      return extend(deque, {
        size: function () {
          return deque.tail - deque.head + 1
        },
        pullTail: function () {
          var value;
          
          if (!deque.isEmpty()) {
            value = deque.array[deque.tail];
            deque.array[deque.tail] = void 0;
            deque.tail -= 1;
            return value
          }
        },
        pushHead: function (value) {
          var i;
          
          if (deque.head === 0) {
            for (i = deque.tail; i <= deque.head; i++) {
              deque.array[i + INCREMENT] = deque.array[i]
            }
            deque.tail += INCREMENT
            deque.head += INCREMENT
          }
          return deque.array[deque.head -= 1] = value
        }
      })
    };
{: .language-javascript}     

Presto, we have reuse through extension, at the cost of encapsulation.

T> Encapsulation and Extension exist in a natural state of tension. A program with elaborate encapsulation resists breakage but can also be difficult to refactor in other ways. Be mindful of when it's best to Compose and when it's best to Extend.

## This and That {#this}

Let's take another look at [extensible objects](#extensible). Here's a Queue:

    var QueueMaker = function () {
      var queue = {
        array: [], 
        head: 0, 
        tail: -1,
        pushTail: function (value) {
          return queue.array[queue.tail += 1] = value
        },
        pullHead: function () {
          var value;
          
          if (queue.tail >= queue.head) {
            value = queue.array[queue.head];
            queue.array[queue.head] = void 0;
            queue.head += 1;
            return value
          }
        },
        isEmpty: function () {
          return queue.tail < queue.head
        }
      };
      return queue
    };

    queue = QueueMaker()
    queue.pushTail('Hello')
    queue.pushTail('JavaScript')
{: .language-javascript}     

Let's make a copy of our queue using the `extend` recipe:

    copyOfQueue = extend({}, queue);
    
    queue !== copyOfQueue
      //=> true
{: .language-javascript}     
    
Wait a second. We know that array values are references. So it probably copied a reference to the original array. Let's make a copy of the array as well:

    copyOfQueue.array = [];
    for (var i = 0; i < 2; ++i) {
      copyOfQueue.array[i] = queue.array[i]
    }
{: .language-javascript}     

Now let's pull the head off the original:

    queue.pullHead()
      //=> 'Hello'
{: .language-javascript}     
      
If we've copied everything properly, we should get the exact same result when we pull the head off the copy:
      
    copyOfQueue.pullHead()
      //=> 'JavaScript'
{: .language-javascript}     
      
What!? Even though we carefully made a copy of the array to prevent aliasing, it seems that our two queues behave like aliases of each other. The problem is that while we've carefully copied our array and other elements over, *the closures all share the same environment*, and therefore the functions in `copyOfQueue` all operate on the first queue's private data, not on the copies.

A> This is a general issue with closures. Closures couple functions to environments, and that makes them very elegant in the small, and very handy for making opaque data structures. Alas, their strength in the small is their weakness in the large. When you're trying to make reusable components, this coupling is sometimes a hindrance.

Let's take an impossibly optimistic flight of fancy:

    var AmnesiacQueueMaker = function () {
      return {
        array: [], 
        head: 0, 
        tail: -1,
        pushTail: function (myself, value) {
          return myself.array[myself.tail += 1] = value
        },
        pullHead: function (myself) {
          var value;
          
          if (myself.tail >= myself.head) {
            value = myself.array[myself.head];
            myself.array[myself.head] = void 0;
            myself.head += 1;
            return value
          }
        },
        isEmpty: function (myself) {
          return myself.tail < myself.head
        }
      }
    };

    queueWithAmnesia = AmnesiacQueueMaker();
    queueWithAmnesia.pushTail(queueWithAmnesia, 'Hello');
    queueWithAmnesia.pushTail(queueWithAmnesia, 'JavaScript')
{: .language-javascript}     
    
The `AmnesiacQueueMaker` makes queues with amnesia: They don't know who they are, so every time we invoke one of their functions, we have to tell them who they are. You can work out the implications for copying queues as a thought experiment: We don't have to worry about environments, because every function operates on the queue you pass in.

The killer drawback, of course, is making sure we are always passing the correct queue in every time we invoke a function. What to do?

### what's all `this`?

Any time we must do the same repetitive thing over and over and over again, we industrial humans try to build a machine to do it for us. JavaScript is one such machine:

    BanksQueueMaker = function () {
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

    banksQueue = BanksQueueMaker();
    banksQueue.pushTail('Hello');
    banksQueue.pushTail('JavaScript') 
{: .language-javascript}     

Every time you invoke a function that is a member of an object, JavaScript binds that object to the name `this` in the environment of the function just as if it was an argument.[^this] Now we can easily make copies:

    copyOfQueue = extend({}, banksQueue)
    copyOfQueue.array = []
    for (var i = 0; i < 2; ++i) {
      copyOfQueue.array[i] = banksQueue.array[i]
    }
      
    banksQueue.pullHead()
      //=> 'Hello'

    copyOfQueue.pullHead()
      //=> 'Hello'
{: .language-javascript}     

Presto, we now have a way to copy arrays. By getting rid of the closure and taking advantage of `this`, we have functions that are more easily portable between objects, and the code is simpler as well.

There is more to `this` than we've discussed here. We'll explore things in more detail later, in [What Context Applies When We Call a Function?](#context).

T> Closures tightly couple functions to the environments where they are created limiting their flexibility. Using `this` alleviates the coupling. Copying objects is but one example of where that flexibility is needed.

[^this]: JavaScript also does other things with `this` as well, but this is all we care about right now.

## What Context Applies When We Call a Function? {#context}

In [This and That](#this), we learned that when a function is called as an object method, the name `this` is bound in its environment to the object acting as a "receiver." For example:

    var someObject = {
      returnMyThis: function () {
        return this;
      }
    };
    
    someObject.returnMyThis() === someObject
      //=> true
{: .language-javascript}     
      
We've constructed a method that returns whatever value is bound to `this` when it is called. It returns the object when called, just as described.

### it's all about the way the function is called

JavaScript programmers talk about functions having a "context" when being called. `this` is bound to the context.[^toobad] The important thing to understand is that the context for a function being called is set by the way the function is called, not the function itself.

[^toobad]: Too bad the language binds the context to the name `this` instead of the name `context`!

This is an important distinction. Consider closures: As we discussed in [Closures and Scope](#closures), a function's free variables are resolved by looking them up in their enclosing functions' environments. You can always determine the functions that define free variables by examining the source code of a JavaScript program, which is why this scheme is known as [Lexical Scope].

[Lexical Scope]: https://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scoping

A function's context cannot be determined by examining the source code of a JavaScript program. Let's look at our example again:

    var someObject = {
      someFunction: function () {
        return this;
      }
    };

    someObject.someFunction() === someObject
      //=> true
{: .language-javascript}     
    
What is the context of the function `someObject.someFunction`? Don't say `someObject`! Watch this:

    var someFunction = someObject.someFunction;

    someFunction === someObject.someFunction
      //=> true
    
    someFunction() === someObject
      //=> false
{: .language-javascript}     
      
It gets weirder:

    var anotherObject = {
      someFunction: someObject.someFunction
    }
    
    anotherObject.someFunction === someObject.someFunction
      //=> true
      
    anotherObject.someFunction() === anotherObject
      //=> true
      
    anotherObject.someFunction() === someObject
      //=> false
{: .language-javascript}     
      
So it amounts to this: The exact same function can be called in two different ways, and you end up with two different contexts. If you call it using `someObject.someFunction()` syntax, the context is set to the receiver. If you call it using any other expression for resolving the function's value (such as `someFunction()`), you get something else. Let's investigate:

    (someObject.someFunction)() == someObject
      //=> true
      
    someObject['someFunction']() === someObject
      //=> true
      
    var name = 'someFunction';
    
    someObject[name]() === someObject
      //=> true
{: .language-javascript}     

Interesting!

    var baz;
    
    (baz = someObject.someFunction)() === this
      //=> true
{: .language-javascript}     
      
How about:

    var arr = [ someObject.someFunction ];
    
    arr[0]() == arr
      //=> true
{: .language-javascript}     
    
It seems that whether you use `a.b()` or `a['b']()` or `a[n]()` or `(a.b)()`, you get context `a`. 

    var returnThis = function () { return this };

    var aThirdObject = {
      someFunction: function () {
        return returnThis()
      }
    }
    
    returnThis() === this
      //=> true
    
    aThirdObject.someFunction() === this
      //=> true
{: .language-javascript}     
      
And if you don't use `a.b()` or `a['b']()` or `a[n]()` or `(a.b)()`, you get the global environment for a context, not the context of whatever function is doing the calling. To simplify things, when you call a function with `.` or `[]` access, you get an object as context, otherwise you get the global environment.

### setting your own context

There are actually two other ways to set the context of a function. And once again, both are determined by the caller. At the very end of [objects everywhere?](#objectseverywhere), we'll see that everything in JavaScript behaves like an object, including functions. We'll learn that functions have methods themselves, and one of them is `call`.

Here's `call` in action:

    returnThis() === aThirdObject
      //=> false

    returnThis.call(aThirdObject) === aThirdObject
      //=> true
      
    anotherObject.someFunction.call(someObject) === someObject
      //=> true
{: .language-javascript}     
      
When You call a function with `call`, you set the context by passing it in as the first parameter. Other arguments are passed to the function in the normal manner. Much hilarity can result from `call` shenanigans like this:

    var a = [1,2,3],
        b = [4,5,6];
        
    a.concat([2,1])
      //=> [1,2,3,2,1]
      
    a.concat.call(b,[2,1])
      //=> [4,5,6,2,1]
{: .language-javascript}     
      
But now we thoroughly understand what `a.b()` really means: It's synonymous with `a.b.call(a)`. Whereas in a browser, `c()` is synonymous with `c.call(window)`.

### apply, arguments, and contextualization

JavaScript has another automagic binding in every function's environment. `arguments` is a special object that behaves a little like an array.[^little]

[^little]: Just enough to be frustrating, to be perfectly candid!

For example:

    var third = function () {
      return arguments[2]
    }

    third(77, 76, 75, 74, 73)
      //=> 75
{: .language-javascript}     

Hold that thought for a moment. JavaScript also provides a fourth way to set the context for a function. `apply` is a method implemented by every function that takes a context as its first argument, and it takes an array or array-like thing of arguments as its second argument. That's a mouthful, let's look at an example:

    third.call(this, 1,2,3,4,5)
      //=> 3

    third.apply(this, [1,2,3,4,5])
      //=> 3
{: .language-javascript}     
      
Now let's put the two together. Here's another travesty:

    var a = [1,2,3],
        accrete = a.concat;
        
    accrete([4,5])
      //=> Gobbledygook!
{: .language-javascript}     

We get the result of concatenating `[4,5]` onto an array containing the global environment. Not what we want! Behold:

    var contextualize = function (fn, context) {
      return function () {
        return fn.apply(context, arguments);
      }
    }
    
    accrete = contextualize(a.concat, a);
    accrete([4,5]);
      //=> [ 1, 2, 3, 4, 5 ]
{: .language-javascript}     
      
Our `contextualize` function returns a new function that calls a function with a fixed context. It can be used to fix some of the unexpected results we had above. Consider:

    var aFourthObject = {},
        returnThis = function () { return this; };
        
    aFourthObject.uncontextualized = returnThis;
    aFourthObject.contextualized = contextualize(returnThis, aFourthObject);
    
    aFourthObject.uncontextualized() === aFourthObject
      //=> true
    aFourthObject.contextualized() === aFourthObject
      //=> true
{: .language-javascript}     
      
Both are `true` because we are accessing them with `aFourthObject.` Now we write:

    var uncontextualized = aFourthObject.uncontextualized,
        contextualized = aFourthObject.contextualized;
        
    uncontextualized() === aFourthObject;
      //=> false
    contextualized() === aFourthObject
      //=> true
{: .language-javascript}     
      
When we call these functions without using `aFourthObject.`, only the contextualized version maintains the context of `aFourthObject`.
      
We'll return to contextualizing methods later, in [Binding](#binding). But before we dive too deeply into special handling for methods, we need to spend a little more time looking at how functions and methods work.

## Method Decorators {#method-decorators}

In [function decorators](#decorators), we learned that a decorator takes a function as an argument, returns a function, and there's a semantic relationship between the two. If a function is a verb, a decorator is an adverb.

Decorators can be used to decorate methods provided that they carefully preserve the function's context. For example, here is a naïve version of `maybe` for one argument:

    function maybe (fn) {
      return function (argument) {
        if (argument != null) {
          return fn(argument)
        }
      }
    }
{: .language-javascript}     

This version doesn't preserve the context, so it can't be used as a method decorator. Instead, we have to write:

    function maybe (fn) {
      return function (argument) {
        if (argument != null) {
          return fn.call(this, argument)
        }
      }
    }
{: .language-javascript}     

Now we can write things like:

    someObject = {
      setSize: maybe(function (size) {
        this.size = size;
        return this
      })
    }
{: .language-javascript}     

And `this` is correctly set:

    someObject.setSize(5)
      //=> { setSize: [Function], size: 5 }
{: .language-javascript}     

Using `.call` or `.apply` and `arguments` is substantially slower than writing function decorators that don't set the context, so it might be right to sometimes write function decorators that aren't usable as method decorators. However, in practice you're far more likely to introduce a defect by failing to pass the context through a decorator than by introducing a performance pessimization, so the default choice should be to write all function decorators in such a way that they are "context agnostic."

In some cases, there are other considerations to writing a method decorator. If the decorator introduces state of any kind (such as `once` and `memoize` do), this must be carefully managed for the case when several objects share the same method through the mechanism of the [prototype](#prototypes) or through sharing references to the same function.

## Summary

T> ### Objects, Mutation, and State
T>
T> * State can be encapsulated/hidden with closures.
T> * Encapsulations can be aggregated with composition.
T> * Encapsulation resists extension.
T> * The automagic binding `this` facilitates sharing of functions.
T> * Functions can be named and declared with a name.
