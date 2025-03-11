# <center>Using Grafana to show Prediction Result</center>

<center></center>

###### tags: `AI/ML`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue, Oct19 , 2023 3:38 PM][color=#38c16a]

:::info
**Introduction :**

**Goal：**

:::
**<center>Table of Content</center>**
[TOC]

## 1. Prepare Rictest Data to training and validate
I prepare 7days rictest data as the below. 
- **Total data** : 524211
- **Training data (6days data to training)** : 449280 
- **Validate data (1days data to validate)** : 74931

**Rictest_Data.csv (Training Data 1/1 ~ 1/6)**
```javascript=
2023-01-01T00:00:00.000Z,S1/B2/C1,0.278191991,0.278191991
2023-01-01T00:00:00.000Z,S7/B2/C1,0.14631002,0.14631002
2023-01-01T00:00:00.000Z,S8/B2/C1,0.220303236,0.220303236
2023-01-01T00:00:00.000Z,S9/B2/C1,0.186427643,0.186427643
2023-01-01T00:00:00.000Z,S1/B13/C1,0,0
...
2023-01-06T23:59:00.000Z,S3/N77/C3,0,0
2023-01-06T23:59:00.000Z,S4/N77/C1,0,0
2023-01-06T23:59:00.000Z,S4/N77/C2,0.015741933,0.001314573
2023-01-06T23:59:00.000Z,S4/N77/C3,0.00246292,0.000155333
2023-01-06T23:59:00.000Z,S1/B2/C1,0.197293213,0.197293213
```
**Rictest_Data_Validate.csv (Training Data 1/7)**
```javascript=
2023-01-07T00:00:00.000Z,S7/B2/C1,0.218794406,0.218794406
2023-01-07T00:00:00.000Z,S8/B2/C1,0.200000003,0.200000003
2023-01-07T00:00:00.000Z,S9/B2/C1,0.215936965,0.215936965
2023-01-07T00:00:00.000Z,S1/B13/C1,0.029375494,0.029375494
2023-01-07T00:00:00.000Z,S1/B13/C2,0.233733064,0.233733064
...
2023-01-07T23:59:00.000Z,S3/N77/C2,0,0
2023-01-07T23:59:00.000Z,S3/N77/C3,0,0
2023-01-07T23:59:00.000Z,S4/N77/C1,0,0
2023-01-07T23:59:00.000Z,S4/N77/C2,0,0
2023-01-07T23:59:00.000Z,S4/N77/C3,0.310629606,0.030365599
```
## 2. Training function

- Added time sorting for data.
- Prediction DL and UL for cell "S1/B13/C1"


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
    
    features = fs_sdk.get_features(trainingjobName, ['_time','Viavi_Cell_Name','DRB_UEThpDl','DRB_UEThpUl'])
    print("Dataframe:")
    print(features)
    
    features = features.sort_values(by='time')
    print("Dataframe after sorting:")
    print(features)
    
    features_cellc2b2 = features[features['Viavi_Cell_Name'] == "S1/B13/C1"]
    print("Dataframe for cell : S1/B13/C1")
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

## 3. Upload prediction result to the Grafana.
Use Rictest Data to predict, I use the **6th final 10data** as the inference data.

