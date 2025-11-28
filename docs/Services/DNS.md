DNS:
> Linux → bind9
>

### Installation

```markdown
sudo apt install -y bind9 bind9utils bind9-dnsutils
```

### Redirecteur

### **Configuration BIND (zone principale pour exemple.com) :**

1. Dans le fichier de zone pour `orleans.sportludique.fr` ou `exemple.com`, ajoutez les enregistrements NS pour le sous-domaine délégué comme ceci :

Informations lié à notre cas
Informations à changer en fonction de la situation

```markdown
; Fichier de zone pour exemple.com (zone publique)
$TTL 3600
@      IN      SOA    dns.orleans.sportludique.fr. ns1.exemple.com. admin.exemple.com. (
                  2023091301 ; Numéro de série
                  3600       ; Intervalle de rafraîchissement
                  1800       ; Intervalle de réessai
                  604800     ; Intervalle d'expiration
                  86400 )    ; Durée minimale de cache
@      IN      NS     dns.orleans.sportludique.fr. ou ns1.exemple.com.
@      IN      NS     ns2.exemple.com.
; Enregistrements IPv4
@      IN      A      [Adresse ip d'un serveur]
www    IN      A      [Adresse ip du serv web]
mail   IN      A      [Adresse ip du serv mail]
dns    IN      A      192.168.45.2
; Délégation du sous-domaine à un autre serveur DNS
sousdomaine.exemple.com.  IN  NS  ns.serveurdns.externe.com.
```

### Exemple de configuration

## Resolveur DNS

### Fichier /etc/bind/named.conf.options

```markdown
    options {
            directory "/var/cache/bind";
            //Autorise les requetes recursives
            recursion yes;  
            //Liste des réseaux autorisés à interoger le resolver
            //Par defaut seul les équipements du meme réseau IP que le serveur peuvent l\'interroger.
            allow-query { 
                172.16.x.0/24;       //LAN
                127.0.0.1;          //LOCALHOST
                192.168.x.0/24;    //DMZ
            };
            //Desactivation de DNSSec
            dnssec-validation no;
            //Ecoute sur l\'ensemble des interfaces IPv4
            listen-on { any; };
    };
```

### Fichier /etc/bind/named.conf.local (du resolver)

Les requetes à destination de la zone locale doivent être redirigées
vers le serveur DNS de la DMZ en interogeant la vue interne.

```markdown
zone "orleans.sportludique.fr" {
    type forward;
    forwarders { 192.168.x.y };  // Remplace par l'IP du serveur DNS ayant autorité sur la zone (dans la DMZ par exemple)
};
```

Les requetes à destination de la zone de l'entreprise
(sportludique.fr) doivent partir vers le serveur resolver de
l'enseignant gérant cette zone.

```markdown
zone "sportludique.fr" {
    type forward;
    forwarders { 121.183.90.205; };  // IP du serveur de l'enseignant ayant autorité sur la zone sportludique.fr
};
```

Toutes les autres requetes sont recursives et interrogent donc les serveurs racines (connu de Bind).
Cela permet d'éviter une potentielle censure en utilisant le serveur DNS d'un Opérateur (celui du prof :-) )

## DNS-Autorité

>
>

### serveur DNS ayant autorité sur la zone orleans.sportludique.fr

### Fichier /etc/bind/named.conf.default-zones

> Toutes les lignes du fichier `named.conf.default-zones` doivent être mises `en commentaire` car ne elles ne sont associées à aucune vue.
>

```markdown
                                                                                                   
// prime the server with knowledge of the root servers
//zone "." {
//      type hint;
//      file "/usr/share/dns/root.hints";
//};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

//zone "localhost" {
//      type master;
//      file "/etc/bind/db.local";
//};

//zone "127.in-addr.arpa" {
//      type master;
//      file "/etc/bind/db.127";
//};

//zone "0.in-addr.arpa" {
//      type master;
//      file "/etc/bind/db.0";
//};

//zone "255.in-addr.arpa" {
//      type master;
//      file "/etc/bind/db.255";
//};

```

### Fichier /etc/bind/named.conf.local

```
    //zone externe
    view "outside" {
        match-clients {
            !172.x.x.0/24;    //requète de provenant pas du LAN
            !192.168.x.0/24;    //requète de provenant pas de la DMZ
            any;                //Toutes les autres adresses
        };
        zone "orleans.sportludique.fr." {
            type master;
            file "/etc/bind/db.orleans.sp.fr.externe";
        };
    };
    view "inside" {
        match-clients {
            172.x.0.0/24;    //requète provenant du LAN
            192.168.x.0/24;    //requète provenant de la DMZ
        };
        zone "orleans.sportludique.fr." {
            type master;
            file "/etc/bind/db.orleans.sp.fr.interne";
        };
    };

```

<aside>

Une vue (ou view en anglais)
Dans Bind9 est une vue un mécanisme de filtrage et de séparation logique du service DNS. Elle permet à un même serveur DNS de répondre différemment selon l’origine de la requête.

</aside>

### Fichier /etc/bind/db.orleans.sp.fr.interne

```markdown
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ns1.orleans.sportludique.fr. root.orleans.sportludique.fr. (
                         2025101301   ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.orleans.sportludique.fr.
ns1     IN      A       192.168.45.2
www     IN      A       192.168.45.3




```

> @       IN      SOA     [ns1.orleans.sportludique.fr](http://dns.orleans.sportludique.fr/).
>
> - **OA (Start Of Authority)** définit **le serveur DNS maître (primaire)** et **les paramètres d’autorité** de la zone.
> - Il est **obligatoirement unique** dans chaque fichier de zone.
> - En plus du DNS primaire, il indique aussi **l’adresse e-mail de l’administrateur** et les **valeurs de synchronisation** (Serial, Refresh, etc.).

> @       IN      NS      [ns1.orleans.sportludique.fr](http://www.orleans.sportludique.fr/).
>
> - Les enregistrements **NS (Name Server)** désignent les **serveurs DNS autoritaires** pour cette zone.
> - Il peut y en avoir **plusieurs** (un primaire, un ou plusieurs secondaires).

> dns     IN      A       192.168.45.2
>
> - ’enregistrement **A (Address)** associe un **nom d’hôte** (ex. `ns1`) à une **adresse IPv4**.
> - C’est le lien entre **nom → IP**.

> monsite IN      CNAME   www
>
> - Le **CNAME (Canonical Name)** crée un **alias** : il fait pointer un nom vers **un autre nom DNS**, **pas directement vers une IP**.
> - Le résolveur suit ensuite ce second nom pour trouver son enregistrement A.

### Fichier /etc/bind/db.orleans.sp.fr.externe

>
>

```markdown

;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA   ns1.orleans.sportludique.fr. root.orleans.sportludique.fr>
                     2025150938         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.orleans.sportludique.fr.

@       IN      A       183.44.45.1
ns1     IN      A       183.44.45.1
www     IN      A       183.44.45.1


```

## Route interne

```
sudo nano /etc/network/interfaces
up ip route add 172.28.96.0/24 via 192.168.45.1 dev ens5
```

## Pour ajouter des LOG

### Fichier /etc/bind/named.conf.options

```markdown
loggin {
  channel queries_log {
     file "/var/log/named/queries.log" version 3 size 10m;
     severity info;
     print-time yes;
    };
    category queries { queries_log; };
 };
```

Pour lire les log

```markdown
sudo tail /var/log/named/queries.log
```

## Commandes de verification

```markdown
sudo named-checkconf
nslookup dns.orleans.sportludique.fr  
dig dns.orleans.sportludique.fr
```
