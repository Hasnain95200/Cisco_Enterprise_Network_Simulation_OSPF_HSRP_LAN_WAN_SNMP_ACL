# Cisco_Enterprise_Network_Simulation_OSPF_HSRP_LAN_WAN_SNMP_ACL
Simulation réseau avancée sous Cisco Packet Tracer intégrant VLAN, routage OSPF, VPN IPsec, ACL,  HSRP, DHCP Relay, SNMP et serveurs Web/DNS. Projet complet avec configurations CLI et topologie finale documentée

[README.md](https://github.com/user-attachments/files/21736502/README.md)
# Projet Réseau Cisco Packet Tracer

## 1. Description générale
Ce projet a pour objectif de concevoir et configurer une infrastructure réseau d’entreprise simulée dans **Cisco Packet Tracer**.  
L’infrastructure est répartie sur deux sites interconnectés par un VPN IPsec site-à-site et met en œuvre plusieurs technologies réseau afin d'assurer la **haute disponibilité**, la **sécurité**, et la **fourniture de services essentiels**.

Les principales fonctionnalités implémentées sont :
- **Routage dynamique OSPF** pour l’interconnexion des réseaux.
- **VPN IPsec site-à-site** pour sécuriser la communication entre les deux sites.
- **VLANs** pour la segmentation du réseau et **HSRP** pour la redondance des passerelles.
- **ACLs** pour le contrôle des accès entre VLANs.
- **SNMP** et **Syslog** pour la supervision et la gestion des logs.
- **Serveurs Web et DNS** hébergés sur le réseau local.
- **DHCP** pour l’attribution automatique des adresses IP aux clients.

---

## 2. Topologie réseau

La topologie est composée de :
- **2 routeurs** (R1 , R3-HSRP) configurés avec HSRP pour assurer la redondance des passerelles , ainsi qu'un 3eme routeur pour le 2eme site (R2).
- **2 switchs principaux** connectant les postes clients et les serveurs et le switche (Switch0) sur le 2eme site .
- **Serveur DHCP** (VLAN 10) attribuant automatiquement les adresses IP aux postes et téléphones.
- **Serveur Web** et **Serveur DNS** accessibles depuis tous les VLANs autorisés par les ACL.
- **VPN IPsec site-à-site** entre les deux sites.
- **SNMP & Syslog** activés sur les routeurs pour supervision.

### Schéma réseau
Voici la topologie finale de notre réseau :

<img width="638" height="306" alt="Topologie_final" src="https://github.com/user-attachments/assets/af991475-ebe5-465b-bd37-e8da4f482b77" />

---

## 3. Plan d’adressage

| Périphérique                      | Interface | VLAN / Réseau     | Adresse IP   | Masque          | Remarques                              |
| --------------------------------- | --------- | ----------------- | ------------ | --------------- | -------------------------------------- |
| **R1 – Site 1 (principal)**       | Fa0/0.10  | VLAN 10 – Admin   | 192.168.10.1 | 255.255.255.0   | HSRP VIP 192.168.10.1, priorité 110    |
|                                   | Fa0/0.20  | VLAN 20 – RH      | 192.168.20.1 | 255.255.255.0   | DHCP relay vers 192.168.10.20          |
|                                   | Fa0/0.30  | VLAN 30 – IT      | 192.168.30.1 | 255.255.255.0   | Source téléphonie IP                   |
|                                   | Fa0/0.99  | VLAN 99 – native  | —            | —               | Pas d’IP                               |
|                                   | Fa0/1     | Lien VPN vers R2  | 10.0.0.1     | 255.255.255.252 | Crypto Map VPN-MAP                     |
| **R2 – Site 2**                   | Fa0/0     | Lien VPN vers R1  | 10.0.0.2     | 255.255.255.252 | Crypto Map VPN-MAP                     |
|                                   | Fa0/1     | LAN Site 2        | 192.168.40.1 | 255.255.255.0   |                                        |
| **R3 – Site 1 (HSRP secondaire)** | Fa0/0.10  | VLAN 10 – Admin   | 192.168.10.3 | 255.255.255.0   | HSRP VIP 192.168.10.1, priorité défaut |
|                                   | Fa0/0.20  | VLAN 20 – RH      | 192.168.20.3 | 255.255.255.0   | DHCP relay                             |
|                                   | Fa0/0.30  | VLAN 30 – IT      | 192.168.30.3 | 255.255.255.0   |                                        |


---

## 4. Étapes de configuration

### Étape 1 —VLAN + IP + ping
Objectif : Créer les VLANs, attribuer les IP sur les sous-interfaces des routeurs, et tester la connectivité entre postes du même VLAN.

Sur le Switch 1 (S1 du site 1) :

enable

configure terminal

! Création des VLAN

vlan 10

 name Vlan Admin
 
vlan 20

 name Vlan RH
vlan 30
 name Vlan IT
vlan 99
 name Vlan native

interface range fa0/2
switchport mode access
switchport access vlan 10
exit


! Port vers le routeur (trunk)
interface fa0/3
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30,99

end
write memory


Sur le Switch 2 (S2 du site 1) :


enable
configure terminal

! Création des VLAN
vlan 10
 name Vlan Admin
vlan 20
 name Vlan RH
vlan 30
 name Vlan IT
vlan 99
 name Vlan native


interface range fa0/2
switchport mode access
switchport access vlan 20
exit


interface range fa0/3
switchport mode access
switchport access vlan 30
exit

! Port vers le routeur (trunk)
interface fa0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30,99

end
write memory

Sur le router Router 1 (R1 du site 1)

conf t
interface Fa0/0.10
encapsulation dot1q 10
ip address 192.168.10.1 255.255.255.0
exit

interface Fa0/0.20
encapsulation dot1q 20
ip address 192.168.20.1 255.255.255.0
exit

interface Fa0/0.30
encapsulation dot1q 30
ip address 192.168.30.1 255.255.255.0
exit

interface Fa0/0.99
encapsulation dot1q 99 native
no ip address
exit

Après configuration manuelle des PC , exemple avec le PC1 dans le Vlan 10 (Admin) : PC1 > Desktop > IP Configuration  :

![Topologie finale](images/Configuration_étape1_PC1.png)


Test de connectivité :

Depuis PC1 vers PC2 :

ping 192.168.20.10 

Réponse :Pinging 192.168.30.30 with 32 bytes of data:

Request timed out.
Reply from 192.168.30.30: bytes=32 time<1ms TTL=127
Reply from 192.168.30.30: bytes=32 time<1ms TTL=127
Reply from 192.168.30.30: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.30.30:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms


### Étape 2 —Ajout d’un deuxième site + Routage dynamique OSPF
Objectif : Créer un deuxième site distant, composé d’un routeur (R2), d’un switch (S0) et d’un PC (PC4), ainsi que mettre en place un routage OSPF entre les deux sites.**

Sur le Router 2 (R2 du site 2) :

enable
conf t

! Interface LAN vers switch/PC2
interface Fa0/1
ip address 192.168.40.1 255.255.255.0
no shutdown
exit

! Interface WAN vers R1
interface Fa0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
exit

end
write m

Sur le Router 1 (R1 du site 1) :

conf t
interface Fa0/1
ip address 10.0.0.1 255.255.255.252
no shutdown
exit

Ensuite configuration OSPF pour les routers R1 et R2 :

Sur R1 :

conf t
router ospf 1
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
network 10.0.0.0 0.0.0.3 area 0
exit

Sur R2 :

conf t
router ospf 1
network 192.168.40.0 0.0.0.255 area 0
network 10.0.0.0 0.0.0.3 area 0
exit

Vérifie que OSPF a bien appris les routes :

Depuis R2 :

show ip route ospf 

Réponse : 

O    192.168.10.0 [110/2] via 10.0.0.1, 00:31:31, FastEthernet0/0
O    192.168.20.0 [110/2] via 10.0.0.1, 00:31:31, FastEthernet0/0
O    192.168.30.0 [110/2] via 10.0.0.1, 00:31:31, FastEthernet0/0

Donc les routes vers les différents sous-réseaux du site 1 sont bien connues et correctement apprises, ce qui confirme le bon fonctionnement d’OSPF.

### Étape 3 : Mise en place d’un VPN IPsec site-à-site entre R1 et R2
Objectif : Ce tunnel VPN va chiffrer les communications entre les deux routeurs — comme si les deux sites étaient reliés par un câble privé sécurisé.


On commence par définir les ACL , pour chiffrer le bon traffic:

Router R1 (site 1):

ip access-list extended VPN-TRAFFIC
 permit ip 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255

Router R2 (site 2):

ip access-list extended VPN-TRAFFIC
 permit ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255


Mise en place de la politique de sécuriter (ISAKMP) :

Sur R1 et R2 :

crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400

Sur R1 :

crypto isakmp key Cisco123 address 10.0.0.2 


Sur R2 :

crypto isakmp key Cisco123 address 10.0.0.1

Création du transform-set :

Sur R1 et R2 :

crypto ipsec transform-set MYSET esp-aes esp-sha-hmac
 mode tunnel

Finir par crée la crypto map :

Sur R1 :

crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set MYSET
 match address VPN-TRAFFIC

Sur R2 :

crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.1
 set transform-set MYSET
 match address VPN-TRAFFIC

On applque ensuite la crypto map sur les interface WAN :

			[R1]------(WAN)------[R2]

Sur R1 :

interface fa0/1
 crypto map VPN-MAP

Sur R2 :

interface fa0/0
 crypto map VPN-MAP

Pour la vérification :

Faire sur R1 ou R2 :

# show crypto isakmp sa

Réponse : IPv4 Crypto ISAKMP SA
dst             src             state          conn-id slot status

IPv6 Crypto ISAKMP SA


# show crypto ipsec sa

Réponse :

interface: FastEthernet0/0
    Crypto map tag: VPN-MAP, local addr 10.0.0.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (192.168.40.0/255.255.255.0/0/0)
   remote  ident (addr/mask/prot/port): (192.168.10.0/255.255.255.0/0/0)
   current_peer 10.0.0.1 port 500
    PERMIT, flags={origin_is_acl,}
   #pkts encaps: 0, #pkts encrypt: 0, #pkts digest: 0
   #pkts decaps: 0, #pkts decrypt: 0, #pkts verify: 0
   #pkts compressed: 0, #pkts decompressed: 0
   #pkts not compressed: 0, #pkts compr. failed: 0
   #pkts not decompressed: 0, #pkts decompress failed: 0
   #send errors 0, #recv errors 0

     local crypto endpt.: 10.0.0.2, remote crypto endpt.:10.0.0.1
     path mtu 1500, ip mtu 1500, ip mtu idb FastEthernet0/0
     current outbound spi: 0x0(0)

     inbound esp sas:

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:

     outbound ah sas:

     outbound pcp sas:

Donc, nous avons bien un tunnel chiffré entre R1 et R2.


### Étape 4 : Sécurité – ACL (Access Control Lists)

Objectif: Contrôler le trafic entre les VLANs ou entre les sites, restreindre certains accès (par exemple, empêcher le VLAN 20 d’accéder au VLAN 10) et protéger les équipements sensibles tels que les routeurs et les serveurs

Dans notre sénario on va interdire tout le trafic IP du VLAN 20 vers VLAN 10 et permettre tout le reste.

Crée un ACL, sur le router R1 :

ip access-list extended ACL-SECURITE
 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 permit ip any any

Ensuite l'appliquer a l'interface du router R1 :

interface fa0/0.10
 ip access-group ACL-SECURITE in

Vérification : On ping du PC2 (vlan 20) vers un autre pc d'un autre réseau :

Sur PC2 (vlan 20): ping 192.168.10.30

Pinging 192.168.10.30 with 32 bytes of data:

Reply from 192.168.20.1: Destination host unreachable.
Reply from 192.168.20.1: Destination host unreachable.
Reply from 192.168.20.1: Destination host unreachable.
Reply from 192.168.20.1: Destination host unreachable.

Ping statistics for 192.168.10.30:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)

