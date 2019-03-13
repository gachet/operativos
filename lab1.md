# Lab 1

[Lab 1 Rubric](lab1-rubric.md)

[Lab 1 Report (Example)](lab1-report-example.md)

## Academic Honesty
Aside from the narrow exception for collaboration on homework, all work submitted in this course must be your own. Cheating and plagiarism will not be tolerated. If you have any questions about a specific case, please ask me.

## Introduction
This project will focus primarily on processes. In this project, you will become familiar with:

1. Implementing a new system call
2. Conditional compilation
3. Creating a new user command
4. The xv6 process structure
5. The control-p console command
6. Writing a project report to properly document project work.

### Reading

The reading for this project is chapters 1 - 3 from the [xv6 Book](xv6-book-rev11.pdf).


## Preparation

Clone a following repository https://github.com/wildart/xv6-public.git and checkout branch `lab1`.

    > git clone https://github.com/wildart/xv6-public.git xv6-labs
    > cd xv6-labs
    > git checkout lab1

If you already cloned this repository, you might want to pull recent changes
by using `git pull` command.


## Part 1: System Call Tracing

Your first task is to modify the xv6 kernel to print out a line for each system call invocation. It is enough to print the name of the system call and the return value; you don't need to print the system call arguments. When completed, you should see output like this when booting xv6:

    ...
    fork -> 2
    exec -> 0
    open -> 3
    close -> 0
    $write -> 1
    write -> 1

That's the `init` process using the `fork()` system call to create a new process and the `exec()` system call invoked to run the shell (sh)
followed by the shell writing the `$` prompt. The last two lines show the output from the shell and the kernel being intermixed. This is a *concurrency*
issue; more about that in class.

This new and wonderful feature of your kernel will cause a trace to be printed
every time a system call is invoked. You will find this ``feature''
**annoying**, so you want to be able to turn this feature on or off as
necessary. You are required to implement your syscall tracing facility
(code and data structures) using *conditional compilation* so that it can easily be turned on or off with a simple compilation flag. The flag name is
`PRINT_SYSCALLS`.

*Hint:* Modify the `syscall()` function in `syscall.c` to do the printing.
There is an array named `syscallnames[]` already defined in `syscall.c` that
is enabled when the `PRINT_SYSCALLS` define is set in the Makefile. Use this
structure and add the information for any new system calls that you create in
the class so that system call tracing will print correctly.

For the x86 architecture, the return code is the value of the `eax` register on return from the callee.

In order to compile xv6 with the `PRINT_SYSCALLS` compilation flag turned on,
add an additional parameter `PRINT_SYSCALLS=1` to `make` command call
as follows:

    > make PRINT_SYSCALLS=1
    > make qemu

### Conditional Compilation

You are required to write your code in such a way so that if the some
compilation flag is turned off (set to 0), e.g. `LAB1` then
xv6 will function the same as the initial distribution. This means that
your code will need to be *wrapped* in preprocessor directives. For example:

    #ifdef LAB1
    int
    sys_date(void)
    {
      struct rtcdate *d;
      // code removed
    }
    #endif // LAB1


The statement `#ifdef LAB1` is a GNU GCC preprocessor directive that
indicates that the code between this line and the matching `#endif` will
only be included in the compiled program if the flag is *turned on*; that
is, *defined*. Preprocessor directives are defined in the project Makefile.

**Caution:** Preprocessor directives do not apply to assembly language
programs (those ending in .S). Because of this, you cannot use a preprocessor
directive in the file `usys.S`. Fortunately, the system is designed such
that any extra code in this file will not impact xv6 functionality when
conditional compilation flags are turned off. Because of this restriction on
`usys.S`, conditional compilation cannot be used in the file `sycalls.h`
either. **These are the only two files where you will add code that is not
subject to conditional compilation.**


## Part 2: `date()` System Call

Your second task is to add a new system call to xv6. Having both the system
call and test program named `date` can be confusing. We will always call
a program that we run from the shell prompt a *command* or *program*
and always call a system call that your programs invoke to obtain kernel
services a *system call*. The goal of the exercise is for you to
become familiar with the principle parts of the system call infrastructure.
Your new system call will get the current UTC time and return it to
the invoking user program. You must use the function `cmostime()` defined
in `lapic.c` to read the real-time clock. The time resolution for
`cmostime()` is one second. The supplied include file `date.h` contains
the definition of the `rtcdate` data structure, a pointer to which you will
provide as an argument to `cmostime()`.

