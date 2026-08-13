# Module 3: Multi-Site Routing and Layer 3 Switching

**Static Routing | DHCP Relay | RIPv2 | Multilayer Switching | Routed Ports | EIGRP**


## Module Overview

This module expands the campus network to Edmonton and Ottawa. I first connected the branches through serial point-to-point links, configured static routes, and placed the branch DHCP scopes on the main-campus router. I then removed the static routes and tested RIPv2 before redesigning the topology with Cisco 3560 multilayer switches and EIGRP.

Because this stage contains several migrations and troubleshooting steps, the walkthrough separates intended configurations from what the screenshots conclusively verify.

## Multi-Site Addressing

| Network or Link | Subnet | Interface Addresses |
|---|---|---|
| Edmonton LAN | `192.168.60.0/24` | Gateway `192.168.60.1` |
| Ottawa LAN | `192.168.70.0/24` | Gateway `192.168.70.1` |
| Main Campus to Edmonton | `172.16.10.0/30` | Main Campus `.1`, Edmonton `.2` |
| Main Campus to Ottawa | `172.16.11.0/30` | Main Campus `.1`, Ottawa `.2` |

## Routing Evolution

| Stage | Purpose | Evidence Status |
|---|---|---|
| Static routes | Establish campus-to-branch and inter-branch paths | Passed for tested paths |
| RIPv2 | Replace manual routes with dynamic advertisements | Partially validated |
| Layer 3 switching | Move routing functions to multilayer switches | Configured |
| EIGRP AS 100 | Exchange campus and branch routes dynamically | Partially validated |

## Phase 6: Connected the Edmonton and Ottawa Branches

### 1. Built the Edmonton Branch

<p align="center">
  <img src="https://i.imgur.com/8mn3zUO.png" width="750" alt="Edmonton branch topology">
</p>

```text
Edmonton-Router(config)# interface FastEthernet0/0
Edmonton-Router(config-if)# ip address 192.168.60.1 255.255.255.0
Edmonton-Router(config-if)# no shutdown
Edmonton-Router(config-if)# do write
```

<p align="center">
  <img src="https://i.imgur.com/qkcZWgC.png" width="750" alt="Configuring the Edmonton gateway">
</p>

<p align="center">
  <img src="https://i.imgur.com/0XTztSj.png" width="750" alt="Configuring the first Edmonton PC">
</p>

<p align="center">
  <img src="https://i.imgur.com/pUXonnv.png" width="750" alt="Configuring the second Edmonton PC">
</p>

### 2. Added the Edmonton WAN Link

<p align="center">
  <img src="https://i.imgur.com/xQzfHrL.png" width="750" alt="Installing the main-campus serial module">
</p>

<p align="center">
  <img src="https://i.imgur.com/WoFpGHl.png" width="750" alt="Installing the Edmonton serial module">
</p>

<p align="center">
  <img src="https://i.imgur.com/CIiynof.png" width="750" alt="Connecting Edmonton to the main campus">
</p>

```text
MC_router(config)# interface Serial0/0/0
MC_router(config-if)# ip address 172.16.10.1 255.255.255.252
MC_router(config-if)# no shutdown
```

<p align="center">
  <img src="https://i.imgur.com/3KRLgue.png" width="750" alt="Configuring the main-campus Edmonton link">
</p>

```text
Edmonton-Router(config)# interface Serial0/0/0
Edmonton-Router(config-if)# ip address 172.16.10.2 255.255.255.252
Edmonton-Router(config-if)# clock rate 64000
Edmonton-Router(config-if)# no shutdown
```

<p align="center">
  <img src="https://i.imgur.com/0gD54XK.png" width="750" alt="Configuring the Edmonton serial link">
</p>

### 3. Verified Edmonton Connectivity

<p align="center">
  <img src="https://i.imgur.com/siz1p6L.png" width="750" alt="Edmonton PC reaching its gateway">
</p>

<p align="center">
  <img src="https://i.imgur.com/w2TeXUs.png" width="750" alt="Testing the Edmonton point-to-point link">
</p>

### 4. Added Static Routes for Edmonton

```text
MC_router(config)# ip route 192.168.60.0 255.255.255.0 172.16.10.2
```

