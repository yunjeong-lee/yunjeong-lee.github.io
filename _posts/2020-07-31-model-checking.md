---
layout: post
title:  "How to make sense of Model Checking"
date:   2020-07-31 14:50:50 +0800
categories: jekyll update
---

I’ve recently started to explore the ideas behind model checking. This post documents my findings through a Q&A format, in hopes of making it easier to follow especially for those who are not familiar with the subject. I start off with basics – _what it does_ and _what it is for_, illustrating the mechanism with simple examples – and answer any questions that came to my mind while thinking it through.

<br/>

#### _**Q1: What is model checking?**_

**A:** When you are designing a program, you want your program to correctly behave based on your implementation design. As the software systems have grown bigger and highly complex, we can no longer rely on manual reasoning to verify programs. Instead, we make use of a more automated mechanism – i.e., model checking – to analyze correct behavior of complex programs. 

That is, model checking is an automated technique that, given a finite-state model of a system and a formal property, exhaustively and systematically checks whether the model satisfies this property, behaving just as the specifications of the property instructed it to. The main goal of model checking is to prove properties of program computations.[^5] In this process, model checkers employ different kinds of algorithms (e.g., partial order reduction, bit-state hashing[^5]) to make the verification runs more effective.

<br/>

#### _**Q2: What are the steps involved in model checking?**_

**A:** The model checking process consists of three different phases: _modeling_, _running_, and _analysis_ phase[^1].

- In the _modeling_ phase, we model the system using the modelling language to express it as finite-state systems and formalize the property using the property specification language.
- The system under consideration is compiled into a state-transition graph (a.k.a. Kripke structure, as shown in Fig 1.[^4] below) _K_, the specification is expressed as a temporal-logic formula _φ_, and the model checker decides whether _K ⊨ φ_, i.e., whether a structure _K_ is a model of the formula _φ_.
    - We may run some simulations of the model as a sanity check during the modeling phase.
    
    <!-- blank space in-between -->    
    <p align="center">
    <img width="320" height="110" src="https://i.postimg.cc/x8t3XVB2/american-austrian-traffic-lights.png">
    </p>
    <center><font size="-1">Fig 1. American and Austrian traffic lights as Kripke structure</font></center>
    <!-- blank space in-between -->


- In the _running_ phase, we run the model checker to assess whether the property holds for a given state in the model.
- Once the model checker is run, there are three possible outcomes in the analysis phase: property being satisfied, violated, or insufficient memory triggering out-of-memory error.
    - If property is satisfied, then check the next property (if any).
    - If it is violated analyze the counterexample generated by the model checker, refine the model or property, and repeat the entire process.
    - If you run out of memory, reduce the size of the model and try it again.
    
    <!-- blank space in-between -->
    <p align="center">
    <img width="320" height="190" src="https://i.postimg.cc/TPBdXyw8/model-checking-methodology.png">
    </p>
    <center><font size="-1">Fig 2. Basic model-checking methodology</font></center>
    <br />

<br/>

#### _**Q3: What are some examples of the model checking tools?**_