Voici une version corrigée et mieux formulée :

Ici, la réponse indique que le destinataire est injoignable, ce qui confirme que l’ACL fonctionne parfaitement.


### Étape 5 : Sécurité – Mise en place d'un serveur DHCP.
Objectif :Attribuer automatiquement et dynamiquement les adresses IP et paramètres réseau aux équipements, afin de simplifier la gestion, réduire les erreurs de configuration et optimiser l’administration du réseau.

On ajoute un serveur DHCP , au vlan 10
Et on configure le switch 1 (S1) :

interface range fa0/4
switchport mode access
switchport access vlan 10

Ensuite, configurer de façon statique le serveur DHCP :  Comme le pc précédement (étape 1)

On ajoute ensuite la commande suivante dans le routeur R1, afin que les PC des autres VLAN puissent obtenir une adresse IP grâce au serveur DHCP.

interface Fa0/0.20 (vlan 20)
ip helper-address 192.168.10.20  ← IP du serveur DHCP dans VLAN 10
exit


interface Fa0/0.30 (vlan 30)
ip helper-address 192.168.10.20  ← IP du serveur DHCP dans VLAN 10
exit

On fini par configurer les pools dans le serveur DHCP :

![Topologie finale](images/Configuration_pool_Serveur_DHCP.png)

