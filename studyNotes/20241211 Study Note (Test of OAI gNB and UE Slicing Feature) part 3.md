# 2024/12/11 Study Note (Test of OAI gNB and UE Slicing Feature) part 3

###### tags: `2024`

info
**Goal:**
- [x] [Test OAI gNB and OAI UE `develop` branch Slicing Feature in CU-DU Split Multi DU with Handover](#1-develop-branch-CU-DU-Split-Multi-DU-with-Handover)
- [ ] Test OAI gNB and OAI UE `rebased-5g-nvs-rc-rf` branch Slicing Feature

**References:**
- [OAI | NR SA Tutorial OAI nrUE](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md)
- [Wilfrid | RAN Slicing Architecture Requirements](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240815%20Study%20Note%20(RAN%20Slicing%20Architecture%20Requirements).md)
- [Wilfrid | ORANSlice: An Open-Source 5G Network Slicing Platform for O-RAN](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20241023%20Study%20Note%20(ORANSlice%20An%20Open-Source%205G%20Network%20Slicing%20Platform%20for%20O-RAN).md#2-what-they-add-in-oai-nrue)
- [Wilfrid | Test of OAI gNB and UE Slicing Feature](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20241011%20Study%20Note%20(Test%20of%20OAI%20gNB%20and%20UE%20Slicing%20Feature).md)
- [Wilfrid | Test of OAI gNB and UE Slicing Feature part 2](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20241106%20Study%20Note%20(Test%20of%20OAI%20gNB%20and%20UE%20Slicing%20Feature)%20part%202.md)
- [sharetechnote | 5G/NR  - Network Slicing](https://www.sharetechnote.com/html/5G/5G_NetworkSlicing.html)
![image](https://hackmd.io/_uploads/rkhGBXCgyl.png)

**Table of Contents:**
- [2024/12/11 Study Note (Test of OAI gNB and UE Slicing Feature) part 3](#2024-12-11-study-note--test-of-oai-gnb-and-ue-slicing-feature--part-3)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
  * [1. `develop` branch CU-DU Split Multi DU with Handover](#1--develop--branch-cu-du-split-multi-du-with-handover)
    + [1.0. Minimum Requirement](#10-minimum-requirement)
      - [1.0.1. OAI gNB with rfsim](#101-oai-gnb-with-rfsim)
      - [1.0.2. OAI UE with rfsim](#102-oai-ue-with-rfsim)
    + [1.1. Topology](#11-topology)
    + [1.2. Environment](#12-environment)
      - [1.2.1. OAI CU, DU and UE](#121-oai-cu--du-and-ue)
    + [1.3. Compile](#13-compile)
    + [1.4. Run](#14-run)
      - [1.4.1. Configuration](#141-configuration)
        * [1.4.1.1. CN Configuration](#1411-cn-configuration)
        * [1.4.1.2. CU Configuration](#1412-cu-configuration)
        * [1.4.1.3. DU 1 Configuration](#1413-du-1-configuration)
        * [1.4.1.4. DU 2 Configuration](#1414-du-2-configuration)
        * [1.4.1.5. UE 1 Configuration](#1415-ue-1-configuration)
      - [1.4.2. Result](#142-result)
        * [1.4.2.1. Initial Run](#1421-initial-run)
        * [1.4.2.2. Handover](#1422-handover)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 0. Summary

1. abc

## 1. `develop` branch CU-DU Split Multi DU with Handover

| Step                                      | Status             |
| ----------------------------------------- | ------------------ |
| NG Setup                                  | :heavy_check_mark: |
| F1 Setup                                  | :heavy_check_mark: |
| DU Configuration Update                   | :heavy_check_mark: |
| UE get mib and sib1 for DL sync           | :heavy_check_mark: |
| UE send PRACH for UL sync                 | :heavy_check_mark: |
| UE get RAR message                        | :heavy_check_mark: |
| UE send RRC setup request                 | :heavy_check_mark: |
| RRC setup complete                        | :heavy_check_mark: |
| NAS registration                          | :heavy_check_mark: |
| [PDU session complete](#1421-Initial-Run) | :heavy_check_mark: |
| [F1 Handover](#1422-Handover)             | :heavy_check_mark: |

### 1.0. Minimum Requirement

- [Source](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/system_requirements.md)

#### 1.0.1. OAI gNB with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 4GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

#### 1.0.2. OAI UE with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 3GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

### 1.1. Topology

![image](https://hackmd.io/_uploads/SJ1rIjI4Jl.png)

### 1.2. Environment

#### 1.2.1. OAI CU, DU and UE

<b>Hardware:</b>

| Item         | Info                                                         |
| ------------ | ------------------------------------------------------------ |
| CPU          | Intel(R) Xeon(R) E5-2695 processor @ 2.10 GHz (18 cores) x 2 |
| Memory       | 64GB (8GB x 8)                                               |
| Disk         | 270GB                                                        |
| Server Model | Supermicro SSG-6028R-E1CR12L                                 |

Command Line Codes
```shell=
# Check CPU Type, freq, cores, numbers
lscpu

# Check total memory
lshw -C memory

# Check total disk
df -h
df --total -h | grep 'total' | awk '{print $2}'

# Check server model
sudo dmidecode -t system
```


<b>Software:</b>

| Item       | Info                                                                                                    |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| OS         | Ubuntu 22.04.5 LTS (jammy)                                                                              |
| Kernel     | 5.15.0-1071-realtime                                                                                    |
| OAI Commit | 054506f5ae88a942926c3d388e732e15dd1224e9 (HEAD -> develop, tag: 2024.w49, origin/develop, origin/HEAD) |

Command Line Codes
```shell=
# Check OS
lsb_release -a

# Check kernel
uname -a
uname -r

# Check OAI commit
git log -1
```


### 1.3. Compile

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#32-build-oai-gnb-and-oai-nrue)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>1. Get openairinterface5g source code</b>

```shell=
# Get openairinterface5g source code
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git ~/openairinterface5g
cd ~/openairinterface5g
git checkout develop
git log -1
```


![image](https://hackmd.io/_uploads/HJUdSzqykg.png)
![image](https://hackmd.io/_uploads/S19YUGqkkg.png)
![image](https://hackmd.io/_uploads/HJfCxpI4yg.png)


<b>2. Install OAI dependencies</b>

```shell=
# Install OAI dependencies
cd ~/openairinterface5g/cmake_targets
./build_oai -I
```


![image](https://hackmd.io/_uploads/HkviUM51Jx.png)


<b>3. nrscope dependencies</b>

```shell=
# nrscope dependencies
sudo apt install -y libforms-dev libforms-bin
```


![image](https://hackmd.io/_uploads/r1isTUqkye.png)


<b>4. Build OAI gNB with telnet</b>

```shell=
# Build OAI gNB
cd ~/openairinterface5g/cmake_targets
./build_oai -w SIMU --ninja --nrUE --gNB --build-lib telnetsrv -C
```


![image](https://hackmd.io/_uploads/SkHIuoUV1x.png)


### 1.4. Run

#### 1.4.1. Configuration

##### 1.4.1.1. CN Configuration

- [Source 1](https://hackmd.io/@Summer063/HJyq005rT)
- [Source 2](https://open5gs.org/open5gs/docs/guide/01-quickstart/)

<b>1. We use example configuration from `open5gs/configs/sample.yaml.in`</b>

<b>2. Be aware that you might need to change 2 things in the configuration file:</b>
- Anything mentioned to be modify by Summer (modify as Summer specify. Values that I use is below)
```shell=
mcc:001
mnc:01

key path :@build_configs_dir@/ -> /home/srsran/

load_extension : @build_subprojects_freeDiameter_extensions_dir@ -> /home/srsran/open5gs/build/subprojects/freeDiameter/extensions/
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
s_nssai:
- sst: 1
  sd: 010203
- sst: 1
  sd: 112233
```

##### 1.4.1.2. CU Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-cu.sa.f1.conf`</b>

<b>2. Be aware that you might need to change 3 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "127.0.0.5"; }); 


NETWORK_INTERFACES :
{
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "127.0.0.17"; 
    GNB_IPV4_ADDRESS_FOR_NGU                 = "127.0.0.18"; 
    GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
};
```
- PLMN List (modify as your desired Slice Configuration. Values that I use is below)
```shell=
plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2;
                snssaiList = (
                                { sst = 1; sd = 0x010203; },
                                { sst = 1; sd = 0x112233; }
                                );
                            });
```
- local/remote_s_address (modify as your desired CU/DU Split IP Address Configuration. Values that I use is below)
```shell=
tr_s_preference = "f1";

    local_s_address = "127.0.0.23";
    remote_s_address = "0.0.0.0";
    local_s_portc   = 2152;
    local_s_portd   = 2152;
    remote_s_portc  = 2152;
    remote_s_portd  = 2152;
```

##### 1.4.1.3. DU 1 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf`</b>

<b>2. Be aware that you might need to change 3 things in the configuration file:</b>
- local_n_address (modify as your DU ip address. Values that I use is below)
```shell=
MACRLCs = (
  {
    num_cc           = 1;
    tr_s_preference  = "local_L1";
    tr_n_preference  = "f1";
    local_n_address = "127.0.0.24";
```
- PLMN List (modify as your desired Slice Configuration. Values that I use is below)
```shell=
plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2;
                snssaiList = (
                                { sst = 1; sd = 0x010203; }
                                );
                            });
```
- remote_n_address (modify as your CU ip address. Values that I use is below)
```shell=
remote_n_address = "127.0.0.23";
    local_n_portc   = 2152;
    local_n_portd   = 2152;
    remote_n_portc  = 2152;
    remote_n_portd  = 2152;
```


##### 1.4.1.4. DU 2 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. Create a copy of `ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf`</b>
![image](https://hackmd.io/_uploads/HkDUFugGyx.png)

<b>2. Be aware that you might need to change 5 things in the configuration file:</b>
- local_n_address (modify as your DU ip address. Values that I use is below)
```shell=
MACRLCs = (
  {
    num_cc           = 1;
    tr_s_preference  = "local_L1";
    tr_n_preference  = "f1";
    local_n_address = "127.0.0.25";
```
- PLMN List (modify as your desired Slice Configuration. Values that I use is below)
```shell=
plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2;
                snssaiList = (
                                { sst = 1; sd = 0x112233; }
                                );
                            });
```
- remote_n_address (modify as your CU ip address. Values that I use is below)
```shell=
remote_n_address = "127.0.0.23";
    local_n_portc   = 2152;
    local_n_portd   = 2152;
    remote_n_portc  = 2152;
    remote_n_portd  = 2152;
```
- DU ID (modify as your desired DU ID. Values that I use is below)
```shell=
gNBs =
(
 {
    ////////// Identification parameters:
    gNB_ID = 0xe01;
    gNB_DU_ID = 0xe01;
    ...
    nr_cellid = 2234568L;
    ...
    servingCellConfigCommon = (
    {
      physCellId = 1;
    ...
```


##### 1.4.1.5. UE 1 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#51-run-oai-nrue)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>1. We use example configuration from `/ci-scripts/conf_files/nrue.uicc.conf`</b>

<b>2. Be aware that you might need to change 3 things in the UE configuration file:</b>
- imsi (modify as your desired ue identity. Values that I use is below)
```shell=
imsi = "001010000000001";
```
- nssai_sst & nssai_sd (modify as your desired Slice Configuration. Values that I use is below)
```shell=
nssai_sst=1;
nssai_sd=66051
```
- dnn (modify as your desired DNN. Values that I use is below)
```shell=
dnn="internet";
```

#### 1.4.2. Result

##### 1.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
screen -r amf
```


![image](https://hackmd.io/_uploads/HyIGdsLNJe.png)



<b>2. Run OAI CU</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-cu.sa.f1.conf --rfsim --sa --telnetsrv --telnetsrv.shrmod ci --telnetsrv.listenport 9091
```


![image](https://hackmd.io/_uploads/rk_-KjL4yg.png)


<b>3. Run OAI DU 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf --rfsim --sa --rfsimulator.serveraddr 127.0.0.1
```


![image](https://hackmd.io/_uploads/SJXvFs8Nkl.png)


<b>4. Run OAI nrUE</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr server -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```


![image](https://hackmd.io/_uploads/BJNTajLN1e.png)



<b>5. Run OAI DU 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du2.sa.band78.106prb.rfsim.conf --rfsim --sa --rfsimulator.serveraddr 127.0.0.1
```


![image](https://hackmd.io/_uploads/SyKlAoI4Je.png)


<b>7. Result explanation</b>
- CU got F1AP Setup Request from first DU & UE with slice 1:010203 get rnti 9905
![image](https://hackmd.io/_uploads/BJChoT8Nyl.png)
- UE is handled by DU 1
![image](https://hackmd.io/_uploads/rJl8hT84kl.png)
- UE PDU session is in slice 1:010203
![image](https://hackmd.io/_uploads/SkP_hpU4yg.png)
![image](https://hackmd.io/_uploads/SJtKnTL4yl.png)

##### 1.4.2.2. Handover

<b>1. Make sure CU, DU1, UE, DU2 are running without problem. If DU and UE stuck, restart DU and UE</b>

<b>2. Login to telnet</b>

```shell=
telnet 127.0.0.1 9091
```


![image](https://hackmd.io/_uploads/B1hy628N1e.png)


<b>3. Trigger handover command to CU through telnet</b>

```shell=
ci trigger_f1_ho
```


![image](https://hackmd.io/_uploads/SkAmPT8Vye.png)


<b>4. Result explanation</b>
- CU have 2 different DU, 3584 and 3585. We see that CU get handover trigger from telnet for UE 9905 to DU 3585
![image](https://hackmd.io/_uploads/B1m4pTI4ke.png)
- DU 1 release UE
![image](https://hackmd.io/_uploads/BJkgRTIV1l.png)
- DU 2 establish connection with UE
![image](https://hackmd.io/_uploads/S13XRaI4yx.png)
- UE change from RNTI 9905 to 4a91
![image](https://hackmd.io/_uploads/Sy2jaTUE1g.png)
- CU logged handover as complete
![image](https://hackmd.io/_uploads/B1uL66U41e.png)
- UE is now attached to DU 2
![image](https://hackmd.io/_uploads/rydLRT8Nke.png)
