# 2024/04/02 Study Note (Akmal's Code Duplication with O1 Enabled)

###### tags: `2024`


**Goal:**
- [x] [1. Compile and Run Akmal's Thesis Code with O1 Enabled](#2-O-DU-Installation-and-Running-Steps)
- [x] [2. Connect to SMO by successfully do PNF Registration](#3-Connect-to-SMO-Steps)
- [x] [3. O-DU Performance Metrics stored in SMO's InfluxDB](#4-O-DU-Performance-Metrics-stored-in-SMOs-InfluxDB)
- [ ] [4. Display PM from InfluxDB to Grafana](#5-Display-PM-from-InfluxDB-to-Grafana)

 
**References:**
- [Akmal Handover | Thesis Repository User Guide](https://hackmd.io/@akmalns/BJKrRdqua#A-Installation)
- [Akmal's Code Duplication](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/Akmal's%20Code%20Duplication.md)
- [Send Metrics to SMO using VES Event and Display It in Grafana](https://hackmd.io/@akmalns/rJkBGdtkp)
- [Docs-for-BMW-Lab.md](https://github.com/bmw-ece-ntust/MIMO-Scheduler-on-Sch-Slice-Based/blob/o1-perf-metrics/Docs-for-BMW-Lab.md)
- [Tori | Integration & Testing [G] O-DU and [I] SMO](https://hackmd.io/@Min-xiang/H165aN2o6)
- [Heidi | [F] SMO to [G] O-DU Slice Enable Scheduler with O1 Enable](https://hackmd.io/@H141319/BkYwSCVSh)


**Contents:**
- [2024/04/02 Study Note (Akmal's Code Duplication with O1 Enabled)](#2024-04-02-study-note--akmal-s-code-duplication-with-o1-enabled-)
          + [tags: `2024`](#tags---2024-)
  * [0. Progress Summary](#0-progress-summary)
  * [1. Meeting with Akmal 2024/03/15](#1-meeting-with-akmal-2024-03-15)
  * [2. O-DU Installation and Running Steps](#2-o-du-installation-and-running-steps)
  * [3. Connect to SMO Steps](#3-connect-to-smo-steps)
  * [4. O-DU Performance Metrics stored in SMO's InfluxDB](#4-o-du-performance-metrics-stored-in-smos-influxdb)
  * [5. Display PM from InfluxDB to Grafana](#5-display-pm-from-influxdb-to-grafana)
  * [6. Problems](#6-problems)
    + [6.1. pnf registration error](#61-pnf-registration-error)
    + [6.2. EGTP : Failed to bind socket](#62-egtp--failed-to-bind-socket)
    + [6.3. DMAAP does not receive VES EVENT from O-DU](#63-dmaap-does-not-receive-ves-event-from-o-du)
    + [6.4. InfluxDB port didn't open](#64-influxdb-port-didn-t-open)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 0. Progress Summary

- System Architecture

![image](https://hackmd.io/_uploads/S1WKhInVA.png)

- Progress Table

| Step                                                | Progress |
| --------------------------------------------------- | -------- |
| [Compile and Run Akmal's Thesis Code with O1 Enabled](#2-O-DU-Installation-and-Running-Steps) | :heavy_check_mark:          |
| [Connect to SMO by successfully do PNF Registration](#3-Connect-to-SMO-Steps)  | :heavy_check_mark:          |
| [O-DU Performance Metrics stored in SMO's InfluxDB](#4-O-DU-Performance-Metrics-stored-in-SMOs-InfluxDB)   | :heavy_check_mark:          |
| [Display PM from InfluxDB to Grafana](#5-Display-PM-from-InfluxDB-to-Grafana)                 | :gear:          |


## 1. Meeting with Akmal 2024/03/15
- Steps to Duplicate Akmal's Work with O1 Enabled and Display the Results in Grafana:
    1. Compile and Run O-DU with O1 Enabled --> Without SMO configuration
    2. Configure O-DU (with O1 Enabled) to connect with SMO (in directory build/config)
    3. Mapping DMaaP in SMO (consult with Heidi or Tori), connect with InfluxDB (need to query data using influx language) and Grafana in SMO
    4. Configure Grafana Dashboard:
        - Data to be displayed: MCS Index & DL Throughput
        - Refresh time (5s) -> Need to configure JSON in Grafana

## 2. O-DU Installation and Running Steps
1. Ubuntu version = 18.04.6
![image](https://hackmd.io/_uploads/HyuWCzV26.png)

2. Linux Machine = 64 bit
![image](https://hackmd.io/_uploads/r1dB0MEhT.png)

3. Clone Directory `git clone https://github.com/bmw-ece-ntust/MIMO-Scheduler-on-Sch-Slice-Based.git`

4. Branch Checkout `git checkout o1-perf-metrics`

![image](https://hackmd.io/_uploads/ryJXg1YJ0.png)

5. Create new NETCONF user

![image](https://hackmd.io/_uploads/HyYPbyYJA.png)

6. Install Netconf Libraries

- Some modifications need to be made in `<O-DU High Directory>/MIMO-Scheduler-on-Sch-Slice-Based/build/scripts/install_lib_O1.sh`:
    - Netopeer2 repository access (see [Tori's note](https://hackmd.io/@Min-xiang/H165aN2o6#311-O-DU-Issue-repository-access) for details) 
    - libnetconf2 version (see [Tori's note](https://hackmd.io/@Min-xiang/H165aN2o6#312-O-DU-Issue-libnetconf2-version) for details)

![image](https://hackmd.io/_uploads/rJxWGeFJA.png)


7. Update the configuration according to system setup:
    1. `startup_config.xml` (no change, use default)
    2. `netconf_server_ipv6.xml` (no change)
    3. `oamVesConfig.json`
        - Change the OAM VES collector IP to the SMO IP
        - The SMO IP in our lab is dcae-ves-collector-service1
        - The OAM VES Collector Port is 31441
    4. `smoVesConfig.json`
        - Change the SMO VES collector IP to the SMO IP
        - The SMO IP in our lab is 192.168.8.6
        - The SMO VES Collector Port is 30205
    5. `netconfConfig.json`
        - Check your NIC MAC Address and IP using `ifconfig` command
        - In ubuntu, your MAC Address usually shown next to `ether`
        - In ubuntu, your IP address usually shown nex to `inet` (IPv4)
        - In my case, the VM MAC Address is `fa:16:3e:e2:81:fe` whereas the IPv4 Address is `10.0.10.30` (see [Tori's Note](https://hackmd.io/@Min-xiang/H165aN2o6#3214-Solution-3-Create-the-nginx-for-VES-Collector) for Details)
    
8. Install the Yang Modules and Load Initial Configuration:

![image](https://hackmd.io/_uploads/HkZ24gYy0.png)

9. Start Netopeer2-server

![image](https://hackmd.io/_uploads/SJXAa-tJC.png)

10. Compile O-DU `make odu MACHINE=BIT64 MODE=FDD O1_ENABLE=YES`
![image](https://hackmd.io/_uploads/ByVAoL4np.png)

11. Compile cu_stub `make cu_stub MACHINE=BIT64 NODE=TEST_STUB MODE=FDD O1_ENABLE=YES`
![image](https://hackmd.io/_uploads/r1KjjI43p.png)

12. Compile ric_stub `make ric_stub MACHINE=BIT64 NODE=TEST_STUB MODE=FDD O1_ENABLE=YES`
![image](https://hackmd.io/_uploads/BJtln8V3T.png)

13. ifconfig (this is the standard IP)
![image](https://hackmd.io/_uploads/rkR-B44n6.png)

14. Run `cu_stub`

![image](https://hackmd.io/_uploads/HJlX8fKyC.png)

- You may encounter:
    - [EGTP : Failed to bind socket](#62-EGTP--Failed-to-bind-socket)

15. Run `ric_stub`

![image](https://hackmd.io/_uploads/Bk_VLfF1C.png)

## 3. Connect to SMO Steps

16. Run `odu`

![image](https://hackmd.io/_uploads/BJ1KBcQl0.png)

- You may encounter:
    - [pnf registration error](#61-pnf-registration-error)

17. Push cell configuration using Netopeer-CLI

![image](https://hackmd.io/_uploads/H14ovoXlA.png)

18. ODU, RIC_STUB, and CU_STUB standby

- ODU:
![image](https://hackmd.io/_uploads/BkMaqoQl0.png)

- RIC_STUB:
![image](https://hackmd.io/_uploads/BywMismxC.png)

- CU_STUB:
![image](https://hackmd.io/_uploads/r1ersjmgC.png)

## 4. O-DU Performance Metrics stored in SMO's InfluxDB

19. First, you need to check which VesCollector is used in the SMO. In my case, its the `onap-dcae-ves-collector-7d7bf96dbc-c8hzh`
![image](https://hackmd.io/_uploads/B1YQT8h4A.png)

20. Try send a VES event from O-DU manually
```shell
curl -X POST \
   -H 'Content-Type: application/json' \
   -u sample1:sample1 \
   -d @file.json \                               #this file need to be in your directory
   https://192.168.8.6:30780/dcae-ves-collector/eventListener/v7
```
 example file.json
```json
{
	"event":	{
		"commonEventHeader":	{
			"domain":	"stndDefined",
			"eventId":	"pm1_1638984365",
			"eventName":	"stndDefined_performanceMeasurementStreaming",
			"eventType":	"performanceMeasurementStreaming",
			"sequence":	1,
			"priority":	"Low",
			"reportingEntityId":	"ODU-High",
			"reportingEntityName":	"ORAN-DEV",
			"sourceId":	"",
			"sourceName":	"ODU-High",
			"startEpochMicrosec":	1696391705807124,
			"lastEpochMicrosec":	1696391711807124,
			"nfNamingCode":	"7odu",
			"nfVendorName":	"POC",
			"nfcNamingCode":	"NFC",
			"timeZoneOffset":	"+00:00",
			"version":	"4.0.1",
			"stndDefinedNamespace":	"o-ran-sc-du-hello-world-pm-streaming-oas3",
			"vesEventListenerVersion":	"7.2.1"
		},
		"stndDefinedFields":	{
			"stndDefinedFieldsVersion":	"1.0",
			"schemaReference":	"https://gerrit.o-ran-sc.org/r/gitweb?p=scp/oam/modeling.git;a=blob_plain;f=data-model/oas3/experimental/o-ran-sc-du-hello-world-pm-streaming-oas3.yaml",
			"data":	{
				"id":	"pm1_1638984365",
				"start-time":	"2023-10-04T03:55:05Z",
				"administrative-state":	"unlocked",
				"operational-state":	"enabled",
				"user-label":	"pm-1",
				"job-tag":	"",
				"granularity-period":	5,
				"measurements":	[{
						"measurement-type-instance-reference":	"/o-ran-sc-du-hello-world:network-function/distributed-unit-functions[id='ODU-High']/cell[id='1']/supported-measurements[performance-measurement-type='user-equipment-average-throughput-downlink']/supported-snssai-subcounter-instances[slice-differentiator='1'][slice-service-type='1']",
						"value":	5493104,
						"unit":	"kbit/s"
					}, {
						"measurement-type-instance-reference":	"/o-ran-sc-du-hello-world:network-function/distributed-unit-functions[id='ODU-High']/cell[id='1']/supported-measurements[performance-measurement-type='user-equipment-MCS-Index']/supported-snssai-subcounter-instances[slice-differentiator='1'][slice-service-type='1']",
						"value":	17.806722689075631,
						"unit":	""
					}]
			}
		}
	}
}
```

![image](https://hackmd.io/_uploads/HkRPa82VR.png)

21. Check if VES event received on VesCollector
```shell
kubectl logs -n onap onap-dcae-ves-collector-...-... 
```
![image](https://hackmd.io/_uploads/B1z0pUnER.png)

22. Your PNF registration should be received by VES collector
![image](https://hackmd.io/_uploads/BJfLYD24R.png)


23. Now, check if O-DU's performance metrics is sent to this ves collector by running the O-DU with O1-enabled (it should be preiodically sent)
![image](https://hackmd.io/_uploads/B1AWcv2VA.png)

24. If PM is sent to VEScollector, next step is to check if dmaap-influxdb-adapter received the ves-event
```shell
kubectl logs -n o1ves o1ves-dmaap-influxdb-adapter-...-... 
```
![image](https://hackmd.io/_uploads/BJjbhSa40.png)

- You may encounter:
    - [dmaap-influxdb-adapter not receiving ves-event](#63-DMAAP-does-not-receive-VES-EVENT-from-O-DU)

25. Login to InfluxDB webpage
```shell
username: admin
password: 

    kubectl get secret -n o1ves o1ves-influxdb2-auth -o json
    # find the password then echo
    echo HzonvHTZquTlulGMNP1xX6QuT3Dprlxu= | base64 -d
```
![image](https://hackmd.io/_uploads/Hy2eJLp4R.png)

- You may encounter:
    - [InfluxDB port didn't open]()

26. Go to data explorer and check if influxDB received the performance metrics data from O-DU
```shell
http://192.168.8.6:30001/orgs/ea215acf6939f73f/data-explorer
```
![image](https://hackmd.io/_uploads/SkCSxUTE0.png)



## 5. Display PM from InfluxDB to Grafana

27. Login to Grafana webpage
```shell
username: admin
password: smo
```
![image](https://hackmd.io/_uploads/S1c0b8p4R.png)
28. Go to administration/data source
![image](https://hackmd.io/_uploads/H1SffITVR.png)

29. Check influxDB data source setting and make sure when you save, you get the green checkmark
```shell
Query Language: Flux
URL: http://o1ves-influxdb2.o1ves
Auth
    Basic auth: Disable
InfluxDB Details
    Organization: influxdb  #on first try, try not to change this
    Token: my-token         #on first try, try not to change this
```
![image](https://hackmd.io/_uploads/Sk3ijUTE0.png)

30. Test query on explore to see if Grafana can query data from influxDB
```shell
from(bucket: "my-bucket")
  |> range(start: -10m)
```
![image](https://hackmd.io/_uploads/r105T8T4C.png)

31. Make a new dashboard and check if the dashboard is working with test data
![image](https://hackmd.io/_uploads/BkQz1DaE0.png)

32. Run O-DU and check the data
![image](https://hackmd.io/_uploads/HJdNNvaEC.png)

33. Need to change parameters to get good graph and prevent segmentation fault



## 6. Problems

### 6.1. pnf registration error

danger
**Problem:** pnf registration error -> This error is because ODU cannot reach SMO (VES collector a.k.a. oamVes) for pnf registration

![image](https://hackmd.io/_uploads/ry4afmFkC.png)



**Solution 1:** if nginx-ingress is already installed in SMO, Change `oamVesConfig` vesV4IpAddress to "dcae-ves-collector-service1" and vesPort to "31441" (or according to nginx-ingress config now)

![image](https://hackmd.io/_uploads/rkl-vjXeA.png)


**Explanation:** In Release I, the VES Collector of SMO converts the internal authentication mechanism to external authentication. To transmit o1-ves events, OSC traefik or NYCU nginx-ingress must be installed for reverse proxy. The o-du configuration also needs to be modified. (see [Tori's note](https://hackmd.io/@Min-xiang/H165aN2o6#3214-Solution-3-Create-the-nginx-for-VES-Collector) for details)



**Solution 2:** if nginx-ingress is not installed in SMO (e.g. SMO was reinstalled), install nginx-ingress first, following Tori's note [here](https://hackmd.io/@Min-xiang/H165aN2o6#3214-Solution-3-Create-the-nginx-for-VES-Collector)




**Solution 3:** if you want to use SMO's default kong-ingress, Change `oamVesConfig` vesV4IpAddress to "192.168.8.6" and vesPort to "30780" (or according to kong-ingress config now). You need to Modify the O-DU source codes:
1. `src/o1/ves/VesEvent.cpp`
```cpp
mVesUrl = "https://" + mVesServerIp + ":" + mVesServerPort + "dcae-ves-collector/eventListener/v7";
```
2. `src/o1/ves/PerfMeasurementEvent.cpp`
```cpp
void PerfMeasurementEvent::createUrl()
{
      #ifdef StdDef
      mVesUrl = "https://" + mVesServerIp + ":" + mVesServerPort + "dcae-ves-collector/eventListener/v7";
      #else
      mVesUrl = "https://" + mVesServerIp + ":" + mVesServerPort + "dcae-ves-collector/eventListener/v7/events";
      #endif
      O1_LOG("\nURL=%s", mVesUrl.c_str());
}
```


### 6.2. EGTP : Failed to bind socket

danger
**Problem:** EGTP : Failed to bind socket

![image](https://hackmd.io/_uploads/SJh0domgA.png)



**Solution 1:** Assign ip address to O-DU, CU-Stub or RIC-Stub if you forgot it.

![image](https://hackmd.io/_uploads/rkuztomxA.png)



**Solution 2:** Kill existing process that use the interface we want to use

![image](https://hackmd.io/_uploads/SyIqYoQeC.png)

**Explanation:** Virtual machine is used together with other people. You can also be disconnected from VM due to connection problem. This means, you don't know if there is already cu_stub/ric_stub/odu running in the background. If yes, you need to kill it first before you run another cu_stub/ric_stub/odu because 1 interface can only be used by 1


### 6.3. DMAAP does not receive VES EVENT from O-DU

danger
**Problem:** dmaap-influxdb-adapter not receiving the ves-event but VES Collector already received




**Solution 1:** Make sure dmaap pods is available

![image](https://hackmd.io/_uploads/rkSxqH6NC.png)



**Solution 2:** Make sure DMAAP topic is set according to VES_MEASUREMENT_OUTPUT

![image](https://hackmd.io/_uploads/B1ANqHpVC.png)


**Explanation:** The default dmaap topic is fault, not measurement. If this is the case, you need to follow [Deployment of dmaap-InfluxDB-adapter, InfluxDB and Grafana](https://hackmd.io/@H141319/BkYwSCVSh#Deployment-of-dmaap-InfluxDB-adapter-InfluxDB-and-Grafana) in Heidi's note by using the proper [topic setting](https://hackmd.io/@H141319/BkYwSCVSh#Saving-the-VES-event-in-InfluxDB)


### 6.4. InfluxDB port didn't open

danger
**Problem:** InfluxDB port didn't open




**Solution 1:** refer to Heidi's [note](https://hackmd.io/@H141319/BkYwSCVSh#InfluxDB-Web-UI)
```shell
kubectl edit svc -n o1ves o1ves-influxdb2
# line 33 add: nodePort:30001
# line 41 type change to NodePort
```