On fait de même pour tous les VLAN

Vérification : Aller sur un PC (1 ou 3) > IP config > et cocher DHCP :

![Topologie finale](images/PC3_DHCP.png)


### Étape 6 : HSRP redondance 
Objectif :Assurer la continuité du service en doublant les routeurs, avec une IP virtuelle comme passerelle pour basculer automatiquement en cas de panne.

Mettre en place un nouveau router R3-HRCP :

			   [R3-HRCP]------[S1]------[R1]----(WAN)------[R2]

Configuration du switch S1 :


interface fa0/1   # Port vers R1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30

interface fa0/5   # Port vers R3-HRSP
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30



Configuration de HRCP Router R3-HRCP :

conf t
interface fa0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.3 255.255.255.0
 standby 1 ip 192.168.10.1
 standby 1 priority 100
 standby 1 preempt

interface fa0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.3 255.255.255.0
 standby 2 ip 192.168.20.1
 standby 2 priority 100
 standby 2 preempt

interface fa0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.3 255.255.255.0
 standby 3 ip 192.168.30.1
 standby 3 priority 100
 standby 3 preempt


configuration de HRCP Router R1 :

iconf t
interface fa0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.2 255.255.255.0
 standby 1 ip 192.168.10.1
 standby 1 priority 110
 standby 1 preempt

