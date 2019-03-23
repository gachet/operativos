# Lab 2

[Lab 2 Rubric](lab2-rubric.md)

## Academic Honesty
Aside from the narrow exception for collaboration on homework, all work submitted in this course must be your own. Cheating and plagiarism will not be tolerated. If you have any questions about a specific case, please ask me.

## Introduction
This project will focus primarily on processes. In this project, you will become familiar with:

1. Implementing new system calls to support process ownership.
2. Implementing a new system call to obtain process information.
3. Implementing tracking for the amount of time a process uses a CPU.
4. Implementing a new user-level command to display process state.
5. Implementing a new user-level command to time process execution.
6. Writing a project report to properly document project work.

## Preparation

Clone a following repository https://github.com/wildart/xv6-public.git and checkout branch **lab2**.

    > git clone https://github.com/wildart/xv6-public.git xv6-labs
    > cd xv6-labs
    > git checkout lab2

If you already cloned this repository, you might want to pull recent changes
by using `git pull` command.


## Part 1: UIDs and GIDs and PPIDs

At this point xv6 has no concept of users or groups. You will begin to add this feature to xv6 by adding a `uid` and `gid` field to the process structure, where you will track process *ownership*.

These will be of type `unsigned int` since negative UIDs and GIDs make no sense in this context. Note that when these values are passed into the kernel, they will be taken off the stack as "int"s. There is no issue with this as you will convert them to unsigned ints immediately. It is, however, critical, that the function prototypes in `user.h` declare values as unsigned as you will see below.

The `ppid` is the "parent process identifier" or parent PID. The `proc` structure does not need a `ppid` field as the parent can, and should, be determined on-the-fly. Look carefully at the existing proc structure in `proc.h` to see what is needed.

*Note:* that the init process is a special case as it has no parent. Your code must account for any process whose parent pointer is `NULL`. For any such pointer, you will display the PPID to be the same as the process PID. Do not modify a parent pointer that is set to `NULL`; leave it that way as it becomes important in a later project.

You will need to add the following system calls:

    uint getuid( void )     // UID of the current process
    uint getgid( void )     // GID of the current process
    uint getppid( void )    // PPID of the current process

    int setuid ( uint )     // set UID
    int setgid ( uint )     // set GID

Your kernel code cannot assume that arguments passed into the kernel are valid and so your kernel code must check the values for the correct range. The `uid` and `gid` fields in the process structure may only take on values 0 <= value <= 32767. You are **required** to provide tests that show this bound being enforced by the kernel-side implementation of the system calls.

The following code is a starting point for writing a test program that demonstrates the correct functioning of your new system calls. This example is missing several important tests and fails to check return codes, which is very bad programming. You should fix the shortcomings of this code or write a new test program that properly demonstrates correct functionality for **all** test cases.

    int main ( void )
    {
        uint uid, gid, ppid;
        uid = getuid();
        printf( 2 , " Current UID is: %d\n", uid );
        printf( 2 , " Setting UID to 100\ n" );
        setuid (100);
        uid = getuid ();
        printf( 2 , " Current UID is: %d\n", uid );

        gid = getgid() ;
        printf( 2 , " Current GID is: %d\n", gid );
        printf( 2 , " Setting GID to 100\ n" );
        setuid (100);
        gid = getgid();
        printf( 2 , " Current GID is: %d\n", gid );

        ppid = getppid();
        printf( 2 , "My parent process is: %d\n", ppid );
        printf( 2 , "Done!\n" );
        exit();
    }

### Other Necessary Modifications

You have modified the process structure to include the `uid` and `gid` for
the process, but that isn't all the work necessary to support these new
features. The `fork()` system call allocates a new process structure and
copies all the information from the original process structure to the new one,
with the exception of the `pid`. But you modified the process structure!
You will need to find the code for the `fork()` system call and make sure to
copy the `uid` and `gid` of the current process to the new child process.

Not all processes are created with `fork()`. The first process, which
eventually becomes the `init` process, is created piece-by-piece at boot time.
The routine `userinit()` in the file `proc.c` is where this initialization
takes place. Add a `#define` statement in `proc.h` for the default `uid` and `gid` of the first process. This will make your code easier to read.

You should also be able to set the `uid` and `gid` of the currently executing shell with appropriate built-in commands. The  shell includes in the parser the ability to identify built-ins as built-ins begin with an underscore.

Use an additional parameter `USE_BUILTINS=1` in `make` command to turn on this functionality. The following built-in commands have been implemented.

    _set uid int
    _set gid int
    _get uid
    _get gid

You should include tests that show the `uid`/`gid` for the shell being changed and that any program you run from the command line inherits the correct uid and gid.

## Part 2: Process Execution Time

Currently, your xv6 system tracks when a process enters the system and
displays *elapsed* time in the console command "control-p". You will now
track how much CPU time a process uses.

