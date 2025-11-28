OpenSSL

# HTTPS

## Obtenir Openssl

<aside>

```markdown
apt-get install openssl
```

</aside>

---

### Configuration openSSL (gestion RFC 2818)

Ce fichier décrit le comportemant que gère openSSL. C'est dans ce fichier que nous
imposerons les champs SAN (Subject Alternative Names).

<aside>

 sudo nano /etc/ssl/openssl.cnf

```markdown
  [ req ]
distinguished_name = req_distinguished_name
req_extensions = v3_req    #Decommenter cette ligne et trouver la section
...
[ v3_req ]
# Extensions to add to a certificate request
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
#########    Ajouter la ligne suivante:
subjectAltName = @alt_names    # le @alt_names est un pointeur vers la section ci dessous
[ alt_names ]
DNS.1 = example.com             #Saisir le nom du domaine concerné (vous remarquez qu'il n'y a pas l'hôte associé)
DNS.2 = www.example.com         #saisir le nom FQDN de votre site
```

</aside>

Malheuresement, ces champs ne peuvent être renseignées par l'utilisateur de manière interactive.
Il faudra donc penser à faire cette manipulation pour chaque site que pour lesquel vous souhaitez générer un certificat.

---

---

Pour notre certificat nous allons creer une autorité de certification qui va signer nos certificats

Il faut donc une deuxième VM nous pouvons aussi le faire sur la meme VM mais pour dissocier les deux nous avons le serveur apache et une autre VM Autorité de certification

1. Créer un dossier de travail (orlssl) dans les VM avec l'arborescence suivante afin de s'organiser un minimum. Vérifiez avec la commande tree à installer si vous ne l'avez pas déjà fait :

<aside>

VM Certificats d’autorité :

```
├── autorite
    ├── certificats
    └── keys
```

</aside>

<aside>

VM Serveur Apache :

```markdown
 ├─ siteweb
       ├── certificats
       ├── keys
       └── requests_certificats
```

</aside>

---

# ServeurWeb

### Génération de la clé privée du serveur web

<div style="border:1px solid #444; border-radius:6px; padding:1em; background:#1e1e1e; color:#eee;">
  <div style="display:flex; gap:1em; border-bottom:1px solid #444; margin-bottom:1em;">
    <button onclick="showTab('tab1')" id="btn1" style="background:#444; color:#fff; border:none; padding:0.5em 1em; border-radius:4px;">RSA</button>
    <button onclick="showTab('tab2')" id="btn2" style="background:#222; color:#aaa; border:none; padding:0.5em 1em; border-radius:4px;">Courbe eliptique</button>
  </div>

  <pre id="tab1" style="display:block; background:#2d2d2d; padding:1em; border-radius:6px;"><code>openssl genrsa 2048 > siteweb/keys/privatekey.key</code></pre>

  <pre id="tab2" style="display:none; background:#2d2d2d; padding:1em; border-radius:6px;"><code>openssl ecparam -genkey -name prime256v1 -out privatekey.key</code></pre>
</div>

1. Vérifier le contenu du fichier généré avec la commande `cat` pour les clés RSA ou pour les **courbes eliptique**

```
openssl ec -in privatekey.key -text -noout

```

Génération d'une demande de certificat pour le serveur web.

La demande de certificat est généré avec openssl via la commande
suivante :

Information très importante

Normallement les modifications dans le fichier de conf d'open SSL sont prise en compte mais on peut forcer leur utilisation

<aside>

<div style="border:1px solid #444; border-radius:6px; padding:1em; background:#1e1e1e; color:#eee;">
  <div style="display:flex; gap:1em; border-bottom:1px solid #444; margin-bottom:1em;">
    <button onclick="showTab('tab1')" id="btn1" style="background:#444; color:#fff; border:none; padding:0.5em 1em; border-radius:4px;">classique</button>
    <button onclick="showTab('tab2')" id="btn2" style="background:#222; color:#aaa; border:none; padding:0.5em 1em; border-radius:4px;">Forcer l’usage du fichier de conf</button>
  </div>

  <pre id="tab1" style="display:block; background:#2d2d2d; padding:1em; border-radius:6px;"><code>openssl req -new -key siteweb/keys/privatekey.key > siteweb/requests_certificats/demande.csr</code></pre>

  <pre id="tab2" style="display:none; background:#2d2d2d; padding:1em; border-radius:6px;"><code>openssl req -new -key siteweb/keys/privatekey.key > siteweb/requests_certificats/demande.csr -config /etc/ssl/openssl.cnf</code></pre>
</div>

</aside>

Le système va vous demander de saisir des champs remplissez-les en adaptant sauf le champ

<aside>

