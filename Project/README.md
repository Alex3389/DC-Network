## Проектная работа

# Тема: Проектирование мультиподовой сети ЦОД на базе технологии EVPN-VXLAN

## Цель проекта:

Спроектировать и собрать схему двух Центров Обработки Данных и объеденить их по технологии Multipod. Схемы должны быть спроектированы на базе архитектуры Spine-Leaf. Объединяющие устройства Border-leaf в каждом ЦОД должны быть соединены через условную сеть провайдера с помощью технологии VPLS. Обеспечить отказоустойчивое подключение клиентского оборудования (хостов) в каждом ЦОД.

## Используемое оборудование

#### POD-1 (DC-1):

1. Spine-1-DC-1 - Arista (vEOS-lab, EOS-4.29.2F)
2. Spine-2-DC-1 - Arista (vEOS-lab, EOS-4.29.2F)
3. Leaf-1-DC-1 - Arista (vEOS-lab, EOS-4.29.2F)
4. Leaf-2-DC-1 - Arista (vEOS-lab, EOS-4.29.2F)
5. Border-leaf-DC-1 - Cisco Nexus (Nexus9000 C9300v)
6. Host-1-DC-1 - Arista (vEOS-lab, EOS-4.29.2F)
7. WM-1 - VPCS
8. WM-2 - VPCS
9. WM-3 - VPCS

#### POD-2 (DC-2):

1. Spine-1-DC-2 - Arista (vEOS-lab, EOS-4.29.2F)
2. Spine-2-DC-2 - Arista (vEOS-lab, EOS-4.29.2F)
3. Leaf-1-DC-2 - Arista (vEOS-lab, EOS-4.29.2F)
4. Leaf-2-DC-2 - Arista (vEOS-lab, EOS-4.29.2F)
5. Border-leaf-DC-2 - Cisco Nexus (Nexus9000 C9300v)
6. Host-1-DC-2 - Arista (vEOS-lab, EOS-4.29.2F)
7. WM-4 - VPCS
8. WM-5 - VPCS
9. WM-6 - VPCS

#### ISP (Internet Service Provider)

1. R1-ISP - Juniper vMX (vmx-14.1R4.8-domestic)
2. R2-ISP - Juniper vMX (vmx-14.1R4.8-domestic)
3. R3-ISP - Juniper vMX (vmx-14.1R4.8-domestic)

_____

## Схема сети

<img width="1859" height="1158" alt="Image" src="https://github.com/user-attachments/assets/a64b9cdb-55e8-438e-88ab-d87a8be9cbaf" />

______

## Адресное пространство:

### POD-1 (DC-1)

| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| Spine-1-DC-1 | Loopback0 | 10.10.1.1/32 | 
|  | Ethernet1 | 172.16.1.1/30 | Link_to_leaf-1-DC-1 |
|  | Ethernet2 | 172.16.1.5/30 | Link_to_leaf-2-DC-1 |
|  | Ethernet8 | 172.16.1.9/30 | Link_to_Border-leaf-DC-1 |
| Spine-2-DC-1 | Loopback0 | 10.10.1.2/32 | 
|  | Ethernet1 | 172.16.2.1/30 | Link_to_leaf-1-DC-1 |
|  | Ethernet2 | 172.16.2.5/30 | Link_to_leaf-2-DC-1 |
|  | Ethernet8 | 172.16.2.9/30 | Link_to_Border-leaf-DC-1 |
| Leaf-1-DC-1 | Loopback0 | 10.10.1.11/32 | 
|  | Ethernet1 | 172.16.1.2/30 | Link_to_Spine-1-DC-1 |
|  | Ethernet2 | 172.16.2.2/30 | Link_to_Spine-2-DC-1 |
|  | svi 10 | 192.168.10.1/24 | |
|  | svi 20 | 192.168.20.1/24 | |
|  | svi 30 | 192.168.30.1/24 | |
| Leaf-2-DC-1 | Loopback0 | 10.10.1.12/32 | 
|  | Ethernet1 | 172.16.1.6/30 | Link_to_Spine-1-DC-1 |
|  | Ethernet2 | 172.16.2.6/30 | Link_to_Spine-2-DC-1 |
|  | svi 10 | 192.168.10.1/24 | |
|  | svi 20 | 192.168.20.1/24 | |
|  | svi 30 | 192.168.30.1/24 | |
| Border-Leaf-DC-1 | Loopback0 | 10.10.1.13/32 | 
|  | Ethernet1 | 172.16.1.10/30 | Link_to_Spine-1-DC-1 |
|  | Ethernet2 | 172.16.2.10/30 | Link_to_Spine-2-DC-1 |
|  | svi 10 | 192.168.10.1/24 | |
|  | svi 20 | 192.168.20.1/24 | |
|  | svi 30 | 192.168.30.1/24 | |
|  | svi 500 | 172.16.77.1/30 | Link_to_DC-2 |
| Host-1-DC-1 |  |  | 
|  | svi 10 | 192.168.10.150/24 | |
|  | svi 20 | 192.168.20.150/24 | |
|  | svi 30 | 192.168.30.150/24 | |




