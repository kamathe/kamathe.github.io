# Reversing Hero - Part 1, file
### `9 April 2020`

So we're trying to solve a set of RE challanges called reversing hero.
If you don't know about it please refer to `Part 0` post for more details.

When starting with Reversing a binary, you need to first understand the format of the binary.
Yeah, we have already ran `file` command on the given binary in the earlier post. However we are
going to dig deeper. Remember we are going to take the longer route !

So let's start afresh by running the `file` command again on our target binary.

```
$ file reversinghero 
reversinghero: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), not stripped
$ 
```

Nothing out of the ordinary, Most of what we expected is there.
However since these are notes for my future self, lets call out each of them one by one.

- ELF format        :   Its the standard format used for executables on Linux and most UNIX's
- 64-bit            :   You need a 64-bit machine to run this, also you need to understand 64-bit assembly :)
- x86-64            :   Its meant to be run on `Intel` or `AMD` architecure, it won't run on other archs
- dynamic linking   :   So it utilizes functions from `libc`, we can refer to the docs
- not stripped      :   We'll lets hope you are able to make use of this bonus given by `xorpd`



We got what we came here for ! Should we move on ? Already ?
I must have run `file` a thousand times over on various files and not once did I look how it did what it did.
Yeah yeah, we have all read about the `magic bytes` and how they are stored in initial memory location of binary.
And yeah we know how `ELF` in hex looks like yada yada !


Let's spend some time on `file` !
I opened the man page for file and started reading.
It seems file command runs 3 set of tests in a specific order on an actual file to classify it.
And the first test that succeeds causes the type to be printed. Which set of tests are these ?


- filesystem tests
- magic tests
- language tests


Well further down we find some more categorization which will be printed

- text
- executable
- data



What does filesystem tests do ?

>*The filesystem tests are based on examining the return from a stat(2) system call.*


What does the magic tests do ? We'll we have an idea !

>*The magic tests are used to check for files with data in particular fixed formats.
>These files have a “magic number” stored in a particular place near the beginning of the file that tells
>the UNIX operating system that the file is a binary executable, and which of several types thereof.*

One important thing to notice before we move to data tests

>*If a file does not match any of the entries in the magic file, it is examined to see if it seems to be a text file.*


What about data tests ?

>*Any file that cannot be identified as having been written in any of the character sets listed above is simply said to be
“data”.*



With some background behind us, let's poke the `file` command itself further.

```
$ file /bin/file
/bin/file: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=1304f666b8d56dc664e73696a426431aa46ec676, stripped
$ 
```


We'll not surprising `file` itself depends on libraries, lets see which those are.
The only one of interest for us is `libmagic.so.1`, rest are standard libs


```
$ ldd /bin/file
	linux-vdso.so.1 =>  (0x00007ffc47b94000)
	libmagic.so.1 => /usr/lib64/libmagic.so.1 (0x00007f031ead0000)
	libz.so.1 => /usr/lib64/libz.so.1 (0x00007f031e8ba000)
	libc.so.6 => /usr/lib64/libc.so.6 (0x00007f031e4ed000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f031eced000)
$ 
```


Next lets find out which functions excatly from the library are called via file command.
We will utilize `ltrace` to do this for us !



```
$ ltrace file reversinghero
```


You will see tons of output on your screen, we will focus on the one revelant one's only.
We immediately see some library functions with the name magic in them being called.
Well `magic_open`, `magic_load` ? No idea at this point what they do, I couldnt find man entries also.
Let's take an educated guess for now and move further.


```
magic_open(0, 0, 0x7ff4b60de280, 0)                                                 = 0x1922f20
magic_load(0x1922f20, 0, 0x1922fd0, 0x1922fd0)                                      = 0
strlen("reversinghero")                                                             = 13
```


After that you see a bunch of calls to `mbrtowc` and `wcwidth`, again no idea what is does.
Maybe they are reading the files in chunks !


