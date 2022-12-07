---
title: "CSharp Alterntive Shellcode Callbacks"
date: 2022-12-09
categories: redteam
published: false
---

For a while now, I've been using alternative callback methods in my C and CSharp payloads instead of the vanilla "CreateThread()" 
or similar Windows API functions. There were several repos on GitHub in C and CSharp used to execute shellcode via Windows callback functions which were very interesting. 


If you are unfamiliar with the term, Windows callback functions are _"code within a managed application that helps an unmanaged DLL function complete 
a task. Calls to a callback function pass indirectly from a managed application, through a DLL function, and back to the managed implementation."_
(Source: [MSDN Windows Callback Functions](https://learn.microsoft.com/en-us/dotnet/framework/interop/callback-functions)

So your code with callback functions would look like this:<br />
![image](https://user-images.githubusercontent.com/35749735/206265903-a15007be-40d8-4031-ab25-9b62ad517c8b.png)


Some of the most common callback functions that you may have heard of are EnumFontFamilies(), EnumPrinters(), and EnumWindows(). On top of that, there 
were MANY more documented callback functions that could be abused to execute shellcode in Windows. Some of the sources shared with  me or that I came across include:
- t
- t
- [VX Underground Windows malware](https://www.vx-underground.org/windows.html)

