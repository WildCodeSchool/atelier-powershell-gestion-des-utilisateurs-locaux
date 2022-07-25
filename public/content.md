# Gestion des utilisateurs locaux

## La gestion en PowerShell

#### Introduction

Les utilisateurs locaux peuvent se gérer de différentes manières. En mode graphiques, mais également en ligne de commande.
C'est le but de cet atelier.

#### 1. Prérequis

* Une VM ou un hôte Windows (à partir de Windows 7 ou à partir de Windows Server 2012). Cet atelier a été fait sous Windows 11.
* La console shell de PowerShell ou bien l’éditeur PowerShell ISE ou bien encore un éditeur comme VSCode.
* ***Attention*** : Si vous ne maitrisez pas votre OS, les modifications de services peuvent amener des dysfonctionnements sur votre ordinateur. Pour plus de sécurité, vous pouvez utiliser une VM.

#### 2. Gestion de la sécurité d'exécution des scripts PowerShell

Dans PowerShell, il existe plusieurs paramètres de stratégie d'exécution des scripts :

* **Default** : paramètre par défaut (équivalent à **Restricted** pour les clients et **RemoteSigned** pour les serveurs)
* **Restricted** : n'autorise pas l'exécution des scripts
* **AllSigned** : n'exécute que les scripts signés
* **RemoteSigned** : exécute les scripts locaux sans obligation de confiance et les scripts d'internet signés
* **Unrestricted** : autorise l'exécution de tous les scripts, ceux en provenance d'internet doivent être autorisés au lancement
* **Bypass** : tous les scripts sont exécutés sans aucun avertissement
* **Undefined** : aucune stratégie d'exécution n'est définie

Ces stratégies d'exécution de scripts ont les étendues suivantes :

* **MachinePolicy** : Défini par une stratégie de groupe pour tous les utilisateurs de l'ordinateur
* **UserPolicy** : Défini par une stratégie de groupe pour l'utilisateur actuel de l'ordinateur
* **Process** : Affecte uniquement la session PowerShell actuelle
* **CurrentUser** : Affecte uniquement l'utilisateur actuel
* **LocalMachine** : Étendue par défaut qui affecte tous les utilisateurs de l'ordinateur.

Démarrer une console PowerShell en tant **qu'administrateur** :

* Commande pour connaître la stratégie en cours :

```powershell
PS C:\Users\wilder> Get-ExecutionPolicy
```

* Commande pour modifier la stratégie :

```powershell
PS C:\Users\wilder> Set-ExecutionPolicy -Scope LocalMachine -ExecutionPolicy Unrestricted
```

***A faire :***

* Tu vas changer la sécurité d'exécution des scripts.
* Ecrit les commandes correspondantes aux 2 cibles suivantes :
  * Mets la stratégie de sécurité d'exécution **Undefined** sur l'étendue **LocalMachine**
  * Mets la stratégie de sécurité d'exécution **bypass** sur l'étendue **CurrentUser**

#### 3. Accès aux comptes locaux du système

* Afficher les comptes locaux:

```powershell
PS C:\Users\wilder> Get-LocalUser
```

* Recherche d'un compte

Ecris le code ci-dessous et exécute-le :

```powershell
#RechercheExistenceCompte.ps1
#$ErrorActionPreference = "SilentlyContinue"
$Nom = Read-Host -Prompt "Saisir un nom de compte local"

If (Get-LocalUser -Name $Nom)
{
    Write-Host "Le compte $Nom existe" -ForegroundColor Green
}
Else
{
    Write-Host "Le compte $Nom n'a pas été trouvé" -ForegroundColor Red
}
#$ErrorActionPreference = "Continue"
```

Que se passe-t'il si tu rentre un compte qui n'existe pas sur le système ?

Decommente les 2 lignes `ErrorActionPreference` et relance le script.

Vois-tu une différence lorsque le compte n'existe pas ?