```
mbrtowc(0x7ffc8788882c, 0x7ffc8788a788, 13, 0x7ffc87888830)                         = 1
wcwidth(114, 0x7ffc8788882c, 3, 114)                                                = 1
mbrtowc(0x7ffc8788882c, 0x7ffc8788a789, 12, 0x7ffc87888830)                         = 1
wcwidth(101, 0x7ffc8788882c, 3, 101)                                                = 1
mbrtowc(0x7ffc8788882c, 0x7ffc8788a78a, 11, 0x7ffc87888830)                         = 1
```


Finally when the command is about to exit we see the expected output
A call to the `magic_file` function followed by a `puts` functin which prints output to screen.
And finally the `file` command exists with `magic_close`


```
magic_file(0x1922f20, 0x7ffc8788a788, 1, 0)                                         = 0x1923770
puts("ELF 64-bit LSB executable, x86-6"...reversinghero: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), not stripped
)                                         = 105
magic_close(0x1922f20, 0x7ff4b672d000, 0x7ff4b60e0a00, -1)                          = 497
+++ exited (status 0) +++
$ 
```



So far, we can guess that `file` utilized the `magic tests` set from above sets and went to work.
Should we stop or move further ? Well lets do a bit more.


Understanding function calls is good, but let's dig deeper into how `file` goes about doing its work.
This time instead of tracing library calls, lets trace the system calls that `file` command makes using `strace`
I've delibrately added the `-f` command to see trace any child processes that might be created

```
$ strace -f file reversinghero 
```



Again tons of output thrown to the screen, lets look at the revelant logs only
The very first thing it does is use the `execve` system call to run the `file` binary

```
execve("/bin/file", ["file", "reversinghero"], [/* 18 vars */]) = 0
```


Moving further it seems that its looking for some files at specified location however it can't find them

```
open("/usr/lib64/tls/x86_64/libmagic.so.1", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
stat("/usr/lib64/tls/x86_64", 0x7fffb6762af0) = -1 ENOENT (No such file or directory)
open("/usr/lib64/tls/libmagic.so.1", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
stat("/usr/lib64/tls", {st_mode=S_IFDIR|0555, st_size=6, ...}) = 0
open("/usr/lib64/x86_64/libmagic.so.1", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
stat("/usr/lib64/x86_64", 0x7fffb6762af0) = -1 ENOENT (No such file or directory)
```


Finally it is able to find some file and it does an `fstat` on it
Remember we had ran the ldd command on file above and it was dependant on a `libmagic.so.1` library.
Well that library is being looked for and read so file can call functions from it.
Same procedure is repeated for the all the libraries that `file` command is dependant on.


```
open("/usr/lib64/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\240C\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=120104, ...}) = 0
mmap(NULL, 2213744, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f2f12fec000
mprotect(0x7f2f13007000, 2097152, PROT_NONE) = 0
mmap(0x7f2f13207000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1b000) = 0x7f2f13207000
close(3)     
```

Moving on from libraries, it tries to read some other files which might be relevant to its functionality.
However it isn't able to find any of them.


```
stat("/home/kamathe/.magic.mgc", 0x7fffb6764070) = -1 ENOENT (No such file or directory)
stat("/home/kamathe/.magic", 0x7fffb6764070) = -1 ENOENT (No such file or directory)
stat("/etc/sysconfig/64bit_strstr_via_64bit_strstr_sse2_unaligned", 0x7fffb6763c70) = -1 ENOENT (No such file or directory)
open("/etc/magic.mgc", O_RDONLY)        = -1 ENOENT (No such file or directory)
```


It does find one file which I didn't knew existed until now, reads it into memory and closes it.


```
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
open("/etc/magic", O_RDONLY)            = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f2f13404000
read(3, "# Magic local data for file(1) c"..., 4096) = 111
read(3, "", 4096)                       = 0
close(3)                
```



What is this file `/etc/magic` all about ? Let's find out
Err don't know ! Lets move further


```
$ ls -l /etc/magic 
-rw-r--r--. 1 root root 111 Jun  6  2018 /etc/magic
$ 
$ file /etc/magic
/etc/magic: magic text file for file(1) cmd, ASCII text
$ 
$ cat /etc/magic 
# Magic local data for file(1) command.
# Insert here your local magic data. Format is described in magic(5).

$ 
```


We see some other files being opened as well which might be relevant


