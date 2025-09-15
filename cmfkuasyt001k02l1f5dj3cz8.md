---
title: "Assembly Language: Memory Allocation, Access, Bytes and Words."
datePublished: Mon Sep 15 2025 08:06:01 GMT+0000 (Coordinated Universal Time)
cuid: cmfkuasyt001k02l1f5dj3cz8
slug: assembly-language-memory-allocation-access-bytes-and-words
tags: assembly-language, assembly, registers, memory-allocation

---

For the CPU to operate, we must store the data we want to operate on; it is stored in RAM. The different buses of the CPU that communicate with the RAM fetch and read that data.

NOTE: Please read the first blog in the series for a better understanding if you are new to assembly language.

## How Memory Allocation Works  

Memory allocation means that we want to store some data in RAM for the CPU to access. In simple terms, we call it initializing and declaring a variable. For the sake of understanding, global variables will be used(even though it is a bad practice).

A register is a very small storage within the CPU, because the RAM takes too much time to fetch and write information. Keeping the information in registers saves time since the CPU is very fast(even older ones) in comparison to the operations of the RAM. Each Register has a capacity; in this case, our register is of 2-bits.

Here in assembly language, we must tell the CPU about the data type and the size of the data. One of the 2-bit variable is called dw(declare word), the 1-bit equivalent is db(declare byte).

A semicolon is used for comments.

We can either directly instruct the register to load up a value(focus on the second line):

```plaintext
[org 0x0100]

mov ax, 11 ;mov value into ax

mov ax, 0x4c00
int  0x21
```

Or we can store it, give it a name, and refer to the value stored at that name:

```plaintext
[org 0x0100]

mov ax,[num1] ; mov value at num1 into ax
add [result],ax ; add whatever is in ax to the var named result

mov ax,[num1+2] ; mov the value next to num1 in to the ax
add [result], ax

mov ax,[num1+4]
add [result],ax

mov ax, 0x4c00
int  0x21

num1: dw 3,2,5
result: dw 0 ; after execution of all instruction we should have 000A in the result var
```

In the above code, we have referenced the value at num1, and since we want to type as little as possible, we declare them together and increment (by 2) to go to the next word.