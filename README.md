# Netzwerkkonzept PoC

## Versuchsaufbau

![neetwork-lab](images/network-lab.drawio.png)

## Zustand der VMs ohne OSPF und mit deaktivierter Firewall

### VM1
Netzwerkkonfiguration:
```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 1.1.1.1/32 brd 1.1.1.1 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:13:53:9e brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.16/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 407sec preferred_lft 407sec
    inet6 fe80::4086:4a:e126:c8f7/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:58:97:88:5b brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/24 brd 172.18.0.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:58ff:fe97:885b/64 scope link
       valid_lft forever preferred_lft forever
4: docker1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:2f:37:dd:97 brd ff:ff:ff:ff:ff:ff
    inet 172.18.1.1/24 brd 172.18.1.255 scope global docker1
       valid_lft forever preferred_lft forever
    inet6 fe80::42:2fff:fe37:dd97/64 scope link
       valid_lft forever preferred_lft forever
```

Containers:
```bash
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
6ae61c5a6a48   alpine    "/bin/sh -c 'while t…"   5 minutes ago   Up 5 minutes             container1
584985861dc3   alpine    "/bin/sh -c 'while t…"   5 minutes ago   Up 5 minutes             container0
```

Routing Table:
```bash
default via 10.0.2.1 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.16 metric 100
169.254.0.0/16 dev enp0s3 scope link metric 1000
172.18.0.0/24 dev docker0 proto kernel scope link src 172.18.0.1
172.18.1.0/24 dev docker1 proto kernel scope link src 172.18.1.1
```

### VM2
Netzwerkkonfiguration:
```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 2.2.2.2/32 brd 2.2.2.2 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:c7:4c:8b brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.17/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 513sec preferred_lft 513sec
    inet6 fe80::ec2b:9862:7d5d:470f/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:32:4d:dc:a1 brd ff:ff:ff:ff:ff:ff
    inet 172.18.2.1/24 brd 172.18.2.255 scope global docker2
       valid_lft forever preferred_lft forever
    inet6 fe80::42:32ff:fe4d:dca1/64 scope link
       valid_lft forever preferred_lft forever
```

Containers:
```bash
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
edbc427d279d   alpine    "/bin/sh -c 'while t…"   7 minutes ago   Up 7 minutes             container3
d10bf3b22595   alpine    "/bin/sh -c 'while t…"   7 minutes ago   Up 7 minutes             container2
```

Routing Table:
```bash
default via 10.0.2.1 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.17 metric 100
169.254.0.0/16 dev enp0s3 scope link metric 1000
172.18.2.0/24 dev docker2 proto kernel scope link src 172.18.2.1
```

### Container Ping Test

```bash
# container0 zu contianer1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
64 bytes from 172.18.1.2: seq=0 ttl=63 time=0.143 ms

# container0 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
^C
--- 172.18.2.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container0 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
^C
--- 172.18.2.3 ping statistics ---
4 packets transmitted, 0 packets received, 100% packet loss

# container1 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=63 time=0.237 ms

# container1 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
^C
--- 172.18.2.2 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss

# container1 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
^C
--- 172.18.2.3 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container2 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
^C
--- 172.18.0.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container2 zu container1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
^C
--- 172.18.1.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container2 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
64 bytes from 172.18.2.3: seq=0 ttl=64 time=0.092 ms

# container3 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
^C
--- 172.18.0.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container3 zu container1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
^C
--- 172.18.1.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container3 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
64 bytes from 172.18.2.2: seq=0 ttl=64 time=0.096 ms
```

Kommunikationstabelle:
||||||
| ---------- | ---------- | ---------- | ---------- | ----------- |
|            | container0 | container1 | container2 | container 3 |
| container0 | yes | yes | no | no |
| container1 | yes | yes | no | no |
| container2 | no | no | yes | yes |
| container3 | no | no | yes | yes |

## FRR OSPFv2

Die Netzwerkkonfiguration in FRR OSPF aktiviert OSPF auf allen Subnetzen, die im konfigurierten OSPF Subnetz enthalten sind.
Die OSPF Netzwerkkonfiguration `network 192.168.1.0/24 area 0.0.0.0` aktiviert OSPF z.B. auf einem Interface mit der IP 192.168.1.129/25, aber nicht auf einem Interface mit der IP 192.168.1.1/23.

