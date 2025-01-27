# 2024/04/15 Study Note (OSC O-DU High I-release FCFS scheduler with RRM Policy Implementation)

###### tags: `2024`


**Goal:**
- [x] [Explain the root cause of weird "OSC RRMPolicy" implementation](#07-Conclusion)
- [x] [Check the validity of the example use case](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240430%20Study%20Note%20(OSC%20O-DU%20High%20I-release%20FCFS%20Multi%20UE%20scheduler%20Starvation%20Example%20Use%20Case).md)
- [x] [Check OSC's FCFS logic (what happens if UE1 have remaining data that was unallocated in first TTI)](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240426%20Study%20Note%20(OSC%20O-DU%20High%20I-release%20FCFS%20scheduler%20Logic%20for%20Remaining%20Data).md)

 
**References:**
- [2024/04/01 Study Note (OSC O-DU High I Multi UE Scheduling for 1 Slice)](https://hackmd.io/@superwilfrid/r1TKoOP10)
- [Dennis' Thesis Oral Presentation](https://docs.google.com/presentation/d/1ci6gSKefXazMuNovP349x5bQSYnTHeWQ/edit)


**Outline:**
- [2024/04/15 Study Note (OSC O-DU High I-release FCFS scheduler with RRM Policy Implementation)](#2024-04-15-study-note--osc-o-du-high-i-release-fcfs-scheduler-with-rrm-policy-implementation-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
    + [0.1. OSC FCFS Data Structure](#01-osc-fcfs-data-structure)
    + [0.2. Categorizing Each Logical Channel into Dedicated or Default](#02-categorizing-each-logical-channel-into-dedicated-or-default)
    + [0.3. Scheduler's Loop for UE](#03-scheduler-s-loop-for-ue)
    + [0.4. Scheduler's Loop for Logical Channel](#04-scheduler-s-loop-for-logical-channel)
    + [0.5. Scheduler's Dividing Logical Channel into Default and Dedicated](#05-scheduler-s-dividing-logical-channel-into-default-and-dedicated)
    + [0.6. Scheduler's PRB Allocation for each logical Channel](#06-scheduler-s-prb-allocation-for-each-logical-channel)
    + [0.7. Conclusion](#07-conclusion)
  * [1. Objective of this Note](#1-objective-of-this-note)
  * [2. Little Background About RRM Policy](#2-little-background-about-rrm-policy)
  * [3. OSC's Data Structure Related to RRM Policy](#3-osc-s-data-structure-related-to-rrm-policy)
    + [3.0. Summary of Important Data Structures](#30-summary-of-important-data-structures)
      - [3.0.1. Slice Data Structure](#301-slice-data-structure)
      - [3.0.2. Cell Data Structure](#302-cell-data-structure)
      - [3.0.3. UE Data Structure](#303-ue-data-structure)
      - [3.0.4. LC Data Structure](#304-lc-data-structure)
      - [3.0.5. LC Link List Data Structure](#305-lc-link-list-data-structure)
      - [3.0.6. UE Link List Data Structure](#306-ue-link-list-data-structure)
    + [3.1. `rrmpolicy.xml`](#31-rrmpolicyxml)
    + [3.2. `odu_config.xml`](#32-odu_configxml)
    + [3.3. `cell_config.xml`](#33-cell_configxml)
    + [3.4. `sch.h`](#34-schh)
    + [3.5. `common_def.h`](#35-common_defh)
    + [3.6. `mac_sch_interface.h`](#36-mac_sch_interfaceh)
    + [3.7. `du_app_mac_inf.h`](#37-du_app_mac_infh)
    + [3.8. `du_cfg.h`](#38-du_cfgh)
    + [3.9. `phy_stub.h`](#39-phy_stubh)
    + [3.10. `mac.h`](#310-mach)
    + [3.11. `sch_fcfs.h`](#311-sch_fcfsh)
  * [4. OSC's Logical Channel Initialization using RRM Policy](#4-oscs-logical-channel-initialization-using-rrm-policy)
    + [4.1. Flow](#41-flow)
    + [4.2. `updateDedLcInfo`](#42-updatededlcinfo)
    + [4.3. `cpyRrmPolicyInDuCfgParams`](#43-cpyrrmpolicyinducfgparams)
    + [4.4. `fillSliceCfgInfo`](#44-fillslicecfginfo)
    + [4.5. `addSliceCfgInSchDb`](#45-addslicecfginschdb)
    + [4.6. `parseRrmPolicyRatio`](#46-parserrmpolicyratio)
    + [4.7. `fillSchUlLcCtxt`](#47-fillschullcctxt)
    + [4.8. `fillSchDlLcCtxt`](#48-fillschdllcctxt)
  * [5. OSC's Logical Channel Scheduling / Resource Allocation](#5-oscs-logical-channel-scheduling--resource-allocation)
    + [5.1. `prbAllocUsingRRMPolicy`](#51-prballocusingrrmpolicy)
    + [5.2. `schFcfsScheduleDlLc`](#52-schfcfsscheduledllc)
    + [5.3. `updateLcListReqPRB`](#53-updatelclistreqprb)
    + [5.4. `schFcfsScheduleSlot`](#54-schfcfsscheduleslot)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 0. Summary

### 0.1. OSC FCFS Data Structure

- Slice RRM Policy
![image](https://hackmd.io/_uploads/rkNwJ5neA.png)

- Cell
![image](https://hackmd.io/_uploads/B1-FycneA.png)

- UE
![image](https://hackmd.io/_uploads/H1F9y9nx0.png)

- Logical Channel
![image](https://hackmd.io/_uploads/r1gaychlA.png)

### 0.2. Categorizing Each Logical Channel into Dedicated or Default

- In function [`updateDedLcInfo`](#42-updateDedLcInfo) (`l2/src/5gnrsch/sch_ue_mgr.c`), each LC will be categorized into Dedicated or Default based on snssai (Single – Network Slice Selection Assistance Information)
![image](https://hackmd.io/_uploads/HJFYlc3xC.png)


### 0.3. Scheduler's Loop for UE

- In scheduler, there is a link list of pending UE node. In each slot indication, scheduler with traverse each of this node one-by-one
![image](https://hackmd.io/_uploads/rJv4QcneA.png)

### 0.4. Scheduler's Loop for Logical Channel

- For each UE, each logical channel will be evaluated for PRB allocation
![image](https://hackmd.io/_uploads/H1MYH9heC.png)

### 0.5. Scheduler's Dividing Logical Channel into Default and Dedicated

- Before PRB allocation of each logical channel, logical channel will be divided into dedicated or default and be put into respective link list
![image](https://hackmd.io/_uploads/SyI4uc2eA.png)

### 0.6. Scheduler's PRB Allocation for each logical Channel

- For each logical channel in a UE, dedicated logical channels will get PRB allocation first before default logical channels
![image](https://hackmd.io/_uploads/Sko7953gA.png)


### 0.7. Conclusion

- Based on **[0.2](#02-Categorizing-Each-Logical-Channel-into-Dedicated-or-Default)** and **[0.5](#05-Schedulers-Dividing-Logical-Channel-into-Default-and-Dedicated)**, OSC FCFS scheduler will divide all logical channels into 2 categories. **Dedicated or default. This categorization can be seen equivalent as 2 different slice.**
- From **[0.6](#06-Schedulers-PRB-Allocation-for-each-logical-Channel)**, **Dedicated slice will be prioritized in each UE**
- But from **[0.3](#03-Schedulers-Loop-for-UE)**, OSC FCFS does not allocate resource for LC in Dedicated slice. **The outer loop is still UE**, not slice. Simple PRB allocation sequence is like this:
    1. First UE's Dedicated logical channels
    2. First UE's Default logical channels
    3. Second UE's Dedicated logical channels
    4. Second UE's Default logical channels
- And from **[`prbAllocUsingRRMPolicy()`](#51-prbAllocUsingRRMPolicy)**, the PRB allocation is sequencial with **no upper bound limit for each LC**. This means first LC could exhaust all PRB, resulting in no PRB available for second LC
- Same goes for **[`schFcfsScheduleSlot()`](#54-schFcfsScheduleSlot)**, the PRB allocation is sequencial with **no upper bound limit for each UE**. This means first UE could exhaust all PRB, resulting in no PRB available for second UE
- This is why, **eventhough OSC FCFS scheduler can have multi UE per TTI, there is a chance that only 1 UE will be scheduled in 1 TTI/slot**

spoiler
![image](https://hackmd.io/_uploads/ryx-322gC.png)


## 1. Objective of this Note

- Based on Wilfrid's and Bimo's discussion on [2024/04/09](https://hackmd.io/@superwilfrid/rk9If1F56#20240409) about Multi UE Scheduling for 1 Slice, there are some analysis that are needed for OSC's FCFS Multi UE scheduling:
    1. Explain the root cause of weird "OSC RRMPolicy" implementation
    2. Check the validity of the example [use case](https://hackmd.io/@superwilfrid/r1TKoOP10#03-Compare)
    3. Check OSC's FCFS logic (what happens if UE1 have remaining data that was unallocated in first TTI)

## 2. Little Background About RRM Policy

- This materials are taken from Dennis' Thesis Oral Presentation

- Network slicing: Slice the network into seperate pieces/slices and each slice can be treated differently from the others

![image](https://hackmd.io/_uploads/Sk_fE_5x0.png)

- RRM Policy: The rule for each slice to follow for resource allocation

![image](https://hackmd.io/_uploads/SyRENdcxR.png)

- More details, check [Dennis' Thesis Oral Presentation](https://docs.google.com/presentation/d/1ci6gSKefXazMuNovP349x5bQSYnTHeWQ/edit) or 3GPP TS 28.541

## 3. OSC's Data Structure Related to RRM Policy

### 3.0. Summary of Important Data Structures

#### 3.0.1. Slice Data Structure

- A slice's RRM Policy is stored using the structure below
- **SchCb[ ]**:
    - SchRrmPolicyOfSlice **sliceCfg**:
        - SchRrmPolicyRatio rrmPolicyRatioInfo:
            - uint8_t **maxRatio**
            - uint8_t **minRatio**
            - uint8_t **dedicatedRatio**

#### 3.0.2. Cell Data Structure

- One cell host multiple UE
- **SchCellCb**:
    - SchUeCb **ueCb[ ]**

#### 3.0.3. UE Data Structure

- One UE has multiple logical channels
- **SchUeCb**:
    - SchUlCb **ulInfo**:
        - SchUlLcCtxt **ulLcCtxt[ ]**
    - SchDlCb **dlInfo**:
        - SchDlLcCtxt **dlLcCtxt[ ]**

#### 3.0.4. LC Data Structure

- Each logical channel is categorized into dedicated or not
- **SchDlLcCtxt**:
    - bool **isDedicated**
    - uint16_t **rsvdDedicatedPRB**
- **SchUlLcCtxt**:
    - bool **isDedicated**
    - uint16_t **rsvdDedicatedPRB**

#### 3.0.5. LC Link List Data Structure

- In FCFS, a data structure is used to store the link list of logical channels
- **SchFcfsLcCb**:
    - CmLListCp **dedLcList** /* dedicated LC Link List */
    - CmLListCp **defLcList** /* default LC Link List */
    - uint16_t **sharedNumPrb**

#### 3.0.6. UE Link List Data Structure

- In FCFS, a data structure is used to store the link list of UEs
- **SchFcfsLcCb**:
    - CmLListCp **dedLcList**
    - CmLListCp **defLcList**
    - uint16_t **sharedNumPrb**



### 3.1. `rrmpolicy.xml`

- File location = `l2/build/config/rrmpolicy.xml`

- Code Snippet (UL RRM Policy):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<RRMPolicyRatio xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-rrmpolicy">
  <id>rrm-1</id>
  <attributes>
    <resourceType>PRB_UL</resourceType>
    <rRMPolicyMemberList>
	<mcc>311</mcc>
	<mnc>48</mnc>
	<sd>000002</sd>
	<sst>1</sst>
    </rRMPolicyMemberList>
    <rRMPolicyMaxRatio>90</rRMPolicyMaxRatio>
    <rRMPolicyMinRatio>40</rRMPolicyMinRatio>
    <rRMPolicyDedicatedRatio>40</rRMPolicyDedicatedRatio>
  </attributes>
</RRMPolicyRatio> 
```

- UL RRM Policy:
    - Type = UL PRB
    - Dedicated Ration = 40%
    - Min Ratio = 40%
    - Max Ratio = 90%


### 3.2. `odu_config.xml`

- File location = `l2/build/config/odu_config.xml`

- Code Snippet (Part 1 - `BWP_DL_CFG`):
```xml=446
...
         <BWP_DL_CFG>
            <BWP_PARAMS>
               <FIRST_PRB>0</FIRST_PRB>
               <NUM_PRB>106</NUM_PRB>
               <NR_SCS>0</NR_SCS>
               <NORMAL_CYCLIC_PREFIX>0</NORMAL_CYCLIC_PREFIX>
            </BWP_PARAMS>
...
```

- NUM PRB DL = 106

- Code Snippet (Part 2 - `BWP_UL_CFG`):
```xml=488
...
         <BWP_UL_CFG>
            <BWP_PARAMS>
               <FIRST_PRB>0</FIRST_PRB>
               <NUM_PRB>106</NUM_PRB>
               <NR_SCS>0</NR_SCS>
               <NORMAL_CYCLIC_PREFIX>0</NORMAL_CYCLIC_PREFIX>
            </BWP_PARAMS>
...
```

- NUM PRB UL = 106


- Code Snippet (Part 3 - DL RRM Policy):
```xml=593
...
   <SLICE_CFG>
      <NUM_RRM_POLICY>1</NUM_RRM_POLICY>
      <MAC_SLICE_RRM_POLICY>
         <RESOURCE_TYPE>0</RESOURCE_TYPE>
         <NUM_RRM_POLICY_MEMBER>1</NUM_RRM_POLICY_MEMBER>
         <RRM_POLICY_MEMBER_LIST>
            <PLMN>
               <MCC>
                  <PLMN_MCC0>3</PLMN_MCC0>
                  <PLMN_MCC1>1</PLMN_MCC1>
                  <PLMN_MCC2>1</PLMN_MCC2>
               </MCC>
               <MNC>
                  <PLMN_MNC0>4</PLMN_MNC0>
                  <PLMN_MNC1>8</PLMN_MNC1>
                  <PLMN_MNC2>0</PLMN_MNC2>
               </MNC>
            </PLMN>
            <SNSSAI>
               <SST>1</SST>
               <SD_SIZE>
               <SD>2</SD>
               <SD>3</SD>
               <SD>4</SD>
               </SD_SIZE>
            </SNSSAI>
         </RRM_POLICY_MEMBER_LIST>
         <RRM_POLICY_RATIO>
            <MAX_RATIO>90</MAX_RATIO>
            <MIN_RATIO>30</MIN_RATIO>
            <DEDICATED_RATIO>10</DEDICATED_RATIO>
         </RRM_POLICY_RATIO>
      </MAC_SLICE_RRM_POLICY>
   </SLICE_CFG>
...
```

- DL RRM Policy:
    - Type = DL PRB (0)
    - Dedicated Ration = 10%
    - Min Ratio = 30%
    - Max Ratio = 90%

### 3.3. `cell_config.xml`
- Not Important

### 3.4. `sch.h`


- File location = `l2/src/5gnrsch/sch.h`

- Code Snippet (Part 1 - `SchDlCb`):
```c=392
...
typedef struct schDlCb
{
   SchDlLcCtxt   dlLcCtxt[MAX_NUM_LC];
}SchDlCb;
...
```

- Explanation = Each DL control block has a list of DL LC. The length of the list is as per `MAX_NUM_LC`

- Code Snippet (Part 2 - `SchDlLcCtxt`):
```c=380
...
typedef struct schLcCtxt
{
   uint8_t lcId;     // logical Channel ID
   uint8_t lcp;      // logical Channel Prioritization
   SchLcState lcState;
   uint32_t bo;
   uint16_t   pduSessionId; /*Pdu Session Id*/
   Snssai  *snssai;      /*S-NSSAI assoc with LCID*/
   bool isDedicated;     /*Flag containing Dedicated S-NSSAI or not*/
   uint16_t rsvdDedicatedPRB;
}SchDlLcCtxt;
...
```

- Explanation = Each DL LC has several attributes. The important attributes are:
    - `isDedicated` = Flag to indicate if the LC is dedicated or not
    - `rsvdDedicatedPRB` = Number of reserved PRB if the channel is dedicated


- Code Snippet (Part 3 - `SchUlCb`):
```c=412
...
typedef struct schUlCb
{
   SchUlLcCtxt ulLcCtxt[MAX_NUM_LC];
}SchUlCb;
...
```

- Explanation = Each UL control block has a list of UL LC. The length of the list is as per `MAX_NUM_LC`

- Code Snippet (Part 4 - `SchUlLcCtxt`):
```c=397
...
typedef struct schUlLcCtxt
{
   SchLcState  lcState;
   uint8_t lcId;
   uint8_t priority;
   uint8_t lcGroup;
   uint8_t schReqId;
   uint8_t pbr;        // prioritisedBitRate
   uint8_t bsd;        // bucketSizeDuration
   uint16_t   pduSessionId; /*Pdu Session Id*/
   Snssai  *snssai;      /*S-NSSAI assoc with LCID*/
   bool isDedicated;     /*Flag containing Dedicated S-NSSAI or not*/
   uint16_t rsvdDedicatedPRB;
}SchUlLcCtxt;
...
```

- Explanation = Each UL LC has several attributes. The important attributes are:
    - `isDedicated` = Flag to indicate if the LC is dedicated or not
    - `rsvdDedicatedPRB` = Number of reserved PRB if the channel is dedicated

- Code Snippet (Part 5 - `SchCb`):
```c=717
...
/**
 * @brief
 * Control block for sch
 */
typedef struct schCb
{
   TskInit                schInit;               /*!< Task Init info */
   SchGenCb               genCfg;                /*!< General Config info */
   SchTimer               schTimersInfo;         /*!< Sch timer queues and resolution */
   SchAllApis             allApis[NUM_SCH_TYPE]; /*!<List of All Scheduler Type dependent Function pointers*/
   SchCellCb              *cells[MAX_NUM_CELL];  /* Array to store cellCb ptr */
   CmLListCp              sliceCfg;              /* Linklist to Store Slice configuration */
   SchStatistics          statistics;            /* Statistics configuration and calculated values */
}SchCb;
...
```

- Explanation = Control block for SCH has a link list to store slice configuration

- Code Snippet (Part 6 - `SchCellCb`):
```c=665
...
/**
 * @brief
 * Cell Control block per cell.
 */
typedef struct schCellCb
{
   uint16_t      cellId;                            /*!< Cell ID */
...
```
```c=689
...
   SchUeCb       ueCb[MAX_NUM_UE];                  /*!< Pointer to UE contexts of this cell */
...
```

- Explanation = Control block for Cell has an array to store UE Control Block


- Code Snippet (Part 7 - `SchUeCb`):
```c=492
...
/**
 * @brief
 * UE control block
 */
typedef struct schUeCb
{
   uint16_t   ueId;
   uint16_t   crnti;
   SchUeCfgCb ueCfg;
   SchUeState state;
   SchCellCb  *cellCb;
   SchCfraResource cfraResource;
   bool       srRcvd;
   bool       bsrRcvd;
   BsrInfo    bsrInfo[MAX_NUM_LOGICAL_CHANNEL_GROUPS];
   SchUlCb    ulInfo;
   SchDlCb    dlInfo;
   SchUlHqEnt ulHqEnt;
...
```

- Explanation = 

- Code Snippet (Part 8 - `lcInfo`):
```c=165
...
/*Following structures to keep record and estimations of PRB allocated for each
 * LC taking into consideration the RRM policies*/
typedef struct lcInfo
{
   uint8_t  lcId;     /*LCID for which BO are getting recorded*/
   uint32_t reqBO;    /*Size of the BO requested/to be allocated for this LC*/
   uint32_t allocBO;  /*TBS/BO Size which is actually allocated*/
   uint8_t  allocPRB; /*PRB count which is allocated based on RRM policy/FreePRB*/
}LcInfo;
...
```

- Explanation = 

- Code Snippet (Part 9 - `SchDlHqProcCb`):
```c=248
...
struct schDlHqProcCb
{
   uint8_t           procId;       /*!< HARQ Process ID */
   SchDlHqEnt        *hqEnt;
   uint8_t           maxHqTxPerHqP;
   CmLList           dlHqEntLnk;
   CmLList           dlSlotLnk;
   SchDlHqTbCb       tbInfo[2];
   uint8_t           k1;
   void              *schSpcDlHqProcCb;  /*!< Scheduler specific HARQ Proc CB */
   CmLList           dlHqProcLink;
   SlotTimingInfo    pucchTime;
#ifdef NR_DRX
   SchDrxHarqCb      dlDrxHarqCb;
#endif
};
...
```

- Explanation = 



- Code Snippet (Part 10 - `schDlHqEnt`):
```c=274
...
struct schDlHqEnt
{
   SchCellCb      *cell;     /*!< Contains the pointer to cell */
   SchUeCb        *ue;       /*!< Contains the pointer to UE */
   CmLListCp      free;      /*!< List of free HARQ processes */
   CmLListCp      inUse;     /*!< List of in-use HARQ processes */
   uint8_t        maxHqTx;   /*!< Maximum number of harq transmissions */
   uint8_t        numHqPrcs; /*!< Number of HARQ Processes */
   SchDlHqProcCb  procs[SCH_MAX_NUM_DL_HQ_PROC];/*!< Downlink harq processes */
};
...
```

- Explanation = a DL HQ Entity is for 1 Cell & 1 UE


### 3.5. `common_def.h`

- File location = `l2/src/cm/common_def.h`

- Code Snippet (Part 1 - `MAX_NUM_LC`):
```c=62
...
#define MAX_NUM_UE_PER_TTI 2
#define MAX_NUM_LC   MAX_DRB_LCID + 1   /*Spec 38.331: Sec 6.4: maxLC-ID Keyword*/
#define MAX_NUM_SRB  3    /* Max. no of Srbs */
#define MAX_NUM_DRB  29   /* spec 38.331, maxDRB */
...
```

- Explanation:
    - `MAX_NUM_LC` is defined as 33 (32 + 1)
    - `MAX_NUM_SRB` is defined as 3
    - `MAX_NUM_DRB` is defined as 3

- Code Snippet (Part 2 - `MAX_DRB_LCID`):
```c=78
...
/* LCID */
#define SRB0_LCID  0
#define SRB1_LCID  1
#define SRB2_LCID  2
#define SRB3_LCID  3
#define MIN_DRB_LCID 4
#define MAX_DRB_LCID 32
...
```

- Explanation:
    - `MAX_DRB_LCID` is defined as 32
    - OSC make 4 LC ID for SRB
    - OSC make 29 LC ID for DRB

- Code Snippet (Part 3 - `MAX_NUM_RB`):
```c=98
...
/* PRB allocation as per 38.101, Section 5.3.2 */
#define TOTAL_PRB_20MHZ_MU0 106
#define TOTAL_PRB_100MHZ_MU1 273
#ifdef NR_TDD
#define MAX_NUM_RB TOTAL_PRB_100MHZ_MU1  /* value for numerology 1, 100 MHz */
#else
#define MAX_NUM_RB TOTAL_PRB_20MHZ_MU0 /* value for numerology 0, 20 MHz */
#endif
...
```

- Explanation:
    - `TOTAL_PRB_20MHZ_MU0` is defined as 106
    - `MAX_NUM_RB` is = 106 = `TOTAL_PRB_20MHZ_MU0`
    - The default duplexing is FDD (!TDD), so the default `MAX_NUM_RB` = 106

### 3.6. `mac_sch_interface.h`

- File location = `l2/src/cm/mac_sch_interface.h`

- Code Snippet (Part 1 - `SchRrmPolicyRatio`):
```c=950
...
/*Ref: ORAN_WG8.V7.0.0 Sec 11.2.4.2.3*/
typedef struct schRrmPolicyRatio
{
   uint8_t maxRatio;
   uint8_t minRatio;
   uint8_t dedicatedRatio;
}SchRrmPolicyRatio;
...
```

- Explanation:
    - RRM Policy has 3 elements, which are `maxRatio`, `minRatio`, and `dedicatedRatio`

- Code Snippet (Part 2 - `SchRrmPolicyOfSlice`):
```c=958
...
typedef struct schRrmPolicyOfSlice
{
   Snssai  snssai;
   SchRrmPolicyRatio rrmPolicyRatioInfo;
}SchRrmPolicyOfSlice;
...
```

- Explanation:
    - Each slice has snssai and RRM Policy

- Code Snippet (Part 3 - `SchLcCfg`):
```c=1966
...
/* Logical Channel configuration */
typedef struct schLcCfg
{
   uint8_t        lcId;
   Snssai         *snssai;
   SchDrbQosInfo  *drbQos;
   SchDlLcCfg     dlLcCfg;
   SchUlLcCfg     ulLcCfg;
}SchLcCfg;
...
```

- Explanation:
    - Each logical channel has an ID, `snssai`, `drbQos`, DL LC Configuration, and UL LC Configuration

- Code Snippet (Part 4 - `SchUlLcCfg`):
```c=1950
...
/* Uplink logical channel configuration */
typedef struct SchUlLcCfg
{
   uint8_t priority;
   uint8_t lcGroup;
   uint8_t schReqId;
   uint8_t pbr;        // prioritisedBitRate
   uint8_t bsd;        // bucketSizeDuration
}SchUlLcCfg;
...
```

- Explanation:
    - For each LC, the UL configuration consist of:
        - priority
        - lc Group
        - prioritised Bit Rate
        - bucket Size Duration

- Code Snippet (Part 5 - `SchDlLcCfg`):
```c=1960
...
/* Downlink logical channel configuration */
typedef struct schDlLcCfg
{
   uint8_t lcp;      // logical Channel Prioritization
}SchDlLcCfg;
...
```

- Explanation:
    - For each LC, the DL configuration consist of logical channel prioritation

### 3.7. `du_app_mac_inf.h`

- File location = `l2/src/cm/du_app_mac_inf.h`

- Code Snippet (Part 1 - `RrmPolicyRatio`):
```c=1717
...
typedef struct rrmPolicyRatio
{
   uint8_t maxRatio;
   uint8_t minRatio;
   uint8_t dedicatedRatio;
}RrmPolicyRatio;
...
```

- Explanation:
    - RRM Policy has 3 elements, which are `maxRatio`, `minRatio`, and `dedicatedRatio`

- Code Snippet (Part 2 - `MacSliceRrmPolicy`):
```c=1730
...
typedef struct macSliceRrmPolicy
{
   ResourceType        resourceType;
   uint8_t             numOfRrmPolicyMem;
   RrmPolicyMemberList **rRMPolicyMemberList;
   RrmPolicyRatio      policyRatio;
}MacSliceRrmPolicy;
...
```

- Explanation:
    - Each slice has `resourceType`, RRM Policy, number of members, and member list

### 3.8. `du_cfg.h`

- File location = `l2/src/du_app/du_cfg.h`

- Code Snippet (Part 1 - Slice Ratio):
```c=187
...
/* Slice Ratio */
#define MAX_RATIO        30
#define MIN_RATIO        20
#define DEDICATED_RATIO  10
...
```

- Explanation:
    - OSC defined the hardcoded RRM policy as below:
        - MAX RATIO = 30%
        - MIN RATIO = 20%
        - DEDICATED RATIO = 10%

- Code Snippet (Part 2 - `RrmPolicyList`):
```c=1193
...
typedef struct rrmPolicyList
{
   char id[1];
   RrmResourceType resourceType;
   uint8_t rRMMemberNum;
   RRMPolicyMemberList rRMPolicyMemberList[2];
   uint8_t rRMPolicyMaxRatio;
   uint8_t rRMPolicyMinRatio;
   uint8_t rRMPolicyDedicatedRatio;
}RrmPolicyList;
...
```

- Explanation:
    - Each `RrmPolicyList` has `resourceType`, RRM Policy (MAX, MIN & Dedicated), number of members, and member list

### 3.9. `phy_stub.h`

- File location = `l2/src/phy_stub/phy_stub.h`

- Code Snippet (Part 1 - `ueCb`):
```c=61
...
/* UE specific information */
typedef struct ueCb
{
   uint8_t  ueId;
   uint16_t crnti;
   bool     rachIndSent;
   bool     isCFRA;
...
```

- Explanation:

- Code Snippet (Part 2 - `ueDb`):
```c=81
...
/* Database to store information for all UE */
typedef struct ueDb
{
   uint8_t   numActvUe;
   UeCb      ueCb[MAX_NUM_UE];
}UeDb;
...
```

- Explanation:

### 3.10. `mac.h`

- File location = `l2/src/5gnrmac/mac.h`

- Code Snippet (Part 1 - `macUeCb`):
```c=216
...
/* UE Cb */
typedef struct macUeCb
{
   uint16_t         ueId;           /* UE Id from DU APP */
   uint16_t         crnti;          /* UE CRNTI */
   MacCellCb        *cellCb;        /* Pointer to cellCb to whihc this UE belongs */
   UeState          state;          /* Is UE active ? */
   MacCfraResource  cfraResource;   /* CF-RA resource */
   MacRaCbInfo      *raCb;          /* RA info */
   MacBsrTmrCfg     bsrTmrCfg;      /* BSR Timer Info */
   UeUlCb           ulInfo;         /* UE specific UL info */
   UeDlCb           dlInfo;         /* UE specific DL info */
   DataTransmissionAction transmissionAction;
}MacUeCb;
...
```

- Explanation:


- Code Snippet (Part 2 - `macCellCb`):
```c=231
...
struct macCellCb
{
   uint16_t        cellId;
   uint16_t        numOfSlots;
   MacCellStatus   state;
   uint16_t        crntiMap;
   MacRaCbInfo     macRaCb[MAX_NUM_UE];
   MacDlSlot       dlSlot[MAX_SLOTS];
   MacUlSlot       ulSlot[MAX_SLOTS];
   uint16_t        numActvUe;
   MacUeCreateReq  *ueCfgTmpData[MAX_NUM_UE];
   MacUeRecfg      *ueRecfgTmpData[MAX_NUM_UE];
   MacUeCb         ueCb[MAX_NUM_UE];
   MacCellCfg      macCellCfg;
   uint8_t         numerology;
   SlotTimingInfo  currTime;
};
...
```

- Explanation: Control block for Cell has an array to store UE Control Block


- Code Snippet (Part 3 - `ueUlCb`):
```c=200
...
/* UE specific UL info */
typedef struct ueUlCb
{
   uint8_t    maxReTx;     /* MAX HARQ retransmission */
   uint8_t    numUlLc;     /* Number of uplink logical channels */       
   UlLcCb     lcCb[MAX_NUM_LC];    /* Uplink dedicated logocal channels */
}UeUlCb;
...
```

- Explanation: 

- Code Snippet (Part 4 - `ueDlCb`):
```c=208
...
/* UE specific DL Info */
typedef struct ueDlCb
{
   DlHarqEnt  dlHarqEnt;      /* DL HARQ entity */
   uint8_t    numDlLc;        /* Number of downlink logical channels */
   DlLcCb     lcCb[MAX_NUM_LC];  /* Downlink dedicated logical channels */
}UeDlCb;
...
```

- Explanation: 


- Code Snippet (Part 5 - `ulLcCb`):
```c=165
...
/* Uplink deidcated logical channel info */
typedef struct ulLcCb
{
   uint8_t    lcId;         /* Logical Channel Id */
   uint8_t    lcGrpId;      /* Logical Channel group */
   MacLcState lcActive;     /* Is LC active ? */
   /*Commenting as S-NSSAI and PDU session will be used in future scope*/
   /*For eg: When we have to send these for AMBR cases*/
   #if 0
   uint16_t   pduSessionId; /*Pdu Session Id*/
   Snssai     *snssai;      /*S-NSSAI assoc with LCID*/
   #endif
}UlLcCb;
...
```

- Explanation: 

- Code Snippet (Part 6 - `dlLcCb`):
```c=179
...
/* Downlink dedicated logical channel info */
typedef struct dlLcCb
{
   uint8_t    lcId;        /* Logical channel Id */ 
   MacLcState lcState;     /* Is LC active ? */
   /*Commenting as S-NSSAI and PDU session will be used in future scope*/
   /*For eg: When we have to send these info via FAPI to phy layer*/
   #if 0
   uint16_t   pduSessionId;/*Pdu Session Id*/
   Snssai     *snssai;    /*S-NSSAI assoc with LCID*/
   #endif
}DlLcCb;
...
```

- Explanation: 

### 3.11. `sch_fcfs.h`

- File location = `l2/src/5gnrsch/sch_fcfs.h`

- Code Snippet (Part 1 - `SchFcfsHqProcCb`):
```c=35
...
typedef struct schFcfsHqProcCb
{
   SchFcfsLcCb lcCb; 
}SchFcfsHqProcCb;
...
```

- Explanation: 

- Code Snippet (Part 2 - `SchFcfsHqProcCb`):
```c=23
...
typedef struct schFcfsLcCb
{
   /* TODO: For Multiple RRMPolicies, Make DedicatedLcInfo as array/Double Pointer 
    * and have separate DedLCInfo for each RRMPolcyMemberList*/
   /* Dedicated LC List will be allocated, if any available*/
   CmLListCp dedLcList;	/*Contain LCInfo per RRMPolicy*/
   CmLListCp defLcList; /*Linklist of LC assoc with Default S-NSSAI(s)*/
   /* SharedPRB number can be used by any LC.
    * Need to calculate in every Slot based on PRB availability*/
   uint16_t sharedNumPrb;
}SchFcfsLcCb;
...
```

- Explanation: 
    - There are 2 logical channel list:
        - dedicated LC link list
        - default LC link list
    - share num PRB is tracked here


- Code Snippet (Part 3 - `SchFcfsCellCb`):
```c=18
...
typedef struct schFcfsCellCb
{
   CmLListCp     ueToBeScheduled;                   /*!< Linked list to store UEs pending to be scheduled, */
}SchFcfsCellCb;
...
```

- Explanation:  Link list of UE to be scheduled in a cell

- Code Snippet (Part 4 - `SchFcfsUeCb`):
```c=46
...
typedef struct schFcfsUeCb
{
   SchFcfsHqCb   hqRetxCb;
}SchFcfsUeCb;
...
```

- Explanation:  Link list of UE to be scheduled in a cell


## 4. OSC's Logical Channel Initialization using RRM Policy

### 4.1. Flow

### 4.2. `updateDedLcInfo`

- File location = `l2/src/5gnrsch/sch_ue_mgr.c`

- Code Snippet:
```c=177
...
/*******************************************************************
 *
 * @brief Function to update/allocate dedLC Info
 *
 * @details
 *
 *    Function : updateDedLcInfo
 *
 *    Functionality: Function to fill DLDedLcInfo
 *
 * @params[arg] scheduler instance,
 *              snssai pointer,
 *              SchRrmPolicy pointer,
 *              SchLcPrbEstimate pointer , It will be filled
 *              isDedicated pointer,(Address of isDedicated flag in LC Context)
 *
 * @return ROK      >> This LC is part of RRM MemberList.
 *         RFAILED  >> FATAL Error
 *         ROKIGNORE >> This LC is not part of this RRM MemberList 
 *
 * ****************************************************************/

uint8_t updateDedLcInfo(Inst inst, Snssai *snssai, uint16_t *rsvdDedicatedPRB, bool *isDedicated)
{
   CmLList *sliceCfg = schCb[inst].sliceCfg.first;
   SchRrmPolicyOfSlice *rrmPolicyOfSlices;

   while(sliceCfg)
   {
      rrmPolicyOfSlices = (SchRrmPolicyOfSlice*)sliceCfg->node;
      if(rrmPolicyOfSlices && (memcmp(snssai, &(rrmPolicyOfSlices->snssai), sizeof(Snssai)) == 0))
      {
         /*Updating latest RrmPolicy*/
         *rsvdDedicatedPRB = \
                             (uint16_t)(((rrmPolicyOfSlices->rrmPolicyRatioInfo.dedicatedRatio)*(MAX_NUM_RB))/100);
         *isDedicated = TRUE;
         DU_LOG("\nINFO  -->  SCH : Updated RRM policy, reservedPOOL:%d",*rsvdDedicatedPRB);
         break;
      }
      sliceCfg = sliceCfg->next;
   }
   /*case: This LcCtxt  is either a Default LC or this LC is part of someother RRM_MemberList*/
   if(*isDedicated != TRUE) 
   {
      DU_LOG("\nINFO  -->  SCH : This SNSSAI is not a part of this RRMPolicy");
   }
   return ROK;	 
}
...
```

- Explanation:
    - for each dedicated logical channel, the number of `rsvdDedicatedPRB` will be set to (`dedicatedRatio`*`MAX_NUM_RB`)/100
    - In our DL case, `rsvdDedicatedPRB` will be set to (10*106)/100 = 10
    - In our UL case, `rsvdDedicatedPRB` will be set to (40*106)/100 = 42


### 4.3. `cpyRrmPolicyInDuCfgParams`

- File location = `l2/src/du_app/du_cfg.c`

- Code Snippet:
```c=104
...
/*******************************************************************
 *
 * @brief Copy Slice Cfg in temp structre in duCfgParams 
 *
 * @details
 *
 *    Function : cpyRrmPolicyInDuCfgParams
 *
 *    Functionality:
 *      - Copy Slice Cfg in temp structre in duCfgParams 
 *
 * @params[in] RrmPolicy rrmPolicy[], uint8_t policyNum, uint8_t memberList
 * @return ROK     - success
 *         RFAILED - failure
 *
 * ****************************************************************/
uint8_t cpyRrmPolicyInDuCfgParams(RrmPolicyList rrmPolicy[], uint8_t policyNum, MacSliceCfgReq *tempSliceCfg)
{
   uint8_t policyIdx = 0, memberListIdx = 0;
   if(policyNum)
   {
      tempSliceCfg->numOfRrmPolicy = policyNum;
      DU_ALLOC_SHRABL_BUF(tempSliceCfg->listOfRrmPolicy, tempSliceCfg->numOfRrmPolicy  * sizeof(MacSliceRrmPolicy*));
      if(!tempSliceCfg->listOfRrmPolicy)
      {
         DU_LOG("\nERROR  --> DU APP : Memory allocation failed in cpyRrmPolicyInDuCfgParams");
         return RFAILED;
      }

      for(policyIdx = 0; policyIdx<tempSliceCfg->numOfRrmPolicy; policyIdx++)
      {
         DU_ALLOC_SHRABL_BUF(tempSliceCfg->listOfRrmPolicy[policyIdx], sizeof(MacSliceRrmPolicy));
         if(!tempSliceCfg->listOfRrmPolicy[policyIdx])
         {
            DU_LOG("\nERROR  --> DU APP : Memory allocation failed in cpyRrmPolicyInDuCfgParams");
            return RFAILED;
         }

         tempSliceCfg->listOfRrmPolicy[policyIdx]->resourceType = rrmPolicy[policyIdx].resourceType;

         tempSliceCfg->listOfRrmPolicy[policyIdx]->numOfRrmPolicyMem = rrmPolicy[policyIdx].rRMMemberNum;

         if(tempSliceCfg->listOfRrmPolicy[policyIdx]->numOfRrmPolicyMem)
         {
            DU_ALLOC_SHRABL_BUF(tempSliceCfg->listOfRrmPolicy[policyIdx]->rRMPolicyMemberList,\
            tempSliceCfg->listOfRrmPolicy[policyIdx]->numOfRrmPolicyMem * sizeof(RrmPolicyMemberList*));

            if(!tempSliceCfg->listOfRrmPolicy[policyIdx]->rRMPolicyMemberList)
            {
               DU_LOG("\nERROR  --> DU APP : Memory allocation failed in cpyRrmPolicyInDuCfgParams");
               return RFAILED;
            }


            for(memberListIdx = 0; memberListIdx<tempSliceCfg->listOfRrmPolicy[policyIdx]->numOfRrmPolicyMem; memberListIdx++)
            {
               DU_ALLOC_SHRABL_BUF(tempSliceCfg->listOfRrmPolicy[policyIdx]->rRMPolicyMemberList[memberListIdx], sizeof(RrmPolicyMemberList));
               if(!tempSliceCfg->listOfRrmPolicy[policyIdx]->rRMPolicyMemberList[memberListIdx])
               {
                  DU_LOG("\nERROR  --> DU APP : Memory allocation failed in cpyRrmPolicyInDuCfgParams");
                  return RFAILED;
               }
               memcpy(&tempSliceCfg->listOfRrmPolicy[policyIdx]->rRMPolicyMemberList[memberListIdx]->snssai.sd,\
               &rrmPolicy[policyIdx].rRMPolicyMemberList[memberListIdx].sd, 3 * sizeof(uint8_t));
               memcpy(&tempSliceCfg->listOfRrmPolicy[policyIdx]->rRMPolicyMemberList[memberListIdx]->snssai.sst,\
               &rrmPolicy[policyIdx].rRMPolicyMemberList[memberListIdx].sst, sizeof(uint8_t));
               memcpy(&tempSliceCfg->listOfRrmPolicy[policyIdx]->rRMPolicyMemberList[memberListIdx]->plmn.mcc,\
               &rrmPolicy[policyIdx].rRMPolicyMemberList[memberListIdx].mcc, 3 * sizeof(uint8_t));
               memcpy(&tempSliceCfg->listOfRrmPolicy[policyIdx]->rRMPolicyMemberList[memberListIdx]->plmn.mnc,\
               &rrmPolicy[policyIdx].rRMPolicyMemberList[memberListIdx].mnc, 3 * sizeof(uint8_t));
            }
            tempSliceCfg->listOfRrmPolicy[policyIdx]->policyRatio.maxRatio = rrmPolicy[policyIdx].rRMPolicyMaxRatio;
            tempSliceCfg->listOfRrmPolicy[policyIdx]->policyRatio.minRatio = rrmPolicy[policyIdx].rRMPolicyMinRatio;
            tempSliceCfg->listOfRrmPolicy[policyIdx]->policyRatio.dedicatedRatio = rrmPolicy[policyIdx].rRMPolicyDedicatedRatio;
         }
      }
   }

   return ROK;
}
...
```

- Explanation:


### 4.4. `fillSliceCfgInfo`

- File location = `l2/src/5gnrmac/mac_msg_hdl.c`

- Code Snippet:
```c=905
...
/*******************************************************************
 *
 * @brief fill Slice Cfg Request info in shared structre
 * 
 * @details
 *
 *    Function : fillSliceCfgInfo 
 *
 *    Functionality:
 *       fill Slice Cfg Request info in shared structre
 *
 * @params[in] SchSliceCfgReq *schSliceCfgReq 
 *             MacSliceCfgReq *macSliceCfgReq;
 * @return ROK     - success
 *         RFAILED - failure
 *
 **********************************************************************/
uint8_t fillSliceCfgInfo(SchSliceCfgReq *schSliceCfgReq, MacSliceCfgReq *macSliceCfgReq)
{
   uint8_t rrmPolicyIdx= 0,cfgIdx = 0, memberListIdx = 0, totalSliceCfgRecvd = 0;
 
   if(macSliceCfgReq->listOfRrmPolicy)
   {
      for(cfgIdx = 0; cfgIdx<macSliceCfgReq->numOfRrmPolicy; cfgIdx++)
      {
          totalSliceCfgRecvd += macSliceCfgReq->listOfRrmPolicy[cfgIdx]->numOfRrmPolicyMem;  
      }

      schSliceCfgReq->numOfConfiguredSlice =  totalSliceCfgRecvd;
      MAC_ALLOC(schSliceCfgReq->listOfSlices, schSliceCfgReq->numOfConfiguredSlice *sizeof(SchRrmPolicyOfSlice*));
      if(schSliceCfgReq->listOfSlices == NULLP)
      {
         DU_LOG("\nERROR  -->  MAC : Memory allocation failed in fillSliceCfgInfo");
         return RFAILED;
      }
      cfgIdx = 0; 

      for(rrmPolicyIdx = 0; rrmPolicyIdx<macSliceCfgReq->numOfRrmPolicy; rrmPolicyIdx++)
      {
         for(memberListIdx = 0; memberListIdx<macSliceCfgReq->listOfRrmPolicy[rrmPolicyIdx]->numOfRrmPolicyMem; memberListIdx++)
         {
            if(macSliceCfgReq->listOfRrmPolicy[rrmPolicyIdx]->rRMPolicyMemberList[memberListIdx])
            {

               MAC_ALLOC(schSliceCfgReq->listOfSlices[cfgIdx], sizeof(SchRrmPolicyOfSlice));
               if(schSliceCfgReq->listOfSlices[cfgIdx] == NULLP)
               {
                  DU_LOG("\nERROR  -->  MAC : Memory allocation failed in fillSliceCfgInfo");
                  return RFAILED;
               }

               memcpy(&schSliceCfgReq->listOfSlices[cfgIdx]->snssai, &macSliceCfgReq->listOfRrmPolicy[rrmPolicyIdx]->rRMPolicyMemberList[memberListIdx]->snssai, sizeof(Snssai));

               schSliceCfgReq->listOfSlices[cfgIdx]->rrmPolicyRatioInfo.maxRatio = macSliceCfgReq->listOfRrmPolicy[rrmPolicyIdx]->policyRatio.maxRatio;
               schSliceCfgReq->listOfSlices[cfgIdx]->rrmPolicyRatioInfo.minRatio = macSliceCfgReq->listOfRrmPolicy[rrmPolicyIdx]->policyRatio.minRatio;
               schSliceCfgReq->listOfSlices[cfgIdx]->rrmPolicyRatioInfo.dedicatedRatio = macSliceCfgReq->listOfRrmPolicy[rrmPolicyIdx]->policyRatio.dedicatedRatio;
               cfgIdx++;
            }
         }
      }
   }
   return ROK;
}
...
```

- Explanation:

### 4.5. `addSliceCfgInSchDb`

- File location = `l2/src/5gnrsch/sch.c`

- Code Snippet:
```c=1710
...
/*******************************************************************************
 *
 * @brief This function is used to store or modify the slice configuration Sch DB
 *
 * @details
 *
 *    Function : addOrModifySliceCfgInSchDb 
 *
 *    Functionality:
 *     function is used to store or modify the slice configuration Sch DB
 *
 * @params[in] SchSliceCfg *storeSliceCfg, SchSliceCfgReq *cfgReq,
 * SchSliceCfgRsp cfgRsp, uint8_t count
 *
 * @return
 *        ROK - Success
 *        RFAILED - Failure
 *
 * ********************************************************************************/
uint8_t addSliceCfgInSchDb(CmLListCp *sliceCfgInDb, SchRrmPolicyOfSlice *cfgReq)
{
   SchRrmPolicyOfSlice *sliceToStore;

   SCH_ALLOC(sliceToStore, sizeof(SchRrmPolicyOfSlice));
   if(sliceToStore)
   {
      memcpy(&sliceToStore->snssai, &cfgReq->snssai, sizeof(Snssai));  
      memcpy(&sliceToStore->rrmPolicyRatioInfo, &cfgReq->rrmPolicyRatioInfo, sizeof(SchRrmPolicyRatio));  
      addNodeToLList(sliceCfgInDb, sliceToStore, NULL);
   }
   else
   {
      DU_LOG("\nERROR  -->  SCH : Memory allocation failed in addOrModifySliceCfgInSchDb");
      return RFAILED;
   }
   return ROK;
}
...
```

- Explanation:

### 4.6. `parseRrmPolicyRatio`

- File location = `l2/src/du_app/du_cfg.c`

- Code Snippet:
```c=4129
...
/*******************************************************************
 *
 * @brief Fill RRM Policy Ratio
 *
 * @details
 *
 *    Function : parseRrmPolicyRatio
 *
 *    Functionality: Fill RRM Policy Ratio
 *
 * @params[in] XML document pointer
 *             XML namespace
 *             Current node in XML
 *             Pointer to structure to be filled
 * @return ROK     - success
 *         RFAILED - failure
 *
 * ****************************************************************/
uint8_t parseRrmPolicyRatio(xmlDocPtr doc, xmlNsPtr ns, xmlNodePtr cur, RrmPolicyRatio *rrmPolicyRatio)
{
   memset(rrmPolicyRatio, 0, sizeof(RrmPolicyRatio));
   cur = cur -> xmlChildrenNode;
   while(cur != NULL)
   {
      if ((!xmlStrcmp(cur->name, (const xmlChar *)"MAX_RATIO")) && (cur->ns == ns))
      {
         rrmPolicyRatio->maxRatio = atoi((char *)xmlNodeListGetString(doc, cur->xmlChildrenNode, 1));
      }

      if ((!xmlStrcmp(cur->name, (const xmlChar *)"MIN_RATIO")) && (cur->ns == ns))
      {
         rrmPolicyRatio->minRatio = atoi((char *)xmlNodeListGetString(doc, cur->xmlChildrenNode, 1));
      }

      if ((!xmlStrcmp(cur->name, (const xmlChar *)"DEDICATED_RATIO")) && (cur->ns == ns))
      {
         rrmPolicyRatio->dedicatedRatio = atoi((char *)xmlNodeListGetString(doc, cur->xmlChildrenNode, 1));
      }

      cur = cur -> next;
   }
   return ROK;
}
...
```

- Explanation:


### 4.7. `fillSchUlLcCtxt`

- File location = `l2/src/5gnrsch/sch_ue_mgr.c`

- Code Snippet:
```c=136
...
/*******************************************************************
 *
 * @brief Function to fill Ul Lc Context in SCH Ue Cb
 *
 * @details
 *
 *    Function : fillSchUlLcCtxt
 *
 *    Functionality: Function to fill Ul Lc Context in SCH Ue Cb
 *
 * @params[in] SchUlLcCtxt pointer,
 *             SchLcCfg pointer
 * @return void
 *
 * ****************************************************************/

void fillSchUlLcCtxt(SchUlLcCtxt *ueCbLcCfg, SchLcCfg *lcCfg)
{
   ueCbLcCfg->lcId = lcCfg->lcId;
   ueCbLcCfg->lcState = SCH_LC_STATE_ACTIVE;
   ueCbLcCfg->priority = lcCfg->ulLcCfg.priority;
   ueCbLcCfg->lcGroup = lcCfg->ulLcCfg.lcGroup;
   ueCbLcCfg->schReqId = lcCfg->ulLcCfg.schReqId;
   ueCbLcCfg->pbr     = lcCfg->ulLcCfg.pbr;
   ueCbLcCfg->bsd     = lcCfg->ulLcCfg.bsd;

   if(lcCfg->drbQos)
   {
      ueCbLcCfg->pduSessionId = lcCfg->drbQos->pduSessionId;
   }
   if(lcCfg->snssai)
   {
     /*In CONFIG_MOD case, no need to allocate SNSSAI memory again*/
     if(ueCbLcCfg->snssai == NULLP)
     {
        SCH_ALLOC(ueCbLcCfg->snssai, sizeof(Snssai));
     }
     memcpy(ueCbLcCfg->snssai, lcCfg->snssai,sizeof(Snssai));
   }
}
...
```

- Explanation:

### 4.8. `fillSchDlLcCtxt`

- File location = `l2/src/5gnrsch/sch_ue_mgr.c`

- Code Snippet:
```c=99
...
/*******************************************************************
 
 *
 * @brief Function to fill Dl Lc Context in SCH Ue Cb
 *
 * @details
 *
 *    Function : fillSchDlLcCtxt
 *
 *    Functionality: Function to fill Dl Lc Context in SCH Ue Cb
 *
 * @params[in] SchDlLcCtxt pointer,
 *             SchLcCfg pointer
 * @return void
 *
 * ****************************************************************/

void fillSchDlLcCtxt(SchDlLcCtxt *ueCbLcCfg, SchLcCfg *lcCfg)
{
   ueCbLcCfg->lcId = lcCfg->lcId;
   ueCbLcCfg->lcp = lcCfg->dlLcCfg.lcp;
   ueCbLcCfg->lcState = SCH_LC_STATE_ACTIVE;
   ueCbLcCfg->bo = 0;
   if(lcCfg->drbQos)
   {
     ueCbLcCfg->pduSessionId = lcCfg->drbQos->pduSessionId;
   }
   if(lcCfg->snssai)
   {
     if(ueCbLcCfg->snssai == NULLP)/*In CONFIG_MOD case, no need to allocate SNSSAI memory*/
     {
        SCH_ALLOC(ueCbLcCfg->snssai, sizeof(Snssai));
     }
     memcpy(ueCbLcCfg->snssai, lcCfg->snssai,sizeof(Snssai));
   }
}
...
```

- Explanation:



## 5. OSC's Logical Channel Scheduling / Resource Allocation


### 5.1. `prbAllocUsingRRMPolicy`


- File location = `l2/src/5gnrsch/sch_common.c`

- Code Snippet (Full code):
spoiler
    ```c=1535
    ...
    /*******************************************************************************************
     *
     * @brief Allocate the PRB using RRM policy
     *
     * @details
     *
     *    Function : prbAllocUsingRRMPolicy
     *
     *    Functionality:
     *      [Step1]: Traverse each Node in the LC list
     *      [Step2]: Check whether the LC has ZERO requirement then clean this LC
     *      [Step3]: Calcualte the maxPRB for this LC.
     *              a. For Dedicated LC, maxPRB = sum of remainingReservedPRB and
     *              sharedPRB
     *              b. For Default, just SharedPRB count
     *      [Step4]: If the LC is the First one to be allocated for this UE then add
     *      TX_PAYLODN_LEN to reqBO 
     *      [Step5]: Calculate the estimate PRB and estimate BO to be allocated
     *               based on reqBO and maxPRB left.
     *      [Step6]: Based on calculated PRB, Update Reserved PRB and Shared PRB counts
     *      [Step7]: Deduce the reqBO based on allocBO and move the LC node to last.
     *      [Step8]: Continue the next loop from List->head
     *
     *      [Loop Exit]:
     *        [Exit1]: If all the LCs are allocated in list
     *        [Exit2]: If PRBs are exhausted
     *
     * @params[in] I/P > lcLinkList pointer (LcInfo list)
     *             I/P > IsDedicatedPRB (Flag to indicate that RESERVED PRB to use 
     *             I/P > mcsIdx and PDSCH symbols count 
     *             I/P & O/P > Shared PRB , reserved PRB Count
     *             I/P & O/P > Total TBS size accumulated
     *             I/P & O/P > isTxPayloadLenAdded[For DL] : Decision flag to add the TX_PAYLOAD_HDR_LEN
     *             I/P & O/P > srRcvd Flag[For UL] : Decision flag to add UL_GRANT_SIZE
     *
     * @return void
     *
     * *******************************************************************************************/
    void prbAllocUsingRRMPolicy(CmLListCp *lcLL, bool isDedicatedPRB, uint16_t mcsIdx,uint8_t numSymbols,\
                      uint16_t *sharedPRB, uint16_t *reservedPRB, bool *isTxPayloadLenAdded, bool *srRcvd)
    {
       CmLList *node = NULLP;
       LcInfo *lcNode = NULLP;
       uint16_t remReservedPRB = 0, estPrb = 0, maxPRB = 0;

       if(lcLL == NULLP)
       {
          DU_LOG("\nERROR --> SCH: LcList not present");
          return;
       }
       node = lcLL->first;

       /*Only for Dedicated LcList, Valid value will be assigned to remReservedPRB
        * For Other LcList, remReservedPRB = 0*/
       if(reservedPRB != NULLP && isDedicatedPRB == TRUE)
       {
          remReservedPRB = *reservedPRB;
       }

       /*[Step1]*/
       while(node)
       {
    #if 0
          /*For Debugging purpose*/
          printLcLL(lcLL);
    #endif
          lcNode = (LcInfo *)node->node;

          /* [Step2]: Below condition will hit in rare case as it has been taken care during the cleaning 
           * process of LCID which was fully allocated. Check is just for safety purpose*/
          if(lcNode->reqBO == 0 && lcNode->allocBO == 0)
          {
             DU_LOG("\nERROR --> SCH: LCID:%d has no requirement, clearing this node",\
                   lcNode->lcId);
             deleteNodeFromLList(lcLL, node);
             SCH_FREE(lcNode, sizeof(LcInfo));
             node = lcLL->first; 
             continue;
          }

          /*[Exit1]: All LCs are allocated(allocBO = 0 for fully unallocated LC)*/
          if(lcNode->allocBO != 0)
          {
             DU_LOG("\nDEBUG  -->  SCH: All LC are allocated [SharedPRB:%d]",*sharedPRB);
             return;
          }

          /*[Exit2]: If PRBs are exhausted*/
          if(isDedicatedPRB)
          {
             /*Loop Exit: All resources exhausted*/
             if(remReservedPRB == 0 && *sharedPRB == 0)
             {
                DU_LOG("\nDEBUG  -->  SCH: Dedicated resources exhausted for LC:%d",lcNode->lcId);
                return;
             }
          }
          else
          {
             /*Loop Exit: All resources exhausted*/
             if(*sharedPRB == 0)
             {
                DU_LOG("\nDEBUG  -->  SCH: Default resources exhausted for LC:%d",lcNode->lcId);
                return;
             }
          }

          /*[Step3]*/
          maxPRB = remReservedPRB + *sharedPRB;

          /*[Step4]*/
          if((isTxPayloadLenAdded != NULLP) && (*isTxPayloadLenAdded == FALSE))
          {
             DU_LOG("\nDEBUG  -->  SCH: LC:%d is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN",\
                   lcNode->lcId);
             *isTxPayloadLenAdded = TRUE;
             lcNode->allocBO = calculateEstimateTBSize((lcNode->reqBO + TX_PAYLOAD_HDR_LEN),\
                   mcsIdx, numSymbols, maxPRB, &estPrb);
             lcNode->allocBO -=TX_PAYLOAD_HDR_LEN;
          }
          else if((srRcvd != NULLP) && (*srRcvd == TRUE))
          {
             DU_LOG("\nDEBUG  --> SCH: LC:%d is the First node to be allocated which includes UL_GRANT_SIZE",\
                   lcNode->lcId);
             *srRcvd = FALSE;
             lcNode->reqBO += UL_GRANT_SIZE;
             lcNode->allocBO = calculateEstimateTBSize(lcNode->reqBO, mcsIdx, numSymbols, maxPRB, &estPrb);
          }
          else
          {
             /*[Step4]*/
             lcNode->allocBO = calculateEstimateTBSize(lcNode->reqBO,\
                   mcsIdx, numSymbols, maxPRB, &estPrb);
          }

          /*[Step6]:Re-adjust the reservedPRB pool count and *SharedPRB Count based on
           * estimated PRB allocated*/
          if((isDedicatedPRB == TRUE) && (estPrb <= remReservedPRB))
          {
             remReservedPRB = remReservedPRB - estPrb;
          }
          else   /*LC requirement need PRB share from SharedPRB*/
          {
             if(*sharedPRB <=  (estPrb - remReservedPRB))
             {
                DU_LOG("\nDEBUG  -->  SCH: SharedPRB is less");
                *sharedPRB = 0;
             }
             else
             {
                *sharedPRB = *sharedPRB - (estPrb - remReservedPRB);
             }
             remReservedPRB = 0;
          }

          /*[Step7]*/
          lcNode->reqBO -= lcNode->allocBO;  /*Update the reqBO with remaining bytes unallocated*/
          lcNode->allocPRB = estPrb;
          cmLListAdd2Tail(lcLL, cmLListDelFrm(lcLL, node));

          /*[Step8]:Next loop: First LC to be picked from the list
           * because Allocated Nodes are moved to the last*/
          node = lcLL->first; 

       }
       return;
    }
    ...
    ```

- Explanation (summarized code):
    1. This function will get the first logical channel from the logical channel link list
    ```c=1586
    node = lcLL->first;
    ```
    2. Check if the link list is dedicated channels link list or not. If yes, set the remaining reserved PRB count
    ```c=1588
    /*Only for Dedicated LcList, Valid value will be assigned to remReservedPRB
    * For Other LcList, remReservedPRB = 0*/
    if(reservedPRB != NULLP && isDedicatedPRB == TRUE)
    {
    remReservedPRB = *reservedPRB;
    }
    ```
    3. Loop for every logical channel in link list
    ```c=1595
    /*[Step1]*/
    while(node)
    {
        ...
    ```
    4. Check if there are still PRB resource. Shared PRB will always be checked for dedicated & default channels. reserved PRB will only be checked if channel is dedicated
    ```c=1623
    /*[Exit2]: If PRBs are exhausted*/
    if(isDedicatedPRB)
    {
        /*Loop Exit: All resources exhausted*/
        if(remReservedPRB == 0 && *sharedPRB == 0)
        {
            DU_LOG("\nDEBUG  -->  SCH: Dedicated resources exhausted for LC:%d",lcNode->lcId);
            return;
        }
    }
    else
    {
        /*Loop Exit: All resources exhausted*/
        if(*sharedPRB == 0)
        {
            DU_LOG("\nDEBUG  -->  SCH: Default resources exhausted for LC:%d",lcNode->lcId);
            return;
        }
    }
    ```
    5. Count max PRB that can be allocatedfor this channel
    ```c=
    /*[Step3]*/
    maxPRB = remReservedPRB + *sharedPRB;
    ```
    6. Calculate the estimate PRB and estimate BO to be allocated based on reqBO and maxPRB left.
    ```c=
    /*[Step4]*/
    if((isTxPayloadLenAdded != NULLP) && (*isTxPayloadLenAdded == FALSE))
    {
        DU_LOG("\nDEBUG  -->  SCH: LC:%d is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN",\
            lcNode->lcId);
        *isTxPayloadLenAdded = TRUE;
        lcNode->allocBO = calculateEstimateTBSize((lcNode->reqBO + TX_PAYLOAD_HDR_LEN),\
            mcsIdx, numSymbols, maxPRB, &estPrb);
        lcNode->allocBO -=TX_PAYLOAD_HDR_LEN;
    }
    else if((srRcvd != NULLP) && (*srRcvd == TRUE))
    {
        DU_LOG("\nDEBUG  --> SCH: LC:%d is the First node to be allocated which includes UL_GRANT_SIZE",\
            lcNode->lcId);
        *srRcvd = FALSE;
        lcNode->reqBO += UL_GRANT_SIZE;
        lcNode->allocBO = calculateEstimateTBSize(lcNode->reqBO, mcsIdx, numSymbols, maxPRB, &estPrb);
    }
    else
    {
        /*[Step4]*/
        lcNode->allocBO = calculateEstimateTBSize(lcNode->reqBO,\
            mcsIdx, numSymbols, maxPRB, &estPrb);
    }
    ```
    7. Count the remaining  reserved PRB and shared PRB after PRB allocation estimation.
        1. If dedicated channel and estimated PRB is smaller than remaining reserved PRB, then decrement remaining reserved PRB
        ```c=1673
        if((isDedicatedPRB == TRUE) && (estPrb <= remReservedPRB))
        {
            remReservedPRB = remReservedPRB - estPrb;
        }
        ```
        2. Estimated PRB is bigger than "remaining reserved PRB + remaining shared PRB", then exhaust all the PRB
        ```c=1679
        if(*sharedPRB <=  (estPrb - remReservedPRB))
        {
            DU_LOG("\nDEBUG  -->  SCH: SharedPRB is less");
            *sharedPRB = 0;
        }
        ```
        3. Estimated PRB is smaller than total of "remaining reserved PRB + remaining shared PRB", decrement shared PRB number
        ```c=1684
        else
        {
            *sharedPRB = *sharedPRB - (estPrb - remReservedPRB);
        }
        remReservedPRB = 0;
        ```
    8. Update logical channel record
    ```c=1691
    /*[Step7]*/
    lcNode->reqBO -= lcNode->allocBO;  /*Update the reqBO with remaining bytes unallocated*/
    lcNode->allocPRB = estPrb;
    ```
    9. Put current logical channel to the last of the lgocial channel link list
    ```c=
    cmLListAdd2Tail(lcLL, cmLListDelFrm(lcLL, node));
    ```
    10. Get next logical channel from link list
    ```c=1696
    /*[Step8]:Next loop: First LC to be picked from the list
    * because Allocated Nodes are moved to the last*/
    node = lcLL->first; 
    ```




### 5.2. `schFcfsScheduleDlLc`


- File location = `l2/src/5gnrsch/sch_fcfs.c`

- Code Snippet (Full code):
spoiler
    ```c=963
    /*******************************************************************
     *
     * @brief Grants resources to LC in downlink 
     *
     * @details
     *
     *    Function : schFcfsScheduleDlLc 
     *
     *    Functionality: Grants resources to LC in uplink
     *
     * @params[in] PDCCH Time
     *
     * @return ROK
     *         RFAILED
     *
     * ****************************************************************/
    uint32_t schFcfsScheduleDlLc(SlotTimingInfo pdcchTime, SlotTimingInfo pdschTime, uint8_t pdschNumSymbols, \
                                   uint16_t *startPrb, bool isRetx, SchDlHqProcCb **hqP)
    {
       SchFcfsHqProcCb *fcfsHqProcCb;
       SchUeCb *ueCb;
       uint8_t lcIdx = 0;
       uint16_t maxFreePRB = 0;
       uint16_t mcsIdx = 0;
       uint32_t accumalatedSize = 0;
       CmLListCp *lcLL = NULLP;
       uint16_t rsvdDedicatedPRB = 0;
       DlMsgSchInfo *dciSlotAlloc;

       /* TX_PAYLOAD_HDR_LEN: Overhead which is to be Added once for any UE while estimating Accumulated TB Size
        * Following flag added to keep the record whether TX_PAYLOAD_HDR_LEN is added to the first Node getting allocated.
        * If both Dedicated and Default LC lists are present then First LC in Dedicated List will include this overhead
        * else if only Default list is present then first node in this List will add this overhead len*/
       bool isTxPayloadLenAdded = FALSE;

       ueCb = (*hqP)->hqEnt->ue;
       dciSlotAlloc = (*hqP)->hqEnt->cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueCb->ueId -1];
       fcfsHqProcCb = (SchFcfsHqProcCb *)((*hqP)->schSpcDlHqProcCb);

       if (isRetx == FALSE)
       {
          /*Re-Initalization per UE*/
          /* scheduled LC data fill */
          dciSlotAlloc->transportBlock[0].numLc = 0;
          isTxPayloadLenAdded = FALSE; /*Re-initlaize the flag for every UE*/
          accumalatedSize = 0;

          for(lcIdx = 0; lcIdx < MAX_NUM_LC; lcIdx++)
          {
             if(ueCb->dlInfo.dlLcCtxt[lcIdx].bo)
             {
                /*Check the LC is Dedicated or default and accordingly LCList will
                * be used*/
                if(ueCb->dlInfo.dlLcCtxt[lcIdx].isDedicated)
                {
                   lcLL = &(fcfsHqProcCb->lcCb.dedLcList);
                   rsvdDedicatedPRB = ueCb->dlInfo.dlLcCtxt[lcIdx].rsvdDedicatedPRB;
                }
                else
                {
                   lcLL = &(fcfsHqProcCb->lcCb.defLcList);
                }

                /*[Step2]: Update the reqPRB and Payloadsize for this LC in the appropriate List*/
                if(updateLcListReqPRB(lcLL, ueCb->dlInfo.dlLcCtxt[lcIdx].lcId,\
                         (ueCb->dlInfo.dlLcCtxt[lcIdx].bo + MAC_HDR_SIZE)) != ROK)
                {
                   DU_LOG("\nERROR  --> SCH : Updation in LC List Failed");
                   /* Free the dl ded msg info allocated in macSchDlRlcBoInfo */
                   if(!dciSlotAlloc->dlMsgPdschCfg)
                   {
                      SCH_FREE(dciSlotAlloc, sizeof(DlMsgSchInfo));
                      (*hqP)->hqEnt->cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueCb->ueId -1] = NULL;
                   }
                   return accumalatedSize;
                }
             }
          }//End of for loop

          if ((fcfsHqProcCb->lcCb.defLcList.count == 0) && (fcfsHqProcCb->lcCb.dedLcList.count == 0))
          {
             DU_LOG("\nDEBUG  -->  SCH : No pending BO for any LC id\n");
             UNSET_ONE_BIT((*hqP)->hqEnt->ue->ueId, (*hqP)->hqEnt->cell->boIndBitMap);

             /* Free the dl ded msg info allocated in macSchDlRlcBoInfo */
             if(!dciSlotAlloc->dlMsgPdschCfg)
             {
                SCH_FREE(dciSlotAlloc, sizeof(DlMsgSchInfo));
                (*hqP)->hqEnt->cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueCb->ueId -1] = NULL;
             }
             /*TRUE because this UE has nothing to be scheduled*/
             return accumalatedSize;
          }
       }

       /*[Step3]: Calculate Best FREE BLOCK with MAX PRB count*/
       maxFreePRB = searchLargestFreeBlock((*hqP)->hqEnt->cell, pdschTime, startPrb, DIR_DL);

       /*[Step4]: Estimation of PRB and BO which can be allocated to each LC in
        * the list based on RRM policy*/

       /*Either this UE contains no reservedPRB pool fir dedicated S-NSSAI or 
        * Num of Free PRB available is not enough to reserve Dedicated PRBs*/
       if(isRetx == FALSE)
       {
          if(maxFreePRB != 0)
          {
             mcsIdx = ueCb->ueCfg.dlModInfo.mcsIndex;

             if((fcfsHqProcCb->lcCb.dedLcList.count == NULLP) || ((maxFreePRB < rsvdDedicatedPRB)))
             { 
                fcfsHqProcCb->lcCb.sharedNumPrb = maxFreePRB;
                DU_LOG("\nDEBUG  -->  SCH : DL Only Default Slice is scheduled, sharedPRB Count:%d",\
                      fcfsHqProcCb->lcCb.sharedNumPrb);

                /*PRB Alloc for Default LCs*/
                prbAllocUsingRRMPolicy(&(fcfsHqProcCb->lcCb.defLcList), FALSE, mcsIdx, pdschNumSymbols,\
                      &(fcfsHqProcCb->lcCb.sharedNumPrb), NULLP, &isTxPayloadLenAdded, NULLP);
             }
             else
             {
                fcfsHqProcCb->lcCb.sharedNumPrb = maxFreePRB - rsvdDedicatedPRB;
                /*PRB Alloc for Dedicated LCs*/
                prbAllocUsingRRMPolicy(&(fcfsHqProcCb->lcCb.dedLcList), TRUE, mcsIdx, pdschNumSymbols,\
                      &(fcfsHqProcCb->lcCb.sharedNumPrb), &(rsvdDedicatedPRB), &isTxPayloadLenAdded, NULLP);

                /*PRB Alloc for Default LCs*/
                prbAllocUsingRRMPolicy(&(fcfsHqProcCb->lcCb.defLcList), FALSE, mcsIdx, pdschNumSymbols, \
                      &(fcfsHqProcCb->lcCb.sharedNumPrb), &(rsvdDedicatedPRB), &isTxPayloadLenAdded, NULLP);
             }
          }
       }

       /*[Step5]:Traverse each LCID in LcList to calculate the exact Scheduled Bytes
        * using allocated BO per LC and Update dlMsgAlloc BO report for MAC */
       if (isRetx == FALSE)
       {
          if(fcfsHqProcCb->lcCb.dedLcList.count != 0)
             updateGrantSizeForBoRpt(&(fcfsHqProcCb->lcCb.dedLcList), dciSlotAlloc, NULLP, &(accumalatedSize));

          updateGrantSizeForBoRpt(&(fcfsHqProcCb->lcCb.defLcList), dciSlotAlloc, NULLP, &(accumalatedSize));
       }
       else
       {
          accumalatedSize = (*hqP)->tbInfo[0].tbSzReq;
       }

       /*Below case will hit if NO LC(s) are allocated due to resource crunch*/
       if (!accumalatedSize)
       {
          if(maxFreePRB == 0)
          {
             DU_LOG("\nERROR  --> SCH : NO FREE PRB!!");
          }
          else
          {
             /*Schedule the LC for next slot*/
             DU_LOG("\nDEBUG  -->  SCH : No LC has been scheduled");
          }
          /* Not Freeing dlMsgAlloc as ZERO BO REPORT to be sent to RLC so that
           * Allocation can be done in next slot*/
          accumalatedSize = 0;
       }

       return accumalatedSize;
    }
    ```


- Explanation (summarized code):
    1. Loop for all logical channel
    ```c=1010
    for(lcIdx = 0; lcIdx < MAX_NUM_LC; lcIdx++)
    {
    ```
    2. Check if the current logical channel have buffer occupancy
    ```c=1012
    if(ueCb->dlInfo.dlLcCtxt[lcIdx].bo)
    {
    ```
    3. Check if LC is dedicated or default:
        1. If LC is dedicated, lcLL is the pointer to `fcfsHqProcCb->lcCb.dedLcList` (dedicated logical channel link list)
        ```c=1014
        /*Check the LC is Dedicated or default and accordingly LCList will
        * be used*/
        if(ueCb->dlInfo.dlLcCtxt[lcIdx].isDedicated)
        {
            lcLL = &(fcfsHqProcCb->lcCb.dedLcList);
            rsvdDedicatedPRB = ueCb->dlInfo.dlLcCtxt[lcIdx].rsvdDedicatedPRB;
        }
        ```
        2. If LC is not dedicated (default), lcLL is the pointer to `fcfsHqProcCb->lcCb.defLcList` (default logical channel link list)
        ```c=1021
        else
        {
            lcLL = &(fcfsHqProcCb->lcCb.defLcList);
        }
        ```
    4. Update the reqPRB and Payloadsize for this LC in the appropriate List
    ```c=1026
    /*[Step2]: Update the reqPRB and Payloadsize for this LC in the appropriate List*/
    if(updateLcListReqPRB(lcLL, ueCb->dlInfo.dlLcCtxt[lcIdx].lcId,\
        (ueCb->dlInfo.dlLcCtxt[lcIdx].bo + MAC_HDR_SIZE)) != ROK)
    {
    ```
    5. End loop for logical channel. Currently, all logical channel already divided into `dedLcList` or `defLcList` link list
    6. Check if there is no logical channel, then end function
    ```c=1041
    ...
    if ((fcfsHqProcCb->lcCb.defLcList.count == 0) && (fcfsHqProcCb->lcCb.dedLcList.count == 0))
    {
    ...
    ```
    ```c=1054
    ...
        return accumalatedSize;
    }
    ...
    ```
    7. Calculate Best FREE BLOCK with MAX PRB count
    ```c=
    /*[Step3]: Calculate Best FREE BLOCK with MAX PRB count*/
    maxFreePRB = searchLargestFreeBlock((*hqP)->hqEnt->cell, pdschTime, startPrb, DIR_DL);
    ```
    8. Check if maxFreePRB is not zero
    ```c=
    if(maxFreePRB != 0)
    {
    ```
    9. Check if this UE has no dedicated LC (dedLcList.count == 0 or NULLP) OR the maxFreePRB is lower than the specified reserved dedicated PRB
        1. If yes, schedule default slice only
        ```c=1072
        if((fcfsHqProcCb->lcCb.dedLcList.count == NULLP) || ((maxFreePRB < rsvdDedicatedPRB)))
        { 
            fcfsHqProcCb->lcCb.sharedNumPrb = maxFreePRB;
            DU_LOG("\nDEBUG  -->  SCH : DL Only Default Slice is scheduled, sharedPRB Count:%d",\
                fcfsHqProcCb->lcCb.sharedNumPrb);

            /*PRB Alloc for Default LCs*/
            prbAllocUsingRRMPolicy(&(fcfsHqProcCb->lcCb.defLcList), FALSE, mcsIdx, pdschNumSymbols,\
                &(fcfsHqProcCb->lcCb.sharedNumPrb), NULLP, &isTxPayloadLenAdded, NULLP);
        }
        ```
        2. If no, schedule dedicated slice first and after that schedule default slice
        ```c=
        else
        {
            fcfsHqProcCb->lcCb.sharedNumPrb = maxFreePRB - rsvdDedicatedPRB;
            /*PRB Alloc for Dedicated LCs*/
            prbAllocUsingRRMPolicy(&(fcfsHqProcCb->lcCb.dedLcList), TRUE, mcsIdx, pdschNumSymbols,\
                &(fcfsHqProcCb->lcCb.sharedNumPrb), &(rsvdDedicatedPRB), &isTxPayloadLenAdded, NULLP);

            /*PRB Alloc for Default LCs*/
            prbAllocUsingRRMPolicy(&(fcfsHqProcCb->lcCb.defLcList), FALSE, mcsIdx, pdschNumSymbols, \
                &(fcfsHqProcCb->lcCb.sharedNumPrb), &(rsvdDedicatedPRB), &isTxPayloadLenAdded, NULLP);
        }
        ```
    10. The rest of the code is for metrics reporting and logging



### 5.3. `updateLcListReqPRB`


- File location = `l2/src/5gnrsch/sch_utils.c`

- Code Snippet (Full code):
spoiler
```c=1486
/**************************************************************************
 *
 * @brief Update ReqPRB for a partiular LCID in LC Linklist 
 *
 * @details
 *
 *    Function : updateLcListReqPRB
 *
 *    Functionality:
 *     Update ReqPRB for a partiular LCID in LC Linklist 
 *
 * @params[in] I/P > lcLinkList pointer (LcInfo list)
 *             I/P > lcId
 *             I/P > reqPRB
 *             I/P > payloadSize
 *
 * @return ROK/RFAILED
 *
 * ***********************************************************************/
uint8_t updateLcListReqPRB(CmLListCp *lcLL, uint8_t lcId, uint32_t payloadSize)
{
   LcInfo    *lcNode = NULLP;
   lcNode = handleLcLList(lcLL, lcId, CREATE);

   if(lcNode == NULLP)
   {
      DU_LOG("\nERROR  -->  SCH : LC is neither present nor able to create in List lcId:%d",lcId);
      return RFAILED;
   }

   lcNode->reqBO = payloadSize;
   lcNode->allocBO = 0;
   lcNode->allocPRB = 0; /*Re-Initializing the AllocPRB*/
   return ROK;
}
```


- Explanation (summarized code):
    - This function purpose is just to update reqPRB size for a particular LCID in LC link list if link list already exist and node with the current LC ID already exist
    - If LC Link List does not exist, then it will create the link list and create the first node
    - If LC link list exist but node with LC ID does not exist, then create the node
    1. Call `handleLcLList` function (which create the LL if LL not exist and create the node if node not exist)
    ```c=1507
    LcInfo    *lcNode = NULLP;
    lcNode = handleLcLList(lcLL, lcId, CREATE);
    ...
    ```
    2. Update reqPRB size for the LCID
    ```c=1515
    ...
    lcNode->reqBO = payloadSize;
    lcNode->allocBO = 0;
    lcNode->allocPRB = 0; /*Re-Initializing the AllocPRB*/
    ...
    ```


### 5.4. `schFcfsScheduleSlot`


- File location = `l2/src/5gnrsch/sch_fcfs.c`

- Code Snippet (Full code):
spoiler
```c=1130
/*******************************************************************
 *
 * @brief Scheduling of Slots in UL And DL 
 *
 * @details
 *
 *    Function : schFcfsScheduleSlot
 *
 *    Functionality: Scheduling of slots in UL and DL specific to 
 *       FCFS scheduling
 *
 * @params[in] Pointer to Cell
 *             Slot timing info
 *             Scheduler instance
 * @return void
 *
 * ****************************************************************/
void schFcfsScheduleSlot(SchCellCb *cell, SlotTimingInfo *slotInd, Inst schInst)
{
   SchFcfsCellCb  *fcfsCell;
   SchFcfsUeCb    *fcfsUeCb;
   SchDlHqProcCb  *hqP = NULLP;
   SchUlHqProcCb  *ulHqP = NULLP;
   CmLList        *pendingUeNode;
   CmLList        *node;
   uint8_t        ueId, ueCount = 0;
   bool           isRarPending = false, isRarScheduled = false;
   bool           isMsg4Pending = false, isMsg4Scheduled = false;
   bool           isDlMsgPending = false, isDlMsgScheduled = false;
   bool           isUlGrantPending = false, isUlGrantScheduled = false;

   fcfsCell = (SchFcfsCellCb *)cell->schSpcCell;

   /* Select first UE in the linked list to be scheduled next */
   pendingUeNode = fcfsCell->ueToBeScheduled.first;
   ueCount = fcfsCell->ueToBeScheduled.count;

   while(pendingUeNode && ueCount > 0)
   {
      /*Since Multi-UE perTTI is not supported, re-init following parameters.*/
      isRarPending = false; isRarScheduled = false;
      isMsg4Pending = false; isMsg4Scheduled = false;
      isDlMsgPending = false; isDlMsgScheduled = false;
      isUlGrantPending = false; isUlGrantScheduled = false;
      if(pendingUeNode->node)
      {
         ueId = *(uint8_t *)(pendingUeNode->node);
         fcfsUeCb = (SchFcfsUeCb *)cell->ueCb[ueId-1].schSpcUeCb;

         /* If RAR is pending for this UE, schedule PDCCH,PDSCH to send RAR and 
          * PUSCH to receive MSG3 as per k0-k2 configuration*/
         if(cell->raReq[ueId-1] != NULLP)
         {
            isRarPending = true;
            isRarScheduled = schProcessRaReq(schInst, cell, *slotInd, ueId);
         }

         /*MSG3 retransmisson*/
         if(cell->raCb[ueId-1].retxMsg3HqProc)
         {            
            schMsg3RetxSchedulingForUe(&(cell->raCb[ueId-1]));
         }

         /* If MSG4 is pending for this UE, schedule PDCCH,PDSCH to send MSG4 and
          * PUCCH to receive UL msg as per k0-k1 configuration  */
         if (cell->ueCb[ueId-1].retxMsg4HqProc) //should work from dlmap later tbd
         {
            /* Retransmission of MSG4 */
            isMsg4Pending = true;
            if(schProcessMsg4Req(cell, *slotInd, ueId, TRUE, &cell->ueCb[ueId-1].retxMsg4HqProc) == ROK)
               isMsg4Scheduled = true;
         }
         else
         {
            /* First transmission of MSG4 */
            if(cell->raCb[ueId-1].msg4recvd)
            {
               isMsg4Pending = true;
               if(schProcessMsg4Req(cell, *slotInd, ueId, FALSE, &cell->ueCb[ueId-1].msg4HqProc) == ROK)
                  isMsg4Scheduled = true;

               /* If MSG4 scheduling failed, free the newly assigned HARQ process */
               if(!isMsg4Scheduled)
                  schDlReleaseHqProcess(cell->ueCb[ueId-1].msg4HqProc);
            }
         }

         if(isRarPending || isMsg4Pending)
         {
            /* If RAR or MSG is successfully scheduled then
             * remove UE from linked list since no pending msgs for this UE */
            if(isRarScheduled || isMsg4Scheduled)
            {
               schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
            }
            /* If RAR/MSG4 is pending but couldnt be scheduled then,
             * put this UE at the end of linked list to be scheduled later */
            else
            {
               cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
            }
         }

#ifdef NR_DRX 
         if((cell->ueCb[ueId-1].ueDrxInfoPres == true) && (cell->ueCb[ueId-1].drxUeCb.drxDlUeActiveStatus != true))
         {
            if(pendingUeNode->node)
            {
               cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
            }
         }
         else 
#endif
         {

            /* DL Data */
            node = NULLP;
            if(fcfsUeCb)
               node = fcfsUeCb->hqRetxCb.dlRetxHqList.first;
            if(node != NULLP)
            {
               /* DL Data ReTransmisson */
               isDlMsgPending = true;
               isDlMsgScheduled = schFillBoGrantDlSchedInfo(cell, *slotInd, ueId, TRUE, ((SchDlHqProcCb**) &(node->node)));
               if(isDlMsgScheduled)
               {
#ifdef NR_DRX 
                  schDrxStopDlHqRetxTmr(cell, &cell->ueCb[ueId-1], ((SchDlHqProcCb**) &(node->node)));
#endif
                  schFcfsRemoveFrmDlHqRetxList(&cell->ueCb[ueId-1], node);
               }
            }
            else
            {
               /* DL Data new transmission */
               if((cell->boIndBitMap) & (1<<ueId))
               {
                  isDlMsgPending = true;               
                  isDlMsgScheduled = schFillBoGrantDlSchedInfo(cell, *slotInd, ueId, FALSE, &hqP);

                  /* If DL scheduling failed, free the newly assigned HARQ process */
                  if(!isDlMsgScheduled)
                     schDlReleaseHqProcess(hqP);
                  else
                  {
#ifdef NR_DRX
                     schHdlDrxInActvStrtTmr(cell, &cell->ueCb[ueId-1], gConfigInfo.gPhyDeltaDl + SCHED_DELTA);
#endif
                  }
               }
            }
         }
#ifdef NR_DRX 
         if((cell->ueCb[ueId-1].ueDrxInfoPres == true) && (cell->ueCb[ueId-1].drxUeCb.drxUlUeActiveStatus != true))
         {
            if(pendingUeNode->node)
            {
               cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
            }
         }
         else 
#endif
         {
            /* Scheduling of UL grant */
            node = NULLP;
            if(fcfsUeCb)
               node = fcfsUeCb->hqRetxCb.ulRetxHqList.first;
            if(node != NULLP)
            {
               /* UL Data ReTransmisson */
               isUlGrantPending = true;
               isUlGrantScheduled = schProcessSrOrBsrReq(cell, *slotInd, ueId, TRUE, (SchUlHqProcCb**) &(node->node));
               if(isUlGrantScheduled)
               {
#ifdef NR_DRX 
                  schDrxStopUlHqRetxTmr(cell, &cell->ueCb[ueId-1], ((SchUlHqProcCb**) &(node->node)));
#endif
                  schFcfsRemoveFrmUlHqRetxList(&cell->ueCb[ueId-1], node);
               }
            }
            else
            {
               /* UL Data new transmission */
               if(cell->ueCb[ueId-1].srRcvd || cell->ueCb[ueId-1].bsrRcvd)
               {
                  isUlGrantPending = true;
                  isUlGrantScheduled = schProcessSrOrBsrReq(cell, *slotInd, ueId, FALSE, &ulHqP);
                  if(!isUlGrantScheduled)
                     schUlReleaseHqProcess(ulHqP, FALSE);
                  else
                  {
#ifdef NR_DRX
                     schHdlDrxInActvStrtTmr(cell, &cell->ueCb[ueId-1], gConfigInfo.gPhyDeltaUl + SCHED_DELTA);
#endif
                  }
               }
            }

            if(isUlGrantPending || isDlMsgPending)
            {
               if((isUlGrantPending && !isUlGrantScheduled) || (isDlMsgPending && !isDlMsgScheduled))
               {
                  cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
               }
               else
               {
                  schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
               }
            }
         }
      }
      if(!isUlGrantPending && !isDlMsgPending && !isRarPending && !isMsg4Pending)
      {
         DU_LOG("\nERROR  -->  SCH: In SchFcfsScheduleSlot, UE:%d is wrongly queued\
               in Pending UE List without any actions, Removing the UE from the list",ueId);
         schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
      }
      if(cell->schDlSlotInfo[slotInd->slot]->prbAlloc.numPrbAlloc >= MAX_NUM_RB)
      {
         DU_LOG("\nINFO   -->  SCH: No PRB available to proceed with next UE");
         return;     
      }
      pendingUeNode = fcfsCell->ueToBeScheduled.first;
      ueCount--;
   }
}
```


- Explanation (summarized code):
    1. From `ueToBeScheduled` Link List, get the first UE
    ```c=1162
    ...
    /* Select first UE in the linked list to be scheduled next */
    pendingUeNode = fcfsCell->ueToBeScheduled.first;
    ueCount = fcfsCell->ueToBeScheduled.count;
    ...
    ```
    2. If RAR is pending, schedule to process RA Request
    ```c=1179
    /* If RAR is pending for this UE, schedule PDCCH,PDSCH to send RAR and 
    * PUSCH to receive MSG3 as per k0-k2 configuration*/
    if(cell->raReq[ueId-1] != NULLP)
    {
        isRarPending = true;
        isRarScheduled = schProcessRaReq(schInst, cell, *slotInd, ueId);
    }
    ```
    3. If MSG3 Retransmission is needed, then schedule
    ```c=1187
    /*MSG3 retransmisson*/
    if(cell->raCb[ueId-1].retxMsg3HqProc)
    {            
        schMsg3RetxSchedulingForUe(&(cell->raCb[ueId-1]));
    }
    ```
    4. If MSG4 is pending, then schedule
    ```c=1206
    ...
    isMsg4Pending = true;
    if(schProcessMsg4Req(cell, *slotInd, ueId, FALSE, &cell->ueCb[ueId-1].msg4HqProc) == ROK)
        isMsg4Scheduled = true;
    ...
    ```
    5. Check if RAR or MSG4 is successfully scheduled. If yes, remove UE from `ueToBeScheduled` link list
    ```c=1222
    ...
    schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
    ...
    ```
    6. Schedule downlink data
    ```c=1266
    ...
    isDlMsgPending = true;               
    isDlMsgScheduled = schFillBoGrantDlSchedInfo(cell, *slotInd, ueId, FALSE, &hqP);
    ...
    ```
    7. Schedule uplink grant
    ```c=1314
    ...
    isUlGrantPending = true;
    isUlGrantScheduled = schProcessSrOrBsrReq(cell, *slotInd, ueId, FALSE, &ulHqP);
    ...
    ```
    8. Check if this UE has ULGrantPending or DLMsgPending. If yes, there are 2 options. If no, go to step 9
        1. If "ULGrantPending but ULGrant is not scheduled" or "DLMsgPending but DLMsg is not scheduled", move this UE to the back of `ueToBeScheduled` Link List
        ```c=1328
        if(isUlGrantPending || isDlMsgPending)
        {
            if((isUlGrantPending && !isUlGrantScheduled) || (isDlMsgPending && !isDlMsgScheduled))
            {
                cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
            }
        ...
        ```
        2. If ULGrant pending and scheduled and DLMsg pending and scheduled, then remove the UE from cell control block
        ```c=1333
        ...
        else
        {
            schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
        }
        ...
        ```
    9. Check if UE has no ULGrantPending, DLMsgPending, RARPending, and MSG4Pending. If yes, UE is wrongly queued and should be removed from cell control block
    ```c=1341
    if(!isUlGrantPending && !isDlMsgPending && !isRarPending && !isMsg4Pending)
    {
        DU_LOG("\nERROR  -->  SCH: In SchFcfsScheduleSlot, UE:%d is wrongly queued\
            in Pending UE List without any actions, Removing the UE from the list",ueId);
        schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
    }
    ```
    10. Check if Allocated PRB for this slot is already more than `MAX_NUM_RB`. If yes, then return because will not schedule the next UE
    ```c=1347
    if(cell->schDlSlotInfo[slotInd->slot]->prbAlloc.numPrbAlloc >= MAX_NUM_RB)
    {
        DU_LOG("\nINFO   -->  SCH: No PRB available to proceed with next UE");
        return;     
    }
    ```
    11. Move to next UE in `ueToBeScheduled` link list
    ```c=1352
    pendingUeNode = fcfsCell->ueToBeScheduled.first;
    ueCount--;
    ```
    