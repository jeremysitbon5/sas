# Rapport de TP

Pour mener a bien les séances de Tp, nous avons installé une machine virtuelle Debian 64 à partir de l'image mini.iso.
On a choisi un disque de **8Gb** et **1Gb** de RAM.  
Mode d'accés réseau par pont.

## Partitionnement:
La commande  **fdisk**   va nous permettre de partitionner le disque.     
Les 4 premieres partitions sont les  **primaires** ensuite viens les **logiques**.          Pour créer une partition **logique** il faut créer la partition **étendu**.
    
    (fdisk /dev/sda1)

### Création du système de fichier:
La commande **mkfs.ext** permet de formater les partitions.
Exemple: 

    mkfs.ext4 /dev/sda1 
ou **mkswap** pour formater la partition du swap.


## Gestion des utilisateurs:
Editer les fichier **/etc/passwd** (utilisateurs du système séparés par des «: »  ) , **/etc/shadow** (mots de passes en chiffrés), **/etc/group** (informations sur les groupes) et **/etc/gshadow** (informations cachées sur les groupes).

### Creation des users:       
Editer les fichiers **/etc/passwd** et **/etc/shadow**
Ou par la commande:

    user add (**-m** pour la création automatique de la home).

## Configuration de ssh:
### installation des package ssh:

    apt install ssh

Dans le fichier **/etc/ssh/sshd_config** , on ajoute les lignes:
*Enable_root ... yes
PermitRootLogin yes*

Puis on relance le service ssh avec:
    
    /etc/init.d/ssh restart 

### Connection à la machine via SSh:

        Ssh root@ipmachine -p 2222
*(2222 étant le port de la vm)*

## Installation du conteneur:
 ### installation du package lxc
    apt install lxc
 ### Statut du service lxc
    systemctl status lxc-net.service

### Activer le service lxc
    systemctl start lxc-net.service
 ### Configuration du réseau des conatainer
le fichier : **/etc/lxc/default.conf** : definit la configuration par default des containers
le repertoire : **/var/lib/lxc/** contient des repertoires avec la configuration des containers deja existant

#### La configuration réseau de Debian par defaut est désactivée.  
On va donc modifier le fichier: **/etc/lxc/default.conf** et ajouter les lignes suivantes:

    lxc.net.0.type = veth
    lxc.net.0.link = lxcbr0
    lxc.net.0.flags = up
    lxc.net.0.hwaddr = 00:16:3e:xx:xx:xx
    lxc.apparmor.profile = generated
    lxc.apparmor.allow_nesting = 1
    
### Décommenter (ou mettre à "true") la ligne suivante dans le fichier */etc/default/lxc*:
    USE_LXC_BRIDGE="true"
### Redemarrer lxc-net:
    systemctl restart lxc-net.service
 
## Voici la réponse de la commande *ip a* avant et aprés la configuration:
    
    ip a
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:24:f5:b7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.24/24 brd 192.168.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 84125sec preferred_lft 84125sec
    inet6 fe80::a00:27ff:fe24:f5b7/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    root@sas:~# systemctl restart lxc-net.service
    root@sas:~# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:24:f5:b7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.24/24 brd 192.168.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 84109sec preferred_lft 84109sec
    inet6 fe80::a00:27ff:fe24:f5b7/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    3: lxcbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 00:16:3e:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 10.0.3.1/24 scope global lxcbr0
       valid_lft forever preferred_lft forever
    
## Création du premier conteneur C1
    lxc-create -n c1 -t debian -- -r buster

### Demarer le container:
    lxc-start -n c1
    
### Verifier l'état du container avec la commande:
    lxc-info c1

Exemple d'utilisation:

    root@sas:~# lxc-stop c1
    root@sas:~# lxc-info c1
    Name:           c1
    State:          STOPPED
    root@sas:~# lxc-start c1
    root@sas:~# lxc-info c1
    Name:           c1
    State:          RUNNING
    PID:            1063
    CPU use:        0.32 seconds
    BlkIO use:      0 bytes
    Memory use:     14.68 MiB
    KMem use:       2.98 MiB
    Link:           vethL95HX5
    TX bytes:      608 bytes
    RX bytes:      486 bytes
    Total bytes:   1.07 KiB

