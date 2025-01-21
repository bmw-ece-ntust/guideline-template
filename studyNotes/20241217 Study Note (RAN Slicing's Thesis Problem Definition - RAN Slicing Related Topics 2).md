# 2024/12/17 Study Note (RAN Slicing's Thesis Problem Definition - RAN Slicing Related Topics 2)

###### tags: `2024`

**Goal:**
- [x] Write [research proposal for NSSI Resource Allocation Optimization](#1-Research-Proposal-1-NSSI-Resource-Allocation-Optimization)
- [x] Write [research proposal for Slice Aware Handover](#2-Research-Proposal-2-Development-of-Slice-Aware-Handover)

**References:**
- [Prof. Ray | Template for BMW Lab.](https://hackmd.io/@RayCheng/rJIuoWmB8)
- [Wilfrid's Thesis Problem Definition - RAN Slicing Architecture](https://hackmd.io/@superwilfrid/BJaSZiYjA)
- [RAN Slicing's Thesis Problem Definition - RAN Slicing Related Topics](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240829%20Study%20Note%20(RAN%20Slicing's%20Thesis%20Problem%20Definition%20-%20RAN%20Slicing%20Related%20Topics).md)
- [O-RAN.WG1.Use-Cases-Detailed-Specification](https://www.o-ran.org/specifications)
- [Network Slice Traffic Demand Prediction for Slice Mobility Management](https://ieeexplore.ieee.org/document/10463320)
- [Wilfrid | Handover Process in 5G NR](https://hackmd.io/@superwilfrid/S1b57bT4kx)

**Table of Contents:**
- [2024/12/17 Study Note (RAN Slicing's Thesis Problem Definition - RAN Slicing Related Topics 2)](#2024-12-17-study-note--ran-slicing-s-thesis-problem-definition---ran-slicing-related-topics-2-)
          + [tags: `2024`](#tags---2024-)
  * [1. Research Proposal 1 (NSSI Resource Allocation Optimization)](#1-research-proposal-1--nssi-resource-allocation-optimization-)
    + [1.1. Research Proposal](#11-research-proposal)
    + [1.2. System Model/Architecture](#12-system-model-architecture)
    + [1.3. Scenarios for Experiments](#13-scenarios-for-experiments)
  * [2. Research Proposal 2 (Development of Slice Aware Handover)](#2-research-proposal-2--development-of-slice-aware-handover-)
    + [2.1. Research Proposal](#21-research-proposal)
    + [2.2. System Model/Architecture](#22-system-model-architecture)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## 1. Research Proposal 1 (NSSI Resource Allocation Optimization)

### 1.1. Research Proposal

References:
1. O-RAN.WG1.Use-Cases-Detailed-Specification

- **Contribution:**
    - Develop O1 Performance Measurement collection module in OSC & OAI
    - Develop AI/ML Model to predict traffic demand patterns of NSSI at different times and location
- **Background:**
    - 5G services have different characteristics, the network traffic tends to be sporadic, where there can be different usage pattern in terms of time, location, UE distribution, and types of applications [1]
    - Need AI/ML model to predict the traffic demand patterns of 5G networks in different times and locations for each network slice, and automatically re-allocates the network resources ahead of the network issues surfaced. [1]
- **Intended Outcome:**
    - To provide optimized RRMPolicyRatio based on traffic demand patterns
- **Application Design:**
    - RAN Nodes (O-CU, O-DU, O-RU) report the PM via O1
    - Non-RT RIC collect PM related to NSSI resource usage from O-RAN Nodes via the O1 interface
    - Non-RT RIC train the AI/ML model based on historical PM, to predict traffic demand patterns of NSSI
    - Non-RT RIC reconfigure RRMPolicyRatio via O1
    - RAN Nodes (O-CU, O-DU, O-RU) update RRMPolicyRatio
- **Findings:**
    - For future study
- **Challenge:**
    - Limited number of PM supported by OAI & OSC DU High via O1 interface
    - No dataset for real traffic demand patterns for NSSI at different times & locations

### 1.2. System Model/Architecture
- System architecture:

![image](https://hackmd.io/_uploads/H18CyOAVke.png)

- Key Parameters (based on O-RAN.WG1.Use-Cases-Detailed-Specification): 

| O-RAN Node                          | Node | OSC DU High        | OAI `develop`           | OAI `slicing`           | Viavi RIC Test     |
| ----------------------------------- | ---- | ------------------ | ----------------------- | ----------------------- | ------------------ |
| RRU.PrbUsedDl<br>(.SNSSAI)          | O-DU | :x:                | :x:                     | :x:                     | :heavy_check_mark: |
| RRU.PrbUsedUl<br>(.SNSSAI)          | O-DU | :x:                | :x:                     | :x:                     | :heavy_check_mark: |
| DRB.UEThp.Dl<br>(.SNSSAI)           | O-DU | :x:                | :heavy_check_mark: (E2) | :heavy_check_mark: (E2) | :heavy_check_mark: |
| DRB.UEThp.Ul<br>(.SNSSAI)           | O-DU | :x:                | :heavy_check_mark: (E2) | :heavy_check_mark: (E2) | :heavy_check_mark: |
| SM.PDUSessionSetupReq<br>(.SNSSAI)  | O-CU | NA                 | :x:                     | :x:                     | :x:                |
| SM.PDUSessionSetupSucc<br>(.SNSSAI) | O-CU | NA                 | :x:                     | :x:                     | :x:                |
| DRB.UEThp.DlDist.Bin<br>(.SNSSAI)   | O-DU | :x:                | :x:                     | :x:                     | :x:                |
| DRB.UEThp.UlDist.Bin<br>(.SNSSAI)   | O-DU | :x:                | :x:                     | :x:                     | :x:                |
| DRB.EstabSucc<br>(.SNSSAI)          | O-CU | NA                 | :x:                     | :x:                     | :x:                |
| RRMPolicyRatio                      | O-DU | :heavy_check_mark: | :x:                     | :heavy_check_mark: (E2) | :heavy_check_mark: |

### 1.3. Scenarios for Experiments

![image](https://hackmd.io/_uploads/Hyv_D_A4ye.png)

## 2. Research Proposal 2 (Development of Slice Aware Handover)

### 2.1. Research Proposal

References:
1. Network Slice Traffic Demand Prediction for Slice Mobility Management

- **Contribution:**
    - Develop RAN Control (RC) E2SM (for Handover) and Cell Configuration & Control (CCC) E2SM (for slice config report) in OAI & OSC DU High
    - Develop slice mismatch detection and slice aware Handover (HO) xApp
- **Background:**
    - One of the predominant challenges inherent to 5G network slicing is the deployment of slices that are non-uniform across the network [1]
- **Intended Outcome:**
    - To handover UE to the correct slice
- **Application Design:**
    - Near-RT RIC collect Cell Configuration related to RAN node slice from the E2 interface (CCC xApp)
    - RAN Nodes (O-CU, O-DU) report UE Context RAN Parameters at Call Process Breakpoint (PDU Session resource setup & UE Context Setup)
    - Near-RT RIC detect slice mismatch between O-CU a and O-DU b for UE c (Slice Mismatch Detection xApp)
    - Near-RT RIC find the correct O-DU d for UE c and Control connected Mode Mobility (Slice Aware HO xApp & RAN Control xApp)
    - RAN Nodes (O-CU, O-DU) HO UE c to correct slice (O-DU d)
- **Findings:**
    - For future study
- **Challenge:**
    - Lack of support of RC E2SM in OAI & OSC DU High
    - Lack of support of CCC E2SM in OSC DU High


### 2.2. System Model/Architecture
- System architecture:

![image](https://hackmd.io/_uploads/HyPMzs0Eke.png)

- Key Functions:

| Function                      | Node        | OSC Near-RT RIC    | OSC DU High | OAI `develop`               | ORANSlice          | Viavi RIC Test                            |
| ----------------------------- | ----------- | ------------------ | ----------- | --------------------------- | ------------------ | ----------------------------------------- |
| CCC xApp                      | Near-RT RIC | :x:                | NA          | :x:                         | :heavy_check_mark: | NA                                        |
| RC xApp                       | Near-RT RIC | :heavy_check_mark: | NA          | :x:                         | :x:                | NA                                        |
| Slice Mismatch Detection xApp | Near-RT RIC | :x:                | NA          | :x:                         | :x:                | NA                                        |
| Slice Aware HO xApp           | Near-RT RIC | :x:                | NA          | :x:                         | :x:                | NA                                        |
| RC E2SM for PDU Session Slice | O-CU        | NA                 | NA          | :x:                         | :x:                | :question:                                |
| RC E2SM for F1 Handover       | O-CU        | NA                 | NA          | :heavy_check_mark: (telnet) | :x:                | :heavy_check_mark: (not sure F1 or Xn HO) |
| CCC E2SM for Node Slice       | O-CU        | NA                 | NA          | :x:                         | :heavy_check_mark: | :heavy_check_mark:                        |
| RC E2SM for DRB Slice         | O-DU        | NA                 | :x:         | :x:                         | :x:                | :question:                                |
| CCC E2SM for Node Slice       | O-DU        | NA                 | :x:         | :x:                         | :heavy_check_mark: | :heavy_check_mark:                        |




