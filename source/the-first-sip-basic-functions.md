![ ](carmack.png)

# The first sip: Basic Functions {#functions}

![The perfect Café Allongé begins with the right beans, properly roasted. JavaScript Allongé begins with functions, properly dissected.](leaf-roaster.jpg)

## As Little As Possible About Functions, But No Less

In JavaScript, functions are values, but they are also much more than simple numbers, strings, or even complex data structures like trees or maps. Functions represent computations to be performed. Like numbers, strings, and arrays, they have a representation. Let's start with the very simplest possible function. In JavaScript, it looks like this:

    function () {}
{: .language-javascript}

This is a function that is applied to no values and produces no value. How do we represent "no value" in JavaScript? We'll find out in a minute. First, let's verify that our function is a value:

    (function () {})
      //=> [Function]
{: .language-javascript}
      
What!? Why didn't it type back `function () {}` for us? This *seems* to break our rule that if an expression is also a value, JavaScript will give the same value back to us. What's going on? The simplest and easiest answer is that although the JavaScript interpreter does indeed return that value, displaying it on the screen is a slightly different matter. `[Function]` is a choice made by the people who wrote Node.js, the JavaScript environment that hosts the JavaScript REPL. If you try the same thing in a browser, you'll see the code you typed.

{pagebreak}

A> I'd prefer something else, but I must accept that what gets typed back to us on the screen is arbitrary, and all that really counts is that it is somewhat useful for a human to read. But we must understand that whether we see `[Function]` or `function () {}`, internally JavaScript has a full and proper function.

### functions and identities

You recall that we have two types of values with respect to identity: Value types and reference types. Value types share the same identity if they have the same contents.Reference types do not.

Which kind are functions? Let's try it. For reasons of appeasing the JavaScript parser, we'll enclose our functions in parentheses:

    (function () {}) === (function () {})
      //=> false
{: .language-javascript}
      
Like arrays, every time you evaluate an expression to produce a function, you get a new function that is not identical to any other function, even if you use the same expression to generate it. "Function" is a reference type.

### applying functions

Let's put functions to work. The way we use functions is to *apply* them to zero or more values called *arguments*. Just as `2 + 2` produces a value (in this case `4`), applying a function to zero or more arguments produces a value as well.

Here's how we apply a function to some values in JavaScript: Let's say that *fn_expr* is an expression that when evaluated, produces a function. Let's call the arguments *args*. Here's how to apply a function to some arguments:

    *fn_expr*`(`*args*`)`
    
Right now, we only know about one such expression: `function () {}`, so let's use it. We'll put it in parentheses[^ambiguous] to keep the parser happy, like we did above: `(function () {})`. Since we aren't giving it any arguments, we'll simply write `()` after the expression. So we write:

    (function () {})()
      //=> undefined
{: .language-javascript}

What is this `undefined`?

[^ambiguous]: If you're used to other programming languages, you've probably internalized the idea that sometimes parentheses are used to group operations in an expression like math, and sometimes to apply a function to arguments. If not... Welcome to the [ALGOL] family of programming languages!

[ALGOL]: https://en.wikipedia.org/wiki/ALGOL

### `undefined`

In JavaScript, the absence of a value is written `undefined`, and it means there is no value. It will crop up again. `undefined` is its own type of value, and it acts like a value type:

    undefined
      //=> undefined
{: .language-javascript}

Like numbers, booleans and strings, JavaScript can print out the value `undefined`.

    undefined === undefined
      //=> true
    (function () {})() === (function () {})()
      //=> true
    (function () {})() === undefined
      //=> true
{: .language-javascript}
      
No matter how you evaluate `undefined`, you get an identical value back. `undefined` is a value that means "I don't have a value." But it's still a value :-)
      
A> You might think that `undefined` in JavaScript is equivalent to `NULL` in SQL. No. In SQL, two things that are `NULL` are not equal to nor share the same identity, because two unknowns can't be equal. In JavaScript, every `undefined` is identical to every other `undefined`.

### void

We've seen that JavaScript represents an undefined value by typing `undefined`, and we've generated undefined values in two ways:

1. By evaluating a function that doesn't return a value `(function () {})()`, and;
2. By writing `undefined` ourselves.

There's a third way, with JavaScript's `void` operator. Behold:

    void 0
      //=> undefined
    void 1
      //=> undefined
    void (2 + 2)
      //=> undefined
{: .language-javascript}
      
`void` is an operator that takes any value and evaluates to `undefined`, always. So, when we deliberately want an undefined value, should we use the first, second, or third form?[^fourth] The answer is, use `void`. By convention, use `void 0`.

