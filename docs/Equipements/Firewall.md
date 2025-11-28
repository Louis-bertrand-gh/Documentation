# Active Directory

---

## Organisation des unités (OU)

---

### Création d’une OU pour chaque service disponible

- Production
- Sous-OU :

  - A
  - B

- Conception
- Informatique

---

## Gestion des utilisateurs

---

### Ajout des utilisateurs

- Dans chaque service

- Dans chaque OU

- Dans chaque groupe associé

---

## Liste des Utilisateur

---

| NOM | PRENOM | SERVICE | EQUIPE |
|-----|--------|---------|--------|
| CHEUVAN | Marie Alice | CONCEPTION | |
| PREHEM-TORTA | Pierric | CONCEPTION | |
| QUETEBOEUFS | Esmeralda | CONCEPTION | |
| BRAEUX | Salvatore | CONCEPTION | |
| REPSODE | Yann | CONCEPTION | |
| GHRASSA | Christiane | CONCEPTION | |
| NWONWU | Fatiha | CONCEPTION | |
| BROUERD | Fatiha | CONCEPTION | |
| BEYREMOGLU | Corinne | CONCEPTION | |
| MAOT | Denise | CONCEPTION | |
| LAMONE | John | CONCEPTION | |
| MOROGLU | Farid | CONCEPTION | |
| PEALLET | Nadège | CONCEPTION | |
| DEMECON | tante | CONCEPTION | |
| GOMES EDUERDO | Yvan | PRODUCTION | A |
| BENOAT | Vincent | PRODUCTION | A |
| NEAL | Erminda | PRODUCTION | A |
| SEBEE | Vincent | PRODUCTION | A |
| DEUPHAN | Laurence | PRODUCTION | A |
| HEMEDE | Yamina | PRODUCTION | A |
| DERON | Jérôme | PRODUCTION | A |
| MERGOT | Myriam | PRODUCTION | A |
| ROUSSEEU | Ali-Osman | PRODUCTION | A |
| CHEMBRAN | Sandrine | PRODUCTION | A |
| DJEDJE YOHOU | Joao Fernando | PRODUCTION | A |
| DAEBY | Virgil | PRODUCTION | A |
| BERBASSOU | Farid | PRODUCTION | A |
| QUETEBOEUFS | Roland | PRODUCTION | A |
| BECF | Khemais | PRODUCTION | A |
| CHRASTEAN | Yamina | PRODUCTION | A |
| DAMBEMBU | Marie Laur | PRODUCTION | A |
| FOFENE | Esther | PRODUCTION | A |
| BECF | Michelle | PRODUCTION | A |
| BOURET | Dalila | PRODUCTION | A |
| DZEBETOU-ECFO | Luis | PRODUCTION | A |
| AGHALEHRAZ | Cedric | PRODUCTION | A |
| PREHEM-TORTA | Viviane | PRODUCTION | A |
| SOLER | Jeanlou | PRODUCTION | A |
| DOUHEBA | Joao | PRODUCTION | A |
| PERASY | Frédérique | PRODUCTION | B |
| GRAGNON | Latha | PRODUCTION | B |
| BREENT MACHEL | Agnes | PRODUCTION | B |
| CHEPASEEU | Pierre Marc | PRODUCTION | B |
| ELAPOUR-EZEDEH | Benamar | PRODUCTION | B |
| RESOEVETSERE | Hélène | PRODUCTION | B |
| ELEXENDRE | Jeanine | PRODUCTION | B |
| DZEFEROVAC EUGUAN | Thérésia | PRODUCTION | B |
| BRAERE | Valerie | PRODUCTION | B |
| DOMANGUES VAEARE | Georges | PRODUCTION | B |
| GONZELVES | Abdelkader | PRODUCTION | B |
| LAVOLSA | Maïmouna | PRODUCTION | B |
| MEZERD | Jean-Eddy | PRODUCTION | B |
| GODOU | Lassaad | PRODUCTION | B |
| BLEASE | Kathia | PRODUCTION | B |
| BAGUEREEU | Marie Ange | PRODUCTION | B |
| LEDEUPHAN | Karine | PRODUCTION | B |
| EME | Marie-Luc | INFORMATIQUE | |
| DORENGE | Marine | INFORMATIQUE | |
| THARENT | Alvaro | INFORMATIQUE | |
| BENTELEB | John | INFORMATIQUE | |

---

## Script powershell utilisé

---

