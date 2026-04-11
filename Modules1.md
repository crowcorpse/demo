## <div align="center"><strong>⚙️</strong></div> <div align="center"><strong>МОДУЛЬ 1</strong></div>

<br/>

<p align="center">
  <a href="https://github.com/kaktotad/demo-sys-2026/blob/main/topologia.png" target="_blank">
    <img src="https://github.com/kaktotad/demo-sys-2026/blob/main/images/topologia.png?raw=true" alt="Топология" width="50%"/>
  </a>
</p>

## 📚 Навигация по методичке

* [Задание №0 Преднастройка](#преднастройка)
* [Задание №1 Имена устройств](#именаустройств)
* [Задание №2 Адресация](#адресация)
* [Задание №3 NAT](#nat)
* [Задание №4 Настройка пользователей](#пользователи)
* [Задание №5 Настройка VLAN](#vlan)
* [Задание №6 Настойка SSH](#ssh)
* [Задание №7 Настройка GRE](#gre)
* [Задание №8 Настройка OSPF](#ospf)
* [Задание №9 Настройка NAT на HQ-RTR и BR-RTR](#lan-nat)
* [Задание №10 Настройка DHCP](#dhcp)
* [Задание №11 Настройка DNS](#dns)
* [Задание №12 Настройка времени](#время)

---

<a id="преднастройка"></a>
## ✔️ 0. Преднастройка

<details>
<summary><strong>Преднастройка выполняеться на ВСЕХ машинах</strong></summary>
Для того что бы вы могли что либо скачивать, необходимо зайти в sources.list командой 

```bash
nano /etc/apt/sources.list
```

и закомментировать находящуюся там строку (поставить знак #) перед каждой строчкой.

Стало: ![image](https://github.com/user-attachments/assets/9fc5e769-7d3e-45c9-b7fa-a9f76d72d374)

Далее необходимой перейти в файл resolv.conf командой

```bash
nano /etc/resolv.conf
```
и прописать

```bash
nameserver 8.8.8.8
```

Стало: ![image](https://github.com/kaktotad/demo-sys-2026/blob/567cb803758ce096cef66c19297bcfc8c7e4d335/images/resolv_google.png)

Далее переходим в файл 
```
nano /etc/sysctl.conf
```

найти строку 
- net.ipv4.ip_forward=0

и включить данный параметр (удалить знак (#) и изменить 0 на цифру (1))
```
net.ipv4.ip_forward=1
```

Стало: ![image](https://github.com/user-attachments/assets/bfd8fb86-d2fd-4244-8e62-b3b4c70d6397)

После чего необходимо применить данные изменения прописав команду 
```bash
sysctl -p
```
</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="именаустройств"></a>
## ✔️ 1. Настройка имён устройств

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>
Настройте имена устройств согласно топологии. Используйте полное доменное имя
</details>

<details>
<summary><strong>Данное задание выполняеться на ВСЕХ машинах</strong></summary>
  
> <strong>ISP</strong>:
```bash
hostnamectl set-hostname isp.au-team.irpo; exec bash
```

> <strong>HQ-RTR</strong>:
```bash
hostnamectl set-hostname hq-rtr.au-team.irpo; exec bash
```

> <strong>BR-RTR</strong>:
```
hostnamectl set-hostname br-rtr.au-team.irpo; exec bash
```

> <strong>HQ-SRV</strong>:
```
hostnamectl set-hostname hq-srv.au-team.irpo; exec bash
```

> <strong>HQ-CLI</strong>:
```
hostnamectl set-hostname hq-cli.au-team.irpo; exec bash
```

> <strong>BR-SRV</strong>:
```
hostnamectl set-hostname br-srv.au-team.irpo; exec bash
```
</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="адресация"></a>
## ✔️ 2. Настройка IP адресов

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>
На всех устройствах необходимо сконфигурировать IPv4:
  
- IP-адрес должен быть из приватного диапазона, в случае, если сеть локальная, согласно RFC1918  
- Локальная сеть в сторону HQ-SRV(VLAN 100) должна вмещать не более 32 адресов  
- Локальная сеть в сторону HQ-CLI(VLAN 200) должна вмещать не менее 16 адресов  
- Локальная сеть для управления(VLAN 999) должна вмещать не более 8 адресов  
- Локальная сеть в сторону BR-SRV должна вмещать не более 16 адресов  
- Настройте адресацию на интерфейсах:  
- Интерфейс, подключенный к магистральному провайдеру, получает адрес по DHCP  
- Настройте маршрут по умолчанию, если это необходимо  
- Настройте интерфейс, в сторону HQ-RTR, интерфейс подключен к сети 172.16.1.0/28  
- Настройте интерфейс, в сторону BR-RTR, интерфейс подключен к сети 172.16.2.0/28  
</details>

<details>
<summary><strong>Конфигурационные файлы адресации для всех устройств</strong></summary>

🌐 ISP
```
nano /etc/network/interfaces
```
```
auto ens192
iface ens192 inet dhcp

auto ens224
iface ens224 inet static
address 172.16.1.1/28

auto ens256
iface ens256 inet static
address 172.16.2.1/28
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/5d4e2b9d4374c862b46b2fc4e2073adb57a21f7e/images/IP-ISP.png)
```
systemctl restart networking
```
```
ip -c a 
```

📡 HQ-RTR
```
nano /etc/network/interfaces
```
```
auto ens192
iface ens192 inet static
address 172.16.1.2/28
gateway 172.16.1.1

auto ens224
iface ens224 inet static
address 192.168.1.1/28
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/1ddc964fbadf2a5ea5476fb492137f6db358073c/images/IP_hq-rtr.png)
```
systemctl restart networking
```
```
ip -c a 
```

📡 BR-RTR
```
nano /etc/network/interfaces
```
```
auto ens192
iface ens192 inet static
address 172.16.2.2/28
gateway 172.16.2.1

auto ens224
iface ens224 inet static
address 192.168.4.1/28
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/1ddc964fbadf2a5ea5476fb492137f6db358073c/images/IP-br-rtr.png)
```
systemctl restart networking
```
```
ip -c a 
```
🖥️ HQ-SRV
```
nano /etc/network/interfaces
```
```
auto ens192
iface ens192 inet static
address 192.168.1.2/27
gateway 192.168.1.1
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/1ddc964fbadf2a5ea5476fb492137f6db358073c/images/IP-hq-srv.png)
```
systemctl restart networking
```
```
ip -c a 
```

🖥️ HQ-CLI
```
nano /etc/network/interfaces
```
```
auto ens192
iface ens192 inet dhcp
```
```
systemctl restart networking
```
```
ip -c a 
```

🖥️ BR-SRV
```
nano /etc/network/interfaces
```
```
auto ens192
iface ens192 inet static
address 192.168.4.2/28
gateway 192.168.4.1
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/1ddc964fbadf2a5ea5476fb492137f6db358073c/images/IP-br-srv.png)
```
systemctl restart networking
```
```
ip -c a 
```
</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="nat"></a>
## ✔️ 3. Настройка NAT на ISP

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>
На ISP настройте динамическую сетевую трансляцию портов для доступа к сети Интернет HQ-RTR и BR-RTR.
</details>

<details>
<summary>Данное задание выполняеться на <strong>ISP</strong> машине</summary>

Для настройки динамической трансляция адресов на ISP в сторону HQ-RTR и BR-RTR для доступа к сети Интернет необходимо настроить Nftables

ISP
```bash
nano /etc/nfatbles.conf
```
В конце файла добавить правило для NAT

```bash
table inet nat  {
  chain POSTROUTING {
  type nat hook postrouting priority srcnat;
  oifname "ens192" masquerade
  }
}
```
Отступы выполяются tab  
![images](https://github.com/kaktotad/demo-sys-2026/blob/d380abb8dbd1d84ffbae2281019685748f19a056/images/nftables-ISP.png)

Далее необходимло перезапустить службу Nftables командой
```bash
systemctl restart nftables
```
Далее вкючаем nfatbles в автозагрузку командой
```bash
systemctl enable --now nftables
```
И проверяем подключение к интернету с машин <strong>HQ-RTR</strong> и <strong>BR-RTR</strong> командой ping
>HQ-RTR
```bash
ping 8.8.8.8
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/0820872186163672aa9d1563a6ee45d73b2f9c0f/images/hq-rtr_ping_google.png)
>BR-RTR
```bash
ping 8.8.8.8
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/0820872186163672aa9d1563a6ee45d73b2f9c0f/images/br-rtr_ping_google.png)
</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="пользователи"></a>
## ✔️ 4. Создание пользоватлей

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>

Создайте локальные учетные записи на серверах HQ-SRV и BR-SRV:
  
- Создайте пользователя sshuser
- Пароль пользователя sshuser с паролем P@ssw0rd
- Идентификатор пользователя 2026
- Пользователь sshuser должен иметь возможность запускать sudo без ввода пароля
- Создайте пользователя net_admin на маршрутизаторах HQ-RTR и BRRTR
- Пароль пользователя net_admin с паролем P@ssw0rd
</details>

<details>
<summary>Задание выполняется на машинах <strong>HQ-SRV</strong>, <strong>BR-SRV</strong>, <strong>HQ-RTR</strong> и <strong>BR-RTR</strong></summary></br>

<details>
<summary>Пользователь sshuser создаеться на машинах <strong>HQ-SRV</strong> и <strong>BR-SRV</strong></summary>

<h3>HQ-SRV</h3>

Для создания пользователя с идентификатором 2026, необходимо ввести команду с указанием флага <strong><code>-u и значением 2026</strong></code>, флаг <code><strong>-U</strong></code> создаст группу пользователей и тем же именем что и сам пользователь

```bash
useradd sshuser -u 2026 -U
```
```bash
passwd sshuser
```
Вводим пароль <code>P@ssw0rd</code>  
Повторяем пароль для его установки  

После того как пароль был установлен, выдаем права <strong><code>root</strong></code> данному пользователю
``` bash
usermod -aG sudo sshuser
```

Далее переходим в конфигурационный файл пользователей
```bash
nano /etc/sudoers
```
и дополняем его строкой
```bash
sshuser  ALL=(ALL) NOPASSWD:ALL
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/f69fbadfa848f9a08bed806276b0837a79d6041e/images/sshuser_visudo.png)

<h3>BR-SRV</h3>

Для создания пользователя с идентификатором 2026, необходимо ввести команду с указанием флага <strong><code>-u и значением 2026</strong></code>, флаг <code><strong>-U</strong></code> создаст группу пользователей и тем же именем что и сам пользователь

```bash
useradd sshuser -u 2026 -U
```
```bash
passwd sshuser
```
Вводим пароль <code>P@ssw0rd</code>  
Повторяем пароль для его установки  

После того как пароль был установлен, выдаем права <strong><code>root</strong></code> данному пользователю
``` bash
usermod -aG sudo sshuser
```

Далее переходим в конфигурационный файл пользователей
```bash
nano /etc/sudoers
```
и дополняем его строкой
```bash
sshuser  ALL=(ALL) NOPASSWD:ALL
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/f69fbadfa848f9a08bed806276b0837a79d6041e/images/sshuser_visudo.png)
</details>

<details>
<summary>Пользователь net_admin создаеться на машинах <strong>HQ-RTR</strong> и <strong>BR-RTR</strong></summary>

<h3>HQ-RTR</h3>


Для создания пользователя, необходимо ввести команду

```bash
useradd net_admin
```
```bash
passwd net_admin
```

Вводим пароль <code>P@ssw0rd</code>  
Повторяем пароль для его установки  

После того как пароль был установлен, выдаем права <strong><code>root</strong></code> данному пользователю
``` bash
usermod -aG sudo net_admin
```

Далее переходим в конфигурационный файл пользователей
```bash
nano /etc/sudoers
```
и дополняем его строкой
```bash
net_admin  ALL=(ALL) NOPASSWD:ALL
```
![image](https://github.com/kaktotad/demo-sys-2026/blob/f69fbadfa848f9a08bed806276b0837a79d6041e/images/sshuser_visudo.png)

<h3>BR-RTR</h3>

Для создания пользователя, необходимо ввести команду

```bash
useradd net_admin
```
```bash
passwd net_admin
```

Вводим пароль <code>P@ssw0rd</code>  
Повторяем пароль для его установки  

После того как пароль был установлен, выдаем права <strong><code>root</strong></code> данному пользователю
``` bash
usermod -aG sudo net_admin
```

Далее переходим в конфигурационный файл пользователей
```bash
nano /etc/sudoers
```
и дополняем его строкой
```bash
net_admin  ALL=(ALL) NOPASSWD:ALL
```
</details>

</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="vlan"></a>
## ✔️ 5. Настройка VLAN

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>
Настройте коммутацию в сегменте HQ следующим образом: 
  
- Трафик HQ-SRV должен принадлежать VLAN 100
- Трафик HQ-CLI должен принадлежать VLAN 200
- Предусмотреть возможность передачи трафика управления в VLAN 999
- Реализовать на HQ-RTR маршрутизацию трафика всех указанных VLAN с использованием одного сетевого адаптера ВМ/физического порта
- Сведения о настройке коммутации внесите в отчёт
</details>

<details>
<summary>Данное задание выполняеться на <strong>HQ-RTR</strong> машине</summary>

- Настройте коммутацию в сегменте HQ следующим образом:
- Трафик HQ-SRV должен принадлежать VLAN 100
- Трафик HQ-CLI должен принадлежать VLAN 200
- Предусмотреть возможность передачи трафика управления в VLAN 999</br>

Для установки адресации на VLAN cети необходимо пупупу

На <strong>HQ-RTR</strong> и <strong>ISP</strong> проверим что файл <strong>/etc/resolve.conf</strong> не слетел, если слетел прописываем
```bash
nameserver 8.8.8.8
```
- Для начала установи пакет <strong>vlan</strong>:

```
apt install vlan -y
```

- После чего добавляем модуль <strong>modprobe 8021q</strong> командой:
```
echo 8021q >> /etc/modules
```
- Далее переходим к конфигурации файла <strong>/etc/network/interfaces</strong> и приводим ее к виду:

```
auto ens192  
iface ens192 inet static  
address 172.16.1.2/28    
gateway 172.16.4.1  
  
auto ens224  
iface ens224 inet static  
address 192.168.1.1/27  
  
auto ens224:1  
iface ens224:1 inet static  
address 192.168.2.1/28    

auto ens224.100  
iface ens224.100 inet static  
address 192.168.1.3/27    
Vlan-raw-device ens224  
  
auto ens224.200  
iface ens224.200 inet static  
address 192.168.2.3/28    
Vlan-raw-device ens224:1

auto ens224.99
iface ens224.99 inet static
address 192.168.99.1/29
```

</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="ssh"></a>
## ✔️ 6. Настройка SSH

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>

Настройте безопасный удаленный доступ на серверах HQ-SRV и BR-SRV: 
  
- Для подключения используйте порт 2026
- Разрешите подключения исключительно пользователю sshuser
- Ограничьте количество попыток входа до двух
- Настройте баннер «Authorized access only».
</details>

<details>
<summary>Данное задание выполняется на <strong>HQ-SRV</strong> и <strong>BR-SRV</strong></summary></br>

Настройте безопасный удаленный доступ на серверах HQ-SRV и BR-SRV:
- Для подключения используйте порт 2026
- Разрешите подключения исключительно пользователю sshuser
- Ограничьте количество попыток входа до двух
- Настройте баннер «Authorized access only»

<h3>HQ-SRV и BR-SRV</h3>
  
Первым шагом будет установка пакетов ssh командой  
```bash
apt install openssh-server -y
```
После чего необходимо открыть файл
```bash
nano /etc/ssh/sshd_config
```
и добавить строчки в файл
```bash
Port 2026
MaxAuthTries 2
PasswordAuthentication yes
Banner /etc/ssh/bannermotd
AllowUsers sshuser
```
После чего требуется создать файл <code>/etc/ssh/bannermotd</code>
```bash
nano /etc/ssh/bannermotd
```
и привести его в вид 
```bash
----------------------
Authorized access only
----------------------
```
</br>

Далее необходимо перезапутить <code>SSH</code> командой
```bash
systemctl restart sshd
```

</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="gre"></a>
## ✔️ 7. Настройка GRE-туннеля

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>

Между офисами HQ и BR, на маршрутизаторах HQ-RTR и BR-RTR необходимо сконфигурировать ip туннель:
  
- На выбор технологии GRE или IP in IP
- Сведения о туннеле занесите в отчёт.
</details>

<details>
<summary><strong>Настройка GRE в файле <code>etc/network/interfaces</code>на <code>HQ-RTR</code></strong></summary>
<br/>
<h3>HQ-RTR</h3>
Для настройки <strong><code>GRE</code></strong> туннеля на <strong>HQ-RTR</strong> переходим в файл конфигурации
```bash
nano /etc/network/interfaces
```
Далее дописываем в конце строчки

```bash
auto gre1
iface gre1 inet tunnel
address 10.10.10.1
netmask 255.255.255.252
mode gre
local 172.16.1.2
endpoint 172.16.2.2
ttl 64
```

Для работы туннеля необходимо в файле <code>/etc/modules</code>

```bash
nano /etc/modules
```
добавить строчку 
```bash
gre_ip
```
Далее необходимо перезагрузить сетевые настройки для применения новых
```bash
systemctl restart networking
```

</details>

<details>
<summary><strong>Настройка GRE в файле <code>etc/network/interfaces</code>на <code>BR-RTR</code></strong></summary>
<br/>
<h3>BR-RTR</h3>
Для настройки <strong><code>GRE</code></strong> туннеля на <strong>BR-RTR</strong> переходим в файл конфигурации

```bash
auto gre1
iface gre1 inet tunnel
address 10.10.10.2
netmask 255.255.255.252
mode gre
local 172.16.2.2
endpoint 172.16.1.2
ttl 64
```

Для работы туннеля необходимо в файле <code>/etc/modules</code>

```bash
nano /etc/modules
```
добавить строчку 
```bash
gre_ip
```
Далее необходимо перезагрузить сетевые настройки для применения новых
```bash
systemctl restart networking
```

</details>

<details>
<summary><strong>Проверка работоспособности <code>GRE</code> туннеля</strong></summary>
<h3>HQ-RTR</h3>

Первое, проверить применились ли наши настройки командой
```bash
ip -c a
```

![iamge](https://github.com/kaktotad/demo-sys-2026/blob/96d9d6e25cede97aaca0af74bcfb62cefa40b2c8/images/ip_a_gre_BR-RTR.png)

Если все хорошо и адаптер появился, то проверяем связть до удаленной машины командой
```bash
ping 10.10.10.1
```
- если проверяете на HQ-RTR, то вводите в команде <code>ping</code> адрес 10.10.10.2
- если проверяете на BR-RTR, то вводите в команде <code>ping</code> адрес 10.10.10.1

![image](https://github.com/kaktotad/demo-sys-2026/blob/ea6ebb25ac892b0edfdf8a1102a7bc06c0dcc3d6/images/GRE_ping_BR_to_HQ.png)

</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="ospf"></a>
## ✔️ 8. Настройка OSPF 

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>

Обеспечьте динамическую маршрутизацию на маршрутизаторах HQ-RTR и BR-RTR:

сети одного офиса должны быть доступны из другого офиса и наоборот. Для обеспечения динамической маршрутизации.
Используйте link state протокол на усмотрение участника:

- Разрешите выбранный протокол только на интерфейсах ip туннеля
- Маршрутизаторы должны делиться маршрутами только друг с другом
- Обеспечьте защиту выбранного протокола посредством парольной
защиты
- Сведения о настройке и защите протокола занесите в отчёт
</details>

<details>
<summary>Настройка OSPF на <strong>HQ-RTR</strong></summary>
<h3>HQ-RTR</h3>

**1.** Устанавливаем пакет frr
```bash
apt install frr -y
```

**2.** В конфигурационном файле <code>/etc/frr/daemons</code> необходимо активировать выбранный протокол `OSPF` для дальнейшей реализации его настройки:
```bash
nano /etc/frr/daemons
```
Находим строку <code>ospfd = no</code> и меняем занчение на <code>yes</code></br>
![image](https://github.com/kaktotad/demo-sys-2026/blob/bedcd28dc3b815d63b58cf2afcdede1a7cddc943/images/frr_daemons_on.png)
</br>
**3.** Далее перезагружаем и добавляем в автозагрузку службу <strong>FRR</strong> командой
```bash
systemctl restart frr && systemctl enable --now frr
```
**4.** Переходим в интерфейс управления симуляцией **`FRR`** командой:
```bash
vtysh
```

**5.** Пишем команды для настройки **маршрутизации:**

Входим в режим глобальной кофигурации
```bash
conf t
```
Переходим в режим конфигурации OSPFv2
```bash
router ospf
```
Переводим все интерфейсы в пассивный режим
```bash
passive-interface default
```
Объявляем локальную сети офиса HQ (сеть VLAN100, VLAN200, VLAN999) и сеть (GRE-туннеля)
```bash
network 192.168.1.0/27 area 0
network 192.168.2.0/28 area 0
network 192.168.3.0/29 area 0
network 10.10.10.0/30 area 0
```
Включаем аутентификацию для области
```bash
area 0 authentication
```
Выходим из режима конфигурации OSPFv2
```bash
exit
```
Переходим в режим конфигурирования интерфейса **gre1**
```bash
int gre1
```
Туннельный интерфейс **gre1** делаем активным, для установления соседства с BR-RTR и обмена внутренними маршрутами. Разрешаем выбранный протокол только на интерфейсах ip туннеля
```bash
no ip ospf network broadcast
```
Переводим интерфейс **gre1** в активный режим. Разрешаем выбранный протокол только на интерфейсах ip туннеля
```bash
no ip ospf passive
```
Включаем аутентификацию с открытым паролем <code>P@ssw0rd</code>
```bash
ip ospf authentication
ip ospf authentication-key password
```
Выходим из режима конфигурации и gre1 и режима глобальной конфигурации
```bash
exit
exit
```
Созраняем текущую конфигурацию
```bash
write
```
Для выхода из <code>vtysh</code>
```
exit
```
Перезапускаем frr
```bash
systemctl restart frr
```

</details>

<details>
<summary>Настройка OSPF на <strong>BR-RTR</strong></summary>
<h3>BR-RTR</h3>

**1.** Устанавливаем пакет frr
```bash
apt install frr -y
```

**2.** В конфигурационном файле <code>/etc/frr/daemons</code> необходимо активировать выбранный протокол `OSPF` для дальнейшей реализации его настройки:
```bash
nano /etc/frr/daemons
```
Находим строку <code>ospfd = no</code> и меняем занчение на <code>yes</code></br>
![image](https://github.com/kaktotad/demo-sys-2026/blob/bedcd28dc3b815d63b58cf2afcdede1a7cddc943/images/frr_daemons_on.png)
</br>
**3.** Далее перезагружаем и добавляем в автозагрузку службу <strong>FRR</strong> командой
```bash
systemctl restart frr && systemctl enable --now frr
```
**4.** Переходим в интерфейс управления симуляцией **`FRR`** командой:
```bash
vtysh
```

**5.** Пишем команды для настройки **маршрутизации:**

Входим в режим глобальной кофигурации
```bash
conf t
```
Переходим в режим конфигурации OSPFv2
```bash
router ospf
```
Переводим все интерфейсы в пассивный режим
```bash
passive-interface default
```
Объявляем локальную сети офиса HQ (сеть VLAN100, VLAN200, VLAN999) и сеть (GRE-туннеля)
```bash
network 192.168.4.0/28 area 0
network 10.10.10.0/30 area 0
```
Включаем аутентификацию для области
```bash
area 0 authentication
```
Выходим из режима конфигурации OSPFv2
```bash
exit
```
Переходим в режим конфигурирования интерфейса **gre1**
```bash
int gre1
```
Туннельный интерфейс **gre1** делаем активным, для установления соседства с BR-RTR и обмена внутренними маршрутами. Разрешаем выбранный протокол только на интерфейсах ip туннеля
```bash
no ip ospf network broadcast
```
Переводим интерфейс **gre1** в активный режим. Разрешаем выбранный протокол только на интерфейсах ip туннеля
```bash
no ip ospf passive
```
Включаем аутентификацию с открытым паролем <code>P@ssw0rd</code>
```bash
ip ospf authentication
ip ospf authentication-key password
```
Выходим из режима конфигурации и gre1 и режима глобальной конфигурации
```bash
exit
exit
```
Созраняем текущую конфигурацию
```bash
write
```
Для выхода из <code>vtysh</code>
```
exit
```
Перезапускаем frr
```bash
systemctl restart frr
```

</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---
<a id="lan-nat"></a>
## ✔️ 9. Настройка NAT на HQ-RTR и BR-RTR 

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>

Настройка динамической трансляции адресов маршрутизаторах HQ-RTR и BR-RTR:

- Настройте динамическую трансляцию адресов для обоих офисов в сторону ISP, все устройства в офисах должны иметь доступ к сети Интернет
</details>

<details>
<summary><stronge>Данное задание выполняется на <code>HQ-RTR</code> и <code>BR-RTR</code></stronge></summary>

<h3>HQ-RTR</h3>
Открываем файл <code>/etc/nftables.conf</code>

```bash
nano /etc/nfatbles.conf
```
Прописываем следующие строки
```bash
table inet nat{
  chain POSTROUTING {
  type nat hook postrouting priority srcnat;
  oifname "ens192" masquerde
  }
}
```

![image](https://github.com/kaktotad/demo-sys-2026/blob/63733d2e985d05425c3355da47b0fec4c8f23d2b/images/nat_RTR_net.png)

где <strong>ens192</strong> - интерфейс HQ-RTR смотрящий к ISP

Перезапускаем службу nftables
```bash
systemctl restart nftables
```
и добавляем в автозагрузку
```bash
systemctl enable --now nftables
```

<h3>BR-RTR</h3>
Открываем файл <code>/etc/nftables.conf</code>

```bash
nano /etc/nfatbles.conf
```

Прописываем следующие строки
```bash
table inet nat{
  chain POSTROUTING {
  type nat hook postrouting priority srcnat;
  oifname "ens192" masquerde
  }
}
```

![image](https://github.com/kaktotad/demo-sys-2026/blob/63733d2e985d05425c3355da47b0fec4c8f23d2b/images/nat_RTR_net.png)

где <strong>ens192</strong> - интерфейс HQ-RTR смотрящий к ISP

Перезапускаем службу nftables
```bash
systemctl restart nftables
```
и добавляем в автозагрузку
```bash
systemctl enable --now nftables
```

</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="dhcp"></a>
## ✔️ 10. Настройка DHCP 

<details>
<summary>ОПИСАНИЕ ЗАДАНАИЯ</summary>

Настройте протокол динамической конфигурации хостов для сети в
сторону HQ-CLI:
- Настройте нужную подсеть
- В качестве сервера DHCP выступает маршрутизатор HQ-RTR
- Клиентом является машина HQ-CLI
- Исключите из выдачи адрес маршрутизатора
- Адрес шлюза по умолчанию – адрес маршрутизатора HQ-RTR
- Адрес DNS-сервера для машины HQ-CLI – адрес сервера HQ-SRV
- DNS-суффикс – au-team.irpo
- Сведения о настройке протокола занесите в отчёт.
</details>
<details>
<summary>Данное задание выполняется на <strong>HQ-RTR</strong> и <strong>HQ-CLI</strong></summary>
<h3>HQ-RTR</h3>

Перед выполнением данного задания необходимо проверить содержание файла <code>/etc/resolv.conf</code>
```bash
nano /etc/resolv.conf
```
и прописать

```bash
nameserver 8.8.8.8
```

Стало: ![image](https://github.com/kaktotad/demo-sys-2026/blob/567cb803758ce096cef66c19297bcfc8c7e4d335/images/resolv_google.png)

Первым шагом необходимо скачать пакет DHCP-сервера командой
```bash
apt install isc-dhcp-server -y
```
<strong>При установке сервер выдаст ошибки, это нормально!!!</strong>
После установки преходим в конфигурацию файла <code>/etc/dhcp/dhcpd.conf</code>
```bash
nano /etc/dhcp/dhcpd.conf -l
```
Листаем файл до 50 строки и раскомментируем блок вставляя свои занчения
```bash
subnet 192.168.200.0 netmask 255.255.255.240 {
  range 192.168.200.2 192.168.200.14;
  option domain-name-servers 192.168.100.62;
  option domain-name "au-team.irpo";
  option routers 192.168.200.1;
}
```
Стало:

![image](https://github.com/kaktotad/demo-sys-2026/blob/6a09aaf11983c8e93bcd2609f23e64e802393953/images/conf_DHCP-server.png)

после чего переходим в конфигурацию файла <code>/etc/default/isc-dhcp-server</code> и меняем ее, добавляя данный текст:
```bash
INTERFACESv4="ens224:1"
```
Включаем сервиc **`DHCP`** и добавляем в автозагрузку на **`HQ-RTR`**:
```
systemctl start isc-dhcp-server
systemctl enable --now isc-dhcp-server
```

<h3>HQ-CLI</h3>

Далее на HQ-CLI необходимо в настройках сет.адаптера выбрать режим **DHCP** и проверить работоспособность
переходим в файл 

```bash
nano /etc/network/interfaces
```

и меняем старый конфиг

```bash
auto ens192
iface ens192 inet static
address 192.168.200.2
netmask 255.255.255.240
gateway 192.168.200.1
dns-nameservers 192.168.100.2
```

на этот
```bash
auto ens192
iface ens192 inet dhcp
```
Далее перезапускаем сетевой интерфейс 
```bahs
systemctl restart networking
```
и проверяем адрес
```bash
ip -c a
```

</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="dns"></a>
## ❌ 11. Настройка DNS

<details>
<summary>ОПИСАНИЕ ЗАДАНИЯ</summary>
Настройте инфраструктуру разрешения доменных имён для офисов HQ и BR:
- Основной DNS-сервер реализован на HQ-SRV
- Сервер должен обеспечивать разрешение имён в сетевые адреса устройств и обратно в соответствии с таблицей 3

- В качестве DNS сервера пересылки используйте любой общедоступный DNS сервер(77.88.8.7, 77.88.8.3 или другие)
</details>
<details>


</details>

[↑ Вернуться к навигации](#-навигация-по-методичке)

---

<a id="время"></a>
## ❌ 12. Настройка часового пояса

*текст*

[↑ Вернуться к навигации](#-навигация-по-методичке)

