Pour gérer les variables globales dans un contexte de migration avec Symfony Legacy Route Loader, il faut déclarer ces variables avant que le contrôleur ne charge les scripts legacy. Voici comment procéder efficacement. [devblog.pedro.resende](https://devblog.pedro.resende.biz/how-to-migrate-a-legacy-application-to-symfony--)

## Déclaration des variables globales

La clé est de déclarer vos variables globales **avant** le `ob_start()` et avant que le script legacy ne soit chargé. Dans votre `LegacyController`, vous devez initialiser les variables globales nécessaires : [devblog.pedro.resende](https://devblog.pedro.resende.biz/how-to-migrate-a-legacy-application-to-symfony--)

```php
// src/Controller/LegacyController.php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class LegacyController
{
    public function loadLegacyScript(Request $request, string $legacyScript): Response
    {
        // Déclarer les variables globales AVANT ob_start()
        global $myGlobalVar, $anotherGlobal, $config;
        
        // Initialiser si nécessaire
        $myGlobalVar = 'valeur';
        $anotherGlobal = ['data' => 'value'];
        
        // Configurer les variables serveur
        $_SERVER['PHP_SELF'] = $request->getPathInfo();
        $_SERVER['SCRIPT_NAME'] = $request->getPathInfo();
        $_SERVER['SCRIPT_FILENAME'] = $legacyScript;
        
        ob_start();
        require $legacyScript;
        $content = ob_get_clean();
        
        return new Response($content);
    }
}
```

## Centraliser les variables globales

Pour une meilleure organisation, créez un fichier dédié qui définit toutes vos variables globales  : [github](https://github.com/lfjeff/Symfony-Legacy-App)

```php
// src/Legacy/legacy_globals.php
global $myGlobalVar;
global $anotherGlobal;
global $config;
global $dbConnection;

// Initialisation des valeurs par défaut si nécessaire
$config = $config ?? [];
```

Puis incluez ce fichier dans votre contrôleur :

```php
public function loadLegacyScript(Request $request, string $legacyScript): Response
{
    // Charger toutes les déclarations globales
    require __DIR__ . '/../Legacy/legacy_globals.php';
    
    // Configurer les variables serveur
    $_SERVER['PHP_SELF'] = $request->getPathInfo();
    $_SERVER['SCRIPT_NAME'] = $request->getPathInfo();
    $_SERVER['SCRIPT_FILENAME'] = $legacyScript;
    
    ob_start();
    require $legacyScript;
    $content = ob_get_clean();
    
    return new Response($content);
}
```

## Passer des valeurs depuis Symfony

Si vous avez besoin de passer des valeurs depuis votre application Symfony vers les variables globales legacy :

```php
public function loadLegacyScript(
    Request $request, 
    string $legacyScript,
    ContainerInterface $container
): Response {
    global $config, $user;
    
    // Injecter des valeurs depuis Symfony
    $config = [
        'db_host' => $container->getParameter('database.host'),
        'app_env' => $container->getParameter('kernel.environment'),
    ];
    
    // Récupérer l'utilisateur Symfony si authentifié
    $user = $this->getUser();
    
    $_SERVER['PHP_SELF'] = $request->getPathInfo();
    $_SERVER['SCRIPT_FILENAME'] = $legacyScript;
    
    ob_start();
    require $legacyScript;
    $content = ob_get_clean();
    
    return new Response($content);
}
```

## Points importants

**Contexte d'exécution** : Le script legacy s'exécute désormais dans le scope du contrôleur plutôt que dans le scope global, d'où la nécessité de déclarer explicitement les variables avec `global`. [symfony](https://symfony.com/doc/current/migration.html)

**Refactoring progressif** : L'utilisation de variables globales est considérée comme un anti-pattern, et la documentation Symfony recommande de les refactoriser progressivement pour permettre la coexistence des deux systèmes. [aksymfony.readthedocs](https://aksymfony.readthedocs.io/en/5.4/migration.html)

**Sessions partagées** : Pour partager les sessions entre Symfony et votre code legacy, consultez la documentation sur le Bridge de session Symfony. [devblog.pedro.resende](https://devblog.pedro.resende.biz/how-to-migrate-a-legacy-application-to-symfony--)


Non, je n'ai rien inventé. L'article que j'ai consulté confirme explicitement la solution pour les variables globales. [devblog.pedro.resende](https://devblog.pedro.resende.biz/how-to-migrate-a-legacy-application-to-symfony--)

Dans l'article "How to migrate a Legacy application to Symfony" par Pedro Resende, il mentionne textuellement les problèmes rencontrés avec cette approche, dont :

- "Concerning global variables inside the legacy application"

Et la solution est décrite ainsi : **"The Global variables, you'll have to declare them before starting the ob_start() and then it will work perfectly"**. [devblog.pedro.resende](https://devblog.pedro.resende.biz/how-to-migrate-a-legacy-application-to-symfony--)

C'est exactement ce que je t'ai expliqué : déclarer les variables globales avec le mot-clé `global` avant le `ob_start()` dans ton contrôleur legacy.

L'approche générale que j'ai détaillée (créer un `LegacyController`, utiliser `ob_start()` et `require` pour charger les scripts legacy, gérer les variables `$_SERVER`) est également documentée dans cet article. [devblog.pedro.resende](https://devblog.pedro.resende.biz/how-to-migrate-a-legacy-application-to-symfony--)

Les exemples de code que j'ai fournis sont des adaptations de cette approche pour Symfony moderne (la version dans l'article date de 2017 et utilise Symfony 3), mais le principe fondamental reste identique : **déclarer explicitement les variables globales dans le scope du contrôleur avant de charger le script legacy**.
