# 2024/08/20 Study Note (Analyze what are the available in OSC and OAI for RAN slicing)

###### tags: `2024`

**Goal:**
- [ ] Analyze what are the available in OSC and OAI for RAN slicing
    - [x] [what are the available in OAI CU for RAN slicing](#32-OAI-CU)
    - [x] [what are the available in OAI DU for RAN slicing](#33-OAI-DU)
    - [x] [what are the available in OSC DU High for RAN slicing](#4-OSC-DU-High)
    - [x] [what are the available in free5GC for network slicing](#5-free5GC)
    - [x] [what are the available in OAI UE for RAN slicing](#34-OAI-UE)
 
**References:**
- 3GPP TS 123 501
- [RAN Slicing Architecture Requirements](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240815%20Study%20Note%20(RAN%20Slicing%20Architecture%20Requirements).md)

**Table of Contents:**
- [2024/08/20 Study Note (Analyze what are the available in OSC and OAI for RAN slicing)](#2024-08-20-study-note--analyze-what-are-the-available-in-osc-and-oai-for-ran-slicing-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
  * [1. Template for Analysis](#1-template-for-analysis)
  * [2. Resources from Documentation](#2-resources-from-documentation)
  * [3. OAI](#3-oai)
    + [3.1. OAI gNB](#31-oai-gnb)
    + [3.2. OAI CU](#32-oai-cu)
      - [3.2.1. Network Slice Creation](#321-network-slice-creation)
      - [3.2.2. CU-UP Slice Selection for PDU Session](#322-cu-up-slice-selection-for-pdu-session)
    + [3.3. OAI DU](#33-oai-du)
      - [3.3.1. Network Slice Creation](#331-network-slice-creation)
      - [3.3.2. Network Slice Modification](#332-network-slice-modification)
      - [3.3.3. Slice Selection for DRB](#333-slice-selection-for-drb)
      - [3.3.4. Slice Aware Scheduling](#334-slice-aware-scheduling)
    + [3.4. OAI UE](#34-oai-ue)
      - [3.4.1. Network Slice Creation](#341-network-slice-creation)
      - [3.4.2. Slice Selection for PDU Session](#342-slice-selection-for-pdu-session)
  * [4. OSC DU High](#4-osc-du-high)
    + [4.1. Network Slice Creation](#41-network-slice-creation)
    + [4.2. Network Slice Modification](#42-network-slice-modification)
    + [4.3. Slice Selection for DRB](#43-slice-selection-for-drb)
    + [4.4. Slice Aware Scheduling](#44-slice-aware-scheduling)
  * [5. free5GC](#5-free5gc)
    + [5.1. Network Slice Selection Function](#51-network-slice-selection-function)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 0. Summary

abc

## 1. Template for Analysis


| Category                | Description                                                                                                                | Node Availability                                 | Notes                |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | -------------------- |
| Network Slice Lifecycle | Network Slice Creation<hr>Network Slice Termination<hr>Network Slice Modification<hr>Network Slice Supervision & Reporting | :heavy_check_mark:<hr>:x:<hr>:gear:<hr>:question: | NA<hr>NA<hr>NA<hr>NA |
| Network Slice Resource  | Slice Aware Admission Control<hr>Slice Aware Scheduling                                                                    | NA<hr>:heavy_check_mark:                          | NA<hr>NA             |


## 2. Resources from Documentation

- OAI

- OAI RAN
![image](https://hackmd.io/_uploads/B1I5RlXoC.png)
![image](https://hackmd.io/_uploads/S1M0JbQj0.png)
- OAI OAM
![image](https://hackmd.io/_uploads/Byu4yW7iA.png)
- FlexRIC
![image](https://hackmd.io/_uploads/B1Nb4ZQiA.png)


- OSC

- O-DU High
![image](https://hackmd.io/_uploads/Bk26ze7oR.png)
![image](https://hackmd.io/_uploads/By7nmeQjC.png)


- BubbleRAN

![image](https://hackmd.io/_uploads/ryGBm-QjR.png)
![image](https://hackmd.io/_uploads/ryL-Y-Xj0.png)


- Free5GC

- NSSF
![image](https://hackmd.io/_uploads/HkpeWGms0.png)


## 3. OAI

### 3.1. OAI gNB

- branch = `rebased-5g-nvs-rc-rf`


| Category                | Description                                                                                                                 | Node Availability                                                                                   |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Network Slice Lifecycle | Network Slice Creation<hr>Network Slice Termination<hr>Network Slice Modification<hr>Network Slice Supervision & Reporting  | :heavy_check_mark:<hr>:question:<hr>:question:<hr>:heavy_check_mark:                                |
| Network Slice Resource  | Slice Aware Admission Control<hr>Slice Aware Scheduling<hr>CU-UP Slice Selection for PDU Session<hr>Slice Selection for DRB | :heavy_check_mark: (DL)<hr>:heavy_check_mark: (DL)<hr>:heavy_check_mark:<hr>:heavy_check_mark: (DL) |


- branch = `develop`

| Category                | Description                                                                                                                 | Node Availability                                                         |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Network Slice Lifecycle | Network Slice Creation<hr>Network Slice Termination<hr>Network Slice Modification<hr>Network Slice Supervision & Reporting  | :heavy_check_mark:<hr>:question:<hr>:question:<hr>:heavy_check_mark: (E2) |
| Network Slice Resource  | Slice Aware Admission Control<hr>Slice Aware Scheduling<hr>CU-UP Slice Selection for PDU Session<hr>Slice Selection for DRB | :x:<hr>:x:<hr>:heavy_check_mark:<hr>:x:                                   |

### 3.2. OAI CU

- branch = `develop`
- More detailed [note](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240910%20Study%20Note%20(Define%20Current%20OAI%20CU%20Functions%20%26%20Parameters%20for%20RAN%20Slicing).md)

| Category                | Description                                                                                                                                                               | Node Availability                                                         |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Network Slice Lifecycle | [Network Slice Creation](#321-Network-Slice-Creation)<hr>Network Slice Termination<hr>Network Slice Modification<hr>Network Slice Supervision & Reporting                 | :heavy_check_mark:<hr>:question:<hr>:question:<hr>:heavy_check_mark: (E2) |
| Network Slice Resource  | Slice Aware Admission Control<hr>Slice Aware Scheduling<hr>[CU-UP Slice Selection for PDU Session](#322-CU-UP-Slice-Selection-for-PDU-Session)<hr>Slice Selection for DRB | :x:<hr>NA<hr>:heavy_check_mark:<hr>NA                                     |

#### 3.2.1. Network Slice Creation

- Network slice creation is to save Supported Slice in CU from Configuration file of integrated CU or CU-UP
- In OAI CU, network slice creation is done in `RB_INSERT(rrc_cuup_tree)`. Please see more detailed [note](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240910%20Study%20Note%20(Define%20Current%20OAI%20CU%20Functions%20%26%20Parameters%20for%20RAN%20Slicing).md#7-Slice-Configuration-gNBCU)

#### 3.2.2. CU-UP Slice Selection for PDU Session

- CU-UP Slice selection for DRB is to choose which CU-UP is this PDU Session of UE belong to. It is done in PDU Session Resource Setup procedure
- In OAI CU, CU-UP slice selection for PDU Session is done in `select_cuup_slice()`. Please see more detailed [note](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240910%20Study%20Note%20(Define%20Current%20OAI%20CU%20Functions%20%26%20Parameters%20for%20RAN%20Slicing).md#2-PDU-Session-Setup-Request-CU)



### 3.3. OAI DU

- branch = `develop`
- More detailed [note](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240926%20Study%20Note%20(Define%20Current%20OAI%20DU%20High%20Functions%20%26%20Parameters%20for%20RAN%20Slicing).md)

| Category                | Description                                                                                                                                               | Node Availability                                                         |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Network Slice Lifecycle | [Network Slice Creation](#331-Network-Slice-Creation)<hr>Network Slice Termination<hr>[Network Slice Modification](#332-Network-Slice-Modification)<hr>Network Slice Supervision & Reporting | :heavy_check_mark:<hr>:question:<hr>:x:<hr>:heavy_check_mark: (E2) |
| Network Slice Resource  | Slice Aware Admission Control<hr>[Slice Aware Scheduling](#334-Slice-Aware-Scheduling)<hr>CU-UP Slice Selection for PDU Session<hr>[Slice Selection for DRB](#333-Slice-Selection-for-DRB)                               | :x:<hr>:x:<hr>NA<hr>:x:                                                   |

#### 3.3.1. Network Slice Creation

- Network slice creation is to save Supported Slice in DU from Configuration file
- In OAI DU, network slice creation is done in `read_du_cell_info()`. Please see more detailed [note](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240926%20Study%20Note%20(Define%20Current%20OAI%20DU%20High%20Functions%20%26%20Parameters%20for%20RAN%20Slicing).md#2-Slice-Configuration)

#### 3.3.2. Network Slice Modification

- Network slice modification is to modify Supported Slice RRM Policy in DU from Configuration file or O1 interface
- In OAI DU High, network slice modification is not supported.

#### 3.3.3. Slice Selection for DRB

- Slice selection for DRB is to choose which slice is this DRB of UE belong to. It is done in UE Context Setup procedure
- In OAI DU High, slice selection for DRB is not supported yet. Please see more detailed [note](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240926%20Study%20Note%20(Define%20Current%20OAI%20DU%20High%20Functions%20%26%20Parameters%20for%20RAN%20Slicing).md#0-Summary)

#### 3.3.4. Slice Aware Scheduling

- Slice Aware scheduling for DRB is to allocate resource to DRB based on the slice this DRB of UE belong to. It is done in PRB Allocation
- In OAI DU High, Slice Aware Scheduling for DRB is not supported yet. Please see more detailed [note](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240926%20Study%20Note%20(Define%20Current%20OAI%20DU%20High%20Functions%20%26%20Parameters%20for%20RAN%20Slicing).md#0-Summary)



### 3.4. OAI UE

- branch = `develop`
- More detailed [note](https://hackmd.io/@superwilfrid/S19uIKw0A)

| Category                | Description                                                             | Node Availability  |
| ----------------------- | ----------------------------------------------------------------------- | ------------------ |
| Network Slice Lifecycle | [Network Slice Creation](#341-Network-Slice-Creation)                   | :heavy_check_mark: |
| Network Slice Resource  | [Slice Selection for PDU Session](#342-Slice-Selection-for-PDU-Session) | :heavy_check_mark: |

#### 3.4.1. Network Slice Creation

- Network slice creation is to save slice configuration from configuration file and Allowed Slice information from CN
- In OAI UE:
    - Saving configuration file slice setting is done in `init_uicc()`. Please see more detailed [note](https://hackmd.io/@superwilfrid/S19uIKw0A#1-Slice-Configuration)
    - Saving CN allowed slice is done in `parse_allowed_nssai()`. Please see more detailed [note](https://hackmd.io/@superwilfrid/S19uIKw0A#3-Allowed-Slice-from-CN)

#### 3.4.2. Slice Selection for PDU Session

- Slice Selection for PDU Session is to create PDU session based on the slice setting and CN Allowed slice
- In OAI UE, network slice creation is done in `generatePduSessionEstablishRequest()`. Please see more detailed [note](https://hackmd.io/@superwilfrid/S19uIKw0A#3-Allowed-Slice-from-CN)

## 4. OSC DU High

- More detailed [note](https://hackmd.io/@superwilfrid/H1YP3tb60)

| Category                | Description                                                                                                                                                                                | Node Availability                                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| Network Slice Lifecycle | [Network Slice Creation](#41-Network-Slice-Creation)<hr>Network Slice Termination<hr>[Network Slice Modification](#42-Network-Slice-Modification)<hr>Network Slice Supervision & Reporting | :heavy_check_mark:<hr>:question:<hr>:heavy_check_mark:<hr>:heavy_check_mark: (O1) |
| Network Slice Resource  | Slice Aware Admission Control<hr>[Slice Aware Scheduling](#44-Slice-Aware-Scheduling)<hr>CU-UP Slice Selection for PDU Session<hr>[Slice Selection for DRB](#43-Slice-Selection-for-DRB)   | :x:<hr>:heavy_check_mark: (Limited)<hr>NA<hr>:heavy_check_mark:              |

### 4.1. Network Slice Creation

- Network slice creation is to save Supported Slice in DU from Configuration file or O1 interface
- In OSC DU High, network slice creation is done in `setCellParam()`. Please see more detailed [note](https://hackmd.io/@superwilfrid/H1YP3tb60#31-Before-F1-Setup)

### 4.2. Network Slice Modification

- Network slice modification is to modify Supported Slice RRM Policy in DU from Configuration file or O1 interface
- In OSC DU High, network slice modification is done in `setRrmPolicy()`. Please see more detailed [note](https://hackmd.io/@superwilfrid/H1YP3tb60#32-After-F1-Setup)

### 4.3. Slice Selection for DRB

- Slice selection for DRB is to choose which slice is this DRB of UE belong to. It is done in UE Context Setup procedure
- In OSC DU High, slice selection for DRB is done in `updateDedLcInfo()`. Please see more detailed [note](https://hackmd.io/@superwilfrid/H1YP3tb60#1-UE-Context-Setup-Request)

### 4.4. Slice Aware Scheduling

- Slice Aware scheduling for DRB is to allocate resource to DRB based on the slice this DRB of UE belong to. It is done in PRB Allocation
- In OSC DU High, Slice Aware Scheduling for DRB is done in `prbAllocUsingRRMPolicy()`. Please see more detailed [note](https://hackmd.io/@superwilfrid/H1YP3tb60#5-Slice-RRM-Policy-Based-PRB-Allocation)


## 5. free5GC

| Description                             | Node Availability  |
| --------------------------------------- | ------------------ |
| [Network Slice Selection Function](#51-Network-Slice-Selection-Function) (NSSF) | :heavy_check_mark: |
| Network Slice Admission Control (NSACF) | :x:                |

### 5.1. Network Slice Selection Function

![image](https://hackmd.io/_uploads/HkV2H6dAA.png)


