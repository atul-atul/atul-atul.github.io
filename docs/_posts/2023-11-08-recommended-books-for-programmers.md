---
title: "Recommended Books For Programmers"
toc: true
date: "2023-11-08"
last_modified_at: 2024-11-07T00:00:01-00:00
tags: 
  - reading
  - "technical-reading"
  - "technology-books"
  - Tech
---

This is another of those Recommended Books For A Programmer posts. Over the years I read a few programming related books. And some of those stayed with me. I have forgotten many and must have forgotten many which should have made it to this list. Also, as I did not formally graduate in Computer Science, you will not see the dragon book or GEB or CLRS on this list. Nor is the list targeted towards interview- so no Grokking Algorithms or Cracking the coding interview on this list. Still, here is an attempt to list a few which I will recommend reading. Only writing about books here. Papers(like Dynamo) or talks(like Are We There Yet?) would make it to a separate post(s).

Let's start with light stuff.

1. **Refactoring by Martin Fowler**  This book has probably had the biggest impact on how I _write_ code. It taught me how to write good code. Not in terms of how to come up with most tight/ succinct code or which data structures to use, etc. The book talks about how to improve the design of existing code and in the process I learned how to write good code so that it will have those attributes of code which make it a good design. It taught me about importance of having safety net of tests around the code I write and also taught me that improving the design is a process. So in a sense, while the book may appear prescriptive, the step by step approach with step 1 (extract method), step 2(rename method), etc. is where the magic lies. A few books exist in this space: Clean Code (far inferior book and series), TDD by example(good one).
2. **Effective Java by Joshua Bloch**  I learned about how to best use features of java and some traps to avoid. But the most important takeaway was about the design of APIs. So far this is the best book on design of APIs I have read. Even if you have read it, read it from that perspective: you want to learn more about designing of APIs rather than java programming best practices. I have [written elsewhere](https://atul-atul.github.io/bylane-learning/) about this lesson.
3. **Design Patterns book by Gang of Four**  This book talks about OO design patterns primarily centered around interfaces. It is dated with examples of window toolkits, etc. And nowadays with functional programming in context, the OO design patterns may seem overkill or too much of structure. But their discussion about interfaces, why, how of a pattern, etc. [stayed with me](https://atul-atul.github.io/a-review-of-GoF/). This book and Refactoring by Fowler taught me the importance of naming things well. For example, if you tell me you will use a builder here and extract a template method there to be implemented by various concrete implementations, I can understand it without much further detailing. Good, intention revealing names give you an abstraction over the concept indicated by those names.
4. **Microservices by Sam Newman**  While the book seems like a lecture series of slides translated to long-form content to be made into a book, the resulting book gave me a lay of the land of sorts of microservices and distributed systems. **Domain Driven Design** by Eric Evans is a recommended read in this area for concepts of bounded contexts, etc.
5. **The Pragmatic Programmer by Dave Thomas and Andy Hunt**  This is a good book to come back to every once in a while and glance through again and again. Better than books like Code Complete, IMO.
6. **Coders at Work and Programmers at Work**  These books are great if you need some sort of looking up to the elders and get motivated.
7. **Java Concurrency In Practice by Brian Goetz**  This book makes you afraid in a way how your code can go wrong in a multithreaded environment. Did not finish this one but would still recommend it.
8. **Rest In Practice by Webber** How to design a good Restful system. Did not finish it but recommended. Everybody designs a good enough REST API. But the APIs could be better than just good enough. 
9. **Javascript the Good Parts by Crockford**  Javascript is not a language anyone would have expected to become as widespread as it is. And this book explains how to use it better. However, this is a very old book and more recently I have heard good feedback about [Eloquent Javascript](https://eloquentjavascript.net/) although I haven't read it. Having the good parts book on this list just to say that you can write good programs in a somewhat badly designed language. Same like pre-java8 you could create a function object (Comparator, for example) and write a somewhat functional code even though java at the time did not support functional programming paradigm.
10. **The Mythical Man Month**  A series of essays. Recommended for a somewhat senior programmer/ lead/ manager.
11. **SICP** by Abelson, Sussman  Did not finish but recommended.
12. **Patterns of Enterprise Application Architecture by Fowler**  An old one. Recommended for the way you can think about Enterprise Application Architecture. And whether you will hear about this book or not, these patterns are everywhere in fairly large systems. They have taken different forms with microservices, big data systems. But you can still learn a lot.
13. **Continuous Delivery by Jez Humble**  The concepts and practices are nowadays so commonplace that [you might already know](https://atul-atul.github.io/continuous-delivery/) many of the things you need to know in the area of delivery, agile, devops, etc. I will also recommend The Phoenix Project but I personally knew the concepts and did not like the book much- but that is because I read it very late.  
      
And now some books that you won't find in many other such lists on the net.

1. **Programming Elixir by Dave Thomas**  A good language and a good book. Similarly I would recommend **Joe Armstrong's Programming Erlang** though I did not finish it. But the concepts are invaluable.
2. **UML Distilled by Fowler**  This is another of those old books about an extinct topic. And another book by Fowler on this list. But quite good.
3. **Release It by Nygard** This is where- even though I say it myself- this post gets better than thousands of other similar posts on the net. This book has been [the best general architecture/ system design book I have read](https://atul-atul.github.io/release-it-book-by-michael-nygard/) in maybe a decade. I should have read it much earlier. Primary goal is to write recovery oriented systems. It talks about stability patterns/ anti-patterns, etc. If you haven't been following similar patterns in your systems, it will make you lose sleep. 
4. **The Art Of Unix Programming by ESR**  This is where you learn the principles like: Write small programs which do one thing and do it well and interact well with other programs, etc.
5. **OO Analysis and Design By Grady Booch**  What is abstraction? What is encapsulation? Learn OO concepts from the masters.
6. **97 Things** series by O'reilly  Programmers, architects, etc. Similar- The Manager's Path, The first 90 days, etc.
7. **The Hidden Life of Trees** Wohlleben  Not programming as such. The author talks about how trees survive, etc. And the book is not science proper. But while reading it I kept thinking about how I could make my software systems similarly adaptive, robust like trees.
8. **Designing Data Intensive Applications** by Kleppmann VERY highly recommended.
