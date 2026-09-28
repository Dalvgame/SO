# Operating Systems Lab 2

## Meet the OS You Will Build

This laboratory introduced xv6, a small Unix-like teaching operating system for RISC-V. I installed the RISC-V and QEMU tools, built xv6 from source, booted it in QEMU, used its shell, examined both user and kernel code, and added a new `sleep` user program.

## Environment and build

The work was completed in Ubuntu under WSL2. The xv6 kernel and user programs were cross-compiled for RISC-V and executed with `qemu-system-riscv64`.

The successful boot displayed:

```text
xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
```

The `hart` messages show additional emulated RISC-V CPU cores starting. `init` then launched the xv6 shell.

## Part 1 Build and boot

I built and launched xv6 using:

```bash
make qemu
```

The current Ubuntu 26.04 package providing the RISC-V emulator is `qemu-system-riscv`. Once installed, the build completed and QEMU successfully booted xv6.

## Part 2 Use xv6

### Programs available in xv6

Three programs included with xv6 are:

- `cat`, which reads files and writes their contents to standard output.
- `echo`, which writes its arguments to standard output.
- `wc`, which counts lines, words, and bytes.

Other available programs included `ls`, `grep`, `sh`, `mkdir`, `rm`, and `kill`.

### Pipe operation

I ran:

```text
$ ls | grep c
cat            2 3 36728
echo           2 4 35592
wc             2 18 37728
sync           2 23 34944
console        3 24 0
```

A pipe requires at least two OS features. First, the OS must create and schedule separate processes for the commands. Second, the kernel must provide interprocess communication through a pipe and connect the first process's standard output file descriptor to the second process's standard input file descriptor.

### xv6 shell compared with Linux

The xv6 shell resembles a Linux shell because it can launch programs, pass arguments, redirect input and output, and connect commands with pipes. It is much smaller and provides fewer programs, editing features, and conveniences than Bash on Linux.

### Other observed output

```text
$ echo hello xv6
hello xv6
$ wc README
48 336 2441 README
```

## Part 3 Read the source

### System calls used by `cat`

The xv6 `user/cat.c` program uses these important system calls:

- `open()` asks the kernel to open a named file and return a file descriptor.
- `read()` asks the kernel to copy bytes from a file descriptor into the program's buffer.
- `write()` asks the kernel to copy bytes from the buffer to an output file descriptor.
- `close()` tells the kernel that the program has finished using a file descriptor.
- `exit()` terminates the process and returns a status value.

The implementation of `sys_read` was found in `kernel/sysfile.c` at line 69. The implementation of `sys_write` was found in the same file at line 83. These line numbers correspond to the xv6 revision used for this laboratory.

### Kernel code and user code

Code under `kernel/` runs with machine privileges and implements process management, memory management, the filesystem, devices, and system calls. Code under `user/` runs without those privileges and must request kernel services through system calls.

The user program is therefore the requester, while the kernel validates and performs the requested operation.

## Part 4 The sleep program

The new program is stored in [`user/sleep.c`](user/sleep.c). It:

1. Checks that exactly one argument was supplied.
2. Prints `usage: sleep <ticks>` to file descriptor 2 when the argument is missing.
3. Converts the argument from text to an integer with `atoi()`.
4. Requests that the kernel pause the process for that number of clock ticks.
5. Exits with status 0 after the delay.

The current upstream xv6 revision names the tick-delay system call `pause(int)` rather than `sleep(int)`. For that reason, the required program is still named `sleep`, but its implementation calls:

```c
pause(atoi(argv[1]));
```

The `UPROGS` list in the Makefile includes `$U/_sleep\`, which causes the build to compile the program and place it in the xv6 filesystem image.

### Test results

```text
$ ls | grep sleep
sleep          2 5 35280
$ sleep 10
$ sleep
usage: sleep <ticks>
```

The first command confirmed that the executable was installed in the xv6 filesystem. `sleep 10` paused and returned normally. Running it without an argument exercised the usage check and produced the expected error message.

## Conclusion

This laboratory connected the visible behavior of a Unix-like system to its implementation. I built and booted an operating system, used processes and a pipe from its shell, traced user-level calls into kernel system-call handlers, and added a program that invokes a real kernel service.
