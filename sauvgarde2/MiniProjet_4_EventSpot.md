# Mini-Projet 4 — EventSpot : Plateforme d'Événements

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation individuelle  
**Durée estimée** : 8 à 10 heures  
**Barème** : /20

---

## Contexte

Vous devez développer **EventSpot**, une plateforme de gestion d'événements. Les organisateurs créent des événements, les participants s'y inscrivent et reçoivent un email de confirmation. L'application expose une API REST et utilise les sessions pour les événements récemment consultés.

---

## Partie 1 — Modélisation et Base de données (3 pts)

> **Concepts** : Doctrine ORM, Entités, Migrations, Relations, Validation (TP1 + TP2)

### Entités

**Evenement** : `titre` (string 255, NotBlank, Length min:5), `description` (text, NotBlank, Length min:30), `dateDebut` (datetime, NotNull), `dateFin` (datetime, NotNull), `lieu` (string 255, NotBlank), `capaciteMax` (integer, Range min:1), `prix` (float, nullable, PositiveOrZero), `categorie` (string 30, Choice: conference/atelier/meetup/formation/concert), `statut` (string 20, Choice: brouillon/publie/complet/annule), `dateCreation` (datetime, auto)

**Inscription** : `dateInscription` (datetime, auto), `statut` (string 15, Choice: confirmee/en_attente/annulee), `commentaire` (text, nullable, Length max:500)

**Lieu** : `nom` (string 100, NotBlank, Unique), `adresse` (string 255, NotBlank), `ville` (string 100, NotBlank), `capacite` (integer, Range min:1)

### Relations

- Lieu → Evenement : **OneToMany** (un lieu accueille plusieurs événements)
- Evenement → Inscription : **OneToMany** (un événement a plusieurs inscriptions)
- User → Evenement : **ManyToOne** (organisateur)
- User → Inscription : **ManyToOne** (participant)

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

### FormTypes

- **EvenementType** : `EntityType` (lieu), `ChoiceType` (catégorie, statut), `DateTimeType` (widget single_text), `MoneyType` (prix)
- **InscriptionType** : commentaire (TextareaType)

### Templates Twig

- Layout Bootstrap 5 avec navbar et footer
- Liste : Tableau avec badges colorés pour catégorie (🎤 Conférence | 🔧 Atelier | 👥 Meetup | 📚 Formation | 🎵 Concert) et statut (📝 Brouillon | 🟢 Publié | 🔴 Complet | ⚫ Annulé)
- Détail : Jauge de remplissage (barre de progression), « X / Y places », bouton S'inscrire ou Complet
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
| Gérer les lieux | `ROLE_ADMIN` |

- `#[IsGranted]` sur les contrôleurs, `is_granted()` dans Twig
- Organisateur automatique : `$evenement->setOrganisateur($this->getUser())`
- Navigation conditionnelle (pseudo, connexion/déconnexion)

---

## Partie 4 — API REST et Services (4 pts)

> **Concepts** : API Platform, Groupes de sérialisation, Swagger, Service, Injection (TP4)

### API Platform

Exposer **Evenement** et **Lieu** avec `#[ApiResource]`.

### Groupes de sérialisation (Evenement)

- `event:read` : titre, description, dateDebut, dateFin, lieu (nom + ville), categorie, statut, capaciteMax, prix
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

Recherche sur la page de liste : par **titre** (partiel), par **catégorie** (ChoiceType), par **ville du lieu** (EntityType ou texte).

```php
// EvenementRepository
public function findByFilters(?string $titre, ?string $categorie, ?string $ville): array
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

## Livrables

- Dépôt **GitHub** avec commits structurés
- `README.md` : installation, config Mailtrap, identifiants de test, schéma des relations
- Base peuplée : 3 lieux, 5 événements, 8 inscriptions, 3 utilisateurs (rôles différents)

---

## 📝 Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations | /3 |
| **CRUD & Templates** : FormTypes, EntityType, ChoiceType, Twig, Flash, CSRF | /4 |
| **Sécurité** : Inscription, Login, Rôles, Hiérarchie, Propriété | /3 |
| **API & Services** : API Platform, Groupes, Swagger, Service injecté | /4 |
| **Session, QueryBuilder, AssetMapper** : Derniers consultés, Recherche, lib externe | /3 |
| **Email & Events** : Mailtrap, TemplatedEmail, Subscriber | /3 |
| **Total** | **/20** |

---


