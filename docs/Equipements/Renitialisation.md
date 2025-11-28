# Guide : Réinitialisation et ROMMON sur Routeurs Cisco

### Conditions préalables

    Accès privilégié (enable mode) sur le routeur.

    Connexion console pour ROMMON ou récupération si le mot de passe est perdu.

    Prendre en compte que ces opérations effacent totalement la configuration, sauf certains paramètres ROMMON (warm-reboot, memory-size iomem).

 Réinitialisation du routeur Cisco

Deux méthodes principales sont disponibles.

## Méthode 1 — Réinitialisation standard (registre 0x2102)

#### 1.1 Vérifier le registre de configuration

```
Router# show version
```

>La ligne doit indiquer : ```Configuration register is 0x2102```

Si nécessaire, corriger :

```

Router# configure terminal
Router(config)# config-register 0x2102
Router(config)# end
```

#### 1.2 Effacer la configuration

```


Router# write erase
```

#### 1.3 Redémarrer

```

Router# reload
System configuration has been modified. Save? [yes/no]: n
```

 > Au redémarrage, répondre no au dialogue :

```

Would you like to enter the initial configuration dialog? [yes/no]:n
```

---

## Méthode 2 — Réinitialisation via registre 0x2142 (ignorer la configuration existante)

#### 2.1 Changer le registre

```

Router# configure terminal
Router(config)# config-register 0x2142
Router(config)# end
```

#### 2.2 Redémarrer

```

Router# reload
System configuration has been modified. Save? [yes/no]: n
```

>Au redémarrage, répondre no au System Configuration Dialog.

#### 2.3 Rétablir le registre standard

```

Router# configure terminal
Router(config)# config-register 0x2102
Router(config)# end
```

#### 2.4 Écraser la configuration existante

```

Router# write memory
```

>Redémarrer à nouveau

```
Router# reload
```

## Le routeur est maintenant complètement restauré aux paramètres d’usine

---

## Méthode 3 - Accéder au mode ROMMON

Le ROMMON est un mode bas niveau pour dépannage, récupération de mot de passe et restauration IOS.

---

### 3.1 Préparer la connexion console

> Câble console : RJ45 → USB / DB9

>Logiciel terminal : PuTTY, TeraTerm, SecureCRT
Paramètres : 9600 baud, 8 bits, pas de parité, 1 bit d’arrêt, pas de contrôle de flux

### 3.2 Redémarrer le routeur

```

Router# reload
```

>ou couper et remettre l’alimentation.
>
### 3.3 Interrompre le boot

> / ! \ Dès l’apparition du message :
Use BREAK or ESC to interrupt boot process

```
Appuyer sur BREAK (Ctrl + Pause/Break) 
ou (Ctrl + Fn + B) 
ou ESC selon le clavier.
```

#### 3.4 Cas particuliers

    Touche BREAK ne fonctionne pas : utiliser le menu “Send Break” de votre terminal.

    Mot de passe perdu et no service password-recovery activé : ROMMON peut être inaccessible.

### 4. Étapes sur Rommon

>L’invite doit apparaître :

```
rommon 1 >
```

#### 4.1 Ignorer la configuration existante

>À l’invite rommon 1 > :

```
rommon 1 > confreg 0x2142
```

> Cette commande indique au routeur de ne pas charger la configuration en NVRAM au prochain boot.

#### 4.2 Redémarrer le routeur depuis ROMMON

```
rommon 2 > reset
```

>Le routeur démarre maintenant sans charger la configuration existante.

#### 4.3 Entrer dans IOS

    Lorsque le System Configuration Dialog apparaît :

Would you like to enter the initial configuration dialog? [yes/no]:

    Répondre : no

    Tu arrives sur le prompt IOS (Router>).

#### 4.4 Rétablir le registre normal

En mode privilégié :

```
Router> enable
Router# configure terminal
Router(config)# config-register 0x2102
Router(config)# end
```

> /!\ Cela permet au routeur de booter normalement au prochain redémarrage.

#### 4.5 Supprimer définitivement l’ancienne configuration

```
Router# write memory
```

> Cette commande écrase la configuration stockée en NVRAM avec la configuration actuelle (qui est vide).

#### 4.6 Redémarrer le routeur

```
Router# reload
```

> /!\ À l’invite du System Configuration Dialog, répondre no.

## Vérifications après réinitialisation

### Vérifier la configuration courante

```
Router# show running-config
```

> /!\ Doit être minimale (pas d’IP, pas de hostname, pas de configuration spécifique).

### Vérifier le registre

```
Router# show version
```

>Doit afficher : ```Configuration register is 0x2102```

### Commandes ROMMON utiles

```
Voir le registre: confreg
Modifier le registre: confreg 0x2142
Charger IOS manuellement: boot flash:xxx.bin
Lister fichiers: dir flash
Redémarrer: reset
```
