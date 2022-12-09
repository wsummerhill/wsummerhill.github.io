---
title: "CSharp Alterntive Shellcode Callbacks"
date: 2022-12-12
categories: redteam
published: false
---

For a while now, people have been using alternative callback methods in C/CSharp payloads instead of the vanilla "CreateThread()" 
or similar Windows API functions. There were several good repos on GitHub that can be used as resources to execute shellcode via Windows callback functions which were very interesting. 


To touch on the technical details of this functionality, Windows callback functions are _"code within a managed application that helps an unmanaged DLL function complete a task. Calls to a callback function pass indirectly from a managed application, through a DLL function, and back to the managed implementation."_
(Source: [MSDN Windows Callback Functions](https://learn.microsoft.com/en-us/dotnet/framework/interop/callback-functions)


Simpuly put, callback functions can be used to execute a task from your code (such as exeucuting shellcode)! And your code with callback functions would look like this:<br />
![image](https://user-images.githubusercontent.com/35749735/206265903-a15007be-40d8-4031-ab25-9b62ad517c8b.png)


Some of the most common callback functions that you may have heard of are EnumFontFamilies(), EnumPrinters(), and EnumWindows(). On top of that, there were MANY more documented callback functions that could be abused to execute shellcode in Windows. Here are some of the sources shared with me or that I came across:
- [DamonMohammadbagher/NativePayload_CB](https://github.com/DamonMohammadbagher/NativePayload_CBT)
- [aahmad097/AlternativeShellcodeExec](https://github.com/aahmad097/AlternativeShellcodeExec)
- [VX Underground Windows malware](https://www.vx-underground.org/windows.html)
- [Wra7h/FlavorTown](https://github.com/Wra7h/FlavorTown)


I realized that there were numerous resources for C/C++ code samples to execute shellcode via callback funtions, but fewer resources  available for CSharp. Therefore, I wanted to convert some C code callback samples to their C# equivalents or attempt to discover undocumented callback functions for shellcode execution.

