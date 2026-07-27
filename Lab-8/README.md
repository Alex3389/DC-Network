## Лабораторная работа №8 - VxLAN. Routing.

#### Цель: реализовать передачу суммарных префиксов через EVPN route-type 5.

#### Схема сети:

<img width="1112" height="558" alt="Image" src="https://github.com/user-attachments/assets/10a473de-6141-491c-86a6-88e050c8d377" />

#### Было реализована полноценная MLAG пара со стороны leaf-1 и leaf-2, а также Multihoming со стороны leaf-3 и leaf-4

### Адресное пространство:

#### На Host-1 был создан дополнительынй интерфейс Loopback 1 с адресом: 8.8.8.8/32, который будет анонсирован во всю фабрику.


| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| Spine-1 | Loopback0 | 10.10.1.1/32 | 
|  | Ethernet1 | 172.16.1.1/30 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.1.5/30 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.1.9/30 | Link_to_leaf-3 |
|  | Ethernet4 | 172.16.1.13/30 | Link_to_leaf-4 |
| Spine-2 | Loopback0 | 10.10.1.2/32 | 
|  | Ethernet1 | 172.16.2.1/30 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.3.5/30 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.2.9/30 | Link_to_leaf-3 |
|  | Ethernet4 | 172.16.2.13/30 | Link_to_leaf-4 |
| Leaf-1 | Loopback0 | 10.10.1.11/32 | 
|  | Ethernet1 | 172.16.1.2/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.2/30 | Link_to_Spine-2 |
|  | Management 1 | 192.168.0.1/30 |  |
|  | Vlan 4094 | 172.16.100.1/30 | PC-leaf-2 |
|  | Vlan 30 | 10.10.30.11/24 | BGP_to_host-1 |
|  | Vlan 200 | 192.168.200.254/24 |  |
| Leaf-2 | Loopback0 | 10.10.1.12/32 | 
|  | Ethernet1 | 172.16.1.6/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.6/30 | Link_to_Spine-2 |
|  | Management 1 | 192.168.0.2/30 |  |
|  | Vlan 4094 | 172.16.100.2/30 | PC-leaf-1 |
|  | Vlan 30 | 10.10.30.12/24 | BGP_to_host-1 |
|  | Vlan 200 | 192.168.200.253/24 |  |
| Leaf-3 | Loopback0 | 10.10.1.13/32 | 
|  | Ethernet1 | 172.16.1.10/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.10/30 | Link_to_Spine-2 |
|  | Vlan 50 | 192.168.50.254/24 |  |
| Leaf-4| Loopback0 | 10.10.1.14/32 | 
|  | Ethernet1 | 172.16.1.14/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.14/30 | Link_to_Spine-2 |
|  | Vlan 50 | 192.168.50.253/24 |  |
 host-1| Loopback1 | 8.8.8.8/32 | 
|  | Vlan 30 | 10.10.30.2/24 | BGP_to_spines |
|  | Vlan 200 | 192.168.200.10/24 |  |
 host-2| Vlan 50 | 192.168.50.10/24 | 
|  |  |  |  |


### Полная конфигурация устройств:

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
   description Link_to_leaf-4
   mtu 9000
   no switchport
   ip address 172.16.1.13/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
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
   neighbor 10.10.1.14 peer group LEAFS
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

<details><summary>Spine-2</summary>

```
spine-2#sho running-config
! Command: show running-config
! device: spine-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine-2
!
spanning-tree mode mstp
!
interface Ethernet1
   description Link_to_leaf-1
   mtu 9000
   no switchport
   ip address 172.16.2.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Link_to_leaf-2
   mtu 9000
   no switchport
   ip address 172.16.2.5/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description Link_to_leaf-3
   mtu 9000
   no switchport
   ip address 172.16.2.9/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet4
   description Link_to_leaf-4
   mtu 9000
   no switchport
   ip address 172.16.2.13/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
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
   ip address 10.10.1.2/32
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
   neighbor 10.10.1.14 peer group LEAFS
   !
   address-family evpn
      neighbor LEAFS activate
   !
   address-family ipv4
      no neighbor LEAFS activate
!
router ospf 1
   router-id 10.10.1.2
   max-lsa 12000
!
end

```

