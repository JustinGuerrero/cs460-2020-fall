---
layout: default
published: False
---

# Programming Assignment 2: <br/> The Bridger Canyon Drive Construction Project
{:.text-center .pb-4}

<!--
<div class="subpage-header" markdown="1">
### **{{ site.data.settings.assignments.a1.longname }}**
{:.text-center}
### The Bridger Canyon Drive Construction Project
{:.text-center}
###### ***Checkpoint Due:*** *{{ site.data.settings.assignments.a1.duecheckpoint }}*
{:.text-center}
###### ***Assignment Due:*** *{{ site.data.settings.assignments.a1.duelong }}*
{:.text-center}
</div>
-->

## Objective

The objectives of this assignment are:

* to give you practical experience in using synchronization to solve concurrency problems, and
* to give you (more) practice with C programming.

**Remember our policies around academic honesty: <br/> You may talk with others and discuss ideas and strategies at a high level, but do not look at anyone else's code for this assignment!**
{:.text-center .p-3}

<!--
## Checkpoint

There is a checkpoint for this assignment.

**TODO: UPDATE WITH MORE INTERESTING TASK/QUESTION...** You must upload a video demonstrating that you have can compile and run the `threads02.c` program we looked at in class.

You must meet with me during office hours
(or some other time that we arrange; office hours are preferred, however)
to show me that you have access to a Linux environment where you can compile and run the `threads02.c` program we looked at in class.

If you don't own a machine that runs Linux,
    I recommend setting up a virtual machine on your personal machine using virtual machine software, such as [VirtualBox].

You might also find [Vagrant] to be a useful tool for working with VMs.

If you do use Vagrant, [here is a cleaned-up copy of my Vagrantfile]({{site.data.settings.code}}/Vagrantfile) that you can use.

[VirtualBox]: https://www.virtualbox.org
[Vagrant]: https://www.vagrantup.com/downloads.html
 -->

## Background

