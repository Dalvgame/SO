# Lab 6 - Inside the xv6 File System (notes)

## Part 1
1. Types in kernel/stat.h: 1 = directory (T_DIR), 2 = file (T_FILE), 3 = device (T_DEVICE).
2. The root directory is inode 1. It has no parent, so its ".." points to itself and "." and ".." are the same entry.
3. One directory entry is 16 bytes (2 bytes for the inode number + 14 bytes for the name). An empty directory a contains "." and "..", so 2 x 16 = 32 bytes.

## Part 2
1. After `ln README r2`: README and r2 show the same inode (2) and links 2. After `rm README`: r2 still shows inode 2, but links 1, and the data is still there (`cat r2 | wc` gives 48 336 2441). The name was removed, the file was not.
2. In kernel/sysfile.c, sys_unlink decrements the count with `ip->nlink--` (line 244). The data blocks are freed only when the link count is 0 and no process has the file open.
3. struct dinode has only: type, major, minor, nlink, size, addrs[]. There is no owner, no permission field and no time field, so xv6 has no permissions; and nothing stores a path, so there are no symbolic links.

## Part 3
1. ls shows `averyveryveryl`: the name was cut to 14 characters, because DIRSIZ is 14.
2. Largest file = (12 direct + 256 indirect) x 1024 bytes = 268 x 1024 = 274432 bytes (268 KB). (NINDIRECT = 1024 / 4 = 256.)
3. nmeta line: 2000 blocks in total = 2 (boot + super) + 31 log + 13 inodes + 1 bitmap + 1953 data blocks.

## Part 4 (find)
find . r2 printed ./a/b/r2, ./a/r2 and ./r2; find a b printed a/b.
Stretch: find . r2 -n shows ./r2 has inode 3, the same inode as cat, so ./r2 is the same file as cat (a hard link). The other two r2 files are different files.
