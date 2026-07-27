## Лабораторная работа №7 - VxLAN. VPC/MLAG

#### Цель: настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming.

#### Схема сети:

<img width="796" height="556" alt="Image" src="https://github.com/user-attachments/assets/72e09a2e-1f93-47b4-ac20-5f9d46d1bd38" />

#### Адресное пространство:

| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| Spine-1 | Loopback0 | 10.10.1.1/32 | 
|  | Ethernet1 | 172.16.1.1/30 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.1.5/30 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.1.9/30 | Link_to_leaf-3 |
| Leaf-1 | Loopback0 | 10.10.1.11/32 | 
|  | Ethernet1 | 172.16.1.2/30 | Link_to_Spine-1 |
|  | Management 1 | 192.168.0.1/30 |  |
|  | Vlan 4094 | 172.16.100.1/30 | PC-leaf-2 |
| Leaf-2 | Loopback0 | 10.10.1.12/32 | 
|  | Ethernet1 | 172.16.1.6/30 | Link_to_Spine-1 |
|  | Management 1 | 192.168.0.2/30 |  |
|  | Vlan 4094 | 172.16.100.2/30 | PC-leaf-1 |
| Leaf-3 | Loopback0 | 10.10.1.13/32 | 
|  | Ethernet1 | 172.16.1.10/30 | Link_to_Spine-1 |


#### Полная конфигурация устройств:

<details><summary>Spine-1</summary>

```
spine-1(config)#sho running-config 
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
spanning-tree mode mstp
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
   neighbor LEAFS peer group
   neighbor LEAFS remote-as 65000
   neighbor LEAFS update-source Loopback0
   neighbor LEAFS route-reflector-client
   neighbor LEAFS timers 5 10
   neighbor LEAFS send-community extended
   neighbor 10.10.1.11 peer group LEAFS
   neighbor 10.10.1.12 peer group LEAFS
   neighbor 10.10.1.13 peer group LEAFS
   !
   address-family evpn
      neighbor LEAFS activate
   !
   address-family ipv4
      no neighbor LEAFS activate
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
leaf-1(config)#sho running-config 
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
no spanning-tree vlan-id 4094
!
vlan 200
   name hosts
!
vlan 4094
   name MLAG
   trunk group MLAG
!
vrf instance MGMT
!
vrf instance ROUTEVNI
!
interface Port-Channel45
   description MLAG
   switchport mode trunk
   switchport trunk group MLAG
   spanning-tree link-type point-to-point
!
interface Port-Channel200
   description Link_to_host
   switchport trunk allowed vlan 200
   switchport mode trunk
   mlag 1
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
   description MLAG
   channel-group 45 mode active
!
interface Ethernet5
   description MLAG
   channel-group 45 mode active
!
interface Ethernet6
!
interface Ethernet7
   description Link_to_host
   switchport trunk allowed vlan 200
   channel-group 200 mode active
!
interface Ethernet8
!
interface Loopback0
   ip address 10.10.1.11/32
   ip ospf area 0.0.0.0
!
interface Management1
   vrf MGMT
   ip address 192.168.0.1/30
!
interface Vlan200
   vrf ROUTEVNI
   ip address virtual 192.168.200.1/24
!
interface Vlan4094
   description PC-leaf-2
   ip address 172.16.100.1/30
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 200 vni 10200
   vxlan vrf ROUTEVNI vni 105555
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf MGMT
ip routing vrf ROUTEVNI
!
mlag configuration
   domain-id LEAFS-1-2
   local-interface Vlan4094
   peer-address 172.16.100.2
   peer-address heartbeat 192.168.0.2 vrf MGMT
   peer-link Port-Channel45
   dual-primary detection delay 1 action errdisable all-interfaces
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   !
   vlan 200
      rd auto
      route-target both 65000:10200
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
   !
   vrf ROUTEVNI
      rd 10.10.1.11:5555
      route-target import evpn 5555:5555
      route-target export evpn 5555:5555
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
vlan 200
   name hosts
!
vlan 4094
   name MLAG
   trunk group MLAG
!
vrf instance MGMT
!
vrf instance ROUTEVNI
!
interface Port-Channel45
   description MLAG
   switchport mode trunk
   switchport trunk group MLAG
   spanning-tree link-type point-to-point
!
interface Port-Channel200
   description Link_to_host
   switchport trunk allowed vlan 200
   switchport mode trunk
   mlag 1
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
   description MLAG
   channel-group 45 mode active
!
interface Ethernet5
   description MLAG
   channel-group 45 mode active
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description Link_to_host
   switchport trunk allowed vlan 200
   channel-group 200 mode active
!
interface Ethernet9
!
interface Loopback0
   ip address 10.10.1.12/32
   ip ospf area 0.0.0.0
!
interface Management1
   vrf MGMT
   ip address 192.168.0.2/30
!
interface Vlan200
   vrf ROUTEVNI
   ip address virtual 192.168.200.1/24
!
interface Vlan4094
   description PC-leaf-1
   ip address 172.16.100.2/30
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 200 vni 10200
   vxlan vrf ROUTEVNI vni 105555
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf MGMT
ip routing vrf ROUTEVNI
!
mlag configuration
   domain-id LEAFS-1-2
   local-interface Vlan4094
   peer-address 172.16.100.1
   peer-address heartbeat 192.168.0.1 vrf MGMT
   peer-link Port-Channel45
   dual-primary detection delay 1 action errdisable all-interfaces
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   !
   vlan 200
      rd auto
      route-target both 65000:10200
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
   !
   vrf ROUTEVNI
      rd 10.10.1.12:5555
      route-target import evpn 5555:5555
      route-target export evpn 5555:5555
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
vlan 200
   name hosts
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
   description Link_to_host
   switchport trunk allowed vlan 200
   switchport mode trunk
!
interface Ethernet8
!
interface Loopback0
   ip address 10.10.1.13/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan200
   vrf ROUTEVNI
   ip address virtual 192.168.200.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 200 vni 10200
   vxlan vrf ROUTEVNI vni 105555
!
ip virtual-router mac-address 00:00:00:00:00:02
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
   vlan 200
      rd auto
      route-target both 65000:10200
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
   !
   vrf ROUTEVNI
      rd 10.10.1.13:5555
      route-target import evpn 5555:5555
      route-target export evpn 5555:5555
      redistribute connected
!
router ospf 1
   router-id 10.10.1.13
   max-lsa 12000
!
end
```

