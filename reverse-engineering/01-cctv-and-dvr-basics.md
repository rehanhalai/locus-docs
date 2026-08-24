# 1. CCTV & DVR Basics (Why is it so hard?)

## How a Normal Computer Works
Think of a normal computer's hard drive like a **Library**.
It has a **File System** (like NTFS on Windows or APFS on Mac), which acts as the **Index / Table of Contents**. 
When you save a file (like `vacation.mp4`), the computer writes the file to an empty shelf, and then updates the Index: *"vacation.mp4 is located on Shelf 5, Row 2"*.

## How a Surveillance DVR Works
A DVR (Digital Video Recorder) doesn't care about a neat library. It cares about **speed and continuous recording**. 
It records 4, 8, or 16 cameras simultaneously, 24/7.
Because it needs to write massive amounts of data instantly, many DVR manufacturers (like Dahua or Hikvision) **throw away the standard Library Index**.
Instead, they use a **Proprietary File System** (a secret, custom-made structure). 

They write video data sequentially, one block after another, directly to the raw disk.

### Circular Recording
A DVR drive eventually gets full. When it does, it doesn't ask you to delete things. It just loops back to the beginning and **overwrites the oldest footage**. This is called circular recording.

## The Problem for Police
When police pull a hard drive out of a criminal's DVR and plug it into a normal Windows or Mac computer, the computer looks for the "Index". 
But it can't read the DVR's secret custom Index. So Windows says: *"This drive is unreadable/corrupted. Do you want to format it?"*

That's why standard software fails, and why this SIH problem exists! We need to build software that can read these secret, custom DVR formats without destroying the evidence.
