
Задать вопрос относительно железок без K9 (Зачем, как обойти, на примере нексуса с прошивкой без K9)

Рекомендации:
- Выделить MGM сеть
- Разрешить вход в MGM-сеть и ПК управления только с определенных админских ПК. через ACL
- Выключать ПК управления на то время, пока нет админских задач

Можно ли сгенерить сертификат SSH для нескольких CN, чтобы обращаться к железке по разным именам?

Ответ:
- Можно, при использовании стороннего центра сертификации, а именно: сгенерить сертификат на несколько имен и после этого установить на нужную железку

Можно ли перенаправить SSH на другой порт, не 22?
- Можно, но при этом понадобится вешать дополнительный ACL, чтобы перекрыть порт 22 и не забыть открыть созданный порт в других ACL:

```
R1(config)#ip ssh port 2001 rota
R1(config)#ip ssh port 2001 rotary 1
R1(config)#line vty 0 4
R1(config-line)#rotary 1
```

# SPECIALIST 20210227
до 24.02.2020 года был отдельный кору CCNA SEC - IINS (implementing network security).Теперь такой сертификации нет, а курс размазан по модулям других курсов.

Также в курс добавлены дополнительные вещи от Павла и его лабы:
- Wireshark
- nmap (ip-сканирование)
- Kali Linux

___Online telnet___ - можно проверить доступность TCP-порта

состояние стенда
![](./pictures/04.jpg)

Основные теоретические моменты.

1. Защита сетевых устройств - это не только защита R, но и SW. На границе - как правило 1 устройство, это же и является точка отказа. 
2. Расширение - Defense in depth (эшелонированная защита):
- R1 - смотрит на прова, внешняя маршрутизация
- FW - за R1
- R2 - внутренняя маршрутизация, внутренние ACL, с нулевкой на FW

![](./pictures/06.jpg)
![](./pictures/07.jpg)

3. Если надо дать доступ извне на сервера, то надо организовать DMZ, но это уже вопросы больше дизайна

![](./pictures/05.jpg)

- Физически серверные надо защищать. Если защиты нет - все можно взломать.
- Надо иметь копии IOS, конфигов (раз в сутки)
- Router Hardening - конфиг должен быть максимально безопасным, с закрытыми внешними портами, сложными паролями, логгированием попыток доступа, ААА, разделение прав по группам

![](./pictures/08.jpg)

Конфиденциальность - использование enable secret, также передача всех конфигов в зашифрованном виде

MGM порт - на части R есть такой порт, который либо можно включить в отдельную MGM сеть, либо использовать как дополнительный порт для LAN

Пароль на R: можно использовать парольную фразу, с пробелами. пароли надо менять хотя бы 1 раз в год

![](./pictures/09.jpg)

Для локального хранения паролей - ___keypass___. 

Для группы сотров - ___TeamPass___

У Каспера - свое решение, как для локального, так и для сетевого хранения

В сетях TCP/IP изначально, в ТЗ, не была заложены безопасность => большинство протоколов (arp, telnet и т.д.) - дырявые. Старые протоколы надо заменять на новые.

TPM - в россии запрещен
Bitlocker - пароль на вход
Шифрование файлов и папок - NTFS encryption
AxCrypt, VeraCrypt - стороннее шифрование программными средствами

___Лабораторки 1-12___

___Лабораторка 1___
![](./pictures/10.jpg)
- [pdf](labs/ЛР1_Настройка_общих_параметров_и_проверка связи.pdf)

Настроили R1, R2, R3: конфиги тут:

- [R1](labs/lab1/R1.txt)
- [R2](labs/lab1/R2.txt)
- [R3](labs/lab1/R3.txt)

Проверка показала успешный пинг на все узлы

___Лабораторка 2___
![](./pictures/10.jpg)
- [pdf](labs/ЛР2_Сканирование_сети_и_узлов.pdf)

Nmap – популярный и мощный инструмент для сканирования сетей и узлов. Nmap работает в режиме командной строки, для простоты GUI zenmap https://nmap.org/.
Задание: просканировать с KaliLinux сеть 192.168.1.0/24
Все просто, как 1-2-3
![](./pictures/11.jpg)


