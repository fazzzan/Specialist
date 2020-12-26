# 10. Basic Router configuration
Не мудрствуя лукаво:

```
hostname hostname
enable secret class

line console 0
password cisco
login

line vty 0 4
password cisco
login
transport input ssh telnet
exit

service password-encryption

banner motd #
*********************************************** 
WARNING: Unauthorized access is prohibited!
***********************************************
#

copy running-config startup-config
```

## Маленькая лабораторка:
```
Enter global configuration mode to configure the name of the router as “R1”.
Router>enable
Router#configure terminal
```

Enter configuration commands, one per line. End with CNTL/Z.

```
Router(config)#hostname R1
```

Configure 'class' as the secret password.
```
R1(config)#enable secret class
```

Configure 'cisco' as the console line password, require users to login, and return to global configuration mode.
```
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
```

For vty lines 0 through 4, configure 'cisco' as the password, require users to login, enable SSH and Telnet access, and return to global configuration mode.
```
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#transport input ssh telnet
R1(config-line)#exit
```

Encrypt all clear text passwords.
```
R1(config)#service password-encryption
```

Enter the banner 'Authorized Access Only!' and use # as the delimiting character.
```
R1(config)#banner motd #Authorized Access Only!#
```

Exit global configuration mode.
```
R1(config)#exit
```
_________________________

## Лабка по настройке базового конфига роутера
Настроил без проблем, ответил на вопросы

# 10.2.1 Настройка интерфейсов
Из новенького - приучиться заполнять description, запомнить настройки ipv6, 
```
R1(config)# interface gigabitEthernet 0/0/0
R1(config-if)# description Link to LAN
R1(config-if)# ip address 192.168.10.1 255.255.255.0
R1(config-if)# ipv6 address 2001:db8:acad:10::1/64
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)#
```
проверка:
   * ___show ip interface brief___
   * ___show ipv6 interface brief___

   * ___show ip route___
   * ___show ipv6 route___

   * ___show interfaces___ - Displays statistics for all interfaces on the device. However, this command will only display the IPv4 addressing information.
   * ___show ip interfaces___ -	Displays the IPv4 statistics for all interfaces on a router.
   * ___show ipv6 interface___ - Displays the IPv6 statistics for all interfaces on a router.

# 10.3.1 Конфигурация Def GW
Поскольку наша задача как сетевиков выпустить ПК в инет - надо запомнить, что без "ворот" любой хост не знает куда отправлять пакеты L3, с тем чтобы они были доставлены куда-то дальше. Def GW имеет одинаковое значение как для IPv4, так и для IPv6

![](./pictures/01.jpg)

На свичах L2 GW задается командой вместе с настройкой Management VLAN
```
int vlan 1
ip addreess 192.168.10.10 255.255.255.0
no shut
exit

ip default gateway 192.168.10.1 255.255.255.0
```

## 2 лабки, совсем не сложные на Def GW
### 10.3.4

![](./pictures/02.jpg)

### 10.3.5

![](./pictures/03.jpg)

### 10.3.6

Лабораторка стала камнем преткновения. Как ни старался - добиться 100% не удалось, причем каждый раз открывал для себя что-то новенькое:
   - ipv6 unicast-routing - включить на R
   - FE80::1 - для соответствующего link-local адреса на обоих IF R
   - несоответствие практически заданным подсетям ipv6 на Gi0/0, Gi0/1 R

![](./pictures/04.jpg)
![](./pictures/05.jpg)

Video тут: https://www.youtube.com/watch?v=uPinwWoF1As

Итак, по этапам:
   - ICMPv6-req зависает в памяти, так как нет MAC в таблице соседей DST шлюза, отправляем ND
   - ND приходит на Floor14, в его таблице соседей появляется чей-то MAC и тут возникает идея icmpv6 отправить с Reception-B, так как Room-146 открыт для просмотра, а что настроено на Room-145 - я не знаю.

