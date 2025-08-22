---
title: "Out of the tar pit"
toc: true
date: "2020-08-03"
last_modified_at: 2020-08-03T00:00:01-00:00
categories:
  - technology
  - reading
tags:
  - "technical-papers"
  - "technical-reading"
  - reading
---

Paper: [https://github.com/papers-we-love/papers-we-love/blob/master/design/out-of-the-tar-pit.pdf](https://github.com/papers-we-love/papers-we-love/blob/master/design/out-of-the-tar-pit.pdf)

**TL;DR:** Software systems are complex due to state, and the order in which things happen (flow control). Some complexity in a system is inherent- it is essential from perspective of users (user requirements- essential complexity); whereas some other complexity arises because of the way programs are built (accidental complexity). Accidental complexity (state and control) should be minimized. Thinking about (essential/accidental ) \* (state/control) can help. Ideally, there would not be any essential control as it may not be mentioned in any user requirements view of the system. The rest of the things can be managed by separating from each other. For this Functional-Relational approach suits better than OOP or other approaches. It helps separate the state and as functional programs are referentially transparent, the flow control is not significant. Also, the programs can be tested better.

* * *

“There are two ways of constructing a software design: One way is to make it so simple that there are obviously no deficiencies and the other way is to make it so complicated that there are no obvious deficiencies. The first method is far more difficult.” Hoare

The major contributor to complexity in many systems is the handling of **state.**

“No Silver Bullet” Brooks: four properties of software systems which make building software hard: Complexity, Conformity, Changeability and Invisibility.

Of these Complexity is the only significant one.

#### Approaches to understanding a system  
Testing and informal reasoning.

Testing: examining from outside. Inadequate... to show the presence of bugs but never to show their absence.

Informal reasoning: Observe and reason about the system from inside. Looking at the code, for example. Formal reasoning using documentation can be often inadequate or misguiding if the documentation is not up-to-date with the system.

Both the above approaches are required for a better understanding of the system.

* * *

### **Causes of Complexity:**

1. **Complexity caused by State:**
    1. State and Tests: A test(system/ unit) in one state tells you nothing at all about the system in a different state. The common approach to testing a stateful system (either at the component or system levels) is to start it up such that it is in some kind of “clean”or “initial” (albeit mostly hidden) state, perform the desired tests using the test inputs and then assume that the system would perform the same way- regardless of its hidden internal state- every time the test is run with those inputs. Often, for large systems even though the number of possible inputs may be very large, the number of possible states the system can be in is often even larger
    2. State And Informal Reasoning: The mental processes which are used to do this informal reasoning often revolve around a case-by-case mental simulation of behavior: “if this variable is in this state, then this will happen — which is correct — otherwise that will happen — which is also correct”. As the number of states/ possible scenarios grows, the effectiveness of this mental approach buckles. Also, when reasoning about a system we need to consider contamination. Any stateless part of the system which interacts with a stateful part of the system can be contaminated.
2. **Complexity caused by Control:** Control is about the order in which things happen.
    1. The difficulty is that when flow control is an implicit part of the language, then every single piece of program must be understood in that context. The compiler works on removing ordering to the extent possible which is double loss: programmer writes statements in order even though she is primarily interested in statements (assignments, relationship between two variables, etc.) rather than ordering of those statements, and then compiler take efforts to reorder those statements- first an artificial ordering is being imposed, and then further work is done to remove it.
    2. Concurrency considerations also impact flow control a lot... and subsequently testing or informal reasoning.
3. **Complexity caused by Code Volume:** Not as serious as this is somewhat easier to control. And the effect can be reduced if above two causes of complexity- state and control- are effectively managed.
4. **Other causes of complexity:** Duplication, dead code, unnecessary abstraction, missed abstraction, poor modularity, poor documentation, etc. All of these other causes come down to the following three (inter-related) principles:
    1. Complexity breeds complexity: Duplication is a prime ex-ample of this — if (due to state, control or code volume) it is not clear that functionality already exists, or it is too complex to understand whether what exists does exactly what is required, there will be a strong tendency to duplicate.
    2. Simplicity is Hard: As it requires significant efforts, simplicity can only be attained if it is recognised, sought and prized.
    3. Power corrupts: The more powerful a language (i.e. the more that is possible within the language), the harder it is to understand systems constructed in it. For example, we need to be very wary of any language that even permits state, regardless of how much it discourages its use.

* * *

### **Classical approaches to managing complexity: OOP (Imperative), Functional, and Logic programming**

1. **Object Oriented Programming.** An object is seen as consisting of some state together with a set of procedures for accessing and manipulating that state. One problem with this is that, if multiple clients access or manipulate the state, then there may be several places where a given constraint must be enforced. (For example, inheritance can let clients to modify state via different access point in the hierarchy.)
    1. In OOP, each object is seen as being a uniquely identifiable entity regardless of its attributes. This is known as intensional identity (in contrast with extensional identity in which things are considered the same if their attributes are the same, as in case of value objects). Object identity does make sense when the state is mutable. The intrinsic concept of object identity stems directly from the use of state, and is (being part of the paradigm itself) unavoidable. This concept of identity adds complexity to the task of reasoning about systems developed in the OOP style. The bottom line is that all forms of OOP rely on state (contained within objects) and in general all behaviour is affected by this state.
    2. OOP offers sequential control flow, shared-state-concurrency which make it complex. Alternatively, actor style message passing mechanism in OOP reduces complexity but is not wide-spread.
2. **Functional Programming**. Based on lambda calculus of Alonzo Church which focuses on statelessness as against OOP which is based on von Neumann architecture which focuses on state modification. Some language are purely functional (Haskell), while some are not (ML, Erlang).
    1. The primary strength of pure functional programming is that by avoiding state (and side-effects) the entire system gains the property of **referential transparency**\- which implies that when supplied with a given set of arguments a function will always return exactly the same result (it will always behave in the same way). Everything which can possibly affect the result is always immediately visible in the actual parameters. The absence of state and side effects makes testing more effective although testing for one set of inputs says nothing about behaviour with another set of inputs.
    2. Control is still problematic with functional approach. But the complexity is reduced as the control is more abstract (fold/ map) rather than explicit iteration in the program. In that sense functional programming languages have less powerful control (which is good as mentioned above).
    3. Concurrency complexity is also reduced due to lack of mutable state management.
    4. To address requirement of mutable state, functional languages make use an extra parameters to functions. This extra parameter is a state (for example, a tuple of previous and current fibonacci numbers to derive next number in the series). Effectively, the mutable state that was hidden inside a corresponding OO method(or some outside shared mutable variable) is replaced by an extra parameter on both the input and output of the pure referentially transparent function. However, the fact is that we are using functional values to simulate state, and if this extra parameter were a collection (compound value) of some kind then _in extreme cases_ it could be used to simulate an arbitrarily large set of mutable variables and ease of reasoning is lost. But while a **single pool global variables** to manage the state has to be reasoned with and understood; the state is not hidden. However, keeping state hidden(encapsulated) in the modules of object oriented approach makes things look simple. With FP approach with extra parameters, the state is visible, but it is visible everywhere and you have to have extra parameter everywhere (function as well as its clients, which is not the case with OOP). However, this adding extra parameter effort is one time (writing the program) but pays huge benefits on an ongoing basis. A program will be written once but it is read, understood, and tested many times during its life.
3. **Logic Programming.** Together with functional programming, logic programming is considered to be a declarative style of programming because the emphasis is on specifying _what_ needs to be done rather than exactly _how_ to do it. Pure logic programming is the approach of doing nothing more than making statements about the problem (and desired solutions). This is done by stating a set of axioms which describe the problem and the attributes required of something for it to be considered a solution. The ideal of logic programming is that there should be an infrastructure which can take the raw axioms and use them to find or check solutions. All solutions are formal logical consequences of the axioms supplied, and “running” the system is equivalent to the construction of a formal proof of each solution.
    1. There are two parts of the program axioms(logical) and operational. A pure set of _logical_ axioms(i.e. assertions about the problem domain), or _operational-_ as a sequence of commands which are applied (in a particular order) to determine whether a goal can be deduced (from the axioms). This adds complexity- it is necessary to be concerned with the operational interpretation of the program whilst writing the axioms. Prolog is a logic programming language where the operational part can be modified/ customized to add new axioms which compromises referential transparency.
    2. Control- while logical programming does not mention specifics of program control, the languages- prolog- _specify_ the control. So it is not abstract and falls short of ideal/pure logic programming. It is implementation specific. Plus, combined with the _operational_ part it becomes difficult to reason.

### Accidents and Essence

1. **Essential Complexity** is inherent in, and the essence of, the problem (as seen by the **users**). The complexity with which the team will _have to_ be concerned, even in the ideal world.
2. **Accidental Complexity** is all the rest — complexity with which the development team would not have to deal in the ideal world (e.g. complexity arising from performance issues and from suboptimal language and infrastructure).

If there is any possible way that the team could produce a system that the _users_ will consider correct without having to be concerned with a given type of complexity, then that complexity is not essential. But as not all possible ways are practical, any real development will need to contend with some accidental complexity.

If the user doesn’t even know what something is (e.g. a thread pool or a loop counter) then it cannot possibly be essential complexity.

* * *

### **Recommended General Approach**

In an ideal scenario, we would like to formally specify the requirement without missing on any relevant requirements and keep this to just enough to meet all requirements. We would like to specify what steps- (what, not how)- we would take on the underlying infrastructure. This is closer to declarative style of programming.

#### Ideal World

1. **State in the ideal world:** Data is either input or derived. And it is either mutable (user wants to modify as per requirements) or immutable. All the data in the user requirements is _essential_ but not may not be relevant to the state. Any other data is whether mutable or immutable is accidental state of the system. In the _ideal_ world, all accidental state should be eliminated.
    1. The input data if not required for further use in the program is there to have some side effects. The input data that is required in future becomes the essential state.
    2. Derived data:
        1. Essential Derived data- Immutable: can always be re-derived (from the input data- i.e. from the essential state) whenever required. As a result we do not need to store it in the ideal world (we just re-derive it when it is required) and it is clearly _accidental_ _state_.
        2. Essential Derived data- Mutable: Again accidental state as this can be re-derived on demand.
        3. Accidental derived data- like data maintained in cache form accidental state and should ideally be eliminated.
2. **Control in the ideal world:** While some state is essential, control generally can be completely omitted from the ideal world and as such is considered entirely accidental. It typically won’t be mentioned in the informal requirements and hence should not appear in the formal requirements (because these are derived with no view to execution). Of course, some control will be needed for the program to run. But the results of the system should be independent of the actual control mechanism which is used.

#### Theoretical and Practical Limitations

An ideal world is similar in many ways to the vision of declarative programming that lies behind functional and logic programming. Unfortunately, functional and logic programming ultimately have to confront both state and control.

Problems with formal specs: Formal requirements/specs derive directly from the informal requirements. In the ideal world we would like to be able to execute the formal requirements without first having to translate them into some other language. Formal specs could be property based or model (state) based. Property based is focused on what rather than how whereas state based focuses on how a (generally stateful) model of system should behave.

In Model based approach, there are limitations on executing formal spec, because say an expression like ¬∃y|f(y, x) may not be executable from formal specs alone and we may need some language, operational component (domain specific language) which can execute such parts of specs. Such operational components, if kept separate will have state- but it will be limited to those components and would be comparatively less complex.

Property based approaches do have capability of isolating and executing such expressions.

Sometimes getting rid of accidental state will give rise to lesser **ease of expression**. One possible situation of this kind is for derived data which is dependent upon both user inputs over time, and its own previous values. In such cases it can be advantageous to maintain the accidental state even in the ideal world. An example of this would be the derived data representing the position state of a computer-controlled opponent in an interactive game- it is at all times derivable by a function of both all prior user movements and the initial starting positions, but this is not the way it is most naturally.

#### Required Accidental Complexity

Performance and ease of expression may _require_ us to maintain state and/or control, and thus accidental complexity.

The accidental complexity should be **avoided** and when required, as in case of performance or ease of expression consideration, the accidental complexity should be separated.

To mitigate the risks we encounter due to accidental complexity, we take two defensive measures.

#### Risks of explicit management of accidental state (the majority of state). 
The recommendation here is that we completely _**avoid explicit management**_ of the accidental state. Instead

1. declare what accidental state should be used, and
2. maintain it on a completely _separate_ infrastructure.

This is reasonable because the infrastructure can make use of the (separate) system logic which specifies how accidental data must be derived. This eliminates any risk of state inconsistency. Indeed, we can effectively forget that the accidental state even exists. More specific examples of this approach are given in the second half of this paper.

Even when ease of expression makes us have required accidental state, we should separate it. If separated it can be treated as if it is another user input and thus essential.

How/ What to separate?

1. Split logic from state
2. within state, split accidental from essential

As there is no essential control, when we separate based on above guidelines, we get Essential logic, Essential State, Useful Accidental State, Useful Accidental Control. Separate languages- domain specific- can be used for these separated components. State related components do not require their language to have control structures like loop, etc.

One implication of this overall structure is that the system (essential + accidental but useful) should still function completely correctly if the “accidental but useful” bits are removed (leaving only the two essential components)- albeit possibly unacceptably slowly. “The logic component determines the meaning . . . whereas the control component only affects its efficiency”.

The vital importance of separation comes simply from the fact that it is separation that allows us to “restrict the power” of each of the components independently.

#### Interactions among the separated components

1. **Essential State** This can be seen as the foundation of the system. The specification of the required state is completely self-contained- it can make no reference to either of the other parts which must be specified. One implication of this is that changes to the essential state specification itself may require changes in both the other specifications, but changes in either of the other specifications may never require changes to the specification of essential state.
2. **Essential Logic** This is in some ways the “heart” of the system- it expresses what is sometimes termed the “business” logic. This logic expresses- in terms of the state- what must be true. It does not say anything about how, when, or why the state might change dynamically- indeed it wouldn’t make sense for the logic to be able to change the state in any way. Changes to the essential state specification may require changes to the logic specification, and changes to the logic specification may re-quire changes to the specification for accidental state and control. The logic specification will make no reference to any part of the accidental specification. Changes in the accidental specification can hence never require any change to the essential logic.
3. **Accidental State and Control** This (by virtue of its accidental nature) is conceptually the least important part of the system. Changes to it can never affect the other specifications (because neither of them make any reference to any part of it), but changes to either of the others may require changes here.

Together the goals of avoid and separate give us reason to hope that we may well be able to retain much of the simplicity of the ideal world in the real one.

It is this separation which allows us to restrict the power of each individual component, and it is this use of restricted languages which is vital in making the overall system easier to comprehend (power corrupts).

* * *

### The Relational Model

Cod's relational model has an elegant approach to **structure** the data, **manipulate** it, maintain its **integrity**, and has **data independence** by means of clear separation between logical and physical model. These characteristics are applicable to all state and data and are not limited to relational data/ DBMS.

1. **Structure**: In Cod's model structure is achieved via relations. **Relation**: A relation is best seen as a homogeneous set of records, each record itself consisting of a heterogeneous set of uniquely named attributes. A relation can contain no duplicates, and it has no ordering. Both of these restrictions are in contrast with the common usage of the word _table_ which can obviously contain duplicate rows (and column names), and- by virtue of being a visual entity on a page- inevitably has both an ordering of its rows and of its columns. Relations can be of the following kinds: **Base Relations** are those which are stored directly and **Derived Relations** (also known as _Views_) are those which are defined in terms of other relations (base or derived). A relation can be considered as being a single (albeit compound) _value_, and any mutable state should be considered not as a“mutable relation” but rather as a variable which at any time can contain a particular relation value. (_snapshot_ from Rich Hickey's [Are We There Yet talk](https://www.infoq.com/presentations/Are-We-There-Yet-Rich-Hickey/)?). The relation should be maintained path independent. So finding employees in a department should be as easy as finding department from an employee. Maintaining JPA like ORDBMS values in employee and department is both slow and likely to cause redundancy, maintainability issues.
2. **Manipulation:** Set/relation operations- Select(restrict), project(new relation), union, product, intersection, difference, join, divide, etc.
3. **Integrity:** is maintained by constraints. Any infrastructure implementing the relational model must ensure that these constraints always hold- specifically attempts to modify the state which would result in violation of the constraints must be either rejected outright or restricted to operate within the bounds of the constraints.
4. **Data Independence**: Separating logical model and physical storage is close parallel to accidental/ essential split.
5. **Extensions** like grouping, aggregating, etc.

* * *

### Functional Relational Programming

The _essential_ components of the system (the logic and the essential state) are based upon functional programming and the relational model. All essential _state_ takes the form of _relations_, and the _essential logic_ is expressed using relational algebra extended with (pure) user-defined functions- the functions that are specific to system at hand, rather than infrastructure.

1. **Architecture:** An FRP system has by virtue of separation, specs separately for
    1. Essential State: Relational definition of the stateful components of the system
        1. Has to be input by user i.e. essential
        2. All state is solely stored in relations
        3. Consist of names and types of relations and values of those relations at runtime
    2. Essential Logic: Derived-relation definitions, integrity constraints and (pure) functions
        1. Consists of two parts: functional and algebraic (relational)
        2. Relational part is main part and functional part is user defined pure functions.
        3. This should uphold integrity constraints at all times (for example, by means of normalization).
        4. No compromise should be made in these Logic components for the sake of performance.
    3. Accidental State and Control: A declarative specification of a set of performance optimizations for the system. Consists of hints on how to achieve the results including considerations for aspects like performance.
        1. State: What should be stored, and what storage mechanisms(ex. index) should be used.
        2. Control: parallel, eager/lazy, etc.
    4. Other: A specification of the required interfaces to the outside world (user and system interfaces). All input must be converted into relational assignments, and all output (and side-effects) must be driven from changes to the values of relvars.
        1. Feeders are components which convert input into relational assignments- i.e. cause changes to the essential state. When an attempt is made to modify the state, the infrastructure should apply integrity constraints and accept or reject the attempted state manipulation.
        2. Observers are components which generate output in response to changes in the state. (ex. database triggers)
        3. Infrastructure: An FRP system is the specification (the four components above), whereas the infrastructure is what is needed to execute this specification. It provides capabilities for each of the above four parts to fulfill its expectations. For example, infra for state should provide types (such as int, boolean, etc) to update the relations (assignment functionality), infra for logic would provide ability to write and execute user defined functions, etc.

The paper then goes on to state the benefits of this architectural approach. And an example of such a system follows.
