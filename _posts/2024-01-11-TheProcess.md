---
title: "OS: Three Easy Pieces 1"
date: 2024-01-11 01:19:00 +530
categories: [notes, os]
tags: [os, theory]
---

# The Abstraction: The Process

- A process is just a **running program**.
- Program by itself is just a bunch of instructions and data sitting on the disk.

**`How to provide the illusion of many CPUs?`**

- This is achieved by `virtualisation` and `time sharing`, which allows the resource to be used by one entity for a while, and then by another.
- To implement virtualisation we need `low-level machinery`(context switch) and `high-level intelligence`(policies).
- Policies are algorithms that make some kind of decision within OS. A `scheduling policy` is responsible for scheduling process to be run.

## A Process

- We summarize a process by the pieces of system it accesses/effects during it runtime _(machine state)_.
- **Machine state** -> The parts of the machine important during execution.

1. `Address Space` -> The memory that the process can address.
2. `Registers` -> Registers like `Instruction Pointer` and `Stack Pointer`.
3. `I/O information` -> The list of open files.

**Low level mechanisms are separated from high level policies to promote modularity.**

## Process API

- **Create** a new process
- **Destroy** kill forcefully
- **Wait** for completion
- **Misc. Control** like suspend and resume
- **Status** information about state and runtime

## Process Creation

![loading: from program to process](/assets/images/3.1.png)

- Load the code and static data into memory.
- `Eager Loading` means loading all at once.
- `Lazy loading` means loading when required.(aging and swapping used)
- `Stack` allocation and intitialization with _argc_ and _argv_.
- `Heap` allocation for `malloc()` and `free()`
- There are three `file descriptors` open by default on **UNIX** systems.
- OS transfers control of CPU to process after jumping to`main()`.

## Process States

![state transitions](/assets/images/3.2.png)

- **Running** process is executing instructions on CPU
- **Ready** able to run but OS has chosen not to run it at this given moment
- **Blocked** process has performed some kind of operation that makes it not ready to run until some other event occurs, ex: `I/O`.

![tracing just cpu](/assets/images/3.3.png)

A process is moved between the ready and running states at the discretion of the OS.

- `Scheduling` -> Moving from ready to running states.
- `Unscheduling` -> Just the opposite.

![tracing cpu and i/o](/assets/images/3.4.png)

OS **Scheduler** is responsible for deciding the state of processes.

## Data Structures

- OS has key data structures that track relevant pieces of information.
- The OS will keep a `process list` to track ready processes, some additional information to track running processes and blocked processes.
- Individual structure that holds information about a process is called `Process Control Block(PCB)` or `process descriptor`.

![xv6 proc structure](/assets/images/3.5.png)

- `Register context` holds the contents of the registers of a stopped process.
- Sometimes the process will have an `initial state` at the time of creation.
- It could be placed in a `final state` where it has exited but has not been cleaned up (**zombie**)
- Final state can be useful for the parent process to track the `return code` for the process.
- Parent then calls `wait()` to wait for the completion of `child`and indicates the OS that it can clean up the data structures relevant to the child process.
