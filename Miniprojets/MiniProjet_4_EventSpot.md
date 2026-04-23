# Mini-Projet 4 — EventSpot : Plateforme d'Événements

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation en binôme  
**Durée estimée** : 16 à 20 heures  
**Barème** : /20

---

## Contexte

Vous devez développer **EventSpot**, une plateforme de gestion d'événements. Les organisateurs créent des événements, les participants s'y inscrivent et reçoivent un email de confirmation. L'application expose une API REST et utilise les sessions pour les événements récemment consultés.

---

## Partie 1 — Modélisation et Base de données (3 pts)

> **Concepts** : Doctrine ORM, Entités, Migrations, Relations, Validation (TP1 + TP2)

### Entités

**Evenement** : `titre` (string 255, NotBlank, Length min:5), `description` (text, NotBlank, Length min:30), `dateDebut` (datetime, NotNull), `dateFin` (datetime, NotNull), `lieu` (string 255, NotBlank), `capaciteMax` (integer, Range min:1), `prix` (float, nullable, PositiveOrZero), `categorie` (string 30, Choice: conference/atelier/meetup/formation/concert), `statut` (string 20, Choice: brouillon/publie/complet/annule), `dateCreation` (datetime, auto), `imageName` (string 255, nullable)

**Inscription** : `dateInscription` (datetime, auto), `statut` (string 15, Choice: confirmee/en_attente/annulee), `commentaire` (text, nullable, Length max:500)

**Lieu** : `nom` (string 100, NotBlank, Unique), `adresse` (string 255, NotBlank), `ville` (string 100, NotBlank), `capacite` (integer, Range min:1)

**TagEvenement** *(nouveau)* : `nom` (string 50, NotBlank, Unique), `couleur` (string 7, Regex: code hex)

### Relations

- Lieu → Evenement : **OneToMany** (un lieu accueille plusieurs événements)
- Evenement → Inscription : **OneToMany** (un événement a plusieurs inscriptions)
- User → Evenement : **ManyToOne** (organisateur)
- User → Inscription : **ManyToOne** (participant)
- Evenement ↔ TagEvenement : **ManyToMany** — Un événement peut avoir plusieurs tags (Networking, Tech, Gratuit, etc.), un tag peut être associé à plusieurs événements

---

## Partie 2 — CRUD, Formulaires et Templates (4 pts)

> **Concepts** : Contrôleurs, Routes, FormType, EntityType, ChoiceType, Twig, Flash, CSRF (TP1 + TP2)

### CRUD Evenement (complet)

| Action | Route | Méthode |
|--------|-------|---------|
| Accueil | `/` | GET (6 prochains événements) |
| Liste | `/evenements` | GET |
| Détail | `/evenements/{id}` | GET |
| Créer | `/evenements/nouveau` | GET/POST |
| Modifier | `/evenements/{id}/modifier` | GET/POST |
| Supprimer | `/evenements/{id}/supprimer` | POST (CSRF) |
| S'inscrire | `/evenements/{id}/inscription` | GET/POST |

### CRUD Lieu : liste + création

### CRUD TagEvenement *(nouveau)* : liste + création + suppression

### FormTypes

- **EvenementType** : `EntityType` (lieu), `ChoiceType` (catégorie, statut), `DateTimeType` (widget single_text), `MoneyType` (prix), `EntityType` (tags, `multiple: true`, `expanded: true`, `by_reference => false`), `FileType` (image, `mapped => false`, JPEG/PNG/WebP, max 2 Mo)
- **InscriptionType** : commentaire (TextareaType)

### Templates Twig

- Layout Bootstrap 5 avec navbar et footer
- Liste : Tableau avec badges colorés pour catégorie (🎤 Conférence | 🔧 Atelier | 👥 Meetup | 📚 Formation | 🎵 Concert) et statut (📝 Brouillon | 🟢 Publié | 🔴 Complet | ⚫ Annulé), **image de l'événement**, badges des tags associés
- Détail : Jauge de remplissage (barre de progression), « X / Y places », bouton S'inscrire ou Complet, **image en grand**, liste des tags
- Messages flash après chaque opération

---

## Partie 3 — Sécurité et Authentification (3 pts)

> **Concepts** : User, Inscription, Login/Logout, Rôles, Hiérarchie, IsGranted, Propriété (TP3)

