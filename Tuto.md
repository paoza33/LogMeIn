🌐 Configuration VLANs et Réseau (pfSense + Switch + PCs)

🟥 Configurer VLANs sur pfSense

    Créer les VLANs :

        Aller dans Interfaces > Assignments > VLANs

        Cliquer sur Add

            Parent Interface : em1

            VLAN Tag :

                VLAN 10 → 10

                VLAN 20 → 20

                VLAN 40 → 40

            VLAN Priority : laisser vide

        Cliquer sur Save

        Répéter pour chaque VLAN.

🟦 Configurer VLANs sur les PCs

🖥️ PC Frontend (VLAN 10 - Direct pfSense)

✅ Dans pfSense :

    Interfaces > Assignments

        Assigner em2 (DMZ) et vérifier qu’elle est activée.

    Static IP (DMZ) :

        IPv4 Address : 192.168.10.1/28

    Activer le DHCP :

        Services > DHCP Server > DMZ (VLAN10)

        Plage IP : 192.168.10.2 - 192.168.10.14

✅ Sur le PC Frontend :

# Configurer la carte réseau en DHCP
sudo nano /etc/network/interfaces
auto ens4
allow-hotplug ens4
iface ens4 inet dhcp

# Supprimer IP statique et relancer la carte réseau
sudo ip addr flush dev ens4
sudo ifdown ens4 && sudo ifup ens4

# Si oubli de supprimer l’IP statique :
sudo ip addr flush dev ens4
sudo dhclient -r
sudo dhclient ens4

# Vérifier l’adresse IP
ip a

🖥️ PC Backend (VLAN 20)

✅ Dans pfSense :

    Interfaces > VLANs : vérifier que VLAN 20 est bien sur em1

    Interfaces > Assignments : ajouter VLAN20

    Activer l’interface

    Static IP (VLAN20) :

        IPv4 Address : 192.168.20.1/28

✅ Switch :

# Port vers PC Backend
conf t
interface Ethernet0/0
switchport mode access
switchport access vlan 20

# Port vers pfSense (em1)
interface EthernetX/X
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 1,10,20,40

✅ Sur le PC Backend :

# Lien Backend <-> pfSense (ens5)
auto ens5
iface ens5 inet static
    address 192.168.20.3
    netmask 255.255.255.240
    gateway 192.168.20.1
    dns-nameservers 192.168.20.1 8.8.8.8

# Lien Backend <-> DB (ens4)
auto ens4
iface ens4 inet static
    address 192.168.30.2
    netmask 255.255.255.248

# Redémarrer la configuration réseau
sudo systemctl restart networking
ip a

🖥️ PC Zabbix (VLAN 40)

✅ Dans pfSense :

    Interfaces > VLANs : vérifier que VLAN 40 est bien sur em1

    Interfaces > Assignments : ajouter VLAN40

    Activer l’interface

    Static IP (VLAN40) :

        IPv4 Address : 192.168.40.1/28

    Activer le DHCP :

        Plage IP : 192.168.40.2 - 192.168.40.14

✅ Switch :

# Port vers le PC Zabbix
conf t
interface Ethernet1/1
switchport mode access
switchport access vlan 40

✅ Sur le PC Zabbix :

sudo nano /etc/network/interfaces
auto ens4
allow-hotplug ens4
iface ens4 inet dhcp

# Supprimer IP statique et relancer la carte réseau
sudo ip addr flush dev ens4
sudo ifdown ens4 && sudo ifup ens4

# Supprimer et redemander IP
sudo ip addr flush dev ens4
sudo dhclient -r
sudo dhclient ens4

# Vérifier
sudo systemctl restart networking
ip a

🟩 Configurer le serveur DB

✅ Sur le PC Backend :

# Activer le forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Rendre le forwarding permanent
sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1

# Attribuer IP statique
auto ens4
iface ens4 inet static
    address 192.168.30.4
    netmask 255.255.255.248

🛡️ Configuration temporaire du Firewall pfSense

✅ VLAN 20 (Backend)

    Pass :

        Source : opt2 subnet

        Destination : DMZ subnets

    Pass :

        Source : DMZ subnet

        Destination : opt2 subnets

    Block :

        Source : any

        Destination : opt2

✅ VLAN 10 (Frontend)

    Pass :

        Source : DMZ subnets

        Destination : opt2 subnets

    Pass :

        Source : opt2

        Destination : DMZ

⚠️ VLAN 40 (Zabbix)

✅ À configurer après selon les besoins.

🔄 Résumé des réseaux

| VLAN | Nom      | Réseau           | pfSense IP      | DHCP Range                    |
|-------|----------|------------------|-----------------|------------------------------|
| 10    | Frontend | 192.168.10.0/28  | 192.168.10.1    | 192.168.10.2 - 192.168.10.14 |
| 20    | Backend  | 192.168.20.0/28  | 192.168.20.1    | (Static pour Backend)         |
| 30    | DB Link  | 192.168.30.0/29  | (Pas de pfSense)| (Pas de DHCP)                |
| 40    | Zabbix   | 192.168.40.0/28  | 192.168.40.1    | 192.168.40.2 - 192.168.40.14 |
