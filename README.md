# Module 3: Multi-Site Routing and Layer 3 Switching

**Static Routing | DHCP Relay | RIPv2 | Multilayer Switching | Routed Ports | EIGRP**

[← Return to the main project](../README.md)

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
  <img src="../images/multisite-routing/01.png" width="750" alt="Edmonton branch topology">
</p>

```text
Edmonton-Router(config)# interface FastEthernet0/0
Edmonton-Router(config-if)# ip address 192.168.60.1 255.255.255.0
Edmonton-Router(config-if)# no shutdown
Edmonton-Router(config-if)# do write
```

<p align="center">
  <img src="../images/multisite-routing/02.png" width="750" alt="Configuring the Edmonton gateway">
</p>

<p align="center">
  <img src="../images/multisite-routing/03.png" width="750" alt="Configuring the first Edmonton PC">
</p>

<p align="center">
  <img src="../images/multisite-routing/04.png" width="750" alt="Configuring the second Edmonton PC">
</p>

### 2. Added the Edmonton WAN Link

<p align="center">
  <img src="../images/multisite-routing/05.png" width="750" alt="Installing the main-campus serial module">
</p>

<p align="center">
  <img src="../images/multisite-routing/06.png" width="750" alt="Installing the Edmonton serial module">
</p>

<p align="center">
  <img src="../images/multisite-routing/07.png" width="750" alt="Connecting Edmonton to the main campus">
</p>

```text
MC_router(config)# interface Serial0/0/0
MC_router(config-if)# ip address 172.16.10.1 255.255.255.252
MC_router(config-if)# no shutdown
```

<p align="center">
  <img src="../images/multisite-routing/08.png" width="750" alt="Configuring the main-campus Edmonton link">
</p>

```text
Edmonton-Router(config)# interface Serial0/0/0
Edmonton-Router(config-if)# ip address 172.16.10.2 255.255.255.252
Edmonton-Router(config-if)# clock rate 64000
Edmonton-Router(config-if)# no shutdown
```

<p align="center">
  <img src="../images/multisite-routing/09.png" width="750" alt="Configuring the Edmonton serial link">
</p>

### 3. Verified Edmonton Connectivity

<p align="center">
  <img src="../images/multisite-routing/10.png" width="750" alt="Edmonton PC reaching its gateway">
</p>

<p align="center">
  <img src="../images/multisite-routing/11.png" width="750" alt="Testing the Edmonton point-to-point link">
</p>

### 4. Added Static Routes for Edmonton

```text
MC_router(config)# ip route 192.168.60.0 255.255.255.0 172.16.10.2
```

<p align="center">
  <img src="../images/multisite-routing/12.png" width="750" alt="Adding the Edmonton static route">
</p>

```text
Edmonton-Router(config)# ip route 192.168.10.0 255.255.255.0 172.16.10.1
Edmonton-Router(config)# ip route 192.168.20.0 255.255.255.0 172.16.10.1
Edmonton-Router(config)# ip route 192.168.100.0 255.255.255.0 172.16.10.1
Edmonton-Router(config)# do write
```

<p align="center">
  <img src="../images/multisite-routing/13.png" width="750" alt="Adding Edmonton return routes">
</p>

<p align="center">
  <img src="../images/multisite-routing/14.png" width="750" alt="Successful main-campus-to-Edmonton ping">
</p>

<p align="center">
  <img src="../images/multisite-routing/15.png" width="750" alt="Successful Edmonton-to-main-campus ping">
</p>

### 5. Configured DHCP Relay for Edmonton

```text
MC_router(config)# ip dhcp pool EDMONTON
MC_router(dhcp-config)# network 192.168.60.0 255.255.255.0
MC_router(dhcp-config)# default-router 192.168.60.1
MC_router(dhcp-config)# do write
```

<p align="center">
  <img src="../images/multisite-routing/16.png" width="750" alt="Creating the Edmonton DHCP pool">
</p>

```text
Edmonton-Router(config)# interface FastEthernet0/0
Edmonton-Router(config-if)# ip helper-address 172.16.10.1
Edmonton-Router(config-if)# do write
```

<p align="center">
  <img src="../images/multisite-routing/17.png" width="750" alt="Configuring Edmonton DHCP relay">
