---
title: "Changer la langue courante en mode CLI"
layout: post
published: true
categories: TYPO3
tags: TYPO3 translation language CLI scheduler
---

Dans le BE, le contexte utilisateur de TYPO3 est basé sur l'utilisateur BE utilisé au login.
De même, en mode CLI (scheduler, command line), le contexte utilisateur est basé sur l'utilisateur BE _cli_lowlevel, ou _cli_scheduler, etc...

Concernant les traductions, elles sont chargées depuis la langue définie au niveau des préférences du compte utilisateur. Cela permet d'afficher l'interface du BE dans la langue voulue. Par exemple, dans les modules du BE, les utilitaires de traductions ($GLOBAL['LANG']->sL ou LocalizationUtility::translate()) se basent sur la langue courante de l'utilisateur BE.

Mais ça se complique quand on souhaite changer de langue courante. C'est par exemple nécessaire dans le cadre d'un processus batch, qui doit effectuer des imports de données dans les différentes langues du sites.
Ce processus, lancé en CLI (par exemple avec l'utilisateur BE _cli_lowlevel), verra chargée comme langue courante la langue préférée de l'utilisateur _cli_lowlevel ! Et ce pendant toute la durée du traitement.

Il faut donc à un moment pouvoir changer de langue courante, afin que nos contenus ou libellés soit correctement traduits. Voici comment procéder.

Dans votre traitement batch (task extbase pour le scheduler par exemple), définissez la langue courante au besoin :

```php
/**
 * Définition de la langue courante
 * Comme ceci, le TranslationUtility chargera les bonnes clés
 *
 * @param [type] $lang code iso de la langue (en, fr, etc.)
 */
private function setLocalizationKey($lang) {
	switch ($lang) {
		case 'en':
			$lang = 'default';
			break;
	}

	$GLOBALS['BE_USER']->uc['lang'] = $lang;

	// http://forge.typo3.org/issues/51920
	\TYPO3\CMS\Extbase\Utility\LocalizationUtility::reset();
}
```

Note : il y a un petit problème de cache dans \TYPO3\CMS\Extbase\Utility\LocalizationUtility au niveau de l'initialisation des traductions. Pour l'instant j'ai dû patcher la classe, pour y ajouter la méthode suivante, mais [ça changera peut-être](http://forge.typo3.org/issues/51920) :

```php
/**
 * Reset the loaded translation
 * If $extensionName is given, the reset apply only for this $extensionName keys.
 *
 * PATCH GAYA (http://forge.typo3.org/issues/51920)
 *
 * @param  [type] $extensionName [optional] name of the extension (camelCase)
 * @return void
 */
static public function reset($extensionName = NULL) {
	if (NULL !== $extensionName) {
		if (isset(self::$LOCAL_LANG[$extensionName])) {
			unset(self::$LOCAL_LANG[$extensionName]);
		}
	} else {
		self::$LOCAL_LANG = array();
	}
}
```