### POD-2 (DC-2)

| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| Spine-1-DC-2 | Loopback0 | 10.20.1.1/32 | 
|  | Ethernet1 | 172.20.1.1/30 | Link_to_leaf-1-DC-2 |
|  | Ethernet2 | 172.20.1.5/30 | Link_to_leaf-2-DC-2 |
|  | Ethernet8 | 172.20.1.9/30 | Link_to_Border-leaf-DC-2 |
| Spine-2-DC-2 | Loopback0 | 10.20.1.2/32 | 
|  | Ethernet1 | 172.20.2.1/30 | Link_to_leaf-1-DC-2 |
|  | Ethernet2 | 172.20.2.5/30 | Link_to_leaf-2-DC-2 |
|  | Ethernet8 | 172.20.2.9/30 | Link_to_Border-leaf-DC-2 |
| Leaf-1-DC-2 | Loopback0 | 10.10.1.11/32 | 
|  | Ethernet1 | 172.20.1.2/30 | Link_to_Spine-1-DC-2 |
|  | Ethernet2 | 172.20.2.2/30 | Link_to_Spine-2-DC-2 |
|  | svi 10 | 192.168.10.1/24 | |
|  | svi 20 | 192.168.20.1/24 | |
|  | svi 30 | 192.168.30.1/24 | |
| Leaf-2-DC-2 | Loopback0 | 10.10.1.12/32 | 
|  | Ethernet1 | 172.20.1.6/30 | Link_to_Spine-1-DC-2 |
|  | Ethernet2 | 172.20.2.6/30 | Link_to_Spine-2-DC-2 |
|  | svi 10 | 192.168.10.1/24 | |
|  | svi 20 | 192.168.20.1/24 | |
|  | svi 30 | 192.168.30.1/24 | |
| Border-Leaf-DC-2 | Loopback0 | 10.10.1.13/32 | 
|  | Ethernet1 | 172.20.1.10/30 | Link_to_Spine-1-DC-2 |
|  | Ethernet2 | 172.20.2.10/30 | Link_to_Spine-2-DC-2 |
|  | svi 10 | 192.168.10.1/24 | |
|  | svi 20 | 192.168.20.1/24 | |
|  | svi 30 | 192.168.30.1/24 | |
|  | svi 500 | 172.16.77.2/30 | Link_to_DC-1 |
| Host-1-DC-2 |  |  | 
|  | svi 10 | 192.168.10.160/24 | |
|  | svi 20 | 192.168.20.160/24 | |
|  | svi 30 | 192.168.30.160/24 | |


### ISP (Internet Service Provider)

| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| R1-ISP | Loopback0 | 10.255.1.1/32 | 
|  | ge-0/0/0.0 | 10.200.1.1/30 | Link_to_R2 |
| R2-ISP | Loopback0 | 10.255.1.2/32 | 
|  | ge-0/0/0.0 | 10.200.1.2/30 | Link_to_R1 |
|  | ge-0/0/1.0 | 10.200.2.2/30 | Link_to_R3 |
| R3-ISP | Loopback0 | 10.255.1.3/32 | 
|  | ge-0/0/1.0 | 10.200.2.1/30 | Link_to_R2 |

_____

## Описание выполненных работ и технологий


В качестве Underlay сети был выбран протокол динамической маршрутизации OSPF. В качестве Control Plane используется протокол динамической маршрутизации iBGP.   
В ходе проектирования каждому POD (DC) был присвоен свой ASN.   
POD-1 (DC-1) - AS 65000   
POD-2 (DC-2) - AS 65200   

Для решения отказоустойчивости подключения хостов, был выбран метод подключения по технологии Multihoming.   
В ходе проектирования было выделено три клиентские подсети находящиеся в одной vrf - WORK_SERVICES:   
192.168.10.0/24   
192.168.20.0/24   
192.168.30.0/24   

На условной сети провайдера, состоящей из трех маршрутизаторов Juniper, был поднят протокол динамической маршрутизации OSPF для обеспечения связности между всеми маршрутизаторами. Также былы подняты протоколы MPLS и LDP.  
Для обеспечения связности между POD (DC) был использован протокол VPLS, vpls-id 500.   
В конечном итоге на линковочные интерфейсы в сторону каждого из POD были настроены интерфейсы с тагом vlan-id 500.

На каждом Хосте (Host-1-DC-1 и Host-1-DC-2) были созданы дополнительно по 3 vlan интерфейса с соответствующими IP-адресами.

----

## Полная конфигурация устройств:

### POD-1 (DC-1):
<br>

<details><summary>Spine-1-DC-1</summary>

```
Spine-1-DC-1#sho running-config
! Command: show running-config
! device: Spine-1-DC-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Spine-1-DC-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description Link_to_leaf-1-DC-1
   mtu 9000
   no switchport
   ip address 172.16.1.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Link_to_leaf-2-DC-1
   mtu 9000
   no switchport
   ip address 172.16.1.5/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
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
   description Link_to_B-leaf-1-DC-1
   mtu 9000
   no switchport
   ip address 172.16.1.9/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet9
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

<details><summary>Spine-2-DC-1</summary>

```

