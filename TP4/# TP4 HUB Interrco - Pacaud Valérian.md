### 3.1. Protection contre l'usurpation d'adresse source
Envoi d'un ping falsifié du routeur HUB:
IPV4:
Spoke71:
```bash
etu@HUB:~$ sudo hping3 -1 -a 100.64.71.12 --fast -c 10 100.64.71.10
HPING 100.64.71.10 (ppp1 100.64.71.10): icmp mode set, 28 headers + 0 data bytes

--- 100.64.71.10 hping statistic ---
10 packets transmitted, 0 packets received, 100% packet loss
round-trip min/avg/max = 0.0/0.0/0.0 ms
```
Spoke72:
```bash
etu@HUB:~$ sudo hping3 -1 -a 100.64.72.12 --fast -c 10 100.64.72.10
HPING 100.64.72.10 (ppp0 100.64.72.10): icmp mode set, 28 headers + 0 data bytes

--- 100.64.72.10 hping statistic ---
10 packets transmitted, 0 packets received, 100% packet loss
round-trip min/avg/max = 0.0/0.0/0.0 ms
```


IPV6:
```bash
etu@HUB:~$ sudo modprobe -v dummy numdummies=1
[sudo] Mot de passe de etu : 
insmod /lib/modules/6.11.5-amd64/kernel/drivers/net/dummy.ko.xz numdummies=0 numdummies=1
etu@HUB:~$ sudo ip link set dev dummy0 up
etu@HUB:~$ sudo ip -6 addr add fda0:7a62:47::e/64 dev dummy0
etu@HUB:~$ sudo ip -6 route del fda0:7a62:47::/64 dev dummy0
etu@HUB:~$ sudo ip -6 route add fda0:7a62:47::/64 dev dummy0 metric 2048
etu@HUB:~$ ip -6 route get fda0:7a62:47::a
fda0:7a62:47::a from :: dev ppp1 src fda0:7a62:47::e metric 1024 pref medium
etu@HUB:~$ sudo ping6 -c 10 fda0:7a62:47::a
PING fda0:7a62:47::a (fda0:7a62:47::a) 56 data bytes

--- fda0:7a62:47::a ping statistics ---
10 packets transmitted, 0 received, 100% packet loss, time 9198ms
```

rp_filter:
```bash
etu@HUB:~$ sudo hping3 -1 --rand-source --fast -c 100 100.64.72.10
HPING 100.64.72.10 (ppp0 100.64.72.10): icmp mode set, 28 headers + 0 data bytes
len=28 ip=100.64.72.10 ttl=63 id=17090 icmp_seq=0 rtt=7.7 ms
len=28 ip=100.64.72.10 ttl=63 id=20850 icmp_seq=1 rtt=3.5 ms
len=28 ip=100.64.72.10 ttl=63 id=9599 icmp_seq=2 rtt=7.3 ms
len=28 ip=100.64.72.10 ttl=63 id=45319 icmp_seq=4 rtt=7.0 ms
len=28 ip=100.64.72.10 ttl=63 id=19624 icmp_seq=5 rtt=2.8 ms
len=28 ip=100.64.72.10 ttl=63 id=13700 icmp_seq=6 rtt=6.7 ms
len=28 ip=100.64.72.10 ttl=63 id=19023 icmp_seq=7 rtt=2.5 ms
len=28 ip=100.64.72.10 ttl=63 id=37393 icmp_seq=8 rtt=6.3 ms
len=28 ip=100.64.72.10 ttl=63 id=26959 icmp_seq=9 rtt=2.1 ms
len=28 ip=100.64.72.10 ttl=63 id=61342 icmp_seq=10 rtt=5.9 ms
len=28 ip=100.64.72.10 ttl=63 id=17361 icmp_seq=11 rtt=1.7 ms
len=28 ip=100.64.72.10 ttl=63 id=285 icmp_seq=12 rtt=5.4 ms
len=28 ip=100.64.72.10 ttl=63 id=27690 icmp_seq=13 rtt=1.2 ms
len=28 ip=100.64.72.10 ttl=63 id=50626 icmp_seq=14 rtt=5.0 ms
len=28 ip=100.64.72.10 ttl=63 id=53812 icmp_seq=15 rtt=8.8 ms
len=28 ip=100.64.72.10 ttl=63 id=58106 icmp_seq=16 rtt=4.6 ms
len=28 ip=100.64.72.10 ttl=63 id=23961 icmp_seq=17 rtt=8.5 ms
[...]
```

