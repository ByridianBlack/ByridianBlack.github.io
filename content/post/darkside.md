---
title: "Darkside Malware"
date: 2022-02-16T10:22:10+07:00
tags: ["darkside","malware","dark"]
images: [
    https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/darkside/0.png,
]
toc: true
---

Darkside is a ransomware notorious for attacking high profile industrial control systems and facilities. It’s most famous attack was against the Colonial Pipeline line in the United States between May 6 – May 12 of 2021. This attack show cased how dangerous ransomware can be and what an effect these types of attacks can have on the country and world. This is an analysis on Darkside.

## Analysis

Darkside utilizes a lot of dynamic allocation as means to hide its functionality. This is done with the usage of LoadLibrary and GetProcessAddress functions. What makes this so difficult is their usage of encrypted strings which have to go through an decryption process in order to be used first. This added to the complexity and could not be statically resolved. To determine what these strings were dynamic analysis was necessary.

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/darkside/0.png)

The decryption process is done entire through the use of the key created through this function, taking in two smaller 16 length keys it creates a 256 byte decryption key which is utilized through the entire malware to dynamically allocate and resolve strings and API. Because of the constant value of the key, the key is a great host base indicator as long as new versions do not change these.

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/darkside/1.png)

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/darkside/2.png)

The API’s are resolved here, looping through dword values and setting them accordingly. Afterwards the dword values are as followed:

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/darkside/3.png)

What really made Darkside ransomware so interesting was its usage of a UAC bypass to encrypt to begin with. A UAC stand for User Access Control, and bypassing it gives a program more control and privileges. In this case Darkside utilized a COM object to achieve these higher privileges. Note the following.

![image](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/darkside/4.png)


Initially Darkside is execute but almost immediately it executes the COM object CMSTPLUA. This being generated in a very specific way; C:\Windows\system32\DllHost.exe /Processid:{3E5FC7F9-9A51-4367-9063-A120244FBEC7}. Was is important about this COM object is that it is given higher privileges because it is listed under trusted COM by Windows. It then executes a ShellExecute function on Darkside, running it now in higher privileges.

List of Approved COM is listed in the registry: Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\UAC\COMAutoApprovalList 

For now, this is all my analysis as taken from this piece of malware. This malware is interesting because of its ability to escalate its privilege and with relative ease and less noise than other means. Like most security exploits, utilizing the trust places in other resources was the downfall here as Darkside was able to capitalize on this and proceed with the encryption process and steal money accordingly.

