---
title: "OS: Three Easy Pieces 3"
date: 2024-01-23 09:48:00 +530
categories: [notes, os]
tags: [os, theory]
---

# Mechanism: Limited Direct Execution

- How to implement virtualization without adding excessive overhead to the system?
- How can we run processes efficiently while retaining control of CPU?

![Without limits](/assets/images/6-1.png)
> The OS must virtualize the CPU in an efficient manner while retaining control over the system. It uses hardware support in order to do this effectively.
{: .prompt-tip}

## Limited Direct Execution

- `Direct execution` -> just run the program directly on the CPU, after creating a process entry on the process list for it.
- Without `Limits` the OS will just be a library and wouldn't be in control of anything.

## Problem 1: Restricted Operations

- A process must be able to perform I/O and other restricted operations without gaining complete control on the system. We can't let any process do whatever it wants (security risk).

> System calls look like procedure calls, except they use the **trap** instruction to trap into OS.
{: .prompt-info}

- `User Mode` -> Code in this mode can't issue I/O requests, if this happens the processor raises an exception and the OS will likely kill the process.
- `Kernel Mode` -> In this mode, the code that runs can do what it likes, including privileged operations. OS(or kernel) runs in this mode.
- `System Calls` allow the kernel to carefully expose key pieces of functionality. To execute them, a program must execute a special `trap` instruction.

> Essentially, user mode and kernel mode are used for **protected control transfer**.
{: .prompt-info}

- `Trap` instruction simultaneously jumps into the kernel and raises the privilege level to *kernel mode*.
- `Return from trap` instruction is called by the OS which returns to the user program and simultaneously reduces the privelege level.

![With limits](/assets/images/6-2.png)

> The hardware needs to be careful to store enough of the caller's registers (on kernel stack on x86) when executing a trap instruction.
{: .prompt-warning}

- `Trap Table` is set up by the kernel at boot time. It specifies which code to run inside the OS when a trap instruction is encountered. This prevents *arbitrary code execution*. The OS informs the hardware of the locations of these `trap handlers`, usually with some kind of special instruction.

- `System-call number` is assigned to each system call. The OS examines this number set by the user program and handles the system call accordingly.

## Problem 2: Switching Between Processes

- How can the OS **regain control** of the CPU so that it can switch between processes?

### A Cooperative Approach: Wait for System Calls

- In this style, the OS *trusts* the processes to behave reasonably and processes that run for too long are assumed to periodically give up the CPU so that the OS can decide to run some other task.
- Systems like these often have a `yield` syscall to transfer control back to the OS.
- This approach is less than ideal, as what would happen if a process enters an infinite loop and the never makes syscalls.

### A Non-Cooperative Approach: The OS Takes Control

> How can the OS take control of the CPU even if processes are not being cooperative and hence avoid rogue processes?
{: .prompt-warning}

- A `timer interrupt` can be programmed to raise an interrupt every so many milliseconds, which halts the current process and runs a preconfigured `interrupt handler` in the OS.
- At boot time, the OS must start the timer (privileged operation) so that it feels safe that the control will be handed back to it eventually.
- Hardware has the responsibility of saving enough of the program state (similar to explicit syscalls).

![Timer interrupt](/assets/images/6-3.png)

### Saving and Restoring Context

- `Scheduler` decides whether to continue running a process or switch to a different one.
- `Context switch` is a low-level piece of code that is executed if the decision to switch is made.
- By switching stacks, the kernel enters the switch code in the context of one process and returns in the context of another.

- Note that there are two types of register saves/restores that happen during this protocol, one when the timer interrupt occurs and the other when the OS decides to switch

## What about Concurrency ?

![xv6 Context Switch](/assets/images/6-4.png)

- The OS needs to be concerned about what happens when during interrupt/trap handling, another interrupt occurs.
- One simple solution is to **disable interrupts** during interrupt processing, this ensures only one interrupt occurs at a time.
- Disabling interrupts for too long could lead to lost interrupts.
- The OS has sophisticated `locking schemes` to protect concurrent access to internal data structures.

> **Locking** can lead to hard-to-find and interesting bugs
{: .prompt-warning}