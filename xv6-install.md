xv6 Installation Instructions
=============================

(adopted from [gcallah.github.io](https://gcallah.github.io/OperatingSystems/xv6Install.html))

These are the toolchain installation instructions for all 3 OS Versions:
Mac/Windows/Linux, use the appropriate one for you. We have tried to make
the instructions available natively on every OS, however as Operating
Systems get upgrades, things invariably break.

## Host OS configuration

### NOTES (Please read first)

-   If you are running Mac OS X 10.11 (El Capitan) or above you might
    have trouble installing, because of some OS changes. If you are
    adventurous, you might try this [stackoverflow
    answer](https://stackoverflow.com/questions/33860902/building-gcc-on-os-x-10-1),
    otherwise use the Virtual Machine.
-   If you are running Windows I mentioned that you could use the Linux
    Subsystem for Windows (LSW): this only works if you have the 64 bit
    version of Windows 10, if you don't, just use the Virtual Machine.
    Also LSW is still beta so it will have some bugs.
-   If you are using Mac/Windows and it's taking you a long time (more
    than 60 minutes) to get it sorted out you should strongly consider
    just using the Virtual Machine or come to office hours to see if we
    can help you. Depending on the problem you might still have to use
    the Virtual Machine.

### Virtual Machine Instructions

-   Download VirtualBox from <https://www.virtualbox.org/> and install.
    Make sure to run the install program as an admin see this
    [video](https://www.youtube.com/watch?v=RV3MDnHvWWA) if you need to
    understand what I mean.
-   Install Ubuntu or a Debian-based version.
-   Follow [installation instructions for Linux](#linux-instructions)

### Linux Instructions

-   If you are using another version of Ubuntu or a Debian-based version
    of Linux, installing the tools you'll need is very simple. At a
    terminal type:

          $ sudo apt-get update && sudo apt-get install git nasm build-essential qemu gdb

### Windows Instructions

As mentioned before this will only work if you have 64bit windows 10.

-   Install the Linux Subsystem for Windows (LSW) following the
    instructions
    [here](http://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/)
-   When you are done installing open a "Bash on Ubuntu on Windows"
    terminal console and type the following:

          $ sudo apt-get update && sudo apt-get install git nasm build-essential qemu qemu-system-x86 gdb

-   Modify the makefile. Find the following line:

          QEMUOPTS = -hdb fs.img xv6.img -smp $(CPUS) -m 512 $(QEMUEXTRA)

-   Modify it to this:

          QEMUOPTS = -hdb fs.img xv6.img -smp $(CPUS) -m 512 $(QEMUEXTRA) -display none

## MacOS Instructions

-   Install VirtualBox.
-   Follow [Virtual Machine Instructions](#virtual-machine-instructions)

## Installing xv6

-   Run following command

        $ git clone https://github.com/wildart/xv6-public.git

-   Make sure you can build and run xv6. To build the OS, use cd to change to the xv6 directory, and then run make to compile xv6:

        $ cd xv6-piblic
        $ make

-   Then, to run it inside of QEMU, you can do:

        $ make qemu

QEMU should appear and show the xv6 command prompt, where you can run programs inside xv6. It will look something like:

    qemu-system-i386 -nographic -drive file=fs.img,index=1,media=disk,format=raw -drive file=xv6.img,index=0,media=disk,format=raw -smp 2 -m 512 
    xv6...
    cpu1: starting 1
    cpu0: starting 0
    sb: size 1000 nblocks 941 ninodes 200 nlog 30 logstart 2 inodestart 32 bmap start 58
    init: starting sh
    $ 



You can play around with running commands such as `ls` , `cat` , etc. by typing them into the QEMU window; for example, this is what it looks like when you run ls in xv6:

    $ ls
    .              1 1 512
    ..             1 1 512
    README         2 2 2327
    cat            2 3 13352
    echo           2 4 12432
    forktest       2 5 8140
    grep           2 6 15180
    init           2 7 13020
    kill           2 8 12476
    ln             2 9 12380
    ls             2 10 14596
    mkdir          2 11 12496
    rm             2 12 12472
    sh             2 13 23116
    stressfs       2 14 13152
    usertests      2 15 56032
    wc             2 16 14012
    zombie         2 17 12204
    console        3 18 0
    $ 

-   Press `Ctrl+a c` to get to QEMU console and type `quit` for stopping xv6. *Note: Alternative combination for QEMU is `Ctrl+Alt+a c`.*
