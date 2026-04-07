---
title: "Reading 1988 Book Introduction to Functional Programming"
toc: true
date: "2025-11-07"
last_modified_at: 2025-11-07T00:00:01-00:00
tags: 
  - reading
  - programming
---
Finished reading Introduction to Functional Programming by Richard Bird and Philip Wadler. The first edition. Published in 1988. This book is so old it's not listed even on the authors' university homepages and wikipedia entries. There is a 1998 version using Haskell that is listed. But I hadn't read a programming book in a while, someone on twitter had mentioned this book, PDF was available, so I read it. Here are a few impressions.

The 1988 book by Bird and Wadler covers some basic concepts which are mainstream now and you see those covered elsewhere. But it is fun (and at times irritating) to read formal definitions in a somewhat closer to mathematics than programming style. You know when you define currying, foldright with symbols, etc. I won't consider the book worth while if you have read a functional programming language book. But sometimes changing the perspective (language-maths) helps. In that sense a decent read.

And now for something completely different:

I don't like when people pick a small area in an ecosystem and become Bible-salesman-like fanatic about it. I have heard about no-carb diets, long distance running, global warming just as much as I have heard about haskell and going cloud-native. I had become (~2011-15) quite fanatic about refactoring, agile, TDD, etc. I am not saying that these are bad things. What's bad is being dogmatic about those.

Engineering is not purity. It is pragmatism. However much software craftsmanship, category theory, monads, clojure excite you, ideas and technologies like these are tools to better meet business needs. But there are pockets in IT people gravitate towards- e.g. emacs, agile, unix, etc. Mainly people do so because there is merit in these pockets and they do like [cherry soda](https://www.youtube.com/watch?v=B7JcGYilsuE).

Functional programming has been one such pocket for some time. You can, of course, go back to von Neumann, Alonzo Church, lisp. But in large scheme of things it is not very old if you limit to multi-core processors, delayed release of Java 8 era, etc. People behave missionary like when it comes to pure functions, loops, declarative, referential transparency, tail-call, values/ value objects, pattern-matching, list-comprehensions, lazy evaluations, currying, map, reduce, filter, etc.

I have written functional programs- in java, scala, scheme, clojure, erlang, elixir, JS, and [to some extent even in haskell](https://x.com/effectfully/status/1930035642337014269). And I have consumed a lot of material advocating functional programming. And thought process wise, it does make you think differently. But you have to manage state, integrate with other systems, etc. So it's worth repeating that engineering is pragmatism. Immutability and purely functional data structures are ok but things work better if they are made usable (like in clojure, and later elsewhere). Also, in my opinion static typing works better but then- as far as functional programming with static typing is concerned- there could be some compromises you may have to live with.