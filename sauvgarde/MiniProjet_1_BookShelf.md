# 📚 Mini-Projet 1 — BookShelf : Bibliothèque en Ligne

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation individuelle  
**Durée estimée** : 6 à 8 heures  
**Barème** : /20

---

## 🎯 Contexte

Vous devez développer une application web **BookShelf** permettant de gérer une bibliothèque personnelle en ligne. Les utilisateurs peuvent parcourir les livres, s'inscrire pour ajouter des livres à leur liste de lecture, et gérer le catalogue.

---

## 🧩 Partie 1 — Modélisation et Base de données (5 pts)

### Entités à créer

#### 📗 Livre
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `titre` | string (255) | NotBlank, Length(min: 3, max: 255) |
| `resume` | text | NotBlank, Length(min: 20) |
| `isbn` | string (13) | NotBlank, Regex (format ISBN) |
| `nbPages` | integer | NotNull, Range(min: 1, max: 5000) |
| `datePublication` | date | NotNull |
| `disponible` | boolean | — |

#### 👤 Auteur
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `nom` | string (100) | NotBlank |
| `prenom` | string (100) | NotBlank |
| `biographie` | text | nullable |
| `nationalite` | string (50) | — |

#### 📂 Genre
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
| User → Livre | **ManyToMany** — Un utilisateur peut avoir plusieurs livres en favoris (liste de lecture) |

### Attendus
- ✅ Entités créées avec `make:entity`
- ✅ Contraintes de validation (`Assert`) sur chaque entité
- ✅ Migrations générées et exécutées
- ✅ Relations correctement configurées (annotations Doctrine)

---

## 🧾 Partie 2 — CRUD et Formulaires (5 pts)

### 2.1 CRUD Livre (obligatoire)

Créez un contrôleur `LivreController` avec les actions suivantes :

| Action | Route | Méthode |
|--------|-------|---------|
| Liste des livres | `/livres` | GET |
| Détail d'un livre | `/livres/{id}` | GET |
| Créer un livre | `/livres/nouveau` | GET/POST |
| Modifier un livre | `/livres/{id}/modifier` | GET/POST |
| Supprimer un livre | `/livres/{id}/supprimer` | POST |

### 2.2 FormTypes

- Créer `LivreType` avec les champs appropriés
- Utiliser `EntityType` pour les champs `auteur` et `genre` (sélecteur lié à la BDD)
- Ajouter un bouton de soumission stylé avec Bootstrap

### 2.3 Templates Twig

- **Liste** : Afficher les livres dans un tableau Bootstrap avec :
  - Titre, Auteur, Genre, Nombre de pages
  - Badge coloré pour la disponibilité (🟢 Disponible / 🔴 Indisponible)
  - Badge avec la couleur du genre
- **Détail** : Page complète avec toutes les informations du livre
- **Formulaire** : Formulaire stylé avec Bootstrap 5
- **Messages flash** après chaque opération (création, modification, suppression)

### 2.4 CRUD Auteur et Genre

Réalisez un CRUD simplifié (liste + création) pour les entités **Auteur** et **Genre**.

---

## 🔐 Partie 3 — Sécurité et Authentification (4 pts)

### 3.1 Système d'utilisateurs

- Créer l'entité `User` avec `make:user`
- Ajouter un champ `pseudo` (string, 50)
- Implémenter l'**inscription** (`make:registration-form`) avec validation
- Implémenter la **connexion/déconnexion** (`make:auth`)

### 3.2 Contrôle d'accès

| Fonctionnalité | Accès requis |
|----------------|-------------|
| Consulter les livres | Tout le monde |
| Créer / Modifier / Supprimer un livre | `ROLE_USER` |
| Gérer les Auteurs et Genres | `ROLE_ADMIN` |

- Utiliser `#[IsGranted]` sur les contrôleurs
- Utiliser `is_granted()` dans Twig pour masquer les boutons
- Configurer la **hiérarchie de rôles** : `ROLE_ADMIN` hérite de `ROLE_USER`

### 3.3 Navigation conditionnelle

- Afficher le pseudo de l'utilisateur connecté dans la navbar
- Boutons Connexion/Inscription si non connecté
- Bouton Déconnexion si connecté

---

## 📖 Partie 4 — Session : Liste de lecture (3 pts)

Implémentez une fonctionnalité de **Liste de lecture** stockée en session :

### 4.1 Ajouter un livre à la liste

- Bouton « 📖 Ajouter à ma liste » sur la page de détail de chaque livre
- Stocker les IDs des livres dans la session (`RequestStack`)
- Empêcher les doublons

### 4.2 Consulter la liste de lecture

- Route `/ma-liste` affichant les livres de la liste de lecture
- Récupérer les IDs depuis la session, puis charger les livres via le Repository
- Afficher le nombre de livres dans la navbar (badge dynamique)

### 4.3 Retirer un livre de la liste

- Bouton « ❌ Retirer » pour chaque livre de la liste
- Message flash de confirmation

---

## 🔍 Partie 5 — QueryBuilder et Recherche (3 pts)

### 5.1 Recherche

Ajoutez un formulaire de recherche sur la page de liste des livres :
- **Recherche par titre** (partielle, insensible à la casse)
- **Filtre par genre** (sélecteur déroulant)
- **Filtre par disponibilité** (case à cocher)

### 5.2 Méthodes personnalisées dans le Repository

Créez dans `LivreRepository` :

```php
// Recherche multi-critères
public function findByFilters(?string $titre, ?Genre $genre, ?bool $disponible): array

// Les 5 derniers livres ajoutés
public function findLastAdded(int $limit = 5): array

// Nombre de livres par genre (pour un dashboard)
public function countByGenre(): array
```

---

## 📦 Livrables

- ✅ Dépôt **GitHub** avec un historique de commits structuré
- ✅ Fichier `README.md` contenant :
  - Instructions d'installation (`composer install`, `migrations`, etc.)
  - Identifiants de test (admin, user)
  - Schéma des relations entre entités
- ✅ Base de données peuplée (au minimum : 3 auteurs, 4 genres, 10 livres, 2 utilisateurs)

---

## 📝 Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations et relations correctes | /5 |
| **CRUD** : Fonctionnel avec FormTypes, EntityType, messages flash | /5 |
| **Sécurité** : Inscription, Login, Rôles, IsGranted | /4 |
| **Session** : Liste de lecture fonctionnelle | /3 |
| **QueryBuilder** : Recherche et méthodes repository | /3 |
| **Total** | **/20** |

### Bonus (+3 pts)

| Bonus | Points |
|-------|--------|
| Pagination des livres (KnpPaginator ou manuelle) | +1 |
| Tri dynamique (par titre, date, nb pages) | +1 |
| Template de base soigné avec Bootstrap et navigation responsive | +1 |

---

## 🎯 Compétences évaluées

| TP | Compétences mobilisées |
|----|----------------------|
| TP1 | Contrôleurs, Routes (paramètres, contraintes), Twig (héritage, boucles, conditions), Doctrine (Entity, Migration) |
| TP2 | FormType, EntityType, Validation (Assert), Relations Doctrine (OneToMany, ManyToMany), Messages Flash, CRUD |
| TP3 | User, Inscription, Login/Logout, Rôles, IsGranted, access_control, Hiérarchie |
| TP5 | Sessions (RequestStack), QueryBuilder |
