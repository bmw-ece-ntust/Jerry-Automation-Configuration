# <center>Qoe assist xapp connect to ML xapp</center>

<center>The implementation of Qoe assist xapp on Near-RT RIC</center>

###### tags: `AI/ML`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue, Nov 4 , 2023 8:39 PM][color=#38c16a]

:::info
**Introduction :**

**Goal：**
- [x] [Prepare Qoe assist xapp](https://hackmd.io/@Jerry0714/rkB__nmXp#1-Prepare-Qoe-assist-xapp)
- [x] [Deploy Qoe assist xapp](https://hackmd.io/@Jerry0714/rkB__nmXp#1-4-Deploy-Qoe-assist-xapp)
- [x] [Prepare Qoe driver xapp](https://hackmd.io/@Jerry0714/rkB__nmXp#2-Prepare-Qp-driver-xapp)
- [x] [Deploy Qoe driver xapp](https://hackmd.io/@Jerry0714/rkB__nmXp#2-3-Deploy-Qp-driver-xapp)
- [ ] Qp-driver xApp send prediction request to Qoe assist xApp.
:::
**<center>Table of Content</center>**
[TOC]

## System Architecture
![www](https://hackmd.io/_uploads/Hyp9paPCT.png)

## 1. Prepare Qoe assist xapp 
```javascript!
git clone "https://gerrit.o-ran-sc.org/r/ric-app/qp-aimlfw"
```
### 1-1. Build Image
```javascript=
cd qp-aimlfw
docker build -f Dockerfile .
```

![Screenshot from 2023-11-04 20-50-48.png](https://hackmd.io/_uploads/By--01476.png)

### 1-2. Check and revise the below config file.
```javascript=
cd config
vim config-file.json
```
**config-file.json**
```javascript=
{
    "name": "qoe-aiml-assist",
    "version": "1.0.0",
    "containers": [
        {
            "name": "qoe-aiml-assist",
            "image": {
                "registry": "nexus3.o-ran-sc.org:10002",
                "name": "o-ran-sc/qoe-aiml-assist",
                "tag": "1.0.0"
            },
            "args": [
                "-c",
                "./qoe-aiml-assist -f /opt/ric/config/config-file.json"
            ],
            "command": [
                "/bin/sh"
            ]
        }
    ],
    "livenessProbe": {
        "httpGet": {
            "path": "ric/v1/health/alive",
            "port": 8080
        },
        "initialDelaySeconds": 5,
        "periodSeconds": 15
    },
    "readinessProbe": {
        "httpGet": {
            "path": "ric/v1/health/ready",
            "port": 8080
        },
        "initialDelaySeconds": 5,
        "periodSeconds": 15
    },
    "appenv": {
        "INFLUX_URL": "",
        "INFLUX_TOKEN": "",
        "INFLUX_BUCKET": "aiml",
        "INFLUX_ORG": "primary",
        "INFLUX_QUERY_START": "-1m",
        "INFLUX_QUERY_STOP": "",
        "RIC_MSG_BUF_CHAN_LEN": 256,
        "MLXAPP_HEADERHOST": "qoe-model.kserve-test.example.com",
        "MLXAPP_HOST": "",
        "MLXAPP_PORT": "3",
        "MLXAPP_REQURL": "v1/models/qoe-model:predict"
    },
    "logger": {
        "level": 5
    },
    "messaging": {
        "ports": [
            {
                "name": "http",
                "container": "qoe-aiml-assist",
                "port": 8080,
                "description": "http service"
            },
            {
                "name": "rmrroute",
                "container": "qoe-aiml-assist",
                "port": 4561,
                "description": "rmr route port for qoe-aiml-assist xapp"
            },
            {
                "name": "rmrdata",
                "container": "qoe-aiml-assist-go",
                "port": 4560,
                "rxMessages": [
                    "RIC_HEALTH_CHECK_REQ",
                    "TS_UE_LIST",
                    "TS_QOE_PRED_REQ"
                ],
                "txMessages": [
                    "RIC_HEALTH_CHECK_RESP",
                    "TS_QOE_PREDICTION"
                ],
                "mtypes" : [
                    {
                        "name":"TESTNAME1",
                        "id":55555
                    },
                    {
                        "name":"TESTNAME2",
                        "id":55556
                    }
                ],
                "policies": [
                    1
                ],
                "description": "rmr data port for qoe-aiml-assist-go"
            }
        ]
    },
    "rmr": {
        "protPort": "tcp:4560",
        "maxSize": 2072,
        "numWorkers": 1,
        "txMessages": [
            "RIC_HEALTH_CHECK_RESP",
            "TS_QOE_PREDICTION"
        ],
        "rxMessages": [
            "RIC_HEALTH_CHECK_REQ",
            "TS_UE_LIST",
            "TS_QOE_PRED_REQ"
        ],
        "policies": [
            1
        ]
    },
    "controls": {
        "fileStrorage": false,
        "logger": {
            "level": 5
        }
    },
    "db" : {
        "waitForSdl": false
    }
}
```
Revise **"appenv"** in the line **"37~49"** of **"config-file.json"** as the below.
```javascript=37
"appenv": {
        "INFLUX_URL": "http://10.0.10.121:32086",
        "INFLUX_TOKEN": "5kGlGI19vJvhzvWRwL5wwoecjKUUg6Ti",
        "INFLUX_BUCKET": "UEdata",
        "INFLUX_ORG": "my-org",
        "INFLUX_QUERY_START": "-10m",
        "INFLUX_QUERY_STOP": "",
        "RIC_MSG_BUF_CHAN_LEN": "256",    
        "MLXAPP_HEADERHOST": "sample-xapp.ricips.example.com",
        "MLXAPP_HOST": "http://10.0.10.121",
        "MLXAPP_PORT": "30166",
        "MLXAPP_REQURL": "v1/models/sample-xapp:predict"
    },
```

### 1-3. Onboard Qoe assist xapp
```javascript=
dms_cli onboard --config_file_path=config/config-file.json --shcema_file_path=config/schema.json
```
If you cat't onboard xapp like this figure.

![Screenshot from 2023-11-05 00-25-07.png](https://hackmd.io/_uploads/ryQOgxEm6.png)

Try to **"delete"** line **"114~119"** as the below.
```javascript=114
"controls": {
        "fileStrorage": false,
        "logger": {
            "level": 5
        }
    },
```

After revise the file, onboard again.

**Result :**

![Screenshot from 2023-11-05 00-27-21.png](https://hackmd.io/_uploads/H1de-eNmp.png)

Check the upload chart.
```javascript=
dms_cli get_charts_list
```

![Screenshot from 2023-11-05 00-29-35.png](https://hackmd.io/_uploads/H1yFWlEQ6.png)

### 1-4. Deploy Qoe assist xapp
Use this command to build Qoe assist xapp image.

```javascript=
docker build -t nexus3.o-ran-sc.org:10002/o-ran-sc/qoe-aiml-assist:1.0.0 .
```

Use this command to deploy.
```javascript=
dms_cli install --xapp_chart_name=qoe-aiml-assist --version=1.0.0 --namespace=ricxapp
```

![Screenshot 2024-03-12 at 4.35.36 PM](https://hackmd.io/_uploads/ByCHVcT6a.png)

Check the xApp is regisition or not.

```javascript=
kubectl logs ricxapp-qoe-aiml-assist-7f7d74f47-fqgjx -f -n ricxapp
```

**Regisition done**

![Screenshot 2024-03-12 at 4.43.13 PM](https://hackmd.io/_uploads/HktRrqap6.png)

## 2. Prepare Qp driver xapp
```javascript=
git clone "https://gerrit.o-ran-sc.org/r/ric-app/qp-driver"
```
### 2-1. Build Image
```javascript=
cd qp-driver/
docker build -f Dockerfile .
```
### 2-2. Onboard Qp driver xapp
```javascript=
dms_cli onboard --config_file_path=xapp-descriptor/config.json --shcema_file_path=xapp-descriptor/controls.json
```
### 2-3. Deploy Qp driver xapp

Use this command to build Qp driver xapp image.

```javascript=
docker build -t nexus3.o-ran-sc.org:10002/o-ran-sc/ric-app-qp-driver:1.1.0 .
```

Use this command to deploy.
```javascript=
dms_cli install --xapp_chart_name=qpdriver --version=1.1.0 --namespace=ricxapp
```

:::spoiler ModuleNotFoundError: No module named 'redis._compat'
:::danger
**ERROR : ModuleNotFoundError: No module named 'redis._compat'**
```javascript=
kubectl logs ricxapp-qpdriver-757d9d9f97-jjxzv -n ricxapp -f
```
![Screenshot 2024-03-15 at 7.06.25 PM](https://hackmd.io/_uploads/HyvvhjWCp.png)

The reason for this issue is that the original installation content of OSC lacked Python dependencies. Therefore, the Python files inside Docker cannot find the missing installation files. We can resolve this by modifying setup.py to include the additional items to be installed.

```javascript=
cd qp-driver/
vim setup.py
```
```javascript=
from setuptools import setup, find_packages

setup(
    name="qpdriver",
    version="1.1.0",
    packages=find_packages(exclude=["tests.*", "tests"]),
    author="O-RAN-SC Community",
    description="QP Driver Xapp for traffic steering use case",
    url="https://gerrit.o-ran-sc.org/r/admin/repos/ric-app/qp-driver",
    install_requires=["ricxappframe>=1.3.0,<2.0.0", "redis-py-cluster==1.3.6"],
    entry_points={"console_scripts": ["start-qpd.py=qpdriver.main:start"]},  # adds a magical entrypoint for Docker
    license="Apache 2.0",
    data_files=[("", ["LICENSE.txt"])],
)
```
In this file I add the new item in line 10
```javascript=10
"redis-py-cluster==1.3.6"
```
After you revise, you need to build the Dockerfile again.
```javascript=
docker build -f Dockerfile .
```
And then you will recieve the new image "05de43e9b2f1".You can delete the old images which you created before.
```javascript=
docker rmi 51782c5bde91
```
or 
```javascript=
docker rmi nexus3.o-ran-sc.org:10002/o-ran-sc/ric-app-qp-driver:1.1.0
```
Add the tag to your new image.
```javascript=
docker tag 05de43e9b2f1 nexus3.o-ran-sc.org:10002/o-ran-sc/ric-app-qp-driver:1.1.0
```
Check your pod have the redis-py-cluster or not.
```javascript=
kubectl exec -it ricxapp-qpdriver-757d9d9f97-kjcbv -n ricxapp -- sh
```
```javascript=
cd usr/local/lib/python3.8/site-packages/
```
![Screenshot 2024-03-15 at 7.25.35 PM](https://hackmd.io/_uploads/ryZSl2-Ra.png)

Check your logs again.
```javascript=
kubectl logs ricxapp-qpdriver-757d9d9f97-kjcbv -n ricxapp -f
```
![Screenshot 2024-03-15 at 7.27.19 PM](https://hackmd.io/_uploads/ryQngnWR6.png)

It is no error now.
:::

## 3. Qp-driver xApp send prediction request to Qoe assist xApp.

Check the Qp-driver xApp log.
```javascript=
kubectl logs ricxapp-qpdriver-757d9d9f97-nw5v9 -n ricxapp -f
```

But it can't send the request to the  Qoe assist xApp.

![Screenshot 2024-03-19 at 9.31.41 PM](https://hackmd.io/_uploads/SJzCXfDRa.png)














## Appendix
:::info
**Some dms_cli command I use :**
- Set up $CHART_REPO_URL
```javascript=
export NODE_PORT=$(kubectl get --namespace ricinfra -o jsonpath="{.spec.ports[0].nodePort}" services r4-chartmuseum-chartmuseum)
export NODE_IP=$(kubectl get nodes --namespace ricinfra -o jsonpath="{.items[0].status.addresses[0].address}")
export CHART_REPO_URL=http://$NODE_IP:$NODE_PORT/charts
echo $CHART_REPO_URL
```
- Check all the charts
```javascript=
dms_cli get_charts_list
```
- Onboard chart
```javascript=
dms_cli onboard --config_file_path=config/config-file.json --shcema_file_path=config/schema.json
```
- Deploy xApp
```javascript=
dms_cli install --xapp_chart_name=qoe-aiml-assist --version=1.0.0 --namespace=ricxapp
```
- Undeploy xApp
```javascript=
dms_cli uninstall --xapp_chart_name=qoe-aiml-assist --namespace=ricxapp
dms_cli uninstall --xapp_chart_name=qpdriver --namespace=ricxapp
```
- Delete chart
```javascript=
curl -X DELETE ${CHART_REPO_URL}/api/charts/qoe-aiml-assist/1.0.0
curl -X DELETE ${CHART_REPO_URL}/api/charts/qpdriver/1.1.0
```
- Degister xApp
```javascript=
curl -X 'POST' "http://$(kubectl get svc -A|grep appmgr|grep 8080|awk '{print $4}'):8080/ric/v1/deregister" -H 'accept: application/json' -H 'Content-Type: application/json' -d '{
"appName": "qoe-aiml-assist",
"appInstanceName": "qoe-aiml-assist"
}'

curl -X 'POST' "http://$(kubectl get svc -A|grep appmgr|grep 8080|awk '{print $4}'):8080/ric/v1/deregister" -H 'accept: application/json' -H 'Content-Type: application/json' -d '{
"appName": "qpdriver",
"appInstanceName": "qpdriver"
}'
```
- Delete api for xApp
```javascript=
curl -X DELETE 0.0.0.0:8080/api/charts/qoe-aiml-assist/1.0.0
curl -X DELETE 0.0.0.0:8080/api/charts/qpdriver/1.1.0
```
:::


```javascript=
curl -X 'POST' 'http://10.103.208.192:8080/ric/v1/register' \
-H 'accept: application/json' \
-H 'Content-Type: application/json' \
-d '{
  "appName": "qpdriver",
  "appVersion": "1.1.0",
  "appInstanceName": "qpdriver",
  "httpEndpoint": "",
  "rmrEndpoint": "10.105.140.116:4560",
  "config": "{\"name\":\"qpdriver\",\"version\":\"1.1.0\",\"containers\":[{\"image\":{\"name\":\"o-ran-sc/ric-app-qp-driver\",\"registry\":\"nexus3.o-ran-sc.org:10002\",\"tag\":\"1.1.0\"},\"name\":\"qpdriver\"}],\"messaging\":{\"ports\":[{\"container\":\"qpdriver\",\"description\":\"rmr route port for qpdriver\",\"name\":\"rmr-route\",\"port\":4561},{\"container\":\"qpdriver\",\"description\":\"rmr receive data port for qpdriver\",\"name\":\"rmr-data\",\"policies\":[],\"port\":4560,\"rxMessages\":[\"TS_UE_LIST\"],\"txMessages\":[\"TS_QOE_PRED_REQ\",\"RIC_ALARM\"]}]},\"rmr\":{\"protPort\":\"tcp:4560\",\"maxSize\":2072,\"numWorkers\":1,\"txMessages\":[\"TS_QOE_PRED_REQ\",\"RIC_ALARM\"],\"rxMessages\":[\"TS_UE_LIST\"],\"policies\":[]},\"controls\":{\"example_int\":10000,\"example_str\":\"value\"}}"
}'
```

service-ricxapp-qoe-aiml-assist-rmr