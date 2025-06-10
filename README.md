# Network

Описание 

Целью данной конфигурации было развернуть виртуальную сеть с маршрутизаторами и серверами, обеспечив взаимодействие между офисами `Office1` и `Office2`, а также их доступ в интернет.

Сеть включает:

`inetRouter`: подключение к интернету и NAT для внутренней сети.
`centralRouter`: маршрутизатор центрального офиса, соединяющий
`Office1`, `Office2` и интернет. `office1Router`, `office2Router`: маршрутизаторы офисов, обеспечивающие связь серверов с центральным маршрутизатором.
`centralServer`, `office1Server`, `office2Server`: серверы, подключенные к своим маршрутизаторам.
Выполняем `vagrant up`

Вклаем `IP forwarding` на всех маршрутизаторах: `sysctl -w net.ipv4.ip_forward=1`

Настраиваем маршруты на `centralRouter` для соединения офисов: `sudo ip route add 192.168.1.0/25 via 192.168.1.1` `sudo ip route add 192.168.2.0/26 via 192.168.2.1`

Настраиваем маршруты на `office1Router` и `office2Router` для связи с центральным маршрутизатором:

На `office1Router` `sudo ip route add 192.168.1.0/25 via 192.168.2.1` На `office2Router` `sudo ip route add 192.168.2.0/26 via 192.168.1.1`

Настраиваем NAT `inetRouter`: `sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`

Убираем конфликтующие маршруты через NAT на серверах `office1Server` и `office2Server`

На `office1Server` `sudo ip route del default via 10.0.2.2` `sudo ip route add default via 192.168.2.1` На `office2Server` `sudo ip route del default via 10.0.2.2` `sudo ip route add default via 192.168.1.1`

[vagrant@office1Server ~]$ ping 192.168.1.3 PING 192.168.1.3 (192.168.1.3) 56(84) bytes of data. 64 bytes from 192.168.1.3: icmp_seq=1 ttl=63 time=2.38 ms 64 bytes from 192.168.1.3: icmp_seq=2 ttl=63 time=1.51 ms 64 bytes from 192.168.1.3: icmp_seq=3 ttl=63 time=1.55 ms 64 bytes from 192.168.1.3: icmp_seq=4 ttl=63 time=1.41 ms ^C --- 192.168.1.3 ping statistics --- 4 packets transmitted, 4 received, 0% packet loss, time 3008ms rtt min/avg/max/mdev = 1.413/1.716/2.384/0.390 ms

[vagrant@office2Server ~]$ ping 192.168.2.3 PING 192.168.2.3 (192.168.2.3) 56(84) bytes of data. 64 bytes from 192.168.2.3: icmp_seq=1 ttl=63 time=1.88 ms 64 bytes from 192.168.2.3: icmp_seq=2 ttl=63 time=1.73 ms 64 bytes from 192.168.2.3: icmp_seq=3 ttl=63 time=1.36 ms 64 bytes from 192.168.2.3: icmp_seq=4 ttl=63 time=1.74 ms ^C --- 192.168.2.3 ping statistics --- 4 packets transmitted, 4 received, 0% packet loss, time 3012ms rtt min/avg/max/mdev = 1.360/1.679/1.885/0.200 ms

[vagrant@office1Server ~]$ ping 192.168.0.2 PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data. 64 bytes from 192.168.0.2: icmp_seq=1 ttl=63 time=2.29 ms 64 bytes from 192.168.0.2: icmp_seq=2 ttl=63 time=1.39 ms 64 bytes from 192.168.0.2: icmp_seq=3 ttl=63 time=1.84 ms

### Теоритическая часть
1. Свободные подсети
   
| Подсеть            | Маска | Адреса  |
| ------------------ | ----- | ------- |
| `192.168.0.16/28`  | /28   | 16–31   |
| `192.168.0.48/28`  | /28   | 48–63   |
| `192.168.0.128/25` | /25   | 128–255 |
| Подсеть            | Маска | Адреса |
| -----------------  | ----- | ------ |
| `192.168.1.64/26`  | /26   | 64–127 |

| Подсеть            | Маска | Комментарий                    |
| ------------------ | ----- | ------------------------------ |
| `192.168.3.0/24`   | /24   | полностью свободна             |
| `192.168.4.0/24`   | /24   | свободна                       |
| …                  |       | … до `192.168.254.0/24`        |
| `192.168.255.4/30` | /30   | свободна (после занятой `/30`) |


2. Посчитать сколько узлов в каждой подсети, включая свободные
   
| Подсеть          | Маска | Узлов всего | Broadcast     |
| ---------------- | ----- | ----------- | ------------- |
| 192.168.255.0/30 | /30   | 2           | 192.168.255.3 |
| 192.168.0.0/28   | /28   | 14          | 192.168.0.15  |
| 192.168.0.32/28  | /28   | 14          | 192.168.0.47  |
| 192.168.0.64/26  | /26   | 62          | 192.168.0.127 |
| 192.168.2.0/26   | /26   | 62          | 192.168.2.63  |
| 192.168.2.64/26  | /26   | 62          | 192.168.2.127 |
| 192.168.2.128/26 | /26   | 62          | 192.168.2.191 |
| 192.168.2.192/26 | /26   | 62          | 192.168.2.255 |
| 192.168.1.0/26   | /26   | 62          | 192.168.1.63  |
| 192.168.1.128/26 | /26   | 62          | 192.168.1.191 |
| 192.168.1.192/26 | /26   | 62          | 192.168.1.255 |

`Нет ошибок при разбиение`
