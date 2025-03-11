# <center>Configure the [J-Release] OSC DU by using OSC SMO TEIV</center>

<center>SMO O1 Ves + O1 Netconf + [J-Release] OSC DU-High + OSC SMO TEIV</center>

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
- [Configure the OAI telnet O1 gNB by using OSC SMO TEIV](https://hackmd.io/@Jerry0714/HJrE-wmXyg)
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

<p class="img"><img src="https://hackmd.io/_uploads/BylDBwmLyg.png"></p>

### 1-2. O-RAN network provisioning Operation
**Spec :** (O-RAN.WG10.TE&IV-R003-v02.00)

**Goal :** O-RAN network provisioning using TE&IV services

**Sequence Diagram**

![Screenshot 2024-09-20 at 6.12.18 AM](https://hackmd.io/_uploads/S1u0WXc60.png)
## 2. [J-Release] OSC DU-High Configuration parameter
The following parameter is we can use netconf to import to trigger the OSC DU.
- Netconf Configuration parameter
    - ManagedElement
        - priorityLabel
    - GNBDUFunction
        - gNBId
        - gNBIdLength
        - gNBDUId
        - priorityLabel
        - resourceType
        - rRMPolicyMemberList
            - mcc
            - mnc
            - sd
            - sst
    - NRCellDU
        - priorityLabel
        - resourceType
        - rRMPolicyMemberList
            - mcc
            - mnc
            - sd
            - sst
        - cellLocalId
        - pLMNInfoList
            - mcc
            - mnc
            - sd
            - sst
        - nRPCI
        - ssbFrequency
        - ssbPeriodicity
        - ssbSubCarrierSpacing
        - ssbOffset
        - ssbDuration
        - nRSectorCarrierRef
        - administrativeState
        - nRTAC
        - arfcnDL
        - arfcnUL
        - arfcnSUL
        - bSChannelBwDL
        - bSChannelBwUL
        - bSChannelBwSUL
- Netconf command
```javascript=
from ncclient import manager 
smo = manager.connect(host='192.168.8.39', port="830", timeout=5, username="netconf",password="netconf!", hostkey_verify=False)
```
```javascript=
conf = '''
<config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:xc="urn:ietf:params:xml:ns:netconf:base:1.0">
  <ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
    <id>me-3</id>
    <attributes>
      <priorityLabel>1</priorityLabel>
    </attributes>
    <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
      <id>gnb-1</id>
      <attributes>
        <gNBId>1</gNBId>
        <gNBIdLength>23</gNBIdLength>
        <gNBDUId>3</gNBDUId>
        <priorityLabel>1</priorityLabel>
        <resourceType>PRB_DL</resourceType>
        <rRMPolicyMemberList>
          <mcc>311</mcc>
          <mnc>48</mnc>
          <sd>000001</sd>
          <sst>1</sst>
        </rRMPolicyMemberList>
        <rRMPolicyMemberList>
          <mcc>311</mcc>
          <mnc>48</mnc>
          <sd>000002</sd>
          <sst>1</sst>
        </rRMPolicyMemberList>
      </attributes>
      <NRCellDU xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrcelldu">
        <id>nrcell-1</id>
        <attributes>
          <priorityLabel>1</priorityLabel>
          <resourceType>PRB_DL</resourceType>
          <rRMPolicyMemberList>
            <mcc>311</mcc>
            <mnc>48</mnc>
            <sd>000001</sd>
            <sst>1</sst>
          </rRMPolicyMemberList>
          <rRMPolicyMemberList>
            <mcc>311</mcc>
            <mnc>48</mnc>
            <sd>000002</sd>
            <sst>1</sst>
          </rRMPolicyMemberList>
          <cellLocalId>1</cellLocalId>
          <pLMNInfoList>
            <mcc>311</mcc>
            <mnc>48</mnc>
            <sd>000001</sd>
            <sst>1</sst>
          </pLMNInfoList>
          <pLMNInfoList>
            <mcc>311</mcc>
            <mnc>48</mnc>
            <sd>000002</sd>
            <sst>1</sst>
          </pLMNInfoList>
          <nRPCI>1</nRPCI>
          <ssbFrequency>3000000</ssbFrequency>
          <ssbPeriodicity>20</ssbPeriodicity>
          <ssbSubCarrierSpacing>15</ssbSubCarrierSpacing>
          <ssbOffset>0</ssbOffset>
          <ssbDuration>1</ssbDuration>
          <nRSectorCarrierRef>CN=John Smith,OU=Sales,O=ACME Limited,L=Moab,ST=Utah,C=US</nRSectorCarrierRef>
          <administrativeState>UNLOCKED</administrativeState>
          <nRTAC>1</nRTAC>
          <arfcnDL>428000</arfcnDL>
          <arfcnUL>390000</arfcnUL>
          <arfcnSUL>100</arfcnSUL>
          <bSChannelBwDL>20</bSChannelBwDL>
          <bSChannelBwUL>20</bSChannelBwUL>
          <bSChannelBwSUL>14</bSChannelBwSUL>
        </attributes>
      </NRCellDU>
    </GNBDUFunction>
  </ManagedElement>
</config>
'''
```
```javascript=
smo.edit_config (target = "running", config=conf)
```
**Result**

![Screenshot 2024-12-27 at 4.23.28 PM](https://hackmd.io/_uploads/HJqMNy3Skx.png)

### 2-1. Create the data field in TE&IV service to store OSC DU-High parameter

Some parameter field is not support in TE&IV, we need to add the field by ourself.
![Screenshot 2024-12-26 at 3.27.47 PM](https://hackmd.io/_uploads/rkP0SFcSJx.png)

### 2-2. Add the OSC DU-High configuration parameter in TE&IV 
We check the all data field in each domain.
- ManagedElement
```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.2.26:8080/topology-inventory/v1alpha11/domains/OAM_TO_RAN/entity-types/ManagedElement/entities/ODU-High | jq
```

![Screenshot 2024-12-26 at 3.42.38 PM](https://hackmd.io/_uploads/S1x-YtqB1g.png)

- GNBDUFunction
```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.2.26:8080/topology-inventory/v1alpha11/domains/OAM_TO_RAN/entity-types/GNBDUFunction/entities/ODU-High | jq
```

![Screenshot 2024-12-26 at 3.53.26 PM](https://hackmd.io/_uploads/BkDFiK9HJx.png)

- NRCellDU
```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.2.26:8080/topology-inventory/v1alpha11/domains/OAM_TO_RAN/entity-types/NRCellDU/entities/ODU-High | jq
```
![Screenshot 2024-12-26 at 4.05.34 PM](https://hackmd.io/_uploads/HkO_AK5SJg.png)

### 2-3. A comparison table of TE&IV parameters with OAI telnet O1 gNB and OSC DU-High.

All parameters listed in the table are defined in TE&IV. The ==orange parameters are custom-added parameters==, and the checkmarks indicate the parameters used by different gNBs, all of which can be configured via O1 Netconf.

![Screenshot 2025-01-16 at 4.28.03 PM](https://hackmd.io/_uploads/Sku47S8D1e.png)

## 3. Implement automatically configure for OSC DU-High
Because we need to automatically generate the xml for netconf, the following testing is checking whether the netconf xml can delete or insert or not.

Using netconf config OSC DU is different than config OAI gNB, we need to set different ManagedElement ID every time otherwise the config will failed. For config OAI gNB, we only use same ManagedElement ID is Ok same with VIAVI.

```javascript=
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
    <id><need to unique></id>
```       

In NETCONF, configuration attributes for OSC DU cannot have their values reduced, but it is allowed to add pLMNInfoList and rRMPolicyMemberList.

- GNBDUFunction
```javascript=
<rRMPolicyMemberList>
    <mcc>311</mcc>
    <mnc>48</mnc>
    <sd>000001</sd>
    <sst>1</sst>
</rRMPolicyMemberList>
<rRMPolicyMemberList>
    <mcc>311</mcc>
    <mnc>48</mnc>
    <sd>000002</sd>
    <sst>1</sst>
</rRMPolicyMemberList>

...(can be more)
```
- NRCellDU
```javascript=
<rRMPolicyMemberList>
    <mcc>311</mcc>
    <mnc>48</mnc>
    <sd>000001</sd>
    <sst>1</sst>
</rRMPolicyMemberList>
<rRMPolicyMemberList>
    <mcc>311</mcc>
    <mnc>48</mnc>
    <sd>000002</sd>
    <sst>1</sst>
</rRMPolicyMemberList>
<pLMNInfoList>
    <mcc>311</mcc>
    <mnc>48</mnc>
    <sd>000001</sd>
    <sst>1</sst>
</pLMNInfoList>
<pLMNInfoList>
    <mcc>311</mcc>
    <mnc>48</mnc>
    <sd>000002</sd>
    <sst>1</sst>
</pLMNInfoList>

...(can be more)
```

But need to make sure at least one rRMPolicyMemberList and pLMNInfoList, otherwise you will face this problem.

![Screenshot 2024-12-27 at 5.20.10 PM](https://hackmd.io/_uploads/ByGPMxhSJl.png)

![Screenshot 2024-12-27 at 5.44.27 PM](https://hackmd.io/_uploads/Sk7_Denr1l.png)

The IDs of GNBDUFunction and NRCellDU can be the same or different, meaning that OSC DU's NETCONF does not distinguish based on the IDs. However, OAI gNB and Viavi do not allow modifications if the IDs are different. This is the key difference between the three.

```javascript=
<GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
    <id><Invalid value></id>
    <NRCellDU xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrcelldu">
        <id><Invalid value></id>
```
### 3-1. TE&IV consumer rApp flow chart and procedure
<style>
  .img2 {
    position:relative;
    width: 70%;
    left: 15%;
  }
</style>

**The main function in TE&IV consumer rApp.**
<p class="img2"><img src="https://hackmd.io/_uploads/SJY71Z3SJx.png"></p>

**The registration monitor function and main loop function in TE&IV consumer rApp.**
![Screenshot 2024-12-27 at 6.13.32 PM](https://hackmd.io/_uploads/S17e0ehr1l.png)

**TE&IV consumer rApp procedure**

Program Initialization
- Create Registration Manager
- Start Registration Monitor Task
- Main Program Loop
    - Check Pod Status
    - Get Service Address
    - Get and Compare States
    - Process State Changes
- State Processing Flow
    - Process Planned Status
        - Check Configuration Files
        - Generate Kafka Messages
    - Process Assigned Status
        - Check Registration Status
        - Update to Deployed Status
    - Process Deployed Status
        - Compare Attributes
        - Update Netconf Configuration
        - Update to Active Status
- Registration Monitor
    - Monitor DMAAP Messages
    - Handle Different Event Types:
        - PNF Registration
        - Alarms
        - Heartbeat
        - File Ready
        - Write to InfluxDB

### 3-2. Add the relationships in TE&IV

![555](https://hackmd.io/_uploads/S1yVseYLyl.png)

More detail procedure overview.

![888](https://hackmd.io/_uploads/SJm3kdRLJe.png)

- In TE&IV service, the parameter store in the different table(Managelement, gNBDUFunction and NRCellDU …), each table have different data field(column). So this is why we need to use relationship to connect them together.
- Because OSC DU-High have Managelement, gNBDUFunction and NRCellDU configuration, so we need to let rApp to recognized and combine configuration together.


```javascript=
ce_specversion:::1.0,ce_id:::26a2bb70-cd05-4e73-bbc9-860a17a1a22a,ce_source:::dmi-plugin:nm-1,ce_type:::ran-logical-topology.merge,content-type:::application/yang-data+json,ce_time:::2024-09-16T09:05:00Z,ce_dataschema:::https://ties:8080/schemas/v1/r1-topology,,,{
"relationships": [
    {
      "o-ran-smo-teiv-rel-oam-ran:MANAGEDELEMENT_MANAGES_GNBDUFUNCTION": [
        {
          "id": "urn:oran:smo:teiv:MANAGEDELEMENT_MANAGES_GNBDUFUNCTION=gnb-1",
          "aSide": "urn:3gpp:sa5:ManagedElement=ODU-High",
          "bSide": "urn:3gpp:sa5:ManagedElement=ODU-High,GNBDUFunction=gnb-1"
        }
      ],
      "o-ran-smo-teiv-ran:GNBDUFUNCTION_PROVIDES_NRCELLDU": [
        {
          "id": "urn:oran:smo:teiv:GNBDUFUNCTION_PROVIDES_NRCELLDU=nrcell-1",
          "aSide": "urn:3gpp:sa5:ManagedElement=ODU-High,GNBDUFunction=gnb-1",
          "bSide": "urn:3gpp:sa5:ManagedElement=ODU-High,GNBDUFunction=gnb-1,NRCellDu=nrcell-1"
        }
      ]
    }
  ]
}
```
- For MANAGEDELEMENT_MANAGES_GNBDUFUNCTION
We check the relationship result in topology-exposure
```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.2.26:8080/topology-inventory/v1alpha11/domains/OAM_TO_RAN/relationship-types/MANAGEDELEMENT_MANAGES_GNBDUFUNCTION/relationships | jq
```
![Screenshot 2025-01-03 at 5.23.33 PM](https://hackmd.io/_uploads/SkyQG4HU1x.png)

- For GNBDUFUNCTION_PROVIDES_NRCELLDU
```javascript=
curl -k --http2 --header "Accept: application/problem+json" -X GET http://10.233.2.26:8080/topology-inventory/v1alpha11/domains/OAM_TO_RAN/relationship-types/GNBDUFUNCTION_PROVIDES_NRCELLDU/relationships | jq
```
![Screenshot 2025-01-03 at 5.24.17 PM](https://hackmd.io/_uploads/HymXfNHUyl.png)

### 3-3. The checking relationship function in rApp.
Using this function to call the TE&IV service API to get the relationship data.
```javascript=
def execute_curl_command_for_relationships(service_address, domain, relationship_type):
    try:
        cmd = [
            'curl', '-k', '--http2',
            '--header', 'Accept: application/problem+json',
            '-X', 'GET',
            f'http://{service_address}:8080/topology-inventory/v1alpha11/domains/{domain}/relationship-types/{relationship_type}/relationships'
        ]
        result = subprocess.run(cmd, capture_output=True, text=True, check=True)
        return result.stdout
    except subprocess.CalledProcessError as e:
        logger.error(f"Failed to execute curl command for relationship {relationship_type}: {e}")
        return None
```
This check_relationships function is to checking the relationship in each of the deployed_ids.
```javascript=
def check_relationships(service_address, full_entity_id, deployed_ids):
    try:
        domain, entity_type, item_id = full_entity_id.split(':', 2)

        if entity_type == "ManagedElement":
            me_gnb_rel = execute_curl_command_for_relationships(
                service_address,
                'OAM_TO_RAN',
                'MANAGEDELEMENT_MANAGES_GNBDUFUNCTION'
            )

            if me_gnb_rel:
                me_gnb_data = json.loads(me_gnb_rel)
                for item in me_gnb_data.get('items', []):
                    relationships = item.get('o-ran-smo-teiv-oam-to-ran:MANAGEDELEMENT_MANAGES_GNBDUFUNCTION', [])
                    for rel in relationships:
                        if item_id == rel.get('aSide', ''):
                            b_side = rel.get('bSide', '')
                            logger.info(f"ManagedElement {item_id} manages GNBDUFunction: {b_side}")
                            gnb_full_id = f"RAN:GNBDUFunction:{b_side}"
                            if gnb_full_id in deployed_ids:
                                logger.info(f"Found matching deployed GNBDUFunction: {gnb_full_id}")
                                return True

        elif entity_type == "GNBDUFunction":
            gnb_cell_rel = execute_curl_command_for_relationships(
                service_address,
                'OAM_TO_RAN',
                'GNBDUFUNCTION_PROVIDES_NRCELLDU'
            )

            if gnb_cell_rel:
                gnb_cell_data = json.loads(gnb_cell_rel)
                for item in gnb_cell_data.get('items', []):
                    relationships = item.get('o-ran-smo-teiv-ran:GNBDUFUNCTION_PROVIDES_NRCELLDU', [])
                    for rel in relationships:
                        if item_id == rel.get('aSide', ''):
                            b_side = rel.get('bSide', '')
                            logger.info(f"GNBDUFunction {item_id} provides NRCellDU: {b_side}")
                            cell_full_id = f"RAN:NRCellDU:{b_side}"
                            if cell_full_id in deployed_ids:
                                logger.info(f"Found matching deployed NRCellDU: {cell_full_id}")
                                return True

        return False

    except Exception as e:
        logger.error(f"Error checking relationships: {str(e)}")
        return False
```
### 3-4. The checking relationship function with relationship group in rApp
```javascript=
def check_relationships(service_address, full_entity_id, deployed_ids):
    """
    Check relationships based on entity type and verify if related entities are in deployed_ids,
    while building relationship groups.
    """
    try:
        # Initialize relationship groups
        relationship_groups = {}
        current_group = 1
        
        # Parse entity type and ID from full_entity_id
        domain, entity_type, item_id = full_entity_id.split(':', 2)
        
        if entity_type == "ManagedElement":
            # Check the GNBDUFunctions managed by this ManagedElement
            me_gnb_rel = execute_curl_command_for_relationships(
                service_address,
                'OAM_TO_RAN',
                'MANAGEDELEMENT_MANAGES_GNBDUFUNCTION'
            )
            
            if me_gnb_rel:
                me_gnb_data = json.loads(me_gnb_rel)
                for item in me_gnb_data.get('items', []):
                    relationships = item.get('o-ran-smo-teiv-oam-to-ran:MANAGEDELEMENT_MANAGES_GNBDUFUNCTION', [])
                    for rel in relationships:
                        if item_id == rel.get('aSide', ''):
                            b_side = rel.get('bSide', '')
                            logger.info(f"ManagedElement {item_id} manages GNBDUFunction: {b_side}")
                            
                            # Check if this GNBDUFunction is in deployed_ids
                            me_full_id = full_entity_id
                            gnb_full_id = f"RAN:GNBDUFunction:{b_side}"
                            
                            if gnb_full_id in deployed_ids:
                                logger.info(f"Found matching deployed GNBDUFunction: {gnb_full_id}")
                                
                                # Add related entities to the same group
                                if me_full_id not in relationship_groups and gnb_full_id not in relationship_groups:
                                    # Create new group if neither entity is grouped
                                    relationship_groups[me_full_id] = current_group
                                    relationship_groups[gnb_full_id] = current_group
                                    logger.info(f"Created new group {current_group} for {me_full_id} and {gnb_full_id}")
                                    current_group += 1
                                elif me_full_id in relationship_groups:
                                    # Add to ME's existing group
                                    relationship_groups[gnb_full_id] = relationship_groups[me_full_id]
                                    logger.info(f"Added {gnb_full_id} to existing group {relationship_groups[me_full_id]}")
                                elif gnb_full_id in relationship_groups:
                                    # Add to GNB's existing group
                                    relationship_groups[me_full_id] = relationship_groups[gnb_full_id]
                                    logger.info(f"Added {me_full_id} to existing group {relationship_groups[gnb_full_id]}")
                                
                                return True
                                
        elif entity_type == "GNBDUFunction":
            # Check the NRCellDUs provided by this GNBDUFunction
            gnb_cell_rel = execute_curl_command_for_relationships(
                service_address,
                'OAM_TO_RAN',
                'GNBDUFUNCTION_PROVIDES_NRCELLDU'
            )
            
            if gnb_cell_rel:
                gnb_cell_data = json.loads(gnb_cell_rel)
                for item in gnb_cell_data.get('items', []):
                    relationships = item.get('o-ran-smo-teiv-ran:GNBDUFUNCTION_PROVIDES_NRCELLDU', [])
                    for rel in relationships:
                        if item_id == rel.get('aSide', ''):
                            b_side = rel.get('bSide', '')
                            logger.info(f"GNBDUFunction {item_id} provides NRCellDU: {b_side}")
                            
                            # Check if this NRCellDU is in deployed_ids
                            gnb_full_id = full_entity_id
                            cell_full_id = f"RAN:NRCellDU:{b_side}"
                            
                            if cell_full_id in deployed_ids:
                                logger.info(f"Found matching deployed NRCellDU: {cell_full_id}")
                                
                                # Add related entities to the same group
                                if gnb_full_id not in relationship_groups and cell_full_id not in relationship_groups:
                                    # Create new group if neither entity is grouped
                                    relationship_groups[gnb_full_id] = current_group
                                    relationship_groups[cell_full_id] = current_group
                                    logger.info(f"Created new group {current_group} for {gnb_full_id} and {cell_full_id}")
                                    current_group += 1
                                elif gnb_full_id in relationship_groups:
                                    # Add to GNB's existing group
                                    relationship_groups[cell_full_id] = relationship_groups[gnb_full_id]
                                    logger.info(f"Added {cell_full_id} to existing group {relationship_groups[gnb_full_id]}")
                                elif cell_full_id in relationship_groups:
                                    # Add to Cell's existing group
                                    relationship_groups[gnb_full_id] = relationship_groups[cell_full_id]
                                    logger.info(f"Added {gnb_full_id} to existing group {relationship_groups[cell_full_id]}")
                                
                                return True
                                
        # Display group information
        if relationship_groups:
            logger.info("Current relationship groups:")
            groups_by_number = {}
            for entity_id, group_num in relationship_groups.items():
                if group_num not in groups_by_number:
                    groups_by_number[group_num] = []
                groups_by_number[group_num].append(entity_id)
            
            for group_num, members in groups_by_number.items():
                logger.info(f"Group {group_num}:")
                for member in members:
                    logger.info(f"  - {member}")
                    
        return False
        
    except Exception as e:
        logger.error(f"Error checking relationships: {str(e)}")
        return False
```

### 3-5. Revising the TE&IV consumer rApp code to support automatically config OSC DU-High.

Currently this is our process_deployed_objects function code. We need to modify this to achieve the following item.
```javascript=
def process_deployed_objects(service_address, deployed_ids, kafka_namespace, kafka_topic, kafka_pod_name):
    logger.info("Processing objects with Deployed status in the TE&IV Service...")
    netconf_config = get_netconf_config()
    if netconf_config is None:
        logger.error("Failed to retrieve Netconf config. Skipping all Deployed object processing.")
        return

    for full_entity_id in deployed_ids:
        logger.info(f"Processing Deployed ID: {full_entity_id}")
        try:
            domain, entity_type, item_id = full_entity_id.split(':', 2)
            copy_item_id = item_id
        except ValueError:
            logger.error(f"Unable to parse full_entity_id: {full_entity_id}")
            continue

        try:
            managed_element_id, gnbdu_function_id, nr_sector_carrier_id, nr_cell_du_id = parse_item_id(item_id)

            if nr_sector_carrier_id:
                logger.info(f"Parsed IDs: ManagedElement={managed_element_id}, GNBDUFunction={gnbdu_function_id}, NRSectorCarrier={nr_sector_carrier_id}")
            elif nr_cell_du_id:
                logger.info(f"Parsed IDs: ManagedElement={managed_element_id}, GNBDUFunction={gnbdu_function_id}, NRCellDU={nr_cell_du_id}")
            else:
                logger.info(f"Parsed IDs: ManagedElement={managed_element_id}, GNBDUFunction={gnbdu_function_id}")

            if entity_type == 'NRSectorCarrier':
                netconf_attributes = find_matching_structure(netconf_config, managed_element_id, gnbdu_function_id, nr_sector_carrier_id)
            elif entity_type == 'NRCellDU':
                netconf_attributes = find_matching_structure(netconf_config, managed_element_id, gnbdu_function_id, None, nr_cell_du_id)
            elif entity_type == 'GNBDUFunction':
                netconf_attributes = find_matching_structure(netconf_config, managed_element_id, gnbdu_function_id)
            elif entity_type == 'ManagedElement':
                netconf_attributes = find_matching_structure(netconf_config, managed_element_id)
                netconf_attributes = None # I add this part
            else:
                logger.warning(f"Unrecognized entity type: {entity_type} for {full_entity_id}")
                continue

            if not netconf_attributes:
                logger.warning(f"No matching attributes found in Netconf config for {full_entity_id}")
                continue

            teiv_attributes = get_teiv_attributes(service_address, domain, entity_type, item_id)
            if not teiv_attributes:
                logger.error(f"Failed to retrieve attributes from TE&IV service for {full_entity_id}")
                continue

            updates_needed = compare_attributes(full_entity_id, netconf_attributes, teiv_attributes)

            if updates_needed:
                success = update_netconf_config(managed_element_id, gnbdu_function_id, nr_sector_carrier_id, nr_cell_du_id, updates_needed, entity_type)
                if success:
                    logger.info(f"Successfully updated Netconf config for {full_entity_id}")
                    # After successful Netconf update, update status to Active
                    new_status = "Active"
                    logger.info(f"Status of {full_entity_id} updated to {new_status}")

                    # Get the correct domain prefix
                    domain_prefix = next((prefix for d, et, prefix in DOMAINS_AND_TYPES if d == domain and et == entity_type), f"{domain}:{entity_type}")

                    # Prepare Kafka message
                    message_content = {
                        "entities": [
                            {
                                domain_prefix: [
                                    {
                                        "id": copy_item_id,
                                        "attributes": {
                                            "status": new_status
                                        }
                                    }
                                ]
                            }
                        ]
                    }

                    headers = {
                        "ce_specversion": "1.0",
                        "ce_id": str(uuid.uuid4()),
                        "ce_source": "dmi-plugin:nm-1",
                        "ce_type": "ran-logical-topology.merge",
                        "content-type": "application/yang-data+json",
                        "ce_time": datetime.utcnow().strftime("%Y-%m-%dT%H:%M:%SZ"),
                        "ce_dataschema": "https://ties:8080/schemas/v1/r1-topology"
                    }

                    # Prepare the full Kafka message with correct format
                    kafka_message = ",".join([f"{k}:::{v}" for k, v in headers.items()])
                    kafka_message += f",,,{json.dumps(message_content)}"

                    # Log the Kafka message for debugging
                    logger.info(f"Kafka message content: {kafka_message}")

                    # Send message to Kafka
                    if produce_messages(kafka_namespace, kafka_topic, kafka_pod_name, 1, content=kafka_message):
                        logger.info(f"Successfully sent update for {full_entity_id} to Kafka to change status to Active")
                    else:
                        logger.error(f"Failed to send status update to Active for {full_entity_id} to Kafka")
                else:
                    logger.error(f"Failed to update Netconf config for {full_entity_id}")
            else:
                logger.info(f"No updates needed for {full_entity_id}")

        except Exception as e:
            logger.error(f"Unexpected error processing {full_entity_id}: {str(e)}")
            logger.exception("Exception details:")
```
- Modify the existing logic to adapt to the OSC DU NETCONF functionality.
- For the OSC DU configuration method, replace the initial approach of reading NETCONF to match the element ID with a direct input of NETCONF XML settings.
- Add functionality to rApp to read JSON-formatted data for pLMNInfoList and rRMPolicyMemberList from TE&IV.

#### 3-5-1. 
```javascript=
def process_deployed_objects(service_address, deployed_ids, kafka_namespace, kafka_topic, kafka_pod_name):
    logger.info("Processing objects with Deployed status in the TE&IV Service...")
    netconf_config = get_netconf_config()
    if netconf_config is None:
        logger.error("Failed to retrieve Netconf config. Skipping all Deployed object processing.")
        return

    # Initialize global variables for relationship grouping
    global relationship_groups
    global current_group
    relationship_groups = {}
    current_group = 1

    # Step 1: Group all deployed objects by their relationships
    logger.info("Starting relationship grouping for all deployed objects...")

    for full_entity_id in deployed_ids:
        logger.info(f"Checking relationships for ID: {full_entity_id}")
        try:
            check_relationships(service_address, full_entity_id, deployed_ids)
        except Exception as e:
            logger.error(f"Error checking relationships for {full_entity_id}: {str(e)}")

    # Display relationship groups summary
    logger.info("=== Relationship Groups Summary ===")
    if relationship_groups:
        # Organize entities by group number
        groups_by_number = {}
        for entity_id, group_num in relationship_groups.items():
            if group_num not in groups_by_number:
                groups_by_number[group_num] = []
            groups_by_number[group_num].append(entity_id)

        # Display each group
        for group_num in sorted(groups_by_number.keys()):
            logger.info(f"\nGroup {group_num}:")
            for member in sorted(groups_by_number[group_num]):
                logger.info(f"  - {member}")
    else:
        logger.info("No relationship groups formed")
    logger.info("=================================")
```
**Result**
![Screenshot 2025-01-15 at 2.40.56 AM](https://hackmd.io/_uploads/Hkw9iEVPJg.png)

#### 3-5-2. Automated generation of multi-configuration NETCONF XML.

![55588](https://hackmd.io/_uploads/B1h0N5ut1e.png)

**In main.py**
```javascript=
def process_deployed_objects(service_address, deployed_ids, kafka_namespace, kafka_topic, kafka_pod_name):
    
    # Display relationship groups summary
    logger.info("=== Relationship Groups Summary ===")
    if relationship_groups:

    # Step 2: Batch update Netconf configuration for grouped entities
    logger.info("Starting batch Netconf configuration update...")
    if relationship_groups:
        success = batch_update_netconf_config(relationship_groups, service_address)
        if success:
            logger.info("Successfully completed batch Netconf configuration update")
        else:
            logger.error("Failed to complete batch Netconf configuration update")
```
**In netconfclinet.py**

```javascript=
def convert_list_attribute(attr_value, attr_name):
    """
    Convert list attributes like rRMPolicyMemberList and pLMNInfoList to proper format
    """
    try:
        if isinstance(attr_value, str):
            # Parse string representation of list
            items = json.loads(attr_value.replace("'", '"'))
        else:
            items = attr_value

        result = ""
        for item in items:
            result += f"""
            <{attr_name}>
                <mcc>{item['mcc']}</mcc>
                <mnc>{item['mnc']}</mnc>
                <sd>{item['sd']}</sd>
                <sst>{item['sst']}</sst>
            </{attr_name}>"""
        return result
    except Exception as e:
        logger.error(f"Error converting list attribute: {str(e)}")
        return ""
```

```javascript=
def format_attributes(attributes, entity_type):
    """
    Format attributes while filtering out None values and status
    """
    config = "<attributes>\n"
    for key, value in attributes.items():
        if value is not None and key != 'status':
            if key in ['rRMPolicyMemberList', 'pLMNInfoList']:
                config += convert_list_attribute(value, key)
            else:
                config += f"<{key}>{value}</{key}>\n"
    config += "</attributes>"
    return config
```

```javascript=
def batch_update_netconf_config(relationship_groups, service_address):
    """
    Batch update Netconf configuration for related entities in groups
    """
    try:
        # Organize entities by group
        groups_by_number = {}
        for entity_id, group_num in relationship_groups.items():
            if group_num not in groups_by_number:
                groups_by_number[group_num] = []
            groups_by_number[group_num].append(entity_id)

        # Process each group
        for group_num, entities in groups_by_number.items():
            logger.info(f"Processing group {group_num} for Netconf update")
            
            # Collect attributes for all entities in the group
            entity_attributes = {}
            for entity_id in entities:
                domain, entity_type, item_id = entity_id.split(':', 2)
                id_content = execute_curl_command_for_id(service_address, domain, entity_type, item_id)
                if id_content:
                    content_json = json.loads(id_content)
                    # Process based on entity type
                    for key, value in content_json.items():
                        if isinstance(value, list) and len(value) > 0:
                            entity = value[0]
                            if isinstance(entity, dict):
                                attributes = entity.get('attributes', {})
                                if attributes:
                                    entity_attributes[entity_id] = {
                                        'type': entity_type,
                                        'attributes': attributes
                                    }
                                    logger.info(f"Found attributes for {entity_id}: {attributes}")

            if not entity_attributes:
                logger.warning(f"No attributes found for group {group_num}, skipping...")
                continue

            # Generate Netconf config for the group
            with manager.connect(
                host='192.168.8.39',
                port=830,
                username="netconf",
                password="netconf!",
                hostkey_verify=False
            ) as m:
                config = """
                <config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
                    <ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
                        <id>dummy-123</id>
                """

                # 排序實體，確保正確的處理順序
                sorted_entities = sorted(
                    entity_attributes.items(),
                    key=lambda x: ['ManagedElement', 'GNBDUFunction', 'NRSectorCarrier', 'NRCellDU'].index(x[1]['type'])
                    if x[1]['type'] in ['ManagedElement', 'GNBDUFunction', 'NRSectorCarrier', 'NRCellDU'] else 999
                )

                # Add ManagedElement attributes if present
                me_entity = next((info for entity_id, info in sorted_entities 
                                if info['type'] == 'ManagedElement'), None)
                if me_entity:
                    config += format_attributes(me_entity['attributes'], 'ManagedElement')

                gnb_opened = False
                for entity_id, info in sorted_entities:
                    entity_type = info['type']
                    attributes = info['attributes']

                    if entity_type == 'GNBDUFunction':
                        config += """
                        <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
                            <id>111</id>
                        """
                        config += format_attributes(attributes, 'GNBDUFunction')
                        gnb_opened = True
                        continue

                    elif entity_type == 'NRSectorCarrier':
                        if not gnb_opened:
                            config += """
                            <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
                                <id>111</id>
                            """
                            gnb_opened = True
                        config += """
                            <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
                                <id>111</id>
                        """
                        config += format_attributes(attributes, 'NRSectorCarrier')
                        config += "</NRSectorCarrier>"

                    elif entity_type == 'NRCellDU':
                        if not gnb_opened:
                            config += """
                            <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
                                <id>111</id>
                            """
                            gnb_opened = True
                        config += """
                            <NRCellDU xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrcelldu">
                                <id>111</id>
                        """
                        config += format_attributes(attributes, 'NRCellDU')
                        config += "</NRCellDU>"

                # Close GNBDUFunction tag if it was opened
                if gnb_opened:
                    config += "</GNBDUFunction>"

                config += """
                    </ManagedElement>
                </config>
                """

                logger.info(f"Sending batch Netconf config for group {group_num}:\n{config}")
                m.edit_config(target="running", config=config)
                logger.info(f"Successfully updated Netconf configuration for group {group_num}")

        return True

    except Exception as e:
        logger.error(f"Failed to update Netconf configuration: {str(e)}")
        logger.exception("Full exception details:")
        return False
```

**Configuration result**
Automatically generate the xml netconf config.
![Screenshot 2025-02-14 at 1.34.06 PM](https://hackmd.io/_uploads/rykhlq191e.png)

The OSC DU-High can successfully running after receive the config.
![Screenshot 2025-02-14 at 1.36.37 PM](https://hackmd.io/_uploads/H1rzWc19ke.png)
