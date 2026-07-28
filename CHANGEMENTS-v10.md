# moonmoon 10.0.0-rc — Portage PHP 8.2

Ce paquet est une version corrigée de moonmoon 9.0.0-rc.3 (dernière release
officielle du projet, qui n'est plus activement maintenu — un ticket ouvert
depuis 2022 demandait justement ce portage sans jamais avoir été traité :
https://github.com/moonmoon/moonmoon/issues/117).

Toutes les corrections ci-dessous ont été testées en conditions réelles
(serveur PHP 8.3, installation complète, connexion admin, gestion des
abonnements, chargement effectif d'un flux RSS) : plus aucune erreur,
avertissement ou dépréciation.

## Correctifs appliqués

1. **`admin/inc/auth.inc.php`** — `hash_equals()` recevait `null` quand le
   cookie de session n'existait pas encore (ex: avant la première
   connexion). Sous PHP 8, cela provoquait une erreur fatale
   (`TypeError`) et rendait la page de connexion admin totalement
   inaccessible. Corrigé avec des valeurs par défaut (`?? ''`).

2. **`admin/subscriptions.php`** — utilisait `get_magic_quotes_gpc()`,
   une fonction supprimée de PHP depuis la version 8.0 (erreur fatale
   "undefined function" dès qu'on ajoutait/modifiait un flux). Ce
   mécanisme n'a plus lieu d'être (les magic quotes n'existent plus dans
   PHP depuis longtemps), le code correspondant a été retiré.

3. **`install.php`** — interpolation de variable au format `${var}` dans
   une chaîne, dépréciée depuis PHP 8.2. Remplacée par la syntaxe
   `{$var}`.

4. **SimplePie (bibliothèque de parsing RSS/Atom)** — la version 1.5
   embarquée est **fatalement incompatible avec PHP 8** (plantage
   systématique dès qu'un flux est chargé, à cause d'un changement de
   signature de `implode()`). C'était le blocage principal empêchant
   moonmoon de fonctionner sous PHP 8. Mise à jour vers **SimplePie 1.9.0**
   (dernière version stable, sept. 2025), compatible PHP 7.2 à 8.4+.
   `composer.json` a été mis à jour en conséquence
   (`"php": "^7.4 || ^8.0"`, `"simplepie/simplepie": "^1.9"`).

5. Suppression des dépendances de développement (PHPUnit, Guzzle...) du
   dossier `vendor/`, inutiles en production et non compatibles PHP 8
   pour certaines (allège aussi l'archive).

## Comment déployer

**Important : ce ZIP contient uniquement le code applicatif.** Il ne
contient pas de données actuelles (configuration, liste de flux, mot de
passe admin, cache). Pour mettre à jour votre installation existante sans
tout perdre :

1. Faite une sauvegarde complète de votre dossier actuel sur l'hébergement
   (au cas où).
2. Repèrez et **conservez** ces fichiers/dossiers de votre installation
   actuelle (ne pas les écraser) :
   - `custom/config.yml`
   - `custom/people.opml`
   - `admin/inc/pwd.inc.php`
   - `custom/cache/` (peut être vidé sans risque si besoin)
   - tout thème personnalisé sous `custom/views/` ou `custom/style/` si tu
     l'as modifié
3. Upload (écrase) tout le reste : `app/`, `admin/*.php`, `vendor/`,
   `index.php`, `atom.php`, `cron.php`, `install.php`, `composer.json`.
4. Vérifiez que votre hébergement utilise bien PHP 8.2 (ou proche) et que
   l'extension `php-xml` est activée (déjà requise par moonmoon).
5. Testez la page d'accueil, la connexion admin, et l'ajout/suppression
   d'un flux.

## Limite connue (non corrigée)

Un problème de sécurité est ouvert depuis 2020 sur le dépôt officiel :
le contenu des flux n'est pas assaini, ce qui permet en théorie à un flux
malveillant d'injecter du JavaScript (XSS) :
https://github.com/moonmoon/moonmoon/issues/111