# Lab 4 Grading Rubric [77 points]

**If report is not in PDF format, report receives zero points.**

## Project Report [23 points]

Format must follow [exemplar](lab1-report-example.md) – state test, state test function, state expected result, show result, provide reasoning for why test is success or failure, properly label test **PASS** or **FAIL**.

All items are worth 1 point unless stated otherwise. Each section header lists the total points for its sections. **Must** use screen shots, not copy-paste.

**If report is not in PDF format, report receives zero points.**

1. Format [1 point]
    - Uses correct overall structure.

2. Required Tests [23 points]
    - Compiles correctly with `LAB_NUMBER` flag set to `0` in `Makefile`.

    - The remaining tests should be done with `LAB_NUMBER` flag set to `1` in `Makefile`.

        1. For `MAXPRIO = 0`: [3 points]
            - Show that the scheduler operates as a single round robin queue, as it did in lab 3.
            - Show that `setpriority()` for any value other than 0 fails.
            - Code should not attempt promotion or demotion when `MAXPRIO == 0`.

        2. For `MAXPRIO = 2`: [3 points]
            - Show that the scheduler always selects the first process on the highest priority non-empty list
            - Show that promotion correctly moves processes on the ready lists to the next higher priority list (if one exists) and maintains correct ordering
            - Show that demotion correctly moves a process to the next lower priority list (if one exists) when the processes budget is used up

        3. For `MAXPRIO = 6`: [3 points]
            - Show that the scheduler always selects the first process on the highest priority non-empty list
            - Show that promotion correctly moves processes on the ready lists to the next higher priority list (if one exists) and maintains correct ordering
            - Show that demotion correctly moves a process to the next lower priority list (if one exists) when the processes budget is used up

        4. `setpriority()` [4 Points]
            - Show that the priority is changed and budget reset when given a valid `PID` and `priority`
            - Show that changing the priority of a process on a *ready* list correctly moves the process to the list corresponding to the new priority
            - Show that setting the priority of a process on a *ready* list to the same priority it already has does not change the position in the list for that process
            - Show that calling `setpriority` with an invalid `PID` and/or `priority` returns a relevant error code and leaves the process priority and budget unmodified

        5. `getpriority()` [3 points]
            - Shows the correct priority for the current process
            - Shows the correct priority for any process other than the current process
            - Returns -1 if `PID` is not found

        6. Updated Commands [3 points]
            - Show that `ps` correctly displays the process priority
            - Show that **ctrl-p** correctly displays the process priority
            - Show that **ctrl-r** correctly displays all ready lists, from highest to lowest priority, and the budget for each process

## Code [54 points]

**The code submission must be the file created by running the `make tar` command in your project directory. Submitting code in any format not produced by `make tar` will result in zero points for the code section.**

### Submission [4 points]
1. with `LAB_NUMBER` flag set to `0` in `Makefile`
    - Correctly compiles and boots
    - `usertests` runs to completion with all tests passed
2. with `LAB_NUMBER` flag set to `4` in `Makefile`
    - Correctly compiles and boots
    - `usertests` runs to completion with all tests passed

### Priority Queues [12 points]
1. Correct implementation of array of ready lists of size `MAXPRIO+1`, where `MAXPRIO` is a new global define and priority is a field in `struct proc`.
2. Correctly initializes all *ready* lists in `userinit` with proper concurrency control
3. Correctly initializes process `priority` when a new process is allocated in `allocproc()`
4. Places processes in `RUNNABLE` state on the *ready* list corresponding to the priority of the process [4 points]
5. Assert **after** removal from ready lists that process was in both the correct state and priority for the list
6. Correctly replaced all instances of iterating over the ready list to iterating over all ready lists [4 points]

### MLFQ Scheduler [3 points]
1. Correct implementation of MLFQ scheduler

### Promotion [11 points]
1. Correct implementation of `PromoteAtTime` field in `ptable`, and `TICKS TO PROMOTE` define in
`custom.h`
2. Correct initialization of `PromoteAtTime`
3. Correctly determines when to promote and updates appropriately after promotion
4. Atomically performs promotion on all appropriate lists [3 points]
5. Correctly changes `priority` and `budget` [3 points]
6. Correctly moves appropriate processes between *ready* lists [2 points]

### Demotion [8 points]
1. Correct implementation of `budget` field in `proc` and its initialization
2. Performs check and demotion, if appropriate, when process leaves the CPU [2 points]
3. Correctly determines when to demote [2 points]
4. Correctly performs demotion [3 points]

### setpriority System Call [9 points]
You have been provided with a test program called `lab4-test.c`. The test program
will perform a complete check for the `setpriority()` system call.
Do not modify the test program.

1. Performs work in `proc.c` with `ptable.lock` held.
2. Correctly goes through all appropriate lists checking for `PID`
3. Correctly updates the `priority` and `budget` – [2 Points]
4. Correctly moves process to a different state list when appropriate [3 points]
5. Correctly returns error code for invalid `priority` or a `PID` that was not found
6. Provided test program, `lab4-test.c`, runs without error for this test.

### getpriority System Call [3 points]
You have been provided with a test program called `lab4-test.c`. The test program
will perform a complete check for the `getpriority()` system call.
Do not modify the test program.

1. Correctly returns the priority of an existing process. [1 point]
2. Correctly returns error code for invalid `PID`. [1 point]
3. Provided test program, `lab4-test.c`, runs without error for this test.

### Updated Commands [4 points]
1. Correctly adds header and value for `priority` to **ctrl-p**
2. Correctly adds header and value for `priority` to `ps` command
3. Updated **ctrl-r**: [2 points]
    - Correctly goes through all *ready* lists
    - Correctly displays budget for each process


## Coding Style [up to -7 Points]
1. Indentation must use spaces, not tabs.
2. Adding additional fields or modifying the `struct ptable` in ways not stated in the instructions, such as adding a hard coded number of *ready* lists. [-2 points]
3. Any console output in `proc.c`, except for control sequences or calls to panic. [-2 points]
4. Not holding the `ptable` lock when working with the state lists [-4 Points]
5. Not wrapping the added code in `#ifdef LAB4 ... #endif`
