## Лабораторная работа №5 - VxLAN. EVPN L2

#### Цель: настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

#### Схема сети:

<img width="976" height="513" alt="Image" src="https://github.com/user-attachments/assets/cce6598b-2f15-4798-9a14-70d8fc44b619" />

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
   neighbor EVPN-PEER send-community
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
spanning-tree mode rapid-pvst
!
vlan 10,20
!
vrf instance TENANT-1
!
cvx
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
   vrf TENANT-1
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf TENANT-1
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   description VTEP
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf TENANT-1 vni 10100
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf TENANT-1
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   !
   vlan-aware-bundle VLAN-AWARE
      rd 65000:1
      route-target both 65000:100
      redistribute learned
      vlan 10,20
   !
   address-family evpn
      neighbor 10.10.1.1 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
   !
   vrf TENANT-1
      rd 65000:10101
      route-target import evpn 65000:10100
      route-target export evpn 65000:10100
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
spanning-tree mode rapid-pvst
!
vlan 10,20
!
vrf instance TENANT-1
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
   vrf TENANT-1
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf TENANT-1
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf TENANT-1 vni 10100
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf TENANT-1
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   !
   vlan-aware-bundle VLAN-AWARE
      rd 65000:2
      route-target both 65000:100
      redistribute learned
      vlan 10,20
   !
   address-family evpn
      neighbor 10.10.1.1 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
   !
   vrf TENANT-1
      rd 65000:10102
      route-target import evpn 65000:10100
      route-target export evpn 65000:10100
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
spanning-tree mode rapid-pvst
!
vlan 10,20
!
vrf instance TENANT-1
!
cvx
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
   vrf TENANT-1
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf TENANT-1
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   description VTEP
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf TENANT-1 vni 10100
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf TENANT-1
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   !
   vlan-aware-bundle VLAN-AWARE
      rd 65000:3
      route-target both 65000:100
      redistribute learned
      vlan 10,20
   !
   address-family evpn
      neighbor 10.10.1.1 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
   !
   vrf TENANT-1
      rd 65000:10103
      route-target import evpn 65000:10100
      route-target export evpn 65000:10100
      redistribute connected
!
router ospf 1
   router-id 10.10.1.13
   max-lsa 12000
!
end
```
</details>

### Проверка работы VxLAN EVPN L2

#### Spine-1
<img width="1017" height="330" alt="Image" src="https://github.com/user-attachments/assets/0077bf70-c7d9-4843-bc29-553c970546bf" />

<img width="975" height="689" alt="Image" src="https://github.com/user-attachments/assets/8992071d-510c-40eb-b644-f279c1e3b5c2" />

#### Leaf-1

<img width="1167" height="901" alt="Image" src="https://github.com/user-attachments/assets/d265cc82-1ae0-4953-b0f1-8bbf7f5f5bda" />

#### Leaf-2

<img width="1169" height="864" alt="Image" src="https://github.com/user-attachments/assets/e182c59d-1cb1-4363-9b6d-3ec79ca1434c" />

#### Leaf-3

<img width="1174" height="848" alt="Image" src="https://github.com/user-attachments/assets/50773121-714b-449b-b669-6b87c06afb9e" />

#### SRV-1

<img width="685" height="291" alt="Image" src="https://github.com/user-attachments/assets/5bd7cadc-88a2-4c78-bfb8-e2de6963141e" />

#### SRV-2

<img width="667" height="288" alt="Image" src="https://github.com/user-attachments/assets/7c921d0d-0156-41ac-b5d9-28433142e701" />

#### SRV-3

<img width="666" height="284" alt="Image" src="https://github.com/user-attachments/assets/a3813222-1500-47c9-b0af-25528fe36bd6" />

#### SRV-4

<img width="664" height="287" alt="Image" src="https://github.com/user-attachments/assets/a96b86dc-f1d5-40ec-ae4f-7206b8ab06b0" />

#### SRV-5

<img width="667" height="270" alt="Image" src="https://github.com/user-attachments/assets/27502082-873f-4469-9980-c00a841bf577" />

#### SRV-6

<img width="684" height="289" alt="Image" src="https://github.com/user-attachments/assets/db7d298d-2960-4900-9dc4-cb78761c3aa1" />
