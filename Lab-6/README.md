## Лабораторная работа №6 - VxLAN. EVPN L3

#### Цель: настроить Overlay на основе VxLAN EVPN для L3 связанности между хостами.

#### Схема сети:

<img width="1074" height="559" alt="Image" src="https://github.com/user-attachments/assets/b4a3dc94-ef98-4de3-aea0-78b61a5b2007" />

#### Адресное пространство:

| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| Spine-1 | Loopback0 | 10.10.1.1/32 | 
|  | Ethernet1 | 172.16.1.1/30 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.1.5/30 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.1.9/30 | Link_to_leaf-3 |
| Leaf-1 | Loopback0 | 10.10.1.11/32 | 
|  | Ethernet1 | 172.16.1.2/30 | Link_to_Spine-1 |
|  | Ethernet7 | access | Link_to_SRV-1 |
|  | Ethernet8 | access | Link_to_SRV-2 |
| Leaf-2 | Loopback0 | 10.10.1.12/32 | 
|  | Ethernet1 | 172.16.1.6/30 | Link_to_Spine-1 |
|  | Ethernet7 | access | Link_to_SRV-3 |
|  | Ethernet8 | access | Link_to_SRV-4 |
| Leaf-3 | Loopback0 | 10.10.1.13/32 | 
|  | Ethernet1 | 172.16.1.10/30 | Link_to_Spine-1 |
|  | Ethernet7 | access | Link_to_SRV-5 |
|  | Ethernet8 | access | Link_to_SRV-6 |
| SRV-1 | Ethernet 0 | 192.168.10.10/24 | 
| SRV-2 | Ethernet 0 | 192.168.20.10/24 |
| SRV-3 | Ethernet 0 | 192.168.10.11/24 |
| SRV-4 | Ethernet 0 | 192.168.20.11/24 |
| SRV-5 | Ethernet 0 | 192.168.10.12/24 |
| SRV-6 | Ethernet 0 | 192.168.20.12/24 |


#### Полная конфигурация устройств:

<details><summary>Spine-1</summary>

```
spine-1#sho running-config 
! Command: show running-config
! device: spine-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine-1
!
spanning-tree mode rapid-pvst
!
interface Ethernet1
   description Link_to_leaf-1
   mtu 9000
   no switchport
   ip address 172.16.1.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Link_to_leaf-2
   mtu 9000
   no switchport
   ip address 172.16.1.5/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description Link_to_leaf-3
   mtu 9000
   no switchport
   ip address 172.16.1.9/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 10.10.1.1/32
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router bgp 65000
   maximum-paths 5
   neighbor EVPN-PEER peer group
   neighbor EVPN-PEER remote-as 65000
   neighbor EVPN-PEER update-source Loopback0
   neighbor EVPN-PEER route-reflector-client
   neighbor EVPN-PEER send-community extended
   neighbor 10.10.1.11 peer group EVPN-PEER
   neighbor 10.10.1.12 peer group EVPN-PEER
   neighbor 10.10.1.13 peer group EVPN-PEER
   !
   address-family evpn
      neighbor EVPN-PEER activate
   !
   address-family ipv4
      no neighbor EVPN-PEER activate
!
router ospf 1
   router-id 10.10.1.1
   max-lsa 12000
!
end
```

</details>

<details><summary>Leaf-1</summary>

```
leaf-1#sho running-config 
! Command: show running-config
! device: leaf-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf-1
!
spanning-tree mode mstp
!
vlan 10,20
!
vrf instance ROUTEVNI
!
interface Ethernet1
   description Link_to_spine-1
   mtu 9000
   no switchport
   ip address 172.16.1.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   switchport access vlan 10
!
interface Ethernet8
   switchport access vlan 20
!
interface Ethernet9
!
interface Loopback0
   ip address 10.10.1.11/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan10
   vrf ROUTEVNI
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf ROUTEVNI
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf ROUTEVNI vni 20000
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf ROUTEVNI
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   !
   vlan 10
      rd 10.10.1.11:10010
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd 10.10.1.11:10020
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
   !
   vrf ROUTEVNI
      rd 10.10.1.11:20000
      route-target import evpn 20000:20000
      route-target export evpn 20000:20000
      redistribute connected
!
router ospf 1
   router-id 10.10.1.11
   max-lsa 12000
!
end
```

</details>

<details><summary>Leaf-2</summary>