**A:** There are quite a few [open-source model checkers](https://en.wikipedia.org/wiki/List_of_model_checking_tools#Overview_of_some_model_checking_tools). They differ by the modelling language (C, Promela, etc.), properties language (LTL, CTL, etc.), performance in verifying properties of different systems, etc. Examples of the model checking tools include [SPIN](http://spinroot.com/spin/whatispin.html), [NuSMV](http://nusmv.fbk.eu/), [UPPAAL](http://www.uppaal.org/), [PRISM](https://www.prismmodelchecker.org/), [PAT](https://pat.comp.nus.edu.sg/), and [TLC](https://en.wikipedia.org/wiki/TLA%2B#Model_checker), among many others.

<br/>

#### _**Q4: What are some examples of programs verified by a model checker?**_

**A:** Let’s first look at the following examples where we use the SPIN model checker, one of the most popular open-source model checking tools used for concurrent and distributed systems. The model is designed in [Promela](https://en.wikipedia.org/wiki/Promela) (short for Process Meta Language) language. <br>

**Example 1.** Verifying non-termination of a traffic light controller[^7]
 
Suppose we have a traffic light controller system where light switches from green to yellow, then to red, and so on, as shown below:

```pomela
mtype = {red, yellow, green};
mtype color = green;
 
active proctype TrafficLightController() {
  do
    ::(color == red) -> color = green;
    ::(color == yellow) -> color = red;
    ::(color == green) -> color = yellow;
  od;
}
```
Once we run SPIN, it results in ‘unreached in proctype’ message as follows: 

```console
unreached in proctype TrafficLightController
trafficlight.pml:16, state 13, "-end-"
(1 of 13 states)
```
This indicates that the proctype never reaches the “end” state, which is what we expected as there is a non-terminating loop in the program.

<br>

**Example 2.** Verifying that traffic lights are never green simultaneously[^6]

Now, let’s say we have the following system of two traffic lights and a controller, and want to verify the safety property that the two lights are never green simultaneously:

```pomela
bit g1=0, g2=0;
 
active proctype TrafficLights()
{
  do
    :: atomic{ (g1==0 && g2==0) -> g1=1; g2=0 }
    :: atomic{ (g1==0 && g2==0) -> g1=0; g2=1 }
    :: atomic{ (g1==1 && g2==0) -> g1=0; g2=0 }
    :: atomic{ (g1==0 && g2==1) -> g1=0; g2=0 }
  od;
}
 
 ltl P1 {[] (! (g1 && g2))}
```
Running SPIN results in the following outcome:

```console
$ spin -a lights_simple.pml
$ gcc -o pan pan.c
$ ./pan -a -n P1 lights_simple.pml
 
(Spin Version 6.5.1 -- 6 June 2020)
     + Partial Order Reduction
 
Full statespace search for:
     never claim           + (P1)
     assertion violations  + (if within scope of claim)
     acceptance   cycles   + (fairness disabled)
     invalid end states    - (disabled by never claim)
 
State-vector 28 byte, depth reached 3, errors: 0
        3 states, stored
        2 states, matched
        5 transitions (= stored+matched)
        0 atomic steps
hash conflicts:         0 (resolved)
```
Note that I have first generated a verifiable source code in pan.c, and then compiled and executed the verifier.

<br/>


**Example 3.** Verifying a mutual exclusion algorithm[^3] [^8]

Suppose we have the following model based on Peterson’s algorithm for mutual exclusion:

```pomela
bool turn;      // indicates whose turn to enter the CS
bool flag[2];   // indicates if a process is ready to enter the CS
 
active [2] proctype user()
{
again:
     flag[_pid] = true;    // _pid: identifier of the process
     turn = _pid;
     (flag[1-_pid] == false || turn == 1-_pid);
 
/* critical section */
 
     flag[_pid] = false;
     goto again:
}
```
We are interested in verifying safety properties that there are only at most two instances with identifiers 0 and 1, and that there is always at most one process entering the critical section. These properties can be verified by adding the assertions and specifying an LTL formula as follows:

```pomela
bool turn, flag[2];
byte ncrit; // counts the number of processes in critical section
 
active [2] proctype user()
{
  assert(_pid == 0 || _pid == 1);
again:
  flag[_pid] = true;
  turn = _pid;
  (flag[1-_pid] == false || turn == 1-_pid);
 
  ncrit++;
  assert(ncrit == 1); /* critical section */
  ncrit--;
 
  flag[_pid] = false;
  goto again:
}

ltl P1 { [] (ncrit <= 1) }
```
Running the SPIN results in the following outcome:

```console
$ spin -a peterson.pml
$ gcc -o pan pan.c
$ ./pan -a -n P1 peterson.pml
 
(Spin Version 6.5.1 -- 6 June 2020)
     + Partial Order Reduction
 
Full statespace search for:
     never claim           + (P1)
     assertion violations  + (if within scope of claim)
     acceptance   cycles   + (fairness disabled)
     invalid end states    - (disabled by never claim)
 
State-vector 36 byte, depth reached 49, errors: 0
       40 states, stored
       27 states, matched
       67 transitions (= stored+matched)
        0 atomic steps
hash conflicts:         0 (resolved)
```

<br/>


#### _**Q5: What are some examples of these properties? How can you classify them?**_

**A:** With the SPIN model checker, we can check deadlocks (as in invalid endstates), correctness of system invariants via user-inserted assertions, unreachable code, LTL formula, and liveness properties through non-progress cycles (livelocks) or acceptance cycles[^2].
 
The properties can be classified into two categories: _safety_ and _liveness_. Informally speaking, safety properties stipulate that “bad things” do not happen during program computations, whereas liveness properties stipulate that “good things” will eventually happen[^5]. _Safety_ properties are verified via _invariants_ such as “x is always less than 5”, or _deadlock freedom_ such as “the system never reaches a state where no actions are possible”. Specifically, SPIN can find a trace leading to the “bad” thing. If there is no such trace, we know that the property holds. On the other hand, _liveness_ properties are verified via _termination_ that “the system will eventually terminate” or response that “if action X occurs then eventually action Y will occur”. SPIN can search for a loop in which the good thing does not happen. If there is no such loop, then we know that the property is satisfied.

<br/>


#### _**Q6: How do we formalize requirements into property specifications?**_

**A:** The requirements are formalized into property specifications via propositional temporal logic, so that we have precise and unambiguous formulation of properties under consideration. As for the SPIN model checker, for example, linear temporal logic (LTL) formulae are used to specify requirements. LTL is a mathematical language for describing linear-time propositions, and its formulae consist of propositional logic and temporal logic operators.

Primary operators are:
- <> (“eventually”): a property will become true at some point in the future, and
- [] (“always”): a property is satisfied now and forever into the future.

For example, the property that the two lights are never green simultaneously, i.e., [] (! (g1 && g2)), from the **Example 2** above can be expressed in LTL as  ▢ ¬(g1∧g2).

Some of the common composite operators include:
- p -> ♢ q          p implies eventually q (response)
- p -> q U r         p implies q until r (precedence)
- ▢♢p               always eventually p (progress)
- ♢▢p               eventually always p (stability)
- ♢p -> ♢q       eventually p implies eventually q (correlation)
 
More details can be found in the reference source[^6].


<br/>

#### _**Q7: What are pros and cons of model checking compared to other comparable techniques?**_

**A:** Other related approaches are testing, abstract interpretation (and other program analyses), and higher-order theorem proving. Comparison of the techniques can be summarized as follows[^4]:


| Testing           | Model Checking | Abstract Interpretation | Theorem Proving       |
| --------------- | ------------------ | -------------------------- | ------------------------ |
| + Fastest and simplest way to detect error      | + Systematic approach to automatic bug detection aimed at model verification       | + Similar to model checking, employs algorithms to prove properties automatically | Compared to model checking, focus is on expressiveness rather than efficiency |
| + Relatively easy to automate | + Applicable at different stages of design process | Built on lattice theory rather than logic | - Typically incorporates manual work, especially for complex systems |
| - [_“Program testing can be used to show the presence of bugs, but is hopelessly inadequate for showing their absence”_](https://en.wikiquote.org/wiki/Edsger_W._Dijkstra#1960s)  | + Well-suited for finding concurrency bugs and proving their absence | Compared to model checking, focus is on efficiency rather than expressiveness, on code rather than models | - Time-consuming and often non-trivial task worthy of a research publication |

<!-- comment: want to decrease the font size and re-format the table above <font size="-1">Description</font> -->

<br/>


#### _**Q8: What is the trend in model checking research?**_

**A:** The handbook of model checking[^4] classifies advances in model checking into two recurrent themes, driving much of the research agenda: the algorithmic challenge and the modelling challenge.

- The _algorithmic challenge_ refers to the problem of designing model-checking algorithms that scale to real-life problems. This is due to the so-called “state-explosion problem”, the combinatorial explosion of states in the Kripke structure. Because state space is not finite in most situations (e.g., unbounded memory, unbounded number of parallel processes), this leads to the modelling challenge.
- The _modelling challenge_ refers to the problem of extending the model-checking framework beyond Kripke structures and temporal logic. We need to extend modelling and specification frameworks to model and specify unbounded iteration and recursion, unbounded concurrency and distribution, unbounded data types, real-time and cyber-physical systems, probabilistic computation, etc. Some extensions maintain decidability, while others sacrifice decidability and maintain the ability to find bugs automatically, systematically, and early in the system design process.

In addition, there has been a recent development in convergence between model checking and other related approaches such as white-box testing techniques, abstract interpretation, and theorem proving.



<br />

<!-- This is <span style="color: red">written in red</span>. -->

------------------
#### References


[^1]: Baier, Christel, Joost-Pieter Katoen, and Kim Guldstrand Larsen. “System Verification.” In Principles of Model Checking, 1–18. Cambridge, MA: MIT Press, 2014.
[^2]: Ben-Ari, Mordechai. “Verification with Temporal Logic.” In Principles of the Spin Model Checker, 68–93. London: Springer, 2008.
[^3]: Chechik, Marsha. “Partial order reduction and Promela/SPIN.” Class lecture, Automated Verification from University of Toronto, Toronto, ON, 2007.
[^4]: Clarke, Edmund M., Thomas A. Henzinger, and Helmut Veith. “Introduction to Model Checking.” In Handbook of Model Checking, 1–22. Springer Publishing Company, 2018.
[^5]: Jhala, Ranjit, and Rupak Majumdar. “Software Model Checking.” ACM Computing Surveys 41, no. 4 (2009): 1–54. https://doi.org/10.1145/1592434.1592438.
[^6]: Murray, Richard M., Nok Wongpiromsarn, and Ufuk Topcu. “Computer Lab 1: Model Checking and Logic Synthesis using Spin.” Class lecture, Specification, Design, and Verification of Networked Control Systems from California Institute of Technology, Pasadena, CA, March 19, 2013.
[^7]: Roychoudhury, Abhik. “Automated Software Validation.” CS5219 Week 6 Resources. Accessed July 18, 2020. https://www.comp.nus.edu.sg/~abhik/5219/old-lesson-plan.html.
[^8]: “Spin Verification Examples and Exercises.” Spin - Formal Verification, November 28, 2004. http://spinroot.com/spin/Man/Exercises.html.

