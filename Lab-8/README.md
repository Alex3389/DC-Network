## Лабораторная работа №8 - VxLAN. Routing.

#### Цель: реализовать передачу суммарных префиксов через EVPN route-type 5.

#### Схема сети:

<img width="1247" height="785" alt="Image" src="https://github.com/user-attachments/assets/3539974c-19d4-472f-bd93-ab0b59df403e" />

#### Было реализована полноценная MLAG пара со стороны leaf-1 и leaf-2, а также Multihoming со стороны leaf-3 и leaf-4

#### Leaf-1 будет выполнять роль Border-leaf. Будет установлено BGP соседство в разных vrf с маршрутизатором (Router), который будет в свою очеред маршрутизировать трафик между разными vrf.

### Адресное пространство:




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
|  | Vlan 20 | 192.168.20.11/24 |  |
|  | Vlan 30 | 192.168.30.11/24 |  |
|  | Ethernet7.200 | 10.200.1.1/30 | Link_to_Router-1_VRF20 |
|  | Ethernet7.203 | 10.203.1.1/30 | Link_to_Router-1_VRF30 |
| Leaf-2 | Loopback0 | 10.10.1.12/32 | 
|  | Ethernet1 | 172.16.1.6/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.6/30 | Link_to_Spine-2 |
|  | Management 1 | 192.168.0.2/30 |  |
|  | Vlan 4094 | 172.16.100.2/30 | PC-leaf-1 |
|  | Vlan 20 | 192.168.20.12/24 |  |
|  | Vlan 30 | 192.168.30.12/24 |  |
| Leaf-3 | Loopback0 | 10.10.1.13/32 | 
|  | Ethernet1 | 172.16.1.10/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.10/30 | Link_to_Spine-2 |
|  | Vlan 20 | 192.168.20.13/24 |  |
|  | Vlan 30 | 192.168.30.13/24 |  |
| Leaf-4| Loopback0 | 10.10.1.14/32 | 
|  | Ethernet1 | 172.16.1.14/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.14/30 | Link_to_Spine-2 |
|  | Vlan 20 | 192.168.20.14/24 |  |
|  | Vlan 30 | 192.168.30.14/24 |  |


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
vlan 20,30
!
vlan 4094
   name MLAG
   trunk group MLAG
!
vrf instance MGMT
!
vrf instance VRF20
!
vrf instance VRF30
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
   switchport access vlan 20
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
   description Link_to_Router-1
   mtu 9000
   switchport trunk allowed vlan 20,30
   switchport mode trunk
   no switchport
!
interface Ethernet7.200
   description Link_to_Router-1_VRF20
   encapsulation dot1q vlan 200
   vrf VRF20
   ip address 10.200.1.1/30
!
interface Ethernet7.203
   description Link_to_Router-1_VRF30
   encapsulation dot1q vlan 203
   vrf VRF30
   ip address 10.203.1.1/30
!
interface Ethernet8
   description Link_to_hosts
   switchport mode trunk
   channel-group 200 mode active
!
interface Ethernet9
!
interface Ethernet10
!
interface Ethernet11
!
interface Loopback0
   ip address 10.10.1.11/32
   ip ospf area 0.0.0.0
!
interface Management1
   vrf MGMT
   ip address 192.168.0.1/30
!
interface Vlan20
   vrf VRF20
   ip address 192.168.20.11/24
   ip virtual-router address 192.168.20.254
!
interface Vlan30
   vrf VRF30
   ip address 192.168.30.11/24
   ip virtual-router address 192.168.30.254
!
interface Vlan4094
   description MLAG
   ip address 172.16.100.1/30
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf VRF20 vni 102222
   vxlan vrf VRF30 vni 103333
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF20
ip routing vrf VRF30
!
ip prefix-list PL_HOSTS_NETS
   seq 10 permit 0.0.0.0/0 ge 32
!
ip prefix-list PL_LINKS_ROUTER
   seq 10 deny 10.200.1.0/30
   seq 20 deny 10.203.1.0/30
!
mlag configuration
   domain-id LEAFS-1-2
   local-interface Vlan4094
   peer-address 172.16.100.2
   peer-address heartbeat 192.168.0.2 vrf MGMT
   peer-link Port-Channel45
   dual-primary detection delay 1 action errdisable all-interfaces
!
route-map RM_HOSTS_OUT deny 10
   match ip address prefix-list PL_HOSTS_NETS
!
route-map RM_HOSTS_OUT permit 50
!
route-map RM_REDISTR permit 10
   match interface Loopback0
   set origin igp
