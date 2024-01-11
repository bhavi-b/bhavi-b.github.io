---
title: "OS: Three Easy Pieces 2"
date: 2024-01-11 15:21:00 +530
categories: [notes, os]
tags: [os, theory]
---

# Interlude: Process API

- We will learn about `fork()`, `exec()` and `wait()` routines.

> What interfaces should the OS present for process creation and control with high functionality, performance and ease of use?
{: .prompt-tip }

## fork() syscall

- It is used to create a new process.
- **Process Identifier (PID)** is used to identify the process.
- The process created using `fork()` is an almost exact copy of the calling process.

![p1.c fork()](/assets/images/5.1.png)

- The `child` has its own copy of the address space of the `parent`
- Parent receives the PID while the child receives 0 as the return code from `fork()`.

![p1 output](/assets/images/5.1a.png)

- The output is **non-deterministic** as we dont know which process will execute first (upto the scheduler).
- This non-determinism leads to interesting problems in `multi-threaded programs`.

## wait() syscall

![p2.c fork() wait()](/assets/images/5.2.png)

- Parent process calls `wait()` or `waitpid()` for the child to finish execution.

![p2 output](/assets/images/5.2a.png)

- This way the output becomes **deterministic**.

## exec() syscall

![p3 fork() wait() exec()](/assets/images/5.3.png)

- Used to run a program different from calling program.
- Calling `exec()` loads the code and data from the executable to be run and overwrites the current code and static data segment.
- Heap, stack and other parts of the memory space are reinitialized.

![p3 output](/assets/images/5.3a.png)

> Essentially the program calling `exec()` is transformed into a different program and hence a successful call to it never returns.
{: .prompt-info}

## Why this API?

- Separating `fork()` and `exec()` is essential in building a UNIX shell, as it allows running code between the two syscalls and hence alters the `environment` of the program about to be run.
- Shell is just a **prompt** waiting for input.

![shell input redirection](/assets/images/5.3b.png)

- In the example, the output is redirected in to `newfile.txt`. This is accomplished by closing the `stdout` before `exec()` and use the file descriptor for `newfile.txt` as the output stream.
- UNIX look for new file descriptors starting at zero, so `STDOUT_FILENO` will be the first available one. Hence, all output from routines such as `printf()` will also be redirected.

![all of the above with redirection](/assets/images/5.4.png)

- UNIX pipes are implemented similarly, with `pipe()`. In our case we use an `in-kernel pipe` which connects the output of one process to the input of another, example: `grep -o foo file | wc -l`

## Process Control and Users

- `kill()` syscall is used to send **signals**(pause,die,etc.) to a process.
- In UNIX shells, `SIGINT`-> **CTRL+C** interrupts(normally terminates), `SIGTSTP` -> **CTRL+Z** pauses mid execution. Mostly, `fg` command resumes the execution.
- `signal()` is used to catch various signals and execute code accordingly.

> Signals are used to deliver external events to a process or **process groups**.
{: .prompt-info}

- **User** is a concept that ensures that no malicious entity sends signals to processes. Normally, a user has full control over their own processes.

## Useful Tools

- `ps` is used to list currently running processes.
- `top` displays the running processes and their resource usage.
- `kill` and `killall` can be used to send signals to processes and must be used carefully.

> Superuser(Root) is the administrator of the system and is not limited in any way. Remember: **With great power, comes great responsibility**.
{: .prompt-tip}
