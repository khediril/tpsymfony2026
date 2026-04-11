# 🗂️ Mini-Projet 2 — TaskFlow : Gestionnaire de Tâches Collaboratif

**Module** : Développement Web — Framework PHP (Symfony 7.4)  
**Type** : Évaluation individuelle  
**Durée estimée** : 6 à 8 heures  
**Barème** : /20

---

## 🎯 Contexte

Vous devez développer **TaskFlow**, une application de gestion de tâches collaboratives. Les utilisateurs peuvent créer des projets, y ajouter des tâches, les assigner à d'autres utilisateurs, et recevoir des notifications par email lorsqu'une tâche leur est assignée.

---

## 🧩 Partie 1 — Modélisation et Base de données (4 pts)

### Entités à créer

#### 📁 Projet
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `nom` | string (100) | NotBlank, Length(min: 3) |
| `description` | text | nullable |
| `dateCreation` | datetime | NotNull (auto) |
| `dateLimite` | date | NotNull, GreaterThan(dateCreation) |
| `statut` | string (20) | Choice: "planifié", "en_cours", "terminé", "annulé" |

#### ✅ Tache
| Propriété | Type | Contraintes |
|-----------|------|-------------|
| `id` | integer (auto) | — |
| `titre` | string (255) | NotBlank, Length(min: 5) |
| `description` | text | nullable |
| `priorite` | string (10) | Choice: "basse", "moyenne", "haute", "urgente" |
| `statut` | string (20) | Choice: "a_faire", "en_cours", "terminee" |
| `dateCreation` | datetime | NotNull (auto) |
| `dateEcheance` | date | nullable |

### Relations

| Relation | Description |
|----------|-------------|
| Projet → Tache | **OneToMany** — Un projet contient plusieurs tâches |
| User → Projet | **ManyToOne** — Un utilisateur (créateur) possède plusieurs projets |
| User → Tache | **ManyToOne** — Une tâche est assignée à un utilisateur |

### Attendus
- ✅ Entités créées et migrées
- ✅ Toutes les contraintes de validation en place
- ✅ Relations fonctionnelles

---

## 🧾 Partie 2 — CRUD et Interface Web (4 pts)

### 2.1 CRUD Projet

| Action | Route | Description |
|--------|-------|-------------|
| Liste | `/projets` | Tableau avec nom, statut (badge coloré), nb tâches, créateur |
| Détail | `/projets/{id}` | Infos du projet + liste de ses tâches |
| Créer | `/projets/nouveau` | Formulaire avec `ProjetType` |
| Modifier | `/projets/{id}/modifier` | Réutilisation du FormType |
| Supprimer | `/projets/{id}/supprimer` | Avec confirmation CSRF |

### 2.2 CRUD Tache

| Action | Route | Description |
|--------|-------|-------------|
| Créer | `/projets/{id}/taches/nouvelle` | Formulaire avec `TacheType` (EntityType pour le projet pré-rempli) |
| Modifier | `/taches/{id}/modifier` | Modifier le statut, la priorité, l'assignation |
| Supprimer | `/taches/{id}/supprimer` | POST avec token CSRF |
| Changer statut | `/taches/{id}/statut/{statut}` | Raccourci pour changer le statut rapidement |

### 2.3 FormTypes

- **ProjetType** : nom, description, dateLimite, statut
- **TacheType** : titre, description, priorité, dateEcheance, assignation (EntityType → User)
- Utiliser `ChoiceType` pour les statuts et priorités

### 2.4 Templates

- Colorer les priorités :
  - 🔵 Basse | 🟢 Moyenne | 🟠 Haute | 🔴 Urgente
- Colorer les statuts :
  - ⬜ À faire | 🟡 En cours | 🟢 Terminée
- Afficher une **barre de progression** du projet (% tâches terminées)
- Messages flash pour toutes les opérations

---

## 🔐 Partie 3 — Sécurité et Propriété (4 pts)

### 3.1 Authentification

- Entité `User` avec champs : email, password, pseudo, roles
- Inscription et Connexion fonctionnelles
- Navigation dynamique (connecté / non connecté)

### 3.2 Autorisations