```
open("/usr/share/misc/magic.mgc", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=2285896, ...}) = 0
mmap(NULL, 2285896, PROT_READ|PROT_WRITE, MAP_PRIVATE, 3, 0) = 0x7f2f0c2b0000
close(3)                                = 0
open("/usr/lib64/gconv/gconv-modules.cache", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=26254, ...}) = 0
mmap(NULL, 26254, PROT_READ, MAP_SHARED, 3, 0) = 0x7f2f1341f000
close(3)          
```


I think we finally get to reading the file in question `reversinghero`


```
lstat("reversinghero", {st_mode=S_IFREG|0664, st_size=595896, ...}) = 0
stat("reversinghero", {st_mode=S_IFREG|0664, st_size=595896, ...}) = 0
open("reversinghero", O_RDONLY)         = 3
fcntl(3, F_GETFL)                       = 0x8000 (flags O_RDONLY|O_LARGEFILE)
fcntl(3, F_SETFL, O_RDONLY|O_LARGEFILE) = 0
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\2\0>\0\1\0\0\0p\5@\0\0\0\0\0"..., 262144) = 262144
```


Upon opening we do a bunch of `preads` on it as can be seen below


```
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0@\0\0\0\0\0@\0@\0\0\0\0\0"..., 56, 64) = 56
pread64(3, "\3\0\0\0\4\0\0\0\220\1\0\0\0\0\0\0\220\1@\0\0\0\0\0\220\1@\0\0\0\0\0"..., 56, 120) = 56
pread64(3, "\1\0\0\0\5\0\0\0\0\0\0\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0"..., 56, 176) = 56
pread64(3, "\1\0\0\0\6\0\0\0\260N\0\0\0\0\0\0\260N`\0\0\0\0\0\260N`\0\0\0\0\0"..., 56, 232) = 56
pread64(3, "\2\0\0\0\6\0\0\0\260N\0\0\0\0\0\0\260N`\0\0\0\0\0\260N`\0\0\0\0\0"..., 56, 288) = 56
pread64(3, "R\345td\4\0\0\0\260N\0\0\0\0\0\0\260N`\0\0\0\0\0\260N`\0\0\0\0\0"..., 56, 344) = 56
pread64(3, "\21\0\0\0\3\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0c\22\t\0\0\0\0\0"..., 64, 595832) = 64
pread64(3, ".shstrtab\0.interp\0.gnu.hash\0.dyn"..., 49, 594548) = 49
pread64(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 64, 594680) = 64
pread64(3, "\0.symtab\0.strtab\0.shstrtab\0.inte"..., 49, 594531) = 49
pread64(3, "\33\0\0\0\1\0\0\0\2\0\0\0\0\0\0\0\220\1@\0\0\0\0\0\220\1\0\0\0\0\0\0"..., 64, 594744) = 64
pread64(3, ".interp\0.gnu.hash\0.dynsym\0.dynst"..., 49, 594558) = 49
pread64(3, "'\0\0\0\5\0\0\0\2\0\0\0\0\0\0\0\260\1@\0\0\0\0\0\260\1\0\0\0\0\0\0"..., 64, 594808) = 64
pread64(3, ".hash\0.dynsym\0.dynstr\0.gnu.versi"..., 49, 594570) = 49
pread64(3, "#\0\0\0\366\377\377o\2\0\0\0\0\0\0\0\360\1@\0\0\0\0\0\360\1\0\0\0\0\0\0"..., 64, 594872) = 64
pread64(3, ".gnu.hash\0.dynsym\0.dynstr\0.gnu.v"..., 49, 594566) = 49
```


Finally the file is closed and a `write` system call is used to write output to the screen


```
close(3)                                = 0
write(1, "reversinghero: ELF 64-bit LSB ex"..., 120reversinghero: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), not stripped
) = 120
munmap(0x7f2f0c2b0000, 2285896)         = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```


Phew ! So much happens underneath to identify the format of a file ?
Reading the man page carefully tells us a bit of the magic files that are being read


```
FILES
     /usr/share/misc/magic.mgc  Default compiled list of magic.
     /usr/share/misc/magic      Directory containing default magic files.
```



I think let's move on from the `file` command and try other binary analysis tools on our target executable
