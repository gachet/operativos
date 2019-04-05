# Lab 3

[Lab 3 Rubric](lab3-rubric.md)

## Academic Honesty

Aside from the narrow exception for collaboration on homework, all work submitted in this course must be your own. Cheating and plagiarism will not be tolerated. If you have any questions about a specific case, please ask me.

## Introduction

This project will modernize process management in xv6. All process management will be impacted. New console control sequences will be implemented to support testing and debugging of the new process management facility.

Increasing the efficiency of process management will be the principle focus of this project. The current mechanisms are inefficient in that they traverse a single array `ptable.proc[]` that contains all processes, regardless of state. In this project, you will:

- Implement a new approach to process management with improved efficiency.The new approach will encompass
    - Process scheduling
    - Process allocation and deallocation.
    - Transactional semantics for all process state transitions. 
- Expand the console debug facility with new control sequences.
- Learn to implement complex concurrency control in the xv6 kernel.

**Note:** We define **transactional semantics** to be that a transaction executes atomically with respect to other code. The failure of such a transaction will leave the relevant part of the system in the *same state* as before the transaction.

In this project you set a `LAB_NUMBER` Makefile flag to value `3` for conditional proper compilation.

## Preparation

Clone a following repository https://github.com/wildart/xv6-public.git and checkout branch **lab3**.

    > git clone https://github.com/wildart/xv6-public.git xv6-labs
    > cd xv6-labs
    > git checkout lab3

If you already cloned this repository, you might want to pull recent changes
by using `git pull` command.

## New Process Lists

You will add `RUNNABLE`, `UNUSED`, `SLEEPING`, `ZOMBIE`, `RUNNING`, and
`EMBRYO` lists to the current approach. Each list corresponds to a state of
the [xv6 state transition model](xv6-state-transition-model.pdf).
Adding these lists to xv6 **will not** require that you abandon the current array
of processes that is created at boot time, but you will *hide* its use. 

Instead, you will add a new array `struct ptrs list` to store these lists in
`struct ptable`, and add a new field `struct proc *next` to the `struct proc` in
`proc.h` that points to the next process for whatever list a process is a member.

You will have `statecount` (6) new pointers in `struct list`:
`list[RUNNABLE].head`, `list[UNUSED].head`, `list[SLEEPING].head`,
`list[ZOMBIE].head`, `list[RUNNING].head`, and `list[EMBRYO].head`
to point to the first process, or head, of each list, or `NULL` if the list is
empty.

You will also have six new pointers that point to the tail of each list:
`list[RUNNABLE].tail`, `list[UNUSED].tail`, `list[SLEEPING].tail`,
`list[ZOMBIE].tail`, `list[RUNNING].tail`, and `list[EMBRYO].tail`.

You will use these pointers to more efficiently traverse the existing process
array when looking for processes in each of the six possible states. 
*You must take into account new concurrency requirements when handling these new
lists.*

Once implemented, every process will be on one of these six lists. Moreover,
except as noted below, the code in `proc.c` will no longer reference the
`ptable.proc[]` array.

The current process table, `ptable`, is defined at the beginning of `proc.c`:
```c
struct {
  struct spinlock lock;
  struct proc proc[NPROC];
} ptable;
```

You will add this new structure just above the `ptable` definition:
```c
#ifdef LAB3
struct ptrs {
  struct proc* head;
  struct proc* tail;
};
#endif
```
This just packages up the head and tail pointers in a `struct`, so we can put them in an array.

Next, add this constant definition,
```c
#ifdef LAB3
#define statecount NELEM(states)
#endif
```

change the `ptable` structure to include our new array of `struct ptrs`,

```c
struct {
  struct spinlock lock;
  struct proc proc[NPROC];
#ifdef LAB3
  struct ptrs list[statecount];
#endif
} ptable;
```

and change the definition of `struct proc` in `proc.h` to have a
pointer, called `next`, to the next item in each list. Each list corresponds to
a *state* of the [xv6 state transition model](xv6-state-transition-model.pdf).

To give an example of how these lists fit together, `ptable.list[RUNNING].head`
will point to the first `struct proc` in the running list. From there, we
can follow the chain of `next` pointers until we reach a `struct proc`
with a `NULL` in `next` pointer.  This is the last node in the list, and
`ptable.list[RUNNING].tail` will point to it.  When a list is empty, its
`head` and `tail` pointers must be `NULL`.

