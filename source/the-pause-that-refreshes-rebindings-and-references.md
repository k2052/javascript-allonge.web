# The Pause That Refreshes: Rebinding and References {#references}

![It is not enough that coffee taste beautiful. Everything about its creation and consumption should reflect coffee's beauty.](bezzera.jpg)

### a simple question

Consider this code:

    var x = 'June 14, 1962',
        y = x;
        
    x === y
      //=> true
{: .language-javascript}

This makes obvious sense, because we know that strings are a value type, so no matter what expression you use to derive the value 'June 14, 1962', you are going to get a string with the exact same identity.

But what about this code?

    var x = [2012, 6, 14],
        y = x;
        
    x === y
      //=> true
{: .language-javascript}

Also true, even though we know that every time we evaluate an expression such as `[2012, 6, 14]`, we get a new array with a new identity. So what is happening in our environments?

### arguments and references

In our discussion of [closures](#closures), we said that environments bind values (like `[2012, 6, 14]`) to names (like `x` and `y`), and that when we use these names as expressions, the name evaluates as the value.

What this means is that when we write something like `y = x`, the name `x` is looked up in the current environment, and its value is a specific array that was created when the expression `[2012, 6, 14]` was first evaluated. We then bind *that exact same value* to the name `y` in a new environment, and thus `x` and `y` are both bound to the exact same value, which is identical to itself.

The same thing happens with binding a variable through a more conventional means of applying a function to arguments:

    var x = [2012, 6, 14];
    
    (function (y) {
      return x === y
    })(x)
      //=> true
{: .language-javascript}

`x` and `y` both end up bound to the exact same array, not two different arrays that look the same to our eyes.

## Arguments and Arrays {#arrays}

JavaScript provides two different kinds of containers for values. We've met one already, the array. Let's see how it treats values and identities. For starters, we'll learn how to extract a value from an array. We'll start with a function that makes a new value with a unique identity every time we call it. We already know that every function we create is unique, so that's what we'll use:

    var unique = function () {
                    return function () {}
                  };
    
      unique()
        //=> [Function]
        
      unique() === unique()
        //=> false
{: .language-javascript}

Let's verify that what we said about references applies to functions as well as arrays:

      var x = unique(),
          y = x;
          
      x === y
        //=> true
{: .language-javascript}

Ok. So what about things *inside* arrays? We know how to create an array with something inside it:

      [ unique() ]
        //=> [ [Function] ]
{: .language-javascript}

That's an array with one of our unique functions in it. How do we get something *out* of it?

      var a = [ 'hello' ];
      
      a[0]
        //=> 'hello'
{: .language-javascript}

Cool, arrays work a lot like arrays in other languages and are zero-based. The trouble with this example is that strings are value types in JavaScript, so we have no idea whether `a[0]` always gives us the same value back like looking up a name in an environment, or whether it does some magic that tries to give us a new value.

We need to put a reference type into an array. If we get the same thing back, we know that the array stores a reference to whatever you put into it. If you get something different back, you know that arrays store copies of things.[^hunh]

[^hunh]: Arrays in all contemporary languages store references and not copies, so we can be forgiven for expecting them to work the same way in JavaScript. Nevertheless, it's a useful exercise to test things for ourself.

Let's test it:

    var unique = function () {
                    return function () {}
                  },
        x = unique(),
        a = [ x ];
        
    a[0] === x
      //=> true
{: .language-javascript}

If we get a value out of an array using the `[]` suffix, it's the exact same value with the same identity. Question: Does that apply to other locations in the array? Yes:

    var unique = function () {
                   return function () {}
                 },
        x = unique(),
        y = unique(),
        z = unique(),
        a = [ x, y, z ];
        
    a[0] === x && a[1] === y && a[2] === z
      //=> true
{: .language-javascript}

## References and Objects {#objects}

JavaScript also provides objects. The word "object" is loaded in programming circles, due to the widespread use of the term "object-oriented programming" that was coined by Alan Kay but has since come to mean many, many things to many different people.

In JavaScript, an object[^pojo] is a map from names to values, a lot like an environment. The most common syntax for creating an object is simple:

    { year: 2012, month: 6, day: 14 }
{: .language-javascript}
    
Two objects created this way have differing identities, just like arrays:

    { year: 2012, month: 6, day: 14 } === { year: 2012, month: 6, day: 14 }
      //=> false
{: .language-javascript}
      
Objects use `[]` to access the values by name, using a string:

    { year: 2012, month: 6, day: 14 }['day']
      //=> 14
{: .language-javascript}

Values contained within an object work just like values contained within an array:

    var unique = function () {
                    return function () {}
                  },
        x = unique(),
        y = unique(),
        z = unique(),
        o = { a: x, b: y, c: z };
        
    o['a'] === x && o['b'] === y && o['c'] === z
      //=> true
{: .language-javascript}
      
Names needn't be alphanumeric strings. For anything else, enclose the label in quotes:

    { 'first name': 'reginald', 'last name': 'lewis' }['first name']
      //=> 'reginald'
{: .language-javascript}
      
If the name is an alphanumeric string conforming to the same rules as names of variables, there's a simplified syntax for accessing the values:

    { year: 2012, month: 6, day: 14 }['day'] ===
        { year: 2012, month: 6, day: 14 }.day
      //=> true
{: .language-javascript}
      
All containers can contain any value, including functions or other containers:

    var Mathematics = { 
      abs: function (a) {
             return a < 0 ? -a : a
           }
    };
            
    Mathematics.abs(-5)
      //=> 5
{: .language-javascript}
      
Funny we should mention `Mathematics`. If you recall, JavaScript provides a global environment that contains some existing object that have handy functions you can use. One of them is called `Math`, and it contains functions for `abs`, `max`, `min`, and many others. Since it is always available, you can use it in any environment provided you don't shadow `Math`.

    Math.abs(-5)
      //=> 5
{: .language-javascript}
      
[^pojo]: Tradition would have us call objects that don't contain any functions "POJOs," meaning Plain Old JavaScript Objects.

## Reassignment and Mutation {#reassignment}

Like most imperative programming languages, JavaScript allows you to re-assign the value of variables. The syntax is familiar to users of most popular languages:

    var age = 49;
    age = 50;
    age
      //=> 50
{: .language-javascript}

We took the time to carefully examine what happens with bindings in environments. Let's take the time to explore what happens with reassigning values to variables. The key is to understand that we are rebinding a different value to the same name in the same environment.

So let's consider what happens with a shadowed variable:

    (function () {
      var age = 49;
      (function () {
        var age = 50;
      })();
      return age;
    })()
      //=> 49
{: .language-javascript}

Binding `50` to age in the inner environment does not change `age` in the outer environment because the binding of `age` in the inner environment shadows the binding of `age` in the outer environment. We go from:

    {age: 49, '..': global-environment}
{: .language-javascript}
    
To:

    {age: 50, '..': {age: 49, '..': global-environment}}
{: .language-javascript}
    
Then back to:

    {age: 49, '..': global-environment}
{: .language-javascript}

{pagebreak}    
However, if we don't shadow `age` by explicitly using `var`, reassigning it in a nested environment changes the original:

    (function () {
      var age = 49;
      (function () {
        age = 50;
      })();
      return age;
    })()
      //=> 50
{: .language-javascript}

Like evaluating variable labels, when a binding is rebound, JavaScript searches for the binding in the current environment and then each ancestor in turn until it finds one. It then rebinds the name in that environment.

![Cupping Grinds](cupping.jpg)

### mutation and aliases

Now that we can reassign things, there's another important factor to consider: Some values can *mutate*. Their identities stay the same, but not their structure. Specifically, arrays and objects can mutate. Recall that you can access a value from within an array or an object using `[]`. You can reassign a value using `[]` as well:

    var oneTwoThree = [1, 2, 3];
    oneTwoThree[0] = 'one';
    oneTwoThree
      //=> [ 'one', 2, 3 ]
{: .language-javascript}

You can even add a value:

    var oneTwoThree = [1, 2, 3];
    oneTwoThree[3] = 'four';
    oneTwoThree
      //=> [ 1, 2, 3, 'four' ]
{: .language-javascript}

You can do the same thing with both syntaxes for accessing objects:

    var name = {firstName: 'Leonard', lastName: 'Braithwaite'};
    name.middleName = 'Austin'
    name
      //=> { firstName: 'Leonard',
      #     lastName: 'Braithwaite',
      #     middleName: 'Austin' }
{: .language-javascript}

We have established that JavaScript's semantics allow for two different bindings to refer to the same value. For example:

    var allHallowsEve = [2012, 10, 31]
    var halloween = allHallowsEve;  
{: .language-javascript}
      
Both `halloween` and `allHallowsEve` are bound to the same array value within the local environment. And also:

    var allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      // ...
    })(allHallowsEve);
{: .language-javascript}