Dies hat folgende drei Konfigurationsoptionen zur Folge:
1. Automatisch werden alle Subnetze der jeweiligen Node z.B. BSP oder CSP per OSPF "advertised" / erreichbar gemacht. Einmalig wird `network 0.0.0.0/0 area 0` konfiguriert.
2. Es wird explizit und 1 zu 1 konfiguriert, welche Subnetze "advertised" werden sollen. Für jedes Subnetz, das mittels OSPF übertragen werden soll, wird eine Konfigurationsregel z.B. `network 192.168.250.1/24 area 0` eingetragen. Dies hat zur Folge, dass mit jedem neuen Subnetz auf der Node analysiert werden muss, ob dieses  Gegebenenfalls muss eine entsprechende Regel konfiguriert werden.
3. Eine Mischung aus 1 und 2. Für machen Subnetze werden Einzelregeln definiert, andere werden unter einer größeren Subnetzregel zusammengefasst.

Konfigurationsaufwände:
| Variante | Vorteile | Nachteile |
| -------- | -------- | --------- |
| 1 | Sehr einfache Konfiguration. Wird ein neues Subnetz auf der Node hinzugefügt, muss die OSPF Konfiguration nicht angepasst werden. | Es können keine Subnetze von der Übertragung ausgenommen werden. |
| 2 | Es können Subnetze von der Übertragung ausgenommen werden. | Komplexere Konfiguration. Mit jedem neuen Subnetz auf der Node muss analysiert werden, ob dieses Subnetz "advertised" werden soll. Mit jedem neuen Subnetz auf der Node muss die OSPF Konfiguration angepasst werden.
| 3 | Es können Subnetze von der Übertragung ausgenommen werden. Potenziell muss nicht mit jedem neuen Subnetz die OSPF Konfiguration angepasst werden. | Sehr komplexe Konfiguration. Bei jedem neuen Subnetz muss analysiert werden, ob dieses "advertised" werden soll. Auch muss geprüft werden, ob das Subnetz bereits in einer OSPF Konfigurationsregel abgebildet ist. Ist dies der Fall, aber das Subnetz soll nicht übertragen werden, muss eine grobere OSPF Konfiguration in mehrere, spezifischere Regeln unterteilt werden. Changes bzgl. eines neuen Subnetzes sind nicht einheitlich.

## Zustand der VMs mit OSPF in Variante 1 und mit deaktivierter Firewall

### VM1
Firewall:
```bash
root@ospf1:~# iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
root@ospf1:~# nft list ruleset
root@ospf1:~#
```

OSPF Config:
```bash
frr version 8.1
frr defaults traditional
hostname ospf1
log syslog informational
no ipv6 forwarding
service integrated-vtysh-config
!
interface lo
 ip address 1.1.1.1/32
exit
!
router ospf
 network 0.0.0.0/0 area 0
exit
!
```

Routing Table:
```bash
default via 10.0.2.1 dev enp0s3 proto dhcp metric 100
2.2.2.2 nhid 30 via 10.0.2.17 dev enp0s3 proto ospf metric 20
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.16 metric 100
169.254.0.0/16 dev enp0s3 scope link metric 1000
172.18.0.0/24 dev docker0 proto kernel scope link src 172.18.0.1
172.18.1.0/24 dev docker1 proto kernel scope link src 172.18.1.1
172.18.2.0/24 nhid 30 via 10.0.2.17 dev enp0s3 proto ospf metric 20
```

### VM2
Firewall:
```bash
root@ospf2:~# iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
root@ospf2:~# nft list ruleset
root@ospf2:~#
```

OSPF Config VM2:
```bash
frr version 8.1
frr defaults traditional
hostname ospf2
log syslog informational
no ipv6 forwarding
service integrated-vtysh-config
!
interface lo
 ip address 2.2.2.2/32
exit
!
router ospf
 network 0.0.0.0/0 area 0
exit
!
```

Routing Table:
```bash
default via 10.0.2.1 dev enp0s3 proto dhcp metric 100
1.1.1.1 nhid 24 via 10.0.2.16 dev enp0s3 proto ospf metric 20
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.17 metric 100
169.254.0.0/16 dev enp0s3 scope link metric 1000
172.18.0.0/24 nhid 24 via 10.0.2.16 dev enp0s3 proto ospf metric 20
172.18.1.0/24 nhid 24 via 10.0.2.16 dev enp0s3 proto ospf metric 20
172.18.2.0/24 dev docker2 proto kernel scope link src 172.18.2.1
```

### Container Ping Test