- User avec `make:user` + champ `pseudo`
- Inscription (`make:registration-form`) et Login/Logout (`make:auth`)

### Hiérarchie : `ROLE_ORGANISATEUR` → `ROLE_ADMIN`

| Fonctionnalité | Accès |
|----------------|-------|
| Voir les événements publiés | Tout le monde |
| S'inscrire à un événement | `ROLE_USER` |
| Créer un événement | `ROLE_ORGANISATEUR` |
| Modifier / Supprimer | **Organisateur** ou `ROLE_ADMIN` |
| Gérer les lieux et tags | `ROLE_ADMIN` |

- `#[IsGranted]` sur les contrôleurs, `is_granted()` dans Twig
- Organisateur automatique : `$evenement->setOrganisateur($this->getUser())`
- Navigation conditionnelle (pseudo, connexion/déconnexion)

---

## Partie 4 — API REST et Services (4 pts)

> **Concepts** : API Platform, Groupes de sérialisation, Swagger, Service, Injection (TP4)

### API Platform

Exposer **Evenement** et **Lieu** avec `#[ApiResource]`.

### Groupes de sérialisation (Evenement)

- `event:read` : titre, description, dateDebut, dateFin, lieu (nom + ville), categorie, statut, capaciteMax, prix, tags (noms)
- `event:write` : titre, description, dateDebut, dateFin, categorie, capaciteMax, prix

Tester via Swagger UI : GET, POST, PUT, DELETE.

### Service `EvenementManager`

```php
class EvenementManager
{
    public function __construct(
        private InscriptionRepository $inscRepo,
        private EvenementRepository $eventRepo
    ) {}
    public function getPlacesRestantes(Evenement $e): int
    public function estInscrit(User $u, Evenement $e): bool
    public function getNbInscrits(Evenement $e): int
    public function getEvenementsParCategorie(): array  // tableau associatif
}
```

Injecter dans le contrôleur (constructeur), afficher stats sur la page de détail et d'accueil.

---

## Partie 5 — Sessions, QueryBuilder et AssetMapper (3 pts)

> **Concepts** : Sessions (RequestStack), QueryBuilder, AssetMapper (TP5)

### Session — Événements récemment consultés

- Sur la page de détail, stocker l'ID de l'événement en session via `RequestStack`
- Afficher un bloc « 📋 Derniers événements consultés » sur la page d'accueil
- Limiter à 5 éléments (FIFO), pas de doublons

### QueryBuilder — Recherche avancée

Recherche sur la page de liste : par **titre** (partiel), par **catégorie** (ChoiceType), par **ville du lieu** (EntityType ou texte), par **tag** (EntityType) *(nouveau)*.

```php
// EvenementRepository
public function findByFilters(?string $titre, ?string $categorie, ?string $ville, ?TagEvenement $tag): array
{
    $qb = $this->createQueryBuilder('e');

    if ($titre) {
        $qb->andWhere('e.titre LIKE :titre')
           ->setParameter('titre', '%' . $titre . '%');
    }
    if ($categorie) {
        $qb->andWhere('e.categorie = :cat')
           ->setParameter('cat', $categorie);
    }
    if ($ville) {
        $qb->innerJoin('e.lieu', 'l')
           ->andWhere('l.ville LIKE :ville')
           ->setParameter('ville', '%' . $ville . '%');
    }
    if ($tag) {
        $qb->innerJoin('e.tags', 't')
           ->andWhere('t = :tag')
           ->setParameter('tag', $tag);
    }

    return $qb->orderBy('e.dateDebut', 'ASC')
              ->getQuery()->getResult();
}

public function findUpcoming(int $limit = 6): array  // prochains événements
```

### AssetMapper

- `importmap:require sweetalert2` pour les confirmations de suppression
- Utiliser dans au minimum 2 pages
- `{{ importmap('app') }}` dans `base.html.twig`

---

## Partie 6 — Email et Événements (3 pts)

> **Concepts** : Mailer (Mailtrap), TemplatedEmail, Event Subscriber (TP5)

### Email de confirmation

Quand un utilisateur s'inscrit à un événement → email via Mailtrap :
- De : `noreply@eventspot.com`
- À : email du participant
- Sujet : « 🎫 Inscription confirmée : {titre} »
- Template `emails/confirmation_inscription.html.twig` : titre, date, lieu, nom du participant

