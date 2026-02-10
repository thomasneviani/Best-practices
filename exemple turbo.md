Oui, c'est tout à fait possible ! Voici comment remplacer un formulaire par un message de succès en utilisant Turbo Stream: [github](https://github.com/hotwired/turbo/issues/138)

## Exemple avec Turbo Stream

### Contrôleur

```php
<?php

namespace App\Controller;

use App\Form\ContactType;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\UX\Turbo\TurboBundle;

class ContactController extends AbstractController
{
    #[Route('/contact', name: 'contact')]
    public function contact(Request $request): Response
    {
        $form = $this->createForm(ContactType::class);
        $form->handleRequest($request);
        
        if ($form->isSubmitted() && $form->isValid()) {
            $data = $form->getData();
            
            // Traitement du formulaire (envoi d'email, etc.)
            // ...
            
            // Si c'est une requête Turbo, on envoie un Turbo Stream
            if (TurboBundle::STREAM_FORMAT === $request->getPreferredFormat()) {
                $request->setRequestFormat(TurboBundle::STREAM_FORMAT);
                
                return $this->render('contact/success_stream.html.twig', [
                    'message' => 'Votre message a été envoyé avec succès !'
                ]);
            }
            
            // Redirection classique pour les requêtes non-Turbo
            $this->addFlash('success', 'Votre message a été envoyé avec succès !');
            return $this->redirectToRoute('contact');
        }
        
        return $this->render('contact/index.html.twig', [
            'form' => $form,
        ]);
    }
}
```

### Template principal (`templates/contact/index.html.twig`)

```twig
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Contactez-nous</h1>
    
    <div id="contact-form-container">
        {{ form_start(form, {
            'attr': {
                'id': 'contact-form'
            }
        }) }}
            {{ form_widget(form) }}
            <button type="submit">Envoyer</button>
        {{ form_end(form) }}
    </div>
{% endblock %}
```

### Template Turbo Stream (`templates/contact/success_stream.html.twig`)

```twig
{# Remplace tout le container du formulaire par le message de succès #}
<turbo-stream action="replace" target="contact-form-container">
    <template>
        <div id="contact-form-container" class="success-message">
            <div class="alert alert-success">
                ✓ {{ message }}
            </div>
        </div>
    </template>
</turbo-stream>
```

## Alternative : Remplacer uniquement le formulaire

Si vous voulez garder le container mais remplacer seulement le formulaire: [discuss.hotwired](https://discuss.hotwired.dev/t/resetting-form-after-turbo-streams-submission/3615)

### Template avec cible spécifique

```twig
<div id="contact-form-container">
    <h2>Formulaire de contact</h2>
    
    <div id="form-content">
        {{ form_start(form, {
            'attr': {
                'id': 'contact-form'
            }
        }) }}
            {{ form_widget(form) }}
            <button type="submit">Envoyer</button>
        {{ form_end(form) }}
    </div>
</div>
```

### Template Turbo Stream

```twig
<turbo-stream action="replace" target="form-content">
    <template>
        <div id="form-content">
            <div class="alert alert-success">
                <h3>✓ Message envoyé !</h3>
                <p>{{ message }}</p>
                <a href="{{ path('home') }}" class="btn btn-primary">
                    Retour à l'accueil
                </a>
            </div>
        </div>
    </template>
</turbo-stream>
```

## Exemple avec action `update` (contenu uniquement)

Si vous voulez changer uniquement le contenu HTML sans modifier la balise cible: [stackoverflow](https://stackoverflow.com/questions/76474312/how-to-remove-element-from-specific-div-using-turbo-stream)

```twig
<turbo-stream action="update" target="contact-form-container">
    <template>
        <div class="success-message">
            <svg class="success-icon" width="64" height="64">
                <circle cx="32" cy="32" r="30" fill="#4CAF50"/>
                <path d="M20 32 L28 40 L44 24" stroke="white" stroke-width="4" fill="none"/>
            </svg>
            <h2>Message envoyé avec succès !</h2>
            <p>Nous vous répondrons dans les plus brefs délais.</p>
        </div>
    </template>
</turbo-stream>
```

## Actions Turbo Stream disponibles [github](https://github.com/hotwired/turbo/issues/138)

- **`replace`** : Remplace l'élément entier (incluant la balise cible)
- **`update`** : Remplace uniquement le contenu HTML interne
- **`remove`** : Supprime complètement l'élément du DOM
- **`append`** : Ajoute le contenu à la fin
- **`prepend`** : Ajoute le contenu au début

L'avantage de cette approche est que le formulaire disparaît et le message de succès apparaît sans rechargement de page, offrant une expérience utilisateur fluide. [blog.skelz0r](https://blog.skelz0r.fr/posts/turbo-form-redirect)
