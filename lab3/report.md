University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [IP-telephony](https://github.com/itmo-ict-faculty/ip-telephony)

Year: 2025  
Group: K34202  
Author: Loskutova Irina  
Lab: Lab3  
Date of create: 15.03.2025  
Date of finished: 15.03.2025 

## Лабораторная работа №3 " Использование Asterisk в качестве SIP proxy"

### Описание
Для выполнения данной лабораторной работы необходимо выполнить настройку Asterisk.

### Цель работы
Изучить программный комплекс Asterisk. Настройка Asterisk для локальных звонков.

### Правила по оформлению
Правила по оформлению отчета по лабораторной работе вы можете изучить по [ссылке](../reportdesign.md)

### Ход работы
1. Установить систему server.
2. Установить Asterisk.
3. Установить soft телефон на рабочую станцию.
4. Настроить SIP каналы.
5. Подключиться к SIP каналам soft телефона.
5. Сделать тестовый звонок на номер 1000

### Ход работы 


#### Устанавка Asterisk  

Обновляем систему и ставим зависимости:  
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential libssl-dev libncurses5-dev \
    libnewt-dev libxml2-dev libsqlite3-dev uuid-dev libjansson-dev \
    libpjproject-dev wget curl git
```

Скачиваем и запускаем Asterisk:  
```
cd /usr/src
sudo wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz
sudo tar -xvf asterisk-20-current.tar.gz
cd asterisk-20.*
sudo ./configure
sudo make -j$(nproc)
sudo make install
sudo make samples
sudo make config
sudo ldconfig

sudo systemctl enable asterisk
sudo systemctl start asterisk
```
![image](https://github.com/user-attachments/assets/0bab2b58-912b-4099-a9f4-74a10b9a21de)

Проверяем, что Asterisk работает:  
```
sudo asterisk -rvvv
```

![image](https://github.com/user-attachments/assets/6ff31d86-1d22-4a36-91c6-9a520c1f562d)

#### Настройка SIP

Редактируем sip.conf:  
`/etc/asterisk/sip.conf`

``` 
[general]
bindport=5060
bindaddr=0.0.0.0
context=default

[1000]
type=friend
username=1000
secret=1000
host=dynamic
context=internal
disallow=all
allow=ulaw
allow=alaw
Сохраняем (Ctrl+X → Y → Enter).
```

#### Настройка маршрутизации вызовов
`/etc/asterisk/extensions.conf`

``` 
[internal]
exten => 1000,1,Dial(SIP/1000)
```

Перезапускаем Asterisk
```
sudo systemctl restart asterisk
```

Проверяем SIP-конфигурацию:  
```
sudo asterisk -rx "sip show peers"
```

![image](https://github.com/user-attachments/assets/939f0999-24b7-46ce-8deb-c8be733e2a76)


#### Настройка Soft-фон
Установим Zoiper и добавим учетную запись:

   - SIP-сервер: IP-адрес сервера 
   - Логин: 1000
   - Пароль: 1000
   - Протокол: UDP
   - Порт: 5060  

![image](https://github.com/user-attachments/assets/c2b909b0-8c39-44c3-ae7d-ac1e500a2687)

![image](https://github.com/user-attachments/assets/c76e43b3-848e-4952-aa2b-276cd2d23a4c)


Теперь попробуем позвонить на 1000 – всё настроено верно, звонок прошел. 
![image](https://github.com/user-attachments/assets/95efbba0-c921-41e4-a014-e67ba16113b6)

Проверим, что SIP-конфигурация обновилась:

![image](https://github.com/user-attachments/assets/36c50ef7-3de4-48fe-ace8-262cc9661382)


### Вывод
В ходе данной работы была произведена настройка Asterisk для локальных звонков.
