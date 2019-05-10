# Final Exam


## When & where

- **Fri 5/17/2019, 6:00 p.m.-8:00 p.m.**
- Room: 6.64.02


## Exam Rules
Be sure to arrive on time. If you arrive after the exam starts, you will not be allowed to take it.

This will be a **closed book**, **closed notes exam**. Calculators, phones, laptops, and tablets are neither needed nor permitted. If you have these devices, you must turn them off, put them out of sight, and not access them for the duration of the exam.

## Topics

You are responsible for the material from since the start of the semester.

Topics that you should know and may be on the exam include:

### OS Concepts

**Text:** 2.1-2.4 (p.55-73), 2.6.2 (p. 76), 2.7-2.7.5.3 (p. 78-86)

- Monolithic vs. modular vs. microkernel structures
- User mode vs. kernel mode (= privileged mode or supervisor mode)
- Getting from user to kernel mode and back again
- Traps (aka software interrupts): INT vs. SYSCALL instructions
- System calls
- Programmable interval timer interrupts: why do you need them?
- Mode switch and context switch
- What's invoved in saving process state?
- Device types: character, block, and network
- Interacting with devices: memory mapped I/O
- Getting status from devices: interrupts and polling
- Moving data to/from devices: programmed I/O or DMA

### Processes

**Text:** 3.0-3.3.2 (p.105-122)