```bash
# container0 zu contianer1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
64 bytes from 172.18.1.2: seq=0 ttl=63 time=0.087 ms

# container0 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
64 bytes from 172.18.2.2: seq=0 ttl=62 time=0.694 ms

# container0 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
64 bytes from 172.18.2.3: seq=0 ttl=62 time=0.860 ms

# container1 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=63 time=0.370 ms

# container1 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
64 bytes from 172.18.2.2: seq=0 ttl=62 time=0.467 ms

# container1 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
64 bytes from 172.18.2.3: seq=0 ttl=62 time=0.670 ms

# container2 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=62 time=0.709 ms

# container2 zu container1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
64 bytes from 172.18.1.2: seq=0 ttl=62 time=0.524 ms

# container2 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
64 bytes from 172.18.2.3: seq=0 ttl=64 time=0.126 ms

# container3 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=62 time=0.527 ms

# container3 zu container1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
64 bytes from 172.18.1.2: seq=0 ttl=62 time=0.580 ms

# container3 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
64 bytes from 172.18.2.2: seq=0 ttl=64 time=0.073 ms
```

Kommunikationstabelle:
||||||
| ---------- | ---------- | ---------- | ---------- | ----------- |
|            | container0 | container1 | container2 | container 3 |
| container0 | yes | yes | yes | yes |
| container1 | yes | yes | yes | yes |
| container2 | yes | yes | yes | yes |
| container3 | yes | yes | yes | yes |

## Zustand der VMs mit OSPF in Variante 1 und Firewall unterbindet Forwarding

### NFT Firewallkonfiguration auf beiden VMs

```bash
table ip ip-traffic-table {
        chain containers {
                type filter hook forward priority filter; policy drop;
        }
}
```

### Container Ping Test

```bash
# container0 zu contianer1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
^C
--- 172.18.1.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container0 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
^C
--- 172.18.2.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container0 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
^C
--- 172.18.2.3 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container1 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
^C
--- 172.18.0.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container1 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
^C
--- 172.18.2.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container1 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
^C
--- 172.18.2.3 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container2 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
^C
--- 172.18.0.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container2 zu container1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
^C
--- 172.18.1.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container2 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
^C
--- 172.18.2.3 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container3 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
^C
--- 172.18.0.2 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss

# container3 zu container1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
^C
--- 172.18.1.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container3 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
^C
--- 172.18.2.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss
```

Kommunikationstabelle:
||||||
| ---------- | ---------- | ---------- | ---------- | ----------- |
|            | container0 | container1 | container2 | container 3 |
| container0 | yes | no | no | no |
| container1 | no | yes | no | no |
| container2 | no | no | yes | no |
| container3 | no | no | no | yes |

## Zustand der VMs mit OSPF in Variante 1 und Firewall Forward Whitelisting L3

### NFT Firewallkonfiguration VM 1

```bash
table ip ip-traffic-table {
        chain containers {
                type filter hook forward priority filter; policy drop;
                meta nftrace set 1
                ip saddr 172.18.2.2 ip daddr 172.18.0.2 accept
                ip saddr 172.18.0.2 ip daddr 172.18.2.2 accept
                ip saddr 172.18.0.2 ip daddr 172.18.1.2 accept
                ip saddr 172.18.1.2 ip daddr 172.18.0.2 accept
        }
}
```

### NFT Firewallkonfiguration VM 2

```bash
table ip ip-traffic-table {
        chain containers {
                type filter hook forward priority filter; policy drop;
                meta nftrace set 1
                ip saddr 172.18.2.2 ip daddr 172.18.0.2 accept
                ip saddr 172.18.0.2 ip daddr 172.18.2.2 accept
                ip saddr 172.18.2.2 ip daddr 172.18.2.3 accept
                ip saddr 172.18.2.3 ip daddr 172.18.2.2 accept
        }
}
```

### Container Ping Test

```bash
# container0 zu contianer1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
64 bytes from 172.18.1.2: seq=0 ttl=63 time=0.296 ms

# container0 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
64 bytes from 172.18.2.2: seq=0 ttl=62 time=0.597 ms

# container0 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
^C
--- 172.18.2.3 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container1 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=63 time=0.059 ms

# container1 zu container2
PING 172.18.2.2 (172.18.2.2): 56 data bytes
^C
--- 172.18.2.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container1 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
^C
--- 172.18.2.3 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss

# container2 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=62 time=0.671 ms

# container2 zu container1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
^C
--- 172.18.1.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container2 zu container3
PING 172.18.2.3 (172.18.2.3): 56 data bytes
64 bytes from 172.18.2.3: seq=0 ttl=64 time=0.149 ms

# container3 zu container0
PING 172.18.0.2 (172.18.0.2): 56 data bytes
^C
--- 172.18.0.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container3 zu container1
PING 172.18.1.2 (172.18.1.2): 56 data bytes
^C
--- 172.18.1.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

# container3 zu container2
PING 172.18.2.3 (172.18.2.3): 56 data bytes
64 bytes from 172.18.2.3: seq=0 ttl=64 time=0.040 ms
```

