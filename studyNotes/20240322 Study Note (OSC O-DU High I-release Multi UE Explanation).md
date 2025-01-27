# 2024/03/22 Study Note (OSC O-DU High I-release Multi UE Explanation)

###### tags: `2024`


**Goal:**
- [x] [Find the source code delta between OSC Non Multi UE [H] and OSC Multi UE [I]](#2-H-release-vs-I-release-Schedule-Slot-Differences)
- [ ] Find out how this difference relates to Jojo's Multi UE scheduling


**References:**
- [Multi UE per slot scheduling DL](https://jira.o-ran-sc.org/browse/ODUHIGH-517)
- [Multi UE | Resource Allocation|PDSCH allocation](https://jira.o-ran-sc.org/browse/ODUHIGH-540)
- [Multi UE per slot scheduling UL](https://jira.o-ran-sc.org/browse/ODUHIGH-556)
- [UL resource allocation and final UL scheduling MUlti-UE](https://jira.o-ran-sc.org/browse/ODUHIGH-570)
- [2024/03/18 Study Note (Compare Jojo's Multi UE Scheduling per TTI to OSC I Release)](https://hackmd.io/@superwilfrid/S1rWv5aTa)


**Tabel of contents:**
- [2024/03/22 Study Note (OSC O-DU High I-release Multi UE Explanation)](#2024-03-22-study-note--osc-o-du-high-i-release-multi-ue-explanation-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary of Jojo vs OSC I release Multi UE per TTI](#0-summary-of-jojo-vs-osc-i-release-multi-ue-per-tti)
  * [1. Notes for O-RAN JIRA for OSC Multi UE](#1-notes-for-o-ran-jira-for-osc-multi-ue)
    + [1.1. ODUHIGH-540](#11-oduhigh-540)
    + [1.2. ODUHIGH-570](#12-oduhigh-570)
  * [2. H release vs I release Schedule Slot Differences](#2-h-release-vs-i-release-schedule-slot-differences)
  * [3. Jojo's vs I release Schedule Slot Differences](#3-jojo-s-vs-i-release-schedule-slot-differences)
    + [3.1. Main Loop for Multi UE](#31-main-loop-for-multi-ue)
    + [3.2. Scheduling for Multi UE](#32-scheduling-for-multi-ue)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 0. Summary of Jojo vs OSC I release Multi UE per TTI

1. the basis of OSC multi UE per TTI is first come first served.
2. the basis of Jojo's multi UE per TTI is Dennis' slice based scheduling

![image](https://hackmd.io/_uploads/SygIGIgyA.png)

- So, Jojo's multi UE per TTI feature is like an upgrade of Dennis single UE per TTI slice based scheduling

![image](https://hackmd.io/_uploads/HyWoGIxJC.png)



## 1. Notes for O-RAN JIRA for OSC Multi UE

### 1.1. ODUHIGH-540

- Multi UE | Resource Allocation|PDSCH allocation:
    1. Old Allocation of PDSCH will continue but loop for multiple UE will be added
    2. Make multiple DCI per slot
    1. Collate Multiple DCI in a slot
    2. At Sch, make the allocation loop to go through multiple ueList (frm Candidate selection)

- Multi UE | Multiple DCI:
    1. Selection of candidate(i.e. pdcch resource allocation) is done way befoere actuall filling of allocation information in DLMsgAlloc structure which will be sent to MAC. In current code, in schDlMsgAlloc(), the values filled are very random and not using USS(ue specific search speace) and coreset and also correct allocated CCEINDEX and AGG LEVEL. Support for this is added here
    2. Since we have enables the possibility of multiple DCI since multiple pdcch can be allocated but again while sending the dci to lwr_mac the DCI is limited to 1. Fixed this limitation as well

- Multiple PDSCH allocation


### 1.2. ODUHIGH-570

- Gerrit: https://gerrit.o-ran-sc.org/r/c/o-du/l2/+/12580

- In LWR_MAC:
    - UL TTI REQ filling, before lwr_mac only filled the data for one UE . Since for Multi-UE case, Multiple pucch and pusch pdu has to be concatenated into one ul tti req thus the handling for the same is introduced.
 
- SCH.h:
    - In SCH database of Ul Slot Info, Introduced the array for schPuschInfo and schPucchInfo. Removed the flag called 'puschUe' and 'pucchUe', as these were restricting the introduction of multiple UE's ul schedling in a slot.
 
- SCH_COMMON.c
    - In Ul resource Alloc function , (schUlResAlloc)
    - DRX related BUG: When theres a PUSCH Alloc present in SchUlSchSlotInfo then it is checked whether that particular UE is in Sleep State (in terms of DRX mode) and if so then that schInfo is not sent to MAC. Here problem is that since UE has been removed from pendingUeList in SCH thus this UE's UL sch info has been lost.
- Solution: To check the UE's DRX state before scheduling is performed(in func SchScheduleSlot). If the UE is in sleep state then UE will be moved back to pendingUeList so that when UE becomes Active then the UL scheduling algorithm can hit.
    - Introduced Multiple UE's puschInfo and pucchInfo handling.
    - Before sending to MAC, check the dataType, there can be case when theres no actual UL sch happening and we were sending empty UL Sch Info to MAC which is waste of resources.
 
- COMMON:
    - Wherever PucchUE flag is checked before K1 allocation, it has been replaced by checking of presence of valid crnti in that particular ueIdx in "schUlSlotInfo->schPucchInfo". If the crnti is 0, it means that this UE is not scheduled in this slot then k1 can be successfully allocation and if crnti has valid value then UE is already scheduled for PUCCH in this slot thus next k1 is checked.
    - Similarly wherever puschUe flag was checked before k2 allocation,  it ahs been replaced by checking of presence of valid crnti in schPuschInfo in that slot.
 

- MAC_SCH_INTERFACE.H
    - Moved CRNTI from common placement i.e. in UlSchedInfo and made it unique for each UL PDU i.e. PUCCH, PUSCH and UCI and also replaced these PDU to be supported uniquely for each UE.


## 2. H release vs I release Schedule Slot Differences

![image](https://hackmd.io/_uploads/ryUHjHlkC.png)

## 3. Jojo's vs I release Schedule Slot Differences

### 3.1. Main Loop for Multi UE

- **Important**: OSC use `schFcfsScheduleSlot` (first come first served) while Jojo use `schSliceBasedScheduleSlot` (Dennis' slice based) 

![image](https://hackmd.io/_uploads/S1LaiHek0.png)

### 3.2. Scheduling for Multi UE

![image](https://hackmd.io/_uploads/Sy1UAHeyC.png)

![image](https://hackmd.io/_uploads/SJP_-UgyC.png)
