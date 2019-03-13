# Lab 2 Grading Rubric [46 points]

**If report is not in PDF format, report receives zero points.**

## Lab Test Report [12 points]

Format must follow [exemplar](lab1-report-example.md) – state test, state test function, state expected result, show result, provide reasoning for why test is success or failure, properly label test **PASS** or **FAIL**.

All items are worth 1 point unless stated otherwise. Each section header lists the total points for its sections Must use screen shots, not copy-paste.

1. with `LAB_NUMBER` flag set to 0 in `Makefile` – [1 Point]
    - Code correctly compiles

    All remaining tests must have the `LAB_NUMBER` flag set to **2** in the `Makefile`.

2. With your UID/GID test program: – [2 Points]
    - Correctly set / get UID and GID and get PPID.
    - Correctly handle attempting to set UID and GID to invalid numbers.

3. Show that `Control-p` correctly prints all new information.

4. Test for correct output of ps command. (Compare to original `Control-p` behavior)

4. Test the built-in shell commands to set UID and GID, and show that child processes correctly inherit the new UID and GID values. (`ps` command can do this)

5. Tests for `getprocs()` with with 64 active processes. [2 Points]
    - Correct output with MAX set to 1, 16, 64, 72
    - Correctly use `Control-p` as a comparison. (Using `wait()` in your test program will help)

6. Test for the correct elapsed CPU time in `Control-p`

7. Tests for the `time` command – [3 Points]
    - Call with no arguments and an invalid argument
    - Call with a valid command argument that takes an argument
    - Show that the calculated time is accurate (`Control-p` elapsed time before and after on a long process would work fine). For example, the date command can help:

            $ date; time usertests; date

         Since the date command is already known to be accurate, bounding the time command
between two date commands is one way to establish that the time command is working
correctly.

## Code [34 points]

**The code submission must be the file created by running the `make tar` command in your project directory. Submitting code in any format not produced by `make tar` will result in zero points for the code section.**


1. Submission – [2 Points]
    1. Correctly compiles and boots `LAB_NUMBER` flag set to 0 in `Makefile`
    2. Correctly compiles and boots `LAB_NUMBER` flag set to 2 in `Makefile`

2. UID/GID/PPID System Calls – [7 points]
    1. Correct implementation of UID and GID in proc struct
    2. Correct implementation of `setuid`, `setgid`, `getuid`, and `getgid`
    Must ensure that the setters for UID or GID values are inside the range 0 ≤ x ≤ 32767.
    Provides appropriate return value on error.
    3. Sets reasonable (probably 0) value for UID/GID defaults in param.h
    4. Sets UID/GID defaults in `userinit()`
    5. New processes properly inherit UID/GID values in `fork()`
    6. Correct implementation of `getppid()`, properly handling `init`'s parent
    7. Grader tests `testsetuid` & `testuidgid` works correctly for this test

3. CPU Time – [3 points]
    1. Correct location when setting cpu ticks in
    2. Updates cpu ticks total in correct location
    3. Grader test suite `lab2-test` works correctly for this test

4. `getprocs()` System Call – [8 points]
    1. Correct usage of `argint()` and `argptr()`
    2. Interacts with `ptable` only in `proc.c`
    3. Correctly copies all `proc struct` values to `uproc` structure
    4. Correct concurrency control for `ptable`
    5. Handles PPID of init correctly
    6. Only acquires active processes, no `UNUSED` or `EMBRYO` processes acquired
    7. Correct algorithm for copying the table entries
    8. Grader test suite `lab2-test` works correctly for this test

5. `ps` Command [4 points]
    1. Error checks for getprocs return
    2. Is consistent with `Control-p`
    3. Correct use of malloc() and free()
    4. Correctly displays all information from `getprocs`

6. `Control-p` – [3 points]
    1. New headers are correct and output aligned
    2. Displays UID, GID, PPID
    3. Displays CPU total time correctly formatted

7. `time` Command [7 points]
    1. Correctly calls exec() in the child process and passes the arguments correctly
    2. Uses the correct system call to retrieve the tick count
    3. Error checks the return code for fork() system call
    4. Parent correctly waits for child
    5. Displays command name. No argument will likely be displayed as "(NULL)"
    6. Correct calculation and formatting of the elapsed time
    7. Test suite works correctly for this test

### Coding Style – [up to -5 Points]

Each of these represents a point that will be deducted from the code section if found.

1. Two (2) spaces of indentation per level must be used. Space characters not tabs
2. Adding additional fields or modifying the proc struct in ways not stated in the instructions, such as adding a PPID field
3. System calls that print to the console, except control sequences, are not allowed
4. System calls that do not return a reasonable value (i.e. negative for errors and zero for no errors, or a positive number if it has meaning)
5. Not wrapping your added code in `#ifdef LAB2 ... #endif` (this makes it easier to find your code for grading) and also allows your code to be compiled with the `LAB_NUMBER` flag set to 0, which is a requirement
6. Not placing the uproc struct in its own **.h** file
