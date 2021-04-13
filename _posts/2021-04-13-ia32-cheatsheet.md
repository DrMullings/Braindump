---
layout: post
title:  "ASM Cheat Sheet"
date:   2021-04-13 15:10:03 +0100
categories: cheatsheets
---

# IA32

## Flags

## Registers

IA32 uses 32 bit registers, it is possible to address the historical parts of the data registers by their historical names:

| 31  | 16  | 8   |    0|
| --- | --- | --- | --- |
| eax | eax | eax | eax |
|     |     | ax  | ax  |
|     |     | ah  | al  |

### General registers

| Name | usual usage |
| ---- | --------------------------- |
| EAX | return values, interim saves |
| EBX | base adress for adressing |
| ECX | counter for loops |
| EDX | I/O data, double precision operators |

### Pointer and index registers

| Name | content |
| ---- | -------- |
| ESI | string source operand |
| EDI | string destination operand |
| EBP | points to current stack frame |
| ESP | points to top element of stack |
| EIP | current instruction |

### Segment registers

| Name | Content |
| ---- | -------- |
| cs | code segment |
| ds | data segment |
| es | extra segment (for data) |
| ss | stack segment |
| fs | (80286) |
| gs | (80286) |

Replaced by paging

### Flags

Saved in EFLAGS register

#### Status flags

Are set after arithmetic or logic operations, contain information about the result
=> usually set indirectly and can be used for conditional jumps

| Name | Full Name | Set if |
| ---- | --------- | ------- |
| ZF | Zero Flag | result of last arithmetic or logic operation was zero |
| SF | Sign Flag | the highest bit of the last operation is 1 => for signed operations it shows a negative result
| CF | Carry Flag | the range of values has been exceeded -> not enough bits to store the result
| OF | Overflow Flag | XOR from the carryover going in the highest bit and the carryover going from the highest bit -> shows range overflow for signed operations, no use in unsigned operations
| AF | Auxiliary Flag | a carry over from the third to the forth bit happens, mostly useful for BCD
| PF | Parity Flag | the result of an operation contains an even number of 1s in the lowest byte

#### System flags

| Name | Full Name | meaning |
| ---- | --------- | ------- |
| IF | Interrupt Flag | if not set no further interrupts will be processed until it is set again
| TF | Trap Flag | if set after each instruction `Interrupt 1` is called -> used for single step debugging
| DF | Direction Flag | contains the direction of strings, if set adresses (`esi` and `edi`) are decremented if unset, vice versa
| IOPL | I/O privilege level field (2 bits) | Added with 80286, contains in which privilege level the current program is running. Access to I/O address space is only granted if CPL (current privilege level) is equal or lower than IOPL. Can only be modified by popf and iret if CPL=0
| RF | Resume Flag | Added with 80386, if set multiple debugging exceptions are suppresed when a breakpoint is set on a certain field.
| VM | Virtual 8086 Mode Flag | Added with 80386, if set the cpu switches from protected mode to vitual 8086 mode

## Instructions

IA32 is an two operand architecture -> maximum two registers can be addressed in one instruction -> one of the registers  needs to be the destination operand -> for IA32 the left operand is always the one to be overwritten
Memory can be accessed by \[1234\], but can never be the destination operand unless the memory address is saved in a register e.g. add \[eax\] ecx

### Intel syntax

1. Destination operand is always on the left
2. Numbers are in decimal system unless appended with an h
3. Oprandsize is usually implicit
4. The effective address of memory access is in \[\], meaning the result of an calculation gives the memory adress

### Data transfer

| Instruction | Operation | Example |
| --- | --- | --- |
| mov dest, source | dest=source | |
| xchg op1, op2 | switch op1 and op2 | |
| movzx op1, op2, | op2 is smaller (e.g 16 bit to 32 bit) than op1, gets prepended with zeroes and then mov'd | |
| movsx op1, op2 | op2 is smaller (e.g. 16 bit to 32 bit) than op1, get prepended with the sign bit and then mov'd| |

### Arithmetic and logic

| Instruction | Operation | Example |
| --- | --- | --- |
| add dest, source | dest=dest+source | |
| sub dest, source | dest=dest-source | |
| adc dest, source | dest=dest+source+CF | addition with carryover |
| sbb dest, source | dest=dest-(source+CF) | substraction with carryover |
| inc op | op+=1 | |
| dec op | op-=1 | |
| mul op | edx:eax=eax\*op | upper 32 bit are in edx |
| div op | eax=edx:eax/op, edx contains modulo | |
| imul op1, \[op2,\[op3\]\] | [1] | | 
| idiv op | like div, but signed | |
| and dest, source | dest=dest&source | bitwise AND |
| or dest, source | dest=dest\|source | bitwise OR |
| xor dest, source | dest=dest\(+\)source | bitwise XOR |
| not op | op=!op | bitwise not |
| neg op | op=0-op | negative op in two's complement |

[1] with one operand: like mul but signed; with two operands: both get multiplied and saved in op1, op2 can be a register, a memory address or a constant; with three operands: op2 and op3 are multiplied and saved in op1, op1 is a register, op2 a register or a memory address and op3 a constant; Either way there is NO carryover

### Shift and rotation
* shift: shifts the register left or right, the last bit shifted out is saved in CF, except SAR where the MSB is pushed to preserve the sign bit, a shift pushes zerores
* rotation: rotates register content, either through CF (rcl, rcr) or not (rol, ror) but the last rotated bit will always be in CF

