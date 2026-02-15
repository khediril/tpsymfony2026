# TP4 — API Platform, Services et Injection de Dépendances

**Module** : Développement Web — Framework PHP  
**Durée** : 3 heures  
**Prérequis** : Avoir terminé le TP3 (sécurité, entités, relations)

---

## 🎯 Objectifs pédagogiques

 À l'issue de ce TP, l'étudiant sera capable de :

1.  **Créer une API manuellement** avec `JsonResponse` et le `Serializer` de Symfony
2.  **Transformer** une application en API REST avec **API Platform**
3.  Utiliser les **Groupes de Sérialisation** pour contrôler les données JSON
4.  Créer et configurer des **Services personnalisés**
5.  Comprendre et appliquer l'**Injection de Dépendances**

---

## 📋 Sommaire

| Partie | Contenu | Durée estimée |
|--------|---------|---------------|
| 0 | Fondamentaux : Qu'est-ce qu'une API REST ? | 20 min |
| 1 | L'API "Manuelle" (Full CRUD & Serializer) | 60 min |
| 2 | Installation et découverte d'API Platform | 30 min |
| 3 | Sérialisation et Validation API | 45 min |
| 4 | Création d'un Service personnalisé | 45 min |
| 5 | Injection de dépendances et logique métier | 15 min |

---

## Partie 0 — Fondamentaux : Qu'est-ce qu'une API REST ? (20 min)

### 0.1 Définition d'une API
Une **API** (*Application Programming Interface*) est un ensemble de règles qui permet à deux logiciels de communiquer entre eux. Dans le cas du web, il s'agit souvent d'un "contrat" entre un **Client** (ex: une application React, mobile) et un **Serveur** (notre projet Symfony).

