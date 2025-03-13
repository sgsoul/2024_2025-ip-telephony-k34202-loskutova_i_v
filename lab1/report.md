University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [IP-telephony](https://github.com/itmo-ict-faculty/ip-telephony)

Year: 2025  
Group: K34202  
Author: Loskutova Irina  
Lab: Lab1  
Date of create: 13.03.2025  
Date of finished: 13.03.2025 

## Лабораторная работа №1 "Базовая настройка ip-телефонов в среде Сisco packet tracer"

### Описание
Для выполнения данной лабораторной работы собирается схема соединения. Необходимо проверить, правильно ли подключены и настроены все узлы устройств.

### Цель работы
Изучить рабочую среду Cisco Packet Tracer, ознакомить- ся с интерфейсами основных устройств, типами кабелей, научиться собирать топологию. Изучить построение сети IP-телефонии с помощью маршрутизатора, коммутатора и IP телефонов Cisco 7960 в среде Packet tracer


### Ход работы 

### Part 1

Добавим 4 коммутатора Switch-PT и 7 компьютеров с именами по умолчанию PC0-C6. Соединим устройства в сеть Ethernet.
![Снимок экрана 2025-03-11 180041](https://github.com/user-attachments/assets/b2496081-52f4-48f2-8600-a2a20fb0b552)


Выдадим IP адрес и маску сети каждому устройству. Проверим. что связь выстроена успешно
![Снимок экрана 2025-03-11 180304](https://github.com/user-attachments/assets/e394faf9-4fbc-488d-ba5b-ff6d9e314980)


### Part 2

Соберем топологию согласно рисунку схеме. 
![Снимок экрана 2025-03-13 175442](https://github.com/user-attachments/assets/0faa06ab-5657-4046-8825-330d539a6480)


В конфигурационном режиме изменим название маршрутизатора:
```
Router(config)# hostname CMERouter
```

Настроим интерфейс fa0/0 на маршрутизаторе Cisco 2811 (CMERouter):
```
CMERouter>enable
CMERouter#conf t
CMERouter(config)#interface FastEthernet0/0
CMERouter(config-if)#ip address 192.168.10.1 255.255.255.0
CMERouter(config-if)#no shut
```

Для автоматической настройки компьютеров и IP-телефонов в сети надо настроить DHCP сервер на маршрутизаторе Cisco 2811. В
конфигурационном режиме создадим пул DHCP адреса с названием ip-tel. Укажем ip адрес нужного VLAN для передач данных:
```
CMERouter(config)#ip dhcp pool ip-tel
CMERouter(dhcp-config)#network 192.168.10.0 255.255.255.0
CMERouter(dhcp-config)#default-router 192.168.10.1
```

Так же при настройке DHCP для передачи голоса необходимо включить опцию 150. Эта опция нужна для того чтобы IPтелефоны использовали настройки CallManager Express с TFTP сервера:
```
CMERouter(dhcp-config)#option 150 ip 192.168.10.1
```

Настроим CallManager Express, то есть телефонный сервис в автоматическом режиме. В данном режиме ведется диалоговый обмен сообщениями, маршрутизатор только запрашивает необходимые параметры. Для включения данного
сервиса в конфигурационном режиме необходима следующая настройка:
```
CMERouter(config)#telephony-service
CMERouter(config-telephony)#max-dn 5
CMERouter(config-telephony)#max-ephones 5
CMERouter(config-telephony)#ip source-address 192.168.10.1 port 2000
```
Задаем максимальное количество номеров, присваиваемых IPтелефоном `max-dn 5`
Задаем максимальное количество IP-телефонов `max-ephones 5`
Маршрутизатор в диалоговом режиме запросит IP адрес голосового шлюза `ip source-address 192.168.10.1 port 2000`


Автоматическое назначение внешних номеров:
```
CMERouter(config-telephony)#auto assign 1 to 1
CMERouter(config-telephony)#auto assign 2 to 2
```

Для того, что настроить интерфейса управления коммутатором в сети VLAN нужно назначить диапазоны портов:
```
SwitchA(config)#interface range fa0/1 – 5
SwitchA(config-if-range)#switchport mode access
SwitchA(config-if-range)#switchport voice vlan 1
```


Для возможности дальнейшего общения необходимо дополнительная конфигурация. Для этого создаем первую логическую
«телефонную» линию (directory number). Каждому телефону можно
определить несколько таких логический линий со своими номерами:
```
CMERouter(config)# ephone-dn 1
CMERouter(config-ephone-dn)# number 100
CMERouter(config-ephone-dn)# ex
CMERouter(config)# ephone-dn 2
CMERouter(config-ephone-dn)# number 300
```
![Снимок экрана 2025-03-13 175324](https://github.com/user-attachments/assets/190d6c06-6a1e-4bf8-9eba-42415cbef996)

Проверим корректность работы системы  - выполним звонки на внутренние номера.
![Снимок экрана 2025-03-13 175358](https://github.com/user-attachments/assets/766d82f3-a099-4435-90a7-e49dfb6c7900)

![Снимок экрана 2025-03-13 175406](https://github.com/user-attachments/assets/5c08ebf9-042c-47cf-aefa-b2f456a1d7e1)


### Вывод
В ходе данной работы была произведена настройкасети IP-телефонии с помощью маршрутизатора, коммутатора и IP телефонов Cisco 7960 в среде Packet tracer.
