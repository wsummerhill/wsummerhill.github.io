---
title: "Malware Development: CSharp Alterntive Shellcode Callbacks"
date: 2022-12-09
categories: malware
published: true
---

For a while now, people have been using alternative callback methods in C/CSharp payloads instead of the vanilla `CreateThread()` 
or similar Windows API functions. There were several good repos on GitHub that can be used as resources to execute shellcode via Windows callback functions which were very interesting. 


To touch on the technical details of this functionality, Windows callback functions are _"code within a managed application that helps an unmanaged DLL function complete a task. Calls to a callback function pass indirectly from a managed application, through a DLL function, and back to the managed implementation."_
(Source: [MSDN Windows Callback Functions](https://learn.microsoft.com/en-us/dotnet/framework/interop/callback-functions)


Simpuly put, callback functions can be used to execute a task from your code (such as exeucuting shellcode)! Your code with callback functions would look like this:<br />
![image](https://user-images.githubusercontent.com/35749735/206265903-a15007be-40d8-4031-ab25-9b62ad517c8b.png)


Some of the most common callback functions that you may have heard of are `EnumFontFamilies()`, `EnumPrinters()`, and `EnumWindows()`. On top of that, there were MANY more documented callback functions that could be abused to execute shellcode in Windows. Here are some of the favourites shared with me or that I came across myself:
- [DamonMohammadbagher/NativePayload_CB](https://github.com/DamonMohammadbagher/NativePayload_CBT)
- [aahmad097/AlternativeShellcodeExec](https://github.com/aahmad097/AlternativeShellcodeExec)
- [VX Underground Windows malware](https://www.vx-underground.org/windows.html)
- [Wra7h/FlavorTown](https://github.com/Wra7h/FlavorTown)


I realized that there were numerous resources for C/C++ code samples to execute shellcode via callback funtions, but fewer resources  available for CSharp. So I decided to convert some C code callback samples to their CSharp equivalents or attempt to discover undocumented callback functions for shellcode execution.


Below is a list of all the callback functions I've created malware samples for (so far) to execute shellcode in CSharp:
```
AddPropSheetPageProc
CertEnumSystemStore
CertEnumSystemStoreLocation
CreateTimerQueueTimer
CryptEnumOIDInfo
DSA_EnumCallback
EncryptedFileRaw
EnumDateFormatsA
EnumFontFamiliesW
EnumLanguageGroupLocalesW
EnumObjects
EnumSystemCodePagesA
EnumSystemGeoID
EnumerateLoadedModules
FiberContextEdit
ImmEnumInputContext
InitOnceExecuteOnce
LdrEnumerateLoadedModules
NotifyIpInterfaceChange
NotifyTeredoPortChange
SetTimer
SetupCommitFileQueueW
StackWalk
SymEnumProcesses
SymRegisterCallback
```

And finally, here is the GitHub repo of all the documented callback function malware samples I created in CSharp:<br />
LINK: [My CSharp Alternative Shellcode Callbacks repo](https://github.com/wsummerhill/CSharp-Alt-Shellcode-Callbacks)

