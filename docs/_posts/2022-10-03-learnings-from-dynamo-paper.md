---
title: "Learnings from Dynamo Paper"
toc: true
date: "2022-10-03"
last_modified_at: 2022-10-03T00:00:01-00:00
tags: 
  - reading
  - "distributed-systems"
  - "dynamo-paper"
  - "technical-papers"
  - "technical-reading"
  - "technology-books"
  - Tech
---

A while back I read [the dynamo paper](https://github.com/papers-we-love/papers-we-love/blob/master/datastores/dynamo-amazons-highly-available-key-value-store.pdf). Dynamo DB has now progressed beyond that paper. If interested: [https://youtu.be/yvBR71D0nAQ](https://youtu.be/yvBR71D0nAQ)

I learned about consistent hashing, vector clocks, quorum, etc. I learned about the choice of trade-offs to live with (letting go of consistency in preference to availability by using sloppy quorum, etc.) Here is a table from the paper describing the techniques used.

![Dynamo- techniques used](/images/dynamo_techniques.png "Dynamo- techniques used and their advantages")

I knew some of these techniques. But of course, combining these to create a system was an eye opener. This was probably the best learning from the paper. No wonder this is a highly respected paper in the system architecture space. Next on the reading list are papers about Bigtable, Spanner and Cassandra. The concepts are out there. You pick some of these and create a fairly complex and reliable system.

I think this is a level of maturity in any field. Can you separate an idea from the context surrounding it and can you create a meaningful context by employing one or more ideas? Before I started typing the last statement it had taken a form in my mind. But as I began typing, I knew where I learned the thought expressed there. Here it is- taken from SICP:

> The acts of the mind, wherein it exerts its power over simple ideas, are chiefly these three: 1. Combining several simple ideas into one compound one, and thus all complex ideas are made. 2. The second is bringing two ideas, whether simple or complex, together, and setting them by one another so as to take a view of them at once, without uniting them into one, by which it gets all its ideas of relations. 3. The third is separating them from all other ideas that accompany them in their real existence: this is called abstraction, and thus all its general ideas are made.
> 
> John Locke, _An Essay Concerning Human Understanding_ (1690)

I had dynamo paper on my reading list for a while. But after reading it I was so impressed that I put the paper as a recommended reading in my project's engineering forum (slack like channel) maybe a month or so back. I have no way of knowing whether anyone actually read the paper.

Now, bear with me while I explain the background about another sideways/ lateral takeaway I owe to the dynamo paper. Something I called [bylane learning](https://atul-atul.github.io/bylane-learning/) in an earlier post. In a much smaller team (7 developers- freshers to 10yrs experience), we regularly have technical discussion sessions. I started these sessions with the agenda to increase the ratio of technical discussion to the total discussion within the team. Mostly, on the early agenda were the prescriptive books like Refactoring, Effective Java, Pragmatic Programmer, Domain Driven Design, etc. Now, with team dynamics taking its own path, the team wanted to know more about microservices, etc. So, I listed a few topics related to distributed systems (some 15-20 items from CAP/PACELC theorem to SAGA pattern). Disclaimer: I don't know these in detail; but know more than my team does. And when discussing load balancing, proxy, caching, etc. we came to the concepts of consistent hashing, vector clocks. There an idea came to my mind and instead of talking about those ideas, I asked three people to study- in a week- about consistent hashing (fresher), vector clock(fresher- because she mentioned she'd studied Lamport clocks during her undergrad course), quorum and sloppy quorum(3 year experience developer). I asked them to read about these without saying that these were some of the techniques used in Dynamo DB. After 10 days or so, they presented these concepts before the team for 15-20 minutes each. Good enough explanations. The icing on the cake was that three year experienced person had copied the content from Martin Kleppmann's Designing Data Intensive Application. On a side note, this is another level of maturity I think where you know how to quickly reach good sources about a new topic you know nothing about (sort of separating signal from noise). When I told the team that the reason I asked them to study about these topics is that these were relevant for distributed systems and also that these were the techniques used in Dynamo architecture, they were pleasantly surprised. When I showed them the table pasted above, and said that they have covered much of the paper, there was a sort of awed silence. The point I wanted to make was that sometimes people have a sort of phobia about reading papers or anything they think is difficult/ obscure. But the phobia is really out of place because it is not that difficult to read a paper. Again the technique is to divide and conquer. But there was a learning for me personally thanks to the dynamo paper. You should not set a goal (here, reading a paper) which looks not-achievable. Instead you should set incremental goals to reach the desired state. I have read about SMART goals many times before. We divide the product vision into backlog, features, stories, etc. So we do take divide and conquer approach. For my own learning as well, I first studied the disparate topics and then after a while read the paper. But I was able to do the same for the team: setting a goal about which they did not have a vision (and explaining the vision would have given rise to defeatist attitude, apathy, uninterestedness) and yet achieve it. Maybe I am reading too much into it. But it felt nice.
