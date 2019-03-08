# Lab 1 Grading Rubric [34 points]

**If report is not in PDF format, report receives zero points.**

## Project Test Report [14 points]

Format must follow [exemplar](lab-report.md) – state test, state test function, state expected result, show result, provide reasoning for why test is success or failure, properly label test **PASS** or **FAIL**.

All items are worth 1 point unless stated otherwise. Each section header lists the total points for
its section. Format must follow exemplar - state test, state expected result, show result, provide sound reasoning for why test is success or failure, properly label test as **PASS** or **FAIL**. Must use screen shots, not copy-paste.

1. with `LAB_NUMBER` flag set to 0 in `Makefile` – [1 Point]
    - Code correctly compiles

2. with `LAB_NUMBER` flag set to 1 in `Makefile` – [1 Point]
    - Code correctly compiles

3. with `PRINT_SYSCALLS` flag turned off in `Makefile` – [1 Point]
    - No trace information is displayed

4. with `PRINT_SYSCALLS` flag turned on in `Makefile` – [2 Points]
    - Tracing facility works correctly
    - Correct system call trace on boot

5. with `LAB_NUMBER` flag set to 0 – [2 Points]
    - Test program `forktest` runs correctly
    - Test program `usertests` runs correctly

6. with `LAB_NUMBER` flag set to 1 – [7 Points]
    - Test program `forktest` runs correctly
    - Test program `usertests` runs correctly
    - The `date` command prints correct information
    - The following test shows that the `date` command and the Linux `date -u` command provide basically the same information. (i.e. Run date command in xv6 → Exit xv6 → Run date command from linux prompt → Compare ouput)
    - `Control-P`
        - Displays correct header
        - Data aligns with appropriate header
        - Correct data is displayed

## Code – [20 points]

### Submission – [1 Point]

1. Code submitted in correct form including all files

### System Call Tracing – [17 Points]

1. with `PRINT_SYSCALLS` flag turned off in `Makefile` – [1 Point]
    - Code compiles

2. with PRINT SYSCALLS flag turned on – [3 Points]
    - Code compiles
    - Tracing facility correctly uses `cprintf()` inside kernel
    - Tracing facility gracefully handles an unknown function number

3. With `LAB_NUMBER` flag set to 0 in `Makefile` – [4 Points]
    - Code compiles
    - Kernel boots
    - Test `forktest` runs to completion correctly
    - Test `usertests` runs to completion correctly

4. With `LAB_NUMBER` flag set to 1 in `Makefile` – [8 Points]
    - Code compiles
    - Kernel boots
    - Test `forktest` runs to completion correctly
    - Test `usertests` runs to completion correctly
    - The `date` command and system call
        - System call correctly passes pointer to structure
        - System call does not print any information
        - Command output correct information
        - Command matches linux command

### `Control-P` – [3 Points]

1. Displays correct header
2. Output aligned with header
3. New information is correctly displayed

### Coding Style – [up to -3 Points]

1. Indentation must use spaces, not tabs.
2. Function declarations be in correct form.
3. Code should match the general code style of *xv6*.
