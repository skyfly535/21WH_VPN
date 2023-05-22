# Мосты, туннели и VPN.

Задание:

1. Между двумя виртуалками поднять vpn в режимах:
  - tun

  - tap

  Описать в чём разница, замерить скорость между виртуальными
  машинами в туннелях, сделать вывод об отличающихся показателях
  скорости.

2. Поднять RAS на базе OpenVPN с клиентскими сертификатами,
подключиться с локальной машины на виртуалку.

Цель:

Создать домашнюю сетевую лабораторию. Научится настраивать мосты и туннели между сетями.
Разобраться в различных типах реализаций мостов и туннелей.

## Развертывание стенда для демострации статической и динамической маршрутизации, OSPF.

Стэнд состоит из хостовой машины под управлением ОС Ubuntu 20.04 на которой развернут Ansible и двух виртуальных машин `centos/7`.

Машины с именами: `server` и `client`.

Разворачиваем инфраструктуру в Vagrant исключительно через Ansible.

Первая часть задания:

Для каждой ВМ я создал отдельный `server.conf` (server.conf, server1.conf соответственно) и привел всоответствие Playbook `vpn.yml`. Так же в каталоге с Playbook должен находится файл для service unit `openvpn@.service`.

Вторая часть задания:

Для ВМ я создал отдельный `server.conf` - `server2.conf`. В каталог к файлам из первой части задания необходимо поместить Playbook `vpn1.yml` и файл клиента OpenVPN `client.conf` для проверки корректности работы стенда.


Все коментарии по каждому блоку указаны в текстах Playbook - `vpn.yml` и `vpn1.yml`.

Выполняем установку стенда

```
vagrant up
```

## Результат работы

### TAP

TAP эмулирует Ethernet устройство и работает на канальном уровне модели OSI, оперируя кадрами Ethernet. Работает broadcast, multicast и loopback.

Проверяем:

на openvpn сервере запускаем iperf3 в режиме сервера
```
[root@server vagrant]# iperf3 -s &
[1] 4225
[root@server vagrant]# -----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 10.10.10.2, port 53178
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 53180
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  20.2 MBytes   169 Mbits/sec                  
[  5]   1.00-2.00   sec  22.7 MBytes   191 Mbits/sec                  
[  5]   2.00-3.00   sec  23.0 MBytes   193 Mbits/sec                  
[  5]   3.00-4.00   sec  23.5 MBytes   197 Mbits/sec                  
[  5]   4.00-5.00   sec  23.1 MBytes   194 Mbits/sec                  
[  5]   5.00-6.00   sec  24.6 MBytes   207 Mbits/sec                  
[  5]   6.00-7.00   sec  23.9 MBytes   200 Mbits/sec                  
[  5]   7.00-8.00   sec  23.6 MBytes   198 Mbits/sec                  
[  5]   8.00-9.00   sec  23.2 MBytes   194 Mbits/sec                  
[  5]   9.00-10.00  sec  23.8 MBytes   199 Mbits/sec                  
[  5]  10.00-11.00  sec  23.9 MBytes   200 Mbits/sec                  
[  5]  11.00-12.00  sec  23.6 MBytes   198 Mbits/sec                  
[  5]  12.00-13.00  sec  25.1 MBytes   210 Mbits/sec                  
[  5]  13.00-14.00  sec  23.7 MBytes   199 Mbits/sec                  
[  5]  14.00-15.00  sec  24.2 MBytes   203 Mbits/sec                  
[  5]  15.00-16.00  sec  23.3 MBytes   196 Mbits/sec                  
[  5]  16.00-17.00  sec  23.2 MBytes   195 Mbits/sec                  
[  5]  17.00-18.00  sec  23.7 MBytes   199 Mbits/sec                  
[  5]  18.00-19.00  sec  24.4 MBytes   205 Mbits/sec                  
[  5]  19.00-20.00  sec  23.6 MBytes   198 Mbits/sec                  
[  5]  20.00-21.00  sec  23.7 MBytes   199 Mbits/sec                  
[  5]  20.00-21.00  sec  23.7 MBytes   199 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-21.00  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-21.00  sec   501 MBytes   200 Mbits/sec                  receiver
iperf3: the client has terminated
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
```
на openvpn клиенте запускаем iperf3 в режиме клиента и замеряем
скорость
```
[root@client vagrant]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  4] local 10.10.10.2 port 53180 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec   115 MBytes   193 Mbits/sec   55    306 KBytes       
[  4]   5.00-10.00  sec   120 MBytes   201 Mbits/sec   34    341 KBytes       
[  4]  10.00-15.00  sec   120 MBytes   201 Mbits/sec   12    277 KBytes       
[  4]  15.00-20.00  sec   119 MBytes   199 Mbits/sec    4    321 KBytes       
^C[  4]  20.00-21.23  sec  29.4 MBytes   202 Mbits/sec    0    381 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-21.23  sec   503 MBytes   199 Mbits/sec  105             sender
[  4]   0.00-21.23  sec  0.00 Bytes  0.00 bits/sec                  receiver
iperf3: interrupt - the client has terminated
```
### TUN

TUN (сетевой туннель) работает на сетевом уровне модели OSI, оперируя IP пакетами в следствии чего broadcast и multicast не работают. 


Проверяем:

