# How to find address of a function within Libc ?
## `15 April 2020`


I was toying with a C program which prints address of variables, functions etc.
My goal was to identify where the allocations are done like on stack, heap etc
The memory mapping of `libc` caught my eye. I thought what if i had to find a address of a function 
which comes from libc. Checked around the web and found these easy to use steps


First your program should be dependant on `libc`, you can find that out via `ldd` command.
Remember path of libc may vary if you compare it with output of `pmaps` command.
This is due to symbolic linking.

```
$ ldd ./mem
	linux-vdso.so.1 (0x00007fff4575e000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f5a43510000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f5a438d2000)
$ 
```

Let's use `objdump` to find address of `printf`


```
$ objdump  -TC /lib64/libc.so.6 | grep " printf$"
0000000000059180 g    DF .text	00000000000000c7  GLIBC_2.2.5 printf
$ 
```

Now let's try to same using `readelf`


```
$ readelf -Ws /lib64/libc.so.6 | grep " printf@@"
   635: 0000000000059180   199 FUNC    GLOBAL DEFAULT   14 printf@@GLIBC_2.2.5
$ 
```


And finally use `nm` utility to find the same information.
All 3 of them list the address as `59180`

```
$ nm -D /lib64/libc.so.6 | grep " printf$"
0000000000059180 T printf
$ 
```

We'll lets use gdb as well to see if we get same information.
Load the libc in gdb and hit `print printf`. We see the same address.


```
$ gdb -q /usr/lib64/libc-2.28.so
Reading symbols from /usr/lib64/libc-2.28.so...(no debugging symbols found)...done.
Missing separate debuginfos, use: yum debuginfo-install glibc-2.28-101.el8.x86_64
(gdb) 
(gdb) 
(gdb) print printf
$1 = {<text variable, no debug info>} 0x59180 <printf>
(gdb) 
```

However when we see live output of `pmaps` command or contents of `maps` when the process
is actually running, we can see the address range where `libc` is currently mapped.


```
$ cat /proc/$(pgrep mem)/maps
00400000-00401000 r-xp 00000000 fd:00 101232209                          /LM/mem
00600000-00601000 r--p 00000000 fd:00 101232209                          /LM/mem
00601000-00602000 rw-p 00001000 fd:00 101232209                          /LM/mem
00f6f000-00f90000 rw-p 00000000 00:00 0                                  [heap]
7f871f30d000-7f871f4c6000 r-xp 00000000 fd:00 33555576                   /usr/lib64/libc-2.28.so
7f871f4c6000-7f871f6c5000 ---p 001b9000 fd:00 33555576                   /usr/lib64/libc-2.28.so
7f871f6c5000-7f871f6c9000 r--p 001b8000 fd:00 33555576                   /usr/lib64/libc-2.28.so
7f871f6c9000-7f871f6cb000 rw-p 001bc000 fd:00 33555576                   /usr/lib64/libc-2.28.so
7f871f6cb000-7f871f6cf000 rw-p 00000000 00:00 0 
```
