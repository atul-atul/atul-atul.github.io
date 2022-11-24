I recently posted this year’s [Stack Overflow Developer Survey 2022 Results][so-survey-results-2022] on engineering channel of my project/ department. It has some 200+ members developers, architects, scrum masters, QAs, etc- many of them being not so active. A sort of short exchange followed. One of the points was about server side rendering which forked into the discussion about whether TDD is dead. (You understand the chain? Popularity of Phoenix -> LiveView -> SSR -> DHH -> Is TDD Dead?)

As a result I ended up watching [the discussion][tdd-discussion] again after many years. The discussion took place in May-June 2014, I [wrote about it][my-impression-about-orig-post] in August 2014 and now watched the discussion again a couple of weeks back. What are my current impressions?

Well, I think, this time I benefitted more from the discussion. Overall the impression stayed the same. Still- some points worth mentioning:

It was a very civil discussion with respect for views all around. IT is an opinionated world and sometimes the discussions turn to your ideas VERSUS mine and animosity. Somehow they avoided this animosity.

At times, Kent Beck was a little defensive, so was DHH. Kent never answered underlying questions like- does TDD guarantee best design, is it must for a great design, is it the only path to great design, is TDD a silver bullet as it is sometimes made to sound like, etc.

I agree a lot about the process and discipline. I think Kent (with ideas like TDD, Extreme programming- team room, pair programming, etc.) has a process/ engineering approach towards software. As such, the field of software is probably not so fully mature yet to be really an engineering field- for example, if we compare it to say mechanical or electrical engineering. There is a lot that is engineering and there is a lot that is not. So these ideas, processes (like TDD, SRE, DevOps, Pair programming) really help.

DHH comes from an entrepreneur mindset- in this discussion. ‘I have a business problem and I want to solve it. Best practices are there to help me; not to constrain me.’ And his primary viewpoints- if I read between the lines- were about the dogma associated with the process- you must have x% code coverage, you must start with a failing test, (occassionally, when I think about them, I hate dogmatic sounding good-for-nothing-consultants), etc.

Another point which DHH stressed was that the people should focus on Green-Refactor part of Red-Green-Refactor cycle as much as they focus on Red-Green. Everybody agreed.

They agreed that experience helps you in designing better.

Another point was that the TDD is particularly helpful for new developers. Martin Fowler mentioned his ThoughtWorks experience for this. And I agree. I had a grad/ fresher who had a couple of mobile apps with 100K+ downloads before he started his first job, GSoC participation, who moved from my company on to FAANG/Crypto companies. On his last day mail after a 2year stint as a fresher at our company, he particularly mentioned how unit testing was something he learned here that added a lot of value. So I agree. But you are not a bad developer if you don’t follow TDD. Software development is not a gated community. Let’s not make it one. As senior developers, we need to help juniors adopt good practices and not scold them if they don’t and question their competence. It is more about including people into your enlightened-with-experience-and-best-practices group, not to push them away by belittling them.

For experienced folks, the TDD may not be much of a value apart from discipline (but a large part of life is discipline). They can make the code maintainable with unit/ integration, etc which are not necessarily written with test-first approach.

I haven’t seen much test code where mocking was not overused. All three of them agreed that they did not prefer mocking.

A quick search [indicates][hey-code-metrics] that DHH’s Hey codebase (which is recent in timeline) has Code to Test ratio of 1:0.7 which is quite good. So he is all for well tested code; just not so much for dogmatic test-driven code.

Again, thanks DHH for generating this discussion via his post [TDD is dead. Long live testing.][orig-article-TDD-is-dead] If you are a senior developer it is one of the must read articles.

[so-survey-results-2022]:https://survey.stackoverflow.co/2022/
[tdd-discussion]:https://martinfowler.com/articles/is-tdd-dead/ 
[hey-code-metrics]:https://twitter.com/dhh/status/1263493729584742401
[orig-article-TDD-is-dead]:https://dhh.dk/2014/tdd-is-dead-long-live-testing.html
[my-impression-about-orig-post]:https://atul-atul.github.io/2014/08/05/Is-TDD-Dead.html