```
leaf-2#sho running-config 
! Command: show running-config
! device: leaf-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf-2
!
spanning-tree mode mstp
!
vlan 10,20
!
vrf instance ROUTEVNI
!
interface Ethernet1
   description Link_to_spine-1
   mtu 9000
   no switchport
   ip address 172.16.1.6/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   switchport access vlan 10
!
interface Ethernet8
   switchport access vlan 20
!
interface Loopback0
   ip address 10.10.1.12/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan10
   vrf ROUTEVNI
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf ROUTEVNI
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf ROUTEVNI vni 20000
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf ROUTEVNI
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   !
   vlan 10
      rd 10.10.1.12:10010
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd 10.10.1.12:10020
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
   !
   vrf ROUTEVNI
      rd 10.10.1.12:20000
      route-target import evpn 20000:20000
      route-target export evpn 20000:20000
      redistribute connected
!
router ospf 1
   router-id 10.10.1.12
   max-lsa 12000
!
end
```

</details>

<details><summary>Leaf-3</summary>

```
leaf-3#sho running-config 
! Command: show running-config
! device: leaf-3 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf-3
!
spanning-tree mode mstp
!
vlan 10,20
!
vrf instance ROUTEVNI
!
interface Ethernet1
   description Link_to_spine-1
   mtu 9000
   no switchport
   ip address 172.16.1.10/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   switchport access vlan 10
!
interface Ethernet8
   switchport access vlan 20
!
interface Loopback0
   ip address 10.10.1.13/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan10
   vrf ROUTEVNI
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf ROUTEVNI
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf ROUTEVNI vni 20000
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf ROUTEVNI
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   !
   vlan 10
      rd 10.10.1.13:10010
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd 10.10.1.13:10020
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
   !
   vrf ROUTEVNI
      rd 10.10.1.13:20000
      route-target import evpn 20000:20000
      route-target export evpn 20000:20000
      redistribute connected
!
router ospf 1
   router-id 10.10.1.13
   max-lsa 12000
!
end
```

</details>

### Проверка работы VxLAN EVPN L3

#### Spine-1

<img width="912" height="205" alt="Image" src="https://github.com/user-attachments/assets/04fa9297-b5e6-4ac4-af0d-31af9e238cc3" />

#### Leaf-1:

<img width="800" height="711" alt="Image" src="https://github.com/user-attachments/assets/11e75a0f-f5c8-4c8f-9234-983f388a37f4" />

<img width="1199" height="875" alt="Image" src="https://github.com/user-attachments/assets/5b472a4b-e133-4cf0-9db9-3781a0d245f9" />


#### Leaf-2:

<img width="792" height="693" alt="Image" src="https://github.com/user-attachments/assets/a8218840-64b5-45dd-b818-4ba3b3e24d2d" />

<img width="1224" height="875" alt="Image" src="https://github.com/user-attachments/assets/9b74e1cb-76b1-4daf-9166-951d883a3311" />


#### Leaf-3:

<img width="765" height="664" alt="Image" src="https://github.com/user-attachments/assets/077e58eb-cc04-40fa-8120-7b5d54009d03" />

<img width="1182" height="876" alt="Image" src="https://github.com/user-attachments/assets/8134d968-4805-423d-8f28-af504506055b" />

### Проверка связности хостов:

#### SRV-1:

<img width="764" height="673" alt="Image" src="https://github.com/user-attachments/assets/8210e5d0-0b4a-4c98-b2f7-a82bf15ad687" />

#### SRV-2:

<img width="760" height="636" alt="Image" src="https://github.com/user-attachments/assets/b4943a3e-8f32-4f67-b1f0-e995c2e41b82" />

#### SRV-3:

<img width="749" height="651" alt="Image" src="https://github.com/user-attachments/assets/b828ebaa-f059-4f3a-b6f1-b443eba19fa6" />

#### SRV-4:

<img width="765" height="678" alt="Image" src="https://github.com/user-attachments/assets/46f5c383-dda1-49c0-8459-0e63a3e83f40" />

#### SRV-5:

<img width="780" height="659" alt="Image" src="https://github.com/user-attachments/assets/d741bb73-36ab-4995-9c83-2f09ecc43d56" />

#### SRV-6:

<img width="739" height="640" alt="Image" src="https://github.com/user-attachments/assets/9249a60e-d714-40f9-ae15-2dc62c226ff6" />
