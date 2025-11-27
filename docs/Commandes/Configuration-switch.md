# Guide des commandes pour Routeur Swicth Cisco

# Rappel des commandes

## Switch

#### Configuration SSH et utilisateurs sur le switch cœur

```h
Coeur> enable
Coeur# conf t
Coeur(config)# enable secret "MotDePasse" [Pour mettre un MDP sur la commande enable]
Coeur(config)# ip domain name "NomDeDomaine.com"
Coeur(config)# ip ssh version 2 
Coeur(config)# crypto key generate rsa
 How many bits in the modulus [512]: 2048
Coeur(config)# username "NomDeL'Utilisateur" privilege 15 secret "MotDePasse" 
*Coeur(config)# line vty 0 15 
Coeur(config)# login local
Coeur(config)# transport input ssh 
Coeur(config)# exit
Coeur# exit
Coeur# write memory
```

### Configuration des VLAN

#### Création d’un VLAN

```h
Switch(config)# vlan 10
Switch(config-vlan)# name Utilisateurs
Switch(config)# vlan 20
Switch(config-vlan)# name Serveurs
```

#### Suppression d’un VLAN

```h
Switch(config)# no vlan 20
```

#### Vérification des VLAN existants

```h
Switch# show vlan brief
```

### Configuration des ports : Trunk et Access

#### Mode Trunk

```h
Switch(config)# interface GigabitEthernet1/0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20
Switch# show interfaces trunk
Switch# show running-config
```

#### Mode Access

```h
Switch(config)# interface GigabitEthernet1/0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch# show interfaces status
Switch# show running-config

```

#### Stack

```h
Switch# show version              
Switch# show switch               
Switch(config)# switch <num> priority 15   
Switch# show switch stack-ports   
Switch# write memory                        
```

#### LACP et EtherChannel

```h
Switch# show etherchannel summary             
Switch(config)# interface range GigabitEthernet1/0/1 - 2  
Switch(config-if-range)# channel-group 1 mode active       
Switch(config)# interface Port-channel 1       
Switch(config-if)# switchport mode trunk       
Switch(config-if)# switchport trunk allowed vlan 10,20  
Switch# show running-config                   
Switch# show interfaces status                
Switch# show etherchannel detail              
```

---

#
