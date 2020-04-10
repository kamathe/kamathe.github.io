# Reversing Hero - Part 4, ldd, ltrace
### `10 April 2020`


Ok, enough of static analysis, using which we were getting nowhere.
Time to switch gears and move to dynamic analysis by executing the binary

Lets add the exectute bit and run the binary


```
$ ls -l reversinghero 
-rw-rw-r--. 1 kamathe kamathe 595896 Mar 28 15:49 reversinghero
$ 
$ chmod  +x ./reversinghero 
$ 
```


Some gibberish being thrown onto the screen, no proper message like enter password or key. 
However look closely, when you execute the first time the output is different, as compared to
when you execute it multiple times later where the output is same.
Hmm so is something happening when you execute it for the first time ?
Only one way to find out !


```
$ ./reversinghero 
@ 1/p1
@ 1/x1
] +
$ 
$ ./reversinghero 
] ---
$ 
$ ./reversinghero 
] ---
$ ./reversinghero 
] ---
$ ./reversinghero 
] ---
$ 
```

Lets first start by finding out which libraries the executable is dependant on using `ldd` command.
Nothing out of the ordinary, just the usual `libc` library.



```
$ ldd reversinghero 
	linux-vdso.so.1 =>  (0x00007ffca069d000)
	libc.so.6 => /lib64/libc.so.6 (0x00007fc980d23000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fc9810f0000)
$ 
```

Now lets execute the binary again, however this time lets use `ltrace` to trace the library
functions being called, Ah ! something interesting finally, lets move on.



```
$ ltrace ./reversinghero 
memset(0x690eaa, '\0', 32)                                                          = 0x690eaa
memcpy(0x690f98, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 32) = 0x690f98
memcpy(0x690fb8, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 32) = 0x690fb8
memcpy(0x7ffcb17380f0, "\2729\375\334\311\305RP\365\021\316\216\213\342\306\263\254\274\303+\257\3572\344\363C\370|\342!\354\004"..., 32) = 0x7ffcb17380f0
mkdir("1", 0700)                                                                    = 0
strcpy(0x7ffcb1737d10, "1")                                                         = 0x7ffcb1737d10
strcat("1", "/")                                                                    = "1/"
strcat("1/", "p1")                                                                  = "1/p1"
fopen("1/p1", "wb")                                                                 = 0x208c010
fwrite("\177ELF\002\001\001", 1, 566160, 0x208c010)                                 = 566160
fclose(0x208c010)                                                                   = 0
printf("@ %s\n", "1/p1"@ 1/p1
)                                                            = 7
strcpy(0x7ffcb1737d10, "1")                                                         = 0x7ffcb1737d10
strcat("1", "/")                                                                    = "1/"
strcat("1/", "x1")                                                                  = "1/x1"
fopen("1/x1", "wb")                                                                 = 0x208c010
fwrite("\177ELF\002\001\001", 1, 6640, 0x208c010)                                   = 6640
fclose(0x208c010)                                                                   = 0
printf("@ %s\n", "1/x1"@ 1/x1
)                                                            = 7
printf("] +\n"] +
)                                                                     = 4
exit(0 <no return ...>
+++ exited (status 0) +++
$ 
```

Looks at these libc functions closely, they seem to do the following

- Create directory named `1` using `mkdir`
- Open a file named `p1`
- Write some data to it
- Print output `1/p1`, so this wasn't gibberish afterall
- Open another file named `x1`
- Write some data to it
- Print output `1/x1`


```
mkdir("1", 0700)                                                                    = 0
strcpy(0x7ffcb1737d10, "1")                                                         = 0x7ffcb1737d10
strcat("1", "/")                                                                    = "1/"
strcat("1/", "p1")                                                                  = "1/p1"
fopen("1/p1", "wb")                                                                 = 0x208c010
fwrite("\177ELF\002\001\001", 1, 566160, 0x208c010)                                 = 566160
fclose(0x208c010)                                                                   = 0
printf("@ %s\n", "1/p1"@ 1/p1
)                                                            = 7
strcpy(0x7ffcb1737d10, "1")                                                         = 0x7ffcb1737d10
strcat("1", "/")                                                                    = "1/"
strcat("1/", "x1")                                                                  = "1/x1"
fopen("1/x1", "wb")                                                                 = 0x208c010
fwrite("\177ELF\002\001\001", 1, 6640, 0x208c010)                                   = 6640
fclose(0x208c010)                                                                   = 0
printf("@ %s\n", "1/x1"@ 1/x1
)                                                            = 7
```


