# <center>The use cases of mMIMO in energy saving</center>

<center>Research on feasible algorithms for energy saving in mMIMO.</center>

###### tags: `mMIMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue2024 8:39 PM][color=#38c16a]

:::info
**Introduction :**

- A note on mMIMO energy-saving algorithms

**Goal：**

:::
**<center>Table of Content</center>**
[TOC]

## 1. Basic concept for energy saving in mMIMO system

### 1-1. [Intelligent Energy Saving Technology and Strategy of 5G RAN](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=9828681)

- **Published in:** 2022 IEEE International Symposium on Broadband Multimedia Systems and Broadcasting (BMSB)
- **Date of Conference:** 15-17 June 2022
- **Publisher:** IEEE

#### 1-1-1. MIMO-related energy-saving technologies:

- **Channel shutdown**
    - automatically shuts down some transmit channels to save energy.
    - automatically adjusts the transmit power of the common channels in the cell.
- **Carrier shutdown**
    - In the leisure scene such as at night, some carriers can be shut down to reduce the energy consumption.
    - ==We can observe uplink and downlink PRB usage.==
    - handover 
![Screenshot 2024-04-25 at 6.05.38 PM](https://hackmd.io/_uploads/Sy3bojPbC.png)
- **Deep Sleep**
    - In shopping malls, subways, gymnasiums, offices and other scenarios, when the network is in on service status for a long time(for example, at night) , it is necessary to turn off the cell and let the AAU enter a deep sleep state.
    - only the eCPRI which communicates with BBU is kept-alive
![Screenshot 2024-04-25 at 6.07.35 PM](https://hackmd.io/_uploads/B1AtijvbR.png)

#### 1-1-2. AI-related energy-saving technologies:
- **Scenario Identification** - ==Using AI models to identify== typical energy-saving ==scenarios== and implement corresponding ==energy-saving strategies==.
- **Traffic Prediction** - Predicting future traffic to enable/disable energy-saving functions in advance.
- **Base Station Cooperative Energy Saving** - ==Incorporating nearby/related base station information to improve energy-saving efficiency through AI joint modeling.==
    - Granger Causality Test
    - ==LSTM==
    - Graph Neural Networks
    - ==Multi-Agent Reinforcement Learning==
- **Network-wide Evaluation** - Using AI to assess the overall performance of the network, preventing excessive energy-saving from impacting service quality.

## 2. AI/ML in mMIMO system
### 2-1. [Evaluation of Machine Learning Algorithms on Power Control of Massive MIMO Systems](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=9908031)
:::info
- **Published in:** 2022 13th International Symposium on Communication Systems, Networks and Digital Signal Processing (CSNDSP)
- **Date of Conference:** 20-22 July 2022
- **Publisher:** IEEE
:::
#### 2-1-1. Introduction
- The paper discusses applying machine learning algorithms for ==power control in massive MIMO systems to maximize spectral efficiency==. Traditional heuristic algorithms (e.g. WMMSE) for power control have good performance but high computational complexity.

#### 2-1-2. Power Control Problem in Massive MIMO Systems
- Explains the importance of power control in cellular and cell-free massive MIMO networks.
- Introduces the ==WMMSE algorithm== approach to ==solve the power control problem==, but it has the drawback of ==high computational complexity==.
- ==Proposes using machine learning algorithms for power control== estimation to reduce computational complexity.

#### 2-1-3. Experimental Setup
- Constructs cellular and cell-free massive MIMO network datasets, each containing 50,000 samples.
- Describes the 6 machine learning regression algorithms to be evaluated: deep neural networks (DNN), deep Q-learning (DQL), support vector machines (SVM-RBF), K-nearest neighbors (KNN), linear regression (LR), and decision trees (DT).

#### 2-1-4. Experimental Results
- ==Evaluates the performance of the above 6 machine learning algorithms== on the two network types.
- Based on spectral efficiency, area under the curve (AUC), etc., the DNN algorithm performs closest to the heuristic WMMSE algorithm and has the shortest execution time.
- DQL algorithm performs second best, while DT performs the worst.
- Compared to WMMSE, using machine learning algorithms can reduce execution time by nearly 50%.

#### 2-1-5. Conclusion
- Machine learning algorithms can well approximate WMMSE power control, with the ==DNN algorithm performing best== with highest computational efficiency.

### 2-2. [Integration of Massive MIMO and Machine Learning in the Present and Future of Power Consumption in Wireless Networks: A Review](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=9905123)

:::info
- **Published in:** 2022 IEEE 7th Forum on Research and Technologies for Society and Industry Innovation (RTSI)
- **Date of Conference:** 24-26 August 2022
- **Publisher:** IEEE
:::

#### 2-2-1. Introduce 
- The introduction section highlights the importance of energy efficiency in wireless networks in 5G and future networks, as well as some existing energy-saving technologies.

#### 2-2-2. The possibility of integrating ML with mMIMO for intelligent power switching of base station antenna arrays to achieve energy savings

- ntroduced machine learning techniques such as ==Reinforcement Learning (RL) and Deep RL==, which enable intelligent decisions on ==activating/deactivating mMIMO antenna arrays based on network load==.
- It is noted that ML can intelligently control the switching of mMIMO antenna arrays by monitoring ==historical traffic patterns and predicting future loads==, thus achieving on-demand power supply and minimizing energy consumption.

#### 2-2-3. What learning methods might be employed to achieve energy efficiency in mMIMO systems?

- ==Supervised learning== can be utilized to model network traffic patterns.
- The ==distributed Q-Learning algorithm== can be employed to ==pre-train base station switching strategies==. Q-Learning is a significant algorithm in reinforcement learning.
- Using ==deep learning algorithms== to enable the transmission system of mMIMO networks to ==learn user locations==, thereby optimizing beamforming directions and downlink power allocation.