- Program vs. process
- Memory map: text, data, bss, heap, stack (don't memorize the list but know what each is)
- Process states: ready, running, blocked (aka waiting), zombie; transitions between states
- Preemptive multitasking
- Process control block (PCB): don't memorize the list of what it stores but know that it at least stores the machine state, process state, memory map, and process ID.
- Process list: keeps track of PCBs.
- Creating a process under Unix with `fork`: know that it creates a copy of the parent
- What is a zombie process?
- Know how a process can tell if it is the parent or the child after a `fork`


### Threads

**Text:** 4.0-4.6 (p.163-188)

- What's a thread vs. a process?
- What do threads in a process share? entire memory map, open files, permissions
- What's a thread control block (TCB)?
- Kernel-level versus user-level threads. Advantages & disadvatages of each.
- Hybrid kernel/user models.
- Thread create and join vs. fork/wait.


### Synchronization

**Text:** 5.0-5.9 (p.203-238), 5.11 (Deadlocks, p. 242-249)

- Definitions: concurrent, asynchronous, independent, synchronous
- What is a race condition?
- Definitions: critical section, mutual exclusion, deadlock, starvation
- What is a spinlock (aka busy waiting)?
- Problems with disabling interrupts.
- Problems with test and set locks in software
- You don't have to know Peterson's algorithm
- Test and set instruction: how does it work?
- Compare and swap instruction: how does it work?
- Fetch and increment instruction: how does it work?
- Avoiding spin locks - turn to OS
- Priority inversion
- Semaphores
    - What is a semaphore?
    - What are the operations on it? Understand initialization, up, and down
    - When does a semaphore put threads to sleep and wake them up?
    - Understand the producer-consumer example 
- Deadlock
    - Conditions for deadlock
    - Wait-for graph
    - Ways of handling: prevention, avoidance, detection, ignore


### Process Scheduling

**Text:** 6.0-6.5.4 (p. 259-291), 6.7-6.7.3 (p.298-307)

- CPU bursts and I/O bursts
- What does a scheduler do?
- When does a scheduler get a chance to dispatch (run) a process?
- Preemptive versus cooperative schedulers
- Turnaround time, response time,
- First-come, first served scheduling
    - What's the scheduling algorithm?
    - Disadvantages
- Round-robin scheduling
    - What's the scheduling algorithm?
    - What is a quantum (aka time slice)?
    - Pros and cons of big and small time slices
    - What are the key advantages and disadvantages?
- Shortest remaining time first scheduling
    - What's the scheduling algorithm?
    - What is the main purpose/advantage?
    - What is the main drawback?
    - Estimating CPU burst time: don't memorize the function but understand that it's a weighted exponential average that is a function of the last measured CPU burst and past history.
- Priority scheduling
    - What's the scheduling algorithm?
    - What is the main purpose/advantage?
    - What is the main drawback?
    - Static vs. dynamic priorities
    - What is starvation?
    - What is process aging?
- Multilevel feedback queues
- Difference from multilevel queues
- Priority classes
- Understand how processes trickle to lower levels based on CPU burst
- Role of process aging
- Advantages & disadvantages
- I will not ask you about guaranteed scheduling, or the completely fair scheduler


### Memory Management

**Text:** 7.1-7.9 (p. 325-365), 8.1-8.9.3 (p. 371-416)

- Static vs. dynamic linking, shared libraries
- Monoprogramming vs multiprogramming
- Single partition monoprogramming
- CPU utilization
- Dynamically relocatable code
- Memory management unit: concept of dynamic address translation
- Logical (virtual) vs Physical (real) addresses
- Base & limit addressing
- Multiple fixed partitions (internal vs. external fragmentation)
- Variable partition multiprogramming
- Segmentation
- Allocation algorithms: first fit, best fit, worst fit.
- Memory compaction (relocation to combine holes)
- Page-based virtual memory:
    - page number and offset (displacement)
    - page table, PTE, page fault
    - direct mapping paging system, page table base register
    - translation lookaside buffer (TLB)
    - hit ratio
    - address space identifier (ASID): why do you want it?
    - multilevel (hierarchical) page tables, partial page table: understand benefit
- Understand the benefits of page-based virtual memory
- Demand paging: understand know the concept
- Memory-mapped files
- Have a good idea of what the page fault handler does, not any specific steps
- Page replacement algorithms
    - FIFO replacement
    - LRU replacement (why can't we use this?)
    - Not frequently used replacement
    - Clock (second chance) replacement
- Locality and the working set
- Working set model
- Thrashing
- Resident set
- Using page fault frequency to manage resident sets


### Mass-Storage Structure 

**Text:** 9.1.1-9.1.2, 9.4-9.5.1; (p. 441-443, 446-454, 457-459, 462-469)

- Disk scheduling algorithms: SCAN, LOOK, C-SCAN, C-LOOK, SSTF, FCFS


### File Systems

**Text:** 10.1-10.3.5, 10.4 (p. 477-496, 500-502), 11.1-11.7.2 (p. 517-544)

- Terms
    - Disk
    - Block
    - Partition
    - Volume
    - Track
    - Cylinder
    - Seek
    - File
    - File name
    - File data
    - Metadata
    - Attribute
    - Directory
    - Superblock
    - inode
    - Clusters vs. Extents
    - Internal vs. external fragmentation
- Virtual File System (VFS)
    - Superblock: just know that it keeps key parameters about the file system: size, name, type
    - inode: what kind of info does it store at the VFS layer?
    - directory entry (dentry)
    - file: don't memorize the file operations but a few of them should make sense
- Mount points: kernel needs to keep a track of mounted file systems
- Implementing file systems
    - Allocation:
        - contiguous
        - extents
        - linked allocation
        - file allocation table (FAT)
        - indexed (inode); combined indexing with indirect blocks
    - Directories - possible structures: linear list, hash, B-Tree
    - Hard links vs. symbolic links
    - Functions: understand the basic operations; don't memorize steps
        - mounting
        - union mounts
        - file system checking (don't memorize steps but understand the point of it)
        - steps in creating files (don't memorize but understand)
        - creating directories: what's different from creating a file?
        - opening a file
        - writing to a file (don't memorize steps but understand what needs to be done if the file has to grow; also, realize that sometimes you need to read to modify just parts of a block)
        - reading from a file
        - deleting a file (understand that it's about reference counting)
- just the basics of a log-structured file system: all data, inode, and metadata changes are written as logs
