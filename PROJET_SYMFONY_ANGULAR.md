# Projet : Application Full-Stack "Blog Manager" (Symfony & Angular)

**Contexte** : Vous avez développé un backend robuste avec Symfony 7.4 (API REST, Sécurité, Doctrine). L'objectif est maintenant de créer une interface moderne et dynamique pour ce backend en utilisant **Angular**.

---

## 🎯 Objectifs du projet

1.  Consommer une **API REST** Symfony avec les services Angular (`HttpClient`).
2.  Mettre en place une architecture **Frontend** propre (Composants, Services, Modèles).
3.  Gérer l'**Authentification** (Login) et la sécurisation des routes (Guards).
4.  Réaliser un **CRUD complet** sur une ressource.
5.  Travailler en mode **Full-Stack** avec une séparation nette des responsabilités.

---

## 🏗️ Architecture Technique

-   **Backend** : Symfony 7.4 + API Platform (votre travail réalisé en TP).
-   **Frontend** : Angular (version 17 ou supérieure).
-   **Style** : Bootstrap 5 ou Angular Material.
-   **Communication** : JSON / REST.

---

## 📋 Fonctionnalités attendues

### 1. Authentification (Cœur du projet)
-   Une page de **Login** permettant de s'authentifier auprès de l'API Symfony.
-   Stockage sécurisé du token d'accès (LocalStorage ou SessionStorage).
-   Affichage du pseudo de l'utilisateur dans la barre de navigation.
-   Bouton de déconnexion.

### 2. Gestion des Articles (CRUD)
-   **Listing** : Affichage de tous les articles sous forme de cartes (Cards).
-   **Filtrage** : Possibilité de filtrer les articles par catégorie.
-   **Détail** : Page affichant l'article complet, son auteur et sa date.
-   **Ajout/Modification** : Un formulaire (Reactive Forms) permettant de créer ou d'éditer un article.
-   **Suppression** : Confirmation avant suppression d'un article.

### 3. Gestion des Catégories
-   Affichage de la liste des catégories existantes.
-   Nombre d'articles par catégorie.

### 4. Expérience Utilisateur (UX)
-   **Guards** : Les pages de création et modification ne doivent être accessibles qu'aux utilisateurs connectés.
-   **Interceptors** : Ajouter automatiquement le token d'authentification dans les headers des requêtes HTTP.
-   **Feedbacks** : Affichage de messages de succès ou d'erreur (Toasts) après chaque action.

---

## 🛠️ Travail à réaliser

### Étape 1 : Préparation du Backend
Assurez-vous que votre API Symfony est configurée pour accepter les requêtes du frontend (**CORS**).
> **Indice** : Utilisez le bundle `nelmio/cors-bundle`.

### Étape 2 : Initialisation Angular
-   Créer un nouveau projet Angular : `ng new blog-frontend`.
-   Générer les composants nécessaires (`login`, `article-list`, `article-detail`, `article-form`, `navbar`).
-   Définir les modèles TypeScript reflétant vos entités Symfony.

### Étape 3 : Services et API
-   Créer un `AuthService` pour la connexion.
-   Créer un `ApiService` (ou `ArticleService`) pour les appels REST.

---

## 📝 Modalités de rendu

-   **Dépôt GitHub** : Un seul dépôt avec deux dossiers (`backend` et `frontend`) ou deux dépôts séparés.
-   **Workflow Git** : Utilisation de branches nommées et de messages de commit clairs.
-   **README** : Un fichier expliquant comment installer et lancer les deux parties de l'application.

---

## ✅ Échelle de notation (Indicative)

| Critère | Points |
|---------|--------|
| Authentification fonctionnelle (Service + Guard) | /5 |
| CRUD Article opérationnel (Liste, Ajout, Modif, Suppr) | /6 |
| Gestion des catégories et filtres | /3 |
| Qualité du code Angular (Architecture, Services) | /3 |
| Design et UX (Bootstrap, Feedbacks) | /2 |
| Qualité du README et de l'installation | /1 |
| **Total** | **/20** |

---

> **💡 Conseil** : Commencez par faire fonctionner le `GET` (affichage de la liste) avant de vous lancer dans l'authentification complexe !
