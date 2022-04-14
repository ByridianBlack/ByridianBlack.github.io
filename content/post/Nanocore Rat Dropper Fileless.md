---
title: " VBS Dropper Script for Nanocore RAT. Fileless Properties. "
date: 2021-09-20T10:22:10+07:00
tags: ["dropper","fileless","nanocore RAT"]
images: [
    https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/nanocore/rat.png,
]
toc: true
---

# Introduction

This piece of malware had some fileless malware properties but because it copied itself to disk it
cannot be categorized as a fileless malware. Rather, this code should be seen as a dropper which
executes a .Net assembly code in memory and reruns itself every time on boot up. There was no
evidence that I could see that shows that the .Net executable was every written to disk. Using a
combination of both PowerShell and the windows .Net compiler, this piece of malware was able
to compile previously encoded information all in memory. The signature was picked up.
Personally, this is the first time I have ever witnessed this, and it shows how much powerful
malware can become when it has only started off with a few lines to begin with.

# Static Analysis

![obfuscated_code](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/nanocore/image-000.png)

Starting off as a VBS script the code was heavily obfuscated. It used a series of split and conversions to transform the data from unreadable to usable data. The code being used here was syntax used for PowerShell, so without Invoking of any unwanted commands I used PowerShell to decode the script. The code was split into multiple parts. The first part was also obfuscated as well with replace functions.

![obfuscated_code](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/nanocore/image-001.png)

This in turn gets transformed into the following:

![obfuscated_code](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/nanocore/image-002.png)

Previously in Figure 1 the following URL was being used: https://transfer.sh/otxKHB/xdswed.txt.
The site for now as of 9/15/2021 is still up and running. Because of this going to the site without the malware execution was the ideal option and the text file was nothing more than a VBS script that was executing PowerShell commands. Figure 3 shows that this segment acts as the dropper, downloading the data as a string and executing it with IEX.

Within this file was another URL: https://transfer.sh/hLJAlo/vbyujg.txt. 

Analyzing the xdsweed.txt file the following was found:

![obfuscated_code](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/nanocore/image-003.png)

Initially obfuscated, this portion of the code shows multiple things. One, a directory called Run is being created in the C:\Users\Public directory. There the registry is being targeted allowing for the newly created directory to be run during startup. This is location where the malware is copying itself into to be run at every startup.


The second part of xdsweed.txt file is downloading the vbyujg.txt file from the URL. Vbyujg.txt is much more dangerous because of what it holds. The following is found:

![obfuscated_code](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/nanocore/image-004.png)

![obfuscated_code](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/nanocore/image-005.png)

 The HH variable is much bigger, but they are both hex encoded strings. 

![obfuscated_code](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/nanocore/image-006.png)

Which appears to be doing the decoding. Using Python I was able to decode this and the result came out to be a PE executable. 

This comes to no surprise as in a line more down in this dropped file I saw the following.

![obfuscated_code](https://raw.githubusercontent.com/ByridianBlack/BlogImages/main/nanocore/image-007.png)


I believe that this this executable in in fact a .Net executable. A .Net executable is built on top of 

the PE format. From here I theorize the following. 

This executable is being written and then stored in the created directories previous found. There it is being started on every bootup. 

I am not yet sure what the executable does but I am certain that it is malicious. Most programs do that act in this way.

From here not much can be learned from static analysis. I must run this executable and determine what is going on. 

These executables are being compiled with the aspnet_compiler.exe function as found in the same downloaded payload. They are then being injected directly into memory without being written to disk.


From here dynamic analysis must be used to verify my findings. There shows no indications of the executable being written to disk but dynamic analysis must be used to confirm these findings.

# Dynamic Analysis

Running the application led to the following.

A copy of the malware was written to the created directory as shown in Figure 4. An executable is also being loaded and run. Using the internet, it was determined that this is in fact the Nanocore RAT. A RAT that first showed up in 2013. Because of the nature of this dropper, getting hold of this sample which seems to only exist in memory is hard and more forensics research on the memory will be done. Further analysis will be done in the future.

# Conclusion

This malware appears to be a vector aimed at the distribution of the Nanocore RAT. Therefore, this can be categorized as both a dropper and spyware. One thing to note is this VBS script can easily be stored inside of a docx file. Also, what should also be noted is how dangerous that this dropper truly is.

Based of the evidence that it has shown, it needed to create a new instance of itself from start up each time because the downloaded .Net executable that it is running only appears to exist in memory. No evidence shows the .Net executable being written to disk. This means that it is harder to detect through conventional means and tools.

A possible method might be to take a snapshot of the RAM memory itself and then to look through that. Again, further analysis will be done.

Lastly, what should be noted is how easily this code can be altered to download a different payload or even to download the same payload but with a different obfuscation algorithm. With, there could be very well different versions of this same malware doing the same thing. That along with the fact that they are using different trusted sources to do terrible things presents another issue. PowerShell, transfer.sh and aspnet_compiler are trusted entities. They cannot simply be blocked. Combating this piece of malware which has fileless malware properties presents a challenge.

### Dropper.vbs
* MD5: f0dba5477d792b0b5e16b79b0b603d7d
* SHA-1: 14bbacec9dcfae539f9a70c2de62f758bbe23767
* SHA-256: 4c6abb682817fd6093d2edaa821ef0c9c9368db4d6be6dce152a45a9782afa27

### vbyujg.txt

* MD5: 957a3ff0a39263ae85017475f598b7da
* SHA-1: ef9ac4dcec16c29986a18ec3c0723fc3e20dd4e9
* SHA-256: 36321adffd7260915b142d9b38b95bad09e6d688adbf37bba17bb4792650b83d

### xdswed.txt
* MD5: e8f38a72c211f12114157777b7e98e51
* SHA-1: d8ea6e6c2d483692fd0f269d4148ac3470f98cb2
* SHA-256: 917f2e4c16ee5c0b96b9fd9b32dd569113745842c2fd0912422f3a7fb7922b53