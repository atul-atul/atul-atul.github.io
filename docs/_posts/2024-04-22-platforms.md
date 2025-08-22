---
title: "Platforms"
toc: true
date: "2024-04-22"
last_modified_at: 2024-11-12T00:00:01-00:00
categories:
  - technology
  - non-tech
  - thinking
  - reading
tags: 
  - "technical-reading"
  - thoughts
---
A little background:
I read "Steve Yegge's platform rant" when it was published. In that post published on Google plus and marked open to public, he talked about how Amazon and Facebook understood the value of platforms better than Google. He removed it a few hours after publishing it and said that it was supposed to be internal to Google only; not public. And later posted how HR (and/ or other people) talked to him about this and tried to understand and then arranged guitar (music) room upon his request (something like that if I remember correctly). I haven't looked for the afterwards but his rant is still available [elsewhere](https://gist.github.com/chitchcock/1281611).

His writing (known as rants, old and very old on medium and blogspot) are/ were some of the best blogs on the net. He recently (a couple of years back) tried something similar on youtube but did not gain much momentum I think. His opinions were always forceful, sometimes off the mark but mostly hitting it. I learned a few things and gained some perspectives by reading those.

A few of his post that have some recall value without looking up are about recruitment, books, kingdom of nouns, and the above mentioned platform rant.

---
I read the platform rant maybe 4-5 times over the years (most recently today before starting this post). It is a must read in my opinion. And it contains the famous Bezos API memo. 

Since the rant was published, Google did get a few things right. Android, maps, GCP, etc. come to mind right away. (Obviously, a few things were already platforms. Steve Y conveniently did not mention android which google had acquired.) Google has always been great at system design. Have you read my post about [bigtable](https://atul-atul.github.io/reading-googles-bigtable-paper/)? But when it comes to platforms Yegge's rant resonates a bit.

That post and the chapter about platforms from the book "Where Good Ideas Come From" have made the importance of platforms pretty much concrete in my mind. Operating systems are more or less platforms by design (unless deliberately closed). But would mobile phones be so smart and successful without application stores? Look at most of the successful companies and you will see a value of platforms. There is more value and monopolizing potential in platforms than corresponding products. But probably platforms take longer term than products to yeild visible value and are a level below the product so products get more attention.

iTunes was a platform. They say that iPod became wildly profitable and Apple got the reach and penetration after iTunes became available on Windows. I am not much into balance sheets but if you are then take a look at Apple's revenue from services... Java Virtual Machine is a platform. Unified Payment Interface (UPI) in India (and outside like Singapore) is a great platform. Just take a look at UPI's wikipedia page and the mind boggling numbers and growth. Uber provides a platform for drivers and patrons to connect and transact with each other. Similarly food delivery apps are basically platforms. These may not be open/ public platforms. But they are platforms. At some point Amazon was the only seller on amazon.com. When their revenues dipped and eBay offered to run Amazon auctions, Amazon started allowing third party sellers and are now a huge platform. Is AWS platform or infra? The question is sort of unnecessary nitpicking. Any service which allows other (third party) users or services to transact is enabling a platform. YouTube is another great platform.

Any product can become a platform. While twitter was designed as API first (meaning it was a platform before its corresponding product), not many of us can deal with this level of abstraction. So we can think of the product first and then about the platform. Even AWS was opened to the world after its use in amazon.com. Plus those who fund the projects need to see product first. And as Yegge mentioned a platform needs a killer app. The Bigtable is a data store (product) and platform as per the original paper. If you extend your product as a platform (which is a difficult thing to do as per the post), then it's likely that your product is the killer app. Will it remain so? Not necessarily. This is why many people used different twitter apps than the official one. 

One of the side effects of designing the product in terms of (externalizable within company/ public) services is that we start placing more importance on defensive design- minimalistic APIs, rate limiting, throttling, security, etc. And integrations by virtue of more mind share become somewhat easier.

---
I was working with a service company and its swiss banking customer. It (the bank) had the best SOA backplane I ever saw in my career. Once we 3-4 people were discussing service orientation with our manager and he said 'Nobody uses SOA anymore. It was hyped a lot but now everybody uses REST resources. Why do you need services?' I sort of understood what he meant and where he was coming from. He was not wrong if he was thinking about SOA products- the likes of IBM ESB and Oracle Fusion. And yet he was so profoundly wrong about services and service orientation. I have no high opinion of myself and honestly I don't think of myself as techie or geek or anything like that. But it is at times like this I can relate to Babbage's exasperation: `'On two occasions I have been asked, 'Pray, Mr. Babbage, if you put into the machine wrong figures, will the right answers come out?' I am not able rightly to apprehend the kind of confusion of ideas that could provoke such a question.'`