Ok, now lets trace what happens when we execute it a second time.
The only difference I see if `mkdir` returns `-1` since directory already exists.
When we ran the binary first time it returned `0` since creating directory was successful
Based on this return status, it simply prints `]---\n]` to screen and exits.
Well is this print gibbersh ? really ?


```
$ ltrace ./reversinghero 
memset(0x690eaa, '\0', 32)                                                          = 0x690eaa
memcpy(0x690f98, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 32) = 0x690f98
memcpy(0x690fb8, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 32) = 0x690fb8
memcpy(0x7ffd5e320e50, "\2729\375\334\311\305RP\365\021\316\216\213\342\306\263\254\274\303+\257\3572\344\363C\370|\342!\354\004"..., 32) = 0x7ffd5e320e50
mkdir("1", 0700)                                                                    = -1
printf("] ---\n"] ---
)                                                                   = 6
exit(3 <no return ...>
+++ exited (status 3) +++
$ 
```


So lets investigate the directory `1` and the two files `p1` and `x1`


```
$ ls
1  reversinghero
$ 
$ ls -l 1/
total 564
-rw-rw-r--. 1 kamathe kamathe 566160 Apr 10 12:58 p1
-rw-rw-r--. 1 kamathe kamathe 6640 Apr 10 12:58 x1
$ 
```

Ok, so both files being dropped are again ELF files, i.e. executables.
Well, we just entered the rabbit hole.


```
$ cd 1
$ 
$ file p1
p1: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), not stripped
$ 
$ file x1
x1: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), not stripped
$ 
```

Let us run `ltrace` on the first binary `p1`
It asks you for an input using `fgets`, so we provide it some random input.
It then compares the length of the input you entered using `strlen` and then
prints something and quits.


```
$ ltrace ./p1 
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7f3494af2a00, 0)                                              = 0x14ec010
fgets(> qqqqq
"qqqqq\n", 66, 0x14ec010)                                                     = 0x6899c8
strcspn("qqqqq\n", "\n")                                                            = 5
strlen("qqqqq")                                                                     = 5
printf("] -\n"] -
)                                                                     = 4
fclose(0x14ec010)                                                                   = 0
exit(1 <no return ...>
+++ exited (status 1) +++
$ 
```

Lets try the other binary named `x1` and see if we get lucky !
It asks us for input, so we just enter some random input.
This one doesn't count the length of the input however it compares the input using `strcmp`.
What is it comparing against ? Err well, something like `d(-_-)b//d(+_+)b\\\\d(-_-)b`
If the input doesn't match it prints something and exists


```
$ ltrace ./x1
printf("? ")                                                                        = 2
fdopen(0, 0x601060, 0x7f8a2cf52a00, 0)                                              = 0x17f2010
fgets(? qqqqq
"qqqqq\n", 27, 0x17f2010)                                                     = 0x6010d0
strcspn("qqqqq\n", "\n")                                                            = 5
strcmp("qqqqq", "d(-_-)b//d(+_+)b\\\\d(-_-)b")                                      = 13
printf("! -\n"! -
)                                                                     = 4
fclose(0x17f2010)                                                                   = 0
exit(1 <no return ...>
+++ exited (status 1) +++
$ 
```

So well, is `d(-_-)b//d(+_+)b\\\\d(-_-)b` our first key to the kingdom ??
Lets find out what happens when we enter it.
Err this time `strcmp` is not called when we enter the expected string, again print something and 
exits. Strange !! Also a deadend !



```
$ ltrace ./x1
printf("? ")                                                                        = 2
fdopen(0, 0x601060, 0x7f521e5b3a00, 0)                                              = 0x189c010
fgets(? d(-_-)b//d(+_+)b\\\\d(-_-)b
"d(-_-)b//d(+_+)b\\\\\\\\d(-_-)", 27, 0x189c010)                              = 0x6010d0
strcspn("d(-_-)b//d(+_+)b\\\\\\\\d(-_-)", "\n")                                     = 26
printf("! -\n"! -
)                                                                     = 4
fclose(0x189c010)                                                                   = 0
exit(1 <no return ...>
+++ exited (status 1) +++
$ 
```

Just curious to see if something happened in the background, however nothing !

```
$ ./x1
? d(-_-)b//d(+_+)b\\\\d(-_-)b
! -
$ ls -a
.  ..  p1  x1
$ 
```

Lets move our focus back to `p1`, maybe `strlen` might be somewhat easier to tackle ?
Again enter `10` characters and see if we get lucky !