#### Dans **/var/lib/lxc/monContainer/config**, on peut constater:

    root@sas:~# cat /var/lib/lxc/c1/config 
    

    # Uncomment the following line to support nesting containers:
    #lxc.include = /usr/share/lxc/config/nesting.conf
    # (Be aware this has security implications)

      lxc.net.0.type = veth
    lxc.net.0.link = lxcbr0
    lxc.net.0.flags = up
    lxc.net.0.hwaddr = 00:16:3e:a0:e6:76
    lxc.apparmor.profile = generated
    lxc.apparmor.allow_nesting = 1
    lxc.rootfs.path = dir:/var/lib/lxc/c1/rootfs

    # Common configuration
    lxc.include = /usr/share/lxc/config/debian.common.conf

    # Container specific configuration
    lxc.tty.max = 4
    lxc.uts.name = c1
    lxc.arch = amd64
    lxc.pty.max = 1024
   
 ### Activer la commande ping dans c1 pour tester la connection.
 #### Lancer c1 avec la commande suivante:
 
    lxc-attach c1
 #### Installer la commande ping avec la commande suivante:
    apt install iputils-ping
 
 #### On met à jour la commande apt-get (si erreur) à la ligne précédente:
    apt-get update
 
 ### Enfin, la commande ping nous répond:
        ping 8.8.8.8 **(adresse de google)**
        64 bytes from 8.8.8.8: icmp_seq=1 ttl=51 time=16.1 ms
        
  Ainsi que la commande:

    root@c1:~# ping google.com
    PING google.com (216.58.213.174) 56(84) bytes of data.
    64 bytes from par21s04-in-f174.1e100.net (216.58.213.174): icmp_seq=1 ttl=51 time=11.5 ms


## clonage deu container c1:

#### Il faut stoper le container c1:

    lxc-stop c1

#### Puis on clone c1 en c2 et c3:
    lxc-copy -n c1 -N c2
    lxc-copy -n c1 -N c3

#### Enfin on verifie la présence des 3 containers avec la commande:
    lxc-ls
**On verifie bien la pésence de C1,C2 et C3.**


## Creation du bridge:
Fonctionnalités:

Persistant dans sysctl.conf: **/etc/sysctl.conf** 
Persistant dans interfaces: **/etc/network/interfaces**
### Décommentez la ligne suivante dans **/etc/sysctl.conf**:
Pour activer le "packet forwarding" pour IPv4:

    net.ipv4.ip_forward=1
Insérez les lignes suivantes dans **/etc/network/interfaces** :

    auto lxc-bridge-nat
    iface lxc-bridge-nat inet static
    bridge_ports none
    bridge_fd 0
    bridge_maxwait 0
    address 192.168.100.1
    netmask 255.255.255.0
    up iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
Puis adaptez la configuration réseau dans **/var/lib/lxc/monContainer/config**:

    # Network configuration
    lxc.net.0.type = veth
    lxc.net.0.link = lxc-bridge-nat
    lxc.net.0.ipv4.address = 192.168.100.10/24
    lxc.net.0.ipv4.gateway = 192.168.100.1
    lxc.net.0.flags = up
    lxc.net.0.hwaddr = 00:16:3e:d5:76:c8  



  



    echo 1 > /proc/sys/net/ipv4/ip_forward





    /etc/init.d/networking restart

Vérifier avec **if config** ou **ip a** d’avoir: la ligne 4: nous montre l'interface du bridge avec son adresse ip.

    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b2:fe:14 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.29/24 brd 192.168.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 80392sec preferred_lft 80392sec
    inet6 fe80::a00:27ff:feb2:fe14/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    3: lxcbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 00:16:3e:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 10.0.3.1/24 scope global lxcbr0
       valid_lft forever preferred_lft forever
    4: lxc-bridge-nat: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global lxc-bridge-nat
       valid_lft forever preferred_lft forever
    inet6 fe80::ecf1:3cff:fe48:c48d/64 scope link 
       valid_lft forever preferred_lft forever

### Redemarrer c1 avec:
    lxc-start c1
### Rentrer dans c1 avec la commande:
    lxc-attach c1

### La commande *ip a* nous renvoi:

    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
    13: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:d5:76:c8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.100.10/24 brd 192.168.100.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fed5:76c8/64 scope link 
       valid_lft forever preferred_lft forever


    ifconfig eth0 192.168.100.2/24
root@c3:~# route add default gw 192.168.100.1
root@c3:~# echo 'nameserver 192.168.1.254' > /etc/resolv.conf
192.168.0.1 est l'addresse ip de la box

Pour résoudre le probleme de ping, on lance la commande 
       
      iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
 Pour la lancer automatiquement au reboot de la machine,
 on la met dans le fichier **/etc/network/if-up.d/iptables**
 
        echo   iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE > /etc/network/if-up.d/iptables
        
On ping sur l'adresse Ip de google:

    root@c1:~# ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=51 time=12.3 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=51 time=12.2 ms
    64 bytes from 8.8.8.8: icmp_seq=3 ttl=51 time=13.9 ms

On ping sur google.com:

    ping google.com
    PING google.com (216.58.198.206) 56(84) bytes of data.
    64 bytes from par10s27-in-f206.1e100.net (216.58.198.206): icmp_seq=1 ttl=51 time=15.3 ms
    64 bytes from par10s27-in-f206.1e100.net (216.58.198.206): icmp_seq=2 ttl=51 time=12.2 ms

