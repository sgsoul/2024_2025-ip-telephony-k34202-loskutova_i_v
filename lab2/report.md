University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [IP-telephony](https://github.com/itmo-ict-faculty/ip-telephony)

Year: 2025  
Group: K34202  
Author: Loskutova Irina  
Lab: Lab1  
Date of create: 13.03.2025  
Date of finished: 13.03.2025 

## Лабораторная работа №2 "Конфигурация voip в среде Сisco packet tracer"

### Описание
Для выполнения данной лабораторной работы собирается схема соединения. Необходимо проверить, правильно ли подключены все узлы устройств. Предварительно удалить все предыдущие конфигурационные файлы на маршрутизаторах Cisco 2811, на коммутаторе Cisco catalyst 3560.

### Цель работы
Иизучить построение сети IP-телефонии с помощью маршрутизатора Cisco 2811, коммутатора Cisco catalyst 3560 и IP телефонов Cisco 7960.

### Ход работы 

### Part 1

Соберём топологию согласно схеме.


![image](https://github.com/user-attachments/assets/e520842e-4bc7-4afb-b10d-411a2efec298)


Необходимо задать пароли для защиты маршрутизатора:
```
CMERouter(config)# line vty 0 4
CMERouter(config-line)# password pass
CMERouter(config-line)# login
CMERouter(config)# line console 0
CMERouter(config-line)# password pass
CMERouter(config-line)# login
```

Конфигурируем интерфейс fa0/0:
```
CMERouter> en
CMERouter#conf t
CMERouter(config)#interface FastEthernet0/0
CMERouter(config-if)#ip address 192.168.10.1 255.255.255.0
CMERouter(config-if)#no shut
```


Для автоматической настройки компьютеров и IP-телефонов в сети надо настроить DHCP сервер на маршрутизаторе Cisco 2811:
```
CMERouter(config)#ip dhcp pool ip-tel
CMERouter(dhcp-config)#network 192.168.10.0 255.255.255.0
CMERouter(dhcp-config)#default-router 192.168.10.1
CMERouter(dhcp-config)#option 150 ip 192.168.10.1
```

Настроим CallManager Express:
```
CMERouter(config)#telephony-service
CMERouter(config-telephony)#max-dn 5
CMERouter(config-telephony)#max-ephones 5
CMERouter(config-telephony)#ip source-address 192.168.10.1 port 2000
```

Настроим интерфейс управления коммутатором в сети
VLAN через назначение диапазона портов:
```
Switch(config)#interface range fa0/1-5
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport voice vlan 1
```
Для возможности дальнейшего общения необходима дополнительная конфигурация. Для этого создаем логические «телефонные» линии (directory number):
```
CMERouter(config)#ephone-dn 1
CMERouter(config-ephone-dn)#number 101
CMERouter(config)#ephone-dn 2
CMERouter(config-ephone-dn)#number 202
CMERouter(config)#ephone-dn 3
CMERouter(config-ephone-dn)#number 303
```

И наконец, проверим связность устройств: 
![image](https://github.com/user-attachments/assets/2c858c4a-e770-4c18-9454-f86c5b509c7d)



### Part 2

Соберем топологию согласно рисунку схеме. 
![image](https://github.com/user-attachments/assets/af9e43d8-834c-4dde-9023-6b3ee1d22f3a)

Создаём vlan и присваиваем им наименования:
```
Switch(config)#vlan 10
Switch(config-vlan)#name data
Switch(config)#vlan 20
Switch(config-vlan)#name ip-tel
Switch(config)#vlan 99
Switch(config-vlan)#name management
```
Настроим vlan 99:
```
Switch(config)#interface vlan 99
Switch(config-if)#ip address 192.168.99.10 255.255.255.0
Switch(config-if)#no shut
```

Задаём маршрут по умолчанию:
```
Switch(config)#ip default-gateway 192.168.99.1
Switch(config)#interface FastEthernet0/1
Switch(config-if)#switchport mode trunk
Switch(config-if)#switchport trunk native vlan 99
```
Настроим интерфейс управления коммутатором в сети VLAN через
назначение диапазона портов:
```
Switch(config)#interface range FastEthernet0/2-4
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 10
Switch(config-if-range)#switchport access vlan 20
Switch(config-if-range)#no shut
```

Включаем интерфейс FastEthernet0/0:
```
Router(config)#interface FastEthernet0/0
Router(config-if)#no shut
```
Создаем логический подынтерфейс для VLAN 10, VLAN 20, VLAN 99:
```
Router(config-if)#int fa0/0.10
Router(config-subif)#ip add 192.168.10.1 255.255.255.0
Router(config-subif)#encapsulation dot1Q 10
Router(config-subif)#ip add 192.168.10.1 255.255.255.0
Router(config-subif)#no shut
Router(config)# interface FastEthernet0/0.20
Router(config-subif)#encapsulation dot1Q 20
Router(config-subif)#ip add 192.168.20.1 255.255.255.0
Router(config-subif)#no shut
Router(config)# interface FastEthernet0/0.99
Router(config-subif)#encapsulation dot1Q 99 native
Router(config-subif)#ip add 192.168.99.1 255.255.255.0
Router(config-subif)#no sh
```

Исключаем из пула адрес интерфейса маршрутизатора и DNS-сервера:
```
Router(config)#ip dhcp excluded-address 192.168.10.1 192.168.10.9
Router(config)#ip dhcp excluded-address 192.168.20.1 192.168.20.9
```
Настраиваем DHCP сервера для передачи голоса и данных на
маршрутизаторе Cisco 2811: 
```
Router(config)#ip dhcp pool data
Router(dhcp-config)#network 192.168.10.0 255.255.255.0
Router(dhcp-config)#default-router 192.168.10.1
Router(config)#ip dhcp pool ip-tel
Router(dhcp-config)#network 192.168.20.0 255.255.255.0
Router(dhcp-config)#default-router 192.168.20.1
Router(dhcp-config)#option 150 ip 192.168.20.1
```
Настроим телефонный сервис в автоматическом режиме:
```
Router(config)#telephony-service
Router(config-telephony)#max-dn 3
Router(config-telephony)#max-ephones 3
Router(config-telephony)#ip source-address 192.168.20.1 port 2000
```
Присваиваем номера для всех IP-телефонов в сети:
```
Router(config)#ephone-dn 1
Router(config-ephone-dn)# 100
Router(config)#ephone-dn 2
Router(config-ephone-dn)# 200
Router(config)#ephone-dn 3
Router(config-ephone-dn)# 300
```
Проверим, что вся настройка выполнена корректно: 
![image](https://github.com/user-attachments/assets/285d55be-bfd4-4022-bf1e-539a68d51161)


### Вывод
В ходе данной работы была произведена настройкасети IP-телефонии с помощью маршрутизатора, коммутатора и IP телефонов Cisco 7960 в среде Packet tracer.
