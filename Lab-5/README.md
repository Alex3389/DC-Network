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
spanning-tree mode rapid-pvst
!
vlan 10,20
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
interface Vxlan1
   description VTEP
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
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
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
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
interface Vxlan1
   description VTEP
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
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
router ospf 1
   router-id 10.10.1.13
   max-lsa 12000
!
end
```
</details>

### Проверка работы VxLAN EVPN L2

#### Spine-1

<img width="878" height="635" alt="Image" src="https://github.com/user-attachments/assets/f5481bbe-8a89-4c83-9973-ccba6e7ba330" />

#### Leaf-1

<img width="1175" height="581" alt="Image" src="https://github.com/user-attachments/assets/3529b47c-24b7-4fed-b275-4748ab4757b1" />

#### Leaf-2

<img width="1172" height="559" alt="Image" src="https://github.com/user-attachments/assets/12a6e6d0-0257-4a3b-9061-59f36d1e5761" />

#### Leaf-3

<img width="1179" height="555" alt="Image" src="https://github.com/user-attachments/assets/c31ffa1d-a696-438a-ae40-c32d395df9c7" />

#### SRV-1

<img width="755" height="675" alt="Image" src="https://github.com/user-attachments/assets/dc5d27cc-5ecd-4df0-85a2-286b3091f0df" />

#### SRV-2

<img width="772" height="650" alt="Image" src="https://github.com/user-attachments/assets/b3650f1b-6406-4a56-b99a-9509fac1f126" />

#### SRV-3

<img width="766" height="646" alt="Image" src="https://github.com/user-attachments/assets/cc4c6249-9198-476b-9701-25c743bcd62d" />

#### SRV-4

<img width="736" height="641" alt="Image" src="https://github.com/user-attachments/assets/84affd16-66fc-4b25-adf9-fe9c955aa416" />

#### SRV-5

<img width="758" height="654" alt="Image" src="https://github.com/user-attachments/assets/f8011a46-9b2c-4b6c-bf28-6c8e21ca25b3" />

#### SRV-6

<img width="767" height="635" alt="Image" src="https://github.com/user-attachments/assets/efc558f2-61bc-4043-97aa-2db10cdf7fdb" />

