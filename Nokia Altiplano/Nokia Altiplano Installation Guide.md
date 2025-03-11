# <center>Nokia Altiplano Installation Guide</center>

<center></center>

###### tags: `Installation Guide`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Jan2 , 2024 3:38 PM][color=#38c16a]

:::info
**Action Item :**
- [x] [INP installation](https://hackmd.io/ODdiUeYqQ3Oszhxp672eXA#2-INP-installation)
- [x] [VNO installation](https://hackmd.io/ODdiUeYqQ3Oszhxp672eXA#3-VNO-installation)
:::
**<center>Table of Content</center>**
[TOC]

## 1. Download all helm charts
## 1-1. Tar the Altiplano file
```javascript=
tar xvf altiplano-volumeclaims-23.3.0-REL-0171.tgz
tar xvf altiplano-secrets-23.3.0-REL-0171.tgz
tar xvf altiplano-crds-23.3.0-REL-0171.tgz
tar xvf altiplano-infra-23.3.0-REL-0171.tgz
tar xvf altiplano-solution-23.3.0-REL-0171.tgz
```

![Screenshot 2024-01-18 at 9.14.39 PM](https://hackmd.io/_uploads/ryxL4sIK6.png)

## 2. INP installation
### 2-1. Create namespace 
```javascript=
kubectl create namespace inp
```
### 2-2. Installation of Persistent volumes
Using below command to install.

```javascript=
sudo bash altiplano-volumeclaims/pvhelpers/createLocalDirectoryStructure.sh /LOCALROOTDIR/inp
```
```javascript=
sed -i 's#/LOCALROOTDIR#/LOCALROOTDIR/inp#g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml

sed -i -e 's/RELEASENAME/inp-infra/g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml

sed -i -e 's/SOLUTION_RELEASE_NAME/inp/g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml

sed -i 's/name: pv-/name: pv-inp-/g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml

sed -i 's/namespace: default/namespace: inp/g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml
```
```javascript=
kubectl apply -f altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml
```

Check you success install or not.
```javascript=
kubectl get pv -A
```

![Screenshot 2024-01-18 at 9.23.17 PM](https://hackmd.io/_uploads/BJ_IIoUKp.png)

### 2-3. Secrets installing
```javascript=
helm -n inp upgrade --install inp-secret altiplano-secrets
```
### 2-4. Crds installing
```javascript=
helm -n inp upgrade --install inp-crd altiplano-crds
```
Check you success deploy or not.

![Screenshot 2024-01-18 at 9.25.39 PM](https://hackmd.io/_uploads/HyS1Ds8t6.png)
### 2-5. Infra installing
Create the installation script.
```javascript=
vim install-infra.sh
```
```javascript=
#!/bin/sh
helm -n inp upgrade --install inp-infra --timeout 1500s --debug \
--set global.K8S_PUBLIC_IP=192.168.8.40 \
--set global.registry=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.registry1=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.registry2=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.registry3=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.registry4=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set altiplano-nspos-rabbitmq.enabled=false \
--set altiplano-grafana.enabled=false \
--set altiplano-cbur.enabled=false \
--set global.INGRESS_STATUS_PORT='18080' \
--set altiplano-ingress.controller.statusPort='18080' \
--values=altiplano-infra/values.yaml \
altiplano-infra
```
Let file can be execute and run the command.
```javascript=
chmod +x ./install-infra.sh
./install-infra.sh
```
:::spoiler **[Problem solved]** If your pod not exiting the Init state please click here.
:::danger
**Problem :**

There are some pods not exiting the Init state during the reinstallation of inp-infra. This is causing the installation process to time out.

![Screenshot 2024-01-18 at 9.33.20 PM](https://hackmd.io/_uploads/rJzhusIKp.png)

**Installation Process :**

![Screenshot 2024-01-19 at 3.42.26 PM](https://hackmd.io/_uploads/SJTbdovF6.png)


**Helm install status**

![Screenshot 2024-01-19 at 3.43.26 PM](https://hackmd.io/_uploads/HylN_owY6.png)


**Check inp-infra-altiplano-indexsearch-secadmin-is logs :**

![Screenshot 2024-01-19 at 3.19.01 PM](https://hackmd.io/_uploads/HJ9uziwF6.png)

ERR: Seems there is no OpenSearch running on inp-infra-altiplano-indexsearch-client-77866f559b-qw2jw:9200 - Will exit

**Check inp-infra-altiplano-indexsearch-secadmin-is logs :**

![Screenshot 2024-01-19 at 3.22.33 PM](https://hackmd.io/_uploads/S1srXoPYa.png)

The logs shows that no route to host: 10.244.0.9/10.244.0.9:9300, means that there may be three possible situation.

- Target Pod or service not ready
- Firewall Issue
- DNS resolution issue

**Check inp-infra-altiplano-indexsearch-manager-0 logs :**

![Screenshot 2024-01-19 at 3.33.47 PM](https://hackmd.io/_uploads/BkczIjDK6.png)

- "host":"inp-infra-altiplano-indexsearch-manager-0.inp"
- "level":"ERROR"
- "logger":"o.o.s.c.ConfigurationLoaderSecurity7"
- "log":{"message":"Failure No shard available for [org.opensearch.action.get.MultiGetShardRequest@787aff92] retrieving configuration for [INTERNALUSERS, ACTIONGROUPS, CONFIG, ROLES, ROLESMAPPING, TENANTS, NODESDN, WHITELIST, ALLOWLIST, AUDIT] (index=.opendistro_security)"}


**[FAILED] 1. Revise the DNS configuration to solve problem** :

DNS config revise to 8.8.8.8

```javascript=
cat /etc/resolv.conf
```

![Screenshot 2024-01-19 at 4.26.36 PM](https://hackmd.io/_uploads/HyABG3DFp.png)

But the pod still cat't work.

**[SUCCESS] 1. Revise the DNS configuration to solve problem** :

Use this command to check your firewall is working or not.

```javascript=
sudo firewall-cmd --state
or 
sudo systemctl status firewalld
```
If is open then stop the firewall.
```javascript=
udo systemctl stop firewalld  # 停止 firewalld 服務
sudo systemctl disable firewalld  # 禁用 firewalld 服務（使其在啟動時不自動啟動）
```

![Screenshot 2024-01-19 at 8.26.46 PM](https://hackmd.io/_uploads/S1w95y_Fp.png)

Try to reinstall and the pod is all running now.

```javascript=
kubectl get pod -A
```

![Screenshot 2024-01-19 at 8.28.06 PM](https://hackmd.io/_uploads/HJiQj1ut6.png)

:::

### 2-6. Installation of altiplano-solution helm chart

Create the installation script.

```javascript=
vim install_solution.sh
```

```javascript=
helm -n inp upgrade --install inp \
--set global.registry=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.K8S_PUBLIC_IP=192.168.8.40 \
--set global.ingressReleaseName=inp-infra \
--set tags.aac=true \
--set tags.virtualizer=true \
--set altiplano-nsp-mistral-notifier.enabled=false \
--set altiplano-nsp-mistral-executor.enabled=false \
--set altiplano-nsp-mistral-engine.enabled=false \
--set altiplano-nsp-mistral-api.enabled=false \
--set altiplano-nspos-app3-tomcat.enabled=false \
--values=altiplano-solution/values.yaml \
altiplano-solution
```

Let file can be execute and run the command.
```javascript=
chmod +x ./install_solution.sh
./install-infra.sh
```

Check all pod is running or not.

```javascript=
kubectl get pod -A
```

![Screenshot 2024-01-19 at 8.28.06 PM](https://hackmd.io/_uploads/rJR76kdFa.png)

Check helm complete install or not.

![Screenshot 2024-01-19 at 8.52.58 PM](https://hackmd.io/_uploads/SJt0leuta.png)

### 2-7. Installation of nokiadashboards
Use below to install nokiadashboards.

```javascript=
cd $HOME/altiplano-infra/dashboards
bash ./createGrafanaDashboardConfigmaps.sh install nokiadashboards
```

Check install or not.

```javascript=
helm list -A
```

![Screenshot 2024-01-19 at 9.13.21 PM](https://hackmd.io/_uploads/HkzKBxut6.png)

### 2-8. login the Nokia Server

https://192.168.8.40/inp-altiplano-ac/

- username : adminuser
- password : password

**Result :**

![Screenshot 2024-01-19 at 9.23.18 PM](https://hackmd.io/_uploads/B1tCvlOKp.png)

## 3. VNO installation

### 3-1. Tar the Altiplano file
```javascript=
tar xvf altiplano-volumeclaims-23.3.0-REL-0171.tgz
tar xvf altiplano-secrets-23.3.0-REL-0171.tgz
tar xvf altiplano-crds-23.3.0-REL-0171.tgz
tar xvf altiplano-infra-23.3.0-REL-0171.tgz
tar xvf altiplano-solution-23.3.0-REL-0171.tgz
```
### 3-1. Create namespace 
```javascript=
kubectl create namespace vno
```
### 3-2. Installation of Persistent volumes
Using below command to install.

```javascript=
sudo bash altiplano-volumeclaims/pvhelpers/createLocalDirectoryStructure.sh /LOCALROOTDIR/vno
```
```javascript=
sed -i 's#/LOCALROOTDIR#/LOCALROOTDIR/vno#g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml

sed -i -e 's/RELEASENAME/vno-infra/g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml

sed -i -e 's/SOLUTION_RELEASE_NAME/vno/g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml

sed -i 's/name: pv-/name: pv-vno-/g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml

sed -i 's/namespace: default/namespace: vno/g' altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml
```
```javascript=
kubectl apply -f altiplano-volumeclaims/pvhelpers/altiplano-pvs-hostpath.yaml
```
### 3-3. Secrets installing

```javascript=
helm -n vno upgrade --install vno-secret altiplano-secrets
```
### 3-4. Infra installing

Change namespace from default to vno and modify some ingress pod in values.yaml file

```javascript=
sed -i 's/Namespace ""/Namespace vno/g' altiplano-infra/values.yaml

sed -i 's/3306:/3307:/g' altiplano-infra/values.yaml

sed -i 's/30080:/30082:/g' altiplano-infra/values.yaml

sed -i 's/30081:/30083:/g' altiplano-infra/values.yaml
```

Create the installation script.

```javascript=
vim install-vno-infra.sh
```
```javascript=
#!/bin/sh
helm upgrade -i vno-infra --namespace vno --timeout 1500s --debug \
--set global.K8S_PUBLIC_IP=192.168.8.40 \
--set global.registry=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.registry1=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.registry2=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.registry3=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.registry4=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.INGRESS_HTTPSPORT_ENABLED=true \
--set global.INGRESS_CONTROLLER_HTTPSPORT='32443' \
--set global.INGRESS_STATUS_PORT='18081' \
--set altiplano-ingress.controller.statusPort='18081' \
--set altiplano-ingress.controller.httpPort='32081' \
--set altiplano-ingress.controller.httpsPort='32443' \
--set altiplano-pts.restrictedToNamespace=true \
--set altiplano-pts.server.service.nodePort='30009' \
--set altiplano-pts.nodeExporter.extraArgs."web\.listen-address"=":9101" \
--set altiplano-pts.nodeExporter.service.podContainerPort='9101' \
--set altiplano-pts.nodeExporter.service.podHostPort='9101' \
--set altiplano-kafka.ingress.externalListeners[0].startPortRangeOnEdgeNode='9094' \
--set altiplano-kafka.ckaf-zookeeper.ingress.edgeNodePort='2182' \
--set altiplano-redis.extraServices.redis.nodePort='30078' \
--set altiplano-indexsearch.bssc-fluentd.fluentd.forward_service.fluentdNodePort='30025' \
--set altiplano-indexsearch.bssc-fluentd.fluentd.forward_service.syslogNodePort='30043' \
--set altiplano-webdav.service.ports.webdavNodePort='30082' \
--set altiplano-webdav.service.ports.httpsNodePort='30083' \
--set altiplano-grafana.enabled=false \
--set altiplano-nspos-rabbitmq.enabled=false \
--set altiplano-cbur.enabled=false \
--values=altiplano-infra/values.yaml \
altiplano-infra
```
Let file can be execute and run the command.
```javascript=
chmod +x ./install-vno-infra.sh
./install-vno-infra.sh
```

### 3-5. Solution installing

Create the installation script.

```javascript=
vim install-vno-Solution.sh
```
```javascript=
helm upgrade --install vno -n vno \
--set global.registry=altiplano-access-controller-product-docker.swdp-fi.support.nokia.com \
--set global.K8S_PUBLIC_IP=192.168.8.40 \
--set global.ingressReleaseName=vno-infra \
--set global.INGRESS_HTTPSPORT_ENABLED='true' \
--set global.INGRESS_CONTROLLER_HTTPSPORT='32443' \
--set tags.aac=true \
--set tags.slice-owner-controller=true \
--set altiplano-nsp-mistral-notifier.enabled=false \
--set altiplano-nsp-mistral-executor.enabled=false \
--set altiplano-nsp-mistral-engine.enabled=false \
--set altiplano-nsp-mistral-api.enabled=false \
--set altiplano-nspos-app3-tomcat.enabled=false \
--set altiplano-ac.env.REPLICATED_SM_INTENT_TOPICS=VNO \
--values=altiplano-solution/values.yaml \
altiplano-solution
```
Let file can be execute and run the command.
```javascript=
chmod +x ./install-vno-Solution.sh
./install-vno-Solution.sh
```
Check all pod is running or not.

```javascript=
kubectl get pod -n vno
```
![Screenshot 2024-03-27 at 10.50.56 PM](https://hackmd.io/_uploads/H1LDfnWkA.png)

```javascript=
kubectl get pod -A
```

![Screenshot 2024-03-27 at 10.52.54 PM](https://hackmd.io/_uploads/BkA0G2b10.png)
![Screenshot 2024-03-27 at 10.53.48 PM](https://hackmd.io/_uploads/BJUzQ3ZyC.png)

### 3-6. Check the web UI

- username : adminuser
- password : password

**Result :**

https://192.168.8.40:32443/vno-altiplano-ac

![Screenshot 2024-03-27 at 10.55.44 PM](https://hackmd.io/_uploads/S1f9mnZJA.png)
