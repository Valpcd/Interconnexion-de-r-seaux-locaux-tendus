# TP4 Interrco - Pacaud Valérian
## Lancement des VM Spoke71 et Spoke72 du TP3
```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP3$ $HOME/masters/scripts/lab-startup.py lab.yaml
spoke71.qcow2 already exists!
Starting spoke71...
~> Virtual machine filename   : spoke71.qcow2
~> RAM size                   : 1024MB
~> SPICE VDI port number      : 6218
~> telnet console port number : 2618
~> MAC address                : b8:ad:ca:fe:01:3e
~> Switch port interface      : tap318, trunk mode
~> IPv6 LL address            : fe80::baad:caff:fefe:13e%dsw-host
spoke71 started!
spoke72.qcow2 already exists!
Starting spoke72...
~> Virtual machine filename   : spoke72.qcow2
~> RAM size                   : 1024MB
~> SPICE VDI port number      : 6219
~> telnet console port number : 2619
~> MAC address                : b8:ad:ca:fe:01:3f
~> Switch port interface      : tap319, trunk mode
~> IPv6 LL address            : fe80::baad:caff:fefe:13f%dsw-host
spoke72 started!
```

Q12: Initiation de nouvelles transactions depuis les conteneurs hébergés sur un routeur Spoke

### **Spoke 71**

```bash
etu@spoke71:~$ incus ls
+------+---------+---------------------+------------------------------------------+-----------+-----------+
| NAME |  STATE  |        IPV4         |                   IPV6                   |   TYPE    | SNAPSHOTS |
+------+---------+---------------------+------------------------------------------+-----------+-----------+
| c0   | RUNNING | 100.64.71.10 (eth0) | fda0:7a62:47::a (eth0)                   | CONTAINER | 0         |
|      |         |                     | fda0:7a62:47:0:216:3eff:fe34:78db (eth0) |           |           |
+------+---------+---------------------+------------------------------------------+-----------+-----------+
| c1   | RUNNING | 100.64.71.11 (eth0) | fda0:7a62:47::b (eth0)                   | CONTAINER | 0         |
|      |         |                     | fda0:7a62:47:0:216:3eff:fe1c:6ba3 (eth0) |           |           |
+------+---------+---------------------+------------------------------------------+-----------+-----------+
| c2   | RUNNING | 100.64.71.12 (eth0) | fda0:7a62:47::c (eth0)                   | CONTAINER | 0         |
|      |         |                     | fda0:7a62:47:0:216:3eff:fe04:9b75 (eth0) |           |           |
+------+---------+---------------------+------------------------------------------+-----------+-----------+
etu@spoke71:~$ for i in {0..2}
do
   echo ">>>>>>>>>>>>>>>>> c$i"
   incus exec c$i -- apt -y install wget
done
>>>>>>>>>>>>>>>>> c0
wget is already the newest version (1.24.5-2+b1).
Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c1
wget is already the newest version (1.24.5-2+b1).
Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c2
wget is already the newest version (1.24.5-2+b1).
Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
```
**IPV4**
```bash
etu@spoke71:~$ for i in {0..2}
do
   echo ">>>>>>>>>>>>>>>>> c$i"
   for j in {0..4}
   do
      incus exec c$i -- wget -4 -O /dev/null https://www.iana.org
      sleep 1
   done
done
>>>>>>>>>>>>>>>>> c0
--2024-11-05 16:12:34--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.33.8
Connecting to www.iana.org (www.iana.org)|192.0.33.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:12:35 (465 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:12:36--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.33.8
Connecting to www.iana.org (www.iana.org)|192.0.33.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:12:37 (287 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:12:38--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.33.8
Connecting to www.iana.org (www.iana.org)|192.0.33.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:12:38 (265 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:12:39--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.33.8
Connecting to www.iana.org (www.iana.org)|192.0.33.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:12:40 (130 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:12:41--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.33.8
Connecting to www.iana.org (www.iana.org)|192.0.33.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:12:42 (306 MB/s) - ‘/dev/null’ saved [6173/6173]

>>>>>>>>>>>>>>>>> c1
--2024-11-05 16:12:43--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.33.8
Connecting to www.iana.org (www.iana.org)|192.0.33.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:12:44 (289 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:12:45--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.33.8
Connecting to www.iana.org (www.iana.org)|192.0.33.8|:443... connected.
HTTP request sent, awaiting response... ^C--2024-11-05 16:12:46--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.33.8
Connecting to www.iana.org (www.iana.org)|192.0.33.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:12:47 (267 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:12:48--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.33.8
Connecting to www.iana.org (www.iana.org)|192.0.33.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:12:49 (287 MB/s) - ‘/dev/null’ saved [6173/6173]
```
**IPV6**
```bash
etu@spoke71:~$ 
etu@spoke71:~$ for i in {0..2}
do
   echo ">>>>>>>>>>>>>>>>> c$i"
   for j in {0..4}
   do
      incus exec c$i -- wget -6 -O /dev/null https://www.iana.org
      sleep 1
   done
done
>>>>>>>>>>>>>>>>> c0
--2024-11-05 16:14:58--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 2620:0:2d0:200::b:8
Connecting to www.iana.org (www.iana.org)|2620:0:2d0:200::b:8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:15:00 (17.1 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:15:01--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 2620:0:2d0:200::b:8
Connecting to www.iana.org (www.iana.org)|2620:0:2d0:200::b:8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:15:02 (68.8 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:15:03--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 2620:0:2d0:200::b:8
Connecting to www.iana.org (www.iana.org)|2620:0:2d0:200::b:8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:15:04 (80.1 MB/s) - ‘/dev/null’ saved [6173/6173]
```