### 3.2. Protection contre les dénis de service ICMP
IPV4
```bash
etu@HUB:~$ sudo hping3 -1 --flood -c 100 100.64.71.10
HPING 100.64.71.10 (ppp1 100.64.71.10): icmp mode set, 28 headers + 0 data bytes
hping in flood mode, no replies will be shown
^C
--- 100.64.71.10 hping statistic ---
1119708 packets transmitted, 0 packets received, 100% packet loss
round-trip min/avg/max = 0.0/0.0/0.0 ms

etu@HUB:~$ ping -q -c 10 100.64.71.10
PING 100.64.71.10 (100.64.71.10) 56(84) bytes of data.

--- 100.64.71.10 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9018ms
rtt min/avg/max/mdev = 0.817/1.227/2.062/0.323 ms
```

IPV6
```bash
etu@HUB:~$ sudo ping -6 -c100 -i 0.0005 -f fda0:7a62:47::b
PING fda0:7a62:47::b (fda0:7a62:47::b) 56 data bytes
....................................................................................................
--- fda0:7a62:47::b ping statistics ---
100 packets transmitted, 0 received, 100% packet loss, time 1584ms
```

### 3.3. Protection contre les robots de connexion au service SSH
Tentative de connexion depuis le HUB

```bash
etu@HUB:~$ ssh -p 2222 etu@10.71.7.2
The authenticity of host '[10.71.7.2]:2222 ([10.71.7.2]:2222)' can't be established.
ED25519 key fingerprint is SHA256:jxdxFO/jYTC/7x8vMF3f9cmC5abEarQMDB2dc2MHJ/Q.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.71.7.2]:2222' (ED25519) to the list of known hosts.
etu@10.71.7.2's password: 
Permission denied, please try again.
etu@10.71.7.2's password: 
Permission denied, please try again.
etu@10.71.7.2's password: 
etu@10.71.7.2: Permission denied (publickey,password).
etu@HUB:~$ ssh -p 2222 etu@10.71.7.2
ssh: connect to host 10.71.7.2 port 2222: Connection refused
```

Connexion en IPV6:
```bash
etu@HUB:~$ ssh etu@fe80::2960:b6a9:cd8b:1b8d%ppp1
The authenticity of host 'fe80::2960:b6a9:cd8b:1b8d%ppp1 (fe80::2960:b6a9:cd8b:1b8d%ppp1)' can't be established.
ED25519 key fingerprint is SHA256:jxdxFO/jYTC/7x8vMF3f9cmC5abEarQMDB2dc2MHJ/Q.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'fe80::2960:b6a9:cd8b:1b8d%ppp1' (ED25519) to the list of known hosts.
etu@fe80::2960:b6a9:cd8b:1b8d%ppp1's password: 
Permission denied, please try again.
etu@fe80::2960:b6a9:cd8b:1b8d%ppp1's password: 
Permission denied, please try again.
etu@fe80::2960:b6a9:cd8b:1b8d%ppp1's password: 
etu@fe80::2960:b6a9:cd8b:1b8d%ppp1: Permission denied (publickey,password).
```
## 4. Filtrage des flux réseaux traversant les routeurs Spoke

Accès WEB depuis le routeur:
IPV4
```bash
etu@HUB:~$ for addr in {10..12}
do
    wget -O /dev/null http://100.64.71.$addr 2>&1 | grep "HTTP "
done
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
```

IPV6
```bash
etu@HUB:~$ for addr in {10..12}
do
    wget -O /dev/null http://[fda0:7a62:47::$(printf "%x" $addr)] 2>&1 | grep "HTTP "
done
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
```

## 5. Traduction d'adresses destination sur le routeur Hub

