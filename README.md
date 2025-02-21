# ORANSlice 

ORANSlice is an end-to-end open-source 5G network slicing framework for O-RAN. 
It (i) extends the 5G protocol stacks of OAI to support RAN slicing; (ii) implements an E2SM-CCC-based SM and xApp for RAN slicing control; (iii) conducts extensive testing and validation on [Arena](https://openrangym.com/experimental-platforms/arena) and [X5G](https://x5g.org/) testbeds to ensure the robustness and effectiveness of the implementation.

The framework of ORANSlice is shown in the diagram below:
<!-- ![End-to-end network slicing in O-RAN](./doc/ORANSlice_Framework.png ) -->
<img src="./doc/ORANSlice_Framework.png" width="500" />


If you use this software, please reference the following paper:
> H. Cheng, S. D’Oro, R. Gangula, S. Velumani, D. Villa, L. Bonati, M. Polese, G. Arrobo, C. Maciocco, T. Melodia, "ORANSlice: An Open-Source 5G Network Slicing
Platform for O-RAN," in Proceedings of The 30th Annual International Conference on Mobile
Computing and Networking (ACM MobiCom ’24), Washington, D.C., USA, November 18–22, 2024, [bibtex](https://ece.northeastern.edu/wineslab/wines_bibtex/Cheng2024oranslice.txt) [pdf](https://arxiv.org/pdf/2410.12978)

This work was supported by the U.S. National Science Foundation under Grant CNS-2117814.


## Deployment Tutorial

As shown in the above diagram, there are three parts in an end-to-end 5G network slicing framework: core network, radio access network, and near-real-time RIC. 
The ORANSlice repo consists of the code of OAI RAN and OAI CN. The repo of near-real-time RIC is in other repos.
In this tutorial, we will show how to deploy the three parts in Ubuntu 22.04.

```
git clone https://github.com/wineslab/ORANSlice.git
```


## 5G Core Network Slicing

We provide OAI and Open5GS core network slicing deployment in docker.

### OAI Core 
Go to folder `oai_cn` and follow the readme to deploy OAI core legacy version (v1.5.1) or latest version (develop branch).

### Open5GS
The code of depolying Open5GS core network in docker is given in [Github Repo](https://github.com/wineslab/docker_open5gs/tree/open5gs_slicing). 

## 5G RAN Slicing

The implementation of RAN Slicing is based on the OAI 5G protocal stack. 

### OAI monolithic gNB

#### Install protobuf-c for E2Agent
```
sudo apt-get update
sudo apt install protobuf-compiler
sudo apt install libprotoc-dev
git clone https://github.com/protobuf-c/protobuf-c && cd protobuf-c/ && ./autogen.sh && ./configure && make && sudo make install && sudo ldconfig
```

Go to folder `oai_ran` and follow the steps to build:
#### Build OAI gNB
```
cd cmake_targets
./build_oai -I
./build_oai -w USRP --ninja --gNB
```

The customized RAN slicing and E2 Agent code are integrated into OAI 2024.w28 release.

### Disaggregated gNB from NVIDIA Aerial and OAI 

Please refer to the tutorial in [ARC-OTA](https://docs.nvidia.com/aerial/aerial-ran-colab-ota/current/) and [Aerial CUDA-Accelerated RAN](https://docs.nvidia.com/aerial/cuda-accelerated-ran/index.html).

### OAI nrUE with multiple-PDU support
Our customized OAI nrUE with multiple-PDU support is given in 
> https://gitlab.eurecom.fr/oai/openairinterface5g/-/tree/NR_UE_multi_pdu

Follow the [official guide](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/NR_UE_multi_pdu/doc/NR_SA_Tutorial_OAI_nrUE.md) to build and run OAI nrUE.

Follow the [guide](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/NR_UE_multi_pdu/doc/RUNMODEM.md#at-command-interface-in-nr-ue) to configure multiple PDU session at nrUE.

**If you do not need multiple-pdu support at nrUE, you can build and run nrUE at `oai_ran`**

### COTS UE
You can use COTS 5G UE modules from Sierra Wireless or Quectel to establish network slicing connection.

## Near-Real-Time RIC

### OSC Near-RT RIC 
The O-RAN Software Community (OSC) Near-RT RIC Release E can be installed following this [guide](https://docs.o-ran-sc.org/projects/o-ran-sc-ric-plt-ric-dep/en/latest/installation-guides.html). Note that for this tutorial, the OSC Near-RT RIC runs on an OpenShift cluster, whereas a plain Kubernetes deployment might require specific networking configurations.

### E2Sim 
The e2sim is a component that can run on the same server as the gNB and has the duty to encapsulate/decapsulate the custom Service Model (SM) buffers to be sent to or received from the gNB. It communicates with the gNB via UDP sockets. The e2sim codebase can be found at this GitHub [Repo](https://github.com/wineslab/o-ran-e2sim/tree/x5g-e2sim).

### xApp

Our xAPP consists of two parts: a C/C++ based xAPP Base Station (BS) Connector and a python-based xAPP logic unit. The code can be found at the 
[Github Repo](https://github.com/wineslab/xapp-oai/tree/oranslice).

#### xApp BS Connector
The xApp BS connector component connects the pythonic xApp with the OSC Near-RT RIC. On one side, it receives the custom SM buffers to be encapsulated in E2AP (E2 Application Protocol) messages and sent to the RIC. On the other side, it retrieves the custom SM buffers from E2AP to be sent to the xApp.

#### xApp logic unit
The xApp logic unit contains some Python examples that can subscribe to selected RAN parameters, receive periodic indication messages, and send control requests to update these parameters.



## Run a 5G Network Slicing System based on ORANSlice

We give a step-by-step example to show how ORANSlice works based on full softwarized 5G Open RAN components.

### Legacy OAI core network

```
cd ~/ORANSlice/oai_cn/oai-cn5g-legacy/
./restart_cn.sh
```

### OAI gNB

RFSim mode:
```
cd ~/ORANSlice/oai_ran/cmake_targets/ran_build/build 
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa  --rfsim
```

Over-the-Air Transmission with USRP X310:
```
cd ~/ORANSlice/oai_ran/cmake_targets/ran_build/build 
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa --usrp-tx-thread-config 1
```

### OAI nrUE

RFSim mode:
```
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/nrUE_slice1.conf --rfsim --rfsimulator.serveraddr 127.0.0.1
```

OTA Transmission with USRP B210:
```
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000  --sa  -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/nrUE_slice1.conf --ue-fo-compensation -E
```

OTA Transmission with USRP X310:
```
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000  --sa  -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/nrUE_slice1.conf --ue-fo-compensation --usrp-args "addr=192.168.40.2"
```

**Modify the IMSI/NSSAI/DNN item of nrUE_slice1.conf to setup another OAI nrUE. For example, nrUE_slice2.conf.**


### E2Sim

```
cd o-ran-e2sim
./run_e2sim.sh <e2term-address> <e2term-port>
```

### xAPP


#### xApp BS Connector

```
cd xapp_bs_connector
./run_xapp.sh
```
Wait for the connector to finish its initialization, which is indicated by, for example, 
> Opened control socket server on port 7000

#### xApp Logic Unit
```
cd base-xapp
python slicing_ctrl_influxdb_xapp_kpm.py
```

### Others

The kpm-xapp.py script can be easily modified to save these metrics in an external dB, such as InfluxDB, and then display them on a dashboard, such as Grafana.

## Troubleshooting

### RAN Slicing functionality 

You can test the RAN Slicing functionality without xApp and near-RT RIC by applying the patch:
```
cd oai_ran
git apply ../doc/rrmPolicyJson.patch
```

After applying the patch, a file `rrmPolicy.json` is added to the root folder or `ORANSlice` and the MAC layer will read `rrmPolicy.json` periodically to update the internal slicing policy.


Accordingly, you need to set 
> SliceConf = "/home/wineslab/ORANSlice/rrmPolicy.json";

in `MACRLCs` of `ORANSlice.gnb.sa.band78.fr1.106PRB.usrpx310.conf` to the correct path of `rrmPolicy.json`.

Tune the value of `min_ratio` and `max_ratio` and see the throughput change of each slice. 
<!-- ### RAN and Core Network Connection Issues


#### gNB and OAI Core

#### nrUE and gNB

#### nrUE and OAI Core -->