Kommunikationstabelle:
||||||
| ---------- | ---------- | ---------- | ---------- | ----------- |
|            | container0 | container1 | container2 | container 3 |
| container0 | yes | yes | yes | no |
| container1 | yes | yes | no | no |
| container2 | yes | no | yes | yes |
| container3 | no | no | yes | yes |


"""
curl 172.18.2.3:9090/api/v1/targets | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1535  100  1535    0     0   487k      0 --:--:-- --:--:-- --:--:--  749k
{
  "status": "success",
  "data": {
    "activeTargets": [
      {
        "discoveredLabels": {
          "__address__": "http://google.de",
          "__meta_url": "http://172.18.2.1:30080/discoveredtargets",
          "__metrics_path__": "/probe",
          "__param_module": "http_2xx",
          "__scheme__": "http",
          "__scrape_interval__": "5s",
          "__scrape_timeout__": "5s",
          "job": "blackbox-targets"
        },
        "labels": {
          "instance": "http://google.de",
          "job": "blackbox-targets"
        },
        "scrapePool": "blackbox-targets",
        "scrapeUrl": "http://172.18.2.2:9115/probe?module=http_2xx&target=http%3A%2F%2Fgoogle.de",
        "globalUrl": "http://172.18.2.2:9115/probe?module=http_2xx&target=http%3A%2F%2Fgoogle.de",
        "lastError": "",
        "lastScrape": "2024-08-15T08:22:07.043431108Z",
        "lastScrapeDuration": 0.16978539,
        "health": "up",
        "scrapeInterval": "5s",
        "scrapeTimeout": "5s"
      },
      {
        "discoveredLabels": {
          "__address__": "http://prometheus.io",
          "__meta_url": "http://172.18.2.1:30080/discoveredtargets",
          "__metrics_path__": "/probe",
          "__param_module": "http_2xx",
          "__scheme__": "http",
          "__scrape_interval__": "5s",
          "__scrape_timeout__": "5s",
          "job": "blackbox-targets"
        },
        "labels": {
          "instance": "http://prometheus.io",
          "job": "blackbox-targets"
        },
        "scrapePool": "blackbox-targets",
        "scrapeUrl": "http://172.18.2.2:9115/probe?module=http_2xx&target=http%3A%2F%2Fprometheus.io",
        "globalUrl": "http://172.18.2.2:9115/probe?module=http_2xx&target=http%3A%2F%2Fprometheus.io",
        "lastError": "",
        "lastScrape": "2024-08-15T08:22:08.942824101Z",
        "lastScrapeDuration": 0.138668454,
        "health": "up",
        "scrapeInterval": "5s",
        "scrapeTimeout": "5s"
      }
    ],
    "droppedTargets": [],
    "droppedTargetCounts": {
      "blackbox-targets": 0
    }
  }
}
"""

"""
curl 172.18.2.4:9090/api/v1/targets | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1155  100  1155    0     0   687k      0 --:--:-- --:--:-- --:--:-- 1127k
{
  "status": "success",
  "data": {
    "activeTargets": [
      {
        "discoveredLabels": {
          "__address__": "success",
          "__metrics_path__": "/probe",
          "__scheme__": "http",
          "__scrape_interval__": "5s",
          "__scrape_timeout__": "5s",
          "job": "script-exporter"
        },
        "labels": {
          "instance": "success",
          "job": "script-exporter"
        },
        "scrapePool": "script-exporter",
        "scrapeUrl": "http://172.18.2.2:9172/probe?name=success",
        "globalUrl": "http://172.18.2.2:9172/probe?name=success",
        "lastError": "",
        "lastScrape": "2024-08-26T07:39:59.801573127Z",
        "lastScrapeDuration": 1.013206768,
        "health": "up",
        "scrapeInterval": "5s",
        "scrapeTimeout": "5s"
      },
      {
        "discoveredLabels": {
          "__address__": "failure",
          "__metrics_path__": "/probe",
          "__scheme__": "http",
          "__scrape_interval__": "5s",
          "__scrape_timeout__": "5s",
          "job": "script-exporter"
        },
        "labels": {
          "instance": "failure",
          "job": "script-exporter"
        },
        "scrapePool": "script-exporter",
        "scrapeUrl": "http://172.18.2.2:9172/probe?name=failure",
        "globalUrl": "http://172.18.2.2:9172/probe?name=failure",
        "lastError": "",
        "lastScrape": "2024-08-26T07:39:59.614235947Z",
        "lastScrapeDuration": 1.004663062,
        "health": "up",
        "scrapeInterval": "5s",
        "scrapeTimeout": "5s"
      }
    ],
    "droppedTargets": [],
    "droppedTargetCounts": {
      "script-exporter": 0
    }
  }
}
"""

