# <center>mMIMO scenario in VIAVI RicTest</center>

<center>Comparison of Massive MIMO Interference Management Based on SSB and CSI-RS and Multi-Site Performance Evaluation</center>

###### tags: `mMIMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue2024 8:39 PM][color=#38c16a]

:::info
**Introduction :**

- Comparison of Massive MIMO Interference Management Based on SSB and CSI-RS and Multi-Site Performance Evaluation

**Goal：**
- [x] [Compare SSB Beam with CSI-RS Beam](#2-1-Compare-SSB-Beam-with-CSI-RS-Beam)
- [x] [Compare SSB Beam with different antenna mask](#2-2-Compare-SSB-Beam-with-different-antenna-mask)
- [x] Simulate the Two-Sites-RIC-Test-SSB-GOB-mMIMO-interference
- [x] Simulate the Two-Sites-RIC-Test-CSI-RS-GOB-mMIMO-interference
- [x] Simulate the Eight-Sites-RIC-Test-SSB-GOB-mMIMO-interference
- [x] Simulate the Eight-Sites-RIC-Test-CSI-RS-GOB-mMIMO-interference
:::
**<center>Table of Content</center>**
[TOC]

## 1. Introduction

This note aims to evaluate and compare the performance of Massive MIMO (mMIMO) based on SSB (Synchronization Signal Block) and CSI-RS (Channel State Information Reference Signal) in different scales of multi-site scenarios.

We have designed a series of test scenarios covering different network configurations ranging from two to eight sites. Each configuration is tested using both SSB and CSI-RS to comprehensively compare the efficiency of these two technologies across various network scales.

The main observation indicators include:

- **Power usage per cell**
- **User Equipment (UE) and cell distribution maps**
- **Reference Signal Received Power (RSRP)**
- **Reference Signal Received Quality (RSRQ)**
- **Cell and user throughput**

:::info
**What is RSRP && RSRQ ?**

**RSRP :**
- RSRP is the average power of a specific reference signal received by the User Equipment (UE).
- This measurement is conducted by the User Equipment (UE) and reflects the signal strength experienced by the actual user.
- The typical range is from approximately -44 dBm (very strong) to -140 dBm (very weak). It is generally considered that:
    - -80 dBm: Excellent signal
    - -80 to -90 dBm: Good signal
    - -90 to -100 dBm: Fair signal
    - < -100 dBm: Weak signal

**RSRQ :**
- RSRQ stands for Reference Signal Received Quality, which is used to measure the quality of the received signal.
- It is calculated by the User Equipment (UE) and reported to the network.
- The typical range is from approximately -3 dB (excellent) to -19.5 dB (poor). Generally:
    - -10 dB: Excellent
    - -10 to -15 dB: Good
    - -15 to -20 dB: Fair to poor
- RSRQ combines information on signal strength and interference levels, thus providing a better reflection of the actual signal quality compared to RSRP alone.
:::
## 2. Simulation Environment
### 2-1. Compare SSB Beam with CSI-RS Beam
:::info
**What is CSI-RS Beam && SSB Beam ?**
**SSB Beam:**
- Primarily used for initial cell search and network synchronization, providing basic system information and stable network access. It is the primary means for User Equipment (UE) to connect to the network.
![image](https://hackmd.io/_uploads/SJMqm48_0.png =500x300)

reference: https://www.mathworks.com/content/dam/mathworks/mathworks-dot-com/images/responsive/supporting/campaigns/offers/5g-beam-management-white-paper/5g-beam-management-white-paper.pdf

**CSI-RS Beam:**
- Used for precise channel state estimation and advanced beam management, providing flexible and dynamic channel information to support more efficient data transmission and interference management.
![image](https://hackmd.io/_uploads/SJ_zV48OC.png =500x300)

reference: https://www.mathworks.com/content/dam/mathworks/mathworks-dot-com/images/responsive/supporting/campaigns/offers/5g-beam-management-white-paper/5g-beam-management-white-paper.pdf

**Correlation Between SSB Beam and CSI-RS Beam**
![image](https://hackmd.io/_uploads/rygUVVUuR.png)


reference: https://arxiv.org/pdf/2312.02178
:::


#### 2-1-1. Two-Sites-RIC-Test-SSB-CSI-RS-Indoor-Compare
- Site 1
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s1
    - Cells layout :
        - Advanced RF model : mMIMO-Urban-SSB
        - Azimuth (degrees) : 240
    - Indoor UE : 1
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB/CSI-RS
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 3
        - Number of vertical beams : 3
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - Antenna mask (0x) : FFFF
- Site 2
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s2
    - Cells layout :
        - Advanced RF model : mMIMO-Urban-CSI-RS
        - Azimuth (degrees) : 240
    - Indoor UE : 1
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB/CSI-RS
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 3
        - Number of vertical beams : 3
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - Antenna mask (0x) : FFFF

**User Equipment (UE) and cell distribution maps**

![Screenshot 2024-07-22 205237](https://hackmd.io/_uploads/HJtLUCouR.png)

**Power usage per cell**
![Power_usage](https://hackmd.io/_uploads/H1_KICjOA.png)
From this figure, you can observe that the ==power consumption== for both cells is the ==same==. 


**Advanced KPI Cell and UE data**
![advanced_cell](https://hackmd.io/_uploads/HymrP0jOR.png)
![advanced_UE](https://hackmd.io/_uploads/HkBpDCjdC.png)


From this two figure, you can observe that the ==KPI== from ==two cell== is the ==same==. But for UE KPI, there is ==small difference== from ==signal quality parameters==. This implied that ==SSB beam's== UE KPI ==better== than CSI-RS beam. 


**mMIMO reports**

- **SSB Beam RSRP**
![ssb_rsrp](https://hackmd.io/_uploads/BysxOCjd0.png)


- **CSI-RS Beam RSRP**
![csirs_rsrp](https://hackmd.io/_uploads/HJamdRi_C.png)

From this two figure, you can observe that ==CSI-RS Beam RSRP== are ==lower== than ==SSB Beam RSRP==. This result is in line with the results of the previous UE KPI, which indicated that SSB beam was better in signal quality.

#### 2-1-2. Two-Sites-RIC-Test-SSB-CSI-RS-Compare

- Site 1
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s1
    - Cells layout :
        - C1 : 
            - ==Advanced RF model : mMIMO-Urban-SSB==
            - Azimuth (degrees) : 240
        - C2 :
            - ==Advanced RF model : mMIMO-Urban-SSB==
            - Azimuth (degrees) : 120
        - C3 :
            - ==Advanced RF model : mMIMO-Urban-SSB==
            - Azimuth (degrees) : 0
    - Indoor UE : 1 for C1
    - Pedestrian : 1 for C2
    - Pedestrian : 6 for C3
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB/CSI-RS
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 3
        - Number of vertical beams : 3
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - Antenna mask (0x) : FFFF
- Site 2
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s2
    - Cells layout :
        - C1 : 
            - ==Advanced RF model : mMIMO-Urban-CSI-RS==
            - Azimuth (degrees) : 240
        - C2 :
            - ==Advanced RF model : mMIMO-Urban-CSI-RS==
            - Azimuth (degrees) : 120
        - C3 :
            - ==Advanced RF model : mMIMO-Urban-CSI-RS==
            - Azimuth (degrees) : 0
    - Indoor UE : 1 for C1
    - Pedestrian : 1 for C2
        - Speed : 0.1(m/s)
    - Pedestrian : 6 for C3
        - Speed : 0.1(m/s)
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB/CSI-RS
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 3
        - Number of vertical beams : 3
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - Antenna mask (0x) : FFFF

**User Equipment (UE) and cell distribution maps**

![Screenshot 2024-07-26 at 4.04.11 AM](https://hackmd.io/_uploads/B11rx4xKR.png)

- This scenario sets up 2 sites with different beam types, each site containing three cells. The first cell simulates an indoor UE (without movement). The second cell simulates 1 pedestrian UE, which only moves within that cell. The third cell simulates 6 UEs moving within that cell. The goal is to observe the energy consumption and the throughput of each UE in all situations, in order to analyze the impact of different beam types.


**Power usage per cell**

![Screenshot 2024-07-26 at 5.33.05 AM](https://hackmd.io/_uploads/BkW3ESxF0.png)

From this diagram, we can see that the power consumption of the indoor UE is only 42.8 W. It can also be observed that the power consumption of the SSB Beam and CSI-RS Beam is the same, both at 42.8 W. In comparison, the power consumption of a single pedestrian UE is 114 W, with SSB and CSI-RS also being the same. For 6 pedestrian UEs, the power consumption reaches 374 W. From this simulation, we can conclude that the power consumption of CSI-RS and SSB is identical.

**SSB && CSI-RS Beam Indoor 2UEs Throughput for C1 in different site**

![Screenshot 2024-07-26 at 4.15.21 AM](https://hackmd.io/_uploads/H1wqzElt0.png)

From this diagram, it can be seen that the throughput of the indoor UE is the same for both CSI-RS and SSB, both maintaining a steady rate of 0.0096. Therefore, there is no difference between the two in this aspect.

**SSB && CSI-RS Beam Pedestrian 2UEs Throughput for C2 in different site**

![Screenshot 2024-07-26 at 5.49.46 AM](https://hackmd.io/_uploads/HJ0t_BeKC.png)

From this diagram, it can be seen that the throughput of the pedestrian UE is also the same for both CSI-RS and SSB, both maintaining a steady rate of 0.0902. Therefore, there is no difference between the two in this aspect. However, the throughput is significantly higher than that of the indoor UE, which may be due to poor indoor signal quality.

**SSB Beam Pedestrian 6UEs Throughput for C3 in site 1**

![Screenshot 2024-07-26 at 5.38.58 AM](https://hackmd.io/_uploads/HJBMISgY0.png)


**CSI-RS Beam Pedestrian 6UEs Throughput for C3 in site 2**

![image](https://hackmd.io/_uploads/SkSWdSxKR.png)

From these figures, it can be seen that SSB Beam throughput in Cell 3 are more stable than CSI-RS Beam throughput. This can happen because the ==SSB== beam is ==wider== than CSI-RS and the signal transmission ==frequency== is ==greater== than CSI-RS. 

reference: https://www.sharetechnote.com/html/5G/5G_Phy_BeamManagement.html

**mMIMO reports**

- Site 1 SSB Beam (3 Cell)
![Screenshot 2024-07-26 at 6.07.07 AM](https://hackmd.io/_uploads/HkGjnBgtC.png)
![Screenshot 2024-07-26 at 6.01.45 AM](https://hackmd.io/_uploads/H1ZtsreFC.png)
![Screenshot 2024-07-26 at 6.02.04 AM](https://hackmd.io/_uploads/BybtiSgYC.png)
![Screenshot 2024-07-26 at 6.02.11 AM](https://hackmd.io/_uploads/HyZFiretA.png)
- Site 2 CSI-RS Beam (3 Cell)
![Screenshot 2024-07-26 at 6.07.07 AM](https://hackmd.io/_uploads/rkr2nrlKR.png)
![Screenshot 2024-07-26 at 6.03.52 AM](https://hackmd.io/_uploads/BkHg3BxK0.png)
![Screenshot 2024-07-26 at 6.03.59 AM](https://hackmd.io/_uploads/B1Sl3HlKR.png)
![Screenshot 2024-07-26 at 6.05.44 AM](https://hackmd.io/_uploads/SJCrnSlF0.png)

From these figure, it can be seen that the ==beams used== are ==only between 1, 6, and 9== and ==PRBUsedDl== from each beam it's ==almost same==, only from ==Cell 1 in Site 1==, it's use ==beam 1== with ==PrbUsedDl 7== in other hand, ==Cell 1 in Site 2==, it's use ==beam 5== and ==PrbUsedDl 7==.  


### 2-2. Compare SSB Beam with different antenna mask

- Site 1
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s1
    - Cells layout :
        - C1 : 
            - Advanced RF model : mMIMO-Urban-SSB
            - Azimuth (degrees) : 240
        - C2 :
            - Advanced RF model : mMIMO-Urban-SSB
            - Azimuth (degrees) : 120
        - C3 :
            - Advanced RF model : mMIMO-Urban-SSB
            - Azimuth (degrees) : 0
    - Indoor UE : 1 for C1
    - Pedestrian : 1 for C2
    - Pedestrian : 6 for C3
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB/CSI-RS
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 3
        - Number of vertical beams : 3
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - ==Antenna mask (0x) : 1111==
- Site 2
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s2
    - Cells layout :
        - C1 : 
            - Advanced RF model : mMIMO-Urban-SSB
            - Azimuth (degrees) : 240
        - C2 :
            - Advanced RF model : mMIMO-Urban-SSB
            - Azimuth (degrees) : 120
        - C3 :
            - Advanced RF model : mMIMO-Urban-SSB
            - Azimuth (degrees) : 0
    - Indoor UE : 1 for C1
    - Pedestrian : 1 for C2
        - Speed : 0.1(m/s)
    - Pedestrian : 6 for C3
        - Speed : 0.1(m/s)
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB/CSI-RS
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 3
        - Number of vertical beams : 3
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - ==Antenna mask (0x) : FFFF==

**User Equipment (UE) and cell distribution maps**

![Screenshot 2024-07-26 at 7.01.38 AM](https://hackmd.io/_uploads/SknuY8eKR.png)

**Power usage per cell**

![Screenshot 2024-07-26 at 7.15.16 AM](https://hackmd.io/_uploads/BkvinIxKA.png)

From this figure, it can be seen that the ==power consumption== from ==site 1 (antenna mask: 1111)== in all cell is ==lower== than site 2 (antenna mask: FFFF). This happen because site 1 use ==fewer antenna== than site 2. 

**SSB Beam Indoor 2UEs Throughput for C1 in different site**

![image](https://hackmd.io/_uploads/HkshxDeFR.png)

From this figure, it can be seen that throughput from Moving UE in site 1 is around 0.0092 and in site 2 is around 0.01. There for, ==throughput== for ==indoor UE== in ==site 1== is lower than site 2.  

**SSB Beam Pedestrian 2UEs Throughput for C2 in different site**

![image](https://hackmd.io/_uploads/H1Om-wgKC.png)

From this figure, it can be seen that the throughput from both site has similar value around 0.09, but from ==site 1==, it's ==more unstable== than site 2.   

**SSB Beam Pedestrian 6UEs Throughput for C3 in site 1**

![image](https://hackmd.io/_uploads/H1eUMPxKA.png)

**SSB Beam Pedestrian 6UEs Throughput for C3 in site 2**

![image](https://hackmd.io/_uploads/rybDQwxFC.png)

From this figure, it can be seen that ==throughput== from site 1 has ==avarage around 0.047 to 0.055== with ==fluctuative grafic==. In other hand, site 2 has ==average around 0.061 to 0.65== with ==more stable grafic==. Therefore, it can be concluded that ==fewer antenna mask (1111)== has ==lower== and ==unstable== throughput that ==full antenna mask (FFFF)==. 

**mMIMO reports**

- Site 1 SSB Beam with antenna mask 1111 (3 Cell)
![Screenshot 2024-07-26 at 6.07.07 AM](https://hackmd.io/_uploads/HkGjnBgtC.png)
![Screenshot 2024-07-26 at 7.50.38 AM](https://hackmd.io/_uploads/SkR4SPxFC.png)
![Screenshot 2024-07-26 at 7.50.53 AM](https://hackmd.io/_uploads/SkRNrvgKA.png)
![Screenshot 2024-07-26 at 7.51.03 AM](https://hackmd.io/_uploads/ryCEHDxF0.png)

- Site 2 SSB Beam with antenna mask FFFF (3 Cell)
![Screenshot 2024-07-26 at 6.07.07 AM](https://hackmd.io/_uploads/HkGjnBgtC.png)
![Screenshot 2024-07-26 at 7.51.17 AM](https://hackmd.io/_uploads/B1rNHPxtR.png)
![Screenshot 2024-07-26 at 7.51.30 AM](https://hackmd.io/_uploads/ryBVHwlFR.png)
![Screenshot 2024-07-26 at 7.51.41 AM](https://hackmd.io/_uploads/BkSVBPgF0.png)

From these figures, it can be seen that site 1 has fewer beam than site 2. It also can be seen that the PRB used in each beam is larger at site 2. 

### 2-3. Two-Sites-RIC-Test-SSB-GOB-mMIMO-interference

- Number of sites : 2/3/4/8
- Cells Configuration : 
    - Cell Profiles : 
        - Profile N77
            - Name : N77
            - Technology : NR-3600
- UE Groups :
    - Indoor
        - Number of UEs : 3 per sites
    - Pedestrian
        -  Number of UEs : 10 per sites


**Cells layout**

![Screenshot 2024-07-17 at 3.41.31 AM](https://hackmd.io/_uploads/BJlzarVuC.png)

**User Equipment (UE) and cell distribution maps**

![Screenshot 2024-07-17 at 2.49.06 AM](https://hackmd.io/_uploads/BJA6gBV_A.png)

**Power usage per cell**

![Screenshot 2024-07-17 at 3.16.28 AM](https://hackmd.io/_uploads/BkG7vBV_C.png)

**UEs RSRP** 

![Screenshot 2024-07-17 at 3.19.47 AM](https://hackmd.io/_uploads/rkV4_SNdA.png)

**UEs Throughput**

![Screenshot 2024-07-17 at 3.33.08 AM](https://hackmd.io/_uploads/rJ0ZoHEdA.png)

**Advanced KPI Cell and UE data**

![Screenshot 2024-07-17 at 3.36.39 AM](https://hackmd.io/_uploads/B1y13S4dR.png)

**mMIMO reports**

![Screenshot 2024-07-17 at 3.38.48 AM](https://hackmd.io/_uploads/B1FUhSN_R.png)

- Indoor-xx neighbouring SSB beams RSRP (dBm)
- Indoor-xx neighbouring CSI-RS beams RSRP (dBm)
    - RSRP(Reference Signal Received Power) is a key indicator for measuring signal strength.
- Advanced Beam KPI Data 
    - Viavi.Cell.Name
    - Viavi.Cell.Beam
        - Beam index in a mMIMO cell.
    - RRU.PrbUsedDl
        - Mean downlink Physical Resource Blocks (PRBs) used for data traffic 
    - RRU.PrbAvailDl
        - Maximum number of Physical Resource Blocks (PRBs) available for downlink
    - RRU.PrbTotDl
        - Total usage (in percentage) of Physical Resource Blocks (PRBs) on the downlink for any purpose
### 2-4. Two-Sites-RIC-Test-CSI-RS-GOB-mMIMO-interference

**Cells layout**

![Screenshot 2024-07-17 at 3.50.50 AM](https://hackmd.io/_uploads/HJkLyIEdR.png)

**User Equipment (UE) and cell distribution maps**

![Screenshot 2024-07-17 at 3.46.05 AM](https://hackmd.io/_uploads/SJlM0HVuR.png)

**Power usage per cell**

![Screenshot 2024-07-17 at 3.49.05 AM](https://hackmd.io/_uploads/SyzCAHNdC.png)

**UEs RSRP** 

![Screenshot 2024-07-17 at 3.52.49 AM](https://hackmd.io/_uploads/HkEpJIVuC.png)

**UEs Throughput**

![Screenshot 2024-07-17 at 3.53.02 AM](https://hackmd.io/_uploads/HJEylLNuC.png)

**Advanced KPI Cell and UE data**

![Screenshot 2024-07-17 at 3.54.40 AM](https://hackmd.io/_uploads/BkwzlIVdA.png)

**mMIMO reports**

![Screenshot 2024-07-17 at 3.55.36 AM](https://hackmd.io/_uploads/Sk3Hg8NOC.png)

### 2-5. Eight-Sites-RIC-Test-SSB-GOB-mMIMO-interference

**User Equipment (UE) and cell distribution maps**

![Screenshot 2024-07-17 at 4.15.02 AM](https://hackmd.io/_uploads/HJszH8VuA.png)

**Power usage per cell**

![Screenshot 2024-07-17 at 4.26.10 AM](https://hackmd.io/_uploads/rkVOw8N_C.png)


**UEs RSRP** 

![Screenshot 2024-07-17 at 4.30.12 AM](https://hackmd.io/_uploads/B1OwOI4uA.png)


**UEs Throughput**

![Screenshot 2024-07-17 at 4.30.44 AM](https://hackmd.io/_uploads/BJHYu8VOR.png)


**Advanced KPI Cell and UE data**

![Screenshot 2024-07-17 at 4.31.48 AM](https://hackmd.io/_uploads/rk96_LN_0.png)


**mMIMO reports**

![Screenshot 2024-07-17 at 4.32.24 AM](https://hackmd.io/_uploads/r19ktLNdA.png)


### 2-6. Eight-Sites-RIC-Test-CSI-RS-GOB-mMIMO-interference

**User Equipment (UE) and cell distribution maps**

![Screenshot 2024-07-17 at 5.11.18 AM](https://hackmd.io/_uploads/rk8GzDVOC.png)


**Power usage per cell**

![Screenshot 2024-07-17 at 5.12.20 AM](https://hackmd.io/_uploads/SkuSfv4OC.png)


**UEs RSRP** 

![Screenshot 2024-07-17 at 5.13.46 AM](https://hackmd.io/_uploads/BkxiGPNu0.png)


**UEs Throughput**

![Screenshot 2024-07-17 at 5.14.28 AM](https://hackmd.io/_uploads/BJvafwVdA.png)


**Advanced KPI Cell and UE data**

![Screenshot 2024-07-17 at 5.15.00 AM](https://hackmd.io/_uploads/BkP1QDNdR.png)


**mMIMO reports**

![Screenshot 2024-07-17 at 5.15.32 AM](https://hackmd.io/_uploads/SkLZQvVuA.png)
