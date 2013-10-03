# The Golden Crema

![You've earned a break!](la_marzocco.jpg)

## Author's Notes

Hello, and thank you for supporting my writing!

An ebook like this is never "done." Like open source software, it lasts as long as people are interested in maintaining it, and then it is abandoned as people move on to something new. But the ideas live on, informing the new books and projects that come after it, just as it was informed by the books and projects that came before it.

Although this book as been "complete" in some sense for months, I continue to use its ideas and to write about things with its ideas in mind. Sometimes I receive confirmation that all is well. Sometimes I receive helpful suggestions. And sometimes, I hear that things could be much, much better.

So please, write and let me know what you think.

Thank you!

### `allong.es`

This book inspired a companion JavaScript library called [allong.es](http://allong.es). It's free to use, of course. Please try it out. It complements libraries you may already be using like Underscore.

### the new nomenclature (2013-04-08)

Based on feedback from people exposed to other programming languages, I've renamed some of the recipe functions. While doing so, I also rewrote the partial application and some other parts of [allong.es](http://allong.es) to exploit symmetry.

The new nomenclature has a few conventions. First, unless suffixed with `Now`, all functions are already curried. So you can write either `map(list, function)` or `map(list)(function)`. There isn't one, but if there was a `map` that wasn't curried, it would be called `mapNow`.

By default, functions take a data structure first and an operation second and are named after a verb, i.e. `map`, `filter`. As noted, they are curried by default.

Binary functions like `map` have a variation with their arguments flipped to have the "verb" first and the subject second. They are suffixed `With`, so you call `map(list, function)` or `mapWith(function, list)`.

Under the new nomenclature, what used to be called `splat` is now called `mapWith`, and when you supply only the function, the currying takes care of returning a function that takes as list as its argument. You're mapping *with* a function.

### apply vs. call

In functional programming tradition, the function `apply` is used for functional application. Variations include `applyLeft` and `applyLast`. JavaScript is a little different: All functions have two methods: `.apply` takes an array of arguments, while `.call` takes separate arguments.

So the recipes in JavaScript Allongé follow the JavaScript convention: Those named `apply` take an array of arguments, while those named `call` take individual arguments. If you're coming to this book with some functional programming under your belt, you'll find the functions work the same way, it's just that there are two of them to handle the two different ways to apply arguments to a function.

## How to run the examples {#online}

If you follow the instructions at [nodejs.org][install] to install NodeJS and JavaScript,[^whoa] you can run an interactive JavaScript [REPL][repl] on your command line simply by typing `node`. This is how the examples in this book were tested, and what many programmers will do.

On OS X, you have the option of running Safaris's JavaScript engine from the command line, or of installing a TextMate bundle. We didn't use this method to test the examples, [so proceed at your own risk](http://metaskills.net/2010/07/09/interactive-javascript-console-with-textmate/). But have fun!

Almost all browsers have a mechanism to function as JavaScript [REPLs][repl], allowing you to type JavaScript expressions into a debug console.

[repl]: https://en.wikipedia.org/wiki/REPL "Read–eval–print loop"
[install]: http://nodejs.org/

[^whoa]: Instructions for installing NodeJS and modules like JavaScript onto a desktop computer is beyond the scope of this book, especially given the speed with which things advance. Fortunately, there are always up-to-date instructions on the web.

## Thanks!

### Daniel Friedman and Matthias Felleisen

![The Little Schemer](little-schemer.jpg)

*JavaScript Allongé* was inspired by [The Little Schemer] by Daniel Friedman and Matthias Felleisen. But where *The Little Schemer's* primary focus is recursion, *JavaScript Allongé's* primary focus is **functions as first-class values**.

{pagebreak}

### Richard Feynman

![QED: The Strange Theory of Light and Matter](qed.jpg)

Richard Feynman's [QED] was another inspiration: A book that explains Quantum Electrodynamics and the "Sum of the Histories" methodology using the simple expedient of explaining how light reflects off a mirror, and showing how most of the things we think are happening--such as light travelling on a straight line,  the angle of reflection equalling the angle of refraction, or that a beam of light only interacts with a small portion of the mirror, or that it reflects off a plane--are all wrong. And everything is explained in simple, concise terms that build upon each other logically.

[JavaScript]: https://developer.mozilla.org/en-US/docs/JavaScript
[The Little Schemer]: http://www.amzn.com/0262560992?tag=raganwald001-20
[QED]: http://www.amzn.com/0691125759?tag=raganwald001-20

{pagebreak}

## CoffeeScript Ristretto

![an intense doppio of programming](coffeescript_ristretto_medium.jpg)

[CoffeeScript Ristretto](http://leanpub.com/coffeescript-ristretto) is the companion book to *JavaScript Allongé*.

## Copyright Notice

The original words in this book are (c) 2012-2013, Reginald Braithwaite. [Some rights reserved][license].

[license]: http://creativecommons.org/licenses/by-sa/3.0/deed.en_US "Creative Commons Attribution-ShareAlike 3.0 Unported License"

### images

* The picture of the author is (c) 2008, [Joseph Hurtado](http://www.flickr.com/photos/trumpetca/), All Rights Reserved. 
* [Cover image](http://www.flickr.com/photos/avlxyz/4907262046) (c) 2010, avlxyz. [Some rights reserved][by-sa]. 
* [Double ristretto menu](http://www.flickr.com/photos/digitalcolony/5054568279/) (c) 2010, Michael Allen Smith. [Some rights reserved][by-sa].
* [Short espresso shot in a white cup with blunt handle](http://www.flickr.com/photos/everydaylifemodern/1353570874/) (c) 2007, EVERYDAYLIFEMODERN. [Some rights reserved][by-nd].
* [Espresso shot in a caffe molinari cup](http://www.flickr.com/photos/everydaylifemodern/434299813/) (c) 2007, EVERYDAYLIFEMODERN. [Some rights reserved][by-nd].
* [Beans in a Bag](http://www.flickr.com/photos/the_rev/2295096211/) (c) 2008, Stirling Noyes. [Some Rights Reserved][by].
* [Free Samples](http://www.flickr.com/photos/thedigitelmyr/6199419022/) (c) 2011, Myrtle Bech Digitel. [Some Rights Reserved][by-sa].
* [Free Coffees](http://www.flickr.com/photos/sagamiono/4391542823/) image (c) 2010, Michael Francis McCarthy. [Some Rights Reserved][by-sa].
* [La Marzocco](http://www.flickr.com/photos/digitalcolony/3924227011/) (c) 2009, Michael Allen Smith. [Some rights reserved][by-sa].
* [Cafe Diplomatico](http://www.flickr.com/photos/15481483@N06/6231443466/) (c) 2011, Missi. [Some rights reserved][by-sa].
* [Sugar Service](http://www.flickr.com/photos/tjgfernandes/2785677276/) (c) 2008 Tiago Fernandes. [Some rights reserved][by].
* [Biscotti on a Rack](http://www.flickr.com/photos/kirstenloza/4805716699/) (c) 2010 Kirsten Loza. [Some rights reserved][by].
* [Coffee Spoons](http://www.flickr.com/photos/jenny-pics/5053954146/) (c) 2010 Jenny Downing. [Some rights reserved][by].
* [Drawing a Doppio](http://www.flickr.com/photos/33388953@N04/4017985434/) (c) 2008 Osman Bas. [Some rights reserved][by].
* [Cupping Coffees](http://www.flickr.com/photos/tangysd/5953453156/) (c) 2011 Dennis Tang. [Some rights reserved][by-sa].
* [Three Coffee Roasters](http://www.flickr.com/photos/digitalcolony/4000837035/) (c) 2009 Michael Allen Smith. [Some rights reserved][by-sa].
* [Blue Diedrich Roaster](http://www.flickr.com/photos/digitalcolony/4309812256/) (c) 2010 Michael Allen Smith. [Some rights reserved][by-sa].
* [Red Diedrich Roaster](http://www.flickr.com/photos/bike/3237859728/) (c) 2009 Richard Masoner. [Some rights reserved][by-sa].
* [Roaster with Tree Leaves](http://www.flickr.com/photos/lacerabbit/2102801319/) (c) 2007 ting. [Some rights reserved][by-nd].
* [Half Drunk](http://www.flickr.com/photos/nalundgaard/4785922266/) (c) 2010 Nicholas Lundgaard. [Some rights reserved][by-sa].
* [Anticipation](http://www.flickr.com/photos/paulmccoubrie/6828131856/) (c) 2012 Paul McCoubrie. [Some rights reserved][by-nd].
* [Ooh!](http://www.flickr.com/photos/mikecogh/7676649034/) (c) 2012 Michael Coghlan. [Some rights reserved][by-sa].
* [Intestines of an Espresso Machine](http://www.flickr.com/photos/yellowskyphotography/5641003165/) (c) 2011 Angie Chung. [Some rights reserved][by-sa].
* [Bezzera Espresso Machine](http://www.flickr.com/photos/andynash/6204253236/) (c) 2011 Andrew Nash. [Some rights reserved][by-sa].
*[Beans Ripening on a Branch](http://www.flickr.com/photos/28705377@N04/5306009552/) (c) 2008 John Pavelka. [Some rights reserved][by].
* [Cafe Macchiato on Gazotta Della Sport](http://www.flickr.com/photos/shavejonathan/2343081208/) (c) 2008 Jon Shave. [Some rights reserved][by].
* [Jars of Coffee Beans](http://www.flickr.com/photos/ilovememphis/7103931235/) (c) 2012 Memphis CVB. [Some rights reserved][by-nd].
* [Types of Coffee Drinks](http://www.flickr.com/photos/mikecogh/7561440544/) (c) 2012 Michael Coghlan. [Some rights reserved][by-sa].
* [Coffee Trees](http://www.flickr.com/photos/dtownsend/6171015997/) (c) 2011 Dave Townsend. [Some rights reserved][by-sa].
* [Cafe do Brasil](http://www.flickr.com/photos/93425126@N00/313053257/) (c) 2003 Temporalata. [Some rights reserved][by-sa].
* [Brown Cups](http://www.flickr.com/photos/digitalcolony/2833809436/) (c) 2007 Michael Allen Smith. [Some rights reserved][by-sa].
* [Mirage](http://www.flickr.com/photos/citizenhelder/5006498068/) (c) 2010 Mira Helder. [Some rights reserved][by].
* [Coffee Van with Bullet Holes](http://www.flickr.com/photos/joncrel/237026246/) (c) 2006 Jon Crel. [Some rights reserved][by-nd].
* [Disassembled Elektra](http://www.flickr.com/photos/nalundgaard/3163852170/) (c) 2009 Nicholas Lundgaard. [Some rights reserved][by-sa].
* [Nederland Buffalo Bills Coffee Shop](http://www.flickr.com/photos/47000103@N05/6525288841/) (c) 2009 Charlie Stinchcomb. [Some rights reserved][by-sa].
* [For the love of coffee](http://www.flickr.com/photos/lotzman/978418891/) (c) 2007 Lotzman Katzman. [Some rights reserved][by].
* [Saltspring Processing Facility Pictures](http://www.flickr.com/photos/kk/sets/72157626168201654/with/5484839102/) (c) 2011 Kris Krug. [Some rights reserved][by-sa].

[by-sa]: http://creativecommons.org/licenses/by-sa/2.0/deed.en
[by-nd]: http://creativecommons.org/licenses/by-nd/2.0/deed.en
[by]: http://creativecommons.org/licenses/by/2.0/deed.en

## About The Author

When he's not shipping JavaScript, Ruby, CoffeeScript and Java applications scaling out to millions of users, Reg "Raganwald" Braithwaite has authored [libraries][lib] for JavaScript, JavaScript and Ruby programming such as Method Combinators, Katy, JQuery Combinators, YouAreDaChef, andand, and others.

[lib]: http://github.com/raganwald

He writes about programming on his "[Homoiconic][homo]" un-blog as well as general-purpose ruminations on his [posterous space][post]. He is also known for authoring the popular [raganwald][rag] programming blog from 2005-2008.

[homo]: http://github.com/raganwald/homoiconic
[post]: http://raganwald.posterous.com
[rag]: http://weblog.raganwald.com

### contact

Twitter: [@raganwald](https://twitter.com/raganwald)  
Email: [reg@braythwayt.com](mailto:reg@braythwayt.com)

![Reg "Raganwald" Braithwaite ](reg2.jpg)
