## Лабораторная работа №1 - Проектирование адресного пространства

#### Цель: собрать схему CLOS и распределить адресное пространство.

#### План работы:

1. Собрать двухуровневую топологию CLOS
2. Распределить адресное пространство для Underlay сети

#### Схема сети:

<img width="742" height="358" alt="Image" src="https://github.com/user-attachments/assets/ec31c50f-e109-4bc9-9b98-f20dfabfcbb5" />

#### Адресное пространство:

| Host | Interface | IP/MASK | Description |
| --- | --- | --- | --- |
| Spine-1 | Loopback0 | 10.10.1.1/32 | 
|  | Ethernet1 | 172.16.1.0/31 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.1.2/31 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.1.4/31 | Link_to_leaf-3
| Spine-2 | Loopback0 | 10.10.1.2/32 | 
|  | Ethernet1 | 172.16.1.6/31 | Link_to_leaf-1 |
|  | Ethernet2 | 172.16.1.8/31 | Link_to_leaf-2 |
|  | Ethernet3 | 172.16.1.10/31 | Link_to_leaf-3 |
| Leaf-1 | Loopback0 | 10.10.1.101/32 | 
|  | Ethernet1 | 172.16.1.1/31 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.1.7/31 | Link_to_Spine-2 |
| Leaf-2 | Loopback0 | 10.10.1.102/32 | 
|  | Ethernet1 | 172.16.1.3/31 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.1.9/31 | Link_to_Spine-2 |
| Leaf-3 | Loopback0 | 10.10.1.103/32 | 
|  | Ethernet1 | 172.16.1.5/31 | Link_to_Spine-1 |
|  | Ethernet2 | 172.16.1.11/31 | Link_to_Spine-2 |

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
   no switchport  
   ip address 172.16.1.0/31  
!  
interface Ethernet2
   description Link_to_leaf-2  
   no switchport  
   ip address 172.16.1.2/31  
!  
interface Ethernet3  
   description Link_to_leaf-3  
   no switchport  
   ip address 172.16.1.4/31  
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
   no switchport  
   ip address 172.16.1.6/31  
!  
interface Ethernet2  
   description Link_to_leaf-2  
   no switchport  
   ip address 172.16.1.8/31  
!  
interface Ethernet3  
   description Link_to_leaf-3  
   no switchport  
   ip address 172.16.1.10/31  
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
   description Link_to_Spine-1  
   no switchport  
   ip address 172.16.1.1/31  
!  
interface Ethernet2  
   description Link_to_Spine-2  
   no switchport  
   ip address 172.16.1.7/31  
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
   ip address 10.10.1.101/32  
!  
interface Management1  
!  
ip routing  
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
   description Link_to_Spine-1  
   no switchport  
   ip address 172.16.1.3/31  
!  
interface Ethernet2  
   description Link_to_Spine-2  
   no switchport  
   ip address 172.16.1.9/31  
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
   ip address 10.10.1.102/32  
!  
interface Management1  
!  
ip routing  
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
   description Link_to_Spine-1  
   no switchport  
   ip address 172.16.1.5/31  
!  
interface Ethernet2  
   description Link_to_Spine-2  
   no switchport  
   ip address 172.16.1.11/31  
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
   ip address 10.10.1.103/32  
!  
interface Management1  
!  
ip routing  
!  
end             

</details>

### Проверка работы сети

#### Spine-1

<img width="754" height="658" alt="Image" src="https://github.com/user-attachments/assets/112c9388-2308-4323-8d27-9bed0bbbbfbc" />

#### Spine-2

<img width="782" height="665" alt="Image" src="https://github.com/user-attachments/assets/524f2c71-4597-432b-9326-4f0e5a790526" />