</p>

### 6. Built the Ottawa Branch

<p align="center">
  <img src="../images/multisite-routing/18.png" width="750" alt="Ottawa branch topology">
</p>

```text
Ottawa-Router(config)# interface FastEthernet0/0
Ottawa-Router(config-if)# ip address 192.168.70.1 255.255.255.0
Ottawa-Router(config-if)# no shutdown
Ottawa-Router(config-if)# do write
```

<p align="center">
  <img src="../images/multisite-routing/19.png" width="750" alt="Configuring the Ottawa gateway">
</p>

<p align="center">
  <img src="../images/multisite-routing/20.png" width="750" alt="Configuring the first Ottawa PC">
</p>

<p align="center">
  <img src="../images/multisite-routing/21.png" width="750" alt="Configuring the second Ottawa PC">
</p>

### 7. Added the Ottawa WAN Link

<p align="center">
  <img src="../images/multisite-routing/22.png" width="750" alt="Installing the Ottawa serial module">
</p>

<p align="center">
  <img src="../images/multisite-routing/23.png" width="750" alt="Expanded multi-site topology">
</p>

```text
MC_router(config)# interface Serial0/0/1
MC_router(config-if)# ip address 172.16.11.1 255.255.255.252
MC_router(config-if)# no shutdown
```

<p align="center">
  <img src="../images/multisite-routing/24.png" width="750" alt="Configuring the main-campus Ottawa link">
</p>

```text
Ottawa-Router(config)# interface Serial0/0/1
Ottawa-Router(config-if)# ip address 172.16.11.2 255.255.255.252
Ottawa-Router(config-if)# clock rate 64000
Ottawa-Router(config-if)# no shutdown
Ottawa-Router(config-if)# do write
```

<p align="center">
  <img src="../images/multisite-routing/25.png" width="750" alt="Configuring the Ottawa serial link">
</p>

### 8. Added Ottawa and Inter-Branch Static Routes

```text
MC_router(config)# ip route 192.168.70.0 255.255.255.0 172.16.11.2
```

<p align="center">
  <img src="../images/multisite-routing/26.png" width="750" alt="Adding the Ottawa static route">
</p>

```text
Ottawa-Router(config)# ip route 192.168.10.0 255.255.255.0 172.16.11.1
Ottawa-Router(config)# ip route 192.168.20.0 255.255.255.0 172.16.11.1
Ottawa-Router(config)# ip route 192.168.30.0 255.255.255.0 172.16.11.1
Ottawa-Router(config)# ip route 192.168.60.0 255.255.255.0 172.16.11.1
Ottawa-Router(config)# do write
```

<p align="center">
  <img src="../images/multisite-routing/27.png" width="750" alt="Adding Ottawa return routes">
</p>

```text
Edmonton-Router(config)# ip route 192.168.70.0 255.255.255.0 172.16.10.1
```

<p align="center">
  <img src="../images/multisite-routing/28.png" width="750" alt="Adding the Edmonton-to-Ottawa route">
</p>

### 9. Configured DHCP Relay for Ottawa

```text
MC_router(config)# ip dhcp pool OTTAWA
MC_router(dhcp-config)# network 192.168.70.0 255.255.255.0
MC_router(dhcp-config)# default-router 192.168.70.1
MC_router(dhcp-config)# do write
```

<p align="center">
  <img src="../images/multisite-routing/29.png" width="750" alt="Creating the Ottawa DHCP pool">
</p>

```text
Ottawa-Router(config)# interface FastEthernet0/0
Ottawa-Router(config-if)# ip helper-address 172.16.11.1
Ottawa-Router(config-if)# do write
```

<p align="center">
  <img src="../images/multisite-routing/30.png" width="750" alt="Configuring Ottawa DHCP relay">
</p>

## Phase 7: Migrated from Static Routing to RIPv2

### 1. Removed the Static Routes

<p align="center">
  <img src="../images/multisite-routing/31.png" width="750" alt="Removing static routes from the main-campus router">
</p>

<p align="center">
  <img src="../images/multisite-routing/32.png" width="750" alt="Removing static routes from Edmonton">
</p>

