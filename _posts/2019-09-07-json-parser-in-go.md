---
layout: post
title:  "How I wrote a JSON parser in Go - Part 1"
date:   2019-09-29 14:49:25 +0100
categories: [Go, Kata, Json, Parser]
---

# Introduction
Hey! Welcome to my first blog post on how I wrote a JSON parser. To follow along properly, some programming experience is helpful but my goal is to make this as easy to follow as possible. If you don't know what JSON is, the best resource on it is [this](https://www.json.org). It has really lovely diagrams that clearly show what's allowed in JSON.

The posts reconstruct the process through which I wrote the parser. It's not a perfect reconstruction because although I have a commit history that I have referenced and some original notes, some time has passed between writing the code and writing this post. The lesson for next time is to write some code and then write a post about it as soon as possible.

My initial reason for writing a JSON parser was that I was not happy with the error reporting in the standard python library and I thought I could do better.

That's not the major reason I wrote this though. I am all about mastery and one of the vital steps in gaining mastery is deliberate practise. This is an exercise in deliberate practise of software engineering. The goal is to try out ideas that I have and others I have come across and put them to the test in a real project. The other goal was to practise Go with a real project. The desired end result is insights that would help me in writing software.


# MVP
To start, I decided to focus on a supporting a subset of JSON grammar. The goal here is to build a foundation upon which I can progressively support more features without having to modify what already works. The features I decided were the foundation for every thing else were
* being able to parse strings without escapes
* being able to parse arrays
* being able to parse objects

## Parsing
The first thing to write was the signature for the parser. I called this `Load` because I was influenced by the python json's libraries naming. The other choices are Parser, Unmarshaller but to be honest when I first wrote this I wasn't thinking too much about naming so I went with what popped in my head first.

```go
func Load(s string) interface{} {
    return nil
}
```

Next, was to write a dispatch mechanism that handles the different types of objects that can be represented in JSON.

```go
func Load(s string) interface{} {
    switch {
    case isString(s):
        return loadString(s)
    case isArray(s):
        return loadArray(s)
    case isObject(s):
        return loadObject(s)
    }
    default:
        panic("unable to parse this string")
}
```

#### Go specific notes:
> In Go, public functions are functions that start with a capital letter. interface{} is the empty interface that matches anything. If you are alarmed by the use of a name like 's', this is consistent with naming used in Go code that uses shortnames often as long as the scope is small. I used to be anti-shortforms but I can see the point - calling this jsonString instead doesn't result in more clarity.

This is written so that when we need to support new types like numbers and literals (null, true, false) we just need to add another case statement and we don't need to modify what already works. I could also have used a map to match check functions (isString, isArray, isObject) to the load functions (loadstring, loadArray, loadObject) but the switch statement works well enough.


### String Parsing
The first step to parsing a string is to check that the string we are given represents a string. The nice thing about json is that if a string starts with a `"` then it's a string. So the isString(s) function is simply
```go
func isString(s) bool {
    return s[0] == '"'
}
```
The loadString function
```go
func loadString(s string) (int, interface{}) {
    current := 0
    start := current
	current++
	for current < len(s) && s[current] != '"' {
		current++
	}
	// start+1 so we don't include the '""
	// and current + 1 since the next not visited character is at current + 1
	return current + 1, s[start+1 : current]
}
```

### Array Parsing
Array parsing happens when Load is given a string that represent an array. Load knows it has a string when the isArray function returns true.
The isArray function looks like this
```go
func isArray(s) bool {
    return s[0] == '['
}
```
My first version of loadArray function looked like this
```go
func loadArray(s string) []interface{} {
    array := make([]interface{}, 0)
    current := 0
	current++ // move past '['
	var item interface{}
	for (current < len(s)) && (s[current] != ']') {
		item = Load(s)
        if s[current] == ',' {
            current++ // move past ','
        }
		array = append(array, item)
	}
	return array
}
```
but this doesn't work. It works for loading `[]`, and `["abc"]` but once you try to load `["abc", "def"]`. it doesn't work like expected.

The problem here is that there is no memory of what we have parsed in the string so far. So this line - `item = Load(s)` just keeps picking up the first item.

This highlighed a fundamental flaw in the current program structure. To fix it, I added a current parameter to the functions whose value is the current position in the string and made the loading functions return in addition to what was loaded, the current position. This decision affects all the functions so I had to change them to look like this.
```go
func Load(s) interface{} {
    value, _ = load(s, 0)
    return value
}

func load(s string, current int) (interface{}, int) {
    switch {
    case isString(s, current):
        return loadString(s, current)
    case isArray(s, current):
        return loadArray(s, current)
    case isObject(s, current):
        return loadObject(s, current)
    }
    default:
        panic("unable to parse this string")
}

func isString(s string, current int) {
    return s[current] == '"'
}

func loadString(s string, current int) (string, int) {
    start := current
	current++ // move past the first '"'
	for current < len(s) && s[current] != '"' {
		current++
	}
	// start+1 so we don't include the  begginning '"'
    // and current + 1 since the next not visited character is at current + 1
    current++ // move past '"'
	return s[start+1 : current-1], current
}

func isArray(s string, current int) bool {
    return s[current] == '['
}

func loadArray(s string) ([]interface{}, int) {
    array := make([]interface{}, 0)
    current := 0
	current++ // move past '['
	var item interface{}
	for (current < len(s)) && (s[current] != ']') {
		item, current = load(s, current)
        if s[current] == ',' {
            current++ // move past ','
        }
		array = append(array, item)
    }
    current++ // move past ']'
	return array, current
}

```
This works for
s = `["abc","def","gef"]`
and the also for nested arrays like
s = `["abc",["def",[]]]`
### Object Parsing
Parsing objects is very similar to parsing arrays. The code looks like this
```go
func isObject(s string, current int) {
    return s[current] == '{'
}

func loadObject(s string, current int) (map[string]interface{}, int) {
    object := make(map[string]interface{}, 0)
    current := 0
	current++ // move past '{'
	var item interface{}
	for (current < len(s)) && (s[current] != '}') {
        key, current = loadString(s, current)
        current++ // move past the ':'
        value, current = load(s, current)
        object[key] = value
        if s[current] == ',' {
            current++ // move past ','
        }
    }
    current++ // move past '}'
	return object, current
}
```

And that's it for the MVP!

It is missing a couple of features but it is a solid foundation for further work. To support numbers, I just need to write an isNumber and loadNumber functions and add them to the case statement just like the rest. When I need to support string escaping, the only thing that needs to change is the loadString function. This code is also very optimistic, it doesn't check for error conditions but that will be handled later.

This is roughly the state at this [commit](https://github.com/opethe1st/GoJson/commit/e0dc214e84a8e40d4607b7cf7b0a115b8222ff11).
Note that the naming in the commit is a bit different. For example, loadSequence instead of loadArray, this was influenced by previously reading the YAML specification which use the term sequence for array. Same thing for mapping instead of object. Naming is important and I have learned to stick to the names used in a specification so this was eventually fixed in later commits.

I haven't included the support for whitespace feature that is in that commit yet but I will add it in the next post which is going to be about how I refactored this code.

So stay tuned!
