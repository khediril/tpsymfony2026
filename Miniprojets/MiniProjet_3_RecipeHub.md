# Mini-Projet 3 — RecipeHub : Plateforme de Partage de Recettes

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation en binôme  
**Durée estimée** : 16 à 20 heures  
**Barème** : /20

---

## Contexte

Vous devez développer **RecipeHub**, une plateforme de partage de recettes de cuisine. Les utilisateurs peuvent parcourir les recettes, en créer, les ajouter à leurs favoris (session), et recevoir un email de notification. L'application expose une API REST.

---

## Partie 1 — Modélisation et Base de données (3 pts)

> **Concepts** : Doctrine ORM, Entités, Migrations, Relations, Validation (TP1 + TP2)

### Entités

**Recette** : `titre` (string 255, NotBlank, Length min:5), `description` (text, NotBlank, Length min:30), `instructions` (text, NotBlank), `tempsPreparation` (integer, Range min:1), `tempsCuisson` (integer, nullable), `difficulte` (string, Choice: facile/moyen/difficile), `nbPersonnes` (integer, Range min:1 max:50), `dateCreation` (datetime, auto), `publiee` (boolean), `imageName` (string 255, nullable)

**Ingredient** : `nom` (string 100, NotBlank), `quantite` (string 50, NotBlank — ex: "200g")

**CategorieRecette** : `nom` (string 50, NotBlank, Unique), `icone` (string 10, nullable — emoji), `description` (text, nullable)

**TagRecette** *(nouveau)* : `nom` (string 50, NotBlank, Unique), `couleur` (string 7, Regex: code hex)

### Relations

- CategorieRecette → Recette : **OneToMany**
- Recette → Ingredient : **OneToMany**
- User → Recette : **ManyToOne** (auteur)
- Recette ↔ TagRecette : **ManyToMany** — Une recette peut avoir plusieurs tags (Végétarien, Sans gluten, etc.), un tag peut être associé à plusieurs recettes

---

## Partie 2 — CRUD, Formulaires et Templates (4 pts)

> **Concepts** : Contrôleurs, Routes, FormType, EntityType, ChoiceType, Twig, Flash, CSRF (TP1 + TP2)

### CRUD Recette (complet)

| Action | Route | Méthode |
|--------|-------|---------|
| Liste | `/recettes` | GET |
| Détail | `/recettes/{id}` | GET |
| Créer | `/recettes/nouvelle` | GET/POST |
| Modifier | `/recettes/{id}/modifier` | GET/POST |
| Supprimer | `/recettes/{id}/supprimer` | POST (CSRF) |

### CRUD Ingrédient (simplifié)

- Ajouter un ingrédient : `/recettes/{id}/ingredients/nouveau`
- Supprimer un ingrédient : `/ingredients/{id}/supprimer` (POST CSRF)

### CRUD CategorieRecette : liste + création

### CRUD TagRecette *(nouveau)* : liste + création + suppression

### FormTypes

- **RecetteType** : `EntityType` (catégorie), `ChoiceType` (difficulté), `CheckboxType` (publiée), `EntityType` (tags, `multiple: true`, `expanded: true`, `by_reference => false`), `FileType` (image, `mapped => false`, JPEG/PNG/WebP, max 2 Mo)
- **IngredientType** : nom, quantité

### Templates Twig

- Layout Bootstrap 5 avec navbar et footer
- Liste : Cards en grille responsive, badges difficulté (🟢 Facile | 🟡 Moyen | 🔴 Difficile), icône catégorie, temps total, **image de la recette**, badges des tags associés
- Détail : Deux colonnes (ingrédients à gauche, instructions à droite), **image en grand**, liste des tags
- Messages flash après chaque opération

---

## Partie 3 — Sécurité et Authentification (3 pts)

> **Concepts** : User, Inscription, Login/Logout, Rôles, Hiérarchie, IsGranted, Propriété (TP3)

- User avec `make:user` + champ `pseudo`
- Inscription (`make:registration-form`) et Login/Logout (`make:auth`)

### Hiérarchie : `ROLE_CUISINIER` → `ROLE_ADMIN`

| Fonctionnalité | Accès |
|----------------|-------|
| Consulter les recettes publiées | Tout le monde |
| Créer une recette | `ROLE_CUISINIER` |
| Modifier / Supprimer | **Auteur** ou `ROLE_ADMIN` |
| Gérer catégories et tags | `ROLE_ADMIN` |

