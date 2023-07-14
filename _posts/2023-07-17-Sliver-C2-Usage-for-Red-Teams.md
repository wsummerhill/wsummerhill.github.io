---
title: "Sliver C2 Deployment for Red Team Engagements"
date: 2023-07-17
categories: redteam
published: false
---

[Sliver](https://github.com/BishopFox/sliver/) has been a popular open-source C2 in recent years and has had continuous improvements since its release. It's cross-platform and easy to setup which were both appealing to me when I first started using it. I wanted to learn how to setup Sliver as a C2 server for red teams so I decided to document it for my own reference and anyone else interested. Here's what we'll cover:

- Sliver installation
- Connecting and Setup
- 

---------------------------------------------

## Sliver Installation

Sliver documentation: [https://github.com/BishopFox/sliver/wiki](https://github.com/BishopFox/sliver/wiki)

Spin up a Linux droplet/EC2/whatever to host your C2 server. Run the initial updates and install MinGW to allow compilation of shellcode/staged/DLL payloads:
```
apt update -y
apt update --fix-missing -y
apt install git mingw-w64 net-tools -y
```

Once the system is updated, Sliver is __very__ easy to install using the setup script: `curl https://sliver.sh/install|sudo bash` ([reference](https://github.com/BishopFox/sliver/#getting-started)). This will automatically start Sliver running in Daemon mode.

You can then create a client config on your Sliver server to allow "multiplayer" connections: `./sliver-server operator --name sliver-user --lhost "<SLIVER-IP>" --save /root/sliver-user.cfg`.<br />
Download the "**sliver-user.cfg**" to your local host using SCP or whatever file transfer app you like which you will use to connect to the server using a Sliver client.

__IMPORTANT STEP:__ *Don't forget to lock down your C2 server firewalls so that only you can access it. You can do this through firewalls in your cloud console or UFW rules. You will also need to open it up to your redirector (in our case over HTTPS) but more on this later.*

I created some Terraform scripts to help automate this whole process found in my [repository here](https://github.com/wsummerhill/Automation-Scripts/tree/main/Sliver-C2-deployment_DigitalOcean).

## Connecting and Setup

On your test system, download a Sliver client for your specific OS from the [link here](https://github.com/BishopFox/sliver/releases). Import your **sliver-user.cfg** config file and connect to the Sliver server with the following commands:<br />
```
/path/to/sliver-client_OS import sliver-user.cfg  # Import client
/path/to/sliver-client_OS  # Connect to Sliver C2
```

## Beacons


## Staged Payloads

References:
- https://github.com/BishopFox/sliver/wiki/Stagers
- https://dominicbreuker.com/post/learning_sliver_c2_06_stagers/

Beacon received from staged listener payload:
![image](https://github.com/wsummerhill/wsummerhill.github.io/assets/35749735/12d8f0df-a5d2-440f-972f-cd091a38b738)
