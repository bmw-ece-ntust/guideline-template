# 2024/04/30 Study Note (OSC O-DU High I-release FCFS Multi UE scheduler Starvation Example Use Case)

###### tags: `2024`


**Goal:**
- [x] [Check OSC O-DU High I-release FCFS Multi UE scheduler Starvation Example Use Case Validity](#0-Summary)

 
**References:**
- [2024/04/15 Study Note (OSC O-DU High I-release FCFS scheduler with RRM Policy Implementation)](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240415%20Study%20Note%20(OSC%20O-DU%20High%20I-release%20FCFS%20scheduler%20with%20RRM%20Policy%20Implementation).md)
- [2024/04/26 Study Note (OSC O-DU High I-release FCFS scheduler Logic for Remaining Data)](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240426%20Study%20Note%20(OSC%20O-DU%20High%20I-release%20FCFS%20scheduler%20Logic%20for%20Remaining%20Data).md)
- [Johnson's Note | Multi-UE test record](https://hackmd.io/@Johnson-72/rJuyEztCT)


**Outline:**
- [2024/04/30 Study Note (OSC O-DU High I-release FCFS Multi UE scheduler Starvation Example Use Case)](#2024-04-30-study-note--osc-o-du-high-i-release-fcfs-multi-ue-scheduler-starvation-example-use-case-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
  * [1. Objective of this Note](#1-objective-of-this-note)
  * [2. OSC I-release Default Configuration](#2-osc-i-release-default-configuration)
  * [3. OSC I-release Experiment Setup](#3-osc-i-release-experiment-setup)
  * [4. OSC I-release Experiment Result Log Analysis](#4-osc-i-release-experiment-result-log-analysis)
    + [4.1. Log](#41-log)
    + [4.2. Log Explanation](#42-log-explanation)
    + [4.3. Explanation Summary](#43-explanation-summary)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>



## 0. Summary

- By running the ODU High in an experiment, we proved that the example use case of OSC Multi UE per slot starvation is valid

- Table of the example usecase in experiment (to see step-by-step explanation, click [here](#42-Log-Explanation)):

| Description       | First in `ueToBeScheduled` |             |             |             |             |
| ----------------- | -------------------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             |                            | 1           | 2           | 3           | 4           |
| UE ID             |                            | 1           | 1           | 2           | 2           |
| LC ID             |                            | 4           | 5           | 4           | 5           |
| SNSSAI            |                            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default |                            | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO      | UE 1                       | 1240        | 1240        | 1240        | 1240        |
| **SFN/SLOT**      | UE 1                       | **457/5**   | **457/5**   | **457/5**   | **457/5**   |
| Allocated BO      | UE 1                       | 496         | 0           | 0           | 0           |
| Remaining BO      | UE 2                       | 749         | 1240        | 1240        | 1240        |
| Requested BO      | UE 2                       | 749         | 1240        | 1240        | 1240        |
| **SFN/SLOT**      | UE 2                       | **457/6**   | **457/6**   | **457/6**   | **457/6**   |
| Allocated BO      | UE 2                       | 0           | 0           | 544         | 0           |
| Remaining BO      | UE 1                       | 749         | 1240        | 701         | 1240        |
| Requested BO      | UE 1                       | 749         | 1240        | 701         | 1240        |
| **SFN/SLOT**      | UE 1                       | **457/7**   | **457/7**   | **457/7**   | **457/7**   |
| Allocated BO      | UE 1                       | 576         | 0           | 0           | 0           |
| Remaining BO      | UE 2                       | 180         | 1240        | 701         | 1240        |
| Requested BO      | UE 2                       | 180         | 1240        | 701         | 1240        |
| **SFN/SLOT**      | UE 2                       | **457/8**   | **457/8**   | **457/8**   | **457/8**   |
| Allocated BO      | UE 2                       | 0           | 0           | 293         | 0           |
| Remaining BO      | UE 1                       | 180         | 1240        | 415         | 1240        |
| ...                  | ...                           | ...            | ...            | ...            | ...            |

- Figure of the example usecase in experiment:

![image](https://hackmd.io/_uploads/rkf-Y8kfC.png)


## 1. Objective of this Note

- Based on Wilfrid's and Bimo's discussion on [2024/04/09](https://hackmd.io/@superwilfrid/rk9If1F56#20240409) about Multi UE Scheduling for 1 Slice, there are some analysis that are needed for OSC's FCFS Multi UE scheduling:
    1. Explain the root cause of weird "OSC RRMPolicy" implementation
    2. Check the validity of the example [use case](https://hackmd.io/@superwilfrid/r1TKoOP10#03-Compare)
    3. Check OSC's FCFS logic (what happens if UE1 have remaining data that was unallocated in first TTI)

- This note is made to answer the 2nd question.


![image](https://hackmd.io/_uploads/ryx-322gC.png)


## 2. OSC I-release Default Configuration

- Each Logical Channel in UE will be categorized as dedicated or default (default = non-dedicated)

![image](https://hackmd.io/_uploads/H1F9y9nx0.png)

![image](https://hackmd.io/_uploads/r1gaychlA.png)

- Based on Dedicated or Default Logical Channel, we can say that OSC implements the network slicing with 2 slices: Dedicated (priority) and Default

![image](https://hackmd.io/_uploads/HJFYlc3xC.png)

![image](https://hackmd.io/_uploads/Sko7953gA.png)

- By default, each UE will have 2 Logical Channel with ID 4 and 5. ID 4 will be in dedicated slice. ID 5 will be in default slice

| Description       |             |             |             |             |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID                  | 1            | 2            | 3            | 4            |
| UE ID             | 1           | 1           | 2           | 2           |
| LC ID             | 4           | 5           | 4           | 5           |
| SNSSAI            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default | Dedicated   | Default     | Dedicated   | Default     |


## 3. OSC I-release Experiment Setup

- To prove that the example [use case](https://hackmd.io/@superwilfrid/r1TKoOP10#03-Compare) of starvation is valid, experiment is done by running OSC O-DU. The test is done by Johnson and the test steps can be seen in this [note](https://hackmd.io/@Johnson-72/rJuyEztCT)

- Important modifications to note on the experiment:
    - RRM Policy, number of LC, etc used on the experiment are mostly default configuration.
    - RACH for multi UE should be enabled
        - Modified File: `~/l2/src/phy_stub/phy_stub_msg_hdl.c`
        - Modified Function: `l1HdlUlTtiReq()`
        ```c
        #if 1 //default is 0 -----> LINE MODIFIED
            /* Send RACH Ind to L2 for second UE */
            if(phyDb.ueDb.ueCb[UE_IDX_1].rachIndSent == false && phyDb.ueDb.ueCb[UE_IDX_0].msgRrcReconfigComp == true)
            {
                phyDb.ueDb.ueCb[UE_IDX_0].isCFRA = false;
                phyDb.ueDb.ueCb[UE_IDX_1].rachIndSent = true;
                l1BuildAndSendRachInd(ulTtiReq->slot, ulTtiReq->sfn, CB_RA_PREAMBLE_IDX);
                phyDb.ueDb.numActvUe++;
            }

            /* Send RACH Ind to L2 for third UE */
            if(phyDb.ueDb.ueCb[UE_IDX_2].rachIndSent == false && phyDb.ueDb.ueCb[UE_IDX_1].msgRrcReconfigComp == true)
            {
                phyDb.ueDb.ueCb[UE_IDX_0].isCFRA = false;
                phyDb.ueDb.ueCb[UE_IDX_2].rachIndSent = true;
                l1BuildAndSendRachInd(ulTtiReq->slot, ulTtiReq->sfn, CB_RA_PREAMBLE_IDX);
                phyDb.ueDb.numActvUe++;
        ```
    - `cu_stub` is modified to generate full load traffic so that full PRB allocation is possible
        - Modified File: `~/l2/src/cu_stub/cu_stub.c`
        - Modified Function: `startDlData()`
        ```c
        - int32_t totalNumOfTestFlow = 200; // before
        + int32_t totalNumOfTestFlow = 1;   // after

        - sleep(1); before
        + // sleep(1); // after
        ```
    - Additional lines of code is added for logging and analysis purposes
        - Modified File: `~/src/5gnrsch/sch_fcfs.c`
        - Modified Function: `schFcfsScheduleSlot()`
        ```c
        printf("\nwilfrid johnson INFO   -->  SCH: retx UE ID = %d scheduled on sfn = %d slot = %d", ueId, slotInd->sfn, slotInd->slot);
        ```

## 4. OSC I-release Experiment Result Log Analysis

### 4.1. Log

- [Full Log](https://drive.google.com/file/d/1Q-ZSEKDJlq1vkRoELRUT5oWQoTlwAKZc/view?usp=sharing)

- Almost Full Log:

```shell=15955
...
INFO   -->  PHY STUB: TX DATA Request at sfn=456 slot=0
INFO   -->  SCH : RACH occassion set for slot 1
===================== DL Throughput Per UE==============================
Number of UEs : 3
UE Id : 1   DL Tpt : 0.00
UE Id : 2   DL Tpt : 0.00
UE Id : 3   DL Tpt : 0.00
==================================================================
INFO   -->  PHY STUB: PRACH PDU
===================== DL Throughput Per UE==============================
Number of UEs : 3
UE Id : 1   DL Tpt : 0.00
UE Id : 2   DL Tpt : 0.00
UE Id : 3   DL Tpt : 0.00
==================================================================
===================== DL Throughput Per UE==============================
Number of UEs : 3
UE Id : 1   DL Tpt : 0.00
UE Id : 2   DL Tpt : 0.00
UE Id : 3   DL Tpt : 0.00
==================================================================
INFO   -->  SCH : RACH occassion set for slot 1
INFO   -->  PHY STUB: PRACH PDU
DEBUG  -->  EGTP : Received DL Message[1]

DEBUG  -->  DU_APP : Processing DL data in duHdlEgtpDlData()
DEBUG  -->  DU_APP : Sending User Data Msg to RLC [TEID, nPDU]:[1, 1]

DEBUG  -->  EGTP : Received DL Message[2]

DEBUG  -->  RLC_DL : Received DL Data
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [1240]
DEBUG  -->  DU_APP : Processing DL data in duHdlEgtpDlData()
DEBUG  -->  DU_APP : Sending User Data Msg to RLC [TEID, nPDU]:[2, 1]

DEBUG  -->  EGTP : Received DL Message[3]

DEBUG  -->  RLC_DL : Received DL Data
DEBUG  -->  DU_APP : Processing DL data in duHdlEgtpDlData()
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
DEBUG  -->  DU_APP : Sending User Data Msg to RLC [TEID, nPDU]:[3, 1]

DEBUG  -->  EGTP : Received DL Message[4]

DEBUG  -->  RLC_DL : Received DL Data
DEBUG  -->  DU_APP : Processing DL data in duHdlEgtpDlData()
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [1240]
DEBUG  -->  DU_APP : Sending User Data Msg to RLC [TEID, nPDU]:[4, 1]

DEBUG  -->  RLC_DL : Received DL Data
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
DEBUG  -->  RLC_DL: SNSSAI(sst:1,sd [2,3, 4]), DL Tpt : 0.00000
DEBUG  -->  RLC_UL: SNSSAI(sst:1,sd [2,3, 4]), UL Tpt : 0.00000
DEBUG  -->  RLC_DL: SNSSAI(sst:5,sd [6,7, 8]), DL Tpt : 0.00000
DEBUG  -->  RLC_UL: SNSSAI(sst:5,sd [6,7, 8]), UL Tpt : 0.00000
===================== DL Throughput Per UE==============================
Number of UEs : 3
DEBUG  -->  RLC: Slice PM send successfully
UE Id : 1   DL Tpt : 0.00
DEBUG  -->  DU APP : Received Slice Metrics
UE Id : 2   DL Tpt : 0.00
UE Id : 3   DL Tpt : 0.00
INFO   -->  DU_APP: SliceId[SST-SD]:1-234, DlTput 0.00000, UlTput:0.00000
==================================================================
INFO   -->  DU_APP: SliceId[SST-SD]:5-678, DlTput 0.00000, UlTput:0.00000
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 457 slot = 5
INFO   --> SCH: SFN:457/Slot:7, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:2, AggLvl:1 and SymbolDuration2 with StartPrb:60, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:2
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
DEBUG  -->  SCH: Default resources exhausted for LC:5
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[747,496,60]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:747,Idx:0, TotalBO Size:496
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[1243,0,0]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:1243,Idx:1, TotalBO Size:496
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:43 is less than reqPrb:59
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 457 slot = 5
INFO   --> SCH: SFN:457/Slot:7, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:3, AggLvl:1 and SymbolDuration2 with StartPrb:63, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:3
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 457 slot 7
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:496, lcId:4, total :496
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:0, lcId:5, total :0
DEBUG  -->  MAC: Received DL data for sfn=457 slot=7 numPdu= 1
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [749]
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 457 slot = 6
INFO   --> SCH: SFN:457/Slot:8, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:4
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
DEBUG  -->  SCH: Default resources exhausted for LC:5
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[699,544,66]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:699,Idx:0, TotalBO Size:544
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[1243,0,0]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:1243,Idx:1, TotalBO Size:544
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:37 is less than reqPrb:64
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 457 slot = 6
INFO   --> SCH: SFN:457/Slot:8, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:1, AggLvl:1 and SymbolDuration2 with StartPrb:57, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:2, AggLvl:1 and SymbolDuration2 with StartPrb:60, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:3, AggLvl:1 and SymbolDuration2 with StartPrb:63, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:5, AggLvl:1 and SymbolDuration2 with StartPrb:69, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:5
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 457 slot 8[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=457 slot=7
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:544, lcId:4, total :1040
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:0, lcId:5, total :0
DEBUG  -->  MAC: Received DL data for sfn=457 slot=8 numPdu= 1
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [701]
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 457 slot = 7
INFO   --> SCH: SFN:457/Slot:9, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:5, AggLvl:1 and SymbolDuration2 with StartPrb:69, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:5
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
DEBUG  -->  SCH: Default resources exhausted for LC:5
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[176,576,69]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:176,Idx:0, TotalBO Size:576
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[1243,0,0]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:1243,Idx:1, TotalBO Size:576
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:34 is less than reqPrb:68
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 457 slot = 7
INFO   --> SCH: SFN:457/Slot:9, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:6, AggLvl:1 and SymbolDuration2 with StartPrb:72, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:6
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 457 slot 9[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=457 slot=8
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:576, lcId:4, total :1616
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:0, lcId:5, total :0
DEBUG  -->  MAC: Received DL data for sfn=457 slot=9 numPdu= 1
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [180]
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 457 slot = 8
INFO   --> SCH: SFN:458/Slot:0, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:4
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
DEBUG  -->  SCH: Default resources exhausted for LC:5
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[411,293,37]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:411,Idx:0, TotalBO Size:293
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[1243,0,0]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:1243,Idx:1, TotalBO Size:293
DEBUG  -->  SCH: LCID5 Deleted successfully
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 457 slot = 8
INFO   --> SCH: SFN:458/Slot:0, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:0, AggLvl:1 and SymbolDuration2 with StartPrb:54, numPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:1, AggLvl:1 and SymbolDuration2 with StartPrb:57, numPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:2, AggLvl:1 and SymbolDuration2 with StartPrb:60, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:2
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 458 slot 0[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=457 slot=9
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:293, lcId:4, total :1909
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:0, lcId:5, total :0
DEBUG  -->  MAC: Received DL data for sfn=458 slot=0 numPdu= 1
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [415]
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 457 slot = 9
INFO   --> SCH: SFN:458/Slot:1, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:1, AggLvl:1 and SymbolDuration2 with StartPrb:57, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:1
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: All LC are allocated [SharedPRB:32]
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[0,183,25]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:0,Idx:0, TotalBO Size:183
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[966,277,32]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:966,Idx:1, TotalBO Size:460
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:46 is less than reqPrb:55
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 457 slot = 9
INFO   --> SCH: SFN:458/Slot:1, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:4
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 458 slot 1
DEBUG  -->  LWR_MAC: MIB PDU 28000600[1;31m
DEBUG  -->  LWR_MAC: MIB sent..[0m[1;34m
DEBUG  -->  LWR_MAC: SIB1 sent...[0m[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: SSB PDU
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=458 slot=0
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:183, lcId:4, total :2092
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:277, lcId:5, total :277
DEBUG  -->  MAC: Received DL data for sfn=458 slot=1 numPdu= 2
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [968]
===================== DL Throughput Per UE==============================
Number of UEs : 3
UE Id : 1   DL Tpt : 2451.20
UE Id : 2   DL Tpt : 1339.20
UE Id : 3   DL Tpt : 0.00
==================================================================
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 458 slot = 0
INFO   --> SCH: SFN:458/Slot:2, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:2, AggLvl:1 and SymbolDuration2 with StartPrb:60, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:2
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: All LC are allocated [SharedPRB:10]
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[0,418,50]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:0,Idx:0, TotalBO Size:418
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[1159,84,10]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:1159,Idx:1, TotalBO Size:502
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:43 is less than reqPrb:60
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 458 slot = 0
INFO   --> SCH: SFN:458/Slot:2, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:6, AggLvl:1 and SymbolDuration2 with StartPrb:72, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:6
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 458 slot 2
INFO   -->  SCH : RACH occassion set for slot 1[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=458 slot=1
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:418, lcId:4, total :2510
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:84, lcId:5, total :361
DEBUG  -->  MAC: Received DL data for sfn=458 slot=2 numPdu= 2
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1160]
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 458 slot = 1
INFO   --> SCH: SFN:458/Slot:3, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:3, AggLvl:1 and SymbolDuration2 with StartPrb:63, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:3
DEBUG  -->  SCH : DL Only Default Slice is scheduled, sharedPRB Count:63
DEBUG  -->  SCH: LC:5 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[459,512,63]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:459,Idx:0, TotalBO Size:512
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:40 is less than reqPrb:61
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 458 slot = 1
INFO   --> SCH: SFN:458/Slot:3, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:4
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 458 slot 3[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: PRACH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=458 slot=2
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:512, lcId:5, total :873
DEBUG  -->  MAC: Received DL data for sfn=458 slot=3 numPdu= 1
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [463]
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 458 slot = 2
INFO   --> SCH: SFN:458/Slot:4, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:4
DEBUG  -->  SCH : DL Only Default Slice is scheduled, sharedPRB Count:66
DEBUG  -->  SCH: LC:5 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[619,544,66]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:619,Idx:0, TotalBO Size:544
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:37 is less than reqPrb:64
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 458 slot = 2
INFO   --> SCH: SFN:458/Slot:4, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:0, AggLvl:1 and SymbolDuration2 with StartPrb:54, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:1, AggLvl:1 and SymbolDuration2 with StartPrb:57, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:2, AggLvl:1 and SymbolDuration2 with StartPrb:60, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:3, AggLvl:1 and SymbolDuration2 with StartPrb:63, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:5, AggLvl:1 and SymbolDuration2 with StartPrb:69, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:5
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 458 slot 4[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: PUCCH PDU
INFO   -->  PHY STUB: Sending UCI Indication to MAC
DEBUG  -->  LWR_MAC: Processing UCI Indication
DEBUG  -->  MAC : Received HARQ UCI Indication

DEBUG  -->  SCH : Received HARQ
DEBUG  -->  MAC : Received SR UCI indication
DEBUG  -->  SCH : Received SR
INFO   -->  PHY STUB: PUCCH PDU
INFO   -->  PHY STUB: Sending UCI Indication to MAC
DEBUG  -->  LWR_MAC: Processing UCI Indication
DEBUG  -->  MAC : Received HARQ UCI Indication

DEBUG  -->  SCH : Received HARQ
DEBUG  -->  MAC : Received SR UCI indication
DEBUG  -->  SCH : Received SR
INFO   -->  PHY STUB: TX DATA Request at sfn=458 slot=3
...
```







### 4.2. Log Explanation
1. From line 15979 to line 16005, each LC received 1240 BO
```shell=15979
DEBUG  -->  EGTP : Received DL Message[1]

DEBUG  -->  DU_APP : Processing DL data in duHdlEgtpDlData()
DEBUG  -->  DU_APP : Sending User Data Msg to RLC [TEID, nPDU]:[1, 1]

DEBUG  -->  EGTP : Received DL Message[2]

DEBUG  -->  RLC_DL : Received DL Data
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [1240]
DEBUG  -->  DU_APP : Processing DL data in duHdlEgtpDlData()
DEBUG  -->  DU_APP : Sending User Data Msg to RLC [TEID, nPDU]:[2, 1]

DEBUG  -->  EGTP : Received DL Message[3]

DEBUG  -->  RLC_DL : Received DL Data
DEBUG  -->  DU_APP : Processing DL data in duHdlEgtpDlData()
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
DEBUG  -->  DU_APP : Sending User Data Msg to RLC [TEID, nPDU]:[3, 1]

DEBUG  -->  EGTP : Received DL Message[4]

DEBUG  -->  RLC_DL : Received DL Data
DEBUG  -->  DU_APP : Processing DL data in duHdlEgtpDlData()
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [1240]
DEBUG  -->  DU_APP : Sending User Data Msg to RLC [TEID, nPDU]:[4, 1]

DEBUG  -->  RLC_DL : Received DL Data
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
```

| Description       |             |             |             |             |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             | 1           | 2           | 3           | 4           |
| UE ID             | 1           | 1           | 2           | 2           |
| LC ID             | 4           | 5           | 4           | 5           |
| SNSSAI            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO                  | 1240            | 1240            | 1240            | 1240            |

2. From line 16021 to line 16047, UE 1 LC 4 received PRB allocation in {SFN = 457, SLOT = 5} and BO is reduced to 749
```shell=16021
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 457 slot = 5
INFO   --> SCH: SFN:457/Slot:7, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:2, AggLvl:1 and SymbolDuration2 with StartPrb:60, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:2
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
DEBUG  -->  SCH: Default resources exhausted for LC:5
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[747,496,60]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:747,Idx:0, TotalBO Size:496
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[1243,0,0]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:1243,Idx:1, TotalBO Size:496
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:43 is less than reqPrb:59
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 457 slot = 5
INFO   --> SCH: SFN:457/Slot:7, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:3, AggLvl:1 and SymbolDuration2 with StartPrb:63, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:3
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 457 slot 7
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:496, lcId:4, total :496
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:0, lcId:5, total :0
DEBUG  -->  MAC: Received DL data for sfn=457 slot=7 numPdu= 1
```


| Description       |             |             |             |             |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             | 1           | 2           | 3           | 4           |
| UE ID             | 1           | 1           | 2           | 2           |
| LC ID             | 4           | 5           | 4           | 5           |
| SNSSAI            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO      | 1240        | 1240        | 1240        | 1240        |
| **SFN/SLOT**      | **457/5**   | **457/5**   | **457/5**   | **457/5**   |
| Allocated BO      | 496         | 0           | 0           | 0           |
| Remaining BO                  | 749            | 1240            | 1240            | 1240            |

3. Since UE 1 LC 4 gets PRB allocation, UE 1 will be deleted from ueToBeScheduled list. Thus, RLC will add UE 1 again into the back of ueToBeScheduled list
```shell=16048
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [749]
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
```

| Description       |             |             |             |             |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             | 1           | 2           | 3           | 4           |
| UE ID             | 1           | 1           | 2           | 2           |
| LC ID             | 4           | 5           | 4           | 5           |
| SNSSAI            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO                  | 749            | 1240            | 1240            | 1240            |

4. UE 2 will be the first in ueToBeScheduled list. From line 16050 to line 16097, UE 2 LC 4 received PRB allocation in {SFN = 457, SLOT = 6} and BO is reduced to 701
```shell=
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 457 slot = 6
INFO   --> SCH: SFN:457/Slot:8, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:4
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
DEBUG  -->  SCH: Default resources exhausted for LC:5
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[699,544,66]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:699,Idx:0, TotalBO Size:544
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[1243,0,0]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:1243,Idx:1, TotalBO Size:544
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:37 is less than reqPrb:64
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 457 slot = 6
INFO   --> SCH: SFN:457/Slot:8, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:1, AggLvl:1 and SymbolDuration2 with StartPrb:57, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:2, AggLvl:1 and SymbolDuration2 with StartPrb:60, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:3, AggLvl:1 and SymbolDuration2 with StartPrb:63, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   --> SCH: In isPrbAvailable, numFreePrb:2 is less than reqPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:5, AggLvl:1 and SymbolDuration2 with StartPrb:69, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:5
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 457 slot 8[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=457 slot=7
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:544, lcId:4, total :1040
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:0, lcId:5, total :0
DEBUG  -->  MAC: Received DL data for sfn=457 slot=8 numPdu= 1
```

| Description       |             |             |             |             |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             | 1           | 2           | 3           | 4           |
| UE ID             | 1           | 1           | 2           | 2           |
| LC ID             | 4           | 5           | 4           | 5           |
| SNSSAI            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO      | 749        | 1240        | 1240        | 1240        |
| **SFN/SLOT**      | **457/6**   | **457/6**   | **457/6**   | **457/6**   |
| Allocated BO      | 0         | 0           | 544           | 0           |
| Remaining BO                  | 749            | 1240            | 701            | 1240            |

5. Since UE 2 LC 4 gets PRB allocation, UE 2 will be deleted from ueToBeScheduled list. Thus, RLC will add UE 2 again into the back of ueToBeScheduled list

```shell=16098
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [701]
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
```

| Description       |             |             |             |             |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             | 1           | 2           | 3           | 4           |
| UE ID             | 1           | 1           | 2           | 2           |
| LC ID             | 4           | 5           | 4           | 5           |
| SNSSAI            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO      | 749         | 1240        | 701         | 1240        |

6. UE 1 will be the first in ueToBeScheduled list. From line 16100 to line 16131, UE 1 LC 4 received PRB allocation in {SFN = 457, SLOT = 7} and BO is reduced to 180

```shell=16100
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 457 slot = 7
INFO   --> SCH: SFN:457/Slot:9, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:5, AggLvl:1 and SymbolDuration2 with StartPrb:69, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:5
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
DEBUG  -->  SCH: Default resources exhausted for LC:5
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[176,576,69]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:176,Idx:0, TotalBO Size:576
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[1243,0,0]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:1243,Idx:1, TotalBO Size:576
DEBUG  -->  SCH: LCID5 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:34 is less than reqPrb:68
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 457 slot = 7
INFO   --> SCH: SFN:457/Slot:9, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:6, AggLvl:1 and SymbolDuration2 with StartPrb:72, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:6
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 457 slot 9[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=457 slot=8
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:576, lcId:4, total :1616
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:0, lcId:5, total :0
DEBUG  -->  MAC: Received DL data for sfn=457 slot=9 numPdu= 1
```

| Description       |             |             |             |             |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             | 1           | 2           | 3           | 4           |
| UE ID             | 1           | 1           | 2           | 2           |
| LC ID             | 4           | 5           | 4           | 5           |
| SNSSAI            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO      | 749        | 1240        | 701        | 1240        |
| **SFN/SLOT**      | **457/7**   | **457/7**   | **457/7**   | **457/7**   |
| Allocated BO      | 576         | 0           | 0           | 0           |
| Remaining BO                  | 180            | 1240            | 701            | 1240            |

7. Since UE 1 LC 4 gets PRB allocation, UE 1 will be deleted from ueToBeScheduled list. Thus, RLC will add UE 1 again into the back of ueToBeScheduled list

```shell=16132
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [180]
DEBUG  -->  SCH : Received RLC BO Status indication LCId [5] BO [1240]
```

| Description       |             |             |             |             |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             | 1           | 2           | 3           | 4           |
| UE ID             | 1           | 1           | 2           | 2           |
| LC ID             | 4           | 5           | 4           | 5           |
| SNSSAI            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO      | 180         | 1240        | 701         | 1240        |

8. UE 2 will be the first in ueToBeScheduled list. From line 16134 to line 16170, UE 2 LC 4 received PRB allocation in {SFN = 457, SLOT = 8} and BO is reduced to 415

```shell=16134
wilfrid johnson INFO   -->  SCH: retx UE ID = 2 scheduled on sfn = 457 slot = 8
INFO   --> SCH: SFN:458/Slot:0, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:4, AggLvl:1 and SymbolDuration2 with StartPrb:66, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:4
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
DEBUG  -->  SCH: Default resources exhausted for LC:5
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[411,293,37]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:411,Idx:0, TotalBO Size:293
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   -->  SCH : LcID:5, [reqBO, allocBO, allocPRB]:[1243,0,0]
INFO   -->  SCH: Added in MAC BO report: LCID:5,reqBO:1243,Idx:1, TotalBO Size:293
DEBUG  -->  SCH: LCID5 Deleted successfully
wilfrid johnson INFO   -->  SCH: retx UE ID = 1 scheduled on sfn = 457 slot = 8
INFO   --> SCH: SFN:458/Slot:0, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:0, AggLvl:1 and SymbolDuration2 with StartPrb:54, numPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:1, AggLvl:1 and SymbolDuration2 with StartPrb:57, numPrb:3
ERROR  -->  SCH: Requested DL PRB unavailable
ERROR -->  SCH: PRBs can't be allocated as they are unavailable
DEBUG  -->  SCH: RBG found for cceIndex:2, AggLvl:1 and SymbolDuration2 with StartPrb:60, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:2
ERROR  -->  SCH: Requested UL PRB unavailable
DEBUG  -->  MAC: Send scheduled result report for sfn 458 slot 0[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=457 slot=9
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:293, lcId:4, total :1909
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:0, lcId:5, total :0
DEBUG  -->  MAC: Received DL data for sfn=458 slot=0 numPdu= 1
```

| Description       |             |             |             |             |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             | 1           | 2           | 3           | 4           |
| UE ID             | 1           | 1           | 2           | 2           |
| LC ID             | 4           | 5           | 4           | 5           |
| SNSSAI            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO      | 180        | 1240        | 701        | 1240        |
| **SFN/SLOT**      | **457/8**   | **457/8**   | **457/8**   | **457/8**   |
| Allocated BO      | 0         | 0           | 293           | 0           |
| Remaining BO                  | 180            | 1240            | 415            | 1240            |

9. The process will repeat until UE 1 LC 4 requested BO = 0. Then UE 2 LC 4 requested BO = 0. Then UE 1 LC 5 requested BO = 0. Then UE 2 LC 5 requested BO = 0.

### 4.3. Explanation Summary

| Description       | First in `ueToBeScheduled` |             |             |             |             |
| ----------------- | -------------------------- | ----------- | ----------- | ----------- | ----------- |
| TE ID             |                            | 1           | 2           | 3           | 4           |
| UE ID             |                            | 1           | 1           | 2           | 2           |
| LC ID             |                            | 4           | 5           | 4           | 5           |
| SNSSAI            |                            | {1,{2,3,4}} | {5,{6,7,8}} | {1,{2,3,4}} | {5,{6,7,8}} |
| Dedicated/Default |                            | Dedicated   | Default     | Dedicated   | Default     |
| Requested BO      | UE 1                       | 1240        | 1240        | 1240        | 1240        |
| **SFN/SLOT**      | UE 1                       | **457/5**   | **457/5**   | **457/5**   | **457/5**   |
| Allocated BO      | UE 1                       | 496         | 0           | 0           | 0           |
| Remaining BO      | UE 2                       | 749         | 1240        | 1240        | 1240        |
| Requested BO      | UE 2                       | 749         | 1240        | 1240        | 1240        |
| **SFN/SLOT**      | UE 2                       | **457/6**   | **457/6**   | **457/6**   | **457/6**   |
| Allocated BO      | UE 2                       | 0           | 0           | 544         | 0           |
| Remaining BO      | UE 1                       | 749         | 1240        | 701         | 1240        |
| Requested BO      | UE 1                       | 749         | 1240        | 701         | 1240        |
| **SFN/SLOT**      | UE 1                       | **457/7**   | **457/7**   | **457/7**   | **457/7**   |
| Allocated BO      | UE 1                       | 576         | 0           | 0           | 0           |
| Remaining BO      | UE 2                       | 180         | 1240        | 701         | 1240        |
| Requested BO      | UE 2                       | 180         | 1240        | 701         | 1240        |
| **SFN/SLOT**      | UE 2                       | **457/8**   | **457/8**   | **457/8**   | **457/8**   |
| Allocated BO      | UE 2                       | 0           | 0           | 293         | 0           |
| Remaining BO      | UE 1                       | 180         | 1240        | 415         | 1240        |
| ...                  | ...                           | ...            | ...            | ...            | ...            |