### **Spoke 72**
```bash
incus ls
+------+---------+---------------------+------------------------------------------+-----------+-----------+
| NAME |  STATE  |        IPV4         |                   IPV6                   |   TYPE    | SNAPSHOTS |
+------+---------+---------------------+------------------------------------------+-----------+-----------+
| c0   | RUNNING | 100.64.72.10 (eth0) | fda0:7a62:48::a (eth0)                   | CONTAINER | 0         |
|      |         |                     | fda0:7a62:48:0:216:3eff:fe81:89a (eth0)  |           |           |
+------+---------+---------------------+------------------------------------------+-----------+-----------+
| c1   | RUNNING | 100.64.72.11 (eth0) | fda0:7a62:48::b (eth0)                   | CONTAINER | 0         |
|      |         |                     | fda0:7a62:48:0:216:3eff:feca:94d8 (eth0) |           |           |
+------+---------+---------------------+------------------------------------------+-----------+-----------+
| c2   | RUNNING | 100.64.72.12 (eth0) | fda0:7a62:48::c (eth0)                   | CONTAINER | 0         |
|      |         |                     | fda0:7a62:48:0:216:3eff:fe03:40d2 (eth0) |           |           |
+------+---------+---------------------+------------------------------------------+-----------+-----------+
etu@spoke72:~$ for i in {0..2}
do
   echo ">>>>>>>>>>>>>>>>> c$i"
   incus exec c$i -- apt -y install wget
done
>>>>>>>>>>>>>>>>> c0
wget is already the newest version (1.24.5-2+b1).
Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c1
wget is already the newest version (1.24.5-2+b1).
Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c2
wget is already the newest version (1.24.5-2+b1).
Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
```
**IPV4**
```bash
etu@spoke72:~$ for i in {0..2}
do
   echo ">>>>>>>>>>>>>>>>> c$i"
   for j in {0..4}
   do
      incus exec c$i -- wget -4 -O /dev/null https://www.iana.org
      sleep 1
   done
done
>>>>>>>>>>>>>>>>> c0
--2024-11-05 16:13:08--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.46.8
Connecting to www.iana.org (www.iana.org)|192.0.46.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:13:09 (121 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:13:10--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.46.8
Connecting to www.iana.org (www.iana.org)|192.0.46.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:13:10 (278 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:13:11--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.46.8
Connecting to www.iana.org (www.iana.org)|192.0.46.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:13:12 (122 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:13:13--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.46.8
Connecting to www.iana.org (www.iana.org)|192.0.46.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:13:13 (123 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:13:14--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.46.8
Connecting to www.iana.org (www.iana.org)|192.0.46.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:13:15 (291 MB/s) - ‘/dev/null’ saved [6173/6173]

>>>>>>>>>>>>>>>>> c1
--2024-11-05 16:13:16--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.46.8
Connecting to www.iana.org (www.iana.org)|192.0.46.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:13:16 (294 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:13:17--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 192.0.46.8
Connecting to www.iana.org (www.iana.org)|192.0.46.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:13:18 (118 MB/s) - ‘/dev/null’ saved [6173/6173]
```
**IPV6**
```bash
etu@spoke72:~$ for i in {0..2}
do
   echo ">>>>>>>>>>>>>>>>> c$i"
   for j in {0..4}
   do
      incus exec c$i -- wget -6 -O /dev/null https://www.iana.org
      sleep 1
   done
done
>>>>>>>>>>>>>>>>> c0
--2024-11-05 16:15:10--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 2620:0:2d0:200::b:8
Connecting to www.iana.org (www.iana.org)|2620:0:2d0:200::b:8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:15:11 (291 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:15:12--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 2620:0:2d0:200::b:8
Connecting to www.iana.org (www.iana.org)|2620:0:2d0:200::b:8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:15:13 (294 MB/s) - ‘/dev/null’ saved [6173/6173]

--2024-11-05 16:15:14--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 2620:0:2d0:200::b:8
Connecting to www.iana.org (www.iana.org)|2620:0:2d0:200::b:8|:443... ^C--2024-11-05 16:15:16--  https://www.iana.org/
Resolving www.iana.org (www.iana.org)... 2620:0:2d0:200::b:8
Connecting to www.iana.org (www.iana.org)|2620:0:2d0:200::b:8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6173 (6.0K) [text/html]
Saving to: ‘/dev/null’

/dev/null             0%[                    ]       0  --.-KB/s          /dev/null           100%[===================>]   6.03K  --.-KB/s    in 0s      

2024-11-05 16:15:17 (68.1 MB/s) - ‘/dev/null’ saved [6173/6173]
```