```h
# Script d'importation d'utilisateurs Active Directory depuis CSV

 
# Importer le module Active Directory

Import-Module ActiveDirectory
 
# Définir les chemins de base de votre infrastructure

$domainDN = "DC=orl,DC=orleans,DC=sportludique,DC=fr"

$ouBasePath = "$domainDN"
 
# Définir les OU des services (sans sous-OU)

$ouServices = @{

    "Conception" = "OU=Conception,$ouBasePath"

    "Informatique" = "OU=Informatique,$ouBasePath"

    "Production" = "OU=Production,$ouBasePath"

}
 
# Définir les groupes principaux par service

$groupesServices = @{

    "Conception" = "GRP_CONCEPTION"

    "Informatique" = "GRP_INFORMATIQUE"

    "Production" = "GRP_PRODUCTION"

}
 
# Définir les sous-groupes Production

$sousGroupesProduction = @{

    "A" = "GRP_PRODUCTION_A"

    "B" = "GRP_PRODUCTION_B"

}
 
# Chemin du fichier CSV

$csvPath = ".\orleans.csv"
 
# Vérifier si le fichier CSV existe

if (-not (Test-Path $csvPath)) {

    Write-Host "Erreur : Le fichier $csvPath n'existe pas !" -ForegroundColor Red

    exit

}
 
# Importer les données du CSV

# Format attendu : NOM;PRENOM;SERVICE;SOUS_OU

$utilisateurs = Import-Csv -Path $csvPath -Delimiter ";" -Encoding UTF8
 
Write-Host "============================================" -ForegroundColor Cyan

Write-Host "Import d'utilisateurs dans Active Directory" -ForegroundColor Cyan

Write-Host "============================================" -ForegroundColor Cyan

Write-Host ""
 
foreach ($user in $utilisateurs) {

    $nom = $user.NOM.Trim()

    $prenom = $user.PRENOM.Trim()

    $service = $user.SERVICE.Trim()

    $sousOU = if ($user.SOUS_OU) { $user.SOUS_OU.Trim() } else { "" }

    # Vérifier que le service existe dans notre mapping

    if (-not $ouServices.ContainsKey($service)) {

        Write-Host "⚠ Utilisateur $prenom $nom : Service '$service' non reconnu. Ignoré." -ForegroundColor Yellow

        continue

    }

    # Construire les informations de l'utilisateur

    $samAccountName = ($prenom.Substring(0,1) + $nom).ToLower()

    $userPrincipalName = "$samAccountName@orl.orleans.sportludique.fr"

    $displayName = "$prenom $nom"

    # Déterminer le chemin OU (avec ou sans sous-OU)

    if ($sousOU -ne "") {

        $ouPath = "OU=$sousOU," + $ouServices[$service]

    }

    else {

        $ouPath = $ouServices[$service]

    }

    # Mot de passe par défaut (à changer lors de la première connexion)

    $defaultPassword = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

    try {

        # Vérifier si l'utilisateur existe déjà

        $existingUser = Get-ADUser -Filter "SamAccountName -eq '$samAccountName'" -ErrorAction SilentlyContinue

        if ($existingUser) {

            Write-Host "⚠ Utilisateur $samAccountName existe déjà. Ignoré." -ForegroundColor Yellow

        }

        else {

            # Créer l'utilisateur

            New-ADUser `

                -Name $displayName `

                -GivenName $prenom `

                -Surname $nom `

                -SamAccountName $samAccountName `

                -UserPrincipalName $userPrincipalName `

                -Path $ouPath `

                -AccountPassword $defaultPassword `

                -Enabled $true `

                -ChangePasswordAtLogon $true `

                -Department $service `

                -Description "Utilisateur du service $service"

            Write-Host "✓ Utilisateur créé : $displayName ($samAccountName)" -ForegroundColor Green

            # Ajouter au groupe principal du service

            $groupePrincipal = $groupesServices[$service]

            $groupePrincipalDN = "CN=$groupePrincipal," + $ouServices[$service]

            try {

                Add-ADGroupMember -Identity $groupePrincipalDN -Members $samAccountName -ErrorAction Stop

                Write-Host "  └─ Ajouté au groupe : $groupePrincipal" -ForegroundColor Cyan

            }

            catch {

                Write-Host "  └─ ⚠ Erreur ajout au groupe $groupePrincipal : $($_.Exception.Message)" -ForegroundColor Yellow

            }

            # Si service Production, ajouter aussi au sous-groupe

            if ($service -eq "Production" -and $sousOU -ne "") {

                if ($sousGroupesProduction.ContainsKey($sousOU)) {

                    $sousGroupe = $sousGroupesProduction[$sousOU]

                    $sousGroupeDN = "CN=$sousGroupe," + $ouServices[$service]

                    try {

                        Add-ADGroupMember -Identity $sousGroupeDN -Members $samAccountName -ErrorAction Stop

                        Write-Host "  └─ Ajouté au sous-groupe : $sousGroupe" -ForegroundColor Cyan

                    }

                    catch {

                        Write-Host "  └─ ⚠ Erreur ajout au sous-groupe $sousGroupe : $($_.Exception.Message)" -ForegroundColor Yellow

                    }

                }

            }

        }

    }

    catch {

        Write-Host "✗ Erreur lors de la création de $displayName : $($_.Exception.Message)" -ForegroundColor Red

    }

}
 