</details>

<details><summary>leaf-1</summary>

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
no spanning-tree vlan-id 4094
!
vlan 30
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
vrf instance VRF1
!
interface Port-Channel45
   switchport mode trunk
   switchport trunk group MLAG
   spanning-tree link-type point-to-point
!
interface Port-Channel200
   description Link_to_hosts
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
   description Link_to_spine-2
   mtu 9000
   no switchport
   ip address 172.16.2.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
!
interface Ethernet4
   description MLAG-PEER
   channel-group 45 mode active
!
interface Ethernet5
   description MLAG-PEER
   channel-group 45 mode active
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description Link_to_hosts
   switchport mode trunk
   channel-group 200 mode active
!
interface Loopback0
   ip address 10.10.1.11/32
   ip ospf area 0.0.0.0
!
interface Management1
   vrf MGMT
   ip address 192.168.0.1/30
!
interface Vlan30
   vrf VRF1
   ip address 10.10.30.11/24
   ip virtual-router address 10.10.30.254
!
interface Vlan200
   vrf VRF1
   ip address 192.168.200.254/24
   ip virtual-router address 192.168.200.1
!
interface Vlan4094
   ip address 172.16.100.1/30
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 200 vni 10200
   vxlan vrf VRF1 vni 105555
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF1
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
   neighbor 10.10.1.2 remote-as 65000
   neighbor 10.10.1.2 update-source Loopback0
   neighbor 10.10.1.2 timers 5 10
   neighbor 10.10.1.2 send-community extended
   !
   vlan 200
      rd auto
      route-target both 200:10200
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
      neighbor 10.10.1.2 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
      no neighbor 10.10.1.2 activate
   !
   vrf VRF1
      rd 10.10.1.11:5555
      route-target import evpn 5555:5555
      route-target export evpn 5555:5555
      neighbor 10.10.30.2 remote-as 3000
      redistribute connected
      !
      address-family ipv4
         neighbor 10.10.30.2 activate
!
router ospf 1
   router-id 10.10.1.11
   max-lsa 12000
!
end


```

</details>

<details><summary>leaf-2</summary>

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
no spanning-tree vlan-id 4094
!
vlan 30
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
vrf instance VRF1
!
interface Port-Channel45
   switchport mode trunk
   switchport trunk group MLAG
   spanning-tree link-type point-to-point
!
interface Port-Channel200
   description Link_to_hosts
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
   description Link_to_spine-2
   mtu 9000
   no switchport
   ip address 172.16.2.6/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
!
interface Ethernet4
   description MLAG-PEER
   channel-group 45 mode active
!
interface Ethernet5
   description MLAG-PEER
   channel-group 45 mode active
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description Link_to_hosts
   switchport mode trunk
   channel-group 200 mode active
!
interface Loopback0
   ip address 10.10.1.12/32
   ip ospf area 0.0.0.0
!
interface Management1
   vrf MGMT
   ip address 192.168.0.2/30
!
interface Vlan30
   vrf VRF1
   ip address 10.10.30.12/24
   ip virtual-router address 10.10.30.254
!
interface Vlan200
   vrf VRF1
   ip address 192.168.200.253/24
   ip virtual-router address 192.168.200.1
!
interface Vlan4094
   ip address 172.16.100.2/30
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 200 vni 10200
   vxlan vrf VRF1 vni 105555
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF1
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
   neighbor 10.10.1.2 remote-as 65000
   neighbor 10.10.1.2 update-source Loopback0
   neighbor 10.10.1.2 timers 5 10
   neighbor 10.10.1.2 send-community extended
   !
   vlan 200
      rd auto
      route-target both 200:10200
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
      neighbor 10.10.1.2 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
      no neighbor 10.10.1.2 activate
   !
   vrf VRF1
      rd 10.10.1.12:5555
      route-target import evpn 5555:5555
      route-target export evpn 5555:5555
      neighbor 10.10.30.2 remote-as 3000
      redistribute connected
      !
      address-family ipv4
         neighbor 10.10.30.2 activate
!
router ospf 1
   router-id 10.10.1.12
   max-lsa 12000
!
end


```

