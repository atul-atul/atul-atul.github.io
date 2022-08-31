---
title: "A review of GoF Deign Patterns Book"
date: "2017-03-20"
tags: 
  - "book"
  - "design-patterns"
  - "gof-design-patterns"
  - "review"
  - "technical-reading"
  - "technology-books"
---

Nowadays, in the world of functional programming, object oriented design patterns are probably not fashionable. But here is a short review of Design Patterns book written by the Gang Of Four.

I started reading this book, left it because I felt bored, tried some other design pattern books and then returned to the GoF book. And it was an eye opener seeing what depth and width this book covered compared to other design pattern books. I understood why everyone in the industry so praised this book and why everyone said understanding and using patterns was difficult. It was like reading about OOP from Grady Booch's book after reading from some learn OOP in 21 hours book.

Nowadays, when I need a quick reference about a pattern, I go to the patterns wikipedia entry and if I need to understand it better and in depth, I go to this book. Every paragraph talks about some core concept (like what is an object's type, really). And page after page there are some core concepts being discussed, so the book demands attention, but it is more than worth the effort.

I have read criticism from Lisp people like Paul Graham about the necessity of patterns. I have tried my hand at lisp. So I understand the criticism to a certain extent. But I think that criticism by functional programmers is mainly directed at OOP. When you are using OOP, the patterns are great help. And this book explains patterns so fundamentally. The book with examples of Window tool-kits, etc. certainly appears to be speaking from the past. But the book is so well written that I could not think of single page where I did not find some fundamental idea. For example, in the quoted section below the authors are talking about object's interface. It is a perfect, complete and yet very concise discussion. **As a challenge give it a try and see if you can remove a sentence or two from the discussion.**

> Every operation declared by an object specifies the operation's name, the objects it takes as parameters, and the operation's return value.This is known as the operation's signature.The set of all signatures defined by an object's operations is called the interface to the object. An object's interface characterizes the complete set of requests that can be sent to the object. Any request that matches a signature in the object's interface may be sent to the object.
> 
> A type is a name used to denote a particular interface. We speak of an object as having the type "Window" if it accepts all requests for the operations defined in the interface named "Window."
> 
> An object may have many types, and widely different objects can share a type.Part of an object's interface may be characterized by one type, and other parts by other types.Two objects of the same type need only share parts of their interfaces.
> 
> Interfaces can contain other interfaces as subsets. We say that a type is a subtype of another if its interface contains the interface of its supertype. Often we speak of a subtype inheriting the interface of its supertype.
> 
> Interfaces are fundamental in object-oriented systems.
> 
> Objects are known only through their interfaces.There is no way to know any thing about an object or to ask it to do anything without going through its interface. An object's interface says nothing about its implementation-different objects are free to implement requests differently.
