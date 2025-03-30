# Brief history of programming languages

## The machine code

Your computer (the processor in it) understands commands in only one language, which is called the **machine code**.

The machine code looks like a bunch of numbers, and all actions the computer can do are encoded into them:
```
f3 0f 1e fa 31 ed 49 89 d1 5e 48 89 e2 48 83 e4
f0 50 54 45 31 c0 31 c9 48 8d 3d e1 00 00 00 ff
15 2b 13 00 00 f4 cc cc cc cc cc cc cc cc cc cc
48 8d 3d 59 23 00 00 48 8d 05 52 23 00 00 48 39
f8 74 15 48 8b 05 16 13 00 00 48 85 c0 74 09 ff
e0 0f 1f 80 00 00 00 00 c3 0f 1f 80 00 00 00 00
48 8d 3d 29 23 00 00 48 8d 35 22 23 00 00 48 29
fe 48 89 f0 48 c1 ee 3f 48 c1 f8 03 48 01 c6 48
d1 fe 74 14 48 8b 05 dd 12 00 00 48 85 c0 74 08
```
As you might guess, the machine code is incredibly difficult to read and write by hand.

To make their lives easier, the programmers came up with...

## The assembly

The assembly is a primitive programming language, that primarily just assigns words to the different machine code numbers.

It can look like this: (assembly on the right, the respective machine code on the left)

```
50                            push  %rax
48 8b 3d 70 12 00 00          mov   0x1270(%rip),%rdi
48 8d 35 7d ee ff ff          lea   -0x1183(%rip),%rsi
ba 0e 00 00 00                mov   $0xe,%edx
e8 47 00 00 00                call  1880
31 c0                         xor   %eax,%eax
59                            pop   %rcx
c3                            ret
```
Programs called "assemblers" are used to convert programs written in assembly into the machine code, so at least you don't have to write the machine code by hand.

The assembly is *still* hard to read and write, so programmers came up with...

## High-level programming languages

Those are the languages where you don't have to directly think about the resulting machine code or assembly. Basically, any other language than the assembly.

Programs in those languages need **compiled**, that is converted to machine code using a program called **a compiler**.

C++ is one of them. I [previously called](./03_what_should_i_learn.md#how-does-c-compare-to-other-languages) C++ a "low-level" language, but there's no contradiction. It's higher level than assembly (which is the traditional meaning of "high-level"), but lower level than most other languages that exist today.