| Instruction | Operation | Example |
| --- | --- | --- |
| shl op,q | shift left by q bits LSB=0 CF=MSF | |
| shr op,q | shift right by q bits MSB=0 CF=LSB | |
| sal op, q | arithmetic shift left by q bits, identical to shl | for two's complement |
| sar op, q | arithmetic shift right by q bits MSB=MSB CF=LSB | for two's complement |
| rol op,q | rotate left by q bits CF=MSB LSB=MSB | |
| ror op,q | rotate right by q bits CF=LSB MSB=LSB | |
| rcl op,q | rotate left through CF by q bits CF=MSB LSB=CF | |
| rcr op,q | rotate right through CF by q bits CD=LSB MSB=CF | |


### Control transfer

Jumps are used to define subroutines and loops

compare operations:

| Instruction | Operation | Example/Explanation |
| --- | --- | --- |
| cmp op1, op2 | compares op1 and op2 by substraction | no value is changed, only flags are set |
| test op1, op2 | compares op1 and op2 by AND | no value is changed, only flags are set |

for unsigned numbers:

| Instruction | Operation | Example/Explanation |
| --- | --- | --- |
| jmp dest | unconditional jump to dest address, can be register content | jmp [ebx+4] |
| ja | jump if above CF=0 ZF=0 | op1 > op2 |
| jae | jump if above or equal CF=0 | op1 >= op2 |
| jb | jump if below CF=1 | op1 < op2 |
| jbe | jump if below or equal CF=1 ZF=1 | op1 <= op2 |
| je | jump if equal ZF=1 | op1=op2 |
| jne | jump if not equal ZF=0 | op1!=op2 |
| jna | jump if not above | equals jbe |
| jnae | jump if not above or equal | equals jb |
| jnb | jump if not below | equals jae |
| jnbe | jump if not below or equal | equals ja |

for signed numbers:

| Instruction  | Operation | Example/Explanation |
| --- | --- | --- |
| jg | jump if greater ZF=0 SF=OF | op1 > op2 |
| jge | jump if greater or equal SF=OF | op1 >= op2 |
| jl | jump if less SF<>OF | op1 < op2 |
| jle | jump if less or equal | op 1 <= op2 |
| jng | jump if not greater | equals jle |
| jnge | jump if not greater or equal | equals jl |
| jnl | jump if not less | equals jge |
| jnle | jump if not less or equal | equals jg

for flag conditions:

| Instruction | Operation | Example/Explanation |
| --- | --- | --- |
| jc | jump if carry | CF=1 |
| jnc | jump if not carry | CF=0 |
| jo | jump if overflow | OF=1 |
| jno | jump if not overflow | OF=0 |
| jp | jump if parity | PF=1 |
| jnp | jump if not parity | PF=0 |
| jpe | jump if parity even | PF=1 |
| jpo | jump if parity odd | PF=0 |
| jz | jump if zero | ZF=1 |
| jnz | jump if not zero | ZF=0 |
| js | jump if sign (<0) | SF=1 |
| jns | jump if no sign (>=0) | SF=0 |

### Subroutines

| Instruction | Operation | Example/Explanation |
| --- | --- | --- |
| call sub | dec esp [esp]=eip eip=sub | call 1234 return address on stack jump in subroutine |
| ret (op) | eip=[esp] inc esp (add esp,op) | ret 8 decrease returnadress from stack by 8 bytes |
| push op | dec esp [esp]=op | push eax, put eax on stack |
| pop op | op=[esp] inc esp | pop eax, loads eax from stack |
| pushf | push flags | |
| popf | pop flags | |
| pushad | push all general regs | |
| popad | pop all general regs | |

* \[ebp-x\] references local variables, x >= 4
* \[ebp+x\] references parameter, x >= 8


### Memory Management

### String instructions

| Instruction | Operation  |
| --- | --- |
| movsd | copies 32 bit dword from [esi] to [edi] |
| stosd | copy eax to[edi] |
| lodsd | copy [esi] to eax |
| cmpsd | compare [esi] to [edi] and set the flags |
| scasd | compare eax to [edi] and set the flags |
| rep | repeat string instruction until ecx=0 |
| repe | repeat while equal, repeat string instruction until ecx=0 or ZF=0 |
| repne | repeat while not equal, repeat string instruction until ecx=0 or ZF=1 |


### other instructions

| Instruction | Operation |
| ... | ... |
| stc | set carry flag |
| clc | clear carry flag |
| cmc | invert carry flag |
| std | set direction flag |
| cld | clear direction flag |
| sti | set interrupt flag |
| cli | clear interrupt flag |
| nop | no operation |
| loop adr | decrement ecx; jump to adr if ecx<>0 |
| loope adr | decrement ecx; jump to adr if ecx<>0 AND ZF=0 |
| loopne adr | decrement ecx; jump to adr if ecx<>0 AND ZF=1 |
| loopz adr | see *loope* |
| loopnz adr | see *loopne* |
| in dest, port | read a value from an external port | 
| out port, source | write a value to an external port |
| cbw | ax=al (signed) convert byte to word |
| cwd | dx:ax=ax (signed) convert word to dword |
| cwde | eax=ax (signed) convert word to dword |
| cdq | edx:eax=eax (signed) convert dword to qword |
| setz op | op=1 if ZF=1, else 0 |
| int op | call interrupt |
| iret | return interrupt |



