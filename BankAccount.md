# Bank Account Case Challenge

## Scenario

Your company has been contacted by a very wealthy and influential entrepreneur with a simple request: perform data recovery to retrieve a single file. The CEO of the company explains that he had a USB flash drive containing one critical file. However, after handing it to the company accountant—who used a Mac computer—the flash drive suddenly became unreadable. The CEO states that the file contained extremely important bank account information: an 18-digit number that he urgently needs to recover. To him, nothing else matters but that one file. The file is of utmost importance, and the CEO is willing to pay any amount necessary to get it back.

## Operating-System

![Kali](/img/KaliLinux.jpg)

## Tools

1. Autopsy 
2. Ghex
3. Linux DD

## Steps I Do

#### 1. Use Autopsy
First, I created a case named Bank Account in Autopsy and added the image file to it. Upon analysis, we can see that there are four partitions or disks present. We only need the fourth partition, which is formatted in FAT32.
![PartitionDisk](/img/BankAccount/1-1.png)

Next, I examined the details of this partition and also chose to extract unallocated files. I navigated to Image Details → FAT Contents, then clicked on EOF. I selected sector number 1, with number of sectors set to 14000, and type set to Unallocated Data, then clicked View.
![ImageDetails](/img/BankAccount/1-2.png)

![Contents](/img/BankAccount/1-3.png)
After that, I changed the display to ASCII, where I observed a "JFIF" string—indicating that the data may contain a JPEG file. I then proceeded to extract the contents for further analysis.

#### 2. Use Ghex
After exporting the content in .raw file format, I opened it in GHex to examine the hex values. I noticed that the JPEG file was extremely fragmented, so I needed to locate and collect all the fragments in order to recover the complete JPEG file.
![Ghex](/img/BankAccount/2-1.png)
The key fragments required for reconstruction include the header (FFD8) along with the following markers: FFE0, FFE1, FFE2, two instances of FFBD, FFC0, four instances of FFC4, FFDA, around 60 instances of FF00, and finally the footer FFD9.

Here is the breakdown of the byte sizes I extracted for each:

    FFD8 and FFE0 = 16 bytes

    FFE1 = 101 bytes

    FFE2 = 10,668 bytes

    FFBD = 71 bytes (each, 2 times)

    FFC0 = 21 bytes

    FFDA = 16 bytes

For FFC4, FF00, and FFD9, I extracted all occurrences found in the image.
The most challenging part was collecting all the FF00 fragments, as they were numerous and scattered throughout the file. I used the dd command-line tool to extract each fragment into separate files, then combined all fragments into a single file using Ghex and saved it with a .jpg extension.
```bash
dd if=file.raw bs=1 skip=$((0x000b0a00)) count=16 of=segment_ffd8.bin
```
```bash
dd if=file.raw bs=1 skip=$((0x13a3be)) count=101 of=segment_ffe1.bin
```
```bash
dd if=file.raw bs=1 skip=$((0x000b0a16)) count=10668 of=segment_ffe2.bin
```
```bash
dd if=file.raw bs=1 skip=$((0x21a25)) count=71 of=segment_ffdb1.bin

dd if=file.raw bs=1 skip=$((0x21a6a)) count=71 of=segment_ffdb2.bin
```
```bash
dd if=file.raw bs=1 skip=$((0x21aaf)) count=21 of=segment_ffc0.bin
```
```bash
dd if=file.raw bs=1 skip=$((0x21c72)) count=16 of=segment_ffda.bin
```

#### 3. Recover JPG
Well, the result is still not perfect, but at least the bank account number is almost visible. This is likely due to some missing fragments that I haven't been able to locate yet. Maybe I’ll be able to find them next time.

![Recovered](/img/BankAccount/3-1.jpg)

According to someone who has completed this challenge, the correct bank account number is *__983456 294001 991201__*. However, I still want to identify exactly which fragments are missing. If you happen to find them, please let me know.

🎯 *"FORENSICS IS FUN"*

© Melkorxr 2025