Write-Host ""

Write-Host "============================================" -ForegroundColor Cyan

Write-Host "Import terminé" -ForegroundColor Cyan

Write-Host "============================================" -ForegroundColor Cyan
```

# Active Directory Secondaire

## Étape 1 : Installation du Rôle AD DS

1. Ouvrez le **Gestionnaire de serveur**
2. Cliquez sur **Gérer** → **Ajouter des rôles et fonctionnalités**
3. Assistant d'ajout de rôles :
   - Page **Avant de commencer** : Cliquez sur **Suivant**
   - **Type d'installation** : Sélectionnez **Installation basée sur un rôle ou une fonctionnalité** → **Suivant**
   - **Sélection du serveur** : Choisissez votre serveur → **Suivant**
   - **Rôles de serveurs** : Cochez **Services AD DS (Active Directory Domain Services)**
   - Une fenêtre popup apparaît, cliquez sur **Ajouter des fonctionnalités** → **Suivant**
   - **Fonctionnalités** : Laissez par défaut → **Suivant**
   - **AD DS** : Lisez les informations → **Suivant**
   - **Confirmation** : Cliquez sur **Installer**
4. Attendez la fin de l'installation (ne fermez pas la fenêtre)

## Étape 2 : Promotion en Contrôleur de Domaine

1. À la fin de l'installation, cliquez sur **Promouvoir ce serveur en contrôleur de domaine** (lien avec drapeau jaune dans le Gestionnaire de serveur)

### Configuration du déploiement

- Sélectionnez **Ajouter un contrôleur de domaine à un domaine existant**
- Domaine : Vérifiez que votre domaine est affiché (ex : entreprise.local)
- Cliquez sur **Modifier** pour fournir les identifiants d'administrateur du domaine si nécessaire
- Cliquez sur **Suivant**

### Options du contrôleur de domaine

- Niveau fonctionnel : Laissez la valeur par défaut
- Cochez **Serveur DNS** (recommandé)
- Cochez **Catalogue global (GC)** (recommandé pour la redondance)
- Entrez un **mot de passe DSRM** (mode restauration des services d'annuaire) - À conserver précieusement !
- Cliquez sur **Suivant**

### Options DNS

- Ignorez l'avertissement éventuel sur la délégation DNS
- Cliquez sur **Suivant**

### Options supplémentaires

- Le système détecte automatiquement le DC source pour la réplication
- Vérifiez que le bon DC est sélectionné
- Cliquez sur **Suivant**

### Vérification de la configuration requise

- L'assistant effectue une vérification complète
- Lisez les avertissements éventuels (certains sont normaux)
- Si tout est OK, cliquez sur **Installer**

Le serveur redémarrera automatiquement après l'installation.

## Étape 3 : Vérification Post-Installation

Après le redémarrage, connectez-vous avec un compte administrateur du domaine.

### Vérification dans le Gestionnaire de serveur

1. Ouvrez le **Gestionnaire de serveur**
2. Vérifiez que le rôle **AD DS** est affiché
3. Aucune erreur ne devrait apparaître

### Vérification de la réplication

1. Ouvrez **Outils** → **Sites et services Active Directory**
2. Naviguez vers : **Sites** → **Default-First-Site-Name** → **Servers** → Votre nouveau DC
3. Vérifiez la présence des connexions de réplication

### Test avec PowerShell

Ouvrez PowerShell en tant qu'administrateur et exécutez :

```powershell

# Vérifier la réplication
repadmin /replsummary

```

## Configuration DNS Recommandée

1. Sur le DC principal, ouvrez **Gestionnaire DNS**
2. Ajoutez l'adresse IP du nouveau DC comme serveur DNS secondaire
3. Sur les clients, configurez le nouveau DC comme DNS secondaire

## Sauvegarde du Mot de Passe DSRM

⚠️ **IMPORTANT** : Conservez le mot de passe DSRM dans un coffre-fort sécurisé. Il est nécessaire pour la restauration en cas de problème grave.