!
route-map RM_REDISTR deny 50
!
route-map RM_ROUTER_OUT permit 10
   match ip address prefix-list PL_LINKS_ROUTER
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
   vlan 20
      rd auto
      route-target both 65000:10020
      redistribute learned
   !
   vlan 30
      rd auto
      route-target both 65000:10030
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
   vrf VRF20
      rd 10.10.1.11:2222
      route-target import evpn 65000:2222
      route-target export evpn 65000:2222
      neighbor 10.200.1.2 remote-as 65002
      neighbor 10.200.1.2 update-source Ethernet7.200
      !
      address-family ipv4
         neighbor 10.200.1.2 activate
         neighbor 10.200.1.2 route-map RM_HOSTS_OUT out
         redistribute connected route-map RM_ROUTER_OUT
   !
   vrf VRF30
      rd 10.10.1.11:3333
      route-target import evpn 65000:3333
      route-target export evpn 65000:3333
      neighbor 10.203.1.2 remote-as 65002
      neighbor 10.203.1.2 update-source Ethernet7.203
      !
      address-family ipv4
         neighbor 10.203.1.2 activate
         neighbor 10.203.1.2 route-map RM_HOSTS_OUT out
         redistribute connected route-map RM_ROUTER_OUT
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
vlan 20,30
!
vlan 4094
   name MLAG
   trunk group MLAG
!
vrf instance MGMT
!
vrf instance VRF20
!
vrf instance VRF30
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
   switchport access vlan 20
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
interface Ethernet9
!
interface Ethernet10
!
interface Ethernet11
!
interface Loopback0
   ip address 10.10.1.12/32
   ip ospf area 0.0.0.0
!
interface Management1
   vrf MGMT
   ip address 192.168.0.2/30
!
interface Vlan20
   vrf VRF20
   ip address 192.168.20.12/24
   ip virtual-router address 192.168.20.254
!
interface Vlan30
   vrf VRF30
   ip address 192.168.30.12/24
   ip virtual-router address 192.168.30.254
!
interface Vlan4094
   description MLAG
   ip address 172.16.100.2/30
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf VRF20 vni 102222
   vxlan vrf VRF30 vni 103333
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF20
ip routing vrf VRF30
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
   vlan 20
      rd auto
      route-target both 65000:10020
      redistribute learned
   !
   vlan 30
      rd auto
      route-target both 65000:10030
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
   vrf VRF20
      rd 10.10.1.12:2222
      route-target import evpn 65000:2222
      route-target export evpn 65000:2222
      redistribute connected
   !
   vrf VRF30
      rd 10.10.1.12:3333
      route-target import evpn 65000:3333
      route-target export evpn 65000:3333
      !
      address-family ipv4
         redistribute connected
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
vlan 20,30
!
vrf instance VRF1
!
vrf instance VRF20
!
vrf instance VRF30
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
   switchport access vlan 20
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
interface Vlan20
   vrf VRF20
   ip address 192.168.20.13/24
   ip virtual-router address 192.168.20.254
!
interface Vlan30
   vrf VRF30
   ip address 192.168.30.13/24
   ip virtual-router address 192.168.30.254
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf VRF20 vni 102222
   vxlan vrf VRF30 vni 103333
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
no ip routing vrf VRF1
ip routing vrf VRF20
ip routing vrf VRF30
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
   vlan 20
      rd auto
      route-target both 65000:10020
      redistribute learned
   !
   vlan 30
      rd auto
      route-target both 65000:10030
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
   vrf VRF20
      rd 10.10.1.13:2222
      route-target import evpn 65000:2222
      route-target export evpn 65000:2222
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF30
      rd 10.10.1.13:3333
      route-target import evpn 65000:3333
      route-target export evpn 65000:3333
      !
      address-family ipv4
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
vlan 20,30
!

vrf instance VRF1
!
vrf instance VRF20
!
vrf instance VRF30
!
interface Port-Channel1
   description Link_to_host
   switchport trunk allowed vlan 20,30
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
interface Ethernet9
!
interface Loopback0
   ip address 10.10.1.14/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan20
   vrf VRF20
   ip address 192.168.20.14/24
   ip virtual-router address 192.168.20.254
