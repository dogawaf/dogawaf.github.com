---
title: "Configurer le RTE de TYPO3"
layout: post
published: false
categories: TYPO3
tags: TYPO3 RTE
---

Le RTE est un monde compliqué. Mais c'est l'outil qui sera le plus utilisé par le rédacteur. Il est donc primordial de le configurer correctement, afin notamment :
- de ne pas avoir de fonctionnalités ou boutons inutiles
- de nettoyer le code HTML sauvegardé en base de données (le rédacteur fera souvent du copier-coller depuis Word)
- d'avoir un rendu visuel proche de ce qui sera affiché sur le site


Configurer l'extension
============================
Dans l'Extension manager, ouvrir la page de configuration de l'extension rtehtmlarea, et modifier les valeurs suivantes :

	Nous allons définir pas mal de choses dans la config. Je préfère activer les fonctionnalités que chercher comment les désactiver.
		basic.defaultConfiguration: "Minimal"

	Parce qu'on veut pouvoir insérer de l'image dans le texte. Des fois le tt_content.image ne suffit pas.
		basic.enableImages: Oui

	On pourrait avoir besoin de balises inline supplémentaires (span, sub, small...). Mais on va dire que par défaut on en a pas besoin.
		basic.enableInlineElements: Non

	On va  limiter l'utilisation de l'attribut style, cela évitera des écarts indésirables d'avec les styles définis dans les CSS.
		basic.allowStyleAttribute: Non


pageTSConfig
userTSConfig
<<<<<<< HEAD
feuille de style rte.css
=======
feuille de style rte.css
>>>>>>> Draft Configurer le RTE de TYPO3
