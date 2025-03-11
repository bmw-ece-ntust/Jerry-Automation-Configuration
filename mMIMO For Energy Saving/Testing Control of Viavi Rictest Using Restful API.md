# <center>Testing Control of Viavi Rictest Using Restful API</center>

<center></center>

###### tags: `mMIMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed 2024/08/15 4:30 PM][color=#38c16a]


**<center>Table of Content</center>**
[TOC]

## 1. Viavi Rictest v1.9 testing
### 1-1. Simulation environment config
- Site 1
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s1
    - Cells layout :
        - C1 : 
            - Advanced RF model : mMIMO-Urban-SSB-1111
            - Azimuth (degrees) : 240
            - Tx Power (dBm) : 37
    - Indoor UE : 1 for C1
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB
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
            - Tx Power (dBm) : 37
    - Indoor UE : 1 for C1
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 1
        - Number of vertical beams : 1
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - ==Antenna mask (0x) : FFFF==

**Simulation Result**

![Screenshot 2024-08-15 at 4.25.29 PM](https://hackmd.io/_uploads/B1gf-24scA.png)


### 1-2. RESTful API testing procedure

We can sutdown some antenna array to achieve energy saving by config this parameter.

```javascript=
"antennaMask":"FFFF"
```

Using this command to get all of the config.
```javascript=
curl -v GET http://192.168.8.28:32441/O1/CM/ManagedElement
```
Get the GnbDuFunction=1,NrCellDu=1 config

```javascript=
curl -v http://192.168.8.28:32441/O1/CM/ManagedElement=1193046,GnbDuFunction=1,NrCellDu=1
```

![Screenshot 2024-08-15 at 4.26.31 PM](https://hackmd.io/_uploads/SyrkhNj5A.png)


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
      "name": "mMIMO-Urban-SSB-1111",
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
![Screenshot 2024-08-15 at 4.33.22 PM](https://hackmd.io/_uploads/ByedaNsqA.png)


Check the config is already revise or not.

```javascript=
curl -v http://192.168.8.28:32441/O1/CM/ManagedElement=1193046,GnbDuFunction=1,NrCellDu=1
```

![Screenshot 2024-08-15 at 4.34.29 PM](https://hackmd.io/_uploads/HkHiaEi9A.png)


### 1-3. RESTful API testing Result
:::danger
The config is already revise, but the result is the same when I check the simulation in VIAVI Rictest.

Simulation Result
![Screenshot 2024-08-15 at 4.35.25 PM](https://hackmd.io/_uploads/BJ6JC4o9R.png)

**Expected Result**
![Screenshot 2024-07-28 at 5.43.33 AM](https://hackmd.io/_uploads/BJ3Gc17F0.png)

**Measure the Rictest O1 Interface Testing**

O1 interface is already recieve the update request and the response is 200 Ok.
![Screenshot 2024-08-15 at 4.46.49 PM](https://hackmd.io/_uploads/HyHE-Hjc0.png)

![Screenshot 2024-08-15 at 4.48.12 PM](https://hackmd.io/_uploads/Sk9GWrs5A.png)
![Screenshot 2024-08-15 at 4.48.42 PM](https://hackmd.io/_uploads/rJqzbHjc0.png)

**Check run time test control data**

The PEE.AvgPower is still 42.75, it dosen't change.
![image](https://hackmd.io/_uploads/BJnK0Es5R.png)

**Check the Rictest pod log**

```javascript=
kubectl logs ric-test-deployment-6cd8bcfc7f-bnd9w -f
```

This figure show that the config is already update.
![Screenshot 2024-08-15 at 4.40.40 PM](https://hackmd.io/_uploads/rkY4kHicC.png)
:::

## 2. Viavi Rictest v2.1 testing
### 2-1. Using RESTful API to control the antenna mask.
#### 2-1-1. Simulation environment config

**This part is same with the v1.9 configuration**

- Site 1
    - Name : N77
    - Technology : NR-3600
    - Area and sites : s1
    - Cells layout :
        - C1 : 
            - Advanced RF model : mMIMO-Urban-SSB-1111
            - Azimuth (degrees) : 240
            - Tx Power (dBm) : 37
    - Indoor UE : 1 for C1
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB
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
            - Tx Power (dBm) : 37
    - Indoor UE : 1 for C1
    - Advanced RF models : mMIMO-Urban
        - Beam type : SSB
        - Beamforming method : mMIMO 3GPP/Digital
        - Number of horizontal beams : 1
        - Number of vertical beams : 1
        - Radiator rows : 4
        - Radiator columns : 4
        - Horizontal spacing (λ) : 1
        - Vertical spacing (λ) : 1
        - ==Antenna mask (0x) : FFFF==

**Simulation Result**

![Screenshot 2024-08-20 at 4.31.07 AM](https://hackmd.io/_uploads/Sy-jsmboR.png)

#### 2-1-2. RESTful API testing procedure

This part is same with the [#1-2. RESTful API testing procedure](#1-2-RESTful-API-testing-procedure)

#### 2-1-3. RESTful API testing Result

:::danger
The configuration has been changed, but the simulation results have not yet changed, encountering the same issue as in version 1.9.

**Expected result**

![Screenshot 2024-08-20 at 5.18.56 AM](https://hackmd.io/_uploads/SkKAINWiC.png)


**Measure the Rictest O1 Interface Testing**
O1 interface is already recieve the update request and the response is 200 Ok.

![Screenshot 2024-08-20 at 5.04.02 AM](https://hackmd.io/_uploads/HyrI74WoR.png)

![Screenshot 2024-08-20 at 5.05.51 AM](https://hackmd.io/_uploads/rJgCXVWsA.png)
![Screenshot 2024-08-20 at 5.06.41 AM](https://hackmd.io/_uploads/ryx-eVNbi0.png)

**Check run time test control data**

The PEE.AvgPower is still 42.75, it dosen't change.

![Screenshot 2024-08-20 at 5.07.53 AM](https://hackmd.io/_uploads/HysE4V-s0.png)

**Check the Rictest pod log**

```javascript=
kubectl logs ric-test-deployment-6cd8bcfc7f-bnd9w -f
```

This figure show that the config is already update.

![Screenshot 2024-08-20 at 5.12.15 AM](https://hackmd.io/_uploads/rJ1BrNZiC.png)

:::

### 2-2. Using RESTful API to control MaxTxPower
#### 2-2-1. Simulation environment config

This part is same with the [#2-1-Simulation-environment-config](#2-1-Simulation-environment-config)

#### 2-2-2. RESTful API testing procedure

Get S1/N77/C1 NrSectorCarrier config.

```javascript=
curl -v http://192.168.8.28:32441/O1/CM/ManagedElement=1193046,GnbDuFunction=1,NrSectorCarrier=1
```
![Screenshot 2024-08-20 at 5.31.36 PM](https://hackmd.io/_uploads/SyYFMJGoC.png)

Create the new config file to revise MaxTxPower : 37 to 10.

**uu.json**
```javascript=
{
  "id": "1",
  "objectInstance": "ManagedElement=1193046,GnbDuFunction=1,NrSectorCarrier=1",
  "attributes": {
    "configuredMaxTxPower": 10
  },
  "CommonBeamformingFunction": {
    "id": "1",
    "objectInstance": "ManagedElement=1193046,GnbDuFunction=1,NrSectorCarrier=1,CommonBeamformingFunction=1",
    "attributes": {
      "digitalAzimuth": 600,
      "digitalTilt": 50
    }
  }
}
```
Use this command yo update the config.
```javascript=
curl -i -X PUT   -H "Content-Type: application/json"   -d @uu.json   http://192.168.8.28:32441/O1/CM/ManagedElement=1193046,GnbDuFunction=1,NrSectorCarrier=1
```

![Screenshot 2024-08-20 at 5.35.22 PM](https://hackmd.io/_uploads/rJ3Dm1fsA.png)

#### 2-2-3. RESTful API testing Result

:::info
**Before update the the MaxTxPower, original is 37**

![Screenshot 2024-08-20 at 5.42.05 PM](https://hackmd.io/_uploads/B1gZH1Mi0.png)


**After update the the MaxTxPower, right now is 10**

![Screenshot 2024-08-20 at 5.41.26 PM](https://hackmd.io/_uploads/SkvbH1Ms0.png)


:::