<p align="center">
  <img src="https://i.imgur.com/akbD9ip.png" width="750" alt="Adding the Edmonton static route">
</p>

```text
Edmonton-Router(config)# ip route 192.168.10.0 255.255.255.0 172.16.10.1
Edmonton-Router(config)# ip route 192.168.20.0 255.255.255.0 172.16.10.1
Edmonton-Router(config)# ip route 192.168.100.0 255.255.255.0 172.16.10.1
Edmonton-Router(config)# do write
```

<p align="center">
  <img src="https://i.imgur.com/Llwy8wD.png" width="750" alt="Adding Edmonton return routes">
</p>

<p align="center">
  <img src="https://i.imgur.com/dpQEjZa.png" width="750" alt="Successful main-campus-to-Edmonton ping">
</p>

<p align="center">
  <img src="https://i.imgur.com/znuHtXV.png" width="750" alt="Successful Edmonton-to-main-campus ping">
</p>

### 5. Configured DHCP Relay for Edmonton

```text
MC_router(config)# ip dhcp pool EDMONTON
MC_router(dhcp-config)# network 192.168.60.0 255.255.255.0
MC_router(dhcp-config)# default-router 192.168.60.1
MC_router(dhcp-config)# do write
```

<p align="center">
  <img src="https://i.imgur.com/0Klf1Tj.png" width="750" alt="Creating the Edmonton DHCP pool">
</p>

```text
Edmonton-Router(config)# interface FastEthernet0/0
Edmonton-Router(config-if)# ip helper-address 172.16.10.1
Edmonton-Router(config-if)# do write
```

<p align="center">
  <img src="https://i.imgur.com/TcGntgS.png" width="750" alt="Configuring Edmonton DHCP relay">
</p>

### 6. Built the Ottawa Branch

<p align="center">
  <img src="https://i.imgur.com/fDsTxnO.png" width="750" alt="Ottawa branch topology">
</p>

```text
Ottawa-Router(config)# interface FastEthernet0/0
Ottawa-Router(config-if)# ip address 192.168.70.1 255.255.255.0
Ottawa-Router(config-if)# no shutdown
Ottawa-Router(config-if)# do write
```

<p align="center">
  <img src="https://i.imgur.com/ZCerHLC.png" width="750" alt="Configuring the Ottawa gateway">
</p>

<p align="center">
  <img src="https://i.imgur.com/5IGdXDt.png" width="750" alt="Configuring the first Ottawa PC">
</p>

<p align="center">
  <img src="https://i.imgur.com/FBI5tyr.png" width="750" alt="Configuring the second Ottawa PC">
</p>

### 7. Added the Ottawa WAN Link

<p align="center">
  <img src="https://i.imgur.com/vNbeG5w.png" width="750" alt="Installing the Ottawa serial module">
</p>

<p align="center">
  <img src="https://i.imgur.com/qMo6IPV.png" width="750" alt="Expanded multi-site topology">
</p>

```text
MC_router(config)# interface Serial0/0/1
MC_router(config-if)# ip address 172.16.11.1 255.255.255.252
MC_router(config-if)# no shutdown
```

<p align="center">
  <img src="https://i.imgur.com/keLHpsD.png" width="750" alt="Configuring the main-campus Ottawa link">
</p>

```text
Ottawa-Router(config)# interface Serial0/0/1
Ottawa-Router(config-if)# ip address 172.16.11.2 255.255.255.252
Ottawa-Router(config-if)# clock rate 64000
Ottawa-Router(config-if)# no shutdown
Ottawa-Router(config-if)# do write
```

<p align="center">
  <img src="https://i.imgur.com/j8jWOoc.png" width="750" alt="Configuring the Ottawa serial link">
</p>

### 8. Added Ottawa and Inter-Branch Static Routes

```text
MC_router(config)# ip route 192.168.70.0 255.255.255.0 172.16.11.2
```

<p align="center">
  <img src="https://i.imgur.com/9AHQiRe.png" width="750" alt="Adding the Ottawa static route">
</p>