There are two nested environments, and each one binds a name to the exact same array value. In each of these examples, we have created two *aliases* for the same value. Before we could reassign things, the most important point about this is that the identities were the same, because they were the same value.

This is vital. Consider what we already know about shadowing:

    var allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      halloween = [2013, 10, 31];
    })(allHallowsEve);
    allHallowsEve
      //=> [2012, 10, 31]
{: .language-javascript}
      
The outer value of `allHallowsEve` was not changed because all we did was rebind the name `halloween` within the inner environment. However, what happens if we *mutate* the value in the inner environment?

    var allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      halloween[0] = 2013;
    })(allHallowsEve);
    allHallowsEve
      //=> [2013, 10, 31]
{: .language-javascript}
      
This is different. We haven't rebound the inner name to a different variable, we've mutated the value that both bindings share. Now that we've finished with mutation and aliases, let's have a look at it.
      
T> JavaScript permits the reassignment of new values to existing bindings, as well as the reassignment and assignment of new values to elements of containers such as arrays and objects. Mutating existing objects has special implications when two bindings are aliases of the same value.

## Reassignment and Mutation {#reassignment}

Like most imperative programming languages, JavaScript allows you to re-assign the value of variables. The syntax is familiar to users of most popular languages:

    var age = 49;
    age = 50;
    age
      //=> 50
{: .language-javascript}

