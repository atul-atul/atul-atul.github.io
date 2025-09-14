---
title: "Integration"
toc: true
date: "2023-11-26"
last_modified_at: 2024-11-07T00:00:01-00:00
tags: 
  - reading
  - "technical-reading"
  - "technology-books"
---

Recently came across the post on Google SRE blog. [Lessons Learned from Twenty Years of Site Reliability Engineering](https://sre.google/resources/practices-and-processes/twenty-years-of-sre-lessons-learned/)

Funny that they don't seem to mention the date on the post. Or I might have missed it. That aside, the thing that struck me most is that they mentioned lesson 5 **"Unit tests alone are not enough - integration testing is also needed"** And they seem to have that learning around 2017. I will try to practise the lesson **Automate your mitigations** but the integration tests lesson resonated.

I have learned and I am ashamed to say I have forgotten this lesson a few times. Mostly I forget it because of teams dynamics. By definition and current style of product development (small teams, incremental development of small stories into features, etc.) you need a common understanding and agreement on integration and integration tests. But you can push others only a little. Ultimately, intrinsic motivation- or lack of it- is much stronger. Even for yourself. Still I try to have integration tests around my code.

* * *

I have had this opinion for a long time. Can't even remember when I formed it. Much before [this tweet](https://twitter.com/KentBeck/status/583284709645406208) from Kent Beck. But he's put it quite well:

`reducing a problem to one or more easier/solved problems is the basic strategy of programming. integration is still painful, though.`

To reduce the pain of integration you should have interfaces (contracts, APIs, versioning, data types, client libraries- Michael Nygard advices against client libraries in Release It) and you should have an agreement on those across teams.

The test pyramid is a good way to think about your tests. With microservices, this is probably seen as anti-pattern and actual test distribution is more of a slice.

But don't worry, if Google can learn the lesson so late- in 2017- then you and your team are not too late. Important to have learnt it, I think.
