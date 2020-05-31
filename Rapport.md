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
### Installation des package ssh:

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
 
### Voici la réponse de la commande *ip a* avant et aprés la configuration:
#### Avant:
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
#### Après:
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

### Exemple d'utilisation:

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
        
  #### Ainsi que la commande:

    root@c1:~# ping google.com
    PING google.com (216.58.213.174) 56(84) bytes of data.
    64 bytes from par21s04-in-f174.1e100.net (216.58.213.174): icmp_seq=1 ttl=51 time=11.5 ms


## clonage du container c1:

### Il faut stoper le container c1:

    lxc-stop c1

### Puis on clone c1 en c2 et c3:
    lxc-copy -n c1 -N c2
    lxc-copy -n c1 -N c3

### Enfin on verifie la présence des 3 containers avec la commande:
    lxc-ls
*On verifie bien la pésence de C1,C2 et C3.*


## Creation du bridge:
### Fonctionnalités:

- Persistant dans sysctl.conf: **/etc/sysctl.conf** 
- Persistant dans interfaces: **/etc/network/interfaces**

#### Insérez les lignes suivantes dans **/etc/network/interfaces** :

    auto lxc-bridge-nat
    iface lxc-bridge-nat inet static
    bridge_ports none
    bridge_fd 0
    bridge_maxwait 0
    address 192.168.100.1
    netmask 255.255.255.0
    
 Puis adaptez la configuration réseau dans **/var/lib/lxc/monContainer/config**:

    # Network configuration
    lxc.net.0.type = veth
    lxc.net.0.link = lxc-bridge-nat
    lxc.net.0.ipv4.address = 192.168.100.10/24
    lxc.net.0.ipv4.gateway = 192.168.100.1
    lxc.net.0.flags = up
    lxc.net.0.hwaddr = 00:16:3e:d5:76:c8  


### Activer le routage IP

    echo 1 > /proc/sys/net/ipv4/ip_forward

### Rendre le routage IP permanent

Décommentez la ligne suivante dans **/etc/sysctl.conf**:
Pour activer le "packet forwarding" pour IPv4:

    net.ipv4.ip_forward=1


Pour activer les changements effectués dans **sysctl.conf** vous aurez besoin pour exécuter la commande:
    

     sysctl -p /etc/sysctl.conf
    
Ou vous pouvez aussi le faire en redémarrant le service:
        
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


    root@c1:~# ifconfig eth0 192.168.100.2/24
    root@c1:~# route add default gw 192.168.100.1

Changer le nameserver dans le fichier **/etc/resolv.conf** :

    root@c1:~# echo 'nameserver 192.168.1.254' > /etc/resolv.conf
   *192.168.1.254 est l'addresse ip de la box*

Pour résoudre le probleme de ping, on lance la commande: 
       
      iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

Pour la lancer automatiquement au reboot de la machine,
 on la met dans le fichier **/etc/network/if-up.d/iptables**
 
        echo   iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE > /etc/network/if-up.d/iptables
        
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


## Installation d’un serveur DHCP

### Sur la machine qu'on souhaite rendre en serveur DHCP (ici c1)
### Installation des paquets isc-dhcp-server:

    apt-get install isc-dhcp-server 
    
    
### Assigner une adresse IP statique pour ce serveur DHCP avec eth0 dans le fichier **/etc/network/interfaces**:

    # L'interface réseau « loopback » (toujours requise)
    auto lo
    iface lo inet loopback
    auto eth0
    iface eth0 inet static
        address 192.168.100.10
        netmask 255.255.255.0
        broadcast 192.168.1.255
        gateway 192.168.100.1
 ### Modification du fichier  **/etc/resolv.conf** :
 
    # Utilisation du serveur DNS public de Google :
    nameserver 8.8.8.8
    nameserver 8.8.4.4
    # (ou bien utilisez les adresses DNS fournies par votre fournisseur d'accès)
    192.168.1.254

 ### Modification du fichier **/etc/dhcp/dhcpd.conf **:
    option domain-name "asr.fr";
    # Utilisation du serveur DNS public de Google (ou bien utilisez l'adresse du serveur    DNS fournie par votre fournisseur d'accès):
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    # Configuration de votre sous-réseau (subnet) souhaité :
    subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.101 192.168.100.254;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.100.255;
    option routers 192.168.100.100;
    option domain-name-servers home;
    }
    default-lease-time 600;
    max-lease-time 7200;
    # Indique que nous voulons être le seul serveur DHCP de ce réseau :
    authoritative;

