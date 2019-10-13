---
layout: post
title:  "How I wrote a JSON parser in Go - Part 1"
date:   2019-09-29 14:49:25 +0100
categories: [Go, Kata, Json, Parser]
---

# Introduction
Hey! Welcome to my first blog post on how I wrote a JSON [parser](https://en.wikipedia.org/wiki/Parsing).

In the posts in this series, I will reconstruct the process through which I wrote it. It's not a perfect reconstruction because although I have a commit [history](https://en.wikipedia.org/wiki/Commit_(version_control)) and some original notes, some time has passed between writing the code and writing these posts. Despite this, I believe it is useful to document not just the end result(i.e working code) but the process (my thoughts, rationale for decisions, order in which I wrote that code etc.) that produced it.

That's the main takeway I want readers to leave with.

My major reason for writing this is to deliberately practise software engineering. By this I mean, trying out software engineering ideas (my ideas and ideas I have come across) in a project and taking note of what works and what doesn't. The other reason was to practise writing Go and practise writing.


# JSON
JSON stands for JavaScript Object Notation. It is a data-interchange format that supports 6 data types.
* string - starts with a `"` and ends with a `"` plus support for escapes e.g `"string"`, `"key"`
* number - represents a number e.g `2.4`, `2.0`, `12`
* boolean - this has only two possible values - `true` and `false`
* null - it is represented by `null`
* array - this contains values of other json types in order e.g `["abc", 123, true]`
* object - this contains key and value pairs (the key must be a string and a value can be any type supported by json) e.g `{"key": 123, "k2": "string"}

for more information, you can check [here](https://www.json.org)

# MVP
For the MVP, I decided to focus on a supporting a subset of JSON grammar. This subset was decided based on what I thought would be a good foundation upon which I could add more features without having to change code that already works.
Those features were
* being able to parse strings without escapes
* being able to parse arrays
* being able to parse objects

## Parsing
The first thing I did was to design the [public interface](https://en.wikipedia.org/wiki/Public_interface) of the parser. This interface has just one function - `Load`. I called this `Load` because I was influenced by python's json library's naming (I didn't realise it at the time).
Other names I could have used are Parser and Unmarshaller but to be honest I just went with what popped in my head first. (I eventually changed this name). The other thing was to decide what the function needed to accept and what it needs to return. In this case, it is pretty clear - it receives a string that should conform to the json grammar and returns the object that the string represents according to the JSON grammar.

```go
func Load(s string) interface{} {
    return nil
}
```

Next, I wrote a switch statement that calls the right load function based on the type of [value](https://www.json.org/) represented by the string passed to the Load function.

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
> In Go, public functions are functions that start with a capital letter. interface{} is the empty interface that accepts a value of any type. If you are alarmed by the use of a name like 's', this is consistent with naming used in a lot of Go code.  Go programmers use shortnames often as long as the scope where it is used is small. I used to be anti-shortnames but I am more comfortable with it now.

The nice thing about how this code is currently structured is that when we need to support new types, I just need to add another case statement without modifying what already works. I could also have written this such that I had a map from the check functions (functions prefixed with `is` and return a bool that indicates if a string represents a given type) to the corresponding load functions (functions prefixed with `load` that actually return the value represented by the string) but the switch statement works well enough right now.


### String Parsing
The first step to parsing a string is to check that the string passed into Load represents a string. It's a string if the first character is a quote so isString is simply this -
```go
func isString(s) bool {
    return s[0] == '"'
}
```
The loadString function looks like this
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
#### Go specific notes:
> note that loadString starts with a lowercase letter, in Go this means that the function is private which mean can't be used outside the package in which it was defined. This is a good thing because it minimizes what you expose to clients of this package which means that you have more freedom to change implementation details like this function.

### Array Parsing
An array is parsed when Load is called with a string argument that represent an array. A string that represents an array starts with `[`. This is checked by the isArray function.

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
		item = Load(s) // load the json value
        if s[current] == ',' {
            current++ // move past ','
        }
		array = append(array, item)
	}
	return array
}
```
but this doesn't work for all cases. It works for loading `[]`, and `["abc"]` but once you try to load `["abc", "def"]`. it doesn't work like expected. Can you figure out what the problem is?


The problem here is that there is no memory of what we have parsed in the string so far. So this line - `item = Load(s)` just keeps picking up the first item.

This is a fundamental design flaw. To fix it, I had to add a current parameter to the all check and load functions. The value of current is the next index of the string that hasn't be parsed. This is a major decision and so I had to modify all the functions to look like the snippet below. Note that the public interface (ie. Load's [function signature](https://en.wikipedia.org/wiki/Type_signature)) didn't need to change at all.

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
note that it doesn't support whitespace. so this s = `["abc", "def"]` doesn't work.
Yay!

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
This works for
s = `{"key":"value","k1":"value2"}`
and when s represents a nested object
s = `{"k1":["abc","cde"],"k2":{"k1":"v2"}}`


And that's it for the MVP!

This is roughly the state at this [commit](https://github.com/opethe1st/GoJson/commit/e0dc214e84a8e40d4607b7cf7b0a115b8222ff11).
Note that the naming in that commit is a bit different. For example, loadSequence instead of loadArray, this was influenced by previously reading the YAML specification which uses the term sequence for array and mapping for object. Naming is important and I have learned to stick to use consistent names. In this case, I want the names to be consistent with the JSON specification. So this was eventually fixed in later commits. That commit also has the support for handling whitespace which I would be discussed in later posts in this series

The next post is going to be about how I refactored this MVP. For example, you would notice that I am passing in `s, current` into all the functions except `Load` is there a missing abstraction that captures what those two variables mean? Also some of the naming is not quite right yet e.g current and Load.

So stay tuned!

-------
Thanks Vanessa, Adaobi and Memuna for the reviews.
