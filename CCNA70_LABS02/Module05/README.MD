# Specialist 20201226
## Необходимость в STP/основные термины/этапы построения STP

В результате развития, инженеры пришли к необходимости построения физически отказоустойчевых подключений к SW более высокого уровня и между собой. 
STP реализован при помощи ___BPDU___ [bridge protocol data unit], которыми  обмениваются Свичи: 

- I этап: выбора корня ___root bridge___ на основании BID: идентификатор SW. Это некоторый составной ___ID=PRI & MAC___ 
     - MAC - можно посмотреть (min из вывода ```show interfaces | in Giga|address```)
     - приоритет [PRI] = 16bit (Default 32768)

![](./pictures/06.jpg)   

```
Switch#show interfaces | in Giga|address
!
GigabitEthernet0/0 is up, line protocol is up (connected) 
  Hardware is iGbE, address is 0c09.c5ce.5300 (bia 0c09.c5ce.5300)
...
  Hardware is iGbE, address is 0c09.c5ce.530f (bia 0c09.c5ce.530f)
!
show spanning-tree | sec Bridge ID
!
Switch#show spanning-tree | sec Bridge ID
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0c09.c5ce.5300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
```
   
При выборе ROOT - побеждает SW с ___min BID___

![](./pictures/07.jpg)   

- II этап: После выбора ROOT - начинают выбирать ___ROOT Port [RP]___ на каждом SW, на основании ___стоимости___ линков до ROOT Bridge. В BPDU рассылает информацию до ROOT, все последуюшие SW рассылают информацию с учетом пришедшей стоимости аплинков до ROOT'а

![](./pictures/07.jpg)

![](./pictures/08.jpg)

![](./pictures/09.jpg)

- III этап: После выбора RP определяем роли остальных портов:
   - На ROOT - все порты DP, FWD
   - На конкурирующих SW в сторону ROOT смотрит RP (FWD)
   - осталось выяснить какой порт на каком SW класть в DOWN. Если стоимости одинаковы, то решаем по стоимости BID посылающего. У кого BID меньше, тот побеждает и оставляет свой порт вкл. ___DP/FWD___, второй - смиряется и гасит свой порт ___NDP/BLK___. При этом SW не отправляет через этот порт никаких пакетов, кроме BPDU.

![](./pictures/10.jpg)     

- IV этап - поддержание и пересчет топологии при падении RP на каком-либо SW

В старом стандарте STP - падение LNK - пересчет топологии

## STP в параллельных линкАх

Если есть несколько параллельно существующих LNK до RB, то на назначение ___NDP/BLK___ влияет ___PID[PortID] = PRI.NBR___:
- PRIority - 8 bit (default 128)
- NBR - номер порта (fa0/1 - 1; Fa0/2 - 2 и т.д.)

Победит порт, который имеет min значение PID: справа ROOT, значит блокироваться будет один из линкОв левого свича, а именно тот, на который прилетел больший PID 128.2

![](./pictures/11.jpg)   

![](./pictures/12.jpg)   

Уточненные этапы выбора состояния, ролей портов

![](./pictures/13.jpg)   

Если злоумышленник решит скоммутировать порты на одном свиче между собой, то опять срабатывает STP, который пройдет все этапы и на основании PID блокирует порт с MAX PID.

Для управления STP - настраиваем приоритеты SW, чтобы ROOT был на max эффективных SW.

В старом STP алгоритм работы:
   - коммутаторы обмениваются BPDU (timer ___Hello___ - 2 Сек )
   - если BPDU не приходит 20 секунд (timer ___MAX Age___), то считаем что сосед "умер" и начинаем пересчет топологии и работу с портом
   - простаивающий NDP порт - из состояния DIS ->- LIStening (15 сек) ->- LRN (изучение приходящих кадров, заполнение MAC - табл, тоже 15 сек) ->- FWD (timer ___Forwarding delay___)

Итого, при пересчете топологии min время простоя 2+20+30=52 секунды, что очень долго по современным меркам

![](./pictures/14.jpg)   

- I этап: ASW1 - ROOT: все порты DP (на основании BID)
- II этап: порты
   - DSW1 - F0/1 - RP
   - DSW2 - F0/2 - RP
   - ASW2 - F0/2 - RP (стоимости до ROOT - одинаковые через оба линка, но на основании min BID - выигрывает линк до DSW2)
- III этап блокировка портов
     - DSW1 - F0/2 - DP/FWD, Gi0/1 - NDP/BLK (на основании BID пришедшего PDU)
     - DSW2 - F0/1 - DP/FWD, Gi0/1 - DP/FWD
     - ASW2 - F0/2 - NDP/BLK; F0/1 - RP/FWD

## Этапы развития STP

