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
- How to create the malicous DLL
- How to find the exported functions needed to side-load the DLL
- Putting everything together for a successful DLL side-load attack

But first...

### What is DLL side-loading?

[Source: MITRE](https://attack.mitre.org/techniques/T1574/002/)
_"Side-loading involves hijacking which DLL a program loads. 
But rather than just planting the DLL within the search order of a program then waiting for the victim application to be invoked, adversaries may 
directly side-load their payloads by planting then invoking a legitimate application that executes their payload(s)."_

So we need to choose a legitimate EXE, find a DLL that it loads from disk, create a malicious DLL which runs shellcode, then copy or upload the EXE and DLL together to the same folder, and upon executing the EXE it should side-load the malicious DLL from within the same folder.

### Choosing an EXE and DLL to side-load
There are various methods we can use to find a legitimate EXE and DLL which it loads from disk. A public repository and great resource called [Hijack Libs](https://hijacklibs.net/) can easily be used to search for known EXEs and DLLs that could be used for DLL side-loading or DLL hijacking. We could use this tool to filter on specific vendors of types of DLL hijacks such as side-loading.

![hijack libs example](https://user-images.githubusercontent.com/35749735/192627446-8402c80e-af7d-4433-97ef-40b4965e3eea.png)

Alternatively, we could also use [Procmon](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) from Windows SysInternals to analyze existing running executables and determine which DLLs are being loaded by the process. We can load Procmon with the following filters to hunt for DLLs being loaded by 64-bit processes.

![image](https://user-images.githubusercontent.com/35749735/192628830-12bbd8e2-3ddf-4b2a-9741-44634c1cfe0c.png)

We can scroll through the output of Procmon to find an executables and DLLs that are currently being loaded on the system. As an example, if we wanted to search for a DLL side-load against Windows Defender (**MsMpEng.exe**), we can add the following filter: "__Process Name -> is -> MsMpEng.exe__". Assuming Windows Defender is currently running, we can then see the DLLs being loaded by the MsMpEng.exe process to side-load. The target DLL **mpsvc.dll** looks like a good target!

![image](https://user-images.githubusercontent.com/35749735/192629760-4a3b9418-fb9a-454b-888d-e895bd894c60.png)

The DLL **mpsvc.dll** is a known DLL side-load that can be found in Hijack Libs at this [URL](https://hijacklibs.net/entries/microsoft/built-in/mpsvc.html).

Another option for finding your own DLL side-loads is to use the publicly available tool [Windows Feature Hunter (WFH)](https://github.com/ConsciousHacker/WFH) from [@ConsciousHacker](https://twitter.com/conscioushacker) which has its own documentation and method for finding vulnerable DLL side-loads on your own system. If you prefer to go the easy route, there is a CSV list within the GitHub repo of discovered EXEs and their DLL side-loads [found here](https://github.com/ConsciousHacker/WFH/blob/main/examples/) which has over 900 DLL side-loads you could abuse.

### Creating a malicious DLL

Once we've found our executable and DLL to target, we then need to create our malicous DLL. In this example we're going to target the **MsMpEng.exe (Windows Defender)** with the DLL side-load of **mpsvc.dll**.

### Finding exported functions from the DLL

Before we can finalize the malicious DLL, we need to get a list of exported functions from the existing DLL on disk. 

### Putting it all together

