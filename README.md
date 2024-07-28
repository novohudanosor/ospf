# ospf
Работа с протоколом OSPF
1. Разворачиваем 3 виртуальные машины
2. Так как мы планируем настроить OSPF, все 3 виртуальные машины должны быть соединены между собой (разными VLAN), а также иметь одну (или несколько) доолнительных сетей, к которым, далее OSPF сформирует маршруты. Исходя из данных требований, мы можем нарисовать топологию сети:
3. ![alt text](./Pictures/OSPF.png)
4. Настройка OSPF.
Отключение фаерволла ufw на всех ВМ:
```
sudo systemctl stop ufw
sudo systemctl disable ufw
```
5. Установка FRR:
```
sudo apt update
sudo apt install frr frr-pythontools -y
```
6. Разрешение маршрутизации транзитных пакетов на всех ВМ:
```
sudo sysctl net.ipv4.conf.all.forwarding=1
```
7. Подключение демона osfpd в FRR:
```
nano /etc/frr/daemons
...
cat /etc/frr/daemons
zebra=yes
ospfd=yes
bgpd=no
ospf6d=no
...
```
8. Настройка OSFP с помощью правки конфигурационного файла /etc/frr/frr.conf:
nano /etc/frr/frr.conf
...
cat /etc/frr/frr.conf
frr version 8.1
frr defaults traditional

hostname router1
log syslog informational
no ipv6 forwarding
service integrated-vtysh-config
!
interface enp0s8
 description r1-r2
 ip address 10.0.10.1/30
 ip ospf mtu-ignore
 !ip ospf cost 1000
 ip ospf hello-interval 10
 ip ospf dead-interval 30
!
interface enp0s9
 description r1-r3
 ip address 10.0.12.1/30
 ip ospf mtu-ignore
 !ip ospf cost 45
 ip ospf hello-interval 10
 ip ospf dead-interval 30
!
interface enp0s10
 description net_router1
 ip address 192.168.10.1/24
 ip ospf mtu-ignore
 !ip ospf cost 45
 ip ospf hello-interval 10
 ip ospf dead interval 30
!
router ospf
 router-id 1.1.1.1
 network 10.0.10.0/30 area 0
 network 10.0.12.0/30 area 0
 network 192.168.10.0/24 area 0
 neighbor 10.0.10.2
 neighbor 10.0.12.2   
!
log file /var/log/frr/frr.log
default-information originate always
``` 