Tu trouveras des informations supplémentaires sur la variable **$ErrorActionPreference** [ici](https://www.oreilly.com/library/view/professional-windows-powershell/9780471946939/9780471946939_using_the_dollar_erroractionpreference_v.html)

* Recherche d'un compte - Méthode avec `ADSI`

Ecris le code ci-dessous et exécute-le :

```powershell
#RechercheExistenceCompte-MethodeADSI.ps1

#La variable $ErrorActionPreference est initialisée à la valeur par défaut
$ErrorActionPreference = "Continue"
$Nom = Read-Host -Prompt "Saisir un nom de compte local"
$Compte = [ADSI]"WinNT://./$Nom"
If ($Compte.Path)
{
    Write-Host "Le compte $($Compte.FullName) existe" -ForegroundColor Green
}
Else
{
     Write-Host "Le compte $Nom n'a pas été trouvé" -ForegroundColor Red
}
```

Dans ce code, il n'y a pas de gestion d'erreur. Essaye de mettre un compte qui n'existe pas et regarde le résultat.

**Remarque** :

* L’accès à la base locale de comptes utilisateurs d’un système Windows est ici réalisé avec l’instruction : **[ADSI]"WinNT://."**
* Pour la nomenclature :
  * Les majuscules et les minuscules doivent être respectées.
  * Le point `.` représente le nom du système sur lequel est lancée l’instruction, il peut être remplacé par le nom de l’ordinateur cible.
* Il faut des droits d’administration sur l’ordinateur distant dans l'utilisation en remote
* Pour filtrer les éléments de la base de comptes, il est possible de spécifier le nom de l’élément recherché,
  * Ici il est contenu dans la variable $nom : **[ADSI]"WinNT://./$nom"**

Test le script avec un compte inexistant et un compte existant.

Affiche les propriétés de la variable **$compte** :

```powershell
PS C:\Users\wilder> $compte | Get-Member
```

Dans le volet de sortie, on peut voir les methodes et propriétés de la variable **$Compte**.

***A faire :***

* Modifie le script **RechercheExistenceCompte-MethodeADSI.ps1** pour afficher 3 autres propriétés de ton choix.

#### 4. Ajout d’un compte local

Ecrire et exécuter le script suivant :

```powershell
#AjoutCompteLocal.ps1
#Ajoute un compte dans la base local du système
$Local = [ADSI]"WinNT://."
$Nom = Read-Host -Prompt "Saisir un nom de compte local"
$Description = Read-Host -Prompt "Saisir une description"
$Compte = [ADSI]"WinNT://./$Nom"
If (!$Compte.Path)
{
    $Utilisateur = $Local.Create("User",$Nom)
    $Utilisateur.InvokeSet("Description",$Description)
    $Utilisateur.CommitChanges()
    Write-Host "$Nom ajouté"
}
Else
{
    Write-Host "$Nom existe déjà"
}
```

**Remarques** :

* Ici on utilise des méthodes de programmation objet pour gérer les utilisateurs.
* La variable **$local** possède une méthode `Create()` :
  * Format : **Create("User", $Var)** où $Var est une variable contenant un label de nom à créer
  * Cela permet de créer un objet de type **user** (premier paramètre), dont le nom du compte est spécifié par le deuxième paramètre, ici la variable **$nom**.
* On demande d'abord 2 propriétés: le **nom d'utilisateur** qui sera stocké dans la variable $Nom, et la **description de l'utilisateur** qui sera stockée dans **$Description**.
* Le résultat est une variable nommée **$utilisateur**.
* Cette variable possède une méthode `InvokeSet()` qui permet de renseigner une information du compte :
  * Format : **InvokeSet("Description", $Var)** où $Var est une variable contenant la chaine de description
  * Cela permet de modifier l'attribut **description** de l'objet **user** créée plus haut (premier paramètre), dont la valeur est contenue dans le deuxième, ici **$description**.
* La méthode `CommitChanges()` valide les changements effectués sur l'objet **$User**.

***A faire :***

* Tester le script en ajoutant un compte avec une description associée, par exemple avec un compte "**Wilder1" et une description "**test de création de compte**".
* Afficher les comptes locaux pour vérifier que le compte est bien crée.
* Modifier le script pour ajouter d'autres propriétés.
* Relancer le script pour vérifier que le compte est bien créé.

#### 5. Ajout d'utilisateurs par fichier

Ecrire et exécuter le script suivant :

```powershell
#CreationDossier.ps1
$Dossier = "DossierTemporaire" #Changer le nom
If (Test-Path "c:\$Dossier")
{
    Write-Host "Le dossier $Dossier existe déjà"
}
Else
{
    New-Item -Path "c:\$Dossier" -ItemType Directory
}
```

**Que fait-il ?**

* Si le dossier n'existe pas, le script va créer automatiquement un dossier sous la racine c:
  * On utilise la variable **$Dossier** qui contient le nom du dossier.

***A faire :***

* Réecrit le code pour que le nom du dossier soit demandé à l'utilisateur.

**Pour la suite :**

* Créée un fichier **Comptes.txt** et mets le dans un dossier de ton choix.
* Télécharge le contenu que tu trouveras [ici](https://github.com/WildCodeSchool/TSSR_Resources/blob/main/Ressources_ateliers/comptes.txt) et colle-le dans un ce fichier
* Tu as maintenant un fichier **comptes.txt** dont les informations sont sous la forme : **nomCompte/nomComplet/Description**

Ecrire le script et exécuter le script suivant :

```powershell
#LectureFichier.ps1
$Fichier = "c:\xxx\comptes.txt" #Ici tu dois remplacer xxx par le nom du dossier dans lequel tu as mis ton fichier comptes.txt
If (Test-Path $Fichier)
{
    $Lignes = Get-Content -Path $Fichier
    Foreach ($Ligne in $Lignes)
    {
        $TabCompte = $Ligne.Split("/")
        Write-Host $TabCompte[0]
    }
}
Else
{
    Write-Host "Fichier $Fichier non-trouvé"
}
```

**Remarques**

* Cmdlet `Foreach` : Cette instruction peut s’interpréter de la manière suivante :
  * Pour chaque ligne **$Ligne** contenue dans le tableau **$Lignes**, la variable **$ligne** va successivement prendre la valeur de chaque ligne du tableau.
  * **$ligne** est une chaîne de caractères.
  * La variable **$ligne** possède une méthode `Split()` qui permet de retourner un tableau construit à partir de la chaîne de caractères contenue dans **$ligne**.
  * Les éléments du tableau correspondent aux chaînes de caractères délimitées par le séparateur `/` .

***A faire :***

* Par rapport à ce qui vient de s'afficher, modifier le script pour que le nom complet et la description soient également affichés en dessous du nom du compte.

**Petit challenge :**

* En utilisant les 2 scripts **AjoutCompteLocal.ps1** et **LectureFichier.ps1** écrire un script :
  * Il va lire le contenu d'un fichier qui contient des informations de comptes
  * Il va ajouter dans la base locale du système, tous les comptes trouvés
* Tester ce nouveau script pour ajouter tous les comptes du fichier **comptes.txt**
* Relancer le script une deuxième fois. Que se passe-t-il pour les comptes déjà existants ?

#### 6. Suppressions de comptes utilisateurs

Ecrire le script suivant:

```powershell
# SuppressionComptes.ps1
$Local = [ADSI]"WinNT://."
#changer le nom du dossier
$Fichier = "c:\xxx\comptes.txt"
If (Test-Path -Path $Fichier)
{
    $Lignes = Get-Content -Path $Fichier
    Foreach ($Ligne in $Lignes)
    {
        $TabCompte = $Ligne.Split("/")
        $Nom = $TabCompte[0]
        $Compte = [ADSI]"WinNT://./$Nom"
        If ($Compte.path)
        {
            $Local.delete("user",$Nom)
            Write-Host "$Nom supprimé"
        }
        Else
        {
            Write-Host "$Nom n'existe pas"
        }
    }
}
Else
{
    Write-Host "$Fichier non-trouvé" -ForegroundColor Red
}
```

**Que fait-il ?**

* Ce script permet de supprimer tous les comptes utilisateurs dont les noms sont contenus dans un fichier texte.
* Les informations sont toujours sous la forme :
  * nomCompte/nomComplet/Description

**Remarques :**

* Comme pour la création, la variable $local possède une méthode `delete()` qui permet de supprimer un objet de type **user**.

***A faire :***

* Executer le script et vérifier la liste des noms affichés
* Afficher les comptes pour vérifier que les comptes du fichier Comptes1.txt ont bien été supprimé.
* Modifier le fichier **Comptes.txt** en y ajoutant les comptes crées au **4.**
* Relancer le script et vérifier si tous les comptes créés dans cet atelier ont été supprimés.