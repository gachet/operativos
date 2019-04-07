# CSCI 375 - Operating Systems

## Labs

Course xv6 repository: https://github.com/wildart/xv6-public.git

First step is setup of a xv6 local development environment. Follow the [xv6 installation](xv6-install.md) instructions.

- [Lab0](lab0.md) (optional, but **highly** recommended, and it will be counted as extra credit)
    - Solution: [head.c](https://github.com/wildart/xv6-public/blob/lab0-solution/head.c)
- [Lab1](lab1.md): System Calls
    - [Rubric](lab1-rubric.md)
    - [Lab 1 Report (Example)](lab1-report-example.md)
    - [Lab 1 Report (Example) in PDF](lab1-report-example.pdf)
    - [Solution](https://github.com/wildart/xv6-public/tree/lab1-solution)
- [Lab2](lab2.md): Processes
    - [Rubric](lab2-rubric.md)
- [Lab3](lab3.md): Process Management
    - [Rubric](lab3-rubric.md)
- [Lab4](lab4.md): MLFQ Scheduler
    - [Rubric](lab4-rubric.md)

## QEMU

You will boot and run your operating system using [QEMU](https://qemu.weilnetz.de/doc/qemu-doc.html), a powerful emulator offering very good performance. You will not be expected to delve into the QEMU layer of the project. The project is structured so that QEMU "looks" like a native X86 processing environment. The principle advantage of QEMU is that dedicated hardware is not necessary. This environment will greatly speed your learning and also shorten development cycles.

### Running xv6 under QEMU

You will run xv6 under the QEMU emulator by using a `make` command.

- `make clean qemu-nox` or `make clean run` will load and run xv6 normally.
- `make clean qemu-nox-gdb` or `make clean debug` will load and run xv6 in debug mode.

To exit xv6, use on of these methods:
1. The `halt` command from the xv6 command line.
2. control-a x. This will terminate the QEMU console and so terminate xv6.
3. control-a c. This will drop you into the QEMU console 5 where you can use `quit` command to exit.

## Resources

- [List of command for xv6 development and debugging](https://pdos.csail.mit.edu/6.828/2014/labguide.html)

- [xv6 Book](xv6-book-rev11.pdf)

- [QEMU](https://qemu.weilnetz.de/doc/qemu-doc.html)

- [Exploring xv6 (video)](https://www.youtube.com/watch?v=ktkAlbcoz7o)

- [More references on various reading materials](https://pdos.csail.mit.edu/6.828/2014/reference.html)

- [Operating System Concepts Essentials](http://codex.cs.yale.edu/avi/os-book/OSE2/index.html) - The textbook’s companion site.

- [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://software.intel.com/en-us/articles/intel-sdm) - If you want the technical down and dirty about all aspects of the IA-32/64 architecture (the instruction set, how interrupts work, etc.), here it is. Multiple volumes and well over a thousand pages.