## 3. Protection de base des routeurs Hub et Spoke

### 3.1. Protection contre l'usurpation d'adresse source

#### IPV4
```bash
etu@spoke71:~$ sudo nft -f /etc/nftables.conf
etu@spoke71:~$ sudo nft list table inet raw
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 0 bytes 0 drop
        }
}
```


```bash
etu@spoke71:~$ sudo nft list table inet raw
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 10 bytes 280 drop
        }
}
```

Mêmes manipulations sur le spoke72:

```bash
etu@spoke72:~$ sudo nft -f /etc/nftables.conf
etu@spoke72:~$ sudo nft list table inet raw
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 0 bytes 0 drop
        }
}
```
```bash
etu@spoke72:~$ sudo nft list table inet raw
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 10 bytes 280 drop
        }
}
```

#### IPV6
Interface dummy
Relevé des compteurs avant ping6 du HUB:
```bash
etu@spoke71:~$ sudo nft list table inet raw
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 108 bytes 11204 drop
        }
```
Relevé des compteurs après ping6...
```bash
etu@spoke71:~$ sudo nft list table inet raw
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 118 bytes 12244 drop
        }
```

#### rp_filter
```bash
etu@spoke71:~$ sudo sysctl -w net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.all.rp_filter = 1
```
```bash
etu@HUB:~$ sudo hping3 -1 --rand-source --fast -c 100 100.64.71.10
HPING 100.64.71.10 (ppp1 100.64.71.10): icmp mode set, 28 headers + 0 data bytes
len=28 ip=100.64.71.10 ttl=63 id=65088 icmp_seq=0 rtt=7.5 ms
len=28 ip=100.64.71.10 ttl=63 id=9166 icmp_seq=1 rtt=3.4 ms
len=28 ip=100.64.71.10 ttl=63 id=49753 icmp_seq=2 rtt=7.3 ms
len=28 ip=100.64.71.10 ttl=63 id=16445 icmp_seq=3 rtt=3.1 ms
len=28 ip=100.64.71.10 ttl=63 id=57407 icmp_seq=4 rtt=6.9 ms
len=28 ip=100.64.71.10 ttl=63 id=41969 icmp_seq=5 rtt=2.5 ms
len=28 ip=100.64.71.10 ttl=63 id=16050 icmp_seq=6 rtt=6.5 ms
len=28 ip=100.64.71.10 ttl=63 id=39695 icmp_seq=7 rtt=2.2 ms
len=28 ip=100.64.71.10 ttl=63 id=20824 icmp_seq=8 rtt=6.0 ms
len=28 ip=100.64.71.10 ttl=63 id=28906 icmp_seq=9 rtt=1.8 ms
len=28 ip=100.64.71.10 ttl=63 id=35626 icmp_seq=10 rtt=5.6 ms
len=28 ip=100.64.71.10 ttl=63 id=45925 icmp_seq=11 rtt=1.3 ms
len=28 ip=100.64.71.10 ttl=63 id=51047 icmp_seq=12 rtt=5.2 ms
len=28 ip=100.64.71.10 ttl=63 id=39029 icmp_seq=13 rtt=9.1 ms
len=28 ip=100.64.71.10 ttl=63 id=47389 icmp_seq=14 rtt=4.9 ms
len=28 ip=100.64.71.10 ttl=63 id=45199 icmp_seq=15 rtt=8.6 ms
len=28 ip=100.64.71.10 ttl=63 id=2963 icmp_seq=16 rtt=4.3 ms
[....]
```