!
interface Vlan30
   vrf VRF30
   ip address 192.168.30.14/24
   ip virtual-router address 192.168.20.254
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf VRF20 vni 102222
   vxlan vrf VRF30 vni 103333
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf VRF1
ip routing vrf VRF20
ip routing vrf VRF30
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
   vlan 20
      rd auto
      route-target both 65000:10020
      redistribute learned
   !
   vlan 30
      rd auto
      route-target both 65000:10030
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
   vrf VRF20
      rd 10.10.1.14:2222
      route-target import evpn 65000:2222
      route-target export evpn 65000:2222
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF30
      rd 10.10.1.14:3333
      route-target import evpn 65000:3333
      route-target export evpn 65000:3333
      !
      address-family ipv4
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
vlan 20,30
!
interface Port-Channel200
   description LACP
   switchport trunk allowed vlan 20,30
   switchport mode trunk
!
interface Ethernet1
   switchport access vlan 20
!
interface Ethernet2
   switchport access vlan 20
!
interface Ethernet3
   switchport access vlan 30
!
interface Ethernet4
   switchport access vlan 30
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
interface Ethernet9
!
interface Management1
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
vlan 20,30
!
interface Port-Channel1
   switchport trunk allowed vlan 20,30
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
   switchport access vlan 20
!
interface Ethernet8
   switchport access vlan 30
!
interface Management1
!
end


```

</details>

<details><summary>Router-1</summary>

```

Router-1#sho running-config
! Command: show running-config
! device: Router-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname Router-1
!
spanning-tree mode mstp
!
vlan 20,30
!
interface Ethernet1
   description Link_to_leaf-1
   no switchport
!
interface Ethernet1.200
   description VRF20_ROUTE
   encapsulation dot1q vlan 200
   ip address 10.200.1.2/30
!
interface Ethernet1.203
   description VRF30_ROUTE
   encapsulation dot1q vlan 203
   ip address 10.203.1.2/30
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
!
interface Ethernet8
!
interface Management1
!
ip routing
!
router bgp 65002
   no bgp default ipv4-unicast
   neighbor 10.200.1.1 remote-as 65000
   neighbor 10.203.1.1 remote-as 65000
   !
   address-family ipv4
      neighbor 10.200.1.1 activate
      neighbor 10.200.1.1 default-originate always
      neighbor 10.203.1.1 activate
      neighbor 10.203.1.1 default-originate always
!
end

```
</details>


## Проверка работы VxLAN. MLAG, Multihoming, Routing.

#### Leaf-1:

<img width="979" height="585" alt="Image" src="https://github.com/user-attachments/assets/22960a68-2b30-49e5-8695-901d3cd159df" />

<img width="1043" height="580" alt="Image" src="https://github.com/user-attachments/assets/eb37300b-5178-4542-a00b-98d6c543d450" />

#### Leaf-2:

<img width="1024" height="603" alt="Image" src="https://github.com/user-attachments/assets/67e9298d-418b-415c-9756-e137f6447b92" />

<img width="1087" height="591" alt="Image" src="https://github.com/user-attachments/assets/3b65aad7-3880-47c4-a9c6-c03a7cf50037" />

#### Leaf-3:

<img width="1066" height="584" alt="Image" src="https://github.com/user-attachments/assets/87631957-0a7c-4312-8a18-0cc36d8ba11b" />

<img width="1111" height="558" alt="Image" src="https://github.com/user-attachments/assets/9c610f18-775e-4cf5-91cb-1289df2b0d39" />

#### Leaf-4:

<img width="992" height="576" alt="Image" src="https://github.com/user-attachments/assets/a74ed72b-543e-40fd-913a-84e120bb2742" />

<img width="1102" height="549" alt="Image" src="https://github.com/user-attachments/assets/fa52e683-706a-42ee-8a12-3f9b821723e8" />

#### Router-1:

<img width="594" height="370" alt="Image" src="https://github.com/user-attachments/assets/aba90019-d7a6-40f9-9858-adda20f4f5db" />

#### VM-1:

<img width="569" height="362" alt="Image" src="https://github.com/user-attachments/assets/23c65eb9-1d5b-4355-93ee-c0cde4fe6949" />

#### VM-2:

<img width="535" height="335" alt="Image" src="https://github.com/user-attachments/assets/345b75be-72a9-4dde-abce-25937e6d1e74" />

#### VM-3:

<img width="578" height="370" alt="Image" src="https://github.com/user-attachments/assets/2d54133d-e272-45c7-90cd-b5a3574415b7" />

#### VM-4:

<img width="564" height="379" alt="Image" src="https://github.com/user-attachments/assets/94eb0755-4a34-4369-b5b7-52201058b618" />