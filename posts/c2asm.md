# C to Assembly
### `28 March 2020`

## Why Assembly ?

If you want to get into Reverse Engineering and Malware Analysis you simply cannot ignore assembly. Period !
Most of the malware is written in compiled languages like C, C++ and others, the Malware authors won't
be courteous enough to provide you with the source code and a README to go with. All you will have is a 
compiled binary. You need to take apart that binary to make sense of what it does, what its purpose is.

To make matters worse there are multiple architectures other than x86 and ARM, and each have a different
assembly language. However you should start somewhere, start with the architecture that you have accces to.
Well at least you don't have to write assembly, you have to simply learn to read it, well just enough to get by.

### Learn from the masters

Often times you will hear experienced reverse engineers about how they would write small programs (often in C)
and then compile it and view the assembly it was generating, they did it so often that the conversion from C
to assembly was imprinted in their minds. So whenever they came across assembly code they could make some
sense of what the equivalent C code would have been.

Well that is very good advice to begin with, however there is a small hiccup, you would have to write an insane
amount of C code, not to forget make use of all the features C has to offer like structs, pointers, loops etc
then compile each of them and look for equivalent assembly. Whew ! Sounds fun already doesn't it.
Also remember C to assembly mapping is never 1:1, what i mean by this is what if you have 2 lines of C source 
code, you won't get exactly 2 lines of assembly. You could get 50, 100 or even 1. Thats tricky isn't it


## Mapping a line of C to its equivalent Assembly

Well fortunately there are tools which can help you overcome this. Not a specialized tool, but a plain old
simple tool bundled with each Linux distro named `objdump`. Lets see a quick example of how to achieve that

Lets take a 'Hello, World!' program 

```
$ cat hello.c
#include <stdio.h>

int main() {
    printf("Hello, World!");
    return 0;
}
$ 
```

Lets compile it to an `object file`, remember `object file` and not an executable. To do this use the `-c` option
Also do not forget the `-g` flag for debugging info. Without this `objdump` wont work as expected
You will get a resulting file with a `.o` extention

```
$ gcc -c hello.c -g
$ ls -l hello.o 
-rw-rw-r--. 1 kamathe kamathe 5608 Mar 28 23:15 hello.o
$ 
```


Now use the `objdump` command with the `-S` option on the `hello.o` object file
Notice the output carefully, it has the lines of C code from our source `.c` file, followed by assembly
language instruction for each of the C source code line. Cool isn't it, and very helpful
For example for running `return 0` in C, it runs an instruction `mov $0x0,%eax` and so on.
Yes you will miss a few things like memory addresses where most of it will be marked `$0x0` and named of
libc functions where it will simply say `callq 13 <main+0x13>` instead of saying `callq printf` or `puts`
But you will have to make do with what you have got and move further

```
$ objdump -S hello.o

hello.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <main>:
#include <stdio.h>

int main() {
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
    printf("Hello, World!");
   4:	bf 00 00 00 00       	mov    $0x0,%edi
   9:	b8 00 00 00 00       	mov    $0x0,%eax
   e:	e8 00 00 00 00       	callq  13 <main+0x13>
    return 0;
  13:	b8 00 00 00 00       	mov    $0x0,%eax
}
  18:	5d                   	pop    %rbp
  19:	c3                   	retq   
$ 
```



## Assembly that covers all C features ?

Above was a simple hello world proram, however there are a awful lot of C features that we need to write 
code for and compile them and run objdump on then and get the equivalent assembly. Hmmm !
This is a problem, lets get to solving it in the easiest way we can.


I am sure we all have read books on Programming, these days its all to easy to upload the sample code used
within the book to the publisher's website so people can simply download it and make use it instead of
writing it themselves. If you learning to program in a specific language i'd suggest you to write the code
yourself and not be lazy. However our purpose here is to learn assembly that is generated from C programs.
Well how about we try to solve our problem with this feature ?

### Steps