```text
Ottawa-Router(config)# ip route 192.168.10.0 255.255.255.0 172.16.11.1
Ottawa-Router(config)# ip route 192.168.20.0 255.255.255.0 172.16.11.1
Ottawa-Router(config)# ip route 192.168.30.0 255.255.255.0 172.16.11.1
Ottawa-Router(config)# ip route 192.168.60.0 255.255.255.0 172.16.11.1
Ottawa-Router(config)# do write
```

<p align="center">
  <img src="https://i.imgur.com/9AHQiRe.png" width="750" alt="Adding Ottawa return routes">
</p>

```text
Edmonton-Router(config)# ip route 192.168.70.0 255.255.255.0 172.16.10.1
```

<p align="center">
  <img src="https://i.imgur.com/m68gM2j.png" width="750" alt="Adding the Edmonton-to-Ottawa route">
</p>

### 9. Configured DHCP Relay for Ottawa

```text
MC_router(config)# ip dhcp pool OTTAWA
MC_router(dhcp-config)# network 192.168.70.0 255.255.255.0
MC_router(dhcp-config)# default-router 192.168.70.1
MC_router(dhcp-config)# do write
```

<p align="center">
  <img src="https://i.imgur.com/iXrem3p.png" width="750" alt="Creating the Ottawa DHCP pool">
</p>

```text
Ottawa-Router(config)# interface FastEthernet0/0
Ottawa-Router(config-if)# ip helper-address 172.16.11.1
Ottawa-Router(config-if)# do write
```

<p align="center">
  <img src="https://i.imgur.com/mrsnUvf.png" width="750" alt="Configuring Ottawa DHCP relay">
</p>

## Phase 7: Migrated from Static Routing to RIPv2

### 1. Removed the Static Routes

<p align="center">
  <img src="https://i.imgur.com/FfEtsDa.png" width="750" alt="Removing static routes from the main-campus router">
</p>

<p align="center">
  <img src="https://i.imgur.com/sUEoB3r.png" width="750" alt="Removing static routes from Edmonton">
</p>

<p align="center">
  <img src="https://i.imgur.com/kQv917v.png" width="750" alt="Removing static routes from Ottawa">
</p>

### 2. Configured RIPv2

```text
MC_router(config)# router rip
MC_router(config-router)# version 2
MC_router(config-router)# network 192.168.10.0
MC_router(config-router)# network 192.168.20.0
MC_router(config-router)# network 192.168.30.0
MC_router(config-router)# network 172.16.10.0
MC_router(config-router)# network 172.16.11.0
MC_router(config-router)# no auto-summary
```

<p align="center">
  <img src="https://i.imgur.com/ClLEFkn.png" width="750" alt="Configuring RIPv2 on the main-campus router">
</p>

```text
Edmonton-Router(config)# router rip
Edmonton-Router(config-router)# version 2
Edmonton-Router(config-router)# network 172.16.10.0
Edmonton-Router(config-router)# network 192.168.60.0
Edmonton-Router(config-router)# no auto-summary
```

<p align="center">
  <img src="https://i.imgur.com/Q5bmnyr.png" width="750" alt="Configuring RIPv2 on Edmonton">
</p>

```text
Ottawa-Router(config)# router rip
Ottawa-Router(config-router)# version 2
Ottawa-Router(config-router)# network 172.16.11.0
Ottawa-Router(config-router)# network 192.168.70.0
Ottawa-Router(config-router)# no auto-summary
```

<p align="center">
  <img src="https://i.imgur.com/Mv1FZVg.png" width="750" alt="Configuring RIPv2 on Ottawa">
</p>

### 3. Tested RIP-Stage Reachability

<p align="center">
  <img src="https://i.imgur.com/xyP1S9M.png" width="750" alt="Successful Ottawa-to-main-campus ping">
</p>

<p align="center">
  <img src="https://i.imgur.com/3WClKxl.png" width="750" alt="Successful Edmonton-to-main-campus ping">
</p>

<p align="center">
  <img src="https://i.imgur.com/x74l5xN.png" width="750" alt="Successful Edmonton-to-Ottawa ping">
</p>


## Phase 8: Redesigned the Network with Layer 3 Switches and EIGRP

### 1. Replaced the Router with a Multilayer Switch   

