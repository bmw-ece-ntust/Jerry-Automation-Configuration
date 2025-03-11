# <center>Training Function</center>

<center>Testing training model by using Rictest Data</center>

###### tags: `AI/ML`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue, Oct19 , 2023 3:38 PM][color=#38c16a]

:::info
**Introduction :**

Understanding the impact of differences in insertion methods on training results, as well as the ability to write data more accurately.

**Goal：**

- [x] Use Rictest Data to train model.
:::
**<center>Table of Content</center>**
[TOC]

## 1. Training function

The OSC provide the training function let you can training model.

```javascript=
def train_export_model(trainingjobName: str, epochs: str, version: str):
    
    import tensorflow as tf
    from numpy import array
    from tensorflow.keras.models import Sequential
    from tensorflow.keras.layers import Dense
    from tensorflow.keras.layers import Flatten, Dropout, Activation
    from tensorflow.keras.layers import LSTM
    import numpy as np
    print("numpy version")
    print(np.__version__)
    import pandas as pd
    import os
    from featurestoresdk.feature_store_sdk import FeatureStoreSdk
    from modelmetricsdk.model_metrics_sdk import ModelMetricsSdk
    
    fs_sdk = FeatureStoreSdk()
    mm_sdk = ModelMetricsSdk()
    
    features = fs_sdk.get_features(trainingjobName, ['measTimeStampRf', 'nrCellIdentity', 'pdcpBytesDl','pdcpBytesUl'])
    print("Dataframe:")
    print(features)

    features_cellc2b2 = features[features['nrCellIdentity'] == "S9/B2/C1"]
    print("Dataframe for cell : c2/B2")
    print(features_cellc2b2)
    print('Previous Data Types are --> ', features_cellc2b2.dtypes)
    features_cellc2b2["pdcpBytesDl"] = pd.to_numeric(features_cellc2b2["pdcpBytesDl"], downcast="float")
    features_cellc2b2["pdcpBytesUl"] = pd.to_numeric(features_cellc2b2["pdcpBytesUl"], downcast="float")
    print('New Data Types are --> ', features_cellc2b2.dtypes)
    
    features_cellc2b2 = features_cellc2b2[['pdcpBytesDl', 'pdcpBytesUl']]
    
    def split_series(series, n_past, n_future):
        X, y = list(), list()
        for window_start in range(len(series)):
            past_end = window_start + n_past
            future_end = past_end + n_future
            if future_end > len(series):
                break
            # slicing the past and future parts of the window
            past, future = series[window_start:past_end, :], series[past_end:future_end, :]
            X.append(past)
            y.append(future)
        return np.array(X), np.array(y)
    X, y = split_series(features_cellc2b2.values,10, 1)
    X = X.reshape((X.shape[0], X.shape[1],X.shape[2]))
    y = y.reshape((y.shape[0], y.shape[2]))
    print(X.shape)
    print(y.shape)
    
    model = Sequential()
    model.add(LSTM(units = 150, activation="tanh" ,return_sequences = True, input_shape = (X.shape[1], X.shape[2])))

    model.add(LSTM(units = 150, return_sequences = True,activation="tanh"))

    model.add(LSTM(units = 150,return_sequences = False,activation="tanh" ))

    model.add((Dense(units = X.shape[2])))
    
    model.compile(loss='mse', optimizer='adam',metrics=['mse'])
    model.summary()
    
    model.fit(X, y, batch_size=10,epochs=int(epochs), validation_split=0.2)
    yhat = model.predict(X, verbose = 0)

    
    xx = y
    yy = yhat
    model.save("./")
    import json
    data = {}
    data['metrics'] = []
    data['metrics'].append({'Accuracy': str(np.mean(np.absolute(np.asarray(xx)-np.asarray(yy))<5))})
    
    mm_sdk.upload_metrics(data, trainingjobName, version)
    mm_sdk.upload_model("./", trainingjobName, version)
```
## 2. Use Pipelines UI to manage your trainnig function
First-time usage of the UI interface requires the following configurations.

