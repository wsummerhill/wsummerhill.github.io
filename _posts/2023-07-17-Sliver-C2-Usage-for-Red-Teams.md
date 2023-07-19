---
title: "Sliver C2 Deployment for Red Team Engagements"
date: 2023-07-17
categories: redteam
published: false
---

[Sliver](https://github.com/BishopFox/sliver/) has been a popular open-source C2 in recent years and has had continuous improvements since its release. It's cross-platform and easy to setup which were both appealing to me when I first started using it. I wanted to learn how to setup Sliver as a C2 server for red teams so I decided to document it for my own reference and anyone else interested. Here's what we'll cover:

- Sliver installation
- Connecting and Setup
- Beacons
- Staged Payloads

---------------------------------------------

## Sliver Installation

Sliver documentation: [https://github.com/BishopFox/sliver/wiki](https://github.com/BishopFox/sliver/wiki)

Spin up a Linux droplet/EC2/whatever to host your C2 server. Run the initial updates and install MinGW to allow compilation of shellcode/staged/DLL payloads:
```
apt update -y
apt update --fix-missing -y
apt install git mingw-w64 net-tools -y
```

Once the system is updated, Sliver is __very__ easy to install using the setup script ([reference](https://github.com/BishopFox/sliver/#getting-started)): <br />
`curl https://sliver.sh/install|sudo bash` <br />
This will automatically start Sliver running in [Daemon mode](https://github.com/BishopFox/sliver/wiki/Daemon-Mode). 

You can then create a client config on your Sliver server to allow "multiplayer" connections: <br />
`./sliver-server operator --name sliver-user --lhost "<SLIVER-IP>" --save /root/sliver-user.cfg`.<br />
Download the output "**sliver-user.cfg**" file to your local host using SCP or whatever file transfer app you like which you will use to connect to the server using a Sliver client.

__IMPORTANT STEP:__ *Don't forget to lock down your C2 server firewalls so that only you can access it. You can do this through firewalls in your cloud console, UFW rules, IPTables, etc. You will also need to open it up to your redirector (in our case over HTTPS) but more on this later.*

I created some Terraform scripts to help automate this whole process found in my [GitHub repository here](https://github.com/wsummerhill/Automation-Scripts/tree/main/Sliver-C2-deployment_DigitalOcean).

## Connecting and Setup

On your test system, download the Sliver client for your specific OS from the [link here](https://github.com/BishopFox/sliver/releases). Import your **sliver-user.cfg** config file from the previous steps and connect to the Sliver server with the following commands:<br />
```
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
To do this with UFW rules, it would look something like this:<br />
```
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

Moving on... Now we'll have to start an HTTPS listener in Sliver which only accepts connectsion from our redirector host. But first, we'll need to setup an SSL certificate with a public/private key (without encryption) to have a valid SSL/TLS connection. You could use a self-signed one (using the `--self-signed` option), but for red teams we would much rather have a valid/signed cert for our HTTPS connections to be less suspicious.

To create your SSL/TLS certificate for HTTPS communications with your listener, you can use whatever service/app you prefer. LetsEncrypt is probably the most common method, and recently I started using AWS Certificate Manager (ACM) certs.<br />
To automate this with LetsEncrypt, I modified one of RedSiege's script for setting up a cert Cobalt Strike and made it compatible with Sliver. [Link HERE](https://github.com/wsummerhill/Automation-Scripts/blob/main/HttpsGenericC2DoneRight.sh).<br />
Drop the `HttpsGenericC2DoneRight.sh` script on your Sliver teamserver and run it with Sudo privileges. Enter in your C2 domain (with an A record pointing to your C2 server) and any private key password:
![image](https://github.com/wsummerhill/wsummerhill.github.io/assets/35749735/fd02dfbc-dc55-4ad4-83d9-1bea28e1a12e)

If everything works properly, the script should output you a domain store file, cert.pem and privkey.pem files in your /root directory. Copy these files to your local system with your Sliver client installed.
![image](https://github.com/wsummerhill/wsummerhill.github.io/assets/35749735/a57e5b26-eee7-4556-aa19-5fa3b8307204)

## Beacons

Sliver C2 implants communicate in 2 different ways: **Beacons** or **Sessions**. For red teams and better opsec we will always be using Beacons which use asynchronous communications that periodically check-in at a certain time interval. The alternative would be Sessions which use an established interactive mode connection (noisy and poor opsec). <br />
Beacons function the same way Cobalt Strike communication channels work using asynchronous connections. The one downside of Sliver is that it doesn't have a sleep mask like Cobalt Strike does, meaning the Beacon code won't be obfuscated/encrypted in-memory while sleeping between connections. 

To create a new Beacon using your HTTPS redirector (you previously set this up right?), use the following command:
``


## Staged Payloads

Useful links:
- https://github.com/BishopFox/sliver/wiki/Stagers
- https://dominicbreuker.com/post/learning_sliver_c2_06_stagers/

Beacon received from staged listener payload:
![image](https://github.com/wsummerhill/wsummerhill.github.io/assets/35749735/12d8f0df-a5d2-440f-972f-cd091a38b738)