On précise l'interface dans **/etc/default/isc-dhcp-server**:
    
    INTERFACESv4="eth0" 
    INTERFACESv6=""
    
On redemarre ensuite le service:
    
    service isc-dhcp-server restart    

Verification du fonctionnement du service avec la commande:

    service isc-dhcp-server status
    
  
## Sur les autres machines (les postes clients, ici c2,c3):

Après avoir lancé le serveur DHCP, il faut attribuer les autres appareils au réseau et ils devraient se voir attribuer automatiquement leur adresse de DHCP. Pour s'assurer d'un ordinateur est configuré pour obtenir son adresse IP en utilisant DHCP, écrire ceci dans le fichier « **/etc/network/interfaces** » :


    # L'interface réseau « loopback » (toujours requise)
    auto lo
    iface lo inet loopback

    # Obtenir l'adresse IP de n'importe quel serveur DHCP
    auto eth0
    iface eth0 inet dhcp
    
## Configuration avancée
 *Attribuer des adresses fixes*
 
Pour attribuer une adresse fixe, par exemple 192.168.100.64. à un ordinateur particulier, ici c2, il faut ajouter au fichier de configuration **/etc/dhcp/dhcpd.conf** de **C1**
les lignes suivantes: ainsi,c2 obtient une adresse IP fixe en 64 et c3 une adresse variable.

    host c2 {
    hardware ethernet 00:16:3e:3c:63:5f;
    fixed-address 192.168.100.64;
    }
    
    host c3 {
    hardware ethernet 00:16:3e:99:df:3c;
    }
    
 #### On retrouve l'adresse mac de **C2** dans le fichier **/var/lib/lxc/c2/config** à la ligne suivante, on fait de meme pour celle de **C3**:
 
    lxc.net.0.hwaddr = 00:16:3e:3c:63:5f    

On peut aussi l'obtenir en exécutant la commande **ifconfig** sur le client quand l'interface est activée.

On verifie ensuite avec la commande **ip a** ou **ifconfig**, qui nous retourne bien:

    inet 192.168.100.64/24 brd 192.168.100.255 scope global dynamic eth0
 
 
## Installation du serveur DNS


Bind9 est un serveur DNS, c'est à dire que Bind9 se charge de traduire un nom de domaine en une ip. C'est ce qui permet d'avoir des adresses telles que http://www.exemple.com au lieu de http://111.222.111.222/exemple.com.
Pour rappel DNS = Domain Name Server (Serveur de nom de domaine)
On installe les packages avec la commande suivante sur le container qui sera le serveur DNS:
    
       apt-get install bind9
       
### Configuration des clients

#### Modification sur le poste c1:

On chosis de passer par la configuration du serveur DHCP et de  modifier le fichier  **/etc/dhcp/dhcpd.conf**, pour utiliser le domaine asr.fr plutot que de modifier le domaine de chaque machine du réseau (domain et search).

    option domain-name "asr.fr";
    option domain-name-servers asr.fr; 
 
 ##### Déclaration des zones:
 Dans le fichier **/etc/bind/named.conf.local** , on ajoute les lignes suivantes :


 
     zone "asr.fr" {
     type master;
     file "/etc/bind/db.asr.fr";
     };

