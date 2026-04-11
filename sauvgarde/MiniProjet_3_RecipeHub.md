# 🍳 Mini-Projet 3 — RecipeHub : Plateforme de Partage de Recettes

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation individuelle  
**Durée estimée** : 6 à 8 heures  
**Barème** : /20

---

## 🎯 Contexte

Vous devez développer **RecipeHub**, une plateforme de partage de recettes de cuisine. Les utilisateurs peuvent parcourir les recettes, les créer, les noter, et les exposer via une API REST. L'application utilise AssetMapper pour gérer les assets frontend.

---

## 🧩 Partie 1 — Modélisation et Base de données (4 pts)

### Entités à créer

#### 🍽️ Recette
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `titre` | string (255) | NotBlank, Length(min: 5, max: 255) |
| `description` | text | NotBlank, Length(min: 30) |
| `instructions` | text | NotBlank |
| `tempsPreparation` | integer | NotNull, Range(min: 1) — en minutes |
| `tempsCuisson` | integer | nullable — en minutes |
| `difficulte` | string (15) | Choice: "facile", "moyen", "difficile" |
| `nbPersonnes` | integer | NotNull, Range(min: 1, max: 50) |
| `dateCreation` | datetime | NotNull (auto) |
| `publiee` | boolean | — |

#### 🥕 Ingredient
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `nom` | string (100) | NotBlank |
| `quantite` | string (50) | NotBlank (ex: "200g", "3 cuillères") |

#### 📂 CategorieRecette
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `nom` | string (50) | NotBlank, Unique |
| `icone` | string (10) | nullable (emoji: 🥗, 🍝, 🍰) |

### Relations

| Relation | Description |
|----------|-------------|
| CategorieRecette → Recette | **OneToMany** — Une catégorie contient plusieurs recettes |
| Recette → Ingredient | **OneToMany** — Une recette contient plusieurs ingrédients |
| User → Recette | **ManyToOne** — Un utilisateur (auteur) peut créer plusieurs recettes |

### Attendus
- ✅ Entités créées, validées et migrées
- ✅ Relations configurées correctement

---

## 🧾 Partie 2 — CRUD et Interface Web (4 pts)

### 2.1 CRUD Recette

| Action | Route | Description |
|--------|-------|-------------|
| Liste | `/recettes` | Cards Bootstrap avec image, titre, catégorie, difficulté, temps total |
| Détail | `/recettes/{id}` | Page complète : ingrédients, instructions, infos auteur |
| Créer | `/recettes/nouvelle` | Formulaire avec `RecetteType` |
| Modifier | `/recettes/{id}/modifier` | Accessible uniquement à l'auteur |
| Supprimer | `/recettes/{id}/supprimer` | POST avec CSRF |

### 2.2 FormTypes

- **RecetteType** : titre, description, instructions, tempsPreparation, tempsCuisson, difficulte (`ChoiceType`), nbPersonnes, categorie (`EntityType`), publiee
- **IngredientType** : nom, quantite
- Dans `RecetteType`, intégrer la gestion des ingrédients avec `CollectionType` :

```php
->add('ingredients', CollectionType::class, [
    'entry_type' => IngredientType::class,
    'allow_add' => true,
    'allow_delete' => true,
    'by_reference' => false,
    'label' => 'Ingrédients',
])
```

> 💡 Si `CollectionType` est trop complexe, une alternative acceptable est de gérer les ingrédients via un CRUD séparé accessible depuis la page de détail de la recette.

### 2.3 Templates

- **Liste** : Afficher les recettes sous forme de **cards** (grille responsive)
  - Badge de difficulté coloré (🟢 Facile | 🟡 Moyen | 🔴 Difficile)
  - Icône de la catégorie
  - Temps total (préparation + cuisson)
- **Détail** : Layout en deux colonnes :
  - Gauche : Ingrédients (liste)
  - Droite : Instructions (étapes numérotées)
- Messages flash pour toutes les opérations

---

## 🔐 Partie 3 — Sécurité (3 pts)

### 3.1 Authentification

- User avec : email, password, pseudo
- Inscription et Login fonctionnels

### 3.2 Autorisations

| Fonctionnalité | Accès |
|----------------|-------|
| Consulter les recettes publiées | Tout le monde |
| Créer une recette | `ROLE_USER` |
| Modifier/Supprimer | **Auteur uniquement** ou `ROLE_ADMIN` |
| Gérer les catégories | `ROLE_ADMIN` |

- L'auteur de la recette est affecté automatiquement avec `$this->getUser()`
- Protection avec `#[IsGranted]` et vérification de propriété

---

## 🌐 Partie 4 — API REST (5 pts)

### 4.1 API Platform

Installez API Platform et exposez les entités suivantes :

```php
#[ApiResource(
    normalizationContext: ['groups' => ['recette:read']],
    denormalizationContext: ['groups' => ['recette:write']],
)]
```

### 4.2 Groupes de sérialisation

