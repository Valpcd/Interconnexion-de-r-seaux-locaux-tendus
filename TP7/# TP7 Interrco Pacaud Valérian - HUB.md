# TP7 Interrco Pacaud Valérian - HUB
## 3. Configurer les Routeurs Hub
### 3.1. Configurer les serveurs PPPoE
```bash
etu@HUB160:~$ systemctl status pppoe-server1.service
● pppoe-server1.service - PPPoE Server
     Loaded: loaded (/etc/systemd/system/pppoe-server1.service; enabled; preset>
     Active: active (running) since Sun 2024-11-17 22:05:03 CET; 10h ago
 Invocation: 293f11932392405b95261a044d090242
   Main PID: 520 (pppoe-server)
      Tasks: 4 (limit: 1032)
     Memory: 3.6M (peak: 5.6M)
        CPU: 21.628s
     CGroup: /system.slice/pppoe-server1.service
             ├─  520 /usr/sbin/pppoe-server -I enp0s1.613 -C BRAS -L 10.161.16.>
             ├─13064 pppd pty "/usr/sbin/pppoe -n -I enp0s1.613 -e 1:b8:ad:ca:f>
             ├─13065 sh -c "/usr/sbin/pppoe -n -I enp0s1.613 -e 1:b8:ad:ca:fe:0>
             └─13066 /usr/sbin/pppoe -n -I enp0s1.613 -e 1:b8:ad:ca:fe:01:3b -S

nov. 18 08:15:29 HUB160 pppd[13064]: rcvd [IPCP ConfReq id=0x2 <addr 10.161.16.>
nov. 18 08:15:29 HUB160 pppd[13064]: sent [IPCP ConfAck id=0x2 <addr 10.161.16.>
nov. 18 08:15:29 HUB160 pppd[13064]: Script /etc/ppp/ip-pre-up started (pid 130>
nov. 18 08:15:29 HUB160 pppd[13064]: Script /etc/ppp/ip-pre-up finished (pid 13>
nov. 18 08:15:29 HUB160 pppd[13064]: local  IP address 10.161.16.1
nov. 18 08:15:29 HUB160 pppd[13064]: remote IP address 10.161.16.2
nov. 18 08:15:29 HUB160 pppd[13064]: Script /etc/ppp/ip-up started (pid 13081)
nov. 18 08:15:29 HUB160 pppd[13064]: Script /etc/ppp/ipv6-up finished (pid 1307>
nov. 18 08:15:29 HUB160 pppd[13064]: rcvd [CCP ConfAck id=0x2]

etu@HUB160:~$ systemctl status pppoe-server2.service
● pppoe-server2.service - PPPoE Server
     Loaded: loaded (/etc/systemd/system/pppoe-server2.service; enabled; preset>
     Active: active (running) since Sun 2024-11-17 22:05:03 CET; 10h ago
 Invocation: adb7d1cc31a14fc6a929c3b8182e5dd7
   Main PID: 511 (pppoe-server)
      Tasks: 4 (limit: 1032)
     Memory: 2.5M (peak: 4.4M)
        CPU: 21.761s
     CGroup: /system.slice/pppoe-server2.service
             ├─  511 /usr/sbin/pppoe-server -I enp0s1.615 -C BRAS -L 10.162.16.>
             ├─13150 pppd pty "/usr/sbin/pppoe -n -I enp0s1.615 -e 1:b8:ad:ca:f>
             ├─13151 sh -c "/usr/sbin/pppoe -n -I enp0s1.615 -e 1:b8:ad:ca:fe:0>
             └─13152 /usr/sbin/pppoe -n -I enp0s1.615 -e 1:b8:ad:ca:fe:01:3c -S

nov. 18 08:18:50 HUB160 pppd[13150]: rcvd [IPCP ConfReq id=0x2 <addr 10.162.16.>
nov. 18 08:18:50 HUB160 pppd[13150]: sent [IPCP ConfAck id=0x2 <addr 10.162.16.>
nov. 18 08:18:50 HUB160 pppd[13150]: Script /etc/ppp/ip-pre-up started (pid 131>
nov. 18 08:18:50 HUB160 pppd[13150]: Script /etc/ppp/ip-pre-up finished (pid 13>
nov. 18 08:18:50 HUB160 pppd[13150]: local  IP address 10.162.16.1
nov. 18 08:18:50 HUB160 pppd[13150]: remote IP address 10.162.16.2
nov. 18 08:18:50 HUB160 pppd[13150]: Script /etc/ppp/ip-up started (pid 13159)
nov. 18 08:18:50 HUB160 pppd[13150]: Script /etc/ppp/ipv6-up finished (pid 1315>
nov. 18 08:18:50 HUB160 pppd[13150]: rcvd [CCP ConfAck id=0x2]
```
```bash
etu@HUB160:~$ ip a | grep ppp
1995: ppp0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1492 qdisc fq_codel state UNKNOWN group default qlen 3
    link/ppp 
    inet 10.161.16.1 peer 10.161.16.2/32 scope global ppp0
1997: ppp1: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1492 qdisc fq_codel state UNKNOWN group default qlen 3
    link/ppp 
    inet 10.162.16.1 peer 10.162.16.2/32 scope global ppp1
```
### 3.2. Installer et configurer FRR
```bash
etu@HUB160:~$ systemctl status frr
● frr.service - FRRouting
     Loaded: loaded (/usr/lib/systemd/system/frr.service; enabled; preset: enab>
     Active: active (running) since Mon 2024-11-18 08:43:50 CET; 1min 14s ago
 Invocation: 8a2b2f20718544ce89810f5839744af6
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 14224 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status>
   Main PID: 14233 (watchfrr)
     Status: "FRR Operational"
      Tasks: 8 (limit: 1032)
     Memory: 15.1M (peak: 29.3M)
        CPU: 304ms
     CGroup: /system.slice/frr.service
             ├─14233 /usr/lib/frr/watchfrr -d -F traditional zebra mgmtd staticd
             ├─14243 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000>
             ├─14248 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             └─14250 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

nov. 18 08:43:50 HUB160 frrinit.sh[14270]: [14270|staticd] done
nov. 18 08:43:50 HUB160 watchfrr[14233]: [VTVCM-Y2NW3] Configuration Read in To>
nov. 18 08:43:50 HUB160 frrinit.sh[14254]: [14254|zebra] done
nov. 18 08:43:50 HUB160 frrinit.sh[14268]: [14268|watchfrr] done
nov. 18 08:43:50 HUB160 watchfrr[14233]: [QDG3Y-BY5TN] zebra state -> up : conn>
nov. 18 08:43:50 HUB160 watchfrr[14233]: [QDG3Y-BY5TN] mgmtd state -> up : conn>
```
```bash
etu@HUB160:~$ vtysh

Hello, this is FRRouting (version 10.2).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

HUB160# sh daemon
 mgmtd zebra ospfd ospf6d watchfrr staticd
```
### 3.3. Configurer les démons OSPFv2 et OSPFv3
```bash
HUB160# sh ip ospf interface enp0s1.583
enp0s1.583 is up
  ifindex 6, MTU 1500 bytes, BW 0 Mbit <UP,LOWER_UP,BROADCAST,RUNNING,MULTICAST>
  Internet Address 172.19.10.40/29, Broadcast 172.19.10.47, Area 0.0.0.0
  MTU mismatch detection: enabled
  Router ID 0.4.4.160, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State Backup, Priority 1
  Designated Router (ID) 0.4.4.170 Interface Address 172.19.10.41/29
  Backup Designated Router (ID) 0.4.4.160, Interface Address 172.19.10.40
  Multicast group memberships: OSPFAllRouters OSPFDesignatedRouters
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
    Hello due in 2.736s
  Neighbor Count is 1, Adjacent neighbor count is 1    <----------
  Graceful Restart hello delay: 10s
  LSA retransmissions: 1
```
```bash
HUB160# sh ipv6 ospf6 interface enp0s1.583
enp0s1.583 is up, type BROADCAST
  Interface ID: 6
  Internet Address:
    inet : 172.19.10.40/29
    inet6: fe80:247::1/64
    inet6: fe80::baad:caff:fefe:13e/64
  Instance ID 0, Interface MTU 1500 (autodetect: 1500)
  MTU mismatch detection: enabled
  Area ID 0.0.0.0, Cost 10
  State BDR, Transmit Delay 1 sec, Priority 1
  Timer intervals configured:
   Hello 10(1.365), Dead 40, Retransmit 5
  DR: 0.4.6.170 BDR: 0.4.4.160                       <----------
  Number of I/F scoped LSAs is 2
    0 Pending LSAs for LSUpdate in Time 00:00:00 [thread off]
    0 Pending LSAs for LSAck in Time 00:00:00 [thread off]
  Graceful Restart hello delay: 10s
  Authentication Trailer is disabled
```
```bash
HUB160# sh ip ospf database

       OSPF Router with ID (0.4.4.160)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age  Seq#       CkSum  Link count
0.4.4.160      0.4.4.160        946 0x8000007c 0x296d 1
0.4.4.170      0.4.4.170       1037 0x800000e1 0xcb50 1

                Net Link States (Area 0.0.0.0)

Link ID         ADV Router      Age  Seq#       CkSum
172.19.10.41   0.4.4.170       1127 0x8000007a 0xb82e

                AS External Link States

Link ID         ADV Router      Age  Seq#       CkSum  Route
0.0.0.0        0.4.4.160       1496 0x8000005f 0x8a37 E2 0.0.0.0/0 [0x0]
10.161.16.2    0.4.4.160       1076 0x80000076 0xe5fc E2 10.161.16.2/32 [0x0]
10.162.16.2    0.4.4.160       1096 0x80000076 0xd908 E2 10.162.16.2/32 [0x0]
10.171.17.2    0.4.4.170        777 0x80000073 0x2ca4 E2 10.171.17.2/32 [0x0]
10.172.17.2    0.4.4.170       1397 0x80000073 0x20af E2 10.172.17.2/32 [0x0]
192.168.115.0  0.4.4.160       1456 0x80000076 0x1c27 E2 192.168.115.0/25 [0x0]
192.168.116.1600.4.4.170       1177 0x80000073 0xd663 E2 192.168.116.160/27 [0x0]
```
```bash
HUB160# 
HUB160# sh ip ospf neighbor

Neighbor ID     Pri State           Up Time         Dead Time Address         Interface                        RXmtL RqstL DBsmL
0.4.4.170         1 Full/DR         8m41s             38.069s 172.19.10.41    enp0s1.583:172.19.10.40              0     0     0

HUB160# sh ipv6 ospf6 neighbor
Neighbor ID     Pri    DeadTime    State/IfState         Duration I/F[State]
0.4.6.170         1    00:00:30     Full/DR              00:08:30 enp0s1.583[BDR]
```
### 3.4. Redistribuer les routes connectées dans OSPF
```bash
HUB160# sh ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O>* 10.171.17.2/32 [110/20] via 172.19.10.41, enp0s1.583, weight 1, 00:19:38
O>* 10.172.17.2/32 [110/20] via 172.19.10.41, enp0s1.583, weight 1, 00:19:38
O   172.19.10.40/29 [110/10] is directly connected, enp0s1.583, weight 1, 01:56:19
O>* 192.168.116.160/27 [110/20] via 172.19.10.41, enp0s1.583, weight 1, 00:19:38
```
### 5.2. Router les réseaux d'hébergement depuis les Hubs
```bash
etu@HUB160:~$ ip route ls dev ppp0
10.161.16.2 proto kernel scope link src 10.161.16.1 
100.64.10.0/24 scope link 
etu@HUB160:~$ ip route ls dev ppp1
10.162.16.2 proto kernel scope link src 10.162.16.1 
100.64.11.0/24 scope link 
etu@HUB160:~$ vtysh

Hello, this is FRRouting (version 10.2).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

HUB160# sh run ospfd
Building configuration...

Current configuration:
!
frr version 10.2
frr defaults traditional
hostname HUB160
log syslog informational
service integrated-vtysh-config
!
interface enp0s1.583
 ip ospf area 0
exit
!
router ospf
 ospf router-id 0.4.4.160
 log-adjacency-changes detail
 redistribute connected      <-------
 default-information originate
exit
!
end
HUB160# sh run ospf6d
Building configuration...

Current configuration:
!
frr version 10.2
frr defaults traditional
hostname HUB160
log syslog informational
service integrated-vtysh-config
!
interface enp0s1.583
 ipv6 ospf6 area 0
exit
!
router ospf6
 ospf6 router-id 0.4.4.160
 log-adjacency-changes detail
 redistribute connected     <-----
exit
!
end
HUB160# sh ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O>* 10.171.17.2/32 [110/20] via 172.19.10.41, enp0s1.583, weight 1, 00:02:38
O>* 10.172.17.2/32 [110/20] via 172.19.10.41, enp0s1.583, weight 1, 00:02:38
O   172.19.10.40/29 [110/10] is directly connected, enp0s1.583, weight 1, 00:03:29
O>* 192.168.116.160/27 [110/20] via 172.19.10.41, enp0s1.583, weight 1, 00:02:38
HUB160# sh ipv6 route ospf6
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIPng, O - OSPFv3, I - IS-IS, B - BGP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O>* 2001:678:3fc:74::/64 [110/20] via fe80::baad:caff:fefe:143, enp0s1.583, weight 1, 00:01:58
```

