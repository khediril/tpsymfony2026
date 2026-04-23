# Mini-Projet 1 — BookShelf : Bibliothèque en Ligne

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation en binôme  
**Durée estimée** : 16 à 20 heures  
**Barème** : /20

---

## Contexte

Vous devez développer **BookShelf**, une application web permettant de gérer une bibliothèque en ligne. Les utilisateurs peuvent parcourir les livres, s'inscrire, ajouter des livres à leur liste de lecture (session), et recevoir un email lorsqu'un nouveau livre est ajouté. L'application expose également une API REST.

---

## Partie 1 — Modélisation et Base de données (3 pts)

> **Concepts évalués** : Doctrine ORM, Entités, Migrations, Relations, Contraintes de validation (TP1 + TP2)

### Entités à créer

#### Livre
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `titre` | string (255) | NotBlank, Length(min: 3, max: 255) |
| `resume` | text | NotBlank, Length(min: 20) |
| `isbn` | string (13) | NotBlank, Regex (format ISBN-13) |
| `nbPages` | integer | NotNull, Range(min: 1, max: 5000) |
| `datePublication` | date | NotNull |
| `disponible` | boolean | — |
| `imageName` | string (255) | nullable |

#### Auteur
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `nom` | string (100) | NotBlank, Length(min: 2) |
| `prenom` | string (100) | NotBlank |
| `biographie` | text | nullable |
| `nationalite` | string (50) | NotBlank |

