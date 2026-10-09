# Cisco Packet Tracer CLI Commands — Blue Yard Logistics

---

## 1. ROUTER CONFIGURATION (Router-on-a-Stick Inter-VLAN Routing)

Router CLI me jayein aur yeh paste karein:

```cisco
enable
configure terminal
hostname R1-Core-Router

! Enable Physical Interface
interface GigabitEthernet0/0
 no ip address
 no shutdown
exit

! VLAN 10 Sub-Interface (Terminal)
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 description Terminal_Operations_Gateway
exit

! VLAN 20 Sub-Interface (Yard)
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 description Yard_Management_Gateway
exit

! VLAN 30 Sub-Interface (Warehouse)
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 description Warehouse_Dispatch_Gateway
exit

! VLAN 40 Sub-Interface (Control Room & App/DB Server)
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
 description Control_Room_Gateway
exit

! Save Config
do write memory