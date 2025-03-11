# <center>[J Release] SMO Topology & Inventory (TEIV)</center>

<center>Introduction of J Release SMO</center>

###### tags: `SMO`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Jan2 , 2024 3:38 PM][color=#38c16a]

:::info
**Introduction：**

Brifly introduce what is SMO TEIV.

**Goal：**
- [x] [J Release] Study SMO Install Topology & Inventory (TEIV)

**Reference：**
- [Release J - Run in Kubernetes](https://wiki.o-ran-sc.org/display/SMO/Release+J+-+Run+in+Kubernetes)
- [[J Release] SMO Topology & Inventory (TEIV) Installation Guide](https://hackmd.io/@Jerry0714/HkTXfrUdC)
- [[J Release] SMO Topology & Inventory (TEIV) User Guide](https://hackmd.io/@Jerry0714/ByElUtroA)
- [[J Release] SMO Topology & Inventory (TEIV) Developer Guide](https://docs.o-ran-sc.org/projects/o-ran-sc-smo-teiv/en/latest/developer-guide.html#)
- [Gerrit smo/teiv.git](https://gerrit.o-ran-sc.org/r/gitweb?p=smo/teiv.git;a=tree;h=ea1ce6271dfe7d94b004c09d89254c4924c937fc;hb=ea1ce6271dfe7d94b004c09d89254c4924c937fc)

:::
**<center>Table of Content</center>**
[TOC]

## J Release

![Screenshot 2024-07-08 at 5.54.04 PM](https://hackmd.io/_uploads/H1FyOEYDR.png)

## History
![Screenshot 2024-07-08 at 5.53.54 PM](https://hackmd.io/_uploads/B1DWdEYvA.png)

## 1. Introduction

Topology Exposure & Inventory is a tool for managing telecommunications network structures. It describes various equipment in the network and their relationships in a universal way, independent of specific vendors. This system automatically updates network changes and supports different types of network structures. It treats network equipment as manageable objects and groups them according to their functions. The purpose of this approach is to make it easier for users to understand the network's status and to query specific network information as needed. Overall, this system helps people better understand and manage complex telecommunications networks.

**Entities && Attributies**

![Screenshot 2024-07-08 at 5.18.37 PM](https://hackmd.io/_uploads/BJNZk4tPR.png)

**Topology & Inventory models**

The Topology & Inventory objects are managed and standardized using YANG models.

![Screenshot 2024-07-08 at 5.22.21 PM](https://hackmd.io/_uploads/rkRAJEYD0.png)

## 2. Query API examples

Topology & Inventory provides APIs to enable users query module data that can be used to understand the existing topology and inventory model.

**Request to fetch a list of all modules:**

```javascript=
GET https://<host>/topology-inventory/<API_VERSION>/schemas
```

**Request to fetch a list of all modules related to the RAN domain:**

```javascript=
GET https://<host>/topology-inventory/<API_VERSION>/schemas?domain=ran
```

**Request to fetch the module data for the o-ran-smo-teiv-ran module:**

```javascript=
GET https://<host>/topology-inventory/<API_VERSION>/schemas/o-ran-smo-teiv-ran/content
```

**Get all entities in the RAN domain:**

```javascript=
GET https://<host>/topology-inventory/<API_VERSION>/domains/RAN/entities
```

**Get all entity types in the RAN domain:**

```javascript=
GET https://<host>/topology-inventory/<API_VERSION>/domains/RAN/entity-types
```

**Get all relationship types in the RAN domain:**

```javascript=
GET https://<host>/topology-inventory/<API_VERSION>/domains/RAN/relationship-types
```

## 3.  Topology & Inventory Data Models

### 3-1. Common YANG extensions

:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-common-yang-extensions {

  yang-version 1.1;
  namespace "urn:o-ran:smo-teiv-common-yang-extensions";
  prefix or-teiv-yext;

  organization "ORAN";
  contact "The Authors";
  description
  "Topology and Inventory YANG extensions model

  This model contains extensions to the YANG language that topology and
  inventory models will use to define and annotate types and relationships.

  Copyright (C) 2024 Ericsson
  Modifications Copyright (C) 2024 OpenInfra Foundation Europe

  Licensed under the Apache License, Version 2.0 (the \"License\");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an \"AS IS\" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.

  SPDX-License-Identifier: Apache-2.0";

    revision "2024-05-24" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    extension biDirectionalTopologyRelationship {
        description
            "Defines a bi-directional relationship in the topology.

            A bi-directional-association (BDA) is a relationship comprising of
            an A-side and a B-side. The A-side is considered the originating
            side of the relationship; the B-side is considered the terminating
            side of the relationship. The order of A-side and B-side is of
            importance and MUST NOT be changed once defined.

            Both A-side and B-side are defined on a type, and are given a role.
            A type may have multiple originating and/or terminating sides of a
            relationship, all distinguished by role name.

            The statement MUST only be a substatement of the 'module' statement.
            Multiple 'bi-directional-topology-relationship' statements are
            allowed per parent statement.

            Substatements to the 'bi-directional-topology-relationship' define
            the A-side and the B-side, respectively, and optionally properties
            of the relationship. Data nodes of types 'leaf' and 'leaf-list' are
            used for this purpose. One of the data nodes MUST be annotated with
            the 'a-side' extension; another data node MUST be annotated with the
            'b-side' extension. Other data nodes define properties of the
            relationship.

            The argument is the name of the relationship. The relationship name
            is scoped to the namespace of the declaring module and MUST be
            unique within the scope.";

        argument relationshipName;
    }

    extension aSide {
        description
            "Defines the A-side of a relationship.

            The statement MUST only be a substatement of a 'leaf' or 'leaf-list'
            statement, which itself must be a substatement of the
            'uni-directional-topology-relationship' statement.

            The data type of the parent 'leaf' or 'leaf-list' MUST be
            'instance-identifier'. Constraints MAY be used as part of the parent
            'leaf' or 'leaf-list' to enforce cardinality.

            The identifier of the parent 'leaf' or 'leaf-list' is used as name
            of the role of the A-side of the relationship. The name of the role
            is scoped to the type on which the A-side is defined and MUST be
            unique within the scope.

            While the parent 'leaf' or 'leaf-list' does not result in a property
            of the relationship, it is RECOMMENDED to avoid using the name of an
            existing type property as role name to avoid potential ambiguities
            between properties of a type, and roles of a relationship on the
            type.

            The argument is the name of the type on which the A-side resides.
            If the type is declared in another module, the type must be
            prefixed, and a corresponding 'import' statement be used to declare
            the prefix.";

        argument aSideType;
    }

    extension bSide {
        description
            "Defines the B-side of a relationship.

            The statement MUST only be a substatement of a 'leaf' or 'leaf-list'
            statement, which itself must be a substatement of the
            'uni-directional-topology-relationship' statement.

            The data type of the parent 'leaf' or 'leaf-list' MUST be
            'instance-identifier'. Constraints MAY be used as part of the parent
            'leaf' or 'leaf-list' to enforce cardinality.

            The identifier of the parent 'leaf' or 'leaf-list' is used as name
            of the role of the B-side of the relationship. The name of the role
            is scoped to the type on which the B-side is defined and MUST be
            unique within the scope.

            While the parent 'leaf' or 'leaf-list' does not result in a property
            of the relationship, it is RECOMMENDED to avoid using the name of an
            existing type property as role name to avoid potential ambiguities
            between properties of a type, and roles of a relationship on the
            type.

            The argument is the name of the type on which the B-side resides.
            If the type is declared in another module, the type must be
            prefixed, and a corresponding 'import' statement be used to declare
            the prefix.";

        argument bSideType;
    }

    extension domain {
        description "Keyword used to carry domain information.";
        argument domainName;
    }

    extension label {
        description
            "The label can be used to give modules and submodules a semantic
            version, in addition to their revision.

            The format of the label is 'x.y.z' - expressed as pattern, it is
            [0-9]+\\.[0-9]+\\.[0-9]+

            The statement MUST only be a substatement of the revision statement.
            Zero or one revision label statements per parent statement are
            allowed.

            Revision labels MUST be unique amongst all revisions of a module or
            submodule.";

        argument semversion;
    }
}
```
:::

### 3-2. Common YANG types

![Screenshot 2024-07-08 at 5.42.43 PM](https://hackmd.io/_uploads/B14jEVFvA.png)

:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-common-yang-types {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-common-yang-types";
    prefix or-teiv-types;

    import o-ran-smo-teiv-common-yang-extensions { prefix or-teiv-yext; }

    import _3gpp-common-yang-types { prefix types3gpp; }

  organization "ORAN";
  contact "The Authors";
  description
  "Topology and Inventory common types model

  This model contains re-usable data types that topology and inventory models
  will frequently use as part of types and relationships.

  Copyright (C) 2024 Ericsson
  Modifications Copyright (C) 2024 OpenInfra Foundation Europe

  Licensed under the Apache License, Version 2.0 (the \"License\");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an \"AS IS\" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.

  SPDX-License-Identifier: Apache-2.0";

    revision "2024-05-24" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    grouping Top_Grp_Type {
        description "Grouping containing the key attribute common to all types.
            All types MUST use this grouping.";

        leaf id {
            type string;
            description "Unique identifier of topology entities. Represents the
                Entity Instance Identifier.";
        }
    }

    container decorators {
        description
            "This container serves as extension point for applications wishing
            to define their own decorators. This is done via augmentations. They
            can only be defined in name value pair.

            This is a consumer data and can be attached to Topology Entity or
            Topology Relation instance, outside of the declared Topology Entity
            or Topology Relationship's attributes. This cannot be instantiated,
            and it MUST NOT be augmented or deviated in any way, unless stated
            otherwise.";
    }

    leaf-list classifiers {
        description
            "Consumer defined tags to topology entities and relationships.

            This is a consumer data and can be attached to Topology Entity or
            Topology Relation instance, outside of the declared Topology Entity
            or Topology Relationship's attributes. This cannot be instantiated,
            and it MUST NOT be augmented or deviated in any way, unless stated
            otherwise.";

        type identityref { base classifier; }
    }

    leaf-list sourceIds {
        description
            "An ordered list of identities that represent the set of native
            source identifiers for participating entities.

            This is a consumer data and can be attached to Topology Entity or
            Topology Relation instance, outside of the declared Topology Entity
            or Topology Relationship's attributes. This cannot be instantiated,
            and it MUST NOT be augmented or deviated in any way, unless stated
            otherwise.";

        type string;
        ordered-by user;
    }

    container metadata {
        description
            "This container serves as extension point to define metadata. They
            can only be defined in name value pair.

            This is a consumer data and can be attached to Topology Entity or
            Topology Relation instance, outside of the declared Topology Entity
            or Topology Relationship's attributes. This cannot be instantiated,
            and it MUST NOT be augmented or deviated in any way, unless stated
            otherwise.";
    }

    identity classifier{
        description "The classifier is used as a base to provide all classifiers
            with identity. ";
    }
}
```
:::
### 3-3. Equipment

![Screenshot 2024-07-08 at 5.43.44 PM](https://hackmd.io/_uploads/SJgyrVYw0.png)

:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-equipment {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-equipment";
    prefix or-teiv-equip;

    import o-ran-smo-teiv-common-yang-types {prefix or-teiv-types; }

    import o-ran-smo-teiv-common-yang-extensions {prefix or-teiv-yext; }

    import ietf-geo-location {
        prefix geo;
        reference "RFC 9179: A YANG Grouping for Geographic Locations";
    }

    organization "ORAN";
    contact "The Authors";
    description
    "RAN Equipment topology model.

    This model contains the topology entities and relations in the
    RAN Equipment domain, which is modelled to understand the physical
    location of equipment such as antennas associated with a cell/carrier
    and their relevant properties e.g. tilt, max power etc.

    Copyright (C) 2024 Ericsson
    Modifications Copyright (C) 2024 OpenInfra Foundation Europe

    Licensed under the Apache License, Version 2.0 (the \"License\");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an \"AS IS\" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and";

    revision "2024-05-24" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    or-teiv-yext:domain EQUIPMENT;

    list AntennaModule {
        description "An Antenna Module represents the physical aspect of an
            antenna.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf antennaModelNumber {
                description "Vendor-specific antenna model identifier. This
                    attribute is part of AISG v3 ADB Standard and has no
                    operational impact.";
                type string;
            }

            leaf mechanicalAntennaBearing {
                description "Antenna bearing on antenna subunit where antenna
                    unit is installed.";
                type int32;
            }

            leaf mechanicalAntennaTilt {
                description "The fixed antenna tilt of the installation, defined
                    as the inclination of the antenna element respect to the
                    vertical plane. It is a signed value. Positive indicates
                    downtilt, and negative indicates uptilt.";
                type int32;
            }

            leaf positionWithinSector {
                description "Antenna unit position within sector. This attribute
                    is part of AISG v3 ADB Standard and has no operational
                    impact.";
                type string;
            }

            leaf totalTilt {
                description "Total antenna elevation including the installed
                    tilt and the tilt applied by the Remote Electrical
                    Tilt (RET).";
                type int32;
            }

            leaf electricalAntennaTilt {
                description "Electrically-controlled tilt of main beam maximum
                    with respect to direction orthogonal to antenna element
                    axis (see 3GPP TS 25.466). Value is signed; tilt down is
                    positive, tilt up is negative.";
                type int32;
            }

            leaf-list antennaBeamWidth {
                description "The angular span of the main lobe of the antenna
                    radiation pattern in the horizontal plane. Measured in
                    degrees.";
                type uint32;
            }

            uses geo:geo-location;
        }
    }

    list Site {
        description "A site is a physical location where an equipment can be
            installed.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf name {
                description "Name of Site";
                type string;
            }

            uses geo:geo-location;

        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship ANTENNAMODULE_INSTALLED_AT_SITE { // 0..n to 0..1

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf installed-at-site {
            description "Antenna Module installed at Site.";
            or-teiv-yext:aSide AntennaModule;
            type instance-identifier;
        }

        leaf-list installed-antennaModule {
            description "Site where Antenna Module is installed.";
            or-teiv-yext:bSide Site;
            type instance-identifier;
        }
    }
}
```
:::
### 3-4. RAN

![ran](https://hackmd.io/_uploads/Bk5GHNKwR.svg)
![ran-relationships](https://hackmd.io/_uploads/SkX1qT_iA.svg)

:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-ran {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-ran";
    prefix or-teiv-ran;

    import o-ran-smo-teiv-common-yang-types {prefix or-teiv-types; }

    import o-ran-smo-teiv-common-yang-extensions {prefix or-teiv-yext; }

    import _3gpp-common-yang-types { prefix types3gpp; }

    import ietf-geo-location {
        prefix geo;
        reference "RFC 9179: A YANG Grouping for Geographic Locations";
    }

    organization "Ericsson AB";
    contact "Ericsson first line support via email";
    description
    "RAN Logical topology model.

    Copyright (c) 2023 Ericsson AB. All rights reserved.

    This model contains the topology entities and relations in the
    RAN Logical domain, which represents the functional capability
    of the deployed RAN that are relevant to rApps use cases.";

    revision "2024-05-02" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    or-teiv-yext:domain RAN;

    list GNBDUFunction {
        description "gNodeB Distributed Unit (gNB-DU).

                    A gNB may consist of a gNB-Centralized Unit
                    (gNB-CU) and a gNB-DU. The CU processes non-real
                    time protocols and services, and the DU processes
                    PHY level protocol and real time services. The
                    gNB-CU and the gNB-DU units are connected via
                    F1 logical interface.

                    The following is true for a gNB-DU:
                    Is connected to the gNB-CU-CP through the F1-C
                    interface.Is connected to the gNB-CU-UP through
                    the F1-U interface. One gNB-DU is connected to only
                    one gNB-CU-CP. One gNB-DU can be connected to
                    multiple gNB-CU-UPs under the control of the same
                    gNB-CU-CP.
                    Note: A gNB may consist of a gNB-CU-CP, multiple
                    gNB-CU-UPs and multiple gNB-DUs. gNB-DU is a concrete
                    class that extends the NG-RAN node object. In Topology, you
                    can create, read, update, and delete the gNB-DU object.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of the GNBDUFunction MO. It contains
                            the full path from the Subnetwork to the
                            GNBDUFunction.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

            container dUpLMNId {
                description "PLMN identifier used as part of PM Events data";
                uses types3gpp:PLMNId;
            }

            leaf gNBDUId {
                description "Unique identifier for the DU within a gNodeB";
                type uint32;
            }

            leaf gNBId {
                description "Identity of gNodeB within a PLMN";
                type uint32;
            }

            leaf gNBIdLength {
                description "Length of gNBId bit string representation";
                type uint32;
            }

            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list GNBCUCPFunction {
        description "gNodeB Centralized Unit Control Plane (gNB-CU-CP)

                    This is a logical node hosting the Radio Resource
                    Control (RRC) and the control plane part of the
                    Packet Data Convergence Protocol (PDCP) of the
                    gNodeB Centralized Unit (gNB-CU) for an E-UTRAN gNodeB
                    (en-gNB) or a gNodeB (gNB). The gNB-CU-CP terminates
                    the E1 interface connected with the gNB-CU-UP and the
                    F1-C interface connected with the gNodeB
                    Distributed Unit (gNB-DU).

                    The following is true for a gNB-CU-CP:
                    Is connected to the gNB-DU through the F1-C interface.
                    Is connected to the gNB-CU-UP through the E1 interface.
                    Only one gNB-CU-CP is connected to one gNB-DU.
                    Only one gNB-CU-CP is connected to one gNB-CU-UP.
                    One gNB-DU can be connected to multiple gNB-CU-UPs
                    under the control of the same gNB-CU-CP.One gNB-CU-UP
                    can be connected to multiple DUs under the control of
                    the same gNB-CU-CP.
                    Note: A gNB may consist of a gNB-CU-CP, multiple
                    gNB-CU-UPs and multiple gNB-DUs. A gNB-CU-CP is a
                    concrete class that extends the NG-RAN node object.
                    In Topology, you can create, read, update, and delete
                    the gNB-CU-CP object.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of the GNBCUCPFunction MO. It contains
                            the full path from the Subnetwork to the
                            GNBCUCPFunction.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

            leaf gNBCUName {
                description "Name of gNodeB-CU";
                type string;
            }

            leaf gNBId {
                description "Identity of gNodeB within a PLMN";
                type uint32;
            }

            leaf gNBIdLength {
                description "Length of gNBId bit string representation";
                type uint32;
            }

            container pLMNId {
                description "PLMN identifier to be used as part
                            of global RAN node identity";
                uses types3gpp:PLMNId;
            }

            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list GNBCUUPFunction {
        description "gNodeB Centralized Unit User Plane (gNB-CU-UP)

                    A gNB-CU-UP is a logical node hosting the User
                    Plane part of the Packet Data Convergence,
                    Protocol (PDCP) of the gNodeB Centralized Unit
                    (gNB-CU) for an E-UTRAN gNodeB (en-gNB), and the
                    User Plane part of the PDCP protocol and the
                    Service Data Adaptation Protocol (SDAP) of the
                    gNB-CU for a gNodeB (gNB). The gNB-CU-UP terminates
                    the E1 interface connected with the gNB-CU-CP and
                    the F1-U interface connected with the gNodeB
                    Distributed Unit (gNB-DU).

                    The following is true for a gNB-CU-UP:
                    Is connected to the gNB-DU through the
                    F1-U interface. Is connected to the gNB-CU-CP through
                    the E1 interface. One gNB-CU-UP is connected to only one
                    gNB-CU-CP. One gNB-DU can be connected to multiple
                    gNB-CU-UPs under the control of the same gNB-CU-CP. One
                    gNB-CU-UP can be connected to multiple DUs under the
                    control of the same gNB-CU-CP.
                    Note: A gNB may consist of a gNB-CU-CP, multiple gNB-CU-UPs
                    and multiple gNB-DUs. A gNB-CU-UP is a concrete class that
                    extends the NG-RAN node object. In Topology, you can
                    create, read, update, and delete the gNB-CU-UP object.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of the GNBCUUPFunction MO. It contains
                            the full path from the Subnetwork to the
                            GNBCUUPFunction.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

            leaf gNBId {
                description "Identity of gNodeB within a PLMN";
                type uint32;
            }

            leaf gNBIdLength {
                description "Length of gNBId bit string representation";
                type uint32;
            }

            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list NRCellCU {
        description "Represents an NR Cell in gNodeB-CU.

                    5G NR is a new radio access technology (RAT)
                    developed by 3GPP for the 5G (fifth generation)
                    mobile network. It is designed to be the global
                    standard for the air interface of 5G networks.

                    5G NR has synchronization signal that is known as
                    Primary Synchronization signal (PSS) and Secondary
                    Synchronization signal (SSS). These signals are
                    specific to NR physical layer and provide the
                    following information required by UE for downlink
                    synchronization: PSS provides Radio Frame Boundary
                    (Position of 1st Symbol in a Radio frame) SSS provides
                    Subframe Boundary (Position of 1st Symbol in a Subframe)
                    Physical Layer Cell ID (PCI) information using both
                    PSS and SSS.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of the NRCellCU MO. It contains
                            the full path from the Subnetwork to the
                            NRCellCU.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

            leaf cellLocalId {
                description "Used together with gNodeB identifier to
                            identify NR cell in PLMN. Used together
                            with gNBId to form NCI.";
                type uint32;
            }

            container plmnId {
                description "PLMN ID for NR CGI. If empty,
                            GNBCUCPFunction::pLMNId is used
                            for PLMN ID in NR CGI";
                uses types3gpp:PLMNId;
            }

            leaf nCI {
                description "NR Cell Identity";
                type uint32;
            }

            leaf nRTAC {
                description "NR Tracking Area Code (TAC)";
                type uint32;
            }

            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list NRCellDU {
        description "Represents an NR Cell in gNodeB-DU.

                    5G NR is a new radio access technology (RAT)
                    developed by 3GPP for the 5G (fifth generation)
                    mobile network. It is designed to be the global
                    standard for the air interface of 5G networks.

                    5G NR has synchronization signal that is known as
                    Primary Synchronization signal (PSS) and Secondary
                    Synchronization signal (SSS). These signals are
                    specific to NR physical layer and provide the
                    following information required by UE for downlink
                    synchronization: PSS provides Radio Frame Boundary
                    (Position of 1st Symbol in a Radio frame) SSS provides
                    Subframe Boundary (Position of 1st Symbol in a Subframe)
                    Physical Layer Cell ID (PCI) information using both
                    PSS and SSS.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of the NRCellDU MO. It contains
                            the full path from the Subnetwork to the
                            NRCellDU.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

            leaf cellLocalId {
                description "Used together with gNodeB identifier to identify NR
                             cell in PLMN. Used together with gNBId to form NCI.";
                type uint32;
            }

            leaf nCI {
                description "NR Cell Identity.";
                type uint32;
            }

            leaf nRPCI {
                description "The Physical Cell Identity (PCI) of the NR cell.";
                type uint32;
            }

            leaf nRTAC {
                description "NR Tracking Area Code (TAC).";
                type uint32;
            }

            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list ENodeBFunction {
        description "An Evolved Node B (eNodeB) is the only mandatory
                    node in the radio access network (RAN) of Long-Term
                    Evolution (LTE). The eNodeB is a complex base
                    station that handles radio communications
                    in the cell and carries out radio resource
                    management and handover decisions. Unlike 2/3G
                    wireless RAN, there is no centralized radio network
                    controller in LTE. It is the hardware that is connected
                    to the mobile phone network that communicates
                    directly with mobile handsets (User Equipment), like a base
                    transceiver station (BTS) in GSM networks. This simplifies
                    the architecture and allows lower response times.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of the ENodeBFunction MO. It contains
                            the full path from the Subnetwork to the
                            ENodeBFunction.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

            leaf eNBId {
                description "The ENodeB ID that forms part of
                            the Cell Global Identity, and is
                            also used to identify the node over
                            the S1 interface";
                type uint32;
            }

            container eNodeBPlmnId {
                description "The ENodeB Public Land Mobile Network
                            (PLMN) ID that forms part of the ENodeB
                            Global ID used to identify the node over
                            the S1 interface. Note: The value (MCC=001, MNC=01)
                            indicates that the PLMN is not initiated.
                            The value can not be used as a valid PLMN Identity.";

                leaf mcc {
                    description "The MCC part of a PLMN identity
                                used in the radio network.";
                    type int32 {
                        range 0..999;
                    }
                }
                leaf mnc {
                    description "The MNC part of a PLMN identity
                                used in the radio network.";
                    type int32 {
                        range 0..999;
                    }
                }
                leaf mncLength {
                    description "The length of the MNC part of a
                                PLMN identity used in the radio network.";
                    type int32 {
                        range 2..3;
                    }
                }
            }

            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list EUtranCell {
        description "Represents an FDD or TDD EUtranCell and
                    contains parameters needed by the cell.
                    It also contains parameters for the
                    mandatory common channels. An EUTRAN stands
                    for Evolved Universal Mobile Telecommunications
                    System (UMTS) Terrestrial Radio Access Network
                    which contains an eNodeB. The eNodeB concrete
                    class is extended from the EUTRAN Node abstract class.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of either the EUtranCellFDD MO or
                            the EUtranCellTDD MO. It contains the full
                            path from the Subnetwork to the EUtranCellFDD or
                            EUtranCellTDD.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

            leaf cellId{
                description "RBS internal ID attribute for EUtranCell.
                            Must be unique in the RBS. Together with the
                            Node ID and Public Land Mobile Network (PLMN)
                            this is a universally unique cell ID";
                type uint32;
            }

            leaf earfcndl {
                description "The channel number for the central downlink frequency.";
                type uint32;
            }

            leaf earfcnul {
                description "Channel number for the central uplink frequency";
                type uint32;
            }

            leaf dlChannelBandwidth {
                description "The downlink channel bandwidth in the FDD cell.";
                type uint32;
            }

            leaf earfcn {
                description "The E-UTRA Absolute Radio Frequency Channel
                            Number (EARFCN) for the TDD cell";
                type uint32;
            }

            leaf channelBandwidth {
                description "The channel bandwidth in the TDD cell.";
                type uint32;
            }

            leaf tac {
                description "Tracking Area Code for the EUtran Cell";
                type uint32;
            }

            leaf duplexType {
                description "Indicator of EUtranCell type, FDD or TDD";
                type enumeration {
                    enum fdd {
                        value 0;
                        description "FDD";
                    }
                    enum tdd {
                        value 1;
                        description "TDD";
                    }
                }
            }

            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list NRSectorCarrier {
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
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of the NRSectorCarrier MO. It contains
                            the full path from the Subnetwork to the
                            NRSectorCarrier.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

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
            
            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list LTESectorCarrier {
        description "The LTE Sector Carrier object provides the
                    attributes for defining the logical characteristics
                    of a carrier (cell) in a sector. A sector is a coverage
                    area associated with a base station having
                    its own antennas, radio ports, and control channels.
                    The concept of sectors was developed to improve co-channel
                    interference in cellular systems, and most wireless systems
                    use three sector cells.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of the SectorCarrier MO. It contains
                            the full path from the Subnetwork to the
                            SectorCarrier.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

            leaf sectorCarrierType {
                description "Indicates whether or not the sector carrier
                            modelled by MO SectorCarrier is a digital sector.";
                type enumeration {
                    enum normal_sector {
                        value 0;
                        description "Not a digital sector";
                    }
                    enum left_digital_sector {
                        value 1;
                        description "Left digital sector for 2DS";
                    }
                    enum right_digital_sector {
                        value 2;
                        description "Right digital sector for 2DS";
                    }
                    enum left_digital_sector_3ds {
                        value 3;
                        description "Left digital sector for 3DS";
                    }
                    enum right_digital_sector_3ds {
                        value 4;
                        description "Right digital sector for 3DS";
                    }
                    enum middle_digital_sector {
                        value 5;
                        description "Middle digital sector for 3DS";
                    }
                }
            }

            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list AntennaCapability {
        description "This MO serves as a mapping between the cell
                    and the RBS equipment used to provide coverage
                    in a certain geographical area. The MO also
                    controls the maximum output power of the sector.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf fdn {
                description "This Full Distinguished Name (FDN) identifies
                            an instance of the SectorEquipmentFunction MO.
                            It contains the full path from the Subnetwork
                            to the SectorEquipmentFunction.";
                type or-teiv-types:_3GPP_FDN_Type;
            }

            leaf-list eUtranFqBands {
                description "List of LTE frequency bands
                            that associated hardware supports";
                type string;
            }

            leaf-list geranFqBands {
                description "List of GERAN frequency bands
                            that associated hardware supports";
                type string;
            }

            leaf-list nRFqBands {
                description "List of NR frequency bands
                            associated hardware supports";
                type string;
            }

            container cmId {
                uses or-teiv-types:CM_ID;
            }
        }
    }

    list Sector {
        description "A group of co-located Cells that
                    have a shared coverage area.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf sectorId {
                description "Universally unique ID generated by the
                            sector's discovery mechanism.";
                type uint64;
            }

            uses geo:geo-location;

            leaf azimuth {
                description "Average value of the azimuths of the cells
                            comprising the sector, determined during
                            sector discovery.";
                type decimal64{
                    fraction-digits 6;
                }
                units "degrees";
            }
        }
    }


    or-teiv-yext:biDirectionalTopologyRelationship ENODEBFUNCTION_PROVIDES_EUTRANCELL { // 1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list provided-euTranCell {
            description "eNodeB Function provides EUTRAN Cell.";
            or-teiv-yext:aSide ENodeBFunction;
            type instance-identifier;
        }

        leaf provided-by-enodebFunction {
            description "EUTRAN Cell provided by eNodeB Function.";
            or-teiv-yext:bSide EUtranCell;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship ENODEBFUNCTION_PROVIDES_LTESECTORCARRIER { // 1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list provided-lteSectorCarrier {
            description "eNodeB Function provides LTE Sector Carrier.";
            or-teiv-yext:aSide ENodeBFunction;
            type instance-identifier;
        }

        leaf provided-by-enodebFunction {
            description "LTE Sector Carrier provided by eNodeB Function.";
            or-teiv-yext:bSide LTESectorCarrier;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship GNBDUFUNCTION_PROVIDES_NRCELLDU { // 1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list provided-nrCellDu {
            description "gNodeB-DU Function provides NR Cell-DU.";
            or-teiv-yext:aSide GNBDUFunction;
            type instance-identifier;
        }

        leaf provided-by-gnbduFunction {
            description "NR Cell-DU provided by gNodeB-DU Function.";
            or-teiv-yext:bSide NRCellDU;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship GNBDUFUNCTION_PROVIDES_NRSECTORCARRIER { // 1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list provided-nrSectorCarrier {
            description "gNodeB-DU Function provides NR Sector Carrier.";
            or-teiv-yext:aSide GNBDUFunction;
            type instance-identifier;
        }

        leaf provided-by-gnbduFunction {
            description "NR Sector Carrier provided by gNodeB-DU Function.";
            or-teiv-yext:bSide NRSectorCarrier;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship GNBCUCPFUNCTION_PROVIDES_NRCELLCU { // 1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list provided-nrCellCu {
            description "gNodeB-CUCP Function provides NR Cell-CU.";
            or-teiv-yext:aSide GNBCUCPFunction;
            type instance-identifier;
        }

        leaf provided-by-gnbcucpFunction {
            description "NR Cell-CU provided by gNodeB-CUCP Function.";
            or-teiv-yext:bSide NRCellCU;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship EUTRANCELL_USES_LTESECTORCARRIER { // 0..1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list used-lteSectorCarrier {
            description "EUTRAN Cell uses LTE Sector Carrier.";
            or-teiv-yext:aSide EUtranCell;
            type instance-identifier;
        }

        leaf used-by-euTranCell {
            description "LTE Sector Carrier used by EUTRAN Cell.";
            or-teiv-yext:bSide LTESectorCarrier;
            type instance-identifier;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship LTESECTORCARRIER_USES_ANTENNACAPABILITY { // 0..n to 0..1

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf used-antennaCapability {
            description "LTE Sector Carrier uses Antenna Capability.";
            or-teiv-yext:aSide LTESectorCarrier;
            type instance-identifier;
        }

        leaf-list used-by-lteSectorCarrier {
            description "Antenna Capability used by LTE Sector Carrier.";
            or-teiv-yext:bSide AntennaCapability;
            type instance-identifier;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship NRCELLDU_USES_NRSECTORCARRIER { // 0..1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list used-nrSectorCarrier {
            description "NR Cell-DU uses NR Sector Carrier.";
            or-teiv-yext:aSide NRCellDU;
            type instance-identifier;
        }

        leaf used-by-nrCellDu {
            description "NR Sector Carrier used by NR Cell-DU.";
            or-teiv-yext:bSide NRSectorCarrier;
            type instance-identifier;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship NRSECTORCARRIER_USES_ANTENNACAPABILITY { // 0..n to 0..1

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf used-antennaCapability {
            description "NR Sector Carrier uses Antenna Capability.";
            or-teiv-yext:aSide NRSectorCarrier;
            type instance-identifier;
        }

        leaf-list used-by-nrSectorCarrier {
            description "Antenna Capability used by NR Sector Carrier.";
            or-teiv-yext:bSide AntennaCapability;
            type instance-identifier;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship SECTOR_GROUPS_NRCELLDU { // 0..1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list grouped-nrCellDu {
            description "Sector groups NR Cell-DU.";
            or-teiv-yext:aSide Sector;
            type instance-identifier;
        }

        leaf grouped-by-sector {
            description "NR Cell-DU grouped by Sector.";
            or-teiv-yext:bSide NRCellDU;
            type instance-identifier;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship SECTOR_GROUPS_EUTRANCELL { // 0..1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list grouped-euTranCell {
            description "Sector groups EUTRAN Cell.";
            or-teiv-yext:aSide Sector;
            type instance-identifier;
        }

        leaf grouped-by-sector {
            description "EUTRAN Cell grouped by Sector.";
            or-teiv-yext:bSide EUtranCell;
            type instance-identifier;
        }
    }
}
```
:::
### 3-5. Relationship: Equipment RAN

![equipment-ran](https://hackmd.io/_uploads/rkdnrEFwC.svg)

:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-rel-equipment-ran {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-rel-equipment-ran";
    prefix or-teiv-rel-equipran;

    import o-ran-smo-teiv-common-yang-types { prefix or-teiv-types; }

    import o-ran-smo-teiv-common-yang-extensions { prefix or-teiv-yext; }

    import o-ran-smo-teiv-equipment { prefix or-teiv-equip; }

    import o-ran-smo-teiv-ran { prefix or-teiv-ran; }


    organization "ORAN";
    contact "The Authors";
    description 
    "RAN Equipment to Logical topology model.

    This model contains the RAN Equipment to Logical topology
    entities and relations.

    Copyright (C) 2024 Ericsson
    Modifications Copyright (C) 2024 OpenInfra Foundation Europe

    Licensed under the Apache License, Version 2.0 (the \"License\");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an \"AS IS\" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

    SPDX-License-Identifier: Apache-2.0";

    revision "2024-05-24" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    or-teiv-yext:domain REL_EQUIPMENT_RAN;

    or-teiv-yext:biDirectionalTopologyRelationship ANTENNAMODULE_SERVES_ANTENNACAPABILITY { // 0..n to 0..m

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list serviced-antennaCapability {
            description "Antenna Capability serviced by this Antenna Module.";
            or-teiv-yext:aSide or-teiv-equip:AntennaModule;
            type instance-identifier;
        }

        leaf-list serving-antennaModule {
            description "Antenna Module serves this Antenna Capability.";
            or-teiv-yext:bSide or-teiv-ran:AntennaCapability;
            type instance-identifier;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship SECTOR_GROUPS_ANTENNAMODULE { // 0..1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list grouped-antennaModule {
            description "Sector groups Antenna Module.";
            or-teiv-yext:aSide or-teiv-ran:Sector;
            type instance-identifier;
        }

        leaf grouped-by-sector {
            description "Antenna Module grouped by Sector.";
            or-teiv-yext:bSide or-teiv-equip:AntennaModule;
            type instance-identifier;
        }
    }
}
```
:::

### 3-6. oam
![oam](https://hackmd.io/_uploads/BkuhSVYPR.svg)

:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-oam {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-oam";
    prefix or-teiv-oam;

    import o-ran-smo-teiv-common-yang-types { prefix or-teiv-types; }

    import o-ran-smo-teiv-common-yang-extensions { prefix or-teiv-yext; }

    organization "ORAN";
    contact "The Authors";
    description 
    "RAN O&M topology model.

    This model contains the topology entities and relations in the
    RAN O&M domain, which are intended to represent management systems
    and management interfaces.

    Copyright (C) 2024 Ericsson
    Modifications Copyright (C) 2024 OpenInfra Foundation Europe

    Licensed under the Apache License, Version 2.0 (the \"License\");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an \"AS IS\" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

    SPDX-License-Identifier: Apache-2.0";

    revision "2024-05-24" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    or-teiv-yext:domain OAM;

    list ManagedElement {
        description "A Managed Element (ME) is a node into a telecommunication
            network providing support and/or service to subscribers. An ME
            communicates with a manager application (directly or indirectly)
            over one or more interfaces for the purpose of being monitored
            and/or controlled.";

        uses or-teiv-types:Top_Grp_Type;
        key id;
    }
}
```
:::

### 3-7. Relationship: OAM RAN

![oam-ran](https://hackmd.io/_uploads/H1_3S4YDR.svg)

:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-rel-oam-ran {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-rel-oam-ran";
    prefix or-teiv-rel-oamran;

    import o-ran-smo-teiv-common-yang-types { prefix or-teiv-types; }

    import o-ran-smo-teiv-common-yang-extensions { prefix or-teiv-yext; }

    import o-ran-smo-teiv-oam { prefix or-teiv-oam; }

    import o-ran-smo-teiv-ran { prefix or-teiv-ran; }

    organization "ORAN";
    contact "The Authors";
    description
    "RAN O&M to Logical topology model.

    This model contains the RAN O&M to Logical topology relations

    Copyright (C) 2024 Ericsson
    Modifications Copyright (C) 2024 OpenInfra Foundation Europe

    Licensed under the Apache License, Version 2.0 (the \"License\");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an \"AS IS\" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

    SPDX-License-Identifier: Apache-2.0";

    revision "2024-05-24" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    or-teiv-yext:domain REL_OAM_RAN;

    or-teiv-yext:biDirectionalTopologyRelationship MANAGEDELEMENT_MANAGES_ENODEBFUNCTION {   // 1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list managed-enodebFunction {
            description "Managed Element manages eNodeB Function.";
            or-teiv-yext:aSide or-teiv-oam:ManagedElement;
            type instance-identifier;
        }

        leaf managed-by-managedElement {
            description "eNodeB Function managed by Managed Element.";
            or-teiv-yext:bSide or-teiv-ran:ENodeBFunction;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship MANAGEDELEMENT_MANAGES_GNBDUFUNCTION {    // 1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list managed-gnbduFunction {
            description "Managed Element manages gNodeB-DU Function.";
            or-teiv-yext:aSide or-teiv-oam:ManagedElement;
            type instance-identifier;
        }

        leaf managed-by-managedElement {
            description "gNodeB-DU Function managed by Managed Element.";
            or-teiv-yext:bSide or-teiv-ran:GNBDUFunction;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship MANAGEDELEMENT_MANAGES_GNBCUCPFUNCTION {    // 1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list managed-gnbcucpFunction {
            description "Managed Element manages gNodeB-CU-CP Function.";
            or-teiv-yext:aSide or-teiv-oam:ManagedElement;
            type instance-identifier;
        }

        leaf managed-by-managedElement {
            description "gNodeB-CU-CP Function managed by Managed Element.";
            or-teiv-yext:bSide or-teiv-ran:GNBCUCPFunction;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship MANAGEDELEMENT_MANAGES_GNBCUUPFUNCTION {    // 1 to 0..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list managed-gnbcuupFunction {
            description "Managed Element manages gNodeB-CU-UP Function.";
            or-teiv-yext:aSide or-teiv-oam:ManagedElement;
            type instance-identifier;
        }

        leaf managed-by-managedElement {
            description "gNodeB-CU-UP Function managed by Managed Element.";
            or-teiv-yext:bSide or-teiv-ran:GNBCUUPFunction;
            type instance-identifier;
            mandatory true;
        }
    }
}
```
:::

### 3-8. Cloud

![cloud](https://hackmd.io/_uploads/ByGQk0_sC.svg)
![cloud-relationships](https://hackmd.io/_uploads/Syf71AuiR.svg)

:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-cloud {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-cloud";
    prefix or-teiv-cloud;

    import o-ran-smo-teiv-common-yang-types {prefix or-teiv-types; }

    import o-ran-smo-teiv-common-yang-extensions {prefix or-teiv-yext; }

    import ietf-geo-location {
        prefix geo;
        reference "RFC 9179: A YANG Grouping for Geographic Locations";
    }

    organization "ORAN";
    contact "The Authors";
    description
    "RAN Cloud topology model.

    This model contains the topology entities and relations in the
    RAN CLOUD domain, which comprises cloud infrastructure and
    deployment aspects that can be used in the topology model.

    Copyright (C) 2024 Ericsson
    Modifications Copyright (C) 2024 OpenInfra Foundation Europe

    Licensed under the Apache License, Version 2.0 (the \"License\");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an \"AS IS\" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

    SPDX-License-Identifier: Apache-2.0";

    revision "2024-05-02" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    or-teiv-yext:domain CLOUD;

    list CloudifiedNF {
        description "A RAN Network Function software that is deployed in the O-Cloud via one or more NF Deployments.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf name {
                description "Name of Cloudified NF";
                type string;
            }
        }
    }

    list NFDeployment {
        description "A software deployment on O-Cloud resources that realizes, all or part of, a Cloudified NF.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf name {
                description "Name of NF Deployment";
                type string;
            }
        }
    }

    list CloudNamespace {
        description "CloudNamespace provide a mechanism for isolating
                    groups of resources within a single cluster.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf name {
                description "Name of Cloud Namespace";
                type string;
            }
        }
    }

    list NodeCluster {
        description "A NodeCluster manages a collection of Nodes.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf name {
                description "Name of Node Cluster";
                type string;
            }
        }
    }

    list CloudSite {
        description "Represents the infrastructure that
                    hosts the NF Deployment.";

        uses or-teiv-types:Top_Grp_Type;
        key id;

        container attributes {
            leaf name {
                description "Name of Cloud Site";
                type string;
            }

            uses geo:geo-location;
        }
    }


    or-teiv-yext:biDirectionalTopologyRelationship CLOUDIFIEDNF_COMPRISES_NFDEPLOYMENT { // 1 to 1..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list comprised-nFDeployment {
            description "Cloudified NF comprises of these NF Deployment.";
            or-teiv-yext:aSide CloudifiedNF;
            type instance-identifier;
            min-elements 1;
        }

        leaf comprised-by-cloudifiedNF {
            description "NF Deployment part of Cloudified NF.";
            or-teiv-yext:bSide NFDeployment;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship NFDEPLOYMENT_DEPLOYED_ON_CLOUDNAMESPACE { // 1..n to 1..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list deployed-on-cloudNamespace {
            description "NF Deployment deployed on Cloud Namespace.";
            or-teiv-yext:aSide NFDeployment;
            type instance-identifier;
            min-elements 1;
        }

        leaf-list deployed-nFDeployment {
            description "Cloud Namespace deploys NF Deployment.";
            or-teiv-yext:bSide CloudNamespace;
            type instance-identifier;
            min-elements 1;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship CLOUDNAMESPACE_DEPLOYED_ON_NODECLUSTER { // 1..n to 1

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf deployed-on-nodeCluster {
            description "Cloud Namespace deployed on Node Cluster.";
            or-teiv-yext:aSide CloudNamespace;
            type instance-identifier;
            mandatory true;
        }

        leaf-list deployed-cloudNamespace {
            description "Node Cluster deploys Cloud Namespace.";
            or-teiv-yext:bSide NodeCluster;
            type instance-identifier;
            min-elements 1;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship NODECLUSTER_LOCATED_AT_CLOUDSITE { // 1..n to 1..n

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list located-at-cloudSite {
            description "Node Cluster located at Cloud Site.";
            or-teiv-yext:aSide NodeCluster;
            type instance-identifier;
            min-elements 1;
        }

        leaf-list location-of-nodeCluster {
            description "Cloud Site is location of Node Cluster.";
            or-teiv-yext:bSide CloudSite;
            type instance-identifier;
            min-elements 1;
        }
    }
}
```
:::

### 3-9. Relationship: CLOUD RAN

![cloud-ran-relationships](https://hackmd.io/_uploads/ByS81C_jA.svg)


:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-rel-cloud-ran {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-rel-cloud-ran";
    prefix or-teiv-cloudtoran;

    import o-ran-smo-teiv-common-yang-types {prefix or-teiv-types; }

    import o-ran-smo-teiv-common-yang-extensions {prefix or-teiv-yext; }

    import o-ran-smo-teiv-cloud {prefix or-teiv-cloud; }

    import o-ran-smo-teiv-ran {prefix or-teiv-ran; }

    organization "ORAN";
    contact "The Authors";
    description
    "RAN Cloud to RAN Logical topology model.

    This model contains the RAN Cloud to RAN Logical topology relations.

    Copyright (C) 2024 Ericsson
    Modifications Copyright (C) 2024 OpenInfra Foundation Europe

    Licensed under the Apache License, Version 2.0 (the \"License\");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an \"AS IS\" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

    SPDX-License-Identifier: Apache-2.0";

    revision "2024-05-02" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    or-teiv-yext:domain REL_CLOUD_RAN;

    or-teiv-yext:biDirectionalTopologyRelationship NFDEPLOYMENT_SERVES_GNBDUFUNCTION { // 0..n to 0..m

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list serviced-gnbduFunction {
            description "gNodeBDU Function serviced by this NF Deployment.";
            or-teiv-yext:aSide or-teiv-cloud:NFDeployment;
            type instance-identifier;
        }

        leaf-list serving-nFDeployment {
            description "NF Deployment that serves this gNodeBDU Function.";
            or-teiv-yext:bSide or-teiv-ran:GNBDUFunction;
            type instance-identifier;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship NFDEPLOYMENT_SERVES_GNBCUCPFUNCTION { // 0..n to 0..m

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list serviced-gnbcucpFunction {
            description "gNodeB-CU-CP Function serviced by this NF Deployment.";
            or-teiv-yext:aSide or-teiv-cloud:NFDeployment;
            type instance-identifier;
        }

        leaf-list serving-nFDeployment {
            description "NF Deployment that serves this gNodeBCUCP Function.";
            or-teiv-yext:bSide or-teiv-ran:GNBCUCPFunction;
            type instance-identifier;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship NFDEPLOYMENT_SERVES_GNBCUUPFUNCTION { // 0..n to 0..m

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf-list serviced-gnbcuupFunction {
            description "gNodeB-CU-UP Function serviced by this NF Deployment.";
            or-teiv-yext:aSide or-teiv-cloud:NFDeployment;
            type instance-identifier;
        }

        leaf-list serving-nFDeployment {
            description "NF Deployment that serves this gNodeBCUUP Function.";
            or-teiv-yext:bSide or-teiv-ran:GNBCUUPFunction;
            type instance-identifier;
        }
    }
}
```
:::

### 3-10. Relationship: OAM CLOUD

![oam-cloud-relationships](https://hackmd.io/_uploads/r1raAaOoC.svg)


:::spoiler Click here to view code
```javascript=
module o-ran-smo-teiv-rel-oam-cloud {
    yang-version 1.1;
    namespace "urn:o-ran:smo-teiv-rel-oam-cloud";
    prefix or-teiv-oamtocloud;

    import o-ran-smo-teiv-common-yang-types {prefix or-teiv-types; }

    import o-ran-smo-teiv-common-yang-extensions {prefix or-teiv-yext; }

    import o-ran-smo-teiv-oam {prefix or-teiv-oam; }

    import o-ran-smo-teiv-cloud {prefix or-teiv-cloud; }

    organization "ORAN";
    contact "The Authors";
    description 
    "RAN O&M to Cloud topology model.

    This model contains the RAN O&M to Cloud topology relations

    Copyright (C) 2024 Ericsson
    Modifications Copyright (C) 2024 OpenInfra Foundation Europe

    Licensed under the Apache License, Version 2.0 (the \"License\");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an \"AS IS\" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

    SPDX-License-Identifier: Apache-2.0";

    revision "2024-05-02" {
        description "Initial revision.";
        or-teiv-yext:label 0.3.0;
    }

    or-teiv-yext:domain REL_OAM_CLOUD;

    or-teiv-yext:biDirectionalTopologyRelationship MANAGEDELEMENT_DEPLOYED_AS_CLOUDIFIEDNF {  // 0..1 to 1

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf deployed-as-cloudifiedNF {
            description "Managed Element deployed as Cloudified NF.";
            or-teiv-yext:aSide or-teiv-oam:ManagedElement;
            type instance-identifier;
        }

        leaf deployed-managedElement {
            description "Cloudified NF deploys Managed Element.";
            or-teiv-yext:bSide or-teiv-cloud:CloudifiedNF;
            type instance-identifier;
            mandatory true;
        }
    }

    or-teiv-yext:biDirectionalTopologyRelationship NFDEPLOYMENT_SERVES_MANAGEDELEMENT { // 1..n to 1

        uses or-teiv-types:Top_Grp_Type;
        key id;

        leaf serviced-managedElement {
            description "Managed Element serviced by this NF Deployment.";
            or-teiv-yext:aSide or-teiv-cloud:NFDeployment;
            type instance-identifier;
            mandatory true;
        }

        leaf-list serving-nFDeployment {
            description "NF Deployment that serves this Managed Element.";
            or-teiv-yext:bSide or-teiv-oam:ManagedElement;
            type instance-identifier;
            min-elements 1;
        }
    }
}
```
:::
## Reference
- https://docs.o-ran-sc.org/projects/o-ran-sc-smo-teiv/en/latest/developer-guide.html
- https://wiki.o-ran-sc.org/pages/viewpage.action?pageId=125042884
- https://gerrit.o-ran-sc.org/r/gitweb?p=smo/teiv.git;a=tree;h=ea1ce6271dfe7d94b004c09d89254c4924c937fc;hb=ea1ce6271dfe7d94b004c09d89254c4924c937fc
