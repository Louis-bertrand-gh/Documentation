# Apache2

## Met à jour la liste des paquets disponibles

```markdown
sudo apt update
```

## Met à jour les paquets installés (optionnel, mais recommandé)

```markdown
sudo apt upgrade -y
 ```

## Installation d'Apache2

 ```
sudo apt install apache2 -y
```

---

---

## Les Vhost

Sous Linux la configuration des Vhosts se fait :

```
/etc/apache2/sites-available/   → contient tous les fichiers de configuration des sites

Fichier : /etc/apache2/sites-available/mon_site.conf  dans notre cas → /etc/apache2/sites-available/www.orleans.sp.fr.conf 

---------------------------------------------------------------------------------

/etc/apache2/sites-enabled/     → contient les sites actuellement activés (liens symboliques pointant vers sites-available )

Fichier : /etc/apache2/sites-enabled/mon_site.conf  dans notre cas → /etc/apache2/sites-enabled/www.orleans.sp.fr.conf
```

### Fichier etc/apache2/sites-availabled/www.orleans.sp.fr.conf

```markdown
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        ServerName www.orleans.sportludique.fr
        ServerAlias www.orleans.sportludique.fr
        DocumentRoot /var/www/siteorl

        <Directory /var/www/siteorl>
            Options Indexes FollowSymLinks
            AllowOverride All
           # Require all granted
    </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf      
</VirtualHost>

```

### Activer un vhost une fois le fichier créé

```markdown
sudo a2ensite mon_site.conf [ici www.orleans.sp.fr.conf]
sudo systemctl reload apache2
```

---

---

## Crée des pages pour <www.orleans.sportludique.com>

```markdown
sudo mkdir /var/www/siteorl
sudo mkdir /var/www/siteorl/html
sudo touch /var/www/siteorl/html/index.html
          ou dans notre cas
sudo touch /var/www/siteorl/index.html
```

```markdown
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Accueil - SportLudique</title>
  <style>
    /* Style de base */
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background-color: #000; /* fond sombre */
      color: white;
      text-align: center;
    }

    /* Bandeau supérieur */
    header {
      background: linear-gradient(90deg, #0B7A2B, #10A339, #00C27A);
      padding: 15px;
      font-size: 1.2em;
      font-weight: bold;
    }

    /* Contenu principal */
    main {
      padding: 60px 20px;
    }

    a {
      display: inline-block;
      background-color: #10A339;
      color: white;
      text-decoration: none;
      padding: 10px 20px;
      border-radius: 6px;
      margin-top: 20px;
      transition: background 0.3s;
    }

    a:hover {
      background-color: #0B7A2B;
    }
  </style>
</head>
<body>

  <header>🏋️‍♂️ SportLudique Project 2025-2026</header>

  <main>
    <h1>Bienvenue sur le projet SportLudique</h1>
    <p>Projet collaboratif des étudiants du BTS SIO – spécialité SISR</p>
    <a href="https://lmeryfulbert.github.io/SportLudique2025-2026/" target="_blank">
      Voir la Documentation
    </a>
  </main>

</body>
</html>
```

---

---

## Créer une route statique (interne) pour accéder au réseaux lan

<aside>

sudo nano /etc/network/interfaces :

```markdown

auto lo
iface lo inet loopback

allow-hotplug ens3
iface ens3 inet static
        address 192.168.45.3/24
        gateway 192.168.45.254
        dns-nameservers 172.28.120.2
        dns-search orl.orleans.sportludique.fr
        up ip route add 172.28.96.0/19 via 192.168.45.1 dev ens3

allow-hotplug ens4
iface ens4 inet static
        address 192.168.140.105/24
```

```markdown
 up ip route add 172.28.96.0/19 via 192.168.45.1 dev ens3
 
 Cela indique : Pour atteindre le réseau 172.28.96.0/19 , passer par la passerelle 192.168.45.1 , en utilisant l’interface ens3 .
 Cela permet d'acceder au site depuis le réseau lan 
```

</aside>

---

## Sur les routeurs
>
> /!\ Sur les routeurs penser à rediriger le port 80 sur le serveur web NAT/PAT
>
## Sur le serveur DNS
>
>/!\ Mettre enregistrement dns : www IN A 192.168.45.3

>

import React, { useState } from 'react';

export default function CryptoKeyGenerator() {
  const [activeAlgo, setActiveAlgo] = useState('RSA');
  const [command, setCommand] = useState('openssl genrsa 2048 > siteweb/keys/privateKey.key');

  // Configuration des algorithmes et leurs commandes
  const algorithms = [
    {
      name: 'RSA',
      command: 'openssl genrsa 2048 > siteweb/keys/privateKey.key'
    },
    {
      name: 'Courbe elliptique',
      command: 'openssl ecparam -genkey -name secp256k1 -out siteweb/keys/privateKey.key'
    },
    {
      name: 'DSA',
      command: 'openssl dsaparam -genkey 2048 | openssl dsa -out siteweb/keys/privateKey.key'
    },
    {
      name: 'Ed25519',
      command: 'openssl genpkey -algorithm ED25519 -out siteweb/keys/privateKey.key'
    }
  ];

  const handleAlgoClick = (algo) => {
    setActiveAlgo(algo.name);
    setCommand(algo.command);
  };

  const copyToClipboard = () => {
    navigator.clipboard.writeText(command);
  };

  return (
    <div className="min-h-screen bg-gray-900 flex items-center justify-center p-8">
      <div className="w-full max-w-3xl bg-gray-800 rounded-lg shadow-2xl p-8">
        <h1 className="text-2xl font-bold text-white mb-6">Générateur de clés cryptographiques</h1>

        {/* Boutons des algorithmes */}
        <div className="flex flex-wrap gap-3 mb-6">
          {algorithms.map((algo) => (
            <button
              key={algo.name}
              onClick={() => handleAlgoClick(algo)}
              className={`px-6 py-2 rounded-lg font-medium transition-all duration-200 ${
                activeAlgo === algo.name
                  ? 'bg-blue-600 text-white shadow-lg'
                  : 'bg-gray-700 text-gray-300 hover:bg-gray-600'
              }`}
            >
              {algo.name}
            </button>
          ))}
        </div>

        {/* Zone de commande */}
        <div className="bg-gray-900 rounded-lg p-6 border border-gray-700">
          <div className="flex items-center justify-between mb-2">
            <span className="text-gray-400 text-sm font-mono">Terminal</span>
            <button
              onClick={copyToClipboard}
              className="px-3 py-1 bg-gray-700 hover:bg-gray-600 text-gray-300 rounded text-sm transition-colors"
            >
              Copier
            </button>
          </div>
          <div className="bg-black rounded p-4">
            <code className="text-green-400 font-mono text-sm break-all">
              {command}
            </code>
          </div>
        </div>

        {/* Instructions */}
        <div className="mt-6 text-gray-400 text-sm">
          <p>• Cliquez sur un algorithme pour générer la commande correspondante</p>
          <p>• Utilisez le bouton "Copier" pour copier la commande dans le presse-papiers</p>
          <p>• La commande génère une clé privée dans le dossier spécifié</p>
        </div>
      </div>
    </div>
  );
}