You will need a shell command that invokes your new `date()` system call
and we have provided one in the file `date.c`.

### Adding a New System Call
Adding a new system call to xv6, while straightforward, requires that several
files be modified. The following files will need to be modified in order to add
the `date()` system call.

* `user.h` contains the function prototypes for the xv6 system calls and library functions. The files `ulib.c`, `printf.c` and `ls.c` comprise the C library for xv6 - pretty small. This file is required to compile user   programs that invoke these routines. The function prototype is

        int date(struct rtcdate*);


* `usys.S` contains the list of system calls made available (exported) by the kernel. Add any new system calls at the end of this list.  Preprocessor directives cannot be used in this file. This file contains the code that *actually invokes* your system call. In xv6, a system call is invoked via a specific trap `int 0x40`. Be sure to understand how the code here will cause the system to change mode to kernel for your process.

* `syscall.h` contains the mappings of system call *name* to system call *number*. This is a critical part of adding a system call as system calls are invoked within the operating system by number, not name.  Follow the existing pattern for adding new system call numbers. In xv6, we require that the name associated with the system call number use the `SYS_` prefix. This is a very good technique for a kernel developer to use. Use the next integer value for the system call number of any new system call that you add. It should be clear, at this point, that system calls are actually invoked via number and not name. Remember that you will not be able to use preprocessor directives in this file.

* `syscall.c` is where the system call entry point will be defined. There are several steps to defining the entry point in this file.

    * The entry point will point to the implementation. The implementation
