# Interconnexion de réseaux locaux & étendus
## TP2 PPPoe
### Manipulation VM Spoke



```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP2$ sudo ovs-vsctl get port tap310 vlan_mode
trunk
```

```yaml
ovs:
  switches:
    - name: dsw-host
      ports:
        - name: tap319 # Spoke port
          type: OVSPort
          vlan_mode: trunk
          trunks: [60, 420, 421] # Avec VLAN d'accès temporaire
#         trunks: [420, 421] # Sans VLAN d'accès temporaire
```

```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP2$ $HOME/masters/scripts/switch-conf.py switch.yaml
----------------------------------------
Configuring switch dsw-host
>> Port tap310 vlan_mode is already set to trunk
>> Port tap310 trunks set to [60, 420, 421]
```
```yaml
  GNU nano 8.1                                                                     lab.yaml *                                                                            
kvm:
  vms:
    - vm_name: spoke
      master_image: debian-testing-amd64.qcow2 # master image to be used
      force_copy: false # do not force copy the master image to the VM image
      memory: 1024
      tapnum: 319
```

```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP2$ $HOME/masters/scripts/lab-startup.py lab.yaml
Copying /home/valerianpacaud/masters/debian-testing-amd64.qcow2 to spoke.qcow2...done
Creating OVMF_CODE.fd symlink...
Creating spoke_OVMF_VARS.fd file...
Starting spoke...
~> Virtual machine filename   : spoke.qcow2
~> RAM size                   : 1024MB
~> SPICE VDI port number      : 6219
~> telnet console port number : 2619
~> MAC address                : b8:ad:ca:fe:01:3f
~> Switch port interface      : tap319, trunk mode
~> IPv6 LL address            : fe80::baad:caff:fefe:13f%dsw-host
spoke started!
```
## 6. Routeur Spoke
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
    enp0s1.420: # VLAN violet
      id: 420
      link: enp0s1
      addresses:
        - fe80:1A4::2/64 #Belek
    enp0s1.421: # VLAN orange
      id: 421
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
etu@spoke:~$ sudo netplan apply
[sudo] Mot de passe de etu : 

** (generate:525): WARNING **: 19:46:45.547: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:524): WARNING **: 19:46:45.857: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:524): WARNING **: 19:46:45.963: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.
etu@spoke:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether b8:ad:ca:fe:01:3f brd ff:ff:ff:ff:ff:ff
    inet6 fe80::baad:caff:fefe:13f/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
3: enp0s1.60@enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether b8:ad:ca:fe:01:3f brd ff:ff:ff:ff:ff:ff
    inet 198.18.60.122/23 metric 100 brd 198.18.61.255 scope global dynamic enp0s1.60
       valid_lft 86379sec preferred_lft 86379sec
    inet6 2001:678:3fc:3c:baad:caff:fefe:13f/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 86380sec preferred_lft 14380sec
    inet6 fe80::baad:caff:fefe:13f/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
4: enp0s1.421@enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether b8:ad:ca:fe:01:3f brd ff:ff:ff:ff:ff:ff
    inet6 fe80::baad:caff:fefe:13f/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
5: enp0s1.420@enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether b8:ad:ca:fe:01:3f brd ff:ff:ff:ff:ff:ff
    inet6 fe80:1a4::2/64 scope link 
       valid_lft forever preferred_lft forever
    inet6 fe80::baad:caff:fefe:13f/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
etu@spoke:~$ ping 9.9.9.9
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.
64 bytes from 9.9.9.9: icmp_seq=1 ttl=47 time=30.9 ms
64 bytes from 9.9.9.9: icmp_seq=2 ttl=47 time=28.6 ms
[   39.678271] systemd-journald[293]: Time jumped backwards, rotating.
64 bytes from 9.9.9.9: icmp_seq=3 ttl=47 time=28.3 ms
64 bytes from 9.9.9.9: icmp_seq=4 ttl=47 time=28.1 ms