We took the time to carefully examine what happens with bindings in environments. Let's take the time to explore what happens with reassigning values to variables. The key is to understand that we are rebinding a different value to the same name in the same environment.

So let's consider what happens with a shadowed variable:

    (function () {
      var age = 49;
      (function () {
        var age = 50;
      })();
      return age;
    })()
      //=> 49
{: .language-javascript}

Binding `50` to age in the inner environment does not change `age` in the outer environment because the binding of `age` in the inner environment shadows the binding of `age` in the outer environment. We go from:

    {age: 49, '..': global-environment}
{: .language-javascript}
    
To:

    {age: 50, '..': {age: 49, '..': global-environment}}
{: .language-javascript}
    
Then back to:

    {age: 49, '..': global-environment}
{: .language-javascript}

{pagebreak}    
However, if we don't shadow `age` by explicitly using `var`, reassigning it in a nested environment changes the original:

    (function () {
      var age = 49;
      (function () {
        age = 50;
      })();
      return age;
    })()
      //=> 50
{: .language-javascript}

Like evaluating variable labels, when a binding is rebound, JavaScript searches for the binding in the current environment and then each ancestor in turn until it finds one. It then rebinds the name in that environment.

![Cupping Grinds](cupping.jpg)

### mutation and aliases

Now that we can reassign things, there's another important factor to consider: Some values can *mutate*. Their identities stay the same, but not their structure. Specifically, arrays and objects can mutate. Recall that you can access a value from within an array or an object using `[]`. You can reassign a value using `[]` as well:

    var oneTwoThree = [1, 2, 3];
    oneTwoThree[0] = 'one';
    oneTwoThree
      //=> [ 'one', 2, 3 ]
{: .language-javascript}

You can even add a value:

    var oneTwoThree = [1, 2, 3];
    oneTwoThree[3] = 'four';
    oneTwoThree
      //=> [ 1, 2, 3, 'four' ]
{: .language-javascript}

You can do the same thing with both syntaxes for accessing objects:

    var name = {firstName: 'Leonard', lastName: 'Braithwaite'};
    name.middleName = 'Austin'
    name
      //=> { firstName: 'Leonard',
      #     lastName: 'Braithwaite',
      #     middleName: 'Austin' }
{: .language-javascript}

