---
title: "Working Around Problems"
toc: true
date: "2023-12-12"
categories: 
  - reading
  - notes
tags: 
  - "distributed-systems"
  - "technical-reading"
  - "technology-books"
  - "thoughts"
---

`"The significant problems we have cannot be solved at the same level of thinking with which we created them."- Einstein`

We generally need a different paradigm of looking at problems if those cannot be solved with our current paradigms.

Also, there may be some problems for which- even if such solutions exist- the solutions could be very costly (cost being in whatever format). In such cases it may be better to work around a problem rather than actually solving the problem. Of course we cannot know this beforehand (unless it can be proven with theory/ maths that no solution exists- NP complete, or say with behavioral patterns in case of people problems). So the idea is to try and find out a solution. But if approaches with our current understanding/ knowledge/ mindset do not work, maybe we can wait a for revolution/ generation change to give us a different mindset, or we can work around the problem to make things work.

I think this may be close to what they call problem vs solution oriented approach. Say, we want to get something done. We encounter a problem. We search for solutions. We cannot solve the problem. So we work around the problem to get that thing done. Because our goal was to not solve the problem but to get that thing done. This working around that problem may bring us to some different obstacles. But if those can be overcome, we are somewhat better off. We may lose some advantages but at least we are not stuck. This- to use a term from medical field- bypass approach can be seen everywhere.

I think we somewhat instinctively know this. Or at least come to learn and accept this. Kids learn this at an early stage. Cannot climb on the bed? Alright, I will first step on- you know what I mean.

So why jot it down here? Well, recently while reading (listening to the audio book\*\*) Designing Data Intensive Applications, I saw this written down sort of formally but in an easy-not-to-notice manner. And that made me think of this. To quote from the book with added highlight (Handling node outages section in chapter on replication):

`There are many things that could potentially go wrong: crashes, power outages, network issues, and more. `**`There is no foolproof way of detecting what has gone wrong, so most systems simply use a timeout`**`: nodes frequently bounce messages back and forth between each other, and if a node doesn’t respond for some period of time—say, 30 seconds—it is assumed to be dead. (If the leader is deliberately taken down for planned maintenance, this doesn’t apply.)`

Just imagine what efforts must have gone in last 30-40 years of computing to arrive at the highlighted part. Professors and researchers must have spent a lot of time in trying to detect what has gone wrong. And maybe in future- with different models and architectures- this can become possible. But at the moment, a timeout kind of approach works better. Of course we may face different problems when we work around a problem. Timeouts may give us problems to solve like- node synchronization, consensus.

* * *

\*\* The DDIA book is somewhat tightly packed with content. So I think a mixed approach would be the best. I purchased a physical copy a couple of years ago. Started reading but left it midway. So purchased the audiobook when I heard that it was narrated nicely. But while listening may be faster, when you are listening, a few things may pass by easily and you may not deliberate on something which you are more likely to do when reading. So, I think, in cases like this, a mixed approach may be better than either. Listen with the text in front of you which you can highlight and take breaks after each section. Currently, I am going with listen-only approach- to get the scope. And when I finish, I will listen again with the physical copy in front of me. At least that's the plan.
