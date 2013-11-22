---
title: "TCA : utiliser un eval personnalisé"
layout: post
published: true
categories: TYPO3
tags: TYPO3 TCA eval custom coordinate gps
---

Dans le eval du TCA, nous avons à disposition une vingtaine de fonctions de validation : trim, required, date...
Mais il est aussi possible d'en créer des personnalisées, comme le suggère la [documentation du TCA](http://docs.typo3.org/typo3cms/TCAReference/Reference/Columns/Input/Index.html) : **tx_* : User defined form evaluations.**

## Déclarer la nouvelle fonction

Déclarer le nouveau validateur dans le ext_localconf.php d'une extension.

*EXT:my_ext/ext_localconf.php*

```php
$GLOBALS['TYPO3_CONF_VARS']['SC_OPTIONS']['tce']['formevals']['tx_myext_validation_tce_coordinate'] = 'EXT:site_sites/class.tx_myext_validation_tce_coordinate.php';
```

## Implémentation de la classe

Implémenter la classe qui s'occupera :

 - de retourner le code JS utilisé dans la validation côté client,
 - et de valider la valeur avant son enregistrement.

*EXT:my_ext/class.tx_myext_validation_tce_coordinate.php*

```php
class tx_myext_validation_tce_coordinate {
	/**
	 * Code de la fonction JS qui validera le champ côté client.
	 * La fonction JS est appelée à chaque onchange du champ,
	 * et reçoit dans l'argument `value` le contenu du champ.
	 *
	 * @return string code JS
	 */
	public function returnFieldJS() {
		return '
			return value
				.replace(\',\', \'.\')
				.replace(/[^0-9\\.-]/, \'\');
		';
	}

	/**
	 * Traitement de la valeur côté serveur, déclenché par le DataHandler,
	 * avant la sauvegarde en BDD.
	 *
	 * @param  [type] $value The field value to be evaluated.
	 * @param  [type] $is_in The "is_in" value of the field configuration from TCA
	 * @param  [type] $set Boolean defining if the value is written to the database or not. Must be passed by reference and changed if needed. Default is TRUE.
	 * @return void
	 */
	public function evaluateFieldValue($value, $is_in, &$set) {
		$value = preg_replace('/[^0-9,\\.-]/', '', $value);
		$negative = substr($value, 0, 1) == '-';
		$value = strtr($value, array(',' => '.', '-' => ''));
		if (strpos($value, '.') === FALSE) {
			$value .= '.0';
		}
		$valueArray = explode('.', $value);
		$dec = array_pop($valueArray);
		$value = join('', $valueArray) . '.' . $dec;
		if ($negative) {
			$value *= -1;
		}
		$value = number_format($value, 6, '.', '');

		return $value;
	}
}
```

## Utilisation


Ensuite il est possible d'utiliser le nouveau validateur dans un eval du TCA.

```php
$TCA['tx_myext_domain_model_bar'] = array(
	'columns' => array(
		'latitude' => array(
			'exclude' => 0,
			'label' => 'LLL:EXT:site_sites/Resources/Private/Language/locallang_db.xlf:tx_sitesites_domain_model_site.latitude',
			'config' => array(
				'type' => 'input',
				'size' => 10,
				'eval' => 'trim,required,tx_myext_validation_tce_coordinate'
			),
		),
		'longitude' => array(
			'exclude' => 0,
			'label' => 'LLL:EXT:site_sites/Resources/Private/Language/locallang_db.xlf:tx_sitesites_domain_model_site.longitude',
			'config' => array(
				'type' => 'input',
				'size' => 10,
				'eval' => 'trim,required,tx_myext_validation_tce_coordinate'
			),
		),
	),
);
```