```bash
etu@spoke71:~$ journalctl -n 500 -f --grep martian
nov. 07 20:11:05 spoke71 kernel: IPv4: martian source 100.64.71.10 from 233.172.159.254, on dev ppp0
[ 4904.644938] IPv4: martian source 100.64.71.10 from 229.124.201.54, on dev ppp0
[ 4904.646964] ll header: 00000000: 45 00 00 1c 0e 8a 00 00 40 01 12 5a e5 7c c9 36
[ 4904.648772] ll header: 00000010: 64 40 47 0a 08 00 b1 8c 34 73
nov. 07 20:11:23 spoke71 kernel: IPv4: martian source 100.64.71.10 from 229.124.201.54, on dev ppp0
[ 4925.560438] IPv4: martian source 100.64.71.10 from 234.239.132.187, on dev ppp0
[ 4925.563440] ll header: 00000000: 45 00 00 1c 64 6b 00 00 40 01 fb 80 ea ef 84 bb
[ 4925.565731] ll header: 00000010: 64 40 47 0a 08 00 ac 8c 37 73
nov. 07 20:11:44 spoke71 kernel: IPv4: martian source 100.64.71.10 from 234.239.132.187, on dev ppp0
[ 4927.465057] IPv4: martian source 100.64.71.10 from 228.20.141.29, on dev ppp0
[ 4927.466906] ll header: 00000000: 45 00 00 1c c9 9e 00 00 40 01 94 c6 e4 14 8d 1d
[ 4927.468393] ll header: 00000010: 64 40 47 0a 08 00 99 8c 37 73
nov. 07 20:11:46 spoke71 kernel: IPv4: martian source 100.64.71.10 from 228.20.141.29, on dev ppp0
[ 4931.874880] IPv4: martian source 100.64.71.10 from 239.112.15.132, on dev ppp0
[ 4931.876733] ll header: 00000000: 45 00 00 1c 22 ea 00 00 40 01 ad b8 ef 70 0f 84
[ 4931.878347] ll header: 00000010: 64 40 47 0a 08 00 6d 8c 37 73
nov. 07 20:11:50 spoke71 kernel: IPv4: martian source 100.64.71.10 from 239.112.15.132, on dev ppp0
[ 4932.275626] IPv4: martian source 100.64.71.10 from 228.239.70.216, on dev ppp0
[ 4932.277394] ll header: 00000000: 45 00 00 1c 6d d5 00 00 40 01 35 fa e4 ef 46 d8
[ 4932.279204] ll header: 00000010: 64 40 47 0a 08 00 69 8c 37 73
nov. 07 20:11:50 spoke71 kernel: IPv4: martian source 100.64.71.10 from 228.239.70.216, on dev ppp0
```