<p align="center">
  <img src="../images/multisite-routing/33.png" width="750" alt="Removing static routes from Ottawa">
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
  <img src="../images/multisite-routing/34.png" width="750" alt="Configuring RIPv2 on the main-campus router">
</p>

```text
Edmonton-Router(config)# router rip
Edmonton-Router(config-router)# version 2
Edmonton-Router(config-router)# network 172.16.10.0
Edmonton-Router(config-router)# network 192.168.60.0
Edmonton-Router(config-router)# no auto-summary
```

<p align="center">
  <img src="../images/multisite-routing/35.png" width="750" alt="Configuring RIPv2 on Edmonton">
</p>

```text
Ottawa-Router(config)# router rip
Ottawa-Router(config-router)# version 2
Ottawa-Router(config-router)# network 172.16.11.0
Ottawa-Router(config-router)# network 192.168.70.0
Ottawa-Router(config-router)# no auto-summary
```

<p align="center">
  <img src="../images/multisite-routing/36.png" width="750" alt="Configuring RIPv2 on Ottawa">
</p>

### 3. Tested RIP-Stage Reachability

<p align="center">
  <img src="../images/multisite-routing/37.png" width="750" alt="Successful Ottawa-to-main-campus ping">
</p>

<p align="center">
  <img src="../images/multisite-routing/38.png" width="750" alt="Successful Edmonton-to-main-campus ping">
</p>

<p align="center">
  <img src="../images/multisite-routing/39.png" width="750" alt="Successful Edmonton-to-Ottawa ping">
</p>

The tests confirm reachability, but no `show ip route rip` or `show ip protocols` output was captured.

## Phase 8: Redesigned the Network with Layer 3 Switches and EIGRP

### 1. Replaced the Router with a Multilayer Switch

<p align="center">
  <img src="../images/multisite-routing/40.png" width="750" alt="Selecting a Cisco 3560 multilayer switch">
</p>

<p align="center">
  <img src="../images/multisite-routing/41.png" width="750" alt="Renaming the multilayer switch MC_Switch">
</p>

### 2. Recreated the Campus VLANs

<p align="center">
  <img src="../images/multisite-routing/42.png" width="750" alt="Verifying VLANs on MC_Switch">
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
  <img src="../images/multisite-routing/43.png" width="750" alt="Assigning MC_Switch access ports">
</p>

<p align="center">
  <img src="../images/multisite-routing/44.png" width="750" alt="Verifying VLAN membership">
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
  <img src="../images/multisite-routing/45.png" width="750" alt="Creating VLAN interfaces on MC_Switch">
</p>

### 4. Recreated the Campus DHCP Pools

<p align="center">
  <img src="../images/multisite-routing/46.png" width="750" alt="Creating DHCP pools on MC_Switch">
</p>

### 5. Converted Branch Links to Routed Ports

<p align="center">
  <img src="../images/multisite-routing/47.png" width="750" alt="Layer 3 branch topology">
</p>

```text
MC_Switch(config)# interface GigabitEthernet0/1
MC_Switch(config-if)# no switchport
MC_Switch(config-if)# ip address 172.16.10.1 255.255.255.252
MC_Switch(config-if)# no shutdown
```

<p align="center">
  <img src="../images/multisite-routing/48.png" width="750" alt="Configuring the Edmonton link on MC_Switch">
</p>

```text
Edmonton-Switch(config)# interface GigabitEthernet0/1
Edmonton-Switch(config-if)# no switchport
Edmonton-Switch(config-if)# ip address 172.16.10.2 255.255.255.252
Edmonton-Switch(config-if)# no shutdown
```

<p align="center">
  <img src="../images/multisite-routing/49.png" width="750" alt="Configuring the Edmonton routed uplink">
</p>

<p align="center">
  <img src="../images/multisite-routing/50.png" width="750" alt="Configuring the Edmonton LAN interface">
</p>

```text
MC_Switch(config)# interface GigabitEthernet0/2
MC_Switch(config-if)# no switchport
MC_Switch(config-if)# ip address 172.16.11.1 255.255.255.252
MC_Switch(config-if)# no shutdown
```

<p align="center">
  <img src="../images/multisite-routing/51.png" width="750" alt="Configuring the Ottawa link on MC_Switch">
