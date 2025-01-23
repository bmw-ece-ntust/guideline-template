# 2024/08/13 Study Note (What parameters need to get from DU)

###### tags: `2024`


**Goal:**
- Q: What parameters need to get from DU? 
- A: **NSSAI** in
    - [x] [RRC Setup Complete in Registration Request](#1-RRC-Setup-Complete-in-Registration-Request)
    - [x] [RRC Setup Complete in UL RRC Message Transfer](#2-RRC-Setup-Complete-in-UL-RRC-Message-Transfer)
    - [x] [UE Context Setup Request](#3-UE-Context-Setup-Request)
    - [x] [UE Context Setup Response](#4-UE-Context-Setup-Response)
    - [x] [RRC Reconfig in DL RRC Message Transfer](#5-RRC-Reconfig-in-DL-RRC-Message-Transfer)



 
**References:**
- 3GPP TS 38.401 | 5G NG-RAN Architecture Description
- Event Helix | 5G Standalone Access Registration
- Event Helix | 5G Standalone Access Registration Signaling Messages
- 3GPP TS 38.473
- 3GPP TS 38.331
- 3GPP TS 24.501 | 5G NAS Protocol
- [ShareTechnote | 5G/NR  - Network Slicing](https://www.sharetechnote.com/html/5G/5G_NetworkSlicing.html)




## 0. Summary

![image](https://hackmd.io/_uploads/HyGVB5t5C.png)

- [2024/08/13 Study Note (What parameters need to get from DU)](#2024-08-13-study-note--what-parameters-need-to-get-from-du-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
  * [1. RRC Setup Complete in Registration Request](#1-rrc-setup-complete-in-registration-request)
    + [1.1. Registration Request NAS Message](#11-registration-request-nas-message)
      - [1.1.1. Fields](#111-fields)
      - [1.1.2. Example](#112-example)
    + [1.2. RRC Setup Complete](#12-rrc-setup-complete)
      - [1.2.1. Fields](#121-fields)
      - [1.2.2. Example](#122-example)
  * [2. RRC Setup Complete in UL RRC Message Transfer](#2-rrc-setup-complete-in-ul-rrc-message-transfer)
    + [2.1. Fields](#21-fields)
    + [2.2. Example](#22-example)
  * [3. UE Context Setup Request](#3-ue-context-setup-request)
    + [3.1. Fields](#31-fields)
    + [3.2. Example](#32-example)
  * [4. UE Context Setup Response](#4-ue-context-setup-response)
    + [4.1. Fields](#41-fields)
    + [4.2. Example](#42-example)
  * [5. RRC Reconfig in DL RRC Message Transfer](#5-rrc-reconfig-in-dl-rrc-message-transfer)
    + [5.1. RRC Reconfig](#51-rrc-reconfig)
      - [5.1.1. Fields](#511-fields)
      - [5.1.2. Example](#512-example)
    + [5.2. DL RRC Message Transfer](#52-dl-rrc-message-transfer)
      - [5.2.1. Fields](#521-fields)
      - [5.2.2. Example](#522-example)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 1. RRC Setup Complete in Registration Request

### 1.1. Registration Request NAS Message

| Point       | Detail                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| Description | The Registration Request NAS message is carried from the UE to the newly assigned AMF                                    |
| Reference   | TS 24.501                                                                                                                |
| Flow        | UE ➔ RRCSetupComplete ➔ gNB ➔ NGAP Initial UE Message ➔ New AMF ➔ Namf_Communication_UEContextTransfer Request ➔ Old AMF |
| Fields      | [See below](#111-Fields)                                                                                                 |

#### 1.1.1. Fields

| Field                                     | Type  |
| ----------------------------------------- | ----- |
| Requested NSSAI                           | NSSAI |
| Other unrelated fields to Network Slicing | -      |


#### 1.1.2. Example


Protocol discriminator = 0x7e (5GS Mobility Management)

Security header = 0x1 (Integrity protected)

Auth code = 0xc2653407

Sequence number = 0x09

Protocol discriminator = 0x7e (5GS Mobility Management)

Security header = 0x0 (Plain 5GS NAS message, not security protected)

**Message type = 0x41 (Registration request)**

5GS registration type:

  Follow-on request bit = 1

  Value = 1 (initial registration)

ngKSI:

  TSC = 0

  NAS key set identifier = 2

5GS mobile identity:

  5G-GUTI

    MCC = 001

    MNC = 01

    AMF Region ID = 128

    AMF Set ID = 4

    AMF Pointer = 1

    5G-TMSI = 0x2fedc8c7

UE security capability:

  0xf0 (5G-EA0=1, 128-5G-EA1=1, 128-5G-EA2=1, 128-5G-EA3=1, 5G-EA4=0, 5G-EA5=0, 5G-EA6=0, 5G-EA7=0)

  0x70 (5G-IA0=0, 128-5G-IA1=1, 128-5G-IA2=1, 128-5G-IA3=1, 5G-IA4=0, 5G-IA5=0, 5G-IA6=0, 5G-IA7=0)

  0xf0 (EEA0=1, 128-EEA1=1, 128-EEA2=1, 128-EEA3=1, EEA4=0, EEA5=0, EEA6=0, EEA7=0)

  0x70 (EIA0=0, 128-EIA1=1, 128-EIA2=1, 128-EIA3=1, EIA4=0, EIA5=0, EIA6=0, EIA7=0)

NAS message container:

  Protocol discriminator = 0x7e (5GS Mobility Management)

  Security header = 0x0 (Plain 5GS NAS message, not security protected)

  Message type = 0x41 (Registration request)

  5GS registration type:

    Follow-on request bit = 1

    Value = 1 (initial registration)

  ngKSI:

    TSC = 0

    NAS key set identifier = 2

  5GS mobile identity:

    5G-GUTI

      MCC = 001

      MNC = 01

      AMF Region ID = 128

      AMF Set ID = 4

      AMF Pointer = 1

      5G-TMSI = 0x2fedc8c7

  5GMM capability:

    0x03 (SGC=0, 5G-IPHC-CP CIoT=0, N3 data=0, 5G-CP CIoT=0, RestrictEC=0, LPP=0, HO attach=1, S1 mode=1)

  UE security capability:

    0xf0 (5G-EA0=1, 128-5G-EA1=1, 128-5G-EA2=1, 128-5G-EA3=1, 5G-EA4=0, 5G-EA5=0, 5G-EA6=0, 5G-EA7=0)

    0x70 (5G-IA0=0, 128-5G-IA1=1, 128-5G-IA2=1, 128-5G-IA3=1, 5G-IA4=0, 5G-IA5=0, 5G-IA6=0, 5G-IA7=0)

    0xf0 (EEA0=1, 128-EEA1=1, 128-EEA2=1, 128-EEA3=1, EEA4=0, EEA5=0, EEA6=0, EEA7=0)

    0x70 (EIA0=0, 128-EIA1=1, 128-EIA2=1, 128-EIA3=1, EIA4=0, EIA5=0, EIA6=0, EIA7=0)

  **Requested NSSAI:   // UE may configure multiple S-NSSAI here**

    **S-NSSAI**

      **Length of S-NSSAI contents = 1 (SST)**

      **SST = 0x01**

  Last visited registered TAI:

    MCC = 001

    MNC = 01

    TAC = 0x000064

  S1 UE network capability:

    0xf0 (EEA0=1, 128-EEA1=1, 128-EEA2=1, 128-EEA3=1, EEA4=0, EEA5=0, EEA6=0, EEA7=0)

    0x70 (EIA0=0, 128-EIA1=1, 128-EIA2=1, 128-EIA3=1, EIA4=0, EIA5=0, EIA6=0, EIA7=0)

    0xc0 (UEA0=1, UEA1=1, UEA2=0, UEA3=0, UEA4=0, UEA5=0, UEA6=0, UEA7=0)

    0x40 (UCS2=0, UIA1=1, UIA2=0, UIA3=0, UIA4=0, UIA5=0, UIA6=0, UIA7=0)

    0x19 (ProSe-dd=0, ProSe=0, H.245-ASH=0, ACC-CSFB=1, LPP=1, LCS=0, 1xSRVCC=0, NF=1)

    0x80 (ePCO=1, HC-CP CIoT=0, ERw/oPDN=0, S1-U data=0, UP CIoT=0, CP CIoT=0, ProSe-relay=0, ProSe-dc=0)

    0xb0 (15 bearers=1, SGC=0, N1mode=1, DCNR=1, CP backoff=0, RestrictEC=0, V2X PC5=0, multipleDRB=0)

  UE's usage setting = 0x01 (Data centric)

  LADN indication:

    Length = 0

    Data =

  **Network slicing indication = 0x00 (DCNI=0, NSSCI=0)**

  5GS update type = 0x01 (EPS-PNB-CIoT=no additional information, 5GS-PNB-CIoT=no additional information, NG-RAN-RCU=0, SMS requested=1)



### 1.2. RRC Setup Complete

| Point       | Detail                                                                                                     |
| ----------- | ---------------------------------------------------------------------------------------------------------- |
| Description | The UE sends the RRC Setup Complete message with a Registration Request in the dedicatedNAS-Message field. |
| Reference   | TS 38.331                                                                                                 |
| Flow        | UE ➔ gNB                                                                                              |
| Fields      | [See below](#121-Fields)                                                                                   |

#### 1.2.1. Fields

| Field                                     | Type                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| s-NSSAI-List                              | sNSSAI                                                       |
| dedicatedNAS-Message                      | [Registration Request](#11-Registration-Request-NAS-Message) |
| Other unrelated fields to Network Slicing | -                                                            |


#### 1.2.2. Example

- Got no example

## 2. RRC Setup Complete in UL RRC Message Transfer

| Point       | Detail                                                                                                 |
| ----------- | ------------------------------------------------------------------------------------------------------ |
| Description | This message is sent by the gNB-DU to transfer the layer 3 message to the gNB-CU over the F1 interface |
| Reference   | TS 38.473                                                                                              |
| Flow        | gNB-DU → gNB-CU                                                                                              |
| Fields      | [See below](#21-Fields)                                                                               |

### 2.1. Fields

| Field                                     | Type                                       |
| ----------------------------------------- | ------------------------------------------ |
| Message Type                              |                                            |
| gNB-CU UE F1AP ID                         |                                            |
| gNB-DU UE F1AP ID                         |                                            |
| RRC-Container                             | [RRCSetupComplete](#12-RRC-Setup-Complete) |
| Selected PLMN ID                          |                                            |
| Other unrelated fields to Network Slicing | -                                          |



### 2.2. Example

- Got no example


## 3. UE Context Setup Request

| Point       | Detail                                                     |
| ----------- | ---------------------------------------------------------- |
| Description | is sent by the gNB-CU to request the setup of a UE context |
| Reference   | TS 38.473                                                  |
| Flow        | gNB-CU → gNB-DU                                                  |
| Fields      | [See below](#31-Fields)                                    |

### 3.1. Fields

| Field                                     | Type                 |
| ----------------------------------------- | -------------------- |
| Message Type                              |                      |
| gNB-CU UE F1AP ID                         |                      |
| gNB-DU UE F1AP ID                         |                      |
| DRB to Be Setup List                      | DRB to Be Setup Item |
| Other unrelated fields to Network Slicing | -                    |


**DRB to Be Setup Item contains NSSAI for each DRB**



### 3.2. Example


```xml=0
<F1AP-PDU>
    <initiatingMessage>
        <procedureCode>5</procedureCode>
        <criticality><reject/></criticality>
        <value>
            <UEContextSetupRequest>
                <protocolIEs>
                    <UEContextSetupRequestIEs>
                        <id>40</id>
                        <criticality><reject/></criticality>
                        <value>
                            <GNB-CU-UE-F1AP-ID>1</GNB-CU-UE-F1AP-ID>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>41</id>
                        <criticality><ignore/></criticality>
                        <value>
                            <GNB-DU-UE-F1AP-ID>1</GNB-DU-UE-F1AP-ID>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>63</id>
                        <criticality><reject/></criticality>
                        <value>
                            <NRCGI>
                                <pLMN-Identity>13 F1 84</pLMN-Identity>
                                <nRCellIdentity>
                                    000000000000000000000000000000000001
                                </nRCellIdentity>
                            </NRCGI>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>107</id>
                        <criticality><reject/></criticality>
                        <value>
                            <ServCellIndex>0</ServCellIndex>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>96</id>
                        <criticality><ignore/></criticality>
                        <value>
                            <CellULConfigured><none/></CellULConfigured>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>9</id>
                        <criticality><reject/></criticality>
                        <value>
                            <CUtoDURRCInformation>
                                <uE-CapabilityRAT-ContainerList>
                                    10 22 02 00 00 00 00 00 00 00 00 00 A0 01 03 80 
                                    00 81 B0
                                </uE-CapabilityRAT-ContainerList>
                            </CUtoDURRCInformation>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>54</id>
                        <criticality><ignore/></criticality>
                        <value>
                            <SCell-ToBeSetup-List>
                                <ProtocolIE-SingleContainer>
                                    <id>53</id>
                                    <criticality><ignore/></criticality>
                                    <value>
                                        <SCell-ToBeSetup-Item>
                                            <sCell-ID>
                                                <pLMN-Identity>13 F1 84</pLMN-Identity>
                                                <nRCellIdentity>
                                                    000000000000000000000000000000000001
                                                </nRCellIdentity>
                                            </sCell-ID>
                                            <sCellIndex>1</sCellIndex>
                                        </SCell-ToBeSetup-Item>
                                    </value>
                                </ProtocolIE-SingleContainer>
                            </SCell-ToBeSetup-List>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>74</id>
                        <criticality><reject/></criticality>
                        <value>
                            <SRBs-ToBeSetup-List>
                                <ProtocolIE-SingleContainer>
                                    <id>73</id>
                                    <criticality><ignore/></criticality>
                                    <value>
                                        <SRBs-ToBeSetup-Item>
                                            <sRBID>2</sRBID>
                                        </SRBs-ToBeSetup-Item>
                                    </value>
                                </ProtocolIE-SingleContainer>
                            </SRBs-ToBeSetup-List>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>35</id>
                        <criticality><reject/></criticality>
                        <value>
                            <DRBs-ToBeSetup-List>
                                <ProtocolIE-SingleContainer>
                                    <id>34</id>
                                    <criticality><ignore/></criticality>
                                    <value>
                                        <DRBs-ToBeSetup-Item>
                                            <dRBID>1</dRBID>
                                            <qoSInformation>
                                                <choice-extension>
                                                    <id>164</id>
                                                    <criticality><ignore/></criticality>
                                                    <value>
                                                        <DRB-Information>
                                                            <dRB-QoS>
                                                                <qoS-Characteristics>
                                                                    <non-Dynamic-5QI>
                                                                        <fiveQI>9</fiveQI>
                                                                        <averagingWindow>0</averagingWindow>
                                                                        <maxDataBurstVolume>0</maxDataBurstVolume>
                                                                    </non-Dynamic-5QI>
                                                                </qoS-Characteristics>
                                                                <nGRANallocationRetentionPriority>
                                                                    <priorityLevel>14</priorityLevel>
                                                                    <pre-emptionCapability><may-trigger-pre-emption/></pre-emptionCapability>
                                                                    <pre-emptionVulnerability><not-pre-emptable/></pre-emptionVulnerability>
                                                                </nGRANallocationRetentionPriority>
                                                                <iE-Extensions>
                                                                    <QoSFlowLevelQoSParameters-ExtIEs>
                                                                        <id>180</id>
                                                                        <criticality><ignore/></criticality>
                                                                        <extensionValue>
                                                                            <PDUSessionID>1</PDUSessionID>
                                                                        </extensionValue>
                                                                    </QoSFlowLevelQoSParameters-ExtIEs>
                                                                </iE-Extensions>
                                                            </dRB-QoS>
                                                            <sNSSAI>
                                                                <sST>01</sST>
                                                                <sD>02 03 04</sD>
                                                            </sNSSAI>
                                                            <flows-Mapped-To-DRB-List>
                                                                <Flows-Mapped-To-DRB-Item>
                                                                    <qoSFlowIdentifier>0</qoSFlowIdentifier>
                                                                    <qoSFlowLevelQoSParameters>
                                                                        <qoS-Characteristics>
                                                                            <non-Dynamic-5QI>
                                                                                <fiveQI>9</fiveQI>
                                                                                <averagingWindow>0</averagingWindow>
                                                                                <maxDataBurstVolume>0</maxDataBurstVolume>
                                                                            </non-Dynamic-5QI>
                                                                        </qoS-Characteristics>
                                                                        <nGRANallocationRetentionPriority>
                                                                            <priorityLevel>14</priorityLevel>
                                                                            <pre-emptionCapability><may-trigger-pre-emption/></pre-emptionCapability>
                                                                            <pre-emptionVulnerability><not-pre-emptable/></pre-emptionVulnerability>
                                                                        </nGRANallocationRetentionPriority>
                                                                    </qoSFlowLevelQoSParameters>
                                                                </Flows-Mapped-To-DRB-Item>
                                                            </flows-Mapped-To-DRB-List>
                                                        </DRB-Information>
                                                    </value>
                                                </choice-extension>
                                            </qoSInformation>
                                            <uLUPTNLInformation-ToBeSetup-List>
                                                <ULUPTNLInformation-ToBeSetup-Item>
                                                    <uLUPTNLInformation>
                                                        <gTPTunnel>
                                                            <transportLayerAddress>
                                                                11000000101010001000001001010010
                                                            </transportLayerAddress>
                                                            <gTP-TEID>00 00 00 01</gTP-TEID>
                                                        </gTPTunnel>
                                                    </uLUPTNLInformation>
                                                </ULUPTNLInformation-ToBeSetup-Item>
                                            </uLUPTNLInformation-ToBeSetup-List>
                                            <rLCMode><rlc-um-bidirectional/></rLCMode>
                                        </DRBs-ToBeSetup-Item>
                                    </value>
                                </ProtocolIE-SingleContainer>
                                <ProtocolIE-SingleContainer>
                                    <id>34</id>
                                    <criticality><ignore/></criticality>
                                    <value>
                                        <DRBs-ToBeSetup-Item>
                                            <dRBID>2</dRBID>
                                            <qoSInformation>
                                                <choice-extension>
                                                    <id>164</id>
                                                    <criticality><ignore/></criticality>
                                                    <value>
                                                        <DRB-Information>
                                                            <dRB-QoS>
                                                                <qoS-Characteristics>
                                                                    <non-Dynamic-5QI>
                                                                        <fiveQI>9</fiveQI>
                                                                        <averagingWindow>0</averagingWindow>
                                                                        <maxDataBurstVolume>0</maxDataBurstVolume>
                                                                    </non-Dynamic-5QI>
                                                                </qoS-Characteristics>
                                                                <nGRANallocationRetentionPriority>
                                                                    <priorityLevel>14</priorityLevel>
                                                                    <pre-emptionCapability><may-trigger-pre-emption/></pre-emptionCapability>
                                                                    <pre-emptionVulnerability><not-pre-emptable/></pre-emptionVulnerability>
                                                                </nGRANallocationRetentionPriority>
                                                                <iE-Extensions>
                                                                    <QoSFlowLevelQoSParameters-ExtIEs>
                                                                        <id>180</id>
                                                                        <criticality><ignore/></criticality>
                                                                        <extensionValue>
                                                                            <PDUSessionID>1</PDUSessionID>
                                                                        </extensionValue>
                                                                    </QoSFlowLevelQoSParameters-ExtIEs>
                                                                </iE-Extensions>
                                                            </dRB-QoS>
                                                            <sNSSAI>
                                                                <sST>05</sST>
                                                                <sD>06 07 08</sD>
                                                            </sNSSAI>
                                                            <flows-Mapped-To-DRB-List>
                                                                <Flows-Mapped-To-DRB-Item>
                                                                    <qoSFlowIdentifier>0</qoSFlowIdentifier>
                                                                    <qoSFlowLevelQoSParameters>
                                                                        <qoS-Characteristics>
                                                                            <non-Dynamic-5QI>
                                                                                <fiveQI>9</fiveQI>
                                                                                <averagingWindow>0</averagingWindow>
                                                                                <maxDataBurstVolume>0</maxDataBurstVolume>
                                                                            </non-Dynamic-5QI>
                                                                        </qoS-Characteristics>
                                                                        <nGRANallocationRetentionPriority>
                                                                            <priorityLevel>14</priorityLevel>
                                                                            <pre-emptionCapability><may-trigger-pre-emption/></pre-emptionCapability>
                                                                            <pre-emptionVulnerability><not-pre-emptable/></pre-emptionVulnerability>
                                                                        </nGRANallocationRetentionPriority>
                                                                    </qoSFlowLevelQoSParameters>
                                                                </Flows-Mapped-To-DRB-Item>
                                                            </flows-Mapped-To-DRB-List>
                                                        </DRB-Information>
                                                    </value>
                                                </choice-extension>
                                            </qoSInformation>
                                            <uLUPTNLInformation-ToBeSetup-List>
                                                <ULUPTNLInformation-ToBeSetup-Item>
                                                    <uLUPTNLInformation>
                                                        <gTPTunnel>
                                                            <transportLayerAddress>
                                                                11000000101010001000001001010010
                                                            </transportLayerAddress>
                                                            <gTP-TEID>00 00 00 02</gTP-TEID>
                                                        </gTPTunnel>
                                                    </uLUPTNLInformation>
                                                </ULUPTNLInformation-ToBeSetup-Item>
                                            </uLUPTNLInformation-ToBeSetup-List>
                                            <rLCMode><rlc-um-bidirectional/></rLCMode>
                                        </DRBs-ToBeSetup-Item>
                                    </value>
                                </ProtocolIE-SingleContainer>
                            </DRBs-ToBeSetup-List>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>184</id>
                        <criticality><ignore/></criticality>
                        <value>
                            <RRCDeliveryStatusRequest><true/></RRCDeliveryStatusRequest>
                        </value>
                    </UEContextSetupRequestIEs>
                    <UEContextSetupRequestIEs>
                        <id>158</id>
                        <criticality><ignore/></criticality>
                        <value>
                            <BitRate>993522893</BitRate>
                        </value>
                    </UEContextSetupRequestIEs>
                </protocolIEs>
            </UEContextSetupRequest>
        </value>
    </initiatingMessage>
</F1AP-PDU>
```


## 4. UE Context Setup Response

| Point       | Detail                                                      |
| ----------- | ----------------------------------------------------------- |
| Description | is sent by the gNB-DU to confirm the setup of a UE context. |
| Reference   | TS 38.473                                                   |
| Flow        | gNB-DU → gNB-CU                                             |
| Fields      | [See below](#41-Fields)                                     |

### 4.1. Fields

| Field                                     | Type           |
| ----------------------------------------- | -------------- |
| Message Type                              |                |
| gNB-CU UE F1AP ID                         |                |
| gNB-DU UE F1AP ID                         |                |
| DRB Setup List                            | DRB Setup Item |
| Other unrelated fields to Network Slicing | -              |


### 4.2. Example


```xml=0
<F1AP-PDU>
    <successfulOutcome>
        <procedureCode>5</procedureCode>
        <criticality><reject/></criticality>
        <value>
            <UEContextSetupResponse>
                <protocolIEs>
                    <UEContextSetupResponseIEs>
                        <id>40</id>
                        <criticality><reject/></criticality>
                        <value>
                            <GNB-CU-UE-F1AP-ID>1</GNB-CU-UE-F1AP-ID>
                        </value>
                    </UEContextSetupResponseIEs>
                    <UEContextSetupResponseIEs>
                        <id>41</id>
                        <criticality><reject/></criticality>
                        <value>
                            <GNB-DU-UE-F1AP-ID>1</GNB-DU-UE-F1AP-ID>
                        </value>
                    </UEContextSetupResponseIEs>
                    <UEContextSetupResponseIEs>
                        <id>39</id>
                        <criticality><reject/></criticality>
                        <value>
                            <DUtoCURRCInformation>
                                <cellGroupConfig>
                                    5C 04 B0 91 10 0A EC 81 D0 61 F0 00 B1 C0 7D 08 
                                    30 F8 00 59 21 3E 84 18 7C 00 1F 08 A9 25 C8 3D 
                                    C6 01 03 D4 C0 4B 20 32 A0 02 20 0F 00 00 00 00 
                                    70 00 10 DC 21 08 00 3F 88 C9 08 10 12 03 12 14 
                                    14 20 01 33 55 40 80 80 08 00 08 00 80 00 88 91 
                                    2A 84 40 00 C9 08 00 05 8C 44 28 A8 59 40 80 80 
                                    20 10 00 20 18 00 00 00 00 00 0C 00 80 E0 45 00
                                </cellGroupConfig>
                            </DUtoCURRCInformation>
                        </value>
                    </UEContextSetupResponseIEs>
                    <UEContextSetupResponseIEs>
                        <id>95</id>
                        <criticality><reject/></criticality>
                        <value>
                            <C-RNTI>100</C-RNTI>
                        </value>
                    </UEContextSetupResponseIEs>
                    <UEContextSetupResponseIEs>
                        <id>27</id>
                        <criticality><reject/></criticality>
                        <value>
                            <DRBs-Setup-List>
                                <ProtocolIE-SingleContainer>
                                    <id>26</id>
                                    <criticality><reject/></criticality>
                                    <value>
                                        <DRBs-Setup-Item>
                                            <dRBID>1</dRBID>
                                            <dLUPTNLInformation-ToBeSetup-List>
                                                <DLUPTNLInformation-ToBeSetup-Item>
                                                    <dLUPTNLInformation>
                                                        <gTPTunnel>
                                                            <transportLayerAddress>
                                                                11000000101010001000001001010001
                                                            </transportLayerAddress>
                                                            <gTP-TEID>00 00 00 01</gTP-TEID>
                                                        </gTPTunnel>
                                                    </dLUPTNLInformation>
                                                </DLUPTNLInformation-ToBeSetup-Item>
                                            </dLUPTNLInformation-ToBeSetup-List>
                                        </DRBs-Setup-Item>
                                    </value>
                                </ProtocolIE-SingleContainer>
                                <ProtocolIE-SingleContainer>
                                    <id>26</id>
                                    <criticality><reject/></criticality>
                                    <value>
                                        <DRBs-Setup-Item>
                                            <dRBID>2</dRBID>
                                            <dLUPTNLInformation-ToBeSetup-List>
                                                <DLUPTNLInformation-ToBeSetup-Item>
                                                    <dLUPTNLInformation>
                                                        <gTPTunnel>
                                                            <transportLayerAddress>
                                                                11000000101010001000001001010001
                                                            </transportLayerAddress>
                                                            <gTP-TEID>00 00 00 02</gTP-TEID>
                                                        </gTPTunnel>
                                                    </dLUPTNLInformation>
                                                </DLUPTNLInformation-ToBeSetup-Item>
                                            </dLUPTNLInformation-ToBeSetup-List>
                                        </DRBs-Setup-Item>
                                    </value>
                                </ProtocolIE-SingleContainer>
                            </DRBs-Setup-List>
                        </value>
                    </UEContextSetupResponseIEs>
                </protocolIEs>
            </UEContextSetupResponse>
        </value>
    </successfulOutcome>
</F1AP-PDU>
```


## 5. RRC Reconfig in DL RRC Message Transfer

### 5.1. RRC Reconfig

| Point       | Detail                                                                                                                                                                                                                                                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Description | The purpose of this message is to modify an RRC connection, e.g. to establish/modify/release RBs, to perform reconfiguration with sync, to setup/modify/release measurements, to add/modify/release SCells and cell groups. As part of the procedure, NAS dedicated information may be transferred from the Network to the UE. |
| Reference   | TS 38.331                                                                                                                                                                                                                                                                                                                      |
| Flow        | gNB ➔ UE                                                                                                                                                                                                                                                                                                                       |
| Fields      | [See below](#511-Fields)                                                                                                                                                                                                                                                                                                       |

#### 5.1.1. Fields

| Field                                     | Type                 |
| ----------------------------------------- | -------------------- |
| dedicatedNAS-MessageList                  | DedicatedNAS-Message |
| Other unrelated fields to Network Slicing | -                    |


**DedicatedNAS-Message contains the [Registration Accept](https://hackmd.io/@superwilfrid/ryKdDBucA#22-Registration-Accept) which contains NSSAI**


#### 5.1.2. Example

- Got no example


### 5.2. DL RRC Message Transfer

| Point       | Detail                                                                                    |
| ----------- | ----------------------------------------------------------------------------------------- |
| Description | is sent by the gNB-CU to transfer the layer 3 message to the gNB-DU over the F1 interface |
| Reference   | TS 38.473                                                                                 |
| Flow        | gNB-CU →gNB-DU                                                                            |
| Fields      | [See below](#521-Fields)                                                                  |

#### 5.2.1. Fields

| Field                                     | Type                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| Message Type                              |                                                              |
| gNB-CU UE F1AP ID                         |                                                              |
| gNB-DU UE F1AP ID                         |                                                              |
| RRC-Container                             | [RRC Reconfig](#51-RRC-Reconfig) |
| Other unrelated fields to Network Slicing | -                                                            |


#### 5.2.2. Example

- Got no example

