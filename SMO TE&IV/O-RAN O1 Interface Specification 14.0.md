# <center>O-RAN O1 Interface Specification 14.0</center>

<center>Study Note for O-RAN.WG10.O1-Interface.0-R004-v14.00</center>

###### tags: `SMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Nov1 , 2024 3:38 PM][color=#38c16a]

:::info
**Introduction：**

This document specifies management services for the O-RAN O1 interface and includes requirements, notifications and protocols.

**Reference：**
- [O-RAN O1 Interface Specification 14.0](https://specifications.o-ran.org/specifications)
:::
**<center>Table of Content</center>**
[TOC]

## 1. Management Services
### 1-1. Provisioning Management Services 
Provisioning management services enable a Provisioning MnS Consumer to configure managed object attributes on the Provisioning MnS Producer, affecting its capabilities in network services, and allow the Producer to report configuration changes back to the Consumer. These services use NETCONF for creating, deleting, modifying, and reading managed object attributes, with YANG-modeled RESTful/HTTP notifications for change alerts to subscribed Consumers.
### 1-2. General NETCONF Requirements
The provisioning management service producer and consumer shall support the following NETCONF operations.
- get
- get-config
- edit-config
- lock
- unlock
- close-session
- kill-session

The provisioning management service producer and consumer shall support the following NETCONF capabilities.
- writable-running
- rollback-on-error
- validate
### 1-3. Create Managed Object Instance
![Screenshot 2024-11-01 at 6.53.43 PM](https://hackmd.io/_uploads/HkgK7VfZyx.png)

### 1-4. Modify Managed Object Instance Attributes
![Screenshot 2024-11-01 at 6.55.25 PM](https://hackmd.io/_uploads/HJynQNf-kl.png)

### 1-5. Delete Managed Object Instance
![Screenshot 2024-11-01 at 6.56.06 PM](https://hackmd.io/_uploads/B1_0XEGW1x.png)

### 1-6. Read Managed Object Instance Attributes
![Screenshot 2024-11-01 at 6.56.50 PM](https://hackmd.io/_uploads/SkmW44zbkg.png)

### 1-7. Notify Managed Object Instance Changes
![Screenshot 2024-11-01 at 7.15.45 PM](https://hackmd.io/_uploads/SyV_dEf-Je.png)

### 1-8. Subscription Control
Subscription Control allows a MnS Consumer to subscribe to notifications emitted by a MnS Producer.

### 1-9. NETCONF Session Establishment
![Screenshot 2024-11-01 at 7.22.23 PM](https://hackmd.io/_uploads/SkgW5EG-1l.png)

### 1-10. NETCONF Session Termination
![Screenshot 2024-11-01 at 7.23.12 PM](https://hackmd.io/_uploads/HJfV9NGbJl.png)

### 1-11. Lock Data Store
![Screenshot 2024-11-01 at 7.23.56 PM](https://hackmd.io/_uploads/BkRI54fbJx.png)

### 1-12. Unlock Data Store
![Screenshot 2024-11-01 at 7.26.00 PM](https://hackmd.io/_uploads/SJiR5Ef-Je.png)

### 1-13. Commit
Provisioning MnS Consumer uses the Commit procedure to commit a configuration change to the running data store of the Provisioning MnS Producer. This is necessary to make the configuration change effective if it was made in the candidate data store. If the configuration change was made in the running data store, the commit procedure is not used.

![Screenshot 2024-11-01 at 7.27.36 PM](https://hackmd.io/_uploads/r15EsVzWJe.png)

### 1-14. Notify Event
Provisioning MnS Producer sends an asynchronous notifyEvent Notification to the Provisioning MnS Consumer to report a network event has occurred with potential service impacts.

![Screenshot 2024-11-01 at 7.33.40 PM](https://hackmd.io/_uploads/HkLj24fWkx.png)

## 2. Fault Supervision Management Services
Fault supervision management services allow a Fault Supervision MnS Producer to report errors and events to a Fault Supervision MnS Consumer and allows a Fault Supervision MnS Consumer to perform fault supervision operations on the Fault Supervision MnS Producer, such as get alarm list. The Fault supervision management services include the optional capability of Fault History Supervision Control and Reporting.

### 2-1. Fault Notification
Fault Supervision MnS Producer sends asynchronous Fault notification event to Fault Supervision MnS Consumer when an alarm occurs, is cleared, or changes severity.

The Fault Supervision MnS Producer shall support the following 3GPP-specified Fault Supervision notifications are:
- notifyNewAlarm
- notifyChangedAlarmGeneral and/or notifyChangedAlarm
- notifyClearedAlarm
- notifyAlarmListRebuilt
- notifyAckStateChanged

### 2-2. Fault Supervision Control
There is one AlarmList per Fault Supervision MnS Producer, created by the Producer. The AlarmList contains one AlarmRecord for each active alarm. The AlarmRecords in the AlarmList can be read by the Fault Supervision MnS Consumer, with an optional filter to retrieve selected AlarmRecords based on the value of attributes in the AlarmRecord. 

### 2-3. Fault History Supervision Control and Reporting
The Fault History Supervision Control and Reporting allows a Fault Supervision MnS Producer to report to a Fault Supervision MnS Consumer relevant information about raised, changed, or cleared alarms in the past. Generalizing, it allows to report information about alarms updates. When an alarm is raised, the Fault Supervision MnS Producer stores the relevant initial data of a new alarm, according to the information exposed by the ==notifyNewAlarm notification==. When an alarm is modified or cleared, the Fault Supervision MnS Producer stores the relevant data of the alarm change, according to the information exposed by the ==notifyChangeAlarm/notifyChangedAlarmGeneral, or notifyClearedAlarm==. When an alarm list is locked or disabled, alarm records are not reliable, as consequences also the capability of the Fault Supervision MnS Producer to store alarm changes is not reliable.

## 3. Performance Assurance Management Services
Performance Assurance Management Services allow a Performance Assurance MnS Producer to report file-based (bulk) and/or streaming (real time) performance data to a Performance Assurance MnS Consumer and allows a Performance Assurance MnS Consumer to perform performance assurance operations on the Performance Assurance MnS Producer, such as selecting the measurements to be reported and setting the frequency of reporting.

### 3-1. Performance Data File Reporting
A Performance Assurance MnS Producer that provides file-based (bulk) performance data reporting shall support the PM data pull-based file reporting method and/or the PM data push-based file reporting method.

- ==Pull Method==: The MnS Producer sends a notifyFileReady notification when the PM file is ready, including the filename and location. If there’s an error during preparation, it sends a notifyFilePreparationError notification. The MnS Consumer retrieves the file based on this notification.
:::success
**Notice!!** In OAI O1 adapter, they use **Pull Method**.
:::
- ==Push Method==: The MnS Producer transfers the file to a designated server or MnS Consumer, with the location specified by the "FileLocation" attribute in the PerfMetricJob IOC. If notification is supported, it sends notifyFileReady or notifyFilePreparationError upon file readiness or error.

#### 3-1-1. Pull-based Performance Data File Reporting Procedure for file ready events

![Screenshot 2024-11-02 at 6.57.07 AM](https://hackmd.io/_uploads/SJJ1T0fZ1g.png)

#### 3-1-2. Pull-based Performance Data File Reporting Procedure for file preparation error events

![Screenshot 2024-11-02 at 6.59.28 AM](https://hackmd.io/_uploads/BybwTAG-yx.png)

#### 3-1-3. Push-based Performance Data File Reporting Procedure with optional file ready event

![Screenshot 2024-11-02 at 7.13.45 AM](https://hackmd.io/_uploads/rkfRxk7-Jl.png)

#### 3-1-4. Push-based Performance Data File Reporting Procedure with optional file preparation error event
![Screenshot 2024-11-02 at 7.15.09 AM](https://hackmd.io/_uploads/rkJfZ17Zye.png)

### 3-2. Performance Data Streaming

![Screenshot 2024-11-02 at 7.23.15 AM](https://hackmd.io/_uploads/BkFe7J7-1x.png)

## 4. File Management Services
### 4-1. File Ready Notification

![Screenshot 2024-11-02 at 7.49.18 AM](https://hackmd.io/_uploads/BykMF1Qb1g.png)

### 4-2. List Available Files
![Screenshot 2024-11-02 at 7.52.31 AM](https://hackmd.io/_uploads/SymCF1m-ye.png)

### 4-3. File Transfer to and from File Management MnS Producer
A need to transfer a file to a known location on the File Management MnS Producer. Some examples of files that could be transferred to the FileManagement MnS Producer are:
- Beamforming configuration file (Opaque Vendor specific data)
- Machine Learning
- Certificates

![Screenshot 2024-11-02 at 8.01.38 AM](https://hackmd.io/_uploads/ByNxnkXZJg.png)

### 4-4. Download File from remote file server
The File Management MnS Consumer has a file that needs to be downloaded to the File Management MnS Producer
such as:
- Software file to upgrade software version executed on the File Management MnS Producer
- Beamforming configuration file (Opaque Vendor specific data)
- Machine Learning
- Certificates

The File Management MnS Consumer triggers the file download. The File Management MnS Producer uses a secure file transfer protocol to download the file from the location specified by the File Management MnS Consumer and then notifies the File Management MnS Consumer of the result of the download. In this use case, the File Management MnS Producer is the client. The file could be located on any File Server reachable by the File Management MnS Producer.
![Screenshot 2024-11-02 at 8.02.48 AM](https://hackmd.io/_uploads/Hki43J7Wyx.png)
:::danger
**Notice!!** There are no File Download notifications defined in the present document.
:::

## 5. Heartbeat Management Capability
Heartbeat management capability allows an MnS Producer to send heartbeats to an MnS Consumer and allows an MnS Consumer to configure the heartbeat
services on an MnS Producer.
### 5-1. Heartbeat Notification
MnS Producer sends asynchronous heartbeat notifications to MnS Consumer at a configurable frequency to allow MnS
Consumer to supervise the connectivity from the MnS Producer.
### 5-2. Heartbeat Control
NETCONF protocol and YANG data models are used to read and configure the heartbeatNtfPeriod and triggerHeartbeatNtf in the HeartbeatControl IOC.

## 6. PNF Startup and Registration Management Services
This services allow a MnS Producer to acquire its network layer parameters either via static procedures (pre-configured in the element) or via dynamic procedures (Plug-n-Connect) during startup.During this process, the MnS Producer also acquires the IP address of the MnS Consumer for MnS Producer registration. Once the MnS Producer registers, the MnS Consumer can then bring the MnS Producer to an operational state.

### 6-1. PNF Plug-n-Connect
PNF Plug-n-Connect (PnC) scenario enables a MnS Producer to obtain the necessary start-up configuration to allow it to register with a MnS Consumer for subsequent management.
### 6-2. PNF Registration
MnS Producer sends an asynchronous ==pnfRegistration== or ==o1NotifyPNFRegistration== event to a  MnS Consumer after PnC to notify MnS Consumer of new MnS Producer to be managed.

![Screenshot 2024-11-03 at 6.11.32 AM](https://hackmd.io/_uploads/B1hAQQV-ke.png)

## 7. PNF Software Management Services
Software management services allow a MnS Consumer to request a physical MnS Producer to download, install, validate and activate a new software package and allow a physical MnS Producer to report its software versions.

### 7-1. Software Inventory
The MnS Consumer sends a Software Inventory Request and retrieves information about the software packages on the MnS Producer.

![Screenshot 2024-11-03 at 6.29.24 AM](https://hackmd.io/_uploads/Syr0wmN-1e.png)

### 7-2. Software Download
Software Download triggers the download of a specific software package to the PNF Software MnS Producer. This download service includes integrity checks on the downloaded software and the installation of the software into the software slot corresponding to the softwarePackage MOI.

![Screenshot 2024-11-03 at 6.32.58 AM](https://hackmd.io/_uploads/HkAo_Q4byg.png)

### 7-3. Software Activation Pre-Check
Activation Pre-check is an optional Use Case that the Service Provider can choose to utilize prior to software activation to confirm that the PNF Software MnS Producer is in a good state to activate the new software and provide information needed for planning the timing of the software replacement--such as whether a reset or a data migration is required.

![Screenshot 2024-11-03 at 6.42.35 AM](https://hackmd.io/_uploads/S1GfsQ4-Jx.png)

### 7-4. Software Activate
PNF Software MnS Consumer triggers the activation of a software package on the PNF Software MnS Producer including data migration and reset if needed.

![Screenshot 2024-11-03 at 6.45.39 AM](https://hackmd.io/_uploads/ryKooQE-Je.png)

## 8. PNF Reset Management Services
PNF Reset Management Services allow a PNF Reset MnS Consumer to trigger a reset of a HW unit of a PNF Reset MnS Producer on command.
### 8-1. PNF Reset Command
![Screenshot 2024-11-03 at 6.54.04 AM](https://hackmd.io/_uploads/BkejpQVW1e.png)

## Annex1. (informative) Guidelines and Example for stndDefined VES Events
### Annex1.1 Guidelines for use of stndDefined VES for sending 3GPP- specified or O-RAN-specified O1 notifications

- Function of stndDefined VES Event:
    - According to the VES Event Listener Specification [18], the ==stndDefined== VES event allows a VES event to carry a notification defined by a standards development organization (SDO) as its payload. 
    - In the O-RAN O1 Interface Specification, a ==stndDefined== VES event can carry either a 3GPP-specified or an O-RAN-specified O1 notification.
- Event Routing and Namespace:
    - The ==domain== field in the VES common event header is used to route the event to the correct consumers and map it to the schema for the event payload. 
    - ==stndDefined== is a new value in the ==domain== field, indicating that the event follows a schema defined by a standards body. 
    - Another added field, ==stndDefinedNamespace==, holds a valid namespace as defined by the standards body and is populated only when ==domain== is set to ==stndDefined==. 
    - 3GPP and O-RAN have each defined their respective namespaces, and a VES collector uses these namespaces along with the ==stndDefined== domain to route the event.
- stndDefinedFields Structure:
    - ==stndDefinedFields== contains three properties:
        - ==schemaReference== (string in URI format): used to verify the correctness of the notification content.
        - ==data== (JSON object): contains the 3GPP or O-RAN notification in JSON format.
        - ==stndDefinedFieldsVersion== (string in enum format): specifies the version of the ==stndDefinedFields== structure.
- Schema Reference and Data Content:
    - ==schemaReference== (if present) can be used to verify the accuracy of the notification content.
    - 3GPP and O-RAN will each publish their notification schemas in OpenAPI format in a public repository, allowing these schema references to be included in the event.
    - The ==data== element holds either a 3GPP O1 notification as specified in 3GPP TS 28.532 [3] or an O-RAN O1 notification as defined by the O-RAN Network Resource Model Specification in JSON format.

### Annex1.2 Example stndDefined VES event for a new alarm notification

The VES Common Header is shown from line 44 through line 58. It contains:
- the domain set to stndDefined
- the stndDefinedNamespace set to 3GPP-FaultSupervision.

The stndDefinedFields structure begins on line 59. It contains:
- the 3GPP schema reference for the 3GPP fault notification type
- the data element which contains the full 3GPP notifyNewAlarm fault notification
- the version of the stndDefinedFields.


```javascript=
{
  "event": {
    "commonEventHeader": {
      "domain": "stndDefined",
      "eventId": "stndDefined-gNB-Nokia-000001",
      "eventName": "stndDefined-gNB-Nokia-ProcessingErrorAlarm-351",
      "lastEpochMicrosec": 1594909352208000,
      "priority": "Normal",
      "reportingEntityName": "NOKb5309",
      "sequence": 0,
      "sourceName": "NOKb5309",
      "startEpochMicrosec": 1594909352208000,
      "stndDefinedNamespace": "3GPP-FaultSupervision",
      "timeZoneOffset": "UTC-05.00",
      "version": "4.1",
      "vesEventListenerVersion": "7.2"
    },
    "stndDefinedFields": {
      "schemaReference": "https://forge.3gpp.org/rep/sa5/MnS/-/blob/Rel-17/OpenAPI/TS28532_FaultMnS.yaml#/components/schemas/NotifyNewAlarm",
      "data": {
        "href": "http://operatorA.com/SubNetwork=south/ManagedElement=1/gNBFunction=1",
        "notificationId": "123",
        "notificationType": "notifyNewAlarm",
        "eventTime": "2023-11-15T23:20:50.52Z",
        "systemDN": "xyz",
        "probableCause": "351",
        "perceivedSeverity": "MAJOR",
        "specificProblem": "7052",
        "additionalText": "xyz",
        "additionalInformation": [],
        "alarmId": "15",
        "alarmType": "PROCESSING_ERROR_ALARM"
      },
      "stndDefinedFieldsVersion": "1.0"
    }
  }
}
```