<p align="center">
  <img src="https://i.imgur.com/r4ZfvlN.png" width="750" alt="Selecting a Cisco 3560 multilayer switch">
</p>

<p align="center">
  <img src="https://i.imgur.com/WM72YW1.png" width="750" alt="Renaming the multilayer switch MC_Switch">
</p>

### 2. Recreated the Campus VLANs

<p align="center">
  <img src="https://i.imgur.com/H62wrlS.png" width="750" alt="Verifying VLANs on MC_Switch">
</p>

```text
MC_Switch(config)# interface fa0/2
MC_Switch(config-if)# switchport access vlan 10
MC_Switch(config)# interface fa0/3
MC_Switch(config-if)# switchport access vlan 20
MC_Switch(config)# interface fa0/4
MC_Switch(config-if)# switchport access vlan 30
MC_Switch(config)# interface fa0/5
MC_Switch(config-if)# switchport access vlan 50
MC_Switch(config)# interface range fa0/6-7
MC_Switch(config-if-range)# switchport access vlan 100
MC_Switch(config-if-range)# do write
```

<p align="center">
  <img src="https://i.imgur.com/8k8pC9L.png" width="750" alt="Assigning MC_Switch access ports">
</p>

<p align="center">
  <img src="https://i.imgur.com/W5i1Cyy.png" width="750" alt="Verifying VLAN membership">
</p>

### 3. Created Switched Virtual Interfaces

```text
MC_Switch(config)# ip routing

MC_Switch(config)# interface vlan 10
MC_Switch(config-if)# ip address 192.168.10.1 255.255.255.0

MC_Switch(config)# interface vlan 20
MC_Switch(config-if)# ip address 192.168.20.1 255.255.255.0

MC_Switch(config)# interface vlan 30
MC_Switch(config-if)# ip address 192.168.30.1 255.255.255.0

MC_Switch(config)# interface vlan 50
MC_Switch(config-if)# ip address 192.168.50.1 255.255.255.0

MC_Switch(config)# interface vlan 100
MC_Switch(config-if)# ip address 192.168.100.1 255.255.255.0
```

<p align="center">
  <img src="https://i.imgur.com/U75ZdZi.png" width="750" alt="Creating VLAN interfaces on MC_Switch">
</p>

### 4. Recreated the Campus DHCP Pools

<p align="center">
  <img src="https://i.imgur.com/X7iOgLS.png" width="750" alt="Creating DHCP pools on MC_Switch">
</p>

### 5. Converted Branch Links to Routed Ports

<p align="center">
  <img src="https://i.imgur.com/pZ4OVUN.png" width="750" alt="Layer 3 branch topology">
</p>

```text
MC_Switch(config)# interface GigabitEthernet0/1
MC_Switch(config-if)# no switchport
MC_Switch(config-if)# ip address 172.16.10.1 255.255.255.252
MC_Switch(config-if)# no shutdown
```

<p align="center">
  <img src="https://i.imgur.com/5sN4g2T.png" width="750" alt="Configuring the Edmonton link on MC_Switch">
</p>

```text
Edmonton-Switch(config)# interface GigabitEthernet0/1
Edmonton-Switch(config-if)# no switchport
Edmonton-Switch(config-if)# ip address 172.16.10.2 255.255.255.252
Edmonton-Switch(config-if)# no shutdown
```

<p align="center">
  <img src="https://i.imgur.com/Q0dgdKJ.png" width="750" alt="Configuring the Edmonton routed uplink">
</p>

<p align="center">
  <img src="https://i.imgur.com/MLTASLW.png" width="750" alt="Configuring the Edmonton LAN interface">
</p>

```text
MC_Switch(config)# interface GigabitEthernet0/2
MC_Switch(config-if)# no switchport
MC_Switch(config-if)# ip address 172.16.11.1 255.255.255.252
MC_Switch(config-if)# no shutdown
```

<p align="center">
  <img src="https://i.imgur.com/c00ivD7.png" width="750" alt="Configuring the Ottawa link on MC_Switch">
</p>

<p align="center">
  <img src="https://i.imgur.com/LLgNXtG.png" width="750" alt="Configuring the Ottawa routed uplink">
</p>

<p align="center">
  <img src="https://i.imgur.com/sHBCtsp.png" width="750" alt="Reviewing the Ottawa LAN interface">
