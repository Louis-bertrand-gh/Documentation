# WordPress

## Installation de MariaDB et des extensions PHP

```bash
sudo apt update

sudo apt install mariadb-server -y

sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y
```

## Instalation de Wordpress

```bash
cd /tmp

wget https://wordpress.org/latest.zip

sudo rm /var/www/html/index.html

sudo apt update 

sudo apt install zip

sudo unzip latest.zip -d /var/www/html

cd /var/www/html

sudo mv wordpress/* /var/www/html/
```

## Configuration de Wordpress

```bash
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

sudo nano /var/www/html/wp-config.php
```

```bash
// ** Réglages MySQL - Ces informations vous sont fournies par votre hébergeur ** //
/** Nom de la base de données de WordPress */
define( 'DB_NAME', 'nom_de_la_base' );

/** Utilisateur MySQL */
define( 'DB_USER', 'nom_utilisateur' );

/** Mot de passe MySQL */
define( 'DB_PASSWORD', 'mot_de_passe' );

/** Adresse de l’hôte MySQL */
define( 'DB_HOST', 'adresse_ip_du_serveur' );

/** Jeu de caractères pour la création des tables. */
define( 'DB_CHARSET', 'utf8' );

/** Type de collation de la base. */
define( 'DB_COLLATE', '' );

/* ... Autres paramètres ... */

/** Méthode d’accès au système de fichiers */
define( 'FS_METHOD', 'direct' );
```

## Redémarrer les services

```bash
sudo systemctl restart apache2
sudo systemctl restart mariadb
```
