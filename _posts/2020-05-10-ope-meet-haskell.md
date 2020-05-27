---
layout: post
title:  "Ope meet Haskell"
date:   2020-05-10 12:00:25 +0100
categories: [programming-language, functional-programming, haskell,]
---

There are two main styles of programming. Imperative and Declarative. Programs written in the imperative style
focus on describing the exact steps the computer should take i.e the "how" while programs written in the declarative style focus on describing the desired result we want i.e the "what".

Functional programming is a declarative style of programming. It is called functional programming because programs written in the functional style are made up of functions which are combined in different ways to form the main program. Functions in functional programming are mathematical functions not procedures i.e given the same input, a function will always return the same output
without any side-effects.

Different languages support different styles of programming. To learn functional programming, I needed to learn a language that supports the
functional style. There are a number of functional languages - F#, Scala, Haskell, Elm. I decided to go with Haskell because from what I read it was the "purest" of the functional languages. For example, with haskell you cannot have any side-effects while with some functional languages you can.

## Haskell
Haskell is a typeful, lazily-evaluated functional language.

`typeful` - Haskell is statically typed. This means that every value in Haskell has a type known and checked at compile time. Unlike some statically typed languages, you don't need to add types to every value. Haskell can infer the type of a variable based on how it is used.

`lazily-evaluated` - values in haskell are not evaluated until they are actually needed. So for example if you have a function that accepts two arguments and returns the first, the second argument will not be
evaluated. This contrasts with most programming languages that instead use eager evaluation i.e the arguments to a function are evaluated before the function body. Lazy evaluation is useful because it means that we can write programs that don't depend on the order of evaluation of arguments and don't evaluate more than they need to.

`functional` - functions are combined in different ways to form programs. Functions in Haskell are first class values. That means they can be
passed in as arguments to functions, returned from functions and stored in data structures just like any other value.