</p>

<p align="center">
  <img src="../images/multisite-routing/52.png" width="750" alt="Configuring the Ottawa routed uplink">
</p>

<p align="center">
  <img src="../images/multisite-routing/53.png" width="750" alt="Reviewing the Ottawa LAN interface">
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
  <img src="../images/multisite-routing/54.png" width="750" alt="Creating the Ottawa DHCP pool">
</p>

<p align="center">
  <img src="../images/multisite-routing/55.png" width="750" alt="Displaying branch DHCP pools">
</p>

```text
MC_Switch(config)# ip dhcp excluded-address 192.168.60.1
MC_Switch(config)# ip dhcp excluded-address 192.168.70.1
```

### 7. Reviewed Layer 3 DHCP Relay

<p align="center">
  <img src="../images/multisite-routing/56.png" width="750" alt="Reviewing Edmonton DHCP relay">
</p>

```text
Edmonton-Switch(config)# interface FastEthernet0/1
Edmonton-Switch(config-if)# ip helper-address 172.16.10.1
```

<p align="center">
  <img src="../images/multisite-routing/57.png" width="750" alt="Correcting Ottawa DHCP relay">
</p>

### 8. Configured EIGRP AS 100

<p align="center">
  <img src="../images/multisite-routing/58.png" width="750" alt="Configuring EIGRP on MC_Switch">
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
  <img src="../images/multisite-routing/59.png" width="750" alt="Configuring EIGRP on Edmonton-Switch">
</p>

```text
Ottawa-Switch(config)# ip routing
Ottawa-Switch(config)# router eigrp 100
Ottawa-Switch(config-router)# network 192.168.70.0 0.0.0.255
Ottawa-Switch(config-router)# network 172.16.11.0 0.0.0.3
```

<p align="center">
  <img src="../images/multisite-routing/60.png" width="750" alt="Configuring EIGRP on Ottawa-Switch">
</p>

### 9. Tested Branch-to-Branch Reachability

<p align="center">
  <img src="../images/multisite-routing/61.png" width="750" alt="Successful Ottawa-to-Edmonton ping">
</p>

<p align="center">
  <img src="../images/multisite-routing/62.png" width="750" alt="Successful Edmonton-to-Ottawa ping">
</p>

## Module Validation Summary

| Area | Result | Evidence and Limitation |
|---|---|---|
| Edmonton LAN and point-to-point link | Passed | Local gateway and transit pings succeeded |
| Edmonton static routing | Passed for tested paths | Main-campus and VLAN 10 paths succeeded |
| Ottawa static routing | Configured | Route commands were captured |
| Router-based DHCP relay | Configured | Correct helper targets captured; branch leases not shown |
| RIPv2 | Partially validated | Pings passed, but no RIP route table was captured |
| Campus SVIs | Configured | Gateway interfaces and addresses captured |
| Routed switch links | Configured | Both transit links documented |
| Ottawa Layer 3 LAN interface | Requires correction | Capture shows `192.168.60.1` instead of `192.168.70.1` |
| Branch DHCP exclusions | Requires correction | Gateway addresses remain inside the allocatable ranges |
| Edmonton Layer 3 DHCP relay | Requires correction | Helper targets the local `.2` address |
| EIGRP | Partially validated | Branch pings passed; neighbor and learned-route evidence was not captured |

## Required Final Corrections

- Correct the Ottawa client-facing interface to `192.168.70.1/24`.
- Exclude `192.168.60.1` and `192.168.70.1` from the branch DHCP pools.
- Point Edmonton’s helper address to `172.16.10.1`.
- Confirm that MC_Switch advertises both transit networks in EIGRP.
- Capture `show ip eigrp neighbors`, `show ip route eigrp`, and `show ip protocols`.
- Save a final clean running configuration after every correction.

## Module Outcome

This stage demonstrated the operational difference between static and dynamic routing and showed how a router-based campus can migrate to multilayer switching. Connectivity tests confirmed that the sites could communicate, while the review identified the additional protocol evidence and addressing corrections required for a fully validated final design.

## Project Navigation

[← Previous: Network Security with ACLs and Port Security](../02-network-security/README.md)  
[Return to the main project](../README.md)
