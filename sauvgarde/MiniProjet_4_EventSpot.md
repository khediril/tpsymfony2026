# 🎫 Mini-Projet 4 — EventSpot : Plateforme de Gestion d'Événements

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation individuelle  
**Durée estimée** : 6 à 8 heures  
**Barème** : /20

---

## 🎯 Contexte

Vous devez développer **EventSpot**, une plateforme de gestion d'événements. Les organisateurs peuvent créer des événements, les participants peuvent s'y inscrire, et le système envoie automatiquement des emails de confirmation. L'application expose une API et utilise l'ensemble des outils Symfony vus en cours.

> ⚠️ **Ce mini-projet est le plus complet**. Il mobilise **toutes les compétences** des 5 TPs. Il est recommandé aux étudiants souhaitant se démarquer.

---

## 🧩 Partie 1 — Modélisation et Base de données (3 pts)

### Entités à créer

#### 🎪 Evenement
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `titre` | string (255) | NotBlank, Length(min: 5) |
| `description` | text | NotBlank, Length(min: 30) |
| `dateDebut` | datetime | NotNull |
| `dateFin` | datetime | NotNull, GreaterThan(dateDebut) |
| `lieu` | string (255) | NotBlank |
| `capaciteMax` | integer | NotNull, Range(min: 1) |
| `prix` | float | nullable, PositiveOrZero |
| `categorie` | string (30) | Choice: "conference", "atelier", "meetup", "formation", "concert" |
| `statut` | string (20) | Choice: "brouillon", "publie", "complet", "annule", "termine" |
| `dateCreation` | datetime | auto |

#### 📝 Inscription
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `dateInscription` | datetime | auto |
| `statut` | string (15) | Choice: "confirmee", "en_attente", "annulee" |
| `commentaire` | text | nullable, Length(max: 500) |

### Relations

| Relation | Description |
|----------|-------------|
| User → Evenement | **OneToMany** — Un organisateur crée plusieurs événements |
| Evenement → Inscription | **OneToMany** — Un événement a plusieurs inscriptions |
| User → Inscription | **ManyToOne** — Un participant a plusieurs inscriptions |

> 💡 L'entité `Inscription` sert de **table de liaison enrichie** entre `User` et `Evenement` (elle contient des données supplémentaires : date, statut, commentaire).

### Attendus
- ✅ Entités, validations, relations et migrations

---

## 🧾 Partie 2 — CRUD et Interface Web (4 pts)

### 2.1 CRUD Evenement

| Action | Route | Description |
|--------|-------|-------------|
| Accueil | `/` | Les 6 prochains événements publiés |
| Liste | `/evenements` | Tous les événements publiés avec filtres |
| Détail | `/evenements/{id}` | Infos complètes + nombre d'inscrits + bouton inscription |
| Créer | `/evenements/nouveau` | Formulaire `EvenementType` |
| Modifier | `/evenements/{id}/modifier` | Organisateur uniquement |
| Supprimer | `/evenements/{id}/supprimer` | Organisateur ou Admin |

### 2.2 FormTypes

- **EvenementType** : tous les champs avec types appropriés
  - `ChoiceType` pour catégorie et statut
  - `DateTimeType` avec widget `single_text`
  - `MoneyType` pour le prix
- **InscriptionType** : commentaire (TextareaType)

### 2.3 Templates

- **Accueil** : Cards des prochains événements en grille responsive
- **Liste** : Tableau avec badges colorés :
  - Catégorie : 🎤 Conférence | 🔧 Atelier | 👥 Meetup | 📚 Formation | 🎵 Concert
  - Statut : 📝 Brouillon | 🟢 Publié | 🔴 Complet | ⚫ Annulé | ✅ Terminé