interface fa0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.2 255.255.255.0
 standby 2 ip 192.168.20.1
 standby 2 priority 110
 standby 2 preempt

interface fa0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.2 255.255.255.0
 standby 3 ip 192.168.30.1
 standby 3 priority 110
 standby 3 preempt

Vérification : on shutdonw R1 :

![Topologie finale](images/R1_shutdown.png)

On ping de pc1->pc3 :ping 192.168.30.30

Pinging 192.168.30.30 with 32 bytes of data:

Request timed out.
Request timed out.
Reply from 192.168.30.30: bytes=32 time<1ms TTL=127
Reply from 192.168.30.30: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.30.30:
    Packets: Sent = 4, Received = 2, Lost = 2 (50% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

Donc la redondance fonctionne.

### Étape 6 : Configurer SNMP et les logs
Objectif :SNMP permet de surveiller et gérer à distance les équipements réseau, tandis que les logs enregistrent les événements et activités pour faciliter le diagnostic et la sécurité.

Configuration minimal de SNMP sur Cisco Packet Tracer sur router R1 :

snmp-server community public RO
snmp-server community private RW
logging buffered 4096
exit

Vérification :

show snmp
show logging

Résultat :

snmp-server community public RO
snmp-server community private RW


### Étape 7 : configurer les serveurs Web et DNS
Objectif : Configurer les serveurs Web permet d’héberger des sites accessibles aux utilisateurs, tandis que configurer les serveurs DNS assure la résolution des noms de domaine en adresses IP pour faciliter la navigation.

On ajoute dans le VLAN 10 le serveur Web ainsi que le serveur DNS.

Pour ce faire, on configure le switch S1 comme à l’étape 1 afin que les serveurs soient dans le bon VLAN et on adapte les interfaces.

Ensuite, on configure le serveur Web :

On le fait directement grâce au serveur DHCP.
	
![Topologie finale](images/ServeurWeb.png)

Et puis ensuite on configure le service du serveur :

![Topologie finale](images/Service_serveurWeb.png)

Vérification : Sur PC1 > Desktop > Web Browser > http://192.168.10.2

![Topologie finale](images/WebBrowser.png)

On voit que l’on a bien accès au Web à partir d’un PC (en l’occurrence le PC1 ici).

Ensuite, on configure le serveur DNS :

On le fait de manière statique.

![Topologie finale](images/Serveur_DNS.png)

Et puis ensuite on configure le service du serveur :

![Topologie finale](images/Service_serveurDNS.png)

Ensuite on configure les PC avec l'adresse IP du serveur DNS( ou directement avec le serveur DHCP)

Vérification : de PC1 -> PC3 : ping pc3-it

Pinging 192.168.30.30 with 32 bytes of data:

Reply from 192.168.30.30: bytes=32 time=1ms TTL=127
Reply from 192.168.30.30: bytes=32 time<1ms TTL=127
Reply from 192.168.30.30: bytes=32 time=10ms TTL=127
Reply from 192.168.30.30: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.30.30:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 10ms, Average = 2ms

Donc le serveur DNS est OK.

#Conclusion :

Ce projet a permis de mettre en place une infrastructure réseau complète et sécurisée intégrant VLAN, routage OSPF, VPN IPsec, ACLs,redondance HSRP, DHCP Relay, supervision SNMP, ainsi que des services Web et DNS. 
La documentation et la configuration associées assurent la reproductibilité et la maintenance future du réseau.

Ce lab démontre la mise en place d’un réseau complet et sécurisé avec routage, sécurité, redondance et services essentiels. La documentation fournie permet de reproduire et maintenir facilement l’infrastructure.




