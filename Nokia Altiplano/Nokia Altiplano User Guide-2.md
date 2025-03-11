# <center>Nokia Altiplano User Guide</center>

<center>The implementation after install the Altiplano platform.</center>

###### tags: `Nokia Altiplano`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue2024 8:39 PM][color=#38c16a]

:::info
**Introduction :**

- Using the application to setup the initial configure.

**Goal：**
- [x] [1. Adding license to Nokia Altiplano.](#2-1-Add-the-license-to-Nokia-Altiplano)
- [x] [2. Connect inp-altiplano-ac to inp-altiplano-av](#2-2-Connect-inp-altiplano-ac-to-inp-altiplano-av)
- [x] [3. Add the device extension](#2-3-Add-the-device-extension)
- [x] [4. Create an ONU template](#3-6-Create-an-ONU-template)
- [x] [5. Add the ONU template and the HW layout to the ONU type](#3-7-Add-the-ONU-template-and-the-HW-layout-to-the-ONU-type)
- [x] [6. Create Device-fx intent](#4-2-Create-Device-fx-intent)
- [x] [7. Create Device-config-fx intent](#4-3-Create-Device-config-fx-intent)
- [x] [8. Create Uplink-connection intent](#4-4-Create-Uplink-connection-intent)
- [x] [9. Create fiber intent](#4-5-Create-fiber-intent)
- [x] [10. Create ONT intent](#4-6-Create-ONT-intent-in-VNO)
- [x] [11. Create L2-infra intent in VNO](#4-7-Create-L2-infra-intent-in-VNO)
- [x] [12. Create L2-User intent in VNO](#4-8-Create-L2-User-intent-in-VNO)
:::
**<center>Table of Content</center>**
[TOC]

## 1. Introduction
### 1-1. System Architecture 
![Picture1](https://hackmd.io/_uploads/S1QEwGu-A.png)


Altiplano's functionality extends beyond just these. Here, I have introduced the features I use:

- **System Administration :**
    - The main functionalities include tasks such as adding certificates, connecting to the Access Controller and Access Virtualizer, and configuring extensions to initialize your Altiplano.
- **Profile Manager :**
    - Profiles are a set of attributes that certain intent types use to configure the network. The Profile Manager application makes it easy to manage the profiles and their association with the relevant intent types.
    - Simply put, it's configuring the properties of intents, and later, I'll introduce what intents are.
- **IBN Provisioning :**
    - The IBN provisioning application provides a view of the current state of intent against the current network state.
- **Device Manager :**
    - To monitor and manage devices, you can view the current status and detailed information of the devices, as well as some performance metrics.

### 1-2. What is Nokia Altiplano ?
Altiplano is a large-scale SDAN (Software-Defined Access Network) manager that can be installed on various devices for optimization. The advantage of Altiplano over other devices lies in its ability to pre-configure without the need to connect to the OLT in advance. When connected to the OLT, it synchronizes the previous configurations with the devices.
- **What is SDAN ?**
    - The OLT will replicate the database into the network management system within the SDAN, known as Altiplano. Based on this configuration, it will make certain settings to facilitate monitoring and optimization.

### 1-3. Altiplano Access Controller && Altiplano Access Virtualizer
![Screenshot 2024-04-18 at 10.32.27 PM](https://hackmd.io/_uploads/rJRZy20lR.png)

- **Altiplano Access Controller (AC)**
    - Access Controller is responsible for the automation and abstraction components.
- **Altiplano Access Virtualizer (AV)**
    - Virtualizer refers to the virtualized placement of the device's database.

![Screenshot 2024-04-18 at 10.53.23 PM](https://hackmd.io/_uploads/HJrlEnAxR.png)

The Access Controller performs an alignment process, comparing the settings of physical functions with those on Altiplano. If they are consistent, synchronization will occur, and the configurations will be applied to the devices.

**Reference :［SDAN TW RBC Sharing Sessionl Intent-customization-20230106._135938-Meeting Recording.mp4**

### 1-4. Intent Flow Chart 

![Screenshot 2024-04-12 at 2.31.10 PM](https://hackmd.io/_uploads/BJa8rULlA.png)
 
**Intent Concept :**
- The configuration is divided into different blocks, each with its own specific intention or objective. These blocks are referred to as "intents".When the device connects to Altiplano, each intent can then transmit its parameters to the device.

**Reference : ProfileManager-Intent (Provided by Nokia)**
**Reference2 :［SDAN TW RBC Sharing Sessionl Intent-customization-20230106._135938-Meeting Recording.mp4**

### 1-5. Current project planning with NYCU
#### 1-5-1. System Architecture

![Screenshot 2024-04-18 at 9.51.29 PM](https://hackmd.io/_uploads/H1oIHTCx0.png)

**Reference : System Architecture provided by NYCU**

### 1-6. Job allocation

![scdwse](https://hackmd.io/_uploads/HyWLEA0-C.png)

- NTUST is responsible for connecting the equipment and setting up the environment.
- NYCU is responsible for connecting SMO for control and access.
## 2. System Administration Application
### 2-1. Add the license to Nokia Altiplano
When you complete the installation. The Altiplano is in empty.
Use System Administration to add license.

![Screenshot 2024-04-02 at 10.01.01 PM](https://hackmd.io/_uploads/ByV315KkR.png)

Check your license is active or not.

![Screenshot 2024-04-02 at 10.01.57 PM](https://hackmd.io/_uploads/B18ke5K1R.png)

### 2-2. Connect inp-altiplano-ac to inp-altiplano-av
Adding device managers to Access Controller Manager Directory

When you login to the Access Controller WebUI for the first time after the installation, you will only see the default internal "LocalManager" in this view.

![Screenshot 2024-03-21 at 3.21.11 PM](https://hackmd.io/_uploads/r1GvewY06.png)

You can add a new device manager to your manager directory using the "Add a new device manager" icon.

![Screenshot 2024-03-21 at 3.28.00 PM](https://hackmd.io/_uploads/BkV5-vKCT.png)

For example :

![Screenshot 2024-03-21 at 4.06.27 PM](https://hackmd.io/_uploads/SkpC5Dt0T.png)

Check you are successfuly connect the inp-altiplano-av or not.

![Screenshot 2024-03-21 at 4.09.51 PM](https://hackmd.io/_uploads/HJgwowKC6.png)

### 2-3. Add the device extension

Using following sftp server to obtain the file.

- Address : sftp -P 222 CHTTLSDAN@125.229.106.74
- Password : xxxxxxxxx

![Screenshot 2024-04-02 at 10.19.58 PM](https://hackmd.io/_uploads/r1Mm45YyA.png)

**Extession file :**
- device-extension-ls-fx-fant-h-fx4-23.3-106.zip
- device-extension-ls-fx-ihub-fant-h-fx4-23.3-106.zip
- device-extension-ls-fx-fwlt-c-23.3-106.zip
- downloaded-ls-fx-slice-owner-23.3-23.3.0-REL_101.zip
- downloaded-ls-fx-slice-manager-23.3-23.3.0-REL_101.zip

Get all the file.
```javascript=
cd csp_cht_sdan
cd FN Network Slicing

get device-extension-ls-fx-fant-h-fx4-23.3-106.zip
get device-extension-ls-fx-ihub-fant-h-fx4-23.3-106.zip
get device-extension-ls-fx-fwlt-c-23.3-106.zip
get downloaded-ls-fx-slice-owner-23.3-23.3.0-REL_101.zip
get downloaded-ls-fx-slice-manager-23.3-23.3.0-REL_101.zip
```

Switch to the extension and add files.

![Screenshot 2024-04-02 at 10.48.56 PM](https://hackmd.io/_uploads/HkgujqK1C.png)

**Result :**

![Screenshot 2024-04-02 at 10.52.02 PM](https://hackmd.io/_uploads/SyXii5FkC.png)
## 3. Profile Manager Application
### 3-1. Create a Classifier for ingress direction on the bridge port

![Screenshot 2024-04-15 at 10.16.07 PM](https://hackmd.io/_uploads/HJzeDhcg0.png)
![Screenshot 2024-04-15 at 10.18.33 PM](https://hackmd.io/_uploads/B1GLP29gC.png)

### 3-2. Create a Policy & add the Classifier that has been made to the policy

![Screenshot 2024-04-15 at 10.20.29 PM](https://hackmd.io/_uploads/rykkd29lA.png)

### 3-3. Add the Pbit-to-TC policy into a QoS policy profile

![Screenshot 2024-08-29 at 1.21.40 AM](https://hackmd.io/_uploads/Hk6y6AhiR.png)


### 3-4. Create a TC2Q mapping profile

![Screenshot 2024-04-15 at 10.32.58 PM](https://hackmd.io/_uploads/B1whc2cx0.png)


### 3-5. Create a TCONT profile

![Screenshot 2024-04-15 at 10.34.15 PM](https://hackmd.io/_uploads/Byjbi3ceA.png)


### 3-6. Create an ONU template

#### 3-6-1. Add a subscriber ingress QoS profiles
![Screenshot 2024-04-15 at 10.51.18 PM](https://hackmd.io/_uploads/HJNb1p9g0.png)

- ONT HW Layout
    - twoCardFourEthportAndOne10G

Detail configure setting

![Screenshot 2024-04-15 at 11.19.11 PM](https://hackmd.io/_uploads/ry-aSa9gR.png)

#### 3-6-2. Add bridge port vlan rules

![Screenshot 2024-04-15 at 11.23.31 PM](https://hackmd.io/_uploads/rJJ9I6qlR.png)

### 3-7. Add the ONU template and the HW layout to the ONU type

For this part, because our ONT device is U-050X-A, so we choose this type to add the template.

![Screenshot 2024-04-15 at 11.27.58 PM](https://hackmd.io/_uploads/H1i5P69lA.png)

- Type 
    - U-050X-A
- Layout
    - twoCardFourEthportAndOne10G launch
- Template
    - HSI-ont-sharing-four1G

### 3-8. Create a Traffic Descriptor Profile

![Screenshot 2024-08-28 at 11.33.59 PM](https://hackmd.io/_uploads/rJ5qQThoR.png)

### 3-9. Create a Speed Profile

![Screenshot 2024-08-28 at 11.39.58 PM](https://hackmd.io/_uploads/HkVgHpnj0.png)

### 3-10. Create a Policy Profile for the egress direction on the bridge port

![Screenshot 2024-08-28 at 11.57.42 PM](https://hackmd.io/_uploads/BkozKahs0.png)

### 3-11. Create a QoS Policy Profile for the egress direction on the bridge port

![Screenshot 2024-08-29 at 12.05.31 AM](https://hackmd.io/_uploads/SJi1i6hsC.png)

### 3-12. Create a User Service Profile on the LS-FX level

![Screenshot 2024-08-29 at 12.16.11 AM](https://hackmd.io/_uploads/S1RD6phsA.png)

### 3-13. Create a User Service Profile on the LS-ONT level

![Screenshot 2024-08-29 at 12.24.56 AM](https://hackmd.io/_uploads/ByOukR2sC.png)


## 4. IBN Provisioning Application
### 4-1. Import intent type
When you complete install, the IBN Provisioning Application wll be empty.You need to add intent type first.

Download the file.
```javascript=
cd csp_cht_sdan
get downloaded-ls-fx-23.3-23.3.0-REL_101.zip
```
and then upload the file to your intent type.
- **intent-device-config-fx-23.3.0-REL_101.zip**
- **intent-device-fx-23.3.0-REL_101.zip**

![Screenshot 2024-04-03 at 1.39.09 AM](https://hackmd.io/_uploads/H1Wu86F1R.png)

### 4-2. Create Device-fx intent
![Screenshot 2024-04-12 at 2.39.11 PM](https://hackmd.io/_uploads/S1fNDLIxC.png)

**Reference : ProfileManager-Intent P.66 (Provided by Nokia)**

Use the following parmeter to setup.

![Screenshot 2024-04-03 at 2.04.28 AM](https://hackmd.io/_uploads/BkcGt6KyA.png)
![Screenshot 2024-04-03 at 2.06.58 AM](https://hackmd.io/_uploads/BJ7UtTKkA.png)

Select the Boards to add the slot.

![Screenshot 2024-04-03 at 2.18.52 AM](https://hackmd.io/_uploads/r172naFk0.png)

If you complete to setup the ntust-fx intent, the ntust-fx.IHUB will setup automaticaly.

Use inp-altiplano-av device manager to see the result.

https://192.168.8.40/inp-altiplano-av/nav-device-manager/#/network/network

**Result :**

![Screenshot 2024-04-03 at 2.22.39 AM](https://hackmd.io/_uploads/SkS-6aFk0.png)

### 4-3. Create Device-config-fx intent

![Screenshot 2024-04-12 at 4.18.36 PM](https://hackmd.io/_uploads/BybORwLlR.png)

**Reference : ProfileManager-Intent P.71 (Provided by Nokia)**

Use the following parmeter to setup.

![Screenshot 2024-04-12 at 4.21.12 PM](https://hackmd.io/_uploads/rk1XJ_LxA.png)


### 4-4. Create Uplink-connection intent

![Screenshot 2024-04-12 at 4.22.55 PM](https://hackmd.io/_uploads/HyFKyOLxR.png)

**Reference : ProfileManager-Intent P.75 (Provided by Nokia)**

Use the following parmeter to setup.

![Screenshot 2024-04-12 at 4.25.20 PM](https://hackmd.io/_uploads/ByLbgOLxR.png)


### 4-5. Create fiber intent

![Screenshot 2024-04-12 at 4.26.36 PM](https://hackmd.io/_uploads/SJH_euUeC.png)

**Reference : ProfileManager-Intent P.78 (Provided by Nokia)**

Use the following parmeter to setup.

![Screenshot 2024-04-12 at 4.28.40 PM](https://hackmd.io/_uploads/SyF6xdIl0.png)


### 4-6. Create ONT intent in VNO

![Screenshot 2024-04-12 at 4.29.58 PM](https://hackmd.io/_uploads/ByDz-dLxA.png)

**Reference : ProfileManager-Intent P.81 (Provided by Nokia)**

:::spoiler Error : No replicated intent found for intent-type fiber with target LTIPON1
:::danger
**ERROR : No replicated intent found for intent-type fiber with target LTIPON1**

![Screenshot 2024-04-11 at 6.33.16 PM](https://hackmd.io/_uploads/SyfRZ_LgA.png)

![Screenshot 2024-04-11 at 6.33.24 PM](https://hackmd.io/_uploads/r1kyfuIgA.png)


When I create this ONT intent, I faced this problem.
Next step : Send Email to ask Nokia Engineer how to deal with this problem.


**Debug :**
I found that ONU profile template has no associations as the below.

![Screenshot 2024-04-15 at 10.51.18 PM](https://hackmd.io/_uploads/HJNb1p9g0.png)

So I add this associations to the "device-config-fx"

![Screenshot 2024-04-23 at 7.51.50 PM](https://hackmd.io/_uploads/Sy8EWmSbC.png)

![Screenshot 2024-04-23 at 7.54.23 PM](https://hackmd.io/_uploads/BJMK-mB-0.png)
Result :

![Screenshot 2024-04-24 at 4.36.26 PM](https://hackmd.io/_uploads/SyONLSLZ0.png)

But when I create the ONT intent again, it still wrong **"No replicated intent found for intent-type fiber with target LTIPON1"**

Check the fiber can detect the ONT or not.

![Screenshot 2024-05-10 at 3.02.06 PM](https://hackmd.io/_uploads/ryp9UrjM0.png)

The figure show that the fiber detect the new ONT.

Revise the fiber **type mpm-gpon-xgs** to **xgs**.

![Screenshot 2024-05-10 at 3.06.18 PM](https://hackmd.io/_uploads/BJqOwBifC.png)

![Screenshot 2024-05-10 at 3.06.50 PM](https://hackmd.io/_uploads/rJQjDriMR.png)

Try to create the ONT intent again.

![Screenshot 2024-05-10 at 3.09.40 PM](https://hackmd.io/_uploads/SySHOrszA.png)

![Screenshot 2024-05-10 at 3.10.04 PM](https://hackmd.io/_uploads/rJJvdHsGR.png)

It still the same error occur.

**Solution**
:::info
Using VNO-altiplano-AC to create ONT Intent

![Screenshot 2024-08-05 at 3.25.41 PM](https://hackmd.io/_uploads/BypYCeCKA.png)

![Screenshot 2024-08-05 at 3.27.00 PM](https://hackmd.io/_uploads/BkPykWAYA.png)

:::

Use the following parmeter to setup.

![Screenshot 2024-08-05 at 3.27.00 PM](https://hackmd.io/_uploads/S1q4ybRFA.png)

![Screenshot 2024-08-05 at 3.35.24 PM](https://hackmd.io/_uploads/rJ3RgZRFC.png)

**U-050X-A**
Profiles --> ONT Type --> All --> U-050X-A --> v1
![Screenshot 2024-08-05 at 3.29.40 PM](https://hackmd.io/_uploads/Syb-xZCtA.png)

### 4-7. Create L2-infra intent in VNO

![Screenshot 2024-08-05 at 3.47.01 PM](https://hackmd.io/_uploads/Skut7bRF0.png)

![Screenshot 2024-08-05 at 5.48.09 PM](https://hackmd.io/_uploads/HJxxg70FC.png)


### 4-8. Create L2-User intent in VNO

![Screenshot 2024-08-05 at 3.47.36 PM](https://hackmd.io/_uploads/BJYjX-AYC.png)

![Screenshot 2024-08-05 at 5.48.46 PM](https://hackmd.io/_uploads/rJkGgQCtR.png)


## 5. Network Slicing setting
### 5-1. Slice-owner

![Screenshot 2024-08-08 at 2.25.19 AM](https://hackmd.io/_uploads/B1N73VW90.png)

Use the following parmeter to setup.

![Screenshot 2024-08-08 at 2.30.23 AM](https://hackmd.io/_uploads/HJhP6EZ5R.png)

Device Managers : inp-altiplano-av

### 5-2. device-fx-slice

![Screenshot 2024-08-08 at 2.37.03 AM](https://hackmd.io/_uploads/ByuB1BZ90.png)

Use the following parmeter to setup.

![Screenshot 2024-08-08 at 2.37.24 AM](https://hackmd.io/_uploads/ryeUJHWqC.png)

### 5-3. fiber-slice

![Screenshot 2024-08-08 at 2.40.48 AM](https://hackmd.io/_uploads/B18fxHZqA.png)

Use the following parmeter to setup.

![Screenshot 2024-08-08 at 2.41.21 AM](https://hackmd.io/_uploads/HyTrxHb5A.png)

### 5-4. uplink-connection-slice

![Screenshot 2024-08-08 at 2.45.12 AM](https://hackmd.io/_uploads/S1sClrZ9C.png)

Use the following parmeter to setup.

![Screenshot 2024-08-08 at 2.46.29 AM](https://hackmd.io/_uploads/BkTfZHZqA.png)

## 6. PM Utilities Application

Here I just list some important PM data, more detail you can check Nokia Altiplano User Guide P.385

Following types of PM data collections are supported by Access Controller:
- Bulk data collection - for network-wide large scale PM data collection. 
- IPFIX collection (for NETCONF based devices supporting IPFIX protocol)
- SDC collection (for SNMP based devices managed through AMS)
- Live data collection - for near real-time monitoring of performance counters
- NC Live collection (for NETCONF based devices)
- RAN Live Collection (for RAN based FastMile devices managed through EdenNet/SON)

The home page of the PM Utilities application is the Metrics Explorer view.

![Screenshot 2024-08-14 at 3.54.15 PM](https://hackmd.io/_uploads/Bk_2MJccC.png)

You must specify the device name, object type, object ID and select the required PM counters to view the metrics data in a graph.

### 6-1. Model Manager
Model Manager is used to import and view the various models. Model Manager consists of the following items 
- Bulk Collection Model
- Live Collection Model
- TCA Model
- KPI
- SOAP ADAPTER.


Live Collection Model will show NETCONF and SNMP Live collection models.

![Screenshot 2024-08-14 at 4.05.58 PM](https://hackmd.io/_uploads/rJNKSyq50.png)

### 6-2. PM Explorer
Once the PM data is collected, EXPLORER tab can be used to view and analyze your PM data. You can choose the required device, object type, required objects and counters to view the PM metrics.

![Screenshot 2024-08-14 at 10.28.26 PM](https://hackmd.io/_uploads/rklX1BqqA.png)

## 7. Network Views Application

**Architecture**
![Screenshot 2024-08-14 at 11.52.26 PM](https://hackmd.io/_uploads/S1ClQIqcA.png)

We can use Network Views Application to see all of the system connection and also can view some KPI metrics.

**For ChannelGroup:Fiber1**
![Screenshot 2024-08-14 at 11.51.13 PM](https://hackmd.io/_uploads/ryQeNIqcA.png)

**For ONT optics statistics**

![Screenshot 2024-08-15 at 12.00.22 AM](https://hackmd.io/_uploads/rJx2N8q5C.png)

**For ONT statistics**

![Screenshot 2024-08-15 at 12.06.06 AM](https://hackmd.io/_uploads/H1JZLIc9R.png)

## 8. Testing Bandwidth Control with Nokia Altiplano
### 8-1. System Architecture

![Screenshot 2024-08-28 at 9.50.34 PM](https://hackmd.io/_uploads/S1F8si3oA.png)


### 8-2. Connect terminal device to ONT

**ONT UNI : 4 LAN and 1 10GE**

![Screenshot 2024-08-21 at 10.31.06 PM](https://hackmd.io/_uploads/HJi49u7jC.png)

**ONT vANI Traffic**

![Screenshot 2024-08-22 at 4.40.04 AM](https://hackmd.io/_uploads/rJaTgRmiA.png)


**LAN 1 Port Traffic Statistics**
![Screenshot 2024-08-22 at 12.04.10 AM](https://hackmd.io/_uploads/SkFxcTXiC.png)

:::danger
In this image, you can see that the LAN 1 port is already connected to a terminal device, but the LAN 2 port does not yet have any VLAN interface configured.

**ONT pot : LAN 1**
![Screenshot 2024-08-22 at 3.25.29 AM](https://hackmd.io/_uploads/H10YJTms0.png)

**ONT pot : LAN 2**
![Screenshot 2024-08-22 at 3.27.52 AM](https://hackmd.io/_uploads/rJna1pXjC.png)

**LAN Port structure**
![Screenshot 2024-08-22 at 3.30.13 AM](https://hackmd.io/_uploads/HkYIeamoR.png)

**Soluation (Failure)**
Turn to Altiplano NC Browser function and select the interface page in inp-av.

![Screenshot 2024-08-28 at 2.47.46 AM](https://hackmd.io/_uploads/rJFhkioi0.png)

This page can see the hole interface in this device.

Add the new interface.
![Screenshot 2024-08-28 at 2.54.24 AM](https://hackmd.io/_uploads/BJtgWsjsC.png)

The create is successful, but no interface show under the LAN2.

![Screenshot 2024-08-28 at 2.57.24 AM](https://hackmd.io/_uploads/HJfMfojoC.png)

**Solution 2**
Try to add the ONU template and reconfigure the ONT and L2 User.

More detail can follow this part [click here](#3-Profile-Manager-Application)

![Screenshot 2024-08-29 at 2.15.37 AM](https://hackmd.io/_uploads/SJC0YkTsA.png)

After adding the ONU template profile, try to update the ONT configuration.

![Screenshot 2024-08-29 at 2.21.56 AM](https://hackmd.io/_uploads/SJDA51TiR.png)

But the issue occur is as below.

![Screenshot 2024-08-29 at 1.44.00 AM](https://hackmd.io/_uploads/SyUXsyaj0.png)

Check the ONT instent code to find where is the problem occur.

![Screenshot 2024-09-03 at 10.41.39 PM](https://hackmd.io/_uploads/ByNHxiVnR.png)

![rtgrtgrtg](https://hackmd.io/_uploads/B1X-gjV3R.png)

**troubleshooting guide**

![image](https://hackmd.io/_uploads/SyMBbVCh0.png)
:::
### 8-3. Testing terminal device traffic
:::danger
When LAN 1 is connected but the PC is not connected to the network, with the IP configuration mode set to DHCP, it's unclear whether the issue is with the MAC's Ethernet connection itself or with the cable. Further troubleshooting is needed before conducting traffic testing.

![Screenshot 2024-08-22 at 2.27.44 AM](https://hackmd.io/_uploads/SJXffT7s0.png)

**What is Self-assigned IP?**

Means that when your Mac can’t obtain an IP address from your network router or DHCP server, it assigns itself an IP address. This address typically falls in the range of 169.254.x.x and is often referred to as a “self-assigned” or “APIPA” address.

**LAN 1 Statistics DHCPV4**
![Screenshot 2024-08-22 at 4.29.36 AM](https://hackmd.io/_uploads/SJ2cA6Xj0.png)

**Testing without using cable to connect device result** 

![Screenshot 2024-08-22 at 4.52.37 AM](https://hackmd.io/_uploads/r1ZwV0mo0.png)

:::
## Reference
- Nokia Altiplano Access Controller and FastMile Controller Release 23.3 User Guide
- ProfileManager-Intent (Provided by Nokia)
- SDAN TW RBC Sharing Sessionl Intent-customization-20230106._135938-Meeting Recording.mp4