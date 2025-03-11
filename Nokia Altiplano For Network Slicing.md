# <center>Nokia Altiplano For Network Slicing</center>

<center>Study Note of the Network Slicing Feature</center>

###### tags: `Nokia Altiplano`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue2024 8:39 PM][color=#38c16a]

:::info
**Introduction :**

- Learning how to use the network slicing in the Nokia Altiplano.

**Goal：**

- [ ] Implement the network slicing in Nokia Altiplano.
:::
**<center>Table of Content</center>**
[TOC]

## 1. Introduction

When the Network Slicing feature is enabled, we can divide the management into two roles: Virtual Network Operators (VNOs) and Infrastructure Providers (InPs). In this example:

![image007](https://hackmd.io/_uploads/Bkmr9n6N0.png)

**Reference : This figure provide by Nokia.**

1. Infrastructure Provider (InP) --> **(Slice Manager)**:

- The InP is primarily responsible for maintaining the network infrastructure, such as hardware equipment, network connections, etc.
- The InP ensures the stability, reliability, and performance of the network and provides the necessary network resources to the VNOs.

2. Virtual Network Operator (VNO) --> **(Slice Owner)**:

- The VNO is the main "user" of the network slicing service.
- The VNO can view and edit network slicing services related to them, such as setting Quality of Service (QoS) parameters, managing user subscriptions, etc.
- The VNO can use the network resources provided by the InP to offer differentiated services based on their business needs.

Through this division of roles, the InP focuses on the maintenance and management of the network infrastructure, while the VNO focuses on using network resources to provide services. This model can improve the utilization efficiency of network resources and allow multiple VNOs to share the same physical network infrastructure, thereby realizing the commercial model of network slicing.

### 1-1. Following terminologies in the slicing scenario
**Slice Manager :** The Slice Manager can view the configuration backup, which is the backups of the physical device in the CONFIG HISTORY. 

## 2. Slicing Manager Connectivity Profile
In order to create a Device Manager to access device slices at the Slicing Manager AC.

:::danger
**Notice :**
Due to the license expiration, all images are sourced from the Nokia Altiplano User Guide.
:::

### 2-1. Select the tab 'SLICING MANAGER CONNECTIVITY PROFILE'

![Screenshot 2024-06-06 at 5.00.10 PM](https://hackmd.io/_uploads/B1w39gyrC.png)

### 2-2. Add a new slicing manager connectivity profile

![Screenshot 2024-06-06 at 5.04.07 PM](https://hackmd.io/_uploads/Sklosekr0.png)

### 2-3. Select the slicing manager profile.

Use the field 'Slicing Manager Connectivity Profile' in the section 'Network Virtualizer' when you creating the device manager.

![Screenshot 2024-06-07 at 2.32.35 PM](https://hackmd.io/_uploads/BJ85FmerA.png)

## 3. Slicing Manager Application
**Slicing Manager Application :**
- Have an overview of the Slice Owners
- View the fiber slices and the managed ONTs of the Slice Owners
- Have a list of Fiber slices of each Slice Owner, and their key parameters

### 3-1. ImportingtheSliceMappingPackage
In the Slicing Manager Application, click on the Settings icon and select 'Models' to navigate to the Models page.

![Screenshot 2024-06-14 at 3.14.27 PM](https://hackmd.io/_uploads/rJFyAwFrR.png)

### 3-2. FeaturesofSlicingManagerApplication
The slicing manager application has two tabs, SLICE OWNERS and FIBERS.

Basic information for SLICE OWNERS page:
- names of the slice owners
- Username
- Number of Fiber slices
- Number of ONTs

![Screenshot 2024-06-14 at 3.19.14 PM](https://hackmd.io/_uploads/BJNZkOtHR.png)

You can click each of them to see more detail.

![Screenshot 2024-06-14 at 3.21.22 PM](https://hackmd.io/_uploads/ByzF1OFSA.png)

On the FIBERS tab, you can view the list of fibers with Fiber Name, Type, PON Port, Allocation Profile, Downstream CIR, Upstream CIR etc. 

![Screenshot 2024-06-14 at 3.22.55 PM](https://hackmd.io/_uploads/ryxkeOtH0.png)

Click on a row to navigate to the details of the selected fiber. 

![Screenshot 2024-06-14 at 3.24.26 PM](https://hackmd.io/_uploads/B1c4edFBC.png)

## 4. Network Slicing Using Device Manager Application
The Web UI of the Network Virtualizer provides the support related to network slicing which can help troubleshooting purposes.

### 4-1. DetailsViewofaDevice
When slices have been created on a device, the icon ![Screenshot 2024-06-14 at 3.33.25 PM](https://hackmd.io/_uploads/rJOUGOYrA.png =3%x) is displayed in the Network and Device views.

![Screenshot 2024-06-14 at 3.37.37 PM](https://hackmd.io/_uploads/H1f8X_YB0.png)

### 4-2. Viewing all Slicescon figured on adevice
The network slices are configured on the device using IBN provisioning.

After clicking on the 'SLICES' tab, all slices created for that device are displayed. A device slice is displayed with:
- The slice name, Example: 'test-device-blue'
- The slice owner of the slice, Example: 'blue'

![Screenshot 2024-06-14 at 3.44.10 PM](https://hackmd.io/_uploads/Hk20E_FSA.png)

### 4-3. Slice Details View
After clicking on the 'DETAILS' button of a slice, the details view of that slice is displayed.

![Screenshot 2024-06-14 at 3.46.36 PM](https://hackmd.io/_uploads/HJ1dS_YSR.png)

"Show device slice datastore" icon ![Screenshot 2024-06-21 at 3.38.22 PM](https://hackmd.io/_uploads/S1dbRjfL0.png =3%x) is provided to display the contents of the slice. 

The contents of the slice is a subset of the device datastore provided as a NETCONF YANG XML document.

![Screenshot 2024-06-21 at 3.46.01 PM](https://hackmd.io/_uploads/H1DIg2GLR.png)

:::danger
**Notice :**
Due to the license expiration, all images are sourced from the Nokia Altiplano User Guide.
:::
## Reference
1. Nokia Altiplano Access Controller and FastMile Controller Release 23.3 User Guide
