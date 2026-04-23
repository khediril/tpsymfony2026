# Mini-Projet 3 — RecipeHub : Plateforme de Partage de Recettes

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation individuelle  
**Durée estimée** : 8 à 10 heures  
**Barème** : /20

---

## Contexte

Vous devez développer **RecipeHub**, une plateforme de partage de recettes de cuisine. Les utilisateurs peuvent parcourir les recettes, en créer, les ajouter à leurs favoris (session), et recevoir un email de notification. L'application expose une API REST.

---

## Partie 1 — Modélisation et Base de données (3 pts)

> **Concepts** : Doctrine ORM, Entités, Migrations, Relations, Validation (TP1 + TP2)

### Entités

**Recette** : `titre` (string 255, NotBlank, Length min:5), `description` (text, NotBlank, Length min:30), `instructions` (text, NotBlank), `tempsPreparation` (integer, Range min:1), `tempsCuisson` (integer, nullable), `difficulte` (string, Choice: facile/moyen/difficile), `nbPersonnes` (integer, Range min:1 max:50), `dateCreation` (datetime, auto), `publiee` (boolean)

**Ingredient** : `nom` (string 100, NotBlank), `quantite` (string 50, NotBlank — ex: "200g")

**CategorieRecette** : `nom` (string 50, NotBlank, Unique), `icone` (string 10, nullable — emoji), `description` (text, nullable)

### Relations

- CategorieRecette → Recette : **OneToMany**
- Recette → Ingredient : **OneToMany**
- User → Recette : **ManyToOne** (auteur)

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

### FormTypes

- **RecetteType** : `EntityType` (catégorie), `ChoiceType` (difficulté), `CheckboxType` (publiée)
- **IngredientType** : nom, quantité

### Templates Twig

- Layout Bootstrap 5 avec navbar et footer
- Liste : Cards en grille responsive, badges difficulté (🟢 Facile | 🟡 Moyen | 🔴 Difficile), icône catégorie, temps total
- Détail : Deux colonnes (ingrédients à gauche, instructions à droite)
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
| Gérer catégories | `ROLE_ADMIN` |

- `#[IsGranted]` sur les contrôleurs, `is_granted()` dans Twig
- Auteur automatique : `$recette->setAuteur($this->getUser())`
- Navigation conditionnelle (pseudo, connexion/déconnexion)

---

## Partie 4 — API REST et Services (4 pts)

> **Concepts** : API Platform, Groupes de sérialisation, Swagger, Service, Injection (TP4)

### API Platform

Exposer **Recette** et **CategorieRecette** avec `#[ApiResource]`.

### Groupes de sérialisation (Recette)

- `recette:read` : titre, description, instructions, temps, difficulté, nbPersonnes, dateCreation, catégorie (nom), ingrédients
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

Recherche sur la page de liste : par **titre** (partiel), par **catégorie** (EntityType), par **difficulté** (ChoiceType).

```php
// RecetteRepository
public function findByFilters(?string $titre, ?CategorieRecette $cat, ?string $diff): array
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

## 📦 Livrables

- Dépôt **GitHub** avec commits structurés
- `README.md` : installation, config Mailtrap, identifiants de test, schéma des relations
- Base peuplée : 4 catégories, 8 recettes avec ingrédients, 2 utilisateurs

---

## Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations | /3 |
| **CRUD & Templates** : FormTypes, EntityType, ChoiceType, Twig, Flash, CSRF | /4 |
| **Sécurité** : Inscription, Login, Rôles, Hiérarchie, Propriété | /3 |
| **API & Services** : API Platform, Groupes, Swagger, Service injecté | /4 |
| **Session, QueryBuilder, AssetMapper** : Favoris, Recherche, lib externe | /3 |
| **Email & Events** : Mailtrap, TemplatedEmail, Subscriber | /3 |
| **Total** | **/20** |

