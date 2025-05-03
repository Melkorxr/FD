# Bank Account Case Challenge

## Scenario

Your company has been contacted by a very wealthy and influential entrepreneur with a simple request: perform data recovery to retrieve a single file. The CEO of the company explains that he had a USB flash drive containing one critical file. However, after handing it to the company accountant—who used a Mac computer—the flash drive suddenly became unreadable. The CEO states that the file contained extremely important bank account information: an 18-digit number that he urgently needs to recover. To him, nothing else matters but that one file. The file is of utmost importance, and the CEO is willing to pay any amount necessary to get it back.

## Operating-System

KALI LINUX

## Tools

1. Autopsy 
2. Ghex
3. Linux DD

## Steps I Do

#### 1. Use Autopsy
First, I created a case named Bank Account in Autopsy and added the image file to it. Upon analysis, we can see that there are four partitions or disks present. We only need the fourth partition, which is formatted in FAT32.

Next, I examined the details of this partition and also chose to extract unallocated files. I navigated to Image Details → FAT Contents, then clicked on EOF. I selected sector number 1, with number of sectors set to 14000, and type set to Unallocated Data, then clicked View.

After that, I changed the display to ASCII, where I observed a "JFIF" string—indicating that the data may contain a JPEG file. I then proceeded to extract the contents for further analysis.

#### 2. Use Ghex
After exporting the content in .raw file format, I opened it in GHex to examine the hex values. I noticed that the JPEG file was extremely fragmented, so I needed to locate and collect all the fragments in order to recover the complete JPEG file.

The key fragments required for reconstruction include the header (FFD8) along with the following markers: FFE0, FFE1, FFE2, two instances of FFBD, FFC0, four instances of FFC4, FFDA, around 60 instances of FF00, and finally the footer FFD9.

Here is the breakdown of the byte sizes I extracted for each:

    FFD8 and FFE0 = 16 bytes

    FFE1 = 101 bytes

    FFE2 = 10,668 bytes

    FFBD = 71 bytes (each, 2 times)

    FFC0 = 21 bytes

    FFDA = 16 bytes

For FFC4, FF00, and FFD9, I extracted all occurrences found in the image.
The most challenging part was collecting all the FF00 fragments, as they were numerous and scattered throughout the file. I used the dd command-line tool to extract each fragment into separate files, then combined all fragments into a single file and saved it with a .jpg extension.

#### 3. Recover JPG
Well, the result is still not perfect, but at least the bank account number is almost visible. This is likely due to some missing fragments that I haven't been able to locate yet. Maybe I’ll be able to find them next time.

According to someone who has completed this challenge, the correct bank account number is 983456 294001 991201. However, I still want to identify exactly which fragments are missing. If you happen to find them, please let me know.

🎯 *This script ensures a smooth and secure MobSF installation. Happy testing!*

© Melkorxr 2025
