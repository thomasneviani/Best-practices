https://symfony.com/doc/current/best_practices.html

Oui, c'est tout à fait recommandé d'utiliser Stimulus, Turbo et AssetMapper dans un projet Symfony moderne, car ils permettent de créer des applications web réactives sans la complexité de Node.js et Webpack. [fsck](https://fsck.sh/en/blog/symfony-assetmapper-no-webpack/)

## Stimulus

**Stimulus** est un framework JavaScript léger qui permet d'ajouter des comportements dynamiques et interactifs aux éléments HTML en associant des contrôleurs JavaScript aux éléments du DOM. [laconsole](https://laconsole.dev/formations/symfony/asset-mapper)

**Cas d'utilisation** : Ajouter de l'interactivité côté client (modales, menus déroulants, validation de formulaires, requêtes Ajax) sans réécrire tout votre HTML en JavaScript.

**Exemple de code** :

```javascript
// assets/controllers/song-controls_controller.js
import { Controller } from '@hotwired/stimulus';
import axios from 'axios';

export default class extends Controller {
    async play(event) {
        event.preventDefault();
        const response = await axios.get('/api/song/play');
        console.log('Playing song:', response.data);
    }
}
```

```twig
{# templates/song/index.html.twig #}
<div {{ stimulus_controller('song-controls') }}>
    <a href="#" {{ stimulus_action('song-controls', 'play') }}>
        <i class="fas fa-play"></i>
    </a>
</div>
```

## Turbo

**Turbo** transforme automatiquement les clics sur les liens et les soumissions de formulaires en requêtes Ajax, offrant une expérience similaire à une Single Page Application sans rechargements complets de page. [symfony](https://symfony.com/bundles/ux-turbo)

**Cas d'utilisation** : Accélérer la navigation et améliorer l'UX en remplaçant des parties spécifiques de la page sans rechargement complet (listes dynamiques, notifications en temps réel).

**Exemple de code** :

```php
// src/Controller/TaskController.php
use Symfony\UX\Turbo\TurboBundle;

public function create(Request $request): Response
{
    // ... traitement du formulaire
    
    if (TurboBundle::STREAM_FORMAT === $request->getPreferredFormat()) {
        $request->setRequestFormat(TurboBundle::STREAM_FORMAT);
        return $this->renderBlock('task/new.html.twig', 'success_stream', [
            'task' => $task,
        ]);
    }
    
    return $this->redirectToRoute('task_success');
}
```

```twig
{# templates/task/new.html.twig #}
{% block success_stream %}
    <turbo-stream action="append" targets="#tasks-list">
        <template>
            <div id="task_{{ task.id }}">{{ task.title }}</div>
        </template>
    </turbo-stream>
{% endblock %}
```

## AssetMapper

**AssetMapper** est le système natif PHP de gestion d'assets qui utilise les modules ES et les import maps du navigateur, éliminant le besoin de Webpack, Node.js et npm. [symfony](https://symfony.com/blog/new-in-symfony-6-3-assetmapper-component)

**Cas d'utilisation** : Gérer vos fichiers JavaScript, CSS et leurs dépendances dans des projets qui utilisent Stimulus/Turbo sans nécessiter une chaîne de build complexe.

**Exemple de code** :

```yaml
# config/packages/asset_mapper.yaml
framework:
    asset_mapper:
        paths:
            - assets/
```

```php
// importmap.php
return [
    'app' => [
        'path' => './assets/app.js',
        'entrypoint' => true,
    ],
    '@hotwired/stimulus' => [
        'version' => '3.2.2',
    ],
    '@symfony/stimulus-bundle' => [
        'path' => '@symfony/stimulus-bundle/loader.js',
    ],
];
```

```javascript
// assets/app.js
import './bootstrap.js';
import './styles/app.css';
```

**Verdict** : Cette combinaison est idéale pour les projets Symfony modernes qui n'ont pas besoin de frameworks JavaScript lourds comme React ou Vue. [fsck](https://fsck.sh/en/blog/symfony-assetmapper-no-webpack/)