**Common Name** qui doit correspondre
exactement au nom de domaine utilisé correspondant à la directive
**Server Name** défini dans le VHost d'apache. (ex :
<www.orleans.sportludique.fr> )

```markdown
Country Name (2 letter code) [AU]:FR
State or Province Name (full name) [Some-State]:Centre-Val-De-Loire
Locality Name (eg, city) []:Orleans
Organization Name (eg, company) [Internet Widgits Pty Ltd]:orleans-sportludique                                                    
Organizational Unit Name (eg, section) []:orleans-sportludique
Common Name (e.g. server FQDN or YOUR name) []:www.orleans.sportludique.fr
Email Address []:
```

</aside>

<aside>

Ce n'est pas la peine de saisir d'autres "extra attributes"...

```markdown
A challenge password [] :
AN optional company name [] :
```

</aside>

> Ce qui permet de générer votre **CSR certificat**. Celle-ci est créée
dans le format PEM encodé en base64, PEM (*Privacy Enhanced Mail*) étant
le format par défaut pour OpenSSL (il s'agit d'un fichier DER encodé en
ASCII et entouré de balises de marquage).
>

<script>
function showTab(tab) {
  document.getElementById('tab1').style.display = tab === 'tab1' ? 'block' : 'none';
  document.getElementById('tab2').style.display = tab === 'tab2' ? 'block' : 'none';
  document.getElementById('btn1').style.background = tab === 'tab1' ? '#444' : '#222';
  document.getElementById('btn2').style.background = tab === 'tab2' ? '#444' : '#222';
}
</script>

---

---

## Format PEM

<aside>

```markdown
cat file1.crt file2.key > file3.pem
```

par exemple :

```markdown
cat ca-orleans.sp.crt > ca-orleans.sp.pem
```

</aside>

---

Deux choix s'offrent désormais à nous :

