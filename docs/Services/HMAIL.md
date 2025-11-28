# HMAIL

## 1. Prérequis

---

#### VM Windows 10 ou 11 Prête à l'emplois

### Installation de HMAIL

```
Aller sur le site officiel :
https://www.hmailserver.com/download
```

---

### Installation de Microsoft .NET Framework 2.0 Service Pack 2 (x64)
>
> Car un message d'erreur va arriver nous disant qu'il faut installer ce fichier  

```
https://www.microsoft.com/en-us/download/details.aspx?id=9834
```

---

---

### 2. Exécutez le fichier .exe

```
Acceptez la licence
Choisissez : Full installation 
```

```
Moteur de base de données :

Use built-in database (simple)
ou External database (si vous utilisez MySQL / MariaDB)

```

```
Définissez un mot de passe d’administration pour hMailServer.

Terminez l’installation.
```

>Ouvrez hMailServer Administrator et connecter vous

---

### 2.1 Configuration du domaine

- Ajouter un domaine

- Dans le panneau de gauche, cliquez sur Domains

- Cliquez sur Add

- Entrez votre domaine (exemples) :
- ```orleans.sportludique.fr``` ou ```[mon_domaine_.com]```

#### Puis mettre un domaine par défaut

- Settings

- Advanced -> Default domain : ```orleans.sportludique.fr``` ou ```[mon_domaine_.com]```

### 2.2 Création d’un compte e-mail

---

- Dans Domains → orleans.sportludique.fr →

- Accounts --  Add…

Entrez le nom de boîte (ex : contact)

Cela donne : <contact@orleans.sportludique.fr>

Définissez un mot de passe

---

## 3. sécurité

## 3.1 Créer les règles de trafic entrant (inbound)
>
>Sur le Pare-feu windows

    pour y acceder dans le cmd:  wf.msc
                                        

> Ajouter des règles pour autoriser les ports des protocoles IMAP SMTP et leur version sécurisé

- Port TCP 143 (Non sécurisé - IMAP)
- Port TCP 993 (Sécurisé - IMAPS)
- Port TCP 25 (Non sécurisé- SMTP)
- Port TCP 587 (STARTTLS)

Dans la colonne de gauche, cliquer sur Inbound Rules

    - À droite → New Rule…
    - Choisir Port  
    - Choisir TCP
    - Select Specific local ports
    - Entrer les ports voulus : 25,143,587,993
    - Choisir Allow the connection
    - Cocher les trois profils : Domain, Private, Public (ou adapter selon votre réseau)
    - Donner un nom : autoriser écoute sur les port IMAP et SMTP et version secure

## 3.2 Créer les règles de trafic sortant (Outbound)

>Faire la même manipulation mais mettre seulement les ports 25 et 587 car seul le protocole SMTP doit sortir

## 3.3 Créer une route statique (Windows)
>
> Pour aller dans le réseau LAN et pouvoir communiquer

```
route add -p [réseau_à_joindre] MASK [masque] [passerelle]
```

> -p pour la rendre permanent

#### pour voir les routes

    route print
---

## 4. Configuration DNS

#### Quel est le processus d'interrogation d'un enregistrement MX ?

- Le logiciel *d'agent de transfert de messages (MTA)* est responsable de l'interrogation des enregistrements MX.
- Lorsqu'un utilisateur *envoie un e-mail*, le MTA envoie une requête DNS pour identifier les serveurs de messagerie des destinataires de l'e-mail.
- Le MTA établit une connexion SMTP avec ces serveurs de messagerie, en commençant par les domaines prioritaires

#### 4.1 configurtion /etc/bind/db.orleans.sp.fr.externe

 ```
 ;
 ; BIND data file for local loopback interface
 ;
 $TTL    604800
 @       IN      SOA   ns1.orleans.sportludique.fr. root.orleans.sportludique.fr. (
                     2025150938         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
 ;
 @       IN      NS      ns1.orleans.sportludique.fr.
 @       IN      NS      ns2.orleans.sportludique.fr.

 @       IN      A       183.44.45.1
 ns1     IN      A       183.44.45.1
 ns2     IN      A       183.44.45.1
 www     IN      A       183.44.45.1
 wp      IN      A       183.44.45.1
 smtp    IN      A       183.44.45.1
        IN      MX      10      smtp.orleans.sportludique.fr
```

- on créer un enregistrement nommé smtp qui pointe vers le routeur extérieur
- on créer un enregistrement MX priorité 10 qui pointe

#### 4.2 configurtion /etc/bind/db.orleans.sp.fr.interne

 ```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ns1.orleans.sportludique.fr. root.orleans.sportludique.fr. (
                         2025111315   ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.orleans.sportludique.fr.
ns1     IN      A       192.168.45.2
www     IN      A       192.168.45.3
mail    IN      A       192.168.45.7
        IN      MX      10      smtp.orleans.sportludique.fr    
smtp    IN      CNAME   mail
imap    IN      CNAME   mail

 ```

- on créer un enregistrement nommé mail qui pointe vers le serveur mail
- puis deux enregistrement smtp et imap qui pointent vers mail avec le CNAME
- on créer un enregistrement MX priorité 10 qui pointe vers smtp.orleans.sportludique.fr

---

## 5. Lier les Utilisateurs de L'AD  

```

```

```

```

---

---

## 6. Créer un compte de messagerie Thunderbird

```

```

---

---

## 7

```

```

```

```

---

---
