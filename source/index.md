# A Pull of the Lever: Prefaces

![Caffe Molinari](caffemolinari.jpg)

> "Café Allongé, also called Espresso Lungo, is a drink midway between an Espresso and Americano in strength. There are two different ways to make it. The first, and the one I prefer, is to add a small amount of hot water to a double or quadruple Espresso Ristretto. Like adding a splash of water to whiskey, the small dilution releases more of the complex flavours in the mouth.
>
> "The second way is to pull an extra long double shot of Espresso. This achieves approximately the same ratio of oils to water as the dilution method, but also releases a different mix of flavours due to the longer extraction. Some complain that the long pull is more bitter and detracts from the best character of the coffee, others feel it releases even more complexity.
>
> "The important thing is that neither method of preparation should use so much water as to result in a sickly, pale ghost of Espresso. Moderation in all things." 

## Foreword by Michael Fogus

As a life-long bibliophile and long-time follower of Reg's online work, I was excited when he started writing books. However, I'm very conservative about books -- let's just say that if there was a aftershave scented to the essence of "Used Book Store" then I would be first in line to buy. So as you might imagine I was "skeptical" about the decision to release JavaScript Allongé as an ongoing ebook, with a pay-what-you-want model. However, Reg sent me a copy of his book and I was humbled. Not only was this a great book, but it was also a great way to write and distribute books. Having written books myself, I know the pain of soliciting and receiving feedback.

The act of writing is an iterative process with (very often) tight revision loops. However, the process of soliciting feedback, gathering responses, sending out copies, waiting for people to actually read it (if they ever do), receiving feedback and then ultimately making sense out of how to use it takes weeks and sometimes months. On more than one occasion I've found myself attempting to reify feedback with content that either no longer existed or was changed beyond recognition. However, with the Leanpub model the read-feedback-change process is extremely efficient, leaving in its wake a quality book that continues to get better as others likewise read and comment into infinitude.

In the case of JavaScript Allongé, you'll find the Leanpub model a shining example of effectiveness. Reg has crafted (and continues to craft) not only an interesting book from the perspective of a connoisseur, but also an entertaining exploration into some of the most interesting aspects of his art. No matter how much of an expert you think you are, JavaScript Allongé has something to teach you... about coffee. I kid.

As a staunch advocate of functional programming, much of what Reg has written rings true to me. While not exclusively a book about functional programming, JavaScript Allongé will provide a solid foundation for functional techniques. However, you'll not be beaten about the head and neck with dogma. Instead, every section is motivated by relevant dialog and fortified with compelling source examples. As an author of programming books I admire what Reg has managed to accomplish and I envy the fine reader who finds JavaScript Allongé via some darkened channel in the Internet sprawl and reads it for the first time.

Enjoy.

