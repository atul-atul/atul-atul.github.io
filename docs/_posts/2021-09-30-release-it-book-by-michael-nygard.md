---
title: "Release It: Book by Michael Nygard"
toc: true
date: "2021-09-30"
last_modified_at: 2021-09-30T00:00:01-00:00
tags: 
  - "architecture"
  - "distributed-systems"
  - "michael-nygard"
  - "release-it"
  - "technical-reading"
  - "technology-books"
---

The book 'Release It' by Michael Nygard is targeted towards Architects, Designer, Programmers of distributed software systems. It has very practical advice. This is one of top five best technical books I read in, say, last five years. It is also probably the book where I underlined a lot of content for later quick look up. I would recommend the book very strongly.

It talks about keeping systems alive, responsive, maintaining system uptime in the first part. In the second part the author discusses good patterns for control, transparency, and availability in production environments. In part three and four we learn about deployments and solving system problems.

The book is filled with such wisdom that I felt afraid that I did not know or use some of these ideas in my career so far. A couple of ideas: design for production, recovery-oriented mindset, 'Every failing system starts with a queue backing up somewhere', 'Security must be baked in. It’s not a seasoning to sprinkle onto your system at the end', 'Change is guaranteed. Survival is not', 'Microservices are a technological solution to an organizational problem.'

There are stability patterns (circuit breakers, timeouts, bulkheads, decoupling middleware, etc.), anti-patterns (client libraries, unbalanced capacities, dogpile, etc.)

In design for production he talks about networks, NICs, physical, virtual machines, containers, logging, transparency, DNS, load balancing, session stickiness, control plane, some OWASP items.

When talking about thing related to deployment the author says: Making deployments faster and more routine has an immediate financial benefit. More than that, though, a virtuous cycle kicks in that gives you new superpowers. Best of all, you can stop wasting human potential on jobs that should be scripts.

He talks about immutable infrastructure, CI, CD, build pipeline, RDBMS migration changes, version management, contract tests, etc.

In Systemic Problems section the author talks about blue-green releases, canary releases, platform team evolutionary architecture, information architecture, URL dualism, chaos engineering, etc. A self-sufficient two-pizza team also means each team member has to cover more than one discipline. The two-pizza team is about reducing external dependencies.

One area I think could have been done better is that the sections could have had numbers (e.g. 5.2.1) instead of differences in font sizes.

This is one of those architecture books that I will visit again and again. While reading I often thought this book will help me in my engineering lead role in a very similar way Martin Fowler's Refactoring helped in my junior programmer role.

Here is a link to the author's talk that relates to a small section (stability patterns and anti-patterns): [https://www.youtube.com/watch?v=VZePNGQojfA](https://www.youtube.com/watch?v=VZePNGQojfA)