--- 9.9.9.9 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 28.114/28.980/30.944/1.145 ms
```

### 6.2. Activation de la fonction routage
```bash
etu@spoke:~$ sudo sysctl --system
* Applique /usr/lib/sysctl.d/10-coredump-debian.conf …
* Applique /etc/sysctl.d/10-routing.conf …
* Applique /usr/lib/sysctl.d/50-default.conf …
* Applique /usr/lib/sysctl.d/50-pid-max.conf …
* Applique /etc/sysctl.conf …
kernel.core_pattern = core
net.ipv4.ip_forward = 1              <-------
net.ipv6.conf.all.forwarding = 1     <-------
net.ipv4.conf.all.log_martians = 1   <-------
kernel.sysrq = 0x01b6
kernel.core_uses_pid = 1
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.enp0s1/20.rp_filter = 2
net.ipv4.conf.enp0s1/420.rp_filter = 2
net.ipv4.conf.enp0s1/421.rp_filter = 2
net.ipv4.conf.enp0s1.rp_filter = 2
net.ipv4.conf.lo.rp_filter = 2
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.enp0s1/20.accept_source_route = 0
net.ipv4.conf.enp0s1/420.accept_source_route = 0
net.ipv4.conf.enp0s1/421.accept_source_route = 0
net.ipv4.conf.enp0s1.accept_source_route = 0
net.ipv4.conf.lo.accept_source_route = 0
net.ipv4.conf.default.promote_secondaries = 1
net.ipv4.conf.enp0s1/20.promote_secondaries = 1
net.ipv4.conf.enp0s1/420.promote_secondaries = 1
net.ipv4.conf.enp0s1/421.promote_secondaries = 1
net.ipv4.conf.enp0s1.promote_secondaries = 1
net.ipv4.conf.lo.promote_secondaries = 1
net.ipv4.ping_group_range = 0 2147483647
net.core.default_qdisc = fq_codel
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.protected_regular = 2
fs.protected_fifos = 1
kernel.pid_max = 4194304
```
### 6.3. Activation du protocole PPP
```bash
etu@spoke:~$ sudo apt list ppp
ppp/testing,now 2.5.0-1+2 amd64  [installé]
```
```bash
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
"spoke_s11"     *               "5p0k3.11"              *
```
```bash
cat << EOF | sudo tee /etc/systemd/system/ppp.service
[Unit]
Description=PPPoE Client Connection
After=network.target
Wants=network.target
BindsTo=sys-subsystem-net-devices-enp0s1.421.device
After=sys-subsystem-net-devices-enp0s1.421.device

[Service]
Type=forking
ExecStart=/usr/bin/pon pppoe-provider
ExecStop=/usr/bin/poff pppoe-provider
Restart=on-failure
RestartSec=20

