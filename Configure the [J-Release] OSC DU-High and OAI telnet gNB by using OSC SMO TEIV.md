---
title: 'Configure the [J-Release] OSC DU-High and OAI telnet gNB by using OSC SMO TEIV'
tags: [SMO]

---

# <center>Configure the [J-Release] OSC DU-High and OAI telnet gNB by using OSC SMO TEIV</center>

<center>SMO O1 Ves + O1 Netconf + [J-Release] OSC DU-High + OAI telnet gNB + OSC SMO TEIV</center>

###### tags: `SMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Jan2 , 2024 3:38 PM][color=#38c16a]

:::info
**Introduction：**

This note is using TEIV consumer rApp to config the OAI telnet O1 gNB and OSC DU-High through O1 netconf.

**Goal：**
- [ ] Complete the rApp function for dynamically change netconf server IP to support the different gNB configuration process.

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

## 1. System Architecture

![ggggg](https://hackmd.io/_uploads/rknJLBt5yl.png)

## 2. Implement the function to dynamically change netconf server IP to support the different gNB configuration process.

![Screenshot 2025-03-05 at 12.30.59 PM](https://hackmd.io/_uploads/B1zV3IHiyx.png)


We need to read the PNF Registration event to get the netconf server IP and port.

**OAI telnet O1 gNB PNF registration ves event**
![Screenshot 2024-10-28 at 3.29.26 PM](https://hackmd.io/_uploads/B1ZyyThe1l.png)

**OSC DU-High PNF registration ves event**
![Screenshot 2025-02-23 at 5.19.17 PM](https://hackmd.io/_uploads/Hy5SYD_q1x.png)

### 2-1. Dynamically change netconf server IP function code.

**netconfclient.py**
```javascript=
def extract_device_name(entity_id):
    try:
        if isinstance(entity_id, str):
            if "ManagedElement=" in entity_id:
                parts = entity_id.split("ManagedElement=")[1].split(",")[0]
                return parts
        return None
    except Exception as e:
        logger.error(f"Error extracting device name: {e}")
        return None

def get_netconf_settings_for_entity(entity_name, reg_manager=None):
    if reg_manager and entity_name in reg_manager.netconf_settings:
        logger.info(f"Using stored NETCONF settings for {entity_name}")
        return reg_manager.netconf_settings[entity_name]
    
    logger.info(f"No specific settings found for {entity_name}, using defaults")
    return NETCONF_SETTINGS

def get_netconf_connection_for_entity(entity_name, reg_manager=None):
    settings = get_netconf_settings_for_entity(entity_name, reg_manager)
    return manager.connect(**settings)
```
**main.py**
```javascript=
def process_deployed_objects(service_address, deployed_ids, kafka_namespace, kafka_topic, kafka_pod_name, reg_manager=None):
    netconf_config = get_netconf_config(reg_manager=reg_manager)
    if relationship_groups:
        success = batch_update_netconf_config(relationship_groups, service_address, reg_manager)
    if updates_needed:
        success = update_netconf_config(
            managed_element_id, gnbdu_function_id, nr_sector_carrier_id, nr_cell_du_id, 
            updates_needed, entity_type, reg_manager
        )
```

ODU-High and OAI telnet O1 gNB PNF Registraiton event. rApp will store the netconf settiing. 

![Screenshot 2025-03-04 at 5.46.15 PM](https://hackmd.io/_uploads/S1bdCSNikg.png)

![Screenshot 2025-03-04 at 5.48.25 PM](https://hackmd.io/_uploads/SkWuRHNsyl.png)

This part rApp will finding the the netconf setting for specific PNF. 

![Screenshot 2025-03-04 at 5.47.04 PM](https://hackmd.io/_uploads/SkbdArNiyg.png)

![Screenshot 2025-03-04 at 5.49.32 PM](https://hackmd.io/_uploads/H1ZuArViJl.png)

However rApp can recognize different PNF netconf setting but some bugs need to be solved.
![Screenshot 2025-03-04 at 5.49.57 PM](https://hackmd.io/_uploads/SkbdASEsyg.png)