**input_qoe5.json**
```javascript=
{"signature_name": "serving_default", "instances": [[[0.092074645, 0.092074645],
[0.064860675, 0.064860675],
[0.080449517, 0.080449517],
[0.089248372, 0.089248372],
[0.100672569, 0.100672569],
[0.167375988, 0.167375988],
[0.001125669, 0.001125669],
[0.180123143, 0.180123143],
[0.187083112, 0.187083112],
[0.154647711, 0.154647711]]]}
```
You can use my upload code to dynamic insert prediction result to the influxDB.
```javascript=
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS
import json
import subprocess
import time
from datetime import datetime, timedelta

# Parameters of InfluxDB instance
myorg = "Jerry"
mybucket = "Rictest_Data_Result"
mytoken = "sF3jR78UFGg26JTp4tF2O14QWcO7n32g"
myurl = "http://192.168.8.12:30001"

# Start time
start_time = datetime.strptime("2023-01-07T00:00:00.000Z", "%Y-%m-%dT%H:%M:%S.%fZ")

while True:
    # Read input file
    with open('./input_qoe5.json', 'r') as file:
        input_data = json.load(file)

    # Get predictions
    model_name = "qoe-model"
    response = json.loads(subprocess.check_output(['curl', '-s', '-H', f'Host: {model_name}.kserve-test.example.com', f'http://192.168.8.44:30650/v1/models/{model_name}:predict', '-d', json.dumps(input_data)]))

    # List to store predictions
    predictions_list = []

    # Extract predictions
    prediction1 = response['predictions'][0][0]
    prediction2 = response['predictions'][0][1]

    # Add predictions to the list
    predictions_list.append({'prediction1': prediction1, 'prediction2': prediction2})

    # Add new predictions in a specific format to the end of the input file
    input_data['instances'][0].append([prediction1, prediction2])

    with open('./input_qoe5.json', 'w') as file:
        json.dump(input_data, file)

    # Connect to InfluxDB
    client = InfluxDBClient(url=myurl, token=mytoken, org=myorg)
    write_api = client.write_api(write_options=SYNCHRONOUS)

    # Add predictions to InfluxDB
    for i in range(0, len(predictions_list)):
        # Data (Point structure)
        _point = Point("liveCellPrediction")\
                .field("Prediction1", float(predictions_list[i]['prediction1']))\
                .field("Prediction2", float(predictions_list[i]['prediction2']))\
                .time(start_time + timedelta(minutes=i))  # Use timestamps for every minute

        # Write the data to InfluxDB
        write_api.write(bucket=mybucket, record=[_point])

    print(f"Finish writing {len(predictions_list)} metrics with predictions to InfluxDB")

    # Update start time for the next upload with the correct time
    start_time += timedelta(minutes=len(predictions_list))

    # Pause for 1 second
    time.sleep(1)

```

## 4. Training result

Use pipeline-Ui to check log. If you don't know how to use, please [click here.](https://hackmd.io/t0ANLfbjQ3OnPM9uoCMJUw#2-Use-Pipelines-UI-to-manage-your-trainnig-function)

![Screenshot 2023-12-29 at 8.41.51 PM](https://hackmd.io/_uploads/HkFlJBhD6.png)


**DataFrame**

![Screenshot 2023-12-29 at 8.38.55 PM](https://hackmd.io/_uploads/rkvDAEhDT.png)


**Dataframe after sorting**

![Screenshot 2023-12-29 at 8.39.08 PM](https://hackmd.io/_uploads/rkfLRN3Pp.png)

**Dataframe for cell : S1/B13/C1**

![Screenshot 2023-12-29 at 8.40.21 PM](https://hackmd.io/_uploads/BksL043wp.png)

## 4. Prediction result

Open Grafana UI to show data. http://192.168.8.12:30000

**Rictest_Data_Validate query code.**

```javascript=
from(bucket: "Rictest_Data_Validate")
  |> range(start: -1y)
  |> filter(fn: (r) => r["Viavi_Cell_Name"] == "S1/B13/C1")
  |> filter(fn: (r) => r["_measurement"] == "liveCell")
```

**Rictest_Data_Result query code.**

```javascript=
from(bucket: "Rictest_Data_Result")
  |> range(start: -1y)
  |> filter(fn: (r) => r["_measurement"] == "liveCellPrediction")
```

**Result :**

![Screenshot 2023-12-29 at 9.16.59 PM](https://hackmd.io/_uploads/r1htOrhDT.png)

![Screenshot 2023-12-29 at 9.28.33 PM](https://hackmd.io/_uploads/rJXoKB3wa.png)

### 4-1. Trying to optimize the prediction result.
Revise the training function.
```javascript=
    model = Sequential()
    model.add(LSTM(units = 128, activation="tanh" ,return_sequences = True, input_shape = (X.shape[1], X.shape[2])))

    model.add(LSTM(units = 64, return_sequences = True,activation="tanh"))

    model.add(LSTM(units = 32, return_sequences = False,activation="tanh" ))

    model.add((Dense(units = X.shape[2])))
    
    model.compile(loss='mse', optimizer='adam',metrics=['mse'])
    model.summary()
    
    model.fit(X, y, batch_size=32,epochs=int(epochs), validation_split=0.2)
    yhat = model.predict(X, verbose = 0)
```
**Result :**

![Screenshot 2023-12-29 at 11.01.37 PM](https://hackmd.io/_uploads/B1S6xvhvp.png)

![Screenshot 2023-12-29 at 11.01.45 PM](https://hackmd.io/_uploads/rJTTgD2Pa.png)