Configurez les groupes pour chaque entité :

#### Recette
| Propriété | `recette:read` | `recette:write` |
|-----------|:-:|:-:|
| titre | ✅ | ✅ |
| description | ✅ | ✅ |
| instructions | ✅ | ✅ |
| tempsPreparation | ✅ | ✅ |
| tempsCuisson | ✅ | ✅ |
| difficulte | ✅ | ✅ |
| nbPersonnes | ✅ | ✅ |
| dateCreation | ✅ | ❌ |
| categorie (nom) | ✅ | ✅ |
| ingredients | ✅ | ❌ |

#### CategorieRecette
| Propriété | `categorie:read` | `categorie:write` |
|-----------|:-:|:-:|
| nom | ✅ | ✅ |
| icone | ✅ | ✅ |
| recettes (titres) | ✅ | ❌ |

### 4.3 Validation API

- Les contraintes de validation (`Assert`) doivent s'appliquer automatiquement à l'API
- Tester un POST avec des données invalides → vérifier la réponse 422

### 4.4 Tests requis (via Swagger UI)

Documentez vos tests dans le README :
1. `GET /api/recettes` → Liste paginée
2. `GET /api/recettes/{id}` → Détail avec ingrédients
3. `POST /api/recettes` → Création (body JSON)
4. `PUT /api/recettes/{id}` → Modification
5. `DELETE /api/recettes/{id}` → Suppression

---

## 🎨 Partie 5 — AssetMapper et QueryBuilder (4 pts)

### 5.1 AssetMapper

- Installer une bibliothèque externe via `importmap:require` (au choix) :
  - **SweetAlert2** : Pour les confirmations de suppression
  - **Animate.css** : Pour des animations sur les cards
  - Ou toute autre bibliothèque pertinente
- Utiliser la bibliothèque dans au minimum **2 pages** différentes
- Vérifier que `{{ importmap('app') }}` est dans `base.html.twig`

### 5.2 QueryBuilder — Recherche avancée

Sur la page de liste des recettes, implémentez :

- **Recherche par titre** (partielle)
- **Filtre par catégorie** (sélecteur déroulant)
- **Filtre par difficulté** (boutons radio ou sélecteur)
- **Filtre par temps total** (ex: moins de 30 min, 30-60 min, plus de 60 min)

Créez dans `RecetteRepository` :

```php
/**
 * Recherche multi-critères avec QueryBuilder
 */
public function findByFilters(
    ?string $titre,
    ?CategorieRecette $categorie,
    ?string $difficulte,
    ?int $tempsMax
): array {
    $qb = $this->createQueryBuilder('r')
        ->andWhere('r.publiee = :publiee')
        ->setParameter('publiee', true);

    if ($titre) {
        $qb->andWhere('r.titre LIKE :titre')
           ->setParameter('titre', '%' . $titre . '%');
    }

    // ... continuer pour les autres filtres

    return $qb->orderBy('r.dateCreation', 'DESC')
              ->getQuery()
              ->getResult();
}
```

### 5.3 Statistiques d'accueil

Créez une page d'accueil `/` affichant :
- Nombre total de recettes publiées
- Les 3 dernières recettes (QueryBuilder avec `setMaxResults`)
- Nombre de recettes par catégorie (requête groupée)

---

## 📦 Livrables

- ✅ Dépôt **GitHub** avec historique de commits structuré
- ✅ Fichier `README.md` avec :
  - Instructions d'installation
  - Documentation de l'API (endpoints et exemples)
  - Captures d'écran de Swagger UI
  - Identifiants de test
- ✅ Base peuplée : au minimum 4 catégories, 8 recettes avec ingrédients, 2 utilisateurs

---

## 📝 Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations | /4 |
| **CRUD** : Recettes et ingrédients, FormTypes, templates Bootstrap | /4 |
| **Sécurité** : Auth, propriété des recettes, rôles | /3 |
| **API** : API Platform, groupes de sérialisation, validation | /5 |
| **AssetMapper & QueryBuilder** : Lib externe, recherche multi-critères | /4 |
| **Total** | **/20** |

### Bonus (+3 pts)

| Bonus | Points |
|-------|--------|
| CollectionType fonctionnel pour les ingrédients | +1 |
| Page d'accueil avec statistiques dynamiques | +1 |
| Pagination des recettes | +1 |

---

## 🎯 Compétences évaluées

| TP | Compétences mobilisées |
|----|----------------------|
| TP1 | Contrôleurs, Routes, Twig (héritage, conditions, boucles, filtres), Doctrine |
| TP2 | FormType, ChoiceType, EntityType, CollectionType, Validation, Relations, Flash, CRUD |
| TP3 | User, Inscription, Login, Rôles, IsGranted, Propriété des données |
| TP4 | API Platform, #[ApiResource], Groupes de sérialisation, Swagger UI, Validation API |
| TP5 | AssetMapper (importmap), QueryBuilder (méthodes Repository avancées) |
