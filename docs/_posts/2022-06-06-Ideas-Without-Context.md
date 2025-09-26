---
title: "Ideas Without Contexts"
toc: true
date: "2022-06-06"
last_modified_at: 2022-06-06T00:00:01-00:00
tags:
  - thoughts
  - Tech
---

As we grow, and start thinking on our own- sometimes from ground up- we notice that context matters a lot. I feel this is generally true in life as in software. If we focus on IT here, I would say that a lot of best practices, ideas, tools, opinions, principles* are somewhat bound to contexts and not applicable everywhere. (* Here I am not using the word principles in the sense in which it’s used in *Seven Habits Of Highly Effective People*.)

For example, design patterns make more sense in Object Oriented world. Of course, patterns are everywhere. But those from the Gang Of Four book make more sense in OO.

When such knowledge is imparted though, it is given to us as if it is the ultimate truth. DRY- don’t repeat yourself- we end up trying to avoid duplication at all costs and turn our system complex with shared libraries which don’t evolve well, etc. Knowledge would tend to wisdom with the context firmly inbuilt. Adding some context to DRY would look like: Don’t repeat yourself within a subdomain boundary (in a bounded context).

Many folks discuss such ideas as if they are the ultimate truths. And a kind of dogma sets in. ‘We must follow TDD’, ‘We will follow microservices architecture on this project’, ‘You should autowire/inject dependency instead of creating it with constructor’, ‘Whatever we work on has to be tracked in a task under a story of a feature that is a part of some epic’.

Too often, we accept such ideas, guidelines without much thought as if these are facts. But a simple thought experiment would get us closer to better understanding. For example, agile or TDD may not be very good ideas in early days when a start-up is basically experimenting to understand what sticks, which is a good product/ feature to build. And a lot many people have said that monoliths are much better than microservices if you don’t have competency to handle microservice architecture. Object oriented design patterns may not be good in a functional programming set-up.

Many developers accept such ideas and then modify our problems so that the solution evolves around these ideas.

A reason why people may end up doing this is because there seems to be safety in it. Why write it using a simple library if Spring framework provides it very easily. It is a herd mentality as well. As a result we end up optimizing for unimportant things. In the earlier example, we focus on using spring framework feature than solving the problem. It’s like writing stupid tests to increase code coverage.

I think people do this because they don’t think. There is so much cool-aid out there that it may appear that resistance is futile and we need to drink it. ‘100% code coverage will automatically get rid of many security issues’ (It does not, by the way). But resistance is not futile. And many times even a little bit of extra thought takes us on a much better path. It- this thinking process- is a bit resource consuming. But I think it results in better outcomes.

The idea for this post was in my mind for a month or so. But Vaughn Vernon wrote [this post][Vaughn-Vernon-post] last week, and it sort of nudged me to write.

[Vaughn-Vernon-post]:https://www.linkedin.com/posts/vaughnvernon_dddesign-distributedsystems-microservices-activity-6937040801256411136-9k2U