spine-2-DC-1#sho running-config
! Command: show running-config
! device: spine-2-DC-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine-2-DC-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description Link_to_leaf-1-DC-1
   mtu 9000
   no switchport
   ip address 172.16.2.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Link_to_leaf-2-DC-1
   mtu 9000
   no switchport
   ip address 172.16.2.5/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
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
   description Link_to_Border-leaf-DC-1
   mtu 9000
   no switchport
   ip address 172.16.2.9/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet9
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
   router-id 10.10.1.2
   max-lsa 12000
!
end

```

</details>

<details><summary>Border-leaf-DC-1</summary>

```

Border-leaf-DC-1# sho running-config

!Command: show running-config
!Running configuration last done at: Fri Jul 31 05:37:33 2026
!Time: Mon Aug  3 05:18:06 2026

version 10.5(2) Bios:version
hostname Border-leaf-DC-1
vdc Border-leaf-DC-1 id 1
  limit-resource vlan minimum 16 maximum 4094
  limit-resource vrf minimum 2 maximum 4096
  limit-resource port-channel minimum 0 maximum 511
  limit-resource m4route-mem minimum 58 maximum 58
  limit-resource m6route-mem minimum 8 maximum 8

nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

no password strength-check
username admin password 5 $5$DJLPAC$zg7IHN7R05vI5u2WZN87SgvkGZV76q9sHvXeaVDH8AB
 role network-admin
ip domain-lookup
copp profile strict
snmp-server user admin network-admin auth md5 0176BC45C29AE8BC09CF0DD8944855DC3D
08 priv aes-128 055FE04CA4FAE8AF33995EF5D80443F9321C localizedV2key
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO
system jumbomtu 2000

fabric forwarding anycast-gateway-mac 0000.0000.0001
vlan 1,10,20,30,99,500
vlan 10
  vn-segment 10010
vlan 20
  vn-segment 10020
vlan 30
  vn-segment 10030
vlan 99
  vn-segment 10099
vlan 500
  name Link_to_AS_65200

route-map RM_NEXTHOP permit 10
  set ip next-hop unchanged
vrf context WORK_SERVICES
  vni 10099
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management


interface Vlan1

interface Vlan10
  no shutdown
  vrf member WORK_SERVICES
  ip address 192.168.10.1/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member WORK_SERVICES
  ip address 192.168.20.1/24
  fabric forwarding mode anycast-gateway

interface Vlan30
  no shutdown
  vrf member WORK_SERVICES
  ip address 192.168.30.1/24
  fabric forwarding mode anycast-gateway

interface Vlan99
  no shutdown
  vrf member WORK_SERVICES
  ip forward

interface Vlan500
  description Link_to_DC-2
  no shutdown
  ip address 172.16.77.1/30
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  global ingress-replication protocol bgp
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 10030
    ingress-replication protocol bgp
  member vni 10099 associate-vrf

interface Ethernet1/1
  description Link_to_spine-1-DC1
  no switchport
  mtu 9000
  ip address 172.16.1.10/30
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  description Link_to_spine-2-DC-1
  no switchport
  mtu 9000
  ip address 172.16.2.10/30
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description Link_to_DC-2
  switchport mode trunk
  switchport trunk allowed vlan 500
  mtu 2000

interface Ethernet1/4
  no switchport
  mtu 9000
  ip address 172.16.80.1/30
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0

interface Ethernet1/5
  
interface Ethernet1/6

interface Ethernet1/7

interface Ethernet1/8

interface Ethernet1/9

interface Ethernet1/10

interface mgmt0
  vrf member management

interface loopback0
  ip address 10.10.1.13/32
  ip router ospf 1 area 0.0.0.0
icam monitor scale

line console
line vty
boot nxos bootflash:/nxos64-cs.10.5.2.F.bin
router ospf 1
  router-id 10.10.1.13
router bgp 65000
  address-family l2vpn evpn
    nexthop route-map RM_NEXTHOP
  neighbor 10.10.1.1
    remote-as 65000
    update-source loopback0
    timers 5 10
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.10.1.2
    remote-as 65000
    update-source loopback0
    timers 5 10
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.20.1.13
    remote-as 65200
    update-source loopback0
    disable-connected-check
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map RM_NEXTHOP out
      rewrite-evpn-rt-asn

```

</details>

<details><summary>Leaf-1-DC-1</summary>

```

leaf-1-DC-1#sho running-config
! Command: show running-config
! device: leaf-1-DC-1 (vEOS-lab, EOS-4.29.2F)
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
hostname leaf-1-DC-1
!
spanning-tree mode mstp
!
vlan 10,20,30
!
vrf instance WORK_SERVICES
!
interface Port-Channel1
   description Link_to_host
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:2000
      designated-forwarder election algorithm preference 200
      route-target import 00:00:00:00:20:00
   lacp system-id 0000.1111.0100
