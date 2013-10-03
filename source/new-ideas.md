# New Ideas {#redecorating}

![The delight of coffee is that it transports you to another world](train_1200.jpg)

(*this bonus chapter is a work-in-progress*)

## How Prototypes and Constructors differ from Classes

In the previous section, we said that JavaScript has "classes" for some definition of the word "class," and we showed how JavaScript provides many of the features found in other "object-oriented languages." For those who want a fuller explanation, this section goes into more detail about how JavaScript's "prototypes" differ from the classes found in a language like Ruby. It is not necessary to read this section to understand programming in JavaScript, but it can be helpful when discussing JavaScript with programmers who are more comfortable talking about classes.

---

Although each "object-oriented" programming language has its own particular set of semantics, the majority in popular use have "classes." A class is an entity responsible for creating objects and defining the behaviour of objects. Classes may be objects in their own right, but if they are, they're different from other types of objects. For example, the `String` class in Ruby is not itself a string, it's an object whose class is `Class`. All objects in a "classical" system have a class, and their class is a "class."

That sounds tautological, until we look at JavaScript. But let's start with a quick review of a popular classist language, Ruby.

### ruby

In Ruby, classes are objects, but they're special objects. For example, here are some of the methods associated with the Ruby class `String`:

    String.methods
      #=> [:try_convert, :allocate, :new, :superclass, :freeze, :===, :==,
           :<=>, :<, :<=, :>, :>=, :to_s, :included_modules, :include?, :name, 
           :ancestors, :instance_methods, :public_instance_methods, 
           :protected_instance_methods, :private_instance_methods, :constants, 
           :const_get, :const_set, :const_defined?, :const_missing, 
           :class_variables, :remove_class_variable, :class_variable_get, 
           :class_variable_set, :class_variable_defined?, :public_constant, 
           :private_constant, :module_exec, :class_exec, :module_eval, :class_eval, 
           :method_defined?, :public_method_defined?, :private_method_defined?, 
           :protected_method_defined?, :public_class_method, :private_class_method, 
           # ...
           :!=, :instance_eval, :instance_exec, :__send__, :__id__] 
{: .language-ruby}

And here are some of the methods associated with an instance of a string:

    String.new.methods
      #=> [:<=>, :==, :===, :eql?, :hash, :casecmp, :+, :*, :%, :[],
           :[]=, :insert, :length, :size, :bytesize, :empty?, :=~,
           :match, :succ, :succ!, :next, :next!, :upto, :index, :rindex,
           :replace, :clear, :chr, :getbyte, :setbyte, :byteslice,
           :to_i, :to_f, :to_s, :to_str, :inspect, :dump, :upcase,
           :downcase, :capitalize, :swapcase, :upcase!, :downcase!,
           :capitalize!, :swapcase!, :hex, :oct, :split, :lines, :bytes,
           :chars, :codepoints, :reverse, :reverse!, :concat, :<<,
           :prepend, :crypt, :intern, :to_sym, :ord, :include?,
           :start_with?, :end_with?, :scan, :ljust, :rjust, :center,
           # ...
           :instance_eval, :instance_exec, :__send__, :__id__]
{: .language-ruby}

As you can see, a "class" in Ruby is very different from an "instance of that class." And the methods of a class are very different from the methods of an instance of that class.

Here's how you define a Queue in Ruby:

    class Queue
      def initialize
        @array, @head, @tail = [], 0, -1
      end
  
      def pushTail value
        @array[@tail += 1] = value
      end
  
      def pullHead
        if !@isEmpty
          @array[@head]).tap { |value|
            @array[@head] = null
            @head += 1
          }
        end
      end
  
      def isEmpty
        !!(@tail < @head)
      end
    end
{: .language-ruby}

There is special syntax for defining a class, and special syntax for defining the behaviour of instances. There are different ways of defining the way new instances are created in classist languages. Ruby uses a "magic method" called `initialize`. Now let's look at JavaScript.

### javascript has constructors and prototypes

JavaScript objects don't have a formal class, and thus there's no special syntax for defining how to create an instance or define its behaviour.

JavaScript instances are created with a *constructor*. The constructor of an instance is a function that was invoked with the `new` operator. In JavaScript, any function can be a constructor, even if it doesn't look like one:

    function square (n) { return n * n; }
      //=> undefined
    square(2)
      //=> 4
    square(2).constructor
      //=> [Function: Number]
    new square(2)
      //=> {}
    new square(2).constructor
      //=> [Function: square]
{: .language-javascript}

As you can see, the `square` function will act as a constructor if you call it with `new`. *There is no special kind of thing that constructs new objects, every function is (potentially) a constructor*.

That's different from a true classical language, where the class is a special kind of object that creates new instances.

How does JavaScript define the behaviour of instances? JavaScript doesn't have a special syntax or special kind of object for that, it has "prototypes." Prototypes are objects, but unlike a classical system, there are no special methods or properties associated with a prototype. Any object can be a prototype, even an empty object. In fact, that's exactly what is associated with a constructor by default:

    function Nullo () {};
    Nullo.prototype
      //=> {}
{: .language-javascript}
      
There's absolutely nothing special about a prototype object. No special class methods, no special constructor of its own, nothing. Let's look at a simple Queue in JavaScript:

    var Queue = function () {
      this.array = [];
      this.head = 0;
      this.tail = -1;
    };
  
    Queue.prototype.pushTail = function (value) {
      return this.array[this.tail += 1] = value;
    };
    Queue.prototype.pullHead = function () {
      var value;
  
      if (!this.isEmpty()) {
        value = this.array[this.head];
        this.array[this.head] = void 0;
        this.head += 1;
        return value;
      }
    };
    Queue.prototype.isEmpty = function () {
      return this.tail < this.head;
    };

    Queue.prototype
      //=>  { pushTail: [Function],
      //      pullHead: [Function],
      //      isEmpty: [Function] }
{: .language-javascript}

The first way a prototype in JavaScript is different from a class in Ruby is that the prototype is an ordinary object with exactly the same properties that we expect to find in an instance: Methods `pushTail`, `pullHead`, and `isEmpty`.

The second way is that *any* object can be a prototype. It can have functions (which act like methods), it can have other values (like numbers, booleans, objects, or strings). It can be an object you're using for something else: An account, a view, a DOM object if you're in the browser, anything.

