# 2024/08/13 Study Note (What parameters need to get from Core)

###### tags: `2024`

**Goal:**
- Q: What parameters need to get from Core? 
- A: **NSSAI** in
    - [x] [Initial UE Message in Registration Request](#1-Initial-UE-Message-in-Registration-Request)
    - [x] [Initial Context Setup Request in Registration Accept](#2-Initial-Context-Setup-Request-in-Registration-Accept)
    - [x] [Initial Context Setup Response in RRC Reconfig](#3-Initial-Context-Setup-Response-in-RRC-Reconfig)
    - [x] [PDU Session Resource Setup Request](#4-PDU-Session-Resource-Setup-Request)
    - [x] [PDU Session Resource Setup Reponse](#5-PDU-Session-Resource-Setup-Reponse)
    - [x] [PDU Session Establishment Request](#6-PDU-Session-Establishment-Request)
    - [x] [PDU Session Establishment Accept](#7-PDU-Session-Establishment-Accept)
 
**References:**
- 3GPP TS 38.300 | NR & NG-RAN Overall Description
- 3GPP TS 38.401 | 5G NG-RAN Architecture Description
- Event Helix | 5G Standalone Access Registration
- Event Helix | 5G Standalone Access Registration Signaling Messages
- 3GPP TS 38.413 | 5G NG-RAN NG Application Protocol
- 3GPP TS 24.501 | 5G NAS Protocol
- [ShareTechnote | 5G/NR  - Network Slicing](https://www.sharetechnote.com/html/5G/5G_NetworkSlicing.html)


## 0. Summary

![image](https://hackmd.io/_uploads/H1ExL8OqR.png)

![image](https://hackmd.io/_uploads/S12WCH-RR.png)

![image](https://hackmd.io/_uploads/HJCNBtOcA.png)

- [2024/08/13 Study Note (What parameters need to get from Core)](#2024-08-13-study-note--what-parameters-need-to-get-from-core-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
  * [1. Initial UE Message in Registration Request](#1-initial-ue-message-in-registration-request)
    + [1.1. Registration Request NAS Message](#11-registration-request-nas-message)
      - [1.1.1. Fields](#111-fields)
      - [1.1.2. Example](#112-example)
    + [1.2. Initial UE Message](#12-initial-ue-message)
      - [1.2.1. Fields](#121-fields)
      - [1.2.2. Example](#122-example)
  * [2. Initial Context Setup Request in Registration Accept](#2-initial-context-setup-request-in-registration-accept)
    + [2.1. Initial Context Setup Request](#21-initial-context-setup-request)
      - [2.1.1. Fields](#211-fields)
      - [2.1.2. Example](#212-example)
    + [2.2. Registration Accept](#22-registration-accept)
      - [2.2.1. Fields](#221-fields)
      - [2.2.2. Example](#222-example)
  * [3. Initial Context Setup Response in RRC Reconfig](#3-initial-context-setup-response-in-rrc-reconfig)
    + [3.1. Fields](#31-fields)
    + [3.2. Example](#32-example)
  * [4. PDU Session Resource Setup Request](#4-pdu-session-resource-setup-request)
    + [4.1. Fields](#41-fields)
    + [4.2. Example](#42-example)
  * [5. PDU Session Resource Setup Reponse](#5-pdu-session-resource-setup-reponse)
    + [5.1. Fields](#51-fields)
    + [5.2. Example](#52-example)
  * [6. PDU Session Establishment Request](#6-pdu-session-establishment-request)
    + [6.1. Fields](#61-fields)
    + [6.2. Example](#62-example)
  * [7. PDU Session Establishment Accept](#7-pdu-session-establishment-accept)
    + [7.1. Fields](#71-fields)
    + [7.2. Example](#72-example)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 1. Initial UE Message in Registration Request

### 1.1. Registration Request NAS Message

| Point       | Detail                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| Description | The Registration Request NAS message is carried from the UE to the newly assigned AMF                                    |
| Reference   | TS 24.501                                                                                                                |
| Flow        | UE ➔ RRCSetupComplete ➔ gNB ➔ NGAP Initial UE Message ➔ New AMF ➔ Namf_Communication_UEContextTransfer Request ➔ Old AMF |
| Fields            | [See below](#111-fields)                                                                                                                         |

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



### 1.2. Initial UE Message

| Point       | Detail                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| Description | The gNB sends the Initial UE Message to the selected AMF. The message carries the Registration Request that was received from the UE in the RRC Setup Complete message. The "RAN UE NGAP ID" and the "RRC Establishment Cause" are also included in the message.                                    |
| Reference   | TS 38.5413                                                                                                                |
| Flow        | gNB ➔ New AMF |
| Fields            | [See below](#121-fields)                                                                                                                         |

#### 1.2.1. Fields

| Field                                     | Description                                                  |
| ----------------------------------------- | ------------------------------------------------------------ |
| Message Type                              | 0x41 (Registration request)                                  |
| NAS-PDU                                   | [Registration Request](#11-Registration-Request-NAS-Message) |
| Network Slicing Indication                | 0x00 (DCNI=0, NSSCI=0)                                       |
| Other unrelated fields to Network Slicing | -                                                            |


#### 1.2.2. Example


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



## 2. Initial Context Setup Request in Registration Accept

### 2.1. Initial Context Setup Request

| Point       | Detail                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| Description | The purpose of the Initial Context Setup procedure is to establish the necessary overall initial UE Context at the NG-RAN node, when required, including PDU session context, the Security Key, Mobility Restriction List, UE Radio Capability and UE Security Capabilities, etc.<br><br> The AMF initiates a session setup with the gNB. The message typically contains the Registration Accept NAS message. The message carries one or more PDU Session setup requests. Each PDU session is addressed with the "PDU Session ID". |
| Reference   | TS 38.413                                                                                                                |
| Flow        | AMF ➔ gNB |
| Fields            | [See below](#211-fields)                                                                                                                         |

#### 2.1.1. Fields

| Field                                     | Type                                           |
| ----------------------------------------- | ---------------------------------------------- |
| Message Type                              | 0x42 (Registration accept)                     |
| AMF UE NGAP ID                            |                                                |
| RAN UE NGAP ID                            |                                                |
| PDU Session Resource Setup Request List   | PDU Session Resource Setup Request Item        |
| Allowed NSSAI                             |                                                |
| NAS-PDU                                   | [Registration Accept](#22-Registration-Accept) |
| Other unrelated fields to Network Slicing | -                                              |


**PDU Session Resource Setup Request Item contains NSSAI**



#### 2.1.2. Example


Protocol discriminator = 0x7e (5GS Mobility Management)

Security header = 0x2 (Integrity protected and ciphered)

Auth code = 0x59b0464b

Sequence number = 0x02

Protocol discriminator = 0x7e (5GS Mobility Management)

Security header = 0x0 (Plain 5GS NAS message, not security protected)

**Message type = 0x42 (Registration accept)**

5GS registration result = 0x09 (Emergency registered=0, NSSAA to be performed=0, SMS allowed=1, 3GPP access)

5G-GUTI:

  5G-GUTI

    MCC = 001

    MNC = 01

    AMF Region ID = 128

    AMF Set ID = 4

    AMF Pointer = 1

    5G-TMSI = 0x4b63aa9a

TAI list:

  Length = 7

  Data = 00 00 f1 10 00 00 64

**Allowed NSSAI:   // Network may configure multiple S-NSSAI here**

  **S-NSSAI**

    **Length of S-NSSAI contents = 1 (SST)**

    **SST = 0x01**

5GS network feature support:

  0x03 (MPSI=0, IWK N26=0, EMF=not supported, EMC=not supported, IMS-VoPS-N3GPP=1, IMS-VoPS-3GPP=1)

  0x00 (5G-UP CIoT=0, 5G-IPHC-CP CIoT=0, N3 data=0, 5G-CP CIoT=0, RestrictEC=both CE mode A and CE mode B are not restricted, MCSI=0, EMCN3=0)

T3512 value:

  Value = 30

  Unit = 5 (1 minute)

Emergency number list:

  Length = 8

  Data = 03 1f 19 f1 03 1f 11 f2



### 2.2. Registration Accept

| Point       | Detail                                                                                                                                                                                                                                                           |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Description | The gNB sends the Initial UE Message to the selected AMF. The message carries the Registration Request that was received from the UE in the RRC Setup Complete message. The "RAN UE NGAP ID" and the "RRC Establishment Cause" are also included in the message. |
| Reference   | TS 24.501                                                                                                                                                                                                                                                        |
| Flow        | New AMF ➔ Initial Context Setup Request ➔ gNB ➔ RRCReconfiguration ➔ UE                                                                                                                                                                                          |
| Fields      | [See below](#221-fields)                                                                                                                                                                                                                                         |

#### 2.2.1. Fields

| Field                                     | Type  |
| ----------------------------------------- | ----- |
| Allowed NSSAI                             | NSSAI |
| Rejected NSSAI                            | NSSAI |
| Configured NSSAI                          | NSSAI |
| Network Slicing Indication                |       |
| Other unrelated fields to Network Slicing | -     |


#### 2.2.2. Example


Protocol discriminator = 0x7e (5GS Mobility Management)

Security header = 0x2 (Integrity protected and ciphered)

Auth code = 0x59b0464b

Sequence number = 0x02

Protocol discriminator = 0x7e (5GS Mobility Management)

Security header = 0x0 (Plain 5GS NAS message, not security protected)

**Message type = 0x42 (Registration accept)**

5GS registration result = 0x09 (Emergency registered=0, NSSAA to be performed=0, SMS allowed=1, 3GPP access)

5G-GUTI:

  5G-GUTI

    MCC = 001

    MNC = 01

    AMF Region ID = 128

    AMF Set ID = 4

    AMF Pointer = 1

    5G-TMSI = 0x4b63aa9a

TAI list:

  Length = 7

  Data = 00 00 f1 10 00 00 64

**Allowed NSSAI:   // Network may configure multiple S-NSSAI here**

  **S-NSSAI**

    **Length of S-NSSAI contents = 1 (SST)**

    **SST = 0x01**

5GS network feature support:

  0x03 (MPSI=0, IWK N26=0, EMF=not supported, EMC=not supported, IMS-VoPS-N3GPP=1, IMS-VoPS-3GPP=1)

  0x00 (5G-UP CIoT=0, 5G-IPHC-CP CIoT=0, N3 data=0, 5G-CP CIoT=0, RestrictEC=both CE mode A and CE mode B are not restricted, MCSI=0, EMCN3=0)

T3512 value:

  Value = 30

  Unit = 5 (1 minute)

Emergency number list:

  Length = 8

  Data = 03 1f 19 f1 03 1f 11 f2



## 3. Initial Context Setup Response in RRC Reconfig

| Point       | Detail                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| Description | is sent by the NG-RAN node to confirm the setup of a UE context. |
| Reference   | TS 38.413                                                                                                                |
| Flow        | gNB ➔ AMF |
| Fields            | [See below](#31-fields)                                                                                                                         |

### 3.1. Fields

| Field                                     | Type                                     |
| ----------------------------------------- | ---------------------------------------- |
| Message Type                              |                                          |
| AMF UE NGAP ID                            |                                          |
| RAN UE NGAP ID                            |                                          |
| PDU Session Resource Setup Response List  | PDU Session Resource Setup Response Item |
| Other unrelated fields to Network Slicing | -                                        |


### 3.2. Example

- Got no example

## 4. PDU Session Resource Setup Request

| Point       | Detail                                                                                                                                              |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Description | This message is sent by the AMF and is used to request the NG-RAN node to assign resources on Uu and NG-U for one or several PDU session resources. |
| Reference   | TS 38.413                                                                                                                                           |
| Flow        | AMF → NG-RAN node                                                                                                                                   |
| Fields      | [See below](#41-fields)                                                                                                                                                    |

### 4.1. Fields

| Field                                     | Type                                    |
| ----------------------------------------- | --------------------------------------- |
| Message Type                              |                                         |
| AMF UE NGAP ID                            |                                         |
| RAN UE NGAP ID                            |                                         |
| PDU Session Resource Setup Request List   | PDU Session Resource Setup Request Item |
| Other unrelated fields to Network Slicing | -                                       |


**PDU Session Resource Setup Request Item contains NSSAI**



### 4.2. Example

- Got no example

## 5. PDU Session Resource Setup Reponse

| Point       | Detail                                                                                                                                              |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Description | This message is sent by the AMF and is used to request the NG-RAN node to assign resources on Uu and NG-U for one or several PDU session resources. |
| Reference   | TS 38.413                                                                                                                                           |
| Flow        | NG-RAN node → AMF                                                                                                                                   |
| Fields      | [See below](#51-fields)                                                                                                                                                    |

### 5.1. Fields

| Field                                     | Type                                     |
| ----------------------------------------- | ---------------------------------------- |
| Message Type                              |                                          |
| AMF UE NGAP ID                            |                                          |
| RAN UE NGAP ID                            |                                          |
| PDU Session Resource Setup Response List  | PDU Session Resource Setup Response Item |
| Other unrelated fields to Network Slicing | -                                        |


### 5.2. Example

- Got no example

## 6. PDU Session Establishment Request

| Point       | Detail                                                                                                                |
| ----------- | --------------------------------------------------------------------------------------------------------------------- |
| Description | The PDU SESSION ESTABLISHMENT REQUEST message is sent by the UE to the SMF to initiate establishment of a PDU session |
| Reference   | TS 24.501                                                                                                             |
| Flow        | UE ➔ gNB ➔ SMF                                                                                                        |
| Fields      | [See Below](#61-fields)                                                                                               |

### 6.1. Fields

| Field                                     | Type                                     |
| ----------------------------------------- | ---------------------------------------- |
| Message Type                              | 0xc1 (PDU session establishment request) |
| Selected NSSAI                            |                                          |
| Other unrelated fields to Network Slicing | -                                        |


### 6.2. Example


Protocol discriminator = 0x7e (5GS Mobility Management)

Security header = 0x2 (Integrity protected and ciphered)

Auth code = 0x37ed3aa2

Sequence number = 0x03

Protocol discriminator = 0x7e (5GS Mobility Management)

Security header = 0x0 (Plain 5GS NAS message, not security protected)

Message type = 0x67 (UL NAS transport)

Payload container type = 1 (N1 SM information)

Payload container:

  Protocol discriminator = 0x2e (5GS Session Management)

  PDU session identity = 2

  Procedure transaction identity = 2

  **Message type = 0xc1 (PDU session establishment request)**

  Integrity protection maximum data rate:

    Maximum data rate per UE for user-plane integrity protection for uplink = 0xff (Full data rate)

    Maximum data rate per UE for user-plane integrity protection for downlink = 0xff (Full data rate)

  PDU session type = 0x3 (IPv4v6)

  Always-on PDU session requested = 1

  Extended protocol configuration options:

    Ext = 1

    Configuration protocol = 0

    Protocol ID = 0x8021 (IPCP)

    Data = 01 00 00 10 81 06 00 00 00 00 83 06 00 00 00 00

    Protocol ID = 0x0001 (P-CSCF IPv6 Address Request)

    Data =

    Protocol ID = 0x0003 (DNS Server IPv6 Address Request)

    Data =

    Protocol ID = 0x000a (IP address allocation via NAS signalling)

    Data =

    Protocol ID = 0x000c (P-CSCF IPv4 Address Request)

    Data =

    Protocol ID = 0x000d (DNS Server IPv4 Address Request)

    Data =

    Protocol ID = 0x000e (MSISDN Request)

    Data =

    Protocol ID = 0x0011 (MS support of Local address in TFT indicator)

    Data =

PDU session ID = 2

Request type = 0x1 (initial request)

**S-NSSAI:**

  **Length of S-NSSAI contents = 4 (SST and SD)**

  **SST = 0x02**

  **SD = 0x0001f4**

DNN = "internet"


## 7. PDU Session Establishment Accept

| Point       | Detail                                                                                                                                                                                     |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Description | The PDU SESSION ESTABLISHMENT ACCEPT message is sent by the SMF to the UE in response to PDU SESSION ESTABLISHMENT REQUEST message and indicates successful establishment of a PDU session |
| Reference   | TS 24.501                                                                                                                                                                                  |
| Flow        | SMF ➔ gNB ➔ UE                                                                                                                                                                             |
| Fields      | [See Below](#71-fields)                                                                                                                                                                    |

### 7.1. Fields

| Field                                     | Type                                    |
| ----------------------------------------- | --------------------------------------- |
| Message Type                              | 0xc2 (PDU session establishment accept) |
| Selected NSSAI                            |                                         |
| Other unrelated fields to Network Slicing | -                                       |


### 7.2. Example


Protocol discriminator = 0x2e (5GS Session Management)

PDU session identity = 1

Procedure transaction identity = 5

**Message type = 0xc2 (PDU session establishment accept)**

Selected PDU session type = 0x1 (IPv4)

Selected SSC mode = 0x1 (1)

Authorized QoS rules:

  QoS rule 1:

    QoS rule identifier = 1

    Rule operation code = 1 (create new QoS rule)

    DQR = 1 (the QoS rule is the default QoS rule)

    Number of packet filters = 1

    Packet filter identifier = 15

      Packet filter direction = 3 (bidirectional)

      Match-all

    QoS rule precedence = 255

    QFI = 1

Session AMBR:

  Session-AMBR for downlink = 5000 Mbps

  Session-AMBR for uplink = 2000000 kbps

5GSM cause = 0x32 (PDU session type IPv4 only allowed)

PDU address:

  SI6LLA = 0

  PDU session type = 1 (IPv4)

  IPv4 = 192.168.3.2

**S-NSSAI:**

  **Length of S-NSSAI contents = 1 (SST)**

  **SST = 0x01**

Mapped EPS bearer contexts:

  Mapped EPS bearer context 1:

    EPS bearer identity = 5

    Operation code = 1 (create new EPS bearer)

    E = 1 (parameters list is included)

    Number of EPS parameters = 2

    Mapped EPS QoS parameters:

      QCI = 9

    APN-AMBR:

      APN-AMBR for downlink = 4864000000 bits

      APN-AMBR for uplink = 1792000000 bits

Authorized QoS flow descriptions:

  QoS flow description 1:

    QFI = 1

    Operation code = 1 (create new QoS flow description)

    E = 1 (parameters list is included)

    Number of parameters = 2

    5QI = 9

    EPS bearer identity = 5

Extended protocol configuration options:

  Ext = 1

  Configuration protocol = 0

  Protocol ID = 0x8021 (IPCP)

  Data = 03 00 00 0a 81 06 08 08 08 08

  Protocol ID = 0x000d (DNS Server IPv4 Address)

  Data = 8.8.8.8

DNN = "internet.mnc001.mcc001.gprs"