!
interface Ethernet1
   description Link_to_spine-1
   mtu 9000
   no switchport
   ip address 172.16.1.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
!
interface Ethernet2
   description Link_to_spine-2
   mtu 9000
   no switchport
   ip address 172.16.2.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
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
   description Link_to_host
   switchport mode trunk
   channel-group 1 mode active
   link tracking group CORE-TRACKING downstream
!
interface Loopback0
   ip address 10.10.1.11/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan10
   vrf WORK_SERVICES
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf WORK_SERVICES
   ip address virtual 192.168.20.1/24
!
interface Vlan30
   vrf WORK_SERVICES
   ip address virtual 192.168.30.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf WORK_SERVICES vni 10099
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf WORK_SERVICES
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
   vlan 10
      rd auto
      route-target both 65000:10010
      redistribute learned
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
   vrf WORK_SERVICES
      rd 65000:1
      route-target import evpn 65000:10099
      route-target export evpn 65000:10099
      redistribute connected
!
router ospf 1
   router-id 10.10.1.11
   max-lsa 12000
!
end

```
</details>

<details><summary>Leaf-2-DC-1</summary>

```
leaf-2-DC-1#sho running-config
! Command: show running-config
! device: leaf-2-DC-1 (vEOS-lab, EOS-4.29.2F)
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
hostname leaf-2-DC-1
!
spanning-tree mode mstp
!
vlan 10,20,30
!
vrf instance WORK_SERVICES
!
interface Port-Channel1
   description Link_to_host
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:2000
      designated-forwarder election algorithm preference 10
      route-target import 00:00:00:00:20:00
   lacp system-id 0000.1111.0100
!
interface Ethernet1
   description Link_to_spine-1
   mtu 9000
   no switchport
   ip address 172.16.1.6/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
!
interface Ethernet2
   description Link_to_spine-2
   mtu 9000
   no switchport
   ip address 172.16.2.6/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
!
interface Ethernet3
!
interface Ethernet4
   description MLAG-PEER
!
interface Ethernet5
   description MLAG-PEER
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description Link_to_host
   switchport mode trunk
   channel-group 1 mode active
   link tracking group CORE-TRACKING downstream
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
!
interface Vlan10
   vrf WORK_SERVICES
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf WORK_SERVICES
   ip address virtual 192.168.20.1/24
!
interface Vlan30
   vrf WORK_SERVICES
   ip address virtual 192.168.30.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf WORK_SERVICES vni 10099
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf WORK_SERVICES
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
   vlan 10
      rd auto
      route-target both 65000:10010
      redistribute learned
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
   vrf WORK_SERVICES
      rd 65000:2
      route-target import evpn 65000:10099
      route-target export evpn 65000:10099
      redistribute connected
!
router ospf 1
   router-id 10.10.1.12
   max-lsa 12000
!
end

```
</details>

<details><summary>Host-1-DC-1</summary>

```
host-1-DC-1#sho running-config
! Command: show running-config
! device: host-1-DC-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname host-1-DC-1
!
spanning-tree mode mstp
!
vlan 10,20,30
!
interface Port-Channel1
   description LACP_to_leafs_1-2
   switchport trunk allowed vlan 10-30
   switchport mode trunk
!
interface Ethernet1
   description Link_to_leaf-1
   channel-group 1 mode active
!
interface Ethernet2
   description Link_to_leaf-2
   channel-group 1 mode active
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
   switchport access vlan 10
!
interface Ethernet7
   switchport access vlan 20
!
interface Ethernet8
   switchport access vlan 30
!
interface Management1
!
interface Vlan10
   ip address 192.168.10.150/24
!
interface Vlan20
   ip address 192.168.20.150/24
!
interface Vlan30
   ip address 192.168.30.150/24
!
ip routing
!
ip route 192.168.10.0/24 192.168.10.1
ip route 192.168.20.0/24 192.168.20.1
ip route 192.168.30.0/24 192.168.30.1
!
end

```
</details>
<br>

### POD-2 (DC-2):

<br>

<details><summary>Spine-1-DC-2</summary>

```
spine-1-DC-2#sho running-config
! Command: show running-config
! device: spine-1-DC-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine-1-DC-2
!
spanning-tree mode mstp
!
interface Ethernet1
   description Link_to_leaf-1-DC-2
   mtu 9000
   no switchport
   ip address 172.20.1.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Link_to_leaf-2-DC-2
   mtu 9000
   no switchport
   ip address 172.20.1.5/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
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
   description Link_to_B-leaf-1-DC-2
   mtu 9000
   no switchport
   ip address 172.20.1.9/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet9
!
interface Loopback0
   ip address 10.20.1.1/32
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router bgp 65200
   maximum-paths 5
   neighbor LEAFS-DC-2 peer group
   neighbor LEAFS-DC-2 remote-as 65200
   neighbor LEAFS-DC-2 update-source Loopback0
   neighbor LEAFS-DC-2 route-reflector-client
   neighbor LEAFS-DC-2 timers 5 10
   neighbor LEAFS-DC-2 send-community extended
   neighbor 10.20.1.11 peer group LEAFS-DC-2
   neighbor 10.20.1.12 peer group LEAFS-DC-2
   neighbor 10.20.1.13 peer group LEAFS-DC-2
   !
   address-family evpn
      neighbor LEAFS-DC-2 activate
   !
   address-family ipv4
      no neighbor LEAFS-DC-2 activate