"Classes" are objects in most "classical" languages, but they are a special kind of object. In JavaScript, prototypes are not a special kind of object, they're just objects.

### summary of the difference between classes and prototypes

A class in a formal classist language can be an object, but it's a special kind of object with special properties and methods. It is responsible for creating new instances and for defining the behaviour of instances.

Instance behaviour in a classist language is defined with special syntax. If changes are allowed dynamically, they are done with special syntax and/or special methods invoked on the class.

JavaScript splits the responsibility for instances into a constructor and a prototype. A constructor in JavaScript can be any function. Constructors are responsible for creating new instances.

A prototype in JavaScript can be any object. Prototypes are responsible for defining the behaviour of instances. prototypes don't have special methods or properties.

Instance behaviour in JavaScript is defined by modifying the prototype directly, e.g. by adding functions to it as properties. There is no special syntax for defining a class or modifying a class.

### so why does this book say that javascript has "classes" for some definition of "class?"

Because, **if**:

1. You use a function as a constructor, and;
2. You use a prototype for defining instance methods, and;
3. The prototype is used strictly for defining the instance methods and nothing else;

**Then**:

You will have something that works just like a simple class-based system, with the constructor function and its prototype acting as the "class."

But if you want more, you have a flexible system that does allow you to do [much much more][fd]. It's up to you.

[fd]: https://github.com/raganwald/homoiconic/blob/master/2013/01/function_and_method_decorators.md#function-and-method-decorators "Function and Method Decorators"

## New-Agnostic Constructors {#new-agnostic}

JavaScript is inflexible about certain things. One of them is invoking `new` on a constructor. In many of our recipes, we can write functions that can handle a variable number of arguments and use `.apply` to invoke a function. For example:

    function fluent (methodBody) {
      return function () {
        methodBody.apply(this, arguments);
        return this
      }
    }
{: .language-javascript}

You can't do the same thing with calling a constructor. This will not work:

    function User (name, password) {
      this.name = name || 'Untitled';
      this.password = password
    };
    
    function withDefaultPassword () {
      var args = Array.prototype.slice.call(arguments, 0);
      args[1] = 'swordfish';
      return new User.apply(this, args);
    } 
    
    withDefaultPassword('James')
      //=> TypeError: function apply() { [native code] } is not a constructor
{: .language-javascript}

Another weakness of constructors is that if you call them without using `new`, you usually get nonsense:

    User('James', 'swordfish')
      //=> undefined
{: .language-javascript}

In David Herman's [Effective JavaScript][ejs], he describes the "New-Agnostic Constructor Pattern." He gives several variations, but the simplest is this:

    function User (name, password) {
      if (!(this instanceof User)) {
        return new User(name, password);
      }
      this.name = name || 'Untitled';
      this.password = password
    };
{: .language-javascript}

Now you can call the constructor without the `new` password:

    User('James', 'swordfish')
      //=> { name: 'James', password: 'swordfish' }
{: .language-javascript}
  
This in turn opens up the possibility of doing dynamic things with constructors that didn't work when you were forced to use `new`:
    
    function withDefaultPassword () {
      var args = Array.prototype.slice.call(arguments, 0);
      args[1] = 'swordfish';
      return User.apply(this, args);
    } 
    
    withDefaultPassword('James')
      //=> { name: 'James', password: 'swordfish' }
{: .language-javascript}
      
(The pattern above has a tradeoff: It works for all circumstances except when you want to set up an inheritance hierarchy.)

[ejs]: http://www.amazon.com/gp/product/B00AC1RP14/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B00AC1RP14&linkCode=as2&tag=raganwald001-20 "Effective JavaScript: 68 Specific Ways to Harness the Power of JavaScript"

## Another New-Agnostic Constructor Pattern

Here's another way to write a new-agnostic constructor:

    function User (name, password) {
      var self = this instanceof User ? this : new User();
      if (name != null) {
        self.name = name;
        self.password = password;
      }
      return self;
    };
{: .language-javascript}
    
The principle is that the constructor initializes an object assigned to the variable `self` and returns it. When you call the constructor with `new`, then `self` will be assigned the current context. But if you call this constructor as a standard function, then it will call itself without parameters and assign the newly created User to `self`.

## New-Agnostic Constructors {#new-agnostic}

JavaScript is inflexible about certain things. One of them is invoking `new` on a constructor. In many of our recipes, we can write functions that can handle a variable number of arguments and use `.apply` to invoke a function. For example:

    function fluent (methodBody) {
      return function () {
        methodBody.apply(this, arguments);
        return this
      }
    }
{: .language-javascript}

You can't do the same thing with calling a constructor. This will not work:

    function User (name, password) {
      this.name = name || 'Untitled';
      this.password = password
    };
    
    function withDefaultPassword () {
      var args = Array.prototype.slice.call(arguments, 0);
      args[1] = 'swordfish';
      return new User.apply(this, args);
    } 
    
    withDefaultPassword('James')
      //=> TypeError: function apply() { [native code] } is not a constructor
{: .language-javascript}

Another weakness of constructors is that if you call them without using `new`, you usually get nonsense:

    User('James', 'swordfish')
      //=> undefined

In David Herman's [Effective JavaScript][ejs], he describes the "New-Agnostic Constructor Pattern." He gives several variations, but the simplest is this:

    function User (name, password) {
      if (!(this instanceof User)) {
        return new User(name, password);
      }
      this.name = name || 'Untitled';
      this.password = password
    };
{: .language-javascript}

Now you can call the constructor without the `new` password:

    User('James', 'swordfish')
      //=> { name: 'James', password: 'swordfish' }
{: .language-javascript}
  
This in turn opens up the possibility of doing dynamic things with constructors that didn't work when you were forced to use `new`:
    
    function withDefaultPassword () {
      var args = Array.prototype.slice.call(arguments, 0);
      args[1] = 'swordfish';
      return User.apply(this, args);
    } 
    
    withDefaultPassword('James')
      //=> { name: 'James', password: 'swordfish' }
 {: .language-javascript}
     
(The pattern above has a tradeoff: It works for all circumstances except when you want to set up an inheritance hierarchy.)

[ejs]: http://www.amazon.com/gp/product/B00AC1RP14/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B00AC1RP14&linkCode=as2&tag=raganwald001-20 "Effective JavaScript: 68 Specific Ways to Harness the Power of JavaScript"

## Another New-Agnostic Constructor Pattern