We have established that JavaScript's semantics allow for two different bindings to refer to the same value. For example:

    var allHallowsEve = [2012, 10, 31]
    var halloween = allHallowsEve;  
{: .language-javascript}
      
Both `halloween` and `allHallowsEve` are bound to the same array value within the local environment. And also:

    var allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      // ...
    })(allHallowsEve);
{: .language-javascript}

There are two nested environments, and each one binds a name to the exact same array value. In each of these examples, we have created two *aliases* for the same value. Before we could reassign things, the most important point about this is that the identities were the same, because they were the same value.

This is vital. Consider what we already know about shadowing:

    var allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      halloween = [2013, 10, 31];
    })(allHallowsEve);
    allHallowsEve
      //=> [2012, 10, 31]
{: .language-javascript}
      
The outer value of `allHallowsEve` was not changed because all we did was rebind the name `halloween` within the inner environment. However, what happens if we *mutate* the value in the inner environment?

    var allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      halloween[0] = 2013;
    })(allHallowsEve);
    allHallowsEve
      //=> [2013, 10, 31]
{: .language-javascript}
      
This is different. We haven't rebound the inner name to a different variable, we've mutated the value that both bindings share. Now that we've finished with mutation and aliases, let's have a look at it.
      
T> JavaScript permits the reassignment of new values to existing bindings, as well as the reassignment and assignment of new values to elements of containers such as arrays and objects. Mutating existing objects has special implications when two bindings are aliases of the same value.

## How to Shoot Yourself in the Foot With Var

As we've seen, JavaScript's environments and bindings are quite powerful: You can bind and rebind names using function arguments or using variables declared with `var`. The takeaway is that when used properly, Javascript's `var` keyword is a great tool.

When used properly.

Let's look at a few ways to use it *improperly*.

### loose use

JavaScript's `var` keyword is scoped to the function enclosing it. This makes sense, because bindings are made in environments, and the environments are associated with function calls. So if you write:

    function foo (bar) {
      var baz = bar * 2;
      
      if (bar > 1) {
        var blitz = baz - 100;
        
        // ...
      }
    }
{: .language-javascript}
    
The name `blitz` is actually scoped to the function `foo`, not to the block of code in the consequent of an `if` statement. There are roughly two schools of thought. One line of reasoning goes like this: Since `bar` is scoped to the function `foo`, you should write the code like this:

    function foo (bar) {
      var baz = bar * 2,
          blitz;
      
      if (bar > 1) {
        blitz = baz - 100;
        
        // ...
      }
    }
{: .language-javascript}

We've separated the "declaration" from the "assignment," and we've made it clear that `blitz` is scoped to the entire function. The other school of thought is that programmers are responsible for understanding how the tools work, and even if you write it the first way, other programmers reading the code ought to know how it works.

So here's a question: Are both ways of writing the code equivalent? Let's set up a test case that would tell them apart. We'll try some aliasing:

    var questionable = 'outer';
    
    (function () {
      alert(questionable);
      
      if (true) {
        var questionable = 'inner';
        alert(questionable)
      }
    })()
{: .language-javascript}
    
What will this code do if we type it into a browser console? One theory is that it will alert `outer` and then `inner`, because when it evaluates the first alert, `questionable` hasn't been bound in the function's environment yet, so it will be looked up in the enclosing environment. Then an alias is bound, shadowing the outer binding, and it will alert `inner`.

This theory is wrong! It actually alerts `undefined` and then `inner`. Even though we wrote the `var` statement later in the code, JavaScript acts as if we'd declared it at the top of the function. This is true even if we never execute the `var` statement:

    var questionable = 'outer';
    
    (function () {
      return questionable;
      
      var questionable = 'inner'
    })()
    
      //=> undefined
{: .language-javascript}
      
So yes, both ways of writing the code work the same way, but only one represents the way it works directly and obviously. For this reason, we put the `var` declarations at the top of every function, always.

### for pete's sake

JavaScript provides a `for` loop for your iterating pleasure and convenience. It looks a lot like the `for` loop in C:

    var sum = 0;
    for (var i = 1; i <= 100; i++) {
      sum = sum + i
    }
    sum
      #=> 5050
{: .language-javascript}

