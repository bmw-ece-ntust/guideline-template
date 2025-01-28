# 2024/04/03 Study Note (Single vs. Multi UE Slice Based Scheduling by Jojo)

###### tags: `2024`


**Goal:**
- [x] [Show code modification (by flowchart) Jojo made from single UE to multi UE](#0-Summary)

 
**References:**
- [2024/04/01 Study Note (OSC O-DU High I Multi UE Scheduling for 1 Slice)](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240401%20Study%20Note%20(OSC%20O-DU%20High%20I%20Multi%20UE%20Scheduling%20for%201%20Slice).md)
- [Development of Multiple UE Scheduling per TTI on OSC DU](https://hackmd.io/@JoJoWei/rkCkRi_Bh#Development-of-Multiple-UE-Scheduling-per-TTI-on-OSC-DU)
- [Dennis' Slice Enabled Scheduler Source Code](https://github.com/bmw-ece-ntust/o-du-l2/blob/slice_enabled_scheduler/src/5gnrsch/sch_slice_based.c)
- [Jojo's Multi UE per TTI Source Code](https://github.com/bmw-ece-ntust/o-du-l2/blob/multi_ue_per_tti/src/5gnrsch/sch_slice_based.c)
- [2024/03/22 Study Note (OSC O-DU High I-release Multi UE Explanation)](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240322%20Study%20Note%20(OSC%20O-DU%20High%20I-release%20Multi%20UE%20Explanation).md)
- [2024/04/15 Study Note (OSC O-DU High I-release FCFS scheduler with RRM Policy Implementation)](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240415%20Study%20Note%20(OSC%20O-DU%20High%20I-release%20FCFS%20scheduler%20with%20RRM%20Policy%20Implementation).md)


**Outline:**
- [2024/04/03 Study Note (Single vs. Multi UE Slice Based Scheduling by Jojo)](#2024-04-03-study-note--single-vs-multi-ue-slice-based-scheduling-by-jojo-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
  * [1. Objective of this Note](#1-objective-of-this-note)
  * [2. Comparing Data Structure Modified from Dennis to Jojo](#2-comparing-data-structure-modified-from-dennis-to-jojo)
    + [2.1. `SchDlSlotInfo`](#21-schdlslotinfo)
    + [2.2. `SchUlSlotInfo`](#22-schulslotinfo)
    + [2.3. `MacUlSlot`](#23-maculslot)
    + [2.4. `UlSchedInfo`](#24-ulschedinfo)
    + [2.5. Conclusion](#25-conclusion)
  * [3. Comparing Data Structure used by Jojo and OSC I-release](#3-comparing-data-structure-used-by-jojo-and-osc-i-release)
    + [3.1. `SchDlSlotInfo`](#31-schdlslotinfo)
    + [3.2. `SchUlSlotInfo`](#32-schulslotinfo)
    + [3.3. `MacUlSlot`](#33-maculslot)
    + [3.4. `UlSchedInfo`](#34-ulschedinfo)
    + [3.5. Conclusion](#35-conclusion)
  * [4. Dennis' Slice Enabled Scheduler Algorithm](#4-dennis-slice-enabled-scheduler-algorithm)
    + [4.1. Additional Data Structure in Dennis' Work](#41-additional-data-structure-in-dennis-work)
    + [4.2. Single UE per TTI Slice Enabled Scheduler Algorithm](#42-single-ue-per-tti-slice-enabled-scheduler-algorithm)
  * [5. Jojo's Multi UE per TTI Algorithm](#5-jojos-multi-ue-per-tti-algorithm)
    + [5.1. Additional Data Structure in Jojo's Work](#51-additional-data-structure-in-jojos-work)
    + [5.2. Multi UE per TTI Slice Enabled Scheduler Algorithm](#52-multi-ue-per-tti-slice-enabled-scheduler-algorithm)
          + [3. Step 3: In RR case, divide number of PRB of Slice equally to each UE](#3-step-3-in-rr-case--divide-number-of-prb-of-slice-equally-to-each-ue)
          + [7. Step 7: If there is PRB left for 1 Slice, then allocate PRB to UE by ueToBeScheduled sequence](#7-step-7-if-there-is-prb-left-for-1-slice--then-allocate-prb-to-ue-by-uetobescheduled-sequence)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 0. Summary

![image](https://hackmd.io/_uploads/ByzO3WHWA.png)

- Comparison Table:

| Dennis                                                                                                                                 | Jojo |
| -------------------------------------------------------------------------------------------------------------------------------------- | ---- |
| Step 1: Check the ID of first UE in ueToBeScheduled List                                                                               | Step 1: Get ueToBeScheduled list     |
| Step 2: Get the minimum number of PRB for Slice 1. Since Single UE per TTI, all number of PRB for Slice 1 will all be allocated to UE1 | Step 2: Get the minimum number of PRB for Slice 1     |
|                                                                                                                                        | [Step 3: In RR case, divide number of PRB of Slice equally to each UE](#3-Step-3-In-RR-case-divide-number-of-PRB-of-Slice-equally-to-each-UE)      |
| Step 3: In RR case, divide number of PRB of UE equally to each LC                                                                      | Step 4: In RR case, divide number of PRB of UE equally to each LC     |
| Step 4: If there is PRB left for 1 UE, then allocate PRB to LC by priority level                                                       | Step 5: If there is PRB left for 1 UE, then allocate PRB to LC by priority level     |
|                                                                                                                                        | Step 6: Go to next UE and repeat 4,5     |
|                                                                                                                                        | [Step 7: If PRB left for 1 Slice, allocate PRB to UE by ueToBeSchedule Seq](#7-Step-7-If-there-is-PRB-left-for-1-Slice-then-allocate-PRB-to-UE-by-ueToBeScheduled-sequence)    |
| Step 5: Go to next slice and repeat 2,3,4                                                                                              | Step 8: Go to next slice and repeat 2,3,4,5,6,7      |
| Step 6: If there is remaining PRB, allocate PRB for default OSC FCFS                                                                   | Step 9: If there is remaining PRB, allocate PRB for default OSC FCFS      |
| Step 7: If there is remaining PRB, do PRB allocation based on Slice Sequence                                                                                                                                       | Step 10: If there is remaining PRB, do PRB allocation based on Slice Sequence     |

- In which functions are these steps implemented? See the figure below

![image](https://hackmd.io/_uploads/HkTgrJ8bA.png)



## 1. Objective of this Note

1. We want to find out how to enable MU per TTI in O-DU
2. To achieve 1, first we compare the data structure
    - Compare the data structure of Dennis Code (Slice Enabled Scheduler, Single UE per TTI) and Jojo's Code (Slice Enabled Scheduler, Multi UE per TTI)
    - Then, we evaluate if Jojo's data structure is similar to OSC I release
3. Second, we compare Slice Enabled Scheduling Algorithm
    - Compare the Scheduler Algorithm of Dennis Code (Slice Enabled Scheduler, Single UE per TTI) and Jojo's Code (Slice Enabled Scheduler, Multi UE per TTI)

- Important thing to remember is that Jojo's Multi UE per TTI is an upgrade of Dennis' Slice Enabled Scheduler
![image](https://hackmd.io/_uploads/SygIGIgyA.png)


## 2. Comparing Data Structure Modified from Dennis to Jojo

- Why do we need to compare the data structure?
    - Because the data structure we compare below is used to store the information of the slot. We can see which are the UEs schedule in a particular slot

### 2.1. `SchDlSlotInfo`

- Brief: Scheduler Allocation for DL per cell

| Elem. No. | Elem. Type   | Desc.                                             | Dennis                    | Jojo                      |
| --------- | ------------ | ------------------------------------------------- | ------------------------- | ------------------------- |
| 1         | SchPrbAlloc  | PRB allocated/available in this slot              | prbAlloc                  | prbAlloc                  |
| 2         | bool         | Flag to determine if SSB is present in this slot  | ssbPres                   | ssbPres                   |
| 3         | uint8_t      | Max SSB index                                     | ssbIdxSupported           | ssbIdxSupported           |
| 4         | SsbInfo      | SSB Info                                          | ssbInfo[MAX_SSB_IDX]      | ssbInfo[MAX_SSB_IDX]      |
| 5         | bool         | Flag to determine if SIB1 is present in this slot | sib1Pres                  | sib1Pres                  |
| 6         | uint8_t      | UE for which PDCCH is scheduled in this slot      | **pdcchUe**               | **pdcchUe[MAX_NUM_UE]**   |
| 7         | uint8_t      | UE for which PDSCH is scheduled in this slot      | **pdschUe**               | **pdschUe[MAX_NUM_UE]**   |
| 8         | RarAlloc     | RAR allocation per UE                             | `*rarAlloc[MAX_NUM_UE]`   | `*rarAlloc[MAX_NUM_UE]`   |
| 9         | DciInfo      |                                                   | `*ulGrant`                | `*ulGrant`                |
| 10        | DlMsgSchInfo | Dl msg allocation per UE                          | `*dlMsgAlloc[MAX_NUM_UE]` | `*dlMsgAlloc[MAX_NUM_UE]` |

- Explanation: Jojo added a list for UE for which PDCCH and PDSCH is scheduled in this slot for DL

### 2.2. `SchUlSlotInfo`

- Brief: Scheduler Allocation for UL per cell

| Elem. No. | Elem. Type   | Desc.                              | Dennis              | Jojo                            |
| --------- | ------------ | ---------------------------------- | ------------------- | ------------------------------- |
| 1         | SchPrbAlloc  | PRB allocated/available per symbol | prbAlloc            | prbAlloc                        |
| 2         | uint8_t      | Current PRB for PUSCH allocation   | puschCurrentPrb     | puschCurrentPrb                 |
| 3         | bool         | PUSCH presence field               | puschPres           | puschPres                       |
| 4         | SchPuschInfo | PUSCH info                         | **`*schPuschInfo`** | **`*schPuschInfo[MAX_NUM_UE]`** |
| 5         | bool         | PUCCH presence field               | pucchPres           | pucchPres                       |
| 6         | SchPucchInfo | PUCCH info                         | **`schPucchInfo`** | **`schPucchInfo[MAX_NUM_UE]`** |
| 7         | uint8_t      | Store UE id for which PUCCH is scheduled      | **pucchUe**               | **pucchUe[MAX_NUM_UE]**   |
| 8         | uint8_t      | Store UE id for which PUSCH is scheduled     | **puschUe**               | **puschUe[MAX_NUM_UE]**   |


- Explanation: Jojo added a list for storing UE id for which PUCCH and PUSCH is scheduled in this slot for DL. Each PUCCH and PUSCH information is also converted into a list

### 2.3. `MacUlSlot`

- Brief: 

| Elem. No. | Elem. Type   | Desc.                              | Dennis              | Jojo                            |
| --------- | ------------ | ---------------------------------- | ------------------- | ------------------------------- |
| 1         | [UlSchedInfo](#24-UlSchedInfo)  |  | **ulInfo**            | **ulInfo[MAX_NUM_UE]**                        |

- Explanation: Jojo added a list for UL information

### 2.4. `UlSchedInfo`

- Brief: UL scheduling information

| Elem. No. | Elem. Type     | Desc.                        | Dennis       | Jojo         |
| --------- | -------------- | ---------------------------- | ------------ | ------------ |
| 1         | uint16_t       | Cell Id                      | cellId       | cellId       |
| 2         | uint16_t       | CRNI                         | crnti        | crnti        |
| 3         | SlotTimingInfo | Slot Info: sfn, slot number  |              |              |
| 4         | uint8_t        | Type of info being scheduled | dataType     | dataType     |
| 5         | SchPrachInfo   | PRACH scheduling info        | prachSchInfo | prachSchInfo |
| 6         | SchPuschInfo   | PUSCH scheduling info        | schPuschInfo | schPuschInfo |
| 7         | SchPuschUci   | PUSCH UCI        | schPuschUci | schPuschUci |
| 8         | SchPucchInfo   | PUCCH and UCI scheduling info       | schPucchInfo | schPucchInfo |


- Explanation: There might be no changes in this data structure, but in Section 3, we can see the importance of including this data structure on our watchlist

### 2.5. Conclusion

- In Dennis' data structure, 1 Slot can only be occupied by Single UE. That is why the slot data structure used also can only store 1 UE information
- In Jojo's data structure, 1 Slot can be occupied by Multi UE. That is why the slot data structure used should be upgraded to store Multi UE information

## 3. Comparing Data Structure used by Jojo and OSC I-release

- Why do we need to compare the data structure?
    - Because the data structure we compare below is used to store the information of the slot. We can see which are the UEs schedule in a particular slot

### 3.1. `SchDlSlotInfo`

- Brief: Scheduler Allocation for DL per cell

| Elem. No. | Elem. Type   | Desc.                                             | Jojo                      | OSC I-rel                    |
| --------- | ------------ | ------------------------------------------------- | ------------------------- | ------------------------- |
| 1         | SchPrbAlloc  | PRB allocated/available in this slot              | prbAlloc                  | prbAlloc                  |
| 2         | bool         | Flag to determine if SSB is present in this slot  | ssbPres                   | ssbPres                   |
| 3         | uint8_t      | Max SSB index                                     | ssbIdxSupported           | ssbIdxSupported           |
| 4         | SsbInfo      | SSB Info                                          | ssbInfo[MAX_SSB_IDX]      | ssbInfo[MAX_SSB_IDX]      |
| 5         | bool         | Flag to determine if SIB1 is present in this slot | sib1Pres                  | sib1Pres                  |
| 6         | uint8_t      | UE for which PDCCH is scheduled in this slot      | **pdcchUe[MAX_NUM_UE]**   | **pdcchUe**               |
| 7         | uint8_t      | UE for which PDSCH is scheduled in this slot      | **pdschUe[MAX_NUM_UE]**   | **(deleted)**               |
| 8         | RarAlloc     | RAR allocation per UE                             | `*rarAlloc[MAX_NUM_UE]`   | `*rarAlloc[MAX_NUM_UE]`   |
| 9         | DciInfo      |                                                   | `*ulGrant`                | `*ulGrant`                |
| 10        | DlMsgSchInfo | Dl msg allocation per UE                          | `*dlMsgAlloc[MAX_NUM_UE]` | `*dlMsgAlloc[MAX_NUM_UE]` |

- Explanation:
    - OSC deleted pdschUe -> Why?
        - pdschUe is the identifier of each UE that is scheduled. Since 1 TTI can have MU, this identifier is deleted.
    - OSC still use pdcchUe -> Why?
        - For RaReq, Msg4Req, and Msg3Retx; OSC only allow 1 UE for PDCCH scheduling and will not allow PDSCH scheduling for another UE in the same slot
    

### 3.2. `SchUlSlotInfo`

- Brief: Scheduler Allocation for UL per cell

| Elem. No. | Elem. Type   | Desc.                                    | Jojo                            | OSC I-rel              |
| --------- | ------------ | ---------------------------------------- | ------------------------------- | ------------------- |
| 1         | SchPrbAlloc  | PRB allocated/available per symbol       | prbAlloc                        | prbAlloc            |
| 2         | uint8_t      | Current PRB for PUSCH allocation         | **puschCurrentPrb**                 | **(deleted)**     |
| 3         | bool         | PUSCH presence field                     | puschPres                       | puschPres           |
| 4         | SchPuschInfo | PUSCH info                               | **`*schPuschInfo[MAX_NUM_UE]`** | **`*schPuschInfo[MAX_NUM_UE]`** |
| 5         | bool         | PUCCH presence field                     | pucchPres                       | pucchPres           |
| 6         | SchPucchInfo | PUCCH info                               | **`schPucchInfo[MAX_NUM_UE]`**  | **`schPucchInfo[MAX_NUM_UE]`**  |
| 7         | uint8_t      | Store UE id for which PUCCH is scheduled | **pucchUe[MAX_NUM_UE]**         | **(deleted)**         |
| 8         | uint8_t      | Store UE id for which PUSCH is scheduled | **puschUe[MAX_NUM_UE]**         | **(deleted)**         |


- Explanation:
    - OSC and Jojo use the same list for PUSCH and PUCCH Info
    - OSC deleted puschCurrentPrb -> Why?
        - This element usage is unclear. Only usage in Init with macro PUSH_START_RB
    - OSC deleted pucchUe and puschUe -> Why?
        - pucchUe and puschUe is the identifier of each UE that is scheduled. Since 1 TTI can have MU, this identifier is deleted. Then, on each of `SchPuschInfo` and `SchPucchInfo` data type, crnti element is added. So when ueId is needed, Scheduler will get it from CRNTI element in `SchPuschInfo` and `SchPucchInfo`

### 3.3. `MacUlSlot`

- Brief: 

| Elem. No. | Elem. Type  | Desc. | Jojo                   | OSC I-rel     |
| --------- | ----------- | ----- | ---------------------- | ---------- |
| 1         | [UlSchedInfo](#34-UlSchedInfo) |       | **ulInfo[MAX_NUM_UE]** | **ulSchInfo** |

- Explanation: OSC actually extend this to MU, just like Jojo. See [UlSchedInfo](#34-UlSchedInfo) for details.

### 3.4. `UlSchedInfo`

- Brief: UL scheduling information

| Elem. No. | Elem. Type     | Desc.                         | Jojo         | OSC I-rel       |
| --------- | -------------- | ----------------------------- | ------------ | ------------ |
| 1         | uint16_t       | Cell Id                       | cellId       | cellId       |
| 2         | uint16_t       | CRNI                          | **crnti**        | **(deleted)**        |
| 3         | SlotTimingInfo | Slot Info: sfn, slot number   |              |              |
| 4         | uint8_t        | Type of info being scheduled  | dataType     | dataType     |
| 5         | SchPrachInfo   | PRACH scheduling info         | prachSchInfo | prachSchInfo |
| 6         | SchPuschInfo   | PUSCH scheduling info         | **schPuschInfo** | **schPuschInfo[MAX_NUM_UE]** |
| 7         | SchPuschUci    | PUSCH UCI                     | **schPuschUci**  | **schPuschUci[MAX_NUM_UE]**  |
| 8         | SchPucchInfo   | PUCCH and UCI scheduling info | **schPucchInfo** | **schPucchInfo[MAX_NUM_UE]** |


- Explanation:
    - OSC deleted CRNTI -> Why?
        - CRNTI is the identifier of each UE. Since 1 TTI can have MU, this identifier is deleted. Then, on each of `SchPuschInfo`, `SchPuschUci`, and `SchPucchInfo` data type, crnti element is added
    - OSC added list for PUSCH and PUCCH. This modification in data structure is similar to what Jojo did. Just different level.

### 3.5. Conclusion

- OSC I-rel and Jojo have their own implementation to store multi UE information in a particular Slot data structure. However, both is not wrong can both enables multi UE per TTI

## 4. Dennis' Slice Enabled Scheduler Algorithm

### 4.1. Additional Data Structure in Dennis' Work

- Additional to [OSC's default data structure](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240415%20Study%20Note%20(OSC%20O-DU%20High%20I-release%20FCFS%20scheduler%20with%20RRM%20Policy%20Implementation).md#01-OSC-FCFS-Data-Structure), Dennis created some additional data structure for his work

- `SchSliceBasedCellCb`

![image](https://hackmd.io/_uploads/S1T779E-C.png)

- `SchSliceBasedSliceCb`

![image](https://hackmd.io/_uploads/ByGUH5NZA.png)

- 'SchSliceBasedLcInfo'

![image](https://hackmd.io/_uploads/B1Tr8q4b0.png)

- 1 UE's Link List of Logical Channel inside of a Slice

![image](https://hackmd.io/_uploads/rk-F_j4bR.png)

- Link List of Logical Channel is sorted based on priorLevel (**smaller priority level = better/higher priority**)

![image](https://hackmd.io/_uploads/B1kc9s4WA.png)



### 4.2. Single UE per TTI Slice Enabled Scheduler Algorithm

1. Step 1: Check the ID of first UE in ueToBeScheduled List

![image](https://hackmd.io/_uploads/Hyd4lTEb0.png)

2. Step 2: Get the minimum number of PRB for Slice 1. Since Single UE per TTI, all number of PRB for Slice 1 will all be allocated to UE1

![image](https://hackmd.io/_uploads/HkTmx0VWA.png)

3. Step 3: In RR case, divide number of PRB of UE equally to each LC

![image](https://hackmd.io/_uploads/HkdGITE-R.png)

4. Step 4: If there is PRB left for 1 UE, then allocate PRB to LC by priority level

![image](https://hackmd.io/_uploads/HyX0Ia4b0.png)

5. Step 5: Go to next slice and repeat 2,3,4

![image](https://hackmd.io/_uploads/Bks1eRVZ0.png)

6. Step 6: If there is remaining PRB, allocate PRB for default OSC FCFS

![image](https://hackmd.io/_uploads/BJ-q5kS-C.png)

7. Step 7: If there is remaining PRB, do PRB allocation based on Slice Sequence

![image](https://hackmd.io/_uploads/Hk6KzeHZA.png)


## 5. Jojo's Multi UE per TTI Algorithm


### 5.1. Additional Data Structure in Jojo's Work

- Additional to [Dennis's data structure](#41-Additional-Data-Structure-in-Dennis-Work), Jojo created some additional data structure for his work

- `SchSliceBasedSliceCb`

![image](https://hackmd.io/_uploads/B1uAHRE-0.png)



### 5.2. Multi UE per TTI Slice Enabled Scheduler Algorithm

1. Step 1: Get ueToBeScheduled List

![image](https://hackmd.io/_uploads/SyqFIC4ZA.png)

2. Step 2: Get the minimum number of PRB for Slice 1.

![image](https://hackmd.io/_uploads/B1mdDC4ZA.png)

###### 3. Step 3: In RR case, divide number of PRB of Slice equally to each UE

![image](https://hackmd.io/_uploads/ryyCgkBZ0.png)

4. Step 4: In RR case, divide number of PRB of UE equally to each LC

![image](https://hackmd.io/_uploads/HkfVVyrW0.png)

5. Step 5: If there is PRB left for 1 UE, then allocate PRB to LC by priority level

![image](https://hackmd.io/_uploads/HJdfQJHWC.png)

6. Step 6: Go to next UE and repeat 4,5

![image](https://hackmd.io/_uploads/BJJvUyBbR.png)

###### 7. Step 7: If there is PRB left for 1 Slice, then allocate PRB to UE by ueToBeScheduled sequence

![image](https://hackmd.io/_uploads/H1EK81rWA.png)

8. Step 8: Go to next Slice and repeat 2,3,4,5,6,7

![image](https://hackmd.io/_uploads/HyXKFyr-R.png)

9. Step 9: If there is remaining PRB, allocate PRB for default OSC FCFS

![image](https://hackmd.io/_uploads/HkQqskBWR.png)

10. Step 10: If there is remaining PRB, do PRB allocation based on Slice Sequence

![image](https://hackmd.io/_uploads/HkctveB-A.png)

