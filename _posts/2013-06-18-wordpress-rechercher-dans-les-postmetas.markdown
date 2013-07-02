---
title: "Etendre la recherche Wordpress à des champs dans wp_postmeta"
layout: post
published: true
categories: Wordpress
tags: wordpress wp-admin acf meta recherche custom_fields
---

Sur Wordpress, par défaut la recherche est effectuée dans les champs `wp_post.title` et `wp_post.content`.

Wordpress permet facilement de _restreindre_ une `wp_query` avec des meta_keys et meta_values, mais ne permet pas d'effectuer une recherche textuelle dans `wp_postmeta` en complément de `wp_post.title` et `wp_post.content`.
On va donc devoir modifier directement la requête SQL de la recherche en ajoutant une liaison vers `wp_postmeta`, et en activant la recherche dans les champs souhaités.

Pour cela on va utiliser le filtre `posts_search`, qui est appelé par `wp_query->get_posts()` lors de la construction de la requête SQL.
Le callback de ce filtre reçoit directement la partie concernant la recherche de la clause WHERE (`$search`), et l'instance de `wp_query`.
Le callback peut alors altérer `$search` avant de le retourner.

Dans l'exemple ci-dessous, nous avons en plus restreint l'activation du filtre dans la partie admin du site, et uniquement lors de la recherche dans le post_type "video".


```php
/**
 * Active la recherche de posts dans les postmetas
 *
 * @param string $search
 * @param WP_Query $wp_query
 * @return string $search
 */
function my_admin_posts_search_filter($search, $wp_query)
{
	global $pagenow, $wpdb;

	if (empty($search)) {
		return $search;
	}

	if (is_admin() && $pagenow == 'edit.php' && $wp_query->query_vars['post_type'] === 'video')) {

		// extraction de la chaine de recherche
		preg_match("/LIKE '(%.*?%)'\)/", $search, $matches);
		$search_term = $matches[1];

		if (!$search_term) {
			return $search;
		}

		// la liste des postmeta dans lesquelles effectuer la recherche
		$meta_keys = array('my_custom_field1', 'my_custom_field2', 'my_custom_field3');

		// clause WHERE pour les postmeta
		$meta_where = '';

		foreach ($meta_keys as $meta_key) {
			$meta_where .= ') OR ('.$wpdb->postmeta.'.meta_key = \''.$meta_key.'\' AND '.$wpdb->postmeta.'.meta_value LIKE \''.$search_term.'\'';
		}

		// on rajoute à la queue du where les clause concernant les metas
		$search = str_replace(')))', $meta_where . ')))', $search);

		// le code suivant permet à wp_query d'activer la jointure vers wp_postmeta
		// le fait de mettre une key vide et une value vide permet de ne pas restreindre la recherche
		// mais juste d'activer la jointure
		$wp_query->meta_query->relation = 'AND';
		$wp_query->meta_query->queries = array(
			array(
				'key' => '',
				'compare' => '!=',
				'value' => '',
			)
		);
	}

	return $search;
}
add_filter('posts_search', 'my_admin_posts_search_filter', 0, 2);
```