Here's another way to write a new-agnostic constructor:

    function User (name, password) {
      var self = this instanceof User ? this : new User();
      if (name != null) {
        self.name = name;
        self.password = password;
      }
      return self;
    };
{: .language-javascript}
    
The principle is that the constructor initializes an object assigned to the variable `self` and returns it. When you call the constructor with `new`, then `self` will be assigned the current context. But if you call this constructor as a standard function, then it will call itself without parameters and assign the newly created User to `self`.

## Mixins {#functional-mixins}

In [A Class By Any Other Name](#class-other-name), we saw that you can emulate "mixins" using our `extend` function. We'll revisit this subject now and spend more time looking at mixing functionality into classes.

First, a quick recap: In JavaScript, a "class" is implemented as a constructor function and its prototype. Instances of the class are created by calling the constructor with `new`. They "inherit" shared behaviour from the constructor's `prototype` property. One way to share behaviour scattered across multiple classes, or to untangle behaviour by factoring it out of an overweight prototype, is to extend a prototype with a mixin.

Here's an evolved class of todo items we saw earlier:

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
    
    Todo.prototype;
      //=> { do: [Function], undo: [Function] }
{: .language-javascript}

And a "mixin:"

    var ColourCoded = {
      setColourRGB: fluent( function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
      }),
      getColourRGB: function () {
        return this.colourCode;
      }
    };
{: .language-javascript}
    
Mixing colour coding into our Todo prototype is straightforward:

    extend(Todo.prototype, ColourCoded);
    
    Todo.prototype;
      //=> { do: [Function],
      //     undo: [Function],
      //     setColourRGB: [Function],
      //     getColourRGB: [Function] }
{: .language-javascript}
    
### what is a "mixin?"

Like "class," the word "mixin" means different things to different people. A Ruby user will talk about modules, for example. And a JavaScript user could in truth say that everything is an object and we're just extending one object (that happens to be a prototype) with the properties of another object (that just happens to contain some functions).

A simple definition that works for most purposes is to define a mixin as: *A collection of behaviour that can be added to a class's existing prototype*. `ColourCoded` above is such a mixin. If we had to actually assign a new prototype to the `Todo` class, that wouldn't be mixing functionality in, that would be replacing functionality.

### functional mixins

The mixin we have above works properly, but our little recipe had two distinct steps: Define the mixin and then extend the class prototype. Angus Croll pointed out that it's far more elegant to define a mixin as a function rather than an object. He calls this a [functional mixin][fm]. Here's our `ColourCoded` recast in functional form:

    function becomeColourCoded (target) {
      target.setColourRGB = fluent( function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
      });
      
      target.getColourRGB = function () {
        return this.colourCode;
      };
      
      return target;
    };
    
    becomeColourCoded(Todo.prototype);
    
    Todo.prototype;
      //=> { do: [Function],
      //     undo: [Function],
      //     setColourRGB: [Function],
      //     getColourRGB: [Function] }
{: .language-javascript}
      
Notice that we mix the functionality into the prototype. This keeps our mixing flexible: You could mix functionality directly into an object if you so choose. Twitter's [Flight] framework uses a variation on this technique that targets the mixin function's context:

    function asColourCoded () {
      this.setColourRGB = fluent( function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
      });
      
      this.getColourRGB = function () {
        return this.colourCode;
      };
      
      return this;
    };
    
    asColourCoded.call(Todo.prototype);
{: .language-javascript}
    
This approach has some subtle benefits: You can use mixins as methods, for example. It's possible to write a context-agnostic functional mixin:

    function colourCoded () {
      if (arguments.length[0] !== void 0) {
        return colourCoded.call(arguments[0]);
      }
      this.setColourRGB = fluent( function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
      });
      
      this.getColourRGB = function () {
        return this.colourCode;
      };
      
      return this;
    };
{: .language-javascript}

Bueno!

[fm]: https://javascriptweblog.wordpress.com/2011/05/31/a-fresh-look-at-javascript-mixins/ "A fresh look at JavaScript Mixins"
[Flight]: https://twitter.github.com/flight

## Class Decorators {#class-decorators}

