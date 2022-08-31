---
title: "Clean as you go"
date: "2019-02-24"
---

A very important thing when writing code or maintaining it is to go on deleting unnecessary pieces of code. Unused code starts becoming clutter and we should always take some time and effort to delete it. Go Marie Kondo over it otherwise it will negatively affect your code's Feng-Shui.

Now, whatever works in production has probably the utmost value. And it should. But unused code left there because it might be needed someday, unused cases, configurations send out a very wrong signal. They tell you that somehow, and you can know how, the code is important. It occupies a corner in your brains. Whenever you are handling a chunk of code, it says road closed and then you have to bypass it. Better try and clean it up. Remember it is there every instant unless you do something about it, when in reality it should not be there for even a single instant. Of course, you need a safety net of tests (unit, integration) around first.

Follow the boy scout rule, and leave the code base cleaner than you found it. Forget if it is your current scope of work or not. Software, which was written even as recently as six months back, has a tendency to acquire and retain garbage, and programmers have a tendency to forget about details of it.

No-one is accountable for it as it is a natural order of things.

The whole thing follows second law of thermodynamics and the system goes on increasing in entropy. Unless you act upon it, disorder will ensue. So clean as you go.

Of course, you have to be careful about it. Add a few tests around it, a few more tests around a bigger scope and then go on cleaning it in small bits at a time. Don't do it in big chunks. Adding tests around surrounding scope- like unit, integration tests- gives you a perspective and better understanding of the behavior. Changes in smaller chunks give you confidence, ability to revert and even better understanding emerges from it.
