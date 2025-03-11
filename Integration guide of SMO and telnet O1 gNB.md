# <center>Integration guide of SMO and telnet O1 gNB</center>

<center>SMO O1 Ves + O1 Netconf + O1 Adapter + telnet O1 gNB</center>

###### tags: `SMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Oct23 , 2024 3:38 PM][color=#38c16a]

:::info
**Introduction：**

This note is to teach you how to simply deploy and connect telnet o1 gNB and o1 adapter and also using SMO to successful get config.

**Goal：**
- [x] [Install telnet O1 gNB](#2-Install-telnet-O1-gNB)
- [x] [Install O1 Adapter](#3-Install-O1-Adapter)
- [x] [O1 adpater netconf function testing](#4-O1-adpater-netconf-function-testing)
- [x] [O1 adapter ves functin testing](#5-O1-adapter-ves-functin-testing)
- [x] [SMO edit netconf config testing](#6-SMO-edit-netconf-config-testing)
- [x] [Demo video](#7-Demo-video)

**Reference：**
- [[Installation guide] o1 adapter](https://hackmd.io/@SylviaCh/BJjlE2CT0)
- [[installation guide] telnet o1 gNB](https://hackmd.io/@SylviaCh/By5OyMOTR)
- [Intergration note of O1 adapter and telnet o1 gNB.](https://hackmd.io/@SylviaCh/HJLHco80C)
:::
**<center>Table of Content</center>**
[TOC]

## 1. System Architecture
![ccccc](https://hackmd.io/_uploads/rkFRu18xkx.png)

## 2. Install telnet O1 gNB
:::info
**What is OAI Telnet interface? And why we use it?**
The OAI Telnet interface provides access to control and monitor the OpenAirInterface (OAI) softmodem program. Brief summary of its functionalities:

- Monitor: Allows observing real-time information about the softmodem, such as:
    - Signal strength
    - Cell configuration
    - Resource utilization
- Control: Enables sending commands to modify various softmodem parameters like:
    - Cell configuration
    - Power levels
    - Operating modes
- Debug: Provides access to internal data for troubleshooting purposes.

**The OAI Telnet interface for O1**
The OAI Telnet extension for O1 is providing specialized functionality for O1 NETCONF Server. Attributes are converted and prepared to the format and structure, required by related yang specification. The groups of information are
- Monitoring information
    - like Channel bandwidth, load for downlink and uplink, SSB Frequency,  ..
    - or number of active UEs for PM Data recording MeanActiveUeDl, MeanActiveUeUl, MaxActiveUeUl and MaxActiveUeDl
- Alarm notifications, like connection state to OAI Telnet interface.
:::
### 2-1. Preparatory work

Install the dependency tools.
```javascript=
sudo apt install cmake
sudo apt install ninja-build
sudo apt install asn1c
sudo apt install ccache
./build_oai -I
```
git clone the telnet-o1 branch
```javascript=
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git --branch telnet-o1 --single-branch
```
### 2-2. Build telnet O1 gNB

```javascript=
cd openairinterface5g/cmake_targets/
./build_oai --ninja -c --gNB --nrUE --build-lib telnetsrv
```
### 2-3. Run telnet O1 gNB

Revise the gnb.sa.band78.fr1.106PRB.usrpb210.conf
```javascript=
cd openairinterface5g/targets/PROJECTS/GENERIC-NR-5GC/CONF/
```
```javascript=
vim gnb.sa.band78.fr1.106PRB.usrpb210.conf
```
:::spoiler gnb.sa.band78.fr1.106PRB.usrpb210.conf
```javascript=
Active_gNBs = ( "gNB-OAI");
# Asn1_verbosity, choice in: none, info, annoying
Asn1_verbosity = "none";

gNBs =
(
 {
    ////////// Identification parameters:
    gNB_ID    =  0xe00;
    gNB_name  =  "gNB-OAI";

    // Tracking area code, 0x0000 and 0xfffe are reserved values
    tracking_area_code  =  1;
    plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2; snssaiList = ({ sst = 1; }) });

    nr_cellid = 12345678L;

    ////////// Physical parameters:

    do_CSIRS                                                  = 1;
    do_SRS                                                    = 1;

    servingCellConfigCommon = (
    {
 #spCellConfigCommon

      physCellId                                                    = 0;

#  downlinkConfigCommon
    #frequencyInfoDL
      # this is 3600 MHz + 43 PRBs@30kHz SCS (same as initial BWP)
      absoluteFrequencySSB                                             = 641280;
      dl_frequencyBand                                                 = 78;
      # this is 3600 MHz
      dl_absoluteFrequencyPointA                                       = 640008;
      #scs-SpecificCarrierList
        dl_offstToCarrier                                              = 0;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        dl_subcarrierSpacing                                           = 1;
        dl_carrierBandwidth                                            = 106;
     #initialDownlinkBWP
      #genericParameters
        # this is RBstart=27,L=48 (275*(L-1))+RBstart
        initialDLBWPlocationAndBandwidth                               = 28875; # 6366 12925 12956 28875 12952
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        initialDLBWPsubcarrierSpacing                                   = 1;
      #pdcch-ConfigCommon
        initialDLBWPcontrolResourceSetZero                              = 12;
        initialDLBWPsearchSpaceZero                                     = 0;

  #uplinkConfigCommon
     #frequencyInfoUL
      ul_frequencyBand                                              = 78;
      #scs-SpecificCarrierList
      ul_offstToCarrier                                             = 0;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      ul_subcarrierSpacing                                          = 1;
      ul_carrierBandwidth                                           = 106;
      pMax                                                          = 20;
     #initialUplinkBWP
      #genericParameters
        initialULBWPlocationAndBandwidth                            = 28875;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        initialULBWPsubcarrierSpacing                               = 1;
      #rach-ConfigCommon
        #rach-ConfigGeneric
          prach_ConfigurationIndex                                  = 98;
#prach_msg1_FDM
#0 = one, 1=two, 2=four, 3=eight
          prach_msg1_FDM                                            = 0;
          prach_msg1_FrequencyStart                                 = 0;
          zeroCorrelationZoneConfig                                 = 13;
          preambleReceivedTargetPower                               = -96;
#preamblTransMax (0...10) = (3,4,5,6,7,8,10,20,50,100,200)
          preambleTransMax                                          = 6;
#powerRampingStep
# 0=dB0,1=dB2,2=dB4,3=dB6
        powerRampingStep                                            = 1;
#ra_ReponseWindow
#1,2,4,8,10,20,40,80
        ra_ResponseWindow                                           = 4;
#ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR
#1=oneeighth,2=onefourth,3=half,4=one,5=two,6=four,7=eight,8=sixteen
        ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR                = 4;
#oneHalf (0..15) 4,8,12,16,...60,64
        ssb_perRACH_OccasionAndCB_PreamblesPerSSB                   = 14;
#ra_ContentionResolutionTimer
#(0..7) 8,16,24,32,40,48,56,64
        ra_ContentionResolutionTimer                                = 7;
        rsrp_ThresholdSSB                                           = 19;
#prach-RootSequenceIndex_PR
#1 = 839, 2 = 139
        prach_RootSequenceIndex_PR                                  = 2;
        prach_RootSequenceIndex                                     = 1;
        # SCS for msg1, can only be 15 for 30 kHz < 6 GHz, takes precendence over the one derived from prach-ConfigIndex
        #
        msg1_SubcarrierSpacing                                      = 1,
# restrictedSetConfig
# 0=unrestricted, 1=restricted type A, 2=restricted type B
        restrictedSetConfig                                         = 0,

        msg3_DeltaPreamble                                          = 1;
        p0_NominalWithGrant                                         =-90;

# pucch-ConfigCommon setup :
# pucchGroupHopping
# 0 = neither, 1= group hopping, 2=sequence hopping
        pucchGroupHopping                                           = 0;
        hoppingId                                                   = 40;
        p0_nominal                                                  = -90;
# ssb_PositionsInBurs_BitmapPR
# 1=short, 2=medium, 3=long
      ssb_PositionsInBurst_PR                                       = 2;
      ssb_PositionsInBurst_Bitmap                                   = 1;

# ssb_periodicityServingCell
# 0 = ms5, 1=ms10, 2=ms20, 3=ms40, 4=ms80, 5=ms160, 6=spare2, 7=spare1
      ssb_periodicityServingCell                                    = 2;

# dmrs_TypeA_position
# 0 = pos2, 1 = pos3
      dmrs_TypeA_Position                                           = 0;

# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      subcarrierSpacing                                             = 1;


  #tdd-UL-DL-ConfigurationCommon
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      referenceSubcarrierSpacing                                    = 1;
      # pattern1
      # dl_UL_TransmissionPeriodicity
      # 0=ms0p5, 1=ms0p625, 2=ms1, 3=ms1p25, 4=ms2, 5=ms2p5, 6=ms5, 7=ms10
      dl_UL_TransmissionPeriodicity                                 = 6;
      nrofDownlinkSlots                                             = 7;
      nrofDownlinkSymbols                                           = 6;
      nrofUplinkSlots                                               = 2;
      nrofUplinkSymbols                                             = 4;

      ssPBCH_BlockPower                                             = -25;
  }

  );


    # ------- SCTP definitions
    SCTP :
    {
        # Number of streams to use in input/output
        SCTP_INSTREAMS  = 2;
        SCTP_OUTSTREAMS = 2;
    };


    ////////// AMF parameters:
    amf_ip_address      = ( { ipv4       = "192.168.8.108";
                              ipv6       = "192.168.8.108";
                              active     = "yes";
                              preference = "ipv4";
                            }
                          );


    NETWORK_INTERFACES :
    {
        GNB_INTERFACE_NAME_FOR_NG_AMF            = "demo-oai";
        GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.8.27/24";
        GNB_INTERFACE_NAME_FOR_NGU               = "demo-oai";
        GNB_IPV4_ADDRESS_FOR_NGU                 = "192.168.8.27/24";
        GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
    };

  }
);

MACRLCs = (
{
  num_cc                      = 1;
  tr_s_preference             = "local_L1";
  tr_n_preference             = "local_RRC";
  pusch_TargetSNRx10          = 150;
  pucch_TargetSNRx10          = 200;
  ulsch_max_frame_inactivity  = 0;
  ul_max_mcs                  = 28;
}
);

L1s = (
{
  num_cc = 1;
  tr_n_preference       = "local_mac";
  prach_dtx_threshold   = 120;
  pucch0_dtx_threshold  = 100;
  ofdm_offset_divisor   = 8; #set this to UINT_MAX for offset 0
}
);

RUs = (
{
  local_rf       = "yes"
  nb_tx          = 1
  nb_rx          = 1
  att_tx         = 12;
  att_rx         = 12;
  bands          = [78];
  max_pdschReferenceSignalPower = -27;
  max_rxgain                    = 114;
  eNB_instances  = [0];
  #beamforming 1x4 matrix:
  bf_weights = [0x00007fff, 0x0000, 0x0000, 0x0000];
  clock_src = "internal";
}
);

THREAD_STRUCT = (
{
  #three config for level of parallelism "PARALLEL_SINGLE_THREAD", "PARALLEL_RU_L1_SPLIT", or "PARALLEL_RU_L1_TRX_SPLIT"
  parallel_config    = "PARALLEL_SINGLE_THREAD";
  #two option for worker "WORKER_DISABLE" or "WORKER_ENABLE"
  worker_config      = "WORKER_ENABLE";
}
);

rfsimulator :
{
  serveraddr = "server";
  serverport = 4043;
  options = (); #("saviq"); or/and "chanmod"
  modelname = "AWGN";
  IQfile = "/tmp/rfsimulator.iqs";
};

security = {
  # preferred ciphering algorithms
  # the first one of the list that an UE supports in chosen
  # valid values: nea0, nea1, nea2, nea3
  ciphering_algorithms = ( "nea0" );

  # preferred integrity algorithms
  # the first one of the list that an UE supports in chosen
  # valid values: nia0, nia1, nia2, nia3
  integrity_algorithms = ( "nia2", "nia0" );

  # setting 'drb_ciphering' to "no" disables ciphering for DRBs, no matter
  # what 'ciphering_algorithms' configures; same thing for 'drb_integrity'
  drb_ciphering = "yes";
  drb_integrity = "no";
};

log_config :
{
  global_log_level                      ="info";
  hw_log_level                          ="info";
  phy_log_level                         ="info";
  mac_log_level                         ="info";
  rlc_log_level                         ="info";
  pdcp_log_level                        ="info";
  rrc_log_level                         ="info";
  ngap_log_level                        ="debug";
  f1ap_log_level                        ="debug";
};

e2_agent = {
  near_ric_ip_addr = "127.0.0.1";
  #sm_dir = "/path/where/the/SMs/are/located/"
  sm_dir = "/usr/local/lib/flexric/"
};
```
:::
```javascript=
cd openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf  --rfsim --sa --telnetsrv --telnetsrv.shrmod o1
```

**Result**
![Screenshot 2024-10-22 at 6.53.49 PM](https://hackmd.io/_uploads/HJ36qgIx1l.png)

### 2-4. Testing telnet O1 gNB telnet function
:::warning
**Notice!!** Note that only one telnet client can be connected at a time.
:::

O1 gNB telnet basic command
```javascript=
echo o1 stats | nc -N 127.0.0.1 9090

echo o1 stats | nc -N 127.0.0.1 9090 | awk '/^{$/, /^}$/' | jq .

echo o1 bwconfig 20 | nc -N 127.0.0.1 9090

echo o1 bwconfig 40 | nc -N 127.0.0.1 9090

echo o1 bwconfig 60 | nc -N 127.0.0.1 9090

echo o1 bwconfig 100 | nc -N 127.0.0.1 9090

echo o1 stop_modem | nc -N 127.0.0.1 9090

echo o1 start_modem | nc -N 127.0.0.1 9090
```

or you can customized bu yourself.

```javascript=
echo o1 config nrcelldu3gpp:ssbFrequency 620736 nrcelldu3gpp:arfcnDL 620020 nrcelldu3gpp:bSChannelBwDL 51 bwp3gpp:numberOfRBs 51 bwp3gpp:startRB 0 | nc -N 127.0.0.1 9090
```

:::danger
**Notice!!** OAI O1 telnet gNB dosen't support customized configuration, so you need to follow the following procedure to enable.
:::

Revise telnetsrv_o1.c
```javascript=
cd openairinterface5g/common/utils/telnetsrv/
vim telnetsrv_o1.c
```
```javascript=293
gNB_MAC_INST *mac = RC.nrmac[0];
NR_ServingCellConfigCommon_t *scc = mac->common_channels[0].ServingCellConfigCommon;
NR_FrequencyInfoDL_t *frequencyInfoDL = scc->downlinkConfigCommon->frequencyInfoDL;
NR_BWP_t *initialDL = &scc->downlinkConfigCommon->initialDownlinkBWP->genericParameters;
NR_FrequencyInfoUL_t *frequencyInfoUL = scc->uplinkConfigCommon->frequencyInfoUL;
NR_BWP_t *initialUL = &scc->uplinkConfigCommon->initialUplinkBWP->genericParameters;

//--gNBs.[0].servingCellConfigCommon.[0].absoluteFrequencySSB 620736            -> SSBFREQ
*scc->downlinkConfigCommon->frequencyInfoDL->absoluteFrequencySSB = ssbfreq;

// --gNBs.[0].servingCellConfigCommon.[0].dl_absoluteFrequencyPointA 620020      -> ARFCNDL
frequencyInfoDL->absoluteFrequencyPointA = arfcn;
AssertFatal(frequencyInfoUL->absoluteFrequencyPointA == NULL, "only handle TDD\n");

// --gNBs.[0].servingCellConfigCommon.[0].dl_carrierBandwidth 51                 -> BWDL
frequencyInfoDL->scs_SpecificCarrierList.list.array[0]->carrierBandwidth = bwdl;

// --gNBs.[0].servingCellConfigCommon.[0].initialDLBWPlocationAndBandwidth 13750 -> NUMRBS + STARTRB
initialDL->locationAndBandwidth = locationAndBandwidth;

// --gNBs.[0].servingCellConfigCommon.[0].ul_carrierBandwidth 51                 -> BWUL?
// we assume the same BW as DL
frequencyInfoUL->scs_SpecificCarrierList.list.array[0]->carrierBandwidth = bwdl;

// --gNBs.[0].servingCellConfigCommon.[0].initialULBWPlocationAndBandwidth 13750 -> ?
// we assume same locationAndBandwidth as DL
initialUL->locationAndBandwidth = locationAndBandwidth;
```
After revise, rebuild the gNB again.

```javascript=
cd openairinterface5g/cmake_targets/
./build_oai --ninja -c --gNB --nrUE --build-lib telnetsrv
```

Run telnet O1 gNB.

```javascript=
cd openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf  --rfsim --sa --telnetsrv --telnetsrv.shrmod o1
```

:::warning
**Notice!!** Every time you want to upadte the config, you need to **"o1 stop_modem"** first.
:::

**Orginal Configuration**

![Screenshot 2024-11-08 at 2.42.48 AM](https://hackmd.io/_uploads/rki4qFqbJl.png)


**After using this command to revise**
```javascript=
echo o1 config nrcelldu3gpp:ssbFrequency 641032 nrcelldu3gpp:arfcnDL 640000 nrcelldu3gpp:bSChannelBwDL 273 bwp3gpp:numberOfRBs 273 bwp3gpp:startRB 0 | nc -N 127.0.0.1 9090
```

![Screenshot 2024-11-08 at 2.41.12 AM](https://hackmd.io/_uploads/H1My9YcWkl.png)


## 3. Install O1 Adapter
### 3-1. Preparatory work

Install the dependency tools.
```javascript=
sudo apt install docker.io
```
Clone the o1 adapter from OAI Gitlab
```javascript=
git clone https://gitlab.eurecom.fr/oai/o1-adapter.git
```
### 3-2. Build O1 Adapter
```javascript=
cd o1-adapter/docker/config
vim config.json
```
:::spoiler config.json
```javascript=
{
    "log-level": 3,
    "software-version": "abcd",
    "network": {
        "host": "192.168.8.27",
        "username": "netconf",
        "password": "netconf!",
        "netconf-port": 1830,
        "sftp-port": 1222
    },

    "ves": {
        "template": {
            "new-alarm": "/adapter/config/ves-new-alarm.json",
            "clear-alarm": "/adapter/config/ves-clear-alarm.json",
            "pnf-registration": "/adapter/config/ves-pnf-registration.json",
            "file-ready": "/adapter/config/ves-file-ready.json",
            "heartbeat": "/adapter/config/ves-heartbeat.json",
            "pm-data": "/adapter/config/pmData-measData.xml"
        },

        "pnf-registration": true,
        "heartbeat-interval": 30,

        "url": "http://192.168.8.6:30417/eventListener/v7",
        "username": "sample1",
        "password": "sample1",

        "file-expiry": 86400,
        "pm-data-interval": 30
    },

    "alarms": {
        "internal-connection-lost-timeout": 3,

        "load-downlink-exceeded-warning-threshold": 50,
        "load-downlink-exceeded-warning-timeout": 30
    },

    "telnet": {
        "host": "192.168.8.27",
        "port": 9090
    },

    "info": {
        "gnb-du-id": 0,
        "cell-local-id": 0,
        "node-id": "gNB-Eurecom-5GNRBox-00001",
        "location-name": "MountPoint 05, Rack 234-17, Room 234, 2nd Floor, Körnerstraße 7, 10785 Berlin, Germany, Europe, Earth, Solar-System, Universe",
        "managed-by": "ManagementSystem=O-RAN-SC-ONAP-based-SMO",
        "managed-element-type": "NodeB",
        "model": "nr-softmodem",
        "unit-type": "gNB"
    }
}
```
:::

Build the Docker image.
```javascript=
cd o1-adapter
sudo ./build-adapter.sh --adapter --no-cache
```
### 3-3. Run O1 adpater
```javascript=
sudo ./start-adapter.sh --adapter
```
**Result**
**O1 adapter waiting to connect**
![Screenshot 2024-10-22 at 5.43.56 PM](https://hackmd.io/_uploads/SJsrnZIg1l.png)
**Telnet client connected**
![Screenshot 2024-10-22 at 7.36.50 PM](https://hackmd.io/_uploads/BJSEh-LgJe.png)
**O1 adapter already get the config from gNB through telnet**
![Screenshot 2024-10-22 at 8.56.41 PM](https://hackmd.io/_uploads/B1LLnZUxJe.png)

### 3-4. Stop the O1 adpater
```javascript=
sudo docker stop adapter-gnb
sudo docker rm adapter-gnb
```
## 4. O1 adpater netconf function testing
```javascript=
python3
from ncclient import manager
smo = manager.connect(host='192.168.8.27', port="1830", timeout=5, username="netconf", password="netconf!", hostkey_verify=False)
smo.get_config(source="running")
```
**Result**
![Screenshot 2024-10-22 at 9.02.58 PM](https://hackmd.io/_uploads/HJZj6WIxJg.png)

:::spoiler Detail
```javascript=
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" 
          message-id="urn:uuid:2856566e-8ebf-4413-890a-f2b41618e0b3">
    <data>
        <ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
            <id>ManagedElement=gNB-Eurecom-5GNRBox-00001</id>
            <attributes>
                <priorityLabel>1</priorityLabel>
            </attributes>
            
            <AlarmList>
                <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,AlarmList=1</id>
                <attributes>
                    <alarmRecords>
                        <alarmId>ManagedElement=gNB-Eurecom-5GNRBox-00001-internalConnectionLoss</alarmId>
                        <perceivedSeverity>CLEARED</perceivedSeverity>
                    </alarmRecords>
                    <alarmRecords>
                        <alarmId>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0,NRCellDU=0-loadDownlinkExceededWarning</alarmId>
                        <perceivedSeverity>CLEARED</perceivedSeverity>
                    </alarmRecords>
                </attributes>
            </AlarmList>

            <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
                <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0</id>
                <attributes>
                    <priorityLabel>1</priorityLabel>
                    <gNBId>3584</gNBId>
                    <gNBIdLength>32</gNBIdLength>
                    <gNBDUId>0</gNBDUId>
                    <gNBDUName>gNB-OAI-DU-0</gNBDUName>
                </attributes>

                <BWP xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-bwp">
                    <id>Downlink</id>
                    <attributes>
                        <priorityLabel>1</priorityLabel>
                        <bwpContext>DL</bwpContext>
                        <isInitialBwp>INITIAL</isInitialBwp>
                        <subCarrierSpacing>30</subCarrierSpacing>
                        <cyclicPrefix>NORMAL</cyclicPrefix>
                        <startRB>0</startRB>
                        <numberOfRBs>106</numberOfRBs>
                    </attributes>
                </BWP>

                <BWP xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-bwp">
                    <id>Uplink</id>
                    <attributes>
                        <priorityLabel>1</priorityLabel>
                        <bwpContext>UL</bwpContext>
                        <isInitialBwp>INITIAL</isInitialBwp>
                        <subCarrierSpacing>30</subCarrierSpacing>
                        <cyclicPrefix>NORMAL</cyclicPrefix>
                        <startRB>0</startRB>
                        <numberOfRBs>106</numberOfRBs>
                    </attributes>
                </BWP>

                <NRCellDU xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrcelldu">
                    <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0,NRCellDu=0</id>
                    <attributes>
                        <priorityLabel>1</priorityLabel>
                        <cellLocalId>0</cellLocalId>
                        <pLMNInfoList>
                            <mcc>001</mcc>
                            <mnc>01</mnc>
                            <sd>ff:ff:ff</sd>
                            <sst>1</sst>
                        </pLMNInfoList>
                        <nPNIdentityList>
                            <idx>0</idx>
                            <plmnid>
                                <mcc>001</mcc>
                                <mnc>01</mnc>
                            </plmnid>
                            <cAGIdList>1</cAGIdList>
                            <nIDList>1</nIDList>
                        </nPNIdentityList>
                        <nRPCI>0</nRPCI>
                        <arfcnDL>640008</arfcnDL>
                        <arfcnUL>640008</arfcnUL>
                        <bSChannelBwDL>40</bSChannelBwDL>
                        <rimRSMonitoringStartTime>2023-06-06T00:00:00+00:00</rimRSMonitoringStartTime>
                        <rimRSMonitoringStopTime>2023-06-06T00:00:00+00:00</rimRSMonitoringStopTime>
                        <rimRSMonitoringWindowDuration>1</rimRSMonitoringWindowDuration>
                        <rimRSMonitoringWindowStartingOffset>1</rimRSMonitoringWindowStartingOffset>
                        <rimRSMonitoringWindowPeriodicity>1</rimRSMonitoringWindowPeriodicity>
                        <rimRSMonitoringOccasionInterval>1</rimRSMonitoringOccasionInterval>
                        <rimRSMonitoringOccasionStartingOffset>1</rimRSMonitoringOccasionStartingOffset>
                        <ssbFrequency>641280</ssbFrequency>
                        <ssbPeriodicity>5</ssbPeriodicity>
                        <ssbSubCarrierSpacing>15</ssbSubCarrierSpacing>
                        <ssbOffset>1</ssbOffset>
                        <ssbDuration>1</ssbDuration>
                        <bSChannelBwUL>40</bSChannelBwUL>
                        <nRSectorCarrierRef>Tuf=Jy,H:u=|</nRSectorCarrierRef>
                        <victimSetRef>Tuf=Jy,H:u=|</victimSetRef>
                        <aggressorSetRef>Tuf=Jy,H:u=|</aggressorSetRef>
                    </attributes>
                </NRCellDU>

                <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
                    <id>ManagedElement=, GNBDUFunction=1775440688</id>
                    <attributes>
                        <priorityLabel>1</priorityLabel>
                        <txDirection>UL</txDirection>
                        <configuredMaxTxPower>50</configuredMaxTxPower>
                        <configuredMaxTxEIRP>100</configuredMaxTxEIRP>
                        <arfcnDL>104</arfcnDL>
                        <arfcnUL>104</arfcnUL>
                        <bSChannelBwDL>60</bSChannelBwDL>
                        <bSChannelBwUL>60</bSChannelBwUL>
                        <sectorEquipmentFunctionRef>Tuf=Jy,H:u=|</sectorEquipmentFunctionRef>
                    </attributes>
                </NRSectorCarrier>
            </GNBDUFunction>
        </ManagedElement>

        <keystore xmlns="urn:ietf:params:xml:ns:yang:ietf-keystore">
            <asymmetric-keys>
                <asymmetric-key>
                    <name>genkey</name>
                    <algorithm>rsa2048</algorithm>
                    <public-key>MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtIYViYkxhrKet9umfD3NlchZ9zG72Gk/9aJTorjChKTfvKPJ4TxK4d0UlyGUfQmiPivSWfymSZBE9Nho1qVmfHI8KeB7U1YFQ2HZlCH7pY5ddRwvEIiwSvzvR/UUA5S8ShWU743vc8rhhlKrIbIQBb9hVRQR+l1qbe+6Trhm+R04DO0lDYgvGCXhSC+4pQCZjbn7c1uSQnjqFGLXXd/uxozPqwgDAT1urLNg8QnJ/Qnl6bmh+m+pSoj/7xXNZLdjbPYmHwlUC+jkVjyYKq2Kp7tVKYftZp+8ZPE3gsE1jd11pDSbNlOhefmgU8w1HXljaqQBnD0Iz/71BN4Xpd5S3wIDAQAB</public-key>
                    <private-key>MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC0hhWJiTGGsp6326Z8Pc2VyFn3MbvYaT/1olOiuMKEpN+8o8nhPErh3RSXIZR9CaI+K9JZ/KZJkET02GjWpWZ8cjwp4HtTVgVDYdmUIfuljl11HC8QiLBK/O9H9RQDlLxKFZTvje9zyuGGUqshshAFv2FVFBH6XWpt77pOuGb5HTgM7SUNiC8YJeFIL7ilAJmNuftzW5JCeOoUYtdd3+7GjM+rCAMBPW6ss2DxCcn9CeXpuaH6b6lKiP/vFc1kt2Ns9iYfCVQL6ORWPJgqrYqnu1Uph+1mn7xk8TeCwTWN3XWkNJs2U6F5+aBTzDUdeWNqpAGcPQjP/vUE3hel3lLfAgMBAAECggEACddrfNQuuoX/XXegjme8zvxysdATdRCxLMjSNW320M4U5879a7A38YigyD0g6rt9OFV27Mf3dXg4135QIi+MiZ6dWrfDEWqASUmgLSa4djCaNg8OHnZ34fFYcXxSFdXkeUd5l0u4xlUaUWfYA4jBEzHOnRmbLKKDcbCJis7GclsxOkoplMU8BEu7Ps2gXkzv1EuUayX3feMZsQS3Ycwh9w/4aAHpSADTlK9xYBmKNWI7EbhlbByKOfkr1T5V0TXNU7L+6D+VQVWYLhUpRZ4D1mvnbSvDca9Ku99cYy/wNJcxB+bxFM/Ni2shs/btBDf8ozuAB+w+ziRsj67ll5ldwQKBgQD5+s+itYJrrF63nvTCRM4Pou+T9Zqtp3jd8DK1k3aPmEWaBNrGPpaLMptejym8Nql4jT9ou0KUGwVQRfUCIp+DP79EqUbmSgRhpEhGYjMUC0+e1hA5r1PmI/38An3ZZ+4JfHYiUIrTyUwCzDGYxALapQ7aqyPNbPzn8EsnbFxUnwKBgQC43w8tFT6J7iFz+TPzveh3ILQFie46A1GR0r0K3HFj0vyG2BtybwbJ3E2HY5CqVvBFhr4XcEKjPw7oV+YSKpU6VqFAjGDWtZdaORngHId+yKwdPI+sWf0Y8BWSIvYIDP7wNvvkGC0E0oJUNgfz6NmaaxOWNeyZtmTwnW6USKEZwQKBgQCm5GY/cQMTs87AtKUgFiOkmNluZOjRyx+MvNJ+G2dqUvUU8OzGsf58DFtidB4fBDd8voB5AZxfmPKhNzNuK4NncuXVh1ZIZV4reiyuoN0NIsgTeUL34DAZVCo7V8aBoTtwpeGQ40jsQFY4/+6U2Tg2lUAniV6rxXnLt8fVGClEbQKBgGiRGpdodcg0nk1nvl/2od+H6utbGhlMOT4fEfhrueM5usZWxCeU7yUMa/nRckk3BY596VV+lOKbT0ZSOXs7BM9LosfM3xVy/xn0RFOEL4uh2+BpmeZlvAf3/Gt9ROZG24hpwU5B8mzQ2RDiwtrOcQ6r1BdZhutmxG9ozNwovJ7BAoGAToNqctephwpoBWbL+HFoqYEBa/wLuv+L7CFVS8WyLUp5hWg+MGf6akmE/pyd7CjNoglskqj11AfR9BD9meGBc8jeMF7xF/0d3WMd4NYN8ntNipgSRza3xJ2VHaWHsgaoWeZ8O/InsvG4Ppkr/XsN8D
```
:::

## 5. O1 adapter ves functin testing

- Netconf Server:
    - "ip": 192.168.8.27:1222
    - "username": netconf
    - "password": netconf!
- FTP Server:
    - "ip": 192.168.8.27:1830
    - "username": netconf
    - "password": netconf!
    - Use this command to login
        - sftp -P 1222 netconf@192.168.8.27

Makesure this part the url is already change to http://192.168.8.6:30417/eventListener/v7
```javascript=
vim o1-adapter/docker/config
```
```javascript=
    "ves": {
        "template": {
            "new-alarm": "/adapter/config/ves-new-alarm.json",
            "clear-alarm": "/adapter/config/ves-clear-alarm.json",
            "pnf-registration": "/adapter/config/ves-pnf-registration.json",
            "file-ready": "/adapter/config/ves-file-ready.json",
            "heartbeat": "/adapter/config/ves-heartbeat.json",
            "pm-data": "/adapter/config/pmData-measData.xml"
        },

        "pnf-registration": true,
        "heartbeat-interval": 30,

        "url": "http://192.168.8.6:30417/eventListener/v7",
        "username": "sample1",
        "password": "sample1",

        "file-expiry": 86400,
        "pm-data-interval": 30
    },
```
Revise other setting, because OAI their source file have some problem.
The reference schema need to follow the onap not the 3gpp provide by OAI.

Documention: https://docs.onap.org/projects/onap-dcaegen2/en/latest/sections/services/ves-http/stnd-defined-validation.html

![Screenshot 2024-10-28 at 2.04.33 PM](https://hackmd.io/_uploads/ry8Ytihe1l.png)

```javascript=
vim ves-file-ready.json
```
Original: https://forge.3gpp.org/rep/sa5/MnS/blob/Rel-18/OpenAPI/TS28532_FileDataReportingMnS.yaml#components/schemas/NotifyFileReady

After revise: https://forge.3gpp.org/rep/sa5/MnS/blob/SA88-Rel16/OpenAPI/PerDataFileReportMnS.yaml

```javascript=
"schemaReference": "https://forge.3gpp.org/rep/sa5/MnS/blob/SA88-Rel16/OpenAPI/PerDataFileReportMnS.yaml",
```

```javascript=
vim ves-new-alarm.json
```

Original: https://forge.3gpp.org/rep/sa5/MnS/blob/Rel-18/OpenAPI/TS28532_FaultMnS.yaml#components/schemas/NotifyNewAlarm

After revise: https://forge.3gpp.org/rep/sa5/MnS/blob/SA88-Rel16/OpenAPI/faultMnS.yaml

```javascript=
"schemaReference": "https://forge.3gpp.org/rep/sa5/MnS/blob/SA88-Rel16/OpenAPI/faultMnS.yaml",
```

:::spoiler ERROR : For more detail you can check here: Invalid input value for %1 %2: %3
:::danger
**ERROR** 

"RequestError":{"ServiceException":{"variables":["attribute","event.stndDefinedFields.schemaReference","Referred external schema not present in schema repository"]}

![Screenshot 2024-10-23 at 3.48.42 PM](https://hackmd.io/_uploads/Hyfqq7Ieyx.png)

For this problem, we can find the information from here.

Documention: https://docs.onap.org/projects/onap-dcaegen2/en/latest/sections/services/ves-http/stnd-defined-validation.html

![Screenshot 2024-10-28 at 2.29.31 PM](https://hackmd.io/_uploads/ByPYkn2l1e.png)

We need to revise the reference schema URL which ONAP can support.

![Screenshot 2024-10-28 at 2.04.33 PM](https://hackmd.io/_uploads/ry8Ytihe1l.png)
:::

After revise the config. We need to revise the port running on docker setting.

```javascript=
cd o1-adapter
vim start-adapter.sh
```
```javascript=
/*Upper part skip*/

if $START_DEV; then
    if $GO; then
        docker exec -it adapter-dev bash
    else
        docker kill adapter-dev
        docker rm adapter-dev

        docker run -d --name adapter-dev --rm -w /adapter/src -v $(pwd)/src:/adapter/src -v $(pwd)/docker/config:/adapter/config -v $(pwd)/docker/scripts/servertest.py:/adapter/scripts/servertest.py -p 1830:830 -p 1222:22 adapter-dev
        if [ ! $? -eq 0 ]; then
            exit 1
        fi
    fi
fi
if $START_ADAPTER; then
    if $GO; then
        docker exec -it adapter-gnb bash
    else
        docker kill adapter-gnb
        docker rm adapter-gnb

        docker run -d --name adapter-gnb -p 1830:830 -p 1222:22 adapter-gnb
        if [ ! $? -eq 0 ]; then
                exit 1
        fi

        docker logs adapter-gnb -f
    fi
fi
```
:::spoiler Original file
```javascript=
#!/bin/bash

# /*
# * Licensed to the OpenAirInterface (OAI) Software Alliance under one or more
# * contributor license agreements.  See the NOTICE file distributed with
# * this work for additional information regarding copyright ownership.
# * The OpenAirInterface Software Alliance licenses this file to You under
# * the OAI Public License, Version 1.1  (the "License"); you may not use this file
# * except in compliance with the License.
# * You may obtain a copy of the License at
# *
# *      http://www.openairinterface.org/?page_id=698
# *
# * Copyright: Fraunhofer Heinrich Hertz Institute
# *
# * Unless required by applicable law or agreed to in writing, software
# * distributed under the License is distributed on an "AS IS" BASIS,
# * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# * See the License for the specific language governing permissions and
# * limitations under the License.
# *-------------------------------------------------------------------------------
# * For more information about the OpenAirInterface (OAI) Software Alliance:
# *      contact@openairinterface.org
# */

START_DEV="false"
START_ADAPTER="false"
GO="false"

until [ -z "$1" ]
do
    case "$1" in 
        --dev)
            START_DEV="true"
            shift
            ;;

        --adapter)
            START_ADAPTER="true"
            shift
            ;;

        --go)
            GO="true"
            shift
            ;;

        *)
            shift
            ;;
    esac
done

if $START_DEV; then
    if $GO; then
        docker exec -it adapter-dev bash
    else
	docker kill adapter-dev
	docker rm adapter-dev

        docker run -d --name adapter-dev --rm -w /adapter/src -v $(pwd)/src:/adapter/src -v $(pwd)/docker/config:/adapter/config -v $(pwd)/docker/scripts/servertest.py:/adapter/scripts/servertest.py -p 1830:830 -p 1222:22 adapter-dev
        if [ ! $? -eq 0 ]; then
            exit 1
        fi
    fi
fi
if $START_ADAPTER; then
    if $GO; then
        docker exec -it adapter-gnb bash
    else
	docker kill adapter-gnb
	docker rm adapter-gnb

        docker run -d --name adapter-gnb -p 11830:830 -p 11221:21 adapter-gnb
        if [ ! $? -eq 0 ]; then
                exit 1
        fi

        docker logs adapter-gnb -f
    fi
fi

```
:::

Revise the IPV4 listening setting.
```javascript=
vim vsftpd.conf
```

```javascript=
listen=YES
```

Rebuild and run.
```javascript=
sudo docker stop adapter-gnb
sudo docker rm adapter-gnb
sudo ./build-adapter.sh --adapter --no-cache
sudo ./start-adapter.sh --adapter
```

**Result:**
- O1 adapter send the PNF registration event to SMO.
![Screenshot 2024-10-28 at 3.29.26 PM](https://hackmd.io/_uploads/B1ZyyThe1l.png)
- O1 adapter automatically hearbeat checking.
![Screenshot 2024-10-28 at 3.27.47 PM](https://hackmd.io/_uploads/BJZkyphg1g.png)
- O1 adapter notify the fileready to SMO.
![Screenshot 2024-10-28 at 3.28.20 PM](https://hackmd.io/_uploads/Byb1ka3xkx.png)
- O1 adapter notify the new alarm.
![Screenshot 2024-11-28 at 8.11.37 PM](https://hackmd.io/_uploads/BJCH0RS7kg.png)
- Checking in FTP server.
![Screenshot 2024-10-28 at 3.32.41 PM](https://hackmd.io/_uploads/HkXcyp3lkx.png)
![Screenshot 2024-10-28 at 3.39.26 PM](https://hackmd.io/_uploads/Hklpyphlye.png)
- Ves data format
![Screenshot 2024-10-28 at 3.40.49 PM](https://hackmd.io/_uploads/HkVzgT3gyx.png)
- pnfRegistration event package
![Screenshot 2024-10-28 at 6.16.57 PM](https://hackmd.io/_uploads/HyE-kfTlJg.png)
- hearbeat event package
![Screenshot 2024-10-29 at 12.25.40 AM](https://hackmd.io/_uploads/rkjMoEplkx.png)
- notify fileready event package
![Screenshot 2024-10-29 at 12.45.38 AM](https://hackmd.io/_uploads/Hy3fxS6ekl.png)

## 6. SMO edit netconf config testing
### 6-1. Connect to OAI O1 adapter netconf server.
```javascript=
python3
from ncclient import manager
smo = manager.connect(host='192.168.8.27', port="1830", timeout=5, username="netconf", password="netconf!", hostkey_verify=False)
```
### 6-2. Get the environment config
```javascript=
smo.get_config(source="running")
```
![Screenshot 2024-11-05 at 5.30.37 AM](https://hackmd.io/_uploads/Bk1Ma3LWkl.png)

Configuration Filter testing.
```javascript=
gconf = '''
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
    <id>ManagedElement=gNB-Eurecom-5GNRBox-00001</id>
    <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
        <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0</id>
        <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
            <id>ManagedElement=, GNBDUFunction=-1167157504</id>
        </NRSectorCarrier>
    </GNBDUFunction>
</ManagedElement>
'''
```
```javascript=
result = smo.get_config(source="running", filter=("subtree", gconf))
print(result)
```
**Result**

![Screenshot 2024-11-05 at 5.33.49 AM](https://hackmd.io/_uploads/H1ApTnIZye.png)

:::spoiler Detail
```javascript=
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="urn:uuid:bdc1641a-89b3-4fc3-aed8-a84c07fd7cc0">
    <data>
        <ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
            <id>ManagedElement=gNB-Eurecom-5GNRBox-00001</id>
            <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
                <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0</id>
                <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
                    <id>ManagedElement=, GNBDUFunction=-1167157504</id>
                    <attributes>
                        <priorityLabel>1</priorityLabel>
                        <txDirection>UL</txDirection>
                        <configuredMaxTxPower>50</configuredMaxTxPower>
                        <configuredMaxTxEIRP>100</configuredMaxTxEIRP>
                        <arfcnDL>104</arfcnDL>
                        <arfcnUL>104</arfcnUL>
                        <bSChannelBwDL>60</bSChannelBwDL>
                        <bSChannelBwUL>60</bSChannelBwUL>
                        <sectorEquipmentFunctionRef>Tuf=Jy,H:u=|</sectorEquipmentFunctionRef>
                    </attributes>
                </NRSectorCarrier>
            </GNBDUFunction>
        </ManagedElement>
    </data>
</rpc-reply>
```
:::
### 6-3. Revise the config to Netconf Server

**Revising bSChannelBwDL and bSChannelBwUL 60 to 20 by using Netconf**
```javascript=
conf = '''
<config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
    <id>ManagedElement=gNB-Eurecom-5GNRBox-00001</id>
    <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
        <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0</id>
        <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
            <id>ManagedElement=, GNBDUFunction=-622106880</id>
            <attributes>
                <bSChannelBwDL>20</bSChannelBwDL>
                <bSChannelBwUL>20</bSChannelBwUL>
            </attributes>
        </NRSectorCarrier>
    </GNBDUFunction>
</ManagedElement>
</config>
'''
```
```javascript=
smo.edit_config(target="running", config=conf)
```
SMO use ncclient netconf to send the new config to OAI O1 gNB.

![Screenshot 2024-11-06 at 4.36.10 PM](https://hackmd.io/_uploads/S1Ehqod-1e.png)

**Response:** <rpc-reply><ok/></rpc-reply>

For OAI O1 adapter part will automatically stop the OAI O1 gNB(softmodem) to send the new config command and then automatically restart the OAI O1 gNB.

![Screenshot 2024-11-06 at 4.40.00 PM](https://hackmd.io/_uploads/BJgaho_W1l.png)
![Screenshot 2024-11-06 at 4.41.13 PM](https://hackmd.io/_uploads/Sk7T3suZ1g.png)

After OAI O1 gNB(softmodem) stop and restart, it will Killing gNB processing ru_thread and using the new config parameter.

![Screenshot 2024-11-06 at 5.10.41 PM](https://hackmd.io/_uploads/SkXoMn_-kl.png)

**Revising bSChannelBwDL and bSChannelBwUL 20 to 100 by using Netconf**

```javascript=
conf = '''
<config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
    <id>ManagedElement=gNB-Eurecom-5GNRBox-00001</id>
    <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
        <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0</id>
        <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
            <id>ManagedElement=, GNBDUFunction=-264190208</id>
            <attributes>
                <bSChannelBwDL>100</bSChannelBwDL>
                <bSChannelBwUL>100</bSChannelBwUL>
            </attributes>
        </NRSectorCarrier>
    </GNBDUFunction>
</ManagedElement>
</config>
'''
```
```javascript=
smo.edit_config(target="running", config=conf)
```

![Screenshot 2024-11-06 at 6.37.51 PM](https://hackmd.io/_uploads/ryxMPpd-Je.png)

**Response:** <rpc-reply><ok/></rpc-reply>

![Screenshot 2024-11-06 at 6.38.58 PM](https://hackmd.io/_uploads/BJrLwpObJe.png)

Because OAI O1 adapter their netconf_data.c only support revise the bSChannelBwDL and bSChannelBwUL. So please follow the [6-4](#6-4-Extension-for-configuration-modifications-to-the-Netconf-Server) to extend the configuration ability.

:::spoiler The reason can't be revise to netconf server 
:::danger

**ERROR**

**Revising arfcnDL and arfcnUL 104 to 50 by using Netconf**

```javascript=
conf = '''
<config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
    <id>ManagedElement=gNB-Eurecom-5GNRBox-00001</id>
    <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
        <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0</id>
        <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
            <id>ManagedElement=, GNBDUFunction=-264190208</id>
            <attributes>
                <arfcnDL>50</arfcnDL>
            </attributes>
        </NRSectorCarrier>
    </GNBDUFunction>
</ManagedElement>
</config>
'''
```
```javascript=
smo.edit_config(target="running", config=conf)
```
![Screenshot 2024-11-07 at 5.00.41 PM](https://hackmd.io/_uploads/Hkr0bWqWJl.png)

**In OAI O1 adapter**

![Screenshot 2024-11-05 at 7.30.04 AM](https://hackmd.io/_uploads/B118tCUbkg.png)

O1 adapter can't accept the recieve config, the error log is **invalid edit data**.

**Revising configuredMaxTxPower 50 to 10 by using Netconf**
```javascript=
conf = '''
<config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
    <id>ManagedElement=gNB-Eurecom-5GNRBox-00001</id>
    <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
        <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0</id>
        <NRSectorCarrier xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrnetwork-nrsectorcarrier">
            <id>ManagedElement=, GNBDUFunction=1140143872</id>
            <attributes>
                <configuredMaxTxPower>10</configuredMaxTxPower>
            </attributes>
        </NRSectorCarrier>
    </GNBDUFunction>
</ManagedElement>
</config>
'''
```
```javascript=
smo.edit_config(target="running", config=conf)
```

![Screenshot 2024-11-05 at 5.59.04 AM](https://hackmd.io/_uploads/BkhnXaIWye.png)

The result show that **ncclient.operations.rpc.RPCError: error: Validation failed**, this part stll find the problem reason.

**Reason**
Because OAI O1 adapter their netconf_data.c only support revise the bSChannelBwDL and bSChannelBwUL.

In netconf_data.c line .2545 ~ .2587
![Screenshot 2024-11-07 at 3.52.04 PM](https://hackmd.io/_uploads/HyW0Gb5-yg.png)
:::

### 6-4. Extension for configuration modifications to the Netconf Server
#### 6-4-1. System architecture with file location

![gggg](https://hackmd.io/_uploads/HySU-4oZke.png)

:::success
The red color file is which I rivise.
:::
The following file need to be revised.
- telnet.h
- telnet.c
- netconf_data.c

#### 6-4-2. Revise telnet.h

```javascript=
cd o1-adapter/src/telnet/
vim telnet.h
```
```javascript=
int telnet_change_config(int ssbFrequency, int arfcnDL, int bSChannelBwDL, int numberOfRBs, int startRB);
```

![Screenshot 2024-11-08 at 1.19.10 PM](https://hackmd.io/_uploads/H14v17o-1x.png)


#### 6-4-3. Revise telnet.c
```javascript=
cd o1-adapter/src/telnet/
vim telnet.c
```
```javascript=
int telnet_change_config(int ssbFrequency, int arfcnDL, int bSChannelBwDL, int numberOfRBs, int startRB) {
        char *response = 0;
        int rc = 0;
        char buffer[512];

        telnet_lock();

        const char *tokens[] = {"softmodem_gnb> ", 0};
        rc = telnet_write("o1 stop_modem\n", 10, tokens, &response);
        if(rc != 0) {
                log_error("telnet_write failed");
                goto failed;
        }

        if(strstr(response, "FAIL")) {
                log_error("telnet o1 stop_modem failed");
                goto failed;
        }
        free(response);
        response = 0;

        sleep(1);

        sprintf(buffer, "o1 config nrcelldu3gpp:ssbFrequency %d nrcelldu3gpp:arfcnDL %d nrcelldu3gpp:bSChannelBwDL %d bwp3gpp:numberOfRBs %d bwp3gpp:startRB %d\n", ssbFrequency, arfcnDL, bSChannelBwDL, numberOfRBs, startRB);
        rc = telnet_write(buffer, 15, tokens, &response);
        if(rc != 0) {
                log_error("telnet_write failed");
                goto failed;
        }

        if(strstr(response, "FAIL")) {
                log_error("telnet o1 config failed");
                goto failed;
        }
        free(response);
        response = 0;

        sleep(1);

        rc = telnet_write("o1 start_modem\n", 10, tokens, &response);
        if(rc != 0) {
                log_error("telnet_write failed");
                goto failed;
        }

        if(strstr(response, "FAIL")) {
                log_error("telnet o1 start_modem failed");
                goto failed;
        }
        free(response);
        response = 0;

        telnet_unlock();

        return 0;

failed:
        telnet_unlock();
        free(response);
        response = 0;

        return 1;
}
```

![Screenshot 2024-11-08 at 1.23.09 PM](https://hackmd.io/_uploads/ryYul7sbye.png)


#### 6-4-4. Revise netconf_data.c

```javascript=
cd o1-adapter/src/netconf/
vim netconf_data.c
```
```javascript=64
int arfcnDL = -1;
int arfcnUL = -1;
int bSChannelBwDL = -1;
int bSChannelBwUL = -1;
int ssbFrequency = -1;
int numberOfRBs = -1;
int startRB = -1;
```
```javascript=166
int netconf_data_update_full(const oai_data_t *oai) {
    startRB = oai->bwp[0].startRB; //New
    numberOfRBs = oai->bwp[0].numberOfRBs; //New
    arfcnDL = oai->nrcelldu.arfcnDL; //New
    arfcnUL = oai->nrcelldu.arfcnUL; //New
    bSChannelBwDL = oai->nrcelldu.bSChannelBwDL; //New
    bSChannelBwUL = oai->nrcelldu.bSChannelBwUL; //New
    ssbFrequency = oai->nrcelldu.ssbFrequency; //New
}
```
```javascript=1378
int netconf_data_update_bwp_dl(const oai_data_t *oai) {
    startRB = oai->bwp[0].startRB; //New
    numberOfRBs = oai->bwp[0].numberOfRBs; //New
}
```
```javascript=1748
int netconf_data_update_nrcelldu(const oai_data_t *oai) {
    arfcnDL = oai->nrcelldu.arfcnDL; //New
    arfcnUL = oai->nrcelldu.arfcnUL; //New
    bSChannelBwDL = oai->nrcelldu.bSChannelBwDL; //New
    bSChannelBwUL = oai->nrcelldu.bSChannelBwUL; //New
    ssbFrequency = oai->nrcelldu.ssbFrequency; //New
}
```
```javascript=2504
static int netconf_data_edit_callback(sr_session_ctx_t *session, uint32_t sub_id, const char *module_name, const char *xpath_running, sr_event_t event, uint32_t request_id, void *private_data) {
    (void)sub_id;
    (void)request_id;
    (void)private_data;
    
    int rc = SR_ERR_OK;

    sr_change_iter_t *it = 0;
    sr_change_oper_t oper;
    sr_val_t *old_value = 0;
    sr_val_t *new_value = 0;

    char *change_path = 0;

    if(xpath_running) {
        asprintf(&change_path, "%s//.", xpath_running);
    }
    else {
        asprintf(&change_path, "/%s:*//.", module_name);
    }

        
    if(event == SR_EV_CHANGE) {
        rc = sr_get_changes_iter(session, change_path, &it);
        if (rc != SR_ERR_OK) {
            log_error("sr_get_changes_iter() failed");
            goto failed;
        }

        int invalidEdit = 0;
        char *invalidEditReason = 0; 
        int New_bSChannelBwDL = -1;
        int New_bSChannelBwUL = -1;
        int New_arfcnDL = -1;
        int New_arfcnUL = -1;
        int New_ssbFrequency = -1;
        int New_numberOfRBs = -1;
        int New_startRB = -1;

        while ((rc = sr_get_change_next(session, it, &oper, &old_value, &new_value)) == SR_ERR_OK) {
            if(oper != SR_OP_MODIFIED) {
                invalidEdit = 1;
                invalidEditReason = strdup("invalid operation (only MODIFY enabled)");
                goto checkInvalidEdit;
            }

            // here we can develop more complete xpath instead of "bSChannelBwDL" if needed
            if(strstr(new_value->xpath, "bSChannelBwDL")) {
               New_bSChannelBwDL = new_value->data.uint32_val;
            }
            else if(strstr(new_value->xpath, "bSChannelBwUL")) {
               New_bSChannelBwUL = new_value->data.uint32_val;
            }
            else if(strstr(new_value->xpath, "arfcnDL")) {
               New_arfcnDL = new_value->data.uint32_val;
            }
            else if(strstr(new_value->xpath, "arfcnUL")) {
               New_arfcnUL = new_value->data.uint32_val;
            }
            else if(strstr(new_value->xpath, "ssbFrequency")) {
               New_ssbFrequency = new_value->data.uint32_val;
            }
            else if(strstr(new_value->xpath, "numberOfRBs")) {
               New_numberOfRBs = new_value->data.uint32_val;
            }
            else if(strstr(new_value->xpath, "startRB")) {
               New_startRB = new_value->data.uint32_val;
            }
            else {
                invalidEdit = 1;
                invalidEditReason = strdup(new_value->xpath);
            } 

            sr_free_val(old_value);
            old_value = 0;
            sr_free_val(new_value);
            new_value = 0;

checkInvalidEdit:
            if(invalidEdit) {
                break;
            }
        }

        if(invalidEdit) {
            log_error("invalid edit data detected: %s", invalidEditReason);
            free(invalidEditReason);
            goto failed_validation;
        }

        /*if((bSChannelBwDL != -1) || (bSChannelBwUL != -1)) {
            if(bSChannelBwDL != bSChannelBwUL) {
                log_error("bSChannelBwDL (%d) != bSChannelBwUL (%d)", bSChannelBwDL, bSChannelBwUL);
                goto failed_validation;
            }
            else {
                // send command
                int rc = telnet_change_bandwidth(bSChannelBwDL);
                if(rc != 0) {
                    log_error("telnet_change_bandwidth failed");
                    goto failed_validation;
                }
            }
        }*/

        if(New_bSChannelBwDL == -1){
            if(bSChannelBwDL == 20){
                New_bSChannelBwDL = 51;
            }else if(bSChannelBwDL == 40){
                New_bSChannelBwDL = 106;
            }else if(bSChannelBwDL == 60){
                New_bSChannelBwDL = 162;
            }else if(bSChannelBwDL == 100){
                New_bSChannelBwDL = 273;
            }
        }

        if(New_bSChannelBwUL == -1){
            if(bSChannelBwUL == 20){
                New_bSChannelBwUL = 51;
            }else if(bSChannelBwUL == 40){
                New_bSChannelBwUL = 106;
            }else if(bSChannelBwUL == 60){
                New_bSChannelBwUL = 162;
            }else if(bSChannelBwUL == 100){
                New_bSChannelBwUL = 273;
            }
        }

        if(New_arfcnDL == -1) New_arfcnDL = arfcnDL;
        if(New_arfcnUL == -1) New_arfcnUL = arfcnUL;
        if(New_ssbFrequency == -1) New_ssbFrequency = ssbFrequency;
        if(New_numberOfRBs == -1) New_numberOfRBs = numberOfRBs;
        if(New_startRB == -1) New_startRB = startRB;

        if(New_arfcnUL != New_arfcnDL) {
            log_error("arfcnUL (%d) != arfcnDL (%d)", New_arfcnUL, New_arfcnDL);
            goto failed_validation;
        }
        else if(New_bSChannelBwDL != New_bSChannelBwUL) {
            log_error("bSChannelBwDL (%d) != bSChannelBwUL (%d)", New_bSChannelBwDL, New_bSChannelBwUL);
            goto failed_validation;
        }else {
            //send command
            int rc = telnet_change_config(New_ssbFrequency, New_arfcnDL, New_bSChannelBwDL, New_numberOfRBs, New_startRB);
            if(rc != 0) {
                log_error("Invalid parameters: arfcnUL=%d, arfcnDL=%d, ssbFrequency=%d, numberOfRBs=%d, startRB=%d, bSChannelBwDL=%d, bSChannelBwUL=%d",
                      New_arfcnUL, New_arfcnDL, New_ssbFrequency, New_numberOfRBs, New_startRB, New_bSChannelBwDL, New_bSChannelBwUL);
                log_error("telnet_change_config failed");
                goto failed_validation;
            }
        }

        sr_free_change_iter(it);
        it = 0;
    }

    free(change_path);
    if(it) {
        sr_free_change_iter(it);
    }
    if(old_value) {
        sr_free_val(old_value);
    }
    if(new_value) {
        sr_free_val(new_value);
    }

    return SR_ERR_OK;

failed:
    free(change_path);
    if(it) {
        sr_free_change_iter(it);
    }
    if(old_value) {
        sr_free_val(old_value);
    }
    if(new_value) {
        sr_free_val(new_value);
    }

    
    return SR_ERR_INTERNAL;

failed_validation:
    free(change_path);
    if(it) {
        sr_free_change_iter(it);
    }
    if(old_value) {
        sr_free_val(old_value);
    }
    if(new_value) {
        sr_free_val(new_value);
    }
    
    return SR_ERR_VALIDATION_FAILED;
}
```
#### 6-4-5. Result

Using this netconf config to update the following parameter.
- ssbFrequency
- arfcnDL
- arfcnUL
- bSChannelBwDL
- bSChannelBwUL
- startRB
- numberOfRBs

```javascript=
conf = '''
<config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
   <ManagedElement xmlns="urn:3gpp:sa5:_3gpp-common-managed-element">
       <id>ManagedElement=gNB-Eurecom-5GNRBox-00001</id>
       <GNBDUFunction xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-gnbdufunction">
           <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0</id>
           <BWP xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-bwp">
               <id>Downlink</id>
               <attributes>
                   <startRB>1</startRB>
                   <numberOfRBs>273</numberOfRBs>
               </attributes>
           </BWP>
           <NRCellDU xmlns="urn:3gpp:sa5:_3gpp-nr-nrm-nrcelldu">
               <id>ManagedElement=gNB-Eurecom-5GNRBox-00001,GNBDUFunction=0,NRCellDu=0</id>
               <attributes>
                   <arfcnDL>640020</arfcnDL>
                   <arfcnUL>640020</arfcnUL>
                   <ssbFrequency>641810</ssbFrequency>
                   <bSChannelBwDL>273</bSChannelBwDL>
                   <bSChannelBwUL>273</bSChannelBwUL>
               </attributes>
           </NRCellDU>
       </GNBDUFunction>
   </ManagedElement>
</config>
'''
```
```javascript=
smo.edit_config(target="running", config=conf)
```
![Screenshot 2024-11-11 at 9.59.52 PM](https://hackmd.io/_uploads/HkFJCKkzkg.png)

Check O1 stats.
```javascript=
echo o1 stats | nc -N 127.0.0.1 9090 | awk '/^{$/, /^}$/' | jq .
```

![Screenshot 2024-11-11 at 10.05.31 PM](https://hackmd.io/_uploads/Syj4kq1GJg.png)

## 7. ODLUX UI interface connected to O1 adapter testing.
SMO ODLUX UI interface website link : http://192.168.8.6:30205/odlux/index.html#/login

- user: admin
- password: Kp8bJ4SXszM0WXlhak3eHlcse2gAw84vaoGGmJvUy2U

![Screenshot 2024-11-14 at 1.17.08 PM](https://hackmd.io/_uploads/S1uJ_-mzJg.png)

Add the node information.
![Screenshot 2024-11-14 at 2.02.55 PM](https://hackmd.io/_uploads/ByZPXzQfyl.png)

Click mount to connect to the node.
![Screenshot 2024-11-14 at 2.06.56 PM](https://hackmd.io/_uploads/Hyk6XfXG1g.png)

:::danger
**ERROR**
Can't connected to the OAI O1 adapter, it stll connecting.
![Screenshot 2024-11-14 at 2.17.55 PM](https://hackmd.io/_uploads/H1ytIfXf1x.png)

:::
## 8. Demo video

Demo video Link: https://www.youtube.com/watch?v=mzHivui9A9w&t=5s