# NETACAD

## Типы Static Route
Бывают:
- Standard
- Default
- Floating
- Summary

Идентификация Next-hop может быть:
- Next-hop route - ___ip - адрес___
- Directly connected static route - ___egress IF___
- fully specified static route - и ip и IF

задаются командами ```ip route``` или ```ipv6 route```:
```
Router(config)# ip route network-address subnet-mask { ip-address | exit-if [ip-address]} [distance]
```
- ip-address - адрес Next-hop? при этом может возникнуть рекурсивный запрос, Next-hop через Next-hop через Next-hop, что приводит к задержкам выплевывания пакета
- exit-IF - через что выплевывать пакет, при этом автоматически создается ___directly-connected___ static маршрут. Используется в p-t-p подключениях
- exit-if ip-address - для fully specified static route
- distance - AD; 1...255; хитрость: для задания плавающего/floating маршрута задаем AD больше чем у dynamic-вычисленного маршрута

```
Router(config)# ipv6 route ipv6-prefix/prefix-length {ipv6-address | exit-intf [ipv6-address]} [distance]
```
- ipv6-prefix - идентификатор IPv6 сети назначения.
- /prefix-length - длина prefix'а сети назначения.
- ipv6-address - next-hop router IPv6 address. 

Че за хрень?? не понял фразы
___Typically used with broadcast networks (i.e., Ethernet). Could create a recursive static route where the router performs an additional lookup to find the exit interface.___

- exit-intf - exit IF куда будут выплевываться пакеты. Создает ___directly connected static route___. Используется для point-to-point.
- exit-intf ipv6-address - для fully specified static route.
- distance - AD; 1...255; хитрость: для задания плавающего/floating маршрута задаем AD больше чем у dynamic-вычисленного маршрута

без static маршрутов связности в такой сетке не будет

![](./pictures/01.jpg)
![](./pictures/02.jpg)

Ахтунг- вопрос
![](./pictures/03.jpg)

## 15.2.1  Next-hop Static Route
Для приведенной выше картинки на R1 сделаем 
- для IPv4:
```
R1(config)# ip route 172.16.1.0 255.255.255.0 172.16.2.2
R1(config)# ip route 192.168.1.0 255.255.255.0 172.16.2.2
R1(config)# ip route 192.168.2.0 255.255.255.0 172.16.2.2
...

R1# show ip route | begin Gateway
Gateway of last resort is not set
...
S     172.16.1.0/24 [1/0] via 172.16.2.2
...
S     192.168.1.0/24 [1/0] via 172.16.2.2
S     192.168.2.0/24 [1/0] via 172.16.2.2

```

- для IPv6:
```
R1(config)# ipv6 unicast-routing
R1(config)# ipv6 route 2001:db8:acad:1::/64 2001:db8:acad:2::2
R1(config)# ipv6 route 2001:db8:cafe:1::/64 2001:db8:acad:2::2
R1(config)# ipv6 route 2001:db8:cafe:2::/64 2001:db8:acad:2::2

...

R1# show ipv6 route
...
S   2001:DB8:ACAD:1::/64 [1/0]
     via 2001:DB8:ACAD:2::2
...
S   2001:DB8:CAFE:1::/64 [1/0]
     via 2001:DB8:ACAD:2::2
S   2001:DB8:CAFE:2::/64 [1/0]
     via 2001:DB8:ACAD:2::2
L   FF00::/8 [0/0]
     via Null0, receive
```

## 15.2.3  Directly Connected Static Route
- R1/IPv4
```
R1(config)# ip route 172.16.1.0 255.255.255.0 s0/1/0
R1(config)# ip route 192.168.1.0 255.255.255.0 s0/1/0
R1(config)# ip route 192.168.2.0 255.255.255.0 s0/1/0
...
R1# show ip route | begin Gateway
Gateway of last resort is not set
...
S     172.16.1.0/24 is directly connected, Serial0/1/0
...
S     192.168.1.0/24 is directly connected, Serial0/1/0
S     192.168.2.0/24 is directly connected, Serial0/1/0
```

- R1/IPv6
```
R1(config)# ipv6 route 2001:db8:acad:1::/64 s0/1/0
R1(config)# ipv6 route 2001:db8:cafe:1::/64 s0/1/0
R1(config)# ipv6 route 2001:db8:cafe:2::/64 s0/1/0

...

R1# show ipv6 route
...
S   2001:DB8:ACAD:1::/64 [1/0]
     via Serial0/1/0, directly connected
...
S   2001:DB8:CAFE:1::/64 [1/0]
     via Serial0/1/0, directly connected
S   2001:DB8:CAFE:2::/64 [1/0]
     via Serial0/1/0, directly connected
L   FF00::/8 [0/0]
     via Null0, receiveIPv6 Routing Table - default - 8 entries
```

## 15.2.5 IPv4 Fully Specified Static Route
Рекомендуется при задании exit-IF  в  Ethernet пользоваться fully specified static route (exit-IF and next-hop ip).
- R1/IPv4
```
R1(config)# ip route 172.16.1.0 255.255.255.0 GigabitEthernet 0/0/1 172.16.2.2
R1(config)# ip route 192.168.1.0 255.255.255.0 GigabitEthernet 0/0/1 172.16.2.2
R1(config)# ip route 192.168.2.0 255.255.255.0 GigabitEthernet 0/0/1 172.16.2.2
...

R1# show ip route | begin Gateway
...
S        172.16.1.0/24 [1/0] via 172.16.2.2, GigabitEthernet0/0/1
...
S     192.168.1.0/24 [1/0] via 172.16.2.2, GigabitEthernet0/0/1
S     192.168.2.0/24 [1/0] via 172.16.2.2, GigabitEthernet0/0/1
```

