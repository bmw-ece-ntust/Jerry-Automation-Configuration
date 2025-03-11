# Integration of the AIMLFW, Non-RT RIC and SMO
###### tags: `AI/ML`
![](https://i.imgur.com/JORnn3y.png =150x)@NTUST
>[name=Jerry][time=Thu, Sep 28, 2023 9:00 PM][color=#38c16a]
:::info
**Goal：**

Integrate SMO, Non-RT RIC and AI/ML

**Action Item：**
- [x] [Complete the instsllation of AIML FW.](#2-1-AIML-FW-and-Data-Lake-Complete)
- [x] [Complete the instsllation of RAN PM setup.](#2-2-RAN-PM-Complete)
- [x] [Interconnect AIML FW with Non-RT RIC.](#4-Use-Non-RT-RIC-DMEData-Management-amp-Exposure-as-data-source-to-training-and-prediction)
- [ ] Interconnect AIML FW with SMO.
:::
**<center>Table of Content</center>**
[TOC]

## 1. Project status

![](https://hackmd.io/_uploads/B1EGlVveT.png)

## 2. AIMLFW, RAN PM and Data Lake(Influx DB) installation completed. 

### 2-1. AIML FW and Data Lake Complete

![](https://hackmd.io/_uploads/BJiBpvwAh.png)

### 2-2. RAN PM Complete

![](https://hackmd.io/_uploads/HkxphhIgT.png)

## 3. Use influxDB as data source to training and prediction.

![](https://hackmd.io/_uploads/rkTClZXga.png)

### 3-1. QoE Data

![](https://hackmd.io/_uploads/B1FHqpQT3.png)

### 3-2. Training QoE model.

![](https://hackmd.io/_uploads/B1qgMu_C3.png)

### 3-3. Create sample data for prediction.

#### 3-3-1. Sample data
```javascript=
{"signature_name": "serving_default", "instances": [[[2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56]]]}
```
#### 3-3-2. Prediction

![](https://hackmd.io/_uploads/HyJW1o_03.png)

## 4. Use Non-RT RIC DME(Data Management & Exposure) as data source to training and prediction.

![](https://hackmd.io/_uploads/BJRUXTUeT.png)

The detail of inside.

![](https://hackmd.io/_uploads/rkWl69B-T.png)

### 4-1. QoE Data
Use this command to enter the Database.
```javascript=
kubectl exec -it influxdb2-0 -n nonrtric -- bash
```
Check the data.
```javascript=
influx query 'from(bucket: "pm-bucket") |> range(start: -1000000000000000000d)'
```

![](https://hackmd.io/_uploads/r1lMMTIeT.png)

### 4-2. Training QoE model.

![](https://hackmd.io/_uploads/HJCwtNhep.png)

### 4-3. Create sample data for prediction.

#### 4-3-1. Sample data
```javascript=
{"signature_name": "serving_default", "instances": [[[2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56],
       [2.56, 2.56]]]}
```
#### 4-3-2. Prediction

![](https://hackmd.io/_uploads/rJ5_RI2gp.jpg)

## 5. Interconnect AIML FW with SMO.

### 5-1. OSC Samsung System Architecture
#### 5-1-1. [OSC Samsung AI/ML FW Architecture](https://wiki.o-ran-sc.org/display/AIMLFEW/Design)

![](https://hackmd.io/_uploads/SkuSG_tW6.png)

#### 5-1-2. [OSC Samsung AI/ML FW Project Scope](https://wiki.o-ran-sc.org/display/AIMLFEW/Project+Scopes+and+modules)

![](https://hackmd.io/_uploads/H1SsIdtZp.png)

#### 5-1-3. AIML FW in the SMO ( O-RAN.WG11.SMO-Security-Analysis-TR.0-R003-v02.00)

![](https://hackmd.io/_uploads/r1IdEdFW6.png)

### 5-2. Combine the AI/ML FW with the SMO

#### 5-2-1. Method 1：Install the SMO in the AI/ML FW machine.

![](https://hackmd.io/_uploads/HJrMg_FWa.png)

Use this installation guide to install the SMO.

[ONAP - Based [H] SMO Deployment](https://hackmd.io/Cf5E97XqTXSr00xC7bzfeA?view)

When I install the SMO in the AI/ML, but system is crash like this figure.

![](https://hackmd.io/_uploads/rkVL0J9bT.jpg)

Restart the PC, some of the pod are crash.

![](https://hackmd.io/_uploads/rJvsl0KZa.jpg)
![](https://hackmd.io/_uploads/SJJhlRtb6.jpg)

However, OSC Sansung has not yet integrated the AIML and SMO components at this time.

![](https://hackmd.io/_uploads/B1mI1xcZT.jpg)
![](https://hackmd.io/_uploads/HJk2kxqWT.jpg)

##### 5-2-1-1. What's the issue occur?

Check the SMO CPU configuration and memory usage.
This is the SMO CPU configuration. CPU(s) : 8

![](https://hackmd.io/_uploads/BJRkD6hba.jpg)

And this is the total memory and memory usage.
Total : 31 Gi
Used : 23 Gi

![](https://hackmd.io/_uploads/ryr8wT2-6.png)

This is the memory usage when AI/ML was installed separately. However, we found that with 14GB plus 23GB, the host simply couldn’t handle it, which led to the crash situation.

![](https://hackmd.io/_uploads/BkQ_vT2bp.png)

It seems that this issue indeed confirms a lack of memory. To address this, **the solution might be to either upgrade the memory capacity or to install the two platforms separately** and then integrate them.

#### 5-2-2. Method 2：Separate the SMO and AI/ML and then to integrate them.

![](https://hackmd.io/_uploads/Hkhjja3Wa.png)

##### 5-2-2-1. login SMO server to take o1ves-influxdb2-0 information and create bucket.

![](https://hackmd.io/_uploads/SJ2uolTZa.png)

Get influxDB information.

```javascript=
kubectl exec -it o1ves-influxdb2-0 -n o1ves -- bash
```

Find **"influxd.bolt"** file and cat.

```javascript=
find / -type f -name "influxd.bolt"
```

![](https://hackmd.io/_uploads/SJNYs-T-6.png)

```javascript=
cat /var/lib/influxdb2/influxd.bolt
```

**<org_name>：** influxdata
**<API_Token>：** PYvZ2YLwKbgARSbfLHw5Wy0kZCnXG6Xl

Use this command to create **UEData** bucket.
```javascript=
influx bucket create -n UEData -o <org_name> -t <API_Token>
```
For example.
```javascript=
influx bucket create -n UEData -o influxdata -t PYvZ2YLwKbgARSbfLHw5Wy0kZCnXG6Xl
```

![](https://hackmd.io/_uploads/r1wlc4p-p.png)

Check if bucket is create or not.

```javascript=
influx bucket list --org influxdata --token PYvZ2YLwKbgARSbfLHw5Wy0kZCnXG6Xl
```

![](https://hackmd.io/_uploads/BJpS94TZ6.png)

##### 5-2-2-2. Insert simulate data in the UEData bucket.

Install the following dependencies.

```javascript=
exit
cd
sudo apt-get install python3-pip
sudo pip3 install pandas
sudo pip3 install influxdb_client
```
Use the insert.py in ric-app/qp repository to upload the qoe data in Influx DB.

```javascript=
git clone -b f-release https://gerrit.o-ran-sc.org/r/ric-app/qp
cd qp/qp
vim insert.py
```

Update insert.py file.

```javascript=
import pandas as pd
from influxdb_client import InfluxDBClient
from influxdb_client.client.write_api import SYNCHRONOUS
import datetime


class INSERTDATA:

   def __init__(self):
        self.client = InfluxDBClient(url = "http://localhost:8086", token="PYvZ2YLwKbgARSbfLHw5Wy0kZCnXG6Xl")


def explode(df):
     for col in df.columns:
             if isinstance(df.iloc[0][col], list):
                     df = df.explode(col)
             d = df[col].apply(pd.Series)
             df[d.columns] = d
             df = df.drop(col, axis=1)
     return df


def jsonToTable(df):
     df.index = range(len(df))
     cols = [col for col in df.columns if isinstance(df.iloc[0][col], dict) or isinstance(df.iloc[0][col], list)]
     if len(cols) == 0:
             return df
     for col in cols:
             d = explode(pd.DataFrame(df[col], columns=[col]))
             d = d.dropna(axis=1, how='all')
             df = pd.concat([df, d], axis=1)
             df = df.drop(col, axis=1).dropna()
     return jsonToTable(df)


def time(df):
     df.index = pd.date_range(start=datetime.datetime.now(), freq='10ms', periods=len(df))
     df['measTimeStampRf'] = df['measTimeStampRf'].apply(lambda x: str(x))
     return df


def populatedb():
     df = pd.read_json('cell.json.gz', lines=True)
     df = df[['cellMeasReport']].dropna()
     df = jsonToTable(df)
     df = time(df)
     db = INSERTDATA()
     write_api = db.client.write_api(write_options=SYNCHRONOUS)
     write_api.write(bucket="UEData",record=df, data_frame_measurement_name="liveCell",org="influxdata")

populatedb()
```

Open new terminal and follow below command to port forward to access Influx DB.

```javascript=
kubectl port-forward svc/o1ves-influxdb2 -n o1ves 8086:80
```

![](https://hackmd.io/_uploads/rkloQHzab6.png)

Back to the terminal to insert data.

```javascript=
python3 insert.py
```

Then the new terminal which you open few seconds ago will show this information like this figure.

![](https://hackmd.io/_uploads/SyrQUz6-T.png)

Inter the influxDB to check the data.

```javascript=
kubectl exec -it o1ves-influxdb2-0 -n o1ves -- bash
influx query  'from(bucket: "UEData") |> range(start: -1000d)' -o influxdata -t PYvZ2YLwKbgARSbfLHw5Wy0kZCnXG6Xl
```

**Result：**

![](https://hackmd.io/_uploads/SkMRUfa-T.png)

##### 5-2-2-3. Back to the AI/ML server to re-configurate the training host setting.

Revise the traininghost setting.

DataLake SMO Setting :
- traininghost:
  - ip_address: 192.168.8.44
- host : 192.168.8.229
- port : 30001
- orgname : influxdata
- bucket : UEData
- token : PYvZ2YLwKbgARSbfLHw5Wy0kZCnXG6Xl

```javascript=
cd aimlfw-dep/
vim RECIPE_EXAMPLE/example_recipe_latest_stable.yaml
```
```javascript=
traininghost:
  ip_address: 192.168.8.44
tm:
  image:
    repository: nexus3.o-ran-sc.org:10002/o-ran-sc/aiml-fw-awmf-tm-docker
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: "1.1.3"

leofs:
  image:
    repository: leofs
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: "latest"

dataextraction:
  image:
    repository: nexus3.o-ran-sc.org:10002/o-ran-sc/aiml-fw-athp-data-extraction-docker
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: "1.0.1"

datalake:
  influxdb:
    host: 192.168.8.229
    port: 30001
    orgname: influxdata
    bucket: UEData
    token: PYvZ2YLwKbgARSbfLHw5Wy0kZCnXG6Xl

kfadapter:
  image:
    repository: nexus3.o-ran-sc.org:10002/o-ran-sc/aiml-fw-athp-tps-kubeflow-adapter-docker
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: "1.1.0"

aimldashboard:
  image:
    repository: nexus3.o-ran-sc.org:10002/o-ran-sc/portal-aiml-dashboard-docker
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: "1.1.2"
  host:
    tm_host: "localhost"
    notebook_host: "localhost"
    debug: "\"false\""

aimlnotebook:
  image:
    repository: nexus3.o-ran-sc.org:10002/o-ran-sc/portal-aiml-notebook-docker
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: "1.1.2"

```

And then reinstall.

```javascript=
bin/uninstall.sh
bin/install.sh -f RECIPE_EXAMPLE/example_recipe_latest_stable.yaml
```

##### 5-2-2-4. Use AI/ML dashboard to create training job.

Please follow this [user guide](https://hackmd.io/@Jerry0714/HJmhjj-kT#3-2-1-Create-Training-function).

![](https://hackmd.io/_uploads/BykGFS6Za.png)

And Now, I can use SMO data to training. The connection of the AI/ML and SMO is complete.

#### 5-2-3. Use model to predict data.

Please follow this [user guide](https://hackmd.io/@Jerry0714/HJmhjj-kT#3-2-3-Deploy-trained-qoe-prediction-model-on-Kserve).

Prediction result :

![](https://hackmd.io/_uploads/rylkvH0ba.jpg)

https://drive.google.com/drive/folders/1ofr6Pf2p1UMDmZYeYU-lONF7kzxbu6RF