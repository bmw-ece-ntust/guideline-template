# 2024/12/16 Study Note (Handover Process in 5G NR)

###### tags: `2024`

**Goal:**
- [ ] abc

**References:**
- [O-RAN Working Group 8 Base Station O-DU and O-CU Software Architecture and APIs](https://www.o-ran.org/specifications)
- [OSC O-DU High Overview](https://docs.o-ran-sc.org/projects/o-ran-sc-o-du-l2/en/latest/overview.html#inter-du-handover-within-o-cu)
- 3GPP TS 138 401 5G; NG-RAN; Architecture description 

**Table of Contents:**
- [2024/12/16 Study Note (Handover Process in 5G NR)](#2024-12-16-study-note--handover-process-in-5g-nr-)
          + [tags: `2024`](#tags---2024-)
  * [1. Summary](#1-summary)
  * [2. O-RAN.WG8.AAD Inter-O-DU Handover within an O-CU](#2-o-ranwg8aad-inter-o-du-handover-within-an-o-cu)
  * [3. OSC DU High Inter-DU Handover within O-CU](#3-osc-du-high-inter-du-handover-within-o-cu)
  * [4. TS 138 401 Inter-gNB-DU Mobility](#4-ts-138-401-inter-gnb-du-mobility)
  * [5. 3GPP TS 138 300 Mobility](#5-3gpp-ts-138-300-mobility)
  * [6. 3GPP TR 38 832](#6-3gpp-tr-38-832)
  * [7. ABC](#7-abc)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## 1. Summary

TBD

## 2. O-RAN.WG8.AAD Inter-O-DU Handover within an O-CU

![image](https://hackmd.io/_uploads/HJNKN-TEJl.png)
![image](https://hackmd.io/_uploads/BJBqE-6E1e.png)
![image](https://hackmd.io/_uploads/Hk9oVZpVke.png)

Assumption: UE is RRC connected with gNB and PDU data session is active.
- The UE sends Measurement Report message to the source gNB-DU. gNB-DU receives the UL Information Transfer message and it send the PUSCH message to the MAC/RLC layer.
- The UL RRC message transfer message is sent by RLC to F1AP over RLC-F1AP interface, later O-DU/F1AP layer sends the UL RRC Message Transfer to F1AP layer at O-CU to convey the UE Measurement Report message.
- F1AP later at O-CU sends the UL RRC Message Transfer message to RRC layer via PDCP interface.
- The gNB-CU makes an handover decision to the another cell belonging to the target gNB-DU.
- Optionally the gNB-CU may send an UE CONTEXT MODIFICATION REQUEST message to the source gNB-DU to query the latest configuration. Optionally the source gNB-DU sends the UE Reconfiguration Request to MAC/RLC layer and get the UE Reconfiguration Response from the respective protocol layers in case gNB-DU application does not store the latest configuration.
- The source gNB-DU responds with an UE CONTEXT MODIFICATION RESPONSE message that includes latest full configuration information.
- The gNB-CU sends an UE CONTEXT SETUP REQUEST message to the target gNB-DU to create an UE context and setup one or more data bearers. The UE CONTEXT SETUP REQUEST message includes HandoverPreparationInformation. The target gNB-DU sends the UE Create Request to the MAC and RLC layer to create the UE context with radio resources and receives UE Create Response from the respective protocol layers.
- The target gNB-DU responds with an UE CONTEXT SETUP RESPONSE message if the target gNB-DU can admit resources for the handover.
- The gNB-CU sends an UE CONTEXT MODIFICATION REQUEST message to the source gNB-DU, which includes RRCReconfiguration message towards the UE. The gNB-CU also indicates the source gNB-DU to stop the data transmission for the UE.
- The source gNB-DU sends the UE Reconfiguration Request to MAC/Scheduler and RLC layer and get the UE Reconfiguration Response from the respective protocol layers. 
- The source gNB-DU optionally sends a Downlink Data Delivery Status frame to inform the gNB-CU about the downlink data PDUs that could not be successfully transmitted to the UE.
- The source gNB-DU forwards the received RRCReconfiguration message to the UE.
- The source gNB-DU responds to the gNB-CU with UE CONTEXT MODIFICATION RESPONSE message.
- UE triggers Random Access procedure at the target gNB-DU.
- Optionally the target gNB-DU sends Downlink Data Delivery Status frame to the gNB-CU.
- The gNB-CU sends the target gNB-DU with the PDCP PDUs that are not successfully transmitted in the source gNB-DU.
- The UE responds to the target gNB-DU with an RRCReconfigurationComplete message.
- The target gNB-DU sends UL RRC MESSAGE TRANSFER message to the gNB-CU to convey the received RRCReconfigurationComplete message.
- The downlink and uplink data packets are sent to/from the UE through the target gNB-DU.
- The gNB-CU sends an UE CONTEXT RELEASE COMMAND message to the source gNB-DU.
- The source gNB-DU sends UE DELETE REQUEST to MAC/RLC layer to release the UE context in the respective protocol layer and receives UE DELETE RESPONSE message.
- The source gNB-DU responds the gNB-CU with an UE CONTEXT RELEASE COMPLETE message. 

## 3. OSC DU High Inter-DU Handover within O-CU

![image](https://hackmd.io/_uploads/HyOkDZ641l.png)

Assumption: UE is RRC connected with DU and PDU data session is active.
- The UE sends Measurement Report message to the source O-DU. This message is sent from O-DU to O-CU in the UL RRC MESSAGE TRANSFER message over F1AP interface.
- Based on UE Measurement Report, O-CU makes a handover decision to another cell belonging to the target O-DU.
- The O-CU sends a UE CONTEXT MODIFICATION REQUEST message to source O-DU to query the latest configuration.
- The DU APP in source O-DU responds with a UE CONTEXT MODIFICATION RESPONSE message that includes latest full configuration information.
- The O-CU sends a UE CONTEXT SETUP REQUEST message to the target O-DU to create an UE context and setup one or more data bearers. The UE CONTEXT SETUP REQUEST message includes Hand-overPreparationInformation. At target O-DU, DU APP sends UE Create Request to MAC and RLC layers to create the UE context with radio resources and receives UE Create Response from the respective protocol layers.
- The target O-DU responds with a UE CONTEXT SETUP RESPONSE message if the target O-DU can admit resources for the handover.
- The O-CU sends a UE CONTEXT MODIFICATION REQUEST message to the source O-DU, which includes RRCReconfiguration message towards the UE. The O-CU also indicates the source O-DU to stop the data transmission for the UE.
- The source O-DU forwards the received RRCReconfiguration message to the UE and then sends the UE Reconfiguration Request to MAC/Scheduler and RLC layer and get the UE Reconfiguration Response from the respective protocol layers.
- The source O-DU responds to the O-CU with UE CONTEXT MODIFICATION RESPONSE message.
- UE triggers Random Access procedure at the target O-DU. This is a contention free random access if UE was informed about its dedicated RACH resources in RRC Reconfiguration message.
- Once Random Access procedure with target O-DU is complete, the UE responds to the target O-DU with a RRCReconfigurationComplete message.
- The target O-DU sends UL RRC MESSAGE TRANSFER message to O-CU to convey the received RRCReconfigurationComplete message.
- The downlink and uplink data packets are sent to/from the UE through the target O-DU.
- The O-CU sends UE CONTEXT RELEASE COMMAND message to the source O-DU.
- The source O-DU sends UE DELETE REQUEST to MAC/RLC layers to release the UE context and receives UE DELETE RESPONSE message.
- The source O-DU responds to O-CU with UE CONTEXT RELEASE COMPLETE message.

## 4. TS 138 401 Inter-gNB-DU Mobility

![image](https://hackmd.io/_uploads/SJwN-Ga4kl.png)

1. The UE sends a MeasurementReport message to the source gNB-DU.
2. The source gNB-DU sends an UL RRC MESSAGE TRANSFER message to the gNB-CU to convey the received MeasurementReport message.
2a. The gNB-CU may send an UE CONTEXT MODIFICATION REQUEST message to the source gNB-DU to query the latest configuration.
2b. The source gNB-DU responds with an UE CONTEXT MODIFICATION RESPONSE message that includes full configuration information.
3. The gNB-CU sends an UE CONTEXT SETUP REQUEST message to the target gNB-DU to create an UE context and setup one or more data bearers. The UE CONTEXT SETUP REQUEST message includes a HandoverPreparationInformation. In case of NG-RAN sharing, the gNB-CU includes the serving PLMN ID (for SNPNs the serving SNPN ID).
4. The target gNB-DU responds to the gNB-CU with an UE CONTEXT SETUP RESPONSE message.
5. The gNB-CU sends a UE CONTEXT MODIFICATION REQUEST message to the source gNB-DU, which includes a generated RRCReconfiguration message and indicates to stop the data transmission for the UE. The source gNB-DU also sends a Downlink Data Delivery
6. The source gNB-DU forwards the received RRCReconfiguration message to the UE.
7. The source gNB-DU responds to the gNB-CU with the UE CONTEXT MODIFICATION RESPONSE message.
8. A Random Access procedure is performed at the target gNB-DU. The target gNB-DU sends a Downlink Data Delivery Status frame to inform the gNB-CU. Downlink packets, which may include PDCP PDUs not successfully transmitted in the source gNB-DU, are sent from the gNB-CU to the target gNB-DU. 
9. The UE responds to the target gNB-DU with an RRCReconfigurationComplete message.
10. The target gNB-DU sends an UL RRC MESSAGE TRANSFER message to the gNB-CU to convey the received RRCReconfigurationComplete message. Downlink packets are sent to the UE. Also, uplink packets are sent from the UE, which are forwarded to the gNB-CU through the target gNB-DU.
11. The gNB-CU sends an UE CONTEXT RELEASE COMMAND message to the source gNB-DU.
12. The source gNB-DU releases the UE context and responds the gNB-CU with an UE CONTEXT RELEASE COMPLETE message.

## 5. 3GPP TS 138 300 Mobility

To make mobility slice-aware in case of Network Slicing, S-NSSAI is introduced as part of the PDU session information that is transferred during mobility signalling. This enables slice-aware admission and congestion control.

**Both NG and Xn handovers are allowed regardless of the slice support of the target NG-RAN node i.e. even if the target NG-RAN node does not support the same slices as the source NG-RAN node.** An example for the case of connected mode mobility across different Registration Areas is shown in Figure 16.3.4.5-1 for the case of NG based handover and in Figure 16.3.4.5-2 for the case of Xn based handover. 
![image](https://hackmd.io/_uploads/HJivNGpEJx.png)
![image](https://hackmd.io/_uploads/rk6VVMp4Jl.png)

## 6. 3GPP TR 38 832 

![image](https://hackmd.io/_uploads/S1Qrdf641l.png)

- **Issue 1: The UE is unaware of the slices supported on different cells or frequencies, which prevents UE from (re)select to the cell or frequency supporting the intended slice.**
- Issue 2: Dedicated priorities would not be available to the UE prior to first RRC connection establishment and only remain valid before T320 expires upon entering IDLE mode. In addition, dedicated priorities are discarded each time when UE entering CONNECTED mode and need to be configured again before UE leaving CONNECTED mode. 
- Issue 3: Operator may require different frequency priority configurations for the specific slice in different areas, however the dedicated priority always overwrites the broadcast priorities if configured. 
- Issue 4: If the serving cell is unable to support the requested slices, the serving cell may need to perform handover to a cell supporting the requested slices or release the RRC connection. That may increase control plane signalling overhead as well as long control plane latency for the UE to access the network.

- Solution 1: Legacy dedicated priority via RRCRelease message.
- Solution 2: Rel-15 mechanisms such as HO, CA, DC and redirection can be used to access the intended slice in different cell.
- Solution 3: Slice related cell selection info, the slice info of serving cell and neighboring cells is provided in the system information or RRCRelease message.
- Solution 4: Slice related cell reselection info (e.g. Cell reselection priority per slice), the slice info of neighboring cells is provided in the system information or RRCRelease message.

## 7. ABC