## How I went about learning Haskell
I started off reading `Learning you a Haskell for great good`. I didn't really enjoy it because I felt like it didn't teach me concepts, it just showed me a lot of Haskell code and it didn't have exercises either.
In my search for another book, I found this [talk]((https://www.youtube.com/watch?v=Bg9ccYzMbxc&t=4s)) by Chris Allen. I then picked up [`A Gentle introduction to Haskell`](https://www.haskell.org/tutorial/haskell-98-tutorial.pdf).
This is the official language tutorial for Haskell 98 so it is a bit dated but it
suited my needs perfectly. It explained Haskell concept by concept and it was also short enough that it didn't feel daunting. I also skimmed through a book recommended by this tutorial -
`Introduction to Functional Programming by Richard Bird`. I enjoyed this book because it taught the functional style pretty well with Haskell as a medium of instruction. It also touched on things like using mathematical laws to
derive more efficient programs. All of this is possible because we can use equational reasoning with pure mathematical functions.
At this point, I had learned enough to write basic Haskell programs but I could tell I still had(still have) major gaps in knowledge so I am currently going through `Programming in
Haskell` by Graham Hutton. I also picked up different papers to understand Monads and Lambda calculus. You can find links in the resources section. If I had to start all over
I would probably learn from [`Haskell from first principles`](https://haskellbook.com/). It is pretty big book (over a 1000 pages. I prefer shorter books). I like that the authors put in effort to test that it is a good book to use to learn. It's a book
I will probably read at a later date to reinforce the things I have learned so far.

I'm still not a Haskell expert. I still need to practise to become fluent with it and I have knwoledge gaps I am aware of like category theory (functors, applicatives etc). I learn how to use Haskell to create production grade programs.


## Key Haskell concepts
Learning Haskell exposed me to a couple of new concepts. I will introduce some of them briefly.

* **Pattern matching** - pattern matching is arguably my favorite Haskell feature. It is a way of deconstructing a complex objects into its components. It is clearer than using accessor methods. I like how it used when defining functions using a set of equations. The resulting definition is very easy to read and the base cases standout. For example,
```haskell
length [] = 0
length (x:xs) = 1 + length xs
```
* **Parametric Polymorphism** - I love how you can write a function that operates on a family of types. In Go for example, you still can't do that. It means functions can be as general as they need to be.
* **Ad-hoc polymorphism via Type classes** - Haskell also suports ad-hoc polymorphism i.e the same function having different implementations based on the type. It is similar to interfaces in a lot of languages. Parametric polymorphism are often used together. Often you will have a function that has a context in its type signature. This means that only types that belong to the class named in the context can be passed to this function.
```haskell
-- this illustrates both parametric and ad-hoc polymorphism
qsort :: Ord a => [a] => [a]
qsort [] = []
qsort (x:rest) = [v | v <- rest, v <= x] ++ [x] ++ [v | v <- rest, v > x]
```
* **Algebraic Data types** - Algebraic Data types are a way of creating more complex data types using either Sum types or Product types. For example,
```haskell
-- a Maybe type is either Nothing or Just 5 - this is a sum type
data Maybe a = Nothing | Just a
-- a pair contains two types - this is a product type
data Pair a b = Pair a b
```
* **Monadic I/O** - Haskell is made up of pure functions with no side-effects. This is a problem because without side-effects a program cannot actually perform useful work. To solve this problem, Haskell uses the IO monad. A monad is a type constructor plus two functions `return` and `bind`(aka `>>=`). Values with type `IO a` are also called actions. Actions can be combined together to form a bigger action. The rules for combining actions is based on the monadic laws. `return` takes an ordinary value and wraps it in IO. `bind` combines two IO actions so that result of one is the input of the other. To make it easier to work with haskell has the do notation that is used to automatically sequence IO actions,

There is a lot more to learn about Haskell. A concise introduction can be found [here](https://github.com/thma/WhyHaskellMatters)

## First impressions

### The good
* Pattern matching is awesome.
* A rich type system and type inference is also awesome since it eliminates a class of bugs.
* I love how functions are defined using a set of equations. I find functions defined using equations very readable and natural because the base cases are explicit when you define recursive functions.
* I love love that you can do equational reasoning with haskell since the compiler guarantees that a pure function is actually pure.

### The not so good
* Compiler messages can be scary. For example, I made a mistake calling fold with two arguments flipped and I got a scary looking very long stack trace. To catch errors like this earlier, I looked for Haskell language tools in particular a Haskell Language server that integrates with VSCode. I found the HIE extension but I couldn't get it to work locally until I used it with a [devcontainer](https://github.com/hmemcpy/haskell-hie-devcontainer). This extension gave me advanced language support like intellisense, jump to definition etc but I couldn't install [stack](https://docs.haskellstack.org/en/stable/README/) in the devcontainer.
* Haskell has a steep learning curve before you are productive. I struggled with doing basic things that I could do with imperative languages.

## Conclusion
So was this worthwhile?

ABSOLUTELY!

I learned a lot. The best part is learning new concepts to compare and contrast with what I already know. Programming without mutation is an
interesting constraint. Lazy evaluation is also different from eager evaluation that all the languages I have learned before use.

Language am I likely to meet next? Erlang.

-----
Resources
* [John Backus's paper](https://www.thocp.net/biographies/papers/backus_turingaward_lecture.pdf)
* [Escape from the Ivory Tower: The Haskell Journey](https://www.youtube.com/watch?v=re96UgMk6GQ)
* [Why functional programming matters](https://www.cs.kent.ac.uk/people/staff/dat/miranda/whyfp90.pdf)
* [Monads for functional programming](https://homepages.inf.ed.ac.uk/wadler/papers/marktoberdorf/baastad.pdf)
* [Comprehending Monads](http://www.cs.ox.ac.uk/jeremy.gibbons/publications/ringads.pdf)
* [Programming in Haskell](https://amzn.to/2AHvDKT)
* [WhyHaskellMatters - shortest haskell tutorial](https://github.com/thma/WhyHaskellMatters)
* [Thinking Functional with Haskell](https://amzn.to/2WG0nV6)

---
Thanks Olamilekan and Lynda for proof reading :)
