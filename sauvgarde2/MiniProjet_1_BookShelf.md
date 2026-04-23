# Mini-Projet 1 — BookShelf : Bibliothèque en Ligne

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation individuelle  
**Durée estimée** : 8 à 10 heures  
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

### Relations

| Relation | Description |
|----------|-------------|
| Auteur → Livre | **OneToMany** — Un auteur peut avoir écrit plusieurs livres |
| Genre → Livre | **OneToMany** — Un genre contient plusieurs livres |
| User → Livre | **ManyToOne** — Un utilisateur (ajouté par) peut ajouter plusieurs livres |

### Attendus
- Entités créées avec `make:entity`
- Contraintes de validation (`Assert`) sur chaque entité
- Migrations générées et exécutées
- Relations correctement configurées

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
  - `DateType` avec widget `single_text` pour la date de publication
  - `CheckboxType` pour `disponible`
- Bouton de soumission stylé avec Bootstrap

### 2.3 CRUD Auteur et Genre

Réalisez un CRUD simplifié (liste + création) pour **Auteur** et **Genre**.

### 2.4 Templates Twig

- **Layout** (`base.html.twig`) : Bootstrap 5, navbar avec navigation, footer
- **Liste des livres** : Tableau avec titre, auteur, genre, nb pages
  - Badge coloré pour la disponibilité (🟢 Disponible / 🔴 Indisponible)
  - Badge avec la couleur du genre
- **Détail** : Toutes les informations du livre, lien vers l'auteur
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
| Gérer les Auteurs et Genres | `ROLE_ADMIN` |

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

Créez dans `LivreRepository` :

```php
public function findByFilters(?string $titre, ?Genre $genre, ?bool $disponible): array
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

## Livrables

- ✅ Dépôt **GitHub** avec historique de commits structuré
- ✅ Fichier `README.md` : instructions d'installation, config Mailtrap, identifiants de test, schéma des relations
- ✅ Base peuplée : 3 auteurs, 4 genres, 10 livres, 2 utilisateurs (rôles différents)

---

## 📝 Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations | /3 |
| **CRUD & Templates** : FormTypes, EntityType, Twig, Flash, CSRF | /4 |
| **Sécurité** : Inscription, Login, Rôles, Hiérarchie, Propriété | /3 |
| **API & Services** : API Platform, Groupes, Swagger, Service injecté | /4 |
| **Session, QueryBuilder, AssetMapper** : Liste de lecture, Recherche, lib externe | /3 |
| **Email & Events** : Mailtrap, TemplatedEmail, Subscriber | /3 |
| **Total** | **/20** |


