# Submission

## Coding Style

In any large lab, matching the existing coding style is important.
Having different coding styles intermixed leads to confusion and bugs.
Students are required to follow the existing coding style in xv6 - this includes
maintaining the indentation style in `.c` and `.h` files using spaces, not tabs.

In particular, pay close attention to function declarations and how the function
name begins the line after the function return type.
For helper functions which are limited in scope to a specific file, you must
declare the function as "static" in the same file in which it is used.

*On using goto*: Please avoid usage of C `goto` instruction. There are times when
it clarity for code clarity and handling awkward error states.
The original xv6 code uses the `goto` instruction in certain places for code
clarity in order to assist your learning. Do not add any additional `goto` statements.
Using a `goto` is considered a style violation and will be penalized with a deduction.
Do not use `goto`. That said, there is one place in
the final lab where a `goto` makes sense because of an awkward error case.

*Indentation*: The indentation style for xv6 is 2 spaces. Many students are
taught to use tabs for indentation, which can make code very hard to read,
especially when there are several levels of indentation.

You can check for leading tabs (a tab of a control sequence) in your
code with the following regular shell command. Note that this only finds a
control sequence if it is the first character on the line. This will not find
all places where tabs are uses for indentation.

	grep -P ^'\t' *.h *.c

Here is the output when run on a provided version of the xv6 source code

	$ grep -P ^'\t' *.h *.c
	$

Note that no lines beginning with a control sequence were found.

For vim users, see the `vim` wiki for information on
[converting tabs to spaces](http://vim.wikia.com/wiki/Converting_tabs_to_spaces).
You can automatically set C-style indentation to 2 spaces and automatically
convert tabs to 2 spaces with this entry in your `.vimrc` file

	:set smartindent
	set expandtab autoindent shiftwidth=2 tabstop=2



**Remember:** Code must match the general code style of xv6.

## Required Tests

The list of required tests are in the lab rubric.

## Preparing Your Code Submission

Students are required to follow the directions in this section for generating an archive file for submission. Failure to follow these directions may result in a (potentially major) points deduction.

### runoff.list
Add any new files that you want to be submitted as part of your assignment to the file `runoff.list`.

Here are example entries:

    # CSCI375 additions by  John Doe
    date.c
    ps.c
    ps.h
    time.c

Add new file names at the end of the file.

### Creating and Testing Your Submission

You are required to test your submission to ensure that it works. You test that your code works with the command `make dist-test`. This command will create the `dist` and `dist-test` subdirectories and then run the xv6 kernel for testing. If you find that files are missing then it is likely that you failed to update the `runoff.list` file. When you are satisfied with the results, create your submission with `make tar`.

The tar file is a copy of your `dist` subdirectory. The tar file created by the make tar command constitutes the code submission for each lab. Students **may not** submit `.rar` files, `.zip` files, etc.

Lab reports are to be submitted separately from the source code. This means that each labs submission will consist of two files.

## Lab Test Report

A lab report will be submitted with each assignment and is required to be in **PDF** format. Hand-written, simple text, MS Word, and similar reports will not be accepted. Systems such as MS Word and LATEX are ideal for producing properly formatted PDFs. There are free packages that claim to produce documents compatible with MS Word and these include OpenOffice, FreeOffice, LibreOffice, and Google Docs.

### Lab Test Report Structure

The report has a formal structure. An example report template is provided for in file [lab1-report-example.pdf](lab1-report-example.pdf), see also [Lab 1 Report (Example)](lab1-report-example.md) on the lab website. Seek clarification from the instructor when some issue is not clear.

You will document your tests and testing strategies to show that your code meets the lab requirements (see each rubric). Do not include source code except in those (rare) cases where a code snippet would greatly improve understanding. Function prototypes and structure declarations are okay to include and not subject to this restriction. Every test must have a corresponding output demonstrating the test. You risk losing points if you fail to properly document your tests; "it is obvious" does not suffice.

Tests must follow this format:

1. Test Name
2. Test Description
3. Expected Results
4. Test Output / Actual Results
5. Discussion
6. Indication of PASS/FAIL

For "Test Output", you are required to use screen shots, not copy-paste. Screen shots with white backgrounds are easier to grade. The lab rubric will list all required tests.

Some labs will provide useful test programs that will let you test some of your code. You are free to use any provided test code to show that you meet the relevant requirements, but you must describe how each of the provided tests work and what requirements are specifically tested in order to get any credit for that test.

*Note: It is possible that some feature of your lab will not be fully functional. If so, you must include a test that shows that the feature fails. You must have a specific test that shows the failure. A lab that does not compile cannot show tests for specific features. If this is unclear, ask.*

## Before You Submit

You are required to test that your submission works properly before submission. There is a `Makefile` target to help you. You are **required** to run `make dist-test` to verify that your kernel compiles and runs correctly, including all tests.
A common error is to not put the name of new files into `runoff.list`.
Once you have verified that the submission is correct, use `make tar` to build the archive file that you will submit. Submit your lab report as a separate PDF document.
**Failure to submit your report in PDF format will result in a grade of zero for the report. No exceptions.**
