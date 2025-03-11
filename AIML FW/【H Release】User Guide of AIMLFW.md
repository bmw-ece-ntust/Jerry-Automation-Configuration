# 【H Release】User Guide of AIMLFW
###### tags: `AI/ML`
![](https://i.imgur.com/JORnn3y.png =150x)@NTUST
>[name=Jerry][time=Fri, Sep 15, 2023 18:00 PM][color=#aaaffa]
:::info
**Introduction：**

Use this user guid to use AI/ML platform.

**Goal：**
- [x] [Rrepare standalone Influx DB as data source for AIMLFW to prediction.](#3-Rrepare-standalone-Influx-DB-as-data-source-for-AIMLFW-to-prediction)
- [ ] [Prepare Non-RT RIC DME(Data Management & Exposure) as data source for AIMLFW](#4-Prepare-Non-RT-RIC-DMEData-Management-amp-Exposure-as-data-source-for-AIMLFW)

**Reference：**
- [【H Release】Installation Guide of AIMLFW](https://hackmd.io/@Jerry0714/rkdJx5gph) 
:::
**<center>Table of Content</center>**
[TOC]

## 1. AIMLFW Design diagram

![](https://hackmd.io/_uploads/BkjAZMXlT.png)

## 2. SSH to the server

- **Server name：** aiml
- **Server IP：** 192.168.8.44
- **Password：** bmwlab

![](https://hackmd.io/_uploads/rkQdWnbkT.png)

### 2-1. Check if the pod all running or not
```javascript=
kubectl get pod -A
```
![](https://hackmd.io/_uploads/BJiBpvwAh.png)

## 3. Rrepare standalone Influx DB as data source for AIMLFW to prediction.
### 3-1. Inter the InfluxDB to check data
```javascript=
kubectl exec -it my-release-influxdb-5b77fc46b4-65f5x -- bash
```
**<Org_name>**：primary
**<API_Token>**：INTKOiswguIKu3OzBS72

Check data is in the influxDB or not.
```javascript=
influx query  'from(bucket: "UEData") |> range(start: -1000d)' -o <Org_name> -t <API_Token>
```
For example.
```javascript=
influx query  'from(bucket: "UEData") |> range(start: -1000d)' -o primary -t INTKOiswguIKu3OzBS72
```

![](https://hackmd.io/_uploads/B1FHqpQT3.png)

<style>
  .img {
    position:relative;
    width: 50%;
    left:25%;
  }
</style>

### 3-2. Deploy trained qoe prediction model on Kserve
Use Enternet to enter the AIMLFW dashboard by using the following url.

```javascript=
http://<Your VM IP>:32005/
```

![](https://hackmd.io/_uploads/SJpE947an.png)

#### 3-2-1. Create Training function
Use menu to select training function to creat.

![](https://hackmd.io/_uploads/BJzpMOu03.png)

Click **"Create Training Functions"** then you will enter the page like this figure.

![](https://hackmd.io/_uploads/BJz3N_uR2.png)

After you click **"qoe-pipline.ipynb"**, you will see like this figure as the below. Create new file and duplicate the code one by one.

**Step 1：** Create new file

![](https://hackmd.io/_uploads/r11Xjs_R2.png)

**Step 2：** Copy the code one by one which need to click **"RUN"** until **"In[4]"**.

![](https://hackmd.io/_uploads/H1pgpou02.png)
![](https://hackmd.io/_uploads/r12G6iO03.png)
![](https://hackmd.io/_uploads/SJM46su0h.png)
![](https://hackmd.io/_uploads/HkfSpjdRn.png)

There has no setting need to modify.

**Step 3：** Copy **"In[5]"** code and modify **name** to the **"qoetest"** before you running **"In[6]"**.

![](https://hackmd.io/_uploads/BkVC0i_R3.png)
![](https://hackmd.io/_uploads/Bkpyxn_Rh.png)

**Step 4：** Copy **"In[7]"** code and modify **pipeline_name** and **"request.post_URI"** to the **"qoetest"** before you running.

If you successful you will recieve 200 response.

![](https://hackmd.io/_uploads/ry_8e2ORh.png)

**Step 5：** After you complete the above configuration, back off the previous page. You will see the Untitled.ipynb which you create before and the new **"qoe_model_pipeline.zip"**.
 
![](https://hackmd.io/_uploads/rkExG3_03.png)

**Step 6：** Download the **".zip"** file and use menu to choose Training function which need to upload **".zip"** file.

![](https://hackmd.io/_uploads/Skrm4hdC2.png)

![](https://hackmd.io/_uploads/HykcB2_R2.png)

**Step 7：** Check the training function is correctly creat or not.

![](https://hackmd.io/_uploads/SkfnUh_02.png)

#### 3-2-2. Create Training job

![](https://hackmd.io/_uploads/SyWgAp7T3.png)

Use the parameter by this figure. **"Training Functions"** which is that you previous create function.

![](https://hackmd.io/_uploads/HkL-CuwC3.png)

<p class="img"><img src="https://hackmd.io/_uploads/HJ5DRwDAn.png"></p>

**<center>Table, Training job parameter</center>**

Back to the menu to select the training job status to check is the model complete or not.

![](https://hackmd.io/_uploads/HJomTL_Rn.png)

Wait for minutes then the model is complete.

![](https://hackmd.io/_uploads/B1qgMu_C3.png)

#### 3-2-3. Deploy trained qoe prediction model on Kserve
Create namespace using command below.
```javascript=
kubectl create namespace kserve-test
```
Create qoe.yaml file with below contents.
```javascript=
cd
vim qoe.yaml
```
Update the file like this figure.
```javascript=
apiVersion: "serving.kserve.io/v1beta1"
kind: "InferenceService"
metadata:
  name: qoe-model
spec:
  predictor:
    tensorflow:
      storageUri: "<update Model URL here>"
      runtimeVersion: "2.5.1"
      resources:
        requests:
          cpu: 0.1
          memory: 0.5Gi
        limits:
          cpu: 0.1
          memory: 0.5Gi
```
Use the below step to get the **storageUri**.

**Step 1：** Click info.
**Step 2：** Duplicate the Model URL.

![](https://hackmd.io/_uploads/HJyWs_uRh.png)

Update "update Model URL".

![](https://hackmd.io/_uploads/SkpWT__Rh.png)

To deploy model update the Model URL in the qoe.yaml file and execute below command to deploy model.

```javascript=
kubectl apply -f qoe.yaml -n kserve-test
```

![](https://hackmd.io/_uploads/H1RqaOu03.png)

Use this command to check **"kserve-test qoe-model-predictor-default-00001-deployment-7d479b69f8-kzv85"** is running or not.
```javascript=
kubectl get pod -A
```
![](https://hackmd.io/_uploads/SyerkY_Rh.png)

Check running state of pod using below command
```javascript=
kubectl get pods -n kserve-test
```
![](https://hackmd.io/_uploads/SkM6kK_R2.png)

### 3-3. Test predictions using model deployed on Kserve
Use below command to obtain Ingress port for Kserve.
```javascript=
kubectl get svc istio-ingressgateway -n istio-system
```
![](https://hackmd.io/_uploads/SkDjeFOCh.png)

Create predict.sh file with following contents
```javascript=
vim predict.sh
```
Duplicate the below content and update the **"IP of host"** where Kserve is deployed and ingress **"port"** of Kserve obtained using above method.
```javascript=
model_name=qoe-model
curl -v -H "Host: $model_name.kserve-test.example.com" http://"IP of where Kserve is deployed":"ingress port for Kserve"/v1/models/$model_name:predict -d @./input_qoe.json
```
The file have been modified as the figure including IP and port.

![](https://hackmd.io/_uploads/BkWro9dA2.png)

:::spoiler click here to find your IP and port.
**My IP：** 192.168.31.129
**My Port：** 32384 
```javascript=
ip a
```

![](https://hackmd.io/_uploads/rkzTu5uA2.png)

Use this command to find **TCP,80** and the number is your port.
```javascript=
kubectl get svc istio-ingressgateway -n istio-system
```

![](https://hackmd.io/_uploads/rJotqcOAh.png)

:::

After complete update, create sample data for predictions in file **input_qoe.json**. Add the following content in input_qoe.json file.
```javascript=
vim input_qoe.json
```
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

![](https://hackmd.io/_uploads/Bykf0qOR2.png)

Use command below to trigger predictions.
```javascript=
source predict.sh
```
**SUCCESSFUL RESULT：If you appear this information, you will see like below and that mean you complete the AI/ML Install and also success to use  training model to predict.**

![](https://hackmd.io/_uploads/HyJW1o_03.png)

## 4. Prepare Non-RT RIC DME(Data Management & Exposure) as data source for AIMLFW

### 4-1. Make sure the RAN PM set up is full complete.

![](https://hackmd.io/_uploads/r1dLWZElT.jpg)
![](https://hackmd.io/_uploads/r14bzZNea.jpg)

### 4-2. Get influxDB2-0 token
```javascript=
cd aimlfw-dep/demos/hrelease/scripts
./get_access_tokens.sh
```
![](https://hackmd.io/_uploads/ByLZE-VeT.jpg)

Influx DB token : 
ExKq5RB2e5B0CCuxLRudl56VpNOC9SxG1S96hhqE2ip3RLRdERBItFcupj1Z-zZCSqYibrn0TgnNygtwoOu_Ag==
### 4-3. Update the RECIPE file of AIMLFW installation with Influx DB details of RANPM setup.

```javascript=
cd
cd aimlfw-dep/RECIPE_EXAMPLE
vim example_recipe_latest_stable.yaml
```

Revise the file as the below figure.

![](https://hackmd.io/_uploads/SJe3Db4g6.jpg)

Reinstall the training host.

```javascript=
cd ..
cd bin/
./uninstall.sh
cd ..
bin/install.sh -f RECIPE_EXAMPLE/example_recipe_latest_stable.yaml
```
![](https://hackmd.io/_uploads/SkGpZzEeT.jpg)

Execute the below script.

```javascript=
cd
cd aimlfw-dep/demos/hrelease/scripts
./prepare_env_aimlfw_access.sh
```

![](https://hackmd.io/_uploads/SkolQGNlT.jpg)

### 4-4. Add Group Feature from AIMLFW dashboard.

Use Enternet to enter the AIMLFW dashboard by using the following url.

```javascript=
http://<Your VM IP>:32005/
```

![](https://hackmd.io/_uploads/SJpE947an.png)

#### 4-4-1. Create Group Feature

![](https://hackmd.io/_uploads/ByXmrMNe6.png)

- **Feature Group Name :** fggnb130601
- **Features :** pdcpBytesDl,pdcpBytesUl

![](https://hackmd.io/_uploads/By7ltGNla.png)

- **DME Host :** 192.168.8.44
- **Bucket Name :** pm-bucket
- **Source Name :** gnb130601
- **DME Port :** 31823
- **DB Token :** ExKq5RB2e5B0CCuxLRudl56VpNOC9SxG1S96hhqE2ip3RLRdERBItFcupj1Z-zZCSqYibrn0TgnNygtwoOu_Ag==
- **Db Org :** est
- **Measured Obj Class :** Measured 1

![](https://hackmd.io/_uploads/BJf7gQNla.png)

Select List Feature Group of menu to check the group is correct or not. 

![](https://hackmd.io/_uploads/H1YOpfEeT.png)

### 4-5. Push qoe data into ranpm setup.

#### 4-5-1. **Method 1 :**

Execute below script to push qoe data into ranpm setup.

```javascript=
./push_qoe_data.sh  <source name mentioned when creating feature group> <Number of rows> <Cell Identity>
```

For example :

```javascript=
./push_qoe_data.sh gnb130601 30 c4/B2
```
Waining for the insert.

![](https://hackmd.io/_uploads/rJTTWQVxa.jpg)

Steps to check if data is upload correctly.

```javascript=
kubectl exec -it influxdb2-0 -n nonrtric -- bash
influx query 'from(bucket: "pm-bucket") |> range(start: -1000000000000000000d)' |grep pdcpBytesDl
```

![](https://hackmd.io/_uploads/SyGYbaLl6.png)

But this method to check the data is not success, below is the debug process.

##### 4-5-1-1. Trace below code to understand how's the data transmit.

**1. push_qoe_data.sh**
```javascript=
#!/bin/bash

if [ $# -lt 3 ]; then
    echo "Give all input parameters, e.g ./push_qoe_data.sh <source name> <max number of rows to take from csv> <cell Identity to be filtered>"
    exit 1
fi

kubectl patch StatefulSet pm-https-server -n ran -p '{"spec":{"template":{"spec":{"containers":[{"name":"pm-https-server", "env":[{"name":"ALWAYS_RETURN", "value":""}]}]}}}}'
kubectl rollout status statefulset/pm-https-server -n ran
kubectl exec -it pm-https-server-0 -n ran -c pm-https-server -- mkdir -p /files
sudo apt install -y python3-pip
pip3 install pandas
wget https://raw.githubusercontent.com/o-ran-sc/ric-app-qp/g-release/src/cells.csv -O qoedata.csv
python3 qoedatapush.py $1 $2 $3

```
**2. qoedatapush.py**

```javascript=
from xml.dom import minidom, Node
import pandas as pd
import os  
import datetime
import subprocess
import gzip
import sys

#Function to create pm file based on qoe data file
def create_xml_document(measurement_list, row, dir_name, sourcename,index):

    time = row['measTimeStampRf']
    du = str(row['du-id'])
    cell = row['nrCellIdentity'].replace('/','_')
    print(time,du,cell)
    doc = minidom.Document()

    measCollecFile = doc.createElement('measCollecFile')
    measCollecFile.setAttribute("xmlns","http://www.3gpp.org/ftp/specs/archive/32_series/32.435#measCollec")
    measCollecFile.setAttribute("xmlns:xsi","http://www.w3.org/2001/XMLSchema-instance")
    measCollecFile.setAttribute("xsi:schemaLocation","http://www.3gpp.org/ftp/specs/archive/32_series/32.435#measCollec")
    doc.appendChild(measCollecFile)

    pi = doc.createProcessingInstruction('xml-stylesheet',
                                         'type="text/xsl" href="MeasDataCollection.xsl"')
    root = doc.firstChild
    doc.insertBefore(pi, root)

    fileHeader = doc.createElement('fileHeader')
    fileHeader.setAttribute('fileFormatVersion',"32.435 V10.0")
    fileHeader.setAttribute('vendorName',"vendor A")
    fileHeader.setAttribute('dnPrefix',"SubNetwork=XX")
    measCollecFile.appendChild(fileHeader)


    fileSender = doc.createElement('fileSender')
    fileSender.setAttribute('localDn',"test")
    fileSender.setAttribute('elementType',"RadioNode")
    fileHeader.appendChild(fileSender)

    measCollec = doc.createElement('measCollec')
    measCollec.setAttribute('beginTime',time)
    fileHeader.appendChild(measCollec)

    measData = doc.createElement('measData')
    measCollecFile.appendChild(measData)


    managedElement = doc.createElement('managedElement')
    managedElement.setAttribute('localDn',"test")
    managedElement.setAttribute('swVersion',"testversion")
    measData.appendChild(managedElement)


    measInfo = doc.createElement('measInfo')
    measInfo.setAttribute('measInfoId',"PM=1,PmGroup=NRCellDU_GNBDU")
    measData.appendChild(measInfo)


    job = doc.createElement('job')
    job.setAttribute('jobId',"nr_all")
    measInfo.appendChild(job)

    granPeriod = doc.createElement('granPeriod')
    granPeriod.setAttribute('duration',"PT900S")
    granPeriod.setAttribute('endTime',time)
    measInfo.appendChild(granPeriod)

    repPeriod = doc.createElement('repPeriod')
    repPeriod.setAttribute('duration',"PT900S")
    measInfo.appendChild(repPeriod)

    measurement_index = 1
    for column in row.index:
        if column in measurement_list:
            measType = doc.createElement('measType')
            measType.setAttribute('p',str(measurement_index))
            measTypeValue = doc.createTextNode(column)
            measType.appendChild(measTypeValue)
            measInfo.appendChild(measType)

            measurement_index = measurement_index + 1

    measValue = doc.createElement('measValue')
    measValue.setAttribute('measObjLdn',"ManagedElement=nodedntest,GNBDUFunction="+ du +",NRCellDU="+ cell)
    measInfo.appendChild(measValue)

    measurement_index = 1
    for column in row.index:
        if column in measurement_list:
            r = doc.createElement('r')
            r.setAttribute('p',str(measurement_index))
            measTypeValue = doc.createTextNode(str(row[column]))
            r.appendChild(measTypeValue)
            measValue.appendChild(r)

            measurement_index = measurement_index + 1
    
    xmlstr = doc.toprettyxml(encoding="utf-8")
    
    format_string = "%Y-%m-%dT%H:%M:%S.%f"
    date_string = time
    datetimestart = datetime.datetime.strptime(date_string, format_string)
    datetimeend = datetimestart + datetime.timedelta(milliseconds=10)
    

    datetime_start_formatted = datetimestart.strftime("%Y%m%d.%H%M")
    datetime_end_formatted = datetimeend.strftime("%H%M")
    secondsfiltered = datetimestart.strftime("%S%f")[:-3]

    filename = dir_name + "/" "A" + datetime_start_formatted + "+0200"+ "-" + datetime_end_formatted + "+0200"+ "_"+ sourcename + "_" + du +"_"+ cell+"_"+ secondsfiltered +".xml"

    with open(filename, "wb") as f:
        f.write(xmlstr)

    #Fix to increment timestamp in seconds
    datetimestart_in_epoch = int(datetimestart.timestamp()*1000 * 1000) + (index *1000 * 1000)
    datetimeend_in_epoch = int(datetimeend.timestamp()*1000 * 1000) + (index *1000 * 1000)
    
    print(datetimeend_in_epoch)
    filename_gz = filename + '.gz'
    f_in = open(filename, 'rb')
    f_out = gzip.open(filename_gz, 'wb')
    f_out.writelines(f_in)
    f_out.close()
    f_in.close()
    return datetimestart_in_epoch,datetimeend_in_epoch,filename_gz

#Function to copy generated files to http_server    
def copy_files_http_server(filename_gz):
    copy_command = ["kubectl", "cp", filename_gz ,"ran/pm-https-server-0:/files", "-c", "pm-https-server"]
    print(subprocess.run(copy_command, capture_output=True))

#Function to push file ready event for each row
def push_file_ready_event(sourcename, filename_gz, datetimestart_in_epoch, datetimeend_in_epoch):
    push_file_ready_event_command = ["./push-to-file-ready-topic-qoe.sh",sourcename,filename_gz,str(datetimestart_in_epoch),str(datetimeend_in_epoch)]
    print(subprocess.run(push_file_ready_event_command, capture_output=True))

#Read input data csv file
if (len(sys.argv) < 4):
    sys.exit('Give all input parameters, e.g. python3 qoedatapush.py <source name> <max number of rows to take from csv> <cell Identity to be filtered>')
dir_name='files'
sourcename = sys.argv[1]
max_rows= int(sys.argv[2])
print('max_rows: ',max_rows)
filtered_cell = sys.argv[3]
print('filtered_cell:',filtered_cell)
if not os.path.exists(dir_name):
    os.makedirs(dir_name)

measurement_list = ['throughput','x','y','availPrbDl','availPrbUl','measPeriodPrb','pdcpBytesUl','pdcpBytesDl','measPeriodPdcpBytes']
df = pd.read_csv('qoedata.csv')
df_selected = df
df_selected = df_selected[df_selected['nrCellIdentity'] == filtered_cell]
df_selected = df_selected.head(max_rows)
print(df_selected.size)
for index,row in df_selected.iterrows():
    datetimestart_in_epoch,datetimeend_in_epoch,filename_gz = create_xml_document(measurement_list, row, dir_name, sourcename, index)
    filename_wo_dir_gz = filename_gz.replace(dir_name+'/','')
    copy_files_http_server(filename_gz)
    push_file_ready_event(sourcename, filename_wo_dir_gz, datetimestart_in_epoch, datetimeend_in_epoch)
```
**3. push-to-file-ready-topic-qoe.sh**

```javascript=
NODE_NAME_BASE=$1
FILENAME=$2
START_EPOCH=$3
END_EPOCH=$4

chmod +x kafka-client-send-file-ready-qoe.sh
kubectl cp kafka-client-send-file-ready-qoe.sh nonrtric/kafka-client:/home/appuser -c kafka-client

kubectl exec kafka-client -c kafka-client -n nonrtric -- bash -c './kafka-client-send-file-ready-qoe.sh '$NODE_NAME_BASE' '$FILENAME' '$START_EPOCH' '$END_EPOCH' '

echo "done"
```
According to the qoedatapush.py use this command to check if the .xml file is correct to copy in the HTTP Server or not.

```javascript=
kubectl exec -it pm-https-server-0 -n ran -- bash
cd files 
ls
```
![](https://hackmd.io/_uploads/By2Dy0ieT.jpg)

According to the push-to-file-ready-topic-qoe.sh file to check the kafka-client has the command or not.

```javascript=
kubectl exec -it kafka-client -n nonrtric -- bash
ls
```

![](https://hackmd.io/_uploads/ryQ5QRjg6.jpg)

#### 4-5-2. **Method 2 : Use .csv file to insert the data in the influxDB**

```javascript=
cd aimlfw-dep/demos/hrelease/scripts/
```
You will see the file as below.

**1. qoedata.csv**
```javascript=
1013,2021-06-11T05:40:33.690,c13/N77,0.3,1664.0,-1109.0,197,197,10,390.1440000000003,390.1440000000003,10.0
1001,2021-06-11T05:40:33.700,c1/B2,0.0,0.0,0.0,120,120,10,0.0,0.0,10.0
1001,2021-06-11T05:40:33.700,c1/B13,0.366513990004023,0.0,0.0,1,1,10,478.4307381953281,478.4307381953281,10.0
1001,2021-06-11T05:40:33.700,c1/N77,0.0,0.0,0.0,273,273,10,0.0,0.0,10.0
1002,2021-06-11T05:40:33.700,c2/B2,0.0,-832.0,-555.0,120,120,10,0.0,0.0,10.0
1002,2021-06-11T05:40:33.700,c2/B13,0.0,-832.0,-555.0,91,91,10,0.0,0.0,10.0
1002,2021-06-11T05:40:33.700,c2/N77,0.0,-832.0,-555.0,273,273,10,0.0,0.0,10.0
1003,2021-06-11T05:40:33.700,c3/B2,0.0,-139.0,-1109.0,120,120,10,0.0,0.0,10.0
1003,2021-06-11T05:40:33.700,c3/B13,0.386118816335772,-139.0,-1109.0,0,0,10,506.0936549476242,506.0936549476242,10.0
...
```
**2. pushdata.py**
```javascript=
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS
import csv
data =[]


# Parameters of InfluxDB instance
myorg = "est"
mybucket = "pm-bucket"
mytoken = "ExKq5RB2e5B0CCuxLRudl56VpNOC9SxG1S96hhqE2ip3RLRdERBItFcupj1Z-zZCSqYibrn0TgnNygtwoOu_Ag=="
myurl="http://192.168.8.44:31812"

with open('qoedata.csv', 'r') as csvfile:
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


for i in range(1, 5001):
    # Data (Point structure)
    _point = Point("ManagedElement=nodedntest")\
            .tag("GNBDUFunction", data[i][0])\
            .tag("NRCellDU", data[i][2])\
            .field("throughput", data[i][3])\
            .field("x", data[i][4])\
            .field("y", data[i][5])\
            .field("availPrbDl", data[i][6])\
            .field("availPrbUl", data[i][7])\
            .field("measPeriodPrb", data[i][8])\
            .field("pdcpBytesUl", data[i][9])\
            .field("pdcpBytesDl", data[i][10])\
            .field("measPeriodPdcpBytes", data[i][11])\
            .time(data[i][1])
            # Write the data to influxDB        
    write_api.write(bucket=mybucket, record=[_point])

print("Finish writing 5000 metircs")
```

Use this command to insert data.

```javascript=
pyhton3 pushdata.py
```

Check if the pm-bucket has the data or not (Use UI).

```javascript=
192.168.8.44:31812
```

![](https://hackmd.io/_uploads/HJFTtynea.jpg)

```javascript=
from(bucket: "pm-bucket")
    |> range(start: -3y)
    |> filter(fn: (r) => r["_measurement"] == "ManagedElement=nodedntest")
    |> filter(fn: (r) => r["_field"] == "throughput")
```
Check if the pm-bucket has the data or not (inter InfluxDB).

```javascript=
kubectl exec -it influxdb2-0 -n nonrtric -- bash
influx query 'from(bucket: "pm-bucket") |> range(start: -1000000000000000000d)'
```

![](https://hackmd.io/_uploads/B1yrhJ2ep.jpg)

#### 4-5-3. Method 3 : Use .csv file to insert rictest data in the influxDB

1. inputData.py

This is all the field of rictest data.
```javascript=
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS
import csv
data =[]

# Parameters of InfluxDB instance
myorg = "influxdata"
mybucket = "RicTest-Data"
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

# timestamp,Viavi.Cell.Name,Viavi.isEnergySaving,DRB.UEThpDl,DRB.UEThpUl,RRU.PrbUsedDl,RRU.PrbUsedUl,RRU.PrbAvailDl,RRU.PrbAvailUl,RRU.PrbTotUl,RRU.PrbTotDl,RRU.MaxLayerDlMimo,CARR.AverageLayersDl,Viavi.Cell.AverageBeamsDl,RRC.ConnMean,RRC.ConnMax,QosFlow.TotPdcpPduVolumeUl,QosFlow.TotPdcpPduVolumeDl,PEE.AvgPower,PEE.Energy,Viavi.PEE.EnergyEfficiency,Viavi.Radio.power,Viavi.Radio.antennaType,Viavi.Radio.azimuth,Viavi.Geo.x,Viavi.Geo.y,Viavi.Geo.z,Viavi.GnbDuId,Viavi.NrPc,Viavi.NrCg,_FrameCnt,_Ns3FrameCnt,RRC.ConnEstabAtt.mo-Data,RRC.ConnEstabAtt.mo-VoiceCall,RRC.ConnEstabAtt.mo-VideoCall,RRC.ConnEstabSucc.mo-Data,RRC.ConnEstabSucc.mo-VoiceCall,RRC.ConnEstabSucc.mo-VideoCall,RRC.ConnEstabFailCause.NetworkReject,Viavi.QoS.Score
for i in range(1, 15653):
    # Data (Point structure)
    _point = Point("ManagedElement=liveCell")\
            .tag("Viavi.Cell.Name", data[i][1])\
            .field("Viavi.isEnergySaving", data[i][2])\
            .field("DRB.UEThpDl", data[i][3])\
            .field("DRB.UEThpUl", data[i][4])\
            .field("RRU.PrbUsedDl", data[i][5])\
            .field("RRU.PrbUsedUl", data[i][6])\
            .field("RRU.PrbAvailDl", data[i][7])\
            .field("RRU.PrbAvailUl", data[i][8])\
            .field("RRU.PrbTotUl", data[i][9])\
            .field("RRU.PrbTotDl", data[i][10])\
            .field("RRU.MaxLayerDlMimo", data[i][11])\
            .field("CARR.AverageLayersDl", data[i][12])\
            .field("Viavi.Cell.AverageBeamsDl", data[i][13])\
            .field("RRC.ConnMean", data[i][14])\
            .field("RRC.ConnMax", data[i][15])\
            .field("QosFlow.TotPdcpPduVolumeUl", data[i][16])\
            .field("QosFlow.TotPdcpPduVolumeDl", data[i][17])\
            .field("PEE.AvgPower", data[i][18])\
            .field("PEE.Energy", data[i][19])\
            .field("Viavi.PEE.EnergyEfficiency", data[i][20])\
            .field("Viavi.Radio.power", data[i][21])\
            .field("Viavi.Radio.antennaType", data[i][22])\
            .field("Viavi.Radio.azimuth", data[i][23])\
            .field("Viavi.Geo.x", data[i][24])\
            .field("Viavi.Geo.y", data[i][25])\
            .field("Viavi.Geo.z", data[i][26])\
            .field("Viavi.GnbDuId", data[i][27])\
            .field("Viavi.NrPc", data[i][28])\
            .field("Viavi.NrCg", data[i][29])\
            .field("_FrameCnt", data[i][30])\
            .field("_Ns3FrameCnt", data[i][31])\
            .field("RRC.ConnEstabAtt.mo-Data", data[i][32])\
            .field("RRC.ConnEstabAtt.mo-VoiceCall", data[i][33])\
            .field("RRC.ConnEstabAtt.mo-VideoCall", data[i][34])\
            .field("RRC.ConnEstabSucc.mo-Data", data[i][35])\
            .field("RRC.ConnEstabSucc.mo-VoiceCall", data[i][36])\
            .field("RRC.ConnEstabSucc.mo-VideoCall", data[i][37])\
            .field("RRC.ConnEstabFailCause.NetworkReject", data[i][38])\
            .field("Viavi.QoS.Score", data[i][39])\
            .time(data[i][0])
            # Write the data to influxDB        
    write_api.write(bucket=mybucket, record=[_point])

print("Finish writing 15652 metircs")
```

Some field we will not use, so you can revise like this.


```javascript=
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS
import csv
data =[]

# Parameters of InfluxDB instance
myorg = "influxdata"
mybucket = "RicTest-Data"
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

You can use this code to convert timestate to UTC_format.

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

After invert the timestamp format. You will see the result like this.

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
...
```

For more detail, you can follow this [study note.](https://hackmd.io/@Jerry0714/SyeDfzfNT)

### 4-6. Revised training function.(Reference [#3-2-1 Create-Training-function](#3-2-1-Create-Training-function))

Revise the parameter like this figure. (Procedure **In [2], [5], [7]**)

![](https://hackmd.io/_uploads/S1SlNE3e6.png)
![](https://hackmd.io/_uploads/S1-EBVnxa.png)
![](https://hackmd.io/_uploads/ryZEIVnla.png)

### 4-7. Use parmeter to training model. (Reference [#3-2-2 Create-Training-job](#3-2-2-Create-Training-job))

![](https://hackmd.io/_uploads/Bk8MuE2e6.png)

Back to the menu to select the training job status to check is the model complete or not.

![](https://hackmd.io/_uploads/HJCwtNhep.png)

### 4-8. Test QoE prediction (Reference [#3-2-3 Deploy-trained-qoe-prediction-model-on-Kserve](#3-2-3-Deploy-trained-qoe-prediction-model-on-Kserve))

Create new qoe-model file.
```javascript=
cd
vim qoe2.yaml
```

Revise the **"storageUri"** as below.

```javascript=
apiVersion: "serving.kserve.io/v1beta1"
kind: "InferenceService"
metadata:
  name: qoe-model
spec:
  predictor:
    tensorflow:
      storageUri: "http://192.168.8.44:32002/model/qoetest2/1/Model.zip"
      runtimeVersion: "2.5.1"
      resources:
        requests:
          cpu: 0.1
          memory: 0.5Gi
        limits:
          cpu: 0.1
          memory: 0.5Gi

```

#### 4-8-1. Test prediction.

```javascript=
source predict.sh
```

Result.

![](https://hackmd.io/_uploads/rJ5_RI2gp.jpg)
