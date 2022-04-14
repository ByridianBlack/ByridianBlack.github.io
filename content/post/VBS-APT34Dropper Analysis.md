---
title: "VBS APT34 Dropper Analysis"
date: 2021-05-22T10:22:10+07:00
tags: ["VBS","APT","dropper"]
images: [
    https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/APT34Dropper/image_000.png,
]
toc: true
---

## Introduction

VBS scripts are malicious codes that can contain PowerShell commands which can have serious damage on the victim machine.. They can be found embedded in other common document formats which are used every day such as PDF documents, and Word documents. While most come off as non-malicious these program sizes can grow very big and be hard to detect to the untrained eye as they usual obstruct themselves through compression and encoding in formats such as base64 or GZIP. The Win32.VBS.APT34Dropper was one of these scripts which I have analyzed.

## Initial Analysis

Opening the VBS script up in Visual Studio Code (vscode) I see a could things. I see file paths and I see data which looks to be encoded in base64. Let us focus in on the first part of this script.


![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/APT34Dropper/image_001.png)

Here I see that the program is making sure that these file paths and files are created and if not then they are creating them for the malware. It is then writing to the script dhupdateChecker.ps1 what appears to be a set of PowerShell commands. Ok, good. I now know I am dealing with a VBS script which will be creating a PowerShell process. This knowledge will help me filter out unnecessary information when I am analyzing system processes.

From here we also know that the scripts GoogleUpdatechecker.vbs and dUpdateCheckers.ps1 are both files which this malware uses so these can be host base indicators and it can also be a hint at what this malware is doing.

By calling itself GoogleUpdateChecker.vbs it is essentially hiding itself. As it is possible going to talk to the internet this name could fly under the radar of someone that is not really looking deeply at a network packet.

## First Part Decode

Moving on. The next part of this is a check if the file hupdateCheckers.base exists. If not, the following information is stored in the file. This format is base64.

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/APT34Dropper/image_002.png)

Once decoded, the following was found:

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/APT34Dropper/image_003.png)

This code gets network information, extracts the computers UID and appears to be contacting and downloading data from the internet.

Also, we see the URL www.mumbai-m.site/update_wapp2.aspx. During dynamic analysis this should be observed.

## Second Part Decode

The next part is essentially doing the same thing of base64 decode so I will just show you the end result.

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/APT34Dropper/image_004.png)


This was very obstructed, and it took my a few minutes to make it neater. I will leave that up to you to do. This code drop does 3 main things.

It forms a unique URL based on the computer you are running on and contacting that URL. What ever is being received is being written to a file. This code is written to dUpdateCheckers.base

The last part of this malware is this last piece which is what gives this malware the best chance of not being detected and gaining persistence.

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/APT34Dropper/image_005.png)

Here we see that a file called cUpdateCheckers.bat is being written to the variable code4. In this is a command which sets a scheduler which runs the scripts we saw earlier every minute. This allows this piece of malware to run even after reset of the victim computer. Even if certain files are deleted if the main component which can regenerate these deleted files is not removed then this script can keep regenerating itself repeatedly with each passing minute.

This malware also deletes the .base files at the end of it so its cleaning itself up to avoid detection. It is also hiding itself behind the PowerShell process all of which is being run in the background with no window ever being show to the user.

## Dynamic Analysis

Dynamic Analysis was done in two parts. Networked and un-networked. I decided to start with un-networked but that showed me everything I have already proved with static analysis. All the files I found in the VBS script were created and its behavior was what I expected. I then moved on to Network analysis. This was done with INetSim and Wireshark.

## Network Set

* Ubuntu 20.02 LTS Virtual Machine
* Windows 7 Pro Virtual Machine (Victim Machine)
* INetSim & Wireshark; Both Hosted on the Ubuntu Virtual Machine.

Creating a Host Only Network that connected the Machines together I was able to redirect all network traffic from the victim machine into the Ubuntu Machine which captured all network traffic. When doing this remember to turn of your firewall.

Capturing the data lead me to see a unique URL every few seconds, where for the majority of it, the words Mumbai-m.site stayed with it but a unique 23 Identifier was appended to it. I was not able to determine what this code was looking for but the constant pining and need for response never stopped.

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/APT34Dropper/image_006.png)

Further analysis using the WayBackMachine on https://web.archive.org lead to no further information but I can come to some conclusion based of the static analysis I saw later.

## Summary and Conclusion:

This script appears to be malicious but in modern days since the original URL it is trying to contact does not appear to be of use anymore this malware cannot do any real harm. When analyzing this piece of code it is important to comment out the oShell.run command at the bottom so that you don’t accidentally cause this malware to run.

From statically and dynamically analyzing this piece of malware I was able to confirm that it is indeed a dropper. It is constantly pinging a website, possible waiting for more information to come its way and to download a file if need be.

This piece of malware took several steps to both hide and protect itself from reboots and deletion and showed itself to have regenerating capabilities what attempted to ensure its survival. Also using steps to place itself in places that are deemed trustworthy like in java directories and look like a standard update tool was a way to make itself into a Trojan and keep from being detected.

An unknowledgeable person would not have been able to handle this which is what makes this malware so dangerous especially because its size is so small and unsuspecting. With a unknowing clicking, much damage can be done to the victims PC in the background, in such a way that a victim would not know its source.

This piece of malware does leave Host and Network indicators which can be found rather easily. It attempts to self-delete parts of its self which can be stopped by editing the script and by analyzing the network packets finding keywords like Mumbai.site could easily lead to its detection.

## Tools & Research Sources

* Procmon, Process Explorer
* Visual Studio Code
* VirtualBox
* INetSim
* Wireshark
* https://www.base64decode.org/
* https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks
* https://github.com/ytisf/theZoo (For Malware Samples)
* https://web.archive.org