-- Fogus, [fogus.me](http://www.fogus.me)

## Foreword by Matthew Knox

A different kind of language requires a different kind of book.

JavaScript holds surprising depths--its scoping rules are neither strictly lexical nor strictly dynamic, and it supports procedural, object-oriented (in several flavors!), and functional programming.  Many books try to hide most of those capabilities away, giving you recipes for writing JavaScript in a way that approximates class-centric programming in other languages.  Not JavaScript Allongé.  It starts with the fundamentals of values, functions, and objects, and then guides you through JavaScript from the inside with exploratory bits of code that illustrate scoping, combinators, context, state, prototypes, and constructors.

Like JavaScript itself, this book gives you a gentle start before showing you its full depth, and like a Cafe Allongé, it's over too soon.  Enjoy!

--Matthew Knox, [mattknox.com](http://mattknox.com)

## Why JavaScript Allongé?

*JavaScript Allongé* solves two important problems for the ambitious JavaScript programmer. First, *JavaScript Allongé* gives you the tools to deal with JavaScript bugs, hitches, edge cases, and other potential pitfalls.

There are plenty of good directions for how to write JavaScript programs. If you follow them without alteration or deviation, you will be satisfied. Unfortunately, software is a complex thing, full of interactions and side-effects. Two perfectly reasonable pieces of advice when taken separately may conflict with each other when taken together. An approach may seem sound at the outset of a project, but need to be revised when new requirements are discovered.

When you "leave the path" of the directions, you discover their limitations. In order to solve the problems that occur at the edges, in order to adapt and deal with changes, in order to refactor and rewrite as needed, you need to understand the underlying principles of the JavaScript programming language in detail.

You need to understand *why* the directions work so that you can understand *how* to modify them to work properly at or beyond their original limitations. That's where *JavaScript Allongé* comes in.

*JavaScript Allongé* is a book about programming with functions, because [JavaScript] is a programming language built on flexible and powerful functions. *JavaScript Allongé* begins at the beginning, with values and expressions, and builds from there to discuss types, identity, functions, closures, scopes, and many more subjects up to working with classes and instances. In each case, *JavaScript Allongé* takes care to explain exactly how things work so that when you encounter a problem, you'll know exactly what is happening and how to fix it.

Second, *JavaScript Allongé* provides recipes for using functions to write software that is simpler, cleaner, and less complicated than alternative approaches that are object-centric or code-centric. JavaScript idioms like function combinators and decorators leverage JavaScript's power to make code easier to read, modify, debug and refactor, thus *avoiding* problems before they happen.

*JavaScript Allongé* teaches you how to handle complex code, and it also teaches you how to simplify code without dumbing it down. As a result, *JavaScript Allongé* is a rich read releasing many of JavaScript's subtleties, much like the Café Allongé beloved by coffee enthusiasts everywhere.

[JavaScript]: https://developer.mozilla.org/en-US/docs/JavaScript

### how the book is organized

*JavaScript Allongé* introduces new aspects of programming with functions in each chapter, explaining exactly how JavaScript works. Code examples within each chapter are small and emphasize exposition rather than serving as patterns for everyday use.

Following each chapter are a series of recipes designed to show the application of the chapters ideas in practical form. While the content of each chapter builds naturally on what was discussed in the previous chapter, the recipes may draw upon any aspect of the JavaScript programming language.

## A Personal Word About The Recipes

As noted, *JavaScript Allongé* alternates between chapters describing the semantics of JavaScript's functions with chapters containing recipes for writing programs with functions. You can read the book in order or read the chapters explaining JavaScript first and return to the recipes later.

The recipes share a common theme: They hail from a style of programming inspired by the creation of small functions that compose with each other. Using these recipes, you'll learn when it's appropriate to write:

    return mapWith(maybe(getWith('name')))(customerList);
{: .language-javascript runnable="false"}

Instead of:

    return customerList.map(function (customer) {
      if (customer) {
        return customer.name
      }
    });
{: .language-javascript}

As well as how it works and how to refactor it when you need. This style of programming is hardly the most common thing anyone does in JavaScript, so the argument can be made that more "practical" or "commonplace" recipes would be helpful. If you never read any other books about JavaScript, if you avoid blog posts and screen casts about JavaScript, if you don't attend workshops or talks about JavaScript, then I agree that this is not One Book to Rule Them All.

But given that there are other resources out there, and that programmers are curious creatures with an unslakable thirst for personal growth, we choose to provide recipes that you are unlikely to find anywhere else in anything like this concentration. The recipes reinforce the lessons taught in the book about functions in JavaScript.

You'll find all of the recipes collected online at [http://allong.es](http://allong.es). They're free to share under the MIT license.

[Reginald Braithwaite](http://braythwayt.com)  
reg@braythwayt.com  
@raganwald

## Legend

Some text in monospaced type like `this` in the text represents some code being discussed. Some monospaced code in its own lines also represents code being discussed:

    this.async = do (async = undefined) ->

      async = (fn) ->
        (argv..., callback) ->
          callback(fn.apply(this, argv))
{: .language-javascript runnable="false"}
          
Sometimes it will contain some code for you to type in for yourself. When it does, the result of typing something in will often be shown using `//=>`, like this:

    2 + 2
      //=> 4
{: .language-javascript}

T> A paragraph marked like this is a "key fact." It summarizes an idea without adding anything new.

X> A paragraph marked like this is a suggested exercise to be performed on your own.

A> A paragraph marked like this is an aside. It can be safely ignored. It contains whimsey and other doupleplusunserious logorrhea that will *not* be on the test.
