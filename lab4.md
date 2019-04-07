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


## Submission

Follow [submission guide](submission.md) to prepare your lab report and source code for evaluation.
