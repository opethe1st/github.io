---
layout: post
title:  "Ope meet Haskell"
date:   2020-05-10 12:00:25 +0100
categories: [haskell, functional-programming, introduction]
redirect_from: /programming-language/functional-programming/haskell/2020/05/10/ope-meet-haskell.html
---

There are two main styles of programming. Imperative Programming and Declarative Programming. Programs written in the imperative style
describe the exact steps the computer should take i.e the "how" while programs written in the declarative style describe the desired result we want i.e the "what".

Functional programming is a declarative style of programming. It is called functional programming because programs written in the functional style are made up of functions which are combined in different ways to form the main program.

Different languages support different styles of programming. To learn functional programming, I had to learn a language that supports the functional style very well - a functional language. There are a number of functional languages e.g F#, Scala, Haskell, Elm. I decided to learn Haskell because it is reputed to be the "purest" of the functional languages.

<!--description-->
### **Haskell**
Haskell is a typeful, lazily-evaluated functional language.

`typeful` - Haskell is statically typed. This means that every expression in Haskell has a type known and checked at compile time. However, unlike some statically typed languages, you don't need to add types to every expression. Haskell can infer the type of an expression based on how it is used.

`lazily-evaluated` - values in Haskell are not evaluated until they are needed. This means that if you have a function that accepts two arguments and returns the first, the second argument will not be
evaluated. Most programming languages use eager evaluation instead ie. the arguments of a function are evaluated before the function body.

`functional` - function are the primary means of abstraction in Haskell. They are mathematical functions not procedures i.e given the same input, a function will always return the same output
without any side-effects. They can be
passed in as arguments to functions, returned from functions and stored in data structures just like any other value.


### **How I went about learning Haskell**
I started off reading `Learning you a Haskell for great good`. I didn't enjoy it because it didn't teach concepts, it just had a lot of Haskell code and it didn't have exercises either.
In my search for another book, I found this [talk]((https://www.youtube.com/watch?v=Bg9ccYzMbxc&t=4s)) by Chris Allen which I recommend. Instead of picking up his book, I picked up [`A Gentle introduction to Haskell`](https://www.haskell.org/tutorial/haskell-98-tutorial.pdf).

`A Gentle Introduction to Haskell` is the official language tutorial for Haskell 98 - the version of Haskell defined in 1998 - so it is dated but it
was exactly what I was looking for. It explained Haskell concept by concept and it is also short enough that it didn't feel daunting to complete. I also skimmed through a book recommended by the tutorial -
`Introduction to Functional Programming by Richard Bird`. I enjoyed this book because it taught the functional style very well. It also introduced me to the technique of using mathematical laws to derive more efficient equivalent functions.

After going through these three materials, I could write basic Haskell programs but I could tell I still had major gaps in my knowledge. To fix this, I will be going through `Programming in
Haskell by Graham Hutton`.

I also picked up different papers that explained Monads and Lambda calculus. You can find links to them in the resources section at the bottom of this post.

If I had to start all over, I would probably learn from [`Haskell from first principles`](https://haskellbook.com/). I didn't because I thought the book was expensive and too big (I like reading books from cover to cover and it looked daunting to go through over a thousand pages). I like that the authors tested it with beginners to make sure it was effective and my skimming through it makes me believe it is quite good.

### **Haskell concepts that stood out for me**
Learning Haskell exposed me to some of new concepts. I will introduce some of them briefly.

* **Pattern matching** - pattern matching is a way of deconstructing a complex objects into its components. Pattern matching is often used when defining a function with a set of equations. The resulting function definition is very easy to read and if recursive, the base cases stand out. e.g
```haskell
length :: [a] -> Int
-- this is one equation for length for the case of the empty list
length [] = 0  -- match empty list, length is zero
-- this is one equation for length for the case of list with more than one item
length (x:xs) = 1 + length xs -- match head:rest
```
* **Parametric Polymorphism** - This means you can write functions that operate over a type that is parametrized. For example, the length function above makes use of parametric polymorphism since it works with a list that contain values of different types. e.g [Int], [Char], [Bool]
* **Ad-hoc polymorphism via Type classes** - Haskell also suports ad-hoc polymorphism i.e a function with different implementations based on the type of value it is called with. To use ad-hoc polymorphism, you need to add a context to the type declaration of your function. In the function below, you can only call qsort with a list that contains values of type `a` if `a` belongs to the `Ord` type class. For `a` to belong to the Ord type class, it needs to have an instance declaration and implement the functions defined in the Ord type class.
```haskell
-- instances of the Ord typeclass have the ordering operators defined.
-- i.e ==, /= , <= etc
-- Ord a is a context
qsort :: Ord a => [a] => [a]
qsort [] = []
qsort (x:rest) = [v | v <- rest, v <= x] ++ [x] ++ [v | v <- rest, v > x]
```
* **Algebraic Data types** - Algebraic Data types are a way of creating more complex data types from simpler ones using either Sum types or Product types. For example,
```haskell
-- a Maybe type is either Nothing or Just 5 - this is a sum type
data Maybe a = Nothing | Just a
-- a pair contains two types - this is a product type
data Pair a b = Pair a b
```
* **Monadic I/O** - Haskell programs are made up of pure functions with no side-effects. This is a problem because without side-effects a program cannot perform useful work. To solve this problem, Haskell uses the IO monad. A monad is a type constructor plus two functions `return` (aka `unit`) and `bind`(aka `>>=`). Values with type `IO a` are also called actions. Actions can be sequenced together to form a bigger action. The rules for sequencing actions is based on the monadic laws. `return` takes an ordinary value and wraps it in IO. `bind` combines two IO actions so that result of one is the input of the other, the result of `bind` is IO a.

These explanations are very brief but they should give you an idea of what Haskell looks like. You can find a relatively concise introduction to Haskell [here](https://github.com/thma/WhyHaskellMatters)

### **First impressions**

#### **The good**
* Pattern matching + writing a function as a set of equations is awesome
* Type inference is awesome.
* I love that you can identify a pure function from it's type signature and that you have to separate pure functions from impure functions.

#### **The not so good**
* Compiler messages can be scary and tooling is a work in progress. For example, I called fold but flipped the position of two arguments and I got a long stack trace that was not useful in locating the error. To prevent errors like this, I looked for Haskell extensions that would integrate with my editor. I found the HIE extension but I couldn't get it to work locally until I used it with a [devcontainer](https://github.com/hmemcpy/haskell-hie-devcontainer) but then I couldn't install [stack](https://docs.haskellstack.org/en/stable/README/) in the devcontainer. 🤦
* Haskell has a steep learning curve. There is a lot to learn even if you are an experienced programmer but with no experience with a similar language.

### **Conclusion**
So was this worthwhile?

ABSOLUTELY!

I learned a lot. The best part is that learning new concepts forced me to think deeply about what I already knew. I have more things to compare and contrast. e.g Lazy evaluation vs. eager evaluation.

I recommend Haskell if you are looking for intellectual stimulation and ideas that will serve you well even if you keep programming in whatever language you are using right now.

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