- **Détail** :
  - Jauge de remplissage (barre de progression Bootstrap)
  - Texte « X / Y places restantes »
  - Bouton « S'inscrire » ou « Complet » selon la capacité
  - Liste des inscrits (visible uniquement par l'organisateur)
- **Layout** : Navigation responsive avec Bootstrap 5

---

## 🔐 Partie 3 — Sécurité et Rôles (3 pts)

### 3.1 Authentification

- User avec : email, password, pseudo, roles
- Inscription et Login fonctionnels

### 3.2 Rôles et Hiérarchie

```yaml
# security.yaml
role_hierarchy:
    ROLE_ORGANISATEUR: ROLE_USER
    ROLE_ADMIN: ROLE_ORGANISATEUR
```

### 3.3 Autorisations

| Fonctionnalité | Accès |
|----------------|-------|
| Voir les événements publiés | Tout le monde |
| S'inscrire à un événement | `ROLE_USER` |
| Créer un événement | `ROLE_ORGANISATEUR` |
| Modifier/Supprimer un événement | **Organisateur de l'événement** ou `ROLE_ADMIN` |
| Voir la liste des inscrits | **Organisateur** ou `ROLE_ADMIN` |
| Changer le statut d'un événement | **Organisateur** ou `ROLE_ADMIN` |

- Utiliser `#[IsGranted('ROLE_ORGANISATEUR')]` sur les actions de création
- Vérifier la propriété dans les actions de modification/suppression

---

## 📖 Partie 4 — Session et Inscription (3 pts)

### 4.1 Processus d'inscription

Lorsqu'un utilisateur connecté clique sur « S'inscrire » :

1. Vérifier que l'événement n'est pas complet (`nbInscrits < capaciteMax`)
2. Vérifier que l'utilisateur n'est pas déjà inscrit
3. Créer une `Inscription` avec statut « confirmee »
4. Si le nombre d'inscrits atteint la capacité : passer le statut de l'événement à « complet »
5. Message flash de confirmation

### 4.2 Annulation d'inscription

- Route `/inscriptions/{id}/annuler` — change le statut à « annulee »
- Si le statut de l'événement était « complet », le repasser à « publie »

### 4.3 Session — Événements récemment consultés

Utiliser la **session** (`RequestStack`) pour stocker les **5 derniers événements consultés** :

- Sur la page de détail, ajouter l'ID de l'événement à la session
- Afficher un bloc « Vos dernières consultations » dans la sidebar ou en bas de la page d'accueil
- Limiter à 5 éléments (FIFO)

---

## 🛠️ Partie 5 — Service et Email (4 pts)

### 5.1 Service `EvenementManager`

Créez un service `src/Service/EvenementManager.php` :

```php
class EvenementManager
{
    public function __construct(
        private EntityManagerInterface $em,
        private MailerInterface $mailer
    ) {}

    /**
     * Inscrit un utilisateur à un événement
     * Retourne true si l'inscription a réussi, false sinon
     */
    public function inscrire(User $user, Evenement $evenement): bool
    {
        // 1. Vérifier la capacité
        // 2. Vérifier les doublons
        // 3. Créer l'inscription
        // 4. Mettre à jour le statut si complet
        // 5. Envoyer l'email de confirmation
        // 6. Persister
    }

    /**
     * Annule l'inscription d'un utilisateur
     */
    public function annulerInscription(Inscription $inscription): void

    /**
     * Vérifie si un utilisateur est inscrit à un événement
     */
    public function estInscrit(User $user, Evenement $evenement): bool

    /**
     * Retourne le nombre de places restantes
     */
    public function getPlacesRestantes(Evenement $evenement): int
}
```

### 5.2 Email de confirmation (Mailtrap)

Envoyez un email de confirmation lors de l'inscription :

- Utiliser `TemplatedEmail`
- Template `emails/confirmation_inscription.html.twig` :
  - Titre de l'événement
  - Date et lieu
  - Numéro d'inscription
  - Message de bienvenue personnalisé

### 5.3 Email d'annulation

Envoyer un email lorsque l'inscription est annulée par le participant.

---

## 🌐 Partie 6 — API REST (3 pts)

### 6.1 API Platform

Exposez les entités avec API Platform :

#### Evenement — API Resource

```php
#[ApiResource(
    operations: [
        new GetCollection(),
        new Get(),
    ],
    normalizationContext: ['groups' => ['event:read']],
)]
```

> ⚠️ Exposer uniquement en **lecture** (GET). La création/modification se fait via l'interface web.

### 6.2 Groupes de sérialisation

| Propriété | `event:read` |
|-----------|:-:|
| titre | ✅ |
| description | ✅ |
| dateDebut | ✅ |
| dateFin | ✅ |
| lieu | ✅ |
| categorie | ✅ |
| statut | ✅ |
| capaciteMax | ✅ |
| prix | ✅ |
| nbInscrits (méthode) | ✅ |

### 6.3 Endpoint personnalisé (bonus)

Ajoutez un endpoint API personnalisé :
- `GET /api/evenements/prochains` → les 10 prochains événements publiés (QueryBuilder)

---

## 📦 Livrables

- ✅ Dépôt **GitHub** avec historique de commits structuré et progressif
- ✅ Fichier `README.md` complet :
  - Instructions d'installation pas à pas
  - Configuration Mailtrap
  - Identifiants de test (user, organisateur, admin)
  - Descriptif des fonctionnalités
  - Captures d'écran
- ✅ Base peuplée : 3 utilisateurs (rôles différents), 5 événements, 8 inscriptions

---

## 📝 Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations | /3 |
| **CRUD** : Événements fonctionnels, FormTypes, templates Bootstrap | /4 |
| **Sécurité** : Auth, 3 rôles, hiérarchie, propriété | /3 |
| **Session & Inscription** : Processus complet, dernières consultations | /3 |
| **Service & Email** : EvenementManager, emails Mailtrap | /4 |
| **API** : API Platform, groupes de sérialisation | /3 |
| **Total** | **/20** |

### Bonus (+4 pts)

| Bonus | Points |
|-------|--------|
| Endpoint API personnalisé (prochains événements) | +1 |
| Event Subscriber (header personnalisé, log des inscriptions) | +1 |
| AssetMapper avec SweetAlert2 (confirmation suppression) | +1 |
| Dashboard organisateur avec statistiques (nb inscrits, taux remplissage) | +1 |

---

## 🎯 Compétences évaluées

| TP | Compétences mobilisées |
|----|----------------------|
| TP1 | Contrôleurs, Routes (paramètres, contraintes, méthodes HTTP), Twig (héritage, boucles, conditions, filtres), Doctrine (Entity, Migration, Repository) |
| TP2 | FormType, ChoiceType, EntityType, MoneyType, DateTimeType, Validation (Assert, GreaterThan), Relations (OneToMany, ManyToOne), Messages Flash, CRUD complet, CSRF |
| TP3 | User, Inscription, Login/Logout, 3 Rôles avec Hiérarchie, IsGranted, access_control, Propriété des données, createAccessDeniedException |
| TP4 | API Platform (lecture seule), Groupes de sérialisation, Swagger UI, Services personnalisés, Injection de dépendances (constructeur) |
| TP5 | Sessions (RequestStack — dernières consultations), Mailer (Mailtrap, TemplatedEmail), QueryBuilder, Event Subscriber (bonus) |
