---
layout: post
title:  "Ope meet Haskell - learning Haskell in 2020"
date:   2020-05-10 12:00:25 +0100
categories: [haskell, functional-programming, programming-language]
---


## Introduction

There are two main styles of programming. There is the imperative style and there is the declarative style. Programs written in the imperative style
consist mostly of statements that modify the state of the computer while programs written in the declarative style describe the desired result without explicit control flow.
Functional programming is a declarative style of programming. It is called functional programming because programs written in the functional style are made up of functions which are used as
building blocks for the main program. Functions in functional programming are mathematical functions not procedures i.e given the same input, a function will always return the same output
without any side-effects. The term functional programming was coined by John Backus in his 1977 Turing Award lecture. John Backus won the Turing award for his work creating Fortran and while
people expected his lecture was going to be about Fortran, he proceeded to talk about functional programming. This talk was very influential and spurred a generation of researchers to create functional languages.

I don't remember when I first heard the term functional programming but it had always piqued my interest especially because of the mathematical nature of it. I finally got around to explore it properly.
Different languages support different styles of programming, the ones I know don't support the functional style very well so I had to learn a new language. There were a couple of choices -
F#, Scala, Haskell, Elm. I decided to go with Haskell because from what I read it was the "purest" of the functional languages even though there was going to be a steep learning curve. For example, with haskell
you cannot normally modify state while with some of these languages you actually could. Immutability is important if you want to avoid side-effects and use pure functions.

## Haskell
Haskell is a typeful, lazily-evaluated purely functional language.

`typefull` - every value in Haskell has a type and programs are checked at compile time to ensure that they don't break type contracts. This is eliminates a certain class of bugs. However, you don't need to
explicitly write types every time. Haskell has very powerful type inference so it can infer the types for you instead.

`lazily-evaluated` - values in haskell are not evaluated until they are actually needed. So for example if you have a function that accepts two arguments and returns the first, the second argument will not be
evaluated. This contrasts with most programming languages that instead use eager evaluation i.e the arguments to a function are evaluated before the function body. Lazy evaluation is useful because it means that
we can write programs that don't depend on the order of evaluation of arguments and don't evaluate more than they need to.

`purely functional` - Haskell has no mainstream mechanisms (there are some unsafe functions) for creating impure functions (by impure, we mean functions that are not mathematical functions). You have to write
pure functions and you can't modify existing objects, you return new objects instead.


