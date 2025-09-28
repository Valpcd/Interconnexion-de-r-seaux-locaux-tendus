# TP5-6 Interrco Pacaud Valérian ROUTEUR 1
```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP5-6$ nano switch.yaml
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP5-6$ $HOME/masters/scripts/switch-conf.py switch.yaml
----------------------------------------
Configuring switch dsw-host
>> Port tap319 vlan_mode is already set to trunk
>> Port tap319 trunks set to [306, 518, 519]
----------------------------------------
```
```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP5-6$ $HOME/masters/scripts/lab-startup.py frr-lab.yaml
Copying /home/valerianpacaud/masters/debian-testing-amd64.qcow2 to R1.qcow2...done
Creating OVMF_CODE.fd symlink...
Creating R1_OVMF_VARS.fd file...
Starting R1...
~> Virtual machine filename   : R1.qcow2
~> RAM size                   : 1024MB
~> SPICE VDI port number      : 6219
~> telnet console port number : 2619
~> MAC address                : b8:ad:ca:fe:01:3f
~> Switch port interface      : tap319, trunk mode
~> IPv6 LL address            : fe80::baad:caff:fefe:13f%dsw-host
R1 started!
```
## 3.2. Activer le routage sur les routeurs virtuels

```bash
etu@R1:~$ cat << EOF | sudo tee /etc/sysctl.d/10-routing.conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
net.ipv4.conf.all.log_martians=1
EOF
[sudo] Mot de passe de etu : 
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
net.ipv4.conf.all.log_martians=1
```
```bash
etu@R1:~$ sudo sysctl net.ipv4.ip_forward net.ipv6.conf.all.forwarding
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```

## 3.3. Appliquer une première configuration réseau

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
      # Infrastructure VLAN
      enp0s1.306:
        id: 306
        link: enp0s1
        addresses:
          - 10.0.50.170/29 
          - 2001:678:3fc:132::2/64
        routes:
          - to: default
            via: 10.0.50.169
          - to: "::/0"
            via: "fe80:132::1" ## Attention erreur dans le fichier d'origine avec les "::"
            on-link: true
      # R1 -> R2
      enp0s1.518:
        id: 518
        link: enp0s1
        addresses:
          - 10.7.12.1/29
          - fe80::206:1/64
      # R1 -> R3
      enp0s1.519:
        id: 519
        link: enp0s1
        addresses:
          - 10.7.13.1/29
          - fe80::206:1/64
```
```bash
etu@R1:~$ sudo netplan apply

** (generate:551): WARNING **: 11:44:25.281: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.
[  137.970795] systemd-ssh-generator[576]: Binding SSH to AF_VSOCK vsock::22.
[  137.971324] systemd-ssh-generator[576]: → connect via 'ssh vsock/4294967295' from host
[  137.971852] systemd-ssh-generator[576]: Binding SSH to AF_UNIX socket /run/ssh-unix-local/socket.
[  137.972377] systemd-ssh-generator[576]: → connect via 'ssh .host' locally

