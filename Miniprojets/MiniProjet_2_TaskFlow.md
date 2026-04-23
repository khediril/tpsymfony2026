# Mini-Projet 2 — TaskFlow : Gestionnaire de Tâches Collaboratif

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation en binôme  
**Durée estimée** : 16 à 20 heures  
**Barème** : /20

---

## Contexte

Vous devez développer **TaskFlow**, une application de gestion de tâches collaboratives. Les utilisateurs peuvent créer des projets, y ajouter des tâches, les assigner, et recevoir des notifications par email. L'application expose une API REST et utilise les sessions pour garder en mémoire les projets récemment consultés.

---

## Partie 1 — Modélisation et Base de données (3 pts)

> **Concepts évalués** : Doctrine ORM, Entités, Migrations, Relations, Contraintes de validation (TP1 + TP2)

### Entités à créer

#### Projet
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `nom` | string (100) | NotBlank, Length(min: 3, max: 100) |
| `description` | text | nullable |
| `dateCreation` | datetime | NotNull (auto) |
| `dateLimite` | date | NotNull |
| `statut` | string (20) | Choice: "planifie", "en_cours", "termine", "annule" |
| `imageName` | string (255) | nullable |

#### Tache
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `titre` | string (255) | NotBlank, Length(min: 5, max: 255) |
| `description` | text | nullable |
| `priorite` | string (10) | Choice: "basse", "moyenne", "haute", "urgente" |
| `statut` | string (20) | Choice: "a_faire", "en_cours", "terminee" |
| `dateCreation` | datetime | NotNull (auto) |
| `dateEcheance` | date | nullable |
| `pieceJointeName` | string (255) | nullable |

