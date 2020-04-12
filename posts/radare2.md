# Introduction to radare2
### `12 April 2020`


## Why radare2 ?

If you look at the `Reverse Engineering` or `Malware Analysis` world, they have their favourite tools of
choice. Either its the ubiquitous `IDAPro` or the new kid on the block `Ghidra` which is gaining traction.
There are off course other contenders like `Binary Ninja` and others. So why choose `radare2` ?
What is so special about radare2 ? 

Well for beginners both `IDAPro` and `Ghidra` are GUI programs, that is you needs access to a X window
or GUI session to be able to use any of these effectively. If you are working on Linux and are comfortable
with the command line `radare2` provides the interface that can be used at command line. No GUI needed.
Off course there are initiatives within radare2 to support GUI via `Cutter` and people are free to use them.


Maturity ?

Yes its true that tools like `IDAPro` and pretty old and battle tested, same goes for `Ghidra` which was 
used for reverse engineering within NSA and was recently opensourced. So the functionality provided by
these must be extremely mature and robust. Not to mention their advanced decompilers which is a critical
component when it comes to getting pseudo-code from assembly. Well I can't argue with that. However
I feel `radare2` gives you a good starting point to get your feet wet, so lets use it !



So what all functionality is required ?


I've installed and used radare2 on and off and loaded a couple of binaries and tried a few command here and 
there. But I am nowhere proficient in using radare2. Also I often forget the commands due to time lapse.
So I need a set of MUST-HAVE use cases and their equivalent how-to's in radare2 to get started.
How do we get that ? We don't even know what we are looking for ? Well let's leverage expertise of the users
of `IDApro` and `Ghidra`. What i mean by that is, if we could closely learn how to do the same tasks that
they do in those tools via radare2, we should be set to go. 


So Following are the tasks that we need to perform using radare2 !
I will keep updating this post as I learn more about radare2


## radare2 installation
## radare2 basic building blocks
## Loading binary in radare2
## Various view available in radare2
## Moving between various views
## radare2 configuration
## Find out various functions within the binary
## Find out the `Imports` functions
## Find out the `Exports` functions
## List various `Strings` present in the binary
## Find out various sections in the binary, eg ELF sections
## Starting project, saving project How to
## Renaming variables, location, function names
## How to add our comments
## Navigating/Seeking to various locations in the binary
## Find Cross-References or XREFS for code and data, listing
## Graph views How to ?
## Finding information on Linux API's, libc functions, system calls etc
## Scripting capability of radare2
## Understanding various plugins, how to install and their uses
## Patching binaries How to
## How to use Debugger functionality
## View CPU registers, memory, stack information at runtime