</details>

<details><summary>leaf-3</summary>

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
link tracking group CORE-TRACKING
   recovery delay 1
!
hostname leaf-3
!
spanning-tree mode mstp
!
vlan 50
   name host-2
!
vrf instance VRF1
!
interface Port-Channel1
   description Link_to_host
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:0050
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:00:05
   lacp system-id 0050.1111.2222
!
interface Ethernet1
   description Link_to_spine-1
   mtu 9000
   no switchport
   ip address 172.16.1.10/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
!
interface Ethernet2
   description Link_to_spine-2
   mtu 9000
   no switchport
   ip address 172.16.2.10/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
!
interface Ethernet3
   description Link_to_host
   switchport mode trunk
   channel-group 1 mode active
   link tracking group CORE-TRACKING downstream
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
interface Ethernet9
!
interface Loopback0
   ip address 10.10.1.13/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan50
   vrf VRF1
   ip address 192.168.50.254/24
   ip virtual-router address 192.168.50.1
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 50 vni 10050
   vxlan vrf VRF1 vni 105555
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf VRF1
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   neighbor 10.10.1.2 remote-as 65000
   neighbor 10.10.1.2 update-source Loopback0
   neighbor 10.10.1.2 timers 5 10
   neighbor 10.10.1.2 send-community extended
   !
   vlan 50
      rd auto
      route-target both 50:10050
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
      neighbor 10.10.1.2 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
      no neighbor 10.10.1.2 activate
   !
   vrf VRF1
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


<details><summary>leaf-4</summary>

```

leaf-4#sho running-config
! Command: show running-config
! device: leaf-4 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
link tracking group CORE-TRACKING
   recovery delay 1
!
hostname leaf-4
!
spanning-tree mode mstp
!
vlan 50
   name host-2
!
vrf instance VRF1
!
interface Port-Channel1
   description Link_to_host
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:0050
      designated-forwarder election algorithm preference 10
      route-target import 00:00:00:00:00:05
   lacp system-id 0050.1111.2222
!
interface Ethernet1
   description Link_to_spine-1
   mtu 9000
   no switchport
   ip address 172.16.1.14/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
!
interface Ethernet2
   description Link_to_spine-2
   mtu 9000
   no switchport
   ip address 172.16.2.14/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
!
interface Ethernet3
   description Link_to_host
   switchport mode trunk
   channel-group 1 mode active
   link tracking group CORE-TRACKING downstream
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
   ip address 10.10.1.14/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan50
   vrf VRF1
   ip address 192.168.50.253/24
   ip virtual-router address 192.168.50.1
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 50 vni 10050
   vxlan vrf VRF1 vni 105555
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf VRF1
!
router bgp 65000
   maximum-paths 5
   neighbor 10.10.1.1 remote-as 65000
   neighbor 10.10.1.1 update-source Loopback0
   neighbor 10.10.1.1 timers 5 10
   neighbor 10.10.1.1 send-community extended
   neighbor 10.10.1.2 remote-as 65000
   neighbor 10.10.1.2 update-source Loopback0
   neighbor 10.10.1.2 timers 5 10
   neighbor 10.10.1.2 send-community extended
   !
   vlan 50
      rd auto
      route-target both 50:10050
      redistribute learned
   !
   address-family evpn
      neighbor 10.10.1.1 activate
      neighbor 10.10.1.2 activate
   !
   address-family ipv4
      no neighbor 10.10.1.1 activate
      no neighbor 10.10.1.2 activate
   !
   vrf VRF1
      rd 10.10.1.14:5555
      route-target import evpn 5555:5555
      route-target export evpn 5555:5555
      redistribute connected
!
router ospf 1
   router-id 10.10.1.14
   max-lsa 12000
!
end


```

