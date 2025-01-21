# 2024/08/26 Study Note (Wilfrid's Thesis Problem Definition - RAN Slicing Architecture)

###### tags: `2024`

**Goal:**
- [ ] Write [research proposal](#12-Wilfrids-Research-Proposal)
- [ ] Write your [System Model/Architecture](#22-Wilfrids-System-ModelArchitecture)
- [ ] Summarize [experimental scenarios or numerical results](#32-Wilfrids-Scenarios-for-Experiments)

**References:**
- [Prof. Ray | Template for BMW Lab.](https://hackmd.io/@RayCheng/rJIuoWmB8)
- Sharing gNB components in RAN slicing: A perspective from 3GPP/NFV standards
- Orion: RAN Slicing for a Flexible and Cost-Effective Multi-Service Mobile Network Architecture

**Table of Contents:**
- [2024/08/26 Study Note (Wilfrid's Thesis Problem Definition - RAN Slicing Architecture)](#2024-08-26-study-note--wilfrid-s-thesis-problem-definition---ran-slicing-architecture-)
          + [tags: `2024`](#tags---2024-)
  * [1. Research Proposal](#1-research-proposal)
    + [1.1. Research Proposal Guide](#11-research-proposal-guide)
    + [1.2. Wilfrid's Research Proposal](#12-wilfrid-s-research-proposal)
  * [2. System Model/Architecture](#2-system-model-architecture)
    + [2.1. System Model/Architecture Guide](#21-system-model-architecture-guide)
    + [2.2. Wilfrid's System Model/Architecture](#22-wilfrid-s-system-model-architecture)
  * [3. Scenarios for your Experiments](#3-scenarios-for-your-experiments)
    + [3.1. Scenarios for your Experiments Guide](#31-scenarios-for-your-experiments-guide)
    + [3.2. Wilfrid's Scenarios for Experiments](#32-wilfrid-s-scenarios-for-experiments)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 1. Research Proposal

### 1.1. Research Proposal Guide
- **Contribution:** briefly summarize your key contribution
- **Background:** the problem you are dealing with; importance and challenge
- **Intended Outcome:** your expected outcome showing that the mentioned problem is solved
- **Application Design:** your proposed method
- **Findings:** observations from your experimental/numerical/simulation results

### 1.2. Wilfrid's Research Proposal

- **Contribution:**
    - Analyze End-to-End RAN Slicing System Architecture Requirements and available open source RAN slicing solutions (advantages, disadvantages, and missing functions)
    - ~~Develop multi L2 single L1 algorithm in OAI L1 nFAPI~~
    - ~~Compare single vs. multi vendor RAN slicing architecture performance~~
- **Background:**
    - There is no open source End-to-End Testing Platform for RAN Slicing
- **Intended Outcome:**
    - To provide a Proof of Concept of End-to-End RAN Slicing System Architecture and split 6 RU Sharing using open source solutions
- **Application Design:**
    - single vendor slices in one VNF in one machine (1)
    - multi vendor slices in multi VNF in one machine (2)
    - multi vendor slices in multi VNF in multi machines (3)
- **Findings:**
    - CPU usage is higher in 2 than 1
    - Processing time difference is not significant between 1 and 2
    - Memory footprint is higher in 2 than 1
- *Challenges*:
    - *Lack of support of RAN Slicing in open source solutions*
    - *Lack of integration in available open source solutions*

## 2. System Model/Architecture

### 2.1. System Model/Architecture Guide
Use figure(s) to show
- the network architrecture
- timing diagram
- key parameters
used in your system model

### 2.2. Wilfrid's System Model/Architecture
- System architecture:

![image](https://hackmd.io/_uploads/Bk76Td3s0.png)

- Key Parameters:
    - $P$ ($0≤P≤P_{CMAX}$): UL transmission power

- Flowchart:
    - TBD

## 3. Scenarios for your Experiments

### 3.1. Scenarios for your Experiments Guide
Each scenario is designed to support your contributions.
- Goal: Three categories: **Problem**, **Novelty** of your approach, and **Price paid** for your approach.
- Fig.: a sample of your figure (with well-defined X0Y axis)
- Expected Results: expected result of the figure

### 3.2. Wilfrid's Scenarios for Experiments

| Scenario | Goal                                       | Figures                                            | Expected Result                       |
| -------- | ------------------------------------------ | -------------------------------------------------- | ------------------------------------- |
| Sc. 1-1  | Problem of reqBO based resource scheduler  | ![image](https://hackmd.io/_uploads/rkRSatwtR.png) | more UE reach $P_{CMAX}$ for Tx Power |
| Sc. 1-2  | Problem of reqBO based resource scheduler  | ![image](https://hackmd.io/_uploads/ryofXcvKC.png) | lower UE Rx Power in gNB              |
| Sc. 2-1  | Novelty of PH based resource scheduler     | ![image](https://hackmd.io/_uploads/SySS0FwFC.png) | less UE reach $P_{CMAX}$ for Tx Power |
| Sc. 2-2  | Novelty of PH based resource scheduler     | ![image](https://hackmd.io/_uploads/HJqzEcvFC.png) | higher UE Rx Power in gNB             |
| Sc. 3    | Price paid for PH based resource scheduler | ![image](https://hackmd.io/_uploads/BynybvdFA.png) | less UE throughput                    |


