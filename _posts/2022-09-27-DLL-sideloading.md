---
layout: post
title: "DLL Side-Loading Exploitation"
date: 2022-09-27
categories: DLL sideload exploit cpp C++
published: false
---

## DLL Side-Loading

I'm going to cover an example of how to perform a DLL side-load from start to finish using a C++ payload and a legitimate DLL commonly found on disk. 

In this post, we'll walk through the following steps:
- How to choose an EXE and DLL to side-load
- How to find the exported functions needed to side-load the DLL
- How to create the malicous DLL
- Put everything together for a successful DLL side-load attack

But first...

### What is DLL side-loading?

[Source: MITRE](https://attack.mitre.org/techniques/T1574/002/)
_"Side-loading involves hijacking which DLL a program loads. 
But rather than just planting the DLL within the search order of a program then waiting for the victim application to be invoked, adversaries may 
directly side-load their payloads by planting then invoking a legitimate application that executes their payload(s)."_

So we need to choose a legitimate EXE, find a DLL that it loads from disk, create a malicious DLL which runs shellcode, then copy or upload the EXE and DLL together to the same folder, and upon executing the EXE it should side-load the malicious DLL from within the same folder.

### Choosing an EXE and DLL to side-load
There are various methods we can use to find a legitimate EXE and DLL that it loads from disk. A public repository called [Hijack Libs](https://hijacklibs.net/) can easily be used to search for known EXEs and DLLs that could be used for DLL side-loading or DLL hijacking.

![hijack libs example](https://user-images.githubusercontent.com/35749735/192627446-8402c80e-af7d-4433-97ef-40b4965e3eea.png)

We could also use [Procmon](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) on Windows to analyze existing running EXEs and determine which DLLs are being loaded by the executable. We can load Procmon with the following filters to hunt for DLLs loaded by 64-bit processes.

![image](https://user-images.githubusercontent.com/35749735/192628830-12bbd8e2-3ddf-4b2a-9741-44634c1cfe0c.png)

We can scroll down the output list to find an executables and DLLs currently being loaded on the system. For this example, if we wanted to search for a DLL side-load against Windows Defender (**MsMpEng.exe**), we can add a filter for "__Process Name -> is -> MsMpEng.exe__". Assuming Windows Defender is currently running, we can then see the DLLs being loaded by the MsMpEng.exe process. The target DLL **mpsvc.dll** looks like a good target!

![image](https://user-images.githubusercontent.com/35749735/192629760-4a3b9418-fb9a-454b-888d-e895bd894c60.png)

The DLL **mpsvc.dll** is also a known DLL side-load that can be found in Hijack Libs at this [URL](https://hijacklibs.net/entries/microsoft/built-in/mpsvc.html).

Another option for finding your own DLL side-loads is to use the publicly available tool [Windows Feature Hunter (WFH)](https://github.com/ConsciousHacker/WFH) which has its own documentation for finding vulnerable DLL side-loads on your own system. If you prefer to go the easy route, there is a CSV list within the GitHub repo of discovered EXEs and their DLL side-loads [found here](https://github.com/ConsciousHacker/WFH/blob/main/examples/WFH_Dridex_System32_08172022.csv) which has over 900 DLL side-loads you could abuse.

### Finding exported functions from the DLL


### Creating a malicious DLL


### Putting it all together