There are two situations where a context switch occurs in xv6: one to put
a process into a CPU and one to take a process out of its current CPU.
The currently running process is removed from its CPU in the routine
`sched()` and a RUNNABLE process is put into a CPU in the routine
`scheduler()`, both are in `proc.c`.

You will need to add two new fields to the process structure.

    uint  cpu_ticks_total;   // total elapsed ticks in CPU
    uint  cpu_ticks_in;      // ticks when scheduled

The `cpu_ticks_in` value will be set when the process enters a CPU.
The `cpu_ticks_total` will be updated when the process is removed from
its CPU. You do not need a `cpu_ticks_out` field.

A new process is allocated in the routine `allocproc()` in the file
`proc.c`. Initialize these two new fields to zero in that routine.

## `ps` Command

Xv6 does not have the `ps` command like Linux, so you will add your own. This command is used to find out information regarding active processes in the system. We define "active" here to be a process in the RUNNABLE, SLEEPING,  RUNNING, or ZOMBIE state. Processes in the UNUSED or EMBRYO states are not considered active. In order to write your `ps` program, you will need to add another system call. See [process state transition diagram](xv6-state-transition-model.pdf).

You will find  `struct proc`, *the process structure*, in the file `proc.h`. When xv6 is running, there is an array of `proc struct`s in the data structure named `ptable` in `proc.c`. All information for each process is in a `struct proc` in the `proc[]` array.

The `ps` command will print the following  information for each active process

- process id (as decimal integer)
- name (as string)
- process uid (as decimal integer)
- process gid (as decimal integer)
- parent process id (as decimal integer)
- process elapsed time (as a floating point number)
- process total CPU time (as a floating point number)
- state (as string)
- size (as decimal integer)

You'll print one line for each process, with a header indicating the contents
for each column. Note that the xv6 `printf` routine does not support floating
point numbers. This is fine since, for the values we will be calculating,
the integer portion before the decimal point can be calculated with integer
division and the integer portion after the decimal can be calculated
using modulo arithmetic.

The system call that you need to add is called `getprocs`:

	int
	getprocs(uint max, struct uproc* table);

Note that there is a new structure `uproc`. This new struct is there because
you do not need all the information stored in the `struct proc` and
you should never make a kernel data structure visible to a user program as
the user program could modify the data. Due to restrictions on the size of
the stack in xv6, you will need to allocate memory for this data structure
in the user program from the heap; that is, use `malloc()`. If you create
the data structure correctly you will be able to access it as though
it were an array from inside the kernel. You will pass in a pointer to
the array of `uproc` structs that the kernel will fill in.
The argument `max` is the maximum number of entries that your array of
`struct uproc`s will hold. Your `getprocs()` call should return
the actual number of entries used in the table on success and -1 on any error.

Your `ps` program should do something sane when an error is returned. You must test the system call with at least `max` set equal to 1, 16, 64, and 72.

Use this definition for the `uproc` structure. Put it in a file named `uproc.h`.

    #define STRMAX 32

    struct uproc {
      uint pid;
      uint uid;
      uint gid;
      uint ppid;
      uint elapsed_ticks;
      uint CPU_total_ticks;
      char state[STRMAX];
      uint size;
      char name[STRMAX];
    };


The value for `STRMAX` should be able to take on *any* non-negative value
and your routine should still work correctly.

## Part 3: `time` Command

Your `time` command will determine the number of seconds that a program takes to run. Below is an example of the command execution:

    $ time forktest
    fork test
    fork test OK
    forktest ran in  0.915 seconds.
    $
    $ time echo "abc"
    "abc"
    echo ran in 0.041 seconds.
    $

The last line of each test is the output from the `time` command.
The previous lines are output from the `forktest` and `echo` commands.
The `echo` test demonstrates how the entire command line is passed to
the named command. As a more complex example, consider this test:

    $ time time echo "abc"
    "abc"
    echo ran in 0.031 seconds.
    time ran in 0.066 seconds.
    $

This test shows the `time` command being called twice.
In reading the line left-to-right, we can see that the first `time` command
is timing the second `time` command. The second `time` command is timing
the `echo` command that will echo the string `"abc"` to standard output.
Since the `echo` command will just print *all* its arguments to standard
output, the exact order of the commands is important in this example.
This next example should help the student to understand better:

    $ time echo "abc" time
    "abc" time
    echo ran in 0.056 seconds.
    $


Your command will behave the same way when no program name is provided:

    $ time
    (null) ran in 0.016 seconds.


Note that this last program is "timing" a `NULL` command. This is correct
behavior and how the time command works on Linux. You do not have to put in
a special check for a `NULL` or invalid command in your code.

You will want to base your timing information on the kernel global variable
called `ticks`. If you review the list of system calls, you will find
an existing call that will suffice.

You will add the `time` command to your system the same way that you added your `date` command.

## Submission

Follow [submission guide](submission.md) to prepare your lab report and source code for evaluation.
