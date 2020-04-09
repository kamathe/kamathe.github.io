# Reversing Hero, Part 2 - Strings
### `10 April 2020`

In our earlier post we used `file` command against the reversinghero binary.
In this one we will run `strings` command against the binary and see if we get any helpful output.

I used plain old strings without any command line arguments in the beginning

```
$ strings reversinghero  | more
```


I found some references to the libc functions that the binary is making use of.
This might help us later to find out the exact locations where these functions are called.
We can have a look at the arguments and find additional information.
Like `mkdir` is being called, we can see what the directory name is and where it is being created.
Same goes for other library functions like `fopen` and `fwrite`, we can look at the arguments and
try to figure out what file is being opened and what is being written to in that file.


```
printf
memset
mkdir
exit
strcpy
strcat
fopen
fwrite
fclose
memcpy
```



Next set of interestring strings that I found are below.
Why are they interesting ? Well the first one is readable and says `expand 32-byte`.
Lets see if that helps later, also we find various strings with `ATAUAV` in them.
Again not sure if it serves some purpose, so just going to put them here.



```
expa
nd 3
2-by
te kH
ATAUAVAWUSSH
@[[]A_A^A]A\
ATAUAVUH
(]A^A]A\
ATAUAVAWUH
M;<$u-H
]A_A^A]A\
ATAUAVI
A^A]A\
] --
] ---
```


And finally towards the end, I could see the challenge name and the authors website.
So kind of hit a dead-end of the strings front. I was hoping to see messages like Congratulations,
or you cleared round X, or try again. Using the memory locations of these messages I was hoping to see
where they were called and then move on from there.


Well it seems the challenge is difficult (as expected), lets keep digging !


```
ReversingHero
www.xorpd.net
```