```
$ ltrace ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7f9a3f221a00, 0)                                              = 0x1b2b010
fgets(> 0123456789             
"0123456789\n", 66, 0x1b2b010)                                                = 0x6899c8
strcspn("0123456789\n", "\n")                                                       = 10
strlen("0123456789")                                                                = 10
printf("] -\n"] -
)                                                                     = 4
fclose(0x1b2b010)                                                                   = 0
exit(1 <no return ...>
+++ exited (status 1) +++
$ 
```



Let's provide a Big enough number, Hmm after that strlen is not called ?
So maybe we need to reduce the number somewhat


```
$ ltrace ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7f339ff5ba00, 0)                                              = 0x1da3010
fgets(> 0123456789012345678901234567890123456789012345678901234567890123456789
"01234567890123456789012345678901"..., 66, 0x1da3010)                         = 0x6899c8
strcspn("01234567890123456789012345678901"..., "\n")                                = 65
printf("] -\n"] -
)                                                                     = 4
fclose(0x1da3010)                                                                   = 0
exit(1 <no return ...>
+++ exited (status 1) +++
$ 
```

After some trial and error, i reached to a number. Guess what it is ? `64`
At 64 we got something different, instead of exiting immediately as before.
We see some `memcpy` and then print and then exit


```
$ ltrace  ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7f66e16f5a00, 0)                                              = 0x1e3d010
fgets(> 0123456789012345678901234567890123456789012345678901234567890123
"01234567890123456789012345678901"..., 66, 0x1e3d010)                         = 0x6899c8
strcspn("01234567890123456789012345678901"..., "\n")                                = 64
strlen("01234567890123456789012345678901"...)                                       = 64
memcpy(0x689af8, "\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#"..., 32) = 0x689af8
memcpy(0x689b18, "\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#"..., 32) = 0x689b18
memcpy(0x7fffba5c5430, "Ix\313n\026\333l\030\030\343\314B\017\377\357\200\003\313yD\f#z@\340\027^\213dQ\241t"..., 32) = 0x7fffba5c5430
printf("] --\n"] --
)                                                                    = 5
fclose(0x1e3d010)                                                                   = 0
exit(2 <no return ...>
+++ exited (status 2) +++
$ 
```


Lets play around with `64` a bit to see if that the right way forward
Again no `strlen` called


```
$ ltrace  ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7f1ea35e1a00, 0)                                              = 0x19de010
fgets(> 01234567890123456789012345678901234567890123456789012345678901234
"01234567890123456789012345678901"..., 66, 0x19de010)                         = 0x6899c8
strcspn("01234567890123456789012345678901"..., "\n")                                = 65
printf("] -\n"] -
)                                                                     = 4
fclose(0x19de010)                                                                   = 0
exit(1 <no return ...>
+++ exited (status 1) +++
$ 
```



Lets verify with 63, we should see strlen again, and yup we do

```
$ ltrace ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7f9051ff0a00, 0)                                              = 0x1d38010
fgets(> 012345678901234567890123456789012345678901234567890123456789012 
"01234567890123456789012345678901"..., 66, 0x1d38010)                         = 0x6899c8
strcspn("01234567890123456789012345678901"..., "\n")                                = 63
strlen("01234567890123456789012345678901"...)                                       = 63
printf("] -\n"] -
)                                                                     = 4
fclose(0x1d38010)                                                                   = 0
exit(1 <no return ...>
+++ exited (status 1) +++
$ 
```

Ok some magic is happening when we enter 64 chars, so lets hold onto it


```
$ ltrace ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7facb9495a00, 0)                                              = 0x24f5010
fgets(> 0123456789012345678901234567890123456789012345678901234567890123
"01234567890123456789012345678901"..., 66, 0x24f5010)                         = 0x6899c8
strcspn("01234567890123456789012345678901"..., "\n")                                = 64
strlen("01234567890123456789012345678901"...)                                       = 64
memcpy(0x689af8, "\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#"..., 32) = 0x689af8
memcpy(0x689b18, "\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#Eg\211\001#"..., 32) = 0x689b18
memcpy(0x7fffe5dc3b00, "Ix\313n\026\333l\030\030\343\314B\017\377\357\200\003\313yD\f#z@\340\027^\213dQ\241t"..., 32) = 0x7fffe5dc3b00
printf("] --\n"] --
)                                                                    = 5
fclose(0x24f5010)                                                                   = 0
exit(2 <no return ...>
+++ exited (status 2) +++
$ 
```


We entered numbers from the start as input, however is that what is really expected ?
Only one way to find out, lets enter some characters like `x`
Err no `memcpy` this time, simple print and exit ! Hmm !


