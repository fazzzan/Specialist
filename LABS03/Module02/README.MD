# NETACAD
## 2.1.2 Включение OSPF
Процесс OSDPFv2 включается командой, с заданием process-ID (1-65535)
```
router ospf <XX>
```
После ввода этой команды, мы приступаем к настройкам процесса OSPF:
- RID [router ID]  - 32-bit значение, похожее на IPv4 адрес. Это значение должно быть уникальным в OSPF домене. Задать его можно:
   - вручную ```router-id <xxxxx>```, где xxxxx - 32-bit значение
   - автоматически: 
       - Если есть хоть один ___lo___ IF, то любой максимальный ip из всех Lo.
       - Если нет Lo, то любой максимальный из всех имеющихся
Для уверенного последующего траблшутинга, часто рекомендуют RID задавать вручную
```
router ospf 10
router-id 1.1.1.1

do sho ip proto | in Router ID
```

![](./pictures/01.jpg)

Если по каким-то причинам надо ___сменить RID___, то делается это ___только с последующим сбросом OSPF___:
```
clear ip ospf process
```

В этот момент отвалятся все соседства и пропадут маршруты в целевые OSPF-сети

Влияет значение RID на алгоритм некоторых действий:
   - ExStart состояние: у кого больший RID, тот станет master и начнет обмене DBD-шками
   - участвует в выборах DR/BDR - тот, у кого больше всех - становится DR, тот у кого чуть меньше - BDR

Ахтунг-вопрос:

![](./pictures/02.jpg)

## 2.2.1
Если перед нами PPP-сеть, то IF, который подключается к OSPF может быть однозначно идентифицирован подсеткой этого IF:

```
network <network-address> <wildcard-mask> area <area-id>
```

- Любой IF, который определен таким образом в OSPF - будет задействован в OSPF-обмене (отправить/получать ospf-пакеты).
- area-id должна быть одинакова на всех роутерах, в одной AREA. Если всего одня area, то рекомендуется использовать area-id=0

Также процесс OSPF можно включить и на каком-либо IF непосредственно:
```
ip ospf <process-id> area <area-id>
```

Для безопасности, IF через которые OSPF ходить не должен (это те. которые не подключены к OSPF-соседям) - надо делать пассивными:
```
router ospf <10>
passive-intarface loopback 0

...

R1# show ip protocols
...
  Passive Interface(s):
    Loopback0
```

## 2.2.11 DR/BDR в PPP - подключении

Даже если подключение PPP - в канале ethernet все равно будут DR/BDR.

![](./pictures/03.jpg)

Если по каким-либо причинам это надо отключить, (например мы скоммутировали 2 роутер по Ethernet-подключению, но мы знаем, что их всего лишь 2)
```
ing gi 0/0
ip ospf netwo point-to-point
```

![](./pictures/04.jpg)

отсюда фича, при работе с Loopback: если нам надо получать не /32 подсеть, то можно сконфигурировать Lo point-to-point и тогда на соседа прилетит /24:

![](./pictures/04.jpg)

## лаба 2.2.13

![](./pictures/06.jpg)

Задачи:
- Configure Router IDs.
- Configure Networks for OSPF Routing.
- Configure Passive Interfaces.
- Verify OSPF configuration.

- [pdf](labs/2.2.13-packet-tracer---point-to-point-single-area-ospfv2-configuration.pdf)
- [pka](2.2.13-packet-tracer---point-to-point-single-area-ospfv2-configuration.pka)

Лаба выполнена, но удовольствие не получено, так как, так как не разобравшись, я подумал что в PT не видно DR/BDR. Пересобрал в GNS3 и понял что соединение PPP, поэтому DR/BDR - не будет: это PPP, детка

![](./pictures/07.jpg)

[gns3](labs/2.2.13_exp.gns3project)

![](./pictures/09.jpg)



___По итогу прочтения - я не совсем понял, зачем так много времени посвящено point-to-point сетям___

## 2.3.1 OSPF Network Types
Второй тип сетей OSPF - multiaccess, этот тип будет отличаться от ppp - сети, так как тут будут выделенные роутеры DR/BDR, и DR будет отвечать за распространение LSA. DR использует MC-пакеты на адрес 224.0.0.5 (всем OSPF - роутерам). BDR - некий пассивный дублер DR, который втихаря все слушает ит также поддерживает соседские отношения со всеми другими соседями AREA.

Все остальные роутеры - DROTHER, и они уже используют MC 224.0.0.6 для взаимодействия друг с другом и DR/BDR. Как ранее говорилось, тот у кого больше RID - станет DR, у кого меньше - BDR и перевыборов не будет, пока не сдохнет DR или BDR.

![](./pictures/08.jpg)

```
show ip ospf interface <Gi0/0/0>
```

![](./pictures/10.jpg)

