# C Puzzle 1
### `29 April 2020`

## Why C ?

I've been wanting to refresh my C skills considering they are must have if you are thinking of doing any systems work.
However reading dry text or books on the syntax gets quiet boring quickly. Yes you need to write programs on
the side to really get a hang of things. Also you need to you read a lot of code from some good open source repo,
to understand the correct way of writing programs. The idea if get comfortable with a given C code base.
However given the free nature of C there are a thousand ways in which people can write C which makes it tricky.
Also there are a lot of  gotcha's which one needs to be aware of when working with C.


## Why Puzzles ?

One way to learn is via puzzle's. It gives you the same kind of learning that writing programs does, a bit more !
I had bought a book titled `The C Puzzle Book` by `Alan R. Feuer` some time back. Going through the puzzles in 
this book will be a perfect companion to reading the dry syntax and features text. So the way this works is,
the book has multiple short C programs which makes use of various C features (and gotcha's). You need to workout
the correct output given by the program based on your understanding of C. Pretty good !


## Method

So we'll follow this process for each of the puzzles

- Work out the program on paper, use any hints given
- Come up with a probable solution (again on paper)
- Double check for other (probable) answers you might have missed (remember gotcha's)
- Check your answer with the one provided later in the book's solution section
- If correct, then good, if not try working it out again (on paper)
- Well finally use a computer to compile the program and see the output for your self
- Bonus ! View the assembly as to how each C instruction is translated to assembly


## Some things to keep in mind

- Programs in book won't run as it is in some cases, reason ? Compiler have improved, book shows its age
- Assembly generated might be different on different machines based on architecture, compiler version etc
- Common sense !

So let's get started !!


# Puzzle 1

Here's the C program given in the book, Also the hint given says refer to `precedence` and `associativity`
section.

```
#include <stdio.h>

int main() {
	int x;
	
	x = - 3 + 4 * 5 - 6; printf("%d\n", x);
	x = 3 + 4 % 5 - 6; printf("%d\n", x);
	x = - 3 * 4 % - 6 / 5; printf("%d\n", x);
	x = ( 7 + 6 ) % 5 / 2; printf("%d\n", x);
}
```


The program does some aritmetic operations and saves the results in varible `x` and prints the variable.
We need to figure out the correct output. Some rules to keep in mind regarding the hint in given order

- Parenthesis
- unary operations
- Multiplication/Division/Modulo
- Addition/subtraction
- For same precedence start from left to right

So given the above rules, I thought the answer to first one was `11` however I get didn't quiet get there
the same way in which the book worked out the solution. Why ? I missed two VERY important details

- Unary operations (Look again, program looks simple, but its tricky)
- Left to right order of bindings (again easy to overlook)
- In cases like there evalution proceeds from inside out (Again very important)


Second

`3 + 4 % 5 - 6` to `3 + 4 - 6` to `7 - 6` to `1`

Third

`- 3 * 4 % - 6 / 5` to `- 12 % - 6 / 5 ` to `0 / 5` to `0`

Fourth

`( 7 + 6 ) % 5 / 2` to `13 % 5 / 2` to `3 / 2` to `1`

So our final answer Look like this

```
11
1
0
1
```


Finally to our bonus point of looking at the Assembly to figure out what happens beneath !


```
	x = - 3 + 4 * 5 - 6; printf("%d\n", x);
  40112e:	c7 45 fc 0b 00 00 00 	movl   $0xb,-0x4(%rbp)
  401135:	8b 45 fc             	mov    -0x4(%rbp),%eax
  401138:	89 c6                	mov    %eax,%esi
  40113a:	bf 10 20 40 00       	mov    $0x402010,%edi
  40113f:	b8 00 00 00 00       	mov    $0x0,%eax
  401144:	e8 e7 fe ff ff       	callq  401030 <printf@plt>
	x = 3 + 4 % 5 - 6; printf("%d\n", x);
  401149:	c7 45 fc 01 00 00 00 	movl   $0x1,-0x4(%rbp)
  401150:	8b 45 fc             	mov    -0x4(%rbp),%eax
  401153:	89 c6                	mov    %eax,%esi
  401155:	bf 10 20 40 00       	mov    $0x402010,%edi
  40115a:	b8 00 00 00 00       	mov    $0x0,%eax
  40115f:	e8 cc fe ff ff       	callq  401030 <printf@plt>
	x = - 3 * 4 % - 6 / 5; printf("%d\n", x);
  401164:	c7 45 fc 00 00 00 00 	movl   $0x0,-0x4(%rbp)
  40116b:	8b 45 fc             	mov    -0x4(%rbp),%eax
  40116e:	89 c6                	mov    %eax,%esi
  401170:	bf 10 20 40 00       	mov    $0x402010,%edi
  401175:	b8 00 00 00 00       	mov    $0x0,%eax
  40117a:	e8 b1 fe ff ff       	callq  401030 <printf@plt>
	x = ( 7 + 6 ) % 5 / 2; printf("%d\n", x);
  40117f:	c7 45 fc 01 00 00 00 	movl   $0x1,-0x4(%rbp)
  401186:	8b 45 fc             	mov    -0x4(%rbp),%eax
  401189:	89 c6                	mov    %eax,%esi
  40118b:	bf 10 20 40 00       	mov    $0x402010,%edi
  401190:	b8 00 00 00 00       	mov    $0x0,%eax
  401195:	e8 96 fe ff ff       	callq  401030 <printf@plt>
  40119a:	b8 00 00 00 00       	mov    $0x0,%eax
```


If you notice carefully Assembly is not much of help here, it does not help you understand
how the precedence or associativity at work. Lets see why that is.

We can immediately rule out the following lines in each of the above sections.
It does nothing but help `printf` do its thing, like below lines can be ignored.


```
  40113a:	bf 10 20 40 00       	mov    $0x402010,%edi
  40113f:	b8 00 00 00 00       	mov    $0x0,%eax
  401144:	e8 e7 fe ff ff       	callq  401030 <printf@plt>
```


Removing above we are left with following


```
	x = - 3 + 4 * 5 - 6; printf("%d\n", x);
  40112e:	c7 45 fc 0b 00 00 00 	movl   $0xb,-0x4(%rbp)
  401135:	8b 45 fc             	mov    -0x4(%rbp),%eax
  401138:	89 c6                	mov    %eax,%esi
	x = 3 + 4 % 5 - 6; printf("%d\n", x);
  401149:	c7 45 fc 01 00 00 00 	movl   $0x1,-0x4(%rbp)
  401150:	8b 45 fc             	mov    -0x4(%rbp),%eax
  401153:	89 c6                	mov    %eax,%esi
  40115f:	e8 cc fe ff ff       	callq  401030 <printf@plt>
	x = - 3 * 4 % - 6 / 5; printf("%d\n", x);
  401164:	c7 45 fc 00 00 00 00 	movl   $0x0,-0x4(%rbp)
  40116b:	8b 45 fc             	mov    -0x4(%rbp),%eax
  40116e:	89 c6                	mov    %eax,%esi
	x = ( 7 + 6 ) % 5 / 2; printf("%d\n", x);
  40117f:	c7 45 fc 01 00 00 00 	movl   $0x1,-0x4(%rbp)
  401186:	8b 45 fc             	mov    -0x4(%rbp),%eax
  401189:	89 c6                	mov    %eax,%esi
```

Again the only thing of important is the first line, lets narrow it down further


```
	x = - 3 + 4 * 5 - 6; printf("%d\n", x);
  40112e:	c7 45 fc 0b 00 00 00 	movl   $0xb,-0x4(%rbp)
	x = 3 + 4 % 5 - 6; printf("%d\n", x);
  401149:	c7 45 fc 01 00 00 00 	movl   $0x1,-0x4(%rbp)
  40115f:	e8 cc fe ff ff       	callq  401030 <printf@plt>
	x = - 3 * 4 % - 6 / 5; printf("%d\n", x);
  401164:	c7 45 fc 00 00 00 00 	movl   $0x0,-0x4(%rbp)
	x = ( 7 + 6 ) % 5 / 2; printf("%d\n", x);
  40117f:	c7 45 fc 01 00 00 00 	movl   $0x1,-0x4(%rbp)
```


Notice what is happening here ? Remember our output `11 1 0 1` ?
It is simply moving the result of the calculations to registers so it can be displayed.
Maybe its a feature of the latest CPU's ? Maybe its a feature of the compiler which 
notes that the calculations are short and quick so why waste time generating assembly
to actually calculate the result, why not do the work in the backend and simply show results.

- $0xb is 11
- $0x1 is 1
- $0x0 is 0
- $0x1 is 1


Phew ! Maybe there is a way to make the assembly actually show how it calculated the result.
But lets save that for another post !
