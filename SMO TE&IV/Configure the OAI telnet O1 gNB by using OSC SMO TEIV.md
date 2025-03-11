# <center>Configure the OAI telnet O1 gNB by using OSC SMO TEIV</center>

<center>SMO O1 Ves + O1 Netconf + OAI O1 Adapter + OAI telnet O1 gNB</center>

###### tags: `SMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Jan2 , 2024 3:38 PM][color=#38c16a]

:::info
**Introduction：**

This note is using TEIV consumer rApp to config the OAI telnet O1 gNB through O1 netconf.

**Goal：**
- [ ] Complete the implementation process of the TEIV consumer rApp.


**Reference：**
- [Release J - Run in Kubernetes](https://wiki.o-ran-sc.org/display/SMO/Release+J+-+Run+in+Kubernetes)
- [[J Release] SMO Topology & Inventory (TEIV)](https://hackmd.io/@Jerry0714/BkkmMXKPR)
- [[J Release] SMO Topology & Inventory (TEIV) Installation Guide](https://hackmd.io/@Jerry0714/HkTXfrUdC)
- [[J Release] SMO Topology & Inventory (TEIV) Developer Guide](https://docs.o-ran-sc.org/projects/o-ran-sc-smo-teiv/en/latest/developer-guide.html#)
- [Gerrit smo/teiv.git](https://gerrit.o-ran-sc.org/r/gitweb?p=smo/teiv.git;a=tree;h=ea1ce6271dfe7d94b004c09d89254c4924c937fc;hb=ea1ce6271dfe7d94b004c09d89254c4924c937fc)
- [O-RAN Work Group 10 Topology Exposure and Inventory Management Services Use Cases and Requirement Specification (O-RAN.WG10.TE&IV-R003-v02.00)](https://specifications.o-ran.org/specifications)
- [Configure the VIAVI RIC Test using OSC SMO TEIV](https://hackmd.io/@Jerry0714/Bk8xKZuTA)
:::
**<center>Table of Content</center>**
[TOC]

## 1. O-RAN network provisioning
### 1-1. System Architecture

<style>
  .img {
    position:relative;
    width: 80%;
    left: 15%;
  }
</style>

<p class="img"><img src="https://hackmd.io/_uploads/B1xmwt_Ske.png"></p>
:::success
**Orange line** is not yet.
**Black line** is already connect.
:::
### 1-2. O-RAN network provisioning Operation
**Spec :** (O-RAN.WG10.TE&IV-R003-v02.00)

**Goal :** O-RAN network provisioning using TE&IV services

**Sequence Diagram**

![Screenshot 2024-09-20 at 6.12.18 AM](https://hackmd.io/_uploads/S1u0WXc60.png)

## 2 . Ves event listening function in TE&IV Service Consumer rApp
This part is to add the function to listen the ves notification in message-router.onap. Because we need to get the PNF registration event, file ready event, fault event and heart beat event to let TE&IV consumer rApp to know the gNB status.

ONAP already define they message handle procedure. The message will go to the following topic define by ONAP.

Reference : https://docs.onap.org/projects/onap-dcaegen2/en/latest/sections/services/ves-http/configuration.html

```javascript=
{
  "collector.dynamic.config.update.frequency": "5",
  "event.transform.flag": "0",
  "collector.schema.checkflag": "1",
  "collector.dmaap.streamid": "fault=ves-fault|syslog=ves-syslog|heartbeat=ves-heartbeat|measurementsForVfScaling=ves-measurement|mobileFlow=ves-mobileflow|other=ves-other|stateChange=ves-statechange|thresholdCrossingAlert=ves-thresholdCrossingAlert|voiceQuality=ves-voicequality|sipSignaling=ves-sipsignaling|notification=ves-notification|pnfRegistration=ves-pnfRegistration|3GPP-FaultSupervision=ves-3gpp-fault-supervision|3GPP-Heartbeat=ves-3gpp-heartbeat|3GPP-Provisioning=ves-3gpp-provisioning|3GPP-PerformanceAssurance=ves-3gpp-performance-assurance",
  "collector.service.port": "8080",
  "collector.schema.file": "{\"v1\":\"./etc/CommonEventFormat_27.2.json\",\"v2\":\"./etc/CommonEventFormat_27.2.json\",\"v3\":\"./etc/CommonEventFormat_27.2.json\",\"v4\":\"./etc/CommonEventFormat_27.2.json\",\"v5\":\"./etc/CommonEventFormat_28.4.1.json\",\"v7\":\"./etc/CommonEventFormat_30.2_ONAP.json\"}",
  "collector.keystore.passwordfile": "/opt/app/VESCollector/etc/passwordfile",
  "streams_publishes": {
    "ves-measurement": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.VES_MEASUREMENT_OUTPUT/"
      }
    },
    "ves-fault": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.SEC_FAULT_OUTPUT/"
      }
    },
    "ves-pnfRegistration": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.VES_PNFREG_OUTPUT/"
      }
    },
    "ves-other": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.SEC_OTHER_OUTPUT/"
      }
    },
    "ves-heartbeat": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.SEC_HEARTBEAT_OUTPUT/"
      }
    },
    "ves-notification": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.VES_NOTIFICATION_OUTPUT/"
      }
    },
    "ves-3gpp-fault-supervision": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.SEC_3GPP_FAULTSUPERVISION_OUTPUT/"
      }
    },
    "ves-3gpp-provisioning": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.SEC_3GPP_PROVISIONING_OUTPUT/"
      }
    },
    "ves-3gpp-heartbeat": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.SEC_3GPP_HEARTBEAT_OUTPUT/"
      }
    },
    "ves-3gpp-performance-assurance": {
      "type": "message_router",
      "dmaap_info": {
        "topic_url": "http://message-router:3904/events/unauthenticated.SEC_3GPP_PERFORMANCEASSURANCE_OUTPUT/"
      }
    }
  },
  "collector.externalSchema.checkflag": 1,
  "collector.externalSchema.schemasLocation": "./etc/externalRepo",
  "collector.externalSchema.mappingFileLocation": "./etc/externalRepo/schema-map.json",
  "event.externalSchema.schemaRefPath": "/event/stndDefinedFields/schemaReference",
  "event.externalSchema.stndDefinedDataPath": "/event/stndDefinedFields/data",
  "collector.service.secure.port": "8443",
  "auth.method": "noAuth",
  "collector.keystore.file.location": "/opt/app/VESCollector/etc/keystore",
  "services_calls": [],
  "header.authlist": "sample1,$2a$10$0buh.2WeYwN868YMwnNNEuNEAMNYVU9.FSMJGyIKV3dGET/7oGOi6"
}
```

So we use the following topic to subscribe and get the event message.
```javascript=
topics = [
        "unauthenticated.SEC_3GPP_PERFORMANCEASSURANCE_OUTPUT",
        "unauthenticated.VES_PNFREG_OUTPUT",
        "unauthenticated.SEC_HEARTBEAT_OUTPUT",
        "unauthenticated.SEC_3GPP_FAULTSUPERVISION_OUTPUT"
    ]
```
And then we can implement the function to get the message from message-router.onap.

**DmaapClient.py**
```javascript=
import os
import json
import logging
import requests
import asyncio

class Configurations:
    def __init__(self):
        self.mr_host = 'http://10.233.33.182'
        self.mr_port = 3904
        self.consumer_group = 'simple-client'
        self.consumer_id = os.environ.get('HOSTNAME', '1')
        self.log_level = 'INFO'
        self.mr_address = self.mr_host + ':' + str(self.mr_port)

class DmaapClient:
    def __init__(self, mr_address, consumer_group, consumer_id):
        self.mr_address = mr_address
        self.consumer_group = consumer_group
        self.consumer_id = consumer_id

    def get_messages(self, topic):
        url = f"{self.mr_address}/events/{topic}/{self.consumer_group}/{self.consumer_id}"
        try:
            logging.info(f"Listening : {url}")
            resp = requests.get(url)
            if resp.status_code == 404:
                logging.warning(f"Topic {topic} not exists")
                return None
            
            if resp.status_code != 200:
                raise Exception(f'MR error: {resp.status_code} {resp.text}')
                
            return resp.json()
        except Exception as e:
            logging.error(f"Failed to get messages: {str(e)}")
            return None

class MessageClient:
    def __init__(self, dmaap_client, topics):
        self.dmaap_client = dmaap_client
        self.topics = topics
        self.terminated = False

    def terminate(self):
        self.terminated = True

    def should_process_message(self, message):
        try:
            event_name = message.get('event', {}).get('commonEventHeader', {}).get('eventName')
            return event_name != 'Viavi_perf3gpp'
        except:
            return True

    async def run_topic(self, topic):
        while not self.terminated:
            logging.debug(f'Pulling messages from topic {topic}...')
            messages = await asyncio.get_event_loop().run_in_executor(
                None, 
                self.dmaap_client.get_messages, 
                topic
            )
            
            if messages:
                for message in messages:
                    try:
                        parsed_message = json.loads(message) if isinstance(message, str) else message
                        # 加入 topic 資訊到日誌中
                        if self.should_process_message(parsed_message):
                            logging.info(f"Received message from {topic}: {parsed_message}")
                        else:
                            logging.debug(f"Filtered out Viavi_perf3gpp message from {topic}")
                    except json.JSONDecodeError as e:
                        logging.error(f"Failed to parse message from {topic}: {str(e)}")
            
            await asyncio.sleep(1)

async def main():
    config = Configurations()
    logging.basicConfig(
        level=logging._nameToLevel[config.log_level],
        format='%(asctime)s - %(levelname)s - %(message)s'
    )

    logging.info('Starting message client')
    logging.info(f'MR address: {config.mr_address}')
    logging.info(f'MR consumer group: {config.consumer_group}')
    logging.info(f'MR consumer id: {config.consumer_id}')

    topics = [
        "unauthenticated.SEC_3GPP_PERFORMANCEASSURANCE_OUTPUT",
        "unauthenticated.VES_PNFREG_OUTPUT",
        "unauthenticated.SEC_HEARTBEAT_OUTPUT",
        "unauthenticated.SEC_3GPP_FAULTSUPERVISION_OUTPUT"
    ]

    dmaap = DmaapClient(config.mr_address, config.consumer_group, config.consumer_id)
    client = MessageClient(dmaap, topics)

    tasks = []
    for topic in topics:
        tasks.append(asyncio.create_task(client.run_topic(topic)))

    try:
        await asyncio.gather(*tasks)
    except Exception as e:
        logging.exception(f'Task failed: {str(e)}')
    finally:
        client.terminate()

if __name__ == '__main__':
    asyncio.run(main())
```
**Result**

- Listening to each topic in message-router.onap. 
![Screenshot 2024-11-28 at 11.40.58 PM](https://hackmd.io/_uploads/ryRfyz8mJg.png)
- Get the PNF registration event from in topic
  (unauthenticated.VES_PNFREG_OUTPUT)
![Screenshot 2024-11-28 at 11.44.04 PM](https://hackmd.io/_uploads/BkT01fLmke.png)
- Get the file ready event in topic
  (unauthenticated.SEC_3GPP_PERFORMANCEASSURANCE_OUTPUT)
![Screenshot 2024-11-28 at 11.44.49 PM](https://hackmd.io/_uploads/HyU-gMLmkx.png)
- Get the heart beat event in topic
  (unauthenticated.SEC_HEARTBEAT_OUTPUT)
![Screenshot 2024-11-28 at 11.45.39 PM](https://hackmd.io/_uploads/B1wNxM8Xyl.png)
- Get the fault event in topic
  (unauthenticated.SEC_3GPP_FAULTSUPERVISION_OUTPUT)
![Screenshot 2024-11-28 at 11.46.58 PM](https://hackmd.io/_uploads/Hyl5eML7yl.png)
### 2-1. To enable the RegistrationManager continuously listen in the background
The **RegistrationManager** will store the entityID who regisiter the rApp, and it will continuously listen in the background.
```javascript=
class RegistrationManager:
    def __init__(self):
        self.registered_entities = set()  # 儲存已註冊的實體
        self.monitor = DmaapMonitor()

    def is_registered(self, assigned_id):
        try:
            managed_element = assigned_id.split("ManagedElement=")[1].split(",")[0]
            return f"ManagedElement={managed_element}" in self.registered_entities
        except Exception as e:
            logger.error(f"Error parsing assigned_id: {e}")
            return False

    async def registration_monitor(self):
        """持續監聽註冊事件的異步任務"""
        while True:
            try:
                messages = await self.monitor.get_messages_async()
                if messages:
                    for message in messages:
                        event = message.get('event', {})
                        common_header = event.get('commonEventHeader', {})
                        if common_header.get('eventType') == 'OAI_pnfRegistration':
                            entity_name = common_header.get('reportingEntityName')
                            if entity_name:
                                self.registered_entities.add(entity_name)
                                logger.info(f"New entity registered: {entity_name}")
            except Exception as e:
                logger.error(f"Error in registration monitor: {e}")
            await asyncio.sleep(1)
```
This part is to checking who is already regisiter the rApp and makesure is there any new configuration in TE&IV service.
```javascript=
async def check_registration(reg_manager, assigned_id, timeout=300):
    start_time = time.time()
    while time.time() - start_time < timeout:
        if reg_manager.is_registered(assigned_id):
            logger.info(f"Entity for {assigned_id} is now registered and has new configuration in TE&IV service")
            return True
        await asyncio.sleep(5)
    logger.warning(f"Timeout waiting for registration of {assigned_id}")
    return False
```
## 3. Implement automatically configuration Result
- **[Status Planned]** First you need to choose the entity you want to config.
- **[Status Assigned]** rApp already assigned the new config to TEIV. Status change to Assigned.
![Screenshot 2024-11-26 at 6.14.57 PM](https://hackmd.io/_uploads/SyR_ex4Qyl.png)
- **[Status Deployed]** Implement the Deployed status. The rApp will listen the any event in the background to get the fault, PNF registration, and so on infornation from gNB.
![Screenshot 2024-11-29 at 4.45.09 PM](https://hackmd.io/_uploads/BJ-TT4n7yl.png)
- **[Status Active]** If the DeployID can match the netconf config in the xml structure, then rApp will compare the attributes between the netconf and TE&IV.
- Thr rApp will using the new config in TE&IV to config the gNB.
![Screenshot 2024-11-27 at 11.13.36 PM](https://hackmd.io/_uploads/ryi2ASn7yl.png)
- We check the new config in OAI telnet O1 gNB.
![Screenshot 2024-11-27 at 11.17.28 PM](https://hackmd.io/_uploads/r1h1xL3myg.png)

## 4. Using Grafana to show the configuration result.

The rapp will automatically to send the metric and netwok event data to influxDB. This implementation is base on reading the ves event.

The following parameter show in the Grafana.

- Network Connection Status
- Network Alarm Status
- Network Connection Status History
- Network Alarm Status History
- gNB Configuration Parameter
- gNB Configuration Status Report
- Total throughput (Gbps)
- gNB Heart Beat Report
- Data Radio Bearer (Number of Users)
- Network Event Logs
- Remote Radio Unit (%)

![Screenshot 2024-12-09 at 4.37.10 PM](https://hackmd.io/_uploads/HJ_JFRCEyx.png)


## 5. Demo video
- https://youtu.be/WMRUBF0ObEE
- https://youtu.be/JhZXl7A6lCM