- Download C source code freely available online (mostly linked to a book)
- Compile all the source code (to object files and NOT executables)
- Run `objdump -S` on all the object files
- Save the above output to file which you can read later
- Profit !

Well lets find a worthy book, i've heard a lot of good things about 'C Primer Plus', also the source code
is easily available online, we're in luck. Lets download the source code and unzip it. We've got all the C
source code we will need. Best part is it will cover all C features in easy to understand examples.

```
$ wget http://www.informit.com/content/images/9780321928429/downloads/9780321928429_CPrimerPlus6E_code.zip

HTTP request sent, awaiting response... 200 OK
Length: 130008 (127K) [application/zip]
Saving to: ‘9780321928429_CPrimerPlus6E_code.zip’

9780321928429_CPrimerPlus6E_code. 100%[=============================================================>] 126.96K   179KB/s    in 0.7s    

2020-03-28 22:56:30 (179 KB/s) - ‘9780321928429_CPrimerPlus6E_code.zip’ saved [130008/130008]
$ 

$ ls -l 9780321928429_CPrimerPlus6E_code.zip 
-rw-rw-r--. 1 kamathe kamathe 130008 Nov  5  2013 9780321928429_CPrimerPlus6E_code.zip
$ 


$ ls -l
total 136
drwxr-xr-x. 20 kamathe kamathe    400 Nov  4  2013 9780321928429_CPrimerPlus6E_code
-rw-rw-r--.  1 kamathe kamathe 130008 Nov  5  2013 9780321928429_CPrimerPlus6E_code.zip
$ 
```

Now that we have our source code lets get to work following the other steps, however there are hundres
of C source code files and we cannot possibly compile each one by one. Let's automate.
The following shell script named dumpasm.sh is self explanatory, i've still commented it, here's what is does

- Loop over all C files, compile to object files both for 32-bit and 64-bit archs
- Loop over all object files and run `objdump -S` on them
- Save results to `asm.log` file and view the results



```
$ pwd
/tmp/asm
$ cat dumpasm.sh 
#!/usr/bin/bash
# URL to get C source code from 'C Primer Plus'
# wget http://www.informit.com/content/images/9780321928429/downloads/9780321928429_CPrimerPlus6E_code.zip
# unzip the file to a directory, then run the script where the file was unzipped

# find all files, iterate over '.c' ones and compile them
# compile the code with '-c' flag
# this is important since we dont want complete executable file
# just the object file for our C code which has the assembly instructions

for fl in $(find 9780321928429_CPrimerPlus6E_code/ -iname "*.c" | xargs)
do
	echo ------- ${fl} --------------
	ofl=${fl%.c}
    	# we compile both for 32 and 64 bit so we can compare the assembly for the two
	gcc -m32 -g -c ${fl} -o ${ofl}_32
	gcc -g -c ${fl} -o ${ofl}_64
done

# dump gcc version to file as well, in case you run this script on multiple machines
# i learned this the hard way, you will thank me later

gcc --version >> asm.log
echo ; echo ; echo >> asm.log

# next iterate over all the compiled object files from above
# dump assembly to file using objdump command

for fl in $(find 9780321928429_CPrimerPlus6E_code/ -type f | grep -iE '_32$|_64$' | xargs)
do
	echo ------- ${fl} start -------------- >> asm.log
    	# objdump -S is the superstar here that dumps assembly per line of C code
	objdump -d -M att -S ${fl} >> asm.log
	echo; echo ; echo >> asm.log
done

# you are done, open the asm.log file and you have all the C -> assembly you need
$ 
```

While running the shell script you will see the name of each `.c` file is is processing

```
$ bash dumpasm.sh 
------- 9780321928429_CPrimerPlus6E_code/XB/wider.c --------------
------- 9780321928429_CPrimerPlus6E_code/XB/strrtok.c --------------
------- 9780321928429_CPrimerPlus6E_code/XB/offset.c --------------
------- 9780321928429_CPrimerPlus6E_code/XB/def.c --------------
$ 
```