```
$ ltrace ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7fcfb9af9a00, 0)                                              = 0x1b10010
fgets(> xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"..., 66, 0x1b10010)                         = 0x6899c8
strcspn("xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"..., "\n")                                = 64
strlen("xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"...)                                       = 64
printf("] -\n"] -
)                                                                     = 4
fclose(0x1b10010)                                                                   = 0
exit(1 <no return ...>
+++ exited (status 1) +++
$ 
```

Lets try some other number instead of `0123..` series from before.
How about enter `9` number `64` times ? Again we see memcpy


```
$ ltrace ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7fb117b20a00, 0)                                              = 0x163a010
fgets(> 9999999999999999999999999999999999999999999999999999999999999999
"99999999999999999999999999999999"..., 66, 0x163a010)                         = 0x6899c8
strcspn("99999999999999999999999999999999"..., "\n")                                = 64
strlen("99999999999999999999999999999999"...)                                       = 64
memcpy(0x689af8, "\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231"..., 32) = 0x689af8
memcpy(0x689b18, "\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231\231"..., 32) = 0x689b18
memcpy(0x7fff5275d1c0, "\245(\305\257s\a\253\271\204=u\347\327\356\301\300l\tTz\226\fa9\260\2628\350\240\247.c"..., 32) = 0x7fff5275d1c0
printf("] --\n"] --
)                                                                    = 5
fclose(0x163a010)                                                                   = 0
exit(2 <no return ...>
+++ exited (status 2) +++
$ 
```

What about `0` enterd `64` times ? Again `memcpy` is being called.
So we are sure that numbers are expected.


```
$ ltrace ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7fec503eca00, 0)                                              = 0x2285010
fgets(> 0000000000000000000000000000000000000000000000000000000000000000
"00000000000000000000000000000000"..., 66, 0x2285010)                         = 0x6899c8
strcspn("00000000000000000000000000000000"..., "\n")                                = 64
strlen("00000000000000000000000000000000"...)                                       = 64
memcpy(0x689af8, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 32) = 0x689af8
memcpy(0x689b18, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 32) = 0x689b18
memcpy(0x7ffce981cd10, "\2729\375\334\311\305RP\365\021\316\216\213\342\306\263\254\274\303+\257\3572\344\363C\370|\342!\354\004"..., 32) = 0x7ffce981cd10
printf("] --\n"] --
)                                                                    = 5
fclose(0x2285010)                                                                   = 0
exit(2 <no return ...>
+++ exited (status 2) +++
$ 
```


Just one more before we give up, lets enter `1`, results as expected

```
$ ltrace ./p1
printf("> ")                                                                        = 2
fdopen(0, 0x605080, 0x7fd729c45a00, 0)                                              = 0xbc0010
fgets(> 1111111111111111111111111111111111111111111111111111111111111111
"11111111111111111111111111111111"..., 66, 0xbc0010)                          = 0x6899c8
strcspn("11111111111111111111111111111111"..., "\n")                                = 64
strlen("11111111111111111111111111111111"...)                                       = 64
memcpy(0x689af8, "\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021"..., 32) = 0x689af8
memcpy(0x689b18, "\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021\021"..., 32) = 0x689b18
memcpy(0x7ffe0d3f4160, "\223\334HzO\347\367*\265Ab\001G\375u\323\304\227\351\277\256'\313%\304}W\354U\f\f\233"..., 32) = 0x7ffe0d3f4160
printf("] --\n"] --
)                                                                    = 5
fclose(0xbc0010)                                                                    = 0
exit(2 <no return ...>
+++ exited (status 2) +++
$ 
```

Ok, lets move onto what `memcpy` does, man pages to the rescue

```
NAME
       memcpy - copy memory area

SYNOPSIS
       #include <string.h>

       void *memcpy(void *dest, const void *src, size_t n);

DESCRIPTION
       The  memcpy() function copies n bytes from memory area src to memory area dest.  The memory areas must not overlap.  Use mem‐
       move(3) if the memory areas do overlap.

RETURN VALUE
       The memcpy() function returns a pointer to dest.
```


Lets compare the definition with what we see is being called !
So `0x689af8` and `0x689b18` are the desinations, source is as expected `0` and length is `32`.
So out `64` length input was divided into 2 chunks of `32` each and written to those memory locations ?
And also the return value is a pointer to the destination which is same as source `0x689af8` and `0x689b18`


```
memcpy(0x689af8, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 32) = 0x689af8
memcpy(0x689b18, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 32) = 0x689b18
memcpy(0x7ffce981cd10, "\2729\375\334\311\305RP\365\021\316\216\213\342\306\263\254\274\303+\257\3572\344\363C\370|\342!\354\004"..., 32) = 0x7ffce981cd10
```



Phew ! So what next ? Not sure, lets think about it a bit and get back