!
router ospf 1
   router-id 10.20.1.1
   max-lsa 12000
!
end

```
</details>

<details><summary>Spine-2-DC-2</summary>

```
spine-2-DC-2#sho running-config
! Command: show running-config
! device: spine-2-DC-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine-2-DC-2
!
spanning-tree mode mstp
!
interface Ethernet1
   description Link_to_leaf-1-DC-2
   mtu 9000
   no switchport
   ip address 172.20.2.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Link_to_leaf-2-DC-2
   mtu 9000
   no switchport
   ip address 172.20.2.5/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
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
   description Link_to_B-leaf-1-DC-2
   mtu 9000
   no switchport
   ip address 172.20.2.9/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.20.1.2/32
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router bgp 65200
   maximum-paths 5
   neighbor LEAFS-DC-2 peer group
   neighbor LEAFS-DC-2 remote-as 65200
   neighbor LEAFS-DC-2 update-source Loopback0
   neighbor LEAFS-DC-2 route-reflector-client
   neighbor LEAFS-DC-2 timers 5 10
   neighbor LEAFS-DC-2 send-community extended
   neighbor 10.20.1.11 peer group LEAFS-DC-2
   neighbor 10.20.1.12 peer group LEAFS-DC-2
   neighbor 10.20.1.13 peer group LEAFS-DC-2
   !
   address-family evpn
      neighbor LEAFS-DC-2 activate
   !
   address-family ipv4
      no neighbor LEAFS-DC-2 activate
!
router ospf 1
   router-id 10.20.1.2
   max-lsa 12000
!
end

```
</details>

<details><summary>Border-leaf-DC-2</summary>

```
Border-leaf-DC-2# sho running-config

!Command: show running-config
!Running configuration last done at: Mon Aug  3 05:50:09 2026
!Time: Mon Aug  3 05:57:38 2026

version 10.5(2) Bios:version
hostname Border-leaf-DC-2
vdc Border-leaf-DC-2 id 1
  limit-resource vlan minimum 16 maximum 4094
  limit-resource vrf minimum 2 maximum 4096
  limit-resource port-channel minimum 0 maximum 511
  limit-resource m4route-mem minimum 58 maximum 58
  limit-resource m6route-mem minimum 8 maximum 8

nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

username admin password 5 $5$JDNCIP$.1vuEkbLvRbNN6ajIOEJ0Rz4pq2UwID8jwxAD8FbRh9
 role network-admin
ip domain-lookup
copp profile strict
snmp-server user admin network-admin auth md5 331836ADE536D41908DB457C4DE47CD680
10 priv aes-128 367A149D860BAF7909C0082042EF25D7841E localizedV2key
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO
system jumbomtu 2000

fabric forwarding anycast-gateway-mac 0000.0000.0001
vlan 1,10,20,30,99,500
vlan 10
  vn-segment 10010
vlan 20
  vn-segment 10020
vlan 30
  vn-segment 10030
vlan 99
  vn-segment 10099
vlan 500
  name Link_to_AS_65000

route-map RM_NEXTHOP permit 10
  set ip next-hop unchanged
vrf context WORK_SERVICES
  vni 10099
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management


interface Vlan1

interface Vlan10
  no shutdown
  vrf member WORK_SERVICES
  ip address 192.168.10.1/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member WORK_SERVICES
  ip address 192.168.20.1/24
  fabric forwarding mode anycast-gateway

interface Vlan30
  no shutdown
  vrf member WORK_SERVICES
  ip address 192.168.30.1/24
  fabric forwarding mode anycast-gateway

interface Vlan99
  no shutdown
  vrf member WORK_SERVICES
  ip forward

interface Vlan500
  description Link_to_DC-1
  no shutdown
  ip address 172.16.77.2/30
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  global ingress-replication protocol bgp
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 10030
    ingress-replication protocol bgp
  member vni 10099 associate-vrf

interface Ethernet1/1
  description Link_to_spine-1-DC-2
  no switchport
  mtu 9000
  ip address 172.20.1.10/30
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  description Link_to_spine-2-DC-2
  no switchport
  mtu 9000
  ip address 172.20.2.10/30
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description Link_to_DC-1
  switchport mode trunk
  switchport trunk allowed vlan 500
  mtu 2000

interface Ethernet1/4
  no switchport
  mtu 9000
  ip address 172.16.80.2/30
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/5

interface Ethernet1/6

interface Ethernet1/7
  switchport access vlan 20

interface Ethernet1/8

interface Ethernet1/9

interface Ethernet1/10

interface mgmt0
  vrf member management

interface loopback0
  ip address 10.20.1.13/32
  ip router ospf 1 area 0.0.0.0
icam monitor scale