### 6.2 Afficher les tables de routage complètes
```bash
HUB160# sh ip route
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

K>* 0.0.0.0/0 [0/0] via 192.168.115.1, enp0s1.115, weight 1, 1d20h08m
L>* 10.161.16.1/32 is directly connected, ppp0, weight 1, 1d20h07m
C>* 10.161.16.2/32 is directly connected, ppp0, weight 1, 1d20h07m
K * 10.161.16.2/32 [0/0] is directly connected, ppp0, weight 1, 1d20h07m
L>* 10.162.16.1/32 is directly connected, ppp1, weight 1, 1d20h07m
C>* 10.162.16.2/32 is directly connected, ppp1, weight 1, 1d20h07m
K * 10.162.16.2/32 [0/0] is directly connected, ppp1, weight 1, 1d20h07m
O>* 10.171.17.2/32 [110/20] via 172.19.10.41, enp0s1.583, weight 1, 1d20h07m
O>* 10.172.17.2/32 [110/20] via 172.19.10.41, enp0s1.583, weight 1, 1d20h07m
K>* 100.64.22.0/24 [0/0] is directly connected, ppp0, weight 1, 1d20h07m
K>* 100.64.23.0/24 [0/0] is directly connected, ppp1, weight 1, 1d20h07m
O   172.19.10.40/29 [110/10] is directly connected, enp0s1.583, weight 1, 1d20h08m
C>* 172.19.10.40/29 is directly connected, enp0s1.583, weight 1, 1d20h08m
L>* 172.19.10.40/32 is directly connected, enp0s1.583, weight 1, 1d20h08m
C>* 192.168.115.0/25 is directly connected, enp0s1.115, weight 1, 1d20h08m
L>* 192.168.115.2/32 is directly connected, enp0s1.115, weight 1, 1d20h08m
O>* 192.168.116.160/27 [110/20] via 172.19.10.41, enp0s1.583, weight 1, 1d20h07m
```
```bash
HUB160# sh ipv6 route
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIPng, O - OSPFv3, I - IS-IS, B - BGP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

K>* ::/0 [0/1024] via fe80:73::1, enp0s1.115 onlink, weight 1, 1d20h08m
C>* 2001:678:3fc:73::/64 is directly connected, enp0s1.115, weight 1, 1d20h08m
K * 2001:678:3fc:73::/64 [0/256] is directly connected, enp0s1.115, weight 1, 1d20h08m
L>* 2001:678:3fc:73::2/128 is directly connected, enp0s1.115, weight 1, 1d20h08m
O>* 2001:678:3fc:74::/64 [110/20] via fe80::baad:caff:fefe:143, enp0s1.583, weight 1, 1d20h07m
K>* fda0:7a62:16::/64 [0/1024] is directly connected, ppp0, weight 1, 1d20h08m
K>* fda0:7a62:17::/64 [0/1024] is directly connected, ppp1, weight 1, 1d20h08m
C * fe80::/64 is directly connected, enp0s1.615, weight 1, 1d20h08m
C * fe80::/64 is directly connected, enp0s1.614, weight 1, 1d20h08m
C * fe80::/64 is directly connected, enp0s1.613, weight 1, 1d20h08m
C * fe80::/64 is directly connected, enp0s1.612, weight 1, 1d20h08m
C * fe80::/64 is directly connected, enp0s1, weight 1, 1d20h08m
C * fe80::/64 is directly connected, enp0s1.115, weight 1, 1d20h08m
C>* fe80::/64 is directly connected, enp0s1.583, weight 1, 1d20h08m
C>* fe80::3093:e753:327e:abf3/128 is directly connected, ppp1, weight 1, 1d20h08m
C>* fe80::6512:ec68:c345:bb2b/128 is directly connected, ppp0, weight 1, 1d20h08m
C>* fe80:247::/64 is directly connected, enp0s1.583, weight 1, 1d20h08m
C>* fe80:264::/64 is directly connected, enp0s1.612, weight 1, 1d20h08m
C>* fe80:266::/64 is directly connected, enp0s1.614, weight 1, 1d20h08m
```
```bash
HUB160# sh int brief
Interface       Status  VRF             Addresses
---------       ------  ---             ---------
enp0s1          up      default         fe80::baad:caff:fefe:13e/64
enp0s1.115      up      default         192.168.115.2/25
                                        + fe80::baad:caff:fefe:13e/64
enp0s1.583      up      default         172.19.10.40/29
                                        + fe80::baad:caff:fefe:13e/64
enp0s1.612      up      default         + fe80::baad:caff:fefe:13e/64
enp0s1.613      up      default         fe80::baad:caff:fefe:13e/64
enp0s1.614      up      default         + fe80:266::1/64
enp0s1.615      up      default         fe80::baad:caff:fefe:13e/64
lo              up      default         
ppp0            up      default         10.161.16.1/32
                                        fe80::a90a:f714:5168:d77/128
ppp1            up      default         10.162.16.1/32
                                        fe80::94e8:c883:7447:2b86/128
```

