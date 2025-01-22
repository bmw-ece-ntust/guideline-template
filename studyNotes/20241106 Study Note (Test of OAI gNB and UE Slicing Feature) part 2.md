# 2024/11/06 Study Note (Test of OAI gNB and UE Slicing Feature) part 2

###### tags: `2024`


**Goal:**
- [x] [Test OAI gNB and OAI UE NEU version Slicing Feature with free5gc](#1-NEU-version-ORANSlice-with-free5gc-without-multi-pdu)
- [x] [Test OAI gNB and OAI UE `develop` branch Slicing Feature in CU-DU Split](#2-develop-branch-CU-DU-Split)
- [x] [Test OAI gNB and OAI UE `develop` branch Slicing Feature in CU-DU Split Multi DU](#3-develop-branch-CU-DU-Split-Multi-DU)
- [x] [Test OAI gNB and OAI UE `develop` branch Slicing Feature in CU CP-UP Split](#4-evelop-branch-CU-CP-UP-Split)
- [x] [Test OAI gNB and OAI UE `develop` branch Slicing Feature in CU-DU Split Multi CU-UP and Multi DU](#5-develop-branch-CU-CP-UP-Split-Multi-CU-UP-Multi-DU)
- [x] [Test OAI gNB and OAI UE `develop` branch Slicing Feature in CU-DU Split Multi CU-UP, Multi DU, Multi Server](#6-develop-branch-CU-CP-UP-Split-Multi-CU-UP-Multi-DU-Multi-Server)
- [ ] Test OAI gNB and OAI UE `rebased-5g-nvs-rc-rf` branch Slicing Feature -> Refer to [part 3](https://hackmd.io/@superwilfrid/Sy12p58Eyg) of note


**References:**
- [OAI | NR SA Tutorial OAI nrUE](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md)
- [Wilfrid | RAN Slicing Architecture Requirements](https://hackmd.io/@superwilfrid/ry21MNnqC)
- [Wilfrid | ORANSlice: An Open-Source 5G Network Slicing Platform for O-RAN](https://hackmd.io/@superwilfrid/SJ5G4zLgJl#2-What-they-add-in-OAI-nrUE)
- [Wilfrid | Test of OAI gNB and UE Slicing Feature](https://hackmd.io/@superwilfrid/rys33zIkJl)
- [sharetechnote | 5G/NR  - Network Slicing](https://www.sharetechnote.com/html/5G/5G_NetworkSlicing.html)
![image](https://hackmd.io/_uploads/rkhGBXCgyl.png)



**Table of Contents:**
[TOC]


## 0. Summary

1. OAI NEU version (ORANSlice) branch has gNB and nrUE that can connect to free5gc by little modification in code (nrUE). [Details](#1-NEU-version-ORANSlice-with-free5gc-without-multi-pdu)
    - gNB that will treat slice (in MAC SCH) based on rrmPolicy

## 1. NEU version ORANSlice with free5gc without multi pdu

- ORANSlice repository = https://github.com/wineslab/ORANSlice/tree/main

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

![image](https://hackmd.io/_uploads/HJZ4zMiGJg.png)

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


#### 1.2.2. free5gc

<b>Software:</b>

| Item   | Info                                                         |
| ------ | ------------------------------------------------------------ |
| Commit | 470b8bff3ff2cd4e485bf54cb4c6bfe7e285793e (HEAD, tag: v3.3.0) |

Command Line Codes
```shell=
# Check free5gc commit
git log -1
```


### 1.3. Compile

NEU's version of OAI only support 1 PDU Session per UE. The reason for this test is:
- OAI UE with Multi PDU Session is located in OAI gitlab under [`NR_UE_multi_pdu`](https://gitlab.eurecom.fr/oai/openairinterface5g/-/tree/NR_UE_multi_pdu?ref_type=heads) branch but use old version of OAI with many bugs
- OAI gNB with rrm policy support is located in wineslab [ORANSlice](https://github.com/wineslab/ORANSlice/tree/main) repository and can run even without the need for UE Multi PDU Session

#### 1.3.1. OAI gnB and UE

- [Source 1](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_slicing_xApp.md)
- [Source 2](https://github.com/wineslab/ORANSlice/blob/main/README.md)

<b>1. Get wineslab's ORANSlice source code</b>

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


<b>7. Modify `/openair3/NAS/NR_UE/nr_nas_msg_sim.c` to send `5GMM_CAPABILITY`</b>

```c=518
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

<b>8. Build OAI gNB</b>

```shell=
# Build OAI gNB
cd cmake_targets
./build_oai -w SIMU --ninja --gNB --nrUE --build-lib "nrscope" -C
```

![image](https://hackmd.io/_uploads/SJbjtFO-yx.png)


### 1.4. Run

#### 1.4.1. Configuration

##### 1.4.1.1. CN Configuration

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


##### 1.4.1.2. gNB Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. We use example configuration from `/targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf`</b>

<b>2. Be aware that you might need to change 3 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "192.168.8..21";
                    ipv6 = "fe80::d728:2dd8:567b:1805";
                    active = "yes";
                    preference = "ipv4";
                    }
                ); 


NETWORK_INTERFACES :
{
    GNB_INTERFACE_NAME_FOR_NG_AMF            = "demo-oai"; 
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.8.53"; 
    GNB_INTERFACE_NAME_FOR_NGU               = "demo-oai"; 
    GNB_IPV4_ADDRESS_FOR_NGU                 = "127.168.8.53"; 
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

##### 1.4.1.3. UE Configuration

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

#### 1.4.2. Result

##### 1.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/OAI_Slicing_RICBypass.md)

<b>1. Run free5gc CN</b>

```shell=
cd ~/free5gc/
sudo ./run.sh
```

![image](https://hackmd.io/_uploads/S1V6pt_WJx.png)


<b>2. Run OAI gNB</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa  --rfsim
```

![image](https://hackmd.io/_uploads/rJGQRYd-kx.png)


<b>3. Run OAI nrUE</b>

```shell=
cd cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf --rfsim --rfsimulator.serveraddr 127.0.0.1
```

![image](https://hackmd.io/_uploads/rkJp0YdW1x.png)


<b>4. Result explanation</b>
- UE 1 with snssai = 1;010203 successfully connect with open5gs and establish pdu sesssion
![image](https://hackmd.io/_uploads/Hyb-y9_bJx.png)
- We can see that this UE connected to slice with rrmPolicy{ded,min,max}={5,10,100}
![image](https://hackmd.io/_uploads/BkUrk9_b1g.png)

## 2. `develop` branch CU-DU Split

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
| [PDU session complete](#2421-Initial-Run) | :heavy_check_mark: |

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

![image](https://hackmd.io/_uploads/SJPJBSkM1g.png)

### 2.2. Environment

#### 2.2.1. OAI CU, DU and UE

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

##### 2.4.1.2. CU Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-cu.sa.band78.106prb.conf`</b>

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

##### 2.4.1.3. DU Configuration

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
                                { sst = 1; sd = 0x010203; },
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


##### 2.4.1.4. UE Configuration

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
nssai_sd=66051;
```
- dnn (modify as your desired DNN. Values that I use is below)
```shell=
dnn="internet";
```

#### 2.4.2. Result

##### 2.4.2.1. Initial Run

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

![image](https://hackmd.io/_uploads/r1F8QwxMJe.png)



<b>2. Run OAI CU</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-cu.sa.band78.106prb.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/By-RmvgM1g.png)


<b>3. Run OAI DU</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/HJLN4wlzJe.png)


<b>4. Run OAI nrUE</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```

![image](https://hackmd.io/_uploads/BJ_CEvxf1x.png)


<b>5. Result explanation</b>
- Slice identification works in OAI develop branch CU-DU Split
![image](https://hackmd.io/_uploads/SJ_JLPlMkg.png)

## 3. `develop` branch CU-DU Split Multi DU

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
| [PDU session complete](#3421-Initial-Run) | :heavy_check_mark: |

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

![image](https://hackmd.io/_uploads/SyRHl0zGkg.png)

### 3.2. Environment

#### 3.2.1. OAI CU, DU and UE

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


### 3.3. Compile

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


### 3.4. Run

#### 3.4.1. Configuration

##### 3.4.1.1. CN Configuration

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

##### 3.4.1.2. CU Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-cu.sa.band78.106prb.conf`</b>

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

##### 3.4.1.3. DU 1 Configuration

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


##### 3.4.1.4. DU 2 Configuration

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
- rfsim serverport (modify as your desired rfsim port. Values that I use is below)
```shell=
rfsimulator: {
serveraddr = "server";
    serverport = 4044;
    options = (); #("saviq"); or/and "chanmod"
    modelname = "AWGN";
    IQfile = "/tmp/rfsimulator.iqs"
}
}
```


##### 3.4.1.5. UE 1 Configuration

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

##### 3.4.1.6. UE 2 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#51-run-oai-nrue)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>1. Create a copy of `/ci-scripts/conf_files/nrue.uicc.conf`</b>

<b>2. Be aware that you might need to change 3 things in the UE configuration file:</b>
- imsi (modify as your desired ue identity. Values that I use is below)
```shell=
imsi = "001010000000002";
```
- nssai_sst & nssai_sd (modify as your desired Slice Configuration. Values that I use is below)
```shell=
nssai_sst=1;
nssai_sst=1122867;
```
- dnn (modify as your desired DNN. Values that I use is below)
```shell=
dnn="internet";
```

#### 3.4.2. Result

##### 3.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
```

![image](https://hackmd.io/_uploads/SkyYj6sJ1l.png)


```shell=
screen -r amf
```

![image](https://hackmd.io/_uploads/r1F8QwxMJe.png)



<b>2. Run OAI CU</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-cu.sa.band78.106prb.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/r1tghdxMkl.png)


<b>3. Run OAI DU 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/ryMc0uxM1x.png)


<b>4. Run OAI DU 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du2.sa.band78.106prb.rfsim.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/ryGg1Kefke.png)



<b>5. Run OAI nrUE 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --rfsimulator.serverport 4044 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue2.uicc.conf
```

![image](https://hackmd.io/_uploads/HkRIotgG1e.png)


<b>6. Run OAI nrUE 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```

![image](https://hackmd.io/_uploads/SJisjteGkx.png)



<b>7. Result explanation</b>
- CU got 2 F1AP Setup Request
![image](https://hackmd.io/_uploads/Hy3F1Ygzyx.png)
- UE 2 with slice 1:112233 get rnti 34c1
![image](https://hackmd.io/_uploads/ByVXsKxM1x.png)
- UE 2 is handled by DU 2
![image](https://hackmd.io/_uploads/rySNntefJl.png)
- UE 2 PDU session is in slice 1:112233
![image](https://hackmd.io/_uploads/HJXf6YeGkl.png)
- UE 1 with slice 1:010203 get rnti 220f
![image](https://hackmd.io/_uploads/BksYaFlGkg.png)
- UE 1 is handled by DU 1
![image](https://hackmd.io/_uploads/rkY0pYeGkx.png)
- UE 1 PDU session is in slice 1:010203
![image](https://hackmd.io/_uploads/S1D20Yez1l.png)


## 4. `develop` branch CU CP-UP Split

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
| [PDU session complete](#4421-Initial-Run) | :heavy_check_mark: |

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

![image](https://hackmd.io/_uploads/SkgqeAzGJg.png)

### 4.2. Environment

#### 4.2.1. OAI CU, DU and UE

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


### 4.3. Compile

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


### 4.4. Run

#### 4.4.1. Configuration

##### 4.4.1.1. CN Configuration

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

##### 4.4.1.2. CU-CP Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-cucp.sa.f1.conf`</b>

<b>2. Be aware that you might need to change 4 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "127.0.0.5"; }); 


NETWORK_INTERFACES :
{
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "127.0.0.17"; 
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
- E1_INTERFACE (modify as your desired CU CP-UP Split IP Address Configuration. Values that I use is below)
```shell=
E1_INTERFACE =
(
  {
    type = "cp";
    ipv4_cucp = "127.0.0.25";
    port_cucp = 38462;
    ipv4_cuup = "127.0.0.26"; # multiple CU-UPs
    port_cuup = 38462;
  }
)
```

##### 4.4.1.3. CU-UP Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#42-run-oai-gnb)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-cuup.sa.f1.conf`</b>

<b>2. Be aware that you might need to change 4 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
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
    local_s_portc   = 2153;
    local_s_portd   = 2153;
    remote_s_portc  = 2153;
    remote_s_portd  = 2153;
```
- E1_INTERFACE (modify as your desired CU CP-UP Split IP Address Configuration. Values that I use is below)
```shell=
E1_INTERFACE =
(
  {
    type = "up";
    ipv4_cucp = "127.0.0.25";
    ipv4_cuup = "127.0.0.26";
  }
)
```


##### 4.4.1.4. DU Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)

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
    local_n_portd   = 2153;
    remote_n_portc  = 2152;
    remote_n_portd  = 2153;
```

##### 4.4.1.5. UE 1 Configuration

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

#### 4.4.2. Result

##### 4.4.2.1. Initial Run

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

![image](https://hackmd.io/_uploads/r1F8QwxMJe.png)

<b>2. Run OAI CU-CP</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-cucp.sa.f1.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/BJjq-NGG1e.png)

<b>3. Run OAI CU-UP</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-cuup -O ../../../ci-scripts/conf_files/gnb-cuup.sa.f1.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/rJDfGVzz1l.png)


<b>4. Run OAI DU</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/Sy7XQ4zM1g.png)


<b>5. Run OAI nrUE</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```

![image](https://hackmd.io/_uploads/rkaHmCzGJg.png)

<b>6. Result explanation</b>
- CU-CP got E1AP Setup Request from CU-UP
![image](https://hackmd.io/_uploads/H1WxfAzzJg.png)
- CU-CP got F1AP Setup Request from DU
![image](https://hackmd.io/_uploads/SkTrfCzG1l.png)
- CU-CP will select appropriate CU-UP based on UE PDU session slice
![image](https://hackmd.io/_uploads/Hk1_fRffJe.png)
- CU-UP will create Tunnel for UE's DRB
![image](https://hackmd.io/_uploads/ryluRGAfMke.png)
- DU create Tunnel for UE's DRB
![image](https://hackmd.io/_uploads/H1oemAGzkg.png)
- PDU Session for UE complete
![image](https://hackmd.io/_uploads/SJhqS4zGkg.png)



## 5. `develop` branch CU CP-UP Split Multi CU-UP Multi DU

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
| [PDU session complete](#5421-Initial-Run) | :heavy_check_mark: |

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

![image](https://hackmd.io/_uploads/SkmnaZ7G1g.png)

### 5.2. Environment

#### 5.2.1. OAI CU, DU and UE

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


### 5.3. Compile

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


### 5.4. Run

#### 5.4.1. Configuration

##### 5.4.1.1. CN Configuration

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

##### 5.4.1.2. CU-CP Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-cucp.sa.f1.conf`</b>

<b>2. Be aware that you might need to change 4 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "127.0.0.5"; }); 


NETWORK_INTERFACES :
{
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "127.0.0.17"; 
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
- E1_INTERFACE (modify as your desired CU CP-UP Split IP Address Configuration. Values that I use is below)
```shell=
E1_INTERFACE =
(
  {
    type = "cp";
    ipv4_cucp = "127.0.0.26";
    port_cucp = 38462;
    ipv4_cuup = "0.0.0.0"; # multiple CU-UPs
    port_cuup = 38462;
  }
)
```

##### 5.4.1.3. CU-UP 1 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-cuup.sa.f1.conf`</b>

<b>2. Be aware that you might need to change 4 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
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
                                { sst = 1; sd = 0x010203; }
                                );
                            });
```
- local/remote_s_address (modify as your desired CU/DU Split IP Address Configuration. Values that I use is below)
```shell=
tr_s_preference = "f1";

    local_s_address = "127.0.0.23";
    remote_s_address = "0.0.0.0";
    local_s_portc   = 2153;
    local_s_portd   = 2153;
    remote_s_portc  = 2153;
    remote_s_portd  = 2153;
```
- E1_INTERFACE (modify as your desired CU CP-UP Split IP Address Configuration. Values that I use is below)
```shell=
E1_INTERFACE =
(
  {
    type = "up";
    ipv4_cucp = "127.0.0.26";
    ipv4_cuup = "127.0.0.27";
  }
)
```

##### 5.4.1.4. CU-UP 2 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. Create a copy of `ci-scripts/conf_files/gnb-cuup.sa.f1.conf`</b>

<b>2. Be aware that you might need to change 5 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
NETWORK_INTERFACES :
{
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "127.0.0.17"; 
    GNB_IPV4_ADDRESS_FOR_NGU                 = "127.0.0.19"; 
    GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
};
```
- PLMN List (modify as your desired Slice Configuration. Values that I use is below)
```shell=
plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2;
                snssaiList = (
                                { sst = 1; sd = 0x112233; }
                                );
                            });
```
- local/remote_s_address (modify as your desired CU/DU Split IP Address Configuration. Values that I use is below)
```shell=
tr_s_preference = "f1";

    local_s_address = "127.0.0.23";
    remote_s_address = "0.0.0.0";
    local_s_portc   = 2154;
    local_s_portd   = 2154;
    remote_s_portc  = 2154;
    remote_s_portd  = 2154;
```
- E1_INTERFACE (modify as your desired CU CP-UP Split IP Address Configuration. Values that I use is below)
```shell=
E1_INTERFACE =
(
  {
    type = "up";
    ipv4_cucp = "127.0.0.26";
    ipv4_cuup = "127.0.0.28";
  }
)
```
- CU_UP_ID (modify as your desired multiple CU UP Configuration. Values that I use is below)
```shell=
gNBs =
(
 {
    ////////// Identification parameters:
    gNB_ID = 0xe01;
    gNB_CU_UP_ID = 0xe01;
    ...
```


##### 5.4.1.5. DU 1 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)

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
    local_n_portd   = 2153;
    remote_n_portc  = 2152;
    remote_n_portd  = 2153;
```

##### 5.4.1.6. DU 2 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)

<b>1. Create a copy of `ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf`</b>
![image](https://hackmd.io/_uploads/HyvBp17Gkx.png)

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
                                { sst = 1; sd = 0x010203; }
                                );
                            });
```
- remote_n_address (modify as your CU ip address. Values that I use is below)
```shell=
remote_n_address = "127.0.0.23";
    local_n_portc   = 2152;
    local_n_portd   = 2154;
    remote_n_portc  = 2152;
    remote_n_portd  = 2154;
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
- rfsim serverport (modify as your desired rfsim port. Values that I use is below)
```shell=
rfsimulator: {
serveraddr = "server";
    serverport = 4044;
    options = (); #("saviq"); or/and "chanmod"
    modelname = "AWGN";
    IQfile = "/tmp/rfsimulator.iqs"
}
}
```



##### 5.4.1.7. UE 1 Configuration

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

##### 5.4.1.2. UE 2 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#51-run-oai-nrue)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>1. Create a copy of `/ci-scripts/conf_files/nrue.uicc.conf`</b>

<b>2. Be aware that you might need to change 3 things in the UE configuration file:</b>
- imsi (modify as your desired ue identity. Values that I use is below)
```shell=
imsi = "001010000000002";
```
- nssai_sst & nssai_sd (modify as your desired Slice Configuration. Values that I use is below)
```shell=
nssai_sst=1;
nssai_sd=1122867
```
- dnn (modify as your desired DNN. Values that I use is below)
```shell=
dnn="internet";
```

#### 5.4.2. Result

##### 5.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)

<b>1. Run open5gs CN</b>

```shell=
sudo ./createtun.sh # not persistent after rebooting
./start_open5gs.sh
```

![image](https://hackmd.io/_uploads/SkyYj6sJ1l.png)


```shell=
screen -r amf
```

![image](https://hackmd.io/_uploads/r1F8QwxMJe.png)

<b>2. Run OAI CU-CP</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-cucp.sa.f1.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/Hy7V8Q4G1e.png)


<b>3. Run OAI CU-UP 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-cuup -O ../../../ci-scripts/conf_files/gnb-cuup.sa.f1.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/rJj_I7VGye.png)


<b>4. Run OAI CU-UP 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-cuup -O ../../../ci-scripts/conf_files/gnb-cuup2.sa.f1.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/r1HiUX4f1x.png)



<b>5. Run OAI DU 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/Skb0IQVG1g.png)


<b>6. Run OAI DU 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du2.sa.band78.106prb.rfsim.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/Hk7gwQ4z1g.png)



<b>7. Run OAI nrUE 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```

![image](https://hackmd.io/_uploads/H16mD7NGyl.png)


<b>8. Run OAI nrUE 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --rfsimulator.serverport 4044 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue2.uicc.conf
```

![image](https://hackmd.io/_uploads/Sy5wwQEzJl.png)

<b>9. Result explanation</b>
- Multi CU-UP and multi DU Connection success
![image](https://hackmd.io/_uploads/r10luXVG1g.png)
- CU-CP choose CU-UP 1 for UE 1 PDU session
![image](https://hackmd.io/_uploads/H1truQVGkg.png)
- CU-UP 1 make DRB tunnel for UE 1 PDU session
![image](https://hackmd.io/_uploads/Byk1FQNzJg.png)
- DU 1 make DRB tunnel for UE 1 PDU session
![image](https://hackmd.io/_uploads/HJ_XF74Mkg.png)
- DU 1 statistics for UE 1
![image](https://hackmd.io/_uploads/BkKsUg7zke.png)
- CU-CP choose CU-UP 2 for UE 2 PDU session
![image](https://hackmd.io/_uploads/rJmvd7EMkx.png)
- CU-UP 2 make DRB tunnel for UE 2 PDU session
![image](https://hackmd.io/_uploads/SJtJwgXGkx.png)
- DU 2 make DRB tunnel for UE 2 PDU session
![image](https://hackmd.io/_uploads/ByNGwgXzJg.png)
- DU 2 statistics for UE 2
![image](https://hackmd.io/_uploads/BJpVwxmf1e.png)


## 6. `develop` branch CU CP-UP Split Multi CU-UP Multi DU Multi Server

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

![image](https://hackmd.io/_uploads/H1tTbMofyx.png)

### 6.2. Environment

#### 6.2.1. OAI CU, DU and UE 2

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


#### 6.2.2. OAI UE 1

<b>Hardware:</b>

| Item         | Info                                                       |
| ------------ | ---------------------------------------------------------- |
| CPU          | Intel(R) Xeon(R) E5-2695 processor @ 2.10 GHz ( cores) x 4 |
| Memory       | 16GB                                                       |
| Disk         | 129GB                                                      |
| Server Model | VMware Virtual Platform                                    |

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
| Kernel     | 5.15.0-116-generic                                                                                      |
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


#### 6.2.3. free5gc
<b>Software:</b>

| Item   | Info                                                         |
| ------ | ------------------------------------------------------------ |
| Commit | 470b8bff3ff2cd4e485bf54cb4c6bfe7e285793e (HEAD, tag: v3.3.0) |

Command Line Codes
```shell=
# Check free5gc commit
git log -1
```


### 6.3. Compile

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


<b>4. Modify `/openair3/NAS/NR_UE/nr_nas_msg_sim.c` to send `5GMM_CAPABILITY`</b>

```c=518
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

<b>5. Build OAI gNB</b>

```shell=
# Build OAI gNB
cd ~/openairinterface5g/cmake_targets
./build_oai -w SIMU --ninja --nrUE --gNB --build-lib "nrscope" -C
```

![image](https://hackmd.io/_uploads/B1wQyv911l.png)


### 6.4. Run

#### 6.4.1. Configuration

##### 6.4.1.1. CN Configuration

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

##### 6.4.1.2. CU-CP Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-cucp.sa.f1.conf`</b>

<b>2. Be aware that you might need to change 4 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
////////// AMF parameters:
amf_ip_address = ({ ipv4 = "192.168.8.21"; }); 


NETWORK_INTERFACES :
{
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.8.53"; 
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
- E1_INTERFACE (modify as your desired CU CP-UP Split IP Address Configuration. Values that I use is below)
```shell=
E1_INTERFACE =
(
  {
    type = "cp";
    ipv4_cucp = "127.0.0.26";
    port_cucp = 38462;
    ipv4_cuup = "0.0.0.0"; # multiple CU-UPs
    port_cuup = 38462;
  }
)
```

##### 6.4.1.3. CU-UP 1 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. We use example configuration from `ci-scripts/conf_files/gnb-cuup.sa.f1.conf`</b>

<b>2. Be aware that you might need to change 4 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
NETWORK_INTERFACES :
{
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.8.53"; 
    GNB_IPV4_ADDRESS_FOR_NGU                 = "192.168.8.53"; 
    GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
};
```
- PLMN List (modify as your desired Slice Configuration. Values that I use is below)
```shell=
plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2;
                snssaiList = (
                                { sst = 1; sd = 0x010203; }
                                );
                            });
```
- local/remote_s_address (modify as your desired CU/DU Split IP Address Configuration. Values that I use is below)
```shell=
tr_s_preference = "f1";

    local_s_address = "127.0.0.23";
    remote_s_address = "0.0.0.0";
    local_s_portc   = 2153;
    local_s_portd   = 2153;
    remote_s_portc  = 2153;
    remote_s_portd  = 2153;
```
- E1_INTERFACE (modify as your desired CU CP-UP Split IP Address Configuration. Values that I use is below)
```shell=
E1_INTERFACE =
(
  {
    type = "up";
    ipv4_cucp = "127.0.0.26";
    ipv4_cuup = "127.0.0.27";
  }
)
```

##### 6.4.1.4. CU-UP 2 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)

<b>1. Create a copy of `ci-scripts/conf_files/gnb-cuup.sa.f1.conf`</b>

<b>2. Be aware that you might need to change 5 things in the configuration file:</b>
- AMF and Network Interface Parameters (modify as your AMF and gNB ip address. Values that I use is below)
```shell=
NETWORK_INTERFACES :
{
    GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.8.53"; 
    GNB_IPV4_ADDRESS_FOR_NGU                 = "192.168.8.53"; 
    GNB_PORT_FOR_NGU                         = 2153; # Spec 2152
    GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
};
```
- PLMN List (modify as your desired Slice Configuration. Values that I use is below)
```shell=
plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2;
                snssaiList = (
                                { sst = 1; sd = 0x112233; }
                                );
                            });
```
- local/remote_s_address (modify as your desired CU/DU Split IP Address Configuration. Values that I use is below)
```shell=
tr_s_preference = "f1";

    local_s_address = "127.0.0.23";
    remote_s_address = "0.0.0.0";
    local_s_portc   = 2154;
    local_s_portd   = 2154;
    remote_s_portc  = 2154;
    remote_s_portd  = 2154;
```
- E1_INTERFACE (modify as your desired CU CP-UP Split IP Address Configuration. Values that I use is below)
```shell=
E1_INTERFACE =
(
  {
    type = "up";
    ipv4_cucp = "127.0.0.26";
    ipv4_cuup = "127.0.0.28";
  }
)
```
- CU_UP_ID (modify as your desired multiple CU UP Configuration. Values that I use is below)
```shell=
gNBs =
(
 {
    ////////// Identification parameters:
    gNB_ID = 0xe01;
    gNB_CU_UP_ID = 0xe01;
    ...
```


##### 6.4.1.5. DU 1 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)

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
    local_n_portd   = 2153;
    remote_n_portc  = 2152;
    remote_n_portd  = 2153;
```

##### 6.4.1.6. DU 2 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)

<b>1. Create a copy of `ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf`</b>
![image](https://hackmd.io/_uploads/HyvBp17Gkx.png)

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
                                { sst = 1; sd = 0x010203; }
                                );
                            });
```
- remote_n_address (modify as your CU ip address. Values that I use is below)
```shell=
remote_n_address = "127.0.0.23";
    local_n_portc   = 2152;
    local_n_portd   = 2154;
    remote_n_portc  = 2152;
    remote_n_portd  = 2154;
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
- rfsim serverport (modify as your desired rfsim port. Values that I use is below)
```shell=
rfsimulator: {
serveraddr = "server";
    serverport = 4044;
    options = (); #("saviq"); or/and "chanmod"
    modelname = "AWGN";
    IQfile = "/tmp/rfsimulator.iqs"
}
}
```


##### 6.4.1.7. UE 1 Configuration

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

##### 6.4.1.2. UE 2 Configuration

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#51-run-oai-nrue)
- [Source 2](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/tree/soc-2024/oam?ref_type=tags#62-slicing-demo)

<b>1. Create a copy of `/ci-scripts/conf_files/nrue.uicc.conf`</b>

<b>2. Be aware that you might need to change 3 things in the UE configuration file:</b>
- imsi (modify as your desired ue identity. Values that I use is below)
```shell=
imsi = "001010000000002";
```
- nssai_sst & nssai_sd (modify as your desired Slice Configuration. Values that I use is below)
```shell=
nssai_sst=1;
nssai_sd=1122867
```
- dnn (modify as your desired DNN. Values that I use is below)
```shell=
dnn="internet";
```

#### 6.4.2. Result

##### 6.4.2.1. Initial Run

- [Source 1](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/F1AP/F1-design.md)
- [Source 2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/E1AP/E1-design.md)

<b>1. Run open5gs CN</b>

```shell=
cd ~/free5gc/
sudo ./run.sh
```

![image](https://hackmd.io/_uploads/rJkk2ctGkg.png)


<b>2. Run OAI CU-CP</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-cucp.sa.f1.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/B1wnapFf1l.png)


<b>3. Run OAI CU-UP 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-cuup -O ../../../ci-scripts/conf_files/gnb-cuup.sa.f1.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/r1zlA6Ff1x.png)


<b>4. Run OAI CU-UP 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-cuup -O ../../../ci-scripts/conf_files/gnb-cuup2.sa.f1.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/ry8X0atMye.png)

<b>5. Run OAI DU 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du.sa.band78.106prb.rfsim.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/B14UCptMye.png)

<b>6. Run OAI DU 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../ci-scripts/conf_files/gnb-du2.sa.band78.106prb.rfsim.conf --rfsim --sa
```

![image](https://hackmd.io/_uploads/HkaO0aFf1x.png)

<b>7. Run OAI nrUE 1</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 192.168.8.53 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue.uicc.conf
```

![image](https://hackmd.io/_uploads/ByGlyAKM1g.png)

<b>8. Run OAI nrUE 2</b>

```shell=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --rfsim --rfsimulator.serveraddr 127.0.0.1 --rfsimulator.serverport 4044 --sa -r 106 --numerology 1 --band 78 -C 3619200000 -O ../../../ci-scripts/conf_files/nrue2.uicc.conf
```

![image](https://hackmd.io/_uploads/HyfHJRFM1x.png)

<b>9. Result explanation</b>
- Multi CU-UP and multi DU Connection success
![image](https://hackmd.io/_uploads/HyTh1AKzke.png)
- CU-CP choose CU-UP 1 for UE 1 PDU session
![image](https://hackmd.io/_uploads/ryFkxRYMkg.png)
- CU-UP 1 make DRB tunnel for UE 1 PDU session
![image](https://hackmd.io/_uploads/B1wXlRtfJe.png)
- DU 1 make DRB tunnel for UE 1 PDU session
![image](https://hackmd.io/_uploads/SyxIgRKzkg.png)
- DU 1 statistics for UE 1
![image](https://hackmd.io/_uploads/BJjNPpFzkg.png)
- CU-CP choose CU-UP 2 for UE 2 PDU session
![image](https://hackmd.io/_uploads/rkHcxRYfye.png)
- CU-UP 2 make DRB tunnel for UE 2 PDU session
![image](https://hackmd.io/_uploads/BysIZCYMkl.png)
- DU 2 make DRB tunnel for UE 2 PDU session
![image](https://hackmd.io/_uploads/SyxFbAtMJg.png)
- DU 2 statistics for UE 2
![image](https://hackmd.io/_uploads/SyjsvTKG1g.png)

