# <center>Optimizing Energy Consumption through RF Channel Reconfiguration</center>

<center>Thesis Proposal</center>

###### tags: `mMIMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Jan2 , 2024 3:38 PM][color=#38c16a]


**<center>Table of Content</center>**
[TOC]

## 1. Introduction
Using AI/ML to predict future traffic, in the mMIMO architecture, dynamically shutting down certain Tx/Rx arrays to achieve Energy Saving (ES) when the traffic falls below a predefined threshold, thereby reducing the power consumption of O-RU.

![Screenshot 2024-01-04 at 8.45.10 PM](https://hackmd.io/_uploads/S1zxF7VOp.png)


## 2. Problem

- OSC cannot automatically control the switching of Tx/Rx arrays in the mMIMO system.

## 3. Challenge

- Balancing the energy consumption introduced by multiple RF channels and the throughput gains achieved by beamforming.
- No existing rApps in OSC monitor the power consumption of the MIMO system (O-RU or E2 nodes), and there are no xApps to control Tx/Rx arrays for power reduction.

## 4. Contribution

- Develop an rApp to monitor the power consumption of MIMO system (O-RU or E2 nodes).
- Develop an ==xApp== for ==reconfiguring RF channels== to enforce energy savings.

## 5. System Architecture
### 5-1. Outline 
![Picture6](https://hackmd.io/_uploads/rJiBixvOa.png)

### 5-2. System Architecture in detail
![image](https://hackmd.io/_uploads/ryMDJiL_6.png)


## 6. RF Channel Reconfiguration flow diagram: AI/ML inference in Near-RT RIC

![drawio-2](https://hackmd.io/_uploads/HknlNpX_6.png)

## 7. Data requirment
### 7-1. Input data

- SMO/E2
    - Load statistics per cell and per carrier 
        - number of active users
        - average number of RRC connections
        - average number of scheduled active users per TTI
        - PRB utilization
        - DL/UL Cell/User throughput
        - PMI/CSI reports.
- O-RU
    - Power consumption metrics
    - ==Information of O-RU for supported Tx/Rx Array selection== along with power consumption (site/O-RU input power needed for certain EE KPIs).

### 7-2. Output data

- Non RT-RIC to Near RT-RIC
    - A1 Policy for ES optimization
- Near-RT RIC to E2 Node to O-RU
    - ==O-RU Tx/Rx Array selection==
    - Modification of the number of SU/MU MIMO spatial streams or data layers.
    - Modification of the number of SSB beams.
    - Modification of the O-RU Antenna Transmit power.

## 8. Environmental Possibility Analysis

Mimo system environment analysis. (Viavi Rictest)

### 8-1. **Massive MIMO: 3GPP Analog/Digital beamforming <font color="#f00">(TVM6230)</font>**

![Screenshot 2024-01-04 at 11.15.42 PM](https://hackmd.io/_uploads/SJ7psrEda.png)

- **mMIMO 3GPP/Analog** uses a single RF chain split among multiple antenna elements and its beam is controlled by adjusting analog phase shifters along the RF path.
- **mMIMO 3GPP/Digital** uses a dedicated RF chain and path for each antenna element.Phases and amplitudes are digitally controlled by baseband processing.

![Screenshot 2024-01-04 at 11.44.22 PM](https://hackmd.io/_uploads/BknJQ8Nda.png)


### 8-2. **mMIMO Energy Saving <font color="#f00">(TVM6229)</font>** -- TeraVM RIC Test - Release Notes v1.8
- mMIMO Energy saving feature is used to support RF channel switch off/on ES use case. It aims to reduce the power consumption of the mMIMO system by partially switching off/on RF channels. Depending on the traffic condition, xApp or rApp can easily switch certain RF channels by applying different antenna masks via APIs. The changes in power consumption are measured and reported in real-time.

![Screenshot 2024-01-04 at 2.39.50 PM](https://hackmd.io/_uploads/HyCaMR7_T.png)


### 8-3. mMIMO licensing path
![Screenshot 2024-01-04 at 10.52.18 PM](https://hackmd.io/_uploads/SyjHUH4O6.png)

- **Advanced RF Model License <font color="#f00">(TVM6203)</font>**: RIC Test Advanced NS3 model Integration for RAN Scenario Generator. This license is required to use RIC Test's Advanced RF Model features.
- **Massive MIMO License <font color="#f00">(TVM6207)</font>**: This license is required to use RIC Test’s Massive MIMO features. Note: The Advanced RF Model License (TVM6203) must also be acquired to use this license.
- **Massive MIMO: SSB Beamforming support <font color="#f00">(TVM6223)</font>**: This license is required to use the mMIMO Beam Group.
- **Massive MIMO: Digital Beamforming <font color="#f00">(TVM6230)</font>**: This license is required to use the 3GPP/Digital beamforming method in mMIMO Beam Group.
- **Massive MIMO - ES Tx/RX switch on/off <font color="#f00">(TVM6229)</font>**: This license is required to use the antenna mask for the Tx/RX switch on/off.
- **Massive MIMO: Beam-Based MLB <font color="#f00">(TVM6220)</font>**: This license is required to use the Beam-Based MLB feature.