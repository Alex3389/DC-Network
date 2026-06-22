## Лабораторная работа №4 - Underlay. eBGP

#### Цель: Настройка eBGP для Underlay сети.

#### План работы:

1. Собрать схему CLOS.
2. Распределить адресное пространство для Underlay сети
3. Настроить eBGP На устройствах (spine, leaf)
4. Проверить IP связность на все устройствах

#### Схема сети:

<img width="719" height="403" alt="Image" src="https://github.com/user-attachments/assets/1923d732-bcbf-4deb-86b2-16fb5a4d3740" />

#### Адресное пространство:

| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| Spine-1 | Loopback0 | 10.10.1.1/32 | 
|  | Ethernet1 | 172.16.1.1/30 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.1.5/30 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.1.9/30 | Link_to_leaf-3 |
| Spine-2 | Loopback0 | 10.10.1.2/32 | 
|  | Ethernet1 | 172.16.2.1/30 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.2.5/30 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.2.9/30 | Link_to_leaf-3 |
| Leaf-1 | Loopback0 | 10.10.1.11/32 | 
|  | Ethernet1 | 172.16.1.2/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.2/30 | Link_to_Spine-2 |
| Leaf-2 | Loopback0 | 10.10.1.12/32 | 
|  | Ethernet1 | 172.16.1.6/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.6/30 | Link_to_Spine-2 |
| Leaf-3 | Loopback0 | 10.10.1.13/32 | 
|  | Ethernet1 | 172.16.1.10/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.10/30 | Link_to_Spine-2 |

#### Полная конфигурация устройств:

<details><summary>Spine-1</summary>

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
service routing protocols model ribd  
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
!  
interface Ethernet2  
   description Link_to_leaf-2  
   mtu 9000  
   no switchport  
   ip address 172.16.1.5/30  
!  
interface Ethernet3  
   description Link_to_leaf-3  
   mtu 9000  
   no switchport  
   ip address 172.16.1.9/30  
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
!  
interface Management1  
!  
ip routing  
!  
route-map RM_REDISTR_Lo0 permit 10  
   match interface Loopback0  
!  
peer-filter PF_AS_RANGE  
   10 match as-range 65111-65113 result accept  
!  
router bgp 65100  
   router-id 10.10.1.1  
   maximum-paths 4 ecmp 4  
   bgp listen range 172.16.0.0/22 peer-group LEAFS peer-filter PF_AS_RANGE  
   neighbor LEAFS peer group  
   neighbor LEAFS bfd  
   neighbor LEAFS timers 5 10  
   !  
   address-family ipv4  
      neighbor LEAFS activate  
      network 10.10.1.1/32  
      network 172.16.1.0/30  
      network 172.16.1.4/30  
      network 172.16.1.8/30  
!  
end  

</details>

<details><summary>Spine-2</summary>

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
service routing protocols model ribd  
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
!  
interface Ethernet2  
   description Link_to_leaf-2  
   mtu 9000  
   no switchport  
   ip address 172.16.2.5/30  
!  
interface Ethernet3  
   description Link_to_leaf-3  
   mtu 9000  
   no switchport  
   ip address 172.16.2.9/30  
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
   ip address 10.10.1.2/32  
!  
interface Management1  
!  
ip routing  
!  
route-map RM_REDISTR_Lo0 permit 10  
   match interface Loopback0  
!  
peer-filter PF_AS_RANGE  
   10 match as-range 65111-65113 result accept  
!  
router bgp 65100  
   router-id 10.10.1.2  
   maximum-paths 4 ecmp 4  
   bgp listen range 172.16.0.0/22 peer-group LEAFS peer-filter PF_AS_RANGE  
   neighbor LEAFS peer group  
   neighbor LEAFS bfd  
   neighbor LEAFS timers 5 10  
   !  
   address-family ipv4  
      neighbor LEAFS activate  
      network 10.10.1.2/32  
      network 172.16.2.0/30  
      network 172.16.2.4/30  
      network 172.16.2.8/30  
!  
end  

</details>

<details><summary>Leaf-1</summary>

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
service routing protocols model ribd  
!  
hostname leaf-1  
!  
spanning-tree mode mstp  
!  
interface Ethernet1  
   description Link_to_spine-1  
   mtu 9000  
   no switchport  
   ip address 172.16.1.2/30  
!  
interface Ethernet2  
   description Link_to_spine-2  
   mtu 9000  
   no switchport  
   ip address 172.16.2.2/30  
!  
interface Ethernet3  
   description TEST_Net  
   mtu 9000  
   no switchport  
   ip address 192.168.1.1/24  
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
   ip address 10.10.1.11/32  
!  
interface Management1  
!  
ip routing  
!  
route-map RM_REDISTR_Lo0 permit 10  
   match interface Loopback0  
!  
router bgp 65111  
   router-id 10.10.1.11  
   maximum-paths 4 ecmp 4  
   neighbor SPINES peer group  
   neighbor SPINES remote-as 65100  
   neighbor SPINES bfd  
   neighbor SPINES timers 5 10  
   neighbor 172.16.1.1 peer group SPINES  
   neighbor 172.16.2.1 peer group SPINES  
   !  
   address-family ipv4  
      neighbor SPINES activate  
      network 10.10.1.11/32  
      network 192.168.1.0/24  
!  
end  

</details>

<details><summary>Leaf-2</summary>

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
service routing protocols model ribd  
!  
hostname leaf-2  
!  
spanning-tree mode mstp  
!  
interface Ethernet1  
   description Link_to_spine-1  
   mtu 9000  
   no switchport  
   ip address 172.16.1.6/30  
!  
interface Ethernet2  
   description Link_to_spine-2  
   mtu 9000  
   no switchport  
   ip address 172.16.2.6/30  
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
interface Loopback0  
   ip address 10.10.1.12/32  
!  
interface Management1  
!  
ip routing  
!  
route-map RM_REDISTR_Lo0 permit 10  
   match interface Loopback0  
!  
router bgp 65112  
   router-id 10.10.1.12  
   maximum-paths 4 ecmp 4  
   neighbor SPINES peer group  
   neighbor SPINES remote-as 65100  
   neighbor SPINES bfd  
   neighbor SPINES timers 5 10  
   neighbor 172.16.1.5 peer group SPINES  
   neighbor 172.16.2.5 peer group SPINES  
   !  
   address-family ipv4  
      neighbor SPINES activate  
      network 10.10.1.12/32  
!  
end  

</details>

<details><summary>Leaf-3</summary>

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
service routing protocols model ribd  
!  
hostname leaf-3  
!  
spanning-tree mode mstp  
!  
interface Ethernet1  
   description Link_to_spine-1  
   mtu 9000  
   no switchport  
   ip address 172.16.1.10/30  
!  
interface Ethernet2  
   description Link_to_spine-2  
   mtu 9000  
   no switchport  
   ip address 172.16.2.10/30  
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
interface Loopback0  
   ip address 10.10.1.13/32  
!  
interface Management1  
!  
ip routing  
!  
router bgp 65113  
   router-id 10.10.1.13  
   maximum-paths 4 ecmp 4  
   neighbor SPINES peer group  
   neighbor SPINES remote-as 65100  
   neighbor SPINES bfd  
   neighbor SPINES timers 5 10  
   neighbor 172.16.1.9 peer group SPINES  
   neighbor 172.16.2.9 peer group SPINES  
   !  
   address-family ipv4  
      neighbor SPINES activate  
      network 10.10.1.13/32  
!  
end  

</details>

### Проверка таблиц маршрутизации, состояния eBGP

Spine-1:



Spine-2:



Leaf-1:



Leaf-2:



Leaf-3:

