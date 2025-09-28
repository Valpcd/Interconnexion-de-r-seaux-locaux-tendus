# Interconnexion de réseaux locaux & étendus
## TP1 Routage inter VLAN

## 3. Raccordement au commutateur de distribution
```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP1$ sudo ovs-vsctl get port tap319 vlan_mode
trunk
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP1$ sudo ovs-vsctl get port tap119 vlan_mode
access
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP1$ sudo ovs-vsctl get port tap119 tag
268
```
```powershell
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP1$ sudo ovs-vsctl set port tap119 tag=410
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP1$ sudo ovs-vsctl get port tap119 tag
410
```
```yaml
ovs:
  switches:
    - name: dsw-host
      ports:
        - name: tap319 #Router port
          type: OVSPort
          vlan_mode: Trunk
          trunks: [210,410]
        - name: tap119 #Hosting VM port
          type: OVSPort
          vlan_mode: access
          tag: 410
```
```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP1$ $HOME/masters/scripts/switch-conf.py switch.yaml
----------------------------------------
Configuring switch dsw-host
>> Port tap319 vlan_mode set to Trunk
>> Port tap319 trunks set to [210, 410]
>> Port tap119 vlan_mode is already set to access
>> Port tap119 tag is already set to 410
----------------------------------------
```
### Lancement des VM:
```yaml
kvm:
  vms:
    - vm_name: router
      master_image: debian-testing-amd64.qcow2 # master image to be used
      force_copy: false # do not force copy the master image to the VM image
      memory: 1024
      tapnum: 319
    - vm_name: hosting
      master_image: debian-testing-amd64.qcow2 # master image to be used
      force_copy: false # do not force copy the master image to the VM image
      memory: 1024
      tapnum: 119
```
```bash
valerianpacaud@oscar:~/vm/M1/Interconnexion_de_réseaux_locaux_&_étendus/TP1$ $HOME/masters/scripts/lab-startup.py lab.yaml
Copying /home/valerianpacaud/masters/debian-testing-amd64.qcow2 to router.qcow2...done
Creating OVMF_CODE.fd symlink...
Creating router_OVMF_VARS.fd file...
Starting router...
~> Virtual machine filename   : router.qcow2
~> RAM size                   : 1024MB
~> SPICE VDI port number      : 6219
~> telnet console port number : 2619
~> MAC address                : b8:ad:ca:fe:01:3f
~> Switch port interface      : tap319, trunk mode
~> IPv6 LL address            : fe80::baad:caff:fefe:13f%dsw-host
router started!
Copying /home/valerianpacaud/masters/debian-testing-amd64.qcow2 to hosting.qcow2...done
Creating hosting_OVMF_VARS.fd file...
Starting hosting...
~> Virtual machine filename   : hosting.qcow2
~> RAM size                   : 1024MB
~> SPICE VDI port number      : 6019
~> telnet console port number : 2419
~> MAC address                : b8:ad:ca:fe:00:77
~> Switch port interface      : tap119, access mode
~> IPv6 LL address            : fe80::baad:caff:fefe:77%vlan410
hosting started!
```
## 4. Rôle routeur
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
             - 172.21.208.2
             - 2001:678:3fc:d2::2
    vlans:
     enp0s1.210:
       id: 210
       link: enp0s1
       addresses:
         - 172.21.208.3/22
         - 2001:678:3fc:d2::3/64
       routes:
         - to: default
           via: 172.21.208.1
         - to: "::/0"
           via: "fe80:d2::1"
           on-link: true
     enp0s1.410:
       id: 410
       link: enp0s1
       addresses:
         - 172.24.10.1/24  #adresses de passerelle du serveur qui héberge les conteneurs
         - fda0:7a62:1A7::1/64
```
```bash
etu@router:~$ sudo netplan apply

** (generate:795): WARNING **: 19:28:57.340: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:794): WARNING **: 19:28:57.603: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:794): WARNING **: 19:28:57.660: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.
[ 4151.228748] 8021q: 802.1Q VLAN Support v1.8
[ 4151.229160] 8021q: adding VLAN 0 to HW filter on device enp0s1
```

```bash
etu@router:~$ networkctl status
● Interfaces: 1, 2, 3, 4
       State: routable                                        
Online state: online                                          
     Address: 172.21.208.3 on enp0s1.210
              172.24.10.1 on enp0s1.410
              2001:678:3fc:d2::3 on enp0s1.210
              2001:678:3fc:d2:baad:caff:fefe:13f on enp0s1.210
              fda0:7a62:1a7::1 on enp0s1.410
              fe80::baad:caff:fefe:13f on enp0s1
              fe80::baad:caff:fefe:13f on enp0s1.210
              fe80::baad:caff:fefe:13f on enp0s1.410
     Gateway: 172.21.208.1 on enp0s1.210
              fe80:d2::1 on enp0s1.210
         DNS: 2001:678:3fc:3::2
              172.21.208.2
              2001:678:3fc:d2::2

[.......]

```
### Tests de connextivité
```bash
etu@router:~$ ping -q -c2 172.21.208.1
PING 172.21.208.1 (172.21.208.1) 56(84) bytes of data.

--- 172.21.208.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 1.631/1.741/1.852/0.110 ms
etu@router:~$ ping 9.9.9.9
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.
64 bytes from 9.9.9.9: icmp_seq=1 ttl=48 time=43.3 ms
64 bytes from 9.9.9.9: icmp_seq=2 ttl=48 time=27.3 ms
64 bytes from 9.9.9.9: icmp_seq=3 ttl=48 time=27.9 ms
64 bytes from 9.9.9.9: icmp_seq=4 ttl=48 time=27.3 ms

--- 9.9.9.9 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 27.263/31.431/43.316/6.865 ms
etu@router:~$ ping -q -c2 fe80:d2::1%enp0s1.210
PING fe80:d2::1%enp0s1.210 (fe80:d2::1%enp0s1.210) 56 data bytes

--- fe80:d2::1%enp0s1.210 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 1.211/9.234/17.258/8.023 ms
etu@router:~$ ping -q -c2 2620:fe::fe
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 40.555/40.557/40.559/0.002 ms
etu@router:~$ host quad9.net
quad9.net has address 216.21.3.77
quad9.net has IPv6 address 2620:0:871:9000::77
quad9.net mail is handled by 20 mx2.quad9.net.
quad9.net mail is handled by 5 mx1.quad9.net.
quad9.net mail is handled by 10 mx4.quad9.net.
```

### 4.2. Activation de la fonction routage
```bash
etu@router:~$ cat << EOF | sudo tee /etc/sysctl.d/10-routing.conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
net.ipv4.conf.all.log_martians=1
EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
net.ipv4.conf.all.log_martians=1
etu@router:~$ sudo sysctl --system
* Applique /usr/lib/sysctl.d/10-coredump-debian.conf …
* Applique /etc/sysctl.d/10-routing.conf …
* Applique /usr/lib/sysctl.d/50-default.conf …
* Applique /usr/lib/sysctl.d/50-pid-max.conf …
* Applique /etc/sysctl.conf …
kernel.core_pattern = core
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
net.ipv4.conf.all.log_martians = 1
kernel.sysrq = 0x01b6
kernel.core_uses_pid = 1
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.enp0s1/210.rp_filter = 2
net.ipv4.conf.enp0s1/410.rp_filter = 2
net.ipv4.conf.enp0s1.rp_filter = 2
net.ipv4.conf.lo.rp_filter = 2
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.enp0s1/210.accept_source_route = 0
net.ipv4.conf.enp0s1/410.accept_source_route = 0
net.ipv4.conf.enp0s1.accept_source_route = 0
net.ipv4.conf.lo.accept_source_route = 0
net.ipv4.conf.default.promote_secondaries = 1
net.ipv4.conf.enp0s1/210.promote_secondaries = 1
net.ipv4.conf.enp0s1/410.promote_secondaries = 1
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

### 4.3. Activation de la traduction d'adresses
```bash
etu@router:~$ sudo apt -y install nftables
Installing:                                     
  nftables

Installing dependencies:
  libjansson4  libnftables1

Paquets suggérés :
  firewalld

Summary:
  Upgrading: 0, Installing: 3, Removing: 0, Not Upgrading: 0
  Download size: 439 kB
  Space needed: 1 326 kB / 70,4 GB available

Réception de :1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Réception de :2 https://deb.debian.org/debian trixie/main amd64 libjansson4 amd64 2.14-2+b2 [39,9 kB]
Réception de :3 https://deb.debian.org/debian trixie/main amd64 libnftables1 amd64 1.1.0-2 [325 kB]
Réception de :4 https://deb.debian.org/debian trixie/main amd64 nftables amd64 1.1.0-2 [73,6 kB]
439 ko réceptionnés en 0s (1 005 ko/s)
Sélection du paquet libjansson4:amd64 précédemment désélectionné.
(Lecture de la base de données... 28673 fichiers et répertoires déjà installés.)
Préparation du dépaquetage de .../libjansson4_2.14-2+b2_amd64.deb ...
Dépaquetage de libjansson4:amd64 (2.14-2+b2) ...                            ] 
Sélection du paquet libnftables1:amd64 précédemment désélectionné.          ] 
Préparation du dépaquetage de .../libnftables1_1.1.0-2_amd64.deb ...
Dépaquetage de libnftables1:amd64 (1.1.0-2) ...                             ] 
Sélection du paquet nftables précédemment désélectionné.                    ] 
Préparation du dépaquetage de .../nftables_1.1.0-2_amd64.deb ...
Dépaquetage de nftables (1.1.0-2) ...█████▊                                 ] 
Paramétrage de libjansson4:amd64 (2.14-2+b2) ...                            ] 
Paramétrage de libnftables1:amd64 (1.1.0-2) ...████████▏                    ] 
Paramétrage de nftables (1.1.0-2) ...██████████████████████████▌            ] 
Traitement des actions différées (« triggers ») pour man-db (2.13.0-1) ...  ] 
Traitement des actions différées (« triggers ») pour libc-bin (2.40-2) ...
Scanning processes...                                                           
Scanning linux images...                                                        

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

```bash
#!/usr/sbin/nft -f
flush ruleset
table ip nat {
 chain postrouting {
 type nat hook postrouting priority 100;
 oifname "enp0s1.210" counter packets 0 bytes 0 masquerade
 }
}
table ip6 nat {
 chain postrouting {
 type nat hook postrouting priority 100;
 oifname "enp0s1.210" counter packets 0 bytes 0 masquerade
 }
}
etu@router:~$ sudo nft -f /etc/nftables.conf
etu@router:~$ systemctl status nftables.service
○ nftables.service - nftables
     Loaded: loaded (/usr/lib/systemd/system/nftables.service; disabled; preset>
     Active: inactive (dead)
       Docs: man:nft(8)
             http://wiki.nftables.org

etu@router:~$ sudo systemctl enable nftables.service
sudo systemctl start nftables.service
sudo systemctl status nftables.service
Created symlink '/etc/systemd/system/sysinit.target.wants/nftables.service' → '/usr/lib/systemd/system/nftables.service'.
● nftables.service - nftables
     Loaded: loaded (/usr/lib/systemd/system/nftables.service; enabled; preset:>
     Active: active (exited) since Mon 2024-09-23 17:44:39 CEST; 36ms ago
 Invocation: 034d2fcdd2ca43e7b48b0e61e3d5702f
       Docs: man:nft(8)
             http://wiki.nftables.org
    Process: 1509 ExecStart=/usr/sbin/nft -f /etc/nftables.conf (code=exited, s>
   Main PID: 1509 (code=exited, status=0/SUCCESS)
   Mem peak: 4M
        CPU: 32ms

sept. 23 17:44:38 router systemd[1]: Starting nftables.service - nftables...
sept. 23 17:44:39 router systemd[1]: Finished nftables.service - nftables.

```
```bash
etu@router:~$ sudo nft list ruleset
table ip nat {
        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
                oifname "enp0s1.210" counter packets 0 bytes 0 masquerade
        }
}
table ip6 nat {
        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
                oifname "enp0s1.210" counter packets 0 bytes 0 masquerade
        }
}
```
### 4.4. Activation de l'adressage automatique pour le réseau de conteneurs
```bash
etu@router:~$ sudo nft list ruleset
table ip nat {
        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
                oifname "enp0s1.210" counter packets 0 bytes 0 masquerade
        }
}
table ip6 nat {
        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
                oifname "enp0s1.210" counter packets 0 bytes 0 masquerade
        }
}
etu@router:~$ sudo apt -y install dnsmasq
Installing:                                     
  dnsmasq