If you are unfamiliar or uncomfortable with using a tail pointer in a linked
list, you can get a refresher over at
[A Comprehensive Guide to Implementation of Singly Linked Lists Using C](https://www.codementor.io/codementorteam/a-comprehensive-guide-to-implementation-of-singly-linked-list-using-c_plus_plus-ondlm5azr) or in numerous other sources. 

### Invariants

- All processes on the ready list are in the `RUNNABLE` state able to be scheduled on a processor.
- All processes on the free list are in the `UNUSED` state able to be allocated to a new process.
- All processes on the sleep list are in the `SLEEPING` state waiting on some future event.
- All processes on the zombie list are in the `ZOMBIE` state waiting to be "reaped" by their parent process or the `init` process.
- All processes on the running list are in the `RUNNING` state executing on a CPU. (The maximum number of elements in this list is NCPU, the number of CPUs in xv6, see `param.h`).
- All processes on the embryo list are in the `EMBRYO` state being initialized.

**Note: A process must be on one, and only one, list at a time.** 

The guarantee that a process exists on one and only one list at a specific time,
reflecting its current state, is an `invariant`, or property that remains
unchanged during execution. For our purposes, the definition of  invariant is
*a quantity or expression that is constant throughout a certain range of conditions.*

The reality is that there will be times when a process will not be on a list - when
it transitions from one state to the next. This "in between" time must be
carefully hidden from all threads of control except the one manipulating the
process by using transactional semantics. Be *very* careful to not violate
invariants when manipulating processes; you risk introducing errors that can
lead to a panic, system hang, or corrupted state within xv6. As a precaution,
your code for removing an item from one of the lists is *required* to check
to ensure that the process removed is actually in the correct state (i.e., not
put there in error) and panic with an appropriate error message if the check
fails. E.g., a process being removed from the `running` list must be checked
to ensure that its state at the time of removal is `RUNNING`.

### Changing Existing Routines for This and Future Assignments

This assignment requires modifications to several existing functions in xv6.
For `exit()`, `wait(), `scheduler(), `yield()`, `sleep()`, `wakeup1()`, and `kill()`,
you **must** create a new version, where the new version is activated with
the `LAB3` flag, while the old version will still be included in the absence of
the new compiler flag.  Start by copying the existing function to the new
routine and then modifying the code to match the specification in this
assignment.  For example:

```c
#ifdef LAB3
void
foo() {
  /* copy of foo() modified for Lab3 */
}
#else
void
foo() {
  /* original version of foo() */
#endif
```

If a function will vary significantly between projects, you may structure your
code as follows:

```c
#ifdef LAB3
void
foo() {
  /* copy of foo() modified for Lab3 */
}
#elif defined(LAB2)
void
foo() {
  /* copy of foo() modified for Lab2 */
}
#elif  defined(LAB1)
void
foo() {
  /* copy of foo() modified for Lab1 */
}
#else
void
foo() {
  /* original version of foo() */
}
#endif
```

Note that the order matters in this structure, the implementations must be
listed from most to least recent project.

For changes in other functions (typically smaller changes), you may choose to
use the approach above, or you may just use conditional compilation within the
body of the function. For example:

```c
#ifdef LAB3
x = /* value x should have in Lab3 */
y = /* function call we should make in Lab3 */
#elif defined(LAB2)
x = /* value x should have in Lab2 */
y = /* function call we should make in Lab2 */
#elif defined(LAB1)
x = /* value x should have in Lab1 */
y = /* function call we should make in Lab1 */
#else
x = /* value x originally had */
y = /* function call we originally made */
#endif
```

This approach is most appropriate for routines where code is modified for each
project, e.g.,`procdump`.


## Initializing Process Lists

The first xv6 process is created in the routine `userinit()` in the file
`proc.c`. This is where you will need to initialize the lists. Two functions
have been provided for you to make initialization easier:

- `initProcessLists()`
- `initFreeList()`

You should initialize all the lists when you first enter this routine. This is
because the routine will then call `allocproc()` which is where a process
will be removed from the free list and allocated.

The new lists are part of the concurrent data structure `ptable`. You must use
proper concurrency control. This means that all code that accesses the new lists
can only perform that access while the `ptable` lock is held. Be sure to release
the lock when you are done with it.

`allocproc()` will place the process into the `EMBRYO` state. At the
very end of `userinit()` you will have a `RUNNABLE` process. The
process needs to be added to the ready list.


## Process List Management

The act of removing a process from a list and moving it to another list is a
manifestation of a *state transition*. All code for managing state
transitions is in the file `proc.c`. It is important that you understand
*list invariants* and *concurrency* before making these changes to
xv6. Be particularly careful to handle the cases where a state transition fails;
you will need to ensure that the process goes back to the list that it was
removed from in order to maintain the invariant. There are several locations in
`proc.c` where a state transition can fail; you will need to update this
processing to the new approach.

**Not all state transitions are listed below**; there are others that you will need
to locate and change as appropriate.

For the `scheduler()` routine in `proc.c`, you will create a new version
using conditional compilation. Start by copying the existing scheduler to
the new routine and then modifying the code to correctly use the new ready list.
This is where a process will transition from the `RUNNABLE` state to the `RUNNING` state.
You will also need list management where the state for a process transitions to
the `RUNNABLE` state. Be sure to add the scheduled process to the running list
when its state transitions accordingly in the *new* `scheduler()`, and remove it from
this list in the appropriate routine when it is about to yield the CPU. Note
that the transition from the `RUNNING` state can put the process into one of
three different states: `RUNNABLE`, `SLEEPING`, or `ZOMBIE`.

You will modify the `allocproc()` routine to use the free list when
searching for a process in the `UNUSED` state. You will also need to manage
the free list where the state for a process transitions to the `UNUSED` state.
Be sure to add the process to the embryo list in the routine `allocproc()`.
The process will transition from the `EMBRYO` to `RUNNABLE` state in
the `userinit()` and `fork()` routines.

You will modify the `wakeup1()` routine to improve the existing `wakeup1()` routine
using the sleep list; be careful, more than one process can be woken here!
Start by copying the existing `wakeup1()` routine to the new routine, and then
modifying the code to correctly use the new sleep list.
You will also need to manage the sleep list where the state for a process
transitions to the `SLEEPING` state.

Similarly, modify the `exit()`, `kill()`, and `wait()` routines to
use the `ready`, `sleep`, `zombie`, `embryo`, and `running` lists to remove
traversals of the `ptable.proc[]` structure while having the same functionality
as the existing code. Carefully copy the existing `exit()`, `wait()` and `kill()`
routines and modify the code to correctly use the new lists.

Keep in mind that all code for managing processes is in the file `proc.c`.
In the current kernel, any time that the state of a process is checked or
changed, one of the values from `enum procstate` in `proc.h` will be used.
This is a very good use of the C enumerator type.

It should be clear that these lists will improve certain performance aspects of
xv6. If this isn't clear to you, be sure to ask as it is rather important.

It's recommend that you implement the `free`, `ready`, and `sleeping` list
transitions first. Creating helper functions will likely aid greatly in the
organization of your code. Code for state transitions not yet converted to use
the new lists can continue to use the `ptable.proc[]` structure without
issue, unless there is a dependency - **be careful!**.

You will find that much of your list manipulation will be similar for most of
the lists. The following generic functions have been provided for you (see the
**List Management Functions** section below).

**You are *required* to use these list management functions.**

```c
static void
stateListAdd(struct ptrs* list, struct proc* p);
static void
stateListAddAtHead(struct ptrs* list, struct proc* p);
static int
stateListRemove(struct ptrs* list, struct proc* p);
```

To remove a process `p` from the running list using this function, write
```c
int rc = stateListRemove(&ptable.list[RUNNING], p);
```
where `rc` is the return code that indicates success or failure.

You might find that you are also writing the same code when checking
the state of removed process and possibly panic the kernel. This is where the
`enum` representation of process states come in handy. You can write a
separate function

```c
static void
assertState(struct proc* p, enum procstate state);
```

that asserts `p->state` is `state` and panics the kernel if the assertion is false
(be sure to provide a useful message in your call to `panic()`).

The `states[]` array can then be used to print out the name of the required
process state in the panic message. For example if you want to check that a
removed process `p` is in the `RUNNABLE` state, you can call

```c
assertState(p, RUNNABLE);
```

A suitable abstraction will help to make your code easier to maintain and less
prone to errors.  Be sure to perform these state checks after you have removed
a process from a list, and before you add it to another list.

For state transitions, you must follow this algorithm. Steps must be in the order listed.

1. Acquire `ptable` lock
2. Remove process from list
3. Assert that the process was in the correct state
4. Update the process
5. Put on new list
6. Release `ptable` lock


As an example, this is how to perform the state transition when the stack
allocation in `allocproc()` fails. Conditional compilation directives removed
for ease of reading.

```c
acquire(&ptable.lock);
stateListRemove(&ptable.list[p->state], p);
assertState(p, EMBRYO);
p->state = UNUSED;
stateListAdd(&ptable.list[p->state], p);
release(&ptable.lock);
```

### Conditional Compilation

In this project, you are required to implement alternate versions of entire
functions in `proc.c`. The reason for this approach is that if you are
having trouble with a specific function, you can "go back" to the old version
of that one routine just by changing the conditional compilation flag enclosing
that one routine. For the next project, it isn't critical that all lists use
this new approach, so it will be possible to turn off certain lists that are not
working correctly and not delay work on the next assignment. Speak with the
course staff if you find yourself needing to turn off certain functionality as
there are dependencies.


## Scheduler
xv6 already uses a round-robin scheduler. It isn't obvious, but if you pay
careful attention to how each CPU is managed, you can readily see that the
routine `scheduler()` implements a round-robin algorithm, albeit inefficiently.
You will be `improving` the efficiency of the scheduler, not implementing a new algorithm.

The round-robin scheduling algorithm is conceptually simple.

1. If there is a process at the head of the `ready list`
    - Remove it
    - Give it the CPU
2. When a process yields the CPU
    - Determine next state/list
    - If the new list is the ready list, place the process at the back of the list; otherwise place it anywhere on the new list
    - Invoke the scheduler via `sched()`

Since xv6 now uses state lists, the new `scheduler()` implementation will
more closely match our high-level description of the round-robin algorithm. Note
that the `scheduler()` routine does not perform step 2.



## New Console Control Sequences

You will add new console control sequences that will print information about certain lists.

- **ctrl-r**: prints information about the ready list
- **ctrl-f**: prints information about the free list
- **ctrl-s**: prints information about the sleep list
- **ctrl-z**: prints information about the zombie list

Detection of control sequences occurs in the routine `consoleintr()` in `console.c`.
Observe how **ctrl-p** is handled as you will do something similar for the new control sequences.

**Warning**: Unlike `procdump()`, which implements **ctrl-p**, the new control
sequences must use correct concurrency control.

### ctrl-r

This control sequence will print the PIDs of all the processes that are
currently on the ready list. Your output should look something like below:

```
Ready List Processes:

PID1 -> PID2 -> PID3 -> ... -> PIDn
```
where `PID1` is the PID of the first process in the ready list (the head), and `PIDn`
is the PID of last process in the list (the tail).

### ctrl-f

This control sequence will print the number of processes that are on the free
list. Your output will be

```
Free List Size: N processes
```
where `N` is the number of elements in the free list.

### ctrl-s

This control sequence will print the PIDs of all the processes that are
currently on the sleep list. Its format will be similar to **ctrl-r** above,
except that you will title it as "Sleep List Processes" instead of
"Ready List Processes".

### ctrl-z
This control sequence will print the PIDs of all the processes that are
currently on the zombie list as well as their parent PID. Its format will be
similar to **ctrl-r** above:

```
Zombie List Processes:

(PID1, PPID1) -> (PID2, PPID2) -> (PID3, PPID3) -> ... -> (PIDn, PPIDn)
```
where `PID1` is the PID of the first process on the list (the head) and `PPID1` is
the PID of the parent of process 1. The PID of the last process on the list (the
tail) is `PIDn` and `PPIDn` is the PID of parent process of process `n`.


## Required Tests

The list of required tests is contained in the project rubric.


## List Management Functions

The following code **must** be used for managing state lists:

### Headers

Place these function headers with all other function headers for `proc.c`.

```c
#ifdef LAB3
static void initProcessLists(void);
static void initFreeList(void);
static void stateListAdd(struct ptrs*, struct proc*);
static int  stateListRemove(struct ptrs*, struct proc* p);
#endif
```

### Implementation

```c
#ifdef LAB3
static void
initProcessLists()
{
  int i;

  for (i = UNUSED; i <= ZOMBIE; i++) {
    ptable.list[i].head = NULL;
    ptable.list[i].tail = NULL;
  }
  #ifdef LAB4
  for (i = 0; i <= MAXPRIO; i++) {
    ptable.ready[i].head = NULL;
    ptable.ready[i].tail = NULL;
  }
  #endif
}

static void
initFreeList(void)
{
  struct proc* p;

  for(p = ptable.proc; p < ptable.proc + NPROC; ++p){
    p->state = UNUSED;
    stateListAdd(&ptable.list[UNUSED], p);
  }
}

static void
stateListAdd(struct ptrs* list, struct proc* p)
{
  if((*list).head == NULL){
    (*list).head = p;
    (*list).tail = p;
    p->next = NULL;
  } else{
    ((*list).tail)->next = p;
    (*list).tail = ((*list).tail)->next;
    ((*list).tail)->next = NULL;
  }
}

static int
stateListRemove(struct ptrs* list, struct proc* p)
{
  if((*list).head == NULL || (*list).tail == NULL || p == NULL){
    return -1;
  }

  struct proc* current = (*list).head;
  struct proc* previous = 0;

  if(current == p){
    (*list).head = ((*list).head)->next;
    // prevent tail remaining assigned when we've removed the only item
    // on the list
    if((*list).tail == p){
      (*list).tail = NULL;
    }
    return 0;
  }

  while(current){
    if(current == p){
      break;
    }

    previous = current;
    current = current->next;
  }

  // Process not found, hit eject.
  if(current == NULL){
    return -1;
  }

  // Process found. Set the appropriate next pointer.
  if(current == (*list).tail){
    (*list).tail = previous;
    ((*list).tail)->next = NULL;
  } else{
    previous->next = current->next;
  }

  // Make sure p->next doesn't point into the list.
  p->next = NULL;

  return 0;
}
#endif
```

## Submission

Follow [submission guide](submission.md) to prepare your lab report and source code for evaluation.
