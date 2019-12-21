---
layout: post
title:  "How I wrote a JSON parser in Go - Part 4 - Optimization"
date:   2019-12-20 15:49:25 +0100
categories: [Go, Kata, Json, Parser]
---


In the last [post](https://opethe1st.github.io/go/kata/json/parser/2019/12/20/adding-more-features.html), I talked about how I implemented all the features in the JSON specification.
That's nice but I was curious to see how my implementation compared performance wise with the existing standard library implementation.

One of the clues that I had a performance problem was when I tried to compare the time it takes to load two huge strings when my code was at this [state](https://github.com/opethe1st/GoJson/commit/943435f24b71ab954f52d910ed0931816e51ead5).
My implementation took forever to load it and I had no idea why. After a bit of guessing, I still couldn't figure it out so I wrote benchmarks.
Go has builtin support for benchmarks. A benchmark looks like this.

```go
func BenchmarkMyMapOfString(b *testing.B) {
	b.ReportAllocs()
	str, err := ioutil.ReadFile("testdata/map_of_string.json")
	if err != nil {
		panic(err)
	}
	Load(str)
}
```

I wrote the benchmarks so I could use them to test the performance of different features - string parsing, array parsing and object parsing. It turns out my string parsing was taking forever. I looked at the standard libraries implementation to figure out why and after a while of looking I noticed the major difference was that they were using the strconv.Unquote function. I used Unquote to parse strings and suddenly my performance was comparable with the standard libraries implementation.
This also made me realise that my model for how Go works wasn't complete. It appears that if you are going to be doing something like this
```go
s := make([]rune, 0)
for i := range(1000){
    s = append(s, rune('a'))
}
```
it is going to be slow because allocations take time. I certainly wasn't thinking about that before. Once this was fixed, the performance of the two implementations was comparable. My job was done!
The code for this is [here](https://github.com/opethe1st/GoJson/commit/7dd0b3ca48d4dd887e373f6677bbef261a18e688)

Json parser ✅

In the last post, I will be summarising what I learned from this project.

Stay tuned!
