# Lab 2 Notes

## Part 2 Use xv6 as Unix

### 1 Three programs shipped with xv6

Three programs included with xv6 are `cat`, `echo`, and `wc`. `cat` prints file contents, `echo` prints its arguments, and `wc` counts lines, words, and bytes. The system also includes programs such as `ls`, `grep`, `sh`, `mkdir`, `rm`, and `kill`.

### 2 OS features required for a pipe

A pipe needs process management so the OS can create and schedule the two programs. It also needs kernel-supported interprocess communication: the kernel creates a pipe with file descriptors and connects the standard output of the first process to the standard input of the second process.

### 3 xv6 shell compared with the Linux shell

The xv6 shell is similar to the Linux shell because it launches programs, passes arguments, supports redirection, and connects commands with pipes. It is much smaller and has fewer commands and interactive conveniences than Bash.

## Part 3 Read the source

### 1 System calls used by user cat

`user/cat.c` uses `open()` to open a named file, `read()` to obtain bytes from a file descriptor, `write()` to send bytes to an output file descriptor, `close()` to release an opened descriptor, and `exit()` to terminate the process. Calls such as `fprintf()` are user-library functions that ultimately use lower-level output services.

### 2 Location of sys read

For the xv6 revision used in this lab, `sys_read` is implemented in `kernel/sysfile.c` at line 69. `sys_write` begins at line 83 in the same file.

### 3 Difference between kernel and user code

Code in `kernel/` runs with privileges and directly manages processes, memory, filesystems, system calls, and devices. Code in `user/` runs without kernel privileges and requests services from the kernel through system calls.

## Program compatibility note

The current upstream xv6 revision declares the tick-delay system call as `pause(int)` instead of `sleep(int)`. The required executable is still named `sleep`, and `user/sleep.c` calls `pause(atoi(argv[1]))` to provide the required delay. It successfully handled both `sleep 10` and the missing-argument case.
