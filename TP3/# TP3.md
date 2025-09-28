# TP3 Pacaud Valérian
### Config du switch.yaml
```yaml
ovs:
  switches:
    - name: dsw-host
      ports:
        - name: tap318 # Spoke port
          type: OVSPort
          vlan_mode: trunk
          trunks: [60, 524, 525] # Avec VLAN d'accès temporaire 60
#         trunks: [524, 525] # Sans VLAN d'accès temporaire
        - name: tap319 # Spoke port
          type: OVSPort
          vlan_mode: trunk
          trunks: [60, 526, 527] # Avec VLAN d'accès temporaire 60
#         trunks: [526, 527] # Sans VLAN d'accès temporaire
```

```yaml
kvm:
  vms:
    - vm_name: spoke71
      master_image: debian-testing-amd64.qcow2 # master image to be used
      force_copy: false # do not force copy the master image to the VM image
      memory: 1024
      tapnum: 318
    - vm_name: spoke72
      master_image: debian-testing-amd64.qcow2 # master image to be used
      force_copy: false # do not force copy the master image to the VM image
      memory: 1024
      tapnum: 319
```

```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP3$ $HOME/masters/scripts/switch-conf.py switch.yaml
----------------------------------------
Configuring switch dsw-host
>> Port tap318 vlan_mode is already set to trunk
>> Port tap318 trunks set to [60, 524, 525]
>> Port tap319 vlan_mode is already set to trunk
>> Port tap319 trunks set to [60, 526, 527]
```
```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP3$ $HOME/masters/scripts/lab-startup.py lab.yaml
Copying /home/valerianpacaud/masters/debian-testing-amd64.qcow2 to spoke71.qcow2...done
Creating OVMF_CODE.fd symlink...
Creating spoke71_OVMF_VARS.fd file...
Starting spoke71...
~> Virtual machine filename   : spoke71.qcow2
~> RAM size                   : 1024MB
~> SPICE VDI port number      : 6218
~> telnet console port number : 2618
~> MAC address                : b8:ad:ca:fe:01:3e
~> Switch port interface      : tap318, trunk mode
~> IPv6 LL address            : fe80::baad:caff:fefe:13e%dsw-host
spoke71 started!
Copying /home/valerianpacaud/masters/debian-testing-amd64.qcow2 to spoke72.qcow2...done
Creating spoke72_OVMF_VARS.fd file...
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

## Configuration réseau du spoke71

```yaml
network:
  version: 2
  ethernets:
    enp0s1:
      dhcp4: false
      dhcp6: false
      accept-ra: false
      nameservers:
        addresses:
          - 172.16.0.2
          - 2001:678:3fc:3::2

  vlans:
    enp0s1.524: # VLAN violet
      id: 524
      link: enp0s1
      addresses:
        - fe80:20C::2/64 ####
    enp0s1.525: # VLAN orange
      id: 525
      link: enp0s1
      addresses: []
    enp0s1.60: # VLAN accès temporaire
      id: 60
      link: enp0s1
      dhcp4: true
      dhcp6: false
      accept-ra: true
```
```bash
etu@spoke71:~$ sudo netplan apply

** (generate:714): WARNING **: 19:24:55.229: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:713): WARNING **: 19:24:55.522: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:713): WARNING **: 19:24:55.583: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.
[ 1418.277777] 8021q: 802.1Q VLAN Support v1.8
[ 1418.278069] 8021q: adding VLAN 0 to HW filter on device enp0s1
```
## Configuration réseau du spoke72

```yaml
network:
  version: 2
  ethernets:
    enp0s1:
      dhcp4: false
      dhcp6: false
      accept-ra: false
      nameservers:
        addresses:
          - 172.16.0.2
          - 2001:678:3fc:3::2

  vlans:
    enp0s1.526: # VLAN violet
      id: 526
      link: enp0s1
      addresses:
        - fe80:20E::2/64
    enp0s1.527: # VLAN orange
      id: 527
      link: enp0s1
      addresses: []
    enp0s1.60: # VLAN accès temporaire
      id: 60
      link: enp0s1
      dhcp4: true
      dhcp6: false
      accept-ra: true
