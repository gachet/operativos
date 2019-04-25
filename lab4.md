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
Each process will have an associated priority (as a `uint`) in the range `[0,...,MAXPRIO]`
that will dictate the ready list to which it belongs when in the `RUNNABLE` state.
This means that there are `MAXPRIO+1` possible priorities for each process.
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

The function prototypes are

```c
    int setpriority(int pid, int priority);
    int getpriority(int pid);
```

where priority ranges from `[0,...,MAXPRIO]`. The `setpriority()` system call will have the
effect of setting the priority for an active process with PID `pid` to `priority` and
resetting the *budget* to the default value. Return an error if the values for `pid`
or `priority` are not correct.
This means that the system call is required to enforce bounds checking on the priority value;
you *may not* assume that a user program only passes correct values.

Here is a simple way for a process to *set* its own priority:

```c
  rc = setpriority(getpid(), newPriority);
```

where `rc` is the return code from the system call and `newPriority` is
an integer. If the value for the priority is invalid, leave the original
priority for the process unchanged and return an error. Likewise, the budget for
a process should not be changed unless the priority is correct.

For the `getpriority()` system call, the return value is the priority of the
process or `-1` if the process is not active.

Here is a simple way for a process to `get` its own priority:

```c
  rc = getpriority(getpid());
```
where `rc` is the return value from the system call.


## Periodic Priority Adjustment

We want a policy regarding adjusting priorities periodically. Consider this
scenario:

        There are many processes in the system. One of the processes has the lowest
    priority while the remaining have the default priority. The process execution
    mix is such that the process with the lowest priority never gets to run.

The result is *starvation*, which the scheduler needs to avoid.

To address the problem of starvation, you will implement a promotion strategy.
The strategy is to periodically increase the priority of all active processes by
one priority level. A process may not have a priority value outside of the range
`[0,...,MAXPRIO]`; that is, the priority of all processes in the `RUNNABLE`,
`SLEEPING`, and `RUNNING` states will periodically be adjusted.
You will use the following approach:

1. Add a new field to the `ptable` structure. Make it an unsigned integer (`uint`)
and call it `PromoteAtTime`. The value stored  will be the `ticks` value at which
promotion will occur. This value is the same for all processes so put it in
the `ptable` structure not each process. While there is an overflow issue here,
you will ignore it for this project. You'll need to initialize `PromoteAtTime` to
`ticks + TICKS_TO_PROMOTE` in `userinit()`, because the scheduler will expect
`PromoteAtTime` to be set to some sane value as soon as it starts running.

2. Create a constant definition `#define TICKS_TO_PROMOTE XXX` in `param.h`,
where `XXX` is the number of ticks that will elapse before all the priorities are adjusted.
Each time that the routine `scheduler()` runs, check to see if it is time to adjust priorities.

3. When the value of `ticks` reaches, or exceeds `PromoteAtTime`
(the scheduler does not run on every tick so it is possible for the ticks value to
exceed the `PromoteAtTime` value. Be careful with your algorithm!):

    - Adjust the priority value for active processes to the next higher priority.
    - Change the priority queue for a process as appropriate. Put any adjusted processes on the back of the queue. Do not move processes for which the priority was not adjusted.
    - Set the value for `PromoteAtTime` to `ticks + TICKS_TO_PROMOTE`. 

**Note:** Initially setting the budget to 300 and `TICKS_TO_PROMOTE` to 3000 produces reasonable
behavior your development. You may need to change these values to test all functionality.


