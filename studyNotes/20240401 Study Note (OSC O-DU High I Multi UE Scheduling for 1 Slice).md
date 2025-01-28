# 2024/04/01 Study Note (OSC O-DU High I Multi UE Scheduling for 1 Slice)

###### tags: `2024`


**Goal:**
- [x] [Compare ODU OSC I-rel (fcfs) and Jojo’s (slice based) scheduler for Multi UE per TTI) for 1 Slice scheduling](#03-Compare)

**References:**
- [2024/03/22 Study Note (OSC O-DU High I-release Multi UE Explanation)](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240322%20Study%20Note%20(OSC%20O-DU%20High%20I-release%20Multi%20UE%20Explanation).md)

**Table of Contents:**
- [2024/04/01 Study Note (OSC O-DU High I Multi UE Scheduling for 1 Slice)](#2024-04-01-study-note--osc-o-du-high-i-multi-ue-scheduling-for-1-slice-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
    + [0.1. OSC I-release (First Come First Served)](#01-osc-i-release-first-come-first-served)
    + [0.2. Jojo (Round Robin)](#02-jojo-round-robin)
    + [0.3. Compare](#03-compare)
  * [1. OSC I-release](#1-osc-i-release)
    + [1.1. First Come First Served](#11-first-come-first-served)
    + [1.2. Overall Loop](#12-overall-loop)
    + [1.3. Loop for UE](#13-loop-for-ue)
    + [1.4. Loop for Logical Channel](#14-loop-for-logical-channel)
    + [1.5. Logical Channel Dedicated and Default List](#15-logical-channel-dedicated-and-default-list)
  * [2. Jojo](#2-jojo)
    + [2.1. Round Robin](#21-round-robin)
    + [2.2. Overall Loop](#22-overall-loop)
    + [2.3. Loop for Slice](#23-loop-for-slice)
    + [2.4. Loop for UE](#24-loop-for-ue)
    + [2.5. Loop for Logical Channel](#25-loop-for-logical-channel)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 0. Summary

### 0.1. OSC I-release (First Come First Served)

- OSC use first come first served inside the slice. So, it will schedule all logical channel for UE1 first. After that, it will schedule next UE LC.
- Inside each UE, logical channel is categorized into dedicated or default. Dedicated logical channel will be prioritized.

![image](https://hackmd.io/_uploads/B1RGzf51A.png)

### 0.2. Jojo (Round Robin)

- For Jojo, the scheduler in one slice can use different options.
- Scheduling Algorithm Options:
    - Round Robin
    - Weight Fair Queue
- For Round Robin Algorithm, one slice PRB will be divided equally among each UE
- And for 1 UE, PRB will be divided equally among each logical channel

![image](https://hackmd.io/_uploads/Bkq1LG510.png)

### 0.3. Compare

- To compare between OSC FCFS and Jojo's RR, we can use the figure below.
- For OSC, since it use FCFS, UE2 will not be scheduled
- For Jojo, since it use RR, all UE will be scheduled, but the data will be divided

![image](https://hackmd.io/_uploads/H1OjinnlA.png)


## 1. OSC I-release

### 1.1. First Come First Served

![image](https://hackmd.io/_uploads/B1RGzf51A.png)

### 1.2. Overall Loop

![image](https://hackmd.io/_uploads/S1u_fzqyC.png)


### 1.3. Loop for UE

![image](https://hackmd.io/_uploads/rkpaGG9kR.png)


### 1.4. Loop for Logical Channel

![image](https://hackmd.io/_uploads/S1Px7f51C.png)

### 1.5. Logical Channel Dedicated and Default List

![image](https://hackmd.io/_uploads/ryzmmz9kR.png)

## 2. Jojo

- Scheduling Algorithm Options:
    - Round Robin
    - Weight Fair Queue

### 2.1. Round Robin

![image](https://hackmd.io/_uploads/Bkq1LG510.png)

### 2.2. Overall Loop

![image](https://hackmd.io/_uploads/BkLBNfcJR.png)

### 2.3. Loop for Slice

![image](https://hackmd.io/_uploads/ryYvEzcyC.png)

### 2.4. Loop for UE

![image](https://hackmd.io/_uploads/S1S54z9kC.png)

### 2.5. Loop for Logical Channel

![image](https://hackmd.io/_uploads/S1lR4fqk0.png)




