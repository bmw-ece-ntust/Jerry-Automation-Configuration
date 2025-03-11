# <center>O1ves Installation Guide</center>

<center>Testing and building source code in local</center>

###### tags: `SMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Oct14 , 2024 3:38 AM][color=#38c16a]

:::info
**Introduction：**

This note is teach you how to build O1ves source code using docker, and upload to chartmuseum to install in kubernetes by helm.

**Goal：**
- [x] Complete the implementation of O1ves.

**Reference**
- [Deployment of dmaap-InfluxDB-adapter, InfluxDB and Grafana](https://hackmd.io/@H131413/ByOoZCmDa#Deployment-of-dmaap-InfluxDB-adapter-InfluxDB-and-Grafana)
- [RIC Test & [I]SMO Integration(O1-Ves)](https://hackmd.io/@Winnie27/r1uReJjxp)
:::
**<center>Table of Content</center>**
[TOC]

## 1. Preparing source code and upload to server

**Source code file givend by NYCU**

![Screenshot 2024-10-14 at 5.23.21 AM](https://hackmd.io/_uploads/SJywc2YJyx.png)

Upload to remote server.

```javascript=
sftp <Your server name>@<Your server ip>

put dmaap-influxdb-adapter-0.1.0.tar.gz
put winlab-smo-charts-master.tar.gz
mv dmaap-influxdb-adapter-0.1.0.tar.gz /root/teiv/charts/chartstorage
mv winlab-smo-charts-master.tar.gz /root/teiv/charts/chartstorage
tar -xzvf dmaap-influxdb-adapter-0.1.0.tar.gz
tar -xzvf winlab-smo-charts-master.tar.gz
```

or you can use GitHub to download the source code.
GitHub Link: https://github.com/jerry133/SMO-o1ves.git
## 2. Building image
Installing docker.
```javascript=
sudo apt update
sudo apt install docker.io
docker --version
```
Building image in dmaap-influxdb-adapter-0.1.0 folder
```javascript=
cd dmaap-influxdb-adapter-0.1.0
docker build -t dmaap-influxdb-adapter:0.1.0 .
```
Checking images in docker.
```javascript=
docker images
```
**Result:**
![Screenshot 2024-10-14 at 5.39.40 AM](https://hackmd.io/_uploads/SJJNAntyke.png)

login your docker account.
```javascript=
docker login
```
![Screenshot 2024-10-14 at 5.53.35 AM](https://hackmd.io/_uploads/HJVuWTFJ1g.png)

Push image in to your docker hub.

```javascript=
docker tag dmaap-influxdb-adapter:0.1.0 jerry900714/dmaap-influxdb-adapter:0.1.0
docker push jerry900714/dmaap-influxdb-adapter:0.1.0
```

Check in your docker hub.
![Screenshot 2024-10-14 at 6.00.52 AM](https://hackmd.io/_uploads/B1pIQTKJye.png)

## 3. Pushing helm chart to chartmuseum
Make sure the following tool have been installed in your server.
- ChartMuseum v0.16.1
- helm-push  v0.10.3
- helm v3.12.3

Chartmuseum already run.
```javascript=
ps aux | grep chartmuseum
```

![Screenshot 2024-10-14 at 9.31.21 AM](https://hackmd.io/_uploads/B1ytEg91Je.png)

Revise image repository link in values.yaml
```javascript=
cd teiv/charts/chartstorage/winlab-smo-charts-master/charts/o1ves-visualization
vim values.yaml
```
**Original repository:** harbor.winfra.cs.nycu.edu.tw/winlab-oran/dmaap-influxdb-adapter
**New repository:** jerry900714/dmaap-influxdb-adapter

![Screenshot 2024-10-14 at 6.18.08 AM](https://hackmd.io/_uploads/Hy24P6YkJe.png)

```javascript=
helm dependency update .
helm package .
helm cm-push . local
helm repo update
helm search repo local
```
![Screenshot 2024-10-14 at 6.46.21 AM](https://hackmd.io/_uploads/ryJRppKk1x.png)

## 4. Installing O1ves by using helm cahrt

```javascript=
helm install --namespace=o1ves o1ves local/o1ves-visualization --create-namespace
```

Checking image in o1ves-dmaap-influxdb-adapter.

```javascript=
kubectl describe pod o1ves-dmaap-influxdb-adapter-67c69fcc6f-j9xq8 -n o1ves
```

![Screenshot 2024-10-14 at 9.46.29 AM](https://hackmd.io/_uploads/B1HQ_xcyJl.png)

**Result**
![Screenshot 2024-10-14 at 4.50.05 AM](https://hackmd.io/_uploads/B1sTEecJJg.png)

**Notice!** The CrashLoopBackOff is a normal result. Because this environment is not in SMO so it can't connect to message-router.onap

**Notice!** If you every time reinstall, you need to use the following procedure to clean hole resources.

:::danger
**Notice!** 
Use this command is very dangerous. You will delete all of the resource including the secret, data and dependencies.
```javascript=
sudo rm -r /dockerdata-nfs/o1ves-*
```
:::
```javascript=
helm uninstall o1ves -n o1ves
kubectl delete namespace o1ves
sudo rm -r /dockerdata-nfs/o1ves-*
```
Otherwise you will face this problem.
:::spoiler config error
:::danger
![Screenshot 2024-10-14 at 7.03.09 AM](https://hackmd.io/_uploads/ryTHKdDxke.png)
:::

The more safety solution you can refer to [6. Sending PM data to influxDB through influxdb-adapter](#6-Sending-PM-data-to-influxDB-through-influxdb-adapter) authorization not found error solution.
## 5. Pushing your helm chart to docker hub (Optional)
```javascript=
cd teiv/charts/chartstorage/winlab-smo-charts-master/charts/o1ves-visualization
docker login docker.io
helm registry login registry-1.docker.io
helm push o1ves-visualization-0.1.0.tgz oci://registry-1.docker.io/jerry900714
```

**Result**
![Screenshot 2024-10-23 at 10.33.14 PM](https://hackmd.io/_uploads/S1bHtYLxke.png)


**Installation command**
```javascript=
helm install o1ves --namespace=o1ves oci://registry-1.docker.io/jerry900714/o1ves-visualization --create-namespace
```
:::warning
**Notice!!**
Don't using this command, otherwise your secrets will not found.
```javascript=
helm install --generate-name --namespace=o1ves oci://registry-1.docker.io/jerry900714/o1ves-visualization --create-namespace
```
And you will face this problem.

![Screenshot 2024-09-17 at 2.28.29 AM](https://hackmd.io/_uploads/SJoB_ODg1l.png)
:::

**Result**

![Screenshot 2024-10-24 at 3.30.02 PM](https://hackmd.io/_uploads/Hk1qvdwgkx.png)

## 6. Sending PM data to influxDB through influxdb-adapter
```javascript=
vim test1.yaml
```
```javascript=
dmaap-influxdb-adapter:
  image:
    pullPolicy: Always
  logLevel: DEBUG
  rules:
    - topic: unauthenticated.SEC_3GPP_PERFORMANCEASSURANCE_OUTPUT
      rules:
        - bucket: Test1
          measurement: o-ran-pm
          matches:
            - key: event.perf3gppFields.measDataCollection.measuredEntityUserName
              value: CellReports
          tags:
            - key: vendor
              value: ntust-taiwan-lab
            - key: sourceName
              field: event.commonEventHeader.sourceName
          fields:
            - key: CellID
              field: event.perf3gppFields.measDataCollection.measInfoList[0].measValuesList[0].measObjInstId
              type: string
            - key: DRB.UEThpDl
              field: event.perf3gppFields.measDataCollection.measInfoList[0].measValuesList[0].measResults[0].sValue
              type: float
            - key:  DRB.UEThpUl
              field: event.perf3gppFields.measDataCollection.measInfoList[0].measValuesList[0].measResults[1].sValue
              type: float
```
```javascript=
sudo helm upgrade o1ves --namespace=o1ves oci://registry-1.docker.io/jerry900714/o1ves-visualization --create-namespace -f test1.yaml
```
Check the upload .yaml file in o1ves.
```javascript=
helm get values o1ves -n o1ves
```

![Screenshot 2024-11-21 at 3.46.40 PM](https://hackmd.io/_uploads/SJLOrvnMyg.png)

Check the data in influxDB.

![Screenshot 2024-11-21 at 3.08.12 PM](https://hackmd.io/_uploads/ry0uLD2Mkx.png)

```javascript=
from(bucket: "Test1")
    |> range(start: -3y)
    |> filter(fn: (r) => r["_measurement"] == "o-ran-pm")
```

If you can't sending the data try to use this method.
:::spoiler ERROR msg=Unauthorized error="authorization not found"
:::danger
**ERROR msg=Unauthorized error="authorization not found"**

![Screenshot 2024-11-21 at 12.11.03 AM](https://hackmd.io/_uploads/ry4t5vhGJe.png)

This error means that your password and token in secret "o1ves-influxdb2-auth" is wrong.

Try to use this command to config the new token and password.
```javascript=
kubectl create secret generic o1ves-influxdb2-auth -n o1ves \
--from-literal=admin-password='Your-password' \
--from-literal=admin-token='Your-token' \
--dry-run=client -o yaml | kubectl apply -f -
```
Restart the o1ves-influxdb2
```javascript=
kubectl rollout restart statefulset/o1ves-influxdb2 -n o1ves
```
Check the new password.
```javascript=
kubectl get secret o1ves-influxdb2-auth -n o1ves -o jsonpath='{.data.admin-password}' | base64 -d
```
Check the new token.
```javascript=
kubectl get secret o1ves-influxdb2-auth -n o1ves -o jsonpath='{.data.admin-token}' | base64 -d
```
Restart the influxdb-adapter.
```javascript=
kubectl delete o1ves-dmaap-influxdb-adapter-57cd76c56-j9vbg -n o1ves
```
:::
## Appendix: A
:::success
**Notice!** 

Not just placing charts in chartstorage allows uploading to ChartMuseum. ChartMuseum will automatically generate charts and download them to chartstorage based on the index-cache.yaml file in chartstorage, keeping ChartMuseum and index-cache.yaml synchronized.
:::
- Start your chartmuseum
```javascript=
cd smo-nearrt-autodep/dep/
sudo chartmuseum --port=18080 --storage=local --storage-local-rootdir=./chartstorage
```
- Add the repository
```javascript=
helm repo add local http://127.0.0.1:18080 
helm repo update
```
- Upload the only chart file to local chartmuseum.
```javascript=
curl --data-binary "@o1ves-visualization-0.1.0.tgz" http://localhost:18080/api/charts
helm repo update
```
- Update dcaegen2 service .yaml file
```javascript=
helm upgrade -f dcaegen2.values.yml -n onap onap-dcaegen2-services local2/dcaegen2-services
```

## Appendix: B 
This appendix teach you how to upload your source code to GitHub account in ==first time==.

- Cleaning git
```javascript=
rm -rf .git
rm .gitattributes
rm .gitignore
```
- Initial git
```javascript=
git init
```
- Setting your git
```javascript=
git config --global user name "jerry133"
git config --global user.email "jerry2880752@gmail.com"
```
- If you want to ignore some file you can use this command.
```javascript=
echo "protobuf-c/" > gitignore
```
- Adding your GitHub repository
```javascript=
git remote add origin https://github.com/jerry133/0AI-telnet-01-Slice-gNB.git
```
- Check your repository
```javascript=
git remote -V
```
- Upload your source code.
```javascript=
git add.
git commit -m "first commit"
git branch -M main
git push -f origin main
```