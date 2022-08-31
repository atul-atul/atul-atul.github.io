---
title: "Libraries Vs Frameworks"
date: "2019-02-19"
---

If you ask even a junior developer, they will tell you that libraries should be preferred to frameworks.

And yet we end up with frameworks.

The main negative impact frameworks cause is constraining the thought process to match the framework way of doing things. _'This is the way you can do these things and it is so simple.'_ Frameworks will make certain things easier for you, no doubt. But in the process things become difficult. This is particularly visible in the long term. _'What you need here is- simply- a row mapper.'_

I am not against frameworks as such but try to use a framework like a library as far as possible.

Recently, in my company, in a team of 8+ developers, I tried this. We were writing ETL kind of jobs/ applications as per the needs of consumers of our data. (The consumer lines of businesses were reluctant to do that themselves as we are migrating some huge workday/ peoplesoft data to our custom data store and they don't want to own the risks. We, on the other hand, want to push the adoption/migration and are helping them out.)

We like a typical java, spring, lombok, REST, spark shop, created some jobs. We used spring boot and since most of the flows seemed similar, we took template method pattern approach where the differences in the process were to be implemented in template methods. A typical Extract Transform Load workflow case where you fetched data using spark, cleansed it as per your rules, and created the output as per your needs. Some 'Aspect Oriented' functionality like mails, performance monitors, etc.

After we on boarded a couple of lines of businesses (3-4 months), the approach started showing cracks. A few weeks later- after success story was released how those few businesses migrated- a whole bunch of businesses lined up. Sometimes it was somewhat different flow, sometimes the data consumption patterns emerged which we had earlier decided that 'we would never support'.

This is where frameworks fall apart. Anything out of the ordinary seems foreign. We want to adapt it to the framework. I am not saying it cannot be done. But it can become a mess.

So we decided to take simple approach. Deliberately simple. We won't depend on any framework. You wan't to use spring-retry functionality, go ahead. Use it in your job. As A Library. Not as a framework. Limit it to your local module. Hide it. The test, for example, is if I can replace your resttemplate calls with a okhttp library APIs in a sweep? If that is possible, you are good. The framework should not leak beyond your boundaries. And a few boundaries within your boundaries.

We also made some observations like:

Make it simple but don't spend too much effort to make it simpler.

You don't have to define every class as a bean and then autowire it. Just create a new instance in the method and let it be garbage collected later.

Instead of having deep hierarchy of layers (n tiers), compose the functionality in a flattened layers. Keep in mind to separate functionality, don't mix it or couple it tightly. Just make it as atomic as possible so that it can be composed. This approach makes unit testing easier as you don't have to mock a lot.

To the extent possible create fluent interfaces like which can be chained without coupling like unix pipe (Java's limitations for DSL).

Addition and removal of a new functionality anywhere is possible only when the functionalities can be composed.

Create small modules in your project and use those as libraries in your project. If these seem reusable publish it outside your project. Best among the similar sounding libraries will win over time.

It's been a success story.
