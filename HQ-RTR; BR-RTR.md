# 1.1. Настройте имена устройств согласно топологии. Используйте полное доменное имя.

```bash
conf t
hostname
   end
   wr mem
```
---
# Базовая настройка устройств

### **HQ-RTR**
-
     *Настройка интерфейса, который идет к ISP*

```bash
ip route 0.0.0.0 0.0.0.0 172.16.4.1
int isp
ip address 172.16.4.2/28
no sh
exit
port te0
service-instance 1
enc untagged
con ip int isp
end
wr mem
```
#### !!!Не забыть сохранить, внимательно смотреть IP-адрес!!!
-
    *Настройка интерфейса в сторону SRV (Vlan100)*

```bash
int srv
ip add 192.168.100.1/27
no sh
exit
port te1
service-instance 100
en dot1q 100
rewrite pop 1
con ip int srv
end
wr mem
```
#### !!!Не забыть сохранить, внимательно смотреть IP-адрес!!!
-
     *Настройка интерфейса в сторону CLI (Vlan200)*

```bash
int cli
ip add 192.168.200.1/26
no sh
exit
port te1
service-instance 200
en dot1q 200
rewrite pop 1
con ip int cli
end
wr mem
```
#### !!!Не забыть сохранить, внимательно смотреть IP-адрес!!!
-
   *Настройка Vlan999*

```bash
int enp1.999
ip add 192.168.99.1/27
no sh
exit
port te1
service-instance 999
en dot1q 999
rewrite pop 1
con ip int enp1.999
end
wr mem
```
#### !!!Не забыть сохранить, внимательно смотреть IP-адрес!!!

### **BR-RTR**
-
   *Настройка интерфейса, который идет к ISP*

```bash
ip route 0.0.0.0 0.0.0.0 172.16.5.1
int isp
ip address 172.16.5.2/28
no sh
exit
port te0
service-instance 1
enc untagged
con ip int isp
end
wr mem
```
#### !!!Не забыть сохранить, внимательно смотреть IP-адрес!!!
-
   *Настройка интерфейса в сторону SRV*

```bash
int srv
ip add 192.168.0.1/25
no sh
exit
port te1
service-instance 1
enc untagged
con ip int srv
end
wr mem
```
#### !!!Не забыть сохранить, внимательно смотреть IP-адрес!!!
---
### 3.2. Создайте пользователя routerfox на маршрутизаторах HQ-RTR и BR-RTR
### 3.2.1. Пароль пользователя routerfox с паролем P@$$word
### 3.2.2. При настройке на EcoRouter пользователь routerfox должен обладать максимальными привилегиями 
#### *Вводим следующие команды на HQ-RTR и BR-RTR*
```bash
username fwflamingo
password P@ssw0rd
role admin
end
wr mem
```
#### *!!!Обращаем внимание на имя пользователя. Не забываем все сохранить!!!*

---
## 5. Между офисами HQ и BR необходимо сконфигурировать ip туннель
### 5.1. Сведения о туннеле занесите в отчёт
### 5.2. На выбор технологии GRE или IP in IP
#### *Рассмотрим GRE*
- *Вводим следующие команды на BR-RTR*
```bash
int tunnel.0
ip add 10.10.10.2/30
ip mtu 1400
ip tunnel 172.16.5.2 172.16.4.2 mode gre
end
wr mem
```
- *Вводим следующие команды на HQ-RTR*
```bash
int tunnel.0
ip add 10.10.10.1/30
ip mtu 1400
ip tunnel 172.16.4.2 172.16.5.2 mode gre
end
wr mem
```
---
## 6. Обеспечьте динамическую маршрутизацию: ресурсы одного офиса должны быть доступны из другого офиса. Для обеспечения динамической маршрутизации используйте link state протокол на ваше усмотрение.
#### 6.1. Разрешите выбранный протокол только на интерфейсах в ip туннеле 
#### 6.2. Маршрутизаторы должны делиться маршрутами только друг с другом
#### 6.3. Обеспечьте защиту выбранного протокола посредством парольной защиты 
#### 6.4. Сведения о настройке и защите протокола занесите в отчёт