### 0.2 Architecture REST
**REST** (*REpresentational State Transfer*) est un style d'architecture basé sur des concepts clés :
-   **Ressource** : Tout objet manipulable (Article, Utilisateur, Catégorie). Chaque ressource possède une adresse unique : l'**URL**.
-   **Endpoint (Point d'entrée)** : L'adresse URL spécifique (ex: `/api/v1/articles`).
-   **Verbes HTTP (Méthodes)** : Définit l'action à effectuer :
    -   `GET` : Récupérer des données.
    -   `POST` : Créer une ressource.
    -   `PUT` : Modifier une ressource (remplacement complet).
    -   `PATCH` : Modifier partiellement une ressource.
    -   `DELETE` : Supprimer une ressource.

### 0.3 Anatomie d'une requête/réponse
-   **Header (En-tête)** : Contient des métadonnées (ex: `Content-Type: application/json`, tokens d'authentification).
-   **Body (Corps)** : Contient les données réelles (souvent en JSON).
-   **Status Code (Code de statut)** : Indique le résultat de la requête :
    -   `200 OK` / `201 Created` : Succès.
    -   `400 Bad Request` : Erreur client (syntaxe).
    -   `401 Unauthorized` / `403 Forbidden` : Problème de droits.
    -   `404 Not Found` : Ressource inexistante.
    -   `500 Internal Server Error` : Erreur serveur.

### 0.4 Normalisation vs Sérialisation
Dans Symfony, transformer un objet PHP en JSON se fait en deux étapes :
1.  **Normalisation** : Transformer un **Objet** complexe en un **Tableau** PHP simple (associatif).
2.  **Sérialisation** : Transformer ce **Tableau** en un format de texte transmissible (**JSON**, XML).
> L'inverse (JSON -> Objet) s'appelle la **Désérialisation**.

---

## ⚙️ Préparation

Reprenez votre projet `tp1_symfony`.

### 🔀 Workflow Git : Branche pour l'API et les Services

```bash
git checkout main
git pull origin main
git checkout -b feature-api-services
```

---

## Partie 1 — L'API "Manuelle" (Full CRUD) (60 min)

Avant d'utiliser un outil automatisé comme API Platform, il est essentiel de comprendre comment Symfony gère nativement les requêtes et les réponses JSON.

### 1.1 Créer un contrôleur API

Générez un contrôleur pour isoler notre logique API :

```bash
php bin/console make:controller Api/ArticleController
```

### 1.2 Implémenter le GET (Liste et Détail)

Modifiez `src/Controller/Api/ArticleController.php` pour retourner des données JSON.

```php
namespace App\Controller\Api;

use App\Entity\Article;
use App\Repository\ArticleRepository;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Serializer\SerializerInterface;

#[Route('/api/v1/articles', name: 'api_articles_')]
class ArticleController extends AbstractController
{
    #[Route('', name: 'list', methods: ['GET'])]
    public function list(ArticleRepository $repository, SerializerInterface $serializer): JsonResponse
    {
        $articles = $repository->findAll();
        $json = $serializer->serialize($articles, 'json', ['groups' => 'article:read']);
        return new JsonResponse($json, Response::HTTP_OK, [], true);
    }

    #[Route('/{id}', name: 'show', methods: ['GET'])]
    public function show(Article $article, SerializerInterface $serializer): JsonResponse
    {
        $json = $serializer->serialize($article, 'json', ['groups' => 'article:read']);
        return new JsonResponse($json, Response::HTTP_OK, [], true);
    }
}
```

### 1.3 Implémenter le POST (Création)

Pour créer un article, nous devons récupérer le contenu du corps de la requête (`body`), le transformer en objet (désérialisation) et le valider.

```php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Validator\Validator\ValidatorInterface;
use Doctrine\ORM\EntityManagerInterface;

#[Route('', name: 'create', methods: ['POST'])]
public function create(
    Request $request, 
    SerializerInterface $serializer, 
    EntityManagerInterface $em,
    ValidatorInterface $validator
): JsonResponse {
    // 1. Désérialiser le JSON vers l'objet Article
    $article = $serializer->deserialize($request->getContent(), Article::class, 'json');
    $article->setDateCreation(new \DateTime());

    // 2. Validation
    $errors = $validator->validate($article);
    if (count($errors) > 0) {
        return $this->json($errors, Response::HTTP_UNPROCESSABLE_ENTITY);
    }

    // 3. Persistance
    $em->persist($article);
    $em->flush();

    return $this->json($article, Response::HTTP_CREATED, [], ['groups' => 'article:read']);
}
```

### 1.4 Implémenter le PUT et DELETE

```php
#[Route('/{id}', name: 'update', methods: ['PUT'])]
public function update(
    Article $article,
    Request $request,
    SerializerInterface $serializer,
    EntityManagerInterface $em
): JsonResponse {
    // On met à jour l'objet existant avec les nouvelles données
    $serializer->deserialize($request->getContent(), Article::class, 'json', [
        'object_to_populate' => $article
    ]);

    $em->flush();
    return $this->json($article, Response::HTTP_OK, [], ['groups' => 'article:read']);
}

#[Route('/{id}', name: 'delete', methods: ['DELETE'])]
public function delete(Article $article, EntityManagerInterface $em): JsonResponse
{
    $em->remove($article);
    $em->flush();
    return new JsonResponse(null, Response::HTTP_NO_CONTENT);
}
```

### 1.5 Les limites de la méthode manuelle

Comme vous pouvez le voir, pour une seule entité, nous avons dû écrire beaucoup de code "répétitif" :
- Routes pour chaque méthode.
- Désérialisation manuelle et gestion des injections.
- Appel manuel du validateur.
- Gestion des codes de statut HTTP (201, 204, 422...).
- Pas de documentation interactive (Swagger).

#### ✏️ Question 1
> Imaginez si vous aviez 20 entités. Combien de méthodes de contrôleur devriez-vous écrire au total avec cette approche ? Quel est le risque majeur en termes de maintenance ?

---

## Partie 2 — API Platform (35 min)

### 2.1 Installation

API Platform est le framework standard pour créer des API avec Symfony.

```bash
composer require api
```

### 2.2 Exposer une Entité

Pour transformer une entité Doctrine en ressource API, il suffit d'ajouter l'attribut `#[ApiResource]`.

Modifiez `src/Entity/Article.php` :

```php
use ApiPlatform\Metadata\ApiResource;
// ...

#[ORM\Entity(repositoryClass: ArticleRepository::class)]
#[ApiResource]
class Article
{
    // ...
}
```

Faites de même pour l'entité `Categorie`.

### 2.3 Découverte de Swagger UI

Ouvrez votre navigateur sur **https://127.0.0.1:8000/api**. 
Vous devriez voir une interface interactive listant vos ressources.

#### ✏️ Question 1
> Testez un `GET` sur `/api/articles`. Quel est le format de la réponse ? (Indice : regardez le `Content-Type`).

#### ✏️ Question 2
> Essayez de créer un article via un `POST`. Pourquoi recevez-vous une erreur si vous n'êtes pas connecté (si vous avez gardé la sécurité du TP3) ?

---

## Partie 3 — Sérialisation et Groupes (45 min)

Par défaut, API Platform expose toutes les propriétés. Pour limiter cela (ex: ne pas exposer le mot de passe d'un utilisateur ou éviter les boucles infinies de relations), on utilise les **Serializer Groups**.

### 3.1 Configurer les groupes sur l'Article

Modifiez `src/Entity/Article.php` :

```php
use Symfony\Component\Serializer\Annotation\Groups;

#[ApiResource(
    normalizationContext: ['groups' => ['article:read']],
    denormalizationContext: ['groups' => ['article:write']],
)]
class Article
{
    #[Groups(['article:read', 'article:write'])]
    private ?string $titre = null;

    #[Groups(['article:read'])]
    private ?\DateTimeInterface $dateCreation = null;

    // ... continuez pour les autres champs
}
```

#### ✏️ Question 3
> Quelle est la différence entre **Normalization** et **Denormalization** ?

### 3.2 Validation API

Les contraintes de validation (`Assert\NotBlank`, etc.) définies au TP2 s'appliquent automatiquement à l'API. Testez l'envoi d'un titre vide en `POST` et observez la réponse 422.

---

## Partie 4 — Services Personnalisés (45 min)

Un **Service** est une classe PHP réutilisable qui effectue une tâche spécifique (logique métier).

### 4.1 Créer un service de "Nettoyage de texte"

Nous allons créer un service qui supprime les mots interdits dans nos articles.

1. Créez le dossier `src/Service`.
2. Créez la classe `TextFormatter.php` :

```php
namespace App\Service;

class TextFormatter
{
    private array $forbiddenWords = ['mauvais', 'nul', 'spam'];

    public function filter(string $text): string
    {
        return str_ireplace($this->forbiddenWords, '****', $text);
    }
}
```

### 4.2 Utiliser le service dans un Contrôleur

Modifiez `src/Controller/ArticlesController.php`. Injectez le service dans la méthode `nouveau` :

```php
use App\Service\TextFormatter;

#[Route('/articles/nouveau', name: 'app_article_nouveau')]
public function nouveau(Request $request, EntityManagerInterface $em, TextFormatter $formatter): Response
{
    // ... après le handleRequest()
    if ($form->isSubmitted() && $form->isValid()) {
        $article->setContenu($formatter->filter($article->getContenu()));
        // ...
    }
}
```

---

## Partie 5 — Injection de Dépendances (15 min)

### 5.1 Injection dans le constructeur (Recommandé)

Plutôt que d'injecter dans chaque méthode, on préfère souvent le constructeur :

```php
private $formatter;

public function __construct(TextFormatter $formatter)
{
    $this->formatter = $formatter;
}
```

#### ✏️ Question 4
> Comment Symfony sait-il quel objet passer au contrôleur ? Qu'est-ce que l'**Autowiring** ?

---

## Partie 5 — Exercice de synthèse (20 min)

### 🧩 Objectif : Un service de calcul de temps de lecture

1. Créer un service `ReadingTimeCalculator` qui prend une chaîne de caractères et retourne un entier (nombre de minutes estimé, ex: 200 mots/min).
2. Déclarer ce service dans `services.yaml` (si l'autowiring ne suffit pas, mais ici il suffira).
3. Afficher le temps de lecture dans le template `detail.html.twig`.
    > **Tip** : Vous pouvez créer une **Extension Twig** pour appeler ce service directement dans vos templates ! (Utilisez `php bin/console make:twig-extension`).

---

## 📝 Récapitulatif des commandes

```bash
# Installation API
composer require api

# Créer une extension Twig
php bin/console make:twig-extension

# Voir tous les services disponibles dans le conteneur
php bin/console debug:autowiring
```

---

## ✅ Critères d'évaluation

| Critère | Points |
|---------|--------|
| API Platform installé et interface Swagger accessible | /3 |
| Entités Article et Categorie exposées en lecture/écriture | /4 |
| Mise en place correcte des groupes de sérialisation | /4 |
| Service TextFormatter créé et fonctionnel | /3 |
| Exercice de synthèse (ReadingTime) réussi | /4 |
| Réponses aux questions théoriques | /2 |
| **Total** | **/20** |
