---
title: Configure the VIAVI RIC Test using OSC SMO TEIV
tags: [SMO]

---

# <center>Configure the VIAVI RIC Test using OSC SMO TEIV</center>

<center>SMO O1 Ves/Netconf + TEIV + VIAVI Rictest</center>

###### tags: `SMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Jan2 , 2024 3:38 PM][color=#38c16a]

:::info
**Introduction：**

This note is used to analyze and plan the implementation process of the TEIV consumer rApp.

**Goal：**
- [ ] Complete the implementation process of the TEIV consumer rApp.


**Reference：**
- [Release J - Run in Kubernetes](https://wiki.o-ran-sc.org/display/SMO/Release+J+-+Run+in+Kubernetes)
- [[J Release] SMO Topology & Inventory (TEIV)](https://hackmd.io/@Jerry0714/BkkmMXKPR)
- [[J Release] SMO Topology & Inventory (TEIV) Installation Guide](https://hackmd.io/@Jerry0714/HkTXfrUdC)
- [[J Release] SMO Topology & Inventory (TEIV) Developer Guide](https://docs.o-ran-sc.org/projects/o-ran-sc-smo-teiv/en/latest/developer-guide.html#)
- [Gerrit smo/teiv.git](https://gerrit.o-ran-sc.org/r/gitweb?p=smo/teiv.git;a=tree;h=ea1ce6271dfe7d94b004c09d89254c4924c937fc;hb=ea1ce6271dfe7d94b004c09d89254c4924c937fc)
- [O-RAN Work Group 10 Topology Exposure and Inventory Management Services Use Cases and Requirement Specification (O-RAN.WG10.TE&IV-R003-v02.00)](https://specifications.o-ran.org/specifications)
:::
**<center>Table of Content</center>**
[TOC]

## 1. O-RAN network provisioning
### 1-1. System Architecture

![bbb](https://hackmd.io/_uploads/S1MCJ1-0A.png)


### 1-2. O-RAN network provisioning Operation
**Spec :** (O-RAN.WG10.TE&IV-R003-v02.00)

**Goal :** O-RAN network provisioning using TE&IV services

**Actors and Roles :** TE&IV Service Producer and TE&IV Service Consumer

**Pre-conditions :**
- O-RAN Network Planning completed as per Clause 4.1.5.3 and inventory objects are created with status as Planned
- The PNF is constructed/installed and ready to be registered and Activated
- O-DU and O-RU are PNFs and the inventory objects corresponding to these RAN Nodes are available in the TE&IV repositories with a Planned status and ready to be registered and activated


**O-RAN Service Deployment Procedure :** 
- **Step 1:** 
    - TE&IV Service Consumer sends a ==query== to the TE&IV Service Producer to get Planned O-RAN Service inventory objects
:::spoiler Implement
Here we can use this query command to get the information throught topology-exposure.
```javascript=
curl -k --http2 --header "Accept: application/yang.data+json, application/problem+json" -X GET http://10.233.102.166:8080/topology-inventory/v1alpha11/domains/CLOUD_TO_RAN/entity-types/NRCellDU/entities | jq
```
:::
- **Step 2:** 
    - Response from TE&IV Service Producer with ==details of the Planned inventory objects==
:::spoiler Implement
![image](https://hackmd.io/_uploads/SkpHgzqpR.png)
:::
- **Step 3:** 
    - TE&IV Service Consumer ==updates== the inventory objects with deployment specific ==parameter values==. The request also tranistions the O-RAN Service inventory object status to Assigned.
:::spoiler Implement
Here, We can use merge to update the specific parameter. This specific parameter will send to the topology-ingestion through kafka.
```javascript=
ce_type:::ran-logical-topology.merge
```
這一步可行，但我們不知道怎麼做到如何在TEIV裡標記已配置(分配)，需確認TEIV是否有欄位可以被標記。
:::
- **Step 4:** 
    - ==Confirmation== of successful inventory object update in TE&IV Repository
:::spoiler Implement
We can use query command to check again the data is already update or not. If not consumer will notice you update is not successful. 
:::
- **Step 5:** 
    - Incase of a ==successful deployment== of the O-RAN Service, the TE&IV Service Consumer sends a request to TE&IV Service Producer to update the inventory object with the details of the O-RAN Service being deployed and the status of the corresponding inventory object is changed to Deployed.
    - In case of a ==failure in deployment== of O-RAN Service, the status of the O-RAN Service inventory objects remains Assigned so that the deployment can be re-initiated without repeated assignment of parameter values. The use case flow stops with the deployment
failure.
:::spoiler Comment
這一步rApp可能需要有偵測DU/CU/Rictest是否已有部署之功能DU/CU/Rictest 也需要有註冊 TEIV consumer rApp 的功能，這樣才能告訴rApp目前已部署並等待配置，rApp也會請求TEIV更新狀態。
:::
- **Step 6:** 
    - ==Confirmation== of the status update on O-RAN Service Inventory Object in TE&IV Repository
:::spoiler Comment
這裡rApp可能要再次來確認TEIV裡對象的狀態是否已被更新
:::
- **Step 7:**
    - After completion of the RAN Node deployment, once the status of the associated inventory objects are updated as Deployed, the O-RAN Service activation is initiated.
    - On successful completion of RAN Node activation, the status of the O-RAN Service inventory object is transitioned to Active.
    - Incase the O-RAN Service activation failed (or if any of the RAN Node
activation failed), the status remains in Deployed. The use case flow stops with the activation failure of O-RAN Service. The subsequent flows are continued once the retry results in a sucesful activation.
:::spoiler Comment
從TEIV再次確認狀態為已部署後，TEIV consumer rApp 透過Netconf來配置TEIV裡已被配置的參數。如果成功配置，請求TEIV更新狀態，若無則保持原來狀態。
:::
- **Step 8:**
    - ==Confirmation== of the status update on O-RAN Service Inventory Object in TE&IV Repository
:::spoiler Comment
這裡rApp可能要再次來確認TEIV裡對象的狀態是否已被更新
:::
### 1-3. Sequence Diagram

![Screenshot 2024-10-09 at 10.22.13 AM](https://hackmd.io/_uploads/HJXgtwmkke.png)


![Screenshot 2024-09-20 at 5.29.18 AM](https://hackmd.io/_uploads/rkzavzcTR.png)


![Screenshot 2024-09-20 at 6.12.18 AM](https://hackmd.io/_uploads/S1u0WXc60.png)


### 1-4. Inventory Object Status Values

Throughout the life cycle of the Network provisioning use case the RAN Inventory object status in the TE&IV repository is transitioned across various values depending on the orchestration and provisioning stage of the associated network resources.The status values are updated by the TE&IV Service Consumer through the services exposed by the TE&IV Service Producer.

For the Network provisioning use case following status values are considered.

![Screenshot 2024-09-20 at 4.22.06 AM](https://hackmd.io/_uploads/rymbd-qpC.png)

The transitioning across the above status values and corresponding actions are shown in the diagram below.

![Screenshot 2024-09-20 at 4.25.50 AM](https://hackmd.io/_uploads/B1xJKW9aA.png)

### 1-5. O1 Netconf parameter supported in VIAVI Rictest
The O1 NETCONF utilizes the 3GPP Release 17 Yang models to implement the ManagedElement:

- GNBDUFunction
    - “id”
    - “attributes”
        - gNBId
        - gNBIdLength
        - gNBDUId
        - gNBDUName
        - NRCellDU
            - “id”
            - “attributes”
                - "cellLocalId"
                - "nRPCI"
                - “pLMNInfoList” includes the attribute “plmnId” with the values “mcc” and “mnc”
                - "ssbFrequency"
                - arfcnDL
                - arfcnUL
                - “CPCIConfigurationFunction” includes the “cSonPciList” attribute with “NRPci” value
        - NRSectorCarrier
            - “id”
            - “attributes”
                - "configuredMaxTxPower"
                - "CommonBeamformingFunction"
                    - “id”
                    - “attributes”
                        - “digitalAzimuth"
                        - "digitalTilt"
        - “viavi-attributes”
        - cellSize
        - latitude
        - longitude
- GNBCUCPFunction
    - “id”
    - “attributes”
        - “gNBId”
        - “gNBIdLength”
        - NRCellCU
            - “id”
            - “attributes”
                - “cellLocalId"
                - NRCellRelation
                    - “id”
                    - “attributes”
                        - “IsESCoveredBy”
                        - “adjacentNRCellRef”
                - CESManagementFunction
                    - “id”
                    - “attributes”
                        - “energySavingControl”
                        - “energySavingState”
## 2. Implementation of TEIV consumer rApp
:::info
**Action Item**
- [x] [Step 1: Query inventory objects.](#2-1-Step-1-ampamp-2)
- [x] [Step 2: Show the query result.](#2-1-Step-1-ampamp-2)
- [x] [Step 3: Update the specefic parameter in TEIV, set the status to "Assigned".](#2-2-Step-3-ampamp-4)
- [x] [Step 4: Confirmation of successful inventory object update in TE&IV Repository](#2-2-Step-3-ampamp-4)
- [x] [Step 5: Implement deployed status operation.](#2-4-Step-5-ampamp-6)
- [x] [Step 6: Confirmation of the status update in TE&IV Repository](#2-4-Step-5-ampamp-6)
- [x] [Step 7: Implement Active status operation.](#2-5-Step-7-ampamp-8)
- [x] [Step 8: Confirmation of the status update in TE&IV Repository](#2-5-Step-7-ampamp-8)
:::
:::success
**Git hub link:** https://github.com/bmw-ece-ntust/TEIV-SC-rApp
:::
### 2-1. Flow chart

[![](https://mermaid.ink/img/pako:eNqNlNtu4jAQhl_F8gpxEyrCMaTSSm1CChRope5BWtILNxnA22BHjtOWBt59jXOA9GpzlRn__8xnx5kMBzwEbONGI6OMShtlTbmFHTRt1HwhCTQNlCd-EUHJSwSJWslQMxZ0R8Te4REXJ-238cDret5JHlEG5_zAsca3Oi_hQ57zXf00j8djo-GzjSDxFv1wfYbUc7N6kkTIZ9RqfUe3q6kCoySin4DeSornXHmrJU624ErDBYp5iBJJZJocc4FzEhweVVqkjFG2OSB3dQcSJSDeaACIhKGApKx3ljMuzxYnX3X16lMaBMpxQGNdKEiFACZ124orV3qERqmAyj_WtF7m8F1MBBSWgtTTFmdL2AYSFIKEQEJ4QHerOd-goMgTFqINMBDKWewUCYi5Oq3LMkteOqrmd7r5JHO2ELyitTosBu_oMSKMQYimbskxyQvU1w5ounoU_LRvFBdp_vJXIZY7npR92Vfn7EvLmyShm3rPqWab5cGsArhUHtB9RUDKfB1hdolQ986_MLgQR3xfY7jXDPM8mFcMl8oDWlQMYZmvM8wvGere5epnHJ4-G4_C-m1Z6NbLPFjq4GH1m1CpeQNNTplUN5ZEheUhv_g-81kQqfNwYa2Q1iSNlIlGkV38kEYiBX8Fu_gPi7D1TkO5tTvxx3XNH9CEclYUcG_Gluf8bwHkGJ4xMWbGvKpzjQ28A7EjNFQTJjuB-1gPEx_b6rXg9bHPjkpKUsmf9izAthQpGFjwdLPF9ppEiYpSfXYuJWpU7KpsTNgfznelRYXYzvAHtntt62rUNvvWoGeanc6wNzTwHtstc3Q1MLtWd9TvdMxuu211jgb-1CXMq257NOyb7YFpjXr9Qd8yMISnsbLIZ6Qelcd_5wKm_A?type=png)](https://mermaid.live/edit#pako:eNqNlNtu4jAQhl_F8gpxEyrCMaTSSm1CChRope5BWtILNxnA22BHjtOWBt59jXOA9GpzlRn__8xnx5kMBzwEbONGI6OMShtlTbmFHTRt1HwhCTQNlCd-EUHJSwSJWslQMxZ0R8Te4REXJ-238cDret5JHlEG5_zAsca3Oi_hQ57zXf00j8djo-GzjSDxFv1wfYbUc7N6kkTIZ9RqfUe3q6kCoySin4DeSornXHmrJU624ErDBYp5iBJJZJocc4FzEhweVVqkjFG2OSB3dQcSJSDeaACIhKGApKx3ljMuzxYnX3X16lMaBMpxQGNdKEiFACZ124orV3qERqmAyj_WtF7m8F1MBBSWgtTTFmdL2AYSFIKEQEJ4QHerOd-goMgTFqINMBDKWewUCYi5Oq3LMkteOqrmd7r5JHO2ELyitTosBu_oMSKMQYimbskxyQvU1w5ounoU_LRvFBdp_vJXIZY7npR92Vfn7EvLmyShm3rPqWab5cGsArhUHtB9RUDKfB1hdolQ986_MLgQR3xfY7jXDPM8mFcMl8oDWlQMYZmvM8wvGere5epnHJ4-G4_C-m1Z6NbLPFjq4GH1m1CpeQNNTplUN5ZEheUhv_g-81kQqfNwYa2Q1iSNlIlGkV38kEYiBX8Fu_gPi7D1TkO5tTvxx3XNH9CEclYUcG_Gluf8bwHkGJ4xMWbGvKpzjQ28A7EjNFQTJjuB-1gPEx_b6rXg9bHPjkpKUsmf9izAthQpGFjwdLPF9ppEiYpSfXYuJWpU7KpsTNgfznelRYXYzvAHtntt62rUNvvWoGeanc6wNzTwHtstc3Q1MLtWd9TvdMxuu211jgb-1CXMq257NOyb7YFpjXr9Qd8yMISnsbLIZ6Qelcd_5wKm_A)

### 2-2. Step 1 && 2

No any object and instance in TE&IV service and TE&IV consumer rApp keep monitor the service healthy. Waiting for new object to be configed.

![Screenshot 2024-10-05 at 11.29.54 PM](https://hackmd.io/_uploads/ByWCj0R0A.png)

TE&IV consumer rApp will automatically detect the entity. Show the currently object status and the object initial status is **Not Planned**

![Screenshot 2024-10-17 at 2.10.54 PM](https://hackmd.io/_uploads/H1bKcmA1Jg.png)


### 2-3. Step 3 && 4
If the object status change to the **Planned**, TE&IV consumer rApp configuring parameters to TE&IV Service and will set the status to **Assigned**.

This part TE&IV consumer rApp will automatically to find the correct config file in rApp. If there are Planned objects match the config file, rApp will set the config to TE&IV service.

![Screenshot 2024-10-17 at 2.22.59 PM](https://hackmd.io/_uploads/HJaO6m0kJx.png)
![Screenshot 2024-10-17 at 2.24.29 PM](https://hackmd.io/_uploads/ByvhTQCyyx.png)


If not match the config file, rApp will tell you there are no config in rApp.

![Screenshot 2024-10-05 at 11.52.58 PM](https://hackmd.io/_uploads/S1k-Zk1kJl.png)

Delete the all config entity in TE&IV service.

![Screenshot 2024-10-05 at 11.57.56 PM](https://hackmd.io/_uploads/HJ3MGy1Jkg.png)

**Notice:** Currently the status report is already revise, please see the Step 5 && 6
### 2-4. Step 5 && 6
When the status change to assigned, means that the rApp already set the config in TE&IV and then will do the following processing.
- Connect to the VIAVI Rictest via the O1 Netconf
- Automatically to detact the deployed entity in Rictest.
- Once detacted successful, the status will change to the Deployed.

![Screenshot 2024-10-16 at 1.09.30 AM](https://hackmd.io/_uploads/ryuyf7hJkg.png)

Processing the assigned tatus.
Request and get Netconf config from Viavi Rictest **192.168.8.28**

![Screenshot 2024-10-16 at 4.16.29 AM](https://hackmd.io/_uploads/Byq6aBh11g.png)

Automatically to detact the deployed entity in Rictest and the status will change to the Deployed.

![Screenshot 2024-10-17 at 1.49.04 PM](https://hackmd.io/_uploads/SJMwS70y1l.png)


Currently status report change to Deployed.

![Screenshot 2024-10-17 at 1.44.15 PM](https://hackmd.io/_uploads/ryC5VXR1kx.png)

### 2-5. Step 7 && 8

- Once deployed status is complete, the rApp will compare the attributes in TE&IV and Netconf config.
- If there are attributes mapped in Netconf config and also the value is different. rApp will set new config to Netconf.
- Status change to active.

#### 2-5-1. rApp compare the attributes in TE&IV and Netconf config.
- For ManagedElement
![Screenshot 2024-10-25 at 2.04.37 PM](https://hackmd.io/_uploads/ryQfBndgJl.png)
    - There are nothing configuration parameter in both of them.
- For GNBDUFunction
![Screenshot 2024-10-25 at 2.18.38 PM](https://hackmd.io/_uploads/Sk3IunOgye.png)
    - In this part rApp automatically detact the parameter in TE&IV
        - gNBDUId
        - gNBIdLength
        - gNBId
        - fdn
        - cmId
        - status
        - dUpLMNId
    - rApp automatically detact the parameter in Netconf
        - gNBDUId
        - gNBIdLength
        - gNBId
        - gNBDUName
        - peeParametersList
    - There are common config parameter between TE&IV and Netconf, but in TE&IV don't have any configure can be setted to Netconf.
    **Result**
    ![Screenshot 2024-10-25 at 2.26.54 PM](https://hackmd.io/_uploads/rJ6Bcndl1x.png)
- For NRSectorCarrier
![Screenshot 2024-10-25 at 1.55.21 PM](https://hackmd.io/_uploads/BJwNSh_x1x.png)
    - In this part rApp automatically detact the parameter in TE&IV
        - frequencyUL
        - arfcnUL
        - arfcnDL
        - frequencyDL
        - cmId
        - fdn
        - status
        - configuredMaxTxPower
        - bSChannelBwDL
    - rApp automatically detact the parameter in Netconf
        - configuredMaxTxPower
    - Because the configuredMaxTxPower is the common config between the TE&IV and Netconf, and also the value is different. The rApp will use TE&IV parameter to set config to Netconf.
    **Result**
    ![Screenshot 2024-10-25 at 2.16.49 PM](https://hackmd.io/_uploads/HJfgd3_lJe.png)
    
#### 2-5-2. Status change to active.

![Screenshot 2024-10-25 at 3.49.38 PM](https://hackmd.io/_uploads/HJFaaTdx1l.png)


#### 2-5-3. Final Result
**Before**

![Screenshot 2024-10-25 at 2.32.17 PM](https://hackmd.io/_uploads/ryh2ohulke.png)

**After using rApp automatically config**

![Screenshot 2024-10-25 at 2.33.08 PM](https://hackmd.io/_uploads/B1oyn3_ekx.png)


### Additional function
If the TEIV ingestion is not working, rApp will automatically solved this problem.

![Screenshot 2024-10-01 at 1.56.03 PM](https://hackmd.io/_uploads/Bky7kGYRA.png)

## 3. Demo video
Demo video Link: https://www.youtube.com/watch?v=lMxky0gEZVA