Hopefully, you can think of a faster way to calculate this sum.[^gauss] And perhaps you have noticed that `var i = 1` is tucked away instead of being at the top as we prefer. But is this ever a problem?

[^gauss]: There is a well known story about Karl Friedrich Gauss when he was in elementary school. His teacher got mad at the class and told them to add the numbers 1 to 100 and give him the answer by the end of the class. About 30 seconds later Gauss gave him the answer. The other kids were adding the numbers like this: `1 + 2 + 3 + . . . . + 99 + 100 = ?` But Gauss rearranged the numbers to add them like this: `(1 + 100) + (2 + 99) + (3 + 98) + . . . . + (50 + 51) = ?` If you notice every pair of numbers adds up to 101. There are 50 pairs of numbers, so the answer is 50*101 = 5050. Of course Gauss came up with the answer about 20 times faster than the other kids.

Yes. Consider this variation:

    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];
      
    for (var i = 0; i < 3; i++) {
      introductions[i] = "Hello, my name is " + names[i]
    }
    introductions
      //=> [ 'Hello, my name is Karl',
      //     'Hello, my name is Friedrich',
      //     'Hello, my name is Gauss' ]
{: .language-javascript}

So far, so good. Hey, remember that functions in JavaScript are values? Let's get fancy!

    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];
      
    for (var i = 0; i < 3; i++) {
      introductions[i] = function (soAndSo) {
        return "Hello, " + soAndSo + ", my name is " + names[i]
      }
    }
    introductions
      //=> [ [Function],
      //     [Function],
      //     [Function] ]
{: .language-javascript}
    
So far, so good. Let's try one of our functions:

    introductions[1]('Raganwald')
      //=> 'Hello, Raganwald, my name is undefined'
{: .language-javascript}
  
What went wrong? Why didn't it give us 'Hello, Raganwald, my name is Friedrich'? The answer is that pesky `var i`. Remember that `i` is bound in the surrounding environment, so it's as if we wrote:

    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'],
        i;
      
    for (i = 0; i < 3; i++) {
      introductions[i] = function (soAndSo) {
        return "Hello, " + soAndSo + ", my name is " + names[i]
      }
    }
    introductions
{: .language-javascript}
  
Now, at the time we created each function, `i` had a sensible value, like `0`, `1`, or `2`. But at the time we *call* one of the functions, `i` has the value `3`, which is why the loop terminated. So when the function is called, JavaScript looks `i` up in its enclosing environment (its  closure, obviously), and gets the value `3`. That's not what we want at all. 

Here's how to fix it, once again with `let` as our guide:

    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];
      
    for (var i = 0; i < 3; i++) {
      (function (i) {
        introductions[i] = function (soAndSo) {
          return "Hello, " + soAndSo + ", my name is " + names[i]
        }
      })(i)
    }
    introductions[1]('Raganwald')
      //=> 'Hello, Raganwald, my name is Friedrich'
{: .language-javascript}
    
That works. What did we do? Well, we created a new function and called it immediately, and we deliberately shadowed `i` by passing it as an argument to our function, which had an argument of exactly the same name. If you dislike shadowing, this alternative also works:

    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];
      
    for (var i = 0; i < 3; i++) {
      (function () {
        var ii = i;
        introductions[ii] = function (soAndSo) {
          return "Hello, " + soAndSo + ", my name is " + names[ii]
        }
      })()
    }
    introductions[1]('Raganwald')
      //=> 'Hello, Raganwald, my name is Friedrich'
{: .language-javascript}
    
Now we're creating a new inner variable, `ii` and binding it  to the value of `i`. The shadowing code seems simpler and less error-prone to us, but both work.

### nope, nope, nope, nope, nope

The final caution about `var` concerns what happens if you omit to declare a variable with var, boldly writing something like:

    fizzBuzz = function () {
      // lots of interesting code elided
      // for the sake of hiring managers
    }
{: .language-javascript}
    
So where is the name `fizzBuzz` bound? The answer is that if there is no enclosing `var` declaration for `fizzBuzz`, the name is bound in the *global* environment. And by global, we mean global. It is visible to every separate compilation unit. All of your npm modules. Every JavaScript snippet in a web page. Every included file.

