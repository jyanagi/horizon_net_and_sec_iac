# Topology
<p align="center">
<img src=https://user-images.githubusercontent.com/75082831/178127630-f6d4e2d9-5c2b-4e90-ad17-53537118fc05.png>
</p>

The following terraform plan was created to simplify Day 2 networking and security operations of Horizon through NSX-T.

The only file that needs modification are the variables.tf within the .\networking and .\security directories to fit the IP addressing, naming standards or tag criteria for an organization.

# Requirements
VMware software-defined data center components are installed
- vSphere 7.x 
  - vDS Version is set to 7.0.0 minimum
- NSX-T 3.x
  - BGP is supported/configured on physical underlay
  - NSX-T Unified Appliance is deployed
  - NSX Host Transport Nodes are configured 
  - Two (2) NSX edge transport nodes are deployed
    - Both edge nodes are Attached to an Edge Cluster
    - No Tier0 gateways are associated with this Edge Cluster

# Instructions

1. Install Terraform (https://learn.hashicorp.com/tutorials/terraform/install-cli)

2. Clone this GIT repository: ```git clone https://github.com/jyanagi/horizon_net_and_sec_iac.git```

3. Navigate to the 'horizon_net_and_sec_iac' directory 

4. Run the following commands: 

   4a. Initializes the Terraform provider
       ```terraform init```

   4b. Pre-Check before deployment of Terraform Plan
       ```terraform plan```

   4c. Deploys objects as defined in Terraform Plan
       ```terraform apply -auto-approve```

<details><summary> 
<h1>Terraform Deployment Overview <i>(Click to Expand)</i></h1>
</summary>

- Two (2) VLAN-backed segments
  - nsx-vlan-3301-seg
    - VLAN ID: 3301
  - nsx-vlan-3311-seg
    - VLAN ID: 3311
- Tier0 Gateway
  - TF_Tier_0
    - HA Mode:  Active/Active
    - Four Uplink Interfaces
      - Fabric-A
        - edge01a-uplink-01 - 10.33.1.1/24
        - edge01b-uplink-01 - 10.33.1.2/24
      - Fabric-B
        - edge01a-uplink-02 - 10.33.11.1/24
        - edge01b-uplink-02 - 10.33.11.2/24
    - BGP
      - Local AS: 3301
      - ECMP Enabled
      - Two (2) BGP Neighbors
        - Fabric-A: 
          - Remote Address: 10.33.1.253
          - Remote AS: 65000
          - BFD: Enabled
        - Fabric-B:
          - Remote Address: 10.33.11.253
          - Remote AS: 65000
          - BFD: Enabled
    - BGP Redistribution Policy
      - Redistributed Networks:
        - Connected Segments and Service Interfaces
        - LB SNAT IP
        - LB VIP
        - NAT IP
- Tier1 Gateway
  - TF_Tier_1
    - HA Mode:  Active/Standby
    - Route Advertisement:
      - Connected Segments and Service Interfaces
      - LB SNAT IP
      - LB VIP
      - NAT IP
- Three (3) Overlay Segments
  - TF-Segment-ENT-SVC
    - Gateway: TF_Tier_1
    - IP: 10.201.10.253/24
    - Scope|Tag:  domain|ent_svc
  - TF-Segment-HZN-UAG
    - Gateway: TF_Tier_1
    - IP: 10.201.20.253/24
    - Scope|Tag: horizon|uag
  - TF-Segment-HZN-CS
    - Gateway: TF_Tier_1
    - IP: 10.201.30.253/24
    - Scope|Tag: horizon|cs
- 10 Security Groups (HZN-GRP-###)
- 27 services (HZN-SVC-###)
- 44 DFW Application Policies 
</details>
