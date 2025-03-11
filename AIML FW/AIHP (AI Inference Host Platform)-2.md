# <center> AIHP (AI Inference Host Platform)</center>

<center>The implementation of kserve_adapter on Near-RT RIC</center>

###### tags: `AI/ML`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue, Oct19 , 2023 3:38 PM][color=#38c16a]

:::info
**Introduction :**

Utilize RICDMS for the management of inference service deployments, with the kserve-adapter packaging the Helm chart to facilitate deployment through RICDMS.

**Goal：**
- [x] [Install Kserve/golang/chartmuseum in the Near-RT RIC.](#2-Install-Kservegolangchartmuseum-on-the-Near-RT-RIC)
- [x] [Implement RICDMS progress.](#3-Run-ricdms-on-the-Near-RT-RIC)
- [x] [Implement kserve_adapter progress.](#4-Run-kserve-adapter-on-the-Near-RT-RIC)
- [x] [Test prediction on the Near-RT RIC.](#4-5-Perform-predictions)
:::
:::warning
**Near-RT RIC configuration :**
1. Hardware：
	- RAM：8G RAM
	- CPU：6 core
	- Disk：40G Storage
2. Installation Environment：
    - IP：192.168.8.225
    - user：ubuntu
	- Host：Windows 10
	- Hypervisor：VMware Workstation 16 Player
	- VM：Ubuntu 20.04 LTS (Focal Fossa)
	- Kubernetes version：1.18.0
:::
**<center>Table of Content</center>**
[TOC]

## 1. Architecture
### 1-1. Project location

![](https://hackmd.io/_uploads/S1lcmJ65Gp.png)

### 1-2. Sequence Diagram

![](https://hackmd.io/_uploads/Sy7c8DCZ6.png)

### 1-3. Context Diagram

![](https://hackmd.io/_uploads/SkjD9s_z6.jpg)

- O-Cloud : To deploy/undeploy kserve inference service, RICDMS requests to deploy/undeploy to Kserve Adapter.
- RIC : Kserve Adapter requests onboarding of inference service to xapp-onboarder before deploy inference service.
- SMO : Kserve Adapter monitors and manages kserve inference service. Kserve Adapter will deliver monitoring information to SMO, and SMO can retrieve inference.

Reference : https://wiki.o-ran-sc.org/display/AIMLFEW/Kserve+Adapter
## 2. Install Kserve/golang/chartmuseum on the Near-RT RIC

### 2-1. Install Kserve
```javascript=
git clone "https://gerrit.o-ran-sc.org/r/aiml-fw/aimlfw-dep"
cd /aimlfw-dep/bin/
./install_kserve.sh
```

:::warning
**Attentation :**

The Kserve need **kubernetes 1.17.0** or higher version. If the version is lower then 1.17.0, you will get this error like this figure.

![](https://hackmd.io/_uploads/rJUAKu0bT.png)
:::

Completion of the install.

![](https://hackmd.io/_uploads/SkKEcydza.png)

### 2-2. Install golang
Reference : [Upgrade Your Go/Golang Version to 1.21(latest): A Step-by-Step Guide](https://jinxankit.medium.com/upgrade-your-go-golang-version-to-1-21-latest-a-step-by-step-guide-1d72294453f8)
```javascript=
git clone "https://gerrit.o-ran-sc.org/r/ric-plt/ricdms"
```
#### 2-2-1. method 1 : Use ubuntu’s package repository
```javascript=
cd ricdms/
apt install golang-go
```
![](https://hackmd.io/_uploads/Hkqv--uz6.png)

#### 2-2-2. method 2 : Installing Go via the Latest Binary Release
Uninstall the existing Go package.
```javascript=
sudo rm -rvf /usr/local/go
```
Download specific binary release for your system.
```javascript=
sudo wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
```
Extract package.
```javascript=
sudo tar -xvf go1.21.0.linux-amd64.tar.gz -C /usr/local
```
Setup Go Environment.
```javascript=
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```
**GOROOT :** is the location where the Go package is installed on your system.
**GOPATH :** is the work directory of your go project.
```javascript=
source ~/.bashrc 
```
Verify Installations.
```javascript=
go version
```

![](https://hackmd.io/_uploads/S1ybW-_Ga.png)

### 2-3. Install chartmuseum
Reference : [Chartmuseum deploy by Helm.](https://blog.csdn.net/lishuailing123/article/details/133313094)

Use this command to add helm repository.
```javascript=
cd
helm repo add chartmuseum https://chartmuseum.github.io/charts
```
Pull the latest version.
```javascript=
helm fetch chartmuseum/chartmuseum
```
Unzip the file.
```javascript=
tar xzvf chartmuseum-3.10.1.tgz
```
Before Install the chartmuseum, revise the configuration.
```javascript=
vim chartmuseum/values.yaml
```
![](https://hackmd.io/_uploads/SJbktHqfT.png)

Install chartmuseum.
```javascript=
helm upgrade --install chartmuseum ./chartmuseum
```
Check complete install or not.
```javascript=
kubectl get pod -A
```
**Result :**

![](https://hackmd.io/_uploads/Hydo9H9MT.png)

Check chart museum IP.
```javascript=
kubectl get svc -A
```
![](https://hackmd.io/_uploads/r1yH_IcG6.png)

Check it has a chart or not.
```javascript=
curl http://10.98.150.17:8080/api/charts
```
![](https://hackmd.io/_uploads/B1iFFI9GT.png)

Because I already complete the upload chart proceesing, if you have upload not yet, you will see this result **{}**.

## 3. Run ricdms on the Near-RT RIC

### 3-1. Install go-swagger first 
```javascript=
cd ricdms
git clone https://github.com/go-swagger/go-swagger
cd go-swagger
go install ./cmd/swagger
```
To verify that go-swagger has been installed, type **“swagger”** and press ENTER. That should give below output. 
```javascript=
Please specify one command of: diff, expand, flatten, generate, init, mixin, serve, validate or version
```
If you recieve the information like this **"swagger: command not found"**, Use this command to solve it.

```javascript=
echo 'export PATH="${GOPATH-"~/go"}/bin:$PATH"' >> ~/.bashrc 
source ~/.bashrc
```
Because you're missing $GOPATH/bin in your path reason why it cannot find it.

Use this command to check again.

```javascript=
swagger version
```
![](https://hackmd.io/_uploads/Hkme7ZOMT.png)

### 3-2. Run ricdms
Build the file.
```javascript=
cd ricdms/
make build
```
![](https://hackmd.io/_uploads/ryHYa-uGT.png)

If you recieve error about **"go: updates to go.mod needed; to update it:go mod tidy"**, use this command to resolve.

```javascript=
go mod tidy
```

:::info
**Reason :**

It is usually because the version of the dependent package used in the project is inconsistent with the version recorded in the go.mod file. This problem can be solved by running "go mod tidy"
:::

Check the ricdms file is create or not.
```javascript=
ls
```
![](https://hackmd.io/_uploads/H1hvQG_fa.png)

Use this command to build image.
```javascript=
make image
```
![Screenshot 2023-11-07 at 3.21.18 PM.png](https://hackmd.io/_uploads/ryMGUDDQT.png)

Add the changes to `deployment/dms-config.yaml` as per your environment (refer your `.kubeconfig` file).

This is your `.kubeconfig` file.
```javascript=
cd 
cd .kube
cat config
```
**".kubeconfig"**
```javascript=
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1URXdNVEEyTVRFd09Gb1hEVE16TVRBeU9UQTJNVEV3T0Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTW5ZCjFxOUVqN3ZPZjgyS09jbGFGQ0MzK2t0SDViTHMzTnlhdEZlNTFaellhS2VSVEI0SHg4SithLzh4MFd3U21Od00KcDF1dWxUTzJETFVVT3R0cHBNYmN2cm1TeFJEQjdwTXBFY0RDK25ldTFtY2hxRjJMMDFvYnF3OUlsVTlQZWROZwpqZXQ1RDl0b3Rrc0poZlpEQ0dDWGxxa0t4Y0czQTcybTMvSkw4bFcySVUzTnFlelBZTlpoZ25YaDBabWJwelBpCjU4ais3bExLVmhJdFRwWWxkMTFob1FSaTVNcjg2Qi9PMWNLbHEzS3BjeVU5NDMzNnNGNGRaN2pwM01oeHh2c2MKVTN3NDdRTDdabldMRGRrVkFZT2xyZzJQS0Q4SlVtSSs5aU9kd3VsdmlQeExMcHZKV3ROQWppZlBpRlN4TE1FUwptVXBVMnZZcG94d2xPZWZzbFdFQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFBOFEraU93aXRiNlYzQ0Y1cTJmVzNxYjkzU00KNXZIUGxUWGphdjQvV2h6VDJkWWVXSFJYNlk1dUxaQzByUDVnM2hnMGFtWXhBR3JFakltYVJWQlBoUVRWbGo4TApiMWpyT1kzdXVMU08xVGVnQ282cFVxUVhlQjFCRjExUWE0NDlLZ1d0VlBBSWx2NjdyM2x0aEhJNFdTTXB3QlcyCmwzVUV4VFJzUjRBandPL2hWREt6alk5RDNSWGorU29uQ0ZCRFlWRmhwcVg0cHlNRFN2OXhITzlYd21rZXYvOVQKWDVKa2pXSHNEckR2dVRzZHYyU3JtOTF6cWVoU2htMHFMVVN4RndCREFEVSt0Y2hyNUMweHEyd3hvYVBhR05tMQpVb1BYRUUzUmdoNlR4QWhjT05SdkRqS0ZHNXh5MXN5Q0pTL1RZTnNqb1NNK0lKd1FUbTd4aFBSR3ZUVT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://10.0.10.121:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJBZ0lJV2Z1MERmemZ2VG93RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpFeE1ERXdOakV4TURoYUZ3MHlOREV3TXpFd05qRXhNVEJhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXl2UUVtRXlrcEFoNkJnSXYKK0h6UVl6WlREblNNcXZWbGkxRVNCRDJNLzJ6UnB6eVVKN285dFc4U2hnU3JSMjE3OVpKTnptd1g1WVpmcWpnWApnTUo3MS84VG43NUc0emlJL3F1STVBWGtsZWpNK01RZ0d6UFl5UlJ6NUdxQUZVWEpiYXJLVGIwYXVPRlNUTnJECi9IeWJ6MVpnUWtPU3pZSXhtdDVlcCtzbDNLUE41aVU3TjJ1ZzVOKzZBdjFvWW5icGt2Y0JBTmordk5SVWV1K3IKcExnWVNTRVBzWmlzUUU4UXRtdG9RTTFPdE1OTlV2RU5kbjNiRVg1OXNZVzhoQ3htbTRIZThnVE5VM1FvcmpmbwpwV1BWS041YnBvMklpdEJIODJ0cmZwU0hidjBsSWtublNURGR6WXlZazlNTnhCbmY5czN5c1BwNFl5eUIrOFg4Ckg3QURPd0lEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFFdWcwNWVBeWFWb1RBa2NEZzl6Rm9VV2Rhb1JDc0xUVWN6egpaK0RuMW1OejN5ZzhvK3J1eGp3VGlsODY5clJrSjFqc29aVUZseHhPUGM5NzhiUk5mM3RPWEdYR1VvQ2piY29uCjNyOFB6eGY3aEpjbWVFVWlkSndvclViU0s4ZVlzQllGVDYyUEI1ZDQxUWx3VDBsK1NVOTlOcnY5aHRvamw4Z2gKOTMyT0JkcjE2cFFqK2tYdmU0YjgyN1puRnlLK1ljSnJRL2h4a0dPMU5BTENSY3FrUElFZnl6NjZ0K09VeFphTApVdWUvcTFCMjNtcStPUkwvc1J2aXZ4QlFOcFd2NW5CUGdOczlSVmVoZjRDeFBRK3FnamY4UlkyQjNhT0FnVXl5Cm5ONXl0WjNhNUY5MXF4NXpSenFGcFdKNE9SYUpOS3lDclRhTVhvZ2lodGk4cjRWVzBWbz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBeXZRRW1FeWtwQWg2QmdJditIelFZelpURG5TTXF2VmxpMUVTQkQyTS8yelJwenlVCko3bzl0VzhTaGdTclIyMTc5WkpOem13WDVZWmZxamdYZ01KNzEvOFRuNzVHNHppSS9xdUk1QVhrbGVqTStNUWcKR3pQWXlSUno1R3FBRlVYSmJhcktUYjBhdU9GU1ROckQvSHliejFaZ1FrT1N6WUl4bXQ1ZXArc2wzS1BONWlVNwpOMnVnNU4rNkF2MW9ZbmJwa3ZjQkFOait2TlJVZXUrcnBMZ1lTU0VQc1ppc1FFOFF0bXRvUU0xT3RNTk5VdkVOCmRuM2JFWDU5c1lXOGhDeG1tNEhlOGdUTlUzUW9yamZvcFdQVktONWJwbzJJaXRCSDgydHJmcFNIYnYwbElrbm4KU1REZHpZeVlrOU1OeEJuZjlzM3lzUHA0WXl5Qis4WDhIN0FET3dJREFRQUJBb0lCQVFDdzl3OWVvVTNhUGczdwpXVTNzMVNCN3NlM0FKLzVVUDMvWWQ4dEc2VWlkbkF3L1Q4STcxZGhpOE1QdEdmc2pZQ0w3WVNQNC95WGpIRVVrClRwNm54bTFvVE9HV283cDUvRnp2K3pCMDYzS0RDS2hacmVIMDlrTnNLaXBYbkVtc3d0bzlodk81ejArU3I5NHMKWjRFSEVyeGxrUUtFSlJuSG1tT1lqNWRud2RHVXphdnlVcDMvSzUwbGZkT3NBby9tNTRwMU5jenZBVHV4WmJMMApQa1JVVTRUMmFETS9vSlgvdG1qT2pXZklTekpVTDFJZXV5Q204eE9vWGQyQjhFOVA2cUNZZmRoWkUzZFdzcEZLCmtFVkRoMDc1bzZDcUw0TWdybVpVRHc2ZDNzVERjWHNWVkR6N0JNcW5BOVRkMjhqb01Dc1M0ZGxBK3Y0K2YzMEUKVHZTWDRDRlJBb0dCQU10eU5FclJpUWc0ZVVBS2Z0QytVUEpSRHU0ZVlZWm9IQWVVeVZ3OWw2Mk12RW9UUFIxMgpzL3A5SllnRkFRaXlZWWVGNTZOaUovY3pXUmYxdzNlS3BaUEV2V21RQmsvSU9RMGd1ckRJWVVVY25FQjZ1YldaCmR5T0szZFRFQjlJMzFLUDNwbUZwSWtnNEtJOTNIYVczdms5cE1KeVp2UXd3VnFobjQybFc0citEQW9HQkFQOWgKTjZrbHI3alVSb0RPd1Q1aUNIQUtFSFJPOURydzQ3dzZReEZ3aXozaE12K0IrNjZjWkwxWk43b2pjUEhnV0lISwoyRk8vUThRM08vSG9nMkZqcDNZYkd2T01kR3l5NzZHbjUzaTl1OVpheHl4MytyUDdWZnZnbDhweTNib3V3ZWV6CnFYVVVSVHNVRUpjZGdNdmRFRkg2aVo2MGhFcm0rc1FTSjFwSkZtZnBBb0dBQWd1MDhPZW9mQmV0U0hLU2tlREkKQ1plOUViSG1neVo2MmF5cVZhNGMzMWJoOGRDOXRaVWkvQ3JUL01rb0dJRktyOFV0N2h1bmtUbkg5SkM1RlhPawpkSmJ1M0tmaEdGNUlESlMrcTlabisvenNxVTFTbnJ2YlVkVXNvOTRRd2hGanB2NXZndDAreGdFaWowYkFXcEU5CmJhaitIeVVBbktYRHlVKzZIcTRMKzZjQ2dZQlZzV0tEQUtGWlRPbW5lVGxBM0pabU9ncFJiTmpwR2tIZ3ZGQWEKL3YvS01OSHpDTVBTVUtwQkd6bm0zTk9lWmlCczFRc0g5d3NmUVVWOUkvOUo0NjJpcE8vRFA2TWxnbG1FamhuTwoyeU8zaHRpRXBISGNpUDdPT1F2V0kvc2c2V1dwZ1JEZ3QzK1BsbWtHdkNDbXg3UWRQZ3VGMUo3N24wd1FGT05kCm1WN0tXUUtCZ1FDbG5ZSVJNV3VIbElsTXRxUklzS1hiQ2dENnNjaTVFdmw3dFlmaDBleTJ2eVBWU0s3cTN1RzUKUTBrdWp2dGtybDN0L2VjbkJjNVFXUnFhS2NteVNkNjhQZTdWeFNESGhXN0hYQ0lMQ1dTbVhaYUpFN3FBZ2xCNwpFWEU4Y3RzVy9LZTBwYysycWJnZzFtb2hRTlRQWUVEVjgrSDNsTWdTKzJnenVZdVFqYStEcEE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```
Revise your dms-config.yaml
```javascript=
cd ricdms/deployment/
vim dms-config.yaml
```
**"dms-config.yaml"**

```javascript=
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-env
data:
  config: |
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: <certificate>
        server: https://<kube-ip>:<kube-port>
      name: <name>
    contexts:
    - context:
        cluster:<cluster-name>
        user: <user>
      name: <name>
    current-context: <context>
    kind: Config
    preferences: {}
    users:
    - name: <name>
      user:
        client-certificate-data: <cliet-cert> 
        client-key-data: <client-key-data>
```
And then revise dms-dep.yaml
```javascript=
vim dms-dep.yaml
```
Revise **"<build image name>"** to **"ric-dms:v1.0"**
```javascript=
apiVersion: v1
kind: ReplicationController
metadata:
  name: dms-server
  labels:
    name: dms-server
    app: dms-server
spec:
  replicas: 1
  selector:
    app: dms-server
  template:
    metadata:
      labels:
        name: dms-server
        app: dms-server
    spec:
      containers:
      - name: dms-server
        image: ric-dms:v1.0
        imagePullPolicy: Always
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
          - containerPort: 8000
            protocol: TCP
        volumeMounts:
        - name: kube-config-vol
          mountPath: /root/.kube
          readOnly: true
      volumes:
        - name: kube-config-vol
          configMap:
            name: kube-env
```
If you can't pull image in the repository, revise **"Always"** to **"IfNotPresent"** in line **"21"**. [See more](#5-1-Problem-1：dms-server-pod-ImagePullBackOff-)

:::info
**Some docker command I use :**
- check container
    - docker ps
- check images
    - docker images
- stop container
    - docker stop <container ID>
- pull images
    - docker pull <REPOSITORY>:<TAG>
:::

Apply the deployment yaml.
```javascript=
cd ..
kubectl apply -f deployment
```

The pod is created.

![Screenshot 2023-11-07 at 10.35.39 PM.png](https://hackmd.io/_uploads/SyaUn6v7p.png)

Revise the **"config-test.yaml"** customOnboard-url port to **"8080"** which is **"37019"** and add new line below download-chart as **download-charts-url-format: "http://127.0.0.1:8080/charts/%s-%s.tgz"**.

```javascript=
cd config/
vim config-test.yaml
```

![](https://hackmd.io/_uploads/rkrCLdcfa.png)

Run ricdms.
```javascript=
export RIC_DMS_CONFIG_FILE=$(pwd)/config/config-test.yaml
./ricdms
```
:::info
**If you want to now who is listen to 8000 port, you can you this command.**
```javascript=
lsof -i :8000
```
**Kill the process.**
```javascript=
kill -9 <PID>
```
:::
**Result :**

![](https://hackmd.io/_uploads/SJvK4zufa.png)

## 4. Run kserve-adapter on the Near-RT RIC

Open **"new terminal"** to install kserve-adapter.

### 4-1. Install Kserve-adapter
```javascript=
cd
git clone "https://gerrit.o-ran-sc.org/r/aiml-fw/aihp/ips/kserve-adapter"
```
Check the go version is up then go1.16.0 version. If not, click [here](#2-2-2-method-2--Installing-Go-via-the-Latest-Binary-Release) to upgrade the go version.
```javascript=
go version
```

Build Kserve-adapter.

```javascript=
cd kserve-adapter/
go get ./cmd/kserve-adapter
go build -o kserve-adapter cmd/kserve-adapter/main.go
```

### 4-2. Onboard sample-xapp descriptor and schema processing
Create namespace.
```javascript=
kubectl create ns ricips
```
Configuration setting.
```javascript=
export PATH=$PATH:/usr/local/go/bin/
```
Update **<Model URL>** in **"sample_config.json"** file.
```javascript=
cd kserve-adapter/pkg/helm/data/ 
vim sample_config.json
```

![](https://hackmd.io/_uploads/Sk0N17dGa.png)

Revise the <Model URL> to **"http://192.168.8.44:32002/model/qoetest6/1/Model.zip"**, this URL can obtain from your AI/ML dashboard where you complete to training.

```javascript=
{
    "xapp_name": "sample-xapp",
    "xapp_type": "inferenceservice",
    "version": "2.2.0",
    "sa_name": "default",
    "inferenceservice": {
        "engine": "tensorflow",
        "storage_uri": "http://192.168.8.44:32002/model/qoetest6/1/Model.zip",
        "runtime_version": "2.5.1",
        "api_version": "serving.kubeflow.org/v1beta1",
        "min_replicas": 1,
        "max_replicas": 1
    }
}
```

After revise, use this command to setting **"KUBECONFIG"**,**"API_SERVER_PORT"**,**"CHART_WORKSPACE_PATH"**,**"RIC_DMS_IP"** and **"RIC_DMS_PORT"** to run main.go.

```javascript=
cd ../../..
KUBECONFIG=/root/.kube/config API_SERVER_PORT=10000 CHART_WORKSPACE_PATH="/root/kserve-adapter/pkg/helm/data" RIC_DMS_IP=127.0.0.1 RIC_DMS_PORT=8000 go run cmd/kserve-adapter/main.go
```

**Result :**

![](https://hackmd.io/_uploads/r1Z5oRuMa.png)

### 4-3. Generating and upload helm package
Check the **"sample_config.json"** && **"sample_schema.json"**.

```javascript=
cd kserve-adapter/pkg/helm/data/
vim sample_config.json
```

**sample_config.json**

```javascript=
{
    "xapp_name": "sample-xapp",
    "xapp_type": "inferenceservice",
    "version": "2.2.0",
    "sa_name": "default",
    "inferenceservice": {
        "engine": "tensorflow",
        "storage_uri": "http://192.168.8.44:32002/model/qoetest6/1/Model.zip",
        "runtime_version": "2.5.1",
        "api_version": "serving.kubeflow.org/v1beta1",
        "min_replicas": 1,
        "max_replicas": 1
    }
}
```
```javascript=
vim sample_schema.json
```
**sample_schema.json**
```javascript=
{
    "definitions": {},
    "$schema": "http://json-schema.org/draft-07/schema#",
    "$id": "http://example.com/root.json",
    "type": "object",
    "title": "The Root Schema",
    "required": [
      "xapp_name",
      "xapp_type",
      "version",
      "sa_name",
      "inferenceservice"
    ],
    "properties": {
      "inferenceservice": {
        "$id": "#/properties/inferenceservice",
        "type": "object",
        "title": "The Inferenceservice Schema",
        "required": [
          "engine",
          "storage_uri",
          "runtime_version",
          "api_version",
          "min_replicas",
          "max_replicas"
        ],
        "properties": {
          "engine": {
            "$id": "#/properties/inferenceservice/properties/engine",
            "type": "string",
            "title": "The Engine Schema",
            "default": "",
            "examples": [
              "tensorflow"
            ]
          },
          "storage_uri": {
            "$id": "#/properties/inferenceservice/properties/storage_uri",
            "type": "string",
            "title": "The Storage_uri Schema",
            "default": "",
            "examples": [
                "s3://mlpipeline/artifacts/sample-rl-pipeline-bbzh4/sample-rl-pipeline-bbzh4-949264085/sample-training-saved-model.tgz"
            ]
          },
          "runtime_version": {
            "$id": "#/properties/inferenceservice/properties/runtime_version",
            "type": "string",
            "title": "The runtime version Schema",
            "default": "",
            "examples": [
                "2.5.1"
            ]
          },
          "api_version": {
            "$id": "#/properties/inferenceservice/properties/api_version",
            "type": "string",
            "title": "The Api_version Schema",
            "default": "",
            "examples": [
                "serving.kubeflow.org/v1alpha2",
                "serving.kubeflow.org/v1beta1"
            ]
          },
          "min_replicas": {
            "$id": "#/properties/inferenceservice/properties/min_replicas",
            "type": "integer",
            "title": "The Min_replicas Schema",
            "default": 0,
            "examples": [
              1
            ]
          },
          "max_replicas": {
            "$id": "#/properties/inferenceservice/properties/max_replicas",
            "type": "integer",
            "title": "The Max_replicas Schema",
            "default": 0,
            "examples": [
              1
            ]
          }
        }
      }
    }
  }
```
Before upload helm package, you need to prepare preparatory work.

Open **"new terminal"** and set up port forwarding.
```javascript=
kubectl port-forward svc/chartmuseum 8080:8080
```
Open **"new terminal"** to keep processing.

Add helm repository.
```javascript=
helm repo add localhost http://127.0.0.1:8080
```
Check you can visit or not.
```javascript=
curl http://127.0.0.1:8080/api/charts
```
**Result :**

![](https://hackmd.io/_uploads/B1iSpU9Ga.png)

Because I already complete the upload chart proceesing, if you have upload not yet, you will see this result **{}**.

**Bad Result :**

![](https://hackmd.io/_uploads/BJ4-AL5GT.png)

Now, check if all terminal is running.

**Terminal 1 : Run ricdms.**

![](https://hackmd.io/_uploads/SJvK4zufa.png)

**Terminal 2 : kserve_adapter run main.go.**

![](https://hackmd.io/_uploads/r1Z5oRuMa.png)

**Terminal 3 : chartmuseum port forwarding.**

![](https://hackmd.io/_uploads/B1iPJwqzp.png)

**Terminal 4 : Upload helm package.**

Use this command to upload helm package.
```javascript=
curl --request POST --url 'http://127.0.0.1:10000/v1/ips/preparation?configfile=pkg/helm/data/sample_config.json&schemafile=pkg/helm/data/sample_schema.json'
```

**Terminal 1 : Recieve request to onboard.**

![](https://hackmd.io/_uploads/SygUZD9zp.png)

**Terminal 2 : Onboard chart to ricdms**

![](https://hackmd.io/_uploads/HJAkGPcM6.png)

**Terminal 3 : Handling connection for 8080.**

![](https://hackmd.io/_uploads/Sym3-P5z6.png)

**Terminal 4 : Check uploaded charts.**

```javascript=
curl http://127.0.0.1:8080/api/charts
```

**Result :**

![](https://hackmd.io/_uploads/B1iSpU9Ga.png)

### 4-4. Deploy the model
Use this command to deploy.
```javascript=
curl --request POST --url 'http://127.0.0.1:10000/v1/ips?name=inference-service&version=1.0.0'
```
**Terminal 1 : Response**

![](https://hackmd.io/_uploads/SJ4ivOcGa.png)

**Terminal 2 : Response**

![](https://hackmd.io/_uploads/rJClO_5Ma.png)

Check deployment.

```javascript=
kubectl get InferenceService -n ricips
```
**Result :**
![](https://hackmd.io/_uploads/SyaIOd5f6.png)

### 4-5. Perform predictions
Use below command to obtain Ingress port for Kserve.
```javascript=
kubectl get svc istio-ingressgateway -n istio-system
```

![](https://hackmd.io/_uploads/BJ00tOczT.png)

```javascript=
cd 
vim predict_inference.sh
```
Duplicate the below content and revise IP and Port.

```javascript=
model_name=sample-xapp
curl -v -H "Host: $model_name.ricips.example.com" http://"IP of where Kserve is deployed":"ingress port for Kserve"/v1/models/$model_name:predict -d @./input_qoe.json
```

**"IP of where Kserve is deployed"** : 10.0.10.217
**"Ingress port for kserve"** : 31011

![](https://hackmd.io/_uploads/B1q8yo5fT.png)

Create sample data to prediction.
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
### 4-6. Result ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)
Use this command to trigger prediction.
```javascript!
source predict_inference.sh
```
**Result :**


![](https://hackmd.io/_uploads/SkvAgoqfa.png)

## 5. Problem set
:::warning
- [x] [Problem 1：dms-server pod ImagePullBackOff](#5-1-Problem-1：dms-server-pod-ImagePullBackOff-)
:::
### 5-1. Problem 1：dms-server pod ImagePullBackOff ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)


![Screenshot 2023-11-07 at 9.59.21 PM.png](https://hackmd.io/_uploads/rkkRG6vQT.png)

:::spoiler Solution
:::success
**Resolved：**

Use this command to see the pod detail.
```javascript=
kubectl describe pod <your pod name>
```
You will see the error is like **"Failed to pull image "ric-dms:v1.0": rpc error: code = Unknown desc = Error response from daemon: manifest for ric-dms:v1.0 not found: manifest unknown: manifest unknown"**

Check you have a image or not.
```javascript=
docker images | grep dms
```
![Screenshot 2023-11-07 at 10.16.33 PM.png](https://hackmd.io/_uploads/HyERUaPXp.png)

We can change pull images method to resolve this problem.

Revise **"Always"** to **"IfNotPresent"**
```javascript=
cd ricdms/deployment/
vim dms-dep.yaml
```
```javascript=
apiVersion: v1
kind: ReplicationController
metadata:
  name: dms-server
  labels:
    name: dms-server
    app: dms-server
spec:
  replicas: 1
  selector:
    app: dms-server
  template:
    metadata:
      labels:
        name: dms-server
        app: dms-server
    spec:
      containers:
      - name: dms-server
        image: ric-dms:v1.0
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
          - containerPort: 8000
            protocol: TCP
        volumeMounts:
        - name: kube-config-vol
          mountPath: /root/.kube
          readOnly: true
      volumes:
        - name: kube-config-vol
          configMap:
            name: kube-env
```
**IfNotPresent method** : If the mirror pull policy is set to IfNotPresent, the mirror will be pulled only if the mirror does not exist. If not specified, the default value is IfNotPresent.

After revise, deploy the pod.

**Result :**
```javascript=
kubectl get pod -A
```
![Screenshot 2023-11-07 at 10.35.39 PM.png](https://hackmd.io/_uploads/S1YYiaw7a.png)

```javascript=
kubectl describe pod dms-server-vvc5x
```
![Screenshot 2023-11-07 at 10.36.31 PM.png](https://hackmd.io/_uploads/BkHts6Dmp.png)

**Reference :** 
- https://blog.csdn.net/longway111/article/details/124028105
- https://www.datree.io/resources/kubernetes-troubleshooting-fixing-imagepullbackoff-state-error
:::
## 5. Reference
- https://stackoverflow.com/questions/60382748/go-swagger-command-not-found
- https://shashankvivek-7.medium.com/go-swagger-a-go-web-framework-worth-learning-af0e9ed75343
- https://wiki.o-ran-sc.org/download/attachments/81297504/kserve_adapter_demo.mp4?api=v2
- https://adamtheautomator.com/install-go-on-ubuntu/
- https://jinxankit.medium.com/upgrade-your-go-golang-version-to-1-21-latest-a-step-by-step-guide-1d72294453f8
- https://docs.o-ran-sc.org/projects/o-ran-sc-aiml-fw-aimlfw-dep/en/latest/installation-guide.html