- *Вводим следующие команды на BR-RTR*
```bash
router ospf 1
router-id 1.1.1.1
network 10.10.10.0/30 area 0
network 192.168.0.0/25 area 0
passive-interface default
no passive-interface tunnel.0
int tunnel.0
ip ospf authentication-key ecorouter
end
wr mem
```
- *Вводим следующие команды на HQ-RTR*
```bash
router ospf 1
router-id 2.2.2.2
network 10.10.10.0/30 area 0
network 192.168.100.0/27 area 0
network 192.168.200.0/26 area 0
passive-interface default
no passive-interface tunnel.0
int tunnel.0
ip ospf authentication-key ecorouter
end
wr mem
```
#### *!!!Обращаем внимание на IP-адрес, префикс. Не забываем все сохранить!!!*

---
## 7. Настройка динамической трансляции адресов.
### 7.1. Настройте динамическую трансляцию адресов для обоих офисов.
### 7.2. Все устройства в офисах должны иметь доступ к сети Интернет 
- *Вводим следующие команды на HQ-RTR*
```bash
int srv
ip nat inside 
int cli
ip nat inside 
int isp
ip nat outside
ip nat pool NAT100 192.168.100.1-192.168.100.30
ip nat pool NAT200 192.168.200.1-192.168.200.62
ip nat pool NAT999 192.168.99.1-192.168.99.30
ip nat source dynamic inside-to-outside pool NAT100 overload interface isp
ip nat source dynamic inside-to-outside pool NAT200 overload interface isp
ip nat source dynamic inside-to-outside pool NAT999 overload interface isp
end
wr mem
```
#### *!!!Обращаем внимание на IP-адрес!!!*

- *Вводим следующие команды на BR-RTR*
```bash
int srv
ip nat inside
int isp
ip nat outside
ip nat pool NAT 192.168.0.1-192.168.0.126
ip nat source dynamic inside-to-outside pool NAT overload interface isp
end
wr mem
```
#### *!!!Обращаем внимание на IP-адрес!!!*

---

## 8. Настройка протокола динамической конфигурации хостов.
### 8.1. Настройте нужную подсеть 
### 8.2. Для офиса HQ в качестве сервера DHCP выступает маршрутизатор HQ-RTR.
### 8.3. Клиентом является машина HQ-CLI. 
### 8.4. Исключите из выдачи адрес маршрутизатора 
### 8.5. Адрес шлюза по умолчанию – адрес маршрутизатора HQ-RTR. 
### 8.6. Адрес DNS-сервера для машины HQ-CLI – адрес сервера HQ-SRV. 
### 8.7. DNS-суффикс для офисов HQ – quantumflux.local
### 8.8. Сведения о настройке протокола занесите в отчёт
- *Вводим следующие команды на HQ-RTR*
```bash
ip pool DHCP200 192.168.200.2-192.168.200.62
dhcp-server 1
pool DHCP200 1
mask 255.255.255.192
gateway 192.168.200.1
dns 192.168.200.1
domain-name quark.net
end
conf t
int cli
dhcp-server 1 
end
wr mem
```
#### *!!!Обращаем внимание на IP-адрес!!!*

- *На HQ-CLI проверяем опции*
```bash
cat /etc/net/ifaces/ens18/options
```
- *Если нужно прописываем следующие строчки*
```bash
echo BOOTPROTO=dhcp >> /etc/net/ifaces/ens18/options
echo SYSTEMD_BOOTPROTO=dhcp4 >> /etc/net/ifaces/ens18/options
systemctl restart network
```
- *Проверяем IP-адрес*
```bash
ip add
```
---
# 10. Настройте часовой пояс на всех устройствах, согласно месту проведения экзамена. 
- *Вводим следующие команды на HQ-RTR и BR-RTR*
```bash
conf t
ntp timezone utc+5
do sh clock
end
wr mem
```