For this assignment, you will need to create multiple threads, and to use **locks** and **condition variables** to synchronize those threads.
You'll be using the `pthreads` package.
Please do refer back to code examples and explanations from the lecture notes, relevant [technical resources](/#resources), our demo code, the man pages, etc.

In particular, you may find the [**POSIX Threads Programming**](https://computing.llnl.gov/tutorials/pthreads/) tutorial to be helpful.

**Note:** The `pthreads` locks are "Mesa" style.
*What does that mean?*
Well, it means that the **broadcast** mechanism exists ---
	but a woken waiter merely gets placed back on the ready queue (it does not necessarily run right away).
TL;DR: you better use `while` statements, not `if` statements ;)

If you are curious, this is what Wikipedia has to say on the topic:

> With nonblocking condition variables (also called "Mesa style" condition variables or "signal and continue" condition variables), signaling does not cause the signaling thread to lose occupancy of the monitor.
> Instead the signaled threads are moved to the e queue.
> There is no need for the s queue.  
> ---[https://en.wikipedia.org/wiki/Monitor_(synchronization)](https://en.wikipedia.org/wiki/Monitor_(synchronization))

**Hint:** While we primarily looked at `pthread` locks in class (`pthread_mutex_t`), you are also free to use other constructs provided by the  `pthreads` library.
For example, you may also find `pthread` condition variables (`pthread_cond_t`) to be useful... ;)

## Impure Operations

As we discussed in class, the real world often permits actions that are impure but convenient;
`pthreads` is no exception: it gives you some extra features not present in textbook-pure synchronization primitives.
For example:

* the lock lets you `trylock`;
* the condition variable lets you give a `timedwait`;
* the semaphore lets you `trywait` and `getvalue`;
* you can initialize a mutex lock to be less uptight about being called twice.

**For purposes of this assignment, you should not use such shortcuts.**
Yes, they make life easy in practice, and you have my blessing to use these shortcuts in your research and professional coding
(provided they don't impair portability!).
But at least once, you should use the "pure" primitives.

## ***Your Assignment:*** Simulate the Bridger Canyon Drive Construction Project

Suppose that some road crews need to clear some snow off of Bridger Canyon Drive between Bozeman and Bridger Bowl.
This construction requires closing one lane of traffic, making it a one-way road for a section of the drive.
Traffic may only traverse the single lane road in one direction at a time.
To further complicate matters, the extra snowfall on the road has weakened the road under this section of the drive, limiting its capacity to at most `MAXCARS` vehicles.
(E.g., try `MAXCARS = 3`.)

Write code that simulates this scenario, where each car is simulated with a thread.

## Coding

In your system, each vehicle should be represented by a thread, which executes the following function when it arrives at the point where the road goes to one-lane traffic only:

    OneVehicle(direction) {
        ArriveBridgerOneWay(direction);
        // now the car is on the one-way section!
        OnBridgerOneWay(direction);
        ExitBridgerOneWay(direction);
        // now the car is off
    }

Direction should be `TO_BRIDGER` or `TO_BOZEMAN`.
(You may certainly add other arguments, or collapse this all into a general argument structure, as appropriate.)
`ArriveBridgerOneWay` must not return until it is safe for the car to get on the one-way.
`OnBridgerOneWay` should, as a side-effect, print the state of the one-way and waiting cars, in some nice format, to make it easier to monitor the behavior of the system.
*(So.... watch out for race conditions here, too!)*

## Requirements

Your simulation should try to mimic real-world conditions as best as possible.
Pay close attention to the following requirements, ensuring that your program addresses each of the following points.

#### I/O

**Inputs.**
Your program should accept (at least) two input arguments:
- `NUMCARS` *(the number of cars to create/simulate)* and
- `MAXCARS` *(the capacity of the one-way road)*.

Your program may accept additional, optional arguments.
For example:

```
Usage: ./b2bsim  NUMCARS  MAXCARS  [RANDSEED]  [VERBOSITY]
```

But your program should still run even if only `NUMCARS` and `MAXCARS` is provided.

Specifying these parameters (and potentially others) will make testing your program much easier,
since you won't have to recompile each time you want to tweak an input parameter.

**Outputs.**
At the very least, your program should output major changes to the state of the simulation when they take place.
For example, when a car **arrives**, and when a car **exits**.
It is also a good idea to output the state of the simulation as a whole periodically.
For example, output the state of the simulation whenever a car gets onto the oneway.
In this case, *state* could include:
- the number of cars waiting to go to Bridger,
- the number of cars waiting to go to Bozeman,
- the current direction of traffic on the oneway,
- a counter for the number of cars that have passed so far,
- the number of cars on the oneway road,
- etc.

#### Synchronization

*"May we have a 'bridgekeeper' thread? (i.e., a monitor)"*

No. The car threads must synchronize themselves; you may **not** have an extra thread directing traffic.

#### Safety

Your simulation should always prohibit "bad interleavings" where:

* cars going opposite directions crash on the one-way.
* the one-way section collapses, because too many cars were on it.

#### Liveness

Your simulation should also ensure that:

* if a car gets on the one-way, it will eventually cross and get off.
* if cars are waiting, and the one-way is empty, a car will get on.

#### Efficiency

Your simulation should also make efficient use of the one-way capacity; or, in other words:

* if there are fewer than `MAXCARS` on the one-way section (say, traveling to Bozeman)...
* and they are traveling sufficiently slowly...
* and there is a car waiting to go to Bozeman...
* then that car will get on the one-way too!

To be clear, if `MAXCARS > 1`, but your solution only allows one car at a time on the one-way section, then that is a problem.

#### Testing & Design

Be sure to test your code to try to produce a good sampling of potential interleavings.

Note, however, that your program should also have a principled design;
  testing may show the presence of bugs, but probably cannot assure you of their absence.

To address your testing and design,
    please submit a README that describes your testing efforts and presents some sample output.

#### Handling Race Conditions

Be sure your code does not have dangerous race conditions.

An example of this might be two cars getting on the one-way section at the same time, heading in opposite directions.

#### Handling Starvation

Be sure that no direction is starved.

An example of this starvation: cars already waiting to go to Bozeman never get to go, because cars keep showing up to go to Bridger Bowl, and they never forfeit access to the one-way section.

{% comment %}
## Hints, Suggestions, and Clarifications

1. I recommend adding support for **command line arguments** for your simulator.
For example, if you can set the number of cars to create (`num_cars`), the capacity of the one-way road (`max_cars`), etc.,
it will make testing your program much easier (since you won't have to recompile each time you want to tweak an input parameter).

2. While we primarily looked at `pthread` locks in class (`pthread_mutex_t`), you are also free to use other constructs provided by the  `pthreads` library.
For example, you may find `pthread` condition variables (`pthread_cond_t`) to be useful...

{% endcomment %}

## What To Turn In & Instructions On Submitting your Assignment

Please submit all of your files for this assignment ***as a zipped folder to D2L***.

Your zipped folder should contains ***at least*** the following files:

1. The source code file(s) you've written for your solution.

2. A `Makefile` for compiling your program.

3. A `README.md` file (written in Markdown), which provides a summary of the program and your approach to achieving synchronization amongst the cars (threads).

4. A `TESTING.md` file (written in Markdown), which provides a summary of how you validated the correctness of your solution.
For example, you should document various configurations of `num_cars`, `max_cars`, etc, and how your program handles different inputs.
A bash script that invokes your program with various inputs is recommended, but not required.

## Rubric

{:.table .table-hover .table-striped .table-bordered .table-sm}
| Criteria                                                                     | Points  | Score |
| ---------------------------------------------------------------------------- | ------- |------ |
| You complete the checkpoint on time                                          | 10      |       |
| You have an on-time assignment reflecting good-faith effort                  | 20      |       |
| The code shows good software engineering principles                          | 10      |       | {%comment%}Easy to follow code, appropriate amount of comments, all allocated memory should be freed, etc.{%endcomment%}
| The submission demonstrated a principled approach to synchronization         | 10      |       |
| The submission demonstrated good testing                                     | 10      |       |
| The code showed safety                                                       | 15      |       |
| The code showed liveness                                                     | 15      |       |
| The code made efficient use of the one-way                                   | 10      |       |
| ---------------------------------------------------------------------------- | ------- |------ |
| **Total**                                                                    | **100** |       |
| ---------------------------------------------------------------------------- | ------- |------ |