##### Ici le nom de domaine est toujours asr.fr et la configuration de la zone pour asr.fr se fera dans le fichier /etc/bind/db.asr.fr
   
   	
    ; BIND data file for local loopback interface
    ;
    $TTL	604800
    @	IN	SOA	server. server.localhost. (
                 2020051204		; Serial
                 604800		; Refresh
                  86400		; Retry
                2419200		; Expire
                 604800 )	; Negative Cache TTL
    ;
    @	IN	NS	server.
    server	IN	A	192.168.100.10
    c2	IN	A	192.168.100.64
    c3	IN 	A	192.168.100.4
    www	IN	CNAME	c3
   

 La commande ci dessous verifie les fichiers de zone:

    named-checkzone asr.fr /etc/bind/db.asr.fr 
 
 Réponse:
    
    zone asr.fr/IN: loaded serial 2
    OK

Vérification des fichiers de configuration:
	
	named-checkconf /etc/bind/named.conf.local

Réponse: Si aucun message n'apparait alors c'est reussi!

On redamare le service:

    /etc/init.d/bind9 restart
    
Dans les fichiers **/etc/resolv.conf** des postes clients on ajoutes les lignes:

    domain asr.fr
    search asr.fr

Enfin, dans le fichier **/etc/bind/named.conf.options**, on ajoute les lignes suivantes:

    forwarders {
	192.168.1.254; #ip du fournisseur d'accès
	8.8.8.8; # DNS de google
 	};

###  Résolution inverse de nom

Dans le fichier **/etc/bind/named.conf.local** on ajoute la zone suivante:


    zone "100.168.192.in-addr.arpa" {
	    type master;
	   file "/etc/bind/db.asr.fr.inverse";
    };
    
Et dans le fichier **/etc/bind/db.asr.fr.inverse** :

    ;
    ; BIND data file for local loopback interface
    ;
    $TTL	604800
    @	IN	SOA	server. 		server.localhost. (
                      2020051204		; Serial
                 604800		; Refresh
                  86400		; Retry
                2419200		; Expire
                 604800 )	; Negative Cache TTL
    ;
    @	IN	NS	server.asr.fr.
    4	IN	PTR 	c3.asr.fr.
    5 	IN	PTR	c2.asr.fr.
    2	IN 	PTR	server.asr.fr.

On redémarre le service avec la commande suivante:

    systemctl restart bind9  


###  Résolution DNS secondaire

### Sur c1, le serveur principal:

On autorise le transfert des données vers c2, le serveur secondaire en modifiant le fichier: **/etc/bind/named.conf.options** : 
        
        allow-transfer { 192.168.100.64; };
        
Dans **/etc/bind/db.asr.fr**, on ajoute la ligne :

     IN      NS      192.168.100.64
     
Dans le fichier **/etc/bind/named.conf.local**, on modifie :

    zone "asr.fr" IN {
        type master;
        file "/etc/bind/db.asr.fr";
        notify yes;
    };
        