</details>

<details><summary>Host-1</summary>

```
host-1#sho running-config 
! Command: show running-config
! device: host-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname host-1
!
spanning-tree mode mstp
!
vlan 200
!
interface Port-Channel200
   description LACP
   switchport trunk allowed vlan 200
   switchport mode trunk
!
interface Ethernet1
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
   description Link_to_leaf-1
   channel-group 200 mode active
!
interface Ethernet8
   description Link_to_leaf-2
   channel-group 200 mode active
!
interface Management1
!
interface Vlan200
   ip address 192.168.200.100/24
!
no ip routing
!
end
```

</details>

<details><summary>Host-2</summary>

```
host-2#sho running-config 
! Command: show running-config
! device: host-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname host-2
!
spanning-tree mode mstp
!
vlan 200
!
interface Ethernet1
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
   description Link_to_leaf-3
   switchport trunk allowed vlan 200
   switchport mode trunk
!
interface Ethernet8
!
interface Management1
!
interface Vlan200
   ip address 192.168.200.102/24
!
ip routing
!
end
```

</details>


### Проверка работы VxLAN. MLAG

#### Leaf-1:

<img width="729" height="415" alt="Image" src="https://github.com/user-attachments/assets/8b22d6d7-d959-4fbf-adc6-528c222dad9d" />

<img width="707" height="504" alt="Image" src="https://github.com/user-attachments/assets/e672e94f-b16a-4c21-94c3-3caa7a4be177" />

#### Leaf-2:

<img width="707" height="396" alt="Image" src="https://github.com/user-attachments/assets/c795e2ab-01ee-4d98-9c18-9ee57e49aee0" />

<img width="618" height="505" alt="Image" src="https://github.com/user-attachments/assets/785c9ba1-ce9e-45a4-8483-8a4f01748b82" />

#### Leaf-3:

<img width="644" height="400" alt="Image" src="https://github.com/user-attachments/assets/940adaa4-11ba-4f57-9135-5b4022bc0030" />

#### Host-1:

<img width="883" height="650" alt="Image" src="https://github.com/user-attachments/assets/44b84289-11ad-4ec3-9991-4610469f3ce5" />

#### Host-2:

<img width="886" height="241" alt="Image" src="https://github.com/user-attachments/assets/597c4e67-d338-4864-a28e-100c9e19ab06" />

 