line console
line vty
boot nxos bootflash:/nxos64-cs.10.5.2.F.bin
router ospf 1
  router-id 10.20.1.13
router bgp 65200
  address-family l2vpn evpn
    nexthop route-map RM_NEXTHOP
  neighbor 10.10.1.13
    remote-as 65000
    update-source loopback0
    disable-connected-check
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map RM_NEXTHOP out
      rewrite-evpn-rt-asn
  neighbor 10.20.1.1
    remote-as 65200
    update-source loopback0
    timers 5 10
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.20.1.2
    remote-as 65200
    update-source loopback0
    timers 5 10
    address-family l2vpn evpn
      send-community
      send-community extended

```
</details>

<details><summary>Leaf-1-DC-2</summary>

```
leaf-1-DC-2#sho running-config
! Command: show running-config
! device: leaf-1-DC-2 (vEOS-lab, EOS-4.29.2F)
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
hostname leaf-1-DC-2
!
spanning-tree mode mstp
!
vlan 10,20,30
!
vrf instance WORK_SERVICES
!
interface Port-Channel1
   description Link_to_host
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1000
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:10:00
   lacp system-id 0000.1111.0099
!
interface Ethernet1
   description Link_to_spine-1-DC-2
   mtu 9000
   no switchport
   ip address 172.20.1.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
!
interface Ethernet2
   description Link_to_spine-2-DC-2
   mtu 9000
   no switchport
   ip address 172.20.2.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
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
   description Link_to_host
   switchport mode trunk
   channel-group 1 mode active
   link tracking group CORE-TRACKING downstream
!
interface Ethernet9
!
interface Loopback0
   ip address 10.20.1.11/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan10
   vrf WORK_SERVICES
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf WORK_SERVICES
   ip address virtual 192.168.20.1/24
!
interface Vlan30
   vrf WORK_SERVICES
   ip address virtual 192.168.30.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf WORK_SERVICES vni 10099
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf WORK_SERVICES
!
router bgp 65200
   maximum-paths 5
   neighbor 10.20.1.1 remote-as 65200
   neighbor 10.20.1.1 update-source Loopback0
   neighbor 10.20.1.1 timers 5 10
   neighbor 10.20.1.1 send-community extended
   neighbor 10.20.1.2 remote-as 65200
   neighbor 10.20.1.2 update-source Loopback0
   neighbor 10.20.1.2 timers 5 10
   neighbor 10.20.1.2 send-community extended
   !
   vlan 10
      rd auto
      route-target both 65200:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65200:10020
      redistribute learned
   !
   vlan 30
      rd auto
      route-target both 65200:10030
   !
   address-family evpn
      neighbor 10.20.1.1 activate
      neighbor 10.20.1.2 activate
   !
   address-family ipv4
      no neighbor 10.20.1.1 activate
      no neighbor 10.20.1.2 activate
   !
   vrf WORK_SERVICES
      rd 65200:1
      route-target import evpn 65200:10099
      route-target export evpn 65200:10099
      redistribute connected
!
router ospf 1
   router-id 10.20.1.11
   max-lsa 12000
!
end

```
</details>

<details><summary>Leaf-2-DC-2</summary>

```
leaf-2-DC-2#sho running-config
! Command: show running-config
! device: leaf-2-DC-2 (vEOS-lab, EOS-4.29.2F)
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
hostname leaf-2-DC-2
!
spanning-tree mode mstp
!
vlan 10,20,30
!
vrf instance WORK_SERVICES
!
interface Port-Channel1
   description Link_to_host
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1000
      designated-forwarder election algorithm preference 10
      route-target import 00:00:00:00:10:00
   lacp system-id 0000.1111.0099
!
interface Ethernet1
   description Link_to_spine-1-DC-2
   mtu 9000
   no switchport
   ip address 172.20.1.6/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
!
interface Ethernet2
   description Link_to_spine-2-DC-2
   mtu 9000
   no switchport
   ip address 172.20.2.6/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   link tracking group CORE-TRACKING upstream
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
   description Link_to_host
   switchport mode trunk
   channel-group 1 mode active
   link tracking group CORE-TRACKING downstream
!
interface Ethernet9
!
interface Loopback0
   ip address 10.20.1.12/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan10
   vrf WORK_SERVICES
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf WORK_SERVICES
   ip address virtual 192.168.20.1/24
!
interface Vlan30
   vrf WORK_SERVICES
   ip address virtual 192.168.30.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf WORK_SERVICES vni 10099
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf WORK_SERVICES
!
router bgp 65200
   maximum-paths 5
   neighbor 10.20.1.1 remote-as 65200
   neighbor 10.20.1.1 update-source Loopback0
   neighbor 10.20.1.1 timers 5 10
   neighbor 10.20.1.1 send-community extended
   neighbor 10.20.1.2 remote-as 65200
   neighbor 10.20.1.2 update-source Loopback0
   neighbor 10.20.1.2 timers 5 10
   neighbor 10.20.1.2 send-community extended
   !
   vlan 10
      rd auto
      route-target both 65200:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65200:10020
      redistribute learned
   !
   vlan 30
      rd auto
      route-target both 65200:10030
   !
   address-family evpn
      neighbor 10.20.1.1 activate
      neighbor 10.20.1.2 activate
   !
   address-family ipv4
      no neighbor 10.20.1.1 activate
      no neighbor 10.20.1.2 activate
   !
   vrf WORK_SERVICES
      rd 65200:2
      route-target import evpn 65200:10099
      route-target export evpn 65200:10099
      redistribute connected
