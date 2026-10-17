### Labs built using:
* [PingMyNetwork Sandbox](https://pingmynetwork.com/event/sandbox-challenge)
---
# Inter-VLAN Routing using ROAS (Router On A Stick)
![TOPO](ROASv0/imgs/TOPO.png)

### 1). Topology Overview
* This simulation allows four separated VLANs talking to each other when you only have one router and one cable going to it using ROAS (Router On A Stick)
* `R1` connects to `SW1` over a single `Trunk`, and that one physical port is split into four sub-interfaces, one per VLAN. Each sub-interface is the default gateway for its subnet.
* Around that core, I added a second switch (`SW2`) behind `SW1` over another `Trunk`, and a central `DHCP` server in its own VLAN. The server hands out addresses to three PCs sitting in three different VLANs, which means the router also has to relay DHCP requests across subnets.
---
### 2). VLAN & Subnet Allocation Table

| VLAN ID | Name       | Prefix         | Default Gateway | Connected Port        |
| :------ | :--------- | :------------- | :-------------- | :-------------------- |
| 10      | Sales      | `10.1.10.0/24` | `10.1.10.1`     | `SW1` (`g0/1`) - PC1  |
| 20      | IT-Network | `10.1.20.0/24` | `10.1.20.1`     | `SW1` (`g0/2`) - PC2  |
| 30      | IT-Server  | `10.1.30.0/24` | `10.1.30.1`     | `SW2` (`g0/1`) - PC3  |
| 40      | Services   | `10.1.40.0/24` | `10.1.40.1`     | `SW1` (`f0/3`) - DHCP |

---
### 3). Device Configurations
#### i). R1
```
!
interface GigabitEthernet0/0
 no shutdown
!
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.1.10.1 255.255.255.0
 ip helper-address 10.1.40.2
 no shutdown
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.1.20.1 255.255.255.0
 ip helper-address 10.1.40.2
 no shutdown
!
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.1.30.1 255.255.255.0
 ip helper-address 10.1.40.2
 no shutdown
!
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.1.40.1 255.255.255.0
 ip helper-address 10.1.40.2
 no shutdown
```
#### ii). SW1
```
!
vlan 10
 name Sales
vlan 20
 name IT-Network
vlan 30
 name IT-Server
vlan 40
 name Services
!
interface FastEthernet0/3
 switchport host
 switchport access vlan 40
!
interface range FastEthernet0/1-2
 switchport mode trunk
 switchport nonegotiate
!
interface GigabitEthernet0/1
 switchport host
 switchport access vlan 10
!
interface GigabitEthernet0/2
 switchport host
 switchport access vlan 20
```
#### iii). SW2
```
!
vlan 30
 name IT-Server
!
interface FastEthernet0/1
 switchport mode trunk
 switchport nonegotiate
!
interface GigabitEthernet0/1
 switchport host
 switchport access vlan 30
```
#### iv). Verification
* PC1
* ![PC1](ROASv0/imgs/PC1.png)
* PC3
* ![PC3](ROASv0/imgs/PC3.png)