```

```bash
etu@spoke72:~$ sudo netplan apply

** (generate:554): WARNING **: 19:30:12.151: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:553): WARNING **: 19:30:12.475: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:553): WARNING **: 19:30:12.525: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.
[ 1703.081698] 8021q: 802.1Q VLAN Support v1.8
[ 1703.082287] 8021q: adding VLAN 0 to HW filter on device enp0s1
```
## Activation de la fonction routage sur les deux spoke

```bash
etu@spoke71:~$ sudo sysctl --system
* Applique /usr/lib/sysctl.d/10-coredump-debian.conf …
* Applique /etc/sysctl.d/10-routing.conf …
* Applique /usr/lib/sysctl.d/50-default.conf …
* Applique /usr/lib/sysctl.d/50-pid-max.conf …
* Applique /etc/sysctl.conf …
```

```bash
etu@spoke72:~$ sudo sysctl --system
* Applique /usr/lib/sysctl.d/10-coredump-debian.conf …
* Applique /etc/sysctl.d/10-routing.conf …
* Applique /usr/lib/sysctl.d/50-default.conf …
* Applique /usr/lib/sysctl.d/50-pid-max.conf …
* Applique /etc/sysctl.conf …
```

## Activation du protocole PPP sur les deux spoke
### Authentifiants de la session PPP
**/etc/ppp/chap-secrets**
```bash
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
"spoke71"       *       "Or4ng3.71"             * 
```
```bash
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
"spoke72"       *       "Or4ng3.72"             *
```
### Config du démon ppp
**/etc/ppp/chap-secrets**
```bash
# Le nom d'utilisateur désigne l'entrée du fichier /etc/ppp/chap-secrets
user spoke71

# Chargement du module PPPoE avec les détails dans la journalisation
plugin rp-pppoe.so rp_pppoe_ac BRAS rp_pppoe_verbose 1

# Interface (VLAN) utilisé pour l'établissement de la session PPP
enp0s1.525

# Les adresses sont attribuées par le "serveur" PPPoE
noipdefault
# L'adresse de résolution DNS est aussi fournie par le serveur PPPoE
usepeerdns
# La session PPP devient la route par défaut du routeur Spoke
defaultroute

# Demande de réouverture de session automatique en cas de rupture
persist

# Le routeur Spoke n'exige pas que le routeur Hub s'authentifie
noauth

# Messages d'informations détaillés dans la journalisation
debug

# Utilisation du protocole IPv6
+ipv6

# Options préconisées par la documentation
noaccomp
default-asyncmap
nodeflate
nopcomp
novj
novjccomp
lcp-echo-interval 10
```
```bash
# Le nom d'utilisateur désigne l'entrée du fichier /etc/ppp/chap-secrets
user spoke72

# Chargement du module PPPoE avec les détails dans la journalisation
plugin rp-pppoe.so rp_pppoe_ac BRAS rp_pppoe_verbose 1

# Interface (VLAN) utilisé pour l'établissement de la session PPP
enp0s1.527

# Les adresses sont attribuées par le "serveur" PPPoE
noipdefault
# L'adresse de résolution DNS est aussi fournie par le serveur PPPoE
usepeerdns
# La session PPP devient la route par défaut du routeur Spoke
defaultroute

# Demande de réouverture de session automatique en cas de rupture
persist

# Le routeur Spoke n'exige pas que le routeur Hub s'authentifie
noauth

# Messages d'informations détaillés dans la journalisation
debug

# Utilisation du protocole IPv6
+ipv6

# Options préconisées par la documentation
noaccomp
default-asyncmap
nodeflate
nopcomp
novj
novjccomp
lcp-echo-interval 10
```
### Création des unités systemd
**/etc/systemd/system/ppp.service**
```bash
[Unit]
Description=PPPoE Client Connection
After=network.target
Wants=network.target
BindsTo=sys-subsystem-net-devices-enp0s1.525.device
After=sys-subsystem-net-devices-enp0s1.525.device

[Service]
Type=forking
ExecStart=/usr/bin/pon pppoe-provider
ExecStop=/usr/bin/poff pppoe-provider
Restart=on-failure
RestartSec=20