!
router ospf 1
   router-id 10.20.1.12
   max-lsa 12000
!
end

```
</details>

<details><summary>Host-1-DC-2</summary>

```

host-1-DC-2#sho running-config
! Command: show running-config
! device: host-1-DC-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname host-1-DC-2
!
spanning-tree mode mstp
!
vlan 10,20,30
!
interface Port-Channel1
   switchport trunk allowed vlan 10,20,30
   switchport mode trunk
!
interface Ethernet1
   description Link_to_leaf-1-DC-2
   switchport mode trunk
   channel-group 1 mode active
!
interface Ethernet2
   description Link_to_leaf-2-DC-2
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
   description Link_to_VM-4
   switchport access vlan 10
!
interface Ethernet7
   description Link_to_VM-5
   switchport access vlan 20
!
interface Ethernet8
   description Link_to_VM-6
   switchport access vlan 30
!
interface Management1
!
interface Vlan10
   ip address 192.168.10.160/24
!
interface Vlan20
   ip address 192.168.20.160/24
!
interface Vlan30
   ip address 192.168.30.160/24
!
ip routing
!
ip route 192.168.10.0/24 192.168.10.1
ip route 192.168.20.0/24 192.168.20.1
ip route 192.168.30.0/24 192.168.30.1
!
end

```
</details>
<br>

### ISP:
<br>
<details><summary>R1-ISP</summary>

```
root@R1-ISP> show configuration
## Last commit: 2026-07-31 05:26:38 UTC by root
version 14.1R4.8;
system {
    host-name R1-ISP;
    root-authentication {
        encrypted-password "$1$t1jeparh$lk3z6JazXCeEWSWRq7zOo."; ## SECRET-DATA
    }
    login {
        user admin {
            uid 2000;
            class super-user;
            authentication {
                encrypted-password "$1$u0lTVYAx$CafFTW/V5.gUY/ohikFNJ/"; ## SECRET-DATA
            }
        }
    }
    syslog {
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
}
interfaces {
    ge-0/0/0 {
        description Link_to_R2;
        mtu 2000;
        unit 0 {
            family inet {
                address 10.200.1.1/30;
            }
            family mpls;
        }
    }
    ge-0/0/3 {
        description Link_to_AS65000;
        flexible-vlan-tagging;
        mtu 2000;
        encapsulation flexible-ethernet-services;
        unit 500 {
            encapsulation vlan-vpls;
            vlan-id 500;
            family vpls;
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.255.1.1/32;
            }
            family mpls;
        }
    }
}
snmp {
    community public {
        authorization read-write;
    }
    trap-group public {
        version v2;
    }
}
protocols {
    mpls {
        interface ge-0/0/0.0;
        interface lo0.0;
    }
    ospf {
        area 0.0.0.0 {
            interface lo0.0 {
                passive;
            }
            interface ge-0/0/0.0 {
                interface-type p2p;
            }
        }
    }
    ldp {
        interface ge-0/0/0.0;
        interface lo0.0;
    }
}
routing-instances {
    AS_VPLS {
        instance-type vpls;
        interface ge-0/0/3.500;
        protocols {
            vpls {
                no-tunnel-services;
                vpls-id 500;
                neighbor 10.255.1.3;
            }
        }
    }
}

root@R1-ISP>

```

</details>

<details><summary>R2-ISP</summary>

```

root@R2-ISP> show configuration
## Last commit: 2026-07-31 05:26:22 UTC by root
version 14.1R4.8;
system {
    host-name R2-ISP;
    root-authentication {
        encrypted-password "$1$mNr1ZhDQ$CSHRpqj6znb4XVv37l3GI0"; ## SECRET-DATA
    }
    login {
        user admin {
            uid 2000;
            class super-user;
            authentication {
                encrypted-password "$1$YDglaB1a$N7WbFAgHzNQZJklZ5YAXl/"; ## SECRET-DATA
            }
        }
    }
    syslog {
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
}
interfaces {
    ge-0/0/0 {
        description Link_to_R1;
        mtu 2000;
        unit 0 {
            family inet {
                address 10.200.1.2/30;
            }
            family mpls;
        }
    }
    ge-0/0/1 {
        description Link_to_R3;
        mtu 2000;
        unit 0 {
            family inet {
                address 10.200.2.2/30;
            }
            family mpls;
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.255.1.2/32;
            }
            family mpls;
        }
    }
}
snmp {
    community public {
        authorization read-write;
    }
    trap-group public {
        version v2;
    }
}
protocols {
    mpls {
        interface ge-0/0/0.0;
        interface ge-0/0/1.0;
        interface lo0.0;
    }
    ospf {
        area 0.0.0.0 {
            interface lo0.0 {
                passive;
            }
            interface ge-0/0/0.0 {
                interface-type p2p;
            }
            interface ge-0/0/1.0 {
                interface-type p2p;
            }
        }
    }
    ldp {
        interface ge-0/0/0.0;
        interface ge-0/0/1.0;
        interface lo0.0;
    }
}

