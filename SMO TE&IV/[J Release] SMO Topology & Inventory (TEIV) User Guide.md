# <center>[J Release] SMO </br>Topology & Inventory (TEIV) </br>User Guide </center>

<center>User Guide for J Release SMO</center>

###### tags: `SMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Jan2 , 2024 3:38 PM][color=#38c16a]

:::info
**Introduction：**

Topology Exposure & Inventory is a tool for managing telecommunications network structures. It describes various equipment in the network and their relationships in a universal way, independent of specific vendors.

**Goal：**
- [x] [J Release] SMO Topology & Inventory (TEIV) Testing

**Reference：**
- [Release J - Run in Kubernetes](https://wiki.o-ran-sc.org/display/SMO/Release+J+-+Run+in+Kubernetes)
- [[J Release] SMO Topology & Inventory (TEIV)](https://hackmd.io/@Jerry0714/BkkmMXKPR)
- [[J Release] SMO Topology & Inventory (TEIV) Installation Guide](https://hackmd.io/@Jerry0714/HkTXfrUdC)
- [[J Release] SMO Topology & Inventory (TEIV) Developer Guide](https://docs.o-ran-sc.org/projects/o-ran-sc-smo-teiv/en/latest/developer-guide.html#)
- [Gerrit smo/teiv.git](https://gerrit.o-ran-sc.org/r/gitweb?p=smo/teiv.git;a=tree;h=ea1ce6271dfe7d94b004c09d89254c4924c937fc;hb=ea1ce6271dfe7d94b004c09d89254c4924c937fc)

:::
**<center>Table of Content</center>**

[TOC]

## System Architecture

![tty](https://hackmd.io/_uploads/S1OFix4Nkl.png)

- Server : 192.168.8.27
- Username : smo-j
## 1. SMO TEIV Supported domains

<table style="border-collapse: collapse; width: 100%;">
  <thead>
    <tr style="background-color: #f8f9fa;">
      <th style="border: 1px solid #dee2e6; padding: 8px; text-align: left;">Domain</th>
      <th style="border: 1px solid #dee2e6; padding: 8px; text-align: left;">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr style="background-color: #f8f9fa;">
      <td style="border: 1px solid #dee2e6; padding: 8px;">RAN</td>
      <td style="border: 1px solid #dee2e6; padding: 8px;">This model contains the topology entities and relations in the RAN domain, which represents the functional capability of the deployed RAN that are relevant to rApps use cases.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #dee2e6; padding: 8px;">EQUIPMENT</td>
      <td style="border: 1px solid #dee2e6; padding: 8px;">This model contains the topology entities and relations in the Equipment domain, which is modeled to understand the physical location of equipment such as antennas associated with a cell/carrier and their relevant properties, for example, tilt, max power, and so on.</td>
    </tr>
    <tr style="background-color: #f8f9fa;">
      <td style="border: 1px solid #dee2e6; padding: 8px;">OAM</td>
      <td style="border: 1px solid #dee2e6; padding: 8px;">This model contains the topology entities and relations in the O&M domain, which are intended to represent management systems and management interfaces.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #dee2e6; padding: 8px;">CLOUD</td>
      <td style="border: 1px solid #dee2e6; padding: 8px;">This model contains the topology entities and relations in the RAN CLOUD domain, which comprises cloud infrastructure and deployment aspects that can be used in the topology model.</td>
    </tr>
    <tr style="background-color: #f8f9fa;">
      <td style="border: 1px solid #dee2e6; padding: 8px;">EQUIPMENT_TO_RAN</td>
      <td style="border: 1px solid #dee2e6; padding: 8px;">This model contains the topology relations between Equipment and RAN.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #dee2e6; padding: 8px;">OAM_TO_RAN</td>
      <td style="border: 1px solid #dee2e6; padding: 8px;">This model contains the topology relations between O&M and RAN.</td>
    </tr>
    <tr style="background-color: #f8f9fa;">
      <td style="border: 1px solid #dee2e6; padding: 8px;">CLOUD_TO_RAN</td>
      <td style="border: 1px solid #dee2e6; padding: 8px;">This model contains the RAN Cloud to RAN Logical topology relations.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #dee2e6; padding: 8px;">OAM_TO_CLOUD</td>
      <td style="border: 1px solid #dee2e6; padding: 8px;">This model contains the RAN O&M to Cloud topology relations.</td>
    </tr>
  </tbody>
</table>

## 2. SMO TEIV Query API examples
### 2-1. Retrieving and using topology modules
**J Release SMO Topology & Inventory (TEIV)**
- Host : 192.168.8.27
- User : j-smo


Install jq to view the json format.
```javascript=
sudo -i
apt install jq
```
Check TEIV pod IP and port.
```javascript=
kubectl get pod -o wide -A
```

![image](https://hackmd.io/_uploads/B1yF6CBsC.png)

```javascript=
kubectl describe pod topology-exposure-54887cd47c-wmrcf -n smo
```

![image](https://hackmd.io/_uploads/r1Cfy18iC.png)

- **IP :** 10.233.102.133
- **Port :** 8080

**Sample request to fetch a list of all modules:**
```javascript=
curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.133:8080/topology-inventory/v1alpha11/schemas | jq
```

![Screenshot 2024-08-23 at 3.39.01 PM](https://hackmd.io/_uploads/HJIjh2SiC.png)
![Screenshot 2024-08-23 at 3.40.26 PM](https://hackmd.io/_uploads/r12lpnSsR.png)

To get a list of all modules for a specific domain, use a domain query parameter. For example, /schemas?domain="domain"

**Sample request to fetch a list of all modules related to the RAN domain:**

```javascript=
curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.133:8080/topology-inventory/v1alpha11/schemas?domain=ran | jq
```

![Screenshot 2024-08-23 at 3.41.49 PM](https://hackmd.io/_uploads/rJAHTnSiR.png)

**Sample request to fetch the module data for the o-ran-smo-teiv-ran module:**

```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.102.133:8080/topology-inventory/v1alpha11/schemas/o-ran-smo-teiv-ran/content
```
![Screenshot 2024-08-23 at 3.56.13 PM](https://hackmd.io/_uploads/H1ynxpHjC.png)

### 2-1. Reading and querying topology and inventory
To get a list of all entities with all properties in a specified domain name, use: > /domains/{domainName}/entities

**Get all the domain first:**
```javascript=
curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.133:8080/topology-inventory/v1alpha11/domains | jq
```
![Screenshot 2024-08-23 at 4.12.24 PM](https://hackmd.io/_uploads/HkidNTBjC.png)
![Screenshot 2024-08-23 at 4.12.55 PM](https://hackmd.io/_uploads/HJ2q4aHiC.png)

**For RAN as example :**
- **Get all entities in the RAN domain:**
```javascript=
curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.133:8080/topology-inventory/v1alpha11/domains/RAN/entities | jq
```
- **Get all entity types in the RAN domain:**
```javascript=
curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.133:8080/topology-inventory/v1alpha11/domains/RAN/entity-types | jq
```
![Screenshot 2024-08-23 at 4.25.40 PM](https://hackmd.io/_uploads/Skc9DaSsR.png)
![Screenshot 2024-08-23 at 4.33.46 PM](https://hackmd.io/_uploads/ryTuF6HoC.png)

- **Get all relationship types in the RAN domain:**

```javascript=
curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.133:8080/topology-inventory/v1alpha11/domains/RAN/relationship-types | jq
```
![Screenshot 2024-08-23 at 4.37.22 PM](https://hackmd.io/_uploads/r1jL56So0.png)
![Screenshot 2024-08-23 at 4.38.37 PM](https://hackmd.io/_uploads/S1gjc6roR.png)

**Querying entities and relationships**
- **targetFilter :** SELECT
- **scopeFilter :** WHERE

If I only interested in NRCellDU entities. Moreover, the user only wants those records that have sourceIds containing “SubNetwork=Ireland”. These fields and filters can be defined in the request as follows:
```javascript=
curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.133:8080/topology-inventory/v1alpha11/domains/RAN?targetFilter=/NRCellDU&scopeFilter=/sourceIds[contains(@item,'SubNetwork=Ireland')] | jq
```
## 3. Topology & Inventory Data Models

For this part you can refer this note [click here.](https://hackmd.io/@Jerry0714/BkkmMXKPR#3-Topology-amp-Inventory-Data-Models)

## 4. Example of enrich Topology & Inventory with geographical data
Geographical enrichment is the adding, modifying, and removing of geographical entities that supports geographical information. Geographical entities are associated to topology data.

Topology & Inventory uses [CloudEvents Kafka®](https://cloudevents.github.io/sdk-java/kafka.html) version 2.5.x.

Use this script to produce CloudEvents.
```javascript=
cd teiv/docker-compose/cloudEventProducer/
vim cloudEventProducerfork8s2.sh 
```
```javascript=
#!/bin/bash

KAFKA_NAMESPACE="${1:-smo}"
KAFKA_TOPIC="${2:-topology-inventory-ingestion}"
KAFKA_POD_NAME="${3:-oran-smo-kafka-controller-0}"
FILE_DUPLICATION_FACTOR="${4:-1}" # How many times to use the same CloudEvent file
FOLDER_OF_CLOUDEVENTS="${5:-events}" # All files in this folder are written to kafka
# number of generated files == number of files in the $FOLDER_OF_CLOUDEVENTS * $FILE_DUPLICATION_FACTOR

echo "Producing messages to Kafka topic '$KAFKA_TOPIC'..."

# Loop to produce multiple messages
for ((i=0; i<$FILE_DUPLICATION_FACTOR; i++)); do
  for file in "$FOLDER_OF_CLOUDEVENTS"/*; do
    if [ -f "$file" ]; then
      # Replace the new line characters with a space and write to kafka
      sed ':a;N;$!ba;s/\n/ /g' $file | \
        kubectl exec -i "$KAFKA_POD_NAME" -n "$KAFKA_NAMESPACE" -- \
        kafka-console-producer.sh \
        --bootstrap-server oran-smo-kafka.smo.svc.cluster.local:9092 --topic "$KAFKA_TOPIC" \
        --property parse.headers=true \
        --property headers.key.separator=::: \
        --property headers.delimiter=,,, \
        --property parse.key=false
    fi
  done
  echo "$((i+1))/$FILE_DUPLICATION_FACTOR rounds completed"
done

echo "Message production completed."
```
### 4-1. Enrich Topology & Inventory with geographical data
We use NRCellDU for example, create geographical data is the same way.

This example creates a new NRCellDU in RAN domain. We use this method to create the new data ==ce_type:::ran-logical-topology.create==
```javascript=
cd teiv/docker-compose/cloudEventProducer/events/
vim cloudEventExampleMerge.txt
```
```javascript=
ce_specversion:::1.0,ce_id:::26a2bb70-cd05-4e73-bbc9-860a17a1a22a,ce_source:::dmi-plugin:nm-1,ce_type:::ran-logical-topology.create,content-type:::application/yang-data+json,ce_time:::2024-09-16T09:05:00Z,ce_dataschema:::https://ties:8080/schemas/v1/r1-topology,,,{
  "entities": [
    {
      "o-ran-smo-teiv-ran:NRCellDU": [
        {
          "id": "urn:3gpp:dn:ManagedElement=NR01,GNBDUFunction=1,NRCellDU=1",
          "attributes": {
            "cellLocalId": 1,
            "nRTAC": 50,
            "nRPCI": 1213,
            "nCI": 155
          },
          "sourceIds": [
            "urn:3gpp:dn:ManagedElement=NR01,GNBDUFunction=1,NRCellDU=1"
          ]
        },
        {
          "id": "urn:3gpp:dn:ManagedElement=NR02,GNBDUFunction=1,NRCellDU=2",
          "attributes": {
            "cellLocalId": 2,
            "nRTAC": 50,
            "nRPCI": 99,
            "nCI": 234
          },
          "sourceIds": [
            "urn:3gpp:dn:ManagedElement=NR01,GNBDUFunction=1,NRCellDU=1"
          ]
        }
      ]
    }
  ]
}
```
```javascript=
cd ..
./cloudEventProducerfork8s2.sh 
```
Check topology-exposure.
NRCellDU already have 2 entities in the topology.

```javascript=
curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.166:8080/topology-inventory/v1alpha11/domains/CLOUD_TO_RAN/entity-types/NRCellDU/entities | jq
```

![Screenshot 2024-09-16 at 3.06.31 AM](https://hackmd.io/_uploads/ByFHgh4a0.png)

We check the attributes for each NRCellDU.

```javascript=
curl -k --http2 --header "Accept: application/yang.data+json, application/problem+json" -X GET http://10.233.102.166:8080/topology-inventory/v1alpha11/domains/CLOUD_TO_RAN/entity-types/NRCellDU/entities/urn:3gpp:dn:ManagedElement=NR01,GNBDUFunction=1,NRCellDU=1 | jq
```
```javascript=
curl -k --http2 --header "Accept: application/yang.data+json, application/problem+json" -X GET http://10.233.102.166:8080/topology-inventory/v1alpha11/domains/CLOUD_TO_RAN/entity-types/NRCellDU/entities/urn:3gpp:dn:ManagedElement=NR02,GNBDUFunction=1,NRCellDU=2 | jq
```

![Screenshot 2024-09-16 at 3.11.33 AM](https://hackmd.io/_uploads/Sk5iZn4pC.png)


Check pgadmin tools.

![Screenshot 2024-09-16 at 3.17.30 AM](https://hackmd.io/_uploads/BySJm3VTA.png)


:::spoiler **(Resolved)** Error The target replication factor of 3 cannot be reached because only 1 broker(s) are registered.
:::danger
**ERROR : The target replication factor of 3 cannot be reached because only 1 broker(s) are registered.**

![Screenshot 2024-09-11 at 5.10.11 AM](https://hackmd.io/_uploads/r1KgI4Rn0.png)

**Problem Description :**
**Producing CloudEvents**
Use this script to send the message to the topic.

```javascript=
#!/bin/bash

KAFKA_NAMESPACE="${1:-smo}"
KAFKA_TOPIC="${2:-topology-inventory-ingestion}"
KAFKA_POD_NAME="${3:-oran-smo-kafka-controller-0}"
FILE_DUPLICATION_FACTOR="${4:-1}" # How many times to use the same CloudEvent file
FOLDER_OF_CLOUDEVENTS="${5:-events}" # All files in this folder are written to kafka
# number of generated files == number of files in the $FOLDER_OF_CLOUDEVENTS * $FILE_DUPLICATION_FACTOR

echo "Producing messages to Kafka topic '$KAFKA_TOPIC'..."

# Loop to produce multiple messages
for ((i=0; i<$FILE_DUPLICATION_FACTOR; i++)); do
  for file in "$FOLDER_OF_CLOUDEVENTS"/*; do
    if [ -f "$file" ]; then
      # Replace the new line characters with a space and write to kafka
      sed ':a;N;$!ba;s/\n/ /g' $file | \
        kubectl exec -i "$KAFKA_POD_NAME" -n "$KAFKA_NAMESPACE" -- \
        kafka-console-producer.sh \
        --broker-list 10.233.102.145:9092 --topic "$KAFKA_TOPIC" \
        --property parse.headers=true \
        --property headers.key.separator=::: \
        --property headers.delimiter=,,, \
        --property parse.key=false
    fi
  done
  echo "$((i+1))/$FILE_DUPLICATION_FACTOR rounds completed"
done

echo "Message production completed."
```

**Consuming CloudEvents**
```javascript=
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'topology-inventory-ingestion',
    bootstrap_servers=['10.233.102.135:9092'],
    auto_offset_reset='earliest',
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

for message in consumer:
    print(f"Received message: {message.value}")
```
**Result**
Producer already send the message to the customer, and the customer successfuly receieve the data.

![Screenshot 2024-09-10 at 2.26.19 PM](https://hackmd.io/_uploads/rklnj8wpn0.png)

But when we check the topology-inventory, it dosen't update the equipment data

**For NRCellDU entities.**

```javascript=
curl -k --http2 --header "Accept: application/json" -X GET http://10.233.102.133:8080/topology-inventory/v1alpha11/domains/CLOUD_TO_RAN/entity-types/NRCellDU/entities | jq
```

![Screenshot 2024-09-10 at 2.28.35 PM](https://hackmd.io/_uploads/SJ4mDwT20.png)

**Use pgadmin tool to check the data**

Use this sql code to query the data.

```javascript=
SELECT * FROM ties_data."NRCellDU";
```

![Screenshot 2024-09-10 at 2.30.47 PM](https://hackmd.io/_uploads/B1GkuvanR.png)

**Check the kafka-console-customer.sh can receieve the message or not.**

Go inside the pod.
```javascript=
kubectl exec -it oran-smo-kafka-controller-0 -n smo -- /bin/bash
```
Kafka Producer
```javascript=
kafka-console-producer.sh --topic topology-inventory-ingestion --broker-list localhost:9092
```
Kafka Consumer
```javascript=
kafka-console-consumer.sh --topic topology-inventory-ingestion --bootstrap-server localhost:9092 --from-beginning
```
![Screenshot 2024-09-10 at 4.45.18 AM](https://hackmd.io/_uploads/H1d7TdlaA.png)

So now we know the problem is the customer actually can't receieve the message.

**Check kafka error log**

![Screenshot 2024-09-11 at 5.10.11 AM](https://hackmd.io/_uploads/r1KgI4Rn0.png)

We found that kafka have some config problem. So this is why problem occur.

**Soluation :**

Referendce : [[bitnami/kafka] messages are not getting generated in topics from producer to consumer #19522](https://github.com/bitnami/charts/issues/19522)

![Screenshot 2024-09-11 at 5.42.06 AM](https://hackmd.io/_uploads/rJwyGYgT0.png)

Try to add the new config to kafka.

```javascript=
kubectl get configmap oran-smo-kafka-controller-configuration -n smo -o yaml > kafka-config8.yaml
```
Add the new config inside.
```javascript=
vim kafka-config8.yaml
```

![Screenshot 2024-09-12 at 11.05.44 PM](https://hackmd.io/_uploads/HyKI7YlpC.png)

Apply the revise config.
```javascript=
kubectl apply -f kafka-config8.yaml -n smo
```
Find the kafka statefulset and then restart. 
```javascript=
kubectl get statefulset -n smo | grep kafka
```
Restart
```javascript=
kubectl rollout restart statefulset oran-smo-kafka-controller -n smo
```

**Result**
**Kafka-controller-0 already running**
![Screenshot 2024-09-12 at 11.09.54 PM](https://hackmd.io/_uploads/ByB84YgaR.png)

**topology-inventory-ingestion-consumer already subscribe to kafka**

```javascript=
kafka-consumer-groups.sh --list --bootstrap-server localhost:9092
```
![Screenshot 2024-09-12 at 11.14.10 PM](https://hackmd.io/_uploads/rk8LrYxpC.png)

**After adding the data check again in the topology-exposure**

**For NRCellDU entities.**

![Screenshot 2024-09-12 at 11.28.04 PM](https://hackmd.io/_uploads/r1L5uFeaR.png)

We can successfuly to adding the data in the topology through kafka producer, but some data are not show on the display.

![Screenshot 2024-09-12 at 11.31.14 PM](https://hackmd.io/_uploads/SkM8tYxpR.png)

**Conclusion :**
Because in their docker compose installation method have this setting "KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1"

![Screenshot 2024-09-12 at 11.17.18 PM](https://hackmd.io/_uploads/BybmvKlaR.png)

But our installation method is run in k8s, so their kafka environment script dosen't have this setting. 
:::


:::spoiler **(Resolved)** Error Broker may not be available
:::danger
**ERROR : Broker may not be available**
![Screenshot 2024-09-07 at 6.42.52 AM](https://hackmd.io/_uploads/ryXq_DTnA.png)

**Solution: Delete the pod and run again**
Because kafka-controller will restart by it self, so that the topology-ingestion will disconnect to the kafka broker.We need to restart the topology-ingestion and will automaticaly to connect to the kafka.
```javascript=
kubectl delete pod topology-ingestion-5c9d478954-rhz6d -n smo
```
![Screenshot 2024-09-10 at 2.40.09 PM](https://hackmd.io/_uploads/BynAYDThR.png)

Consumer in topology-ingestion is already subscribe to the topic.

![Screenshot 2024-09-11 at 3.39.40 AM](https://hackmd.io/_uploads/BJ5kWX0n0.png)
:::
### 4-2. Testing modify enriched data from Topology & Inventory

This example updates an existing NRCellDU entity.

We need to change ==ce_type:::ran-logical-topology.create== to ==ce_type:::ran-logical-topology.merge==
```javascript=
ce_specversion:::1.0,ce_id:::26a2bb70-cd05-4e73-bbc9-860a17a1a22a,ce_source:::dmi-plugin:nm-1,ce_type:::ran-logical-topology.merge,content-type:::application/yang-data+json,ce_time:::2024-09-16T09:05:00Z,ce_dataschema:::https://ties:8080/schemas/v1/r1-topology,,,{
  "entities": [
    {
      "o-ran-smo-teiv-ran:NRCellDU": [
        {
          "id": "urn:3gpp:dn:ManagedElement=NR01,GNBDUFunction=1,NRCellDU=1",
          "attributes": {
            "cellLocalId": 1,
            "nRTAC": 0,
            "nRPCI": 0,
            "nCI": 0
          },
          "sourceIds": [
            "urn:3gpp:dn:ManagedElement=NR01,GNBDUFunction=1,NRCellDU=1"
          ]
        },
        {
          "id": "urn:3gpp:dn:ManagedElement=NR02,GNBDUFunction=1,NRCellDU=2",
          "attributes": {
            "cellLocalId": 2,
            "nRTAC": 0,
            "nRPCI": 0,
            "nCI": 0
          },
          "sourceIds": [
            "urn:3gpp:dn:ManagedElement=NR01,GNBDUFunction=1,NRCellDU=1"
          ]
        }
      ]
    }
  ]
}
```
After revise the data and produce an event, we check the attributes for each NRCellDU.

```javascript=
curl -k --http2 --header "Accept: application/yang.data+json, application/problem+json" -X GET http://10.233.102.166:8080/topology-inventory/v1alpha11/domains/CLOUD_TO_RAN/entity-types/NRCellDU/entities/urn:3gpp:dn:ManagedElement=NR01,GNBDUFunction=1,NRCellDU=1 | jq
```
```javascript=
curl -k --http2 --header "Accept: application/yang.data+json, application/problem+json" -X GET http://10.233.102.166:8080/topology-inventory/v1alpha11/domains/CLOUD_TO_RAN/entity-types/NRCellDU/entities/urn:3gpp:dn:ManagedElement=NR02,GNBDUFunction=1,NRCellDU=2 | jq
```

![Screenshot 2024-09-16 at 3.27.55 AM](https://hackmd.io/_uploads/HygUrnNpC.png)

Check pgadmin tools.

![Screenshot 2024-09-16 at 3.29.59 AM](https://hackmd.io/_uploads/HJq6HhVT0.png)



### 4-3. Testing delete enriched data from Topology & Inventory

This example delete an existing NRCellDU entity.

We need to change ==ce_type:::ran-logical-topology.merge== to ==ce_type:::ran-logical-topology.delete==

```javascript=
ce_specversion:::1.0,ce_id:::26a2bb70-cd05-4e73-bbc9-860a17a1a22a,ce_source:::dmi-plugin:nm-1,ce_type:::ran-logical-topology.delete,content-type:::application/yang-data+json,ce_time:::2024-09-16T09:05:00Z,ce_dataschema:::https://ties:8080/schemas/v1/r1-topology,,,{
"entities": [
    {
      "o-ran-smo-teiv-ran:NRCellDU": [
        {
          "id": "urn:3gpp:dn:ManagedElement=NR01,GNBDUFunction=1,NRCellDU=1"
        },
        {
          "id": "urn:3gpp:dn:ManagedElement=NR02,GNBDUFunction=1,NRCellDU=2"
        }
      ]
    }
  ]
}
```
After revise the data and produce an event, we check the entities for NRCellDU.

![Screenshot 2024-09-16 at 3.43.29 AM](https://hackmd.io/_uploads/Hk64KhNTR.png)

Check the pgadmin tools.

![Screenshot 2024-09-16 at 3.45.34 AM](https://hackmd.io/_uploads/SJyOtnNa0.png)


## 5. Using pgAdmin tool to connect to the PostgreSQL
pgAdmin it is one of the most popular PostgreSQL management tools and offers a rich graphical interface.

Installation link : https://www.postgresql.org/ftp/pgadmin/pgadmin4/v8.11/macos/

![Screenshot 2024-09-06 at 3.51.58 AM](https://hackmd.io/_uploads/B15-3tw20.png)

### 5-1. SSH Tunnel Setup in local
```javascript=
ssh -L 5432:localhost:5432 smo-j@192.168.8.27
```
### 5-2. kubectl port-forward in remote server
```javascript=
kubectl port-forward -n smo oran-smo-postgresql-0 5432:5432
```
### 5-3. pgAdmin tool procedure

#### 5-3-1. Register Server

![Screenshot 2024-09-06 at 4.01.26 AM](https://hackmd.io/_uploads/HkfLCKPnR.png)

#### 5-3-2. Connect to the server

![Screenshot 2024-09-06 at 4.06.29 AM](https://hackmd.io/_uploads/SJLUkqD20.png)

- Host name/address: 127.0.0.1
- Port: 5432
- Username: topology_exposure_user
- Password: teiv

You can use this command to find your Password or Username.

**Password :**
```javascript=
kubectl get secret -n smo oran-smo-postgresql -o jsonpath='{.data.postgres-password}' | base64 --decode
```
**Username :**
```javascript=
kubectl exec -n smo oran-smo-postgresql-0 -- env | grep USER
```

**Result :**
![Screenshot 2024-09-06 at 4.12.44 AM](https://hackmd.io/_uploads/BynplcwnR.png)

#### 5-3-3. Testing inquire data in topology_exposure_db

Select the Query tool.

![Screenshot 2024-09-06 at 4.53.59 PM](https://hackmd.io/_uploads/HyWHQB_3A.png)

Use this sql code to query the data.

```javascript=
SELECT * FROM ties_data."AntennaModule";
```
![Screenshot 2024-09-06 at 4.00.20 AM](https://hackmd.io/_uploads/rJj_Z9D2A.png)

**Result :**
![Screenshot 2024-09-06 at 4.16.09 AM](https://hackmd.io/_uploads/BJ9qZ5vnC.png)

#### 5-3-4. Adding data in to the database and check in TEIV exposure.
Use NRCellDU as example.

```javascript=
SELECT * FROM ties_data."NRCellDU";
```

![Screenshot 2024-09-10 at 2.44.36 PM](https://hackmd.io/_uploads/ry_kjDT3A.png)

Check in TEIV exposure.

![Screenshot 2024-09-10 at 2.46.30 PM](https://hackmd.io/_uploads/HkY8svT3A.png)
## 6. TEIV add customized attributes
TEIV uses YANG files to define data formats and generate data structures. We can query the schema via the API to understand the contents of the YANG files.

We use **NRSectorCarrier** for example:

Check **NRSectorCarrier** schema
```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.102.188:8080/topology-inventory/v1alpha11/schemas/o-ran-smo-teiv-ran/content
```
![Screenshot 2024-09-26 at 10.02.11 AM](https://hackmd.io/_uploads/HkVnxBz0R.png)

Check **NRSectorCarrier** data attributes
```javascript=
curl -k --http2 --header "Accept: application/yang.data+json, application/problem+json" -X GET http://10.233.102.191:8080/topology-inventory/v1alpha11/domains/RAN/entity-types/NRSectorCarrier/entities/Test | jq
```
![Screenshot 2024-09-26 at 10.28.49 AM](https://hackmd.io/_uploads/SJXeDBGC0.png)

**Create customized attributes in NRSectorCarrier by using pgadmin tool.**

Here we create a new column in NRSectorCarrier
- New Column: configuredMaxTxPower
- Data Type: bigint 

![Screenshot 2024-09-25 at 2.43.24 PM](https://hackmd.io/_uploads/ByyxTGdCR.png)

**Result:**

![Screenshot 2024-09-30 at 8.46.45 PM](https://hackmd.io/_uploads/HkSCpGuCC.png)

**hash_info TABLE in ties_model**
This is important part you need to add the new word in hash table otherwise you will face "INTERNAL_SERVER_ERROR" problem.

Add the configuredNaxTxPower in TABLE.
- name: configuredMaxTxPower
- hashedValue: configuredMaxTxPower
- type: COLUMN

![Screenshot 2024-10-03 at 4.03.47 PM](https://hackmd.io/_uploads/rJnXlCiCR.png)

**Notice!!!**
After adding the new column in PostgreSQL database, you need to restart the topology-exposure again.

```javascript=
kubectl delete pod topology-exposure-54887cd47c-9sl7r -n smo
```

Check the NRSectorCarrier.

```javascript=
curl -k --http2 --header "Accept: application/yang.data+json, application/problem+json" -X GET http://10.233.102.146:8080/topology-inventory/v1alpha11/domains/RAN/entity-types/NRSectorCarrier/entities/Test | jq
```

![Screenshot 2024-09-30 at 8.52.20 PM](https://hackmd.io/_uploads/H11D1muAC.png)


### 6-1. Create a new schema
#### 6-1-1. Method 1: Using schema Create API to generate the new schema
This part we use a wifi for example, we generate the wifi schema so that we can use customized attributes to stored the data.

**o-ran-smo-teiv-wifi.yang**

```javascript=
module o-ran-smo-teiv-wifi {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-wifi";
    prefix or-teiv-wifi;
  
    import o-ran-smo-teiv-common-yang-types {prefix or-teiv-types; }
  
    import o-ran-smo-teiv-common-yang-extensions {prefix or-teiv-yext; }

    organization "Ericsson AB";
    contact "Ericsson first line support via email";
    description
    "RAN WIFI topology model.

    Copyright (c) 2023 Ericsson AB. All rights reserved.

    This model contains the topology entities and relations in the
    RAN CLOUD domain, which comprises cloud infrastructure and
    deployment aspects that can be used in the topology model.";
  
    revision "2024-05-02" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    or-teiv-yext:domain WIFI;
    
    list WIFIconfig {
        description "The NR Sector Carrier object provides
                    the attributes for defining the logical
                    characteristics of a carrier (cell) in a
                    sector. A sector is a coverage area associated
                    with a base station having its own antennas,
                    radio ports, and control channels. The concept
                    of sectors was developed to improve co-channel
                    interference in cellular systems, and most wireless
                    systems use three sector cells.";
        
        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf arfcnDL {
                description "NR Absolute Radio Frequency Channel
                            Number (NR-ARFCN) for downlink";
                type uint32;
            }

            leaf arfcnUL {
                description "NR Absolute Radio frequency Channel Number
                            (NR-ARFCN) for uplink.";
                type uint32;
            }

            leaf frequencyDL {
                description "RF Reference Frequency of downlink channel";
                type uint32;
            }

            leaf frequencyUL {
                description "RF Reference Frequency of uplink channel";
                type uint32;
            }

            leaf bSChannelBwDL {
                description "BS Channel bandwidth in MHz for downlink.";
                type uint32;
            }

            leaf configuredMaxTxPower {
                description "Maximum Power Transmission.";
                type uint32;
            }
        }
    }
}
```

Use this command to send the file.

```javascript=
curl -k -v --http2 -X POST -H "Accept: application/problem+json" -H "Content-Type: multipart/form-data" -F "file=@o-ran-smo-teiv-wifi.yang" "http://10.233.102.136:8080/topology-inventory/v1alpha11/schemas"
```

**Result**

![Screenshot 2024-09-26 at 4.42.43 PM](https://hackmd.io/_uploads/HyEcCqMRA.png)

:::warning
But when we check the TEIV exposure, WIFI schema is not here.

![Screenshot 2024-09-26 at 4.48.14 PM](https://hackmd.io/_uploads/Sylklof0A.png)

:::

#### 6-1-2. Method 2: Using pgsql-schema-generator to generate the new schema

We create the new files.

Sorce file reference: https://gerrit.o-ran-sc.org/r/changes/smo%2Fteiv~13294/revisions/1/files/pgsql-schema-generator%2FcopyYangFiles.sh/download
```javascript=
cd teiv/pgsql-schema-generator/
vim copyYangFiles.sh
```
```javascript=
#! /bin/bash
#
# ============LICENSE_START=======================================================
# Copyright (C) 2024 Ericsson
# Modifications Copyright (C) 2024 OpenInfra Foundation Europe
# ================================================================================
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# SPDX-License-Identifier: Apache-2.0
# ============LICENSE_END=========================================================
#

set -e

SOURCE_DIR="../teiv/src/main/resources/models"
TARGET_DIR="./src/main/resources/generate-defaults"

mkdir -p "$TARGET_DIR"

if [ "$(ls -A "$SOURCE_DIR")" ]; then
    cp -r "$SOURCE_DIR"/* "$TARGET_DIR"
else
    echo "Source directory is empty or does not exist."
fi
```
```javascript=
chmod 777 copyYangFiles.sh
./copyYangFiles.sh
```

![Screenshot 2024-09-27 at 11.19.16 AM](https://hackmd.io/_uploads/S1L1mMV00.png)


In this folder, we create a new yang file for wifi. [See here](#6-1-Create-a-new-schema)

![Screenshot 2024-09-27 at 6.54.45 PM](https://hackmd.io/_uploads/Hk2xlfN0A.png)

After adding the new schema, use this command to build.

```javascript=
cd /teiv/pgsql-schema-generator
apt install maven

mvn exec:java -Dexec.mainClass="org.oran.smo.teiv.pgsqlgenerator.DatabaseSchemaGeneratorApplication"
or 
mvn spring-boot:run
```

**Result:**

![Screenshot 2024-09-30 at 11.40.03 AM](https://hackmd.io/_uploads/HJIip9DCA.png)

If you want to rebuild the data architecture, you can use this command .
```javascript=
cd /teiv/pgsql-schema-generator
mvn clean install
```
Run again.
```javascript=
mvn exec:java -Dexec.mainClass="org.oran.smo.teiv.pgsqlgenerator.DatabaseSchemaGeneratorApplication"
```
:::spoiler Troubleshooting
:::danger
If is not successful

![Screenshot 2024-09-27 at 11.41.31 AM](https://hackmd.io/_uploads/SJX74zNAR.png)

try to use this to build first.

```javascript=
cd
mvn clean install -Dmaven.test.skip=true
```

![Screenshot 2024-09-27 at 4.16.54 PM](https://hackmd.io/_uploads/SJ-AEfNCR.png)

This error means that you don't have a java version 17 environment, use this command to install java.

```javascript=
sudo apt update
sudo apt install openjdk-17-jdk
sudo update-alternatives --config java
java -version
```

![Screenshot 2024-09-27 at 4.30.56 PM](https://hackmd.io/_uploads/rk9tHG40A.png)

Once the Yang Parser 1.0.1-SNAPSHOT is success, we run again.

**Result :**

```javascript=
cd /teiv/pgsql-schema-generator
mvn exec:java -Dexec.mainClass="org.oran.smo.teiv.pgsqlgenerator.DatabaseSchemaGeneratorApplication"
```
![Screenshot 2024-09-27 at 7.30.39 PM](https://hackmd.io/_uploads/H1KtDf40C.png)

![Screenshot 2024-09-27 at 6.11.42 PM](https://hackmd.io/_uploads/SJB2VzVC0.png)

When we chenk the file in the target folder, we found the 2 new init_sql file in here.

![Screenshot 2024-09-30 at 11.40.03 AM](https://hackmd.io/_uploads/HJIip9DCA.png)

:::

#### 6-1-3. Method 3: Using pgadmin tool to generate the new schema
Here, we can manually add the WIFI schema (in ties_model) and the WIFI data table (in ties_data) based on the two initialization SQL files generated by method2.

- 00_init-oran-smo-teiv-data.sql
- 01_init-oran-smo-teiv-model.sql

**For ties_model we insert the WIFi schema, please follow the setting in o-ran-smo-teiv-wifi.yang**

![Screenshot 2024-09-30 at 9.08.58 PM](https://hackmd.io/_uploads/Hk2xQXdCR.png)

**In module_reference TABLE**

```javascript=
SELECT name, namespace, domain, "includedModules", revision, content, "ownerAppId", status
	FROM ties_model.module_reference;
```

Add the new WIFI schema.
- name: o-ran-smo-teiv-wifi
- namespace: urn:o-ran:smo-teiv-wifi
- domain: WIFi
- includedModules: []
- revision: 2024-05-02
- content: <Duplicate the hash content in 01_init-oran-smo-teiv-model.sql>
- ownerAppId: BUILT_IN_MODULE
- status: IN_USAGE

![Screenshot 2024-09-30 at 9.19.25 PM](https://hackmd.io/_uploads/rytYrXOR0.png)

For content you can found in 01_init-oran-smo-teiv-model.sql
![Screenshot 2024-09-30 at 9.16.50 PM](https://hackmd.io/_uploads/S1oSwmuCA.png)

**In entity_info TABLE**

```javascript=
SELECT name, "moduleReferenceName"
	FROM ties_model.entity_info;
```

Add the new WIFI entity.
- name: WIFIconfig **(Notice! This name is your TABLE name which will create later)**
- moduleReferenceName: o-ran-smo-teiv-wifi

![Screenshot 2024-09-30 at 9.34.29 PM](https://hackmd.io/_uploads/ByPgFm_RA.png)

**For ties_data we add the WIFi TABLE, please follow the setting in o-ran-smo-teiv-wifi.yang**

Using cammand to add the new table. You can fund in the 00_init-oran-smo-teiv-data.sql
```javascript=
CREATE TABLE IF NOT EXISTS ties_data."WIFIconfig" (
        "id"                    VARCHAR(511),
        "arfcnDL"                       BIGINT,
        "arfcnUL"                       BIGINT,
        "frequencyDL"                   BIGINT,
        "frequencyUL"                   BIGINT,
        "bSChannelBwDL"                 BIGINT,
        "configuredMaxTxPower"                  BIGINT,
        "CD_sourceIds"                  jsonb,
        "CD_classifiers"                        jsonb,
        "CD_decorators"                 jsonb
);

ALTER TABLE ONLY ties_data."WIFIconfig" ALTER COLUMN "CD_sourceIds" SET DEFAULT '[]';

ALTER TABLE ONLY ties_data."WIFIconfig" ALTER COLUMN "CD_classifiers" SET DEFAULT '[]';

ALTER TABLE ONLY ties_data."WIFIconfig" ALTER COLUMN "CD_decorators" SET DEFAULT '{}';
```

Setting id to primary key.

```javascript=
SELECT ties_data.create_constraint_if_not_exists(
        'WIFIconfig',
 'PK_WIFIconfig_id',
 'ALTER TABLE ties_data."WIFIconfig" ADD CONSTRAINT "PK_WIFIconfig_id" PRIMARY KEY ("id");'
);
```

**Result:**

![Screenshot 2024-09-30 at 9.48.29 PM](https://hackmd.io/_uploads/Bker37OA0.png)

**hash_info TABLE in ties_model**
Because you create a new TABLE and also a new primary key, add the new word in hash table.This prat relative to the adding configuredMaxTxPower part.

![Screenshot 2024-10-03 at 4.11.58 PM](https://hackmd.io/_uploads/rk5oGAs0C.png)

**Notice!!!**
After complete, we need to restart the topology-exposure.

```javascript=
kubectl delete pod topology-exposure-54887cd47c-9sl7r -n smo
```

Check the new schema in topology-exposure.
```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.102.146:8080/topology-inventory/v1alpha11/schemas | jq
```

![Screenshot 2024-09-30 at 9.55.30 PM](https://hackmd.io/_uploads/H1dJCX_CA.png)

Check the new schema content.

```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.102.146:8080/topology-inventory/v1alpha11/schemas/o-ran-smo-teiv-wifi/content
```
![Screenshot 2024-09-30 at 9.57.56 PM](https://hackmd.io/_uploads/SJP_CXdRR.png)

Check the WIFI domains.

```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.102.146:8080/topology-inventory/v1alpha11/domains | jq
```

![Screenshot 2024-09-30 at 10.01.00 PM](https://hackmd.io/_uploads/H1amJNOAC.png)

Check WIFIconfig data.

```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.102.146:8080/topology-inventory/v1alpha11/domains/WIFI/entity-types | jq
```
![Screenshot 2024-09-30 at 10.03.50 PM](https://hackmd.io/_uploads/S1aC14uCC.png)

Check WIFI data in WIFIconfig.

```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.102.146:8080/topology-inventory/v1alpha11/domains/WIFI/entity-types/WIFIconfig/entities | jq
```

![Screenshot 2024-09-30 at 10.05.31 PM](https://hackmd.io/_uploads/Bkh6x4d0R.png)

```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.102.146:8080/topology-inventory/v1alpha11/domains/WIFI/entity-types/WIFIconfig/entities/Testing | jq
```

![Screenshot 2024-09-30 at 10.08.31 PM](https://hackmd.io/_uploads/r1Vb-4OAA.png)

### 6-2. Testing
Create the new data in WIFI domain.

**Setting configuredMaxTxPower to 50000.**

```javascript=
cd teiv/docker-compose/cloudEventProducer/events/
vim cloudEventExampleMerge.txt
```
```javascript=
ce_specversion:::1.0,ce_id:::26a2bb70-cd05-4e73-bbc9-860a17a1a22a,ce_source:::dmi-plugin:nm-1,ce_type:::ran-logical-topology.create,content-type:::application/yang-data+json,ce_time:::2024-09-16T09:05:00Z,ce_dataschema:::https://ties:8080/schemas/v1/r1-topology,,,{
"entities": [
    {
      "o-ran-smo-teiv-wifi:WIFIconfig": [
        {
          "id": "FinalTesting",
          "attributes": {
            "configuredMaxTxPower": 50000
          }
        }
      ]
    }
  ]
}
```
```javascript=
cd ..
./cloudEventProducerfork8s2.sh
```

**Result:**
![Screenshot 2024-09-30 at 10.37.24 PM](https://hackmd.io/_uploads/r1rnPNd00.png)

Delete the data by using **ran-logical-topology.delete**

**Result:**
![Screenshot 2024-09-30 at 10.41.40 PM](https://hackmd.io/_uploads/ByLhuN_0A.png)


## Appendix
### 1. Kafka useful command

```javascript=
kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name topology-inventory-ingestion --alter --add-config retention.ms=1000
```
```javascript=
kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name topology-inventory-ingestion --alter --delete-config retention.ms
```

```javascript=
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topology-inventory-ingestion --from-beginning
```

```javascript=
kubectl get configmap oran-smo-kafka-controller-configuration -n smo -o yaml > kafka-config6.yaml
```

```javascript=
kubectl get statefulset -n smo | grep kafka
```

```javascript=
kubectl rollout restart statefulset oran-smo-kafka-controller -n smo
```

```javascript=
sudo lsof -i :5432
```
