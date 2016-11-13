# Recap

Siège = Machine 4 **Port 13**
Agence principale = Machine 1 **Port 17**
Agence principale 2 = Machine 2 **Port 2**
Agence secondaire = Machine 3 **Port 15**

Port 7 → Siège (tous les VLAN du siège)  
Port 8 → Agence Principale (idem)
Port 9 → Agence Principale 2
Port 10 → Agence Secondaire

## Initilisation

On up les cartes em1 sur toutes les machines

    $ sudo ifconfig em1 up

On créé les VLAN sur le switch 192.168.168.169, on a besoin d'un port pour plusieurs Vlan quasi tout le temps sauf pour l'agence secondaire et l'agence principale
Du coup on va créer un Vlan taggé sur les ports **15 et 17** pour les Vlans (config switch)


## Siège *13* → 10.1.0.0 255.255.0.0

- Vlan 2 : Poste utilisateur → 10.1.2.0

    PC-Site4-1 : 10.1.2.10
    PC-Site4-2 : 10.1.2.11

    Sonic X2 : 10.1.2.1

    centreon 1 10.1.1.11

```
$ vconfig add em1 2  
$ sudo ifconfig em1.2 up
```

- Vlan 3 : Serveur de donnée → 10.1.1.0

    Sonic X3 : 10.1.1.1

        $ vconfig em1 add 3  
        $ sudo ifconfig em1.3 up
    
    PLD-MARS-Debian.web 10.1.1.10

- Vlan 4 : DHCP → 10.1.3.1

## Agence principale *17* → 10.2.0.0 255.255.0.0

- Vlan 5 : Poste utilisateur → 10.2.2.0  

    PC-Site1-1 : 10.2.2.10  
    PC-Site2-2 : 10.2.2.11

- Vlan 6 : Serveur → 10.2.1.0   

    PLD-Mars-Debian.web-1 : 10.2.1.10

- Vlan 7 : Automate → 10.2.3.0  

    DAB-Site1-1 : 10.2.3.10  
    DAB-Site1-2 : 10.2.3.11

## Agence secondaire *15* → 10.4.0.0 255.255.0.0

- Vlan 8 : Poste utilisateur → 10.4.2.0  

    PC-Site3-1 : 10.4.2.10  
    PC-Site3-2 : 10.4.2.11  

- Vlan 9 : Automate → 10.4.3.0  

    DAB-Site3-1 : 10.4.3.10  
    DAB-Site3-2 : 10.4.3.11  


## Agence principale 2 *2* → 10.3.0.0 255.255.0.0

- Vlan 10 : Poste utilisateur → 10.3.2.0  

    PC-Site2-1 : 10.3.2.10  
    PC-Site2-2 : 10.3.2.11

- Vlan 11 : Serveur → 10.3.1.0  

    PLD-Mars-Debian.web-2 : 10.3.1.10

- Vlan 12 : Automates → 10.3.3.0  

    DAB-Site2-1 : 10.3.3.10
    DAB-Site2-2 : 10.3.3.11


- Vlan 13 : interco port 1 ← switch port 22 non taggé → Vyos Agence Principale (machine 1 port 1,2 taggé IP 10.0.9.6)

- Vlan 14 : interco port 17 ← switch port 23 non taggé → Vyos Agence secondaire (machine 1 port 17 taggé IP 10.0.9.133)

- Vlan 15 : interco port 16 ← switch port 24 non taggé → Vyos Agence Principale 2 (machine 2 port 2 taggé IP 10.00.9.113)

- interco port 13,14 ← routeur port X5 IP 10.0.9.98 → Siège Machine 4 

## Config DAB

```
vi /etc/sysconfig/network-scripts/ifcfg-eth0

BOOTPROTO=none
NETMASK=
IPADDR=
NETWORK=
GATEWAY=
```

## Config Serveur

```
vi /etc/network/interfaces

iface eth0 inet static
    adress 10.3.1.10
    netmask 255.255.255.0
    gateway 10.3.1.1
```
    

Une fois les Vlan créé et les cartes virtuelles créé, on configure les VM pour qu'ils utilisent les cartes virtuelles (connexion par pont) 
On affecte des adresses IP à chaque VM en fonction des IP que l'on a défini


On configure l'ip du vayos comme passerelle des periphérques des sous réseau et on configure le vayos pour que pour une carte réseau virtuel son adresse
soit l'adresse mise dans la passerelle

Problème, on peut bien communiquer sur deux vlan differents avec des vm relié au même Vyos mais on ne peut pas communiquer avec les autres sièges pour l'instant
 