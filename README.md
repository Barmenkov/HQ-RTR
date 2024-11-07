# HQ-RTR
Mac-address
1. vESR = **show interface status**
2. Eco = **show interface**
   Создаем сущность интерфейса и назначаем IP
```
int TO-ISP
ip address 172.16.4.2/28
no shutdown
```
Привязываем созданный интерфейс к физическому протоколу

```
port ge0
service-instance SI-ISP
encapsulation untagged
connect ip interface TO-ISP
```
Создаем интерфейсы, которые будут обрабатывать трафик vlan 100, 200, 999
```
interface HQ-SRV
 ip mtu 1500
 ip address 192.168.0.1/26
!
interface HQ-CLI
 ip mtu 1500
 ip address 192.168.0.65/28
!
interface HQ-MGMT
 ip mtu 1500
 ip address 192.168.0.81/29
!
```
```
port ge1
 mtu 9234
 service-instance ge1/vlan100
  encapsulation dot1q 100
  rewrite pop 1
  connect ip interface HQ-SRV
 service-instance ge1/vlan200
  encapsulation dot1q 200
  rewrite pop 1
  connect ip interface HQ-CLI
 service-instance ge1/vlan999
  encapsulation dot1q 999
  rewrite pop 1
  connect ip interface HQ-MGMT
end
sh ip int br
```
Создаем GRE туннель
```
interface tunnel.1
 ip mtu 1400
 ip address 172.16.1.1/30
 ip tunnel 172.16.4.2 172.16.5.2 mode gre
```

Задаем маршрут по умолчанию в сторону ISP

```
ip route 0.0.0.0/0 172.16.4.1
```

```
router ospf 1
 network 172.16.1.0 0.0.0.3 area 0.0.0.0
 network 192.168.0.0 0.0.0.255 area 0.0.0.0 [Адрес в этой строчке указываем свою сеть. Пример: 10.0.0.0]
!
interface tunnel.1
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 Demo2025
 ip ospf network point-to-point
```
```
sh ip ospf neighbor 
sh ip ospf interface brief
sh ip route
```
```
interface TO-ISP
 ip nat outside
!
interface HQ-SRV
 ip nat inside
!
interface HQ-CLI
 ip nat inside
!
interface HQ-MGMT
 ip nat inside
!
ip nat pool NAT_POOL 192.168.0.1-192.168.0.254 [Пишем пул адресов подсети HQ-SRV. Т.е пишем адрес vlan100 и через тире адрес vlan999 ]
!
ip nat source dynamic inside-to-outside pool NAT_POOL overload interface TO-ISP
!
ping 8.8.8.8
```
# Настройка DHCP на HQ-RTR (EcoRouter)

```
ip pool HQ-NET200 1
 range 192.168.0.66-192.168.0.70
!
dhcp-server 1
 lease 86400
 mask 255.255.255.0
 pool HQ-NET200 1
  dns 192.168.0.2
  domain-name au-team.irpo
  gateway 192.168.0.65
  mask 255.255.255.240
!
interface HQ-CLI
 dhcp-server 1
!
sh dhcp-server clients HQ-CLI
```
* Проверить адрес на HQ-CLI командой **ip a**
```
conf t
username net_admin
password P@ssw0rd
role admin
activate
```
```
conf t
ntp timezone utc+5
```
Проверяем:

```
show ntp timezone
```