```javascript=
kubectl -n kubeflow edit svc ml-pipeline-ui
```
```javascript=
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"ml-pipeline-ui","application-crd-id":"kubeflow-pipelines"},"name":"ml-pipeline-ui","namespace":"kubeflow"},"spec":{"ports":[{"name":"http","port":80,"protocol":"TCP","targetPort":3000}],"selector":{"app":"ml-pipeline-ui","application-crd-id":"kubeflow-pipelines"}}}
  creationTimestamp: "2023-11-15T17:34:01Z"
  labels:
    app: ml-pipeline-ui
    application-crd-id: kubeflow-pipelines
  name: ml-pipeline-ui
  namespace: kubeflow
  ownerReferences:
  - apiVersion: app.k8s.io/v1beta1
    blockOwnerDeletion: true
    controller: false
    kind: Application
    name: pipeline
    uid: 07fd243d-5de5-4a2b-8760-c0c8b1cd8d9c
  resourceVersion: "452720"
  uid: 0f51e89b-294b-4254-8831-ed22f0a4f11b
spec:
  clusterIP: 10.97.74.143
  clusterIPs:
  - 10.97.74.143
  externalIPs:
  - 192.168.8.44
  externalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: http
    nodePort: 31816
    port: 80
    protocol: TCP
    targetPort: 3000
  selector:
    app: ml-pipeline-ui
    application-crd-id: kubeflow-pipelines
  sessionAffinity: None
  type: LoadBalancer
status:
  loadBalancer: {}
```

Revise **"type: ClusterIP"** to **"type: LoadBalancer"**

```javascript=42
type: LoadBalancer
```

Add new config **"externalIPs"** let you can access the webpage.

```javascript=
externalIPs:
- 192.168.8.44
```

Check you successfully revise or not.