### Event Subscriber

```php
class EventSpotSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [KernelEvents::RESPONSE => 'onKernelResponse'];
    }
    public function onKernelResponse(ResponseEvent $event): void
    {
        $event->getResponse()->headers->set('X-EventSpot-Version', '1.0');
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
    upload_directory: '%kernel.project_dir%/public/uploads/evenements'

services:
    App\Service\FileUploader:
        arguments:
            $targetDirectory: '%upload_directory%'
```

### 7.3 Intégration dans le contrôleur

- À la **création** d'un événement : si une image est fournie (affiche, bannière), l'uploader et stocker le nom dans `imageName`
- À la **modification** : supprimer l'ancienne image avant d'enregistrer la nouvelle
- À la **suppression** : supprimer le fichier physique associé
- Afficher l'image en **bannière** sur la page de détail
- Afficher l'image en **miniature** dans la liste des événements
- Les événements sans image affichent un **placeholder** avec l'icône de la catégorie

---

## 🆕 Partie 8 — DataFixtures et Pagination (2 pts)

> **Concepts évalués** : DataFixtures, FakerPHP, KnpPaginatorBundle, Références entre fixtures (TP6)

### 8.1 DataFixtures avec FakerPHP

Créez des fixtures séparées par entité avec le système de **références** et `DependentFixtureInterface` :

#### `LieuFixtures.php`
- 5 lieux prédéfinis avec adresses réalistes (Centre de congrès, Salle polyvalente, Amphithéâtre universitaire, Espace coworking, Parc municipal)

#### `TagEvenementFixtures.php`
- 8 tags prédéfinis avec couleurs (Networking, Tech, Gratuit, Startup, Formation, Culture, Sport, Famille)

#### `UserFixtures.php`
- 1 admin (`admin@eventspot.com` / `admin123`, `ROLE_ADMIN`)
- 2 organisateurs (`orga1@eventspot.com` / `orga123` et `orga2@eventspot.com` / `orga123`, `ROLE_ORGANISATEUR`)
- 5 participants générés par Faker (`ROLE_USER`)
- Utiliser `UserPasswordHasherInterface` pour hasher les mots de passe

#### `EvenementFixtures.php` (dépend de LieuFixtures, UserFixtures, TagEvenementFixtures)
- 15 événements avec titres et descriptions réalistes
- Variété de catégories, statuts, prix et dates
- Chaque événement associé à 1 à 4 tags aléatoires (ManyToMany)
- Événements à venir et passés

#### `InscriptionFixtures.php` (dépend de EvenementFixtures, UserFixtures)
- 30 inscriptions réparties dans les événements
- Statuts variés (confirmée, en attente, annulée)
- Respecter la capacité maximale

### 8.2 Pagination avec KnpPaginatorBundle

- Installer `knplabs/knp-paginator-bundle`
- Paginer la **liste des événements** : 9 événements par page
- Paginer la **liste des inscriptions** d'un événement (page de détail) : 10 inscriptions par page
- Ajouter le **tri** par titre, date de début et prix via `knp_pagination_sortable()`
- Afficher le **nombre total de résultats**

---

## 🆕 Partie 9 — Extension Twig et Commande Console (2 pts)

> **Concepts évalués** : Extensions Twig personnalisées, Commandes Console, SymfonyStyle (TP7)

### 9.1 Extension Twig `EventSpotExtension`

Créez `src/Twig/EventSpotExtension.php` avec :

#### Filtre `time_ago`
Convertit une date en format relatif :
```twig
{{ evenement.dateCreation|time_ago }}
{# Affiche : "il y a 3 jours", "il y a 2 mois", etc. #}
```

#### Filtre `price_format`
Formate le prix d'un événement :
```twig
{{ evenement.prix|price_format }}
{# 0 ou null → "Gratuit 🎉", 15.5 → "15,50 €", 100 → "100,00 €" #}
```

#### Fonction `capacity_badge`
Génère un badge HTML indiquant le niveau de remplissage :
```twig
{{ capacity_badge(nbInscrits, capaciteMax) }}
{# < 50% → badge vert "Places disponibles" #}
{# 50-80% → badge orange "Places limitées" #}
{# 80-99% → badge rouge "Dernières places" #}
{# 100% → badge noir "Complet" #}
```

