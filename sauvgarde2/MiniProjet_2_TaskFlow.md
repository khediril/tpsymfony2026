# Mini-Projet 2 — TaskFlow : Gestionnaire de Tâches Collaboratif

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation individuelle  
**Durée estimée** : 8 à 10 heures  
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
| Etiquette → Tache | **OneToMany** — Une étiquette peut être appliquée à plusieurs tâches |

### Attendus
- Entités créées avec `make:entity`
- Contraintes de validation (`Assert`) sur chaque entité
- Migrations générées et exécutées
- Relations correctement configurées

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

- **ProjetType** : nom, description, dateLimite, statut (`ChoiceType`)
- **TacheType** : titre, description, priorite (`ChoiceType`), dateEcheance, assignation (`EntityType` → User), etiquette (`EntityType` → Etiquette)
- Boutons de soumission stylés avec Bootstrap

### 2.4 CRUD Étiquette

Réalisez un CRUD simplifié (liste + création) pour **Étiquette**.

### 2.5 Templates Twig

- **Layout** (`base.html.twig`) : Bootstrap 5, navbar avec navigation, footer
- **Liste des projets** : Tableau avec nom, statut (badge coloré), nombre de tâches, créateur
  - 🔵 Planifié | 🟡 En cours | 🟢 Terminé | 🔴 Annulé
- **Détail projet** : Infos du projet + tableau de ses tâches avec badges de priorité
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

Créez dans `ProjetRepository` :

```php
public function findByFilters(?string $nom, ?string $statut, ?User $createur): array
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

## 📦 Livrables

- ✅ Dépôt **GitHub** avec historique de commits structuré
- ✅ Fichier `README.md` : instructions d'installation, config Mailtrap, identifiants de test, schéma des relations
- ✅ Base peuplée : 2 utilisateurs (rôles différents), 3 étiquettes, 3 projets, 10 tâches

---

## Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations | /3 |
| **CRUD & Templates** : FormTypes, EntityType, ChoiceType, Twig, Flash, CSRF | /4 |
| **Sécurité** : Inscription, Login, Rôles, Hiérarchie, Propriété | /3 |
| **API & Services** : API Platform, Groupes, Swagger, Service injecté | /4 |
| **Session, QueryBuilder, AssetMapper** : Derniers consultés, Recherche, lib externe | /3 |
| **Email & Events** : Mailtrap, TemplatedEmail, Subscriber | /3 |
| **Total** | **/20** |

