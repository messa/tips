Strace
======

Strace tracks syscalls between user process and operating system (Linux, BSD, etc.)

Description from the manpage:

```
DESCRIPTION
       In the simplest case strace runs the specified command until it  exits.
       It  intercepts  and  records  the  system  calls  which are called by a
       process and the signals which are received by a process.  The  name  of
       each  system  call,  its  arguments and its return value are printed on
       standard error or to the file specified with the -o option.

       strace is a useful diagnostic, instructional, and debugging tool.  Sys‐
       tem  administrators,  diagnosticians  and trouble-shooters will find it
       invaluable for solving problems with programs for which the  source  is
       not  readily available since they do not need to be recompiled in order
       to trace them.  Students, hackers and the overly-curious will find that
       a  great  deal  can  be  learned about a system and its system calls by
       tracing even ordinary programs.  And programmers will find  that  since
       system  calls  and  signals  are  events that happen at the user/kernel
       interface, a close examination of this boundary is very useful for  bug
       isolation, sanity checking and attempting to capture race conditions.
```


Example
-------

```
$ strace echo Hello, World!
execve("/bin/echo", ["echo", "Hello,", "World!"], [/* 52 vars */]) = 0
brk(NULL)                               = 0x5645ec78e000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f58a4d8c000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=139787, ...}) = 0
mmap(NULL, 139787, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f58a4d69000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\0\4\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1689360, ...}) = 0
mmap(NULL, 3795296, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f58a47cd000
mprotect(0x7f58a4962000, 2097152, PROT_NONE) = 0
mmap(0x7f58a4b62000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x195000) = 0x7f58a4b62000
mmap(0x7f58a4b68000, 14688, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f58a4b68000
close(3)                                = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f58a4d67000
arch_prctl(ARCH_SET_FS, 0x7f58a4d67700) = 0
mprotect(0x7f58a4b62000, 16384, PROT_READ) = 0
mprotect(0x5645ea88a000, 4096, PROT_READ) = 0
mprotect(0x7f58a4d8f000, 4096, PROT_READ) = 0
munmap(0x7f58a4d69000, 139787)          = 0
brk(NULL)                               = 0x5645ec78e000
brk(0x5645ec7af000)                     = 0x5645ec7af000
open("/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=1679776, ...}) = 0
mmap(NULL, 1679776, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f58a4bcc000
close(3)                                = 0
fstat(1, {st_mode=S_IFCHR|0600, st_rdev=makedev(136, 11), ...}) = 0
write(1, "Hello, World!\n", 14Hello, World!
)         = 14
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```

You see how the program loads libraries, allocates memory, writes to standard output and then exits.


strace | grep: filter the strace output for keywords
----------------------------------------------------

Skip lot of noise you know what are you looking for:

```
$ strace echo Hello 2>&1 | grep open
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
open("/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
open("/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
```


strace -f: track child processes
--------------------------------

Many programs are using more than one OS process.

```
       -f          Trace  child  processes  as  they  are created by currently
                   traced processes as a result of the fork(2),  vfork(2)  and
                   clone(2) system calls.  Note that -p PID -f will attach all
                   threads of process PID if it is  multi-threaded,  not  only
                   thread with thread_id = PID.
```


strace -s: print loger strings
------------------------------

By default all strings are truncated:

```
$ strace echo this is a very very very very long string >/dev/null
...
write(1, "this is a very very very very lo"..., 42) = 42
```

You can set the truncate threshold to a larger number:

```
       -s strsize  Specify  the  maximum  string size to print (the default is
                   32).  Note that filenames are not  considered  strings  and
                   are always printed in full.
```

```
$ strace -s 1024 echo this is a very very very very long string >/dev/null
...
write(1, "this is a very very very very long string\n", 42) = 42
```

strace -t: print timestamps
---------------------------

```
       -t          Prefix each line of the trace with the time of day.

       -tt         If given twice, the time printed will include the microsec‐
                   onds.

       -ttt        If  given  thrice,  the  time  printed  will  include   the
                   microseconds and the leading portion will be printed as the
                   number of seconds since the epoch.

       -T          Show the time spent in system calls.  This records the time
                   difference between the beginning and the end of each system
                   call.
```

Example:

```
$ strace -tt echo Hello 
22:16:19.073775 execve("/bin/echo", ["echo", "Hello"], [/* 52 vars */]) = 0
22:16:19.074654 brk(NULL)               = 0x55e8fc80f000
22:16:19.074809 access("/etc/ld.so.nohwcap", F_OK) = -1 ENOENT (No such file or directory)
...
```


Links
-----

[strace.io](https://strace.io/)

[strace(1) manpage](https://linux.die.net/man/1/strace)

[Wikipedia: Strace](https://en.wikipedia.org/wiki/Strace)

[Root.cz: Trasujeme se strace](https://www.root.cz/clanky/trasujeme-se-strace/)

[Root.cz: Trasování a ladění nativních aplikací v Linuxu](https://www.root.cz/clanky/trasovani-a-ladeni-nativnich-aplikaci-v-linuxu/#k06)
