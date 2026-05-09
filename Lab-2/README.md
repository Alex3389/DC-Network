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

