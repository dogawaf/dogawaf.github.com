---
title: "Fonctions utiles au développeur TYPO3"
layout: post
published: true
categories: TYPO3
tags: TYPO3 debug
---

var_dump
========
var_dump(), print_r() ou debug() ne sont pas très utiles dans le monde d'extbase. Les objets étant très volumineux et avec de nombreuses interdépendances entre eux, il est difficile de s'y retrouver dans le print_r() d'un objet, si tant est que le navigateur parvienne à charger la page.
Utilisez donc ceci :

```php
\TYPO3\CMS\Extbase\Utility\DebuggerUtility::var_dump();
```
cObj / cObj->data
=================
Dans un controller, le cObj est ici :

```php
$cObj = $this->configurationManager->getContentObject();
```
Et les data de l'enregistrement courant (cObj->data) sont donc là :

```php
$cObj = $this->configurationManager->getContentObject();
$row = $cObj->data;
```
Activer le Devlog
=================
Installer et activer l'extension devlog
Aller dans l'install tool, et cocher la case "enable_DLOG"

Traduire une chaîne
===================
Il faut que la chaîne soit définie le fichier locallang de l'extension `$extensionName`.
Il faut vider le cache de TYPO3 (all ou temp_cached) à chaque modification d'un fichier locallang.

```php
\TYPO3\CMS\Extbase\Utility\LocalizationUtility::translate($key, $extensionName);
```

Aperçu d'un tt_content dans le module 'Page' du BO
==================================================
Cela ce passe ici : typo3/sysext/backend/Classes/View/PageLayoutView.php::tt_content_drawItem()
Cette méthode reçoit le record, et en construit un aperçu, en fonction de son CType.

Il existe un hook pour personnaliser cet aperçu :

```php
$GLOBALS['TYPO3_CONF_VARS']['SC_OPTIONS']['cms/layout/class.tx_cms_layout.php']['tt_content_drawItem']
```
Le hook doit implémenter l'interface `\TYPO3\CMS\Backend\View\PageLayoutViewDrawItemHookInterface`.
L'appel est fait comme ceci :

```php
$hookObject->preProcess($parentObject, $drawItem, $headerContent, $itemContent, $row);
```
L'idée, c'est de déterminer si le hook souhaite préfixer l'aperçu de `$row`, ou le remplacer complètement (dans ce cas `$drawItem` doit être mis à FALSE, et TYPO3 ne gérera pas l'aperçu).

Paramètres reçus:

```
_$parentObject_ : l'instance de PageLayoutView
_$drawItem_ : reçu en référence, c'est un booléen qui indique si le rendu est géré par le hook, ou par TYPO3. Il faut donc le passe à FALSE si on veut bypasser le rendu par défaut.
_$headerContent_ : ligne d'entête (par défaut, c'est le contenu de `$row['header']`).
_$itemContent_ : ligne de contenu
_$row_ : record de tt_content
```

_Attention à bien échapper les valeurs issues de `$row` avec `htmlspecialchars`._

Il y a aussi deux autres hooks pour l'affichage des informations du CType=list (les plugins).

```php
// pour un list_type en particulier:
$GLOBALS['TYPO3_CONF_VARS']['SC_OPTIONS']['cms/layout/class.tx_cms_layout.php']['list_type_Info'][<LIST_TYPE>]
// ou pour tous les list_type:
$GLOBALS['TYPO3_CONF_VARS']['SC_OPTIONS']['cms/layout/class.tx_cms_layout.php']['list_type_Info']['_DEFAULT']
```
Ici, il faut aussi retourner des informations, qui seront ensuite ajoutées dans l'aperçu du tt_content.

Paramètres reçus par le hook:

```
_$pObj_ : l'instance de PageLayoutView
_$row_ : record de tt_content
_$infoArr_ : tableau vide... ne sert à rien a priori.

```

