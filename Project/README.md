# Проектирование сетевой фабрики на основне VxLAN EVPN.

## Цель проекта:

Спроектировать и собрать схему двух Центров Обработки Данных и объеденить их по технологии Multipod. Схемы должны быть спроектированы на базе архитектуры Spine-Leaf. Объединяющие устройства Border-leaf в каждом ЦОД должны быть соединены через условную сеть провайдера с помощью технологии VPLS. Обеспечить отказоустойчивое подключение клиентского оборудования (хостов) в каждом ЦОД.

## Используемое оборудование и вендоров

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

----

## Полная конфигурация устройств:

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



```

</details>