** (process:549): WARNING **: 11:44:25.631: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:549): WARNING **: 11:44:25.706: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.
[  138.422963] 8021q: 802.1Q VLAN Support v1.8
[  138.423275] 8021q: adding VLAN 0 to HW filter on device enp0s1
```

## 4. Installer le paquet frr et lancer les démons de routage OSPF
```bash
etu@R1:~$ systemctl status frr
● frr.service - FRRouting
     Loaded: loaded (/usr/lib/systemd/system/frr.service; enabled; preset: enab>
     Active: active (running) since Tue 2024-11-12 11:03:37 CET; 37s ago
 Invocation: 4f137da023274fc0a667806935ff9926
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 1916 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=>
   Main PID: 1925 (watchfrr)
     Status: "FRR Operational"
      Tasks: 8 (limit: 1032)
     Memory: 15.1M (peak: 27.6M)
        CPU: 330ms
     CGroup: /system.slice/frr.service
             ├─1925 /usr/lib/frr/watchfrr -d -F traditional zebra mgmtd staticd
             ├─1935 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 900000>
             ├─1940 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             └─1942 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

nov. 12 11:03:37 R1 staticd[1942]: [VTVCM-Y2NW3] Configuration Read in Took: 00>
nov. 12 11:03:37 R1 frrinit.sh[1962]: [1962|staticd] done
nov. 12 11:03:37 R1 mgmtd[1940]: [VTVCM-Y2NW3] Configuration Read in Took: 00:0>
nov. 12 11:03:37 R1 frrinit.sh[1945]: [1945|mgmtd] done
```

```bash
etu@R1:~$ sudo grep "service integrated-vtysh-config" /etc/frr/vtysh.conf
          sudo grep "service integrated-vtysh-config" /etc/frr/vtysh.conf
service integrated-vtysh-config
```
```bash
etu@R1:~$ groups
etu adm sudo users frrvty frr
```
Activation des démons OSPF:
```bash
etu@R1:~$ systemctl status frr
● frr.service - FRRouting
     Loaded: loaded (/usr/lib/systemd/system/frr.service; enabled; preset: enab>
     Active: active (running) since Tue 2024-11-12 11:06:19 CET; 13s ago
 Invocation: 11827f9690a24c2bb107d359494a024f
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 2095 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=>
   Main PID: 2104 (watchfrr)
     Status: "FRR Operational"
      Tasks: 12 (limit: 1032)
     Memory: 23M (peak: 35.8M)
        CPU: 363ms
     CGroup: /system.slice/frr.service
             ├─2104 /usr/lib/frr/watchfrr -d -F traditional zebra mgmtd ospfd o>
             ├─2116 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 900000>
             ├─2121 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─2123 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             ├─2126 /usr/lib/frr/ospf6d -d -F traditional -A ::1
             └─2129 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

nov. 12 11:06:19 R1 frrinit.sh[2137]: [2137|ospf6d] done
nov. 12 11:06:19 R1 frrinit.sh[2149]: [2149|staticd] done
nov. 12 11:06:19 R1 watchfrr[2104]: [QDG3Y-BY5TN] zebra state -> up : connect s>
nov. 12 11:06:19 R1 watchfrr[2104]: [QDG3Y-BY5TN] mgmtd state -> up : connect s>
lines 1-23
nov. 12 11:06:19 R1 watchfrr[2104]: [QDG3Y-BY5TN] ospfd state -> up : connect s>
lines 2-24
nov. 12 11:06:19 R1 watchfrr[2104]: [QDG3Y-BY5TN] ospf6d state -> up : connect >
```
```bash
etu@R1:~$ vtysh
          vtysh

Hello, this is FRRouting (version 10.1.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

R1# sh daemon
 mgmtd zebra ospfd ospf6d watchfrr staticd
```

## 5. Valider les communications entre routeurs
```bash
etu@R1:~$ ip route ls proto kernel
10.0.50.168/29 dev enp0s1.306 scope link src 10.0.50.170 
10.7.12.0/29 dev enp0s1.518 scope link src 10.7.12.1 
10.7.13.0/29 dev enp0s1.519 scope link src 10.7.13.1 
etu@R1:~$ ip -6 route ls proto kernel
2001:678:3fc:132::/64 dev enp0s1.306 metric 256 pref medium
fe80::/64 dev enp0s1 metric 256 pref medium
fe80::/64 dev enp0s1.518 metric 256 pref medium
fe80::/64 dev enp0s1.519 metric 256 pref medium
fe80::/64 dev enp0s1.306 metric 256 pref medium
```
PING R1 -> R2
```bash
etu@R1:~$ ping -qc2 10.7.12.2
PING 10.7.12.2 (10.7.12.2) 56(84) bytes of data.

--- 10.7.12.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.452/0.741/1.030/0.289 ms
```
## 6. Configurer les démons OSPFv2 et OSPFv3
```bash
etu@R1:~$ vtysh

Hello, this is FRRouting (version 10.1.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

R1# show daemon
 mgmtd zebra ospfd ospf6d watchfrr staticd
R1# show ip ospf
R1# sh ipv6 ospf
% OSPFv3 instance not found
R1# 
```
```bash
etu@R1:~$ vtysh

Hello, this is FRRouting (version 10.1.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

R1# show daemon
 mgmtd zebra ospfd ospf6d watchfrr staticd
R1# show ip ospf
R1# sh ipv6 ospf
% OSPFv3 instance not found
R1# conf t
R1(config)# router ospf
R1(config-router)# ospf router-id 0.1.4.70
R1(config-router)# log detail
R1(config-router)# R1# 

R1# conf t
R1(config)# router ospf6
R1(config-ospf6)# ospf6 router-id 0.1.6.70
R1(config-ospf6)# log detail
R1(config-ospf6)# R1# 
R1# 
```
```bash
R1# sh run ospfd
Building configuration...

Current configuration:
!
frr version 10.1.1
frr defaults traditional
hostname R1
log syslog informational
service integrated-vtysh-config
!
router ospf
 ospf router-id 0.1.4.70
 log-adjacency-changes detail
exit
!
end
```
```bash
R1# sh run ospf6d
Building configuration...

Current configuration:
!
frr version 10.1.1
frr defaults traditional
hostname R1
log syslog informational
service integrated-vtysh-config
!
router ospf6
 ospf6 router-id 0.1.6.70
 log-adjacency-changes detail
exit
!
end
```
```bash
R1# sh ip ospf
 OSPF Routing Process, Router ID: 0.1.4.70
 Supports only single TOS (TOS0) routes
 This implementation conforms to RFC2328
 RFC1583Compatibility flag is disabled
 OpaqueCapability flag is disabled
 Initial SPF scheduling delay 0 millisec(s)
 Minimum hold time between consecutive SPFs 50 millisec(s)
 Maximum hold time between consecutive SPFs 5000 millisec(s)
 Hold time multiplier is currently 1
 SPF algorithm has not been run
 SPF timer is inactive
 LSA minimum interval 5000 msecs
 LSA minimum arrival 1000 msecs
 Write Multiplier set to 20 
 Refresh timer 10 secs
 Maximum multiple paths(ECMP) supported 256
 Administrative distance 110
 Number of external LSA 0. Checksum Sum 0x00000000
 Number of opaque AS LSA 0. Checksum Sum 0x00000000
 Number of areas attached to this router: 0
 All adjacency changes are logged
```
```bash
R1# sh ipv6 ospf
 OSPFv3 Routing Process (0) with Router-ID 0.1.6.70
 Running 00:05:53
 LSA minimum arrival 1000 msecs
 Maximum-paths 256
 Administrative distance 110
 Initial SPF scheduling delay 0 millisec(s)
 Minimum hold time between consecutive SPFs 50 millsecond(s)
 Maximum hold time between consecutive SPFs 5000 millsecond(s)
 Hold time multiplier is currently 1
 SPF algorithm has not been run
 SPF timer is inactive
 Number of AS scoped LSAs is 0
 Number of areas in this router is 0
 Authentication Sequence number info
  Higher sequence no 0, Lower sequence no 0
 All adjacency changes are logged
```
### Activer les protocoles de routage OSPFv2 et OSPFv3 pour les réseaux d'interconnexion de chaque routeur
```bash
R1# sh ip route connected
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

C>* 10.0.50.168/29 is directly connected, enp0s1.306, 00:09:07
C>* 10.7.10.0/24 is directly connected, vlan70, 00:09:07
C>* 10.7.12.0/29 is directly connected, enp0s1.518, 00:09:07
C>* 10.7.13.0/29 is directly connected, enp0s1.519, 00:09:07
```
```bash
R1# sh ipv6 route connected
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIPng, O - OSPFv3, I - IS-IS, B - BGP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

C>* 2001:678:3fc:132::/64 is directly connected, enp0s1.306, 00:10:17
C>* fda0:7a62:46::/64 is directly connected, vlan70, 00:10:16
C * fe80::/64 is directly connected, vlan70, 00:10:16
C * fe80::/64 is directly connected, asw-host, 00:10:17
C * fe80::/64 is directly connected, enp0s1.518, 00:10:17
C * fe80::/64 is directly connected, enp0s1.306, 00:10:17
C * fe80::/64 is directly connected, enp0s1, 00:10:17
C>* fe80::/64 is directly connected, enp0s1.519, 00:10:18
```
```bash
R1# conf t
R1(config)# int enp0s1.518
R1(config-if)# ip ospf area 0
R1(config-if)# ipv6 ospf6 area 0
R1(config-if)# int enp0s1.519
R1(config-if)# ip ospf area 0
R1(config-if)# ipv6 ospf6 area 0
R1(config-if)# R1# 
```
```bash
R1# sh ip ospf interface enp0s1.518
enp0s1.518 is up
  ifindex 3, MTU 1500 bytes, BW 0 Mbit <UP,LOWER_UP,BROADCAST,RUNNING,MULTICAST>
  Internet Address 10.7.12.1/29, Broadcast 10.7.12.7, Area 0.0.0.0
  MTU mismatch detection: enabled
  Router ID 0.1.4.70, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 0.1.4.70 Interface Address 10.7.12.1/29
  No backup designated router on this network
  Multicast group memberships: OSPFAllRouters OSPFDesignatedRouters
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
    Hello due in 9.201s
  Neighbor Count is 0, Adjacent neighbor count is 0
  Graceful Restart hello delay: 10s

R1# sh ip ospf interface enp0s1.519
enp0s1.519 is up
  ifindex 4, MTU 1500 bytes, BW 0 Mbit <UP,LOWER_UP,BROADCAST,RUNNING,MULTICAST>
  Internet Address 10.7.13.1/29, Broadcast 10.7.13.7, Area 0.0.0.0
  MTU mismatch detection: enabled
  Router ID 0.1.4.70, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 0.1.4.70 Interface Address 10.7.13.1/29
  No backup designated router on this network
  Multicast group memberships: OSPFAllRouters OSPFDesignatedRouters
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
    Hello due in 3.165s
  Neighbor Count is 0, Adjacent neighbor count is 0
  Graceful Restart hello delay: 10s

R1# sh ipv6 ospf6 interface enp0s1.518
enp0s1.518 is up, type BROADCAST
  Interface ID: 3
  Internet Address:
    inet : 10.7.12.1/29
    inet6: fe80::206:1/64
    inet6: fe80::baad:caff:fefe:13f/64
  Instance ID 0, Interface MTU 1500 (autodetect: 1500)
  MTU mismatch detection: enabled
  Area ID 0.0.0.0, Cost 10
  State DR, Transmit Delay 1 sec, Priority 1
  Timer intervals configured:
   Hello 10(8.533), Dead 40, Retransmit 5
  DR: 0.1.6.70 BDR: 0.0.0.0
  Number of I/F scoped LSAs is 1
    0 Pending LSAs for LSUpdate in Time 00:00:00 [thread off]
    0 Pending LSAs for LSAck in Time 00:00:00 [thread off]
  Graceful Restart hello delay: 10s
  Authentication Trailer is disabled
R1# sh ipv6 ospf6 interface enp0s1.519
enp0s1.519 is up, type BROADCAST
  Interface ID: 4
  Internet Address:
    inet : 10.7.13.1/29
    inet6: fe80::206:1/64
    inet6: fe80::baad:caff:fefe:13f/64
  Instance ID 0, Interface MTU 1500 (autodetect: 1500)
  MTU mismatch detection: enabled
  Area ID 0.0.0.0, Cost 10
  State DR, Transmit Delay 1 sec, Priority 1
  Timer intervals configured:
   Hello 10(5.855), Dead 40, Retransmit 5
  DR: 0.1.6.70 BDR: 0.0.0.0
  Number of I/F scoped LSAs is 1
    0 Pending LSAs for LSUpdate in Time 00:00:00 [thread off]
    0 Pending LSAs for LSAck in Time 00:00:00 [thread off]
  Graceful Restart hello delay: 10s
  Authentication Trailer is disabled
```
### Vérifier que l'identifiant de routeur a correctement été attribué
```bash
R1# sh ip ospf interface enp0s1.518
enp0s1.518 is up
  ifindex 4, MTU 1500 bytes, BW 0 Mbit <UP,LOWER_UP,BROADCAST,RUNNING,MULTICAST>
  Internet Address 10.7.12.1/29, Broadcast 10.7.12.7, Area 0.0.0.0
  MTU mismatch detection: enabled
  Router ID 0.1.4.70, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 0.1.4.70 Interface Address 10.7.12.1/29
  Backup Designated Router (ID) 0.2.4.70, Interface Address 10.7.12.2
  Saved Network-LSA sequence number 0x80000003
  Multicast group memberships: OSPFAllRouters OSPFDesignatedRouters
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
    Hello due in 0.740s
  Neighbor Count is 1, Adjacent neighbor count is 1
  Graceful Restart hello delay: 10s

R1# sh ipv6 ospf6 interface enp0s1.518
enp0s1.518 is up, type BROADCAST
  Interface ID: 4
  Internet Address:
    inet : 10.7.12.1/29
    inet6: fe80::206:1/64
    inet6: fe80::baad:caff:fefe:13f/64
  Instance ID 0, Interface MTU 1500 (autodetect: 1500)
  MTU mismatch detection: enabled
  Area ID 0.0.0.0, Cost 10
  State DR, Transmit Delay 1 sec, Priority 1
  Timer intervals configured:
   Hello 10(6.332), Dead 40, Retransmit 5
  DR: 0.1.6.70 BDR: 0.2.6.70
  Number of I/F scoped LSAs is 2
    0 Pending LSAs for LSUpdate in Time 00:00:00 [thread off]
    0 Pending LSAs for LSAck in Time 00:00:00 [thread off]
  Graceful Restart hello delay: 10s
  Authentication Trailer is disabled
R1# sh ip ospf interface enp0s1.519
enp0s1.519 is up
  ifindex 5, MTU 1500 bytes, BW 0 Mbit <UP,LOWER_UP,BROADCAST,RUNNING,MULTICAST>
  Internet Address 10.7.13.1/29, Broadcast 10.7.13.7, Area 0.0.0.0
  MTU mismatch detection: enabled
  Router ID 0.1.4.70, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 0.1.4.70 Interface Address 10.7.13.1/29
  No backup designated router on this network
  Multicast group memberships: OSPFAllRouters OSPFDesignatedRouters
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
    Hello due in 5.326s
  Neighbor Count is 0, Adjacent neighbor count is 0
  Graceful Restart hello delay: 10s

R1# sh ipv6 ospf6 interface enp0s1.519
enp0s1.519 is up, type BROADCAST
  Interface ID: 5
  Internet Address:
    inet : 10.7.13.1/29
    inet6: fe80::206:1/64
    inet6: fe80::baad:caff:fefe:13f/64
  Instance ID 0, Interface MTU 1500 (autodetect: 1500)
  MTU mismatch detection: enabled
  Area ID 0.0.0.0, Cost 10
  State DR, Transmit Delay 1 sec, Priority 1
  Timer intervals configured:
   Hello 10(9.091), Dead 40, Retransmit 5
  DR: 0.1.6.70 BDR: 0.0.0.0
  Number of I/F scoped LSAs is 1
    0 Pending LSAs for LSUpdate in Time 00:00:00 [thread off]
    0 Pending LSAs for LSAck in Time 00:00:00 [thread off]
  Graceful Restart hello delay: 10s
  Authentication Trailer is disabled
```
## 7. Publier les routes par défaut via OSPF

```bash
etu@R1:~$ ip route ls default
default via 10.0.50.169 dev enp0s1.306 proto static 
etu@R1:~$ ip -6 route ls default
default via fe80:132::1 dev enp0s1.306 proto static metric 1024 onlink pref medium
```
```bash
R1# sh ip route kernel
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

K>* 0.0.0.0/0 [0/0] via 10.0.50.169, enp0s1.306, 00:05:17
R1# sh ipv6 route kernel
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIPng, O - OSPFv3, I - IS-IS, B - BGP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

K>d ::/0 [0/1024] via fe80:132::1, enp0s1.306 onlink, 00:05:31
K d 2001:678:3fc:132::/64 [0/512] is directly connected, enp0s1.306, 00:05:31
```

Route par défaut
```bash
R1# conf t
R1(config)# router ospf
R1(config-router)# default-information originate
R1(config-router)# R1# 
R1# sh ip ospf database external

       OSPF Router with ID (0.1.4.70)

                AS External Link States 

  LS age: 20
  Options: 0x2  : *|-|-|-|-|-|E|-
  LS Flags: 0xb  
  LS Type: AS-external-LSA
  Link State ID: 0.0.0.0 (External Network Number)
  Advertising Router: 0.1.4.70
  LS Seq Number: 80000001
  Checksum: 0x7dff
  Length: 36

  Network Mask: /0
        Metric Type: 2 (Larger than any link state path)
        TOS: 0
        Metric: 10
        Forward Address: 0.0.0.0
        External Route Tag: 0

R1# sh ip ospf
 OSPF Routing Process, Router ID: 0.1.4.70
 Supports only single TOS (TOS0) routes
 This implementation conforms to RFC2328
 RFC1583Compatibility flag is disabled
 OpaqueCapability flag is disabled
 Initial SPF scheduling delay 0 millisec(s)
 Minimum hold time between consecutive SPFs 50 millisec(s)
 Maximum hold time between consecutive SPFs 5000 millisec(s)
 Hold time multiplier is currently 1
 SPF algorithm last executed 39.054s ago
 Last SPF duration 116 usecs
 SPF timer is inactive
 LSA minimum interval 5000 msecs
 LSA minimum arrival 1000 msecs
 Write Multiplier set to 20 
 Refresh timer 10 secs
 Maximum multiple paths(ECMP) supported 256
 Administrative distance 110
 This router is an ASBR (injecting external routing information)
[....]
```
Ospfv3:
```bash
R1# conf t
R1(config)# router ospf6
R1(config-ospf6)# default-information originate
R1(config-ospf6)# R1# 
R1# sh ipv6 ospf6 database as-external

        AS Scoped Link State Database

Type LSId           AdvRouter       Age   SeqNum                        Payload
ASE  0.0.0.1        0.1.6.70         22 80000001                             ::

R1# sh ipv6 ospf6
 OSPFv3 Routing Process (0) with Router-ID 0.1.6.70
 Running 00:49:42
 LSA minimum arrival 1000 msecs
 Maximum-paths 256
 Administrative distance 110
 Initial SPF scheduling delay 0 millisec(s)
 Minimum hold time between consecutive SPFs 50 millsecond(s)
 Maximum hold time between consecutive SPFs 5000 millsecond(s)
 Hold time multiplier is currently 1
 SPF algorithm last executed 00:00:33 ago, reason R+, R-, A
 Last SPF duration 0 sec 52 usec
 SPF timer is inactive
 Number of AS scoped LSAs is 1                  <-----
 Number of areas in this router is 1
 Authentication Sequence number info
  Higher sequence no 0, Lower sequence no 0
 All adjacency changes are logged

 Area 0
     Number of Area scoped LSAs is 1
     Interface attached to this area: enp0s1.518 enp0s1.519
     SPF last executed 33.588939s ago
```
### Assurer la traduction d'adresses source sur l'interface de sortie du routeur R1 vers l'Internet 
```bash
etu@R1:~$ sudo systemctl enable --now nftables.service
Created symlink '/etc/systemd/system/sysinit.target.wants/nftables.service' → '/usr/lib/systemd/system/nftables.service'.
[ 5286.124441] systemd-ssh-generator[2473]: Binding SSH to AF_VSOCK vsock::22.
[ 5286.125388] systemd-ssh-generator[2473]: → connect via 'ssh vsock/4294967295' from host
[ 5286.126415] systemd-ssh-generator[2473]: Binding SSH to AF_UNIX socket /run/ssh-unix-local/socket.
[ 5286.127256] systemd-ssh-generator[2473]: → connect via 'ssh .host' locally
etu@R1:~$ sudo nft list ruleset
          sudo nft list ruleset
table inet nat {
        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
                oifname "enp0s1.306" masquerade
        }
}
```
## 8. Ajouter un réseau d'hébergement à chaque routeur
```bash
etu@R1:~$ systemctl status dnsmasq
          systemctl status dnsmasq
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; preset: >
     Active: active (running) since Tue 2024-11-12 12:39:43 CET; 4s ago
 Invocation: 5f61f3c3050d43dfbacdb493d1cad6ee
    Process: 3834 ExecStartPre=/usr/share/dnsmasq/systemd-helper checkconfig (c>
    Process: 3840 ExecStart=/usr/share/dnsmasq/systemd-helper exec (code=exited>
    Process: 3846 ExecStartPost=/usr/share/dnsmasq/systemd-helper start-resolvc>
   Main PID: 3845 (dnsmasq)
      Tasks: 1 (limit: 1032)
     Memory: 672K (peak: 2.4M)
        CPU: 69ms
     CGroup: /system.slice/dnsmasq.service
             └─3845 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -r>

nov. 12 12:39:43 R1 dnsmasq[3845]: options à la compilation : IPv6 GNU-getopt D>
nov. 12 12:39:43 R1 dnsmasq-dhcp[3845]: DHCP, plage d'adresses IP 10.7.10.1 -- >
nov. 12 12:39:43 R1 dnsmasq-dhcp[3845]: noms IPv6 dérivés de DHCPv4 sur vlan70
nov. 12 12:39:43 R1 dnsmasq-dhcp[3845]: annonces de routeurs sur vlan70
nov. 12 12:39:43 R1 dnsmasq-dhcp[3845]: noms IPv6 dérivés de DHCPv4 sur fda0:7a>
nov. 12 12:39:43 R1 dnsmasq-dhcp[3845]: annonces de routeurs sur fda0:7a62:46::>
```
```bash
etu@R1:~$ incus ls                                 
+------+---------+-------------------+------------------------------------------+-----------+-----------+
| NAME |  STATE  |       IPV4        |                   IPV6                   |   TYPE    | SNAPSHOTS |
+------+---------+-------------------+------------------------------------------+-----------+-----------+
| c0   | RUNNING | 10.7.10.10 (eth0) | fda0:7a62:46:0:216:3eff:fe24:fc04 (eth0) | CONTAINER | 0         |
+------+---------+-------------------+------------------------------------------+-----------+-----------+
| c1   | RUNNING | 10.7.10.11 (eth0) | fda0:7a62:46:0:216:3eff:fea1:648f (eth0) | CONTAINER | 0         |
+------+---------+-------------------+------------------------------------------+-----------+-----------+
| c2   | RUNNING | 10.7.10.12 (eth0) | fda0:7a62:46:0:216:3eff:fee7:5404 (eth0) | CONTAINER | 0         |
+------+---------+-------------------+------------------------------------------+-----------+-----------+
```
### Vérifier la publication des réseaux de conteneurs dans l'aire OSPF
```bash
R1# sh ip ospf interface vlan70
vlan70 is up
  ifindex 8, MTU 1500 bytes, BW 0 Mbit <UP,LOWER_UP,BROADCAST,RUNNING,MULTICAST>
  Internet Address 10.7.10.1/24, Broadcast 10.7.10.255, Area 0.0.0.0
  MTU mismatch detection: enabled
  Router ID 0.1.4.70, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 0.1.4.70 Interface Address 10.7.10.1/24
  No backup designated router on this network
  Multicast group memberships: <None>
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
    No Hellos (Passive interface)
  Neighbor Count is 0, Adjacent neighbor count is 0
  Graceful Restart hello delay: 10s

R1# sh ipv6 ospf6 interface vlan70
vlan70 is up, type BROADCAST
  Interface ID: 8
  Internet Address:
    inet : 10.7.10.1/24
    inet6: fe80::b037:67ff:fe0b:9146/64
    inet6: fda0:7a62:46::1/64
  Instance ID 0, Interface MTU 1500 (autodetect: 1500)
  MTU mismatch detection: enabled
  Area ID 0.0.0.0, Cost 10
  State DR, Transmit Delay 1 sec, Priority 1
  Timer intervals configured:
   No Hellos (Passive interface)
  DR: 0.1.6.70 BDR: 0.0.0.0
  Number of I/F scoped LSAs is 1
    0 Pending LSAs for LSUpdate in Time 00:00:00 [thread off]
    0 Pending LSAs for LSAck in Time 00:00:00 [thread off]
  Graceful Restart hello delay: 10s
  Authentication Trailer is disabled
```
### Accès à Internet depuis les conteneurs 
```bash
etu@R1:~$ for i in {0..2}
do        for i in {0..2}
do  echo ">>>>>>>>>>>>>>>>> c$i"
    echo ">>>>>>>>>>>>>>>>> c$i"9.9.9.9
    incus exec c$i -- ping -qc2 9.9.9.9
done
>>>>>>>>>>>>>>>>> c0
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 28.160/36.941/45.723/8.781 ms
>>>>>>>>>>>>>>>>> c1
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 28.169/28.987/29.806/0.818 ms
>>>>>>>>>>>>>>>>> c2
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 28.028/28.987/29.946/0.959 ms
etu@R1:~$ for i in {0..2}
do        for i in {0..2}
do  echo ">>>>>>>>>>>>>>>>> c$i"
    echo ">>>>>>>>>>>>>>>>> c$i"2620:fe::fe
    incus exec c$i -- ping -qc2 2620:fe::fe
done
>>>>>>>>>>>>>>>>> c0
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 39.280/40.258/41.237/0.978 ms
>>>>>>>>>>>>>>>>> c1
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 39.621/39.996/40.371/0.375 ms
>>>>>>>>>>>>>>>>> c2
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 39.975/39.988/40.002/0.013 ms
etu@R1:~$ for i in {0..2}
do        for i in {0..2}
do  echo ">>>>>>>>>>>>>>>>> c$i"
    echo ">>>>>>>>>>>>>>>>> c$i"
    incus exec c$i -- apt updatel-upgrade
    incus exec c$i -- apt -y full-upgrade
done
>>>>>>>>>>>>>>>>> c0
Get:1 http://deb.debian.org/debian trixie InRelease [172 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [49.6 kB]
Hit:3 http://deb.debian.org/debian-security trixie-security InRelease
Fetched 221 kB in 1s (237 kB/s)
All packages are up to date.    
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c1
Get:1 http://deb.debian.org/debian trixie InRelease [172 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [49.6 kB]
Hit:3 http://deb.debian.org/debian-security trixie-security InRelease
Fetched 221 kB in 1s (264 kB/s)
All packages are up to date.    
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c2
Get:1 http://deb.debian.org/debian trixie InRelease [172 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [49.6 kB]
Hit:3 http://deb.debian.org/debian-security trixie-security InRelease
Fetched 221 kB in 1s (303 kB/s)
All packages are up to date.    
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
```