Installing dependencies:
  dns-root-data  dnsmasq-base

Summary:
  Upgrading: 0, Installing: 3, Removing: 0, Not Upgrading: 0
  Download size: 568 kB
  Space needed: 1 292 kB / 70,4 GB available

Réception de :1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Réception de :2 https://deb.debian.org/debian trixie/main amd64 dnsmasq-base amd64 2.90-4 [498 kB]
Réception de :3 https://deb.debian.org/debian trixie/main amd64 dnsmasq all 2.90-4 [66,0 kB]

[.....]

```

```bash
etu@router:~$ sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.dist
```
```bash
etu@router:~$ cat << EOF | sudo tee /etc/dnsmasq.conf
# Specify Container VLAN interface
interface=enp0s1.410
# Enable DHCPv4 on Container VLAN
dhcp-range=172.24.10.20,172.24.10.200,3h
# Enable IPv6 router advertisements
enable-ra
# Enable SLAAC
dhcp-range=::,constructor:enp0s1.410,ra-names,slaac
# Optional: Specify DNS servers
dhcp-option=option:dns-server,172.16.0.2,9.9.9.9
dhcp-option=option6:dns-server,[2001:678:3fc:3::2],[260:fe::fe]
# Avoid DNS listen port conflict between dnsmasq and systemd-resolved
bind-interfaces
listen-address=172.24.10.1
EOF
```

```bash
etu@router:~$ systemctl status dnsmasq
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; preset: >
     Active: active (running) since Mon 2024-09-23 17:51:56 CEST; 1min 13s ago
 Invocation: 7a7c9b09ef6045329e9c750d890d8b14
    Process: 1753 ExecStartPre=/usr/share/dnsmasq/systemd-helper checkconfig (c>
    Process: 1759 ExecStart=/usr/share/dnsmasq/systemd-helper exec (code=exited>
    Process: 1765 ExecStartPost=/usr/share/dnsmasq/systemd-helper start-resolvc>
   Main PID: 1764 (dnsmasq)
      Tasks: 1 (limit: 1086)
     Memory: 908K (peak: 2.7M)
        CPU: 102ms
     CGroup: /system.slice/dnsmasq.service
             └─1764 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -r>

sept. 23 17:52:18 router dnsmasq-dhcp[1764]: RTR-ADVERT(enp0s1.410) fda0:7a62:1>
sept. 23 17:52:37 router dnsmasq-dhcp[1764]: RTR-ADVERT(enp0s1.410) fda0:7a62:1>
sept. 23 17:52:43 router dnsmasq-dhcp[1764]: RTR-ADVERT(enp0s1.410) fda0:7a62:1>
sept. 23 17:52:51 router dnsmasq-dhcp[1764]: DHCPDISCOVER(enp0s1.410) b8:ad:ca:>
sept. 23 17:52:51 router dnsmasq-dhcp[1764]: DHCPOFFER(enp0s1.410) 172.24.10.13>
sept. 23 17:52:51 router dnsmasq-dhcp[1764]: DHCPREQUEST(enp0s1.410) 172.24.10.>
sept. 23 17:52:51 router dnsmasq-dhcp[1764]: DHCPACK(enp0s1.410) 172.24.10.130 >
sept. 23 17:52:51 router dnsmasq-dhcp[1764]: SLAAC-CONFIRM(enp0s1.410) fda0:7a6>
sept. 23 17:52:52 router dnsmasq-dhcp[1764]: RTR-ADVERT(enp0s1.410) fda0:7a62:1>
```
## 5. Rôle serveur de conteneurs
### 5.1. Configuration de l'interface du serveur

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s1:
      dhcp4: false
      dhcp6: false
      accept-ra: true
      addresses:
        - 172.24.10.2/22
        - fda0:7a62:19A::2/64
      routes:
        - to: default
          via: 172.24.10.1
        - to: "::/0"
          via: fda0:7a62:19A::1
          on-link: true
      nameservers:
        addresses:
          - 172.16.0.2
          - 2001:678:3fc:3::2
```
```bash
etu@hosting:~$ sudo netplan apply

** (generate:996): WARNING **: 18:12:00.077: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:995): WARNING **: 18:12:00.364: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:995): WARNING **: 18:12:00.445: Permissions for /etc/netplan/enp0s1.yaml are too open. Netplan configuration should NOT be accessible by others.
```

### Tests de connecivité

```bash
etu@hosting:~$ ping -q -c2 9.9.9.9
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 45.314/45.344/45.375/0.030 ms
etu@hosting:~$ ping -q -c2 2620:fe::fe
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 41.256/41.286/41.317/0.030 ms
etu@hosting:~$ host kernel.org
kernel.org has address 139.178.84.217
kernel.org has IPv6 address 2604:1380:4641:c500::1
kernel.org mail is handled by 10 smtp1.kernel.org.
kernel.org mail is handled by 10 smtp2.kernel.org.
kernel.org mail is handled by 10 smtp3.kernel.org.
```
### 5.2. Installation du gestionnaire de conteneurs Incus

```bash
etu@hosting:~$ sudo apt -y install incus
Installing:                                     
  incus

Installing dependencies:
  attr           incus-client   liblxc1t64    libsubid5       uidmap
  dns-root-data  libcowsql0     liblzo2-2     lxcfs
  dnsmasq-base   libjansson4    libnftables1  rsync
  incus-agent    liblxc-common  libraft0      squashfs-tools

Paquets suggérés :
  btrfs-progs  lvm2            incus-tools
  ceph-common  zfsutils-linux  python3-braceexpand

Paquets recommandés :
  minio-client

Summary:
  Upgrading: 0, Installing: 18, Removing: 0, Not Upgrading: 4
  Download size: 29,3 MB
  Space needed: 98,0 MB / 70,6 GB available

Réception de :1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
[....]
```

```bash
etu@hosting:~$ groups etu
etu : etu adm sudo users incus incus-admin
```
Après reconnection:

```bash
etu@hosting:~$ groups
etu adm sudo users incus-admin incus
```
```bash
etu@hosting:~$ incus admin init
[ 7524.438484] audit: type=1400 audit(1727108785.251:13): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lxc-container-default" pid=1689 comm="apparmor_parser"
[ 7524.441662] audit: type=1400 audit(1727108785.255:14): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lxc-container-default-cgns" pid=1689 comm="apparmor_parser"
[ 7524.444852] audit: type=1400 audit(1727108785.255:15): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lxc-container-default-with-mounting" pid=1689 comm="apparmor_parser"
[ 7524.447946] audit: type=1400 audit(1727108785.255:16): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lxc-container-default-with-nesting" pid=1689 comm="apparmor_parser"
Would you like to use clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Where should this storage pool store its data? [default=/var/lib/incus/storage-pools/default]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: no
Would you like to use an existing bridge or host interface? (yes/no) [default=no]: yes
Name of the existing bridge or host interface: enp0s1
Would you like the server to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes]: 
Would you like a YAML "init" preseed to be printed? (yes/no) [default=no]: 
```
Profil par défaut des conteneurs:
```bash
tu@hosting:~$ incus profile show default
config: {}
description: Default Incus profile
devices:
  eth0:
    name: eth0
    nictype: macvlan
    parent: enp0s1
    type: nic
  root:
    path: /
    pool: default
    type: disk
name: default
used_by: []
project: default
```
Lancement des 3 conteneurs:

```bash
etu@hosting:~$ for i in {0..2}; do incus launch images:debian/trixie c$i; done
Launching c0
Retrieving image: rootfs: 100% (60.81MB/s)[ 7678.892935] audit: type=1400 audit(1727108939.714:17): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-ad9f6a5c-2c2b-4939-9457-aa0b7d5b4d3e" pid=1754 comm="apparmor_parser"
Retrieving image: Unpacking image: 100% (1.29GB/s)[ 7678.949585] audit: type=1400 audit(1727108939.770:18): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-ad9f6a5c-2c2b-4939-9457-aa0b7d5b4d3e" pid=1760 comm="apparmor_parser"
[ 7678.993220] audit: type=1400 audit(1727108939.814:19): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-cfabcffa-29f4-4e23-8c36-30f3fcde8107" pid=1762 comm="apparmor_parser"
[ 7683.704940] audit: type=1400 audit(1727108944.526:20): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-cfabcffa-29f4-4e23-8c36-30f3fcde8107" pid=1771 comm="apparmor_parser"
[ 7683.909109] audit: type=1400 audit(1727108944.730:21): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus-c0_</var/lib/incus>" pid=1795 comm="apparmor_parser"
[ 7684.001090] phys47O9e2: renamed from mac231226ff
[ 7684.003562] eth0: renamed from phys47O9e2
[ 7684.070462] Not activating Mandatory Access Control as /sbin/tomoyo-init does not exist.
Launching c1                                       
[ 7684.374197] audit: type=1400 audit(1727108945.194:22): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-e2ab27a7-f81a-461e-a0da-4dd6172649f6" pid=1924 comm="apparmor_parser"
Retrieving image: Unpacking image: 100% (3.08GB/s)[ 7684.430505] audit: type=1400 audit(1727108945.250:23): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-e2ab27a7-f81a-461e-a0da-4dd6172649f6" pid=1938 comm="apparmor_parser"
[ 7684.458625] audit: type=1400 audit(1727108945.278:24): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-4cb28be5-8ff9-4935-9c8d-90f717507d5f" pid=1948 comm="apparmor_parser"
[ 7688.463844] audit: type=1400 audit(1727108949.282:25): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-4cb28be5-8ff9-4935-9c8d-90f717507d5f" pid=2006 comm="apparmor_parser"
[ 7688.726523] audit: type=1400 audit(1727108949.546:26): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus-c1_</var/lib/incus>" pid=2028 comm="apparmor_parser"
[ 7688.960721] physQuouhR: renamed from macc962ef9c
[ 7688.963825] eth0: renamed from physQuouhR
[ 7689.019183] Not activating Mandatory Access Control as /sbin/tomoyo-init does not exist.
Launching c2                                       
[ 7689.225853] audit: type=1400 audit(1727108950.046:27): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-3596363f-9b74-4e1f-953c-70ffb5da2de1" pid=2082 comm="apparmor_parser"
Retrieving image: Unpacking image: 100% (814.55MB/s)[ 7689.264091] audit: type=1400 audit(1727108950.082:28): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-3596363f-9b74-4e1f-953c-70ffb5da2de1" pid=2136 comm="apparmor_parser"
[ 7689.298338] audit: type=1400 audit(1727108950.118:29): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus_archive-3f34e208-1b66-404a-9754-a5e55a2e0ac1" pid=2165 comm="apparmor_parser"
[ 7693.222012] audit: type=1400 audit(1727108954.043:30): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="incus_archive-3f34e208-1b66-404a-9754-a5e55a2e0ac1" pid=2237 comm="apparmor_parser"
[ 7693.378222] audit: type=1400 audit(1727108954.199:31): apparmor="STATUS" operation="profile_load" profile="unconfined" name="incus-c2_</var/lib/incus>" pid=2260 comm="apparmor_parser"
[ 7693.488631] phys5dDbTL: renamed from mac50b10e92
[ 7693.493298] eth0: renamed from phys5dDbTL
[ 7693.553526] Not activating Mandatory Access Control as /sbin/tomoyo-init does not exist.
```

```bash
etu@hosting:~$ incus ls                              
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
| NAME |  STATE  |         IPV4         |                   IPV6                    |   TYPE    | SNAPSHOTS |
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
| c0   | RUNNING | 172.24.10.20 (eth0)  | fda0:7a62:1a7:0:216:3eff:fe52:f659 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
| c1   | RUNNING | 172.24.10.67 (eth0)  | fda0:7a62:1a7:0:216:3eff:feef:de19 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
| c2   | RUNNING | 172.24.10.193 (eth0) | fda0:7a62:1a7:0:216:3eff:fe99:7c59 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
```
On remarque que l'adressage automatique fonctionne car les addresses attribués aux conteneurs font partie de la plage DHCP configuré sur le routeur.
--------------------------------------
Test des communications réseau depuis les conteneurs:
```bash
etu@hosting:~$ for i in {0..2}
do
 echo ">>>>>>>>>>>>>>>>> c$i"
 incus exec c$i -- ping -qc2 9.9.9.9
 incus exec c$i -- ping -qc2 2620:fe::fe
done
>>>>>>>>>>>>>>>>> c0
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 28.401/37.316/46.232/8.915 ms
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 40.318/40.834/41.351/0.516 ms
>>>>>>>>>>>>>>>>> c1
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 29.258/36.973/44.688/7.715 ms
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 40.037/48.675/57.314/8.638 ms
>>>>>>>>>>>>>>>>> c2
PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.

--- 9.9.9.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 28.235/28.465/28.695/0.230 ms
PING 2620:fe::fe (2620:fe::fe) 56 data bytes

--- 2620:fe::fe ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 39.838/49.353/58.868/9.515 ms
```
--------------------------------

```bash
etu@hosting:~$ bash run-commands-in-containers.sh
>>>>>>>>>>>>>>>>> c0
Get:1 http://deb.debian.org/debian trixie InRelease [169 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [49.6 kB]
Get:3 http://deb.debian.org/debian-security trixie-security InRelease [43.5 kB]
Get:4 http://deb.debian.org/debian trixie/main amd64 Packages.diff/Index [27.9 kB]
Get:5 http://deb.debian.org/debian trixie/main Translation-en.diff/Index [27.9 kB]
Get:6 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-22-0804.28.pdiff [5327 B]
Get:7 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-22-1411.20.pdiff [4445 B]
Get:8 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-22-2009.32.pdiff [655 B]
Get:9 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-0204.24.pdiff [25.3 kB]
Get:10 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-0809.13.pdiff [614 B]
Get:11 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-1410.16.pdiff [1310 B]
Get:11 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-1410.16.pdiff [1310 B]
Get:12 http://deb.debian.org/debian trixie/main Translation-en 2024-09-22-1411.20.pdiff [1122 B]
Get:13 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-0204.24.pdiff [2995 B]
Get:14 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-0809.13.pdiff [466 B]
Get:15 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-1410.16.pdiff [476 B]
Get:15 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-1410.16.pdiff [476 B]
Fetched 361 kB in 2s (209 kB/s)                            
All packages are up to date.    
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c1
Get:1 http://deb.debian.org/debian trixie InRelease [169 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [49.6 kB]
Get:3 http://deb.debian.org/debian-security trixie-security InRelease [43.5 kB]
Get:4 http://deb.debian.org/debian trixie/main amd64 Packages.diff/Index [27.9 kB]
Get:5 http://deb.debian.org/debian trixie/main Translation-en.diff/Index [27.9 kB]
Get:6 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-22-0804.28.pdiff [5327 B]
Get:7 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-22-1411.20.pdiff [4445 B]
Get:8 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-22-2009.32.pdiff [655 B]
Get:9 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-0204.24.pdiff [25.3 kB]
Get:10 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-0809.13.pdiff [614 B]
Get:11 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-1410.16.pdiff [1310 B]
Get:11 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-1410.16.pdiff [1310 B]
Get:12 http://deb.debian.org/debian trixie/main Translation-en 2024-09-22-1411.20.pdiff [1122 B]
Get:13 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-0204.24.pdiff [2995 B]
Get:14 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-0809.13.pdiff [466 B]
Get:15 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-1410.16.pdiff [476 B]
Get:15 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-1410.16.pdiff [476 B]
Fetched 361 kB in 2s (214 kB/s)                            
All packages are up to date.    
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c2
Get:1 http://deb.debian.org/debian trixie InRelease [169 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [49.6 kB]
Get:3 http://deb.debian.org/debian-security trixie-security InRelease [43.5 kB]
Get:4 http://deb.debian.org/debian trixie/main amd64 Packages.diff/Index [27.9 kB]
Get:5 http://deb.debian.org/debian trixie/main Translation-en.diff/Index [27.9 kB]
Get:6 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-22-0804.28.pdiff [5327 B]
Get:7 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-22-1411.20.pdiff [4445 B]
Get:8 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-22-2009.32.pdiff [655 B]
Get:9 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-0204.24.pdiff [25.3 kB]
Get:10 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-0809.13.pdiff [614 B]
Get:11 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-1410.16.pdiff [1310 B]
Get:11 http://deb.debian.org/debian trixie/main amd64 Packages 2024-09-23-1410.16.pdiff [1310 B]
Get:12 http://deb.debian.org/debian trixie/main Translation-en 2024-09-22-1411.20.pdiff [1122 B]
Get:13 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-0204.24.pdiff [2995 B]
Get:14 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-0809.13.pdiff [466 B]
Get:15 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-1410.16.pdiff [476 B]
Get:15 http://deb.debian.org/debian trixie/main Translation-en 2024-09-23-1410.16.pdiff [476 B]
Fetched 361 kB in 2s (218 kB/s)                            
All packages are up to date.    
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0

```

Adressage statique des conteneurs:

```bash
etu@hosting:~$ bash static-addressing.sh
>>>>>>>>>>>>>>>>> c0
0% [Working]
Hit:1 http://deb.debian.org/debian trixie InRelease
Hit:2 http://deb.debian.org/debian trixie-updates InRelease
Hit:3 http://deb.debian.org/debian-security trixie-security InRelease
All packages are up to date.    
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c1
Hit:1 http://deb.debian.org/debian trixie InRelease
Hit:2 http://deb.debian.org/debian trixie-updates InRelease
Hit:3 http://deb.debian.org/debian-security trixie-security InRelease
All packages are up to date.    
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
>>>>>>>>>>>>>>>>> c2
Hit:1 http://deb.debian.org/debian trixie InRelease
Hit:2 http://deb.debian.org/debian trixie-updates InRelease
Hit:3 http://deb.debian.org/debian-security trixie-security InRelease
All packages are up to date.    
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
etu@hosting:~$ incus ls
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
| NAME |  STATE  |         IPV4         |                   IPV6                    |   TYPE    | SNAPSHOTS |
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
| c0   | RUNNING | 172.24.10.20 (eth0)  | fda0:7a62:1a7:0:216:3eff:fe52:f659 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
| c1   | RUNNING | 172.24.10.67 (eth0)  | fda0:7a62:1a7:0:216:3eff:feef:de19 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
| c2   | RUNNING | 172.24.10.193 (eth0) | fda0:7a62:1a7:0:216:3eff:fe99:7c59 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+-------------------------------------------+-----------+-----------+
```