на openvpn сервере запускаем iperf3 в режиме сервера
```
[root@server ~]# iperf3 -s &
[1] 4882
[root@server ~]# -----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 10.10.10.2, port 56470
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 56472
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.01   sec  20.4 MBytes   170 Mbits/sec                  
[  5]   1.01-2.00   sec  22.6 MBytes   191 Mbits/sec                  
[  5]   2.00-3.00   sec  24.1 MBytes   202 Mbits/sec                  
[  5]   3.00-4.00   sec  24.7 MBytes   207 Mbits/sec                  
[  5]   4.00-5.00   sec  23.5 MBytes   197 Mbits/sec                  
[  5]   5.00-6.00   sec  23.2 MBytes   194 Mbits/sec                  
[  5]   6.00-7.00   sec  25.0 MBytes   210 Mbits/sec                  
[  5]   7.00-8.00   sec  24.1 MBytes   202 Mbits/sec                  
[  5]   8.00-9.00   sec  21.2 MBytes   178 Mbits/sec                  
[  5]   9.00-10.00  sec  24.2 MBytes   203 Mbits/sec                  
[  5]  10.00-11.00  sec  24.8 MBytes   208 Mbits/sec                  
[  5]  11.00-12.00  sec  25.0 MBytes   210 Mbits/sec                  
[  5]  12.00-13.00  sec  23.2 MBytes   194 Mbits/sec                  
[  5]  13.00-14.00  sec  24.7 MBytes   207 Mbits/sec                  
[  5]  14.00-15.00  sec  24.7 MBytes   207 Mbits/sec                  
[  5]  15.00-16.00  sec  24.5 MBytes   206 Mbits/sec                  
[  5]  16.00-17.00  sec  24.5 MBytes   205 Mbits/sec                  
[  5]  17.00-18.00  sec  23.7 MBytes   198 Mbits/sec                  
[  5]  18.00-19.00  sec  23.9 MBytes   200 Mbits/sec                  
[  5]  19.00-20.00  sec  23.4 MBytes   196 Mbits/sec                  
[  5]  20.00-21.00  sec  24.4 MBytes   205 Mbits/sec                  
[  5]  20.00-21.00  sec  24.4 MBytes   205 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-21.00  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-21.00  sec   505 MBytes   202 Mbits/sec                  receiver
iperf3: the client has terminated
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
```

на openvpn клиенте запускаем iperf3 в режиме клиента и замеряем
скорость в туннеле

```
[vagrant@client ~]$ sudo -i
[root@client ~]# 
[root@client ~]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  4] local 10.10.10.2 port 56472 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.01   sec   118 MBytes   197 Mbits/sec   41    394 KBytes       
[  4]   5.01-10.00  sec   118 MBytes   198 Mbits/sec  163    301 KBytes       
[  4]  10.00-15.00  sec   122 MBytes   205 Mbits/sec    8    333 KBytes       
[  4]  15.00-20.01  sec   120 MBytes   202 Mbits/sec   93    214 KBytes       
^C[  4]  20.01-21.20  sec  29.0 MBytes   204 Mbits/sec    0    296 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-21.20  sec   507 MBytes   201 Mbits/sec  305             sender
[  4]   0.00-21.20  sec  0.00 Bytes  0.00 bits/sec                  receiver
iperf3: interrupt - the client has terminated
```
Разницы между замерами не видно, так как наш стенд это виртуальные машины. В `tun` для связи между хоставми необходима маршрутизация, то есть сетевой уровень связности (L3), а в `tap` возможно объединить только при наличии канальной связности устройств (L2). В условия вирьуальной среды 2 и 3 уровни модели OSI сглажены.

### RAS на базе OpenVPN

Слегка доработаем `Vagrantfile` из первой части ДЗ

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
    config.vm.define "server" do |server| # конфиг сервера
      server.vm.hostname = "server.loc"
      server.vm.network "private_network", ip: "192.168.56.10"
    end

    config.vm.provision "ansible" do |ansible|  # вызываем playbook для развертывания нужной инфраструктуры на машинах
      ansible.verbose = "v"
      ansible.playbook = "vpn1.yml" # файл playbook
    end

  end
```
и стартуем занаво

```
vagrant up
```
после развертывания стенда пробуем пингануть внутренний IP адрес

сервера в туннеле:
```
root@root-ubuntu:/home/roman/project# ping -c 4 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
^C
--- 10.10.10.1 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1005ms
```
сервер не отвечает.

После этого на хосте выполняем

```
sudo openvpn --config client.conf
```
и повторяем попытку

```
root@root-ubuntu:/home/roman/project# ping -c 4 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.502 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=1.33 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=1.21 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=1.34 ms
--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3018ms
rtt min/avg/max/mdev = 0.502/1.097/1.344/0.347 ms
```
Также проверяем на хостовой машине, что сеть туннеля импортирована в таблицу маршрутизации.

```
root@root-ubuntu:/home/roman/project# netstat -rn
Таблица маршутизации ядра протокола IP
Destination Gateway Genmask Flags MSS Window irtt Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG        0 0          0 wlp0s20f3
10.10.10.0      10.10.10.5      255.255.255.0   UG        0 0          0 tun0
10.10.10.5      0.0.0.0         255.255.255.255 UH        0 0          0 tun0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 wlp0s20f3
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U         0 0          0 br-c3b7f18b4168
192.168.1.0     0.0.0.0         255.255.255.0   U         0 0          0 wlp0s20f3
192.168.20.0    0.0.0.0         255.255.255.0   U         0 0          0 vboxnet5
192.168.30.0    0.0.0.0         255.255.255.0   U         0 0          0 vboxnet4
192.168.40.0    0.0.0.0         255.255.255.0   U         0 0          0 vboxnet3
192.168.50.0    0.0.0.0         255.255.255.0   U         0 0          0 vboxnet2
192.168.56.0    0.0.0.0         255.255.255.0   U         0 0          0 vboxnet0
```