## How I went about learning Haskell
I started off, reading `Learning you a Haskell for great good`. I didn't really enjoy it because I felt like it didn't teach me concepts, it just showed me a lot of Haskell code and it didn't have exercises either.
In my search for another book, I found this [talk]((https://www.youtube.com/watch?v=Bg9ccYzMbxc&t=4s)) by Chris Allen. I then picked up [`A Gentle introduction to Haskell`](https://www.haskell.org/tutorial/haskell-98-tutorial.pdf).
This is the official language tutorial for Haskell 98 so it is a bit dated but it
suited my needs perfectly. It explained Haskell concept by concept and it was also short enough that it didn't feel daunting. I also skimmed through a book recommended by this tutorial -
`Introduction to Functional Programming by Richard Bird`. I enjoyed this book because it taught the functional style pretty well with Haskell as a medium of instruction. It also touched on things like using mathematical laws to
derive more efficient programs - all of this being made possible because of equational reasoning and the mathematical properties that functions have.
At this point, I had learned enough to write basic Haskell programs but I could tell I still had(still have) major gaps in knowledge so I am currently going through `Programming in
Haskell` by Graham Hutton. I also picked up different papers to understand concepts like Monads. I am still going through these papers. You can find links in the resources section. At this point, I should say that if I had to start all over
I would probably use [`Haskell from first principles`](https://haskellbook.com/). It is pretty thick book (which I don't like, I prefer shorter books) but I like that the authors put in effort to test that it is a good book to use to learn. I just might to through it properly at a later date to reinforce all the things I have already learned. It is also good that it contains information about what to do and also about what to do when things go wrong. This is important because
Haskell's compiler errors can be scary. What is left is to practise and be fluent in haskell, learn enough category theory to get by (applicatives, functor, monads, monoids etc) and actually build a thing using Haskell.


## Key Haskell concepts

* **Pattern matching** - pattern matching is arguably my favorite Haskell feature. It is a way of deconstructing a complex objects into its components. It is clearer and more elegant that having to use accessor methods. I also like how it used when defining functions using a set of equation. The resulting definition is very easy to read. For example,
```haskell
length [] = 0
length (x:xs) = 1 + length xs
```
* **Parametric Polymorphism** - I love how you can write a function that operates on a family of types. In Go for example, you still can't do that. It means functions can be as general as they need to be.
* **Ad-hoc polymorphism via Type classes** - Haskell also suports ad-hoc polymorphism i.e the same function having different implementation based on the type. It is similar to interfaces in a lot of languages. Parametric polymorphism are often used together. Often you will have a function that has a context in it's type signature. This means that only types that belong to the class named in the context can be passed to this function.
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
* **Monadic I/O** - Haskell is made up of pure functions with no side-effects. This is a problem because without side-effects a program cannot actually perform useful work. To solve this problem, Haskell has the IO type. It is used for encapsulating computation that is done later in the main loop where monads come in play is in defining the laws that govern how computation is sequenced. A monad is a type function with two operations - `return` and `>>=`(bind). `return` takes an ordinary value in lifts it into the world of IO while bind combines two IO computations so that result of one is the input of the other. The other important bit about this are the Monadic laws which every monads should obey even though they are not typically enforcable. These laws are useful because we can replace an expression with an equivalent expression that for example is more efficient.

```haskell
-- hello world in Haskell
main :: IO ()
main = do
    name <- getLine
    putStr "hello " ++ name

-- the above is syntatic sugar for this code. The bind operation for the IO monad is used here to create an action from two actions
main :: IO ()
main = getLine >>= \name -> putStr "hello " ++ name
```

There is a lot more to learn about Haskell. A concise introduction can be found [here](https://github.com/thma/WhyHaskellMatters)

## First impressions
### The good
Pattern matching is awesome. A rich type system is also awesome since it eliminates a class of bugs. I love how functions are defined using equations. I find that very readable and natural if you think about defining recursive functions. You take care of the base cases
with an equation for them. I love love that you can do equational reasoning with haskell and it is not just by convention, the compiler actually guaratees that a pure function is actually pure.
### The not so good
Compiler messages can be scary. For example in this ![gif](haskell-scary-error.gif "Scary error"), the error was that I flipped two arguments but the compiler returned some elaborate error. This made me search for language tools like liters and a language server extension that would catch these kinds of errors as I write them but I couldn't get HIE to work locally and I had to use a [devcontainer](https://github.com/hmemcpy/haskell-hie-devcontainer). This is pretty nifty and worked - I got advanced language support like intellisense, jump to definition etc but I couldn't install [stack](https://docs.haskellstack.org/en/stable/README/) on it. Haskell is also a struggle because of it's steep learning curve.

## Conclusion
So was this worthwhile? ABSOLUTELY!

I learned a lot. For example, I had taken eager evaluation for granted now I had to write code in a language where everything is lazily-evaluated instead. I also didn't realise how pervasive assignments are in my typical code.
I am no expert yet but I intend to be competent enough to write production-grade haskell at some point.
So what's the next language that is worth learning so I can expand my mind? Maybe Erlang.
----
Resources.
* [John Backus's paper](https://www.thocp.net/biographies/papers/backus_turingaward_lecture.pdf)
* [Escape from the Ivory Tower: The Haskell Journey](https://www.youtube.com/watch?v=re96UgMk6GQ)
* [Why functional programming matters](https://www.cs.kent.ac.uk/people/staff/dat/miranda/whyfp90.pdf)
* [Monads for functional programming](https://homepages.inf.ed.ac.uk/wadler/papers/marktoberdorf/baastad.pdf)
* [Comprehending Monads](http://www.cs.ox.ac.uk/jeremy.gibbons/publications/ringads.pdf)
* [Programming in Haskell](https://amzn.to/2AHvDKT)
* [WhyHaskellMatters - shortest haskell tutorial](https://github.com/thma/WhyHaskellMatters)
* [Thinking Functional with Haskell](https://amzn.to/2WG0nV6)