- `#[IsGranted]` sur les contrôleurs, `is_granted()` dans Twig
- Auteur automatique : `$recette->setAuteur($this->getUser())`
- Navigation conditionnelle (pseudo, connexion/déconnexion)

---

## Partie 4 — API REST et Services (4 pts)

> **Concepts** : API Platform, Groupes de sérialisation, Swagger, Service, Injection (TP4)

### API Platform

Exposer **Recette** et **CategorieRecette** avec `#[ApiResource]`.

### Groupes de sérialisation (Recette)

- `recette:read` : titre, description, instructions, temps, difficulté, nbPersonnes, dateCreation, catégorie (nom), ingrédients, tags (noms)
- `recette:write` : titre, description, instructions, temps, difficulté, nbPersonnes

Tester via Swagger UI : GET, POST, PUT, DELETE.

### Service `RecetteAnalyser`

```php
class RecetteAnalyser
{
    public function __construct(private RecetteRepository $repo) {}
    public function getTempsTotal(Recette $r): int          // prep + cuisson
    public function getTotalRecettesPubliees(): int
    public function getRecettesParCategorie(): array         // tableau associatif
    public function getMoyenneIngredients(): float
}
```

Injecter dans le contrôleur (constructeur), afficher les stats sur la page d'accueil.

---

## Partie 5 — Sessions, QueryBuilder et AssetMapper (3 pts)

> **Concepts** : Sessions (RequestStack), QueryBuilder, AssetMapper (TP5)

### Session — Recettes favorites

- Bouton « ⭐ Ajouter aux favoris » sur la page de détail
- Stocker les IDs en session via `RequestStack`
- Route `/mes-favoris` affichant les recettes favorites
- Retirer une recette (bouton ❌), badge dynamique dans la navbar, pas de doublons

### QueryBuilder — Recherche avancée

Recherche sur la page de liste : par **titre** (partiel), par **catégorie** (EntityType), par **difficulté** (ChoiceType), par **tag** (EntityType) *(nouveau)*.

```php
// RecetteRepository
public function findByFilters(?string $titre, ?CategorieRecette $cat, ?string $diff, ?TagRecette $tag): array
{
    $qb = $this->createQueryBuilder('r');

    if ($titre) {
        $qb->andWhere('r.titre LIKE :titre')
           ->setParameter('titre', '%' . $titre . '%');
    }
    if ($cat) {
        $qb->andWhere('r.categorie = :cat')
           ->setParameter('cat', $cat);
    }
    if ($diff) {
        $qb->andWhere('r.difficulte = :diff')
           ->setParameter('diff', $diff);
    }
    if ($tag) {
        $qb->innerJoin('r.tags', 't')
           ->andWhere('t = :tag')
           ->setParameter('tag', $tag);
    }

    return $qb->orderBy('r.dateCreation', 'DESC')
              ->getQuery()->getResult();
}

public function findLastPublished(int $limit = 3): array
```

### AssetMapper

- `importmap:require sweetalert2` pour les confirmations de suppression
- Utiliser dans au minimum 2 pages
- `{{ importmap('app') }}` dans `base.html.twig`

---

## Partie 6 — Email et Événements (3 pts)

> **Concepts** : Mailer (Mailtrap), TemplatedEmail, Event Subscriber (TP5)

### Email de notification

Quand une recette est publiée → email via Mailtrap :
- De : `noreply@recipehub.com`
- Sujet : « 🍽️ Nouvelle recette : {titre} »
- Template `emails/nouvelle_recette.html.twig` : titre, catégorie, difficulté, temps, auteur

### Event Subscriber

```php
class RecipeHubSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [KernelEvents::RESPONSE => 'onKernelResponse'];
    }
    public function onKernelResponse(ResponseEvent $event): void
    {
        $event->getResponse()->headers->set('X-RecipeHub-Version', '1.0');
    }
}
```

---

## 🆕 Partie 7 — Upload d'images (2 pts)

> **Concepts évalués** : Upload de fichiers, Service FileUploader, gestion des fichiers physiques (TP7)

### 7.1 Service FileUploader

Créez `src/Service/FileUploader.php` :

