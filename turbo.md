Pour activer Stimulus et Turbo dans Symfony, voici les fichiers de configuration nécessaires: [ux.symfony](https://ux.symfony.com/stimulus)

https://symfony.com/bundles/ux-turbo/current/index.html

## Fichiers de configuration Stimulus

### `assets/bootstrap.js`
Ce fichier initialise l'application Stimulus: [symfonycasts](https://symfonycasts.com/screencast/symfony/stimulus)

```javascript
import { startStimulusApp } from '@symfony/stimulus-bundle';

const app = startStimulusApp();

// Enregistrer des contrôleurs personnalisés ou tiers ici
// app.register('some_controller_name', SomeImportedController);
```

### `assets/controllers.json`
Ce fichier permet d'activer les contrôleurs Stimulus des packages UX: [github](https://github.com/symfony/stimulus-bridge)

```json
{
  "controllers": {
    "@symfony/ux-turbo": {
      "turbo-core": {
        "enabled": true,
        "fetch": "eager"
      },
      "mercure-turbo-stream": {
        "enabled": false,
        "fetch": "eager"
      }
    }
  },
  "entrypoints": []
}
```

### `importmap.php`
Le package Turbo ajoute automatiquement son entrée lors de l'installation: [symfonycasts](https://symfonycasts.com/screencast/asset-mapper/ux-packages)

```php
<?php

return [
    // ... autres imports
    '@hotwired/stimulus' => [
        'version' => '3.2.2',
    ],
    '@symfony/stimulus-bundle' => [
        'path' => './vendor/symfony/stimulus-bundle/assets/dist/loader.js',
    ],
    '@hotwired/turbo' => [
        'version' => '7.3.0',
    ],
];
```

## Installation

Pour installer Stimulus: [ux.symfony](https://ux.symfony.com/stimulus)
```bash
composer require symfony/stimulus-bundle
```

Pour installer Turbo: [symfonycasts](https://symfonycasts.com/screencast/turbo/install)
```bash
composer require symfony/ux-turbo
```

## Configuration dans les templates

Dans votre fichier `base.html.twig`, assurez-vous d'avoir: [github](https://github.com/symfony/stimulus-bridge)

```twig
{% block stylesheets %}
    {{ ux_controller_link_tags() }}
{% endblock %}

{% block javascripts %}
    {{ importmap('app') }}
{% endblock %}
```

La fonction `ux_controller_link_tags()` génère les balises CSS pour les packages UX qui en ont besoin. [github](https://github.com/symfony/stimulus-bridge)

Les recettes Symfony créent automatiquement ces fichiers lors de l'installation, donc après avoir exécuté les commandes `composer require`, tout devrait être fonctionnel sans configuration supplémentaire. [symfonycasts](https://symfonycasts.com/screencast/symfony/turbo)


Voici un exemple complet d'implémentation de Turbo avec un formulaire et un contrôleur Symfony: [symfony](https://symfony.com/bundles/ux-turbo)

## Exemple 1 : Turbo Stream pour ajouter une tâche

### Contrôleur (`src/Controller/TaskController.php`)

```php
<?php

namespace App\Controller;

use App\Entity\Task;
use App\Form\TaskType;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\UX\Turbo\TurboBundle;

class TaskController extends AbstractController
{
    #[Route('/task/new', name: 'task_new')]
    public function new(Request $request): Response
    {
        $form = $this->createForm(TaskType::class, new Task());
        $emptyForm = clone $form; // Pour réinitialiser le formulaire après soumission
        
        $form->handleRequest($request);
        
        if ($form->isSubmitted() && $form->isValid()) {
            $task = $form->getData();
            
            // Sauvegarde de la tâche en base de données
            $entityManager = $this->getDoctrine()->getManager();
            $entityManager->persist($task);
            $entityManager->flush();
            
            // Détection si la requête vient de Turbo
            if (TurboBundle::STREAM_FORMAT === $request->getPreferredFormat()) {
                $request->setRequestFormat(TurboBundle::STREAM_FORMAT);
                
                return $this->renderBlock('task/new.html.twig', 'success_stream', [
                    'task' => $task,
                    'form' => $emptyForm,
                ]);
            }
            
            // Redirection classique pour les requêtes non-Turbo
            return $this->redirectToRoute('task_success', [], Response::HTTP_SEE_OTHER);
        }
        
        return $this->render('task/new.html.twig', [
            'form' => $form,
        ]);
    }
}
```

### Template (`templates/task/new.html.twig`)

```twig
{% extends 'base.html.twig' %}

{% block body %}
    <div id="task-list">
        {# Liste des tâches existantes #}
    </div>

    <div id="task-form">
        {% block task_form %}
            {{ form_start(form, {'attr': {'id': 'new-task-form'}}) }}
                {{ form_widget(form) }}
                <button type="submit">Ajouter</button>
            {{ form_end(form) }}
        {% endblock %}
    </div>

    {# Bloc pour les Turbo Streams #}
    {% block success_stream %}
        {# Ajouter la nouvelle tâche à la liste #}
        <turbo-stream action="append" target="task-list">
            <template>
                <div class="task">
                    {{ task.title }}
                </div>
            </template>
        </turbo-stream>
        
        {# Réinitialiser le formulaire #}
        <turbo-stream action="replace" targets="form[name=task]">
            <template>
                {% block task_form %}
                    {{ form(form) }}
                {% endblock %}
            </template>
        </turbo-stream>
    {% endblock %}
{% endblock %}
```

## Exemple 2 : Gestion des erreurs de validation

### Contrôleur avec validation [symfony](https://symfony.com/bundles/ux-turbo/current/index.html)

```php
#[Route('/product/new', name: 'product_new')]
public function newProduct(Request $request): Response
{
    $form = $this->createForm(ProductFormType::class);
    $form->handleRequest($request);
    
    if ($form->isSubmitted() && $form->isValid()) {
        // Sauvegarder le produit
        
        return $this->redirectToRoute('product_list');
    }
    
    // Retourner un code 422 si le formulaire est soumis mais invalide
    $response = new Response(null, $form->isSubmitted() ? 422 : 200);
    
    return $this->render('product/new.html.twig', [
        'form' => $form->createView()
    ], $response);
}
```

### Template (`templates/product/new.html.twig`)

```twig
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Nouveau produit</h1>
    
    {{ form_start(form, {
        'attr': {
            'action': path('product_new')
        }
    }) }}
        {{ form_widget(form) }}
        <button type="submit">Créer</button>
    {{ form_end(form) }}
{% endblock %}
```

## Exemple 3 : Turbo Frame pour formulaire modal [symfonycasts](https://symfonycasts.com/screencast/turbo/modal-frame)

### Template avec Turbo Frame

```twig
<turbo-frame id="product-form-modal">
    {{ form_start(form, {
        'attr': {
            'action': path('product_new')
        }
    }) }}
        {{ form_widget(form) }}
        <button type="submit">Enregistrer</button>
    {{ form_end(form) }}
</turbo-frame>
```

### Contrôleur

```php
#[Route('/product/new', name: 'product_new')]
public function newProduct(Request $request): Response
{
    // Détection de la requête depuis un Turbo Frame
    $frameId = $request->headers->get('Turbo-Frame');
    
    $form = $this->createForm(ProductFormType::class);
    $form->handleRequest($request);
    
    if ($form->isSubmitted() && $form->isValid()) {
        // Sauvegarder le produit
        
        return $this->redirectToRoute('product_list');
    }
    
    return $this->render('product/new.html.twig', [
        'form' => $form,
    ]);
}
```

## Points clés [symfonycasts](https://symfonycasts.com/screencast/turbo/turbo-stream)

1. **Détection Turbo** : Utilisez `TurboBundle::STREAM_FORMAT === $request->getPreferredFormat()` pour détecter les requêtes Turbo [symfony](https://symfony.com/bundles/ux-turbo)
2. **Code de statut** : Retournez 422 pour les formulaires invalides afin que Turbo gère correctement les erreurs [symfony](https://symfony.com/bundles/ux-turbo/current/index.html)
3. **Action dans le form** : Spécifiez toujours l'attribut `action` pour les formulaires dans des Turbo Frames [symfonycasts](https://symfonycasts.com/screencast/turbo/frame-form-action)
4. **Turbo Streams** : Utilisez les actions `append`, `prepend`, `replace`, `update`, `remove` pour manipuler le DOM [symfonycasts](https://symfonycasts.com/screencast/turbo/turbo-stream)