## MLFQ Algorithm
Your algorithm is a variation of [this algorithm](https://en.wikipedia.org/wiki/Multilevel_feedback_queue) where you will utilize a time *budget*,
rather than basing promotion on the fraction of a time slice used.

Each time that the MLFQ algorithm runs, the process at the front of the highest
priority non-empty queue will be selected to run. This means that each time the
algorithm looks for a new process, the algorithm must start by checking the
highest priority queue and only checking a lower queue if no higher priority
jobs are available.

Approach:

- Each process is assigned its own *budget*.

- Each priority level has an associated FIFO queue. Each queue is serviced in a round robin fashion.

- A newly created process is inserted at the end (tail) of the highest priority FIFO queue when it is moved from the `EMBRYO` to the `RUNNABLE` state.

- At some point the process reaches the head of the queue and is assigned to a CPU. The system already records the time at which the process entered the CPU in the process structure.

- If the process exits before the time slice expires, it leaves the system.

- When a process is removed from the CPU, the budget is updated according to this formula:
*budget = budget - (time_out - time_in)*. If *budget < 0* then the process will be *demoted* and placed at the tail of the next lower priority queue when it again reaches the `RUNNABLE` state. The budget value will be reset to the default.
If the *budget* is not expired, the process will not be demoted and will be placed at the tail of the appropriate queue when it again reaches the `RUNNABLE` state.

- Periodically a *promotion timer* will expire. The expiration of this timer will cause each process to be *promoted* one priority level. Promoted processes are placed at the tail of the new queue. On promotion, set the process budget to the default value.

## Modified Commands

### `ps` Command

The `ps` command must now report the priority level of each process. This will require that a new `priority` field be added to the `uproc` struct.

```
$ ps

PID     Name        UID        GID     PPID     Prio    Elapsed  CPU       State   Size
1       init         0          0       1       7       1.041    0.030     sleep   12288
2       sh           0          0       1       6       1.035    0.031     sleep   16384
3       ps           0          0       2       7       0.020    0.002     run     49152
```

### ctrl-p Console Command

The priority level of each active process will be displayed.

```
PID     Name        UID        GID     PPID     Prio    Elapsed CPU       State   Size    PCs
1       init        0          0       1        7       267.28  0.030     sleep   12288   80105469 801050a3 80107391 80106483 8010796f 80107755
2       sh          0          0       1        6       267.22  0.031     sleep   16384   80105469 80100a4f 801020f4 80101326 80106643 80106483 8010796f 80107755
```

### ctrl-r Console Command

You will modify the output to include priority and budget as follows:

```
Ready List Processes:

MAXPRIO: (PID_[MAXPRIO,1], B_[MAXPRIO,1]) → (PID_[MAXPRIO,2], B_[MAXPRIO,2]) → . . . → (PID_[MAXPRIO,n], B_[MAXPRIO,n])

MAXPRIO-1: (PID_[MAXPRIO-1,1], B_[MAXPRIO-1,1]) → (PID_[MAXPRIO-1,2], B_[MAXPRIO-1,2]) → . . . → (PID_[MAXPRIO-1,m], B_[MAXPRIO-1,m])

MAXPRIO-2: (PID_[MAXPRIO-2,1], B_[MAXPRIO-2,1]) → (PID_[MAXPRIO-2,2], B_[MAXPRIO-2,2]) → . . . → (PID_[MAXPRIO-2,p], B_[MAXPRIO-2,p])

...

0: (PID_[0,1], B_[0,1]) → (PID_[0,2], B_[0,2]) → . . . → (PID_[0,q], B_[0,q])

```

where `PID_[i,j]` is the `PID` of the `j`th process in the `i`th *ready* list
and `B_[i,j]` is the `budget` of the `j`th process in the `i`th *ready* list.

## Discussion
You may not assume a fixed value for the number of priority queues. Instead, you
will use a `#define` that is visible in both user and kernel mode
(see `MAXPRIO` in `setpriority()` above).
Your scheduling algorithm is **required** to be flexible enough to adapt to wide
variations in the number of queues. For example, if the number of queues is
"1" then the algorithm will behave as a simple round robin scheduler. Your
testing must account for the number of queues being "1", "3", "7", i.e.
`MAXPRIO` values of `0`, `2`, and `6`.

Determining the length of the promotion timer is tricky. There is no one,
obvious value that works best in all cases. You will need to experiment with
different values to see which one works best for you. This process is called
*hand tuning*. In particular, your timer must be long enough so that you
can properly test that your MLFQ algorithm and queue handling works properly.
Your testing **must** show that your demotion and promotion strategies are
working correctly.

You will need at least one test that shows the correct priority level
information for both the `ps` and **ctrl-p** commands.

An example program that creates many children and causes them to periodically
reset their priority is `schedtest.c`. You are free to use or modify this program
as you see fit for testing.

You will have to give some thought as to where to put the logic that updates the
process budget and, when the budget expires, demotes the process and resets its
budget. In order to accurately determine how many ticks of budget time have been
used up, you'll need to do this bookkeeping when the process is transitioning
out of the `RUNNING` state (a process transitioning to the `ZOMBIE` state need not
be demoted as a zombie can never run again).
Check out the xv6 state transition diagram to see what functions correspond to
these transitions. Additionally, you'll need different list management logic for
different transitions, since a running process can transition to multiple
states, each of which corresponds to a different destination list. Because of
this, it is not appropriate to put all of the logic for budget management and
demotion into one function (e.g., `sched()`). Some of this code will be
the same for all transitions, so you are expected to place the common code in a
function; but some of it will be different and realizing this is key in getting
this project right.


## Hints/Suggestions
Test promotion, demotion, and scheduler functions separately. For example, you
would not want processes to be demoted and/or promoted when you're trying to
make sure that your scheduler selects the highest priority process. You can turn
"off" promotion or demotion by setting the relevant constants to a really high
value. An alternate approach would be to develop new system calls for setting
the default budget and promotion timer.

Using the elapsed CPU time feature from Lab 2, you can demonstrate some
features simultaneously by creating several infinitely looping processes at each
priority level and then using **ctrl-p**. Think about the elapsed CPU time of a
starved process.

## Submission

Follow [submission guide](submission.md) to prepare your lab report and source code for evaluation.
