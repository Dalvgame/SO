# Lab 5 - The File System with Your Own Hands (Linux)

Ubuntu 26.04.1 LTS in VirtualBox, user dalvgame (not root). Working folder: ~/os-lab5

## Part 1. The tree and the file types
1. Types from the first letter of `ls -l`: /etc = d (directory), /etc/passwd = - (regular file), /bin = l (symbolic link), /dev/null = c (character device), /tmp = d (directory, with the sticky bit t).
2. /bin is a symbolic link to usr/bin (`/bin -> usr/bin`).
3. Character device: /dev/null (also /dev/console). Block device: /dev/sda (partitions /dev/sda1, /dev/sda2).
4. /usr is 4.8G and /var/log is 192M, so /usr is about 25 times bigger.

## Part 2. Inodes and links
1. In `stat`, `Inode: 1332217` is the inode number and `Links:` is the number of names the file has.
2. After `ln notes.txt hard.txt` both names show inode 1332217 and link count 2. `rm notes.txt` removes only the name (a directory entry) and decreases the link count to 1. The data stays until the count is 0, so `cat hard.txt` still printed hello.
3. `soft.txt` stores only the path text `hard.txt`, so it is 8 bytes (8 characters). After `rm hard.txt` the path points to nothing (a dangling link), so `cat soft.txt` fails although the link still exists.
4. A new directory has 2 links: its name in the parent directory, and its own `.` entry. Each subdirectory adds one more link through its `..` entry, so two subdirectories give 4.
5. A file uses whole 4096-byte blocks, so a 2-byte file takes 4.0K. `truncate -s 1G` makes a sparse file: the size is 1 GB but no data blocks are allocated, so `du` shows 0.

## Part 3. Permissions
1. 600 = rw------- (a private file, for example an SSH key). 644 = rw-r--r-- (an ordinary document). 755 = rwxr-xr-x (a program or a public directory). 700 = rwx------ (a private directory such as ~/.ssh).
2. The kernel checks the mode bits, and the owner has no special rights. With mode 000 the owner is refused too. The owner can change the mode back with chmod (`chmod u+r`), which is what the next step did.
3. umask is 0002. Files start from 666 and directories from 777; removing the bits of the umask gives 664 (rw-rw-r--) for new.txt and 775 (rwxrwxr-x) for newdir.
4. On a directory, r allows listing the names (`ls box` worked, but `cat box/f` was denied); x allows entering it and reaching an item by name (`cat box/f` worked, but `ls box` was denied).
5. The s in `-rwsr-xr-x /usr/bin/passwd` is the setuid bit: the program runs with the rights of its owner (root). passwd needs it because it must write the password file /etc/shadow, which only root can read (`-rw-r----- root shadow`). The t in `drwxrwxrwt /tmp` is the sticky bit: everybody can create files there, but only the owner of a file can delete or rename it.

## Part 4. Disks and the kernel as files
1. Disk sda (31G) has two partitions: sda1 (1M) and sda2 (31G). sda2 is mounted at / with the file system type ext4.
2. The root file system has 12G used of 31G (40%).
3. /proc/cpuinfo shows 2 CPU cores, and MemTotal is 3480328 kB (about 3.3 GiB of RAM).
4. /proc/self/status showed the name `head`, because /proc/self always refers to the process that reads it, and that process was `head`.

## Stretch
(a) `gzip -c /etc/passwd > notes.txt ; file notes.txt` reports gzip data. The first bytes `1f 8b` (the magic number) tell `file` the real type, not the file name.
(b) `find / -xdev -perm -4000 -type f` lists the setuid programs, for example passwd, su, sudo, mount, umount, pkexec, chsh, chfn, gpasswd, fusermount3, ntfs-3g and pppd.
(c) After `exec 3< bigfile ; rm bigfile`, `ls -l /proc/$$/fd` shows `3 -> .../bigfile (deleted)`: the name is gone but the open handle keeps the data; the space is freed only when the last handle is closed (`exec 3<&-`).
(d) `mkfs.ext4` on a 64 MB image created 16384 inodes (`df -i /mnt`: 16384 total, 12 used).

## Conclusion
Names, inodes and data blocks are separate things: a name is only a directory entry, the inode holds the metadata and the link count, and the data is freed when the last link and the last open handle are gone. Permissions are checked against the mode bits (owner, group, others), and the special bits s and t solve specific problems. Disks, partitions and even kernel information are available as files under /dev and /proc.
