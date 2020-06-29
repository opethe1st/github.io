---
layout: post
title:  "How I wrote a JSON parser in Go - Part 2 - Refactoring"
date:   2019-11-19 14:49:25 +0100
tags: [Go, Kata, Json, Parser]
redirect_from: /go/kata/json/parser/2019/11/19/refactoring-json-parser.html
---


In the previous [post](https://opethe1st.github.io/go/kata/json/parser/2019/10/13/json-parser-in-go.html), I talked about how I wrote a minimal viable parser
i.e one that could parse in strings, arrays and objects. It works as advertised but I noticed there were a couple of things I could do to improve on the code before I added more features from the JSON specification.

### Abstractions wanted

So one of the things I noticed was that almost all functions have this input signature - `(s string, current int)`. This was a clue that there was a missing abstraction.
<!--description-->
#### What does s and current represent?
s is the entire string to be parsed and current is the next index of that string that has not yet been parsed. Conceptually, they are related.
For example, it doesn't make sense for current to be a negative number since it represents a position in a string. It also doesn't make sense for it to be greater than the length of s either. So what is the right abstraction for both variables?

My first thought was to create a class called scanner
that will encapsulate these two variables. This is similar to the approach used in the standard library implementation (I skimmed through it briefly after I was done with the MVP)

It looks like this
```golang
type Scanner struct{
    s string
    current int
}

func (s *Scanner) consumeWhiteSpace(){

}

func (s *Scanner) isObjectStart(){

}

func (s *scanner) isObjectEnd(){

}

func (s *scanner) loadObject(){

}
//... other missing methods
```

But I wasn't convinced this was the best solution. It would mean that the parsing function would just be calling methods on an
instance of this class and my design sense felt it was doing too much. It knows about when the current index is at a mapping start
or end (i.e the next thing to be read is a '{' or '}'),. It knows how to return object created from the string and index etc.

After some time thinking, it occurred to me that the right solution was to model s and current as an iterator. The responsibility of an iterator is to support iterating over the items of a particular object.
The code for the iterator looks like this.
```go
package json


type iterator struct{
	s string
	offset int
}

func (iter *iterator) getCurrent() byte {
	return iter.s[iter.offset]
}

func (iter *iterator) advance() {
	iter.offset++
}

func (iter *iterator) isEnd() bool {
	return iter.offset >= len(iter.s)
}
```
The iterator is very lightweight and focused. The invariant that offset shouldn't be greater than the length of s isn't enforced here but it is added later and it was easy to add.
I rewrote all the functions that previously accepted (s string, current int) to accept (iter *iterator) instead and all my tests still passed!


This is the state at this [commit](https://github.com/opethe1st/GoJson/commit/c7e59fb537ee6e05b06ad6638d8ab55c792b0571).

In the next [post](https://opethe1st.github.io/go/kata/json/parser/2019/12/20/adding-more-features.html) I will talk about how I added support for the other features
in the JSON specification. This will show that my approach still works when I implement more features - without having to change what currently works. This is a desirable quality when writing code.
