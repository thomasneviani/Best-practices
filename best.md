***

# Documentation complète : Symfony + AssetMapper + Turbo + Stimulus

***

## Structure du projet Symfony

Le projet suit l'architecture standard Symfony :

* **assets/** : JavaScript, CSS, contrôleurs Stimulus
* **bin/** : scripts exécutables (console Symfony)
* **config/** : configuration (services, routes, packages)
* **migrations/** : migrations Doctrine
* **public/** : point d'entrée web (`index.php`) et assets compilés
* **src/** : cœur de l'application (Controllers, Entities, Forms, Security, etc.)
* **templates/** : vues Twig
* **tests/** : tests automatisés
* **translations/** : fichiers de traduction
* **var/** : cache et logs
* **vendor/** : dépendances PHP (Composer)

***

## AssetMapper : gestion moderne des assets sans build

**AssetMapper** est le système recommandé par Symfony pour gérer tes assets (CSS, JavaScript, images) de manière simple et moderne. [symfony](https://symfony.com/doc/current/frontend/asset_mapper.html)

### Principe de fonctionnement

* **Aucun bundler** (pas de Webpack, Vite, npm) requis [symfony](https://symfony.com/doc/current/frontend/asset_mapper.html)
* Utilise les **import maps** natifs du navigateur pour charger les modules JavaScript [symfony](https://symfony.com/blog/new-in-symfony-6-3-assetmapper-component)
* Versionne automatiquement les fichiers pour le **cache busting** [symfony](https://symfony.com/doc/current/frontend/asset_mapper.html)
* Compile les assets pour la production via une simple commande [laconsole](https://laconsole.dev/formations/symfony/asset-mapper)

### Configuration de base

Le fichier `config/packages/asset_mapper.yaml` définit les répertoires d'assets  : [laconsole](https://laconsole.dev/formations/symfony/asset-mapper)

```yaml
framework:
  asset_mapper:
    paths:
      - assets/
```

Le fichier `importmap.php` à la racine du projet mappe les modules JavaScript  : [symfony](https://symfony.com/blog/new-in-symfony-6-3-assetmapper-component)

```php
return [
  'app' => [
    'path' => './assets/app.js',
    'entrypoint' => true,
  ],
  '@hotwired/stimulus' => [
    'version' => '3.2.1',
  ],
];
```

### Fonctionnement en développement vs production

**En développement** : le serveur Symfony sert automatiquement les assets depuis le dossier `assets/`. [laconsole](https://laconsole.dev/formations/symfony/asset-mapper)

**En production** : la commande `php bin/console asset-map:compile` copie tous les assets versionnés dans `public/assets/` et génère les fichiers `manifest.json` et `importmap.json` pour un chargement ultra-rapide. [symfony](https://symfony.com/blog/new-in-symfony-6-3-assetmapper-component)

### Intégration avec Stimulus et Turbo

AssetMapper permet d'importer directement Stimulus et Turbo dans ton JavaScript  : [discourse.hkvstore](https://discourse.hkvstore.com/t/using-symfony-stimulus-bundle-with-assetmapper/11253)

```javascript
// assets/app.js
import { Application } from '@hotwired/stimulus';
import './stimulus_bootstrap.js';
```

Les contrôleurs Stimulus placés dans `assets/controllers/` sont automatiquement découverts et chargés. [discourse.hkvstore](https://discourse.hkvstore.com/t/using-symfony-stimulus-bundle-with-assetmapper/11253)

***

## Turbo vs Stimulus : rôles complémentaires

### Turbo

* Intercepte automatiquement les **liens** et **formulaires**
* Transforme la navigation en requêtes Ajax
* Donne une expérience **SPA sans rechargement de page**
* Permet des mises à jour partielles via **Turbo Frames** et **Turbo Streams**

**À utiliser pour :**

* Navigation entre pages
* Soumissions de formulaires
* Rafraîchissement partiel de contenu
* Temps réel (avec Mercure)

***

### Stimulus

* Framework JavaScript léger basé sur des **contrôleurs**
* Ajoute de l'interactivité ciblée au HTML existant
* JavaScript structuré, minimal et lisible

**À utiliser pour :**

* Modales, menus, toggles
* Validation en temps réel
* Autocomplétion, animations
* Interactions spécifiques non couvertes par Turbo

***

## Bonnes pratiques

* **AssetMapper** gère le chargement et le versionnement des assets
* **Turbo** gère la navigation et les mises à jour automatiques
* **Stimulus** ajoute la logique JS personnalisée
* **Fetch / Axios** restent utiles pour :
  * Appels API complexes
  * Logique métier côté client
  * Cas hors des patterns Turbo

***

## Check-list : Turbo, Stimulus ou Fetch ?

### 🧭 Utilise **Turbo** si :

* Tu gères la **navigation entre pages**
* Tu soumets un **formulaire standard**
* Le serveur renvoie du **HTML**
* Tu veux mettre à jour une partie de la page
* Tu fais du **temps réel** (Turbo Streams + Mercure)

👉 **Règle** : *serveur → HTML → DOM*

***

### 🎛️ Utilise **Stimulus** si :

* Tu dois **disable / enable** un bouton
* Tu dois **afficher / masquer** un élément
* Tu gères un **état visuel** (loading, actif, erreur)
* Tu ajoutes des **micro-interactions UI**
* Tu manipules des classes CSS ou attributs HTML
* Tu réagis à des événements (`click`, `input`, `change`)

👉 **Règle** : *état UI local → JavaScript → DOM*

***

### 🌐 Utilise **Fetch / Axios** si :

* Tu appelles une **API JSON**
* Tu as une **logique métier côté client**
* Tu ne veux pas renvoyer du HTML
* Le cas ne correspond pas aux patterns Turbo

👉 **Règle** : *données → JSON → logique client*

***

## Règle mentale rapide 🧠

> **AssetMapper** = chargement et versionnement des assets
> **Turbo** = navigation et rendu serveur
> **Stimulus** = état et interactivité de l'interface
> **Fetch** = données et logique client

***

## Verdict

👉 **AssetMapper + Turbo + Stimulus** est une combinaison idéale pour :

* Des projets Symfony modernes
* Une UX fluide
* Moins de JavaScript
* Zéro dépendance à des frameworks lourds (React, Vue)
* Zéro build complexe (pas de Webpack, npm, Node.js)

Simple, efficace, maintenable 💙