This is almost never what you want. And when you do want it, JavaScript provides alternatives such as binding to `window.fizzBuzz` in a browser, or `this.fizzBuzz` in node. In short, eschew undeclared variables. Force yourself to make a habit of using `var` all of the time, and explicitly binding variables to the `window` or `this` objects when you truly want global visibility.

## When Rebinding Meets Recursion {#recursive}

We've talked about binding values in environments, and now we're talking about rebinding values and mutating values. Let's take a small digression. As we've seen, in JavaScript functions are values. So you can bind a function just like binding a string, number or array. Here's a function that tells us whether a (small and positive) number is even:

    var even = function (num) {
      return (num === 0) || !(even(num - 1))
    }
    
    even(0)
      //=> true
      
    even(1)
      //=> false
      
    even(42)
      //=> true
{: .language-javascript}
    
You can alias a function value:

    var divisibleByTwo = even;
    
    divisibleByTwo(0)
      //=> true
      
    divisibleByTwo(1)
      //=> false
      
    divisibleByTwo(42)
      //=> true
{: .language-javascript}
      
What happens when we redefine a recursive function live `even`? Does `dividibleByTwo` still work? Let's try aliasing it and reassigning it:

    even = void 0;
    
    divisibleByTwo(0)
      //=> true
    
    divisibleByTwo(1)
      //=> TypeError
{: .language-javascript}
      
What happened? Well, our new `divisibleByTwo` function wasn't really a self-contained value. When we looked at functions, we talked about "pure" functions that only access their arguments and we looked at "closures" that have free variables. Recursive functions defined like this are closures, not pure functions, because when they "call themselves," what they actually do is look themselves up by name in their enclosing environment. Thus, they depend upon a specific value (themselves) being bound in their enclosing environment. Reassign to that variable (or rebind the name, same thing), and you break their functionality.

### named function expressions

