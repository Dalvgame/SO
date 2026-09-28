# Operating Systems Labs

- [Lab 1 report](report.md)
- [Lab 1 terminal session](lab1-session.txt)
- [Lab 2 report](lab2/README.md)
- [Lab 2 required notes](lab2/notes.md)
- [Lab 2 terminal session](lab2/lab2-session.txt)

# Lab 1

## The OS as a Resource Manager

This laboratory explored how Linux manages four central resources: files and directories, processes, memory, and devices and storage. The commands were executed in Ubuntu under WSL2, using Linux kernel `6.18.33.2-microsoft-standard-WSL2` on the `x86_64` architecture.

## Part 1 Files and directories

### Observations

1. The files I created were owned by user `userdalvgame` and group `userdalvgame`.
2. After `chmod 600 note.txt`, the ten permission characters were `-rw-------`. The first character, `-`, identifies a regular file. The owner has read and write permissions, while the group and other users have no permissions.
3. Two directories directly under `/` are:
   - `/etc`, which contains system and application configuration files.
   - `/home`, which contains users' home directories.

### Interesting command output

```text
-rw------- 1 userdalvgame userdalvgame 24 Sep 28 16:58 note.txt
```

This output showed the owner, group, size, and owner-only permissions applied to `note.txt`.

## Part 2 Processes

### Observations

1. Process 1 had PID `1` and was `systemd`, running as `/sbin/init`. It was the initial process and service manager in this Linux environment.
2. `ps aux | wc -l` returned `29`. Since one line was the column heading, this represented approximately 28 process entries at that moment. The number changed slightly while the system was running; `top` later displayed 27 tasks.
3. The sleeping process had state `S (sleeping)`, meaning it was waiting rather than actively using the CPU.

### Interesting command output

```text
PID COMMAND         COMMAND
  1 systemd         /sbin/init
```

This output identified the first process running in the WSL Linux environment.

## Part 3 Memory

### Observations

1. The system had approximately `9.6 GiB` of total RAM. At the moment of measurement, `8.9 GiB` was free and `9.0 GiB` was available.
2. Swap is disk space that the operating system can use for inactive memory pages when needed. The system had `3.0 GiB` of swap configured on `/dev/sdc`, with `0 B` in use.
3. The `sleep` process used `7696 kB` of resident memory (`VmRSS`), or approximately `7.5 MiB`. This was somewhat more than I expected for a program that only waits, but even a small process needs memory for its executable code, libraries, stack, and process data.

### Interesting command output

```text
               total        used        free      shared  buff/cache   available
Mem:           9.6Gi       613Mi       8.9Gi       3.5Mi       212Mi       9.0Gi
Swap:          3.0Gi          0B       3.0Gi
```

This output showed how Linux divided the available physical memory and swap space.

## Part 4 Devices and storage

### Observations

1. The root filesystem `/` was mounted from `/dev/sdd` and used the `ext4` filesystem.
2. `/dev/console` is one entry under `/dev`. It represents the system console and gives programs a file-like interface for communicating with it.
3. "Everything is a file" means that Linux presents devices and other operating-system resources through file-like paths, allowing programs to interact with them through operations similar to reading and writing ordinary files.

### Interesting command output

```text
TARGET SOURCE   FSTYPE OPTIONS
/      /dev/sdd ext4   rw,relatime,discard,errors=remount-ro,data=ordered
```

This output identified the virtual block device and filesystem used for the Linux root directory.

## Conclusion

The operating system manages files and directories, which I examined with `ls -l`, and processes, which I inspected with `ps aux`. It manages memory, which I observed with `free -h`. It also manages devices and storage, which I examined using `lsblk` and `findmnt /`.
