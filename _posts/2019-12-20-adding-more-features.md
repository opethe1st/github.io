---
layout: post
title:  "How I wrote a JSON parser in Go - Part 3 - More features"
date:   2019-12-20 14:49:25 +0100
categories: [Go, Kata, Json, Parser]
---

In the previous [post](https://opethe1st.github.io/go/kata/json/parser/2019/11/19/refactoring-json-parser.html), I wrote about how I added the iterator abstraction. In this one, I will be discussing how I added new features.
<!--description-->
# Parsing null, true and false
The first thing I noticed was that to parse null, true and false, I could use the same function which I called `loadKeyword`. `loadKeyword` accepts an iterator, the value to parse and the object that should be returned.
The function looks like this

```go
func loadKeyword(iter *iterator, keyword string, value interface{}) interface{} {
	for _, val := range keyword {
		if rune(iter.getCurrent()) != val {
			panic(fmt.Sprintf("There was an error while reading in %s", keyword))
		}
		iter.advance()
	}
	return value
}
```
and these lines were added to the load function.

```go
case iter.getCurrent() == 'n':
    return loadKeyword(iter, "null", nil)
case iter.getCurrent() == 't':
    return loadKeyword(iter, "true", true)
case iter.getCurrent() == 'f':
    return loadKeyword(iter, "false", false)

```
This means we can now parse these strings correctly
```go
obj := Load("{"k1": true,"k2": false,"k3": null}")
obj := Load("true")
obj := Load("false")
obj := Load("null")
```

# Parsing numbers
I decided to support parsing integers first. This just required writing an `isNumber` function and a `loadNumber` function.
The code looks like this
```go
func isNumber(iter *iterator) bool {
	switch iter.getCurrent() {
	case '1', '2', '3', '4', '5', '6', '7', '8', '9':
		return true
	}
	return false
}

func loadNumber(iter *iterator) interface{} {
	num := 0
	for !iter.isEnd() && unicode.IsDigit(rune(iter.getCurrent())) {
		num *= 10
		val, _ := strconv.ParseInt(string(iter.getCurrent()), 10, 64)
		num += int(val)
		iter.advance()
	}
	return num
}
```
For negative numbers, I just checked for the sign at the beginning and then proceeded with parsing the rest of the number and then multiplying by the sign at the end.
For decimals, I added logic for checking for '.' and parsing the fractional part.
For exponents, I added logic for checking of 'e' or 'E' and parsing the exponent.
The code for this can be found [here](https://github.com/opethe1st/GoJson/commit/2be3870e00ddf68ed6e9791f02ba509f56b7a265).

That's all of the features in the JSON specification. 😄.

Notice that to add these new features I didn't have to change already working code and I had tests to prove that it all works like expected.

That's the all, right?

Wrong.

The next step when writing a library is to benchmark it and then optimise. I was curious how my code compared with the standard libraries implementation. This step is the subject of the next blog [post](https://opethe1st.github.io/go/kata/json/parser/2019/12/20/optimization.html)

Stay tuned!
