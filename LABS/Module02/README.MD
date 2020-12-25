# Эта глава посвящена общим/базовым понятиям настройки железа.
## Базовый конфиг свича
https://study-ccna.com/no-ip-domain-lookup-command/
```
configure terminal
no ip domain lookup
hostname <Name>
enable secret <class>
line con 0
password <cisco>
exit
line vty 0 15
password <cisco>
login
exit
banner motd #Restricted access#
service password encryption
int vlan 1
ip address <IP_MASK>
no shut
exit
```

## Режимы работы/включение конфигурирования:
При включении мы попадаем в ___User Exec Mode DEVICE___ [ >] -> ```enable``` -> ___Privileged Exec Mode DEVICE___[#]->```configure terminal```->___Global Configuration Mode DEVICE___ [(conf)#]

обратно: 
   * [>] - ```exit```
   * [#] - ```end```

Линии/IF:
   * Line config mode (ssh/telnet/console/AUX)
   * Interface config mode (switch or Router интерфейсы)

## Таблица сокращенных нажатий:

   * ___Tab___ -	Completes a partial command name entry.
   * ___Backspace___ -	Erases the character to the left of the cursor.
   * ___Ctrl+D___ -	Erases the character at the cursor.
   * ___Ctrl+K___ -	Erases all characters from the cursor to the end of the command line.
   * ___Esc D___ -	Erases all characters from the cursor to the end of the word.
   * ___Ctrl+U or Ctrl+X___ -	Erases all characters from the cursor back to the beginning of the command line.
   * ___Ctrl+W___ -	Erases the word to the left of the cursor.
   * ___Ctrl+A___ -	Moves the cursor to the beginning of the line.
Left Arrow or Ctrl+B - 	Moves the cursor one character to the left.
   * ___Esc B___ - 	Moves the cursor back one word to the left.
   * ___Esc F___ -	Moves the cursor forward one word to the right.
   * ___Right Arrow or Ctrl+F___ - 	Moves the cursor one character to the right.
   * ___Ctrl+E___ -	Moves the cursor to the end of command line.
   * ___Up Arrow or Ctrl+P___ -	Recalls the previous command in the history buffer, beginning with the most recent command.
   * ___Down Arrow or Ctrl+N___ -	Goes to the next line in the the history buffer.
   * ___Ctrl+R or Ctrl+I or Ctrl+L___ -	Redisplays the system prompt and command line after a console message is received.

## Keystroke	Description
   * ___Ctrl-C___ -	When in any configuration mode, ends the configuration mode and returns to privileged EXEC mode. When in setup mode, aborts back to the command prompt.
   * ___Ctrl-Z___ -	When in any configuration mode, ends the configuration mode and returns to privileged EXEC mode.
   * ___Ctrl-Shift-6___ -	All-purpose break sequence used to abort DNS lookups, traceroutes, pings, etc.

## Название устройств
   * Start with a letter
   * Contain no spaces
   * End with a letter or digit
   * Use only letters, digits, and dashes
   * Be less than 64 characters in length

## Требования к паролям
   * Use passwords that are more than eight characters in length.
   * Use a combination of upper and lowercase letters, numbers, special characters, and/or numeric sequences.
   * Avoid using the same password for all devices.
   * Do not use common words because they are easily guessed.

## Для защиты ___privileged EXEC mode___
Как было сказано, при входе мы в ___user EXEC mode___. Надо задать пароль для защиты ___privileged EXEC mode___  при вводе команды ```enable```
```
Sw-Floor-1# configure terminal
Sw-Floor-1(config)# enable secret class
Sw-Floor-1(config)# exit
Sw-Floor-1#
```

## Также надо защищать вход через ___console/VTY___
```
w-Floor-1# configure terminal
Sw-Floor-1(config)# line console 0
Sw-Floor-1(config-line)# password cisco
Sw-Floor-1(config-line)# login
Sw-Floor-1(config-line)# end
Sw-Floor-1#
```
Для запароливания VTY lines (telnet/ssh) [Cisco support up to 16 VTY lines (simultaneously): 0...15]
```
Sw-Floor-1# configure terminal
Sw-Floor-1(config)# line vty 0 15
Sw-Floor-1(config-line)# password cisco 
Sw-Floor-1(config-line)# login 
Sw-Floor-1(config-line)# end
Sw-Floor-1#
```

## Шифрования plaintext Passwords в конфиге
plaintext сущетсвует как в настройке ```enable```, так и в настройках ___line___
```
Sw-Floor-1# configure terminal
Sw-Floor-1(config)# service password-encryption
Sw-Floor-1(config)#
```

## Banner Messages
Предупреждаем хакеров о том что они вламываются на охраняемую территорию
```
Sw-Floor-1# configure terminal
Sw-Floor-1(config)# banner motd #Authorized Access Only#
```
## Configuration Files:     
   * ___startup-config___ - в NVRAM/Flash памяти, которая does not lose its contents when the device is powered off. ```erase startup-config ``` - clear all startup configurations
   * ___running-config___ - в Random Access Memory (RAM - теряет настройки при перезапуске). Modifying a running configuration affects the operation of a Cisco device immediately.

```reload``` - remove an unsaved running config, ```copy startup-config running config``` - способ попытаться восстановить часть настроек, которые неосмотрительно применились, не всегда работает

## IP Addresses
   * IPv4 - 32 bit + 32 bit mask
   * IPv6 - 128 bit, каждые 4 бита - 1 х16 символ, группа из 4 символов отделяется от другой знаком ":"

# Что узнали в модуле 2

All end devices and network devices require an operating system (OS). The user can interact with the shell using a command-line interface (CLI) to use a keyboard to run CLI-based network programs, use a keyboard to enter text and text-based commands, and view output on a monitor.

As a security feature, the Cisco IOS software separates management access into the following two command modes: ___User EXEC Mode___ and ___Privileged EXEC Mode___.

___Global configuration mode___ need to access other specific configuration modes. From global config mode, the user can enter different subconfiguration modes. Each of these modes allows the configuration of a particular part or function of the IOS device. Two common subconfiguration modes include: ___Line Configuration Mode___ and ___Interface Configuration Mode___. To move in and out of global configuration mode, use the configure terminal privileged EXEC mode command. To return to the privileged EXEC mode, enter the exit global config mode command.

Each IOS command has a specific format or syntax and can only be executed in the appropriate mode. The general syntax for a command is the command followed by any appropriate keywords and arguments. The IOS has two forms of help available: context-sensitive help and command syntax check.

The first configuration command on any device should be to give it a unique device name or hostname. Network devices should always have passwords configured to limit administrative access. Cisco IOS can be configured to use hierarchical mode passwords to allow different access privileges to a network device. Configure and encrypt all passwords. Provide a method for declaring that only authorized personnel should attempt to access the device by adding a banner to the device output.

There are two system files that store the device configuration: startup-config and running-config. Running configuration files can be altered if they have not been saved. Configuration files can also be saved and archived to a text document.

IP addresses enable devices to locate one another and establish end-to-end communication on the internet. Each end device on a network must be configured with an IP address. The structure of an IPv4 address is called dotted decimal notation and is represented by four decimal numbers between 0 and 255.

IPv4 address information can be entered into end devices manually, or automatically using Dynamic Host Configuration Protocol (DHCP). In a network, DHCP enables automatic IPv4 address configuration for every end device that is DHCP-enabled. To access the switch remotely, an IP address and a subnet mask must be configured on the SVI. To configure an SVI on a switch, use the interface vlan 1 global configuration command. Vlan 1 is not an actual physical interface but a virtual one.

In the same way that you use commands and utilities to verify a PC host’s network configuration, you also use commands to verify the interfaces and address settings of intermediary devices like switches and routers. The show ip interface brief command verifies the condition of the switch interfaces. The ping command can be used to test connectivity to another device on the network or a website on the internet.



