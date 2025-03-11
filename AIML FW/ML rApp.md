# <center>ML rApp</center>

<center>Implement ML rApp connect to ES rApp</center>

###### tags: `AI/ML`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Thu, Mar7 , 2024 3:38 PM][color=#38c16a]

:::info
**Introduction :**
- [x] [Create and testing ML rApp.](#2-Create-and-testing-ML-rApp)
- [x] [Using ML rApp to get prediction result.](#3-Using-ML-rApp-to-get-prediction-result)
- [ ] Connent ML rApp to ES rApp.
- [ ] ES rApp use ML rapp prediction result to control.
:::
**<center>Table of Content</center>**
[TOC]

## 1. System Architecture
![dd](https://hackmd.io/_uploads/Bkq5hxAaa.png)

## 2. Create and testing ML rApp
### 2-1. Revise the config to generate the rApp kserve-instance

**Resource :** [kserve-instance.json](https://gerrit.o-ran-sc.org/r/gitweb?p=nonrtric/plt/rappmanager.git;a=blob;f=csar-generator/resources/Files/Acm/instances/kserve-instance.json;h=aaf63a1f78f1687791897706ead86e8bee2649a8;hb=refs/heads/i-release)

Use this resource to Revise kserve-instance.json and enter to the below path to revise the kserve-instance.json.
```javascript=
cd winnie/rappmanager/sample-rapp-generator/ml-kserve/Files/Acm/instances
vim kserve-instance.json
```
- **AI/ML storageUri :** http://192.168.8.44:32002/model/rictestmodel11/1/Model.zip


```javascript=
{
  "name": "KserveInstance2",
  "version": "1.0.1",
  "compositionId": "DO_NOT_CHANGE_THIS_COMPOSITION_ID",
  "description": "Demo automation composition instance 0",
  "elements": {
    "3e90b916-7f9a-48ee-80ab-1b547d441e41": {
      "id": "3e90b916-7f9a-48ee-80ab-1b547d441e41",
      "definition": {
        "name": "onap.policy.clamp.ac.element.KserveAutomationCompositionElement",
        "version": "1.2.3"
      },
      "description": "Starter Automation Composition Element for the Demo",
      "properties": {
        "kserveInferenceEntities": [
          {
            "kserveInferenceEntityId": {
              "name": "entity1",
              "version": "1.0.1"
            },
            "name": "ML-rApp",
            "namespace": "kserve-test",
            "payload": "{\"apiVersion\": \"serving.kserve.io/v1beta1\",\"kind\": \"InferenceService\",\"metadata\": {\"name\": \"qoe-model\"},\"spec\": {\"predictor\": {\"model\":{\"modelFormat\": {\"name\": \"tensorflow\"},\"storageUri\": \"http://192.168.8.44:32002/model/rictestmodel11/1/Model.zip\"}}}}"
          }
        ]
      }
    }
  }
}
```
This modification is based on the configuration previously deployed on the AI/ML platform.
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
Resource :[【H Release】User Guide of AIMLFW](https://hackmd.io/@Jerry0714/HJmhjj-kT)
### 2-2. Generate rApp .csar file.

Using this user guide to create rApp kserve instance.
Resource : https://hackmd.io/@Winnie27/rJjXkxatp

```javascript=
cd ../../../..
./generate.sh ml-kserve
```

### 2-3. Deploy rApp kserve-instance 

#### 2-3-1. Onboard rApp

```javascript=
curl -i -X POST -H "Content-Type: multipart/form-data" -F "file=@ml-kserve.csar" http://10.233.61.144:8080/rapps/rappid1
```

![Screenshot 2024-03-12 at 12.31.37 AM](https://hackmd.io/_uploads/H1VbM33aa.png)

#### 2-3-2. Prime rApp

```javascript=
cd winnie/rappmanager/deploy_rapp
```
```javascript=
curl -i -X PUT -H "Content-Type: application/json" -d @prime.json http://10.233.61.144:8080/rapps/rappid1
```
![Screenshot 2024-03-12 at 12.30.31 AM](https://hackmd.io/_uploads/rkC3b23aa.png)

#### 2-3-3. Create rApp kserve-instance
```javascript=
curl -i -X POST -H "Content-Type: application/json" -d @createrappinstance.json http://10.233.61.144:8080/rapps/rappid1/instance
```
![Screenshot 2024-03-12 at 12.27.05 AM](https://hackmd.io/_uploads/HJylZ33p6.png)

#### 2-3-4. Deploy rApp kserve-instance
```javascript=
curl -i -X PUT -H "Content-Type: application/json" -d @deploy.json http://10.233.61.144:8080/rapps/rappid1/instance/{rApp instance ID}
```

![Screenshot 2024-03-12 at 7.29.15 PM](https://hackmd.io/_uploads/BkRc2hTp6.png)

Check successful or not.

```javascript=
kubectl logs rappmanager-0 -n nonrtric -f
```

:::spoiler Deploy Failed resolved.
:::danger
**Deploy Failed**

HTTP/1.1 502 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Mon, 11 Mar 2024 16:33:18 GMT

{"message":"Unable to deploy ACM"}

![Screenshot 2024-03-12 at 12.35.31 AM](https://hackmd.io/_uploads/ryFym3nap.png)

You can check rApp manager what issue occur.

```javascript=
kubectl logs rappmanager-0 -n nonrtric -f
```

Some parameter you need to notice.
- **"name": "KserveInstance0"** //This must be unused.
- **"compositionId": "DO_NOT_CHANGE_THIS_COMPOSITION_ID"** //Don't chang this parameter.
- **"3e90b916-7f9a-48ee-80ab-1b547d441e41": {
      "id": "3e90b916-7f9a-48ee-80ab-1b547d441e41",** //This is UUID format, this UUID should be unique.
:::

#### 2-3-5. Check pod is running or not.

```javascript=
kubectl get pod -n kserve-test
```

![Screenshot 2024-03-12 at 8.19.19 PM](https://hackmd.io/_uploads/BytLd6TTp.png)

You can also check rappmanager.

```javascript=
cd winnie/rappmanager/scripts/demo
sudo ./view.sh
```

![Screenshot 2024-03-12 at 8.21.48 PM](https://hackmd.io/_uploads/S16yKTTaa.png)

Kserve Participant already deploy now.

## 3. Using ML rApp to get prediction result.

Check your ML rApp IP and port.

```javascript=
kubectl get svc -n kserve-test
```

![Screenshot 2024-03-12 at 8.47.40 PM](https://hackmd.io/_uploads/Hye-J0T6a.png)

- **ML rApp IP :** 10.233.52.212
- **ML rApp port :** 80

Create predict_inference.sh file with following contents
```javascript=
vim predict_inference.sh
```
```javascript=
model_name=qoe-model
curl -v -H "Host: $model_name.kserve-test.example.com" http://"ML rApp IP":"ML rApp port"/v1/models/$model_name:predict -d @./input_qoe.json
```

![Screenshot 2024-03-12 at 8.57.47 PM](https://hackmd.io/_uploads/rJbvWCpT6.png)


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
### 3-1. Result ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)
Use this command to trigger prediction.
```javascript!
source predict_inference.sh
```
**Result :**

![Screenshot 2024-03-12 at 8.59.55 PM](https://hackmd.io/_uploads/ryG1MA6T6.png)

## 4. Connent ML rApp to ES rApp