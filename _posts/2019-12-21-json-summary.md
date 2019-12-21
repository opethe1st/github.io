---
layout: post
title:  "How I wrote a JSON parser in Go - Part 5 - Summary"
date:   2019-12-21 14:49:25 +0100
categories: [Go, Kata, Json, Parser]
---

So what were the main lessons I learned from this?

# Software engineering lessons

I feel like I have a solid grasp on how to write libraries. My process is: -
* start by designing the public interface
* write a MVP with the core features
* refactor your code - if there are missing abstractions add them, if you took shortcuts fix them
* add more features and refactor again if necessary
* optimize space and time

I also learned that a domain driven design mindset is very helpful. It meant that I wanted my code to correspond with the specification as much as possible. It also meant that I was using the same names as in the specification.
This is very important.


# Other lessons
* Done is better than perfect. It took me too long to write about this project. I should have focused on getting it done and moving on.
* I learned that next time I have a project like this, I need to set a time limit so it doesn't drag on forever. It is hard to get back to things when I have context switched to other things.
* I need to make sure I incorporate writing about the project into my process of working on it. My current idea for enforcing this, is to make sure that when I make a PR in the description I write the blog post there and then have an action that published it when it is merged.
