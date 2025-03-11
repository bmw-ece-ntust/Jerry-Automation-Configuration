# <center>[J Release] SMO </br>Topology & Inventory (TEIV) </br>Installation Guide </center>

<center>Installation for J Release SMO</center>

###### tags: `SMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Jan2 , 2024 3:38 PM][color=#38c16a]

:::info
**Introduction：**

Topology Exposure & Inventory is a tool for managing telecommunications network structures. It describes various equipment in the network and their relationships in a universal way, independent of specific vendors.

**Goal：**
- [x] [J Release] SMO Install Topology & Inventory (TEIV)

**Reference：**
- [Release J - Run in Kubernetes](https://wiki.o-ran-sc.org/display/SMO/Release+J+-+Run+in+Kubernetes)
- [[J Release] SMO Topology & Inventory (TEIV)](https://hackmd.io/@Jerry0714/BkkmMXKPR)
- [[J Release] SMO Topology & Inventory (TEIV) User Guide](https://hackmd.io/@Jerry0714/ByElUtroA)
- [[J Release] SMO Topology & Inventory (TEIV) Developer Guide](https://docs.o-ran-sc.org/projects/o-ran-sc-smo-teiv/en/latest/developer-guide.html#)
- [Gerrit smo/teiv.git](https://gerrit.o-ran-sc.org/r/gitweb?p=smo/teiv.git;a=tree;h=ea1ce6271dfe7d94b004c09d89254c4924c937fc;hb=ea1ce6271dfe7d94b004c09d89254c4924c937fc)

:::
:::warning
**1. Hardware requests：**
- RAM： GB RAM
- CPU： cpu cores
- Disk： GB harddisk

**2. Installation Environment：**
- Host：192.168.8.27

**3. Platform Environment：**
- Kubernetes：needs (kubernetes v1.19 +)
    - Client Version：v1.28.0
    - Server Version：v1.28.0
- Helm：v3.12.3
- Docker：
    - Client Version：None
    - Server Version：None
:::

**<center>Table of Content</center>**
[TOC]

## 1. Prerequisite
### 1-1. Install Kubernetes with kubespray
```javascript=
python3 --version
apt install -y python3-pip

git clone https://github.com/kubernetes-sigs/kubespray
cd kubespray 
pip install -r requirements.txt

sed -i 's/\(kube_version: \)[^"]*/\1v1.28.0/' inventory/local/group_vars/k8s_cluster/k8s-cluster.yml
ansible-playbook -i inventory/local/hosts.ini --become --become-user=root cluster.yml
```

![Screenshot 2024-07-19 at 4.24.15 PM](https://hackmd.io/_uploads/S1nymiDOC.png)

## 2. Dawnload [J] SMO TEIV file
```javascript=
sudo -i
apt install git
git clone "https://gerrit.o-ran-sc.org/r/smo/teiv" -b j-release
```
**Folder**

![Screenshot 2024-08-14 at 3.35.42 AM](https://hackmd.io/_uploads/HkuhrEt5A.png)


![Screenshot 2024-08-14 at 3.38.05 AM](https://hackmd.io/_uploads/SJx6U4Y5A.png)


## 3. Deploy TEIV
```javascript=
cd teiv/charts
vim install-teiv.sh
```
Revise the config to this.
```javascript=53
sudo apt-get install make -y
(cd /root/teiv/charts/smo && make all)
```
And then, run the script.
```javascript=
chmod 777 install-teiv.sh
./install-teiv.sh
```

![Screenshot 2024-07-19 at 5.07.07 PM](https://hackmd.io/_uploads/rJ6p2sDu0.png)

Using this command to check whether you are successful to install or not.

```javascript=
kubectl get pod -A
```

![Screenshot 2024-07-19 at 5.09.28 PM](https://hackmd.io/_uploads/r1KUTsPO0.png)
## 4. Testing

[Please click on this page to view.](https://hackmd.io/@Jerry0714/ByElUtroA)
:::spoiler Error problem is already solved.
:::danger
1. Test TEIV API response

    ```bash=
    curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.133:8080/domains/getAllDomains
    ```
    
    - :warning: The response is 400 it means the request is invalid
    
        ![image](https://hackmd.io/_uploads/r1t02Iq5R.png)
2. Get log from the TEIV container
    ```bash=
    kubectl logs -f topology-exposure-54887cd47c-wmrcf -n smo
    ```
    - :warning: Error log from the TEIV container
    
        ![image](https://hackmd.io/_uploads/HyKPaUq9C.png)
        
        > The log shows that the http request is not being processed correctly resulting on http 400 reponse. Need more information to determine weather this issue occur because of wrong http query or it is the TEIV's internal problem.
:::
## 5. Question List
1. Was TEIV originally developed as part of the SMO, or is it an independent feature, like a plugin?
2. Is it necessary to install additional components to use TEIV? The installation results only show four components.
3. Is the TEIV functionality currently working properly? After installation, we conducted some tests, but no results were displayed. [Click here](#4-Testing)
    - We notice that for J Release Goal and K Release Goal:
        -  J Release Achievements: Introduce O-RAN-SC TEIV seedcode & ==achieve a first release==
        -  K Release Objectives:
            -  ==Continue to develop TEIV platform functions==
            -  Help O-RAN SC community to integrate sources of data and models to populate the registry with well-modeled data useful for different domains, scenario and use-cases
            -  Demonstrate capabilities of TEIV functions with informative use-cases & demonstrators.
            -  Integrate new and existing topology models, in particular supporting ongoing discussions and specification activities in O-RAN Alliance WG10  

## 6. Install TE&IV service in SMO
Makesure the following tool have been installed in SMO.
- ChartMuseum v0.16.1
- helm-push  v0.10.3
- helm v3.12.3

Revise the installation script to install TE&IV service.
```javascript=
cd teiv/charts/
vim install-teiv.sh
```
```javascript=
ROOT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null && pwd )"
CM_PORT="8879"
HELM_LOCAL_REPO="$ROOT_DIR/chartstorage"

echo "Starting ChartMuseum on port $CM_PORT..."
nohup chartmuseum --port=$CM_PORT --storage="local" --storage-local-rootdir=$HELM_LOCAL_REPO >/dev/null 2>&1 &
echo $! > $ROOT_DIR/CM_PID.txt

helm repo remove local
helm repo add local http://localhost:8879

sudo apt-get install make -y
(cd /root/teiv/charts/smo && make all)

# Creating namespace
kubectl create ns smo

# Installing helm charts
helm install --debug oran-smo local/smo --namespace smo

# Show the installed pods
kubectl get po -n smo
```
**Result:**

![Screenshot 2024-10-13 at 9.31.51 PM](https://hackmd.io/_uploads/r1YAoBYkkg.png)

