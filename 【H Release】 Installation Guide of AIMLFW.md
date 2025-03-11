# 【H Release】 Installation Guide of AIMLFW
###### tags: `AI/ML`
![](https://i.imgur.com/JORnn3y.png =150x)@NTUST
>[name=Jerry][time=Tue, Sep 6, 2023 10:20 AM][color=#38c16a]
:::info
**Introduction：**

The AIMLFW (AI/ML Framework) is designed by OSC Samsung. If you want to training model to deploy, you need to install the platform first. 

**Goal：**
- [x] [Install AIMLFW in H-Release](#2-Software-Installation-and-Deployment)
- [x] [RAN PM installation.](#6-InstallationRANPM-Only)

**Reference：**

- [OSC samsung AI/ML installation guide.](https://docs.o-ran-sc.org/projects/o-ran-sc-aiml-fw-aimlfw-dep/en/h-release/installation-guide.html#installation-guide)
- [【H Release】 Installation Guide of Non-RT RIC](https://wiki.o-ran-sc.org/display/RICNR/Release+H+-+Run+in+Kubernetes)
- [【H Release】User Guide of AIMLFW](https://hackmd.io/@Jerry0714/HJmhjj-kT)
- [Integration of the AIMLFW, Non-RT RIC and SMO](https://hackmd.io/@Jerry0714/ryCs5xmep)
- [Non-RT RIC RAN PM Measurement](https://docs.o-ran-sc.org/projects/o-ran-sc-nonrtric-plt-ranpm/en/latest/overview.html)
:::
:::warning
**1. Hardware requests：**
- RAM：16 GB RAM
- CPU：8 cpu cores
- Disk：60 GB harddisk

**2. Installation Environment：**
- Host：Ubuntu 22.04 LTS 

**3. Platform Environment：**
- Kubernetes：
    - Client Version：v1.21.0
    - Server Version：v1.21.14
- Helm：v3.12.3
- Docker：
    - Client Version：v24.0.6
    - Server Version：v24.0.6
:::
**<center>Table of Content</center>**
[TOC]

## 1. AIMLFW Design diagram

![](https://hackmd.io/_uploads/SyzgkHzT2.png)

## 2. Software Installation and Deployment
### 2-1. SSH to the server

- **Server name：** aiml
- **Server IP：** 192.168.8.44
- **Password：** bmwlab

![](https://hackmd.io/_uploads/HkYp96NJ6.png)

### 2-2. Dawnload aimlfw file
```javascript=
sudo apt install git
git clone "https://gerrit.o-ran-sc.org/r/aiml-fw/aimlfw-dep"
reboot
```
```javascript=
cd aimlfw-dep
bin/install_traininghost.sh
```
After you complete installation, you may see the figure like this.

```javascript=
kubectl get pod -A
```

![](https://hackmd.io/_uploads/Sk9PDEQp2.png)

```javascript=
kubectl get svc --all-namespaces
```

![](https://hackmd.io/_uploads/rJlyu4mph.png)

Use Enternet to check the AIMLFW dashboard by using the following url.

```javascript=
http://<Your VM IP>:32005/
```

![](https://hackmd.io/_uploads/SJpE947an.png)

## 3. Install Influx DB as datalake
### 3-1. Install Influx DB and create bucket
```javascript=
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/influxdb
```

![](https://hackmd.io/_uploads/Bk34s4ma2.png)

Use this command to find your influxdb pod.

```javascript=
kubectl get pods -A
```

![](https://hackmd.io/_uploads/ryM1pNmTh.png)

After you find, use this command to in to the pod.

```javascript=
kubectl exec -it <pod name> bash
```

For example：

```javascript=
kubectl exec -it my-release-influxdb-5b77fc46b4-65f5x -- bash
```

![](https://hackmd.io/_uploads/BkIJCE762.png)

Cat the file that you can get username, org name, org id and access token.

```javascript=
cat bitnami/influxdb/influxd.bolt | tr -cd "[:print:]"
```

![](https://hackmd.io/_uploads/HkgKyBQTh.png)

![](https://hackmd.io/_uploads/Hygc1r7ah.png)

**<org_name>**：primary
**<API_Token>**：sA8sHWpiNZk4Tgkw1hT7

Execute below from inside Influx DB container to create a bucket:

```javascript=
influx bucket create -n UEData -o <org_name> -t <API_Token>
```

![](https://hackmd.io/_uploads/rJdv0FZxp.png)

You can check the all bucket by this command.

```javascript=
influx bucket list --org <org_name> --token <API_Token>
```

![](https://hackmd.io/_uploads/rJFqRY-ga.png)

### 3-2. Accessing Applications in the Cluster Using Port Forwarding to send data.
Install the following dependencies.

```javascript=
sudo apt-get install python3-pip
sudo pip3 install pandas
```

![](https://hackmd.io/_uploads/r1NpGSXT3.png)

```javascript=
sudo pip3 install influxdb_client
```

![](https://hackmd.io/_uploads/Hyq77SXT2.png)

Use the insert.py in ric-app/qp repository to upload the qoe data in Influx DB.

```javascript=
git clone -b f-release https://gerrit.o-ran-sc.org/r/ric-app/qp
cd qp/qp
```

Update **< token >** in insert.py file.

```javascript=
import pandas as pd
from influxdb_client import InfluxDBClient
from influxdb_client.client.write_api import SYNCHRONOUS
import datetime


class INSERTDATA:

   def __init__(self):
        self.client = InfluxDBClient(url = "http://localhost:8086", token="<token>")


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
     write_api.write(bucket="UEData",record=df, data_frame_measurement_name="liveCell",org="primary")

populatedb()
```

![](https://hackmd.io/_uploads/rJziLHm6n.png)

Follow below command to port forward to access Influx DB.

**Step 1：Check your influx service name and port**

```javascript=
kubectl get service -A
```

![](https://hackmd.io/_uploads/H1UDdrXa2.png)
    
**My influx service name**：my-release-influxdb
**My port**：8086/TCP,8088/TCP.
    
**Step 2：Open new terminal and follow below command to port forward to access Influx DB**

```javascript=
kubectl port-forward svc/<Your influxDB service name> 8086:<Your influxDB service port> -n ricplt
```
    
For example：
    
```javascript=
kubectl port-forward svc/my-release-influxdb 8086:8086
```

If successful you will get this informaton in your new terminal.

![](https://hackmd.io/_uploads/S1DIYr7pn.png)

**Step 3：Back to the terminal and run this command to insert data**

```javascript=
python3 insert.py
```
If you successful, you will get this information "Hadling connection for 8086"

![](https://hackmd.io/_uploads/ryjZ9rQpn.png)

To check inserted data in Influx DB , execute below command inside the Influx DB container:

**First**：You need to in to influxdb pod.

```javascript=
kubectl exec -it my-release-influxdb-575dfc7f65-hs8w2 bash
```

**Second：Check the data in the container.**
```javascript=
influx query  'from(bucket: "UEData") |> range(start: -1000d)' -o primary -t <Your token>
```
For example：
```javascript=
influx query  'from(bucket: "UEData") |> range(start: -1000d)' -o primary -t sA8sHWpiNZk4Tgkw1hT7
```
and you will see the information like this figure.

![](https://hackmd.io/_uploads/B1FHqpQT3.png)

<style>
  .img {
    position:relative;
    width: 50%;
    left:25%;
  }
</style>

## 4. Installing Kserve (Final Step)

To install Kserve run the below commands.

```javascript=
./bin/install_kserve.sh
```
If you success you will see like this figure and **complete the AIML installation**.

![](https://hackmd.io/_uploads/BJiBpvwAh.png)

## 5. Non-RT RIC installation. (Selection)

### 5-1. Prepare Non-RT RIC installation
```javascript=
git clone --recurse-submodules https://gerrit.o-ran-sc.org/r/it/dep.git
```
### 5-2. Setup Helm Charts
```javascript=
## Setup ChartMuseum
./dep/smo-install/scripts/layer-0/0-setup-charts-museum.sh

## Setup HELM3
./dep/smo-install/scripts/layer-0/0-setup-helm3.sh

## Build ONAP/ORAN charts
./dep/smo-install/scripts/layer-1/1-build-all-charts.sh
```
### 5-3. Deploy components
```javascript=
./dep/smo-install/scripts/layer-2/2-install-nonrtric-only.sh
```
### 5-4. Check if Non-RT RIC Component
```javascript=
kubectl get pod -A
```
![](https://hackmd.io/_uploads/ryl3vjO1p.png)

## 6. Installation(RANPM Only)
:::info
**About Ranpm：**

**Ranpm** is a component that is only available in the **latest version (H-Release Non-RT RIC)**.
:::

Before installation you need to build up below component to deploy on the kubernetes.

- https-server
- pm-rapp

Build https-server image.
```javascript=
cd dep/ranpm/https-server
./build.sh no-push
```
![](https://hackmd.io/_uploads/rJ_a6TAkT.png)

Build pm-rapp image.
```javascript=
cd dep/ranpm/pm-rapp
./build.sh no-push
```
![](https://hackmd.io/_uploads/rkae0pAy6.png)

Check if the image is create or not.
```javascript=
docker images
```

![](https://hackmd.io/_uploads/rkjnMuFJp.png)

Revise the app-deployment.yaml file.
```javascript=
cd dep/ranpm/install/helm/ran/templates
vim app-deployment.yaml
```

Revise the file **image: pm-https-server:latest #{{ .Values.global.extimagerepo }}/pm-https-server:latest** like this figure.

![](https://hackmd.io/_uploads/rkTXjEega.png)

Revise the app-pod.yaml file.

```javascript=
cd dep/ranpm/install/helm/nrt-pm-rapp/templates
vim app-pod.yaml
```

Looking for yor container to revise the file **"pm-rapp:latest #{{ .Values.global.extimagerepo }}/pm-rapp:latest"** 

![](https://hackmd.io/_uploads/rJKB4Belp.png)
### 6-1. Installs the main parts of the ranpm setup (Install-nrt.sh)

To make sure you have a **"istio"** and **"jq"** component.
```javascript=
cd dep/ranpm/install
./install-nrt.sh
```

### 6-2. Installs the producer for influx db (install-pm-log.sh)
```javascript=
./install-pm-log.sh
```

![](https://hackmd.io/_uploads/S1Ky3pAkp.png)

### 6-3. Sets up an alternative job to produce data stored in influx db. (install-pm-influx-job.sh)
```javascript=
./install-pm-influx-job.sh
```
![](https://hackmd.io/_uploads/rJKYnaAy6.png)

### 6-4. Installs a rapp that subscribe and print out received data. (install-pm-rapp.sh)
```javascript=
./install-pm-rapp.sh
```
### 6-5. Result (All pod is running)

![](https://hackmd.io/_uploads/SkPvnK-ep.png)
![](https://hackmd.io/_uploads/ryt2Gdgl6.png)
![](https://hackmd.io/_uploads/ryBK7_ela.png)

## 7. Problem set
:::warning
- [x] [Problem 1：The connection to the server 192.168.8.44:6443 was refused - did you specify the right host or port?](#7-1-Problem-1：6443-connection-refused-)
- [x] [Problem 2：InfluxDB datalake not available.](#7-2-Rroblem-2：InfluxDB-datalake-can’t-able-to-training-)
- [x] [Problem 3：Activator Pod keep in CrashLoopBackoff](#7-3-Rroblem-3：Activator-Pod-keep-in-CrashLoopBackoff-)
- [x] [Problem 4：Failed to allocate directory watch: Too many open files](#7-4-Problem-4：Failed-to-allocate-directory-watch-Too-many-open-files-)
- [x] [Problem 5：The kafka is keep waiting for other pod.](#7-5-Problem-5%EF%BC%9AThe-kafka-is-keep-waiting-for-other-pod-)
- [x] [Problem 6：The Non-RT RIC Installation can’t not use example_recipe.yaml to install.](#7-6-Problem-6：Non-RT-RIC-Installation-can’t-not-use-example_recipeyaml-to-install-)
- [x] [Problem 7：ml-pipeline Pod and ml-pipeline-persistenceagent keep in CrashLoopBackoff](#7-7-Problem-7：ml-pipeline-Pod-and-ml-pipeline-persistenceagent-keep-in-CrashLoopBackoff-)
- [x] [Problem 8：pm-https-server-0 pod InvalidImageName error](#7-8-Problem-8：pm-https-server-0-pod-InvalidImageName-error-)
- [x] [Problem 9：pm-rapp pod InvalidImageName error](#7-9-Problem-9：pm-rapp-pod-InvalidImageName-error-)
- [x] [Problem 10：influxdb can’t install by the helm.](#7-10-Problem-10：influxdb-can’t-install-by-the-helm-)
- [x] [Problem 11：Can't locate the aimlfw-common helm package in the local repo.](#7-11-Problem-11：Can’t-locate-the-aimlfw-common-helm-package-in-the-local-repo-)
- [ ] [Problem 12：External domains cannot use the AI/ML Dashboard to train models.](#7-12-Problem-12：External-domains-cannot-use-the-AI/ML-Dashboard-to-train-models)
:::
### 7-1. Problem 1：6443 connection refused ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)
When I restart the ubuntu,and type kubectl get pod -A,it always show in this figure.
You can refer to this website to address the issue. [[click here]](https://discuss.kubernetes.io/t/the-connection-to-the-server-host-6443-was-refused-did-you-specify-the-right-host-or-port/552)

![](https://hackmd.io/_uploads/S1JKXRKph.png)

:::spoiler Solution
:::success
**Resolved：**

And use below command to result the problem.
```javascript=
sudo -i
swapoff -a
exit
strace -eopenat kubectl version
```
:::
### 7-2. Rroblem 2：InfluxDB datalake can't able to training. ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)
If InfluxDB datalake not available you can follow the OSC sansung installation guide which have been mention the issue to be slove. [[click here]](https://docs.o-ran-sc.org/projects/o-ran-sc-aiml-fw-aimlfw-dep/en/h-release/installation-guide.html#software-installation-and-deployment)

:::spoiler Solution
:::success
**Resolved：**

**In case Influx DB datalake not available**, it can be installed using the steps mentioned in section Install Influx DB as datalake. Once installed the access details of the datalake can be updated in **RECIPE_EXAMPLE/example_recipe_latest_stable.yaml**. Once updated, follow the below steps for reinstall of some components:

Use this command to update your "ip_adress" and "datalake".
```javascript=
cd aimlfw-dep/
vim RECIPE_EXAMPLE/example_recipe_latest_stable.yaml
```
![](https://hackmd.io/_uploads/rJVtFiXTh.png)

Use this command to re-install.
```javascript=
bin/uninstall.sh
bin/install.sh -f RECIPE_EXAMPLE/example_recipe_latest_stable.yaml
```
If you successful you will see like this figure.

![](https://hackmd.io/_uploads/HyqW7_P02.png)
:::
### 7-3. Rroblem 3：Activator Pod keep in CrashLoopBackoff ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)

Activator Pod keep in CrashLoopBackoff. For detailed issues, please [[click here]](https://hackmd.io/@Jerry0714/Hk6MyGiph).

![](https://hackmd.io/_uploads/HkPWi54T3.png)

:::spoiler Solution
:::success
This is a **Kubernetes DNS issue**, and it may require modifying the "resolv.conf".

Use below command to revise DNS configuration.

```javascript=
sudo vim  /etc/systemd/resolved.conf
```

![](https://hackmd.io/_uploads/ry52X2pC3.png)

Revise this configuration.

![](https://hackmd.io/_uploads/rkT9V2pRn.png)

Use below command to restart.

Restart the systemd-resolved service.
```javascript=
sudo systemctl restart systemd-resolved
```
Start the systemd-resolved service
```javascript=
sudo systemctl enable systemd-resolved
```
Back up the systemd-resolved hosting file resolv.conf
```javascript=
sudo mv /etc/resolv.conf /etc/resolv.conf.bak
```
Rebuild the managed files
```javascript=
sudo ln -s /run/systemd/resolve/resolv.conf /etc/
```
Restart NetworkManager
```javascript=
sudo systemctl restart NetworkManager
```
![](https://hackmd.io/_uploads/r1hmr3pRh.png)

Choose the Wired to revise the DNS IP as 8.8.8.8 8.8.4.4

![](https://hackmd.io/_uploads/SyjSL3TRn.png)

Check the DNS configuration is revised or not.
```javascript=
cat /etc/resolv.conf
```

![](https://hackmd.io/_uploads/rkp7jVRR2.png)

Use this command to restart the PC.
```javascript=
reboot
```
Check the Activator pod is full running or not.

![](https://hackmd.io/_uploads/SklVE2NACh.png)

The activoator pod is ready and running.
:::
### 7-4. Problem 4：Failed to allocate directory watch: Too many open files ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)
If you use below this command.
```javascript=
sudo systemctl restart systemd-resolved
```
you might receieved this error **"Failed to allocate directory watch: Too many open files"**

Reference : [systemctl 无法启动新服务：lxc: Too many open files](https://zhuanlan.zhihu.com/p/222506941)
:::spoiler Solution
:::success
**Resolved：**
It's possible that the inotify limit was reached, and after making changes, the service can start properly.

Use below command to result the problem.
```javascript=
cd
vim /etc/sysctl.conf
```
Add to configuration to this file
```javascript=
fs.inotify.max_user_instances=512
fs.inotify.max_user_watches=262144
```
Reactivate
```javascript=
sysctl -p
```
:::
### 7-5. Problem 5：The kafka is keep waiting for other pod. ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)

When I Use this command to install nrt.
```javascript=
cd dep/ranpm/install
./install-nrt.sh
```

But kafka is keep waiting and still to retry.

![](https://hackmd.io/_uploads/Hk4aead1p.png)
![](https://hackmd.io/_uploads/BJ5Rgad16.png)

:::spoiler Solution
:::success
**Resolved：**

We need to update the file parameters because there are issues with the ones obtained from Git, and the error occur is about failed to resolve server.

![](https://hackmd.io/_uploads/rkSRnPgxp.png)

Enter the file and looking for the config.
```javascript=
cd dep/ranpm/install/helm/nrt-base-1/charts/strimzi-kafka/templates
```
![](https://hackmd.io/_uploads/SJAuCmggp.png)

Add new config in the file **"log.message.format.version: "3.3""**.

![](https://hackmd.io/_uploads/SJCkkVgg6.png)

Reuse the command to reinstall.
```javascript=
cd dep/ranpm/install
./install-nrt.sh
```

And you will keep running.

![](https://hackmd.io/_uploads/rJxyeExx6.png)

**Reference：**
[K8s - 使用Strimzi快速搭建Kafka全家桶教程1（安装配置、单节点样例）](https://www.hangge.com/blog/cache/detail_3099.html#)
[K8s - 使用Strimzi快速搭建Kafka全家桶教程2（集群部署、暴露外部访问端口)](https://www.hangge.com/blog/cache/detail_3100.html)
:::
### 7-6. Problem 6：Non-RT RIC Installation can't not use example_recipe.yaml to install. ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)

Follow the OSC Non-RT RIC installation guide, when I run this command **"sudo dep/bin/deploy-nonrtric -f dep/nonrtric/RECIPE_EXAMPLE/example_recipe.yaml"** to install, there must be a problem occure.

![](https://hackmd.io/_uploads/HyhfjGykT.png)

:::spoiler Solution
:::success
**Resolved：**

Use another install method to install Non-RT RIC.

**Step 1. Prepare Non-RT RIC installation**
```javascript=
git clone --recurse-submodules https://gerrit.o-ran-sc.org/r/it/dep.git
```
**Step 2. Setup Helm Charts**
```javascript=
## Setup ChartMuseum
./dep/smo-install/scripts/layer-0/0-setup-charts-museum.sh

## Setup HELM3
./dep/smo-install/scripts/layer-0/0-setup-helm3.sh

## Build ONAP/ORAN charts
./dep/smo-install/scripts/layer-1/1-build-all-charts.sh
```
**Step 3. Deploy components**
```javascript=
./dep/smo-install/scripts/layer-2/2-install-nonrtric-only.sh
```
**Step 4. Check if Non-RT RIC Component**
```javascript=
kubectl get pod -A
```
![](https://hackmd.io/_uploads/ryl3vjO1p.png)

:::
### 7-7. Problem 7：ml-pipeline Pod and ml-pipeline-persistenceagent keep in CrashLoopBackoff ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)

If you reboot the system, you might meet some problem like ml-pipeline Pod CrashLoopBackoff.

![](https://hackmd.io/_uploads/BkMNYXlg6.png)

:::spoiler Solution
:::success
**Resolved：**
Because the ml-pipeline can't check the minio bucket, and connection refuse to the leofs pod.

Check what data is loss.

![Screenshot 2023-11-15 at 8.23.46 AM](https://hackmd.io/_uploads/r1E4ecW4a.png)

Use this command to restart the leof.
```javascript=
cd aimlfw-dep/tools/leofs/bin
./install_leofs.sh
```
After you restart, use this command to delete ml-pipeline pod.

```javascript=
kubectl delete pod <your pod name> -n kubeflow 
```

Then your ml-pipeline is not crash anymore.

![](https://hackmd.io/_uploads/BkDxNQxea.png)

:::
### 7-8. Problem 8：pm-https-server-0 pod InvalidImageName error ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)

![](https://hackmd.io/_uploads/r1AG_Neea.png)

:::spoiler Solution
:::success
**Resolved：**

We need to update the file parameters because there are issues with the ones obtained from Git, the extimagerepo has no value.

```javascript=
cd dep/ranpm/install/helm
cat global-values.yaml
```
![](https://hackmd.io/_uploads/H1a8n4eeT.png)

Enter the file.
```javascript=
cd dep/ranpm/install/helm/ran/templates
vim app-deployment.yaml
```
Revise the file **image: pm-https-server:latest #{{ .Values.global.extimagerepo }}/pm-https-server:latest** like this figure.

![](https://hackmd.io/_uploads/rkTXjEega.png)

Use this command to uninstall ran.
```javascript=
helm ls -A
helm uninstall -n ran ran
```

![](https://hackmd.io/_uploads/rJ9c1BllT.png)

After uninstall the helm, reinstall the nrt again.

```javascript=
cd dep/ranpm/install
./install-nrt.sh
```

Then you can see the http-server keep running.

![](https://hackmd.io/_uploads/rJjM2Nglp.png)

:::
### 7-9. Problem 9：pm-rapp pod InvalidImageName error ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)

![](https://hackmd.io/_uploads/B126_7xe6.png)

:::spoiler Solution
:::success
**Resolved：**

We need to update the file parameters because there are issues with the ones obtained from Git.

```javascript=
cd dep/ranpm/install/helm/nrt-pm-rapp/templates
vim app-pod.yaml
```

looking for yor container to revise the file **"pm-rapp:latest #{{ .Values.global.extimagerepo }}/pm-rapp:latest"** 

![](https://hackmd.io/_uploads/rJKB4Belp.png)

After revise the file, uninstall the pm-rapp.
```javascript=
cd dep/ranpm/install
./uninstall-pm-rapp.sh
```

Use this command to reinstall.
```javascript=
./install-pm-rapp.sh
```

Check the pod is running or not.
```javascript=
kubectl get pod -A
```
![](https://hackmd.io/_uploads/H18BIBlea.png)

:::
### 7-10. Problem 10：influxdb can't install by the helm. ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)

![](https://hackmd.io/_uploads/rk9SMY-e6.png)

:::spoiler Solution
:::success
After searching the net, It is a helm version problem and we need to update the helm version to the latest.

Click below link to enter in the file **"helm/scripts/get-helm-3"** and copy https:// link like this figure.

[latest helm version 3.12.3](https://github.com/helm/helm/tree/v3.12.3)

![](https://hackmd.io/_uploads/HkoiVK-ep.png)

Past the link in the command and use this to updrade.
```javascript=
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/v3.12.3/scripts/get-helm-3
chmod 700 get_helm.sh
```
Reinstall the influxdb.
```javascript=
helm install my-release bitnami/influxdb
```
Result：

![](https://hackmd.io/_uploads/SyaNwY-ea.png)

:::
### 7-11. Problem 11：Can't locate the aimlfw-common helm package in the local repo. ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)

![](https://hackmd.io/_uploads/H1h5lqWxT.png)

:::spoiler Solution
:::success
**Resolved：**
There have something file missing. Use this command to reinstall.

```javascript=
cd aimlfw-dep/bin
./install_common_templates_to_helm.sh
```

![](https://hackmd.io/_uploads/HJCpyhfg6.jpg)

Check the chart is insatll or not.

```javascript=
helm search repo | grep aiml
```
 
![](https://hackmd.io/_uploads/rJOll3MxT.jpg)

Trying to reinstall the yaml.
```javascript=
bin/install.sh -f RECIPE_EXAMPLE/example_recipe_latest_stable.yaml
```
If you successful you will see like this figure.

![](https://hackmd.io/_uploads/HyqW7_P02.png)
:::
### 7-12. Problem 12：External domains cannot use the AI/ML Dashboard to train models. 

I have successfully trained the model, but it cannot be accessed from an external domain.

![](https://hackmd.io/_uploads/rkeAcrRW6.jpg)

To view the trained model from the local end.

![](https://hackmd.io/_uploads/SkkciSA-a.png)

:::spoiler Solution
:::success
**Resolved：**
Open your terminal and use this command.
```javascript=
ssh -L 32005:localhost:32005 aiml@192.168.8.44
```
:::
## 8. Appendix
### 8-1. Software Uninstallation 
```javascript=
bin/uninstall_traininghost.sh
```
### 8-2. Uninstall Kserve
```javascript=
./bin/uninstall_kserve.sh
```

https://hackmd.io/Cf5E97XqTXSr00xC7bzfeA?view

https://hackmd.io/lZK4w7EdTiyH-So34_FZRw#Deployment-of-dmaap-InfluxDB-adapter-InfluxDB-and-Grafana

https://hackmd.io/Cf5E97XqTXSr00xC7bzfeA?view