```php
class FileUploader
{
    public function __construct(
        private string $targetDirectory,
        private SluggerInterface $slugger,
    ) {}

    public function upload(UploadedFile $file): string
    {
        $originalFilename = pathinfo($file->getClientOriginalName(), PATHINFO_FILENAME);
        $safeFilename = $this->slugger->slug($originalFilename);
        $fileName = $safeFilename . '-' . uniqid() . '.' . $file->guessExtension();

        $file->move($this->getTargetDirectory(), $fileName);
        return $fileName;
    }

    public function remove(string $fileName): void
    {
        $filePath = $this->getTargetDirectory() . '/' . $fileName;
        if (file_exists($filePath)) {
            unlink($filePath);
        }
    }

    public function getTargetDirectory(): string
    {
        return $this->targetDirectory;
    }
}
```

### 7.2 Configuration

Dans `config/services.yaml` :

```yaml
parameters:
    upload_directory: '%kernel.project_dir%/public/uploads/recettes'

services:
    App\Service\FileUploader:
        arguments:
            $targetDirectory: '%upload_directory%'
```

### 7.3 Intégration dans le contrôleur

- À la **création** d'une recette : si une image est fournie, l'uploader et stocker le nom dans `imageName`
- À la **modification** : supprimer l'ancienne image avant d'enregistrer la nouvelle
- À la **suppression** : supprimer le fichier physique associé
- Afficher l'image dans le **détail** (grande taille) et en **miniature dans les cards** de la liste per
- Les cards sans image affichent un **placeholder** avec l'icône de la catégorie

---

## 🆕 Partie 8 — DataFixtures et Pagination (2 pts)

> **Concepts évalués** : DataFixtures, FakerPHP, KnpPaginatorBundle, Références entre fixtures (TP6)

### 8.1 DataFixtures avec FakerPHP

Créez des fixtures séparées par entité avec le système de **références** et `DependentFixtureInterface` :

#### `CategorieRecetteFixtures.php`
- 6 catégories prédéfinies avec icônes (🥗 Entrée, 🍝 Plat, 🍰 Dessert, 🥤 Boisson, 🍕 Snack, 🥣 Soupe)

#### `TagRecetteFixtures.php`
- 8 tags prédéfinis avec couleurs (Végétarien, Végan, Sans Gluten, Bio, Rapide, Familial, Festif, Économique)

#### `UserFixtures.php`
- 1 admin (`admin@recipehub.com` / `admin123`, `ROLE_ADMIN`)
- 1 cuisinier (`chef@recipehub.com` / `chef123`, `ROLE_CUISINIER`)
- 5 utilisateurs générés par Faker (`ROLE_USER`)
- Utiliser `UserPasswordHasherInterface` pour hasher les mots de passe

#### `RecetteFixtures.php` (dépend de toutes les autres)
- 20 recettes avec titres et descriptions culinaires réalistes
- Chaque recette associée à une catégorie, un auteur aléatoire
- Chaque recette associée à 1 à 4 tags aléatoires (ManyToMany)
- Temps de préparation/cuisson et difficulté variés

#### `IngredientFixtures.php` (dépend de RecetteFixtures)
- 3 à 8 ingrédients par recette (générés par Faker)

### 8.2 Pagination avec KnpPaginatorBundle

- Installer `knplabs/knp-paginator-bundle`
- Paginer la **liste des recettes** : 9 recettes par page (grille 3×3)
- Ajouter le **tri** par titre, date de création et temps de préparation via `knp_pagination_sortable()`
- Afficher le **nombre total de résultats**
- Paginer la **page des favoris** : 6 recettes par page

---

## 🆕 Partie 9 — Extension Twig et Commande Console (2 pts)

> **Concepts évalués** : Extensions Twig personnalisées, Commandes Console, SymfonyStyle (TP7)

### 9.1 Extension Twig `RecipeHubExtension`

Créez `src/Twig/RecipeHubExtension.php` avec :

#### Filtre `time_ago`
Convertit une date en format relatif :
```twig
{{ recette.dateCreation|time_ago }}
{# Affiche : "il y a 3 jours", "il y a 2 mois", etc. #}
```

#### Filtre `cooking_time_format`
Formate le temps de cuisson en heures et minutes :
```twig
{{ recette.tempsPreparation|cooking_time_format }}
{# 90 → "1h30", 45 → "45min", 120 → "2h" #}
```

