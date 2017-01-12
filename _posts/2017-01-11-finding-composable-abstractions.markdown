---
layout: post
title:  "Finding composable abstractions"
date:   2017-01-11 22:58:55 +0100
comments: true
---

Is there a method that reliably leads to composable abstractions?

I'm well aware of focusing first on the interface, not on the implemenation. However, I like Eric Normand's [approach][approach] of finding a physical metaphor even before you define the interface and come back to the physical metaphor when the implementation turns out to be flawed. "I've never met a good abstraction I couldn't turn into a good metaphor." he says. When you want to build composable abstractions you have to start with the composition, not the implemenation. Programmers often start the other way around, they want to see some results on their screen.

His process towards good abstractions is

1. Find a physical metaphor
  - Metaphors contain answers to questions
  - Keep you grounded while abstracting
  - It is easier to talk about real things that people can agree or disagree with
2. Construction of meaning
  - Define the interface
  - Define the parts and their relationships
  - Avoid corner cases while you can. Corner cases are multiplicate when you compose them.
  - Focus on composition
3. Implementation
  - Refactoring to achieve meta-properties

[approach]: https://www.youtube.com/watch?v=jJIUoaIvD20