Как мы помним, роутеры OSPF находятся в каком-то из перечисленных состояний:
- ___FULL/DROTHER___ - DR или BDR в состоянии ___full___-соседства по отношению к non-DR or BDR роутерам. Обмен с такими присутствует в полном объеме: Hello packets, updates (LSU), queries (LSR), replies (LSR), and acknowledgments (LSAck).
- ___FULL/DR___ - ___full___-соседство с DR соседом. Обмен также в полном объеме: Hello packets, updates, queries, replies, and acknowledgments.
- ___FULL/BDR___ - ___full___-соседство с BDR соседом. Также обмен в полном объеме: Hello packets, updates, queries, replies, and acknowledgments.
- ___2-WAY/DROTHER___ - non-DR и non-BDR (DROTHER) роутеры также имеют отношения соседства между собой. Обмен ТОЛЬКО Hello packets.

Смотрим на состояние: нормальное состояние - FULL. Если R завис в каком-то другом состоянии (за исключением 2-WAY, как показано выше), то у него есть проблемы формирования соседства (разные таймеры, MTU и т.д.)

## 2.3.6 Default DR/BDR Election Process
DR/BDR выборы начинаются сразу, как только на OSPF-роутере включится IF. Выборы занимают несколько секунд и поэтому если кто-то из роутеров проспал этот момент, он уже никак не сможет повлиять на результат: новые R не вмешиваются в работу сети.  Критерии выбора DR/BDR - следующие:
1.  На основании приоритета (___см. описание hello-пакета, Router Priority - для выбора DR/BDR - 1 default, вручную задаем 0 - 255 (чем больше, тем лучше)___), роутер сам себе выбирает DR. Роутер со вторым по величине приоритетом становится BDR. Приоритет задается на IF, если его значение задать в 0, то IF роутера никогда не будет назначен DR/BDR (типа открестился от почетной обязанности). Приоритет по-умолчанию для multiaccess broadcast interfaces ___1___. Так как чаще всего никому нафиг эти приоритеты менять не надо, они так и остаются в 1 и поэтому придумали еще один спосб назначения.
2. При обмене Hello-пакетами, роутеры получают информацию о RID других участников обмена. Поскольку, как я написал, приоритеты на IF никто не меняет, роутеры начинают меряться RID-ами, и у кого больше - тот DR. Второй по величине RID - BDR.

Ну и тут уже надо понимать, что по-хорошему новым и центровым роутерам надо назначать максимальный RID, не забывая сбрасывать ```clear ip ospf process```. А если забыли, то RID возьмется автоматически:
   - максимальный lo адрес
   - максимальный из активных сетевой IPv4 IF.

Вышесказанное, относительно устоявшегося положения вещей с DR/BDR относится и ко вновь включенному R: опаздавший сидит и ждет пока не освободится место самого буйного в сумасшедшем доме.

## 2.3.8 ip OSPF priority
Эта настройка нужна чтобы выиграть выборы OSPF. Таким образом R прекрасно может сочетать в себе DR в одной AREA и быть DROTHER в другой. Если поставить 0 - значит R никогда не станет DR/BDR
```
R1(config)# interface GigabitEthernet 0/0/0 
R1(config-if)# ip ospf priority 255 
R1(config-if)# end 
```

В результате, R1 станет DR

![](./pictures/11.jpg)


___Лабка 2.3.11___

- [pdf](labs/2.3.11-packet-tracer---determine-the-dr-and-bdr.pdf)
- [pka](labs/2.3.11-packet-tracer---determine-the-dr-and-bdr.pka)

Задачи:
![](./pictures/12.jpg)
- Part 1: Examine DR and BDR Changing Roles
- Part 2: Modify OSPF Priority and Force Elections

просмотреть какая роль в настоящий момент у R можно командами
```
sho ip ospf int gi 0/0/0
...
shi ip ospf nei
```

![](./pictures/13.jpg)

Поперезагружал роутеры, в итоге RC - стал DROther, а DR - RB, BDR - RA, затем сделал ```debug ip ospf adj``` и отключил Gi0/0/0 на RA, который к тому времени стал DR

![](./pictures/15.jpg)

Через некоторое время я увидел как ospf поплыл и заметил отвал ___FULL->DOWN___ соседа, с которым он был DROTHER/BDR. Ну и понятно, что инициировались перевыборы. В итоге:
- DR - остался RB
- BDR - стал RC.

А вот и пакетики LSU/LSR/LSAck подвезли:

На L3:

1. The device multicasts out an OSPF Hello packet on GigabitEthernet0/0.
2. The device encapsulates the data into an IP packet.
3. The device sets the TTL on the packet.
4. The destination IP address is a broadcast or multicast address. The device sets the destination address as the next-hop.

L2:

1. The next-hop IP address is a multicast. The ARP process sets the frame's destination MAC address to a multicast MAC address.
2. The device encapsulates the PDU into an Ethernet frame.

L1:

1. GigabitEthernet0/0 sends out the frame.

Весь обмен идет MC, но нутрянку OSPF- пакета я не посмотрю в PT, ее надо смотреть в GNS

После смены приоритетов, в соответствии с задланием, картина состояний роутеров стала следующей

![](./pictures/16.jpg)

Выполненно 100%

## 2.4 Modify Single-Area OSPFv2

### Cisco OSPF Cost Metric

еще раз: данный link-state протокол использует поныятие стоимости для вычисления маршрута