</details>

<details><summary>host-1</summary>

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
service routing protocols model ribd
!
hostname host-1
!
spanning-tree mode mstp
!
vlan 30,200
!
interface Port-Channel200
   description LACP
   switchport trunk allowed vlan 30,200
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
   description Link_to_leaf-1
   channel-group 200 mode active
!
interface Ethernet7
!
interface Ethernet8
   description Link_to_leaf-2
   channel-group 200 mode active
!
interface Loopback1
   ip address 8.8.8.8/32
!
interface Management1
!
interface Vlan30
   ip address 10.10.30.2/24
!
interface Vlan200
   ip address 192.168.200.10/24
!
ip routing
!
ip route 0.0.0.0/0 192.168.200.1
!
router bgp 3000
   router-id 10.10.30.2
   maximum-paths 2 ecmp 2
   neighbor 10.10.30.11 remote-as 65000
   neighbor 10.10.30.12 remote-as 65000
   !
   address-family ipv4
      neighbor 10.10.30.11 activate
      neighbor 10.10.30.12 activate
      network 8.8.8.8/32
!
end


```

</details>

<details><summary>host-2</summary>

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
service routing protocols model ribd
!
hostname host-2
!
spanning-tree mode mstp
!
vlan 50
!
interface Port-Channel1
   switchport mode trunk
!
interface Ethernet1
   description Link_to_leaf-3
   switchport mode trunk
   channel-group 1 mode active
!
interface Ethernet2
   description Link_to_leaf-4
   switchport mode trunk
   channel-group 1 mode active
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
!
interface Ethernet8
!
interface Management1
!
interface Vlan50
   ip address 192.168.50.10/24
!
ip routing
!
ip route 0.0.0.0/0 192.168.50.1
!
end


```

</details>


## Проверка работы VxLAN. MLAG, Multihoming, Routing.

#### На host-1 был поднят протокол bgp (AS3000). Между host-1 и leaf-1, leaf-2 была поднята BGP сессия в VRF1. Cо стороны host-1 был анонсирован префикс 8.8.8.8/32

#### Leaf-1:

<img width="1092" height="1033" alt="Image" src="https://github.com/user-attachments/assets/4c1d3abb-217e-4fdc-a4e3-8de1035963d4" />

<img width="513" height="416" alt="Image" src="https://github.com/user-attachments/assets/337075d8-00aa-4ebf-a67c-69b8dd14089e" />

#### Leaf-2:

<img width="1076" height="1022" alt="Image" src="https://github.com/user-attachments/assets/3cbb2259-e714-4d2b-9e2e-0b1dfffc4f0d" />

<img width="521" height="417" alt="Image" src="https://github.com/user-attachments/assets/01a2b697-0cc0-41d2-9614-9616caad238f" />

#### Leaf-3:

<img width="1085" height="1096" alt="Image" src="https://github.com/user-attachments/assets/e25c0cbb-0b77-471b-8002-380d7ba86617" />

<img width="732" height="194" alt="Image" src="https://github.com/user-attachments/assets/c0ff3523-bb46-4e05-acab-ccf0a5c4de36" />

#### Leaf-4:

<img width="1066" height="1105" alt="Image" src="https://github.com/user-attachments/assets/f48c2192-783e-4181-be12-5dcc591fb9ac" />

<img width="661" height="180" alt="Image" src="https://github.com/user-attachments/assets/b0ae07d8-03ab-4e9b-971f-26537a1d5717" />

#### host-1:

<img width="775" height="337" alt="Image" src="https://github.com/user-attachments/assets/2095e6dd-e008-473f-a40e-6c34467d79a0" />

#### host-2:

<img width="749" height="408" alt="Image" src="https://github.com/user-attachments/assets/0451efce-4872-4a2c-af1e-866fb602cb0d" />