### Sur c2:
On ajoutes les lignes ci-dessous dans le fichier **/etc/bind/named.conf.local**:


    zone "asr.fr" {
	type slave;
	file "/etc/bind/db.asr.fr";
	masters {192.168.100.10; };
    allow-notify {192.168.100.10;};


    zone "100.168.192.in-addr.arpa" {
	type slave;
	file "/etc/bind/db.asr.fr.inverse";
	masters {192.168.100.10; };
    };
       
*192.168.100.10* est l'adresse Ip de c1.
*allow-notify {192.168.100.10;}* permet l'autorisation de l'envoi de notification vers le serveur principal (c1).

Explications:

	
	IN		Signifie internet, c’est à dire que la zone après le IN est 				destinée à internet

	SOA		Star Of Authority indiquant le serveur de nom faisant autorité 				c’est à dire le DNS principal.
	
	Serial		N° de série à incrémenter à chaque modification de ce fichier. 				Par convention, on écrit: année-mois-jour-numéro_à_2_chiffres. 				doit comporter 10 chiffres

	$TTL	TTL (Time to Live) pour cette zone. Nombre de secondes pendant lesquels 		les informations de la zone peuvent être considérées comme valides et 			être mises en cache
	
	Refresh		A l'expiration du délai Refresh exprimé en secondes, le serveur 			esclave va entrer en communication avec le maitre, si il ne le 				trouve pas, il fera une nouvelle tentative au bout du délai 				Retry si au bout du délai Expire il considérera que le serveur 				n'est plus disponible.
	
	Retry		Nombre de secondes avant d'effectuer une nouvelle demande au 				serveur maître en cas de non réponse.
	
	Expire		Temps (en secondes) d'expiration du serveur principal en cas de 			non réponse.
	
	Negative cache TTL  définit la durée de vie d'une réponse NXDOMAIN de notre part.
	
	IP		l'ip de votre serveur
	
	NS 		renseigne le nom des serveurs de noms pour le domaine.
	
	PTR 		 c'est simplement la résolution inverse (le contraire du type A).


## Serveur d'annuaire NIS

### Installation

#### Installation partie serveur
On installe le paquet  ypserv avec la commande suivante sur le container qui sera le serveur NIS:  
NIS est un protocole client/serveur permettant la centralisation des informations sur un réseau UNIX.
 	
	apt  install nis

Il nous sera demandé le nom de notre domaine NIS. C'est un choix arbitraire, il faut juste que ce soit le même pour le serveur et les clients.
Dans le sujet du TD le nom demandé est asr.fr.
Le nom du serveur NIS doit être pleinnement qualifié et suivi du nom d'hôte de la machine:

Dans **/etc/hosts**:

	192.168.100.10 server.asr.fr c1


 Editez le fichier **/etc/sysconfig/network** et rajoutez la ligne:
 NISDOMAIN=<nom du domaine NIS>

		NISDOMAIN=asr.fr

- Éditez le fichier **/etc/default/nis** pour définir notre machine comme serveur maître.

Remplacez "NISSERVER=false" par:

	"NISSERVER=master"

- Éditez le fichier **/var/yp/Makefile**

Remplacez "ALL =   passwd  group hosts rpc services netid protocols netgrp" par "ALL =   passwd shadow  group hosts rpc services netid protocols netgrp"

- Exécutez la commande:

		/usr/lib/yp/ypinit -m
		
On nous demande d'entrer tous les nom de domaine des serveurs esclaves et de valider la configuration avec *(Ctrl+D)*.

On peut ajouter un groupe et un utilisateur à ce groupe au serveur NIS:

	groupadd -g 55 groupe1
	useradd -G groupe1 jerem
	passwd jerem

On verifie que  l'utilisateur a bien été ajouté en faisant un cat **/etc/group**:
	
	groupe1:x:55:jerem
	jerem:x:1001:

Étant donné que l'on a modifié les fichiers group et passwd, il faut mettre à jour les tables du serveur NIS.

	cd /var/yp; make

- Redémarrez NIS:

		/etc/init.d/nis restart
	
####  Installation partie client
##### Apres avoir installé nis sur le post client (apt-get install nis):

- Editez le fichier **/etc/yp.conf**

Rajouter la ligne :
domain <nom du domaine> server <nom du serveur NIS>

	domain asr.fr server server.asr.fr


- Editez le fichier **/etc/nsswitch.conf**

Rajoutez "nis" après "compat" devant tout les éléments qui devront être centralisés  (quels fichiers synchroniser avec le serveur):

	passwd:         compat  nis
	group:          compat  nis
	shadow:         compat nis
	gshadow:        files

	hosts:          files dns nis

On redéfini le serveur NIS dans le fichier **/etc/hosts**:

	192.168.100.10 server.asr.fr c1

Il ne reste plus qu'à redémarrer:

	systemctl restart rpcbind nis 

Nous pouvons tester si la configuration a réussi avec les commandes:
- **ypwhich** qui retourne bien le FQDN de notre serveur: **server.asr.fr**.
-**ypdomainname**qui retourne le nom du domaine NIS:**asr.fr**.

*FQDN = fully qualified domain name ( nom de domaine pleinement qualifié)*
Pour changer le mot de passe de l'utilisateur, il suffit d'executer la commande **yppasswd**.

On peut se connecter à l’utilisateur que nous avons créé.

	Login :jerem
	Password :jerem
	
	
## Lightweight Directory Access Protocol (Ldap)

### Installation et configuration

#### Intallation des paquets Open LDAP :
	
	apt‐get install slapd ldap‐utils

#### Configuration de slapd :
	
	dpkg‐reconfigure slapd
Après configuration de notre slapd nous pouvons maintenant modifier notre fichier de configuration qui se trouve dans le répertoire **/etc/ldap/ldap.conf**.

	BASE     dc=asr,dc=local
	URI      ldap://openldap.mondomaine.local/

#### Configuration des fichiers LDIF et DIT

Créez un nouveau fichier ldif:
Insérer les lignes suivantes:

	# 1.
	dn: cn=config
	changetype: modify
	replace: olcLogLevel
	olcLogLevel: stats
	# 2.1.
	dn: olcDatabase={1}hdb,cn=config
	changetype: modify
	add: olcDbIndex
	olcDbIndex: uid eq
	-
	# 2.2.
	add: olcDbIndex
	olcDbIndex: cn eq
	-
	# 2.3.
	add: olcDbIndex
	olcDbIndex: ou eq
	-
	# 2.4.
	add: olcDbIndex
	olcDbIndex: dc eq



On prend les modification en compte avec la commande:

	ldapmodify -QY EXTERNAL -H ldapi:/// -f ~/olc-mod1.ldif

Nous allons maintenant créer des unités d’organisations (OU), une pour nos utilisateurs et une pour les groupes.

	dn: ou=Utilisateurs,dc=asr.fr,dc=local
	ou: Utilisateurs
	objectClass: organizationalUnit
	
	dn: ou=Groupes,dc=asr.fr,dc=local
	ou: Groupes
	objectClass: organizationalUnit
	
On ajoute ces OU à notre base de donnée ldap comme suit:

	ldapadd ‐x ‐D cn=admin,dc=asr.fr,dc=com ‐W ‐f ou.ldif
	Enter LDAP Password: 
	adding new entry "ou=Utilisateurs,dc=asr.fr,dc=local"
	adding new entry "ou=Groupes,dc=asr.fr,dc=local"
-x : utilise une authentification simple
-D : spécifie le "Dinstinguished Name" qui va se connecter à notre base LDAP 
-f : spécifie le nom du fichier contenant les informations à rajouter

#### Création d'utilisateur
user.ldif pour l’ajout d’utilisateurs.

	dn: cn=myuser,ou=Groupes,dc=asr.fr,dc=local
	cn: myuser
	gidNumber: 20000
	objectClass: posixGroup

	dn: uid=myuser,ou=Utilisateurs,dc=asr.fr,dc=local
	uid: myuser
	uidNumber: 20000
	gidNumber: 20000
	cn: myuser
	sn: myuser

	objectClass: posixAccount
	objectClass: shadowAccount
	objectClass: organizationalPerson
	loginShell: /bin/bash
	homeDirectory: /home/myuser
	userPassword: myuser


Pour ajouter les utilisateur à notre base de donnée LDAP :

	ldapadd ‐x ‐D cn=admin,dc=asr.fr,dc=com ‐W ‐f user.ldif
	Enter LDAP Password:
	adding new entry "cn=myuser,ou=Groupes,dc=asr.fr,dc=local"
	adding new entry "uid=myuser,ou=Utilisateurs,dc=asr.fr,dc=local"
	
	
On peut verifier que l'utilisateur a bien été créé avec la commande suivante:

	ldapsearch -xLLL uid=myuser

	
