# Lab 4: MLFQ Scheduler

[Lab 4 Rubric](lab4-rubric.md)

## Academic Honesty

Aside from the narrow exception for collaboration on homework, all work submitted in this course must be your own. Cheating and plagiarism will not be tolerated. If you have any questions about a specific case, please ask me.


## Prerequisites
You **must** have all of Lab 2 finished, except for the `time` command. You **must** also have completed the *ready*, *running* and *sleep* list implementation from Lab 3, this includes making sure that the relevant invariants of these lists are enforced.


## Introduction

In the previous lab, you rewrote the old `scheduler()` to use the ready
list when searching for a `RUNNABLE` process instead of iterating through the
process array. Your rewritten scheduler is still functionally equivalent to the
old one: both are simple, round-robin schedulers, but the new one changes the
complexity from `O(n)` to `O(1)`. However, fairness is not addressed.

In this project, you will implement a new "fairer" scheduler for xv6. You will
become familiar with:

- Implementing an MLFQ scheduler for xv6.
- Implementing a new system call for setting process priority.
- Implementing a new system call for getting process priority.

### Reading

The reading for this project is chapter 5 from the [xv6 Book](xv6-book-rev11.pdf).


## Preparation

Clone a following repository https://github.com/wildart/xv6-public.git and checkout branch **lab4**.

    > git clone https://github.com/wildart/xv6-public.git xv6-labs
    > cd xv6-labs
    > git checkout lab4

If you already cloned this repository, you might want to pull recent changes
by using `git pull` command.

In this project you set a `LAB_NUMBER` Makefile flag to value `4` for proper
conditional compilation.

Do **not** use the `ptable.proc[]` structure when implementing this lab except
in `getprocs()`, `procdump()`, and *free* list initialization; use your lists instead.


## Overview
In class, you learned about several approaches to scheduling processes. Each has
its strengths and weaknesses. One approach was called "Muli-Level Feedback Queue" or MLFQ.
You will implement a variation that uses a slightly different approach
for preventing process starvation.

The approach that you will implement utilizes both "demotion", based on a
*budget*, for fairness and "periodic promotion" for starvation prevention.


## Process Priority
Each process will have an associated priority (as a `uint`) in the range `[0, MAXPRIO]`
that will dictate the ready list to which it belongs when in the `RUNNABLE` state.
This means that there are `MAXPRIO+1`possible priorities for each process.
Upon allocation, each process will have the same initial (default) priority value,
the highest priority. The process priority value may be changed programatically
during process execution via the `setpriority()` system call.

The highest priority in the system will be `MAXPRIO` with each value less than
`MAXPRIO` being a successively lower priority, the lowest being `0`. That is,

- ∀ Pᵢ ∈ {0, 1, 2, ..., MAXPRIO}

- P(MAXPRIO) > P(MAXPRIO-1) > ... > P(0)

**Note:** Because the priority field is an unsigned integer `uint` there cannot
be a negative priority.

### Modifying the *Ready* List
You will need to change your declaration of the ready list in `struct ptable` to the following:

```c
  #define statecount NELEM(states)

  static struct {
    struct spinlock lock;
    struct proc proc[NPROC];
    struct ptrs list[statecount];
    struct ptrs ready[MAXPRIO+1];
    uint PromoteAtTime;
  } ptable;
```

where each index of the *ready* list corresponds to a priority queue in the MLFQ.
You will need to modify your *ready* list code (including `initProcessLists`)
to now work with process priority. Observe that your invariant for the *ready*
list will now change to the following:

- the *ready* lists contain all of the `RUNNABLE` processes in the system with each process in `ptable.ready[i]` having a priority of `i`, `0 ≤ i ≤ MAXPRIO`.

**Note:** Be careful when modifying the *ready* list insertion code in `kill()`,
especially if you are using the old routine.


## The `setpriority()` and `getpriority()` System Calls



## Submission

Follow [submission guide](submission.md) to prepare your lab report and source code for evaluation.