for our system call will be in another file. You indicate that with the
``extern'' keyword.

           extern int sys_date(void);

        which you will add to the end of the list of similar `extern` declarations. You  must declare the routine as taking a void argument!

    * Add to the syscall function definition. The function dispatch table

        	int (*syscalls[])(void) = {

        which declares the mapping from symbol name (as used by `usys.S` and declared in `syscall.h`) to the function name, which is the name of the function that will be called for that symbol. Add

        	[SYS_date]    sys_date,

        to the end of this structure.

        The routine `syscall()`, at the bottom of `syscall.c`, invokes the correct system call based on these definitions; see `vectors.pl`. The data structure `syscalls[]` is an array of function pointers which is often called a *function dispatch table*.

    * `sysproc.c` is where you will implement `sys_date()`. The   implementation of your `date()` system call is very straightforward. All you will need to do is add the new routine to `sysproc.c`; get the argument off the stack (This is why the function argument is `void`.); call the `cmostime()` routine correctly; and return a code indicating *SUCCESS* or *FAILURE*, which will most likely be `0` because this routine cannot actually fail (Your date command, however, must check for an error and do something reasonable). Note that since you passed into the kernel a *pointer* to the `rtcdate` structure, any changes made to the structure within the kernel will be reflected in user space. The first part of your implementation is provided for you

            int
            sys_date(void)
            {
              struct rtcdate *d;

              if(argptr(0, (void*)&d, sizeof(struct rtcdate)) < 0)
                return -1;
              return 0;
            }

	    The rest of the routine is left as an exercise for the student. See the xv6 book for how to properly use the `argptr()` routine. Be sure to read and understand the comments for the `argptr()` routine in  `syscall.c` as this will become critical in later projects.

    * `defs.h` is where function prototypes for kernel-wide function calls, that are not in `sysproc.c` or `sysfile.c`, are defined. This file is not modified in project one but will be modified in subsequent projects.


### The `date` Command

We cannot directly test a system call and instead have to write a program that
we can invoke as a command at the shell prompt. An example program, `date.c`
has been provided. However, it is not included in the default compilation mode.
In order to compile `date.c` file set `LAB_NUMBER` variable in `Makefile` to `1`.

The `date.c` program is designed to mimic the Linux command `date -u`.

The `printf()` routine takes a file descriptor as its first argument: 1 is
standard output (`stdout`) and 2 is error output (`stderr`).

The function `main()` **must** terminate with a call to `exit()` and
not `return()` or simply fall off the end of the routine.
This is a **very** common source of compilation errors.

The `date.c` and `date.h` files are already added to `runoff.list` so that
the commands `make dist-test` and `make tar` will work correctly. In future
projects, you will need to add new source code files to `runoff.list` yourself.


## Part 3: Control-P

The xv6 kernel supports a special control sequence, `control-p`, which
displays process information on the console (see `console.c`, `case C(’P’)`).
It is intended as a debugging tool and you will expand the information reported
as you add new features to the xv6 process structure. Note that the routine
`procdump()`, at the end of `proc.c`, implements the bulk of the `control-p`
functionality.

You will modify `struct proc` in `proc.h` to include a new field,
`uint start_ticks`. You will modify the routine `allocproc()` in `proc.c`
so that this field is initialized to the value for the
existing global kernel variable `ticks`. This allows the process to "know" when
it was created. This will allow us to later calculate how long the process has
been active. Put the initialization code right before the return at the end of
the routine. Access to `ticks` variable requires a lock, see `sys_uptime`
system call for more details.

You will then modify the `procdump()` routine to print out the amount of
time that each process has been active in system.
This is fairly straightforward since each `tick` is a millisecond.
Report the elapsed time in seconds, accurate to the millisecond.
The longest running process in xv6 will always be the `init` process,
but the first shell will be close behind. There is no need to modify
the `printf()` or `cprintf()` routines to print this information.
You will also modify the routine to print the size of the process.
The process size is already part of the process structure, so you are
just printing it out, not calculating it.

The first line of output should be a set of labels for each column. You will be
adding to the `control-p` output in later projects and the header will
help with interpreting the output. The last set of information printed from
`procdump()` contains the program counters as returned by the routine
`getcallerpcs()`. The header for this last section can just be labeled
``PCs''.

Keep the code in `procdump()` as clean as you can. It may be useful for
you to create a helper function so as not to clutter the code in that routine.

### Example Output

    PID     Name         Elapsed    State   Size     PCs
    1       init         1.543      sleep   12288    80104bf3 80104968 80106560 80105767 801068f0 801066eb
    2       sh           1.499      sleep   16384    80104bf3 80100a37 80101f68 80101265 8010591e 80105767 801068f0 801066eb

The `procdump()` routine will be modified in many of the projects. Using
conditional compilation and restricting your changes to the routine itself
can get very messy. This approach duplicates some code, but is easier for
you to maintain and must less prone to error. Here is the suggested
implementation for `procdump()`. The details of the helper functions are
left to the student.

    void
    procdump(void)
    {
      static char *states[] = {
      [UNUSED]    "unused",
      [EMBRYO]    "embryo",
      [SLEEPING]  "sleep ",
      [RUNNABLE]  "runble",
      [RUNNING]   "run   ",
      [ZOMBIE]    "zombie"
      };
      int i;
      struct proc *p;
      char *state;
      uint pc[10];

    #if defined(LAB3) || defined (LAB4)
    #define HEADER "\nPID\tName         UID\tGID\tPPID\tPrio\tElapsed\tCPU\tState\tSize\t PCs\n"
    #elif defined(LAB2)
    #define HEADER "\nPID\tName         UID\tGID\tPPID\tElapsed\tCPU\tState\tSize\t PCs\n"
    #elif defined(LAB1)
    #define HEADER "\nPID\tName         Elapsed\tState\tSize\t PCs\n"
    #else
    #define HEADER "\n"
    #endif

      cprintf(HEADER); // Show header

      for(p = ptable.proc; p < &ptable.proc[NPROC]; p++){
        if(p->state == UNUSED)
        continue;
        if(p->state >= 0 && p->state < NELEM(states) && states[p->state])
        state = states[p->state];
        else
        state = "???";

    #if defined(LAB3) || defined (LAB4)
        procdump_L34(p, state);
    #elif defined(LAB2)
        procdump_L2(p, state);
    #elif defined(LAB1)
        procdump_L1(p, state);
    #else
        cprintf("%d %s %s", p->pid, state, p->name);
    #endif
        if(p->state == SLEEPING){
        getcallerpcs((uint*)p->context->ebp+2, pc);
        for(i=0; i<10 && pc[i] != 0; i++)
            cprintf(" %p", pc[i]);
        }
        cprintf("\n");
      }
    }


Now, you can define functions `procdump_*` to handle specific output required
for a certain lab. Note that this is just one way to handle the complexities of
the changes to `procdump()` and you are not required to use this method.
You can use any method that seems reasonable to you and produces the correct output.

## Submission

Follow [submission guide](submission.md) to prepare your lab report and source code for evaluation.
