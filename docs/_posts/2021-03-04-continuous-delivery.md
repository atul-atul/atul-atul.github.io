---
title: "Continuous Delivery"
toc: true
date: "2021-03-04"
categories: 
  - "books"
tags: 
  - "book"
  - "continuous-delivery"
  - "notes"
  - "technical-reading"
  - "technology-books"
---

Continuous Delivery by Jez Humble and Dave Farley is now an old book. You probably already follow most of the practices it recommends.

The book and live lesson talk about how to deliver your software product/ service to production continuously. The phrase 'continuous delivery' comes straight from the agile manifesto. So agile and iterative processes form a backbone of the continuous delivery including the value proposition about why you should deliver continuously.

Apart from the usual stuff I knew and practiced, I had a few takeaways.

In the past I have followed and sometimes set up deployment pipelines. The general flows were like this:

Upon commit to develop branch trigger a continuous integration build (CI). The CI build has unit and integration tests.

As soon as this build is green run code analysis build (sonarqube). It checks code quality with static code analysis and code coverage. The idea of not doing this as a part of CI build is to have the CI build as fail fast build. If my changes compile fine and the unit and integration tests (mostly with mocked integrating responses) pass, then we are ready for next steps.

Once sonar build is green we are ready to trigger a snapshot build. This build generates snapshot binaries and pushes those to say your internal artifactory site. This build will be triggered not many times during the day and the snapshots generated may also not always be deployed in the environment. So it may be manual trigger. And it may still depend on CI and sonar builds to be green. So you may run CI builds 50 times a day whereas you may run snapshot build ~ 5 times when you perceive that your binary is ready for release. Devs/ QA can trigger this build.

Deploy the binary in an integration environment where it actually integrates with other services and APIs including third party services like SFTP sites, etc. I recommend having a couple of integration environment (call them SIT, UAT, etc.) Devs can deploy snapshots to SIT. Here all the tests possibly including load and performance tests run. In these environments the service calls are not mocked but they may not connect to production environments of those third party services/ integrating APIs. The build connects to the third party services which are non production environments. Connecting this build to third party production APIs may increase cost. Also there could be concerns of production data being touched from your non production environment. One disadvantage is that their non prod services may not be prod like in terms of response times, data quality, etc. Once SIT build is ok, the dashboard shows it as green along with the build and artifact details.

Next it is QA discretion. The QA will take the build for promotion to UAT and test it there. Maybe they release a new version with same commit so as not to test a snapshot build anymore. Many of these tests will be exploratory and more likely manual. All the automated tests should have already run in previous environments.

Once OK, tag the build/ commit id for release into production. And get it into production. Do not deploy snapshots into production.

A few new learnings from the author:

Have fail-fast builds, first run the tests which are more likely to fail. Some long running tests like load and performance over time tests, other non functional tests should be run on a dedicated environment which branches out from the deployment pipeline. Refactor flaky tests.

Test data management is also important in various environment. No test data could be production quality but it helps if the data is of good quality. You can also run some tests in production. (I have heard this advice often- netflix, etc.- but have not yet had a buy in from stakeholders on any of my projects yet to run tests in production.)

All configuration should be in code repository. This is such an important point. Of course do NOT include secrets, passwords, and API keys in the code repo. But everything else should be in the repo. This may include incremental database scripts, properties and configuration params. The idea is not to have it in some restricted place- for example only OPs team/ DBA know where to look and find.

The deployment should be immutable. That is it should be from a given version of binaries and all the config should be present in the code. And it should be sort of a promote operation (lift and shift) from a lower environment to a higher environment. This makes it reversible if something goes wrong and you need to roll it back to previous stable state. I also think that separation of release responsibility can help avoid such things like an image with snapshot version getting pushed into production.

Manage data incrementally and include all the scripts in the code base. In case of reversal, add incremental steps and scripts to do so.

Components should only talk to each other via service and API calls. (Jef Bezos memo from Steve Yegge's Amazon Vs Google platform rant. A must read.)

People may need slack time as they may be more stressed with CD mode of working.

You cannot do away with Ops.

A tester is a role not a person and test code base needs to managed, maintained and refactored with equal diligence as that of production code.

Pair programming helps. It gives you a wider perspective. Pair program with diverse people- QAs, devs from other modules, juniors, etc.

I personally do not buy much into the idea of BDD but I have seen its utility value. I have written cucumber/gherkin tests and I have seen the confidence they generate in BAs and business folks when they see the test steps in plain english, etc. So if it helps I would go with the author and write those tests by pair programing with someone from QA- ideally BAs and QAs pair program here.

Architecture is something that is difficult to change later on as the project/ product progresses (Martin Fowler). But for continuous delivery, the architecture needs to be evolutionary. (More about this later from Neal Ford, Rebecca Parsons, Pat Kua book.) The architecture undergoes changes and it evolves and does not really have an end state.

Having blue green releases can help reduce downtime to almost zero. Also include feature toggles (a new feature is on or off) but remove them later when they have served their purpose. Do not reuse feature toggles or share them across features. Preferably do not combine two or more feature toggles. This can result in disaster.

I have observed this personally that Conway's Law holds true. You produce software with same communication structure as that of your organization's.

Have small (1 or 2 pizza), independent teams (You write it, you own it).

In any company or project the normal distribution and new technology/ process adoption curve will be there. The adoption of new process technology should be such that late adopters adopt it before new there is new process/ technology on the block.

I have deliberately kept possibly the most important thing for the last. I often say that in software the size matters- smaller the better. So always have small features, incrementally shipped in small deliveries.