[Install]
WantedBy=multi-user.target
EOF
```

```bash
etu@spoke:~$ sudo systemctl daemon-reload
```
```bash
etu@spoke:~$ systemctl status ppp.service
● ppp.service - PPPoE Client Connection
     Loaded: loaded (/etc/systemd/system/ppp.service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-09-24 18:11:41 CEST; 13min ago
 Invocation: 42e288c0e3034c8e9589fa6953e12bf3
   Main PID: 906 (pppd)
      Tasks: 1 (limit: 1086)
     Memory: 1.2M (peak: 3.3M)
        CPU: 65ms
     CGroup: /system.slice/ppp.service
             └─906 /usr/sbin/pppd call pppoe-provider
```
```bash
etu@spoke:~$ journalctl -n 100 -f -u ppp.service
sept. 24 18:17:41 spoke pppd[906]: Unable to complete PPPoE Discovery phase 1
sept. 24 18:18:11 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:18:11 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:18:11 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:18:16 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:18:16 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:18:16 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:18:26 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:18:26 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:18:26 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:18:46 spoke pppd[906]: Timeout waiting for PADO packets
sept. 24 18:18:46 spoke pppd[906]: Unable to complete PPPoE Discovery phase 1
sept. 24 18:19:16 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:19:16 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:19:16 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:19:21 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:19:21 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:19:21 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:19:31 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:19:31 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:19:31 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:19:51 spoke pppd[906]: Timeout waiting for PADO packets
sept. 24 18:19:51 spoke pppd[906]: Unable to complete PPPoE Discovery phase 1
sept. 24 18:20:22 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:20:22 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:20:22 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:20:27 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:20:27 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:20:27 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:20:37 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:20:37 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:20:37 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:20:57 spoke pppd[906]: Timeout waiting for PADO packets
sept. 24 18:20:57 spoke pppd[906]: Unable to complete PPPoE Discovery phase 1
sept. 24 18:21:27 spoke pppd[906]: Send PPPOE Discovery V1T1 PADI session 0x0 length 12
sept. 24 18:21:27 spoke pppd[906]:  dst ff:ff:ff:ff:ff:ff  src b8:ad:ca:fe:01:3f
sept. 24 18:21:27 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:21:27 spoke pppd[906]: Recv PPPOE Discovery V1T1 PADO session 0x0 length 44
sept. 24 18:21:27 spoke pppd[906]:  dst b8:ad:ca:fe:01:3f  src b8:ad:ca:fe:01:36
sept. 24 18:21:27 spoke pppd[906]:  [AC-name BRAS] [service-name] [AC-cookie f7 a8 2a f3 99 e7 63 33 e1 81 4e 77 a4 de 52 41 73 03 00 00] [host-uniq 8a 03 00 00]
sept. 24 18:21:27 spoke pppd[906]: Access-Concentrator: BRAS
sept. 24 18:21:27 spoke pppd[906]: Cookie: f7 a8 2a f3 99 e7 63 33 e1 81 4e 77 a4 de 52 41 73 03 00 00
sept. 24 18:21:27 spoke pppd[906]: AC-Ethernet-Address: b8:ad:ca:fe:01:36
sept. 24 18:21:27 spoke pppd[906]: --------------------------------------------------
sept. 24 18:21:32 spoke pppd[906]: Send PPPOE Discovery V1T1 PADR session 0x0 length 36
sept. 24 18:21:32 spoke pppd[906]:  dst b8:ad:ca:fe:01:36  src b8:ad:ca:fe:01:3f
sept. 24 18:21:32 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00] [AC-cookie f7 a8 2a f3 99 e7 63 33 e1 81 4e 77 a4 de 52 41 73 03 00 00]
sept. 24 18:21:32 spoke pppd[906]: Recv PPPOE Discovery V1T1 PADS session 0x1 length 12
sept. 24 18:21:32 spoke pppd[906]:  dst b8:ad:ca:fe:01:3f  src b8:ad:ca:fe:01:36
sept. 24 18:21:32 spoke pppd[906]:  [service-name] [host-uniq 8a 03 00 00]
sept. 24 18:21:32 spoke pppd[906]: PPP session is 1
sept. 24 18:21:32 spoke pppd[906]: Connected to B8:AD:CA:FE:01:36 via interface enp0s1.421
sept. 24 18:21:32 spoke pppd[906]: using channel 1
sept. 24 18:21:32 spoke pppd[906]: Using interface ppp0
sept. 24 18:21:32 spoke pppd[906]: Connect: ppp0 <--> enp0s1.421
sept. 24 18:21:32 spoke pppd[906]: sent [LCP ConfReq id=0x1 <mru 1492> <magic 0xe6fd8e8b>]
sept. 24 18:21:33 spoke pppd[906]: rcvd [LCP ConfReq id=0x1 <mru 1492> <auth eap> <magic 0xad02d49e>]
sept. 24 18:21:33 spoke pppd[906]: sent [LCP ConfAck id=0x1 <mru 1492> <auth eap> <magic 0xad02d49e>]
sept. 24 18:21:35 spoke pppd[906]: sent [LCP ConfReq id=0x1 <mru 1492> <magic 0xe6fd8e8b>]
sept. 24 18:21:35 spoke pppd[906]: rcvd [LCP ConfAck id=0x1 <mru 1492> <magic 0xe6fd8e8b>]
sept. 24 18:21:35 spoke pppd[906]: sent [LCP EchoReq id=0x0 magic=0xe6fd8e8b]
sept. 24 18:21:35 spoke pppd[906]: rcvd [LCP EchoReq id=0x0 magic=0xad02d49e]
sept. 24 18:21:35 spoke pppd[906]: sent [LCP EchoRep id=0x0 magic=0xe6fd8e8b]
sept. 24 18:21:35 spoke pppd[906]: rcvd [EAP Request id=0x45 Identity <Message "Name">]
sept. 24 18:21:35 spoke pppd[906]: EAP: Identity prompt "Name"
sept. 24 18:21:35 spoke pppd[906]: sent [EAP Response id=0x45 Identity <Name "spoke_s11">]
sept. 24 18:21:35 spoke pppd[906]: rcvd [LCP EchoRep id=0x0 magic=0xad02d49e]
sept. 24 18:21:35 spoke pppd[906]: rcvd [EAP Request id=0x46 MD5-Challenge <Value 4f ed fa 69 33 26 14 a4 22 ba 54 b2 da 4a c2 d7 2e 36 2b> <Name "hub">]
sept. 24 18:21:35 spoke pppd[906]: sent [EAP Response id=0x46 MD5-Challenge <Value 42 02 df 7b d3 88 4b c8 dd f2 f1 1b 2a 1b 49 d9> <Name "spoke_s11">]
sept. 24 18:21:35 spoke pppd[906]: rcvd [EAP Success id=0x47]
sept. 24 18:21:35 spoke pppd[906]: EAP authentication succeeded
sept. 24 18:21:35 spoke pppd[906]: peer from calling number B8:AD:CA:FE:01:36 authorized
sept. 24 18:21:35 spoke pppd[906]: sent [IPCP ConfReq id=0x1 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
sept. 24 18:21:35 spoke pppd[906]: sent [IPV6CP ConfReq id=0x1 <addr fe80::54ff:0d5f:f6bf:e83f>]
sept. 24 18:21:35 spoke pppd[906]: rcvd [CCP ConfReq id=0x1 <bsd v1 15>]
sept. 24 18:21:35 spoke pppd[906]: sent [CCP ConfReq id=0x1]
sept. 24 18:21:35 spoke pppd[906]: sent [CCP ConfRej id=0x1 <bsd v1 15>]
sept. 24 18:21:35 spoke pppd[906]: rcvd [IPCP ConfReq id=0x1 <addr 10.4.11.1>]
sept. 24 18:21:35 spoke pppd[906]: sent [IPCP ConfAck id=0x1 <addr 10.4.11.1>]
sept. 24 18:21:35 spoke pppd[906]: rcvd [IPV6CP ConfReq id=0x1 <addr fe80::01da:652d:9739:b8b7>]
sept. 24 18:21:35 spoke pppd[906]: sent [IPV6CP ConfAck id=0x1 <addr fe80::01da:652d:9739:b8b7>]
sept. 24 18:21:35 spoke pppd[906]: rcvd [IPCP ConfNak id=0x1 <addr 10.4.11.2> <ms-dns1 172.16.0.2> <ms-dns2 172.16.0.2>]
sept. 24 18:21:35 spoke pppd[906]: sent [IPCP ConfReq id=0x2 <addr 10.4.11.2> <ms-dns1 172.16.0.2> <ms-dns2 172.16.0.2>]
sept. 24 18:21:35 spoke pppd[906]: rcvd [IPV6CP ConfAck id=0x1 <addr fe80::54ff:0d5f:f6bf:e83f>]
sept. 24 18:21:35 spoke pppd[906]: local  LL address fe80::54ff:0d5f:f6bf:e83f
sept. 24 18:21:35 spoke pppd[906]: remote LL address fe80::01da:652d:9739:b8b7
sept. 24 18:21:35 spoke pppd[906]: Script /etc/ppp/ipv6-up started (pid 918)
sept. 24 18:21:35 spoke pppd[906]: rcvd [CCP ConfAck id=0x1]
sept. 24 18:21:35 spoke pppd[906]: rcvd [CCP ConfReq id=0x2]
sept. 24 18:21:35 spoke pppd[906]: sent [CCP ConfAck id=0x2]
sept. 24 18:21:35 spoke pppd[906]: rcvd [IPCP ConfAck id=0x2 <addr 10.4.11.2> <ms-dns1 172.16.0.2> <ms-dns2 172.16.0.2>]
sept. 24 18:21:35 spoke pppd[906]: Script /etc/ppp/ip-pre-up started (pid 919)
sept. 24 18:21:35 spoke pppd[906]: Script /etc/ppp/ip-pre-up finished (pid 919), status = 0x0
sept. 24 18:21:35 spoke pppd[906]: local  IP address 10.4.11.2
sept. 24 18:21:35 spoke pppd[906]: remote IP address 10.4.11.1
sept. 24 18:21:35 spoke pppd[906]: primary   DNS address 172.16.0.2
sept. 24 18:21:35 spoke pppd[906]: secondary DNS address 172.16.0.2
sept. 24 18:21:35 spoke pppd[906]: Script /etc/ppp/ip-up started (pid 923)
sept. 24 18:21:35 spoke pppd[906]: Script /etc/ppp/ipv6-up finished (pid 918), status = 0x0
sept. 24 18:21:35 spoke pppd[906]: Script /etc/ppp/ip-up finished (pid 923), status = 0x0
```
```bash
etu@spoke:~$ sudo netplan status
Unknown device type: ppp
Unknown device type: ppp
Unknown device type: ppp
Unknown device type: ppp
Unknown device type: ppp
     Online state: offline
       DNS Search: .

●  1: lo ethernet UNKNOWN/UP (unmanaged)
      MAC Address: 00:00:00:00:00:00
        Addresses: 127.0.0.1/8
                   ::1/128

●  2: enp0s1 ethernet UP (networkd: enp0s1)
      MAC Address: b8:ad:ca:fe:01:3f (Red Hat, Inc.)
        Addresses: fe80::baad:caff:fefe:13f/64 (link)
           Routes: fe80::/64 metric 256

●  3: enp0s1.60 vlan UP (unmanaged)
      MAC Address: b8:ad:ca:fe:01:3f
        Addresses: fe80::baad:caff:fefe:13f/64 (link)
           Routes: fe80::/64 metric 256

●  4: enp0s1.421 vlan UP (networkd: enp0s1.421)
      MAC Address: b8:ad:ca:fe:01:3f
        Addresses: fe80::baad:caff:fefe:13f/64 (link)
           Routes: fe80::/64 metric 256

●  5: enp0s1.420 vlan UP (networkd: enp0s1.420)
      MAC Address: b8:ad:ca:fe:01:3f
        Addresses: fe80:1a4::2/64 (link)
                   fe80::baad:caff:fefe:13f/64 (link)
           Routes: fe80::/64 metric 256
                   fe80:1a4::/64 metric 256

●  6: ppp0 other UNKNOWN/UP (unmanaged)
        Addresses: 10.4.11.2/32
                   fe80::54ff:d5f:f6bf:e83f/128 (link)
           Routes: default (boot, link)
                   10.4.11.1 from 10.4.11.2 (link)
                   fe80::1da:652d:9739:b8b7 metric 256
                   fe80::54ff:d5f:f6bf:e83f metric 256
```
test de connecivité après redémarrage:

```bash
Debian GNU/Linux trixie/sid spoke ttyS0

spoke login: etu
Mot de passe : 
Linux spoke 6.10.9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.10.9-1 (2024-09-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
etu@spoke:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether b8:ad:ca:fe:01:3f brd ff:ff:ff:ff:ff:ff
    inet6 fe80::baad:caff:fefe:13f/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
3: enp0s1.421@enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether b8:ad:ca:fe:01:3f brd ff:ff:ff:ff:ff:ff
    inet6 fe80::baad:caff:fefe:13f/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
4: enp0s1.420@enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether b8:ad:ca:fe:01:3f brd ff:ff:ff:ff:ff:ff
    inet6 fe80:1a4::2/64 scope link 
       valid_lft forever preferred_lft forever
    inet6 fe80::baad:caff:fefe:13f/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
5: ppp0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1492 qdisc fq_codel state UNKNOWN group default qlen 3
    link/ppp 
    inet 10.4.11.2 peer 10.4.11.1/32 scope global ppp0
       valid_lft forever preferred_lft forever
    inet6 fe80::2833:f814:3a49:a1be peer fe80::2c80:80a1:3a6b:122e/128 scope link nodad 
       valid_lft forever preferred_lft forever
etu@spoke:~$ ping 9.9.9.9
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.
64 bytes from 9.9.9.9: icmp_seq=1 ttl=46 time=46.5 ms
64 bytes from 9.9.9.9: icmp_seq=2 ttl=46 time=29.5 ms
64 bytes from 9.9.9.9: icmp_seq=3 ttl=46 time=29.4 ms
64 bytes from 9.9.9.9: icmp_seq=4 ttl=46 time=28.9 ms

--- 9.9.9.9 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 28.873/33.563/46.468/7.454 ms
etu@spoke:~$ host fcebook.com
fcebook.com has address 185.60.219.3
fcebook.com has IPv6 address 2a03:2880:f08e:4:face:b00c:0:2
etu@spoke:~$ host facebook.com
facebook.com has address 185.60.219.35
facebook.com has IPv6 address 2a03:2880:f17b:88:face:b00c:0:25de
facebook.com mail is handled by 10 smtpin.vvv.facebook.com.
```

## 7. Réseau d'hébergement de conteneurs

```bash
etu@spoke:~$ ip route ls
default dev ppp0 scope link 
10.4.11.1 dev ppp0 proto kernel scope link src 10.4.11.2 

etu@spoke:~$ ip -6 route ls
fe80::/64 dev enp0s1 proto kernel metric 256 pref medium
fe80::/64 dev enp0s1.421 proto kernel metric 256 pref medium
fe80::/64 dev enp0s1.420 proto kernel metric 256 pref medium
fe80:1a4::/64 dev enp0s1.420 proto kernel metric 256 pref medium
```
### 7.1. Ajout du commutateur virtuel

```bash
etu@spoke:~$ sudo apt list openvswitch-switch
[sudo] Mot de passe de etu : 
openvswitch-switch/testing,now 3.4.0-1 amd64  [installé]
```

Modification du fichier de configuration réseau:

```yaml
network:
  version: 2
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
    enp0s1.420: # VLAN violet
      id: 420
      link: enp0s1
      addresses:
        - fe80:1A4::2/64
    enp0s1.421: # VLAN orange
      id: 421
      link: enp0s1
      addresses: []

```
```bash
etu@spoke:~$ sudo netplan apply

** (generate:1125): WARNING **: 19:29:50.797: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:1124): WARNING **: 19:29:51.144: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:1124): WARNING **: 19:29:51.199: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:1124): WARNING **: 19:29:51.304: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.
[ 3080.943677] ovs-system: entered promiscuous mode
[ 3080.946856] Timeout policy base is empty
[ 3080.976406] asw-host: entered promiscuous mode
etu@spoke:~$ sudo ovs-vsctl show
7848fd5d-d694-41dd-ba77-71237711400b
    Bridge asw-host
        fail_mode: standalone
        Port asw-host
            Interface asw-host
                type: internal
    ovs_version: "3.4.0"