- CST IEEE 802.1D
PRI - 16 bit, но только безумный захочет управлять PRI таким количетством бит
- PVST+ для каждого VLAN - свое дерево: часть бит сократили и отдали для передачи инфы о VLAN  - только Cisco
- RSTP (Rapid STP) IEEE 802.1w
- PVRSTP+ (Per VLAN Rapid STP+) - только Cisco. При увеличении числа VLAN - требуется большое число ресурсов (100 VLAN, 100 BPDU в 2 сек, 100 ROOT и т.д.)
- Cisco придумала упростить ситуацию, объединяя часть деревьев с одинаковыми деревьями в инстансы (1-50 VLAN: 1 инстанс, 51-100 VLAN: 2 инстанс) MIST
- Общепринятый вариант на основе MIST - 802.1S MST/MSTP (multiple spanning tree)

Могут возникнуть проблемы совметсной работы SW поддерживающих разные версии STP => предложение - отказаться от STP, за счет внедрения на верхнем уровне L3, но тут тоже могут быть разные подходы в реализации

![](./pictures/15.jpg)   

Договорились над этой сетью строить VXLAN - программно-определяемые сети SDN

[Лаба](./labs/STP.pkt)   

![](./pictures/16.jpg)   
![](./pictures/17.jpg)  

```
DSW1#show spanni summ
Switch is in pvst mode
Root bridge for:
Extended system ID           is enabled
Portfast Default             is disabled
PortFast BPDU Guard Default  is disabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is disabled
UplinkFast                   is disabled
BackboneFast                 is disabled
Configured Pathcost method used is short

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     1         0        0          2          3

---------------------- -------- --------- -------- ---------- ----------
1 vlans                      1         0        0          2          3
```

Просматривать STP лучше за конкретный VLAN ```sho spann vlan XX```

В CST - состояние порта называется ___NDP___

В RSTP - состояние порта называется Alternate, Backup - немного другая терминология, но роль порта от этого не меняется.

```
DSW1#show spann vlan 1
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5EDE.80DA
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00E0.F957.E098
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Root FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Gi0/1            Altn BLK 4         128.25   P2p
```

Для ROOTа все порты DP/FWD

```
ASW1#show spann vlan 1
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5EDE.80DA
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0005.5EDE.80DA
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
```

![](./pictures/18.jpg) 

Чтобы SW стал root - меняем на нем приоритет в нижнюю сторону. Еще можно установить SW в root primary (уменьшает текущее значение на 32768) или root secondary (___почитать___). 

НО

___Лучше вручную: primary 4196, secondary 8192___

Для ускорения перевода порта в DSG состояния (по умолчанию порт пытается получить штук 10 BPDU, что долго для современных ПК на SSD), необходимо включать на Access портах:
- ```spanning tree portfast``` - включение быстрого определения включения порта
- ```spanning tree bpduguard``` - включение блокировки порта, при получении BPDU на порте. Смешной случай, когда на ПК несколько сетевух, которые при бриджинге (технология ОС) тоже генерят BPDU
- включение на порту storm control - позволяет ограничить на порту количество приходящих кадров: UC/MC/BC

```
(config-if)# storm-control broadcast level 30.00 10.00
(config-if)# storm-control action shutdown
```
При достижении первого порого (30.00) - порт будет отключаться, при падении broadcast-трафика до уровня второго (10.00) — включаться.

Чтобы в этом примере коммутатор автоматически восстанавливал порт, скажем, через 3 минуты (это 180 секунд) после отключения, необходимо в глобальной конфигурации выполнить:

```
(config)# errdisable recovery cause storm-control
(config)# errdisable recovery interval 180
```

Все остальное - "от лукавого", но надо запомнить, что для тонкой настройки порта:
- ```loopguard```
- ```rootguard```

У SW Cisco есть также штатное средство обнаружения петель, основанное на периодической отправке keepalive-сообщений. Эта опция обычно включена по-умолчанию, и в случае срабатывания порт отключается. Для механизма keepalive можно также настроить автоматическое включение интерфейса:

```
# errdisable recovery cause loopback
```

# NETACAD, Module 5.0 - STP

STP предназначена для разрыва петель, возникающих при построении отказоустойчивых сегментов, петель L2

![](./pictures/01.jpg)   

Без STP пакеты кружились бы хороводом по закольцованным сегментам, размножаясь и плодясь, что может привести к переполнению буфера SW и превращению его в хаб

![](./pictures/02.jpg)   

# 5.1.7 Алгоритм STP
   - ___выбор ROOT___

![](./pictures/02.jpg)   

   - ___блокировка избыточных линков___

![](./pictures/03.jpg)   

   - ___работа по безкольцевой топологии___

![](./pictures/04.jpg)   

   - ___отработка сбоев/разблокировака портов___

![](./pictures/05.jpg)   

# 5.3.1 Эволюция STP
![](./pictures/20.jpg)   

___Почитать___:

STP в опасности:

https://unlix.ru/stp-%d0%b2-%d0%be%d0%bf%d0%b0%d1%81%d0%bd%d0%be%d1%81%d1%82%d0%b8/#more-793