root@R2-ISP>

```

</details>

<details><summary>R3-ISP</summary>

```

root@R3-ISP> show configuration
## Last commit: 2026-07-31 05:34:36 UTC by root
version 14.1R4.8;
system {
    host-name R3-ISP;
    root-authentication {
        encrypted-password "$1$6qsDOaKi$DZ7e2EDqQUvdRQKjL29JQ/"; ## SECRET-DATA
    }
    login {
        user admin {
            uid 2000;
            class super-user;
            authentication {
                encrypted-password "$1$TxuQ8qC2$4r/w/DEBOjQNDkFMSZ5ed1"; ## SECRET-DATA
            }
        }
    }
    syslog {
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
}
interfaces {
    ge-0/0/1 {
        description Link_to_R2;
        mtu 2000;
        unit 0 {
            family inet {
                address 10.200.2.1/30;
            }
            family mpls;
        }
    }
    ge-0/0/3 {
        description Link_to_AS65200;
        flexible-vlan-tagging;
        mtu 2000;
        encapsulation flexible-ethernet-services;
        unit 500 {
            encapsulation vlan-vpls;
            vlan-id 500;
            family vpls;
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.255.1.3/32;
            }
            family mpls;
        }
    }
}
snmp {
    community public {
        authorization read-write;
    }
    trap-group public {
        version v2;
    }
}
protocols {
    mpls {
        interface ge-0/0/1.0;
        interface lo0.0;
    }
    ospf {
        area 0.0.0.0 {
            interface lo0.0 {
                passive;
            }
            interface ge-0/0/1.0 {
                interface-type p2p;
            }
        }
    }
    ldp {
        interface ge-0/0/1.0;
        interface lo0.0;
    }
}
routing-instances {
    AS_VPLS {
        instance-type vpls;
        interface ge-0/0/3.500;
        protocols {
            vpls {
                no-tunnel-services;
                vpls-id 500;
                neighbor 10.255.1.1;
            }
        }
    }
}

root@R3-ISP>

```

</details>

_____________
<br>

## Проверка связности в каждом Поде EVPN
<br>

### POD-1 (DC-1):

<img width="711" height="158" alt="Image" src="https://github.com/user-attachments/assets/1443a70f-38d4-40fc-9bef-94be88262c70" />
<b>

<img width="752" height="164" alt="Image" src="https://github.com/user-attachments/assets/ee6c3684-73be-442a-91f4-84d27b3e1abe" />
<br>
<br>

### POD-2 (DC-2):

<img width="724" height="171" alt="Image" src="https://github.com/user-attachments/assets/ed9bf09a-d7c4-4326-82f5-57ab9212a1f3" />
<br>

<img width="730" height="161" alt="Image" src="https://github.com/user-attachments/assets/f302d146-9e11-4a3d-9054-b213aea23c49" />

__________

## Проверка связности между подсетями в рамках каждого Пода

### POD-1 (DC-1):
<b>

<img width="554" height="517" alt="Image" src="https://github.com/user-attachments/assets/d4bc0d99-239d-488c-afb2-9969cd0c5c3a" />
<b>

### POD-2 (DC-2):
<b>
<img width="539" height="532" alt="Image" src="https://github.com/user-attachments/assets/c7337b87-ef86-4b8b-9f35-90a8d92d6e6b" />
<b>

------
<b>

## Проверка связности между Подами через VPLS

#### Состояние VPLS: 
<img width="720" height="539" alt="Image" src="https://github.com/user-attachments/assets/51552367-b599-4e8f-88d8-b1f92ca79a8c" />

#### ICMP Между Подами:

<img width="524" height="678" alt="Image" src="https://github.com/user-attachments/assets/0a3742f1-256e-42c6-addb-6bfde5a78a92" />


<img width="568" height="408" alt="Image" src="https://github.com/user-attachments/assets/4cdcafe4-6935-473d-98ce-9489f5be6f65" />



__________

## Отказоустойчивость подключения хостов:

#### Host-1-DC-1:
<img width="657" height="225" alt="Image" src="https://github.com/user-attachments/assets/6e32f1a2-51ef-4b88-912f-64997c556c87" />
<br>

#### Host-1-DC-2:
<img width="708" height="209" alt="Image" src="https://github.com/user-attachments/assets/be154338-30ec-4db6-bba1-27fd11f91c01" />

____________

## Вывод:

Проект был разработан и реализован согласно поставленной задачи. Связность между VM (виртуальными машинами) и Хостами как внутри одного пода так и между ними реализована. Отказоустойчивое подключение хостов по технологии Multihoming выполнено.