[Functional Mixins](#functional-mixins) look a lot like the function and method decorators we've seen. The big difference is that the mixin alters its subject, whereas the function decorators return a new function that wraps the old one. That can be handy if we wish, for example, to have some Todos that are not colour coded and we don't want to have a wild hierarchy of inheritance or if we wish to dynamically mix functionality into a class.

> There is a strong caveat: At this time, JavaScript is inflexible about dynamically paramaterizing calls to constructors. Therefore,
the function decorator pattern being discussed here only works with constructors that are [new agnostic](#new-agnostic)[^create] and that can
create an "empty object."

[^create]: Another approach that works with ECMASCRipt 5 and later is to base all classes around [Object.create](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/create)

Once again, our Todo class:

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
    
    Todo.prototype;
      //=> { do: [Function], undo: [Function] }
{: .language-javascript}

Here's our `ColourCoded` as a class decorator: It returns a new class rather than modifying `ToDo`:

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
    
    var ColourTodo = AndColourCoded(Todo);
    
    Todo.prototype;
      //=> { do: [Function], undo: [Function] }
    
    var colourTodo = new ColourTodo('Write more JavaScript');
    colourTodo.setColourRGB(255, 255, 0);
      //=> { name: 'Write more JavaScript',
      //     done: false,
      //     colourCode: { r: 255, g: 255, b: 0 } }
      
    colourTodo instanceof Todo
      //=> true
      
    colourTodo instanceof ColourTodo
      //=> true
{: .language-javascript}

Although the implementation is more subtle, class decorators can be an improvement on functional mixins when you wish to avoid destructively modifying an existing prototype.

## Interlude: Tortoises, Hares, and Teleporting Turtles {#tortoises}

A good long while ago (The First Age of Internet Startups), someone asked me one of those pet algorithm questions. It was, "Write an algorithm to detect a loop in a linked list, in constant space."

I'm not particularly surprised that I couldn't think up an answer in a few minutes at the time. And to the interviewer's credit, he didn't terminate the interview on the spot, he asked me to describe the kinds of things going through my head.

I think I told him that I was trying to figure out if I could adapt a hashing algorithm such as XORing everything together. This is the "trick answer" to a question about finding a missing integer from a list, so I was trying the old, "Transform this into [a problem you've already solved](http://www-users.cs.york.ac.uk/susan/joke/3.htm#boil)" meta-algorithm. We moved on from there, and he didn't reveal the "solution."

I went home and pondered the problem. I wanted to solve it. Eventually, I came up with something and tried it (In Java!) on my home PC. I sent him an email sharing my result, to demonstrate my ability to follow through. I then forgot about it for a while. Some time later, I was told that the correct solution was:

    var LinkedList, list, tortoiseAndHareLoopDetector;

    LinkedList = (function() {

      function LinkedList(content, next) {
        this.content = content;
        this.next = next != null ? next : void 0;
      }

      LinkedList.prototype.appendTo = function(content) {
        return new LinkedList(content, this);
      };

      LinkedList.prototype.tailNode = function() {
        var nextThis;
        return ((nextThis = this.next) != null ? nextThis.tailNode() : void 0) || this;
      };

      return LinkedList;

    })();

    tortoiseAndHareLoopDetector = function(list) {
      var hare, tortoise, nextHare;
      tortoise = list;
      hare = list.next;
      while ((tortoise != null) && (hare != null)) {
        if (tortoise === hare) {
          return true;
        }
        tortoise = tortoise.next;
        hare = (nextHare = hare.next) != null ? nextHare.next : void 0;
      }
      return false;
    };

    list = new LinkedList(5).appendTo(4).appendTo(3).appendTo(2).appendTo(1);

    tortoiseAndHareLoopDetector(list);
      //=> false

    list.tailNode().next = list.next;

    tortoiseAndHareLoopDetector(list);
      //=> true
{: .language-javascript}
  
This algorithm is called "The Tortoise and the Hare," and was discovered by Robert Floyd in the 1960s. You have two node references, and one traverses the list at twice the speed of the other. No matter how large it is, you will eventually have the fast reference equal to the slow reference, and thus you'll detect the loop.

At the time, I couldn't think of any way to use hashing to solve the problem, so I gave up and tried to fit this into a powers-of-two algorithm. My first pass at it was clumsy, but it was roughly equivalent to this:

    var list, teleportingTurtleLoopDetector;

    teleportingTurtleLoopDetector = function(list) {
      var i, rabbit, speed, turtle;
      speed = 1;
      turtle = rabbit = list;
      while (true) {
        for (i = 0; i <= speed; i += 1) {
          rabbit = rabbit.next;
          if (rabbit == null) {
            return false;
          }
          if (rabbit === turtle) {
            return true;
          }
        }
        turtle = rabbit;
        speed *= 2;
      }
      return false;
    };

    list = new LinkedList(5).appendTo(4).appendTo(3).appendTo(2).appendTo(1);

    teleportingTurtleLoopDetector(list);
      //=> false

    list.tailNode().next = list.next;

    teleportingTurtleLoopDetector(list);
      //=> true
{: .language-javascript}
  
Years later, I came across a discussion of this algorithm, [The Tale of the Teleporting Turtle](http://www.penzba.co.uk/Writings/TheTeleportingTurtle.html). It seems to be faster under certain circumstances, depending on the size of the loop and the relative costs of certain operations.

What's interesting about these two algorithms is that they both *tangle* two separate concerns: How to traverse a data structure, and what to do with the elements that you encounter. In [Functional Iterators](#functional-iterators), we'll investigate one pattern for separating these concerns.

## Functional Iterators {#functional-iterators}

Let's consider a remarkably simple problem: Finding the sum of the elements of an array. In iterative style, it looks like this:

    function sum (array) {
      var number, total, len;
      total = 0;
      for (i = 0, len = array.length; i < len; i++) {
        number = array[i];
        total += number;
      }
      return total;
    };
{: .language-javascript}
  
What's the sum of a linked list of numbers? How about the sum of a tree of numbers (represented as an array of array of numbers)? Must we re-write the `sum` function for each data structure?

There are two roads ahead. One involves a generalized `reduce` or `fold` method for each data structure. The other involves writing an [Iterator](https://developer.mozilla.org/en-US/docs/JavaScript/New_in_JavaScript/1.7#Iterators) for each data structure and writing our `sum` to take an iterator as its argument. Let's use iterators, especially since we need two different iterators for the same data structure, so a single object method is inconvenient.

Since we don't have iterators baked into the underlying JavaScript engine yet, we'll write our iterators as functions:

    var LinkedList, list;

    LinkedList = (function() {

      function LinkedList(content, next) {
        this.content = content;
        this.next = next != null ? next : void 0;
      }

      LinkedList.prototype.appendTo = function(content) {
        return new LinkedList(content, this);
      };

      LinkedList.prototype.tailNode = function() {
        var nextThis;
        return ((nextThis = this.next) != null ? nextThis.tailNode() : void 0) || this;
      };

      return LinkedList;

    })();

    function ListIterator (list) {
      return function() {
        var node;
        node = list != null ? list.content : void 0;
        list = list != null ? list.next : void 0;
        return node;
      };
    };

    function sum (iter) {
      var number, total;
      total = 0;
      number = iter();
      while (number != null) {
        total += number;
        number = iter();
      }
      return total;
    };

    list = new LinkedList(5).appendTo(4).appendTo(3).appendTo(2).appendTo(1);

    sum(ListIterator(list));
      //=> 15

    function ArrayIterator (array) {
      var index;
      index = 0;
      return function() {
        return array[index++];
      };
    };

    sum(ArrayIterator([1, 2, 3, 4, 5]));
      //=> 15
{: .language-javascript}
  
Summing an array that can contain nested arrays adds a degree of complexity. Writing a function that iterates recursively over a data structure is an interesting problem, one that is trivial in a language with [coroutines](https://en.wikipedia.org/wiki/Coroutine). Since we don't have Generators yet, and we don't want to try to turn our loop detection inside-out, we'll Greenspun our own coroutine by maintaining our own stack.

> This business of managing your own stack may seem weird to anyone born after 1970, but old fogeys fondly remember that after walking barefoot to and from University uphill in a blizzard both ways, the interview question brain teaser of the day was to write a "Towers of Hanoi" solver in a language like BASIC that didn't have reentrant subroutines.

    function LeafIterator (array) {
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

    sum(LeafIterator([1, [2, [3, 4]], [5]]));
      //=> 15
{: .language-javascript}
  
We've successfully separated the issue of what one does with data from how one traverses over the elements.

### folding

Just as pure functional programmers love to talk monads, newcomers to functional programming in multi-paradigm languages often drool over [folding] a/k/a mapping/injecting/reducing. We're just a level of abstraction away:

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

    function foldingSum (iter) {
      return fold(iter, (function(x, y) {
        return x + y;
      }), 0);
    };

    foldingSum(LeafIterator([1, [2, [3, 4]], [5]]));
      //=> 15
{: .language-javascript}
  
Fold turns an iterator over a finite data structure into an accumulator. And once again, it works with any data structure. You don't need a different kind of fold for each kind of data structure you use.

[folding]: https://en.wikipedia.org/wiki/Fold_(higher-order_function)

### unfolding and laziness

Iterators are functions. When they iterate over an array or linked list, they are traversing something that is already there. But they could, in principle, manufacture the data as they go. Let's consider the simplest example:

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

    fromOne = NumberIterator(1);

    fromOne();
      //=> 1
    fromOne();
      //=> 2
    fromOne();
      //=> 3
    fromOne();
      //=> 4
    fromOne();
      //=> 5
{: .language-javascript}
  
And here's another one:

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
      
    fib = FibonacciIterator()

    fib()
      //=> 1
    fib()
      //=> 1
    fib()
      //=> 2
    fib()
      //=> 3
    fib()
      //=> 5
{: .language-javascript}
  
A function that starts with a seed and expands it into a data structure is called an *unfold*. It's the opposite of a fold. It's possible to write a generic unfold mechanism, but let's pass on to what we can do with unfolded iterators.

This business of going on forever has some drawbacks. Let's introduce an idea: A function that takes an Iterator and returns another iterator. We can start with `take`, an easy function that returns an iterator that only returns a fixed number of elements:

    take = function(iter, numberToTake) {
      var count;
      count = 0;
      return function() {
        if (++count <= numberToTake) {
          return iter();
        } else {
          return void 0;
        }
      };
    };

    oneToFive = take(NumberIterator(1), 5);

    oneToFive();
      //=> 1
    oneToFive();
      //=> 2
    oneToFive();
      //=> 3
    oneToFive();
      //=> 4
    oneToFive();
      //=> 5
    oneToFive();
      //=> undefined
{: .language-javascript}
  
With `take`, we can do things like return the squares of the first five numbers:

    square(take(NumberIterator(1), 5))

      //=> [ 1,
      //     4,
      //     9,
      //     16,
      //     25 ]
{: .language-javascript}
  
How about the squares of the odd numbers from the first five numbers?

    square(odds(take(NumberIterator(1), 5)))
      //=> TypeError: object is not a function
{: .language-javascript}
  
Bzzzt! Our `odds` function returns an array, not an iterator.

    square(take(odds(NumberIterator(1)), 5))
      //=> RangeError: Maximum call stack size exceeded
{: .language-javascript}
  
You can't take the first five odd numbers at all, because `odds` tries to get the entire set of numbers and accumulate the odd ones in an array. This can be fixed. For unfolds and other infinite iterators, we need more functions that transform one iterator into another:

  function iteratorMap (iter, unaryFn) {
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

  function squaresIterator (iter) {
    return iteratorMap(iter, function(n) {
      return n * n;
    });
  };

  function iteratorFilter (iter, unaryPredicateFn) {
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

  function oddsFilter (iter) {
    return iteratorFilter(iter, odd);
  };
{: .language-javascript}
  
Now we can do things like take the sum of the first five odd squares of fibonacci numbers:

    foldingSum(take(oddsFilter(squaresIterator(FibonacciIterator())), 5))
      //=> 205
{: .language-javascript}
  
This solution composes the parts we already have, rather than writing a tricky bit of code with ifs and whiles and boundary conditions.

### summary

Untangling the concerns of how to iterate over data from what to do with data leads us to thinking of iterators and working directly with iterators. For example, we can map and filter iterators rather than trying to write separate map and filter functions or methods for each type of data structure. This leads to the possibility of working with lazy or infinite iterators.

### caveat

Please note that unlike most of the other functions discussed in this book, iterators are *stateful*. There are some important implications of stateful functions. One is that while functions like `take(...)` appear to create an entirely new iterator, in reality they return a decorated reference to the original iterator. So as you traverse the new decorator, you're changing the state of the original!

For all intents and purposes, once you pass an iterator to a function, you can expect that you no longer "own" that iterator, and that its state either has changed or will change.

## Refactoring to Functional Iterators

In [Tortoises, Hares, and Teleporting Turtles](#tortoises), we looked at the "Tortoise and Hare" algorithm for detecting a linked list. Like many such algorithms, it "tangles" two different concerns:

1. The mechanism for iterating over a list.
2. The algorithm for detecting a loop in a list.

    var LinkedList = (function() {

      function LinkedList(content, next) {
        this.content = content;
        this.next = next != null ? next : void 0;
      }

      LinkedList.prototype.appendTo = function(content) {
        return new LinkedList(content, this);
      };

      LinkedList.prototype.tailNode = function() {
        var nextThis;
        return ((nextThis = this.next) != null ? nextThis.tailNode() : void 0) || this;
      };

      return LinkedList;

    })();

    function tortoiseAndHareLoopDetector (list) {
      var hare, tortoise, nextHare;
      tortoise = list;
      hare = list.next;
      while ((tortoise != null) && (hare != null)) {
        if (tortoise === hare) {
          return true;
        }
        tortoise = tortoise.next;
        hare = (nextHare = hare.next) != null ? nextHare.next : void 0;
      }
      return false;
    };
{: .language-javascript}

### functional iterators

We then went on to discuss how to use [functional iterators](#functional-iterators) to untangle concerns like this. A functional iterator is a stateful function that iterates over a data structure. Every time you call it, it returns the next element from the data structure. If and when it completes its traversal, it returns `undefined`.

For example, here is a function that takes an array and returns a functional iterator over the array:

    function ArrayIterator (array) {
      var index = 0;
      return function() {
        return array[index++];
      };
    };
{: .language-javascript}

Iterators allow us to write (or refactor) functions to operate on iterators instead of data structures. That increases reuse. We can also write higher-order functions that operate directly on iterators such as mapping and selecting. That allows us to write lazy algorithms.

### refactoring the tortoise and hare

Now we'll refactor the Tortoise and Hare to use iterators instead of directly operate on linked lists. We'll add an `.iterator()` method to linked lists, and we'll rewrite our loop detector function to take an "iterable" instead of a list:

    LinkedList.prototype.iterator = function() {
      var list = this;
      return function() {
        var value = list != null ? list.content : void 0;
        list = list != null ? list.next : void 0;
        return value;
      };
    };

    function tortoiseAndHareLoopDetector (iterable) {
      var tortoise = iterable.iterator(),
          hare = iterable.iterator(), 
          tortoiseValue, 
          hareValue;
      while (((tortoiseValue = tortoise()) != null) && ((hare(), hareValue = hare()) != null)) {
        if (tortoiseValue === hareValue) {
          return true;
        }
      }
      return false;
    };

    list = new LinkedList(5).appendTo(4).appendTo(3).appendTo(2).appendTo(1);

    tortoiseAndHareLoopDetector(list);
      //=> false

    list.tailNode().next = list.next;

    tortoiseAndHareLoopDetector(list);
      //=> true
{: .language-javascript}

We have now refactored it into a function that operates on anything that responds to the `.iterate()` method. It's classic "Duck Typed" Object-Orientation. So, how shall we put it to work?

## A Drunken Walk Across A Chequerboard

Here's another job interview puzzle.[^yecch]

[^yecch]: This book does not blindly endorse asking programmers to solve this or any abstract problem in a job interview.

*Consider a finite checkerboard of unknown size. On each square we randomly place an arrow pointing to one of its four sides. For convenience, we shall uniformly label the directions: N, S, E, and W. A chequer is placed randomly on the checkerboard. Each move consists of moving the red chequer one square in the direction of the arrow in the square it occupies. If the arrow should cause the chequer to move off the edge of the board, the game halts.*

*As a player moves the chequer, he calls out the direction of movement, e.g. "N, E, N, S, N, E..." Write an algorithm that will determine whether the game halts strictly from the called out directions, in constant space.*

### the insight

Our solution will rest on the observation that as the chequer follows a path, if it ever visits a square for a second time, it will cycle indefinitely without falling off the board. Otherwise, on a finite board, it must eventually fall off the board after at most visiting every square once.

Therefore, if we think of this as detecting whether the chequer revisits a square in constant space, we can see this is isomorphic to detecting whether a linked list has a loop by checking to see whether it revisits the same node.

### the game

In essence, we're given an object that has a `.iterator()` method. That gives us an iterator, and each time we call the iterator, we get a direction. Here it is:

    var DIRECTION_TO_DELTA = {
      N: [1, 0],
      E: [0, 1],
      S: [-1, 0],
      W: [0, -1]
    };

    var Game = (function () {
      function Game (size) {
        var i,
            j;
        
        this.size = size
                    ? Math.floor(Math.random() * 8) + 8
                    : size ;
        this.board = [];
        for (i = 0; i < this.size; ++i) {
          this.board[i] = [];
          for (j = 0; j < this.size; ++j) {
            this.board[i][j] = 'NSEW'[Math.floor(Math.random() * 4)];
          }
        }
        this.initialPosition = [
          2 + Math.floor(Math.random() * (this.size - 4)), 
          2 + Math.floor(Math.random() * (this.size - 4))
        ];
        return this;
      };
      
      Game.prototype.contains = function (position) {
        return position[0] >= 0 && position[0] < this.size && position[1] >= 0 && position[1] < this.size;
      };
      
      Game.prototype.iterator = function () {
        var position = [this.initialPosition[0], this.initialPosition[1]];
        return function () {
          var direction;
          if (this.contains(position)) {
            direction = this.board[position[0]][position[1]];
            position[0] += DIRECTION_TO_DELTA[direction][0];
            position[1] += DIRECTION_TO_DELTA[direction][1];
            return direction;
          }
          else {
            return void 0;
          }
        }.bind(this);
      };
      
      return Game;
      
    })();

    var i = new Game().iterator();
      //=> [Function]
    i();
      //=> 'N'
    i();
      //=> 'S'
    i();
      //=> 'N'
    i();
      //=> 'S'
      //   ...
{: .language-javascript}

In the example above, we have the smallest possible repeating path: The chequer shuttles back and forth between two squares. It will not always be so obvious when a game does not terminate.

### stateful mapping

Our goal is to transform the iteration of directions into an iteration that the Tortoise and Hare can use to detect revisiting the same square. Our approach is to convert the directions into offsets representing the position of the chequer relative to its starting position.

We'll use a `statefulMap`:

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
{: .language-javascript}

`statefulMap` takes in iterator and maps it to a new iterator. Unlike a "regular" map, it is computes its elements on demand, so it will not run indefinitely when given an iteration representing an infinitely looping chequer. We need a stateful map because we are tracking a position that changes over time even when given the same direction over and over again.

Here's how we use `statefulMap`:

    function RelativeIterator (directionIterator) {
      return statefulMap(directionIterator, function (relativePositionStr, directionStr) {
        var delta = DIRECTION_TO_DELTA[directionStr],
            matchData = relativePositionStr.match(/(-?\d+) (-?\d+)/),
            relative0 = parseInt(matchData[1], 10),
            relative1 = parseInt(matchData[2], 10);
        return "" + (relative0 + delta[0]) + " " + (relative1 + delta[1]);
      }, "0 0");
    };

    var i = RelativeIterator(new Game().iterator());
    i();
      //=> '-1 0'
    i();
      //=> '-1 -1'
    i();
      //=> '-2 -1'
    i();
      //=> '-2 0'
    i();
      //=> '-2 1'
    i();
      //=> '-3 1'
    i();
      //=> '-3 2'
    i();
      //=> '-3 3'
{: .language-javascript}

We're almost there! The refactored `tortoiseAndHareLoopDetector` expects an "iterable," an object that implements the  `.iterate()` method. Let's refactor `RelativeIterable` to accept a game and return an iterable instead of accepting an iteration and returning an iteration:

    function RelativeIterable (game) {
      return {
        iterator: function () {
            return statefulMap(game.iterator(), function (relativePositionStr, directionStr) {
              var delta = DIRECTION_TO_DELTA[directionStr],
                  matchData = relativePositionStr.match(/(-?\d+) (-?\d+)/),
                  relative0 = parseInt(matchData[1], 10),
                  relative1 = parseInt(matchData[2], 10);
              return "" + (relative0 + delta[0]) + " " + (relative1 + delta[1]);
            }, "0 0");
        }
      };
    };

    var i = RelativeIterable(new Game()).iterator();
    i();
      //=> '0 -1'
    i();
      //=> '1 -1'
    i();
      //=> '1 0'
    i();
      //=> '2 0'
    i();
      //=> undefined
{: .language-javascript}

### the solution

So. We can take a `Game` instance and produce an iterable that iterates over regular strings representing relative positions. If it terminates on its own, the game obviously terminates. And if it repeats itself, the game does not terminate.

Our refactored `tortoiseAndHareLoopDetector` takes an iterable and detects this for us. Writing a detector function is trivial:

    function terminates (game) {
      return !tortoiseAndHareLoopDetector(RelativeIterable(game));
    }

    terminates(new Game(4));
      //=> false
    terminates(new Game(4));
      //=> true
    terminates(new Game(4));
      //=> false
    terminates(new Game(4));
      //=> false
{: .language-javascript}

### preliminary conclusion

Untangling the mechanism of following a linked list from the algorithm of searching for a loop allows us to repurpose the Tortoise and Hare algorithm to solve a question about a path looping.

### no-charge extra conclusion

Can we also refactor the "Teleporting Turtle" algorithm to take an iterable? If so, we should be able to swap algorithms for our game termination detection without rewriting everything in sight. Let's try it:

We start with:

    function teleportingTurtleLoopDetector (list) {
      var i, rabbit, speed, turtle;
      speed = 1;
      turtle = rabbit = list;
      while (true) {
        for (i = 0; i <= speed; i += 1) {
          rabbit = rabbit.next;
          if (rabbit == null) {
            return false;
          }
          if (rabbit === turtle) {
            return true;
          }
        }
        turtle = rabbit;
        speed *= 2;
      }
      return false;
    };

    list = new LinkedList(5).appendTo(4).appendTo(3).appendTo(2).appendTo(1);

    teleportingTurtleLoopDetector(list);
      //=> false

    list.tailNode().next = list.next;

    teleportingTurtleLoopDetector(list);
      //=> true
{: .language-javascript}

And refactor it to become:

  function teleportingTurtleLoopDetector (iterable) {
    var i, rabbit, rabbitValue, speed, turtleValue;
    speed = 1;
    rabbit = iterable.iterator();
    turtleValue = rabbitValue = rabbit();
    while (true) {
      for (i = 0; i <= speed; i += 1) {
        rabbitValue = rabbit();
        if (rabbitValue == null) {
          return false;
        }
        if (rabbitValue === turtleValue) {
          return true;
        }
      }
      turtleValue = rabbitValue;
      speed *= 2;
    }
    return false;
  };

  list = new LinkedList(5).appendTo(4).appendTo(3).appendTo(2).appendTo(1);

  teleportingTurtleLoopDetector(list);
    //=> false

  list.tailNode().next = list.next;

  teleportingTurtleLoopDetector(list);
    //=> true
{: .language-javascript}

Now we can plug it into our termination detector:

    function terminates (game) {
      return !teleportingTurtleLoopDetector(RelativeIterable(game));
    }

    terminates(new Game(4));
      //=> false
    terminates(new Game(4));
      //=> false
    terminates(new Game(4));
      //=> false
    terminates(new Game(4));
      //=> false
    terminates(new Game(4));
      //=> true
{: .language-javascript}

Refactoring an algorithm to work with iterators allows us to use the same algorithm to solve different problems and to swap algorithms for the same problem. This is natural, we have created an abstraction that allows us to plug different items into either side of of its interface.

## Trampolining {#trampolining}

> A trampoline is a loop that iteratively invokes [thunk]-returning functions ([continuation-passing style][cps]). A single trampoline is sufficient to express all control transfers of a program; a program so expressed is trampolined, or in trampolined style; converting a program to trampolined style is trampolining. Trampolined functions can be used to implement [tail-recursive] function calls in stack-oriented programming languages.--[Wikipedia][trampolining]

[thunk]: https://en.wikipedia.org/wiki/Thunk_(functional_programming)
[cps]: https://en.wikipedia.org/wiki/Continuation-passing_style
[tail-recursive]: https://en.wikipedia.org/wiki/Tail-recursive_function
[trampolining]: https://en.wikipedia.org/wiki/Trampoline_(computing)

This description is exactly how one ought to answer the question "define trampolining" on an examination, because it demonstrates that you've learned the subject thoroughly. But if asked to *explain* trampolining, a more tutorial-focused approach is called for.

Let's begin with a use case.

### recursion, see recursion

Consider implementing `factorial` in recursive style:

    function factorial (n) {
      return n
      ? n * factorial(n - 1)
      : 1
    }
{: .language-javascript}

The immediate limitation of this implementation is that since it calls itself *n* times, to get a result you need a stack on *n* stack frames in a typical stack-based programming language implementation. And JavaScript is such an implementation.

This creates two problems: First, we need space O*n* for all those stack frames. It's as if we actually wrote out `1 x 1 x 2 x 3 x 4 x ...` before doing any calculations. Second, most languages have a limit on the size of the stack much smaller than the limit on the amount of memory you need for data.

For example:

    factorial(10)
      //=> 3628800
    factorial(32768)
      //=> RangeError: Maximum call stack size exceeded
{: .language-javascript}

We can easily rewrite this in iterative style, but there are other functions that aren't so amenable to rewriting and using a simple example allows us to concentrate on the mechanism rather than the "domain."

### tail-call elimination

Lisp programmers in days of yore would rewrite functions like this into "Tail Recursive Form," and that made it possible for their compilers to perform [Tail-Call Optimization][tco]. Meaning, that when a function returns the result of calling itself, the language doesn't actually perform another function call, it turns the whole thing into a loop for you.

[tco]: https://en.wikipedia.org/wiki/Tail-call_optimization

What we need to do is take the expression `n * factorial(n - 1)` and push it down into a function so we can just call it with parameters. When a function is called, a *stack frame* is created that contains all the information needed to resume execution with the result. Stack frames hold a kind of pointer to where to carry on evaluating, the function parameters, and other bookkeeping information.[^bookkeeping]

[^bookkeeping]: Did you know that "bookkeeping" is the only word in the English language containing three consecutive letter pairs? You're welcome.

If we use the symbol `_` to represent a kind of "hole" in an expression where we plan to put the result, every time `factorial` calls itself, it needs to remember `n * _` so that when it gets a result back, it can multiply it by `n` and return that. So the first time it calls itself, it remembers `10 * _`, the second time it calls itself, it remembers `9 * _`, and all these things stack up like this when we call `factorial(10)`:

     1 * _
     2 * _
     3 * _
     4 * _
     5 * _
     6 * _
     7 * _
     8 * _
     9 * _
    10 * _
{: .language-javascript}

Finally, we call `factorial(0)` and it returns `1`. Then the top is popped off the stack, so we calculate `1 * 1`. It returns `1` again and we calculate `2 * 1`. That returns `2` and we calculate `3 * 2` and so on up the stack until we return `10 * 362880` and return `3628800`, which we print.

How can we get around this? Well, imagine if we don't have a hole in a computation to return. In that case, we wouldn't need to "remember" anything on the stack. To make this happen, we need to either return a value or return the result of calling another function without any further computation.

Such a call is said to be in "tail position" and to be a "tail call." The "elimination" of tail-call elimination means that we don't perform a full call including setting up a new stack frame. We perform the equivalent of a "jump." 

For example:

    function factorial (n) {
      var _factorial = function myself (acc, n) {
        return n
        ? myself(acc * n, n - 1)
        : acc
      };
      
      return _factorial(1, n);
    }
{: .language-javascript}

Now our function either returns a value or it returns the result of calling another function without doing anything with that result. This gives us the correct results, but we can see that current implementations of JavaScript don't perform this magic "tail-call elimination."

    factorial(10)
      //=> 3628800
    factorial(32768)
      //=> RangeError: Maximum call stack size exceeded
{: .language-javascript}

So we'll do it ourselves.

### trampolining

One way to implement tail-call elimination is also handy for many other general things we might want to do with control flow, it's called *trampolining*. What we do is this:

When we call a function, it returns a *thunk* that we call to get a result. Of course, the thunk can return another thunk, so every time we get a result, we check to see if it's a thunk. If not, we have our final result.

A *thunk* is a function taking no arguments that delays evaluating an expression. For example, this is a thunk: `function () { return 'Hello World'; }`.

An extremely simple and useful implementation of trampolining can be found in the [Lemonad] library. It works provided that you want to trampoline a function that doesn't return a function. Here it is: 

[Lemonad]: http://fogus.github.com/lemonad/

    L.trampoline = function(fun /*, args */) {
      var result = fun.apply(fun, _.rest(arguments));

      while (_.isFunction(result)) {
        result = result();
      }

      return result;
    };
{: .language-javascript}

We'll rewrite it in combinatorial style for consistency and composeability:

    var trampoline = function (fn) {
      return variadic( function (args) {
        var result = fn.apply(this, args);

        while (result instanceof Function) {
          result = result();
        }

        return result;
      });
    };
{: .language-javascript}

Now here's our implementation of `factorial` that is wrapped around a trampolined tail recursive function:

    function factorial (n) {
      var _factorial = trampoline( function myself (acc, n) {
        return n
        ? function () { return myself(acc * n, n - 1); }
        : acc
      });
      
      return _factorial(1, n);
    }

    factorial(10);
      //=> 362800
    factorial(32768);
      //=> Infinity
{: .language-javascript}

Presto, it runs for `n = 32768`. Sadly, JavaScript's built-in support for integers cannot keep up, so we'd better fix the "infinity" problem with a "big integer" library:[^big]

[^big]: The use of a third-party big integer library is not essential to understand trampolining.

    npm install big-integer

    var variadic = require('allong.es').variadic,
        bigInt = require("big-integer");

    var trampoline = function (fn) {
      return variadic( function (args) {
        var result = fn.apply(this, args);

        while (result instanceof Function) {
          result = result();
        }

        return result;
        
      });
    };

    function factorial (n) {
      var _factorial = trampoline( function myself (acc, n) {
        return n.greater(0)
        ? function () { return myself(acc.times(n), n.minus(1)); }
        : acc
      });
      
      return _factorial(bigInt.one, bigInt(n));
    }

    factorial(10).toString()
      //=> '3628800'
    factorial(32768)
      //=> GO FOR LUNCH
{: .language-javascript}

Well, it now takes a very long time to run, but it is going to get us the proper result and we can print that as a string, so we'll leave it calculating in another process and carry on.

The limitation of the implementation shown here is that because it tests for the function returning a function, it will not work for functions that return functions. If you want to trampoline a function that returns a function, you will need a more sophisticated mechanism, but the basic principle will be the same: The function will return a thunk instead of a value, and the trampolining loop will test the returned thunk to see if it represents a value or another computation to be evaluated.

### trampolining co-recursive functions

If trampolining was only for recursive functions, it would have extremely limited value: All such functions can be re-written iteratively and will be much faster (although possibly less elegant). However, trampolining eliminates all calls in tail position, including calls to other functions.

Consider this delightfully simple example of two co-recursive functions:

    function even (n) {
      return n == 0
        ? true
        : odd(n - 1);
    };

    function odd (n) {
      return n == 0
        ? false
        : even(n - 1);
    };
{: .language-javascript}

Like our `factorial`, it consumes *n* stack space of alternating calls to `even` and `odd`:

    even(32768);
      //=> RangeError: Maximum call stack size exceeded
{: .language-javascript}

Obviously we can solve this problem with modulo arithmetic, but consider that what this shows is a pair of functions that call other functions in tail position, not just themselves. As with factorial, we separate the public interface that is not trampolined from the trampolined implementation:

    var even = trampoline(_even),
        odd  = trampoline(_odd);

    function _even (n) {
      return n == 0
        ? true
        : function () { return _odd(n - 1); };
    };

    function _odd (n) {
      return n == 0
        ? false
        : function () { return _even(n - 1); };
    };
{: .language-javascript}

And presto:

    even(32768);
      //=> true
{: .language-javascript}

Trampolining works with co-recursive functions, or indeed any function that can be rewritten in tail-call form.

### summary

*Trampolining* is a technique for implementing tail-call elimination. Meaning, if you take a function (whether recursive, co-recursive, or any other form) and rewrite it in tail-call form, you can eliminate the need to create a stack frame for every 'invocation'.

Trampolining is very handy in a language like JavaScript, in that it allows you to use a recursive style for functions without worrying about limitations on stack sizes.
