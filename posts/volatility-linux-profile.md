# Build Volatility profile for Linux
### `15 April 2020`


## Why build a profile for Linux when using Volatility ?

Well, when you clone the volatility repo and run the `--info` command to find out existing profiles 
that are available, it could happen that the profile for the Linux machine you are running is not 
available. You can try running the following command and grep for `Linux`, remember Capital `L`

```
python2 volatility/vol.py --info | grep Linux
```


If you're profile is listed there, well lucky you ! If not we can build a profile for the machine on which
we want to try out Volatility using the following steps.


## Pre-requisite packages

Apart from having volatility repo cloned, you need to install the following packages installed. I am trying
out these examples on a Redhat system, find the appropriate packages for Ubuntu.

```
yum install libdwarf-tools.x86_64
yum install kernel-headers
```


Next move to a directory called `tools/linux` within the Volatility repo that you have downloaded.

```
$ pwd
/home/LM/volatility/tools/linux
$ 
```


Run `make` command while you are in the above directory.


```
$ make
make -C //lib/modules/4.18.0-193.el8.x86_64/build CONFIG_DEBUG_INFO=y M="/home/LM/volatility/tools/linux" modules
make[1]: Entering directory '/usr/src/kernels/4.18.0-193.el8.x86_64'
  CC [M]  /home/LM/volatility/tools/linux/module.o
  Building modules, stage 2.
  MODPOST 1 modules
WARNING: modpost: missing MODULE_LICENSE() in /home/LM/volatility/tools/linux/module.o
see include/linux/module.h for more information
  CC      /home/LM/volatility/tools/linux/module.mod.o
  LD [M]  /home/LM/volatility/tools/linux/module.ko
make[1]: Leaving directory '/usr/src/kernels/4.18.0-193.el8.x86_64'
dwarfdump -di module.ko > module.dwarf
make -C //lib/modules/4.18.0-193.el8.x86_64/build M="/home/LM/volatility/tools/linux" clean
make[1]: Entering directory '/usr/src/kernels/4.18.0-193.el8.x86_64'
  CLEAN   /home/LM/volatility/tools/linux/.tmp_versions
  CLEAN   /home/LM/volatility/tools/linux/Module.symvers
make[1]: Leaving directory '/usr/src/kernels/4.18.0-193.el8.x86_64'
$  
```


Upon successful run of `make` command you should see a `module.dwarf` file within the folder.

```
$ ls
kcore  Makefile  Makefile.enterprise  module.c  module.dwarf
$ 
$ ls -l module.dwarf 
-rw-r--r--. 1 root root 3702708 Apr 14 14:20 module.dwarf
$
```

We will also need the System.map file which is present in `/boot` directory

```
$ ls -l /boot/System.map-4.18.0-193.el8.x86_64 
-rw-------. 1 root root 3909996 Mar 27 10:47 /boot/System.map-4.18.0-193.el8.x86_64
$
```

So, coming to the final step where the actual profile is built.
We need to run the zip command, follow the exact path for saving the zip file within volatility repo.
Give the profile a meaningfull name like `Redhat08-02-4-18-193.zip` in my case.
You need to profile 2 arguments one is the `module.dwarf` file and another is the `System.map` file
You should see a `adding:` message as below.


```
$ zip volatility/volatility/plugins/overlays/linux/Redhat08-02-4-18-193.zip volatility/tools/linux/module.dwarf /boot/System.map-4.18.0-193.el8.x86_64 
  adding: volatility/tools/linux/module.dwarf (deflated 91%)
  adding: boot/System.map-4.18.0-193.el8.x86_64 (deflated 79%)
$
```

But, how do we know if the profile worked at all ? We'll run the `--info` command again and grep for 
`Linux`, you should see your new profile there. Remember to run this again the `vol.py` from the repo 
location where you have put the new profile .zip file. Else it won't work.
If you see your profile name there, you are good to go.


```
$ pwd
/home/LM
$ ls
LiME  volatility
$
$ python2 volatility/vol.py --info | grep Linux
Volatility Foundation Volatility Framework 2.6.1
LinuxRedhat08-02-4-18-193x64 - A Profile for Linux Redhat08-02-4-18-193 x64
linux_aslr_shift           - Automatically detect the Linux ASLR shift
linux_banner               - Prints the Linux banner information
linux_yarascan             - A shell in the Linux memory image
LinuxAMD64PagedMemory          - Linux-specific AMD 64-bit address space.
$
```


What good is a profile, if you can't run the various command or plugins available for Linux via volatility ?
Let's try a couple of commands. You can grep for `linux_` in the `--info` command output to see all 
available commands.


```
$ python2 volatility/vol.py --info | grep linux_psscan
Volatility Foundation Volatility Framework 2.6.1
linux_psscan               - Scan physical memory for processes
$
```

I tried running a couple of command, however they did not work as expected. It seems the Linux distro and
kernel I was using was the latest and Volatility is a bit behind on supporting the latest one's.
However atleast now we know how to build a profile for a Linux distro for which one doesn't exist !!
We learned something today !! Yay !!


```
$ python2 volatility/vol.py -f LiME/Linux64.mem --profile=LinuxRedhat08-02-4-18-193x64 linux_psscan
Volatility Foundation Volatility Framework 2.6.1

Offset             Name                 Pid             PPid            Uid             Gid    DTB                Start Time
------------------ -------------------- --------------- --------------- --------------- ------ ------------------ ----------



$ python2 volatility/vol.py -f LiME/Linux64.mem --profile=LinuxRedhat08-02-4-18-193x64 linux_cpuinfo

Processor    Vendor           Model
------------ ---------------- -----
^CInterrupted
$
```