"""
 curl 172.18.2.4:9090/api/v1/targets | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2600    0  2600    0     0   134k      0 --:--:-- --:--:-- --:--:--  141k
{
  "status": "success",
  "data": {
    "activeTargets": [
      {
        "discoveredLabels": {
          "__address__": "http://google.de",
          "__meta_url": "http://172.18.2.1:30080/discoveredtargets",
          "__metrics_path__": "/probe",
          "__param_module": "http_2xx",
          "__scheme__": "http",
          "__scrape_interval__": "5s",
          "__scrape_timeout__": "5s",
          "job": "blackbox-targets"
        },
        "labels": {
          "instance": "http://google.de",
          "job": "blackbox-targets"
        },
        "scrapePool": "blackbox-targets",
        "scrapeUrl": "http://172.18.2.3:9115/probe?module=http_2xx&target=http%3A%2F%2Fgoogle.de",
        "globalUrl": "http://172.18.2.3:9115/probe?module=http_2xx&target=http%3A%2F%2Fgoogle.de",
        "lastError": "",
        "lastScrape": "2024-08-26T07:44:32.420434152Z",
        "lastScrapeDuration": 0.159538804,
        "health": "up",
        "scrapeInterval": "5s",
        "scrapeTimeout": "5s"
      },
      {
        "discoveredLabels": {
          "__address__": "http://prometheus.io",
          "__meta_url": "http://172.18.2.1:30080/discoveredtargets",
          "__metrics_path__": "/probe",
          "__param_module": "http_2xx",
          "__scheme__": "http",
          "__scrape_interval__": "5s",
          "__scrape_timeout__": "5s",
          "job": "blackbox-targets"
        },
        "labels": {
          "instance": "http://prometheus.io",
          "job": "blackbox-targets"
        },
        "scrapePool": "blackbox-targets",
        "scrapeUrl": "http://172.18.2.3:9115/probe?module=http_2xx&target=http%3A%2F%2Fprometheus.io",
        "globalUrl": "http://172.18.2.3:9115/probe?module=http_2xx&target=http%3A%2F%2Fprometheus.io",
        "lastError": "",
        "lastScrape": "2024-08-26T07:44:32.360463671Z",
        "lastScrapeDuration": 0.155145155,
        "health": "up",
        "scrapeInterval": "5s",
        "scrapeTimeout": "5s"
      },
      {
        "discoveredLabels": {
          "__address__": "success",
          "__metrics_path__": "/probe",
          "__scheme__": "http",
          "__scrape_interval__": "5s",
          "__scrape_timeout__": "5s",
          "job": "script-exporter"
        },
        "labels": {
          "instance": "success",
          "job": "script-exporter"
        },
        "scrapePool": "script-exporter",
        "scrapeUrl": "http://172.18.2.2:9172/probe?name=success",
        "globalUrl": "http://172.18.2.2:9172/probe?name=success",
        "lastError": "",
        "lastScrape": "2024-08-26T07:44:29.823393784Z",
        "lastScrapeDuration": 1.010565381,
        "health": "up",
        "scrapeInterval": "5s",
        "scrapeTimeout": "5s"
      },
      {
        "discoveredLabels": {
          "__address__": "failure",
          "__metrics_path__": "/probe",
          "__scheme__": "http",
          "__scrape_interval__": "5s",
          "__scrape_timeout__": "5s",
          "job": "script-exporter"
        },
        "labels": {
          "instance": "failure",
          "job": "script-exporter"
        },
        "scrapePool": "script-exporter",
        "scrapeUrl": "http://172.18.2.2:9172/probe?name=failure",
        "globalUrl": "http://172.18.2.2:9172/probe?name=failure",
        "lastError": "",
        "lastScrape": "2024-08-26T07:44:34.625235804Z",
        "lastScrapeDuration": 1.007479414,
        "health": "up",
        "scrapeInterval": "5s",
        "scrapeTimeout": "5s"
      }
    ],
    "droppedTargets": [],
    "droppedTargetCounts": {
      "blackbox-targets": 0,
      "script-exporter": 0
    }
  }
}
"""