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
| git status | List all new or modified files |
| git diff | Show file differences that haven't been staged |
| git status | List all new or modified files |
| git diff | Show file differences that haven't been staged |
| git status | List all new or modified files |
| git diff | Show file differences that haven't been staged |
| git status | List all new or modified files |
| git diff | Show file differences that haven't been staged |