Начинаем снова:
   - ICMPv6 зависает в памяти, выплевываем MC NDP
   - NDP приходит на Room-146

![](./pictures/06.jpg)

   - Room-146 отправляет на все порты, так как его таблица - пуста. В таблице соседей Floor14 появляется link-local адрес Reception-B
![](./pictures/07.jpg)
   - Floor14 отвечает на NDP, так как в DST icmpv6 - его адрес FE80::2, также в его таблице соседей появляются IPv6 и MAC linc-local адреса Reception-B
![](./pictures/08.jpg)
   - Room-146 получает ND от Floor-14 и так как его таблица MAC - заполнена, отдает его именно в Gi1/0/2
![](./pictures/09.jpg)
   - reception-B получает NDP-ответ и теперь знает на какой MAC отправлять кадр с вложенным ICMPv6
   - Кадр приходит на Room-146, значения MAC-DST в таблице коммутации, кадр - UC, пересылаем кадр на порт Gi1/0/24
   - Кадр приходит на Floor-14 и
     - тот видит что кадр пришел на его MAC
     - в подсеть, которая есть у него в таблице маршрутизации за IF Gi0/0. 
     - IPv6 DST - Unicast, но такого вдреса нет в таблице соседей, 
     - поэтому снова запускается NDP: Замораживаем ICMPv6-req и отправляем Neighbour Solicitation, занеся в таблицу соседей ICMPv6 Manager-A, без MAC

![](./pictures/09.jpg)   

   - Что происходит на Room-145 на мне посмотреть, но пока таблицы соседей и коммутации в Room-146 не меняются, так как пока все процедуры происходят на Room-145
   - Приходит ответ NDP от Manager-A: 
     - MAC DST соответствует MAC Gi0/0, развертываем дальше
     - IPv6 DST соответствует IPv6 Gi0/0, развертываем дальше
     - Заносим в таблицу соседей MAC Manager-A, который нам ответил на этот NDP
     - Отправляем ICMPv6 по-назначению, на Manager-A

![](./pictures/11.jpg)  

   - Manager-A получает ICMPv6
     - MAC - его, развертываем дальше
     - IPv6 - его, развертываем дальше 
     - Внутри пакета ICMPv6-echo-req, надо отвечать
     - DST IPv6 - в другой подсети и не BC, а также нет в кэше
     - Есть DefGW, в пакет подставляем его IP, 
     - для кадра - не хватает MAC DST, надо опять запускать NDP NS, чтобы выяснить MAC для ICMPv6 Gi0/0
     -  next-hop IP address is multicast address. The ND Process sets the frame's destination MAC address to a multicast MAC address.
   - Кадр пришел на Room-145
     - SRC MAC - известен
     - DST MAC - MC, рассылаем на все порты
   - Кадр пришел на Floor-14
     - MAC - BC, развертываем дальше 
     - IPv6 - MC, лезем внуть
     - Пакет - ICMPv6, NDP: NS target IPv6 соответствует IPv6 Gi0/0
     - Обюновляем таблицу соседей: заносим в нее Link-local ICMPv6/MAC Manager-A
     - Готовим NDP- ответ

![](./pictures/12.jpg)  

   - После того как Manager-A уточнил MAC Floor-14, он подготовил кадр с ответом ICMPv6-echo-reply для Reception-B и отправил его на Gi0/0 Floor14
   - Floor-14 получил ICMPv6 Echo-reply:
     - MAC совпадает с MAC Gi0/0
     - IPv6 DST находится в другом сегменте, на который есть маршрут, и который directly-connected
     - IPv6 DST - UC, но в таблице соседей его нет, надо опять запускать NDP NS, на предмет выяснения MAC-адреса

![](./pictures/13.jpg)  

После как отработает NDP - таблица соседей на Floor14 будет заполнена Link-local и ICMPv6 учатсников обмена и дальнейши обмен пойдет без NDP

И вот теперь становится понятно откуда взялась эта вторая волна NDP, при обратном движении ICMPv6-echo-reply