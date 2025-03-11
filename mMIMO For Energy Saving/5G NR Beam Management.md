# <center>5G NR Beam Management</center>

<center>Learning basic concept for Beam Management.</center>

###### tags: `mMIMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue2024 8:39 PM][color=#38c16a]

:::info
**Introduction :**

- Using the application to setup the initial configure.

**Goal：**
- [x] Reading 5G NR Beam Management white paper
:::
**<center>Table of Content</center>**
[TOC]

## 1. Concept of Beamforming
Beamforming technology achieves directional signal transmission by the coordinated operation of multiple radiating elements, producing a focused main lobe and weaker side lobes. Increasing the number of radiating elements can further enhance the concentration of the main lobe and reduce the interference from side lobes.

![Screenshot 2024-08-08 at 4.29.49 AM](https://hackmd.io/_uploads/BklUKIW5A.png)

<center><b>Figure, Beamforming with two and four and eight radiating elements.</b></center>

### 1-1. Antenna Element Arrays
If the radio wavelength λ times apart, two antennas result into a directional signal lobe due to the constructive interference. Increasing the number of antenna elements in an array will narrow down the main lobe width.

![Screenshot 2024-08-08 at 4.44.03 AM](https://hackmd.io/_uploads/B1rshUbq0.png)

<center><b>Figure, Cartesian plot with different antenna elements.</b></center>

With multiple antennas array, separating antenna by the fraction of λ will generate a main lobe. As antenna distance increase (λ/4, λ/2, 1λ ), main lobe gets narrower and multiple side lobes appears. When the distance d matches 1λ sidelobe almost as strong as main lobe will appear at ±90°.

![Screenshot 2024-08-08 at 4.39.31 AM](https://hackmd.io/_uploads/ryU5sIb5R.png)

<center><b>Figure, Distance between antenna elements vs. Side lobes behaviour.</b>b></center>

### 1-2. Types of Beamforming
#### 1-2-1. Analogue Beamforming
Analog beamforming achieves efficient and low-cost signal processing and transmission through a single data stream and phase shifters. However, its limitation of managing only a single data stream reduces its effectiveness in 5G systems that require multiple beams.

![Screenshot 2024-08-08 at 4.47.45 AM](https://hackmd.io/_uploads/HJDta8Z5C.png)

<center><b>Figure, Analogue Beamforming.</b></center>

#### 1-2-2. Digital Beamforming

Baseband beamforming technology achieves ==multi-beam management== and ==multi-user== support by equipping each antenna with independent transceivers and data converters, providing greater flexibility and capability. However, it has ==higher hardware requirements== and is ==mainly suitable for base stations== rather than terminal devices.

![Screenshot 2024-08-08 at 4.57.11 AM](https://hackmd.io/_uploads/HJC3kvZ9R.png)

<center><b>Figure, Digital Beamforming.</b></center>

#### 1-2-3. Hybrid Beamforming

Hybrid beamforming systems combine analog and digital technologies, achieving multi-stream precoding at both the baseband and RF layers. This approach maintains high performance while reducing costs by minimizing the number of physical RF chains, providing a balanced solution for 5G and future communication systems.

![Screenshot 2024-08-08 at 4.58.00 AM](https://hackmd.io/_uploads/SJnbgDb5C.png)

<center><b>Figure, Hybrid Beamforming.</b></center>

### 1-3. Beam Steering and Beam Switching
Beam steering and switching technologies achieve precise signal direction and user tracking by dynamically adjusting the antenna radiation pattern. They play a critical role in 5G millimeter-wave communication but require complex signal processing and precise timing control.

![Screenshot 2024-08-08 at 5.12.10 AM](https://hackmd.io/_uploads/H1fvmDW5R.png)

<center><b>Figure, Beam steering with four radiating elements with identical phase shift.</b></center>

![Screenshot 2024-08-08 at 5.12.32 AM](https://hackmd.io/_uploads/S1-wQwbqA.png)

<center><b>Figure, Beam switching with four radiating elements, one pair in ‘X’ phase and another in ‘Y’ Phase.</b></center>

### 1-4. Massive MIMO
When the number of antennas is more than the number of users or UEs it is termed as massive MIMO. 

Massive MIMO technology, by utilizing a large number of antennas combined with diversity, spatial multiplexing, and beamforming techniques, significantly enhances the capacity, data rate, and coverage of 5G networks while providing crucial support for beam management.

Applications in 5G NR:
- Impact on physical layer and higher layer resources
- ==Use of SS/PBCH blocks and CSI-RS==

![Screenshot 2024-08-08 at 5.21.23 AM](https://hackmd.io/_uploads/SJvPSwWqA.png)

<center><b>Figure, Massive MIMO.</b></center>

## 2. Beam Management

Beam management is a crucial L1/L2 procedure in 5G NR systems. Through the RACH and RRC connection phases, it enables the acquisition, selection, and maintenance of the optimal beam between the gNodeB and UE, laying the foundation for efficient uplink and downlink communication.

![Screenshot 2024-08-08 at 5.27.28 AM](https://hackmd.io/_uploads/ByHC8wZ9C.png)

<center><b>Figure, Beam Management phase transition.</b></center>

**RACH Procedure :**
- A gNodeB will transmit SSB burst periodically (5 ms, 10 ms, 20 ms, 40 ms, 80 ms, or 160 ms) default being 20 ms.
- The gNodeB periodically transmits burst signals containing multiple SSBs. The UE selects beams by measuring these SSBs, reports the best beam to the gNodeB, and initiates PRACH transmission on that beam, thereby achieving initial access and beam management.

**RRC Connected :**
- Once in RRC Connected status, beam refinement between gNodeB and UE is initiated to finely tune the best beams both at gNodeB and UE side. And then after continuous monitoring and -finetuning, beam refinement also helps in maintaining connection with strongest signal strength.

### 2-1. Beam Management Phases
![Screenshot 2024-08-08 at 5.46.04 AM](https://hackmd.io/_uploads/BJMEoP-qR.png)

<center><b>Figure, Beam Management Phases (Details on P-1, P-2, P-3).</b></center>

**P1 :**
**P2 :**
**P3 :**


## Reference
- https://apsr.anritsu.com/en-in/tm/5g-nr-beam-management?click-to-form=download-from-mobile-devices
