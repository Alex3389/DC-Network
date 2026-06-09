## Лабораторная работа №3 - Underlay. ISIS

#### Цель: Настройка ISIS для Underlay сети.

#### План работы:

1. Собрать схему CLOS.
2. Распределить адресное пространство для Underlay сети
3. Настроить ISIS На устройствах (spine, leaf)
4. Проверить IP связность на все устройствах

#### Схема сети:

<img width="724" height="346" alt="Image" src="https://github.com/user-attachments/assets/1eca46cc-5f17-45a7-ac87-a73942160544" />

#### Адресное пространство:

| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| Spine-1 | Loopback0 | 10.10.1.1/32 | 
|  | Ethernet1 | 172.16.1.1/30 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.1.5/30 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.1.9/30 | Link_to_leaf-3
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
   isis enable UNDERLAY  
   isis network point-to-point  
!  
interface Ethernet2  
   description Link_to_leaf-2  
   mtu 9000  
   no switchport  
   ip address 172.16.1.5/30  
   isis enable UNDERLAY  
   isis network point-to-point  
!  
interface Ethernet3  
   description Link_to_leaf-3  
   mtu 9000  
   no switchport  
   ip address 172.16.1.9/30  
   isis enable UNDERLAY  
   isis network point-to-point  
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
   isis enable UNDERLAY  
!  
interface Management1  
!  
ip routing  
!  
router isis UNDERLAY  
   net 49.0001.0100.1000.1001.00  
   router-id ipv4 10.10.1.1  
   log-adjacency-changes  
   !  
   address-family ipv4 unicast  
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
   isis enable UNDERLAY  
   isis network point-to-point  
!  
interface Ethernet2  
   description Link_to_leaf-2  
   mtu 9000  
   no switchport  
   ip address 172.16.2.5/30  
   isis enable UNDERLAY  
   isis network point-to-point  
!  
interface Ethernet3  
   description Link_to_leaf-3  
   mtu 9000  
   no switchport  
   ip address 172.16.2.9/30  
   isis enable UNDERLAY  
   isis network point-to-point  
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
   isis enable UNDERLAY  
!  
interface Management1  
!  
ip routing  
!  
router isis UNDERLAY  
   net 49.0001.0100.1000.1002.00  
   router-id ipv4 10.10.1.2  
   log-adjacency-changes  
   !  
   address-family ipv4 unicast  
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
   isis enable UNDERLAY  
   isis network point-to-point  
!  
interface Ethernet2  
   description Link_to_spine-2  
   mtu 9000  
   no switchport  
   ip address 172.16.2.2/30  
   isis enable UNDERLAY  
   isis network point-to-point  
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
   ip address 10.10.1.11/32  
   isis enable UNDERLAY  
!  
interface Management1  
!  
ip routing  
!  
router isis UNDERLAY  
   net 49.0001.0100.1000.1011.00  
   router-id ipv4 10.10.1.11  
   log-adjacency-changes  
   !  
   address-family ipv4 unicast  
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
   isis enable UNDERLAY  
   isis network point-to-point  
!  
interface Ethernet2  
   description Link_to_spine-2  
   mtu 9000  
   no switchport  
   ip address 172.16.2.6/30  
   isis enable UNDERLAY  
   isis network point-to-point  
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
   isis enable UNDERLAY  
!  
interface Management1  
!  
ip routing  
!  
router isis UNDERLAY  
   net 49.0001.0100.1000.1012.00  
   router-id ipv4 10.10.1.12  
   log-adjacency-changes  
   !  
   address-family ipv4 unicast  
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
   isis enable UNDERLAY  
   isis network point-to-point  
!  
interface Ethernet2  
   description Link_to_spine-2  
   mtu 9000  
   no switchport  
   ip address 172.16.2.10/30  
   isis enable UNDERLAY  
   isis network point-to-point  
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
   isis enable UNDERLAY  
!  
interface Management1  
!  
ip routing  
!  
router isis UNDERLAY  
   net 49.0001.0100.1000.1013.00  
   router-id ipv4 10.10.1.13  
   log-adjacency-changes  
   !  
   address-family ipv4 unicast  
!  
end  

</details>

### Проверка таблиц маршрутизации, состояния ISIS и тестовый ping 

Spine-1:

<img width="1066" height="869" alt="Image" src="https://github.com/user-attachments/assets/0287af7a-a94f-4ff9-bf5a-60f254c025b8" />

Spine-2:

<img width="1066" height="863" alt="Image" src="https://github.com/user-attachments/assets/ac794711-d229-471a-95be-8417a97fb6fa" />

Leaf-1:

<img width="997" height="847" alt="Image" src="https://github.com/user-attachments/assets/3302e196-965d-402b-9beb-d7c0ce9b59af" />

Leaf-2:

<img width="992" height="850" alt="Image" src="https://github.com/user-attachments/assets/909d2339-fe1c-420c-be93-9b1d597ea5b0" />

Leaf-3:

<img width="1004" height="843" alt="Image" src="https://github.com/user-attachments/assets/76140977-2757-4483-9220-60adadea51bd" />
