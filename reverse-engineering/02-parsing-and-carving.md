# 2. Sector Parsing & Carving (The Core Challenge)

If Windows can't read the DVR's secret "Index" (as we learned in Part 1), how do we get the video out?
We have to ignore the Index completely and read the raw physical disk directly.

## What is a Sector?
A hard drive is divided into millions of tiny physical blocks called **Sectors**. (Usually 512 bytes of data per sector). 
When an investigator copies a 500 GB hard drive for forensics, they make a raw "Bit-Stream Image" (often saved as a `.dd` file). This is a massive digital clone of every single sector, exact 1s and 0s.

## What is Parsing?
**Parsing** means reading the raw data and making sense of the rules. 
If we study a Dahua DVR, we might figure out their secret rule: *"Every video chunk starts with a special 4-letter code, like `DHAV`"*. 
Our Python backend will **parse** (read through) the raw sectors looking for these specific structures.

## What is Carving? (The Jigsaw Puzzle Analogy)
Imagine someone ripped the Table of Contents out of a 10,000-page book. How do you find Chapter 4?
You have to flip through every single page until you see the big bold words **"Chapter 4"**.

**File Carving** is exactly that. 
When a criminal deletes a video on a DVR, the DVR usually just deletes the "Index entry" pointing to the video. The actual video data is still sitting physically on the disk sectors!

Our software will scan every single sector of the 500 GB drive. Whenever it spots the secret starting code (`DHAV` or `H264`), it says, *"Aha! A video starts here!"* It then copies the data from that sector onwards until the video chunk ends. 

**This is how forensic tools attempt to recover deleted CCTV footage.** The software "carves" video fragments out of the raw unallocated space without needing a Table of Contents. Recovery success depends on whether the underlying disk sectors have been physically overwritten by the DVR's circular buffer; overwritten sectors are irrecoverable.