```

Ajout d'une interface:

```yaml
network:
  version: 2
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
    enp0s1.420: # VLAN violet
      id: 420
      link: enp0s1
      addresses:
        - fe80:1A4::2/64
    enp0s1.421: # VLAN orange
      id: 421
      link: enp0s1
      addresses: []
  vlan20:     # VLAN vert
      id: 20
      link: asw-host
      addresses:
        - 100.64.11.1/24
        - fda0:7a62:14::1/64
        - fda0:14::1/64
```
```bash
etu@spoke:~$ sudo netplan apply

** (generate:1272): WARNING **: 19:41:52.387: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:1271): WARNING **: 19:41:52.726: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:1271): WARNING **: 19:41:52.838: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:1271): WARNING **: 19:41:52.962: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.
[ 3802.918164] vlan20: entered promiscuous mode
```
### 7.2. Routage du réseau d'hébergement

```bash
etu@spoke:~$ ip route ls default
default dev ppp0 scope link 

etu@spoke:~$ ip -6 route ls default
default dev ppp0 metric 1024 pref medium

etu@spoke:~$ ping -c2 2620:fe::fe
PING 2620:fe::fe (2620:fe::fe) 56 data bytes
64 bytes from 2620:fe::fe: icmp_seq=1 ttl=58 time=43.3 ms
64 bytes from 2620:fe::fe: icmp_seq=2 ttl=58 time=40.2 ms

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 40.163/41.719/43.276/1.556 ms
```

### 7.3. Adressage automatique dans le réseau d'hébergement

```conf
cat << EOF | sudo tee /etc/dnsmasq.conf
# Specify Container VLAN interface
interface=vlan20

