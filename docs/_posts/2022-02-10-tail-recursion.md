---
title: "Tail Recursion"
toc: true
date: "2022-02-10"
---

Ok. Everyone knows what recursion and tail recursion is. Then why write this post? Well, I have copied the content below from Joe Armstrong's thesis. And he has presented it the same idea 'a little bit' differently.

A function call is tail-recursive if all the last calls in the function body are calls to other functions in the system.

For example, consider the following functions:

p() ->
	 ... 
	q(),
	…
q() -> 
	r(), 
	s().

  
At some point in the execution of p the function q is called. The final function call in q is a call to s. When s returns it returns a value to q, but q does nothing with the value and just returns this value unmodified to p.  
**The call to s at the end of the function q is called a tail-call** and on a traditional stack machine tail-calls can be compiled by merely jumping into the code for s. No return address has to be pushed onto the stack, since the return address on the stack at this point in the execution is correct and the function s will not return to q but to the correct place where q was called from in the body of p.

**A function is tail-recursive if all possible execution paths in the function finish with tail-calls.**

The important thing to note about tail-recursive functions is that they can run in loops without consuming stack space. Such function are often called “iterative functions.” Many functions can be written in either an iterative of non-iterative (recursive) style. To illustrate this, the factorial function can be written in these two different styles.

Firstly the non tail-recursive way:

factorial(0) -> 1;
factorial(N) -> N \* factorial(N-1).

To write this in a tail-recursive manner requires the use of an additional function:

factorial(N) -> factorial\_1(N, 1).
factorial\_1(0, X) -> X;
factorial\_1(N, X) -> factorial\_1(N-1, N\*X).

Many non-tail recursive functions can be made tail-recursive, by introducing an auxiliary function, with an additional argument