Mêmes manipulations sur le spoke72

```bash
etu@spoke72:~$ sudo sysctl -w net.ipv4.conf.all.rp_filter=1
[sudo] Mot de passe de etu : 
net.ipv4.conf.all.rp_filter = 1
```
```bash
etu@spoke72:~$ journalctl -n 500 -f --grep martian
[  294.379738] IPv4: martian source 100.64.72.10 from 238.221.79.75, on dev ppp0
[  294.381461] ll header: 00000000: 45 00 00 1c 3f 8c 00 00 40 01 50 e2 ee dd 4f 4b
[  294.383063] ll header: 00000010: 64 40 48 0a 08 00 8c 8c 68 73
nov. 07 20:19:32 spoke72 kernel: IPv4: martian source 100.64.72.10 from 238.221.79.75, on dev ppp0
[  295.982966] IPv4: martian source 100.64.72.10 from 226.91.241.83, on dev ppp0
[  295.984991] ll header: 00000000: 45 00 00 1c 5f 78 00 00 40 01 9b 6f e2 5b f1 53
[  295.986575] ll header: 00000010: 64 40 48 0a 08 00 7c 8c 68 73
nov. 07 20:19:34 spoke72 kernel: IPv4: martian source 100.64.72.10 from 226.91.241.83, on dev ppp0
```


### 3.2. Protection contre les dénis de service ICMP
#### IPV4
Relevé des compteurs sur le spoke71:

```bash
etu@spoke71:~$ sudo nft list table inet raw
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 0 bytes 0 drop
        }

        chain icmpfilter {
                type filter hook prerouting priority raw; policy accept;
                icmp type echo-request limit rate 10/second burst 5 packets counter packets 70 bytes 2520 accept
                icmp type echo-request counter packets 1119648 bytes 31350144 drop
        }
}
```
#### IPV6
Relevé des compteur avant ping6:
```bash
etu@spoke71:~$ sudo nft list table inet raw
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 118 bytes 12244 drop
        }

        chain icmpfilter {
                type filter hook prerouting priority raw; policy accept;
                icmp type echo-request limit rate 10/second burst 5 packets counter packets 70 bytes 2520 accept
                icmp type echo-request counter packets 1119648 bytes 31350144 drop
        }
}
```
Relevé après ping6:
```bash
etu@spoke71:~$ sudo nft list table inet raw
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 218 bytes 22644 drop
        }

        chain icmpfilter {
                type filter hook prerouting priority raw; policy accept;
                icmp type echo-request limit rate 10/second burst 5 packets counter packets 70 bytes 2520 accept
                icmp type echo-request counter packets 1119648 bytes 31350144 drop
        }
}
```
On vérifie qu'on obtient un retour correct suite à des requêtes émises “à un rythme normal”.
```bash
etu@HUB:~$ ping -qc10 fda0:7a62:47::b
PING fda0:7a62:47::b (fda0:7a62:47::b) 56 data bytes

--- fda0:7a62:47::b ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9013ms
rtt min/avg/max/mdev = 1.068/1.386/2.309/0.342 ms
```



### 3.3. Protection contre les robots de connexion au service SSH