Обнаружил узлы 
- 192.168.1.1
- 192.168.1.3
- 192.168.1.101
- 192.168.1.5

Сканирование узла 192.168.1.1 показало что открыт порт 23:
![](./pictures/13.jpg)
![](./pictures/14.jpg)

Также раскусили что данный хост - Cisco IOS 15.1
![](./pictures/12.jpg)
![](./pictures/15.jpg)

Попытались подключиться по telnet, но видно что надо установить пароль на вход в IOS

![](./pictures/16.jpg)

___Лабораторка 3___
![](./pictures/10.jpg)
- [pdf](labs/ЛР3_Изучение_хранения_паролей_и_варианты_взлома.pdf)

Посмотрим как хранятся пароли в Cisco:
- открытом виде
- зашифрованных сервисом шифрования паролей
- захешированных с помощью хеш-функции md5 и scrypt

Попробовать взломать пароли при взломщика паролей John The Ripper (https://www.openwall.com/john/ ).

Открытый пароль зашифровали сервисом шифрования, при этом видно что: Перед паролем 
будет стоять тип 7. Тип 7 указывает, что этот пароль был получен с помощью сервиса шифрования паролей
![](./pictures/17.jpg)

```
R1(config)#enable password cisco
R1(config)#do show run | in pass
enable password cisco
...
R1(config)#service password-encryption 
R1(config)#do show run | in pass
service password-encryption
enable password 7 00071A150754

R1(config)#enable password Ci$co       
R1(config)#do show run | in pass
service password-encryption
enable password 7 153102480725
```
Дешифровали 7-й пароль на сайте: https://packetlife.net/toolbox/type7/
![](./pictures/18.jpg)

Далее попробуем захешировать пароль и взломать его John Ripper:
```
R1(config)#enable secret cisco
R1(config)#do show run | in sec 
R1(config)#do show run | in secret
enable secret 5 $1$96uH$mmnn64oQ.YJDz/t6YMFff/
```
![](./pictures/19.jpg)

Алгоритм хеширования выбирается с помощью algorithm-type

![](./pictures/20.jpg)

Хеш MD5 — type 5, SHA256 — type 8 и Scrypt — type 9 (самый устойчивый к взлому). Если не указывать algorithm-type, будет по умолчанию использован MD5. Алгоритмы хеширования доступны как для команды ___enable___, так и для команды ___username___.
```
enable algorithm-type scrypt secret cisco
```

На уже более-менее прилично мпароле ___enable secret Ci$c0_123WS___ я устал ждать подбора.




___Лабораторка 4___
![](./pictures/10.jpg)
- [pdf](labs/ЛР4_Настройка_имён_пользователей_и_паролей.pdf)

Картинка все та же самая.

Задача: 
- Настроить на R1, R3 минимальную длину пароля 10 символов
```
service password-encryption
security passwords min-length 10
enable algorithm-type scrypt secret cisco12345
username admin algorithm-type scrypt secret cisco12345
```
- Включить баннер motd, для ___CONSOLE___ подключения - локальную базу данных, автозакрытие через 5 минут (по-умолчанию 10), 
```
banner motd #Unauthorized access strictly prohibited!#
...
line con 0
login local
exec-timeout 5
exit
```
- Включить для VTY подключения  - локальную базу данных, автозакрытие через 5 минут (по-умолчанию 10)
```
line vty 0 4
login local
exec-timeout 5
tran in telnet
```



___Лабораторка 5___
Картинку см. выше
- [pdf](labs/ЛР5_Изучение_протоколов_Telnet_и_SSH.pdf)

Задача:
- Запустить на линке Wireshark и захватить логин/пароль, просмотреть что было введено

![](./pictures/22.jpg)
Видно, что логинг/пароль передается в незашифрованном виде. 

- установить на R1-R3 транспорт ssh ver 2 (как поменять порт - см. в начале), опять проверить какие пакеты захвачены
```
ip domain-name acad.local
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
...
line vty 0 4
tran in ssh
end
```
вот как будет выглядеть зашифрованная ssh сессия

![](./pictures/23.jpg)

___проверка:___
```
show ip ssh 
```
![](./pictures/24.jpg)

```
show control-plane host open-ports
```

![](./pictures/25.jpg)


Количество линий VTY, для маршрутизатора больше чем 0...4: их обычно по умолчанию 5, так что для оставшихся ```no tran in```. Тогда получим при 6-й сессии входа:

![](./pictures/21.jpg)


___Лабораторка 6___
Картинку см. выше
- [pdf](labs/ЛР6_Усиление_защиты_маршрутизатора_от_атак_на_доступ.pdf)

Настройки могут быть применены как маршрутизатору, так и к свичу

Задача:
1. Настройка задержки при вводе пароля (задержка для замедления подбора пароля)
2. Настройка логгирования успешных/неуспешных попыток входа (фиксация подбора пароля)
3. Настройка блокировки входа при нескольких неудачных попыток ввода пароля (замедление подбора пароля)
4. Настройки ACL для сохранения доступа к хосту во время включения блокировки

```
1. 
R1(config)# login delay 10
2. 
R1(config)# login on-success log 
R1(config)# login on-failure log
3. 
R1(config)# login block-for 300 attempts 3 within 120
4.
R1(config)# ip access-list standard ACL-LOGIN-QM 
R1(config-std-nacl)# permit host 192.168.1.101 
R1(config-std-nacl)# exit 
R1(config)# login quiet-mode access-class ACL-LOGIN-QM 
R1(config)# exit 
```

Проверка ```show login```, ```show access-list``` (будет присутствовать "автоматически созданный acl")



# Securing Network Device
## 2.0.1.1
- Защищаем физический доступ к железке
- паролирование доступа по сети
- SSH на вход
- различные уровни досупа персонала
- SNMP v2, v3
- NTP
- Syslog

Отключаем старые сервисы

Внедряем ААА

внедряем аутентификацию протоколов динамической маршрутизации (OSPF MD5/SHA)

## 2.1.2.2
Настраиваем доступ со сложным паролем и ограничиваем время прогстоя
```
security passwo min-len <10> - задаем дилну пароля
service password-encryption - шифруем пароли в конфиге
line vty 0 4
exec-timeout 3 30 - 3,5 минуты на простой, вместо 10
lone con 0
exec-timeout 3 30 - 3,5 минуты на простой, вместо 10
```

Также можно отключить избыточные линии, чтобы никто другой кроме нескольких админов не залез на железку

## 2.1.2.3 - Алгоритмы пароля
0 - незашифрованный
...
9 - SCRYPT HASHED secret - надо ввести хэш пароля, который можно просмотреть на любом уже настроенном устройстве

![](./pictures/02.jpg)

задается как в secret, так и в con/vty

![](./pictures/01.jpg)



## 2.1.3.4
```

R2(config)# ip access-list standard PERMIT-ADMIN
R2(config-std-nacl)# permit 192.168.10.10
R2(config-std-nacl)# exit

R2(config)# login block-for 15 attempts 5 within 60 
R2(config)# login quiet-mode access-class PERMIT-ADMIN
R2(config)# login delay 10
R2(config)# login on-success log
R2(config)# login on-failure log
R2(config)# exit
R2#
...
R2# show login 
     A login delay of 10 seconds is applied.
     Quiet-Mode access list PERMIT-ADMIN is applied.
     All successful login is logged.
     All failed login is logged.

     Router enabled to watch for login Attacks.
     If more than 5 login failures occur in 60 seconds or less,
     logins will be disabled for 15 seconds.

     Router presently in Normal-Mode.
     Current Watch Window
         Time remaining: 15 seconds.
         Login failures for current window: 0.
     Total login failures: 0.

R2#
```

## 2.1.4.2 Configuring SSH

## 2.2.2 Role-Based CLI

![](./pictures/03.jpg)