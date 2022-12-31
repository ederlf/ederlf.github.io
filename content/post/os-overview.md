# Operating systems overview

- Operating systems direct operational resources:
    - control of CPU, memory, peripheral
    - How these resources are allocated to applications

- Enforce policies
    - Fair resource usage, limits to usage
    e.g: The number of open files that can be open per process
         Thresholds for memory daemons.

- Helps to achieve complex tasks
    - Simplifies the view of the applications running on the platfom
    - Abstracts hardware details and offers a system call interface for the applications. 

## What is an OS?

Computing systems consist of one or multiple processing elements 

CPUs: Today CPUs are composed of multiple cores
Memory
Network devices
Graphics processing cards
Storage devices

All these hardware components are used by multiple applications.

"The operating system is the layer of system software that sits between complex hardware and the applications"

There is not a formal definition, but we can define the functionalities one should provide

1. Hide the complexity of underlying hardware from application
    - No need to worry about disk sector or blocks
    e.g: Hides complexity of different types of storage
    Interface with applications with Read/write

    - Network devices -> offer a send/recv interface for networking events.

2. Provide resource management
    OS decides how many and how much of the resources the applications can use.

    Overall the OS is responsible for all resource management tasks on behalf of applications

3. The operating system must ensure that resources are isolated, therefore protected.

    - Applications must not access each other's memory. (Protection against reading data and overwriting data.)

In summary:

The OS is a layer of systems software that:
    - Has direct access to the underlying hardware
    - Hides the hardware complexity
    - Manages the hardware for the applications, according to pre-defined policies
    - Ensures that applications are isolated and protected from each other.


Some examples of OSes:
    Desktop/Server
        Windows
        Unix-based -> family of OSes originated from UNIX: MAX OS X (BSD), Linux
    Embedded
        Android
        iOS
        Symbian

## OS elements

Abstractions:
    - process, thread, file, socket, memory page

Mechanisms:
    - create, schedule, open, write, allocate

Policies:
    - least-recently used (LRU), earliest deadline first (EDF)

Memory management example:
    The OS uses an abstraction called memory page (An addressable region of memory of a fixed size)

    Mechanisms to operate on the page
        - allocate, map to process

                            (swapping)                           
    Process <-> page <-> DRAM <-> disk

    The OS uses policies to decide whether the memory is stored in the Disk or DRAM. Least recently used are stored in the disk. (i.e, more likely that this data will not be used anytime soon)

## Design Principles

Separation between mechanisms and policies
    - mechanisms should be flexible to support many policies.
    e.g., LRU, LFU, random

Optimize for the common case
    - Where the OS will be used?
    - What will the user want to execute?
    - What are the workload requirements?

    Based on the common case we pick a specific policy that makes sense and that can be supported given the abstractions that the underlying system supports.

## OS protections boundary

The operating systems operates in privileged system: kernel mode
    - Can access hardware

The applications operate on unprivileged mode: user mode

    Attempts to execute privileged instructions from user node cause a trap. The application will be interrupted and the hardware gives control to the OS.
    
    The OS checks what causes the trap and verifies if the userspace can access the instruction or exit the program with an error.

Systems calls are used by applications to request the OS to perform operation on their behalf. For example, memory allocation, file reading/write

# System call flowchart

user process -> calls system call    return from system call | User mode
               \                       /                            
                \                    return
                 trap, mode bit=0    mode bit= 1             | kernel mode
                  \                 /
                   Execute system call

On a system call control is passed to the OS, in privileged mode, and is executed by the kernel.

Executing a system call changes the execution context, take arguments and jumps in the memory of the kernel to execute the system call,

The kernel can execute any instruction from the system call.

Returning requires to change context again, passing arguments and jumping around in the memory.

System calls are not necessarily cheap operations
    - write arguments
    - save relevant data at a well defined location
    - make system call using the system call number
    
Arguments can be passed directly the user program or indirectly by specifying the address.

Synchronous mode -> wait until the return
Asynchronous mode -> Keep executing

## Crossing the OS Boundary

User/Kernel transitions
    - traps on illegal instructions or memory access requiring special privilege.
    - involves a number of instructions
        e.g, ~ 50 to 100ns on 2GHz machine running Linux
    - switches locality
        Applications may suffer or benefit based on how a context switch changes what is in the cache.
        A cache is hot if the request data is in the cache, cold otherwise.

## OS Services

    The OS has a scheduler component responsible by controlling the access to the CPU

    There is a memory manager

    Block device manager manages disks.

    Services are available via systems calls.

    Examples:
        Send a signal to a process -> kill
        Set the group identity id -> setgid
        Mount a file system -> mount
        read/write system parameters -> sysctl

## Monolithic OS
    
    Every possible service and type of hardware are part of the OS. (Includes several types of file systems, scheduling, file mgt, device drivers, memory management)

    Benefits: everything included -> inlining, compile time optimization

    Downside: too much state, hard to maintain, memory requirements, performance.

## Modular OS
    
    Everything can be added as module.

    The OS specifies interfaces that must be implemented.
    You can install new modules

    + maintainable
    + smaller footprint
    + less resource needs

    negative -> indirection can reduce performance

## Micro kernel

    Implements only the most basic primitives

    Example: Provides process and address space management.

    All the other needs are implemented by the applications.

    + smaller size
    + verifiable

    negative: portability, cost, software development



















