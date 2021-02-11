# NETACAD
## 5.1.1 Создание ACL
Прежде чем взять и все сломать, шутка, надо ACL как следуют спланировать.
- Работаем в текстовом редакторе
- Работаем в песочнице
- Используем ___remark___
- Копипастим строки на железо
- Тестируем ACL, чтобы не получить эффекта неожиданности

## 5.1.2/3 Standard IPv4 ACL Syntax
- Numbered 
```
 access-list access-list-number {deny | permit | remark text} source [source-wildcard] [log]
```
- Named 
```
 ip access-list standard access-list-name
... 
Standard Access List configuration commands:
  <1-2147483647>  Sequence Number
  default         Set a command to its defaults
  deny            Specify packets to reject
  exit            Exit from access-list configuration mode
  no              Negate a command or set its defaults
  permit          Specify packets to forward
  remark          Access list entry comment
```

## 5.1.4
После того как вы сваяли ACL - надо его применить хоть куда-то, например на IF ))

```
Router(config-if) # ip access-group {access-list-number | access-list-name} {in | out}
```

Пример настройки ACL: 

```
access-list 10 remark ACE permits ONLY host 192.168.10.10 to the internet
access-list 10 permit host 192.168.10.10
access-list 10 remark ACE permits all host in LAN 2
access-list 10 permit 192.168.20.0 0.0.0.255
...
do show access-lists
Standard IP access list 10
    10 permit 192.168.10.10
    20 permit 192.168.20.0, wildcard bits 0.0.0.255
...
R1(config)# interface Serial 0/1/0
R1(config-if)# ip access-group 10 out
R1(config-if)# end
...
R1# show ip int Serial 0/1/0 | include access list
  Outgoing Common access list is not set
  Outgoing access list is 10
  Inbound Common access list is not set
  Inbound  access list is not set
```

Еще 1 пример:
![](./pictures/02.jpg)

___Лаборатока 5.1.8___

![](./pictures/03.jpg)

- [pdf](labs/5.1.8-packet-tracer---configure-numbered-standard-ipv4-acls.pdf)
- [pka](labs/5.1.8-packet-tracer---configure-numbered-standard-ipv4-acls.pka)

Задачи:
- Part 1: Спланироват ACL
- Part 2: Сконфигурировать, применить и проверить Standard ACL
   - Evaluate two network policies and plan ACL implementations.
      - a. The following network policies are implemented on R2:
         - Запретить 192.168.11.0/24 доступ до WebServer в подсети 192.168.20.0/24.
         - Все остальное - разрешено.
      - b. The following network policies are implemented on R3:
         - Запретить обмен между 192.168.10.0/24 и 192.168.30.0/24.
         - All other access is permitted.

Как помним, стандартные ACL - лучше держать максимально далеко от себя, поэтому:
- var a , ___ACL 1___
   - 1-й ACE, запрещающий доступ ACL от 192.168.11.0/24 до WebServer 192.168.20.254 лечше нависить на R2, OUT IF, в направлении OUT, чтобы этот ACL не наделал бед и не зарезал другой возможный поток из подсетки 192.168.10.0/24 или из какой-то другой.
   - 2-й ACE будет явно разрешать все остальное (permit any).
- var b, тоже ___ACL 1___
   - Запретить исходящий из 192.168.10.0/24 в 192.168.30/24 так чтобы он не пересекался с другими потоками можно только на R3: ACE на IF в сторону подсетки 192.168.30.0/24 , в направлении OUT.
   - 2-й ACE будет permit all.

Перед выполнением лабы - надо проверить что везде есть ip-связность:
пинги ходят между всеми хостами
- var a
```
R2(config-if)#do show access-li
Standard IP access list 1
    10 deny 192.168.11.0 0.0.0.255
    20 permit any

R2(config-if)#do show ip int gi0/0 | in access
  Outgoing access list is 1
  Inbound  access list is not set
  IP access violation accounting is disabled
```
Результат применения - pc2 не в состоянии обмениваться ip-трафиком с сервером, но вполне себе обменивается с другими участниками
![](./pictures/04.jpg)

- var b
```
R3(config-if)#do show access-li
Standard IP access list 1
    10 deny 192.168.10.0 0.0.0.255
    20 permit any

R3(config-if)#do show ip int gi0/0 | in access
  Outgoing access list is 1
  Inbound  access list is not set
  IP access violation accounting is disabled
```
Результат применения - pc1 не в состоянии обмениваться ip-трафиком с подсетью PC3, но вполне себе обменивается с другими участниками, РС3 - также может общаться со всеми, кроме подсети PC1
![](./pictures/05.jpg)

при этом понятно, что echo-request в подсеть PC1 - прилетают, и даже echo-reply улетают обратно, но вот пробиться от PC1 до PC3 на R3 они уже не могут, так как ACL закрывает этот трафик

![](./pictures/06.jpg)

100%


___Лаборатока 5.1.9___
Все просто - вешается 1 ACL на IF глядищий на file-сервер, в направлении OUT.

___Лаборатока 5.4.12___

![](./pictures/01.jpg)

- [pdf](labs/5.4.12-packet-tracer---configure-extended-ipv4-acls---scenario-1.pdf)
- [pka](labs/5.4.12-packet-tracer---configure-extended-ipv4-acls---scenario-1.pka)

Задачи:
- Part 1: Configure, Apply and Verify an Extended Numbered ACL
- Part 2: Configure, Apply and Verify an Extended Named ACL:

Создали 2 ACL:
```
access-list 100 permit tcp 172.22.34.64 0.0.0.31 host 172.22.34.62 eq ftp
access-list 100 permit icmp 172.22.34.64 0.0.0.31 host 172.22.34.62
...
ip access-list extended HTTP_ONLY
 permit tcp 172.22.34.96 0.0.0.15 host 172.22.34.62 eq www
 permit icmp 172.22.34.96 0.0.0.15 host 172.22.34.62
```

повесили их на соответствующие IF, в ближайшем от отправителя месте ( на входе в IF Gi0/0, Gi0/1)
```
interface GigabitEthernet0/0
 ip access-group 100 in
 !
interface GigabitEthernet0/1
 ip access-group HTTP_ONLY in
 ```

  и проверили:
- трафик icmp сохранился от обоих устройств
- с PC1 - не работает http на 172.22.34.62
- c PC2 - не работает ftp на 172.22.34.62