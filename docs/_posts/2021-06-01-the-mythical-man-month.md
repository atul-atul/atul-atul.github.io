---
title: "The Mythical Man-Month"
toc: true
date: "2021-06-01"
last_modified_at: 2021-06-01T00:00:01-00:00
tags: 
  - books
  - "estimation"
  - "project-management"
  - "software"
  - "technical-papers"
  - "technical-reading"
  - "technology-books"
  - "the-mythical-man-month"
  - Tech
---

My notes from the essay by Fred Brooks.

* * *

TL;DR:

Progress is not a measure of man-month as man and month are not interchangeable. Adding more people to make software delivery faster creates its own burdens. Bearing a child takes nine months, no matter how many women are assigned.

* * *

More projects go awry for lack of time than all other causes combined. Why?

1. Poor estimation techniques. They also assume all will go well.
2. We confuse efforts with progress.
3. Due to uncertainty about our estimates, we give in to pressures about estimates.
4. Schedule progress is poorly maintained in software engineering.
5. On schedule slippage we add more manpower.

In many areas, the physical limitations of the medium place constraint on the ideas. However, programming is more of a thought process and so we assume that the medium is tractable; that there are not many limitations on the medium and are optimistic. But our ideas (as they do not hit the physical world before implementation) are unjustifiably optimistic.

Cost varies as per number of people and time. But progress is not a measure of man-month as man and month are not interchangeable.

Men and months are interchangeable only when we can partition work that requires no communication among many workers. Some sequential tasks cannot be partitioned to run in parallel. Bearing a child takes nine months, no matter how many women are assigned.

When partitioned tasks require communication, it is an added burden due to training and co-ordination (intercommunication). Training (technology, tools, strategy) cannot be partitioned. And intercommunication burden increases as number of people on teams increases n(n-1)/2. So 3 people require 3 time pairwise communication as required by 2. If instead of individuals there are teams and corresponding conferences among them, it adds further burden. So sometimes division of original task can become a bad idea as it incurs these additional burdens.

Testing is usually the most mis-scheduled part of programming. Failure to allow enough time to system testing is particularly disastrous as this delay comes at the end of the schedule.

The number of months of a project depends upon its sequential constraints. The maximum number of men depends upon the number of independent subtasks. From these two quantities one can derive schedule using fewer men and more months. One cannot, however, get workable schedules using more men and fewer months. More software projects have gone awry for lack of calendar time than for all other causes combined.
