---
layout: post
title: "X230 Coreboot Story using Skulls"
date: 2020-03-04
---

YES. This is a noob story, you've been `warned`.
 
**Preambule**  
Actually, before i decide to using try hardware flash using skulls coreboot, i've already try install custom bios images from [1vyrain](https://github.com/n4ru/1vyrain/){:target="_blank"}  to remove whitelist hardware and pop out BIOS advanced menu, shout out to [1vyrain](https://1vyra.in/){:target="_blank"}  
<img src="\images\coreboot\1vyrain.jpg" alt="1vyrain" title="1vyrain" width="300" height="200" />  

[Skulls](https://github.com/merge/skulls){:target="_blank"}  is pre-built coreboot images for Thinkpad X230.  
I have two X230 and one of them i'm using as workhorse for my daily driver. 
Actually flashing done in under 1 hour if all smooth. I'm doing this in under 24 hours with a lot of confusion so i want to share experience do this. I suggest you have spare laptop if something bad happen.
Before flashing, i've read some article below, just make sure you read all section thoroughly and understand what you're doing, do your homework.
Always remember that you can brick your laptop when you do this wrong.  
Here's the great link to consider before do flashing :  
[skulls](https://github.com/merge/skulls/blob/master/x230/README.md){:target="_blank"}  
[chucknemeth](https://www.chucknemeth.com/laptop/lenovo-x230/flash-lenovo-x230-coreboot){:target="_blank"}  
[stefanolaguardia](https://www.stefanolaguardia.eu/2019/06/08/howto-install-coreboot-and-disable-intel-me-on-lenovo-thinkpad-x230/){:target="_blank"}  
[joeyd](https://steemit.com/tutorial/@joeyd/run-don-t-walk-from-the-blob){:target="_blank"}

**Tools i've been using :**  
Thinkpad X230  
Another laptop/PC installed ubuntu  
CH341a  
Female to Female jumper cables (6 unit)  
SOIC8 clip  
Screwdriver

<img src="\images\coreboot\X230 and colleague.jpg" alt="X230 and colleague" title="X230 and colleague" width="270" height="200" />
	<img src="\images\coreboot\CH341a f2fcables SOIC8.jpg" alt="CH341a f2fcables SOIC8" title="CH341a f2fcables SOIC8" width="300" height="200" />
		<img src="\images\coreboot\Screwdriver.jpg" alt="Screwdriver" title="Screwdriver" width="150" height="250" />

**TL;DR**
1. Backup backup backup.  
Always and always backup your BIOS/Factory ROM, check and re-check again using **"shasums"** until it exactly same when you already get the copies of you factory
ROM. As far as i know, if anything goes wrong, you can restore again your factory ROM.
2. CH341a 'noob' issue.   
I've try to make connection between my host laptop and BIOS chip using CH341a and SOIC8 clip, but whenever i check connection using flashrom, 
it always error `"Calibrating delay loop... OK. Couldn't open device 1a86:5512. Error: Programmer initialization failed."`
Damn, i've almost desperate that i've thinking is this CH341a is defect. I've googling this problem for hours and still i've found that no other face
this same issue. Until, i've got [this](https://axlforsat.wordpress.com/2018/08/05/tutorial-flash-ic-reciever-dengan-ch341a-dan-test-clip/){:target="_blank"}.
When i've got CH341a at the first time, i noticed that in the packet there is yellow little thing that i don't know what is that for. Turns out that is IC
'jumper'.
In short when you want CH341a to upgrade IC receiver you must put that 'jumper' in pin 1 and pin 2. After do that, connection in flashrom is detected.
Take me hours before i know this, so when you have a problem, remember : start trace from the root.
<img src="\images\coreboot\jumper off.jpg" alt="jumper off" title="jumper off" width="300" height="200" />
	<img src="\images\coreboot\jumper on.jpg" alt="jumper on" title="jumper on" width="300" height="200" />
3. Windows compatibility issue.  
I've using Windows 10 for my daily driver and ubuntu 18 in other laptop for development purpose. Before flashing my Windows 10 main laptop, i've tested flashing in
ubuntu 18 laptop and everything going smoothly. I can boot to ubuntu and all is going well. Different story when i'm flashing my Win10 laptop, i can't booting into
OS, just stuck for hours at seabios payload. Since skulls using SeaBIOS for coreboot payload, i've found some article and discussion about [windows compatibility using
coreboot](https://www.coreboot.org/SeaBIOS#Windows){:target="_blank"}. I quote the article says `"SeaBIOS has been tested with Windows XP, Windows 2008, Windows Vista (64/32 bit), Windows 7 (32 bit and 64 bit). However, Windows has a very strict ACPI interpreter, and some coreboot boards do not have a complete ACPI definition. As a result, some coreboot boards may fail during Windows boot (eg, it may fail with a STOP 0xA5 code). Many boards do have working ACPI and are able to boot XP/Vista/Windows 7. Please check the board documentation or ask on the mailing list if unsure of the status."`
So i think the problem is ACPI and damn, i've already flash my Win10 laptop when i read this and i've too lazy to turning back so i've take the red pill, i've copy all my data and re-install again Win10 (take me damn half day to do this).
4. Re-install windows issue.  
When you install Windows in skulls coreboot, just remember this : make sure that every setting in skulls coreboot is exactly the same when you install Windows and after install Windows. I face this issue that after i install Windows and booting back, Windows going BSOD immediately, even before i install Windows driver in Device Manager. Turns out, i've changed the ACPI setting when restart windows,so this issue happen. Not sure, but maybe that's the problem.
5. Memtest86 coreboot issue.  
Still happen till now, always stuck when do memtest, memory configuration is dual memory with same speed but different size and brand.  
But when try using single channel, it pass with no error. Nonetheless, no problem for me using dual channel as daily driver config.  
<img src="\images\coreboot\dual channel.jpg" alt="dual channel" title="dual channel" width="300" height="200" />
	<img src="\images\coreboot\single channel.jpg" alt="single channel" title="single channel" width="300" height="200" />

Ok, that's quick recap what i've done when try flashing using skulls, nevertheless i've been really enjoy do this hardware flashing.  
Here some photos i took when do flashing, have fun.   

<img src="\images\coreboot\1.jpg" alt="1" title="1" width="300" height="200" />
	<img src="\images\coreboot\2.jpg" alt="2" title="2" width="300" height="200" />
		------------------------------------------------------------------------------------------------------------------
		<img src="\images\coreboot\3.jpg" alt="3" title="3" width="300" height="200" />
	<img src="\images\coreboot\4.jpg" alt="4" title="4" width="300" height="200" />
		------------------------------------------------------------------------------------------------------------------
		<img src="\images\coreboot\5.jpg" alt="5" title="5" width="300" height="200" />
	<img src="\images\coreboot\6.jpg" alt="6" title="6" width="300" height="200" />
