# 1. Произведите базовую настройку устройств:
### 1.1. Настройте имена устройств согласно топологии. Используйте полное доменное имя.

Для HQ-SRV, BR-SRV
```bash
   hostnamectl set-hostname
   systemctl restart network
   reboot
   ```
---
# Базовая настройка устройств
### **HQ-SRV**
```bash
echo BOOTPROTO=static >> /etc/net/ifaces/ens18/options
echo TYPE=eth >> /etc/net/ifaces/ens18/options
echo 192.168.100.2/25 > /etc/net/ifaces/ens18/ipv4address
echo nameserver 8.8.8.8 > /etc/net/ifaces/ens18/resolv.conf
echo default via 192.168.100.1 > /etc/net/ifaces/ens18/ipv4route
systemctl restart network
```
#### !!!Не забыть сохранить, внимательно смотреть IP-адрес!!!

### **BR-SRV**

```bash
echo BOOTPROTO=static >> /etc/net/ifaces/ens18/options
echo TYPE=eth >> /etc/net/ifaces/ens18/options
echo 192.168.0.2/25 > /etc/net/ifaces/ens18/ipv4address
echo nameserver 8.8.8.8 > /etc/net/ifaces/ens18/resolv.conf
echo default via 192.168.0.1 > /etc/net/ifaces/ens18/ipv4route
systemctl restart network
```
#### !!!Не забыть сохранить, внимательно смотреть IP-адрес!!!
---
# 3. Создание локальных учетных записей 
### 3.1. Создайте пользователя cyberpanda на серверах HQ-SRV и BR-SRV
### 3.1.1. Пароль пользователя  cyberpanda с паролем P@ssw0rd
### 3.1.2. Идентификатор пользователя 1001
### 3.1.3. Пользователь cyberpanda должен иметь возможность запускать sudo без дополнительной аутентификации.
#### *Вводим следующие команды на HQ-SRV и BR-SRV*
```bash
useradd quantumquokka -u 1031
passwd quantumquokka
usermod -aG wheel quantumquokka
echo "quantumquokka ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
systemctl restart network
```
#### *!!!Обращаем внимание на имя пользователя, идентификатор. Не забываем все сохранить!!!*
---
# 4. Настройка безопасного удаленного доступа на серверах HQ-SRV и BR-SRV: 
### 4.1. Для подключения используйте порт 2020
### 4.2. Разрешите подключения только пользователю cyberpanda
### 4.3. Ограничьте количество попыток входа до двух
### 4.4. Настройте баннер «Authorized access only»
#### *Вводим следующие команды на HQ-SRV и BR-SRV*
```bash
echo Port 2030 >> /etc/openssh/sshd_config
echo MaxAuthTries 2 >> /etc/openssh/sshd_config
echo AllowUsers cloudcougar >> /etc/openssh/sshd_config
echo Banner /root/banner >> /etc/openssh/sshd_config
echo "Authorized access only" > /root/banner
systemctl restart network
```
#### *!!!Обращаем внимание на имя пользователя, а так же на порт. Не забываем все сохранить!!!*
---
# 10. Настройте часовой пояс на всех устройствах, согласно месту проведения экзамена. 
- *Для HQ-SRV и BR-SRV вводим следующие команды*
```bash
apt-get reinstall tzdata
timedatectl set-timezone Asia/Yekaterinburg
timedatectl status
systemctl restart network
```