```javascript=
kubectl get svc -A | grep ml-pipeline-ui 
```
![Screenshot 2023-11-17 at 3.15.14 AM](https://hackmd.io/_uploads/BkwU5kNE6.png)

Open website and use this URL.

```javascript=
192.168.8.44:31816
```

**Result :**

![Screenshot 2023-11-17 at 3.17.41 AM](https://hackmd.io/_uploads/Skvyo1NEp.png)

## 3. Compare different data insert method  
### 3-1. Rictest Data
#### 3-1-1. Original data (CellReports.csv)
```javascript=
1697094793000.00 ,S1/B2/C1,0,0
1697094793000.00 ,S7/B2/C1,0,0
1697094793000.00 ,S8/B2/C1,0,0
1697094793000.00 ,S9/B2/C1,0.035999998,0.035999998
1697094793000.00 ,S1/B13/C1,0,0
1697094793000.00 ,S1/B13/C2,0,0
1697094793000.00 ,S1/B13/C3,0.027000001,0.027000001
1697094793000.00 ,S2/B13/C1,0.027000001,0.027000001
1697094793000.00 ,S2/B13/C2,0.027000001,0.027000001
1697094793000.00 ,S2/B13/C3,0,0
1697094793000.00 ,S3/B13/C1,0.027000001,0.027000001
1697094793000.00 ,S3/B13/C2,0,0
1697094793000.00 ,S3/B13/C3,0.027000001,0.027000001
1697094793000.00 ,S4/B13/C1,0.027000001,0.027000001
1697094793000.00 ,S4/B13/C2,0.027000001,0.027000001
...
```

Use this code to invert.

```javascript=
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Created on Mon Nov 13 01:28:30 2023

@author: wuchengze
"""

import csv
import datetime

# 開啟原始CSV檔案和新的CSV檔案
with open('/Users/wuchengze/Desktop/CellReports.csv', 'r') as input_file, open('/Users/wuchengze/Desktop/modified_CellReports.csv', 'w', newline='') as output_file:
    reader = csv.reader(input_file)
    writer = csv.writer(output_file)

    # 讀取原始CSV資料行
    for row in reader:
        # 將時間戳轉換為新格式
        unix_ts = float(row[0]) / 1000  # 假設時間戳在CSV中的第一列（索引0）
        time = datetime.datetime.utcfromtimestamp(unix_ts)
        formatted_time = time.strftime('%Y-%m-%dT%H:%M:%S.%f')[:-3] + 'Z'

        # 更新CSV行中的時間戳
        row[0] = formatted_time

        # 寫入新的CSV檔案
        writer.writerow(row)
```

If there is a time zone error, you can refer to this code.

```javascript=
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Created on Mon Nov 13 01:28:30 2023

@author: wuchengze
"""

import csv
import datetime
from pytz import utc, timezone

# 開啟原始CSV檔案和新的CSV檔案
with open('/Users/wuchengze/Desktop/CellReports1.csv', 'r') as input_file, open('/Users/wuchengze/Desktop/modified_CellReports.csv', 'w', newline='') as output_file:
    reader = csv.reader(input_file)
    writer = csv.writer(output_file)

    # 讀取原始CSV資料行
    for row in reader:
        # 將時間戳轉換為新格式
        unix_ts = int(row[0])  # 假設時間戳在CSV中的第一列（索引0）
        time = datetime.datetime.utcfromtimestamp(unix_ts).replace(tzinfo=utc)
        time_taipei = time.astimezone(timezone('Asia/Taipei'))
        formatted_time = time_taipei.strftime('%Y-%m-%dT%H:%M:%S.%f')[:-3] + 'Z'

        # 更新CSV行中的時間戳
        row[0] = formatted_time

        # 寫入新的CSV檔案
        writer.writerow(row)
```

#### 3-1-2. modified_CellReports.csv
```javascript=
2023-10-12T07:13:13.000Z,S1/B2/C1,0,0
2023-10-12T07:13:13.000Z,S7/B2/C1,0,0
2023-10-12T07:13:13.000Z,S8/B2/C1,0,0
2023-10-12T07:13:13.000Z,S9/B2/C1,0.035999998,0.035999998
2023-10-12T07:13:13.000Z,S1/B13/C1,0,0
2023-10-12T07:13:13.000Z,S1/B13/C2,0,0
2023-10-12T07:13:13.000Z,S1/B13/C3,0.027000001,0.027000001
2023-10-12T07:13:13.000Z,S2/B13/C1,0.027000001,0.027000001
2023-10-12T07:13:13.000Z,S2/B13/C2,0.027000001,0.027000001
2023-10-12T07:13:13.000Z,S2/B13/C3,0,0
2023-10-12T07:13:13.000Z,S3/B13/C1,0.027000001,0.027000001
2023-10-12T07:13:13.000Z,S3/B13/C2,0,0
2023-10-12T07:13:13.000Z,S3/B13/C3,0.027000001,0.027000001
2023-10-12T07:13:13.000Z,S4/B13/C1,0.027000001,0.027000001
2023-10-12T07:13:13.000Z,S4/B13/C2,0.027000001,0.027000001
2023-10-12T07:13:13.000Z,S4/B13/C3,0.027000001,0.027000001
2023-10-12T07:13:13.000Z,S5/B13/C1,0.027000001,0.027000001
```
### 3-2. Insert Method 1 (Data with tag)

I want to know what's the different between data with tag or not.

Create python file.
```javascript=
vim insert-ESdata.py
```
```javascript=
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS
import csv
data =[]

# Parameters of InfluxDB instance
myorg = "influxdata"
mybucket = "ES-Data"
mytoken = "PYvZ2YLwKbgARSbfLHw5Wy0kZCnXG6Xl"
myurl= "http://192.168.8.229:30001"

with open('CellReports.csv', 'r') as csvfile:
    reader = csv.reader(csvfile, delimiter=',')
    
    for row in reader:
        data.append(list(row))


"""
Instantiate write api
"""

# Instantiate the client
client = InfluxDBClient(url=myurl,token=mytoken,org=myorg)

# Instantiate a write client using the client object 
write_api = client.write_api(write_options=SYNCHRONOUS)

for i in range(0, 15652):
    # Data (Point structure)
    _point = Point("liveCell")\
            .tag("Viavi_Cell_Name", data[i][1])\
            .field("DRB_UEThpDl", data[i][2])\
            .field("DRB_UEThpUl", data[i][3])\
            .time(data[i][0])
            # Write the data to influxDB        
    write_api.write(bucket=mybucket, record=[_point])

print("Finish writing 15653 metircs")
```
Use this command to insert data.

```javascript=
python3 insert-ESdata.py
```

**Result :**

![Screenshot 2023-11-17 at 4.08.40 AM](https://hackmd.io/_uploads/SysAUxV46.png)

![Screenshot 2023-11-17 at 4.08.07 AM](https://hackmd.io/_uploads/HJqJvlEVT.png)

#### 3-2-1. Revise Training function feature field
```javascript=
def train_export_model(trainingjobName: str, epochs: str, version: str):
    
    import tensorflow as tf
    from numpy import array
    from tensorflow.keras.models import Sequential
    from tensorflow.keras.layers import Dense
    from tensorflow.keras.layers import Flatten, Dropout, Activation
    from tensorflow.keras.layers import LSTM
    import numpy as np
    print("numpy version")
    print(np.__version__)
    import pandas as pd
    import os
    from featurestoresdk.feature_store_sdk import FeatureStoreSdk
    from modelmetricsdk.model_metrics_sdk import ModelMetricsSdk
    
    fs_sdk = FeatureStoreSdk()
    mm_sdk = ModelMetricsSdk()
    
    features = fs_sdk.get_features(trainingjobName, ['_time', 'Viavi_Cell_Name', 'DRB_UEThpDl','DRB_UEThpUl'])
    print("Dataframe:")
    print(features)

    features_cellc2b2 = features[features['Viavi_Cell_Name'] == "S9/B2/C1"]
    print("Dataframe for cell : S9/B2/C1")
    print(features_cellc2b2)
    print('Previous Data Types are --> ', features_cellc2b2.dtypes)
    features_cellc2b2["DRB_UEThpDl"] = pd.to_numeric(features_cellc2b2["DRB_UEThpDl"], downcast="float")
    features_cellc2b2["DRB_UEThpUl"] = pd.to_numeric(features_cellc2b2["DRB_UEThpUl"], downcast="float")
    print('New Data Types are --> ', features_cellc2b2.dtypes)
    
    features_cellc2b2 = features_cellc2b2[['DRB_UEThpDl', 'DRB_UEThpUl']]
    
    def split_series(series, n_past, n_future):
        X, y = list(), list()
        for window_start in range(len(series)):
            past_end = window_start + n_past
            future_end = past_end + n_future
            if future_end > len(series):
                break
            # slicing the past and future parts of the window
            past, future = series[window_start:past_end, :], series[past_end:future_end, :]
            X.append(past)
            y.append(future)
        return np.array(X), np.array(y)
    X, y = split_series(features_cellc2b2.values,10, 1)
    X = X.reshape((X.shape[0], X.shape[1],X.shape[2]))
    y = y.reshape((y.shape[0], y.shape[2]))
    print(X.shape)
    print(y.shape)
    
    model = Sequential()
    model.add(LSTM(units = 150, activation="tanh" ,return_sequences = True, input_shape = (X.shape[1], X.shape[2])))

    model.add(LSTM(units = 150, return_sequences = True,activation="tanh"))

    model.add(LSTM(units = 150,return_sequences = False,activation="tanh" ))

    model.add((Dense(units = X.shape[2])))
    
    model.compile(loss='mse', optimizer='adam',metrics=['mse'])
    model.summary()
    
    model.fit(X, y, batch_size=10,epochs=int(epochs), validation_split=0.2)
    yhat = model.predict(X, verbose = 0)

    
    xx = y
    yy = yhat
    model.save("./")
    import json
    data = {}
    data['metrics'] = []
    data['metrics'].append({'Accuracy': str(np.mean(np.absolute(np.asarray(xx)-np.asarray(yy))<5))})
    
    mm_sdk.upload_metrics(data, trainingjobName, version)
    mm_sdk.upload_model("./", trainingjobName, version)
```
Revise as the below.
```javascript=20
features = fs_sdk.get_features(trainingjobName, ['_time', 'Viavi_Cell_Name', 'DRB_UEThpDl','DRB_UEThpUl'])
print("Dataframe:")
print(features)

features_cellc2b2 = features[features['Viavi_Cell_Name'] == "S9/B2/C1"]
print("Dataframe for cell : S9/B2/C1")
print(features_cellc2b2)
print('Previous Data Types are --> ', features_cellc2b2.dtypes)
features_cellc2b2["DRB_UEThpDl"] = pd.to_numeric(features_cellc2b2["DRB_UEThpDl"], downcast="float")
features_cellc2b2["DRB_UEThpUl"] = pd.to_numeric(features_cellc2b2["DRB_UEThpUl"], downcast="float")
print('New Data Types are --> ', features_cellc2b2.dtypes)
    
features_cellc2b2 = features_cellc2b2[['DRB_UEThpDl', 'DRB_UEThpUl']]
```

[You can refer to these steps for training the model.](https://hackmd.io/655-sCWCQC2RhfCFg5c8bQ?view#3-2-Deploy-trained-qoe-prediction-model-on-Kserve)

#### 3-2-2. Use pipeline UI to check log

Select **"Run"** page and you will see all the training result you have.

![Screenshot 2023-11-17 at 6.27.41 AM](https://hackmd.io/_uploads/r1D3PGVVa.png)

Choose the latest one and click log to see the training detail.

![Screenshot 2023-11-17 at 6.32.10 AM](https://hackmd.io/_uploads/S11qOGEVa.png)

According to below training function code.
```javascript=20
features = fs_sdk.get_features(trainingjobName, ['_time', 'Viavi_Cell_Name', 'DRB_UEThpDl','DRB_UEThpUl'])
print("Dataframe:")
print(features)
```
**Result :**

Dataframe total data : **"15652"**

![Screenshot 2023-11-17 at 6.37.06 AM](https://hackmd.io/_uploads/SktO9MNET.png)

According to below training function code. Predict Viavi_Cell_Name **"S9/B2/C1"** 

```javascript=
features_cellc2b2 = features[features['Viavi_Cell_Name'] == "S9/B2/C1"]
print("Dataframe for cell : S9/B2/C1")
print(features_cellc2b2)
```

Dataframe for Cell : S9/B2/C1 , Total data : **"301"**

![Screenshot 2023-11-17 at 6.43.45 AM](https://hackmd.io/_uploads/HkhSsM4VT.png)

#### 3-2-3. Model layer

![Screenshot 2023-11-17 at 6.48.11 AM](https://hackmd.io/_uploads/r1bYhMN46.png)

#### 3-2-4. Prepare sample data to predict

[You can follow this procedure to predict.](https://hackmd.io/655-sCWCQC2RhfCFg5c8bQ?view#3-3-Test-predictions-using-model-deployed-on-Kserve)

```javascript=
{"signature_name": "serving_default", "instances": [[[0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1]]]}
```
#### 3-2-4. Prediction Result (Data with tag)

![Screenshot 2023-11-17 at 7.12.00 AM](https://hackmd.io/_uploads/rkzCZQN4a.png)

### 3-3. Insert Method 2 (Data without tag)

Create python file.
```javascript=
vim insert-ESdata2.py
```
```javascript=
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS
import csv
data =[]

# Parameters of InfluxDB instance
myorg = "influxdata"
mybucket = "ES-Data2"
mytoken = "PYvZ2YLwKbgARSbfLHw5Wy0kZCnXG6Xl"
myurl= "http://192.168.8.229:30001"

with open('CellReports.csv', 'r') as csvfile:
    reader = csv.reader(csvfile, delimiter=',')

    for row in reader:
        data.append(list(row))


"""
Instantiate write api
"""

# Instantiate the client
client = InfluxDBClient(url=myurl,token=mytoken,org=myorg)

# Instantiate a write client using the client object 
write_api = client.write_api(write_options=SYNCHRONOUS)

for i in range(0, 15652):
    # Data (Point structure)
    _point = Point("liveCell")\
            .field("timestamp", data[i][0])\
            .field("Viavi_Cell_Name", data[i][1])\
            .field("DRB_UEThpDl", data[i][2])\
            .field("DRB_UEThpUl", data[i][3])
            # Write the data to influxDB        
    write_api.write(bucket=mybucket, record=[_point])

print("Finish writing 15653 metircs")
```
Use this command to insert data.

```javascript=
python3 insert-ESdata.py
```

**Result :**

![Screenshot 2023-11-17 at 4.08.40 AM](https://hackmd.io/_uploads/SysAUxV46.png)

![Screenshot 2023-11-17 at 1.36.51 PM](https://hackmd.io/_uploads/HytW2ON4a.png)

#### 3-3-1. Revise Training function feature field
```javascript=
def train_export_model(trainingjobName: str, epochs: str, version: str):
    
    import tensorflow as tf
    from numpy import array
    from tensorflow.keras.models import Sequential
    from tensorflow.keras.layers import Dense
    from tensorflow.keras.layers import Flatten, Dropout, Activation
    from tensorflow.keras.layers import LSTM
    import numpy as np
    print("numpy version")
    print(np.__version__)
    import pandas as pd
    import os
    from featurestoresdk.feature_store_sdk import FeatureStoreSdk
    from modelmetricsdk.model_metrics_sdk import ModelMetricsSdk
    
    fs_sdk = FeatureStoreSdk()
    mm_sdk = ModelMetricsSdk()
    
    features = fs_sdk.get_features(trainingjobName, ['timestamp', 'Viavi_Cell_Name', 'DRB_UEThpDl','DRB_UEThpUl'])
    print("Dataframe:")
    print(features)

    features_cellc2b2 = features[features['Viavi_Cell_Name'] == "S9/B2/C1"]
    print("Dataframe for cell : S9/B2/C1")
    print(features_cellc2b2)
    print('Previous Data Types are --> ', features_cellc2b2.dtypes)
    features_cellc2b2["DRB_UEThpDl"] = pd.to_numeric(features_cellc2b2["DRB_UEThpDl"], downcast="float")
    features_cellc2b2["DRB_UEThpUl"] = pd.to_numeric(features_cellc2b2["DRB_UEThpUl"], downcast="float")
    print('New Data Types are --> ', features_cellc2b2.dtypes)
    
    features_cellc2b2 = features_cellc2b2[['DRB_UEThpDl', 'DRB_UEThpUl']]
    
    def split_series(series, n_past, n_future):
        X, y = list(), list()
        for window_start in range(len(series)):
            past_end = window_start + n_past
            future_end = past_end + n_future
            if future_end > len(series):
                break
            # slicing the past and future parts of the window
            past, future = series[window_start:past_end, :], series[past_end:future_end, :]
            X.append(past)
            y.append(future)
        return np.array(X), np.array(y)
    X, y = split_series(features_cellc2b2.values,10, 1)
    X = X.reshape((X.shape[0], X.shape[1],X.shape[2]))
    y = y.reshape((y.shape[0], y.shape[2]))
    print(X.shape)
    print(y.shape)
    
    model = Sequential()
    model.add(LSTM(units = 150, activation="tanh" ,return_sequences = True, input_shape = (X.shape[1], X.shape[2])))

    model.add(LSTM(units = 150, return_sequences = True,activation="tanh"))

    model.add(LSTM(units = 150,return_sequences = False,activation="tanh" ))

    model.add((Dense(units = X.shape[2])))
    
    model.compile(loss='mse', optimizer='adam',metrics=['mse'])
    model.summary()
    
    model.fit(X, y, batch_size=10,epochs=int(epochs), validation_split=0.2)
    yhat = model.predict(X, verbose = 0)

    
    xx = y
    yy = yhat
    model.save("./")
    import json
    data = {}
    data['metrics'] = []
    data['metrics'].append({'Accuracy': str(np.mean(np.absolute(np.asarray(xx)-np.asarray(yy))<5))})
    
    mm_sdk.upload_metrics(data, trainingjobName, version)
    mm_sdk.upload_model("./", trainingjobName, version)
```
Revise as the below.
```javascript=20
features = fs_sdk.get_features(trainingjobName, ['timestamp', 'Viavi_Cell_Name', 'DRB_UEThpDl','DRB_UEThpUl'])
print("Dataframe:")
print(features)

features_cellc2b2 = features[features['Viavi_Cell_Name'] == "S9/B2/C1"]
print("Dataframe for cell : S9/B2/C1")
print(features_cellc2b2)
print('Previous Data Types are --> ', features_cellc2b2.dtypes)
features_cellc2b2["DRB_UEThpDl"] = pd.to_numeric(features_cellc2b2["DRB_UEThpDl"], downcast="float")
features_cellc2b2["DRB_UEThpUl"] = pd.to_numeric(features_cellc2b2["DRB_UEThpUl"], downcast="float")
print('New Data Types are --> ', features_cellc2b2.dtypes)
    
features_cellc2b2 = features_cellc2b2[['DRB_UEThpDl', 'DRB_UEThpUl']]
```
#### 3-3-2. Use pipeline UI to check log

![Screenshot 2023-11-17 at 2.06.45 PM](https://hackmd.io/_uploads/HylMXK4Ep.png)

Select latest training job.

![Screenshot 2023-11-17 at 2.08.15 PM](https://hackmd.io/_uploads/SyBDQtVNT.png)

According to below training function code.
```javascript=20
features = fs_sdk.get_features(trainingjobName, ['timestamp', 'Viavi_Cell_Name', 'DRB_UEThpDl','DRB_UEThpUl'])
print("Dataframe:")
print(features)
```
**Result :**

Dataframe total data : **"15652"**

![Screenshot 2023-11-17 at 2.11.52 PM](https://hackmd.io/_uploads/ryctVFV46.png)

According to below training function code. Predict Viavi_Cell_Name **"S9/B2/C1"** 

```javascript=
features_cellc2b2 = features[features['Viavi_Cell_Name'] == "S9/B2/C1"]
print("Dataframe for cell : S9/B2/C1")
print(features_cellc2b2)
```

Dataframe for Cell : S9/B2/C1 , Total data : **"301"**

![Screenshot 2023-11-17 at 2.12.26 PM](https://hackmd.io/_uploads/rybh4YVEp.png)


#### 3-3-3. Model layer

![Screenshot 2023-11-17 at 2.12.52 PM](https://hackmd.io/_uploads/SJza4KN4T.png)

#### 3-3-4. Prepare sample data to predict
[You can follow this procedure to predict.](https://hackmd.io/655-sCWCQC2RhfCFg5c8bQ?view#3-3-Test-predictions-using-model-deployed-on-Kserve)

```javascript=
{"signature_name": "serving_default", "instances": [[[0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1],
       [0.1, 0.1]]]}
```
#### 3-3-5. Result (Data without tag)
![Screenshot 2023-11-17 at 2.20.00 PM](https://hackmd.io/_uploads/H14XLYV46.png)

## 4. Conclusion

Compare different insert method dataframe.

### 4-1. Insert Method 1 (Data with tag)

Dataframe for Cell : S9/B2/C1 , Total data : **"301"**

![Screenshot 2023-11-17 at 6.43.45 AM](https://hackmd.io/_uploads/HkhSsM4VT.png)

Prediction result :

![Screenshot 2023-11-17 at 7.12.00 AM](https://hackmd.io/_uploads/rkzCZQN4a.png)

### 4-2. Insert Method 2 (Data without tag)

Dataframe for Cell : S9/B2/C1 , Total data : **"301"**

![Screenshot 2023-11-17 at 2.12.26 PM](https://hackmd.io/_uploads/rybh4YVEp.png)

Prediction result :

![Screenshot 2023-11-17 at 2.20.00 PM](https://hackmd.io/_uploads/H14XLYV46.png)

### 4-3. Conclusion

The input data extraction for both Method 1 and Method 2 is the same, and it is identical to the original data. It can be observed that the predictions of different models are approximately the same. **Therefore, the conclusion is that the presence or absence of tags has no impact.**