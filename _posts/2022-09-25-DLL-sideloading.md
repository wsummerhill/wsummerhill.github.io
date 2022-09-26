---
layout: post
title: "DLL Side-Loading Exploitation"
date: 2022-09-25
categories: DLL sideload exploit cpp C++
published: false
---

## DLL Side-Loading

I'm going to cover an example of how to perform a DLL side-load from start to finish using a C++ payload and a legitimate DLL commonly found on disk. 

In this post I'll walk through the following steps:
- How to choose an EXE and DLL to side-load
- How to find the exported functions needed to side-load the DLL
- How to create the malicous DLL
- Put everything together for a successful DLL side-load attack

But first...

### What is DLL side-loading?

(Source: MITRE)[https://attack.mitre.org/techniques/T1574/002/]
_"Side-loading involves hijacking which DLL a program loads. 
But rather than just planting the DLL within the search order of a program then waiting for the victim application to be invoked, adversaries may 
directly side-load their payloads by planting then invoking a legitimate application that executes their payload(s)."_

### Choosing an EXE and DLL to side-load


### Finding exported functions from the DLL


### Creating a malicious DLL


### Putting it all together