# Enable DHCPv4 on Container VLAN
dhcp-range=100.64.11.20,100.64.11.200,3h

# Enable IPv6 router advertisements
enable-ra

# Enable SLAAC
dhcp-range=::,constructor:vlan40,ra-names,slaac

# Optional: Specify DNS servers
dhcp-option=option:dns-server,172.16.0.2,9.9.9.9
dhcp-option=option6:dns-server,[2001:678:3fc:3::2],[260:fe::fe]

# Avoid DNS listen port conflict between dnsmasq and systemd-resolved
bind-interfaces
listen-address=100.64.11.1
EOF
```

```bash
etu@spoke:~$ sudo systemctl restart dnsmasq
etu@spoke:~$ systemctl status dnsmasq
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; preset: >
     Active: active (running) since Wed 2024-09-25 08:32:39 CEST; 2s ago
 Invocation: eaba577d2a184a4fa0c02ad4f6858de1
    Process: 3482 ExecStartPre=/usr/share/dnsmasq/systemd-helper checkconfig (c>
    Process: 3487 ExecStart=/usr/share/dnsmasq/systemd-helper exec (code=exited>
    Process: 3495 ExecStartPost=/usr/share/dnsmasq/systemd-helper start-resolvc>
   Main PID: 3494 (dnsmasq)
      Tasks: 1 (limit: 1086)
     Memory: 720K (peak: 2.7M)
        CPU: 94ms
     CGroup: /system.slice/dnsmasq.service
             └─3494 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -r>
```
## 8. Conteneurs système Incus

### 8.1. Installation du gestionnaire de conteneurs Incus



```bash
etu@spoke:~$ sudo apt -y install incus
Installing:                                     
  incus

Installing dependencies:
  attr          libcowsql0     liblzo2-2  lxcfs           uidmap
  incus-agent   liblxc-common  libraft0   rsync
  incus-client  liblxc1t64     libsubid5  squashfs-tools
[.....]

etu@spoke:~$ groups
etu adm sudo users incus-admin incus
```
### 8.2. Configuration et lancement des conteneurs

```bash
etu@spoke:~$ incus admin init
[50322.551925] audit: type=1400 audit(1727246233.421:13): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lxc-container-default" pid=4067 comm="apparmor_parser"
[50322.558487] audit: type=1400 audit(1727246233.421:14): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lxc-container-default-cgns" pid=4067 comm="apparmor_parser"
[50322.565214] audit: type=1400 audit(1727246233.425:15): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lxc-container-default-with-mounting" pid=4067 comm="apparmor_parser"
[50322.569052] audit: type=1400 audit(1727246233.425:16): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lxc-container-default-with-nesting" pid=4067 comm="apparmor_parser"
Would you like to use clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Where should this storage pool store its data? [default=/var/lib/incus/storage-pools/default]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: no
Would you like to use an existing bridge or host interface? (yes/no) [default=no]: yes
Name of the existing bridge or host interface: asw-host
Would you like the server to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes]: 
Would you like a YAML "init" preseed to be printed? (yes/no) [default=no]: 
```

```bash
etu@spoke:~$ incus profile show default
config: {}
description: Default Incus profile
devices:
  eth0:
    name: eth0
    nictype: macvlan
    parent: asw-host
    type: nic
  root:
    path: /
    pool: default
    type: disk
name: default
used_by: []
project: default
```

```bash
etu@spoke:~$ incus profile device set default eth0 nictype bridged
etu@spoke:~$ incus profile device get default eth0 nictype
bridged
```

```bash
etu@spoke:~$ incus profile device set default eth0 vlan 20
etu@spoke:~$ incus profile device get default eth0 vlan
20
```

Lancement des conteneurs:

```bash
etu@spoke:~$ for i in {0..2}; do incus launch images:debian/trixie c$i; done
Launching c0
Retrieving image: rootfs: 100% (50.58MB/s)[50562.247980] audit: type=1400 audit(1727246473.116:17): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-96cbf509-044e-4f7d-bae3-7acd68c94966" pid=4156 comm="apparmor_parser"
Retrieving image: Unpacking image: 100% (1.51GB/s)[50562.297555] audit: type=1400 audit(1727246473.164:18): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-96cbf509-044e-4f7d-bae3-7acd68c94966" pid=4162 comm="apparmor_parser"
[50562.335728] audit: type=1400 audit(1727246473.204:19): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-71b7621f-e7ac-4bac-b0df-cedf0f2c7700" pid=4164 comm="apparmor_parser"
[50567.125007] audit: type=1400 audit(1727246477.992:20): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-71b7621f-e7ac-4bac-b0df-cedf0f2c7700" pid=4173 comm="apparmor_parser"
[50567.215675] vetha5ac428f: entered promiscuous mode
[50567.318829] audit: type=1400 audit(1727246478.188:21): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus-c0_</var/lib/incus>" pid=4207 comm="apparmor_parser"
[50567.442417] physYTC3r9: renamed from veth37b47d39
[50567.444806] eth0: renamed from physYTC3r9
[50567.498172] Not activating Mandatory Access Control as /sbin/tomoyo-init does not exist.
Launching c1                                       
[50567.795261] audit: type=1400 audit(1727246478.664:22): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-c9700f46-82cf-4dee-b4a5-ed5209aac992" pid=4336 comm="apparmor_parser"
Retrieving image: Unpacking image: 100% (1.64GB/s)[50567.832015] audit: type=1400 audit(1727246478.700:23): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-c9700f46-82cf-4dee-b4a5-ed5209aac992" pid=4343 comm="apparmor_parser"
[50567.864298] audit: type=1400 audit(1727246478.732:24): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-187d3e8f-5084-47dd-aedc-78b329651636" pid=4357 comm="apparmor_parser"
[50572.212780] audit: type=1400 audit(1727246483.080:25): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-187d3e8f-5084-47dd-aedc-78b329651636" pid=4421 comm="apparmor_parser"
[50572.265080] veth737352e8: entered promiscuous mode
[50572.366980] audit: type=1400 audit(1727246483.236:26): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus-c1_</var/lib/incus>" pid=4454 comm="apparmor_parser"
[50572.482208] physcVDaER: renamed from veth8adb054a
[50572.485405] eth0: renamed from physcVDaER
[50572.553802] Not activating Mandatory Access Control as /sbin/tomoyo-init does not exist.
Launching c2                                       
[50573.144666] audit: type=1400 audit(1727246484.012:27): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-a5a75691-85f9-42f5-97af-74b8fd704f7b" pid=4494 comm="apparmor_parser"
Retrieving image: Unpacking image: 100% (1.02GB/s)[50573.185921] audit: type=1400 audit(1727246484.056:28): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-a5a75691-85f9-42f5-97af-74b8fd704f7b" pid=4533 comm="apparmor_parser"
[50573.215685] audit: type=1400 audit(1727246484.084:29): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-b7e08da0-8f70-459f-b03e-d3e36d78a652" pid=4579 comm="apparmor_parser"
[50577.434326] audit: type=1400 audit(1727246488.304:30): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-b7e08da0-8f70-459f-b03e-d3e36d78a652" pid=4665 comm="apparmor_parser"
[50577.479764] vethd9a567c3: entered promiscuous mode
[50577.660613] audit: type=1400 audit(1727246488.528:31): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus-c2_</var/lib/incus>" pid=4698 comm="apparmor_parser"
[50577.763116] phys3JJb3m: renamed from veth4215420e
[50577.766722] eth0: renamed from phys3JJb3m
[50577.832306] Not activating Mandatory Access Control as /sbin/tomoyo-init does not exist.
```
### PB: pas d'addressage ipv6...
```bash
etu@spoke:~$ incus ls                              
+------+---------+----------------------+------+-----------+-----------+
| NAME |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+------+---------+----------------------+------+-----------+-----------+
| c0   | RUNNING | 100.64.11.95 (eth0)  |      | CONTAINER | 0         |
+------+---------+----------------------+------+-----------+-----------+
| c1   | RUNNING | 100.64.11.44 (eth0)  |      | CONTAINER | 0         |
+------+---------+----------------------+------+-----------+-----------+
| c2   | RUNNING | 100.64.11.111 (eth0) |      | CONTAINER | 0         |
+------+---------+----------------------+------+-----------+-----------+
```