You recall that in [Naming Functions](#named-function-expressions), we saw that when you create a named function expression, you bind the name of the function within its body but not the environment of the function expression, meaning you can write:

    var even = function myself (num) {
      return (num === 0) || !(myself(num - 1))
    }

    var divisibleByTwo = even;
    even = void 0;
    
    divisibleByTwo(0)
      //=> true
      
    divisibleByTwo(1)
      //=> false
      
    divisibleByTwo(42)
      //=> true
{: .language-javascript}

This is different, because the function doesn't refer to a name bound in its enclosing environment, it refers to a name bound in its own body. It is now a pure function. In fact, you can even bind it to the exact same name in its enclosing environment and it will still work:

    var even = function even (num) {
      return (num === 0) || !(even(num - 1))
    }

    var divisibleByTwo = even;
    even = void 0;
    
    divisibleByTwo(0)
      //=> true
      
    divisibleByTwo(1)
      //=> false
      
    divisibleByTwo(42)
      //=> true
{: .language-javascript}
      
The `even` inside the function refers to the name bound within the function by the named function expression. It may have the same name as the `even` bound in the enclosing environment, but they are two different bindings in two different environments. Thus, rebinding the name in the enclosing environment does not break the function.

You may ask, what if we rebind `even`  inside of itself. Now will it break?

    var even = function even (num) {
      even = void 0;
      return (num === 0) || !(even(num - 1))
    }

    var divisibleByTwo = even;
    even = void 0;
    
    divisibleByTwo(0)
      //=> true
      
    divisibleByTwo(1)
      //=> false
      
    divisibleByTwo(42)
      //=> true
{: .language-javascript}

Strangely, *no it doesn't*. The name bound by a named function expression is read-only. Why do we say strangely? Because other quasi-declarations like function declarations do *not* behave like this.

So, when we want to make a recursive function, the safest practice is to use a named function expression.

### limits

Named function expressions have limits. Here's one such limit: You can do simple recursion, but not mutual recursion. For example:

    var even = function (num) even { return (num === 0) || odd( num - 1) };
    var odd  = function (num) odd  { return (num  >  0) && even(num - 1) };
    
    odd = 'unusual';

    even(0)
      //=> true
    
    even(1)
      //=> TypeError
{: .language-javascript}

Using named function expressions doesn't help us, because `even` and `odd` need to be bound in an environment accessible to each other, not just to themselves. You either have to avoid rebinding the names of these functions, or use a closure to build a [module](#modules):

    var operations = (function () {})(
          var even = function (num) { return (num === 0) || odd( num - 1) };
          var odd  = function (num) { return (num  >  0) && even(num - 1) };
          return {
            even: even,
            odd:  odd
          }
        ),
        even = operations.even,
        odd = operations.odd;
{: .language-javascript}
       
Now you can rebind one without breaking the other, because the names outside of the closure have no effect on the bindings inside the closure:

    odd = 'unusual;
    
    even(0)
      //=> true
      
    even(1)
      //=> false
      
    even(42)
      //=> true
{: .language-javascript}      

T> As has often been noted, refactoring *to* a pattern is more important than designing *with* a pattern. So don't rush off to write all your recursive functions this way, but familiarize yourself with the technique so that if and when you run into a subtle bug, you can recognize the problem and know how to fix it.

## From Let to Modules {#modules}

### transient let

In the section on [let and Var](#let), we learned that we can create a new environment any time we want by combining a function definition with a function invocation, to whit:

    (function () {
      //
    })();
{: .language-javascript}      

Because this function is invoked, if it contains a return statement, it evaluates to a value of some kind. So you can, for example, write something like:

    var factorialOfTwentyFive = (function () {
      var factorial = function (num) {
        if (num  <  2 ) {
          return 1
        }
        else return num * factorial (num - 1)
      }
      return factorial(25)
    })();
{: .language-javascript}      

This could have been written using a named function to avoid the need for a let, but as we'll see in the [memoize](#recipe) later, sometimes there's good reason to write it like this. In any event, our let serves to create a scope for the `factorial` function. Presumably we write it this way to signal that we do not want to use it elsewhere, so putting it inside of a let keeps it invisible from the rest of the code.

You'll note that once we've calculated the factorial of 25, we have no further need for the environment of the function, so it will   be thrown away. This is what we might call a **transient let**: Nothing we bind in the `let` is returned from the `let` or otherwise passed out though assignment, so the environment of the let is discarded when the let finishes being evaluated.

### private closure

The transient let only uses its environment to generate the result, then it can be discarded. Another type of let is the **private closure**. This let returns a closure that references one or more bindings in the let's environment. For example:

    var counter = (function () {
      var value = 0;
      
      return function () {
        return value++
      }
    })();
    
    counter()
      //=> 0
      
    counter()
      //=> 1
      
    counter()
      //=> 2
{: .language-javascript}      

`counter` is bound to a function closure that references the binding `value` in the let's environment. So the environment isn't transient, it remains active until the function bound to the name `counter` is discarded. Private closures are often used to manage state as we see in the counter example, but they can also be used for helper functions.

For example, this date format function cribbed from somewhere or other has a helper function that isn't used anywhere else:

    function formatDate (time) {
      var date;

      if (time) {
        date = unformattedDate(time);
        // Have to massage the date because
        // we can't create a date 
        // based on GMT which the server gives us

        if (!(/-\d\d:\d\d/.test(time))) {
          date.setHours(
            date.getHours() - date.getTimezoneOffset()/60);
        }

        var diff = (
            (new Date()).getTime() - date.getTime()
          ) / 1000;
        day_diff = Math.floor(diff / 86400);

        if ( isNaN(day_diff) || day_diff < 0  )
          return;

        return '<span title="' + date.toUTCString() + '">' + (day_diff == 0 && (
            diff < 60 && "just now" ||
            diff < 120 && "1 minute ago" ||
            diff < 3600 && Math.floor( diff / 60 ) + " minutes ago" ||
            diff < 7200 && "1 hour ago" ||
            diff < 86400 && Math.floor( diff / 3600 ) + " hours ago") ||
          day_diff == 1 && "Yesterday" ||
          day_diff < 7 && day_diff + " days ago" ||
          day_diff < 31 && Math.ceil( day_diff / 7 ) + " weeks ago" ||
          (day_diff < 360 && day_diff >= 31) && Math.ceil(day_diff / 31) + 
            ' month' + (day_diff == 31 ? '' : 's') + ' ago' ||
            day_diff > 360 && Math.floor( day_diff / 360) + " years " + 
            Math.floor(day_diff%360/32) + " months ago") + '</span>';
      }
      else return '-'
      
      function unformattedDate (time) {
        return new Date((time || "").replace(/[-+]/g,"/").
          replace(/[TZ]/g," ").replace(/\/\d\d:\d\d/, ''));
      }
    }
{: .language-javascript}      
    
Every time we call `formatDate`, JavaScript will create an entirely new `unformattedDate` function. That is not necessary, since it's a pure function. In theory, a sufficiently smart interpreter would notice this and only create one function. In practice, we can rewrite it to use a private closure and only create one helper function:

    var formatDate = (function () {
      return function (time) {
        var date;

        if (time) {
          date = unformattedDate(time);
          // Have to massage the date because we can't create a date 
          // based on GMT which the server gives us

          if (!(/-\d\d:\d\d/.test(time))) {
            date.setHours(date.getHours() - date.getTimezoneOffset()/60);
          }

          var diff = ((new Date()).getTime() - date.getTime()) / 1000;
          day_diff = Math.floor(diff / 86400);

          if ( isNaN(day_diff) || day_diff < 0  )
            return;

          return '<span title="' + date.toUTCString() + '">' + (day_diff == 0 && (
              diff < 60 && "just now" ||
              diff < 120 && "1 minute ago" ||
              diff < 3600 && Math.floor( diff / 60 ) + " minutes ago" ||
              diff < 7200 && "1 hour ago" ||
              diff < 86400 && Math.floor( diff / 3600 ) + " hours ago") ||
            day_diff == 1 && "Yesterday" ||
            day_diff < 7 && day_diff + " days ago" ||
            day_diff < 31 && Math.ceil( day_diff / 7 ) + " weeks ago" ||
            (day_diff < 360 && day_diff >= 31) && Math.ceil(day_diff / 31) +
              ' month' + (day_diff == 31 ? '' : 's') + ' ago' ||
              day_diff > 360 && Math.floor( day_diff / 360) + 
              " years " + Math.floor(day_diff%360/32) + " months ago") + '</span>';
        }
        else return '-'
      }
      
      function unformattedDate (time) {
        return new Date((time || "").replace(/[-+]/g,"/").replace(/[TZ]/g," ").replace(/\/\d\d:\d\d/, ''));
      }
    })();
{: .language-javascript}      

The function `unformattedDate` is still private to `formatDate`, but now we no longer need to construct an entirely new function every time `formatDate` is called.

### modules

Once the power of creating a new environment with a let (or "immediately invoked function expression") is tasted, it won't be long before you find yourself building modules with lets. Modules are any collection of functions that have some private and some public-facing elements.

Consider a module designed to draw some bits on a virtual screen. The public API consists of a series of draw functions. The private API includes a series of helper functions. This is exactly like the **private closure**, the only difference being that we want to return multiple public functions instead of just one.

It looks like this:

    var DrawModule = (function () {
      
      return {
        drawLine: drawLine,
        drawRect: drawRect,
        drawCircle: drawCircle
      }
      
      // public methods
      var drawLine = function (screen, leftPoint, rightPoint) { ... }
      var drawRect = function (screen, topLeft, bottomRight) { ... }
      var drawCircle = function (screen, center, radius) { ... }
      
      // private helpers
      function bitBlt (screen, ...) { ... }
      function resize (screen, ...) { ... }
      
    })();
{: .language-javascript}      
    
You can then call the public functions using `DrawModule.drawCircle(...)`. The concept scales up to include the concept of state (such as setting default line styles), but when you look at it, it's really just the private closure let with a little more complexity in the form of returning an object with more than one function.

## Summary

T> ### Rebinding
T>
T> * JavaScript permits reassignment/rebinding of variables.
T> * Arrays and Objects are mutable.
T> * References permit aliasing of reference types.
T> * We may need to take special care to prevent ourselves from accidentally breaking recursive functions.
T> * For loops are convenient, but require care to avoid scoping bugs.