The first form works but it's cumbersome. The second form works most of the time, but it is possible to break it by reassigning `undefined` to a different value, something we'll discuss in [Reassignment and Mutation](#reassignment). The third form is guaranteed to always work, so that's what we will use.[^void]

[^fourth]: Experienced JavaScript programmers are aware that there's a fourth way, using a function argument. This was actually the preferred mechanism until `void` became commonplace.

[^void]: As an exercise for the reader, we suggest you ask your friendly neighbourhood programming language designer or human factors subject-matter expert to explain why a keyword called `void` is used to generate an `undefined` value, instead of calling them both `void` or both `undefined`. We have no idea.

### functions with no arguments and their bodies

Back to our function. We evaluated this:

    (function () {})()
      //=> undefined
{: .language-javascript}

Let's recall that we were applying the function `function () {}` to no arguments (because there was nothing inside of `()`). So how do we know to expect `undefined`? That's easy:

When we define a function[^todonamed], we write the word `function`. We then put a (possibly empty) list of arguments, then we give the function a *body* that is enclosed in braces `{...}`. Function bodies are (possibly empty) lists of JavaScript *statements* separated by semicolons.

Something like: { statement^1^; statement^2^; statement^3^; ... ; statement^n^ }

We haven't discussed these *statements*. What's a statement?

[^todonamed]: TODO: Named functions, probably discussed in a whole new section when we discuss `var` hoisting.

There are many kinds of JavaScript statements, but the first kind is one we've already met. An expression is a JavaScript statement. Although they aren't very practical, the following are all valid JavaScript functions, and they all evaluate to undefined when applied:

    (function () { 2 + 2 })
    
    (function () { 1 + 1; 2 + 2 })
{: .language-javascript}
    
You can also separate statements with line breaks.[^asi] The convention is to use some form of consistent indenting:

    (function () {
      1 + 1; 
      2 + 2 
    })
    
    (function () { 
      (function () { 
        (function () { 
          (function () {
          }) 
        }) 
      }); 
      (function () {
      }) 
    })
{: .language-javascript}
    
That last one's a doozy, but since a function body can contain a statement, and a statement can be an expression, and a function is an expression.... You get the idea.

[^asi]: Readers who follow internet flame-fests may be aware of something called [automatic semi-colon insertion](http://lucumr.pocoo.org/2011/2/6/automatic-semicolon-insertion/). Basically, there's a step where JavaScript looks at your code and follows some rules to guess where you meant to put semicolons in should you leave them out. This feature was originally created as a kind of helpful error-correction. Some programmers argue that since it's part of the language's definition, it's fair game to write code that exploits it, so they deliberately omit any semicolon that JavaScript will insert for them.

So how do we get a function to return a value when applied? With the `return` keyword and any expression:

    (function () { return 0 })()
      //=> 0
      
    (function () { return 1 })()
      //=> 1
      
    (function () { return 'Hello ' + 'World' })()
      // 'Hello World'
{: .language-javascript}
      
The `return` keyword creates a return statement that immediately terminates the function application and returns the result of evaluating its expression.

### functions that evaluate to functions

If an expression that evaluates to a function is, well, an expression, and if a return statement can have any expression on its right side... *Can we put an expression that evaluates to a function on the right side of a function expression?*

Yes:

    function () {
      return (function () {}) 
    }
{: .language-javascript}
    
That's a function! It's a function that when applied, evaluates to a function that when applied, evaluates to `undefined`.[^mouthful] Let's use a simpler terminology. Instead of saying "that when applied, evaluates to \_\_\_\_\_," we will say "gives \_\_\_\_\_." And instead of saying "gives undefined," we'll say "doesn't give anything."

So we have *a function, that gives a function, that doesn't give anything*. Likewise:

    function () { 
      return (function () { 
        return true 
      }) 
    }
{: .language-javascript}
    
That's a function, that gives a function, that gives `true`:

    (function () { 
      return (function () { 
        return true 
      }) 
    })()()
      //=> true
{: .language-javascript}
      
Well. We've been very clever, but so far this all seems very abstract. Diffraction of a crystal is beautiful and interesting in its own right, but you can't blame us for wanting to be shown a practical use for it, like being able to determine the composition of a star millions of light years away. So... In the next chapter, "[I'd Like to Have an Argument, Please](#fargs)," we'll see how to make functions practical.

[^mouthful]: What a mouthful! This is why other languages with a strong emphasis on functions come up with syntaxes like ` -> -> undefined`

## Ah. I'd Like to Have an Argument, Please.[^mp] {#fargs}

[^mp]: [The Argument Sketch](http://www.mindspring.com/~mfpatton/sketch.htm) from "Monty Python's Previous Record" and "Monty Python's Instant Record Collection"

Up to now, we've looked at functions without arguments. We haven't even said what an argument *is*, only that our functions don't have any.

A> Most programmers are perfectly familiar with arguments (often called "parameters"). Secondary school mathematics discusses this. So you know what they are, and I know that you know what they are, but please be patient with the explanation!

Let's make a function with an argument:

    function (room) {}
{: .language-javascript}
  
This function has one argument, `room`, and no body. Here's a function with two arguments and no body:

    function (room, board) {}
{: .language-javascript}
  
I'm sure you are perfectly comfortable with the idea that this function has two arguments, `room`, and `board`. What does one do with the arguments? Use them in the body, of course. What do you think this is?

    function (diameter) { return diameter * 3.14159265 }
{: .language-javascript}

It's a function for calculating the circumference of a circle given the diameter. I read that aloud as "When applied to a value representing the diameter, this function *returns* the diameter times 3.14159265."

Remember that to apply a function with no arguments, we wrote `(function () {})()`. To apply a function with an argument (or arguments), we put the argument (or arguments) within the parentheses, like this:

    (function (diameter) { return diameter * 3.14159265 })(2)
      //=> 6.2831853
{: .language-javascript}
      
You won't be surprised to see how to write and apply a function to two arguments:

    (function (room, board) { return room + board })(800, 150)
      //=> 950
{: .language-javascript}
      
T> ### a quick summary of functions and bodies
T>
T> How arguments are used in a body's expression is probably perfectly obvious to you from the examples, especially if you've used any programming language (except for the dialect of BASIC--which I recall from my secondary school--that didn't allow parameters when you called a procedure).
T>
T> Expressions consist either of representations of values (like `3.14159265`, `true`, and `undefined`), operators that combine expressions (like `3 + 2`), some special forms like `[1, 2, 3]` for creating arrays out of expressions, or `function (`*arguments*`) {`*body-statements*`}` for creating functions.
T>
T> One of the important possible statements is a return statement. A return statement accepts any valid JavaScript expression.
T>
T> This loose definition is recursive, so we can intuit (or use our experience with other languages) that since a function can contain a return statement with an expression, we can write a function that returns a function, or an array that contains another array expression. Or a function that returns an array, an array of functions, a function that returns an array of functions, and so forth:
T>
T> <<(code/f1.js)

### call by value {#call-by-value}

Like most contemporary programming languages, JavaScript uses the "call by value" [evaluation strategy]. That means that when you write some code that appears to apply a function to an expression or expressions, JavaScript evaluates all of those expressions and applies the functions to the resulting value(s).

[evaluation strategy]: http://en.wikipedia.org/wiki/Evaluation_strategy

So when you write:

    (function (diameter) { return diameter * 3.14159265 })(1 + 1)
      //=> 6.2831853
{: .language-javascript}

What happened internally is that the expression `1 + 1` was evaluated first, resulting in `2`. Then our circumference function was applied to `2`.[^f2f]

[^f2f]: We said that you can't apply a function to an expression. You *can* apply a function to one or more functions. Functions are values! This has interesting applications, and they will be explored much more thoroughly in [Functions That Are Applied to Functions](#consumers).

### variables and bindings

Right now everything looks simple and straightforward, and we can move on to talk about arguments in more detail. And we're going to work our way up from `function (diameter) { return diameter * 3.14159265 }` to functions like:

    function (x) { return (function (y) { return x }) }
{: .language-javascript}
    
A> `function (x) { return (function (y) { return x }) }` just looks crazy, as if we are learning English as a second language and the teacher promises us that soon we will be using words like *antidisestablishmentarianism*. Besides a desire to use long words to sound impressive, this is not going to seem attractive until we find ourselves wanting to discuss the role of the Church of England in 19th century British politics.
A>
A> But there's another reason for learning the word *antidisestablishmentarianism*: We might learn how prefixes and postfixes work in English grammar. It's the same thing with `function (x) { return (function (y) { return x }) }`. It has a certain important meaning in its own right, and it's also an excellent excuse to learn about functions that make functions, environments, variables, and more.
    
In order to talk about how this works, we should agree on a few terms (you may already know them, but let's check-in together and "synchronize our dictionaries"). The first `x`, the one in `function (x) ...`, is an *argument*. The `y` in `function (y) ...` is another argument. The second `x`, the one in `{ return x }`, is not an argument, *it's an expression referring to a variable*. Arguments and variables work the same way whether we're talking about `function (x) { return (function (y) { return x }) }`  or just plain `function (x) { return x }`.

Every time a function is invoked ("invoked" means "applied to zero or more arguments"), a new *environment* is created. An environment is a (possibly empty) dictionary that maps variables to values by name. The `x` in the expression that we call a "variable" is itself an expression that is evaluated by looking up the value in the environment.

How does the value get put in the environment? Well for arguments, that is very simple. When you apply the function to the arguments, an entry is placed in the dictionary for each argument. So when we write:

    (function (x) { return x })(2)
      //=> 2
{: .language-javascript}

What happens is this:

1. JavaScript parses this whole thing as an expression made up of several sub-expressions.
1. It then starts evaluating the expression, including evaluating sub-expressions
1. One sub-expression, `function (x) { return x }` evaluates to a function.
1. Another, `2`, evaluates to the number 2.
1. JavaScript now evaluates applying the function to the argument `2`. Here's where it gets interesting...
1. An environment is created.
1. The value '2' is bound to the name 'x' in the environment.
1. The expression 'x' (the right side of the function) is evaluated within the environment we just created.
1. The value of a variable when evaluated in an environment is the value bound to the variable's name in that environment, which is '2'
1. And that's our result.

When we talk about environments, we'll use an [unsurprising syntax][json] for showing their bindings: `{x: 2, ...}`. meaning, that the environment is a dictionary, and that the value `2` is bound to the name `x`, and that there might be other stuff in that dictionary we aren't discussing right now.

[json]: http://json.org/

### call by sharing

Earlier, we distinguished JavaScript's *value types* from its *reference types*. At that time, we looked at how JavaScript distinguishes objects that are identical from objects that are not. Now it is time to take another look at the distinction between value and reference types.

There is a property that JavaScript strictly maintains: When a value--any value--is passed as an argument to a function, the value bound in the function's environment must be identical to the original.

We said that JavaScript binds names to values, but we didn't say what it means to bind a name to a value. Now we can elaborate: When JavaScript binds a value-type to a name, it makes a copy of the value and places the copy in the environment. As you recall, value types like strings and numbers are identical to each other if they have the same content. So JavaScript can make as many copies of strings, numbers, or booleans as it wishes.

What about reference types? JavaScript does not place copies of reference values in any environment. JavaScript places *references* to reference types in environments, and when the value needs to be used, JavaScript uses the reference to obtain the original.

Because many references can share the same value, and because JavaScript passes references as arguments, JavaScript can be said to implement "call by sharing" semantics. Call by sharing is generally understood to be a specialization of call by value, and it explains why some values are known as value types and other values are known as reference types.

And with that, we're ready to look at *closures*. When we combine our knowledge of value types, reference types, arguments, and closures, we'll understand why this function always evaluates to `true` no matter what argument you apply it to:

    function (value) {
      return (function (copy) {
        return copy === value
      })(value)
    }
{: .language-javascript}

## Closures and Scope {#closures}

It's time to see how a function within a function works:

    (function (x) {
      return function (y) {
        return x
      }
    })(1)(2)
      //=> 1

First off, let's use what we learned above. Given `(`*some function*`)(`*some argument*`)`, we know that we apply the function to the argument, create an environment, bind the value of the argument to the name, and evaluate the function's expression. So we do that first with this code:

    (function (x) {
      return function (y) {
        return x
      }
    })(1)
      //=> [Function]
{: .language-javascript}
      
The environment belonging to the function with signature `function (x) ...` becomes `{x: 1, ...}`, and the result of applying the function is another function value. It makes sense that the result value is a function, because the expression for `function (x) ...`'s body is:

      function (y) {
        return x
      }
{: .language-javascript}

So now we have a value representing that function. Then we're going to take the value of that function and apply it to the argument `2`, something like this:

      (function (y) {
        return x
      })(2)
{: .language-javascript}

So we seem to get a new environment `{y: 2, ...}`. How is the expression `x` going to be evaluated in that function's environment? There is no `x` in its environment, it must come from somewhere else.

A> This, by the way, is one of the great defining characteristics of JavaScript and languages in the same family: Whether they allow things like functions to nest inside each other, and if so, how they handle variables from "outside" of a function that are referenced inside a function. For example, here's the equivalent code in Ruby:
A>
A> <<(code/k.rb)
A>
A> Now let's enjoy a relaxed Allongé before we continue!

### If functions without free variables are pure, are closures impure?

The function `function (y) { return x }` is interesting. It contains a *free variable*, `x`.[^nonlocal] A free variable is one that is not bound within the function. Up to now, we've only seen one way to "bind" a variable, namely by passing in an argument with the same name. Since the function `function (y) { return x }` doesn't have an argument named `x`, the variable `x` isn't bound in this function, which makes it "free."

[^nonlocal]: You may also hear the term "non-local variable." [Both are correct.](https://en.wikipedia.org/wiki/Free_variables_and_bound_variables) 

Now that we know that variables used in a function are either bound or free, we can bifurcate functions into those with free variables and those without:

  * Functions containing no free variables are called *pure functions*.
  * Functions containing one or more free variables are called *closures*.
  
Pure functions are easiest to understand. They always mean the same thing wherever you use them. Here are some pure functions we've already seen:

    function () {}
    
    function (x) {
      return x
    }
      
    function (x) {
      return function (y) {
        return x
      }
    }
{: .language-javascript}

The first function doesn't have any variables, therefore doesn't have any free variables. The second doesn't have any free variables, because its only variable is bound. The third one is actually two functions, one in side the other. `function (y) ...` has a free variable, but the entire expression refers to `function (x) ...`, and it doesn't have a free variable: The only variable anywhere in its body is `x`, which is certainly bound within `function (x) ...`.

From this, we learn something: A pure function can contain a closure.

X> If pure functions can contain closures, can a closure contain a pure function? Using only what we've learned so far, attempt to compose a closure that contains a pure function. If you can't, give your reasoning for why it's impossible.

Pure functions always mean the same thing because all of their "inputs" are fully defined by their arguments. Not so with a closure. If I present to you this pure function `function (x, y) { return x + y }`, we know exactly what it does with `(2, 2)`. But what about this closure: `function (y) { return x + y }`? We can't say what it will do with argument `(2)` without understanding the magic for evaluating the free variable `x`.

### it's always the environment

To understand how closures are evaluated, we need to revisit environments. As we've said before, all functions are associated with an environment. We also hand-waved something when describing our environment. Remember that we said the environment for `(function (x) { return (function (y) { return x }) })(1)` is `{x: 1, ...}` and that the environment for `(function (y) { return x })(2)` is `{y: 2, ...}`? Let's fill in the blanks!

The environment for `(function (y) { return x })(2)` is *actually* `{y: 2, '..': {x: 1, ...}}`. `'..'` means something like "parent" or "enclosure" or "super-environment." It's `function (x) ...`'s environment, because the function `function (y) { return x }` is within `function (x) ...`'s body. So whenever a function is applied to arguments, its environment always has a reference to its parent environment.

And now you can guess how we evaluate `(function (y) { return x })(2)` in the environment `{y: 2, '..': {x: 1, ...}}`. The variable `x` isn't in `function (y) ...`'s immediate environment, but it is in its parent's environment, so it evaluates to `1` and that's what `(function (y) { return x })(2)` returns even though it ended up ignoring its own argument.

A> `function (x) { return x }` is called the I Combinator or Identity Function. `function (x) { return (function (y) { return x }) }` is called the K Combinator or Kestrel. Some people get so excited by this that they write entire books about them, some are [great][mock], some--how shall I put this--are [interesting][interesting] if you use Ruby.

[mock]: http://www.amzn.com/0192801422?tag=raganwald001-20
[interesting]: https://leanpub.com/combinators "Kestrels, Quirky Birds, and Hopeless Egocentricity"

Functions can have grandparents too:

    function (x) {
      return function (y) {
        return function (z) {
          return x + y + z
        }
      }
    }
{: .language-javascript}

This function does much the same thing as:

    function (x, y, z) {
      return x + y + z
    }
{: .language-javascript}

Only you call it with `(1)(2)(3)` instead of `(1, 2, 3)`. The other big difference is that you can call it with `(1)` and get a function back that you can later call with `(2)(3)`.

{pagebreak}

A> The first function is the result of [currying] the second function. Calling a curried function with only some of its arguments is sometimes called [partial application]. Some programming languages automatically curry and partially evaluate functions without the need to manually nest them.

[currying]: https://en.wikipedia.org/wiki/Currying
[partial application]: https://en.wikipedia.org/wiki/Partial_application

### shadowy variables from a shadowy planet

An interesting thing happens when a variable has the same name as an ancestor environment's variable. Consider:

    function (x) {
      return function (x, y) {
        return x + y
      }
    }
{: .language-javascript}

The function `function (x, y) { return x + y }` is a pure function, because its `x` is defined within its own environment. Although its parent also defines an `x`, it is ignored when evaluating `x + y`. JavaScript always searches for a binding starting with the functions own environment and then each parent in turn until it finds one. The same is true of:

    function (x) {
      return function (x, y) {
        return function (w, z) {
          return function (w) {
            return x + y + z
          }
        })
      }
    }
{: .language-javascript}
          
When evaluating `x + y + z`, JavaScript will find `x` and `y` in the great-grandparent scope and `z` in the parent scope. The `x` in the great-great-grandparent scope is ignored, as are both `w`s. When a variable has the same name as an ancestor environment's binding, it is said to *shadow* the ancestor.

This is often a good thing.

### which came first, the chicken or the egg?

This behaviour of pure functions and closures has many, many consequences that can be exploited to write software. We are going to explore them in some detail as well as look at some of the other mechanisms JavaScript provides for working with variables and mutable state.

But before we do so, there's one final question: Where does the ancestry start? If there's no other code in a file, what is `function (x) { return x }`'s parent environment?

JavaScript always has the notion of at least one environment we do not control: A global environment in which many useful things are bound such as libraries full of standard functions. So when you invoke `(function (x) { return x })(1)` in the REPL, its full environment is going to look like this: `{x: 1, '..': `*global environment*`}`.

Sometimes, programmers wish to avoid this. If you don't want your code to operate directly within the global environment, what can you do? Create an environment for them, of course. Many programmers choose to write every JavaScript file like this:

    // top of the file
    (function () {
      
      // ... lots of JavaScript ...
      
    })();
    // bottom of the file
{: .language-javascript}

The effect is to insert a new, empty environment in between the global environment and your own functions: `{x: 1, '..': {'..': `*global environment*`}}`. As we'll see when we discuss mutable state, this helps to prevent programmers from accidentally changing the global state that is shared by code in every file when they use the [var keyword](#var) properly.

## Let's Talk Var {#let}

Up to now, all we've really seen are *anonymous functions*, functions that don't have a name. This feels very different from programming in most other languages, where the focus is on naming functions, methods, and procedures. Naming things is a critical part of programming, but all we've seen so far is how to name arguments.

There are other ways to name things in JavaScript, but before we learn some of those, let's see how to use what we already have to name things. Let's revisit a very simple example:

    function (diameter) {
      return diameter * 3.14159265
    }
{: .language-javascript}
      
What is this "3.14159265" number? [Pi], obviously. We'd like to name it so that we can write something like:

    function (diameter) {
      return diameter * Pi
    }
{: .language-javascript}
    
In order to bind `3.14159265` to the name `Pi`, we'll need a function with a parameter of `Pi` applied to an argument of `3.14159265`. If we put our function expression in parentheses, we can apply it to the argument of `3.14159265`:

    (function (Pi) {
      return ????
    })(3.14159265)
{: .language-javascript}
    
What do we put inside our new function that binds `3.14159265` to the name `Pi` when evaluated? Our circumference function, of course:

[Pi]: https://en.wikipedia.org/wiki/Pi

    (function (Pi) {
      return function (diameter) {
        return diameter * Pi
      }
    })(3.14159265)
{: .language-javascript}
   
This expression, when evaluated, returns a function that calculates circumferences. It differs from our original in that it names the constant `Pi`. Let's test it:

    (function (Pi) {
      return function (diameter) {
        return diameter * Pi
      }
    })(3.14159265)(2)
      //=> 6.2831853
{: .language-javascript}
    
That works! We can bind anything we want in an expression by wrapping it in a function that is immediately invoked with the value we want to bind.

### immediately invoked function expressions

JavaScript programmers regularly use the idea of writing an expression that denotes a function and then immediately applying it to arguments. Explaining the pattern, Ben Alman coined the term [Immediately Invoked Function Expression][iife] for it, often abbreviated "IIFE." As we'll see in a moment, an IIFE need not have parameters:

    (function () {
      // ... do something here...
    })();
{: .language-javascript}
    
When an IIFE binds values to names (as we did above with `Pi`), retro-grouch programmers often call it "let."[^let] And confusing the issue, upcoming versions of JavaScript have support for a `let` keyword that has a similar binding behaviour.

### var {#var}

Using an IIFE to bind names works very well, but only a masochist would write programs this way in JavaScript. Besides all the extra characters, it suffers from a fundamental semantic problem: there is a big visual distance between the name `Pi` and the value `3.14159265` we bind to it. They should be closer. Is there another way?

Yes.

Another way to write our "circumference" function would be to pass `Pi` along with the diameter argument, something like this:

    function (diameter, Pi) {
      return diameter * Pi
    }
{: .language-javascript}
    
And you could use it like this:

    (function (diameter, Pi) {
      return diameter * Pi
    })(2, 3.14159265)
      //=> 6.2831853
{: .language-javascript}
      
This differs from our example above in that there is only one environment, rather than two. We have one binding in the environment representing our regular argument, and another our "constant." That's more efficient, and it's *almost* what we wanted all along: A way to bind `3.14159265` to a readable name.

JavaScript gives us a way to do that, the `var` keyword. We'll learn a lot more about `var` in future chapters, but here's the most important thing you can do with `var`:

    function (diameter) {
      var Pi = 3.14159265;
      
      return diameter * Pi
    }
{: .language-javascript}

The `var` keyword introduces one or more bindings in the current function's environment. It works just as we want:

    (function (diameter) {
      var Pi = 3.14159265;
      
      return diameter * Pi
    })(2)
      //=> 6.2831853
{: .language-javascript}
      
You can bind any expression. Functions are expressions, so you can bind helper functions:

    function (d) {
      var calc = function (diameter) {
        var Pi = 3.14159265;
      
        return diameter * Pi
      };
      
      return "The diameter is " + calc(d)
    }
{: .language-javascript}
  
Notice `calc(d)`? This underscores what we've said: if you have an expression that evaluates to a function, you apply it with `()`. A name that's bound to a function is a valid expression evaluating to a function.[^namedfn]

[^namedfn]: We're into the second chapter and we've finally named a function. Sheesh.

A> Amazing how such an important idea--naming functions--can be explained *en passant* in just a few words. That emphasizes one of the things JavaScript gets really, really right: Functions as "first class entities." Functions are values that can be bound to names like any other value, passed as arguments, returned from other functions, and so forth.

You can bind more than one name-value pair by separating them with commas. For readability, most people put one binding per line:

    function (d) {
      var Pi   = 3.14159265,
          calc = function (diameter) {
            return diameter * Pi
          };
      
      return "The diameter is " + calc(d)
    }
{: .language-javascript}
    
These examples use the `var` keyword to bind names in the same environment as our function. We can also create a new scope using an IIFE if we wish to bind some name sin part of a function:

    function foobar () {

      // do something without foo or bar
      
      (function () {
        var foo = 'foo',
            bar = 'bar';
          
        // ... do something with foo and bar ...
      
      })();

      // do something else without foo or bar
      
    }
{: .language-javascript}

[^let]: To be pedantic, both main branches of Lisp today define a special construct called "let." One, Scheme, [uses `define-syntax` to rewrite `let` into an immediately invoked function expression that binds arguments to values](https://en.wikipedia.org/wiki/Scheme_(programming_language)#Minimalism) as shown above. The other, Common Lisp, leaves it up to implementations to decide how to implement `let`.

[iife]: http://www.benalman.com/news/2010/11/immediately-invoked-function-expression/

## Naming Functions {#named-function-expressions}

Let's get right to it. This code does *not* name a function:

    var repeat = function (str) {
      return str + str
    };
{: .language-javascript}
   
It doesn't name the function "repeat" for the same reason that `var answer = 42` doesn't name the number `42`. That snippet of code binds an anonymous function to a name in an environment, but the function itself remains anonymous.

JavaScript *does* have a syntax for naming a function, it looks like this:

    var bindingName = function actualName () {
      //...
    };
{: .language-javascript}

In this expression, `bindingName` is the name in the environment, but `actualName` is the function's actual name. This is a *named function expression*. That may seem confusing, but think of the binding names as properties of the environment, not the function itself. And indeed the name *is* a property:

    bindingName.name
      //=> 'actualName'
{: .language-javascript}

In this book we are not examining JavaScript's tooling such as debuggers baked into browsers, but we will note that when you are navigating call stacks in all modern tools, the function's binding name is ignored but its actual name is displayed, so naming functions is very useful even if they don't get a formal binding, e.g.

    someBackboneView.on('click', function clickHandler () {
      //...
    });
{: .language-javascript}

Now, the function's actual name has no effect on the environment in which it is used. To whit:

    var bindingName = function actualName () {
      //...
    };
    
    bindingName
      //=> [Function: actualName]

    actualName
      //=> ReferenceError: actualName is not defined
{: .language-javascript}
      
So "actualName" isn't bound in the environment where we use the named function expression. Is it bound anywhere else? Yes it is:

    var fn = function even (n) {
      if (n === 0) {
        return true
      }
      else return !even(n - 1)
    }
    
    fn(5)
      //=> false
    
    fn(2)
      //=> true
{: .language-javascript}
      
`even` is bound within the function itself, but not outside it. This is useful for making recursive functions.
    
### function declarations

We've actually buried the lede.[^lede] Naming functions for the purpose of debugging is not as important as what we're about to discuss. There is another syntax for naming and/or defining a function. It's called a *function declaration*, and it looks like this:

    function someName () {
      // ...
    }
{: .language-javascript}
    
This behaves a *little* like:

    var someName = function someName () {
      // ...
    }
{: .language-javascript}
    
In that it binds a name in the environment to a named function. However, consider this piece of code:

    (function () {
      return someName;
      
      var someName = function someName () {
        // ...
      }
    })()
      //=> undefined  
{: .language-javascript}
      
This is what we expect given what we learned about [var](#var): Although `someName` is declared later in the function, JavaScript behaves as if you'd written:

    (function () {
      var someName;
      
      return someName;
      
      someName = function someName () {
        // ...
      }
    })()
{: .language-javascript}

What about a function declaration without `var`?

    (function () {
      return someName;
      
      function someName () {
        // ...
      }
    })()
      //=> [Function: someName]
{: .language-javascript}

Aha! It works differently, as if you'd written:

    (function () {
      var someName = function someName () {
        // ...
      }
      return someName;
    })()
{: .language-javascript}

That difference is intentional on the part of JavaScript's design to facilitate a certain style of programming where you put the main logic up front, and the "helper functions" at the bottom. It is not necessary to declare functions in this way in JavaScript, but understanding the syntax and its behaviour (especially the way it differs from `var`) is essential for working with production code.

### function declaration caveats[^caveats]

Function declarations are formally only supposed to be made at what we might call the "top level" of a function. Although some JavaScript environments may permit it, this example is technically illegal and definitely a bad idea:

    // function declarations should not happen inside of 
    // a block and/or be conditionally executed
    if (frobbishes.arePizzled()) {
      function complainToFactory () {
        // ...
      }
    }
{: .language-javascript}

The big trouble with expressions like this is that they may work just fine in your test environment but work a different way in production. Or it may work one way today and a different way when the JavaScript engine is updated, say with a new optimization.

Another caveat is that a function declaration cannot exist inside of *any* expression, otherwise it's a function expression. So this is a function declaration:

    function trueDat () { return true }
{: .language-javascript}

But this is not:

    (function trueDat () { return true })
{: .language-javascript}
    
The parentheses make this an expression.

[^lede]: A lead (or lede) paragraph in literature refers to the opening paragraph of an article, essay, news story or book chapter. In journalism, the failure to mention the most important, interesting or attention-grabbing elements of a story in the first paragraph is sometimes called "burying the lede."

[^caveats]: A number of the caveats discussed here were described in Jyrly Zaytsev's excellent article [Named function expressions demystified](http://kangax.github.com/nfe/).

## Combinators and Function Decorators {#combinators}

### higher-order functions

As we've seen, JavaScript functions take values as arguments and return values. JavaScript functions are values, so JavaScript functions can take functions as arguments, return functions, or both. Generally speaking, a function that either takes functions as arguments or returns a function (or both) is referred to as a "higher-order" function.

Here's very simple higher-order function that takes a function as an argument:

    function repeat (num, fn) {
      var i, value;
      
      for (i = 1; i <= num; ++i)
        value = fn(i);
      
      return value;
    }
    
    repeat(3, function () { 
      console.log('Hello') 
    })
      //=>
        'Hello'
        'Hello'
        'Hello'
        undefined
{: .language-javascript}
    
Higher-order functions dominate *JavaScript Allongé*. But before we go on, we'll talk about some specific types of higher-order functions.

### combinators

The word "combinator" has a precise technical meaning in mathematics:

> "A combinator is a higher-order function that uses only function application and earlier defined combinators to define a result from its arguments."--[Wikipedia][combinators]

[combinators]: https://en.wikipedia.org/wiki/Combinatory_logic "Combinatory Logic"

If we were learning Combinatorial Logic, we'd start with the most basic combinators like `S`, `K`, and `I`, and work up from there to practical combinators. We'd learn that the fundamental combinators are named after birds following the example of Raymond Smullyan's famous book [To Mock a Mockingbird][mock].

[mock]: http://www.amazon.com/gp/product/B00A1P096Y/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B00A1P096Y&linkCode=as2&tag=raganwald001-20

In this book, we will be using a looser definition of "combinator:" Higher-order pure functions that take only functions as arguments and return a function. We won't be strict about using only previously defined combinators in their construction.

Let's start with a useful combinator: Most programmers call it *Compose*, although the logicians call it the B combinator or "Bluebird." Here is the typical[^bluebird] programming implementation:

    function compose (a, b) {
      return function (c) {
        return a(b(c))
      }
    }
{: .language-javascript}

Let's say we have:

    function addOne (number) {
      return number + 1
    }
    
    function double (number) {
      return number * 2
    }
{: .language-javascript}

With `compose`, anywhere you would write

    function doubleOfAddOne (number) {
      return double(addOne(number))
    }
{: .language-javascript}
    
You could also write:

    var doubleOfAddOne = compose(double, addOne);
{: .language-javascript}
   
This is, of course, just one example of many. You'll find lots more perusing the recipes in this book. While some programmers believe "There Should Only Be One Way To Do It," having combinators available as well as explicitly writing things out with lots of symbols and keywords has some advantages when used judiciously.

### a balanced statement about combinators

Code that uses a lot of combinators tends to name the verbs and adverbs (like `double`, `addOne`, and `compose`) while avoiding language keywords and the names of nouns (like `number`). So one perspective is that combinators are useful when you want to emphasize what you're doing and how it fits together, and more explicit code is useful when you want to emphasize what you're working with.

### function decorators {#decorators}

A *function decorator* is a higher-order function that takes one function as an argument, returns another function, and the returned function is a variation of the argument function. Here's a ridiculous example of a decorator:

    function not (fn) {
      return function (argument) {
        return !fn(argument)
      }
    }
{: .language-javascript}

So instead of writing `!someFunction(42)`, you can write `not(someFunction)(42)`. Hardly progress. But like `compose`, you could write either

    function something (x) {
      return x != null
    }
{: .language-javascript}

And elsewhere, he writes:

    function nothing (x) {
      return !something(x)
    }
{: .language-javascript}

Or:

    var nothing = not(something);
{: .language-javascript}

`not` is a function decorator because it modifies a function while remaining strongly related to the original function's semantics. You'll see other function decorators in the recipes, like [once](#once), [mapWith](#mapWith), and [maybe](#maybe). Function decorators aren't strict about being pure functions, so there's more latitude for making decorators than combinators.

[^bluebird]: As we'll discuss later, this implementation of the B Combinator is correct in languages like Scheme, but for truly general-purpose use in JavaScript it needs to correctly manage the [function context](#context).

## Building Blocks {#buildingblocks}

When you look at functions within functions in JavaScript, there's a bit of a "spaghetti code" look to it. The strength of JavaScript is that you can do anything. The weakness is that you will. There are ifs, fors, returns, everything thrown higgledy piggledy together. Although you needn't restrict yourself to a small number of simple patterns, it can be helpful to understand the patterns so that you can structure your code around some basic building blocks.

### composition

One of the most basic of these building blocks is *composition*:

    function cookAndEat (food) {
      return eat(cook(food))
    }
{: .language-javascript}
    
It's really that simple: Whenever you are chaining two or more functions together, you're composing them. You can compose them with explicit JavaScript code as we've just done. You can also generalize composition with the B Combinator or "compose" that we saw in [Combinators and Decorators](#combinators):

    function compose (a, b) {
      return function (c) {
        return a(b(c))
      }
    }

    var cookAndEat = compose(eat, cook);
{: .language-javascript}
    
If that was all there was to it, composition wouldn't matter much. But like many patterns, using it when it applies is only 20% of the benefit. The other 80% comes from organizing your code such that you can use it: Writing functions that can be composed in various ways.

In the recipes, we'll look at a decorator called  [once](#once): It ensures that a function can only be executed once. Thereafter, it does nothing. Once is useful for ensuring that certain side effects are not repeated. We'll also look at [maybe](#maybe): It ensures that a function does nothing if it is given nothing (like `null` or `undefined`) as an argument.

Of course, you needn't use combinators to implement either of these ideas, you can use if statements. But `once` and `maybe` compose, so you can chain them together as you see fit:

    function actuallyTransfer(from, to, amount) {
      // do something
    }
    
    var invokeTransfer = once(maybe(actuallyTransfer(...)));
{: .language-javascript}
    
### partial application

Another basic building block is *partial application*. When a function takes multiple arguments, we "apply" the function to the arguments by evaluating it with all of the arguments, producing a value. But what if we only supply some of the arguments? In that case, we can't get the final value, but we can get a function that represents *part* of our application.

Code is easier than words for this. The [Underscore] library provides a higher-order function called *map*.[^headache] It applies another function to each element of an array, like this:

    _.map([1, 2, 3], function (n) { return n * n })
      //=> [1, 4, 9]
{: .language-javascript}
      
This code implements a partial application of the map function by applying the function `function (n) { return n * n }` as its second argument:

    function squareAll (array) {
      return _.map(array, function (n) { return n * n })
    }
{: .language-javascript}

The resulting function--`squareAll`--is still the map function, it's just that we've applied one of its two arguments already. `squareAll` is nice, but why write one function every time we want to partially apply a function to a map? We can abstract this one level higher. `mapWith` takes any function as an argument and returns a partially applied map function.

    function mapWith (fn) {
      return function (array) {
        return _.map(array, fn)
      }
    }
    
    var squareAll = mapWith(function (n) { return n * n });
    
    squareAll([1, 2, 3])
      //=> [1, 4, 9]
{: .language-javascript}

We'll discuss mapWith again in [the recipes](#mapWith). The important thing to see is that partial application is orthogonal to composition, and that they both work together nicely:

    var safeSquareAll = mapWith(maybe(function (n) { return n * n }));
    
    safeSquareAll([1, null, 2, 3])
      //=> [1, null, 4, 9]
{: .language-javascript}

We generalized composition with the `compose` combinator. Partial application also has a combinator, which we'll see in the [partial](#partial) recipe.

[^bind]: Modern JavaScript provides a limited form of partial application through the `Function.prototype.bind` method. This will be discussed in greater length when we look at function contexts.

[^headache]: Modern JavaScript implementations provide a map method for arrays, but Underscore's implementation also works with older browsers if you are working with that headache.

[Underscore]: http://underscorejs.org

## I'd Like to Have Some Arguments. Again. {#arguments-again}

As we've discussed, when a function is applied to arguments (or "called"), JavaScript binds the values of arguments to the function's argument names in an environment created for the function's execution. What we didn't discuss is that JavaScript also binds some "magic" names in addition to any you put in the argument list.

You should never attempt to define your own bindings against these names. Consider them read-only at all times. The first is called `this` and it is bound to something called the function's [context](#context). We will explore that when we start discussing objects and classes. The second is very interesting, it's called `arguments`, and the most interesting thing about it is that it contains a list of arguments passed to the function:

    function plus (a, b) {
      return arguments[0] + arguments[1]
    }
    
    plus(2,3)
      //=> 5
{: .language-javascript}
      
Although `arguments` looks like an array, it isn't an array:[^pojo] It's more like an object[^pojo] that happens to bind some values to properties with names that look like integers starting with zero:

    function args (a, b) {
      return arguments
    }
    
    args(2,3)
      //=> { '0': 2, '1': 3 }
{: .language-javascript}

`arguments` always contains all of the arguments passed to a function, regardless of how many are declared. Therefore, we can write `plus` like this:

    function plus () {
      return arguments[0] + arguments[1]
    }
    
    plus(2,3)
      //=> 5
{: .language-javascript}

When discussing objects, we'll discuss properties in more depth. Here's something interesting about `arguments`:

    function howMany () {
      return arguments['length']
    }
    
    howMany()
      //=> 0
    
    howMany('hello')
      //=> 1
    
    howMany('sharks', 'are', 'apex', 'predators')
      //=> 4
{: .language-javascript}
      
The most common use of the `arguments` binding is to build functions that can take a variable number of arguments. We'll see it used in many of the recipes, starting off with [partial application](#simple-partial) and [ellipses](#ellipses).
      
[^pojo]: We'll look at [arrays](#arrays) and [plain old javascript objects](#objects) in depth later.

## Summary

T> ### Functions
T>
T> * Functions are values that can be part of expressions, returned from other functions, and so forth.
T> * Functions are *reference values*.
T> * Functions are applied to arguments.
T> * The arguments are passed by sharing, which is also called "pass by value."
T> * Function bodies have zero or more expressions.
T> * Function application evaluates to the value of the last expression evaluated or `undefined`.
T> * Function application creates a scope. Scopes are nested and free variable references closed over.
T> * Variables can shadow variables in an enclosing scope.
T> * `let` is an idiom where we create a function and call it immediately in order to bind values to names.
T> * JavaScript uses `var` to bind variables within a function's scope.