#### Etiquette
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `nom` | string (50) | NotBlank, Unique |
| `couleur` | string (7) | NotBlank, Regex: code hex (#3498DB) |

### Relations

| Relation | Description |
|----------|-------------|
| Projet → Tache | **OneToMany** — Un projet contient plusieurs tâches |
| User → Projet | **ManyToOne** — Un utilisateur (créateur) possède plusieurs projets |
| User → Tache | **ManyToOne** — Une tâche est assignée à un utilisateur |
| Tache ↔ Etiquette | **ManyToMany** — Une tâche peut avoir plusieurs étiquettes, une étiquette peut être appliquée à plusieurs tâches |

### Attendus
- Entités créées avec `make:entity`
- Contraintes de validation (`Assert`) sur chaque entité
- Migrations générées et exécutées
- Relations correctement configurées (y compris la table de jointure ManyToMany)

---

##  Partie 2 — CRUD, Formulaires et Templates (4 pts)

> **Concepts évalués** : Contrôleurs, Routes, FormType, EntityType, ChoiceType, Twig (héritage, boucles, conditions), Messages Flash, CSRF (TP1 + TP2)

### 2.1 CRUD Projet (complet)

| Action | Route | Méthode |
|--------|-------|---------|
| Liste des projets | `/projets` | GET |
| Détail d'un projet (+ ses tâches) | `/projets/{id}` | GET |
| Créer un projet | `/projets/nouveau` | GET / POST |
| Modifier un projet | `/projets/{id}/modifier` | GET / POST |
| Supprimer un projet | `/projets/{id}/supprimer` | POST (CSRF) |

### 2.2 CRUD Tache

| Action | Route | Méthode |
|--------|-------|---------|
| Créer une tâche dans un projet | `/projets/{id}/taches/nouvelle` | GET / POST |
| Modifier une tâche | `/taches/{id}/modifier` | GET / POST |
| Supprimer une tâche | `/taches/{id}/supprimer` | POST (CSRF) |

### 2.3 FormTypes

- **ProjetType** : nom, description, dateLimite, statut (`ChoiceType`), image (`FileType`, `mapped => false`, JPEG/PNG/WebP, max 2 Mo)
- **TacheType** : titre, description, priorite (`ChoiceType`), dateEcheance, assignation (`EntityType` → User), etiquettes (`EntityType` → Etiquette, `multiple: true`, `expanded: true`, `by_reference => false`), pièce jointe (`FileType`, `mapped => false`, max 5 Mo)
- Boutons de soumission stylés avec Bootstrap

### 2.4 CRUD Étiquette

Réalisez un CRUD simplifié (liste + création + suppression) pour **Étiquette**.

### 2.5 Templates Twig

- **Layout** (`base.html.twig`) : Bootstrap 5, navbar avec navigation, footer
- **Liste des projets** : Tableau avec nom, statut (badge coloré), nombre de tâches, créateur, image en miniature
  - 🔵 Planifié | 🟡 En cours | 🟢 Terminé | 🔴 Annulé
- **Détail projet** : Infos du projet + image + tableau de ses tâches avec badges de priorité et badges d'étiquettes colorées
  - 🔵 Basse | 🟢 Moyenne | 🟠 Haute | 🔴 Urgente
- **Barre de progression** du projet (% tâches terminées)
- **Messages flash** après chaque opération

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
    ROLE_CHEF_PROJET: ROLE_USER
    ROLE_ADMIN: ROLE_CHEF_PROJET
```

### 3.3 Contrôle d'accès

| Fonctionnalité | Accès requis |
|----------------|-------------|
| Voir les projets | Tout le monde |
| Créer un projet | `ROLE_CHEF_PROJET` |
| Modifier / Supprimer un projet | **Créateur du projet** ou `ROLE_ADMIN` |
| Créer / Assigner une tâche | `ROLE_USER` |
| Gérer les étiquettes | `ROLE_ADMIN` |

- Utiliser `#[IsGranted]` sur les contrôleurs
- Utiliser `is_granted()` dans Twig pour masquer les boutons
- Le créateur du projet est stocké automatiquement : `$projet->setCreateur($this->getUser())`

### 3.4 Navigation conditionnelle

- Pseudo de l'utilisateur connecté dans la navbar
- Boutons Connexion / Inscription si non connecté
- Bouton Déconnexion si connecté

---

## Partie 4 — API REST et Services (4 pts)

> **Concepts évalués** : API Platform, #[ApiResource], Groupes de sérialisation, Swagger UI, Services personnalisés, Injection de dépendances (TP4)

### 4.1 API Platform

Installer API Platform et exposer les entités :

```php
#[ApiResource(
    normalizationContext: ['groups' => ['projet:read']],
    denormalizationContext: ['groups' => ['projet:write']],
)]
```

### 4.2 Groupes de sérialisation

| Propriété Projet | `projet:read` | `projet:write` |
|------------------|:---:|:---:|
| nom | ✅ | ✅ |
| description | ✅ | ✅ |
| dateCreation | ✅ | ❌ |
| dateLimite | ✅ | ✅ |
| statut | ✅ | ✅ |
| créateur (pseudo) | ✅ | ❌ |
| taches (titres + statut) | ✅ | ❌ |

- Configurer les groupes sur l'entité **Tache** également (`tache:read`, `tache:write`)
- Tester via **Swagger UI** (`/api`) : GET, POST, PUT, DELETE

### 4.3 Service personnalisé `ProjetStatsCalculator`

Créez `src/Service/ProjetStatsCalculator.php` :

```php
class ProjetStatsCalculator
{
    public function __construct(private TacheRepository $tacheRepository) {}

    // Pourcentage d'avancement (tâches terminées / total)
    public function getProgressPercentage(Projet $projet): int

    // Nombre de tâches par statut
    public function getTaskCountByStatus(Projet $projet): array

    // Vérifie si le projet est en retard (date limite dépassée + tâches non terminées)
    public function isOverdue(Projet $projet): bool

    // Durée restante en jours
    public function getRemainingDays(Projet $projet): int
}
```

- Injecter le service dans le contrôleur (injection par constructeur)
- Afficher les statistiques sur la page de détail du projet

---

## Partie 5 — Sessions, QueryBuilder et AssetMapper (3 pts)

> **Concepts évalués** : Sessions (RequestStack), QueryBuilder, AssetMapper / Importmap (TP5)

### 5.1 Session — Projets récemment consultés

Utiliser la **session** pour stocker les **5 derniers projets consultés** :

- Sur la page de détail d'un projet, ajouter son ID à la session via `RequestStack`
- Afficher un bloc « 📋 Derniers projets consultés » sur la page d'accueil ou dans une sidebar
- Limiter à 5 éléments (FIFO — le plus ancien est retiré)
- Empêcher les doublons

### 5.2 QueryBuilder — Recherche avancée

Formulaire de recherche sur la page de liste des projets :

- **Recherche par nom** (partielle, insensible à la casse)
- **Filtre par statut** (sélecteur déroulant ChoiceType)
- **Filtre par créateur** (sélecteur déroulant EntityType → User)
- **Filtre par étiquette** (sélecteur déroulant EntityType → Etiquette — recherche les projets ayant au moins une tâche avec cette étiquette) *(nouveau)*

Créez dans `ProjetRepository` :

```php
public function findByFilters(?string $nom, ?string $statut, ?User $createur, ?Etiquette $etiquette): array
{
    $qb = $this->createQueryBuilder('p');

    if ($nom) {
        $qb->andWhere('p.nom LIKE :nom')
           ->setParameter('nom', '%' . $nom . '%');
    }
    if ($statut) {
        $qb->andWhere('p.statut = :statut')
           ->setParameter('statut', $statut);
    }
    if ($createur) {
        $qb->andWhere('p.createur = :createur')
           ->setParameter('createur', $createur);
    }
    if ($etiquette) {
        $qb->innerJoin('p.taches', 't')
           ->innerJoin('t.etiquettes', 'e')
           ->andWhere('e = :etiquette')
           ->setParameter('etiquette', $etiquette);
    }

    return $qb->orderBy('p.dateCreation', 'DESC')
              ->getQuery()->getResult();
}

public function findMostRecentProjects(int $limit = 5): array
```

### 5.3 AssetMapper

- Installer une bibliothèque externe via `importmap:require` :
  - **SweetAlert2** pour les confirmations de suppression
- Utiliser la bibliothèque dans au minimum **2 pages**
- Vérifier que `{{ importmap('app') }}` est présent dans `base.html.twig`

---

## Partie 6 — Email et Événements (3 pts)

> **Concepts évalués** : Mailer (Mailtrap), TemplatedEmail, Event Subscriber (TP5)

### 6.1 Notification par email

Lorsqu'une tâche est **assignée à un utilisateur**, envoyer un email de notification :

- **De** : `noreply@taskflow.com`
- **À** : l'email de l'utilisateur assigné
- **Sujet** : « ✅ Nouvelle tâche assignée : {titre} »
- **Corps** : Template Twig `emails/tache_assignee.html.twig` contenant :
  - Titre de la tâche
  - Nom du projet
  - Priorité et date d'échéance
  - Nom de l'assignateur

Configuration : Utiliser **Mailtrap** (cf. Guide d'installation Mailtrap).

### 6.2 Event Subscriber

Créer `src/EventSubscriber/TaskFlowSubscriber.php` :

```php
class TaskFlowSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [
            KernelEvents::RESPONSE => 'onKernelResponse',
        ];
    }

    public function onKernelResponse(ResponseEvent $event): void
    {
        // Ajouter un header personnalisé X-TaskFlow-Version: 1.0
        $event->getResponse()->headers->set('X-TaskFlow-Version', '1.0');
    }
}
```

---

## 🆕 Partie 7 — Upload de fichiers (2 pts)

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

Dans `config/services.yaml`, configurez **deux répertoires d'upload** :

```yaml
parameters:
    upload_directory_projets: '%kernel.project_dir%/public/uploads/projets'
    upload_directory_taches: '%kernel.project_dir%/public/uploads/taches'
```

### 7.3 Upload d'images pour les projets

- **Image de couverture** du projet : JPEG/PNG/WebP, max 2 Mo
- À la **création** : uploader et stocker le nom dans `imageName`
- À la **modification** : supprimer l'ancienne image si nouvelle image fournie
- À la **suppression** : supprimer le fichier physique
- Afficher l'image dans la **liste** (miniature) et le **détail** (grande taille)

### 7.4 Pièces jointes pour les tâches

- **Pièce jointe** à une tâche : PDF, DOCX, images (max 5 Mo)
- Upload via le formulaire de création/modification de tâche
- Lien de téléchargement de la pièce jointe dans le détail de la tâche
- Gestion de la suppression du fichier à la suppression de la tâche

---

## 🆕 Partie 8 — DataFixtures et Pagination (2 pts)

> **Concepts évalués** : DataFixtures, FakerPHP, KnpPaginatorBundle, Références entre fixtures (TP6)

### 8.1 DataFixtures avec FakerPHP

Créez des fixtures séparées par entité avec le système de **références** et `DependentFixtureInterface` :

#### `EtiquetteFixtures.php`
- 6 étiquettes prédéfinies avec couleurs (Bug, Feature, Urgent, Documentation, Amélioration, Design)

#### `UserFixtures.php`
- 1 admin (`admin@taskflow.com` / `admin123`, `ROLE_ADMIN`)
- 1 chef de projet (`chef@taskflow.com` / `chef123`, `ROLE_CHEF_PROJET`)
- 5 utilisateurs générés par Faker (`ROLE_USER`)
- Utiliser `UserPasswordHasherInterface` pour hasher les mots de passe

#### `ProjetFixtures.php` (dépend de UserFixtures)
- 8 projets avec noms et descriptions réalistes (Faker)
- Statuts variés, dates réalistes

#### `TacheFixtures.php` (dépend de ProjetFixtures, UserFixtures, EtiquetteFixtures)
- 40 tâches réparties dans les projets
- Assignées à des utilisateurs aléatoires
- Chaque tâche associée à 1 à 3 étiquettes aléatoires (ManyToMany)
- Priorités et statuts variés

### 8.2 Pagination avec KnpPaginatorBundle

- Installer `knplabs/knp-paginator-bundle`
- Paginer la **liste des projets** : 6 projets par page
- Paginer les **tâches** dans le détail du projet : 10 tâches par page
- Ajouter le **tri par colonnes** (nom, date de création, date limite) via `knp_pagination_sortable()`
- Afficher le **nombre total de résultats**

---

## 🆕 Partie 9 — Extension Twig et Commande Console (2 pts)

> **Concepts évalués** : Extensions Twig personnalisées, Commandes Console, SymfonyStyle (TP7)

### 9.1 Extension Twig `TaskFlowExtension`

Créez `src/Twig/TaskFlowExtension.php` avec :

#### Filtre `time_ago`
Convertit une date en format relatif :
```twig
{{ tache.dateCreation|time_ago }}
{# Affiche : "il y a 3 jours", "il y a 2 heures", etc. #}
```

#### Filtre `priority_icon`
Retourne un icône emoji selon la priorité :
```twig
{{ tache.priorite|priority_icon }}
{# Affiche : 🔵 (basse), 🟢 (moyenne), 🟠 (haute), 🔴 (urgente) #}
```

#### Fonction `progress_bar`
Génère une barre de progression HTML Bootstrap :
```twig
{{ progress_bar(pourcentage) }}
{# Affiche : <div class="progress"><div class="progress-bar" style="width: 65%">65%</div></div> #}
{# Couleur dynamique : vert > 75%, jaune > 50%, orange > 25%, rouge sinon #}
```

Utiliser ces filtres/fonctions dans au moins **3 templates** différents.

### 9.2 Commande Console `app:taskflow:report`

Créez `src/Command/TaskFlowReportCommand.php` :

```php
#[AsCommand(
    name: 'app:taskflow:report',
    description: 'Génère un rapport sur l\'état des projets',
)]
class TaskFlowReportCommand extends Command
{
    // Injection de ProjetRepository, TacheRepository

    // Argument optionnel : --projet=ID pour un projet spécifique
    // Option --overdue : afficher uniquement les projets en retard
    
    // Afficher :
    // - Nombre total de projets et tâches
    // - Répartition par statut (projets et tâches)
    // - Projets en retard (date limite dépassée + tâches non terminées)
    // - Top 5 des utilisateurs les plus actifs (nombre de tâches assignées)
    // Avec SymfonyStyle : title(), table(), warning(), success()
}
```

Exécution :
```bash
php bin/console app:taskflow:report
php bin/console app:taskflow:report --overdue
```

---

## 🆕 Partie 10 — Tests (2 pts)

> **Concepts évalués** : PHPUnit, Tests unitaires, Tests fonctionnels, Tests API (TP6)

### 10.1 Tests unitaires du service `ProjetStatsCalculator`

Créez `tests/Service/ProjetStatsCalculatorTest.php` :

- Tester `getProgressPercentage()` retourne 0 pour un projet sans tâches terminées
- Tester `getProgressPercentage()` retourne 100 quand toutes les tâches sont terminées
- Tester `isOverdue()` retourne `true` si la date limite est dépassée et il reste des tâches
- Tester `getRemainingDays()` retourne un nombre négatif si le projet est en retard

### 10.2 Tests fonctionnels des contrôleurs

Créez `tests/Controller/ProjetControllerTest.php` :

- Tester que la page `/projets` retourne un code **200**
- Tester que la page `/projets` contient un tableau
- Tester que la page `/projets/nouveau` est **interdite** sans authentification (code **302**)
- Tester la **création d'un projet** via soumission de formulaire et vérifier la redirection
- Tester que le **message flash** apparaît après création

### 10.3 Tests de l'API

Créez `tests/Api/ProjetApiTest.php` :

- Tester `GET /api/projets` retourne **200** et le bon content-type (`application/ld+json`)
- Tester `POST /api/projets` avec des données valides retourne **201**
- Tester `POST /api/projets` avec un nom vide retourne **422**

### 10.4 Configuration de test

- Créer `.env.test` avec une base de données de test séparée
- Les fixtures doivent être chargées avant l'exécution des tests

---

## 📦 Livrables

- ✅ Dépôt **GitHub** avec historique de commits structuré (commits réguliers et descriptifs)
- ✅ Fichier `README.md` : instructions d'installation, config Mailtrap, identifiants de test, schéma des relations
- ✅ Fixtures fonctionnelles : `php bin/console doctrine:fixtures:load` charge toutes les données
- ✅ Base peuplée via fixtures : 7 utilisateurs, 6 étiquettes, 8 projets, 40 tâches
- ✅ Tests exécutables : `php bin/phpunit` passe sans erreurs

---

## Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations (OneToMany + ManyToMany) | /3 |
| **CRUD & Templates** : FormTypes, EntityType, ChoiceType, Twig, Flash, CSRF | /4 |
| **Sécurité** : Inscription, Login, Rôles, Hiérarchie, Propriété | /3 |
| **API & Services** : API Platform, Groupes, Swagger, Service injecté | /4 |
| **Session, QueryBuilder, AssetMapper** : Derniers consultés, Recherche, lib externe | /3 |
| **Email & Events** : Mailtrap, TemplatedEmail, Subscriber | /3 |
| **Upload de fichiers** : FileUploader, images projets, pièces jointes tâches | /2 |
| **DataFixtures & Pagination** : Faker, références, KnpPaginator, tri | /2 |
| **Extension Twig & Console** : Filtres, fonctions, commande report | /2 |
| **Tests** : Unitaires, fonctionnels, API, configuration test | /2 |
| **Total** | **/28** |

> **Note** : Le barème est sur 28 points. La note finale sera convertie sur 20.


