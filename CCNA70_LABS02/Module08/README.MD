# SPECIALIST 20210116 #
UDP/546? UDP/547
3 способа получения ip
1. SLAAC - Stateless address auticonfi
   - RA (router Advertisement)
2. Stateless DHCPv6/DHCPv6 Lite - здесь DHCP-сервер выдает только опции, а адрес - выдает маршрутизатор
3. Statefull DHCPv6 - все операции делает DHCP-сервер

Пакеты DORA преобразовали:
DHCPv4/DHCPv6
Discover/Solicit - поиск DHCPv6
Offer/Advertise - сервер рассказывает о себе и говорит какая сеть доступна
Request/Request - клиент говорит что хочу сеть, сервер проверяет на le.kz; захоченного ip
Ack/Reply - подтверждает, если нет дубляжа

Итак, DHCPv6 делает:
- Может вести табличку кто и что получил
- Нет понятия "время аренды"
- Использует DUID  для идентификации клиента - DHCP Unic ID - Идентифиактор клиента, который клиент генерит сам. Делается это якобы для анонимности, так как раньше по MAC можно было "следить" за клиентом. В DHCPv6 уже не получится выяснить по MAC - клиента

Итак, при включении клиент отправляет RS, а роутер отвечает RA с флагами SLAAC/DHCPv6 Lite/DHCP:
- A(utoconfig) ___1___/___1___/___0___ - генери сам, других настроек не будет
- O(ther) ___0___/___1___/___0___ - генери сам, а настройки от DHCP
- M(anaged) ___0___/___0___/___1___  - вообще все получай от DHCPv6
Из поыта: вполне устраивает SLAAC
![](./pictures/04.jpg)

# NETACAD #
# DHCP v6 #
методы DCHP для ipv6:
- SLAAC [stateless address autoconfiguration] - (аналог APIPA для Global UC addresses)
- DHCPv6
## Ipv6 GUA assignment ##
SLAAC  или DHCPv6 - все равно на IF будет GUA и LLA. Пример статического GUA

![](./pictures/01.jpg)

При получении динамического ipv6-адреса, хост будет использовать какой-либо из 3 методов определенных в пойманном сообщении ICMPv6 Router Advertisement [RA]. К, находящийся в одном сегменте с ПК, рассылает эти самые RA-шки:
- Ipv6 Router может быть настроен на рассылку RA, которые получает клиент
- LLA - адресавтоматически создается при включении IF. ___%<№№>___ - для идентификацции ZoneID или ScopeID, это используется OS для идентификации определенного IF.

Пример LLA
```
C:\PC1> ipconfig
Windows IP Configuration
Ethernet adapter Ethernet0:
   Connection-specific DNS Suffix  . : 
   IPv6 Address. . . . . . . . . . . : 
   Link-local IPv6 Address . . . . . : fe80::fb:1d54:839f:f595%21
   IPv4 Address. . . . . . . . . . . : 169.254.202.140
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 
```

В примере нет GUA- адреса, так как возможно там нет и R, рассылающего RA-шки. Итак, методы получения GUA: Все они используют RA-шки, для указания хосту метода настройки Ipv6

## Назначение Ipv6 GUA 
По-сути 2 способа, с небольшими отличиями:
- ___Stateless___
   - ___SLAAC only___ RA содержит:
      - network prefix
      - prefix-length
      - def GW addr
   - ___SLACC with DHCP server___ RA указывает кто тут DHCP, чтобы клиент обратился к нему за доп. информацией
- ___Stetefull___ - RA говорит клиенту прийти за всеми настройками на DHCPv6

![](./pictures/02.jpg)

### 3 Флага RA ###
Необходимы, чтобы клиент рздуплил с каким режимом DHCPон имеет дело:
- ___A flag___ - Address Autoconfiguration признак: чувак, сам себе настраивай адрес GUA методом [___Stateless___ Address Autoconfiguration] - SLAAC.
- ___O flag___ - Other Configuration признак: Other информация есть на ___Stateless___ DHCPv6 server.
- ___M flag___ - Managed Address Configuration признак: ___Stateful___ DHCPv6 выдаст IPv6 GUA.

![](./pictures/03.jpg)

## SLAAC ##
Не все сети имеют DHCP, но всем клиентам надо выходить в инет, поэтому хосты могут опираясь на метод SLAAC сами генерить себе Ipv6-адреса. При этом в сети отсутствует узел, хранящий информацию о том за кем и какой адрес закреплен.

RA (каждые 200 сек) в этом случае выполняет информационную функцию, и хост использует информацию из RA

## DHCPv6 ##
## Config DHCPv6 Sesrver ##
