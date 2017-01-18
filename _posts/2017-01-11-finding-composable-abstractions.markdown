---
layout: post
title:  "Finding composable abstractions"
date:   2017-01-11
share: y
disqus: y
---

**Is there a method that reliably leads to composable abstractions?**

I'm well aware that I should focus first on the interface, not on the implemenation. However, I like Eric Normand's approach (see video below) of finding a physical metaphor even before you define the interface and come back to the metaphor when the implementation turns out to be flawed. He says

> "I've never met a good abstraction I couldn't turn into a good metaphor."

Eric uses cut-outs and the movement of the hand to design the rotation and translation of a composable vector graphics system. A good metaphor keeps you grounded while designing the system, so you don't overabstract.

When you want to build composable abstractions you have to start with the composition, not the implementation. Programmers often start the other way around because they want to see some results on their screen. Subsequent refactorings can merely reach a local maximum, not the global one.

> Choice of abstraction matters. There is no way ro refactor Aristotelian into Newtonian Physics.

During implementation revisit your physical metaphor, it contains the answer to defects.

His **process** towards a good abstraction is

1. Find a physical metaphor
  - Metaphors contain answers to questions
  - Keep you grounded while abstracting
  - It is easier to talk about real things that people can agree or disagree with
2. Construction of meaning
  - Define the interface
  - Define the parts and their relationships
  - Avoid corner cases while you can, they are multiplicate when you compose them
  - Focus on composition
3. Implementation
  - Revisit your physical metaphor, it contains the answer
  - Refactoring to achieve meta-properties

<div class='video'>
  <iframe width="560" height="315" src="https://www.youtube.com/embed/jJIUoaIvD20" frameborder="0" allowfullscreen></iframe>
</div>
