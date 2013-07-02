---
title: "Mettre à disposition des constantes Typoscript dans un template Fluidpages ?"
layout: post
published: true
categories: TYPO3
tags: TYPO3 typoscript
---

En interne, Fluidpages passe le relais à Flux pour faire le rendu final de votre page, et Flux initialise une vue et lui passe les settings. Comme on le sait, les settings disponibles dans un template fluid sont issus du setup de votre plugin. Mais dans le cas d'un rendu de page avec Fluidpages, quel est ce plugin ?

Selon la [documentation de Fluidpages](https://github.com/FluidTYPO3/fluidpages) :

> Note: the extension key you use when registering templates will also be used as extension name for the controller context used in rendering the Fluid template [...]

> [...] Fluid Pages emulates a Page object and an associated PageController [...]

Cela veut dire que vous pouvez définir des settings sur ce plugin "émulé" : si votre extension s'appelle `site`, alors le contexte d'exécution du template sera l'extension `site`, et le nom du plugin sera `page` (Page est le nom du plugin fixé par Fluidpages, vous n'avez pas la possibilité de le modifier).

Vous pouvez alors définir des settings avec le setup suivant :

```
plugin.tx_site_page.settings {
	maVar = {$maConstante}
}
```

Et l'utiliser dans votre template, pour rendre un menu par exemple :

```html
<v:page.menu.directory pages="{settings.maVar}" />
```