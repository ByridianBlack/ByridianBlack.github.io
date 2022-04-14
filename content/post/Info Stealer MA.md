---
title: "Win32/InfoStealer Dexter Malware"
date: 2021-05-22T10:22:10+07:00
tags: ["keylogger","resource","dll"]
images: [
    https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/keylogger_main_image.jpeg,
]
toc: true
---
 
# Introduction
 
Win32/InfoStealer.Dexter is part of a family of malware with the purpose of stealing information such as credit card numbers, passwords or various techniques. This sample held anti analysis techniques, along with basic steal techniques to hide its true abilities. This could also be caterogized as a trojan.
 
# Static Analysis
 
Upon copying the piece of malware onto the workstation it was inspected with PE Explorer and PEID to determine if it was packed. While PEID showed that there was no commerical packer used on this executable, the difference between the virtual Size and raw size difference found in PE explorer told a different story.

![pe_explorer](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/image-000.png)

Openin the .rsrc section the following was a Found. A PE ptogram. This will be set aside for later but for now it should be noted that this was indeed the keylogger portion of the executable.

![packed_resource_section](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/image-001.png)

What should be noted here is that even upon saving the binary inside the rsrc section and opening it in IDA pro, I could not see any functions, leading me to believe that it was compressed or encrypted in some way to stop analysis.

## IDA PRO Analysis
Analysis using IDA pro was effective and there were little to no anti disassbly techniques used. The following data was found when using the strings utility:

1. Software\\\Microsoft\\\Windows\\\CurrentVersion\\\run
2. UpdateMutex:
3. 151.248.115.107
4. Gateway.php

## Binary Analysis

The executable takes in a command line argumne tof UpdateMutex: and upon doing so it was determined that a new Mutex of UpdateMutex:< current Process ID > is being created. This mutex is later destroyed but this shows that this mutex is being created dynamically.

The main mutex being created is called WindowsServiceStabilityMutex which can be used as a host based indicator.
![main_mutex](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/image-002.png)

## Heavy Usage of GetProcAddress

The function GetProcAddress was called many times to resolve API in a way which would not have showed up in the IAT table. The functions which were being resolved were RtlCompressBuffer, RtlDecompressBuffer, and RtlGetCompressWorkSpaceSize. Upon further research I discovered that this is a common malware method for packing parts of malware. It is a much more older method but the method of storing a packed form of data in a section and then unpack it during execution still exists.

![packing](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/image-003.png)

In this case, this data was stored in the resource section and written to a buffer, This buffer was then written into a file called SecureDll.dll. This data was the same bianry discovered earlier but this time decompressed and able to be analyzed. SecureDll.dll name itself shows that it is trying to hide itself as a legit windows dll. The dll itself was tored in the same directory of the executed program but hidden. It was simple enough to unhide it.

![dll_hide](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/image-004.png)

Upon further analysis I discovered that this dll was a keylogger. The hackers relied on the unpacking and packing of their original malware package along with it hiding itself rather than obfusctration techniques. Their dll file was estentially untouched and very easily to read. Further analysis showed that it was capturing keyboard hooks.

![keylog](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/image-005.png)

# Dynamic Analysis

Dynamic Analysis was done in two portions. Connected and disconnected from the network. For the connected portion, I used a host-only netowrk, utilized inetsim to simiulate a network that the program could connect to. The network informationwas then ccaptured using wireshark.

## Un-Networked:

By double clicking the executable it deletes itself. Therefore the program must be then run via the command line. As mentioned before the command line argument UpdateMutex: can be used but this kept the program in a constant loop. The belief here is that this had something to do with the older version of the software or OS I was using that the program didn't account for. I then opted to just use win33.exe on the command line which yield the results wanted. 

Using regshot, it was discovered that this program was copying itself to the location C:\Documents and Settings\%Username%\Application Data\Java Security Plugin\ javaplugin.exe.

![registry](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/image-006.png)

This program path is then added to the registry as the system startup autorun. 

Running the program also started up an iExplorer.exe process. Using procmon, I was able to zero in on what this process is doing. For the most part it was idle. It wasn't until typing stared in IExplorer, that I saw that it was writing to a file called strokes.log. The keylogger in SecureDLL.dll was only being used when writing in IExplorer. This means that something was possibly injected into the program.

Upon analyzing the strokes.log, there was a lot of junk which was actually encrypted. This form of encryption was very simple algorithm of just re-arrangement of the letters. 

## Connected

Whatever connected was being used is no more so the connection was faulty. Only requests were made to the IP 151.248.115.107 but clearly no response were made back. Looking through the internet with whois type websites I found that the IP was not online. 

Because the malware was making a request to the IP though, it did confirm that the malware was indeed talking to the IP.

To get around this I used INetSim and a Host-Only Network. Using the 151.248.115 network family I was able to route all traffic from the virtual machine to my computer and analyze all the network traffic.

![ip_config](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/image-007.png)

Upon connection, the first thing this piece of malware did was send a POST request of the following:

![network_send](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/info_stealer_malware/image-08.png)

This is being sent every 6 minutes

Waiting for IDLE, I never captured any network information that indicated that the captured key data was being sent over the network but it is always a possibility that peridocially, out of the cope of time I have analyzed this malware that it will.