Utiliser ces filtres/fonctions dans au moins **3 templates** différents.

### 9.2 Commande Console `app:eventspot:report`

Créez `src/Command/EventSpotReportCommand.php` :

```php
#[AsCommand(
    name: 'app:eventspot:report',
    description: 'Génère un rapport sur les événements et inscriptions',
)]
class EventSpotReportCommand extends Command
{
    // Injection de EvenementRepository, InscriptionRepository, LieuRepository

    // Option --upcoming : afficher uniquement les événements à venir
    // Option --lieu=NOM : filtrer par lieu
    
    // Afficher :
    // - Nombre total d'événements par statut
    // - Nombre total d'inscriptions par statut
    // - Taux de remplissage moyen
    // - Répartition par catégorie
    // - Top 3 des événements les plus populaires (nombre d'inscrits)
    // - Revenu total estimé (prix × inscriptions confirmées)
    // Avec SymfonyStyle : title(), table(), note(), success()
}
```

Exécution :
```bash
php bin/console app:eventspot:report
php bin/console app:eventspot:report --upcoming
php bin/console app:eventspot:report --lieu="Centre de congrès"
```

---

## 🆕 Partie 10 — Tests (2 pts)

> **Concepts évalués** : PHPUnit, Tests unitaires, Tests fonctionnels, Tests API (TP6)

### 10.1 Tests unitaires du service `EvenementManager`

Créez `tests/Service/EvenementManagerTest.php` :

- Tester `getPlacesRestantes()` retourne la bonne valeur (capacité - inscrits)
- Tester `getPlacesRestantes()` retourne 0 quand l'événement est complet
- Tester `estInscrit()` retourne `true` si l'utilisateur est inscrit
- Tester `estInscrit()` retourne `false` si l'utilisateur n'est pas inscrit
- Tester `getNbInscrits()` ne compte que les inscriptions confirmées

### 10.2 Tests fonctionnels des contrôleurs

Créez `tests/Controller/EvenementControllerTest.php` :

- Tester que la page `/evenements` retourne un code **200**
- Tester que la page `/` (accueil) contient les prochains événements
- Tester que la page `/evenements/nouveau` est **interdite** sans authentification (code **302**)
- Tester la **création d'un événement** via soumission de formulaire et vérifier la redirection
- Tester que le **message flash** apparaît après création

### 10.3 Tests de l'API

Créez `tests/Api/EvenementApiTest.php` :

- Tester `GET /api/evenements` retourne **200** et le bon content-type (`application/ld+json`)
- Tester `POST /api/evenements` avec des données valides retourne **201**
- Tester `POST /api/evenements` avec un titre vide retourne **422**

### 10.4 Configuration de test

- Créer `.env.test` avec une base de données de test séparée
- Les fixtures doivent être chargées avant l'exécution des tests

---

## 📦 Livrables

- ✅ Dépôt **GitHub** avec commits structurés
- ✅ `README.md` : installation, config Mailtrap, identifiants de test, schéma des relations
- ✅ Fixtures fonctionnelles : `php bin/console doctrine:fixtures:load` charge toutes les données
- ✅ Base peuplée via fixtures : 5 lieux, 8 tags, 15 événements, 30 inscriptions, 8 utilisateurs
- ✅ Tests exécutables : `php bin/phpunit` passe sans erreurs

---

## 📝 Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations (OneToMany + ManyToMany) | /3 |
| **CRUD & Templates** : FormTypes, EntityType, ChoiceType, Twig, Flash, CSRF | /4 |
| **Sécurité** : Inscription, Login, Rôles, Hiérarchie, Propriété | /3 |
| **API & Services** : API Platform, Groupes, Swagger, Service injecté | /4 |
| **Session, QueryBuilder, AssetMapper** : Derniers consultés, Recherche, lib externe | /3 |
| **Email & Events** : Mailtrap, TemplatedEmail, Subscriber | /3 |
| **Upload d'images** : FileUploader, gestion création/modification/suppression | /2 |
| **DataFixtures & Pagination** : Faker, références, KnpPaginator, tri | /2 |
| **Extension Twig & Console** : Filtres, fonctions, commande report | /2 |
| **Tests** : Unitaires, fonctionnels, API, configuration test | /2 |
| **Total** | **/28** |

> **Note** : Le barème est sur 28 points. La note finale sera convertie sur 20.


