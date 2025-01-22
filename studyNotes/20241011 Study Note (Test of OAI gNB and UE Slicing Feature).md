# 2024/10/11 Study Note (Test of OAI gNB and UE Slicing Feature)

###### tags: `2024`


**Goal:**
- [x] [Test OAI gNB and OAI UE `develop` branch Slicing Feature without modification (UE not request slice)](#1425-Run-2-UE-with-different-slice-configuration)
- [x] [Test OAI gNB and OAI UE `develop` branch Slicing Feature without modification (UE not request slice) and Free5GC](#2423-2-UE-with-2-different-slice)
- [x] [Test OAI gNB and OAI UE NEU version Slicing Feature](#6-NEU-version-ORANSlice-with-open5gs-without-multi-pdu)
- [ ] Test OAI gNB and OAI UE `rebased-5g-nvs-rc-rf` branch Slicing Feature -> Refer to [part 2](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20241106%20Study%20Note%20(Test%20of%20OAI%20gNB%20and%20UE%20Slicing%20Feature)%20part%202.md) of note

**References:**
- [OAI | NR SA Tutorial OAI nrUE](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md)
- [Wilfrid | RAN Slicing Architecture Requirements](https://hackmd.io/@superwilfrid/ry21MNnqC)
- [Wilfrid | ORANSlice: An Open-Source 5G Network Slicing Platform for O-RAN](https://hackmd.io/@superwilfrid/SJ5G4zLgJl#2-What-they-add-in-OAI-nrUE)
- [sharetechnote | 5G/NR  - Network Slicing](https://www.sharetechnote.com/html/5G/5G_NetworkSlicing.html)
![image](https://hackmd.io/_uploads/rkhGBXCgyl.png)

**Table of Contents:**
- [2024/10/11 Study Note (Test of OAI gNB and UE Slicing Feature)](#2024-10-11-study-note--test-of-oai-gnb-and-ue-slicing-feature-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
  * [1. Develop branch without Modification](#1-develop-branch-without-modification)
    + [1.0. Minimum Requirement](#10-minimum-requirement)
      - [1.0.1. OAI gNB with rfsim](#101-oai-gnb-with-rfsim)
      - [1.0.2. OAI UE with rfsim](#102-oai-ue-with-rfsim)
    + [1.1. Topology](#11-topology)
    + [1.2. Environment](#12-environment)
      - [1.2.1. OAI gNB and UE](#121-oai-gnb-and-ue)
    + [1.3. Compile](#13-compile)
    + [1.4. Run](#14-run)
      - [1.4.1. Configuration](#141-configuration)
        * [1.4.1.1. CN Configuration](#1411-cn-configuration)
        * [1.4.1.2. gNB Configuration](#1412-gnb-configuration)
        * [1.4.1.3. UE Configuration](#1413-ue-configuration)
      - [1.4.2. Result](#142-result)
        * [1.4.2.1. Initial Run](#1421-initial-run)
        * [1.4.2.2. Modify OAI UE slice configuration](#1422-modify-oai-ue-slice-configuration)
        * [1.4.2.3. Modify OAI gNB & CN slice configuration](#1423-modify-oai-gnb---cn-slice-configuration)
        * [1.4.2.4. Modify OAI gNB & CN slice configuration](#1424-modify-oai-gnb---cn-slice-configuration)
        * [1.4.2.5. Run 2 UE with different slice configuration](#1425-run-2-ue-with-different-slice-configuration)
        * [1.4.2.6. Add 2 slice configuration for UE in CN](#1426-add-2-slice-configuration-for-ue-in-cn)
  * [2. Develop branch without Modification and Free5GC](#2-develop-branch-without-modification-and-free5gc)
    + [2.0. Minimum Requirement](#20-minimum-requirement)
      - [2.0.1. OAI gNB with rfsim](#201-oai-gnb-with-rfsim)
      - [2.0.2. OAI UE with rfsim](#202-oai-ue-with-rfsim)
    + [2.1. Topology](#21-topology)
    + [2.2. Environment](#22-environment)
      - [2.2.1. OAI gNB and UE](#221-oai-gnb-and-ue)
      - [2.2.2. free5gc](#222-free5gc)
    + [2.3. Compile](#23-compile)
    + [2.4. Run](#24-run)
      - [2.4.1. Configuration](#241-configuration)
        * [2.4.1.1. CN Configuration](#2411-cn-configuration)
        * [2.4.1.2. gNB Configuration](#2412-gnb-configuration)
        * [2.4.1.3. UE Configuration](#2413-ue-configuration)
        * [2.4.1.4. Add UE's IMSI to CN](#2414-add-ue-s-imsi-to-cn)
      - [2.4.2. Result](#242-result)
        * [2.4.2.1. Initial Run](#2421-initial-run)
        * [2.4.2.2. Do small modification so that OAI nrUE send Capability5GMM](#2422-do-small-modification-so-that-oai-nrue-send-capability5gmm)
        * [2.4.2.3. 2 UE with 2 different slice](#2423-2-ue-with-2-different-slice)
  * [3. NEU version OAI-Slicing-Intel](#3-neu-version-oai-slicing-intel)
    + [3.0. Minimum Requirement](#30-minimum-requirement)
      - [3.0.1. OAI gNB with rfsim](#301-oai-gnb-with-rfsim)
      - [3.0.2. OAI UE with rfsim](#302-oai-ue-with-rfsim)
    + [3.1. Topology](#31-topology)
    + [3.2. Environment](#32-environment)
      - [3.2.1. OAI gNB and UE](#321-oai-gnb-and-ue)
      - [3.2.2. free5gc](#322-free5gc)
    + [3.3. Compile](#33-compile)
      - [3.3.1. OAI UE](#331-oai-ue)
      - [3.3.2. OAI gnB and ue_slice_manager](#332-oai-gnb-and-ue-slice-manager)
    + [3.4. Run](#34-run)
      - [3.4.1. Configuration](#341-configuration)
        * [3.4.1.1. CN Configuration](#3411-cn-configuration)
        * [3.4.1.2. gNB Configuration](#3412-gnb-configuration)
        * [3.4.1.3. UE Configuration](#3413-ue-configuration)
      - [3.4.2. Result](#342-result)
        * [3.4.2.1. Initial Run](#3421-initial-run)
  * [4. NEU version ORANSlice](#4-neu-version-oranslice)
    + [4.0. Minimum Requirement](#40-minimum-requirement)
      - [4.0.1. OAI gNB with rfsim](#401-oai-gnb-with-rfsim)
      - [4.0.2. OAI UE with rfsim](#402-oai-ue-with-rfsim)
    + [4.1. Topology](#41-topology)
    + [4.2. Environment](#42-environment)
      - [4.2.1. OAI gNB and UE](#421-oai-gnb-and-ue)
      - [4.2.2. free5gc](#422-free5gc)
    + [4.3. Compile](#43-compile)
      - [4.3.1. OAI UE](#431-oai-ue)
      - [4.3.2. OAI gnB](#432-oai-gnb)
    + [4.4. Run](#44-run)
      - [4.4.1. Configuration](#441-configuration)
        * [4.4.1.1. CN Configuration](#4411-cn-configuration)
        * [4.4.1.2. gNB Configuration](#4412-gnb-configuration)
        * [4.4.1.3. UE Configuration](#4413-ue-configuration)
      - [4.4.2. Result](#442-result)
        * [4.4.2.1. Initial Run](#4421-initial-run)
  * [5. NEU version ORANSlice with open5gs](#5-neu-version-oranslice-with-open5gs)
    + [5.0. Minimum Requirement](#50-minimum-requirement)
      - [5.0.1. OAI gNB with rfsim](#501-oai-gnb-with-rfsim)
      - [5.0.2. OAI UE with rfsim](#502-oai-ue-with-rfsim)
    + [5.1. Topology](#51-topology)
    + [5.2. Environment](#52-environment)
      - [5.2.1. OAI gNB and UE](#521-oai-gnb-and-ue)
    + [5.3. Compile](#53-compile)
      - [5.3.1. OAI UE](#531-oai-ue)
      - [5.3.2. OAI gnB](#532-oai-gnb)
    + [5.4. Run](#54-run)
      - [5.4.1. Configuration](#541-configuration)
        * [5.4.1.1. CN Configuration](#5411-cn-configuration)
        * [5.4.1.2. gNB Configuration](#5412-gnb-configuration)
        * [5.4.1.3. UE Configuration](#5413-ue-configuration)
      - [5.4.2. Result](#542-result)
        * [5.4.2.1. Initial Run](#5421-initial-run)
        * [5.4.2.2. Change UE nssai_sd](#5422-change-ue-nssai-sd)
        * [5.4.2.3. Delete UE nssai_sd](#5423-delete-ue-nssai-sd)
  * [6. NEU version ORANSlice with open5gs without multi pdu](#6-neu-version-oranslice-with-open5gs-without-multi-pdu)
    + [6.0. Minimum Requirement](#60-minimum-requirement)
      - [6.0.1. OAI gNB with rfsim](#601-oai-gnb-with-rfsim)
      - [6.0.2. OAI UE with rfsim](#602-oai-ue-with-rfsim)
    + [6.1. Topology](#61-topology)
    + [6.2. Environment](#62-environment)
      - [6.2.1. OAI gNB and UE](#621-oai-gnb-and-ue)
    + [6.3. Compile](#63-compile)
      - [6.3.1. OAI gnB and UE](#631-oai-gnb-and-ue)
    + [6.4. Run](#64-run)
      - [6.4.1. Configuration](#641-configuration)
        * [6.4.1.1. CN Configuration](#6411-cn-configuration)
        * [6.4.1.2. gNB Configuration](#6412-gnb-configuration)
        * [6.4.1.3. UE Configuration](#6413-ue-configuration)
      - [6.4.2. Result](#642-result)
        * [6.4.2.1. Initial Run](#6421-initial-run)
        * [6.4.2.2. Change Slice RRM Policy to minimal](#6422-change-slice-rrm-policy-to-minimal)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 0. Summary

1. OAI `develop` branch has gNB and nrUE that can identify snssai for slice. But will treat every slice (in MAC SCH) the same and cannot accept rrmPolicy. [Details](#1-Develop-branch-without-Modification)
2. OAI `develop` branch has gNB and nrUE that can connect to free5gc by little modification in code (nrUE). [Details](#2-Develop-branch-without-Modification-and-Free5GC)
3. OAI NEU version (ORANSlice) has gNB that will treat slice (in MAC SCH) based on rrmPolicy. [Details](#6-NEU-version-ORANSlice-with-open5gs-without-multi-pdu)
4. OAI NEU version (ORANSlice)'s nrUE that has multi-PDU session feature currently still has a lot of problem (segmentation fault) because it is based on older version of OAI. [Details](#5-NEU-version-ORANSlice-with-open5gs)

## 1. Develop branch without Modification

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

![image](https://hackmd.io/_uploads/Hk1-TJAJkg.png)

### 1.2. Environment

#### 1.2.1. OAI gNB and UE

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
| OAI Commit | e15fa14c5758d74dde78d6c01b8eaf1d7cc07261  (HEAD -> develop, tag: 2024.w40, origin/develop, origin/HEAD) |

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
![image](https://hackmd.io/_uploads/SysI2I9JJg.png)


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


<b>4. Build OAI gNB</b>

```shell=
# Build OAI gNB
cd ~/openairinterface5g/cmake_targets
./build_oai -w SIMU --ninja --nrUE --gNB --build-lib "nrscope" -C
```

![image](https://hackmd.io/_uploads/B1wQyv911l.png)

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
```

##### 1.4.1.2. gNB Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf`</b>

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
plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2; snssaiList = ({ sst = 1}) });
```
- min_rxtxtime (modify as your desired min_rxtxtime. Values that I use is below)
```shell=
# i use this because i cannot pass --gNBs.[0].min_rxtxtime 6 when running OAI gNB
gNBs =
(
 {
    ...
    min_rxtxtime = 6; # value 6 is because default USRP value
    ...
```


##### 1.4.1.3. UE Configuration

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
```
- dnn (modify as your desired DNN. Values that I use is below)
```shell=
dnn="internet";
```

#### 1.4.2. Result

##### 1.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
```


![image](https://hackmd.io/_uploads/SkyYj6sJ1l.png)


```shell=
screen -r amf
```


![image](https://hackmd.io/_uploads/HkXCopoyyl.png)



<b>2. Run OAI gNB</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
# sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --sa
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --rfsim --sa
```


![image](https://hackmd.io/_uploads/rJnN-g0y1g.png)


<b>3. Run OAI nrUE</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```


![image](https://hackmd.io/_uploads/r1ZJZx0J1g.png)


<b>4. Stop OAI UE, OAI gNB and open5gs</b>

```shell=
# To stop OAI UE, just ctrl + c on OAI UE terminal

# To stop OAI UE, just ctrl + c on OAI gNB terminal

# Stop open5gs
./stop_open5gs.sh
```


![image](https://hackmd.io/_uploads/BJPqxeRJ1x.png)


<b>5. Result explanation</b>
- Since we set the 5G System (CN, RAN & UE) to have only 1 slice, we can see that there is only 1 slice (both on Allowed NSSAI & PDU Session)
![image](https://hackmd.io/_uploads/B1jcteCkyl.png)
![image](https://hackmd.io/_uploads/Hk5IceR1Jg.png)

##### 1.4.2.2. Modify OAI UE slice configuration

<b>0. Configuration change</b>
- We will modify the UE slice configuration to test if UE will request different slice to CN
```shell=
nssai_sst=2;
```

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
```

```shell=
screen -r amf
```

<b>2. Run OAI gNB</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
# sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --sa
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --rfsim --sa
```

<b>3. Run OAI nrUE</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```


![image](https://hackmd.io/_uploads/ryrUTeAJyx.png)


<b>4. Result explanation</b>
- Since we set the 5G System (CN & RAN) to have only 1 slice sst=1, we can see that there is only 1 slice on Allowed NSSAI
![image](https://hackmd.io/_uploads/SkpakXRJkx.png)

- UE will check this allowed slice and compare to slice configuration from configuration file. If not the same, then UE will not do PDU Session Establishment Request
![image](https://hackmd.io/_uploads/ryrUTeAJyx.png)

##### 1.4.2.3. Modify OAI gNB & CN slice configuration

- [Source 1](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>0. Configuration change</b>
- We will modify the CN slice configuration to test if CN will send 2 different slice to UE
```shell=
...
amf:
    ...
    s_nssai:
      - sst: 1
      - sst: 2
    ...
...
nssf:
    ...
    nsi:
      - addr: 127.0.0.10
        port: 7777
        s_nssai:
          sst: 1
      - addr: 127.0.0.10
        port: 7777
        s_nssai:
          sst: 2
    ...
...
```

- We will modify the gNB slice configuration also
```shell=
plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2; snssaiList = ({ sst = 1; }, { sst = 2; }) });
```

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
screen -r amf
```


![image](https://hackmd.io/_uploads/HkLgRmAyJl.png)


<b>2. Run OAI gNB</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
# sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --sa
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --rfsim --sa
```


![image](https://hackmd.io/_uploads/HJKnRQA1yx.png)


<b>3. Run OAI nrUE</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```


![image](https://hackmd.io/_uploads/ByOoyVA1yg.png)


<b>4. Result explanation</b>
- UE did not request PDU Session because its allowed slice is different with CN's allowed slice
![image](https://hackmd.io/_uploads/ByOoyVA1yg.png)

- We find that CN only send 1 allowed slice for UE eventhough we configure 2 network slice for the 5GS (CN & RAN)
![image](https://hackmd.io/_uploads/S1ZQwSRkJx.png)

##### 1.4.2.4. Modify OAI gNB & CN slice configuration

<b>0. Change UE Slice from CN</b>
- Since UE only get 1 slice sst=1 in allowed slice, maybe the problem is UE's slice setting in the CN
- We try to change UE's slice in the open5gs webui:
    - Before:
danger
![image](https://hackmd.io/_uploads/SyQY0HA1yx.png)

    - After:

![image](https://hackmd.io/_uploads/ByrhRHA1Jl.png)


<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
screen -r amf
```

<b>2. Run OAI gNB</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
# sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --sa
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --rfsim --sa
```

<b>3. Run OAI nrUE</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```


![image](https://hackmd.io/_uploads/HyDjJ8AJyg.png)


<b>4. Result explanation</b>
- UE request PDU Session because the allowed slice from CN for this UE is the same with UE slice configuration
![image](https://hackmd.io/_uploads/HyDjJ8AJyg.png)

- We find that CN only send 1 allowed slice for UE eventhough we configure 2 network slice for the 5GS (CN & RAN). So UE Slice configuration in the CN really depends on the value we set in CN
![image](https://hackmd.io/_uploads/ry3GzIAJJe.png)

- Another interesting value is the CN ask gNB to prepare PDU Session for slice sst=2

##### 1.4.2.5. Run 2 UE with different slice configuration

<b>0. Make a second UE</b>
- Let's make a second UE in the open5gs webui. Set the IMSI to be different than the first UE
![image](https://hackmd.io/_uploads/r1xVNUR1yx.png)

- Let's set it to slice sst=1
![image](https://hackmd.io/_uploads/H12SELAkke.png)

- Let's create configuration file for UE 2 by copying UE 1 configuration and change its IMSI
```shell=
nssai_sst=1;
```

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
screen -r amf
```

<b>2. Run OAI gNB</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
# sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --sa
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --rfsim --sa
```

<b>3. Run OAI nrUE 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```


![image](https://hackmd.io/_uploads/SybpU8Ckke.png)


<b>4. Run OAI nrUE 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue2.uicc.conf
```


![image](https://hackmd.io/_uploads/r1wfPUAyyg.png)


<b>5. Result explanation</b>
- Both UE successfully register and request PDU Session

- We can see that UE 1 PDU Session has slice sst=2
![image](https://hackmd.io/_uploads/H1tnP8RJ1l.png)

- We can see that UE 2 PDU Session has slice sst=1
![image](https://hackmd.io/_uploads/H1bGuUAkke.png)

##### 1.4.2.6. Add 2 slice configuration for UE in CN 

<b>0. Allow 2 slice for UE 1</b>
- Let's add another slice for UE 1 in the open5gs webui
![image](https://hackmd.io/_uploads/HkdeDjygkl.png)
![image](https://hackmd.io/_uploads/ryYrYoJx1e.png)

- Here, <u>we set both sst=1 & sst=2 as default s-NSSAI</u> which will result in <u>UE get 2 allowed NSSAI</u>. If we only set either 1 slice as default s-NSSAI, UE will only get 1 allowed NSSAI

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
screen -r amf
```

<b>2. Run OAI gNB</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
# sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --sa
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --rfsim --sa
```

<b>3. Run OAI nrUE 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```


![image](https://hackmd.io/_uploads/ByKcqj1e1e.png)


<b>5. Result explanation</b>
- UE successfully register and request PDU Session
![image](https://hackmd.io/_uploads/B1BsqjJxJx.png)

- We can see that UE 1 get 2 allowed slice sst=1 and sst=2
![image](https://hackmd.io/_uploads/S1lPkjsyxke.png)

- Since OAI UE only request PDU session of the slice from the config file (and it has to be same with allowed slice), UE 1 request PDU session with sst=2
![image](https://hackmd.io/_uploads/S15Kos1gJe.png)


## 2. Develop branch without Modification and Free5GC


| Step                                                                                     | Status             |
| ---------------------------------------------------------------------------------------- | ------------------ |
| NG Setup                                                                                 | :heavy_check_mark: |
| F1 Setup                                                                                 | :heavy_check_mark: |
| DU Configuration Update                                                                  | :heavy_check_mark: |
| UE get mib and sib1 for DL sync                                                          | :heavy_check_mark: |
| UE send PRACH for UL sync                                                                | :heavy_check_mark: |
| UE get RAR message                                                                       | :heavy_check_mark: |
| UE send RRC setup request                                                                | :heavy_check_mark: |
| RRC setup complete                                                                       | :heavy_check_mark: |
| NAS registration                                                                         | :heavy_check_mark: |
| [PDU session complete](#2422-Do-small-modification-so-that-OAI-nrUE-send-Capability5GMM) | :heavy_check_mark: |

### 2.0. Minimum Requirement

- [Source](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/system_requirements.md)

#### 2.0.1. OAI gNB with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 4GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

#### 2.0.2. OAI UE with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 3GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

### 2.1. Topology

![image](https://hackmd.io/_uploads/BkgSDJtl1l.png)

### 2.2. Environment

#### 2.2.1. OAI gNB and UE

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

| Item       | Info                                                                                                   |
| ---------- | ------------------------------------------------------------------------------------------------------ |
| OS         | Ubuntu 22.04.5 LTS (jammy)                                                                             |
| Kernel     | 5.15.0-1071-realtime                                                                                   |
| OAI Commit | e15fa14c5758d74dde78d6c01b8eaf1d7cc07261 (HEAD -> develop, tag: 2024.w40, origin/develop, origin/HEAD) |

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


#### 2.2.2. free5gc

<b>Software:</b>

| Item   | Info                                                         |
| ------ | ------------------------------------------------------------ |
| Commit | 470b8bff3ff2cd4e485bf54cb4c6bfe7e285793e (HEAD, tag: v3.3.0) |

Command Line Codes
```shell=
# Check free5gc commit
git log -1
```



### 2.3. Compile

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
![image](https://hackmd.io/_uploads/SysI2I9JJg.png)


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


<b>4. Build OAI gNB</b>

```shell=
# Build OAI gNB
cd ~/openairinterface5g/cmake_targets
./build_oai -w SIMU --ninja --nrUE --gNB --build-lib "nrscope" -C
```


![image](https://hackmd.io/_uploads/B1wQyv911l.png)


### 2.4. Run

#### 2.4.1. Configuration

##### 2.4.1.1. CN Configuration

- [Source 1](https://hackmd.io/@Yueh-Huan/S1eC0g3Gi)
- [Source 2](https://free5gc.org/guide/Configuration/)

<b>1. Be aware that you might need to change 2 things in the amf configuration file (`amfcfg.yaml`):</b>
- mcc mnc (modify as your desired mnc mcc. Values that I use is below)
```shell=
- plmnid:
    mcc:001
    mnc:01
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
snssaiList:
    - sst: 1
      sd: 010203
    - sst: 1
      sd: 112233
```

<b>2. Be aware that you might need to change 2 things in the smf configuration file (`smfcfg.yaml`):</b>
- mcc mnc (modify as your desired mnc mcc. Values that I use is below)
```shell=
plmnList:
    - mcc:001
      mnc:01
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
snssaiInfos:
    - sNssai:
        sst: 1
        sd: 010203
      ...
    - sNssai:
        sst: 1
        sd: 112233
      ...
```
```shell=
sNssaiUpfInfos:
    - sNssai:
        - sst: 1
          sd: 010203
      ...
    - sNssai:
        - sst: 1
          sd: 112233
      ...
```

<b>3. Be aware that you might need to change 3 things in the nssf configuration file (`nssfcfg.yaml`):</b>
- mcc mnc (modify as your desired mnc mcc. Values that I use is below)
```shell=
supportedPlmnList:
    - mcc:001
      mnc:01
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
supportedSnssaiList:
    - sst: 1
      sd: 010203
    - sst: 1
      sd: 112233
```
- nsi (modify as your desired network slice instance. Values that I use is below)
```shell=
nsiList:
    - snssai:
        sst: 1
        sd: 010203
      nsiInformationList:
          - nrfId:
            nsiId: 22
    - snssai:
        sst: 1
        sd: 112233
      nsiInformationList:
          - nrfId:
            nsiId: 23
```



##### 2.4.1.2. gNB Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf`</b>

<b>2. Be aware that you might need to change 3 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "192.168.8.21"; }); 


NETWORK_INTERFACES :
{
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.8.53"; 
    GNB_IPV4_ADDRESS_FOR_NGU                 = "192.168.8.53"; 
    GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
};
```
- PLMN List (modify as your desired Slice Configuration. Values that I use is below)
```shell=
plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0x010203 }, { sst = 1, sd = 0x112233 }) });
```
- min_rxtxtime (modify as your desired min_rxtxtime. Values that I use is below)
```shell=
# i use this because i cannot pass --gNBs.[0].min_rxtxtime 6 when running OAI gNB
gNBs =
(
 {
    ...
    min_rxtxtime = 6; # value 6 is because default USRP value
    ...
```

##### 2.4.1.3. UE Configuration

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

##### 2.4.1.4. Add UE's IMSI to CN

- [Source 1](https://free5gc.org/guide/5-install-ueransim/#3-install-free5gc-webconsole)

<b>1. We need to add UE's IMSI to free5gc database</b>
![image](https://hackmd.io/_uploads/B1icCDneJl.png)

<b>2. Remember to set the UE's slice</b>
![image](https://hackmd.io/_uploads/BJw0Rv3l1e.png)

<b>3. Directly add imsi for 2 UE</b>
![image](https://hackmd.io/_uploads/Bkgfkung1e.png)

#### 2.4.2. Result

##### 2.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>1. Run free5gc CN</b>

```shell=
cd ~/free5gc/
./run.sh
```


![image](https://hackmd.io/_uploads/SJva7d3gkl.png)


<b>2. Run OAI gNB</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
# sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --sa
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --rfsim --sa
```


![image](https://hackmd.io/_uploads/ryf8r_2x1x.png)


<b>3. Run OAI nrUE</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```

danger
![image](https://hackmd.io/_uploads/Bk1a6_nl1g.png)


<b>4. Result explanation</b>
- AMF does not have Capability5GMM and will send registration reject to UE
![image](https://hackmd.io/_uploads/S1Mf0_3lke.png)

##### 2.4.2.2. Do small modification so that OAI nrUE send Capability5GMM

<b>0. Modify `/openair3/NAS/NR_UE/nr_nas_msg_sim.c` to send `5GMM_CAPABILITY`</b>
- Modify the `nr_nas_msg_sim.c` source code
```c=524
//#if 0
  /* This cannot be sent in clear, the core network Open5GS rejects the UE.
   * TODO: do we have to send this at some point?
   * For the time being, let's keep it here for later proper fix.
   */
  mm_msg->registration_request.presencemask |= REGISTRATION_REQUEST_5GMM_CAPABILITY_PRESENT;
  mm_msg->registration_request.fgmmcapability.iei = REGISTRATION_REQUEST_5GMM_CAPABILITY_IEI;
  mm_msg->registration_request.fgmmcapability.length = 1;
  mm_msg->registration_request.fgmmcapability.value = 0x7;
  size += 3;
//#endif
```

- Remember to recompile OAI, particularlly the OAI nrUE
```shell=
# Build OAI gNB
cd ~/openairinterface5g/cmake_targets
./build_oai -w SIMU --ninja --nrUE --gNB --build-lib "nrscope" -C
```
![image](https://hackmd.io/_uploads/Bk4Z0nhekg.png)

<b>1. Run free5gc CN</b>

```shell=
cd ~/free5gc/
./run.sh
```

<b>2. Run OAI gNB</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
# sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --sa
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --rfsim --sa
```

<b>3. Run OAI nrUE</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```


![image](https://hackmd.io/_uploads/Hk4xtThlyl.png)


<b>4. Result explanation</b>
- Since we modified nrUE code to send `5GMM_CAPABILITY`, AMF accept UE registration
![image](https://hackmd.io/_uploads/BkSpda2xyx.png)

- We can see that UE 1 get 2 allowed slice sst=1 sd=010203 and sd=112233
![image](https://hackmd.io/_uploads/ByjCERhxJe.png)

- OAI UE only request PDU session of the slice from the config file (and it has to be same with allowed slice), UE 1 request PDU session with sst=1 sd=010203
![image](https://hackmd.io/_uploads/Skk2SA2xJx.png)

##### 2.4.2.3. 2 UE with 2 different slice

<b>0. Modify `/ci-scripts/conf_files/nrue2.uicc.conf` to set the correct slice</b>
```c=
nssai_sst=1;
nssai_sd=1122867;
```

<b>1. Run free5gc CN</b>

```shell=
cd ~/free5gc/
./run.sh
```

<b>2. Run OAI gNB</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
# sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --sa
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --rfsim --sa
```

<b>3. Run OAI nrUE 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```


![image](https://hackmd.io/_uploads/Bkk_OR2xJe.png)


<b>4. Run OAI nrUE 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue2.uicc.conf
```


![image](https://hackmd.io/_uploads/HJN3_02l1e.png)


<b>5. Result explanation</b>
- We can see that UE 1 get 2 allowed slice but only request PDU session in sst=1 sd=010203
![image](https://hackmd.io/_uploads/H1QBFRhlJl.png)

- We can see that UE 1 get 2 allowed slice but only request PDU session in sst=1 sd=112233
![image](https://hackmd.io/_uploads/BJsPcC2xyx.png)


## 3. NEU version OAI-Slicing-Intel

| Step                                                 | Status             |
| ---------------------------------------------------- | ------------------ |
| NG Setup                                             | :heavy_check_mark: |
| F1 Setup                                             | :heavy_check_mark: |
| DU Configuration Update                              | :heavy_check_mark: |
| [UE get mib and sib1 for DL sync](#3421-Initial-Run) | :gear:             |
| UE send PRACH for UL sync                            | :x:                |
| UE get RAR message                                   | :x:                |
| UE send RRC setup request                            | :x:                |
| RRC setup complete                                   | :x:                |
| NAS registration                                     | :x:                |
| PDU session complete                                 | :x:                |

 

### 3.0. Minimum Requirement

- [Source](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/system_requirements.md)

#### 3.0.1. OAI gNB with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 4GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

#### 3.0.2. OAI UE with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 3GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

### 3.1. Topology

![image](https://hackmd.io/_uploads/ryUGLvyWyg.png)

### 3.2. Environment

#### 3.2.1. OAI gNB and UE

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

| Item               | Info                                                                                                                  |
| ------------------ | --------------------------------------------------------------------------------------------------------------------- |
| OS                 | Ubuntu 22.04.5 LTS (jammy)                                                                                            |
| Kernel             | 5.15.0-1071-realtime                                                                                                  |
| OAI Commit for gNB | fd3b52bbbb47a0a17cfb56c8a304f38459cef89e (HEAD -> NR_UE_multi_pdusession, origin/NR_UE_multi_pdusession, origin/HEAD) |
| OAI Commit for UE  | 458be35862258c7dbdbb38b0fecdf12ce8aac4da (HEAD -> NR_UE_multi_pdu, origin/NR_UE_multi_pdu)                            |

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


#### 3.2.2. free5gc

<b>Software:</b>

| Item   | Info                                                         |
| ------ | ------------------------------------------------------------ |
| Commit | 470b8bff3ff2cd4e485bf54cb4c6bfe7e285793e (HEAD, tag: v3.3.0) |

Command Line Codes
```shell=
# Check free5gc commit
git log -1
```



### 3.3. Compile

NEU's version of OAI is divided into 2 parts:
- OAI UE with Multi PDU Session is located in OAI gitlab under [`NR_UE_multi_pdu`](https://gitlab.eurecom.fr/oai/openairinterface5g/-/tree/NR_UE_multi_pdu?ref_type=heads) branch
- OAI gNB with rrm policy support and `ue_slice_manager` to control OAI UE pdu session is located in wineslab [OAI-Slicing-Intel](https://github.com/wineslab/OAI-Slicing-Intel/tree/NR_UE_multi_pdusession) repository

I have checked the 2 repositories and none have all the complete functions. So we need to download both of them

#### 3.3.1. OAI UE

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#32-build-oai-gnb-and-oai-nrue)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. Get openairinterface5g source code</b>

```shell=
# Get openairinterface5g source code
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
cd openairinterface5g
git checkout NR_UE_multi_pdu
git log -1
```


![image](https://hackmd.io/_uploads/BJ2n2S1-Jg.png)
![image](https://hackmd.io/_uploads/HkwepH1-1g.png)
![image](https://hackmd.io/_uploads/r1cGTrJbJx.png)


<b>2. Install OAI dependencies</b>

```shell=
# Install OAI dependencies
cd cmake_targets
./build_oai -I
```


![image](https://hackmd.io/_uploads/ry5x0rkZ1l.png)


<b>3. nrscope dependencies</b>

```shell=
# nrscope dependencies
sudo apt install -y libforms-dev libforms-bin
```


![image](https://hackmd.io/_uploads/rkCXAHJWye.png)


<b>4. Build OAI UE</b>

```shell=
# Build OAI UE
cd cmake_targets
./build_oai -w SIMU --ninja --nrUE --build-lib "nrscope" -C
```


![image](https://hackmd.io/_uploads/HyfVkLybkl.png)


#### 3.3.2. OAI gnB and ue_slice_manager

- [Source 1](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_slicing_xApp.md)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. Get wineslab's openairinterface5g source code</b>

```shell=
# Get wineslab's openairinterface5g source code
git clone https://github.com/wineslab/OAI-Slicing-Intel.git
cd OAI-Slicing-Intel
git checkout NR_UE_multi_pdusession
git log -1
```


![image](https://hackmd.io/_uploads/HJiOeLyZ1e.png)
![image](https://hackmd.io/_uploads/SksJ-8kb1e.png)
![image](https://hackmd.io/_uploads/H1eb-LkW1x.png)


<b>2. Comment out lines 725~726 & 769 in `radio/rfsimulator/simulator.c`</b>

```c=722
...
// check the header and start block transfer
if ( b->headerMode==true && b->remainToTransfer==0) {
//    AssertFatal( (t->typeStamp == UE_MAGICDL  && b->th.magic==ENB_MAGICDL) ||
//                 (t->typeStamp == ENB_MAGICDL && b->th.magic==UE_MAGICDL), "Socket Error in protocol");
    b->headerMode=false;
...
```
```c=769
LOG_W(HW,"UEsock: %d Tx/Rx shift too large Tx:%lu, Rx:%lu\n", fd, t->lastWroteTS, b->lastReceivedTS);
```


<b>3. Install OAI dependencies</b>

```shell=
# Install OAI dependencies
cd cmake_targets
./build_oai -I
```


![image](https://hackmd.io/_uploads/B1WyGIybkl.png)


<b>4. nrscope dependencies</b>

```shell=
# nrscope dependencies
sudo apt install -y libforms-dev libforms-bin
```


![image](https://hackmd.io/_uploads/BkFlGI1bke.png)


<b>5. Build OAI gNB</b>

```shell=
# Build OAI gNB
cd cmake_targets
sudo ./build_oai --ninja --gNB --build-lib "nrscope" -C
#cd ran_build/build
#make rfsimulator
```


![image](https://hackmd.io/_uploads/HyGsULkW1e.png)


<b>6. Build ue_slice_manager</b>

```shell=
# Build OAI gNB
cd cmake_targets/ran_build/build
sudo ninja ue_slice_manager
```


![image](https://hackmd.io/_uploads/B1IKu8JWyl.png)


### 3.4. Run

#### 3.4.1. Configuration

##### 3.4.1.1. CN Configuration

- [Source 1](https://hackmd.io/@Yueh-Huan/S1eC0g3Gi)
- [Source 2](https://free5gc.org/guide/Configuration/)

<b>1. Be aware that you might need to change 2 things in the amf configuration file (`amfcfg.yaml`):</b>
- mcc mnc (modify as your desired mnc mcc. Values that I use is below)
```shell=
- plmnid:
    mcc:001
    mnc:01
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
snssaiList:
    - sst: 1
      sd: 010203
    - sst: 1
      sd: 112233
```

<b>2. Be aware that you might need to change 2 things in the smf configuration file (`smfcfg.yaml`):</b>
- mcc mnc (modify as your desired mnc mcc. Values that I use is below)
```shell=
plmnList:
    - mcc:001
      mnc:01
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
snssaiInfos:
    - sNssai:
        sst: 1
        sd: 010203
      ...
    - sNssai:
        sst: 1
        sd: 112233
      ...
```
```shell=
sNssaiUpfInfos:
    - sNssai:
        - sst: 1
          sd: 010203
      ...
    - sNssai:
        - sst: 1
          sd: 112233
      ...
```

<b>3. Be aware that you might need to change 3 things in the nssf configuration file (`nssfcfg.yaml`):</b>
- mcc mnc (modify as your desired mnc mcc. Values that I use is below)
```shell=
supportedPlmnList:
    - mcc:001
      mnc:01
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
supportedSnssaiList:
    - sst: 1
      sd: 010203
    - sst: 1
      sd: 112233
```
- nsi (modify as your desired network slice instance. Values that I use is below)
```shell=
nsiList:
    - snssai:
        sst: 1
        sd: 010203
      nsiInformationList:
          - nrfId:
            nsiId: 22
    - snssai:
        sst: 1
        sd: 112233
      nsiInformationList:
          - nrfId:
            nsiId: 23
```



##### 3.4.1.2. gNB Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.slice.conf`</b>

<b>2. Be aware that you might need to change 2 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "192.168.8.21";
                    ipv6 = "fe80::d728:2dd8:567b:1805";
                    active = "yes";
                    preference = "ipv4";
                    }
                ); 


NETWORK_INTERFACES :
{
    GNB_INTERFACE_NAME_FOR_NG_AMF            = "demo-oai"; 
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.8.53"; 
    GNB_INTERFACE_NAME_FOR_NGU               = "192.168.8.53"; 
    GNB_IPV4_ADDRESS_FOR_NGU                 = "192.168.8.53"; 
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

##### 3.4.1.3. UE Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#51-run-oai-nrue)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf`</b>

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

#### 3.4.2. Result

##### 3.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. Run free5gc CN</b>

```shell=
cd ~/free5gc/
sudo ./run.sh
```


![image](https://hackmd.io/_uploads/Hy0ofO1Zke.png)


<b>2. Run OAI gNB</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.slice.conf --sa --rfsim -E --continuous-tx
```


![image](https://hackmd.io/_uploads/r1vQXdy-1e.png)


<b>3. Run OAI nrUE</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --nokrnmod --sa -r 106 --numerology 1 --band 78 -C 3619200000 --ue-fo-compensation -E -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf --net-slice
```

danger
gNB will experience segmentation fault in process of RACH with UE


<b>4. Result explanation</b>
- gNB experience segmentation fault because it use the older version of OAI



## 4. NEU version ORANSlice

| Step                                  | Status             |
| ------------------------------------- | ------------------ |
| NG Setup                              | :heavy_check_mark: |
| F1 Setup                              | :heavy_check_mark: |
| DU Configuration Update               | :heavy_check_mark: |
| UE get mib and sib1 for DL sync       | :heavy_check_mark: |
| UE send PRACH for UL sync             | :heavy_check_mark: |
| UE get RAR message                    | :heavy_check_mark: |
| UE send RRC setup request             | :heavy_check_mark: |
| RRC setup complete                    | :heavy_check_mark: |
| [NAS registration](#4421-Initial-Run) | :gear:             |
| PDU session complete                  | :x:                |

  

### 4.0. Minimum Requirement

- [Source](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/system_requirements.md)

#### 4.0.1. OAI gNB with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 4GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

#### 4.0.2. OAI UE with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 3GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

### 4.1. Topology

![image](https://hackmd.io/_uploads/H11Gsmw-Jx.png)

### 4.2. Environment

#### 4.2.1. OAI gNB and UE

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

| Item               | Info                                                                                       |
| ------------------ | ------------------------------------------------------------------------------------------ |
| OS                 | Ubuntu 22.04.5 LTS (jammy)                                                                 |
| Kernel             | 5.15.0-1071-realtime                                                                       |
| OAI Commit for gNB | 8d3a156a809304a9f7a6036faf422b11951e547a (HEAD -> main, origin/main, origin/HEAD)          |
| OAI Commit for UE  | 458be35862258c7dbdbb38b0fecdf12ce8aac4da (HEAD -> NR_UE_multi_pdu, origin/NR_UE_multi_pdu) |

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


#### 4.2.2. free5gc

<b>Software:</b>

| Item   | Info                                                         |
| ------ | ------------------------------------------------------------ |
| Commit | 470b8bff3ff2cd4e485bf54cb4c6bfe7e285793e (HEAD, tag: v3.3.0) |

Command Line Codes
```shell=
# Check free5gc commit
git log -1
```



### 4.3. Compile

NEU's version of OAI is divided into 2 parts:
- OAI UE with Multi PDU Session is located in OAI gitlab under [`NR_UE_multi_pdu`](https://gitlab.eurecom.fr/oai/openairinterface5g/-/tree/NR_UE_multi_pdu?ref_type=heads) branch
- OAI gNB with rrm policy support and `ue_slice_manager` to control OAI UE pdu session is located in wineslab [ORANSlice](https://github.com/wineslab/ORANSlice/tree/main) repository

I have checked the 2 repositories and none have all the complete functions. So we need to download both of them

#### 4.3.1. OAI UE

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#32-build-oai-gnb-and-oai-nrue)
- [Source 2](https://github.com/wineslab/ORANSlice/blob/main/README.md)

<b>1. Get openairinterface5g source code</b>

```shell=
# Get openairinterface5g source code
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
cd openairinterface5g
git checkout NR_UE_multi_pdu
git log -1
```


![image](https://hackmd.io/_uploads/BJ2n2S1-Jg.png)
![image](https://hackmd.io/_uploads/HkwepH1-1g.png)
![image](https://hackmd.io/_uploads/r1cGTrJbJx.png)


<b>2. Install OAI dependencies</b>

```shell=
# Install OAI dependencies
cd cmake_targets
./build_oai -I
```


![image](https://hackmd.io/_uploads/ry5x0rkZ1l.png)


<b>3. nrscope dependencies</b>

```shell=
# nrscope dependencies
sudo apt install -y libforms-dev libforms-bin
```


![image](https://hackmd.io/_uploads/rkCXAHJWye.png)


<b>4. Enable UE send Capability5GMM in `nr_nas_msg_sim.c` source code</b>

```c=524
//#if 0
  /* This cannot be sent in clear, the core network Open5GS rejects the UE.
   * TODO: do we have to send this at some point?
   * For the time being, let's keep it here for later proper fix.
   */
  mm_msg->registration_request.presencemask |= REGISTRATION_REQUEST_5GMM_CAPABILITY_PRESENT;
  mm_msg->registration_request.fgmmcapability.iei = REGISTRATION_REQUEST_5GMM_CAPABILITY_IEI;
  mm_msg->registration_request.fgmmcapability.length = 1;
  mm_msg->registration_request.fgmmcapability.value = 0x7;
  size += 3;
//#endif
```

<b>5. Build OAI UE</b>

```shell=
# Build OAI UE
cd cmake_targets
./build_oai -w SIMU --ninja --nrUE --build-lib "nrscope" -C
```


![image](https://hackmd.io/_uploads/HyfVkLybkl.png)


#### 4.3.2. OAI gnB

- [Source 1](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_slicing_xApp.md)
- [Source 2](https://github.com/wineslab/ORANSlice/blob/main/README.md)

<b>1. Get wineslab's openairinterface5g source code</b>

```shell=
# Get wineslab's openairinterface5g source code
git clone https://github.com/wineslab/ORANSlice.git
cd ORANSlice
git log -1
```


![image](https://hackmd.io/_uploads/rkWfWTZbJx.png)
![image](https://hackmd.io/_uploads/r1WmZpZW1l.png)
![image](https://hackmd.io/_uploads/B1aQZaWZ1e.png)


<b>2. Install OAI dependencies</b>

```shell=
# Install OAI dependencies
cd oai_ran
cd cmake_targets
./build_oai -I
```


![image](https://hackmd.io/_uploads/H1cJMp-byx.png)


<b>4. nrscope dependencies</b>

```shell=
# nrscope dependencies
sudo apt install -y libforms-dev libforms-bin
```


![image](https://hackmd.io/_uploads/BkFlGI1bke.png)


<b>5. Install protobuf-c for E2Agent</b>

```shell=
# Go to home directory
sudo apt-get update
sudo apt install protobuf-compiler
sudo apt install libprotoc-dev
git clone https://github.com/protobuf-c/protobuf-c && cd protobuf-c/ && ./autogen.sh && ./configure && make && sudo make install && sudo ldconfig
```


![image](https://hackmd.io/_uploads/ByCFNTWbyg.png)


<b>6. Apply Patch to test the RAN Slicing functionality without xApp and near-RT RIC</b>

```shell=
cd oai_ran
git apply ../doc/rrmPolicyJson.patch
ls
```


![image](https://hackmd.io/_uploads/BybPuTWW1g.png)


<b>7. Build OAI gNB</b>

```shell=
# Build OAI gNB
cd cmake_targets
./build_oai -w SIMU --ninja --gNB --build-lib "nrscope" -C
```


![image](https://hackmd.io/_uploads/H1bYSa--yx.png)


### 4.4. Run

#### 4.4.1. Configuration

##### 4.4.1.1. CN Configuration

- [Source 1](https://hackmd.io/@Yueh-Huan/S1eC0g3Gi)
- [Source 2](https://free5gc.org/guide/Configuration/)

<b>1. Be aware that you might need to change 2 things in the amf configuration file (`amfcfg.yaml`):</b>
- mcc mnc (modify as your desired mnc mcc. Values that I use is below)
```shell=
- plmnid:
    mcc:001
    mnc:01
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
snssaiList:
    - sst: 1
      sd: 010203
    - sst: 1
      sd: 112233
```

<b>2. Be aware that you might need to change 2 things in the smf configuration file (`smfcfg.yaml`):</b>
- mcc mnc (modify as your desired mnc mcc. Values that I use is below)
```shell=
plmnList:
    - mcc:001
      mnc:01
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
snssaiInfos:
    - sNssai:
        sst: 1
        sd: 010203
      ...
    - sNssai:
        sst: 1
        sd: 112233
      ...
```
```shell=
sNssaiUpfInfos:
    - sNssai:
        - sst: 1
          sd: 010203
      ...
    - sNssai:
        - sst: 1
          sd: 112233
      ...
```

<b>3. Be aware that you might need to change 3 things in the nssf configuration file (`nssfcfg.yaml`):</b>
- mcc mnc (modify as your desired mnc mcc. Values that I use is below)
```shell=
supportedPlmnList:
    - mcc:001
      mnc:01
```
- s_nssai (modify as your desired Slice Configuration. Values that I use is below)
```shell=
supportedSnssaiList:
    - sst: 1
      sd: 010203
    - sst: 1
      sd: 112233
```
- nsi (modify as your desired network slice instance. Values that I use is below)
```shell=
nsiList:
    - snssai:
        sst: 1
        sd: 010203
      nsiInformationList:
          - nrfId:
            nsiId: 22
    - snssai:
        sst: 1
        sd: 112233
      nsiInformationList:
          - nrfId:
            nsiId: 23
```



##### 4.4.1.2. gNB Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf`</b>

<b>2. Be aware that you might need to change 3 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "192.168.8.21";
                    ipv6 = "fe80::d728:2dd8:567b:1805";
                    active = "yes";
                    preference = "ipv4";
                    }
                ); 


NETWORK_INTERFACES :
{
    GNB_INTERFACE_NAME_FOR_NG_AMF            = "demo-oai"; 
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.8.53"; 
    GNB_INTERFACE_NAME_FOR_NGU               = "192.168.8.53"; 
    GNB_IPV4_ADDRESS_FOR_NGU                 = "192.168.8.53"; 
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
- SliceConf Directory (modify as your `rrmPolicy.json` location. Values that I use is below)
```shell=
MACRLCs = (
{
  ...
  SliceConf = "/home/srsran/Documents/wilfrid/oranslice_gnb_v2/ORANSlice/oai_ran/rrmPolicy.json";
}
);
```

<b>3. You also need to change the RRM Policy of slice in the json file:</b>
```json=
{
	"rrmPolicyRatio" :
	[
		{
			"sst":1,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		},
		{
			"sst":1,
			"sd":66051,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		},
		{
			"sst":1,
			"sd":1122867,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		}
	]
}
```

##### 4.4.1.3. UE Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#51-run-oai-nrue)
- [Source 2](https://github.com/wineslab/ORANSlice/blob/main/README.md)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf`</b>

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

#### 4.4.2. Result

##### 4.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. Run free5gc CN</b>

```shell=
cd ~/free5gc/
sudo ./run.sh
```


![image](https://hackmd.io/_uploads/Hy0ofO1Zke.png)


<b>2. Run OAI gNB</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa  --rfsim
```


![image](https://hackmd.io/_uploads/r1vQXdy-1e.png)


<b>3. Run OAI nrUE</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf --rfsim --rfsimulator.serveraddr 127.0.0.1
```

danger
![image](https://hackmd.io/_uploads/ryZRK7GZJg.png)


<b>4. Result explanation</b>
- UE experience segmentation fault after sending registration complete message


## 5. NEU version ORANSlice with open5gs

| Step                                  | Status             |
| ------------------------------------- | ------------------ |
| NG Setup                              | :heavy_check_mark: |
| F1 Setup                              | :heavy_check_mark: |
| DU Configuration Update               | :heavy_check_mark: |
| UE get mib and sib1 for DL sync       | :heavy_check_mark: |
| UE send PRACH for UL sync             | :heavy_check_mark: |
| UE get RAR message                    | :heavy_check_mark: |
| UE send RRC setup request             | :heavy_check_mark: |
| RRC setup complete                    | :heavy_check_mark: |
| [NAS registration](#5421-Initial-Run) | :gear:             |
| PDU session complete                  | :x:                |
 
### 5.0. Minimum Requirement

- [Source](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/system_requirements.md)

#### 5.0.1. OAI gNB with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 4GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

#### 5.0.2. OAI UE with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 3GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

### 5.1. Topology

![image](https://hackmd.io/_uploads/SyjwdXPWJx.png)

### 5.2. Environment

#### 5.2.1. OAI gNB and UE

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

| Item               | Info                                                                                       |
| ------------------ | ------------------------------------------------------------------------------------------ |
| OS                 | Ubuntu 22.04.5 LTS (jammy)                                                                 |
| Kernel             | 5.15.0-1071-realtime                                                                       |
| OAI Commit for gNB | 8d3a156a809304a9f7a6036faf422b11951e547a (HEAD -> main, origin/main, origin/HEAD)          |
| OAI Commit for UE  | 458be35862258c7dbdbb38b0fecdf12ce8aac4da (HEAD -> NR_UE_multi_pdu, origin/NR_UE_multi_pdu) |

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


### 5.3. Compile

NEU's version of OAI is divided into 2 parts:
- OAI UE with Multi PDU Session is located in OAI gitlab under [`NR_UE_multi_pdu`](https://gitlab.eurecom.fr/oai/openairinterface5g/-/tree/NR_UE_multi_pdu?ref_type=heads) branch
- OAI gNB with rrm policy support and `ue_slice_manager` to control OAI UE pdu session is located in wineslab [ORANSlice](https://github.com/wineslab/ORANSlice/tree/main) repository

I have checked the 2 repositories and none have all the complete functions. So we need to download both of them

#### 5.3.1. OAI UE

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#32-build-oai-gnb-and-oai-nrue)
- [Source 2](https://github.com/wineslab/ORANSlice/blob/main/README.md)

<b>1. Get openairinterface5g source code</b>

```shell=
# Get openairinterface5g source code
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
cd openairinterface5g
git checkout NR_UE_multi_pdu
git log -1
```


![image](https://hackmd.io/_uploads/BJ2n2S1-Jg.png)
![image](https://hackmd.io/_uploads/HkwepH1-1g.png)
![image](https://hackmd.io/_uploads/r1cGTrJbJx.png)


<b>2. Install OAI dependencies</b>

```shell=
# Install OAI dependencies
cd cmake_targets
./build_oai -I
```


![image](https://hackmd.io/_uploads/ry5x0rkZ1l.png)


<b>3. nrscope dependencies</b>

```shell=
# nrscope dependencies
sudo apt install -y libforms-dev libforms-bin
```


![image](https://hackmd.io/_uploads/rkCXAHJWye.png)


<b>4. Disable UE send Capability5GMM in `nr_nas_msg_sim.c` source code</b>

```c=524
#if 0
  /* This cannot be sent in clear, the core network Open5GS rejects the UE.
   * TODO: do we have to send this at some point?
   * For the time being, let's keep it here for later proper fix.
   */
  mm_msg->registration_request.presencemask |= REGISTRATION_REQUEST_5GMM_CAPABILITY_PRESENT;
  mm_msg->registration_request.fgmmcapability.iei = REGISTRATION_REQUEST_5GMM_CAPABILITY_IEI;
  mm_msg->registration_request.fgmmcapability.length = 1;
  mm_msg->registration_request.fgmmcapability.value = 0x7;
  size += 3;
#endif
```

<b>5. Build OAI UE</b>

```shell=
# Build OAI UE
cd cmake_targets
./build_oai -w SIMU --ninja --nrUE --build-lib "nrscope" -C
```


![image](https://hackmd.io/_uploads/HyfVkLybkl.png)


#### 5.3.2. OAI gnB

- [Source 1](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_slicing_xApp.md)
- [Source 2](https://github.com/wineslab/ORANSlice/blob/main/README.md)

<b>1. Get wineslab's openairinterface5g source code</b>

```shell=
# Get wineslab's openairinterface5g source code
git clone https://github.com/wineslab/ORANSlice.git
cd ORANSlice
git log -1
```


![image](https://hackmd.io/_uploads/rkWfWTZbJx.png)
![image](https://hackmd.io/_uploads/r1WmZpZW1l.png)
![image](https://hackmd.io/_uploads/B1aQZaWZ1e.png)


<b>2. Install OAI dependencies</b>

```shell=
# Install OAI dependencies
cd oai_ran
cd cmake_targets
./build_oai -I
```


![image](https://hackmd.io/_uploads/H1cJMp-byx.png)


<b>4. nrscope dependencies</b>

```shell=
# nrscope dependencies
sudo apt install -y libforms-dev libforms-bin
```


![image](https://hackmd.io/_uploads/BkFlGI1bke.png)


<b>5. Install protobuf-c for E2Agent</b>

```shell=
# Go to home directory
sudo apt-get update
sudo apt install protobuf-compiler
sudo apt install libprotoc-dev
git clone https://github.com/protobuf-c/protobuf-c && cd protobuf-c/ && ./autogen.sh && ./configure && make && sudo make install && sudo ldconfig
```


![image](https://hackmd.io/_uploads/ByCFNTWbyg.png)


<b>6. Apply Patch to test the RAN Slicing functionality without xApp and near-RT RIC</b>

```shell=
cd oai_ran
git apply ../doc/rrmPolicyJson.patch
ls
```


![image](https://hackmd.io/_uploads/BybPuTWW1g.png)


<b>7. Build OAI gNB</b>

```shell=
# Build OAI gNB
cd cmake_targets
./build_oai -w SIMU --ninja --gNB --build-lib "nrscope" -C
```


![image](https://hackmd.io/_uploads/H1bYSa--yx.png)


### 5.4. Run

#### 5.4.1. Configuration

##### 5.4.1.1. CN Configuration

- [Source 1](https://hackmd.io/@Summer063/HJyq005rT)
- [Source 2](https://open5gs.org/open5gs/docs/guide/01-quickstart/)

<b>1. Be aware that you might need to change 2 things in the configuration file (`amfcfg.yaml`):</b>
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

##### 5.4.1.2. gNB Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf`</b>

<b>2. Be aware that you might need to change 3 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "127.0.0.5";
                    ipv6 = "fe80::3780:e95c:53e6:3fa6";
                    active = "yes";
                    preference = "ipv4";
                    }
                ); 


NETWORK_INTERFACES :
{
    GNB_INTERFACE_NAME_FOR_NG_AMF            = "demo-oai"; 
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "127.0.0.17"; 
    GNB_INTERFACE_NAME_FOR_NGU               = ""; 
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
- SliceConf Directory (modify as your `rrmPolicy.json` location. Values that I use is below)
```shell=
MACRLCs = (
{
  ...
  SliceConf = "/home/srsran/Documents/wilfrid/oranslice_gnb_v2/ORANSlice/oai_ran/rrmPolicy.json";
}
);
```

<b>3. You also need to change the RRM Policy of slice in the json file:</b>
```json=
{
	"rrmPolicyRatio" :
	[
		{
			"sst":1,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		},
		{
			"sst":1,
			"sd":66051,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		},
		{
			"sst":1,
			"sd":1122867,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		}
	]
}
```

##### 5.4.1.3. UE Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#51-run-oai-nrue)
- [Source 2](https://github.com/wineslab/ORANSlice/blob/main/README.md)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf`</b>

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

#### 5.4.2. Result

##### 5.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
```


![image](https://hackmd.io/_uploads/SkmF4ZIbkl.png)


```shell=
screen -r amf
```


![image](https://hackmd.io/_uploads/rk42Eb8-kl.png)


<b>2. Run OAI gNB</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa  --rfsim
```


![image](https://hackmd.io/_uploads/ryPfr-L-Jg.png)


<b>3. Remember to set UE's slice config in open5gs webui</b>


![image](https://hackmd.io/_uploads/ryIYUZI-yx.png)



![image](https://hackmd.io/_uploads/rkGpIWI-Je.png)


<b>4. Run OAI nrUE</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf --rfsim --rfsimulator.serveraddr 127.0.0.1
```

danger
![image](https://hackmd.io/_uploads/rJuWDbI-Jg.png)


<b>5. Result explanation</b>
- UE experience segmentation fault after it send Registration Complete
- By using GDB, we can see that the error is caused by allowed nssai check
![image](https://hackmd.io/_uploads/SkR4_W8-ye.png)
- Deduction 1: maybe we wrongly set the nssai_sd in UE side. Change nssai_sd of ue config to 010203
```shell=
nssai_sst=1;
nssai_sd=010203
```

##### 5.4.2.2. Change UE nssai_sd

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
```


![image](https://hackmd.io/_uploads/SkmF4ZIbkl.png)


```shell=
screen -r amf
```


![image](https://hackmd.io/_uploads/rk42Eb8-kl.png)


<b>2. Run OAI gNB</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa  --rfsim
```


![image](https://hackmd.io/_uploads/ryPfr-L-Jg.png)


<b>3. Run OAI nrUE</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf --rfsim --rfsimulator.serveraddr 127.0.0.1
```

danger
![image](https://hackmd.io/_uploads/S1MoYW8-1e.png)


<b>4. Result explanation</b>
- UE experience segmentation fault after it send Registration Complete
- By using GDB, we can see that the error is caused by allowed nssai check
![image](https://hackmd.io/_uploads/Bk4z9-8byl.png)
- Deduction 2: maybe allowed_nssai is empty. But we can see from gNB Log that allowed NSSAI is not empty
![image](https://hackmd.io/_uploads/BJIqAb8byl.png)
- Deduction 3: maybe we cannot use the nssai_sd in this OAI nrUE. Try delete the nssai_sd from ue.conf
```shell=
nssai_sst=1;
```

##### 5.4.2.3. Delete UE nssai_sd

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
```


![image](https://hackmd.io/_uploads/SkmF4ZIbkl.png)


```shell=
screen -r amf
```


![image](https://hackmd.io/_uploads/rk42Eb8-kl.png)


<b>2. Run OAI gNB</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa  --rfsim
```


![image](https://hackmd.io/_uploads/ryPfr-L-Jg.png)


<b>3. Run OAI nrUE</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf --rfsim --rfsimulator.serveraddr 127.0.0.1
```

danger
![image](https://hackmd.io/_uploads/HyIFxzU-ke.png)


<b>4. Result explanation</b>
- UE experience segmentation fault after it send Registration Complete
- Deduction 4: Check what change between successfull trial using open5gs. Turns out, the NEU OAI UE code is the old version
```c=
if (initialNasMsg.length > 0) {
  send_nas_uplink_data_req(nas, &initialNasMsg);
  LOG_I(NAS, "Send NAS_UPLINK_DATA_REQ message(RegistrationComplete)\n");
}
+++ const int nssai_idx = get_user_nssai_idx(nas_allowed_nssai, nas);
+++ if (nssai_idx < 0) {
--- if (!valid_user_nssai(nas->nas_allowed_nssai, &nas->nas_user->pdp_context[0])) {
    LOG_E(NAS, "NSSAI parameters not match with allowed NSSAI. Couldn't request PDU session.\n");
} else {
    /* Default PDU session */
    request_pdusession(&nas->nas_user->pdp_context[0], nas->UE_id);
}

```

## 6. NEU version ORANSlice with open5gs without multi pdu

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
| [PDU session complete](#6421-Initial-Run) | :heavy_check_mark: |


### 6.0. Minimum Requirement

- [Source](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/system_requirements.md)

#### 6.0.1. OAI gNB with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 4GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

#### 6.0.2. OAI UE with rfsim

<b>Hardware:</b>

| Item   | Info       |
| ------ | ---------- |
| CPU    | 2 @ > 2GHz |
| Memory | 3GB        |

<b>Software:</b>

| Item | Info            |
| ---- | --------------- |
| OS   | Ubuntu 20/22/24 |

### 6.1. Topology

![image](https://hackmd.io/_uploads/r1RF87D-kx.png)

### 6.2. Environment

#### 6.2.1. OAI gNB and UE

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

| Item               | Info                                                                              |
| ------------------ | --------------------------------------------------------------------------------- |
| OS                 | Ubuntu 22.04.5 LTS (jammy)                                                        |
| Kernel             | 5.15.0-1071-realtime                                                              |
| OAI Commit for gNB | 8d3a156a809304a9f7a6036faf422b11951e547a (HEAD -> main, origin/main, origin/HEAD) |

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


### 6.3. Compile

NEU's version of OAI only support 1 PDU Session per UE. The reason for this test is:
- OAI UE with Multi PDU Session is located in OAI gitlab under [`NR_UE_multi_pdu`](https://gitlab.eurecom.fr/oai/openairinterface5g/-/tree/NR_UE_multi_pdu?ref_type=heads) branch but use old version of OAI with many bugs
- OAI gNB with rrm policy support is located in wineslab [ORANSlice](https://github.com/wineslab/ORANSlice/tree/main) repository and can run even without the need for UE Multi PDU Session

#### 6.3.1. OAI gnB and UE

- [Source 1](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_slicing_xApp.md)
- [Source 2](https://github.com/wineslab/ORANSlice/blob/main/README.md)

<b>1. Get wineslab's openairinterface5g source code</b>

```shell=
# Get wineslab's openairinterface5g source code
git clone https://github.com/wineslab/ORANSlice.git
cd ORANSlice
git log -1
```


![image](https://hackmd.io/_uploads/rkWfWTZbJx.png)
![image](https://hackmd.io/_uploads/r1WmZpZW1l.png)
![image](https://hackmd.io/_uploads/B1aQZaWZ1e.png)


<b>2. Install OAI dependencies</b>

```shell=
# Install OAI dependencies
cd oai_ran
cd cmake_targets
./build_oai -I
```


![image](https://hackmd.io/_uploads/H1cJMp-byx.png)


<b>4. nrscope dependencies</b>

```shell=
# nrscope dependencies
sudo apt install -y libforms-dev libforms-bin
```


![image](https://hackmd.io/_uploads/BkFlGI1bke.png)


<b>5. Install protobuf-c for E2Agent</b>

```shell=
# Go to home directory
sudo apt-get update
sudo apt install protobuf-compiler
sudo apt install libprotoc-dev
git clone https://github.com/protobuf-c/protobuf-c && cd protobuf-c/ && ./autogen.sh && ./configure && make && sudo make install && sudo ldconfig
```


![image](https://hackmd.io/_uploads/ByCFNTWbyg.png)


<b>6. Apply Patch to test the RAN Slicing functionality without xApp and near-RT RIC</b>

```shell=
cd oai_ran
git apply ../doc/rrmPolicyJson.patch
ls
```


![image](https://hackmd.io/_uploads/BybPuTWW1g.png)


<b>7. Build OAI gNB</b>

```shell=
# Build OAI gNB
cd cmake_targets
./build_oai -w SIMU --ninja --gNB --nrUE --build-lib "nrscope" -C
```


![image](https://hackmd.io/_uploads/HkugymLZke.png)


### 6.4. Run

#### 6.4.1. Configuration

##### 6.4.1.1. CN Configuration

- [Source 1](https://hackmd.io/@Summer063/HJyq005rT)
- [Source 2](https://open5gs.org/open5gs/docs/guide/01-quickstart/)

<b>1. Be aware that you might need to change 2 things in the configuration file (`all_open5gs.yaml`):</b>
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

##### 6.4.1.2. gNB Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf`</b>

<b>2. Be aware that you might need to change 3 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "127.0.0.5";
                    ipv6 = "fe80::3780:e95c:53e6:3fa6";
                    active = "yes";
                    preference = "ipv4";
                    }
                ); 


NETWORK_INTERFACES :
{
    GNB_INTERFACE_NAME_FOR_NG_AMF            = "demo-oai"; 
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "127.0.0.17"; 
    GNB_INTERFACE_NAME_FOR_NGU               = ""; 
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
- SliceConf Directory (modify as your `rrmPolicy.json` location. Values that I use is below)
```shell=
MACRLCs = (
{
  ...
  SliceConf = "/home/srsran/Documents/wilfrid/oranslice_gnb_v2/ORANSlice/oai_ran/rrmPolicy.json";
}
);
```

<b>3. You also need to change the RRM Policy of slice in the json file:</b>
```json=
{
	"rrmPolicyRatio" :
	[
		{
			"sst":1,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		},
		{
			"sst":1,
			"sd":66051,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		},
		{
			"sst":1,
			"sd":1122867,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		}
	]
}
```

##### 6.4.1.3. UE Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#51-run-oai-nrue)
- [Source 2](https://github.com/wineslab/ORANSlice/blob/main/README.md)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf`</b>

<b>2. Be aware that you might need to change 3 things in the UE configuration file:</b>
- imsi (modify as your desired ue identity. Values that I use is below)
```shell=
imsi = "001010000000001";
```
- nssai_sst & nssai_sd (modify as your desired Slice Configuration. Values that I use is below)
```shell=
nssai_sst=1;
nssai_sd=0x010203
```
- dnn (modify as your desired DNN. Values that I use is below)
```shell=
dnn="internet";
```

#### 6.4.2. Result

##### 6.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
```


![image](https://hackmd.io/_uploads/SkmF4ZIbkl.png)


```shell=
screen -r amf
```


![image](https://hackmd.io/_uploads/rk42Eb8-kl.png)


<b>2. Run OAI gNB</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa  --rfsim
```


![image](https://hackmd.io/_uploads/ryPfr-L-Jg.png)


<b>3. Run OAI nrUE</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf --rfsim --rfsimulator.serveraddr 127.0.0.1
```


![image](https://hackmd.io/_uploads/r1EYZ7IbJl.png)


<b>4. Result explanation</b>
- UE 1 with snssai = 1;010203 successfully connect with open5gs and establish pdu sesssion
![image](https://hackmd.io/_uploads/H1dAWQ8-Jl.png)
- We can see that this UE connected to slice with rrmPolicy{ded,min,max}={5,10,100}
![image](https://hackmd.io/_uploads/rJRM77L-Je.png)


##### 6.4.2.2. Change Slice RRM Policy to minimal

<b>0. Lets change slice 010203 rrm policy to be minimal to see the effect on UE</b>

```shell=
{
	"rrmPolicyRatio" :
	[
		{
			"sst":1,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		},
		{
			"sst":1,
			"sd":66051,
			"dedicated_ratio":0,
			"min_ratio":0,
			"max_ratio":0
		},
		{
			"sst":1,
			"sd":1122867,
			"dedicated_ratio":5,
			"min_ratio":10,
			"max_ratio":100
		}
	]
}
```


<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
```


![image](https://hackmd.io/_uploads/SkmF4ZIbkl.png)


```shell=
screen -r amf
```


![image](https://hackmd.io/_uploads/rk42Eb8-kl.png)


<b>2. Run OAI gNB</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa  --rfsim
```


![image](https://hackmd.io/_uploads/ryPfr-L-Jg.png)


<b>3. Run OAI nrUE</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf --rfsim --rfsimulator.serveraddr 127.0.0.1
```


![image](https://hackmd.io/_uploads/HygxSmI-Jg.png)


<b>4. Result explanation</b>
- UE 1 with snssai = 1;010203 successfully connect with open5gs and establish pdu sesssion
![image](https://hackmd.io/_uploads/H1LGSQLZJl.png)
- However, since we modify the slice rrmpolicy, UE 1 cannot send any data in PDU session because cannot get PRB resource
![image](https://hackmd.io/_uploads/H1rOS7UWyg.png)