</p>

The intended Ottawa LAN configuration is:

```text
Ottawa-Switch(config)# interface FastEthernet0/1
Ottawa-Switch(config-if)# no switchport
Ottawa-Switch(config-if)# ip address 192.168.70.1 255.255.255.0
Ottawa-Switch(config-if)# no shutdown
```

### 6. Added the Branch DHCP Pools

```text
MC_Switch(config)# ip dhcp pool OTTAWA
MC_Switch(dhcp-config)# network 192.168.70.0 255.255.255.0
MC_Switch(dhcp-config)# default-router 192.168.70.1
```

<p align="center">
  <img src="https://i.imgur.com/KpNZyyh.png" width="750" alt="Creating the Ottawa DHCP pool">
</p>

<p align="center">
  <img src="https://i.imgur.com/K7zhFjh.png" width="750" alt="Displaying branch DHCP pools">
</p>

```text
MC_Switch(config)# ip dhcp excluded-address 192.168.60.1
MC_Switch(config)# ip dhcp excluded-address 192.168.70.1
```

### 7. Reviewed Layer 3 DHCP Relay

<p align="center">
  <img src="https://i.imgur.com/d1aGkRe.png" width="750" alt="Reviewing Edmonton DHCP relay">
</p>

```text
Edmonton-Switch(config)# interface FastEthernet0/1
Edmonton-Switch(config-if)# ip helper-address 172.16.10.1
```

<p align="center">
  <img src="https://i.imgur.com/6vctwUk.png" width="750" alt="Correcting Ottawa DHCP relay">
</p>

### 8. Configured EIGRP AS 100

<p align="center">
  <img src="https://i.imgur.com/rJ8dtLW.png" width="750" alt="Configuring EIGRP on MC_Switch">
</p>

```text
MC_Switch(config)# ip routing
MC_Switch(config)# router eigrp 100
MC_Switch(config-router)# network 192.168.10.0 0.0.0.255
MC_Switch(config-router)# network 192.168.20.0 0.0.0.255
MC_Switch(config-router)# network 192.168.30.0 0.0.0.255
MC_Switch(config-router)# network 192.168.50.0 0.0.0.255
MC_Switch(config-router)# network 192.168.100.0 0.0.0.255
MC_Switch(config-router)# network 172.16.10.0 0.0.0.3
MC_Switch(config-router)# network 172.16.11.0 0.0.0.3
```

```text
Edmonton-Switch(config)# ip routing
Edmonton-Switch(config)# router eigrp 100
Edmonton-Switch(config-router)# network 192.168.60.0 0.0.0.255
Edmonton-Switch(config-router)# network 172.16.10.0 0.0.0.3
```

<p align="center">
  <img src="https://i.imgur.com/NHdS8fa.png" width="750" alt="Configuring EIGRP on Edmonton-Switch">
</p>

```text
Ottawa-Switch(config)# ip routing
Ottawa-Switch(config)# router eigrp 100
Ottawa-Switch(config-router)# network 192.168.70.0 0.0.0.255
Ottawa-Switch(config-router)# network 172.16.11.0 0.0.0.3
```

<p align="center">
  <img src="https://i.imgur.com/TaqrlX2.png" width="750" alt="Configuring EIGRP on Ottawa-Switch">
</p>

### 9. Tested Branch-to-Branch Reachability

<p align="center">
  <img src="https://i.imgur.com/fLwn0xI.png" width="750" alt="Successful Ottawa-to-Edmonton ping">
</p>

<p align="center">
  <img src="https://i.imgur.com/UAqty1o.png" width="750" alt="Successful Edmonton-to-Ottawa ping">
</p>




## Module Outcome

This stage demonstrated the operational difference between static and dynamic routing and showed how a router-based campus can migrate to multilayer switching. Connectivity tests confirmed that the sites could communicate, while the review identified the additional protocol evidence and addressing corrections required for a fully validated final design.

## Project Navigation

[← Previous: Network Security with ACLs and Port Security](../02-network-security/README.md)  
[Return to the main project](https://github.com/AdeniyiAdesakin/cisco-packet-tracer-campus-network)
