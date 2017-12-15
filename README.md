### Intro

My personal notes on x86 assembly and its usage in security

### Contents

* [Programming Structures](#programming-structures)


### Registers

#### General Purpose Registers

32 Bit | 16 Bit (0-15) | Higher 8 bit (8-15) | Lower 8 Bit (0-7)
-------|--------|--------------|------------
EAX|AX|AH|AL
EBX|BX|BH|BL
ECX|CX|CH|CL
EDX|DX|DH|HL
ESP|SP
EBP|BP
ESI|SI
EDI|DI

* EAX - Accumulator Register - used for storing operands and result data
* EBX - Base register - Pointer to Data
* ECX - Counter register - Loop operations
* EDX - Data register - I/O Pointer
* ESI & EDI - Data Pointer Registers for memory operations
* ESP - Stack Pointer Register
* EBP - Stack Data Pointr Register

#### Segment Registers (16 Bits):

* CS -Code Segment
* DS - Data Segment
* SS - Stack Segment
* ES - Data Segment
* FS - Data Segment
* GS - Data Segment

#### Flags/Eflags

Flags indicate whas happening in the cpu

Bit | Flag | Type | Desccription
----|------|------|-----
21|ID|System|ID Flag
20|VIP|System|Virtua-z Interrupt Pending
19|VIF|System|Virtual Interrupt Flag
18|AC|System|Alignment Check
17|VM|System|Virtual-8086 Mode
16|RF|System|Resume Flag
15|0||Reserved
14|NT|System|Nested Task
13&12|IOPL|System|I/O Priviledge Level
11|OF|Status|Overflow Flag
10|DF|Control|Direction Flag
9|IF|System|Interrupt Enable Flag
8|TF|System|Trap Flag
7|SF|Status|Sign Flag
6|ZF|Status|Zero Flag
5|0||Reserved
4|AF|Status|Auxiliary Cary Flag
3|0||Reserved
2|PF|Status|Parity Flag
1|1||Reserved
0|CF|Status|Carry Flag

### Assemble & Link

nasm -f elf32 -o HelloWorld.o HelloWorld.asm
ld -o HelloWorld HelloWorld.o

### Program Structure 

```asm
; example program

; defines program start
global _start

; code section
section .text
; start of execution
_start:

; initialised data section
section .data

; uninitialised data section
section .bss
```
<a name="programming-structures"></a>
### Programming Structures 

#### Loops

Loop uses ecx and automatically decrements it and breaks when it hits zero

```asm
mov ecx, 0x5
label1:
	;loop body
	loop label1
```

#### Procedure

Procedures defined by labels in NASM and finished with ret

```asm
call TestProcedure

TestProcedure:
  ; Procedure body
	ret
```

Passing Arguments to a procedure
	Via Registers
	Passed on stack
	Passed by data structures in memory referenced by registers / or on stack

Saving/Storing Registers:
pushad
popad

Saving / Restoring Flags:

pushdf
popfd

save framepointers and restore
enter <size>, 1
leave + ret

### Linux Interrupts and systems calls

Use interupt 0x80 to execute systemcalls

System calls can be seen in [unistd_32.h](/unistd_32.h)

Registers for system calls:

EAX:	System Call Number	return value after call
EBX:	1st Arugment
ECX:	2nd Arugment
EDX:	3rd Argument
ESI:	4th Argument
EDI:	5th Argument

### Common code

exit gracefully:

```asm
mov al, 0x1	; set syscall 1 
xor ebx, ebx	; return 0
int 0x80	; call interrupt
```

writing to screen:

```asm
section .text
_start:
        mov eax, 0x4	; write syscall
        mov ebx, 0x1	; stdout
        mov ecx, msg	; address of message
        mov edx, mlen	; length of message
        int 0x80	; call interrupt

section .data
        msg: db "Hello World!", 0xA  ; with newline
        mlen equ $-message

```
execve:

  
  
### Removing nulls (00) from shellcode  
  
Assigning zero: 

xor eax, eax	; zeros xor
cdq

Assigning number to variable:

mov eax, 0x1	; will pad with zeros
mov al, 0x1	; will be null free

### Dynamic Addresses issue in shellcode
  
Predefined memory in assembly cannot be accessed in shellcode due to varying memory address at runtime.

jmp-call-pop technique:

```asm
jmp short jmp_stage:

call_stage:
	pop eax					; pops out address of StringVariable

jmp_stage:
	call call_stage:   			; push address of StringVariable onto stack
	StringVariable db "Hello World!", 0xA
```  
Storing data on stack: 

```asm
xor eax,eax       ; zero eax
push eax          ; push 00 nulls to terminate string
; push string onto stack in reverse //bin/sh (Pad with extra slash to make sure its dword wide)

push 0x68732f6e   ; hs/n
push 0x69622f2f   ; ib//

mov eax, esp      ; Move the memory location of start of stack (String start) into register     

```

### Extracing opcodes 

objdump -d exit -M intel

extract the opcodes from correct .text sections 

use rip_shellcode.py

### Running shellocde in C program

```C
#include <stdio.h>
#include <string.h>

unsigned char shellcode[] =
"SHELLCODE";

main()
{
	printf("Shellcode Length: %d\n", strlen(shellcode));
	int (*ret)() = (int(*)())code;

	ret();
}
```
int (*ret)() - define ret as a pointer to a function that has no arugments and returns int
(int(*)())shellcode - casts shellcode to a pointer to a function that has no arguments and returns 

gcc -fno-stack-protector -z execstack shellcode.c -o shellcode




  
#### Assembly Instructions

code | operation | description | example
-----|-----------|-------|---------
     | mov||
     |int|Calls interrupt|
        

