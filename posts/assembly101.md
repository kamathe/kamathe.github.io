# Assembly 101
### `12 April 2020`


Why Assembly ?

As i dig into more and more of the `reversinghero` binary and hit deadends, there is no other option
but to go to the source, by source I mean instructions that are getting executed on the CPU and the data
that it is working on in memory. This is true for other `crackme` style binaries as well as `malwares`.
So I though lets have a beginner level understanding of assembly which can help me get started.


How ?

There are various resources to refer to when it comes to assembly however we need beginner material.
Not too dense, at the same time covering important points. Also keep in mind most of the material out
there is for 32-bit executables, so we need to do some work on 64-bit assembly. I chose a easier path.
For this section I am referring to a popular malware analysis book which has a chapter on assembly, 
well disassembly to be more specific. Since most malware books are for people who are looking to set
foot in this domain, they keep the assembly information short, crisp yet very useful.
So let's get started !


## Fundamental data types

bit         -   Either a 0 or 1
8 bits      -   1 byte
4 bites     -   nibble (each 4 bit referered by hexadecimal)
word        -   2 bytes (16 bits)
dword       -   4 bytes (32 bits)
quadword    -   8 bytes (64 bits)


Remember a byte or a sequence of bytes CAN be interpreted differently.
So what a sequence of bytes MEAN depends on HOW it is used.

## Memory

RAM stores machine `code` (instructions) and `data`. Main memory is an array of bytes labelled by
memory addresses (in hex). Addresses start at 0 and go upto the supported machine hardware or software.


Data is stored in `little-endian` format. Lower byte order stored in lowe address and following bytes are
stored in higher addresses

Example

byte            [55]
                [l] 

word        [8B , EC]
            [h]  [l]

dword [00,01,36,CF]
      [high] [low]


## CPU Registers

Binary contains of various sectins but can roughtly be divided into `.code` and `.data` sections.
Its important to find the address of these sections in the binary, also find the equivalent addresses
in memory once the binary is executed.



General Purpose Registers

- 8 general purpose register [eax , ebx , ecx , edx , esp , ebp , esi , edi]
- Register are 32-bit (4 bytes) in size
- They can be access as 32-bit (4 bytes), 16-bit (2 bytes), 8-bit (1 byte)
- Lower 16 bits (2 bytes) [ax , bx , cx , dx , sp , bp , si , di]
- Lower 8 bits (1 byte)     [al , bl , cl , dl]
- Higher 8 bits ( 1bytes)   [as , ah , bh , ch]
- Instruction pointer [eip]
- EFLAGS register - 32-bit register, each bit in this register is a glag
- Segment registers [cs , ss , ds , es , fs , gs]


Example 

0xC6A93174

[C8,A9,31,74]   -   EAX

      [31,74]   -   AX

       AH   AL
      [31] [74]


## Data transfer Instructions


### Moving a `Constant` to `Register`


Move constants or immediate values into a register

mov eax, 10             # move 10 to eax, same as eax = 10
mov bx, 7               # move 7 to bx register, bx = 7
mov eax, 64h            # move hex value 0x64 (100) into eax, eax = 0x64

### Moving `Values` from `Register` to `Register`

Move value from one register to another

mov eax, ebx            # move `contents` of ebx into eax

mov ebx, 10             # move 10 into ebx first
mov eax, ebx            # then move `contents` of ebx to eax

### Moving `Values` from `Memory` to `Registers`

int val = 100
100 is stored as sequence of 4 bytes (0x64 i.e. 00 00 00 64) in memory
Whereever its stored in memory has an address like (0x403000)


mov eax, [0x403000]     # move `contents` present at address 0x403000 into eax, hence [], IMP
mov eax, [ebx]          # move `value` at address specified by ebx register into eax
mov eax, [ebx+ecx]      # move `value` at address specified by ebx+ecx
mov ebx, [ebp-4]        # move `value` at address specific by ebp-4
mov eax, dword ptr [ebp-4]  # dword ptr indicates 4-byte value moved from mem address specific by ebp-4


### Moving `Values` from `Registers` to `Memory`


Move value from register to memory

mov [0x403000], eax     # move value in eax to memory location starting at 0x403000
mov [ebx], eax          # move 4 byte value in eax to memory location specific by ebx


dword ptr just specifies that dword value (4 bytes) is moved

mov dword ptr [402000], 13498h  # move dword value 0x13496 to 0x402000
mov dword ptr [ebx], 100        # move dword value 100 into address specified by ebx
mov word ptr [ebx], 100         # move a word (2 bytes)100 into address specific by ebx



## Arithmetic Operations

Assembly instructions for Arithmetic

add eax, 42                     # same as eax = eax + 42
add eax, ebx                    # same as eax = eax + ebx
add [ebx], 42                   # add 42 to value in address specified by ebx
sub eax, 64h                    # subtract hex value -x64 from eax, eax = eax - 0x64


inc eax                         # eax = eax + 1
dec ebx                         # ebx = ebx - 1

mul ebx                         # ebx is multipled with eax, result stored in edx and eax
mul bx                          # bx is multipled with ax, result stored in dx and ax

div ebx                         # divides value in EDX:EAX by EBX

## Bitwise Operations

Assembly instructions that operates on bits.
Bits are numbered starting from far right (least signification) has a bit position of 0.
Bit position increases towards left.


Example

5D (0101 1101)

[0,1,0,1  1,1,0,1]

[7,6,5,4  3,2,1,0]  - bit position

not instruction             # takes one operand (servers as source and dest and inverts all bits)
not eax                     

