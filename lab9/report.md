# Lab 9 - Watching System Calls with strace

Ubuntu 26.04.1 LTS in VirtualBox, user dalvgame.

## Part 1
`strace -f -e trace=openat,read,write,close ls -l` showed a burst of openat/read/close calls while ls loaded its libraries and read the directory, followed by write calls when it printed. One simple `ls -l` makes roughly 272 system calls in total (the number of lines strace printed for it).

**Which system call prints to the screen, and to which descriptor?** The call is **write**, and it writes to descriptor **1** (standard output): `write(1, "total ...", ...)`.

## Part 2
My program's calls appeared as: openat("note.txt", O_WRONLY|O_CREAT|O_TRUNC, 0644) = 3, write(3, "hello, kernel\n", 14) = 14, close(3) = 0, openat("note.txt", O_RDONLY) = 3, read(3, ..., 63) = 14, close(3) = 0.

**Why is the first descriptor 3 and not 0, 1 or 2?** Descriptors 0, 1 and 2 are already open when the program starts: standard input, standard output and standard error. The shell opens them and the child process inherits them across fork and exec. The kernel always hands out the smallest free number, so the first new file gets 3.

## Part 3
Counting with `strace -c`: (write the numbers you saw here).
