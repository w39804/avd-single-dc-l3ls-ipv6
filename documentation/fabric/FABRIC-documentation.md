# FABRIC

## Table of Contents

- [Fabric Switches and Management IP](#fabric-switches-and-management-ip)
  - [Fabric Switches with inband Management IP](#fabric-switches-with-inband-management-ip)
- [Fabric Topology](#fabric-topology)
- [Fabric IP Allocation](#fabric-ip-allocation)
  - [Fabric Point-To-Point Links](#fabric-point-to-point-links)
  - [Point-To-Point Links Node Allocation](#point-to-point-links-node-allocation)
  - [Loopback Interfaces (BGP EVPN Peering)](#loopback-interfaces-bgp-evpn-peering)
  - [Loopback0 Interfaces Node Allocation](#loopback0-interfaces-node-allocation)
  - [VTEP Loopback VXLAN Tunnel Source Interfaces (VTEPs Only)](#vtep-loopback-vxlan-tunnel-source-interfaces-vteps-only)
  - [VTEP Loopback Node allocation](#vtep-loopback-node-allocation)

## Fabric Switches and Management IP

| POD | Type | Node | Management IP | Platform | Provisioned in CloudVision | Serial Number |
| --- | ---- | ---- | ------------- | -------- | -------------------------- | ------------- |
| FABRIC | l2leaf | host01 | 172.20.20.111/24 | cEOSLab | Provisioned | - |
| FABRIC | l2leaf | host02 | 172.20.20.112/24 | cEOSLab | Provisioned | - |
| FABRIC | l2leaf | host03 | 172.20.20.113/24 | cEOSLab | Provisioned | - |
| FABRIC | l2leaf | host04 | 172.20.20.114/24 | cEOSLab | Provisioned | - |
| FABRIC | l3leaf | leaf01 | 172.20.20.101/24 | cEOSLab | Provisioned | - |
| FABRIC | l3leaf | leaf02 | 172.20.20.102/24 | cEOSLab | Provisioned | - |
| FABRIC | l3leaf | leaf03 | 172.20.20.103/24 | cEOSLab | Provisioned | - |
| FABRIC | l3leaf | leaf04 | 172.20.20.104/24 | cEOSLab | Provisioned | - |
| FABRIC | l3leaf | leaf05 | 172.20.20.105/24 | cEOSLab | Provisioned | - |
| FABRIC | l3leaf | leaf06 | 172.20.20.106/24 | cEOSLab | Provisioned | - |
| FABRIC | l3leaf | leaf07 | 172.20.20.107/24 | cEOSLab | Provisioned | - |
| FABRIC | l3leaf | leaf08 | 172.20.20.108/24 | cEOSLab | Provisioned | - |
| FABRIC | spine | spine01 | 172.20.20.201/24 | cEOSLab | Provisioned | - |
| FABRIC | spine | spine02 | 172.20.20.202/24 | cEOSLab | Provisioned | - |

> Provision status is based on Ansible inventory declaration and do not represent real status from CloudVision.

### Fabric Switches with inband Management IP

| POD | Type | Node | Management IP | Inband Interface |
| --- | ---- | ---- | ------------- | ---------------- |

## Fabric Topology

| Type | Node | Node Interface | Peer Type | Peer Node | Peer Interface |
| ---- | ---- | -------------- | --------- | ----------| -------------- |
| l2leaf | host01 | Ethernet1 | l3leaf | leaf01 | Ethernet4 |
| l2leaf | host01 | Ethernet2 | l3leaf | leaf02 | Ethernet4 |
| l2leaf | host02 | Ethernet1 | l3leaf | leaf03 | Ethernet4 |
| l2leaf | host02 | Ethernet2 | l3leaf | leaf04 | Ethernet4 |
| l2leaf | host03 | Ethernet1 | l3leaf | leaf05 | Ethernet4 |
| l2leaf | host03 | Ethernet2 | l3leaf | leaf06 | Ethernet4 |
| l2leaf | host04 | Ethernet1 | l3leaf | leaf07 | Ethernet4 |
| l2leaf | host04 | Ethernet2 | l3leaf | leaf08 | Ethernet4 |
| l3leaf | leaf01 | Ethernet1 | spine | spine01 | Ethernet1 |
| l3leaf | leaf01 | Ethernet2 | spine | spine02 | Ethernet1 |
| l3leaf | leaf01 | Ethernet5 | mlag_peer | leaf02 | Ethernet5 |
| l3leaf | leaf01 | Ethernet6 | mlag_peer | leaf02 | Ethernet6 |
| l3leaf | leaf02 | Ethernet1 | spine | spine01 | Ethernet2 |
| l3leaf | leaf02 | Ethernet2 | spine | spine02 | Ethernet2 |
| l3leaf | leaf03 | Ethernet1 | spine | spine01 | Ethernet3 |
| l3leaf | leaf03 | Ethernet2 | spine | spine02 | Ethernet3 |
| l3leaf | leaf03 | Ethernet5 | mlag_peer | leaf04 | Ethernet5 |
| l3leaf | leaf03 | Ethernet6 | mlag_peer | leaf04 | Ethernet6 |
| l3leaf | leaf04 | Ethernet1 | spine | spine01 | Ethernet4 |
| l3leaf | leaf04 | Ethernet2 | spine | spine02 | Ethernet4 |
| l3leaf | leaf05 | Ethernet1 | spine | spine01 | Ethernet5 |
| l3leaf | leaf05 | Ethernet2 | spine | spine02 | Ethernet5 |
| l3leaf | leaf05 | Ethernet5 | mlag_peer | leaf06 | Ethernet5 |
| l3leaf | leaf05 | Ethernet6 | mlag_peer | leaf06 | Ethernet6 |
| l3leaf | leaf06 | Ethernet1 | spine | spine01 | Ethernet6 |
| l3leaf | leaf06 | Ethernet2 | spine | spine02 | Ethernet6 |
| l3leaf | leaf07 | Ethernet1 | spine | spine01 | Ethernet7 |
| l3leaf | leaf07 | Ethernet2 | spine | spine02 | Ethernet7 |
| l3leaf | leaf07 | Ethernet5 | mlag_peer | leaf08 | Ethernet5 |
| l3leaf | leaf07 | Ethernet6 | mlag_peer | leaf08 | Ethernet6 |
| l3leaf | leaf08 | Ethernet1 | spine | spine01 | Ethernet8 |
| l3leaf | leaf08 | Ethernet2 | spine | spine02 | Ethernet8 |

## Fabric IP Allocation

### Fabric Point-To-Point Links

| Uplink IPv4 Pool | Available Addresses | Assigned addresses | Assigned Address % |
| ---------------- | ------------------- | ------------------ | ------------------ |

### Point-To-Point Links Node Allocation

| Node | Node Interface | Node IP Address | Peer Node | Peer Interface | Peer IP Address |
| ---- | -------------- | --------------- | --------- | -------------- | --------------- |

### Loopback Interfaces (BGP EVPN Peering)

| Loopback Pool | Available Addresses | Assigned addresses | Assigned Address % |
| ------------- | ------------------- | ------------------ | ------------------ |

### Loopback0 Interfaces Node Allocation

| POD | Node | Loopback0 |
| --- | ---- | --------- |

### VTEP Loopback VXLAN Tunnel Source Interfaces (VTEPs Only)

| VTEP Loopback Pool | Available Addresses | Assigned addresses | Assigned Address % |
| ------------------ | ------------------- | ------------------ | ------------------ |

### VTEP Loopback Node allocation

| POD | Node | Loopback1 |
| --- | ---- | --------- |