- envoyer le fichier demande.csr à un organisme (le tiers de confiance ou **l\'autorité de certification (CA)**) et ainsi obtenir le certificat dûment signé par la clé privée de l'organisme (après avoir payé),
- **ou bien signer nous-même notre certificat avec une autorité que nous allons créer pour l'occasion.** Elle ne sera évidement pas reconnu comme étant de confiance. Cela ne peut donc pas être mis en production dans le monde réel.

Ici nous allons utiliser notre deuxième VM comme autorité de certification

1. Verifier la présence des champs Subject Alternatives Names

```markdown
openssl req -in siteweb/requests_certificats/demande.csr -noout -text
```

![](https://lmeryfulbert.github.io/SportLudique2025-2026/medias/cours/openssl/SAN.png)

---

---

## VM Autorité de certification

## Création du certificat de l'autorité de certification

Pour signer un certificat, vous devez devenir votre propre autorité de
certification, cela implique donc de posséder une clé privée et un
certificat auto-signé.

1. La création de la clé privée de l\'autorité de certification se fait
comme précédemment :

    <aside>

    ```markdown
    openssl genrsa -des3 2048 > autorite/keys/private_ca.key
    ```

    </aside>

Attention ne pas mélanger le fichier correspondant à notre serveur et
ceux correspondant à l'autorité de certification.

2. Ensuite, à partir de la clé privée, on crée un certificat d’autorité (ca.crt) x509 pour une
durée de validité d'un an auto-signé :

```
openssl req -new -x509 -days 365 -key /autorite/keys/private_ca.key > autorite/certificats/ca.crt
```

## Traitement de la demande de certificat de notre serveur par l'autorité de certification fictive

1. La commande qui signe la demande de certificat est la suivante :

<aside>

<div style="border:1px solid #444; border-radius:6px; padding:1em; background:#1e1e1e; color:#eee;">
  <div style="display:flex; gap:1em; border-bottom:1px solid #444; margin-bottom:1em;">
    <button onclick="showTab('tab1')" id="btn1" style="background:#444; color:#fff; border:none; padding:0.5em 1em; border-radius:4px;">classique </button>
    <button onclick="showTab('tab2')" id="btn2" style="background:#222; color:#aaa; border:none; padding:0.5em 1em; border-radius:4px;">Dans notre cas</button>
  </div>

  <pre id="tab1" style="display:block; background:#2d2d2d; padding:1em; border-radius:6px;"><code>openssl x509 -req -in [demande].csr -out [certificat_signe].crt -CA [certificat autorite].crt -CAkey [privatekey_ca].key -CAcreateserial -CAserial ca.srl</code></pre>

  <pre id="tab2" style="display:none; background:#2d2d2d; padding:1em; border-radius:6px;"><code>openssl x509 -req -in [demande-orleans].csr -out [ca-orleans.sp].crt -CA [ca].crt -CAkey [privatekey_ca].key -CAcreateserial -CAserial ca.srl</code></pre>
</div>

</aside>

<aside>

### Forcer l'usage des champs Subject Alternatives Names (SAN)

<div style="border:1px solid #444; border-radius:6px; padding:1em; background:#1e1e1e; color:#eee;">
  <div style="display:flex; gap:1em; border-bottom:1px solid #444; margin-bottom:1em;">
    <button onclick="showTab('tab1')" id="btn1" style="background:#444; color:#fff; border:none; padding:0.5em 1em; border-radius:4px;">classique </button>
    <button onclick="showTab('tab2')" id="btn2" style="background:#222; color:#aaa; border:none; padding:0.5em 1em; border-radius:4px;">Dans notre cas</button>
  </div>

  <pre id="tab1" style="display:block; background:#2d2d2d; padding:1em; border-radius:6px;"><code>oopenssl x509 -req -in [demande].csr -out [certificat_signe].crt -CA [certif_autorite].crt -CAkey [privatekey_ca].key -CAcreateserial -CAserial ca.srl -extfile /etc/ssl/openssl.cnf -extensions v3_req</code></pre>

  <pre id="tab2" style="display:none; background:#2d2d2d; padding:1em; border-radius:6px;"><code>openssl x509 -req -in [demande-orleans].csr -out [ca-orleans.sp].crt
-CA [ca].crt -CAkey [privatekey_ca].key -CAcreateserial
-CAserial ca.srl -extfile /etc/ssl/openssl.cnf -extensions v3_req</code></pre>
</div>

</aside>

---

---

## Envois de Fichier d’un post à un autre

<aside>

<div style="border:1px solid #444; border-radius:6px; padding:1em; background:#1e1e1e; color:#eee;">
  <div style="display:flex; gap:1em; border-bottom:1px solid #444; margin-bottom:1em;">
    <button onclick="showTab('tab1')" id="btn1" style="background:#444; color:#fff; border:none; padding:0.5em 1em; border-radius:4px;">classique </button>
    <button onclick="showTab('tab2')" id="btn2" style="background:#222; color:#aaa; border:none; padding:0.5em 1em; border-radius:4px;">Dans notre cas</button>
  </div>

  <pre id="tab1" style="display:block; background:#2d2d2d; padding:1em; border-radius:6px;"><code>scp /chemin/du/fichier/correspondant Utilisateur@adresse_ip_destinataire:chemin/du/fichier/destination</code></pre>

  <pre id="tab2" style="display:none; background:#2d2d2d; padding:1em; border-radius:6px;"><code> scp orlssl/autorite/certs/ca-orleans.sp.crt etudiant@192.168.140.105:/etc/ssl/cert</code></pre>
</div>

</aside>

---

---

### Serveur Apache2

## Fichier de conf

<aside>

sudo nano /etc/apache2/sites-available/www.orleans.sportludique.fr-secure.conf

```markdown
<VirtualHost *:80>
ServerName [www.orleans.sportludique.fr](http://www.orleans.sportludique.fr/)
ServerAlias [orleans.sportludique.fr](http://orleans.sportludique.fr/)
# Redirection permanente vers HTTPS
Redirect permanent / <https://www.orleans.sportludique.fr>
</VirtualHost>

<VirtualHost *:443>
 ServerName www.orleans.sportludique.fr
 ServerAlias www.orleans.sportludique.fr
 DocumentRoot /var/www/siteorl
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    SSLEngine on

      #  SSLCertificateFile directive is needed.
        SSLCertificateFile      /etc/ssl/certs/ca-orleans.sp.crt
        SSLCertificateKeyFile   /etc/ssl/private/privatekey-orleans.key
        SSLCACertificateFile /etc/ssl/certs/ca.crt

        <Directory /var/www/siteorl>
             Options -Indexes +FollowSymLinks
             AllowOverride All  
        </Directory>

</VirtualHost>
```

</aside>
---

## Activer le site web

<aside>

```markdown
sudo a2ensite www.orleans.sportludique.fr-secure.conf
```

</aside>

## Activer SSL

<aside>

```markdown
sudo a2enmod ssl
```

</aside>

## Restart apache2

<aside>

```markdown
sudo systemctl restart apache2
```

</aside>

---

<aside>

```markdown
sudo cp /etc/ssl/certs/ca.crt /var/www/siteorl
```

</aside>

> /!\ Sur les routeurs penser à rediriger le port 443 sur le serveur web comme fait precedement avec le port 80
>

---

## Navigateur

<aside>

```markdown
on importe le fichier ca.crt 

l’importer dans le navigateur 
Exemple sur firefox "Paramètre" → rechercher "certificats" afficher les certificats "importer" "ca.crt"
```

</aside>
