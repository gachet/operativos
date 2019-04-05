# Lab 3 Grading Rubric [50 points]

**If report is not in PDF format, report receives zero points.**

## Lab Test Report [6 points]

Format must follow [exemplar](lab1-report-example.md) – state test, state test function, state expected result, show result, provide reasoning for why test is success or failure, properly label test **PASS** or **FAIL**.

All items are worth 1 point unless stated otherwise. Each section header lists the total points for its sections. **Must** use screen shots, not copy-paste.

**If report is not in PDF format, report receives zero points.**

1. with `LAB_NUMBER` flag set to `0` in `Makefile`
    - Code correctly compiles.
    - Program `forktest` runs correctly.
    - Program `usertests` runs correctly.
2. with `LAB_NUMBER` flag set to `3` in `Makefile`
    - Code correctly compiles.
    - Program `forktest` runs correctly.
    - Program `usertests` runs correctly.


## Required Tests [8 points]

Format must follow exemplar – state test, state test function, state expected result, show result,
provide reasoning for why test is success or failure, properly label test **PASS** or **FAIL**.

All tests should be done with your `LAB_NUMBER` flag set to `3` in `Makefile`:

1. Show the free list is correctly initialized after xv6 is fully booted.
2. Show the free list is correctly updated when a process is allocated and deallocated.
3. Demonstrate that round-robin scheduling is maintained.
4. Show the sleep list is correctly updated when a process sleeps and is woken up.
5. Process Death - [3 points]
    - Show the kill system call correctly causes a process to move to the zombie list.
    - Show the exit system call correctly causes a process to move to the zombie list.
    - Show the wait system call correctly causes a process to move to the free list.
6. Show the console commands **control-r**, **control-s**, **control-f**, and **control-z** are correct.

## Code [36 points]

**The code submission must be the file created by running the `make tar` command in your project directory. Submitting code in any format not produced by `make tar` will result in zero points for the code section.**

### Submission [5 points]
1. with `LAB_NUMBER` flag set to `0` in `Makefile`
    - Correctly compiles and boots
    - `usertests` runs to completion with all tests passed
2. with `LAB_NUMBER` flag set to `3` in `Makefile`
    - Correctly compiles and boots
    - `usertests` runs to completion with all tests passed
3. All changes to existing state management must be implemented in a new function.

### State Lists [25 points]

Each of these is in reference to changes made to implement the state lists. If you
do not implement the state lists for a given function, you will receive zero points for
that function.

1. Correct implementation of `struct ptrs` in `proc.c` and adding of `list` field to
`ptable` structure.
2. Correct initialization of state lists and placement of `init` process on ready list in `userinit`
3. allocproc [2 points]
4. fork [2 points]
5. exit [3 points]
6. wait [3 points]
7. scheduler [4 points]
8. yield [2 points]
9. sleep [1 point]
10. wakeup1 [4 points]
11. kill [3 points]

### Helper Functions [2 points]

1. Uses the provided helper functions for adding and removing processes from lists.
2. Asserts that processes being removed from a list are in the appropriate state for that list.
(Again, this doesn't have to be done in a helper function but it is the cleanest way.)

### Control Commands [4 points]
1. Correctly adds code for detecting and handling **control-r**, **control-s**, **control-f**, and **control-z** in `console.c`
2. Correctly performs all list interactions and printing in `proc.c`
3. Each command correctly prints all required information
4. Each command prints in the expected format


### Coding Style – [up to -6 Points]

Each of these represents a point that will be deducted from the code section if found.

1. Two (2) spaces of indentation per level must be used. Indentation must use spaces, not tabs.
2. Adding additional fields or modifying the `ptable struct` in ways not stated in the instructions, such as adding all state lists individually to the `ptable`.
3. Adding print statements in `proc.c`, outside of the new control commands.
4. Not holding the `ptable` lock when modifying the state lists [-2 points]
5. Not wrapping your added code in `#ifdef LAB3 ... #endif`
