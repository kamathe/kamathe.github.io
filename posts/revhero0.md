# Reversing Hero - Part 0
### `8 April 2020`

## What is Reversing Hero ?

While doing the usual twitter idling during this lockdown period, I came across a website 
called [ReversingHero](https://www.reversinghero.com/) created by `xorpd`. 
If you are interested in RE like me, you should visit the website.
It claims to provides a set of RE challenges which you can utilize to hone your RE skills.




>*ReversingHero is a 15-challenges computer program, designed to teach you Reverse Engineering. 
>It begins from the real basics, and continues into more advanced topics.*




Sounds amazing




>*To move on to next challenge, you have the solve the previous one. This makes sure that you progress gradually, and reach a challenge only when you are ready to solve it.*




So what skills are needed to get started with this ? As per the site this should get you started




>*You need to have basic understanding of x86 or x64 assembly. It is also recommended to have some experience with some kind of low level development (like C), and some scripting development (like Python).*




I have been looking for similar CTF style challenges to get my feet wet.
I visited various websites like `crackme.xyz` however didn't actually download any binaries.
Well coming from `xorpd` this looks like a good starting point for me.
Maybe a bit too much for newbie like me, but what the hell ? Lets see how far we go !


Well without waiting further, lets see whats in store for us.
I immediately started with downloading reversinghero.


```
$ wget https://www.reversinghero.com/reversinghero
Saving to: ‘reversinghero’

100%[==============================================================================================>] 595,896     --.-K/s   in 0.02s   

2020-04-08 23:46:38 (28.8 MB/s) - ‘reversinghero’ saved [595896/595896]

$ 
```


Lets see, what we got !
We got a single 64-bit ELF executable file, its dynamically linked and not stripped (lucky me !)
We also took the md5sum of the file, just in case a new binary gets uploaded to the website with some changes.
We should be able to recognize the difference.


```
$ ls -l reversinghero 
-rw-rw-r--. 1 kamathe kamathe 595896 Mar 28 15:49 reversinghero
$ 
$ file reversinghero 
reversinghero: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), not stripped
$ 
$ md5sum reversinghero 
c0f1c3ee6279706356f14696bd5ea882  reversinghero
$ 
```


So we got the challenge, what next ?
Do we immediately fire up our favorite RE tool and start digging ?
That's not really what i have in mind for now.
I want to go slow on this one and take my time, I already assume this is going to be difficult.
So instead of watching reversing videos and trying out the same things on this binary i want to take the longer route.
I am going to utilize standard binary tools available natively on Linux to probe the binary first.
Find as much information about it as i can and lets take it from there.



Well, this is `Part 0` so you can expect future posts on this as I make progress.
Wish me luck !
