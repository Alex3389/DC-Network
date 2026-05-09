## Лабораторная работа №2 - Underlay. OSPF

#### Цель: Настройка OSPF для Underlay сети.

#### План работы:

1. Собрать схему CLOS.
2. Распределить адресное пространство для Underlay сети
3. Настроить OSPF На устройствах (spine, leaf)
4. Проверить IP связность на все устройствах

#### Схема сети:

<img width="734" height="352" alt="Image" src="https://github.com/user-attachments/assets/a9e69230-2f2c-4c2c-befa-08a56681b6fb" />

#### Адресное пространство:

| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| Spine-1 | Loopback0 | 10.10.1.1/32 | 
|  | Ethernet1 | 172.16.1.1/30 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.1.5/30 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.1.9/30 | Link_to_leaf-3
| Spine-2 | Loopback0 | 10.10.2.1/32 | 
|  | Ethernet1 | 172.16.2.1/30 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.2.5/30 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.2.9/30 | Link_to_leaf-3 |
| Leaf-1 | Loopback0 | 10.10.0.101/32 | 
|  | Ethernet1 | 172.16.1.2/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.2/30 | Link_to_Spine-2 |
| Leaf-2 | Loopback0 | 10.10.0.102/32 | 
|  | Ethernet1 | 172.16.1.6/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.6/30 | Link_to_Spine-2 |
| Leaf-3 | Loopback0 | 10.10.0.103/32 | 
|  | Ethernet1 | 172.16.1.10/30 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.2.10/30 | Link_to_Spine-2 |

#### Полная конфигурация устройств:

<details><summary>Spine-1</summary>

spine-1#sho running-config    
  
Command: show running-config  
!    
 device: spine-1 (vEOS-lab, EOS-4.29.2F)  
!  
boot system flash:/vEOS-lab.swi  
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
   mtu 9214  
   no switchport  
   ip address 172.16.1.1/30  
   ip ospf network point-to-point  
   ip ospf area 0.0.0.0  
!  
interface Ethernet2  
   description Link_to_leaf-2  
   mtu 9214  
   no switchport  
   ip address 172.16.1.5/30  
   ip ospf network point-to-point  
   ip ospf area 0.0.0.0  
!  
interface Ethernet3  
   description Link_to_leaf-3  
   mtu 9214  
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
router ospf 1  
   router-id 10.10.1.1  
   max-lsa 12000  
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
   mtu 9214  
   no switchport  
   ip address 172.16.2.1/30  
   ip ospf network point-to-point  
   ip ospf area 0.0.0.0  
!  
interface Ethernet2  
   description Link_to_leaf-2  
   mtu 9214  
   no switchport  
   ip address 172.16.2.5/30  
   ip ospf network point-to-point  
   ip ospf area 0.0.0.0  
!  
interface Ethernet3  
   description Link_to_leaf-3  
   mtu 9214  
   no switchport  
   ip address 172.16.2.9/30  
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
   ip address 10.10.2.1/32  
   ip ospf area 0.0.0.0  
!  
interface Management1  
!  
ip routing  
!  
router ospf 1  
   router-id 10.10.2.1  
   max-lsa 12000  
!  
end

</details>

<details><summary>Leaf-1</summary>

leaf-1#sho running-config   
!  
Command: show running-config  
!  
device: leaf-1 (vEOS-lab, EOS-4.29.2F)  
!  
!  
boot system flash:/vEOS-lab.swi  
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
   mtu 9214  
   no switchport  
   ip address 172.16.1.2/30  
   ip ospf network point-to-point  
   ip ospf area 0.0.0.0  
!  
interface Ethernet2  
   description Link_to_spine-2  
   mtu 9214  
   no switchport  
   ip address 172.16.2.2/30  
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
!  
interface Loopback0  
   ip address 10.10.0.101/32  
   ip ospf area 0.0.0.0  
!  
interface Management1  
!  
ip routing  
!  
router ospf 1  
   router-id 10.10.0.101  
   max-lsa 12000  
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
   mtu 9214  
   no switchport  
   ip address 172.16.1.6/30  
   ip ospf network point-to-point  
   ip ospf area 0.0.0.0  
!  
interface Ethernet2  
   description Link_to_spine-2  
   mtu 9214  
   no switchport  
   ip address 172.16.2.6/30  
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
!  
interface Loopback0  
   ip address 10.10.0.102/32  
   ip ospf area 0.0.0.0  
!  
interface Management1  
!  
ip routing  
!  
router ospf 1  
   router-id 10.10.0.102  
   max-lsa 12000  
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
   mtu 9214  
   no switchport  
   ip address 172.16.1.10/30  
   ip ospf network point-to-point  
   ip ospf area 0.0.0.0  
!  
interface Ethernet2  
   description Link_to_spine-2  
   mtu 9214  
   no switchport  
   ip address 172.16.2.10/30  
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
!  
interface Loopback0  
   ip address 10.10.0.103/32  
   ip ospf area 0.0.0.0  
!  
interface Management1  
!  
ip routing  
!  
router ospf 1  
   router-id 10.10.0.103  
   max-lsa 12000  
!  
end       

</details>

### Проверка таблицы маршрутизации и состояния OPSF 

Spine-1:

<img width="1004" height="617" alt="Image" src="https://github.com/user-attachments/assets/e031e1f5-f9db-4b0e-abf2-cb7c464041c0" />  

Spine-2:

<img width="996" height="619" alt="Image" src="https://github.com/user-attachments/assets/96f7a23e-abd3-411d-b92d-42a33479fe27" />  

Leaf-1:

<img width="998" height="621" alt="Image" src="https://github.com/user-attachments/assets/12c63d52-4805-4729-aa0b-41c77f194993" />  

Leaf-2:

<img width="1002" height="618" alt="Image" src="https://github.com/user-attachments/assets/ca3e358f-bf5b-4837-af03-c8071882d01f" />  

Leaf-3:

<img width="1004" height="618" alt="Image" src="https://github.com/user-attachments/assets/862dd6de-cab7-4732-9e1c-c45be6efad1e" />  