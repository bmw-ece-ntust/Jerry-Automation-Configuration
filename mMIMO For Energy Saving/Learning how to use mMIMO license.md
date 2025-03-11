# <center>VIAVI RicTest</center>

<center>Learning how to use mMIMO license</center>

###### tags: `mMIMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue2024 8:39 PM][color=#38c16a]

:::info
**Introduction :**

- Introduce mMIMO license function in VIAVI RicTest.

**Goal：**

- [x] Learning how to use mMIMO license in VIAVI RicTest.
:::
**<center>Table of Content</center>**
[TOC]

## 1. Introduction

Through this license, it is possible to simulate the mMIMO environment and analyze its energy consumption, cell traffic, and various indicators. By using antenna masks and related settings, energy saving can be achieved.

## 2. Advanced RF Models Tab
Select the cell configuration and switch to the advanced RF models tab to set the mMIMO Beam Group in Antenna model.

![Screenshot 2024-06-19 at 6.31.57 PM](https://hackmd.io/_uploads/ryB3mEg8C.png)

You can use following setting to configure your beam parameter. 

![Screenshot 2024-06-19 at 6.38.24 PM](https://hackmd.io/_uploads/S1PLBVgIR.png)

For **Beamforming method**, we have three types:
- Sector Beam/Analog
- mMIMO 3GPP/Analog
- mMIMO 3GPP/Digital

### 2-1. Sector Beam/Analog

For Auto configuration, each beam's beamwidth and beam direction are automatically calculated based on the number of horizontal/vertical beams.

![Screenshot 2024-06-20 at 4.09.24 AM](https://hackmd.io/_uploads/Bymfjhe8R.png)

The beam configuration can be switched from Auto to Manual.

For Manual configuration - the beamwidth and beam direction of each beam are manually

![Screenshot 2024-06-20 at 4.21.59 AM](https://hackmd.io/_uploads/Bk0g0hl8C.png)

### 2-2. mMIMO 3GPP/Analog

![Screenshot 2024-06-20 at 4.32.03 AM](https://hackmd.io/_uploads/SkPwlaxUA.png)

Auto - In auto mode, the beam width and beam direction are automatically calculated based on the number of horizontal/vertical beams.

![Screenshot 2024-06-20 at 4.35.05 AM](https://hackmd.io/_uploads/SkoZbTxIA.png)

Manual - Each beam can be manually configured.

![Screenshot 2024-06-20 at 4.36.19 AM](https://hackmd.io/_uploads/Hk1DZax80.png)

### 2-3. mMIMO 3GPP/Digital

![Screenshot 2024-06-20 at 4.37.36 AM](https://hackmd.io/_uploads/BybiWTg8R.png)

Auto - The beam width and beam direction are automatically calculated based on the number of horizontal/vertical beams. The settings of the antenna panel apply to all beams:

- Number of horizontal/vertical beams - the number of horizontal/vertical beams
- **Antenna mask(0x)** - ==the mask of enabled radiators in hexadecimal. For example, if the antenna mask in hex is 0x3333, the mask in a binary number is {0 0 1 1 0 0 1 1 0 0 1 1 0 0 1 1}, which means the left half of the radiators are active and the right half is off.==

![Screenshot 2024-06-20 at 4.42.28 AM](https://hackmd.io/_uploads/r1UpMTxUR.png)

Manual: Each beam can be manually configured with the following parameters:

![Screenshot 2024-06-20 at 4.44.22 AM](https://hackmd.io/_uploads/r1DEmagIC.png)

## 3. Simulate the mMIMO environment. 
### 3-1. mMIMO environment setting.
Scenario Generation

![Screenshot 2024-07-01 at 2.08.54 PM](https://hackmd.io/_uploads/H1YHdakwC.png)

Cells Configuration

![Screenshot 2024-07-01 at 2.10.38 PM](https://hackmd.io/_uploads/HkMidTJD0.png)

mMIMO-Urban

![Screenshot 2024-07-01 at 2.12.23 PM](https://hackmd.io/_uploads/SJ71KpJw0.png)
![Screenshot 2024-07-01 at 2.12.30 PM](https://hackmd.io/_uploads/SJwktTkPC.png)

![Screenshot 2024-07-15 at 2.29.09 PM](https://hackmd.io/_uploads/rJ7rzHMdC.png)

Indoor Ue Setting

![Screenshot 2024-07-12 at 5.42.40 PM](https://hackmd.io/_uploads/SJGZjd0DC.png)

Pedestrian UE Setting 

![Screenshot 2024-07-12 at 5.45.08 PM](https://hackmd.io/_uploads/ryOSsdRwR.png)


### 3-2. Simulate the environment

- Site 1
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s1
    - Cells layout :
        - C1 : 
            - Advanced RF model : mMIMO-Urban-SSB
            - Azimuth (degrees) : 240
    - Indoor UE : 1 for C1
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB/CSI-RS
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 1
        - Number of vertical beams : 1
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - ==Antenna mask (0x) : FFFF==
- Site 2
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s2
    - Cells layout :
        - C1 : 
            - Advanced RF model : mMIMO-Urban-SSB
            - Azimuth (degrees) : 240
    - Indoor UE : 1 for C1
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB/CSI-RS
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 1
        - Number of vertical beams : 1
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - ==Antenna mask (0x) : FFFF==

**Simulation Result**

![Screenshot 2024-07-28 at 5.11.35 AM](https://hackmd.io/_uploads/By1ozk7t0.png)

**mMIMO reports**

![Screenshot 2024-07-28 at 5.44.31 AM](https://hackmd.io/_uploads/Sk7wcJXFA.png)


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

## 4. Using RESTful API to control the antenna mask.

We can sutdown some antenna array to achieve energy saving by config this parameter.

```javascript=
"antennaMask":"FFFF"
```

Using this command to get all of the config.
```javascript=
curl -v GET http://192.168.8.28:32441/O1/CM/ManagedElement
```
![Screenshot 2024-07-04 at 9.29.37 AM](https://hackmd.io/_uploads/SkOMjOQvR.png)

Get the GnbDuFunction=1,NrCellDu=1 config

```javascript=
curl -v http://192.168.8.28:32441/O1/CM/ManagedElement=1193046,GnbDuFunction=1,NrCellDu=1
```

![Screenshot 2024-07-04 at 6.04.09 PM](https://hackmd.io/_uploads/SJbCmxEDR.png)

Revise the GnbDuFunction=1,NrCellDu=1 antenna mask config "antennaMask": "FFFF" to "antennaMask": "1111".

**mMIMO-config.json file**
```javascript=
{
  "id": "1",
  "objectInstance": "ManagedElement=1193046,GnbDuFunction=1,NrCellDu=1",
  "attributes": {
    "cellLocalId": 1,
    "nrPci": 1,
    "plmnInfoList": [
      {
        "plmnId": {
          "mcc": "001",
          "mnc": "01"
        }
      }
    ],
    "ssbFrequency": 3600,
    "arfcnDL": 640000,
    "arfcnUL": 640000,
    "administrativeState": "UNLOCKED"
  },
  "viavi-attributes": {
    "cellSize": "medium",
    "cellName": "S1/N77/C1",
    "siteName": "S1",
    "latitude": 0.0,
    "longitude": 0.0,
    "advancedRfModel": {
      "name": "mMIMO-Urban-SSB",
      "scenario": "UMa-Buildings",
      "antennaModel": {
        "type": "mMIMO Beam Group",
        "models": [
          {
            "gain": 0,
            "beamType": "SSB",
            "groupId": 1,
            "method": {
              "type": "3GPP-Digital",
              "beamConf": {
                "nHorBeams": 1,
                "nVerBeams": 1,
                "nRows": 4,
                "nCols": 4,
                "hSpacing": 1,
                "vSpacing": 1,
                "type": "Auto",
                "antennaMask": "1111",
                "numActiveAnt": 16
              },
              "beamforming": "Digital"
            },
            "azimuth": 240,
            "tilt": 5
          }
        ],
        "height": 25,
        "azimuth": 240,
        "tilt": 5
      },
      "shadowingEnabled": true,
      "beams": {
        "SSB": [
          {
            "type": "3GPP",
            "beamforming": "Digital",
            "gain": 0,
            "beamType": "SSB",
            "groupId": 1,
            "azimuth": 240,
            "tilt": 5,
            "nRows": 4,
            "nCols": 4,
            "hSpacing": 1,
            "vSpacing": 1,
            "antennaMask": "1111",
            "id": 1,
            "txPercent": 6.25,
            "digitalAzimuth": 0.0,
            "digitalTilt": 0.0
          }
        ]
      }
    }
  },
  "CPCIConfigurationFunction": {
    "id": "1",
    "objectInstance": "ManagedElement=1193046,GnbDuFunction=1,NrCellDu=1,CPCIConfigurationFunction=1",
    "attributes": {
      "cSonPciList": {
        "NRPci": 1
      }
    }
  }
}
```
Send the config to the rictest.

```javascript=
curl -i -X PUT   -H "Content-Type: application/json"   -d @mMIMO-config.json   http://192.168.8.28:32441/O1/CM/ManagedElement=1193046,GnbDuFunction=1,NrCellDu=1
```

Check the config is already revise or not.

```javascript=
curl -v http://192.168.8.28:32441/O1/CM/ManagedElement=1193046,GnbDuFunction=1,NrCellDu=1
```

![Screenshot 2024-07-04 at 6.15.21 PM](https://hackmd.io/_uploads/r1t88eVwR.png)

```javascript=
{
  "id": "1",
  "objectInstance": "ManagedElement=1193046,GnbDuFunction=1,NrCellDu=1",
  "attributes": {
    "cellLocalId": 1,
    "nrPci": 1,
    "plmnInfoList": [
      {
        "plmnId": {
          "mcc": "001",
          "mnc": "01"
        }
      }
    ],
    "ssbFrequency": 3600,
    "arfcnDL": 640000,
    "arfcnUL": 640000,
    "administrativeState": "UNLOCKED"
  },
  "viavi-attributes": {
    "cellSize": "medium",
    "cellName": "S1/N77/C1",
    "siteName": "S1",
    "latitude": 0.0,
    "longitude": 0.0,
    "advancedRfModel": {
      "name": "mMIMO-Urban-SSB",
      "scenario": "UMa-Buildings",
      "antennaModel": {
        "type": "mMIMO Beam Group",
        "models": [
          {
            "gain": 0,
            "beamType": "SSB",
            "groupId": 1,
            "method": {
              "type": "3GPP-Digital",
              "beamConf": {
                "nHorBeams": 1,
                "nVerBeams": 1,
                "nRows": 4,
                "nCols": 4,
                "hSpacing": 1,
                "vSpacing": 1,
                "type": "Auto",
                "antennaMask": "1111",
                "numActiveAnt": 16
              },
              "beamforming": "Digital"
            },
            "azimuth": 240,
            "tilt": 5
          }
        ],
        "height": 25,
        "azimuth": 240,
        "tilt": 5
      },
      "shadowingEnabled": true,
      "beams": {
        "SSB": [
          {
            "type": "3GPP",
            "beamforming": "Digital",
            "gain": 0,
            "beamType": "SSB",
            "groupId": 1,
            "azimuth": 240,
            "tilt": 5,
            "nRows": 4,
            "nCols": 4,
            "hSpacing": 1,
            "vSpacing": 1,
            "antennaMask": "1111",
            "id": 1,
            "txPercent": 6.25,
            "digitalAzimuth": 0.0,
            "digitalTilt": 0.0
          }
        ]
      }
    }
  },
  "CPCIConfigurationFunction": {
    "id": "1",
    "objectInstance": "ManagedElement=1193046,GnbDuFunction=1,NrCellDu=1,CPCIConfigurationFunction=1",
    "attributes": {
      "cSonPciList": {
        "NRPci": 1
      }
    }
  }
}
```
:::danger
The config is already revise, but the result is the same when I check the simulation in VIAVI Rictest.

![Screenshot 2024-07-28 at 5.40.03 AM](https://hackmd.io/_uploads/HySBYJmtR.png)

**Expected Result**
![Screenshot 2024-07-28 at 5.43.33 AM](https://hackmd.io/_uploads/BJ3Gc17F0.png)

**Measure the Rictest O1 Interface Testing**

![Screenshot 2024-07-30 at 4.01.56 PM](https://hackmd.io/_uploads/SyXSAG8YR.png)


:::
## 5. Using O1 Netconf to control the antenna mask.
### 5-1. Connect to Netconf Server

Reference : https://hackmd.io/@Winnie27/r1BajOitT

```javascript=
python3
from ncclient import manager
smo = manager.connect(host='192.168.8.28', port="31948", timeout=5, username="root", password="viavi", hostkey_verify=False)
```

### 5-2. Get the environment config

```javascript=
smo.get_config(source="running")
```

![Screenshot 2024-07-29 at 2.28.52 PM](https://hackmd.io/_uploads/ryvn82NYR.png)

**Configuration Filter -1**

```javascript=
gconf = '''
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
  <id>1193046</id>
  <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
    <id>1</id>
    <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
      <id>1</id>
      <CommonBeamformingFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-commonbeamformingfunction">
        <id>1</id>
      </CommonBeamformingFunction>
    </NRSectorCarrier>
  </GNBDUFunction>
</ManagedElement>
'''
```
```javascript=
result = smo.get_config(source="running", filter=("subtree", gconf))
print(result)
```
**Result**

![Screenshot 2024-07-29 at 5.48.44 PM](https://hackmd.io/_uploads/r1TFBJBFA.png)

**Configuration Filter -2**

```javascript=
gconf = '''
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
  <id>1193046</id>
  <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
    <id>1</id>
    <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
      <id>1</id>
    </NRSectorCarrier>
  </GNBDUFunction>
</ManagedElement>
'''
```
```javascript=
result = smo.get_config(source="running", filter=("subtree", gconf))
print(result)
```
**Result**

![Screenshot 2024-07-30 at 4.25.34 PM](https://hackmd.io/_uploads/B1Wc7mIK0.png)


### 5-3. Revise the config to Netconf server

```javascript=
conf= '''
<config xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
  <id>1193046</id>
  <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
    <id>1</id>
    <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
      <id>1</id>
      <CommonBeamformingFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-commonbeamformingfunction">
        <id>1</id>
        <attributes>
          <digitalAzimuth>-1800</digitalAzimuth>
          <digitalTilt>50</digitalTilt>
        </attributes>
      </CommonBeamformingFunction>
    </NRSectorCarrier>
  </GNBDUFunction>
</ManagedElement>
</config>
'''
```
```javascript=
smo.edit_config (target = "running", config=conf)
```

:::danger
**ERROR**

![Screenshot 2024-07-29 at 5.54.44 PM](https://hackmd.io/_uploads/BJv-PkSYA.png)
:::

**Revise MaxTxPower 37 to 10 to Netconf**

```javascript=
conf = '''
<config xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
  <id>1193046</id>
  <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
    <id>1</id>
    <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
      <id>1</id>
      <attributes>
         <configuredMaxTxPower>10</configuredMaxTxPower>
      </attributes>
    </NRSectorCarrier>
  </GNBDUFunction>
</ManagedElement>
</config>
'''
```
```javascript=
smo.edit_config (target = "running", config=conf)
```
:::info
**SUCCESS**

![Screenshot 2024-07-30 at 4.29.14 PM](https://hackmd.io/_uploads/SkBtEXUt0.png)

![Screenshot 2024-07-30 at 4.30.33 PM](https://hackmd.io/_uploads/SynnEQIFA.png)

:::

## 6. Reference 
- TeraVM RIC Test - User Guide v2.0