If eax contained following

FF FF 00 00     i.e.    11111111 11111111 00000000 00000000

it would invert to following

00 00 FF FF     i.e.    00000000 00000000 11111111 11111111

and bl, cl                  # same as bl = bl & cl
or eax, ebx                 # same as eax = eax | ebx
xor eax, eax                # same as eax = eax ^ eax, clears eax register

shl dst, count              # shift left takes destination and count
shl al, 2                   # contens of al register shifted left by 2 bits

Above 2 left-most bits are removed and 2 bits 0 are appended to the right

shr dst, count              # shift right

rol al, 2                   # rotate left
ror al, 2                   # rotate right

## Branching and Conditions

Branching instructions transfer control of execution to a different memory address.
Flags register is alteres based on output of conditionals.
Two kinds of jumps  -   conditional and unconditional.


jmp <address>               # unconditional jump

cmp operation                # subtracts 2nd operand (source) from first (destination), alters flags
                            # doesn't store results, ZF=1 if result if not 0

cmp eax, 5                  # subtracts eax from 5, sets flag, result not stored, ZF=1 since result not 0
test eax, eax               # performs bitwise operation AND, alters flags, doesn't store results


jz                          # jump if zero, alias je, flags zf=1
jnz                         # jump is not zero, alias jne, flags zf=0
jl                          # jump if less, alias jnge, flags sf=1
jle                         # jump if less or equal, alias jng, flag zf=1 or sf=1
jg                          # jump if greater, alias jnle, flag zf=0, sf=0
jge                         # jump if greater or equal, alias  jnl, flag sf=0
jc                          # jump if carry, alias jb, jnae, flag cf=1
jnc                         # jump if not carry, alias jnb, jae

If statement

```
if (x == 0) {
    x = 5;
}
x = 2;
```

equivalent assembly for if

```
cmp dword ptr [x], 0
jne end_if
mov dword ptr [x], 5
end_if:
mov dword ptr [x], 2
```


If-else statement

```
if (x == 0) {
    x = 5;
}
else {
    x = 1;
}
```


Equivalent assembly

```
cmp dword ptr [x], 0
jne else
mov dword ptr [x], 5
jmp end
else:
mov dword ptr [x], 1
end:
```



If-elseif-else condition


```
if (x == 0) {
    x = 5;
}
else if ( x == 1) {
    x = 6;
}
else {
    x = 7;
}
```


Equivalent assembly

```
cmp dword ptr [ebp-4], 0
jnz else_if
mov dword ptr [ebp-4] , 5
jmp short end

else_if:
cmp dword ptr [ebp-4], 1
jnz else
mov dword ptr [ebp-4], 6
jmp short end

else:
mov dword ptr [ebp-4], 7
end:
```



## Loops


Either for or while loop

For loop

```
int i;
for (i = 0; i < 5; i++) {
}
```

Same as While loop

```
int i = 0;
while (i < 5) {
    i++
}
```


Equivalent assembly


```
move [i], 0

while_start:
cmp [i], 5
jge end
mov eax, [i]
add eax, 1
mov [i], eax
jmp while_start

end:
```


## Functions

Stack is an area of memory that gets allocated when a thread is created.
Stack is organized in a LIFO (Last In First Out) manner.
Data is `pushed` onto the stack by using push instruction.
Data is `removed` from the stack by using pop instruction.


push source                         # push source on top of stack
pop destination                     # copies value from top of stack to destination


push 3
push 4
pop ebx
pop edx


call <some_function>               # call some function
ret                                # return


Example C code

```
int test(int a, int b) {
    int x, y;
    x = a;
    y = b;
    return 0;
}


int main() {
    test(2, 3);
    return 0;
}
```

Stack arrangement for main

push 3
push 2
call test
add esp, 8
xor eax, eax

Assembly instructions for test

push ebp
mov ebp, esp
sub esp, 8
mov eax, [ebp+8]
mov [ebp-4], eax
mov ecx, [ebp+0ch]
mov [ebp-8], ecx
xor eax, eax
mov esp, ebp
pop ebp
ret

## Array and Strings

lets assume array of some data type

```
int nums[3] = {1, 2, 3}
```


Array name `nums` is the pointer constant that points to first element
How is address of other elements calculated ?

- base index of array
- index of element
- size of each element in array


```
nums[0] = [nums+0*4] = [0x4000+0*4] = [0x4000] = 1
nums[1] = [nums+1*4] = [0x4000+1*4] = [0x4000] = 2
nums[2] = [nums+2*4] = [0x4000+2*4] = [0x4000] = 3
```


General format of accessing elements of an array

```
[ base_address + index * size_of_element ]
```

## Structures
## 64-bit architecture

Difference between 32-bit and 64-bit architecture

- 32-bit registers (4 bytes)    :       [eax,ebx,ecx,edx,esi,edi,ebp,esp]
- 64-bit registers (8 bytes)    :       [rax,rbx,rcx,rdx,rsi,rdi,rbp,rsp]
- 64-bit has 8 NEW registers    :       [r8,r9,r10,r11,r12,r13,r14,r15]


- x64 handles 64-bit (8 bytes) data, all addresses and pointers are 64-bit in size
- x64 has RIP instruction pointer which is 64-bit in size
- x64 supports `rip-relative` addressing, i.e. RIP register can now be used to reference mem locations

- In 32-bit function parameters were pushed on stack
- In x64 arch first 4 parameters passed in [rcx,rdx,r8,r9], additional params saved on stack

