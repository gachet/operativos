# Lab 0

**Due Date: Feb 15 at midnight.**

## Academic Honesty
Aside from the narrow exception for collaboration on homework, all work submitted in this course must be your own. Cheating and plagiarism will not be tolerated. If you have any questions about a specific case, please ask me.

## Part 1: Hello World (20 pts)

This lab will provide a first glance on xv6 work. You'll write a program for xv6 that, when run, prints "Hello world" to the xv6 console.

This can be broken up into a few steps:
1. Create a file in the xv6 directory named hello.c
2. Put code you need to implement printing "Hello world" into hello.c
3. Edit the file Makefile , find the section UPROGS (which contains a list of programs to be built), and add a line to tell it to build your Hello World program. When you're done that portion of the Makefile should look like:

		UPROGS=\
			_cat\
			_echo\
			_forktest\
			_grep\
			_init\
			_kill\
			_ln\
			_ls\
			_mkdir\
			_rm\
			_sh\
			_stressfs\
			_usertests\
			_wc\
			_zombie\
			_hello\

4. Run `make` to build xv6, including your new program (repeating steps 2 and 4 until you have compiling code)
5. Run `make qemu` to launch xv6, and then type hello in the QEMU window. You should see "Hello world" be printed out.

## Part 2: `head` command (50 pts)

Write a program that prints the first 10 lines of its input. If a filename is provided on the command line (i.e., `head FILE` ) then `head` should open it, read and print the first 10 lines, and then close it. If no filename is provided, head should read from standard input.

    $ head README
    xv6 is a re-implementation of Dennis Ritchie's and Ken Thompson's Unix
    Version 6 (v6). xv6 loosely follows the structure and style of v6,
    but is implemented for a modern x86-based multiprocessor using ANSI C.

    ACKNOWLEDGMENTS

    xv6 is inspired by John Lions's Commentary on UNIX 6th Edition (Peer
    to Peer Communications; ISBN: 1-57398-013-7; 1st edition (June 14,
    2000)). See also https://pdos.csail.mit.edu/6.828/, which
    provides pointers to on-line resources for v6


You should also be able to invoke it without a file, and have it read from standard input. For example, you can use a pipe to direct the output of another xv6 command into `head`:

    $ grep the README | head
    Version 6 (v6). xv6 loosely follows the structure and style of v6,
    xv6 borrows code from the following sources:
        JOS (asm.h, elf.h, mmu.h, bootasm.S, ide.c, console.c, and others)
        Plan 9 (entryother.S, mp.h, mp.c, lapic.c)
    In addition, we are grateful for the bug reports and patches contributed by
    The code in the files that constitute xv6 is
    To run xv6, install the QEMU PC simulators. To run in QEMU, run "make qemu".
    To create a typeset version of the code, run "make xv6.pdf". This
    requires the "mpage" utility. See http://www.mesa.nl/pub/mpage/.

The above command searches for all instances of the word the in the file README, and then prints the first 10 matching lines.

Hints:

1. Many aspects of this are similar to the `wc` program: both can read from standard input if no arguments are passed or read from a file if one is given on the command line. Reading its code will help you if you get stuck.

## Part 3: Extending `head` (30 pts)

The traditional UNIX head utility can print out a configurable number of lines from the start of a file. Implement this behavior in your version of `head`. The number of lines to be printed should be specified via a command line argument as `head -NUM FILE`, for example `head -3 README` to print the first 3 lines of the file README. The expected output of that command is:

        $ head -3 README
        xv6 is a re-implementation of Dennis Ritchie's and Ken Thompson's Unix
        Version 6 (v6). xv6 loosely follows the structure and style of v6,
        but is implemented for a modern x86-based multiprocessor using ANSI C.

If the number of lines is not given (i.e., if the first argument does not start with `-`), the number of lines to be printed should default to 10 as in the previous part.

Hints:

1. You can convert a string to an integer with the `atoi` function.
2. You may want to use pointer arithmetic to get a string suitable for passing to `atoi`.

# Submission

Submit `hello.c` and `head.c` files through the appropriate Blackbord group submission.
