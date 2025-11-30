# Base de Données

## Installation de MariaDB

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install mariadb-server -y

sudo systemctl status mariadb
```

## Création de la base de données et de l’utilisateur

```bash
sudo mysql -u root -p

CREATE DATABASE Wordpress;

CREATE USER 'Nom-utilisateur'@'Adresse_IP_du_serveur_web' IDENTIFIED BY 'Mots_de_passe';

GRANT ALL PRIVILEGES ON *Wordpress.* TO 'Nom-utilisateur'@'Adresse_IP_du_serveur_web';

FLUSH PRIVILEGES;
```

## Modification de la bind-address

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Modifier bind-address = 127.0.0.1 par bind-address = 0.0.0.0

## Redémarrer le service MariaDB

```bash
sudo systemctl restart mariadb
```