[Install]
WantedBy=multi-user.target
```
```bash
[Unit]
Description=PPPoE Client Connection
After=network.target
Wants=network.target
BindsTo=sys-subsystem-net-devices-enp0s1.527.device
After=sys-subsystem-net-devices-enp0s1.527.device

[Service]
Type=forking
ExecStart=/usr/bin/pon pppoe-provider
ExecStop=/usr/bin/poff pppoe-provider
Restart=on-failure
RestartSec=20

[Install]
WantedBy=multi-user.target
```
### Lancement du servcie ppp
```bash
etu@spoke71:~$ sudo systemctl daemon-reload
etu@spoke71:~$ sudo systemctl enable ppp.service
Created symlink '/etc/systemd/system/multi-user.target.wants/ppp.service' → '/etc/systemd/system/ppp.service'.
etu@spoke71:~$ sudo systemctl start ppp.service
etu@spoke71:~$ systemctl status ppp.service
● ppp.service - PPPoE Client Connection
     Loaded: loaded (/etc/systemd/system/ppp.service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-10-08 17:03:47 CEST; 25min ago
 Invocation: 17434f04f9044a7a8c5029047edb9586
   Main PID: 2017 (pppd)
      Tasks: 1 (limit: 1086)
     Memory: 1.2M (peak: 3.1M)
        CPU: 163ms
     CGroup: /system.slice/ppp.service
             └─2017 /usr/sbin/pppd call pppoe-provider

oct. 08 17:22:25 spoke71 pppd[2017]: rcvd [IPCP ConfAck id=0x6 <addr 10.71.7.2>>
oct. 08 17:22:25 spoke71 pppd[2017]: Script /etc/ppp/ip-pre-up started (pid 207>
oct. 08 17:22:25 spoke71 pppd[2017]: Script /etc/ppp/ip-pre-up finished (pid 20>
oct. 08 17:22:25 spoke71 pppd[2017]: local  IP address 10.71.7.2  <-----
oct. 08 17:22:25 spoke71 pppd[2017]: remote IP address 10.71.7.1  <-----
oct. 08 17:22:25 spoke71 pppd[2017]: primary   DNS address 172.16.0.2
oct. 08 17:22:25 spoke71 pppd[2017]: secondary DNS address 172.16.0.2
oct. 08 17:22:25 spoke71 pppd[2017]: Script /etc/ppp/ip-up started (pid 2077)
oct. 08 17:22:25 spoke71 pppd[2017]: Script /etc/ppp/ipv6-up finished (pid 2072>
oct. 08 17:22:25 spoke71 pppd[2017]: Script /etc/ppp/ip-up finished (pid 2077),>
```

```bash
etu@spoke72:~$ sudo systemctl daemon-reload
etu@spoke72:~$ sudo systemctl enable ppp.service
Created symlink '/etc/systemd/system/multi-user.target.wants/ppp.service' → '/etc/systemd/system/ppp.service'.
etu@spoke72:~$ sudo systemctl start ppp.service
etu@spoke72:~$ systemctl status ppp.service
● ppp.service - PPPoE Client Connection
     Loaded: loaded (/etc/systemd/system/ppp.service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-10-08 17:03:49 CEST; 26min ago
 Invocation: 6933ec2f90944aeaa65be16fa7e4b818
   Main PID: 2050 (pppd)
      Tasks: 1 (limit: 1086)
     Memory: 1.2M (peak: 2.9M)
        CPU: 156ms
     CGroup: /system.slice/ppp.service
             └─2050 /usr/sbin/pppd call pppoe-provider

oct. 08 17:22:25 spoke72 pppd[2050]: rcvd [IPCP ConfAck id=0x6 <addr 10.72.7.2>>
oct. 08 17:22:25 spoke72 pppd[2050]: Script /etc/ppp/ip-pre-up started (pid 210>
oct. 08 17:22:25 spoke72 pppd[2050]: Script /etc/ppp/ip-pre-up finished (pid 21>
oct. 08 17:22:25 spoke72 pppd[2050]: local  IP address 10.72.7.2  <----
oct. 08 17:22:25 spoke72 pppd[2050]: remote IP address 10.72.7.1  <----
oct. 08 17:22:25 spoke72 pppd[2050]: primary   DNS address 172.16.0.2
oct. 08 17:22:25 spoke72 pppd[2050]: secondary DNS address 172.16.0.2
oct. 08 17:22:25 spoke72 pppd[2050]: Script /etc/ppp/ip-up started (pid 2112)
oct. 08 17:22:25 spoke72 pppd[2050]: Script /etc/ppp/ipv6-up finished (pid 2107>
oct. 08 17:22:25 spoke72 pppd[2050]: Script /etc/ppp/ip-up finished (pid 2112),>
```

### Vérification des routes par défaut après désactivation du réseau temporaire

```bash
etu@spoke71:~$ ip route ls
default dev ppp0 scope link 
10.71.7.1 dev ppp0 proto kernel scope link src 10.71.7.2 

etu@spoke71:~$ ip -6 route ls
fe80::74fb:e512:3556:b080 dev ppp0 proto kernel metric 256 pref medium
fe80::912a:ae57:152a:5607 dev ppp0 proto kernel metric 256 pref medium
fe80::/64 dev enp0s1 proto kernel metric 256 pref medium
fe80::/64 dev enp0s1.524 proto kernel metric 256 pref medium
fe80::/64 dev enp0s1.525 proto kernel metric 256 pref medium
fe80:20c::/64 dev enp0s1.524 proto kernel metric 256 pref medium

etu@spoke71:~$ ip route get 9.9.9.9
9.9.9.9 dev ppp0 src 10.71.7.2 uid 1000 
    cache 
```
```bash
etu@spoke72:~$ ip route ls
default dev ppp0 scope link 
10.72.7.1 dev ppp0 proto kernel scope link src 10.72.7.2 

etu@spoke72:~$ ip -6 route ls
fe80::7cd5:3f02:70d1:158e dev ppp0 proto kernel metric 256 pref medium
fe80::c5d5:fd30:e250:ef8 dev ppp0 proto kernel metric 256 pref medium
fe80::/64 dev enp0s1 proto kernel metric 256 pref medium
fe80::/64 dev enp0s1.526 proto kernel metric 256 pref medium
fe80::/64 dev enp0s1.527 proto kernel metric 256 pref medium
fe80:20e::/64 dev enp0s1.526 proto kernel metric 256 pref medium

etu@spoke72:~$ ip route get 9.9.9.9
9.9.9.9 dev ppp0 src 10.72.7.2 uid 1000 
    cache 
```
### Vérification des journaux

```bash
etu@spoke71:~$ journalctl --grep pppoe | grep -i pad
[....]
oct. 01 19:39:08 spoke71 pppd[495]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
oct. 01 19:39:08 spoke71 pppd[495]: Recv PPPOE Discovery V1T1 PADO session 0x0 length 44
oct. 01 19:39:13 spoke71 pppd[495]: Send PPPOE Discovery V1T1 PADR session 0x0 length 36
oct. 01 19:39:13 spoke71 pppd[495]: Recv PPPOE Discovery V1T1 PADS session 0x1 length 12
```

```bash
etu@spoke72:~$ journalctl --grep pppoe | grep -i pad
[....]
oct. 01 19:37:05 spoke72 pppd[494]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
oct. 01 19:37:05 spoke72 pppd[494]: Recv PPPOE Discovery V1T1 PADO session 0x0 length 44
oct. 01 19:37:10 spoke72 pppd[494]: Send PPPOE Discovery V1T1 PADR session 0x0 length 36
oct. 01 19:37:10 spoke72 pppd[494]: Recv PPPOE Discovery V1T1 PADS session 0x1 length 12
```

## 5. Interconnexion IPv4 et IPv6
```bash
etu@spoke71:~$ ping -qc2 9.9.9.9
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 30.474/38.751/47.029/8.277 ms
```

```bash
etu@spoke72:~$ ping -qc2 9.9.9.9
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 29.229/38.053/46.878/8.824 ms
```
### Ping entre Spoke71 et Spoke72

```bash
etu@spoke72:~$ ping -qc2 9.9.9.9
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 29.229/38.053/46.878/8.824 ms
```
```bash
etu@spoke72:~$ ping 10.71.7.2
PING 10.71.7.2 (10.71.7.2) 56(84) bytes of data.
64 bytes from 10.71.7.2: icmp_seq=1 ttl=63 time=2.00 ms
64 bytes from 10.71.7.2: icmp_seq=2 ttl=63 time=2.17 ms
64 bytes from 10.71.7.2: icmp_seq=3 ttl=63 time=2.29 ms
64 bytes from 10.71.7.2: icmp_seq=4 ttl=63 time=1.65 ms

--- 10.71.7.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 1.645/2.024/2.290/0.242 ms
```
### Routes par défaut (IPv4 & IPv6)
```bash
etu@spoke71:~$ ip -6 route get 2620:fe::fe
2620:fe::fe from :: dev ppp0 src fe80::ccd:d4ea:74da:27d8 metric 1024 pref medium
```
```bash
etu@spoke72:~$ ip -6 route get 2620:fe::fe
2620:fe::fe from :: dev ppp0 src fe80::5cb7:6dac:5e4b:61c8 metric 1024 pref medium
```
### Commutateurs virtuels
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s1:
      dhcp4: false
      dhcp6: false
      accept-ra: false

  openvswitch: {}

  bridges:
    asw-host:
      openvswitch: {}

  vlans:
    enp0s1.524: # VLAN violet
      id: 524
      link: enp0s1
      addresses:
        - fe80:20C::2/64 ####
    enp0s1.525: # VLAN orange
      id: 525
      link: enp0s1
      addresses: []
    vlan50:     # VLAN vert
      id: 50
      link: asw-host
      addresses:
        - 100.64.71.1/24
        - fda0:7a62:47::1/64
        - fe80:47::1/64
```
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s1:
      dhcp4: false
      dhcp6: false
      accept-ra: false

  openvswitch: {}

  bridges:
    asw-host:
      openvswitch: {}

  vlans:
    enp0s1.526: # VLAN violet
      id: 526
      link: enp0s1
      addresses:
        - fe80:20E::2/64
    enp0s1.527: # VLAN orange
      id: 527
      link: enp0s1
      addresses: []
    vlan51:     # VLAN vert
      id: 51
      link: asw-host
      addresses:
        - 100.64.72.1/24
        - fda0:7a62:48::1/64
        - fe80:48::1/64
```
### Comminication entre les routeurs spoke
```bash
etu@spoke71:~$ ping -qc2 100.64.72.1
PING 100.64.72.1 (100.64.72.1) 56(84) bytes of data.

--- 100.64.72.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 2.619/2.630/2.641/0.011 ms

etu@spoke71:~$ ping -qc2 fda0:7a62:48::1
PING fda0:7a62:48::1 (fda0:7a62:48::1) 56 data bytes

--- fda0:7a62:48::1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.909/2.043/2.178/0.134 ms
```
```bash
etu@spoke72:~$ ping -qc2 100.64.71.1
PING 100.64.71.1 (100.64.71.1) 56(84) bytes of data.

--- 100.64.71.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 2.028/2.117/2.207/0.089 ms

etu@spoke72:~$ ping -qc2 fda0:7a62:47::1
PING fda0:7a62:47::1 (fda0:7a62:47::1) 56 data bytes

--- fda0:7a62:47::1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 2.240/2.326/2.412/0.086 ms
```
## 6. Installation et gestion des conteneurs

```bash
etu@spoke71:~$ incus ls                            
+------+---------+----------------------+------------------------------------------+-----------+-----------+
| NAME |  STATE  |         IPV4         |                   IPV6                   |   TYPE    | SNAPSHOTS |
+------+---------+----------------------+------------------------------------------+-----------+-----------+
| c0   | RUNNING | 100.64.71.157 (eth0) | fda0:7a62:47:0:216:3eff:fe34:78db (eth0) | CONTAINER | 0         |
+------+---------+----------------------+------------------------------------------+-----------+-----------+
| c1   | RUNNING | 100.64.71.168 (eth0) | fda0:7a62:47:0:216:3eff:fe1c:6ba3 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+------------------------------------------+-----------+-----------+
| c2   | RUNNING | 100.64.71.108 (eth0) | fda0:7a62:47:0:216:3eff:fe04:9b75 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+------------------------------------------+-----------+-----------+
```
### Tests de conectivité 
```bash
>>>>>>>>>>>>>>>>> c0
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 29.669/30.345/31.021/0.676 ms
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 39.555/54.648/69.741/15.093 ms
>>>>>>>>>>>>>>>>> c1
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 29.857/30.236/30.615/0.379 ms
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 42.947/43.056/43.166/0.109 ms
>>>>>>>>>>>>>>>>> c2
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 30.664/30.791/30.918/0.127 ms
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 40.662/41.870/43.079/1.208 ms
```
### Adressage statique
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
```

### Mêmes étapes sur le second spoke
```bash
etu@spoke72:~$ incus ls                              
+------+---------+----------------------+------------------------------------------+-----------+-----------+
| NAME |  STATE  |         IPV4         |                   IPV6                   |   TYPE    | SNAPSHOTS |
+------+---------+----------------------+------------------------------------------+-----------+-----------+
| c0   | RUNNING | 100.64.72.158 (eth0) | fda0:7a62:48:0:216:3eff:fe81:89a (eth0)  | CONTAINER | 0         |
+------+---------+----------------------+------------------------------------------+-----------+-----------+
| c1   | RUNNING | 100.64.72.128 (eth0) | fda0:7a62:48:0:216:3eff:feca:94d8 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+------------------------------------------+-----------+-----------+
| c2   | RUNNING | 100.64.72.188 (eth0) | fda0:7a62:48:0:216:3eff:fe03:40d2 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+------------------------------------------+-----------+-----------+
```

```bash
>>>>>>>>>>>>>>>>> c0
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 29.915/30.644/31.374/0.729 ms
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 40.408/49.754/59.101/9.346 ms
>>>>>>>>>>>>>>>>> c1
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 29.808/30.134/30.461/0.326 ms
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 41.303/41.768/42.233/0.465 ms
>>>>>>>>>>>>>>>>> c2
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 29.468/29.962/30.457/0.494 ms
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 40.385/40.980/41.575/0.595 ms
```
```bash
etu@spoke72:~$ incus ls
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
```
### Ouvrir et tester des services Web dans les réseaux d'hébergement
```bash
>>>>>>>>>>>>>>>>> c0
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    
LISTEN    0         511                0.0.0.0:http             0.0.0.0:*       
LISTEN    0         511                   [::]:http                [::]:*       
>>>>>>>>>>>>>>>>> c1
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    
LISTEN    0         511                0.0.0.0:http             0.0.0.0:*       
LISTEN    0         511                   [::]:http                [::]:*       
>>>>>>>>>>>>>>>>> c2
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    
LISTEN    0         511                0.0.0.0:http             0.0.0.0:*       
LISTEN    0         511                   [::]:http                [::]:*       
etu@spoke71:~$ 
```
```bash
>>>>>>>>>>>>>>>>> c0
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    
LISTEN    0         511                0.0.0.0:http             0.0.0.0:*       
LISTEN    0         511                   [::]:http                [::]:*       
>>>>>>>>>>>>>>>>> c1
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    
LISTEN    0         511                0.0.0.0:http             0.0.0.0:*       
LISTEN    0         511                   [::]:http                [::]:*       
>>>>>>>>>>>>>>>>> c2
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    
LISTEN    0         511                0.0.0.0:http             0.0.0.0:*       
LISTEN    0         511                   [::]:http                [::]:*       
etu@spoke72:~$ 
```

### Comment valider l'accès à ces services Web à partir du routeur Spoke situé à l'autre extrémité de la topologie en triangle ?
```bash
etu@spoke71:~$ for addr in {10..12}
do
    wget -O /dev/null http://100.64.71.$addr 2>&1 | grep "HTTP "
done
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
```
```bash
etu@spoke71:~$ for addr in {10..12}
do
    wget -O /dev/null http://[fda0:7a62:47::$(printf "%x" $addr)] 2>&1 | grep "HTTP "
done
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
```
---
```bash
etu@spoke72:~$ for addr in {10..12}
do
    wget -O /dev/null http://100.64.72.$addr 2>&1 | grep "HTTP "
done
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
```
```bash
etu@spoke72:~$ for addr in {10..12}
do
    wget -O /dev/null http://[fda0:7a62:48::$(printf "%x" $addr)] 2>&1 | grep "HTTP "
done
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
requête HTTP transmise, en attente de la réponse… 200 OK
```
### Comment matérialiser le chemin suivi par les paquets IPv4 ou IPv6 entre les deux routeurs Spoke ?

```bash
etu@spoke71:~$ for i in {0..2}
do
    echo ">>>>>>>>>>>>>>>>> c$i"
    incus exec c$i -- tracepath -n 100.64.72.$(($i + 10))
done
>>>>>>>>>>>>>>>>> c0
 1?: [LOCALHOST]                      pmtu 1500
 1:  100.64.71.1                                           0.692ms 
 1:  100.64.71.1                                           0.096ms 
 2:  100.64.71.1                                           0.064ms pmtu 1492
 2:  10.71.7.1                                             1.414ms 
 3:  10.72.7.2                                             1.970ms 
 4:  100.64.72.10                                          3.172ms reached
     Resume: pmtu 1492 hops 4 back 4 
>>>>>>>>>>>>>>>>> c1
 1?: [LOCALHOST]                      pmtu 1500
 1:  100.64.71.1                                           0.707ms 
 1:  100.64.71.1                                           0.055ms 
 2:  100.64.71.1                                           0.039ms pmtu 1492
 2:  10.71.7.1                                             1.170ms 
 3:  10.72.7.2                                             1.805ms 
 4:  100.64.72.11                                          2.440ms reached
     Resume: pmtu 1492 hops 4 back 4 
>>>>>>>>>>>>>>>>> c2
 1?: [LOCALHOST]                      pmtu 1500
 1:  100.64.71.1                                           0.707ms 
 1:  100.64.71.1                                           0.069ms 
 2:  100.64.71.1                                           0.049ms pmtu 1492
 2:  10.71.7.1                                             1.039ms 
 3:  10.72.7.2                                             1.848ms 
 4:  100.64.72.12                                          2.448ms reached
     Resume: pmtu 1492 hops 4 back 4 
```
```bash
etu@spoke71:~$ for i in {0..2}
do
    echo ">>>>>>>>>>>>>>>>> c$i"
    incus exec c$i -- tracepath -n fda0:7a62:48::$(printf "%x" $(($i + 10)))
done
>>>>>>>>>>>>>>>>> c0
 1?: [LOCALHOST]                        0.039ms pmtu 1500
 1:  fda0:7a62:47::1                                       1.240ms 
 1:  fda0:7a62:47::1                                       0.144ms 
 2:  fda0:7a62:47::1                                       0.110ms pmtu 1492
 2:  2001:678:3fc:6b:baad:caff:fefe:137                    1.540ms 
 3:  fda0:7a62:48::1                                       2.465ms 
 4:  fda0:7a62:48::a                                       3.301ms reached
     Resume: pmtu 1492 hops 4 back 4 
>>>>>>>>>>>>>>>>> c1
 1?: [LOCALHOST]                        0.038ms pmtu 1500
 1:  fda0:7a62:47::1                                       2.529ms 
 1:  fda0:7a62:47::1                                       0.161ms 
 2:  fda0:7a62:47::1                                       0.070ms pmtu 1492
 2:  2001:678:3fc:6b:baad:caff:fefe:137                    1.388ms 
 3:  fda0:7a62:48::1                                       2.011ms 
 4:  fda0:7a62:48::b                                       3.706ms reached
     Resume: pmtu 1492 hops 4 back 4 
>>>>>>>>>>>>>>>>> c2
 1?: [LOCALHOST]                        0.029ms pmtu 1500
 1:  fda0:7a62:47::1                                       0.746ms 
 1:  fda0:7a62:47::1                                       0.097ms 
 2:  fda0:7a62:47::1                                       0.068ms pmtu 1492
 2:  2001:678:3fc:6b:baad:caff:fefe:137                    1.519ms 
 3:  fda0:7a62:48::1                                       2.320ms 
 4:  fda0:7a62:48::c                                       3.142ms reached
     Resume: pmtu 1492 hops 4 back 4 
```

```bash
etu@spoke72:~$ for i in {0..2}
do
    echo ">>>>>>>>>>>>>>>>> c$i"
    incus exec c$i -- tracepath -n 100.64.71.$(($i + 10))
done
>>>>>>>>>>>>>>>>> c0
 1?: [LOCALHOST]                      pmtu 1500
 1:  100.64.72.1                                           0.723ms 
 1:  100.64.72.1                                           0.044ms 
 2:  100.64.72.1                                           0.035ms pmtu 1492
 2:  10.72.7.1                                             1.214ms 
 3:  10.71.7.2                                             1.729ms 
 4:  100.64.71.10                                          2.602ms reached
     Resume: pmtu 1492 hops 4 back 4 
>>>>>>>>>>>>>>>>> c1
 1?: [LOCALHOST]                      pmtu 1500
 1:  100.64.72.1                                           2.296ms 
 1:  100.64.72.1                                           0.091ms 
 2:  100.64.72.1                                           0.092ms pmtu 1492
 2:  10.72.7.1                                             1.119ms 
 3:  10.71.7.2                                             1.666ms 
 4:  100.64.71.11                                          2.200ms reached
     Resume: pmtu 1492 hops 4 back 4 
>>>>>>>>>>>>>>>>> c2
 1?: [LOCALHOST]                      pmtu 1500
 1:  100.64.72.1                                           1.268ms 
 1:  100.64.72.1                                           0.075ms 
 2:  100.64.72.1                                           0.043ms pmtu 1492
 2:  10.72.7.1                                             1.242ms 
 3:  10.71.7.2                                             1.711ms 
 4:  100.64.71.12                                          2.579ms reached
     Resume: pmtu 1492 hops 4 back 4 
```
```bash
etu@spoke72:~$ for i in {0..2}
do
    echo ">>>>>>>>>>>>>>>>> c$i"
    incus exec c$i -- tracepath -n fda0:7a62:47::$(printf "%x" $(($i + 10)))
done
>>>>>>>>>>>>>>>>> c0
 1?: [LOCALHOST]                        0.040ms pmtu 1500
 1:  fda0:7a62:48::1                                       1.014ms 
 1:  fda0:7a62:48::1                                       0.068ms 
 2:  fda0:7a62:48::1                                       0.050ms pmtu 1492
 2:  2001:678:3fc:6b:baad:caff:fefe:137                    1.403ms 
 3:  fda0:7a62:47::1                                       1.711ms 
 4:  fda0:7a62:47::a                                       2.840ms reached
     Resume: pmtu 1492 hops 4 back 4 
>>>>>>>>>>>>>>>>> c1
 1?: [LOCALHOST]                        0.045ms pmtu 1500
 1:  fda0:7a62:48::1                                       0.589ms 
 1:  fda0:7a62:48::1                                       0.105ms 
 2:  fda0:7a62:48::1                                       0.071ms pmtu 1492
 2:  2001:678:3fc:6b:baad:caff:fefe:137                    1.102ms 
 3:  fda0:7a62:47::1                                       1.552ms 
 4:  fda0:7a62:47::b                                       2.273ms reached
     Resume: pmtu 1492 hops 4 back 4 
>>>>>>>>>>>>>>>>> c2
 1?: [LOCALHOST]                        0.031ms pmtu 1500
 1:  fda0:7a62:48::1                                       0.762ms 
 1:  fda0:7a62:48::1                                       0.071ms 
 2:  fda0:7a62:48::1                                       0.043ms pmtu 1492
 2:  2001:678:3fc:6b:baad:caff:fefe:137                    1.102ms 
 3:  fda0:7a62:47::1                                       1.640ms 
 4:  fda0:7a62:47::c                                       2.786ms reached
     Resume: pmtu 1492 hops 4 back 4 
```