#### Fonction `difficulty_stars`
Génère des étoiles pour la difficulté :
```twig
{{ difficulty_stars(recette.difficulte) }}
{# facile → ⭐, moyen → ⭐⭐, difficile → ⭐⭐⭐ #}
```

Utiliser ces filtres/fonctions dans au moins **3 templates** différents.

### 9.2 Commande Console `app:recipehub:stats`

Créez `src/Command/RecipeHubStatsCommand.php` :

```php
#[AsCommand(
    name: 'app:recipehub:stats',
    description: 'Affiche les statistiques de la plateforme de recettes',
)]
class RecipeHubStatsCommand extends Command
{
    // Injection de RecetteRepository, CategorieRecetteRepository, IngredientRepository

    // Option --detail : affiche le détail par catégorie
    // Option --top=N : affiche le top N des recettes les plus longues
    
    // Afficher :
    // - Nombre total de recettes (publiées / brouillons)
    // - Nombre d'ingrédients total
    // - Répartition par catégorie
    // - Répartition par difficulté
    // - Temps de préparation moyen
    // - Top 3 des auteurs les plus prolifiques
    // Avec SymfonyStyle : title(), table(), success()
}
```

Exécution :
```bash
php bin/console app:recipehub:stats
php bin/console app:recipehub:stats --detail
php bin/console app:recipehub:stats --top=5
```

---

## 🆕 Partie 10 — Tests (2 pts)

> **Concepts évalués** : PHPUnit, Tests unitaires, Tests fonctionnels, Tests API (TP6)

### 10.1 Tests unitaires du service `RecetteAnalyser`

Créez `tests/Service/RecetteAnalyserTest.php` :

- Tester `getTempsTotal()` additionne correctement préparation et cuisson
- Tester `getTempsTotal()` gère le cas où `tempsCuisson` est null (retourne uniquement `tempsPreparation`)
- Tester `getTotalRecettesPubliees()` ne compte que les recettes publiées
- Tester `getMoyenneIngredients()` retourne 0 si aucune recette

### 10.2 Tests fonctionnels des contrôleurs

Créez `tests/Controller/RecetteControllerTest.php` :

- Tester que la page `/recettes` retourne un code **200**
- Tester que la page `/recettes` contient des cards de recettes
- Tester que la page `/recettes/nouvelle` est **interdite** sans authentification (code **302**)
- Tester la **création d'une recette** via soumission de formulaire et vérifier la redirection
- Tester que le **message flash** apparaît après création

### 10.3 Tests de l'API

Créez `tests/Api/RecetteApiTest.php` :

- Tester `GET /api/recettes` retourne **200** et le bon content-type (`application/ld+json`)
- Tester `POST /api/recettes` avec des données valides retourne **201**
- Tester `POST /api/recettes` avec un titre vide retourne **422**

### 10.4 Configuration de test

- Créer `.env.test` avec une base de données de test séparée
- Les fixtures doivent être chargées avant l'exécution des tests

---

## 📦 Livrables

- ✅ Dépôt **GitHub** avec commits structurés
- ✅ `README.md` : installation, config Mailtrap, identifiants de test, schéma des relations
- ✅ Fixtures fonctionnelles : `php bin/console doctrine:fixtures:load` charge toutes les données
- ✅ Base peuplée via fixtures : 6 catégories, 8 tags, 20 recettes avec ingrédients, 7 utilisateurs
- ✅ Tests exécutables : `php bin/phpunit` passe sans erreurs

---

## Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations (OneToMany + ManyToMany) | /3 |
| **CRUD & Templates** : FormTypes, EntityType, ChoiceType, Twig, Flash, CSRF | /4 |
| **Sécurité** : Inscription, Login, Rôles, Hiérarchie, Propriété | /3 |
| **API & Services** : API Platform, Groupes, Swagger, Service injecté | /4 |
| **Session, QueryBuilder, AssetMapper** : Favoris, Recherche, lib externe | /3 |
| **Email & Events** : Mailtrap, TemplatedEmail, Subscriber | /3 |
| **Upload d'images** : FileUploader, gestion création/modification/suppression | /2 |
| **DataFixtures & Pagination** : Faker, références, KnpPaginator, tri | /2 |
| **Extension Twig & Console** : Filtres, fonctions, commande stats | /2 |
| **Tests** : Unitaires, fonctionnels, API, configuration test | /2 |
| **Total** | **/28** |

> **Note** : Le barème est sur 28 points. La note finale sera convertie sur 20.


