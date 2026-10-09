# Blue Yard Logistics — Operational Network Plan

## IP Subnet & VLAN Allocation Table

| Zone | VLAN ID | Network Subnet | Subnet Mask | Default Gateway | DNS Server |
|---|---|---|---|---|---|
| **Terminal Operations** | 10 | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.1` | `192.168.40.10` |
| **Yard Management** | 20 | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.1` | `192.168.40.10` |
| **Warehouse & Dispatch** | 30 | `192.168.30.0/24` | `255.255.255.0` | `192.168.30.1` | `192.168.40.10` |
| **Central Control Room** | 40 | `192.168.40.0/24` | `255.255.255.0` | `192.168.40.1` | `192.168.40.10` |

## Device Port & IP Assignment

| Device Name | Device Type | Connected Switch Port | VLAN | IP Address |
|---|---|---|---|---|
| **R1-Core** | Cisco 2911 Router | `Gig0/0` (Trunk to Core Switch) | 10,20,30,40 | Sub-interfaces |
| **SW-Core** | Cisco 2960 Switch | `Gig0/1` (Trunk to Router), `Fa0/1-4` | Trunk | - |
| **PC-Berth1** | End Device PC | `SW-Terminal Fa0/2` | 10 | `192.168.10.10` |
| **PC-GateIn** | End Device PC | `SW-Terminal Fa0/3` | 10 | `192.168.10.11` |
| **PC-Crane1** | End Device PC | `SW-Yard Fa0/2` | 20 | `192.168.20.10` |
| **PC-YardSlot**| End Device PC | `SW-Yard Fa0/3` | 20 | `192.168.20.11` |
| **PC-Warehouse**| End Device PC | `SW-Warehouse Fa0/2` | 30 | `192.168.30.10` |
| **Server-AppDB**| Central Server | `SW-Control Fa0/2` | 40 | `192.168.40.10` |
| **PC-Admin** | Operations PC | `SW-Control Fa0/3` | 40 | `192.168.40.50` |