| Fonctionnalité | Accès |
|----------------|-------|
| Voir les projets publics | Tout le monde |
| Créer un projet | `ROLE_USER` |
| Modifier/Supprimer un projet | **Créateur du projet uniquement** ou `ROLE_ADMIN` |
| Créer/Assigner une tâche | `ROLE_USER` (membre du projet) |

### 3.3 Propriété des données

- Le créateur du projet est automatiquement `$this->getUser()`
- Seul le créateur peut modifier ou supprimer son projet
- Vérifier avec : `if ($projet->getCreateur() !== $this->getUser()) { throw ... }`

---

## 🛠️ Partie 4 — Service personnalisé (3 pts)

### 4.1 Service `ProjetStatsCalculator`

Créez un service `src/Service/ProjetStatsCalculator.php` qui calcule :

```php
class ProjetStatsCalculator
{
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

### 4.2 Utilisation

- Injecter le service dans le contrôleur (injection par constructeur)
- Afficher les statistiques sur la page de détail du projet
- **Bonus** : Créer une **Extension Twig** pour appeler le service directement dans les templates

---

## 📧 Partie 5 — Email et Événements (5 pts)

### 5.1 Notification par email

Lorsqu'une tâche est **assignée à un utilisateur**, envoyez un email de notification :

- **De** : `noreply@taskflow.com`
- **À** : l'email de l'utilisateur assigné
- **Sujet** : « Nouvelle tâche assignée : {titre} »
- **Corps** : Template Twig `emails/tache_assignee.html.twig` contenant :
  - Titre de la tâche
  - Nom du projet
  - Priorité
  - Date d'échéance
  - Lien vers la tâche

Configuration : Utiliser **Mailtrap** (voir le Guide d'installation Mailtrap).

### 5.2 Event Subscriber

Créez un `TacheSubscriber` qui écoute l'événement `kernel.response` :

```php
class TacheSubscriber implements EventSubscriberInterface
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
        // Ajouter un header X-Total-Projets: {nombre}
    }
}
```

### 5.3 Email de résumé

Créez une route `/projets/{id}/envoyer-resume` (réservée au créateur) qui envoie un email récapitulatif du projet :
- Statistiques d'avancement (via le service `ProjetStatsCalculator`)
- Liste des tâches avec leur statut
- Utiliser un `TemplatedEmail` avec un template Twig stylé

---

## 📦 Livrables

- ✅ Dépôt **GitHub** avec un historique de commits structuré
- ✅ Fichier `README.md` avec :
  - Instructions d'installation
  - Configuration Mailtrap (sans les identifiants réels)
  - Identifiants de test
  - Captures d'écran de l'application
- ✅ Base peuplée : au minimum 2 utilisateurs, 3 projets, 10 tâches

---

## 📝 Grille d'évaluation

| Critère | Points |
|---------|--------|
| **Modélisation** : Entités, validations, relations | /4 |
| **CRUD** : Projets et Tâches fonctionnels, FormTypes, badges colorés | /4 |
| **Sécurité** : Auth complète, propriété des projets, rôles | /4 |
| **Service** : ProjetStatsCalculator fonctionnel et injecté | /3 |
| **Email & Events** : Notification email, Subscriber, email de résumé | /5 |
| **Total** | **/20** |

### Bonus (+3 pts)

| Bonus | Points |
|-------|--------|
| Extension Twig pour les statistiques du projet | +1 |
| Tableau de bord (dashboard) avec compteurs globaux | +1 |
| Filtrage des tâches par priorité et statut (QueryBuilder) | +1 |

---

## 🎯 Compétences évaluées

| TP | Compétences mobilisées |
|----|----------------------|
| TP1 | Contrôleurs, Routes paramétrées, Twig (héritage, boucles, conditions), Doctrine |
| TP2 | FormType, ChoiceType, EntityType, Validation, Relations (OneToMany, ManyToOne), Flash, CRUD |
| TP3 | User, Inscription, Login, Rôles, IsGranted, Propriété des données |
| TP4 | Services personnalisés, Injection de dépendances (constructeur), Extension Twig |
| TP5 | Mailer (Mailtrap, TemplatedEmail), Event Subscriber (KernelEvents) |
