---
layout: post
title: Learning CSAPP
date: 2025-08-12 16:30:00
description: My studying notes of CSAPP.
tags: cs
categories: StudyNotes
---

## Instructions

MOV(data movement):
* format: `mov source dest`
* Source operand can be: Immediate, Register, Memory.
* Destination operand can be: Register, Memory.
* Source and Dest can't both be **Memory**.

Stack:
* format: `pushq %rax`, `popq %rbx`

LEA(load effective address):
* format: `leaq 7(%rdi, %rsi, 4), %rax`, which means load address `7 + %rdi + %rsi * 4` to target register `%rax`.
* But actually you can use numbers instead of addresses in this instruction.

Unary Operations:
* `INC D`: increment 1
* `DEC D`: decrement 1
* `NEG D`: negate
* `NOT D`: complement

Binary Operations:
* `ADD  S, D`:  D <- D + S
* `SUB  S, D`:  D <- D - S
* `IMUL S, D`:  D <- D * S
* `XOR  S, D`:  D <- D ^ S (exclusive-or)
* `OR   S, D`:  D <- D | S (or)
* `AND  S, D`:  D <- D & S

Shift Operations:
* `SAL k, D`: D <- D << k
* `SHL k, D`: same as SAL
* `SAR k, D`: D <- D >> k, arithmetic right shift
* `SHR k, D`: logical right shift

Other Operations: ask AI.