Once complete you will find object code files with `_32` for 32-bit programs and `_64` for 64 bit
programs, so this lets compare 32-bit assembly with 64-bit assembly. How cool !


```
$ find 9780321928429_CPrimerPlus6E_code/ | grep 32$ | tail -5
9780321928429_CPrimerPlus6E_code/Ch02/stillbad_32
9780321928429_CPrimerPlus6E_code/Ch02/two_func_32
9780321928429_CPrimerPlus6E_code/Ch01/inform_32
9780321928429_CPrimerPlus6E_code/Ch01/listing1_32
9780321928429_CPrimerPlus6E_code/Ch01/listing2_32

$ find 9780321928429_CPrimerPlus6E_code/ | grep 64$ | tail -5
9780321928429_CPrimerPlus6E_code/Ch02/stillbad_64
9780321928429_CPrimerPlus6E_code/Ch02/two_func_64
9780321928429_CPrimerPlus6E_code/Ch01/inform_64
9780321928429_CPrimerPlus6E_code/Ch01/listing1_64
9780321928429_CPrimerPlus6E_code/Ch01/listing2_64
$ 
```

Finally all results are appended to `asm.log` file, You can view this at leisure and learn all about
C to assembly conversion on x86 architecure



```
$ ls -l asm.log 
-rw-rw-r--. 1 kamathe kamathe 2008489 Mar 28 22:59 asm.log
$ 
```

Lets see the very first example from out `asm.log` file. Well no surprise, its verfy similar to the
'Hello, World!' program we saw earlier. Compare the outputs for 32-bit and 64-bit.
The name of C source code file is appended at the beginning of each code listing as well.


```
------- 9780321928429_CPrimerPlus6E_code/Ch01/listing2_64 start --------------

9780321928429_CPrimerPlus6E_code/Ch01/listing2_64:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <main>:
#include <stdio.h>
int main(void)
{
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
    printf("Concrete contains gravel and cement.\n");
   4:	bf 00 00 00 00       	mov    $0x0,%edi
   9:	e8 00 00 00 00       	callq  e <main+0xe>
    
    return 0;
   e:	b8 00 00 00 00       	mov    $0x0,%eax
}
  13:	5d                   	pop    %rbp
  14:	c3                   	retq   

------- 9780321928429_CPrimerPlus6E_code/Ch01/listing2_32 start --------------

9780321928429_CPrimerPlus6E_code/Ch01/listing2_32:     file format elf32-i386


Disassembly of section .text:

00000000 <main>:
#include <stdio.h>
int main(void)
{
   0:	8d 4c 24 04          	lea    0x4(%esp),%ecx
   4:	83 e4 f0             	and    $0xfffffff0,%esp
   7:	ff 71 fc             	pushl  -0x4(%ecx)
   a:	55                   	push   %ebp
   b:	89 e5                	mov    %esp,%ebp
   d:	51                   	push   %ecx
   e:	83 ec 04             	sub    $0x4,%esp
    printf("Concrete contains gravel and cement.\n");
  11:	83 ec 0c             	sub    $0xc,%esp
  14:	68 00 00 00 00       	push   $0x0
  19:	e8 fc ff ff ff       	call   1a <main+0x1a>
  1e:	83 c4 10             	add    $0x10,%esp
    
    return 0;
  21:	b8 00 00 00 00       	mov    $0x0,%eax
}
  26:	8b 4d fc             	mov    -0x4(%ebp),%ecx
  29:	c9                   	leave  
  2a:	8d 61 fc             	lea    -0x4(%ecx),%esp
  2d:	c3                   	ret    

```


## Conclusion

Hope this gets you started to learn about Assembly from compiled C source code files, however remember
this is just a start, there are multiple ways in which the assembly can differ. For example if the compiler,
`gcc` is in this case employs optimization then the assembly generated will be very different than what
we see above. Also malware might use various obfuscation techniques to throw off an analyst, however knowing
what normal looks like will help you to identify what abnormal looks like. So keep going !