- R1/IPv6 - тут есть тонкость: если next-hop ip задается LLA, то надо указывать exit-IF, иначе будет ошибка (IPv6 LLA - не содержится в IPV6-таблице маршрутизации, LLA - адрес уникален только в пределах линка)
```
R1(config)# ipv6 route 2001:db8:acad:1::/64 fe80::2
%Interface has to be specified for a link-local nexthop
R1(config)# ipv6 route 2001:db8:acad:1::/64 s0/1/0 fe80::2

...

R1# show ipv6 route static | begin 2001:db8:acad:1::/64
S   2001:DB8:ACAD:1::/64 [1/0]
    via FE80::2, Seria0/1/0
```
___Способы проверки___
```
    show ip route static
    show ip route static | be Gateway
    show ip route <network>
    show running-config | section ip route
...
    show ipv6 route static
    show ipv6 route static | be App
    show ipv6 route <network>
    show running-config | section ipv6 route

```

## 15.3.1 Default Static Route
Имеем дело со stub-сеткой или единственным выходом к прову

![](./pictures/04.jpg)

```
IPv4
Router(config)# ip route 0.0.0.0 0.0.0.0 {ip-address | exit-intf}

Ipv6
Router(config)# ipv6 route ::/0 {ipv6-address | exit-intf}


R1# show ip route static
...
Gateway of last resort is 172.16.2.2 to network 0.0.0.0
S* 0.0.0.0/0 [1/0] via 172.16.2.2 

R1# show ipv6 route static
...
S   ::/0 [1/0]
     via 2001:DB8:ACAD:2::2
```
В выводах static маршрут идет с маской /0 или ::/0

## 15.4.1 Floating Static Routes
floating маршрут нужен на случай если у вас больше чем 1 выход в инет и необходимо чтобы этот маршрут автоматически подставлялся в таблицу маршрутизации при падении основного линка. Для этого AD делаем больше чем у основного маршрута (меньше AD - больше доверия маршруту)/ маршрут с большим AD - менее желателен. Напомним, что по-умолчанию, AD static маршрута = 1. AD основных протоколов:
- EIGRP = 90
- OSPF = 110
- IS-IS = 115

![](./pictures/05.jpg)

![](./pictures/06.jpg)

Настроим float- маршрут и проверим таблицу маршрутизации:
```
R1(config)# ip route 0.0.0.0 0.0.0.0 172.16.2.2
R1(config)# ip route 0.0.0.0 0.0.0.0 10.10.10.2 5
R1(config)# ipv6 route ::/0 2001:db8:acad:2::2
R1(config)# ipv6 route ::/0 2001:db8:feed:10::2 5

R1# show ip route static | begin Gateway
Gateway of last resort is 172.16.2.2 to network 0.0.0.0
S*   0.0.0.0/0 [1/0] via 172.16.2.2

R1# show ipv6 route static | begin S : 
S   ::/0 [1/0]
     via 2001:DB8:ACAD:2::2
R1#
```
Когда упадет R2, таблица маршрутов станет:
```
R1# show ip route static | begin Gateway
Gateway of last resort is 10.10.10.2 to network 0.0.0.0
S*    0.0.0.0/0 [5/0] via 10.10.10.2

R1# show ipv6 route static | begin :: 
S   ::/0 [5/0] 
     via 2001:DB8:FEED:10::2
R1#
```

## 15.5.1 Host Routes
Это маршруты к конкретным хостам с масками /32 и /128 для Ipv6. Появится такой маршрут в таблице сможет только когда:
- автоматиески при конфигурировании ip на IF (___L___)
- static host route
- host route полученный другими методами (позже)

![](./pictures/07.jpg)

```
Branch# show ip route | begin Gateway
Gateway of last resort is not set
      198.51.100.0/24 is variably subnetted, 2 subnets, 2 masks
C        198.51.100.0/30 is directly connected, Serial0/1/0
L        198.51.100.1/32 is directly connected, Serial0/1/0

Branch# show ipv6 route | begin :: 
C   2001:DB8:ACAD:1::/64 [0/0]
     via Serial0/1/0, directly connected
L   2001:DB8:ACAD:1::1/128 [0/0]
     via Serial0/1/0, receive
L   FF00::/8 [0/0]
     via Null0, receive
```
Добавим вручную и проверим:
```
Branch(config)# ip route 209.165.200.238 255.255.255.255 198.51.100.2
Branch(config)# ipv6 route 2001:db8:acad:2::238/128 2001:db8:acad:1::2
Branch(config)# exit

...

Branch# show ip route | begin Gateway
...
S        209.165.200.238 [1/0] via 198.51.100.2

Branch# show ipv6 route
S   2001:DB8:ACAD:2::238/128 [1/0]
     via 2001:DB8:ACAD:1::2
Branch#
```
Отдельнывй случай - IPv6 Static Host Route через Link-Local Next-Hop
```
Branch(config)# no ipv6 route 2001:db8:acad:2::238/128 2001:db8:acad:1::2
Branch(config)# ipv6 route 2001:db8:acad:2::238/128 serial 0/1/0 fe80::2
Branch# show ipv6 route | begin ::
...
S   2001:DB8:ACAD:2::238/128 [1/0]
     via FE80::2, Serial0/1/0
```

___Лаба - 15.6.1___

- [pdf](labs/15.6.1-packet-tracer---configure-ipv4-and-ipv6-static-and-default-routes.pdf)
- [pka](labs/15.6.1-packet-tracer---configure-ipv4-and-ipv6-static-and-default-routes.pka)

![](./pictures/08.jpg)

___Лаба - 15.6.2___

- [pdf](labs/15.6.2-lab---configure-ipv4-and-ipv6-static-and-default-routes.pdf)
- [...](labs/...)