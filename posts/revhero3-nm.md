# Reversing Hero - Part 3, nm command
### `10 April 2020`

We will use the nm command against are target binary and see if there is any interesting output.
nm might help you identify functions or variables within the binary.
However since this is a specially crafted CTF style binary we don't know what's missing.
A better way to tackle this is to compare it with a regular run-of-the-mill binary.
However most binaries on the system are stripped, which makes nm essentialy useless.
So I wrote a small hello world program and used nm on that for comparison.



Running nm on target binary, shows external lib functions being used, however we know that already.
Notice how `main` function is missing from the target binary ?
Also no mention of any of function within the binary which might help us make progress




```
$ nm reversinghero 
0000000000690e63 B __bss_start
0000000000604eb0 d _DYNAMIC
0000000000690e63 D _edata
0000000000691018 B _end
                 U exit@@GLIBC_2.2.5
                 U fclose@@GLIBC_2.2.5
                 U fopen@@GLIBC_2.2.5
                 U fwrite@@GLIBC_2.2.5
0000000000605000 d _GLOBAL_OFFSET_TABLE_
                 U memcpy@@GLIBC_2.14
                 U memset@@GLIBC_2.2.5
                 U mkdir@@GLIBC_2.2.5
                 U printf@@GLIBC_2.2.5
0000000000400570 T _start
                 U strcat@@GLIBC_2.2.5
                 U strcpy@@GLIBC_2.2.5
$ 
```


Notice the output of nm carefully, we see 3 things which tell us something about the crafted binary

- regular binary has the `main` function
- regular binary has a hell lot of other information which is missing from the crafted binary


```
$ cat hello.c 
#include <stdio.h>

int main() {
    printf("Hello, world!");
    return 0;
}
$ 
$ file hello
hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=3bde3c260b23c37aa732ff0517dc80b039545d48, not stripped
$ 
$ nm hello
000000000060102c B __bss_start
000000000060102c b completed.6355
0000000000601028 D __data_start
0000000000601028 W data_start
0000000000400460 t deregister_tm_clones
00000000004004d0 t __do_global_dtors_aux
0000000000600e18 t __do_global_dtors_aux_fini_array_entry
00000000004005c8 R __dso_handle
0000000000600e28 d _DYNAMIC
000000000060102c D _edata
0000000000601030 B _end
00000000004005b4 T _fini
00000000004004f0 t frame_dummy
0000000000600e10 t __frame_dummy_init_array_entry
0000000000400708 r __FRAME_END__
0000000000601000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
00000000004005e0 r __GNU_EH_FRAME_HDR
00000000004003c8 T _init
0000000000600e18 t __init_array_end
0000000000600e10 t __init_array_start
00000000004005c0 R _IO_stdin_used
0000000000600e20 d __JCR_END__
0000000000600e20 d __JCR_LIST__
00000000004005b0 T __libc_csu_fini
0000000000400540 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
000000000040051d T main
                 U printf@@GLIBC_2.2.5
0000000000400490 t register_tm_clones
0000000000400430 T _start
0000000000601030 D __TMC_END__
$ 
```

What about debugger symbols ?



```
$ nm -a reversinghero 
0000000000690e68 b .bss
0000000000690e63 B __bss_start
0000000000605068 d .data
0000000000604eb0 d .dynamic
0000000000604eb0 d _DYNAMIC
0000000000400318 r .dynstr
0000000000400210 r .dynsym
0000000000690e63 D _edata
0000000000404770 r .eh_frame
0000000000691018 B _end
                 U exit@@GLIBC_2.2.5
                 U fclose@@GLIBC_2.2.5
                 U fopen@@GLIBC_2.2.5
                 U fwrite@@GLIBC_2.2.5
0000000000605000 d _GLOBAL_OFFSET_TABLE_
00000000004001f0 r .gnu.hash
000000000040037c r .gnu.version
0000000000400398 r .gnu.version_r
0000000000605000 d .got.plt
00000000004001b0 r .hash
0000000000400190 r .interp
                 U memcpy@@GLIBC_2.14
                 U memset@@GLIBC_2.2.5
                 U mkdir@@GLIBC_2.2.5
00000000004004c0 t .plt
                 U printf@@GLIBC_2.2.5
00000000004003c8 r .rela.plt
0000000000400570 T _start
                 U strcat@@GLIBC_2.2.5
                 U strcpy@@GLIBC_2.2.5
0000000000400570 t .text
$ 
```


Comparison with regular binary


