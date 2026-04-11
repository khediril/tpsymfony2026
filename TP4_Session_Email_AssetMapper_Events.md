# TP4 — Sessions, Email, AssetMapper et Événements

**Module** : Développement Web — Framework PHP  
**Durée** : 3 heures  


---

## 🎯 Objectifs pédagogiques

À l'issue de ce TP, l'étudiant sera capable de :

1.  Gérer les **assets** (CSS, JS, Images) de manière moderne avec **AssetMapper**
2.  Manipuler les **Sessions** utilisateur pour stocker des données temporaires
3.  Écrire des **requêtes personnalisées** complexes avec le **QueryBuilder**
4.  Configurer et utiliser le composant **Mailer** pour l'envoi d'emails
5.  Automatiser des tâches via les **Événements** et les **Subscribers**

---

## 📋 Sommaire

| Partie | Contenu | Durée estimée |
|--------|---------|---------------|
| 1 | AssetMapper : Le Frontend moderne | 40 min |
| 2 | Sessions : Persistance temporaire | 30 min |
| 3 | QueryBuilder : Requêtes sur mesure | 40 min |
| 4 | Mailer : Communication par email | 40 min |
| 5 | Events : Automatisation par Subscribers | 30 min |

---

## ⚙️ Préparation

Reprenez votre projet `tp1_symfony`.

### 🔀 Workflow Git : Branche pour les fonctionnalités avancées

```bash
git checkout main
git pull origin main
git checkout -b feature-advanced-tools
```

---

## Partie 1 — AssetMapper : Le Frontend moderne (40 min)

Depuis Symfony 6.3, **AssetMapper** est l'outil par défaut pour gérer les assets sans avoir besoin de Node.js ou de Webpack.

### 1.1 Installation et structure

AssetMapper utilise un système appelé **Importmap**. Vérifiez que vous avez le bundle :

```bash
composer require symfony/asset-mapper
```

Explorez le dossier `assets/`. Vous y trouverez `app.js` et `styles/app.css`.

### 1.2 Ajouter une bibliothèque externe (ex: SweetAlert2)

Avec AssetMapper, plus besoin de `npm install`. Utilisez la commande dédiée :

```bash
php bin/console importmap:require sweetalert2
```

Cela ajoute la bibliothèque dans `importmap.php` et la rend disponible dans vos fichiers JS.

### 1.3 Utilisation dans Twig

Dans `base.html.twig`, notez l'utilisation de la fonction `importmap()` :

```twig
<head>
    {{ importmap('app') }}
</head>
```

## Partie 2 — Sessions : Persistance temporaire (30 min)

La session permet de conserver des informations d'une page à l'autre (ex: un panier, les derniers articles consultés).

### 2.1 Utilisation de la Session

Dans un contrôleur, vous pouvez accéder à la session via l'objet `Request` ou en injectant `RequestStack`.

Exemple : Enregistrer le nom de l'utilisateur dans la session lors de sa visite.

```php
use Symfony\Component\HttpFoundation\RequestStack;

public function index(RequestStack $requestStack): Response
{
    $session = $requestStack->getSession();
    
    // Récupérer une donnée
    $nbVisites = $session->get('nb_visites', 0);
    
    // Stocker une donnée
    $session->set('nb_visites', $nbVisites + 1);
    
    return $this->render(...);
}
```

---

## Partie 3 — QueryBuilder : Requêtes sur mesure (40 min)

Parfois, `findAll()` ou `findBy()` ne suffisent pas. On utilise alors le **QueryBuilder** dans le Repository.

### 3.1 Créer une méthode personnalisée

Modifiez `src/Repository/ArticleRepository.php`. Ajoutons une méthode pour récupérer les 3 derniers articles publiés :

```php
public function findLastPublished(int $limit): array
{
    return $this->createQueryBuilder('a')
        ->andWhere('a.publie = :val')
        ->setParameter('val', true)
        ->orderBy('a.id', 'DESC')
        ->setMaxResults($limit)
        ->getQuery()
        ->getResult();
}
```

### 3.2 Utilisation dans le contrôleur

```php
$derniersArticles = $repository->findLastPublished(3);
```

#### ✏️ Question 2
> Pourquoi est-il préférable d'utiliser `setParameter()` plutôt que de concaténer directement une variable dans la requête ? (Concept de **JS Injection** / **SQL Injection**).

---

## Partie 4 — Mailer : Communication par email (40 min)

### 4.1 Installation

```bash
composer require symfony/mailer
```

### 4.2 Configuration (Mailtrap pour le test)

Dans le fichier `.env`, configurez votre DSN de transport (ex: Mailtrap) :
`MAILER_DSN=smtp://user:pass@smtp.mailtrap.io:2525?encryption=tls&auth_mode=login`

### 4.3 Envoyer un email

Dans le contrôleur, injectez `MailerInterface`.

```php
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Email;

public function sendEmail(MailerInterface $mailer): Response
{
    $email = (new Email())
        ->from('hello@example.com')
        ->to('you@example.com')
        ->subject('Nouvel Article !')
        ->text('Un nouvel article a été publié sur le blog.')
        ->html('<p>Un nouvel article a été publié sur le blog.</p>');

    $mailer->send($email);
    // ...
}
```

---

## Partie 5 — Events : Automatisation par Subscribers (30 min)

Les **Event Subscribers** permettent d'écouter des événements système ou personnalisés.

### 5.1 Créer un Subscriber

Générez un subscriber qui écoute l'événement de création d'un article :

```bash
php bin/console make:subscriber ArticleNotificationSubscriber
```

### 5.2 Exemple : Envoyer un email après chaque création

Modifiez le subscriber pour écouter l'événement Doctrine `PostPersist` (ou un événement kernel). Pour simplifier, nous allons écouter le moment où une réponse est envoyée :

```php
namespace App\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\ResponseEvent;
use Symfony\Component\HttpKernel\KernelEvents;

class ArticleNotificationSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [
            KernelEvents::RESPONSE => 'onKernelResponse',
        ];
    }

    public function onKernelResponse(ResponseEvent $event): void
    {
        // Logique : ajouter un header personnalisé à toutes les réponses
        $event->getResponse()->headers->set('X-Symfony-TP', 'TP5-Finished');
    }
}
```

---

## 📝 Récapitulatif des commandes

```bash
# Gérer les assets
php bin/console importmap:require <package>

# Debugger les événements
php bin/console debug:event-dispatcher

# Debugger le mailer
php bin/console debug:autowiring mailer
```