```bash
etu@spoke71:~$ apt show fail2ban | grep  Description

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Description: ban hosts that cause multiple authentication errors
```
```bash
etu@spoke71:~$ sudo lsof -i tcp:2222 -sTCP:listen
COMMAND PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    734 root    7u  IPv4   4901      0t0  TCP *:2222 (LISTEN)
sshd    734 root    8u  IPv6   4903      0t0  TCP *:2222 (LISTEN)
etu@spoke71:~$ ss -tapl '( sport = :2222 )' | fmt -t -w80
State  Recv-Q Send-Q Local Address:Port Peer Address:PortProcess
LISTEN 0      128          0.0.0.0:2222      0.0.0.0:*
LISTEN 0      128             [::]:2222         [::]:*
```
```bash
etu@spoke71:~$ systemctl status fail2ban.service
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/usr/lib/systemd/system/fail2ban.service; enabled; preset:>
     Active: active (running) since Thu 2024-11-07 20:37:08 CET; 7s ago
 Invocation: 1da503d36bb54dd18ad0193669e17da5
       Docs: man:fail2ban(1)
   Main PID: 2268 (fail2ban-server)
      Tasks: 5 (limit: 1032)
     Memory: 24.1M (peak: 24.8M)
        CPU: 265ms
     CGroup: /system.slice/fail2ban.service
             └─2268 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

nov. 07 20:37:08 spoke71 systemd[1]: Started fail2ban.service - Fail2Ban Servic>
nov. 07 20:37:09 spoke71 fail2ban-server[2268]: Server ready
lines 1-14/14 (END)
```
```bash
etu@spoke71:~$ sudo fail2ban-client status
Status
|- Number of jail:      1
`- Jail list:   sshd
```
```bash
etu@spoke71:~$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|  `- Journal matches:  _SYSTEMD_UNIT=ssh.service + _COMM=sshd
`- Actions
   |- Currently banned: 0
   |- Total banned:     0
   `- Banned IP list:
```

coté spoke71, après la tentative de connexion raté
```bash
tu@spoke71:~$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     3
|  `- Journal matches:  _SYSTEMD_UNIT=ssh.service + _COMM=sshd
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   10.71.7.1
```
```bash
etu@spoke71:~$ sudo nft list ruleset
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 0 bytes 0 drop
        }

        chain icmpfilter {
                type filter hook prerouting priority raw; policy accept;
                icmp type echo-request limit rate 10/second burst 5 packets counter packets 70 bytes 2520 accept
                icmp type echo-request counter packets 1119648 bytes 31350144 drop
        }
}
table inet f2b-table {
        set addr-set-sshd {
                type ipv4_addr
                elements = { 10.71.7.1 }
        }

        chain f2b-chain {
                type filter hook input priority filter - 1; policy accept;
                tcp dport 2222 ip saddr @addr-set-sshd reject with icmp port-unreachable
        }
}
```
#### IPV6
```bash
etu@spoke71:~$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     6
|  `- Journal matches:  _SYSTEMD_UNIT=ssh.service + _COMM=sshd
`- Actions
   |- Currently banned: 1
   |- Total banned:     2
   `- Banned IP list:   10.71.7.1 fe80::4483:a56e:a4d2:c4f6
```

