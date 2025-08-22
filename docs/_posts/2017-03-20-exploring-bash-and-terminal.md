---
title: "Exploring Bash And Terminal"
toc: true
date: "2017-03-20"
last_modified_at: 2017-03-20T00:00:01-00:00
categories:
  - technology
tags: 
  - bash
  - terminal
---

Let me mention that probably because of Mac I have started appreciating unix. Unix was sort of arcane land for me. I used it only when required on projects. And even now I don't use it much. But I liked MacOS.

And posts like [Mastering Bash And Terminal](https://www.blockloop.io/mastering-bash-and-terminal) help. I loved that post. Possibly, I would have liked to read comments. Of course, HN comes to help with [some additional useful tips](https://news.ycombinator.com/item?id=13400350).

The original post mentions following sequence.

```
cd ~/go/
cd ~/tmp/
cd - # <- this puts you back to ~/go/
cd - # <- this puts you back to ~/tmp/
```

Isn't that good?

I did not know that following command will take you to your home directory.

```
cd --
```