#### Genre
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `nom` | string (50) | NotBlank, Unique |
| `description` | text | nullable |
| `couleur` | string (7) | Regex: code couleur hex (#FF5733) |

#### Tag *(nouveau)*
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `nom` | string (50) | NotBlank, Unique |
| `couleur` | string (7) | Regex: code hex (#3498DB) |

### Relations

| Relation | Description |
|----------|-------------|
| Auteur → Livre | **OneToMany** — Un auteur peut avoir écrit plusieurs livres |
| Genre → Livre | **OneToMany** — Un genre contient plusieurs livres |
| User → Livre | **ManyToOne** — Un utilisateur (ajouté par) peut ajouter plusieurs livres |
| Livre ↔ Tag | **ManyToMany** — Un livre peut avoir plusieurs tags, un tag peut être associé à plusieurs livres |

### Attendus
- Entités créées avec `make:entity`
- Contraintes de validation (`Assert`) sur chaque entité
- Migrations générées et exécutées
- Relations correctement configurées (y compris la table de jointure ManyToMany)

---

## Partie 2 — CRUD, Formulaires et Templates (4 pts)

> **Concepts évalués** : Contrôleurs, Routes, FormType, EntityType, ChoiceType, Twig (héritage, boucles, conditions), Messages Flash, CSRF (TP1 + TP2)

### 2.1 CRUD Livre (complet)

| Action | Route | Méthode |
|--------|-------|---------|
| Liste des livres | `/livres` | GET |
| Détail d'un livre | `/livres/{id}` | GET |
| Créer un livre | `/livres/nouveau` | GET / POST |
| Modifier un livre | `/livres/{id}/modifier` | GET / POST |
| Supprimer un livre | `/livres/{id}/supprimer` | POST (CSRF) |

### 2.2 FormTypes

- **LivreType** : tous les champs avec les types appropriés
  - `EntityType` pour les champs `auteur` et `genre` (sélecteur lié à la BDD)
  - `EntityType` avec `multiple: true`, `expanded: true` pour les `tags` (checkboxes). **Attention** : utiliser `by_reference => false`
  - `DateType` avec widget `single_text` pour la date de publication
  - `CheckboxType` pour `disponible`
  - `FileType` pour l'image de couverture (`mapped => false`, contraintes : JPEG/PNG/WebP, max 2 Mo)
- Bouton de soumission stylé avec Bootstrap

### 2.3 CRUD Auteur et Genre

Réalisez un CRUD simplifié (liste + création) pour **Auteur** et **Genre**.

### 2.4 CRUD Tag *(nouveau)*

Réalisez un CRUD simplifié (liste + création + suppression) pour **Tag**.

### 2.5 Templates Twig

- **Layout** (`base.html.twig`) : Bootstrap 5, navbar avec navigation, footer
- **Liste des livres** : Tableau avec titre, auteur, genre, nb pages
  - Badge coloré pour la disponibilité (🟢 Disponible / 🔴 Indisponible)
  - Badge avec la couleur du genre
  - Badges colorés pour les tags associés à chaque livre
  - **Image de couverture en miniature** si elle existe
- **Détail** : Toutes les informations du livre, lien vers l'auteur, image de couverture en grand, liste des tags
- **Formulaire** : Rendu avec `form_row()`, Bootstrap
- **Messages flash** après chaque opération (success, danger)

---

## Partie 3 — Sécurité et Authentification (3 pts)

> **Concepts évalués** : User, Inscription, Login/Logout, Rôles, Hiérarchie, IsGranted, Propriété des données (TP3)

### 3.1 Système d'utilisateurs

- Créer l'entité `User` avec `make:user` + champ `pseudo` (string, 50)
- **Inscription** avec `make:registration-form` et validation
- **Connexion / Déconnexion** avec `make:auth`

### 3.2 Rôles et hiérarchie

```yaml
role_hierarchy:
    ROLE_BIBLIOTHECAIRE: ROLE_USER
    ROLE_ADMIN: ROLE_BIBLIOTHECAIRE
```

### 3.3 Contrôle d'accès

| Fonctionnalité | Accès requis |
|----------------|-------------|
| Consulter les livres | Tout le monde |
| Ajouter un livre | `ROLE_BIBLIOTHECAIRE` |
| Modifier / Supprimer un livre | **Celui qui l'a ajouté** ou `ROLE_ADMIN` |
| Gérer les Auteurs, Genres et Tags | `ROLE_ADMIN` |

- Utiliser `#[IsGranted]` sur les contrôleurs
- Utiliser `is_granted()` dans Twig pour masquer les boutons
- L'utilisateur qui ajoute un livre est stocké automatiquement : `$livre->setAjoutePar($this->getUser())`

### 3.4 Navigation conditionnelle

- Pseudo de l'utilisateur connecté dans la navbar
- Boutons Connexion / Inscription si non connecté
- Bouton Déconnexion si connecté

---

## Partie 4 — API REST et Services (4 pts)

> **Concepts évalués** : API Platform, #[ApiResource], Groupes de sérialisation, Swagger UI, Services personnalisés, Injection de dépendances (TP4)

### 4.1 API Platform

Installer API Platform (`composer require api`) et exposer les entités :

```php
#[ApiResource(
    normalizationContext: ['groups' => ['livre:read']],
    denormalizationContext: ['groups' => ['livre:write']],
)]
```

### 4.2 Groupes de sérialisation

| Propriété Livre | `livre:read` | `livre:write` |
|-----------------|:---:|:---:|
| titre | ✅ | ✅ |
| resume | ✅ | ✅ |
| isbn | ✅ | ✅ |
| nbPages | ✅ | ✅ |
| datePublication | ✅ | ✅ |
| disponible | ✅ | ✅ |
| auteur (nom + prénom) | ✅ | ❌ |
| genre (nom) | ✅ | ❌ |
| tags (noms) | ✅ | ❌ |

- Configurer les groupes sur l'entité **Genre** également (`genre:read`, `genre:write`)
- Tester via **Swagger UI** (`/api`) : GET, POST, PUT, DELETE

### 4.3 Service personnalisé `BibliothequeStats`

Créez `src/Service/BibliothequeStats.php` :

```php
class BibliothequeStats
{
    public function __construct(private LivreRepository $livreRepository) {}

    // Nombre total de livres
    public function getTotalLivres(): int

    // Nombre de livres disponibles
    public function getLivresDisponibles(): int

    // Nombre de livres par genre (tableau associatif)
    public function getLivresParGenre(): array

    // Temps de lecture estimé (somme nbPages / 30 pages par heure)
    public function getTempsLectureTotal(): float
}
```

- Injecter le service dans le contrôleur (injection par constructeur)
- Afficher les statistiques sur la page d'accueil

---

## Partie 5 — Sessions, QueryBuilder et AssetMapper (3 pts)

> **Concepts évalués** : Sessions (RequestStack), QueryBuilder, AssetMapper / Importmap (TP5)

### 5.1 Session — Liste de lecture

Implémenter une **liste de lecture** stockée en session :

- **Bouton « 📖 Ajouter à ma liste »** sur la page de détail de chaque livre
- Stocker les IDs des livres dans la session via `RequestStack`
- **Route `/ma-liste`** : afficher les livres de la liste (récupérés via le Repository)
- **Retirer un livre** de la liste (bouton ❌)
- Afficher le **nombre de livres dans la navbar** (badge dynamique)
- Empêcher les doublons

### 5.2 QueryBuilder — Recherche avancée

Formulaire de recherche sur la page de liste :

- **Recherche par titre** (partielle, insensible à la casse)
- **Filtre par genre** (sélecteur déroulant EntityType)
- **Filtre par disponibilité** (case à cocher)
- **Filtre par tag** (sélecteur déroulant EntityType) *(nouveau)*

Créez dans `LivreRepository` :

```php
public function findByFilters(?string $titre, ?Genre $genre, ?bool $disponible, ?Tag $tag): array
{
    $qb = $this->createQueryBuilder('l');

    if ($titre) {
        $qb->andWhere('l.titre LIKE :titre')
           ->setParameter('titre', '%' . $titre . '%');
    }
    if ($genre) {
        $qb->andWhere('l.genre = :genre')
           ->setParameter('genre', $genre);
    }
    if ($disponible !== null) {
        $qb->andWhere('l.disponible = :dispo')
           ->setParameter('dispo', $disponible);
    }
    if ($tag) {
        $qb->innerJoin('l.tags', 't')
           ->andWhere('t = :tag')
           ->setParameter('tag', $tag);
    }

    return $qb->orderBy('l.datePublication', 'DESC')
              ->getQuery()->getResult();
}

public function findLastAdded(int $limit = 5): array
```

### 5.3 AssetMapper

- Installer une bibliothèque externe via `importmap:require` :
  - **SweetAlert2** pour les confirmations de suppression
- Utiliser la bibliothèque dans au minimum **2 pages**
- Vérifier que `{{ importmap('app') }}` est présent dans `base.html.twig`

---

## 📧 Partie 6 — Email et Événements (3 pts)

> **Concepts évalués** : Mailer (Mailtrap), TemplatedEmail, Event Subscriber (TP5)

### 6.1 Notification par email

Lorsqu'un nouveau livre est ajouté, envoyer un email de notification :

- **De** : `noreply@bookshelf.com`
- **À** : une adresse configurable (ou l'admin)
- **Sujet** : « 📗 Nouveau livre ajouté : {titre} »
- **Corps** : Template Twig `emails/nouveau_livre.html.twig` contenant :
  - Titre et auteur du livre
  - Genre et ISBN
  - Nombre de pages
  - Nom de l'utilisateur qui l'a ajouté

Configuration : Utiliser **Mailtrap** (cf. Guide d'installation Mailtrap).

### 6.2 Event Subscriber

Créer `src/EventSubscriber/BookShelfSubscriber.php` :

```php
class BookShelfSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [
            KernelEvents::RESPONSE => 'onKernelResponse',
        ];
    }

    public function onKernelResponse(ResponseEvent $event): void
    {
        // Ajouter un header personnalisé X-BookShelf-Version: 1.0
        $event->getResponse()->headers->set('X-BookShelf-Version', '1.0');
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
    upload_directory: '%kernel.project_dir%/public/uploads/livres'

services:
    App\Service\FileUploader:
        arguments:
            $targetDirectory: '%upload_directory%'
```

### 7.3 Intégration dans le contrôleur

- À la **création** d'un livre : si une image est fournie, l'uploader et stocker le nom dans `imageName`
- À la **modification** : supprimer l'ancienne image avant d'enregistrer la nouvelle
- À la **suppression** : supprimer le fichier physique associé
- Afficher l'image dans le **détail** et en **miniature** dans la liste

---

## 🆕 Partie 8 — DataFixtures et Pagination (2 pts)

> **Concepts évalués** : DataFixtures, FakerPHP, KnpPaginatorBundle, Références entre fixtures (TP6)

### 8.1 DataFixtures avec FakerPHP

Créez des fixtures séparées par entité avec le système de **références** et `DependentFixtureInterface` :

#### `GenreFixtures.php`
- 6 genres prédéfinis avec couleurs (Roman, SF, Policier, Fantasy, Biographie, Histoire)
- Stocker les références pour les autres fixtures

#### `AuteurFixtures.php`
- 5 auteurs avec noms/prénoms réalistes (Faker `fr_FR`), biographies et nationalités

#### `TagFixtures.php`
- 8 tags prédéfinis avec couleurs (Bestseller, Classique, Coup de cœur, Nouveau, etc.)

#### `UserFixtures.php`
- 1 admin (`admin@bookshelf.com` / `admin123`, `ROLE_ADMIN`)
- 1 bibliothécaire (`biblio@bookshelf.com` / `biblio123`, `ROLE_BIBLIOTHECAIRE`)
- 5 utilisateurs générés par Faker (`ROLE_USER`)
- Utiliser `UserPasswordHasherInterface` pour hasher les mots de passe

#### `LivreFixtures.php` (dépend de toutes les autres)
- 30 livres avec données Faker
- Chaque livre associé à un auteur, un genre, un utilisateur aléatoire
- Chaque livre associé à 1 à 4 tags aléatoires
- ISBN-13 générés avec `$faker->isbn13()`

### 8.2 Pagination avec KnpPaginatorBundle

- Installer `knplabs/knp-paginator-bundle`
- Paginer la **liste des livres** : 10 livres par page
- Ajouter le **tri par colonnes** (titre, date de publication, nombre de pages) via `knp_pagination_sortable()`
- Afficher le **nombre total de résultats**
- Paginer la **liste des auteurs** : 5 auteurs par page

---

## 🆕 Partie 9 — Extension Twig et Commande Console (2 pts)

> **Concepts évalués** : Extensions Twig personnalisées, Commandes Console, SymfonyStyle (TP7)

### 9.1 Extension Twig `BookShelfExtension`

Créez `src/Twig/BookShelfExtension.php` avec :

#### Filtre `time_ago`
Convertit une date en format relatif :
```twig
{{ livre.datePublication|time_ago }}
{# Affiche : "il y a 3 jours", "il y a 2 mois", etc. #}
```

#### Filtre `reading_time`
Estime le temps de lecture basé sur le nombre de pages :
```twig
{{ livre.nbPages|reading_time }}
{# Affiche : "2h30 de lecture" (30 pages/heure) #}
```

#### Fonction `book_status_badge`
Génère un badge HTML pour la disponibilité :
```twig
{{ book_status_badge(livre.disponible) }}
{# Affiche : <span class="badge bg-success">Disponible</span> #}
```

Utiliser ces filtres/fonctions dans au moins **3 templates** différents.

### 9.2 Commande Console `app:bookshelf:stats`

Créez `src/Command/BookShelfStatsCommand.php` :

```php
#[AsCommand(
    name: 'app:bookshelf:stats',
    description: 'Affiche les statistiques de la bibliothèque',
)]
class BookShelfStatsCommand extends Command
{
    // Injection de LivreRepository, AuteurRepository, GenreRepository

    // Option --detail : affiche le détail par genre
    // Option --format : choisir le format de sortie (table, json)
    
    // Afficher :
    // - Nombre total de livres
    // - Nombre de livres disponibles / indisponibles
    // - Nombre d'auteurs et de genres
    // - Temps de lecture total estimé
    // - Top 3 des genres les plus populaires
    // Avec SymfonyStyle : title(), table(), success()
}
```

Exécution :
```bash
php bin/console app:bookshelf:stats
php bin/console app:bookshelf:stats --detail
```

---

## 🆕 Partie 10 — Tests (2 pts)

> **Concepts évalués** : PHPUnit, Tests unitaires, Tests fonctionnels, Tests API (TP6)

### 10.1 Tests unitaires du service `BibliothequeStats`

Créez `tests/Service/BibliothequeStatsTest.php` :

- Tester `getTotalLivres()` retourne le bon nombre
- Tester `getLivresDisponibles()` ne compte que les livres disponibles
- Tester `getTempsLectureTotal()` calcule correctement (nbPages / 30)
- Tester `getLivresParGenre()` retourne un tableau associatif correct

### 10.2 Tests fonctionnels des contrôleurs

Créez `tests/Controller/LivreControllerTest.php` :

- Tester que la page `/livres` retourne un code **200**
- Tester que la page `/livres` contient un tableau de livres
- Tester que la page `/livres/nouveau` est **interdite** sans authentification (code **302** → redirection login)
- Tester la **création d'un livre** via soumission de formulaire et vérifier la redirection
- Tester que le **message flash** apparaît après création

### 10.3 Tests de l'API

Créez `tests/Api/LivreApiTest.php` :

- Tester `GET /api/livres` retourne **200** et le bon content-type (`application/ld+json`)
- Tester `POST /api/livres` avec des données valides retourne **201**
- Tester `POST /api/livres` avec des données invalides retourne **422**

### 10.4 Configuration de test

- Créer `.env.test` avec une base de données de test séparée
- Les fixtures doivent être chargées avant l'exécution des tests

---

## 📦 Livrables

- ✅ Dépôt **GitHub** avec historique de commits structuré (commits réguliers et descriptifs)
- ✅ Fichier `README.md` : instructions d'installation, config Mailtrap, identifiants de test, schéma des relations
- ✅ Fixtures fonctionnelles : `php bin/console doctrine:fixtures:load` charge toutes les données
- ✅ Base peuplée via fixtures : 5 auteurs, 6 genres, 8 tags, 30 livres, 7 utilisateurs
- ✅ Tests exécutables : `php bin/phpunit` passe sans erreurs

---

## 📝 Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations (OneToMany + ManyToMany) | /3 |
| **CRUD & Templates** : FormTypes, EntityType, Twig, Flash, CSRF, Tags | /4 |
| **Sécurité** : Inscription, Login, Rôles, Hiérarchie, Propriété | /3 |
| **API & Services** : API Platform, Groupes, Swagger, Service injecté | /4 |
| **Session, QueryBuilder, AssetMapper** : Liste de lecture, Recherche, lib externe | /3 |
| **Email & Events** : Mailtrap, TemplatedEmail, Subscriber | /3 |
| **Upload d'images** : FileUploader, gestion création/modification/suppression | /2 |
| **DataFixtures & Pagination** : Faker, références, KnpPaginator, tri | /2 |
| **Extension Twig & Console** : Filtres, fonctions, commande stats | /2 |
| **Tests** : Unitaires, fonctionnels, API, configuration test | /2 |
| **Total** | **/28** |

> **Note** : Le barème est sur 28 points. La note finale sera convertie sur 20.