## 4. Filtrage des flux réseaux traversant les routeurs Spoke
```bash
etu@spoke71:~$ sudo systemctl restart nftables.service
etu@spoke71:~$ sudo nft list ruleset
table inet raw {
        chain rpfilter {
                type filter hook prerouting priority raw; policy accept;
                iifname "ppp0" fib saddr . iif oif 0 counter packets 0 bytes 0 drop
        }

        chain icmpfilter {
                type filter hook prerouting priority raw; policy accept;
                icmp type echo-request limit rate 10/second burst 5 packets counter packets 0 bytes 0 accept
                icmp type echo-request counter packets 0 bytes 0 drop
        }
}
table inet filter {
        chain forward {
                type filter hook forward priority filter; policy drop;
                oifname "ppp0" ct state new counter packets 0 bytes 0 accept
                ct state established,related accept
                iifname "ppp0" meta l4proto { icmp, ipv6-icmp } ct state new counter packets 0 bytes 0 accept
                iifname "ppp0" tcp dport 2222 ct state new counter packets 0 bytes 0 accept
                iifname "ppp0" meta l4proto { tcp, udp } th dport { 80, 443 } ct state new counter packets 0 bytes 0 accept
                counter packets 0 bytes 0 comment "count dropped packets"
        }
}
```
Tester les règles relatives aux flux sortants 
```bash
etu@spoke71:~$ for i in {0..2}
do
    echo ">>>>>>>>>>>>>>>>> c$i"
    incus exec c$i -- apt update
done
>>>>>>>>>>>>>>>>> c0
Get:1 http://deb.debian.org/debian trixie InRelease [172 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [49.6 kB]
Get:3 http://deb.debian.org/debian-security trixie-security InRelease [43.5 kB]
Get:4 http://deb.debian.org/debian trixie/main amd64 Packages.diff/Index [27.9 kB]
Ign:4 http://deb.debian.org/debian trixie/main amd64 Packages.diff/Index
Get:5 http://deb.debian.org/debian trixie/main Translation-en.diff/Index [27.9 kB]
Ign:5 http://deb.debian.org/debian trixie/main Translation-en.diff/Index
Get:6 http://deb.debian.org/debian trixie/main amd64 Packages [9343 kB]
Get:7 http://deb.debian.org/debian trixie/main Translation-en [6242 kB]        
Fetched 15.9 MB in 8s (2099 kB/s)                                              
125 packages can be upgraded. Run 'apt list --upgradable' to see them.
>>>>>>>>>>>>>>>>> c1
Get:1 http://deb.debian.org/debian trixie InRelease [172 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [49.6 kB]
Get:3 http://deb.debian.org/debian-security trixie-security InRelease [43.5 kB]
Get:4 http://deb.debian.org/debian trixie/main amd64 Packages.diff/Index [27.9 kB]
Ign:4 http://deb.debian.org/debian trixie/main amd64 Packages.diff/Index
Get:5 http://deb.debian.org/debian trixie/main Translation-en.diff/Index [27.9 kB]
Ign:5 http://deb.debian.org/debian trixie/main Translation-en.diff/Index
Get:6 http://deb.debian.org/debian trixie/main amd64 Packages [9343 kB]
Get:7 http://deb.debian.org/debian trixie/main Translation-en [6242 kB]        
Fetched 15.9 MB in 8s (2091 kB/s)                                              
125 packages can be upgraded. Run 'apt list --upgradable' to see them.
>>>>>>>>>>>>>>>>> c2
Get:1 http://deb.debian.org/debian trixie InRelease [172 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [49.6 kB]
Get:3 http://deb.debian.org/debian-security trixie-security InRelease [43.5 kB]
Get:4 http://deb.debian.org/debian trixie/main amd64 Packages.diff/Index [27.9 kB]
Ign:4 http://deb.debian.org/debian trixie/main amd64 Packages.diff/Index
Get:5 http://deb.debian.org/debian trixie/main Translation-en.diff/Index [27.9 kB]
Ign:5 http://deb.debian.org/debian trixie/main Translation-en.diff/Index
Get:6 http://deb.debian.org/debian trixie/main amd64 Packages [9343 kB]
Get:7 http://deb.debian.org/debian trixie/main Translation-en [6242 kB]        
Fetched 15.9 MB in 8s (1920 kB/s)                                              
125 packages can be upgraded. Run 'apt list --upgradable' to see them.
etu@spoke71:~$ sudo nft list table inet filter
table inet filter {
        chain forward {
                type filter hook forward priority filter; policy drop;
                oifname "ppp0" ct state new counter packets 18 bytes 1578 accept
                ct state established,related accept
                iifname "ppp0" meta l4proto { icmp, ipv6-icmp } ct state new counter packets 0 bytes 0 accept
                iifname "ppp0" tcp dport 2222 ct state new counter packets 0 bytes 0 accept
                iifname "ppp0" meta l4proto { tcp, udp } th dport { 80, 443 } ct state new counter packets 0 bytes 0 accept
                counter packets 0 bytes 0 comment "count dropped packets"
        }
}
```

Tester les règles relatives aux flux entrants 
```bash
etu@spoke71:~$ sudo nft list table inet filter
table inet filter {
        chain forward {
                type filter hook forward priority filter; policy drop;
                oifname "ppp0" ct state new counter packets 18 bytes 1578 accept
                ct state established,related accept
                iifname "ppp0" meta l4proto { icmp, ipv6-icmp } ct state new counter packets 0 bytes 0 accept
                iifname "ppp0" tcp dport 2222 ct state new counter packets 0 bytes 0 accept
                iifname "ppp0" meta l4proto { tcp, udp } th dport { 80, 443 } ct state new counter packets 6 bytes 420 accept
                counter packets 0 bytes 0 comment "count dropped packets"
        }
}
```