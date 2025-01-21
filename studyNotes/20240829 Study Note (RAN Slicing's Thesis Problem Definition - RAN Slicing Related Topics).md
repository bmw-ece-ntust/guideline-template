# 2024/08/29 Study Note (RAN Slicing's Thesis Problem Definition - RAN Slicing Related Topics)

###### tags: `2024`

**Goal:**
- [x] Write [research proposal related to RAN Slicing in DU](#1-Research-Proposal-1-Development-of-Slice-Specific-Scheduler-on-OSC-DU)
- [x] Write [research proposal related to RAN Slicing in CU](#2-Research-Proposal-2-Development-of-Slice-Admission-Control-on-OAI-CU)
- [x] Write [research proposal related to RAN Slicing in RIC](#3-Research-Proposal-3-Development-of-Slice-Aware-TSHO-xApprApp-in-RIC)

**References:**
- [Prof. Ray | Template for BMW Lab.](https://hackmd.io/@RayCheng/rJIuoWmB8)
- Sharing gNB components in RAN slicing: A perspective from 3GPP/NFV standards
- Orion: RAN Slicing for a Flexible and Cost-Effective Multi-Service Mobile Network Architecture
- [Wilfrid's Thesis Problem Definition - RAN Slicing Architecture](https://hackmd.io/@superwilfrid/BJaSZiYjA)

**Table of Contents:**
- [2024/08/29 Study Note (RAN Slicing's Thesis Problem Definition - RAN Slicing Related Topics)](#2024-08-29-study-note--ran-slicing-s-thesis-problem-definition---ran-slicing-related-topics-)
          + [tags: `2024`](#tags---2024-)
  * [1. Research Proposal 1 (Development of Slice Specific Scheduler on OSC DU)](#1-research-proposal-1--development-of-slice-specific-scheduler-on-osc-du-)
    + [1.1. Research Proposal](#11-research-proposal)
    + [1.2. System Model/Architecture](#12-system-model-architecture)
  * [2. Research Proposal 2 (Development of Slice Admission Control on OAI CU)](#2-research-proposal-2--development-of-slice-admission-control-on-oai-cu-)
    + [2.1. Research Proposal](#21-research-proposal)
    + [2.2. System Model/Architecture](#22-system-model-architecture)
  * [3. Research Proposal 3 (Development of Slice Aware TS/HO xApp/rApp in RIC)](#3-research-proposal-3--development-of-slice-aware-ts-ho-xapp-rapp-in-ric-)
    + [3.1. Research Proposal](#31-research-proposal)
    + [3.2. System Model/Architecture](#32-system-model-architecture)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## 1. Research Proposal 1 (Development of Slice Specific Scheduler on OSC DU)

### 1.1. Research Proposal
- **Contribution:**
    - Develop scheduling algorithm for eMBB (guarantee Throughput) and uRLLC (guarantee Delay)
    - Implement multi vendor slices DU environment and compare with single vendor slices environment
- **Background:**
    - OSC DU MAC scheduler is not slice requirement aware (only 1 algorithm)
- **Intended Outcome:**
    - To provide 2 scheduling algorithm based on slice requirement
- **Application Design:**
    - single vendor slices in one VNF in one machine (1)
    - multi vendor slices in multi VNF in one machine (2)
    - multi vendor slices in multi VNF in multi machines (3)
- **Findings:**
    - Algorithm effectiveness is upperbounded/limited by number of UE -> need Radio Admission Control
- *Challenge:*
    - *Lack of E2E environment for algorithm testing*
    - *Nobody test OSC DU for E2E*
    - *Lack of number of UE for real data*

### 1.2. System Model/Architecture
- System architecture:

![image](https://hackmd.io/_uploads/SyNQjOpo0.png)

- Key Parameters (based on O-RAN.WG2.A1TD):
    - $maxUlPacketDelayPerUe$: Maximum delay for UL packets in ms as the performance target in Slice SLA
    - $guaUlThptPerSlice$: Guaranteed data rate as kbps in uplink to be served by the network slice in Slice SLA

## 2. Research Proposal 2 (Development of Slice Admission Control on OAI CU)

### 2.1. Research Proposal

- **Contribution:**
    - Develop Radio Admission Control algorithm that is aware of slice requirement (guarantee Throughput or Delay)
    - Implement multi vendor slices CU environment and compare with single vendor slices environment
- **Background:**
    - OAI CU-UP does not have Radio Admission Control algorithm for network slice
- **Intended Outcome:**
    - To provide Radio Admission Control algorithm that is aware of slice requirement
- **Application Design:**
    - single vendor slices in one VNF in one machine (1)
    - multi vendor slices in multi VNF in one machine (2)
    - multi vendor slices in multi VNF in multi machines (3)
- **Findings:**
    - In multi vendor environment, there should be a way to handover UE to original requested slice -> Slice aware TS/HO xApp
    - Radio Admission Control can be placed in RIC
- *Challenge:*
    - *Lack of support of network slicing in L3 stack*
    - *Lack of open source integration solution*

### 2.2. System Model/Architecture
- System architecture:

![image](https://hackmd.io/_uploads/r1QnAdTj0.png)

- Key Parameters (based on O-RAN.WG2.A1TD):
    - $maxUlPacketDelayPerUe$: Maximum delay for UL packets in ms as the performance target in Slice SLA
    - $guaUlThptPerSlice$: Guaranteed data rate as kbps in uplink to be served by the network slice in Slice SLA
    - $maxNumberOfUes$: Maximum number of RRC connected UEs to be served by the network slice concurrently in Slice SLA

## 3. Research Proposal 3 (Development of Slice Aware TS/HO xApp/rApp in RIC)
### 3.1. Research Proposal
- **Contribution:**
    - Develop Slice Aware TS/HO xApp/rApp in RIC
    - Develop Xn handover procedure in OAI CU
- **Background:**
    - RIC does not have slice aware TS/HO xApp/rApp
- **Intended Outcome:**
    - To provide TS/HO xApp/rApp that is aware of slice availability in gNB
- **Application Design:**
    - single vendor slices in one VNF in one machine (1)
    - multi vendor slices in multi VNF in one machine (2)
    - multi vendor slices in multi VNF in multi machines (3)
- **Findings:**
    - To Be Added
- *Challenge:*
    - *Lack of open source environment for testing*
    - *Lack of Xn handover function in OAI gNB*

### 3.2. System Model/Architecture
- System architecture:

![image](https://hackmd.io/_uploads/H1hcgKpiR.png)

- Key Parameters (based on O-RAN.WG2.A1TD):
    - $maxUlPacketDelayPerUe$: Maximum delay for UL packets in ms as the performance target in Slice SLA
    - $guaUlThptPerSlice$: Guaranteed data rate as kbps in uplink to be served by the network slice in Slice SLA
    - $maxNumberOfUes$: Maximum number of RRC connected UEs to be served by the network slice concurrently in Slice SLA