```
$ nm -a hello
0000000000000000 a 
000000000060102c b .bss
000000000060102c B __bss_start
0000000000000000 n .comment
000000000060102c b completed.6355
0000000000000000 a crtstuff.c
0000000000000000 a crtstuff.c
0000000000601028 d .data
0000000000601028 D __data_start
0000000000601028 W data_start
0000000000400460 t deregister_tm_clones
00000000004004d0 t __do_global_dtors_aux
0000000000600e18 t __do_global_dtors_aux_fini_array_entry
00000000004005c8 R __dso_handle
0000000000600e28 d .dynamic
0000000000600e28 d _DYNAMIC
0000000000400318 r .dynstr
00000000004002b8 r .dynsym
000000000060102c D _edata
0000000000400618 r .eh_frame
00000000004005e0 r .eh_frame_hdr
0000000000601030 B _end
00000000004005b4 T _fini
00000000004005b4 t .fini
0000000000600e18 t .fini_array
00000000004004f0 t frame_dummy
0000000000600e10 t __frame_dummy_init_array_entry
0000000000400708 r __FRAME_END__
0000000000601000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
00000000004005e0 r __GNU_EH_FRAME_HDR
0000000000400298 r .gnu.hash
0000000000400358 r .gnu.version
0000000000400360 r .gnu.version_r
0000000000600ff8 d .got
0000000000601000 d .got.plt
0000000000000000 a hello.c
00000000004003c8 T _init
00000000004003c8 t .init
0000000000600e10 t .init_array
0000000000600e18 t __init_array_end
0000000000600e10 t __init_array_start
0000000000400238 r .interp
00000000004005c0 R _IO_stdin_used
0000000000600e20 d .jcr
0000000000600e20 d __JCR_END__
0000000000600e20 d __JCR_LIST__
00000000004005b0 T __libc_csu_fini
0000000000400540 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
000000000040051d T main
0000000000400254 r .note.ABI-tag
0000000000400274 r .note.gnu.build-id
00000000004003f0 t .plt
0000000000400420 t .plt.got
                 U printf@@GLIBC_2.2.5
0000000000400490 t register_tm_clones
0000000000400380 r .rela.dyn
0000000000400398 r .rela.plt
00000000004005c0 r .rodata
0000000000400430 T _start
0000000000400430 t .text
0000000000601030 D __TMC_END__
$ 
```


Sort numerically by address


```
$ 
$ nm -n reversinghero 
                 U exit@@GLIBC_2.2.5
                 U fclose@@GLIBC_2.2.5
                 U fopen@@GLIBC_2.2.5
                 U fwrite@@GLIBC_2.2.5
                 U memcpy@@GLIBC_2.14
                 U memset@@GLIBC_2.2.5
                 U mkdir@@GLIBC_2.2.5
                 U printf@@GLIBC_2.2.5
                 U strcat@@GLIBC_2.2.5
                 U strcpy@@GLIBC_2.2.5
0000000000400570 T _start
0000000000604eb0 d _DYNAMIC
0000000000605000 d _GLOBAL_OFFSET_TABLE_
0000000000690e63 B __bss_start
0000000000690e63 D _edata
0000000000691018 B _end
$ 
```


And for the regular file



```
$ nm -n hello
                 w __gmon_start__
                 U __libc_start_main@@GLIBC_2.2.5
                 U printf@@GLIBC_2.2.5
00000000004003c8 T _init
0000000000400430 T _start
0000000000400460 t deregister_tm_clones
0000000000400490 t register_tm_clones
00000000004004d0 t __do_global_dtors_aux
00000000004004f0 t frame_dummy
000000000040051d T main
0000000000400540 T __libc_csu_init
00000000004005b0 T __libc_csu_fini
00000000004005b4 T _fini
00000000004005c0 R _IO_stdin_used
00000000004005c8 R __dso_handle
00000000004005e0 r __GNU_EH_FRAME_HDR
0000000000400708 r __FRAME_END__
0000000000600e10 t __frame_dummy_init_array_entry
0000000000600e10 t __init_array_start
0000000000600e18 t __do_global_dtors_aux_fini_array_entry
0000000000600e18 t __init_array_end
0000000000600e20 d __JCR_END__
0000000000600e20 d __JCR_LIST__
0000000000600e28 d _DYNAMIC
0000000000601000 d _GLOBAL_OFFSET_TABLE_
0000000000601028 D __data_start
0000000000601028 W data_start
000000000060102c B __bss_start
000000000060102c b completed.6355
000000000060102c D _edata
0000000000601030 B _end
0000000000601030 D __TMC_END__
$ 
```


Sort nm output by symbol-size, well again our target binary gives us nothing


```
$ nm -g --size-sort hello
0000000000000002 T __libc_csu_fini
0000000000000004 R _IO_stdin_used
000000000000001a T main
0000000000000065 T __libc_csu_init
$ 


$ nm -g --size-sort reversinghero 
$ 
```
