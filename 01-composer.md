# Composer

## Qu'est-ce que composer ?
[composer](https://getcomposer.org/) est le gestionnaire de librairie standard du moment pour PHP.

## À quoi ça sert ?
C'est l'outil en ligne de commande qui nous permet d'installer des librairies et plus.

Suivez ce [lien](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos) pour débuter et votre formateur vous guidera pour l'installation.

## Utilisation basique

Trois commandes sont très souvent utilisées, en voici deux :
La première pour télécharger une librairie dans le cadre d'un projet spécifique :
```bash
composer require nom/librairie
```
* **Signification des termes :**
   * **nom** : C'est le nom du fournisseur (ou de l'organisation) qui développe ou maintient la bibliothèque. Par exemple, cela pourrait être un nom de développeur, une entreprise ou une communauté. Exemple : `laravel`, `symfony`, `guzzlehttp`.

   * **librairie** : C'est le nom spécifique de la bibliothèque (ou package) que vous souhaitez installer. C'est généralement une fonctionnalité ou un ensemble de fonctionnalités. Exemple : `framework`, `http-client`, `psr7`.

La deuxième pour installer une librairie globalement sur notre OS :
```bash
composer global require nom/librairie
```
L'utilisation du mot-clé `global` est la différence, on utilisera cette commande pour installer un framework.

## Installation de librairies avec composer.json
La troisième commande est utilisée dans le cas ou on souhaite installer manuellement nos librairies à l'aide d'un fichier de configuration `composer.json`
```json
{
    "require": {
        "stripe/stripe-php": "^7.28",
        "intervention/image": "^2.4"
    }
}
```
Cet exemple est pour l'installation de la librairie stripe, un fournisseur de solution de paiement en ligne, et d'une librairie très connue qui permet de gérer les images.
Une fois qu'on a écrit ce qu'on veut comme librairies ou framework dans ce fichier, on tape la commande
```bash
composer install
```
Composer installe les librairies et nous fournit un autoloader dans le fichier `vendor/autoload.php` que l'on inclut dans notre fichier principal et qui nous évitent des lignes de `require`
Quand vous clonerez une application depuis github ou autre service de ce type, cette dernière commande sera utilisée.


---
---


La spécification de version `1.12.3` suit le système **SemVer (Versionnage Sémantique)**. Chaque chiffre a une signification spécifique :

**Structure :**
**1.12.3** peut être décomposé en trois parties : **MAJEUR.MINOR.PATCH**

### 1. MAJEUR (1)

* **Signification :** 
 Indique une version majeure du projet.
* Impact :
Une mise à jour de ce chiffre signifie que des **changements importants** (ou incompatibles) ont été faits. Votre code pourrait avoir besoin d'être modifié pour s'adapter à cette nouvelle version.
* **Exemple :**
Passer de **1.x.x** à **2.0.0** peut introduire des changements qui cassent la compatibilité avec les versions précédentes.

### 2. MINEUR (12)
* **Signification :**
Représente des nouvelles fonctionnalités ajoutées de manière compatible avec les versions précédentes.
* **Impact :**
Les versions mineures sont souvent ajoutées sans risque pour votre code existant.
* **Exemple :**
Passer de 1.12.3 à 1.13.0 signifie l'ajout d'une fonctionnalité sans casser le code existant.

### 3. PATCH (3)
* **Signification :**
Indique une correction de bugs ou des améliorations mineures.
* **Impact :**
Les corrections de patch n'affectent pas les fonctionnalités existantes. Elles résolvent généralement de petits problèmes ou améliorent la stabilité.
* **Exemple :**
Passer de 1.12.3 à 1.12.4 signifie que seulement des bugs mineurs ont été corrigés.

#

**Exemple concret :**
* **Version 1.12.3 :**
   * **1** : Première version majeure stable du projet.
   * **12** : 12ème version mineure (nouvelles fonctionnalités ajoutées depuis la version 1.0.0).
   * **3** : Troisième correction (ou amélioration) de la version 1.12.

**Pourquoi c’est important ?**

Ce système permet aux développeurs de :

 1. Comprendre rapidement l'impact d'une mise à jour.
 2. Gérer la compatibilité avec d'autres bibliothèques.
 3. Prévoir les risques liés à une mise à jour.

En résumé, chaque partie de la version donne des informations précieuses sur les changements introduits !
---
---

Dans les fichiers `composer.json`, les symboles mathématiques placés devant les numéros de version (comme `^`, `~`, `>=`, `<=`) indiquent à Composer quelles versions d'une bibliothèque peuvent être installées. 

Voici ce que ces symboles signifient de façon simple :

1. `^` **(chapeau)** :
* **Signification** : Autorise les mises à jour mineures, mais pas les mises à jour majeures.
* **Exemple** : `"stripe/stripe-php": "^7.28"` permet d'installer la version `7.28` et toutes les versions mineures ou correctives (comme `7.29` ou `7.30`), mais pas `8.0`.
* **Utilisation courante** : Garantit que les nouvelles versions installées n'introduiront pas de changements majeurs qui pourraient casser votre code.


2. `~` **(tilde)** :
* **Signification** : Autorise les mises à jour correctives (patches) et mineures, mais bloque la prochaine version majeure.
* **Exemple** : `"monpackage": "~1.2"` autorise les versions comme `1.2.1` ou `1.3.0`, mais pas `2.0.0`.

3. `>=` **(supérieur ou égal)** :
* **Signification** : Installe la version indiquée ou toute version plus récente.
* **Exemple** : `"monpackage": ">=1.5"` installe la version `1.5` ou toute version plus récente (`1.6`, `2.0`, etc.).

4. `<=` **(inférieur ou égal)** :
* **Signification** : Installe la version indiquée ou toute version plus ancienne.
* **Exemple** : `"monpackage": "<=2.0"` limite l'installation à la version `2.0` ou antérieure.

5. * **(joker)** :
* **Signification** : Accepte n'importe quelle version correspondant au motif.
* **Exemple** : `"monpackage": "1.2.*"` accepte toutes les versions `1.2.x`.

**Résumé** :
* `^` : Accepte les mises à jour qui ne cassent pas la compatibilité.
* `~` : Accepte les mises à jour mineures seulement.
* `>=` / `<=` : Définit une plage de versions minimale ou maximale.
* `*` : Accepte toutes les versions correspondant à un motif spécifique.
  
Ces symboles aident à gérer les versions de manière flexible tout en s'assurant que votre projet reste stable.

