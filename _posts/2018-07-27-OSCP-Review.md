---
title: "OSCP Review"
date: 2018-07-27
published: true
---

# Review of my OSCP experience
[Course link](https://www.offensive-security.com/pwk-oscp/)

I signed up for the OSCP in January of 2017 for 90 consecutive days of lab access. At the time I haven’t pentested as a profession before but had experience with Kali and BackTrack in a 1 year post-graduate program for Computer and Network Systems Security. This course, as well as my computer science and programming background, helped me have a base understanding of some of the concepts in the course and with scripting abilities which proved to be useful (Python, specifically).

The Penetration Testing with Kali Linux (PWK) course is the online course designed for the OSCP exam. I’d highly recommend everyone be efficient in Kali and know their way around a Linux box before even attempting the PWK course. Being proficient with Metasploit is another useful skill to have which is invaluable to know before starting. One can go through the free online [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) course beforehand which has key concepts for Metasploit that are continuously used in the PWK course.


## Lab Review

The lab was full of fun, challenging, frusturating, and hair-pulling hacking goodness. Since beginning the lab I worked on it most nights and somewhat on the weekends. It started off like most others - quickly knocking down a handful of root shells on easy to hack boxes. Simple exploits involving Metasploit, SMB or default creds were quickly discovered on these which gave me some quick confidence in the first week or two of the lab time. I should note that once I got my confidence up a bit it quickly diminished by totally getting destroyed later on.


In the lab, the boxes which don’t have any dependencies should be focused on first (i.e. they don’t require other boxes). While going through each machine, don’t give up right away when you hit a difficult one. Try it for a day or two at least and read some forums about it if needed. Sometimes the forums will tell you if it requires a dependency or not which helps. After exploiting each machine, MAKE SURE to do proper post-exploitation steps.

Post-exploitation steps may include:
- Look in the root folder and /home directory of users (Linux)
- Look at the desktop of the Admin user and any other users on the system (Windows)
- Search ‘My Documents’ or ‘My Downloads’ for all users (Windows)
- Much more for you to figure out…


The three big machines are **SUFFERANCE**, **HUMBLE** and **PAIN**. Each are extremely difficult in their own way but very rewarding. Don’t skip on attempting these but maybe save them for when you’re ready and have already owned a fair amount of machines. 


In my first 3 months of lab access I was able to get 35 to 40 machines by the time my lab expired. I was pretty happy with this but still wasn’t sure about how I’d do in the exam considering its difficulty level I kept hearing about. I didn’t go too far into the additional networks (outside of the main network) since I didn’t have time but I got most of the main network and the dependent machines. I booked my exam before finishing the lab since there’s a wait time for an available exam time slot and I recommend you do the same if you don’t want to wait after finishing your lab. I had about a week before my exam after my lab time ended which seemed fair. 

Pre-exam review:
- Practice privilege escalation on both Linux and Windows!
- Get your privilege escalation scripts/tools ready
- Have some commonly used exploits ready to go - downloaded/compiled for quick use
- Review the buffer overflow again to be sure you can knock it out right away, but be prepared for some curve balls in case
- Finally, review any weak spots you may have


## Exam #1 Experience

I scheduled the exam to begin on a Saturday morning around 10:00am. I figured this way I could get a good sleep before, have breakfast, then get ready to roll with a full day of hacking ahead and reporting the following day. I started off with the buffer overflow and made some crucial mistakes (even though I told myself I wouldn’t). I knew how to do it well beforehand, but I didn’t go thoroughly enough through the bad characters and missed some which then messed up my shellcode from executing properly. I got frusturated early and went back to it later that night and finally figured it out. But it’s safe to say it wasn’t a good start. 

After that I reviewed Nmap files created from a simple bash script I wrote which I ran at the beginning of the exam. The script ran a SYN scan on all 65,535 ports and a quick UDP scan on the top 1000 ports, which were then saved in output files for each target (USEFUL TIP). 

Throughout the day I eventually got the 25 points from the BO and only 10 points on one of the boxes but didn’t get the privilege escalation. Late in the evening I was getting worn down and de-motivated so I took a food break to refresh and think about what to do with the time remaining. I figured I wasn’t going to pass and got a few hours of sleep before trying agian in the morning before my exam time expired. By the end of it I had 2 boxes with 3 remaining - only half the points required to pass. In my case it didn’t even make sense to bother with the report since I knew I had already failed. 

_I was demotivated at first after failing my first attempt but I knew what I needed to do in order to pass._

The next month was dedicated to another 30 days of lab time and I knew what I had to do to pass that grueling exam. I went through all the boxes on the main network one by one without looking at my old notes and tried to hack them as fast as I could. This way I was preparing myself for the exam by doing things as fast as possible, including privilege escalation, since I knew you can’t mess around in that 24 hour time period. 

After that, I focused on privilege escalation for both Windows and Linux from some of the machines which I already knew the credentials for. Then it was go-time again… 


## Exam #2 Experience

I booked the exam at a similar time and day as the previous attempt. I was well rested and felt prepared for what the next 24 hours would bring. I started off strong with the buffer overflow and mad sure I didn’t make the same mistakes as last time by missing any bad characters. Within the first 20 minutes I had the BO for 25 quick points and was onto the rest of the network. 

I didn’t focus on any particular machine and just attempted what looked simple enough from the outside to gain some more confidence. I got one of the machines pretty quickly for another 20 points then started working on the 25 point box after. Later in the day I got a low-priv shell on the 25 point machine but couldn’t figure out the privilege escalation yet. 

With 60 points total and about 8 hours into the exam I was feeling confident. I soon got the other 20 point machine and felt unbeleivably elated and releived that I was going to pass! I made sure to grab ALL the screenshots required since this could make or break my final score, and I highly recommend you do the same after each machine and before your exam time expires. I got a few hours of sleep that night while feeling more relaxed and happy than I have been in a long time. The morning was reserved for looking at the remaining priv-esc and the last 10 point box, but more importantly gathering any remaining required screenshots for the report. 

The next day I worked on the report for most of the day and wrote a total of ~28 pages. I made sure to package everything up properly and finally submitted it off to Offsec. Now all I can do is patiently wait and question the validity of my screenshots (even though I double-checked all of them). 


Several days later I received one of the best emails to ever show up in my inbox.

### **_I PASSED! What a amazing journey._**


## Final word

I can easily say that the PWK course and OSCP exam was the best experience I’ve ever had with a certification thus far. 
The material is presented to you and it’s up to you to figure out how to use it and learn from it which translates across many domains - not just pentesting or infosec. Failure is part of the learning process which contributes to the end result if you can stick it out. You just have to learn from that failure and keep trying, and of course Try Harder.

I’m looking forward to getting into the OSCE course soon for another tough and daunting challenge. Thanks OffSec.


Student ID: **OS-26982**
