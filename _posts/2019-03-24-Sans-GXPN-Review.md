---
title: "Sans GXPN - Course Review"
date: 2019-03-24
categories: sans gxpn course certification cert pentest pentesting
published: true
---

In October of 2018 I had the lucky opportunity to take the Sans SEC660 ["Advanced Penetration Testing, Exploit Writing, and Ethical Hacking"](https://www.sans.org/cyber-security-courses/advanced-penetration-testing-exploits-ethical-hacking/) in-person course. I chose to take the course in Amsterdam since I heard great things about the instructor teaching the course there - Jake Williams. If you don't know Jake I suggest you look him up on twitter at [@MalwareJake](https://twitter.com/MalwareJake). He has seemingly infinite knowledge of security and can deep dive into any subject you ask him about on the spot. 

## Course Experience

The course itself was intense and very fast-paced. Jake went through each subject in the course quickly so you have to be paying attention to keep up while simultaneously taking notes. There's so much information in Sans courses that we frequently heard the phrase "drinking from a fire hose" which I can attest to. There is some time to stop and ask questions but that is usually only carved out during exercise/lab time, at lunch or after the course is done for the day.

### Day 1: Network Attacks
Day 1 gets started with a quick review of the course and setup of the lab environment then gets right into the material. The content starts off with network-based attacks around NAC, VLAN, MitM and some new Cisco protocols that I never knew of before. Some of the tools like Ettercap/Bettercap were review for me, although they went more in depth than what I previously knew. Some other MitM tools like Loki and ISR Evilgrade were mostly new to me as well. The afternoon composed of IPv6 attacks, more MiTM (sslstrip stuff) and SNMP based attacks. 

It was a good introductory day overall. Nothing overwhelming yet so I was feeling pretty good about it. The one thing I noticed immediately was the fast pace of the course and the considerate amounts of content covered.

### Day 2: Encryption and Post-exploitation
The second day got right into crypto once we got settled in. There was a quick review on stream ciphers, block ciphers, their mode types, and then went straight to the crypto-based attacks such as CBC bit flipping attacks, Oracle padding attacks and hash-length extension attacks. I thought I had a general idea of what these were before, and have fired off some exploits on them in the past, but to actually go deep into the low-level details of how the attacks worked on a cryptographic level was very interesting and complex (also fast-paced to follow along to). 

After that, the day covered restricted desktops, Powershell stuff (nothing new here) and escapes and privilege escalations. There were some new techniques involved for escapes, but not very often do I find myself in the situation where I need to use this type of knowledge. The priv esc stuff was mostly review since I've done the OSCP, but it still helps to cover some of the specific details around these attacks again. 

The labs for this day were fun. We got put in a structured environment and had to hack our way to the "boss's" machine and recover his sensitive files. It involved some enumeration, application escapes, privilege escalations and pivots to additional servers. 

Things are fast and the crypto was hard but I'm still feeling alright. Onto day 3!

### Day 3: Python and Scapy
Day 3 was a bit of review for me with the Python but it was a good introduction into fuzzing as well. The course goes over two main fuzzers - Sulley and American Fuzzy Lop (AFL). The exercises were useful to actually get your hands dirty with the fuzzers and the additional tools that work with them. 

### Day 4: Linux Exploitation
Now to start the real fun stuff. The Linux exploitation day started off with really fast-paced coverage of Linux memory and files then went straight into the GDB debugger and shellcode. There were a ton of new concepts here that I had no idea even existed before and it turns out this stuff is really important to know for exploitation development and to pass the exam. 

After that, the real Linux exploitation began. We first did a standard buffer overflow with GDB, then we learned about Ret2libc style attacks and went through a Ret2sys attack on a local Linux VM. There was a brief introduction into ROP and gadgets which was just a small taste of what was to come the next day. The more advanced stack smashing techniques included a very difficult to understand canary exploit (which I eventually got with help from a classmate and the instructor) then an intro to ASLR bypasses. We completed two separate ASLR buffer overflow techniques, one with the linux-gate-so.1 static library and another using the Exec() family of functions for newer version of Linux.

It's safe to say it was a really intense day and the first time I felt really over my head with this course. I went back to review more that night and do an additional bootcamp buffer overflow exercise since we didn't get to it during after-class hours. 

### Day 5: Windows Exploitation
If there's anything myself and many other classmates were feeling after today it's this: mind blown. Here's why...

The last day of class started out with a detailed outline of Windows memory, object files and OS security controls. It's boring yet important to cover these first since this is what we'll be exploiting very shortly.

After that, we got straight into overflow attacks to defeat hardware DEP with ROP.  We first did the basic buffer overflow against our target app to go through the steps of setting up the app and exploiting it while using a debugger. The next overflow was an SEH overwrite to control the SEH chain of exceptions and gain stack execution. This technique was a new concept for me so it was good to learn another new type of BO attack. 

The last buffer overflow covered in the course - and the most intense - was using ROP gadgets to execute the VirtualProtect() function to enable execution on pages of memory where your shellcode resides. This techniques works to bypass both hardware DEP and ASLR at the same time. If you've never done a ROP chain before then you're in for a nice yet excruciating surprise. By the time the instructor went through the attack and all of the gadgets the entire class was basically speechless. I think a couple people could follow along but I was one of the many who was mostly lost the entire time since it's very fast-paced and a totally new and complex technique. Thankfully you don't have to write your own ROP gadgets, however just the large exploit and script we used was hard enough to follow.

This was easily the most difficult day, and I felt wiped by the end of it since everything has been leading up to this point. The ROP chain attack was the nail in the coffin. But I still survived, and it was time to take a short break before CTFing tomorrow. 

### Day 6: CTF
CTF day - yay! The CTF was quite fun since it utilized tools and techniques learned throughout the previous 5 days. There were some good buffer overflows which got exceedingly difficult (I couldn't come close to doing them in time) and a good variety of both Windows and Linux attacks, crypto and cool trivia. My team ended up winning (thanks to them) so we got the coveted  660 challenge coin...

<img src="https://user-images.githubusercontent.com/35749735/186721665-efc957e8-d8bc-4587-b2fb-ce99607bb0cd.png" alt="image" width="400"/>
<img src="https://user-images.githubusercontent.com/35749735/186721710-cf23c418-38f2-48a2-a958-f52464e9afb8.png" alt="image" width="400"/>

-----

## Exam Prep 
Here's a short list of useful requirements to prep for the exam:

1. Read the books as many times as possible. Highlight ALL the important stuff, write it down, say it out load, then read it 5 more times.
2. Re-do ALL of the exercises and boot camps as much as possible. This can't be stressed enough. Especially the buffer overflows in books 4 and 5!
3. Make a strong cheat sheet and color-tab your books. I'd highly recommend some methods in the "[Better GIAC testing with pancakes](https://tisiphone.net/2015/08/18/giac-testing/)" article.
4. Use both of the practice exams and take them seriously as if they were a real exam. Take notes during the practice exams too if you have time. They are in the EXACT same format as the real exam so this was critical for me to have more confidence going into the real exam. And practice the sections that you did poorly on even further before the final exam.

### Exam
I took the exam about 3 months after I started the course and I had scheduled it for a Saturday early in the afternoon. I figured there may be a lot of tricky questions or curve balls so I couldn't help but overthink some things going into it which is only natural.

The exam was 60 questions long and scheduled for 3 hours. The last 6 questions at the end are reserved for the in-browser VM practical questions. Knowing this, make sure you divide your time properly so you have enough time for the practical questions. I planned to spend 45 minutes on the practical questions but didn't end up needing that long at all.

Once I got into the exam I knew what I felt comfortable immediately. I gained a lot of confidence and was certain most of my answers were correct. It's important to read everything very carefully, both the questions and answers, to make sure you don't miss anything and choose the right answer. Once I got to the remaining 6 practical questions I knew I was doing well. These questions were just like the practice exams so I knew what to expect. Most of them were easy if you're prepared, and there was one really fun exploit requiring a buffer overflow to answer.

Overall, I had a great and unique experience with my first Sans course and exam. The bootcamp style course is evidently intense and covers A LOT of information. This made studying a lot longer since there is so much material to know and review. Passing that exam felt amazing especially after feeling so prepared. I'd do it again but I'm happy to finally take a break until I figure out what to do next.

[YourAcclaim Link](https://www.credly.com/badges/416bcf6f-c8e6-4a9d-85a9-289e9756b3e9)
