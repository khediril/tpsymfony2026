# TP3 — Sécurité, Authentification et Autorisation avec Symfony 7.4

**Module** : Développement Web — Framework PHP  
**Durée** : 3 heures  
**Prérequis** : Avoir terminé le TP2 (formulaires, relations Doctrine, Bootstrap)

---

## 🎯 Objectifs pédagogiques

À l'issue de ce TP, l'étudiant sera capable de :

1.  Créer une entité **User** compatible avec le système de sécurité de Symfony
2.  Mettre en place un système d'**authentification** complet (Login/Logout)
3.  Implémenter un formulaire d'**inscription** avec hashage des mots de passe
4.  Gérer les **autorisations** et le contrôle d'accès (Roles, Access Control)
5.  Établir une relation entre les données (Articles) et les utilisateurs (**Propriété des données**)
6.  Utiliser les attributs `#[IsGranted]` pour protéger les contrôleurs

---

## 📋 Sommaire

| Partie | Contenu | Durée estimée |
|--------|---------|---------------|
| 1 | L'entité User et le système de sécurité | 45 min |
| 2 | Inscription et hashage de mots de passe | 40 min |
| 3 | Authentification (Login / Logout) | 35 min |
| 4 | Autorisation et Contrôle d'accès | 40 min |
| 5 | Exercice de synthèse : Propriété des articles | 20 min |

---

## ⚙️ Préparation

Reprenez votre projet `tp1_symfony` (qui contient déjà le travail du TP2).

```bash
cd tp1_symfony
symfony server:start
```

### 🔀 Workflow Git : Synchroniser et créer une branche pour la sécurité

Avant de commencer, authentifiez-vous et créez une branche dédiée :

```bash
git checkout main
git pull origin main
git checkout -b feature-security-auth
```

---

## Partie 1 — L'entité User et le système de sécurité (45 min)

### 1.1 Installer le bundle Security

Si ce n'est pas déjà fait (l'option `--webapp` l'installe normalement), installez le bundle de sécurité :

```bash
composer require security
```

### 1.2 Créer l'entité User

Utilisez le MakerBundle pour générer l'entité utilisateur. Symfony propose une commande spécifique pour configurer correctement les interfaces nécessaires (`UserInterface`, `PasswordAuthenticatedUserInterface`).

```bash
php bin/console make:user
```

**Répondez aux questions ainsi :**
- The name of the User class? **User**
- Do you want to store user data in the database (via Doctrine)? **yes**
- Which property will be the "display name" (login ID)? **email**
- Does this user need to hash passwords? **yes**

Cette commande génère :
- `src/Entity/User.php`
- `src/Repository/UserRepository.php`
- Met à jour `config/packages/security.yaml`

### 1.3 Analyse du fichier `security.yaml`

Ouvrez `config/packages/security.yaml`. Ce fichier centralise toute la configuration de sécurité.

#### ✏️ Question 1
> Qu'est-ce qu'un **Password Hasher** ? Pourquoi Symfony utilise-t-il l'algorithme `auto` par défaut ?

#### ✏️ Question 2
> À quoi sert la section `providers` ? Quel est le rôle du `UserProvider` dans Symfony ?

### 1.4 Mise à jour et Migration

Ajoutons un champ `pseudo` à notre entité User :

```bash
php bin/console make:entity User
# Ajoutez le champ 'pseudo' (string, 50, not nullable)
```

Générez et exécutez la migration :

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

---

## Partie 2 — Inscription et Hashage (40 min)

### 2.1 Générer le formulaire d'inscription

Symfony facilite la création du système d'inscription :

```bash
php bin/console make:registration-form
```

**Répondez aux questions :**
- Do you want to add a unique check for the email? **yes**
- Do you want to automatically authenticate the user after registration? **yes**
- (Si demandé) Voulez-vous envoyer un email de vérification ? **no** (pour simplifier le TP)

Cette commande génère :
- `src/Form/RegistrationFormType.php`
- `src/Controller/RegistrationController.php`
- `templates/registration/register.html.twig`

### 2.2 Adapter le template Bootstrap

Ouvrez `templates/registration/register.html.twig`. Remplacez le contenu pour utiliser les classes Bootstrap 5 :

```twig
{% extends 'base.html.twig' %}

{% block title %}Inscription{% endblock %}

{% block body %}
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card shadow">
                <div class="card-body p-5">
                    <h1 class="h3 mb-4 font-weight-normal text-center">Créer un compte</h1>

                    {{ form_errors(registrationForm) }}

                    {{ form_start(registrationForm) }}
                        <div class="mb-3">
                            {{ form_row(registrationForm.email, {'attr': {'class': 'form-control'}}) }}
                        </div>
                        <div class="mb-3">
                            {{ form_row(registrationForm.plainPassword, {
                                'label': 'Mot de passe',
                                'attr': {'class': 'form-control'}
                            }) }}
                        </div>
                        <div class="form-check mb-3">
                            {{ form_row(registrationForm.agreeTerms, {'attr': {'class': 'form-check-input'}}) }}
                        </div>

                        <button type="submit" class="btn btn-primary w-100 py-2 mt-3">S'inscrire</button>
                    {{ form_end(registrationForm) }}
                </div>
            </div>
        </div>
    </div>
{% endblock %}
```

> **💡 Note** : Le champ `plainPassword` n'existe pas en base de données. Il est utilisé uniquement pour récupérer le mot de passe en clair, qui est ensuite hashé dans le contrôleur via `UserPasswordHasherInterface`.

#### ✏️ Question 3
> Dans `RegistrationController`, pourquoi utilise-t-on `plainPassword` au lieu de `password` directement pour le formulaire ? Quel est le risque si on stockait le mot de passe en clair ?

---

## Partie 3 — Authentification : Login / Logout (35 min)

### 3.1 Créer le système d'authentification

```bash
php bin/console make:auth
```

**Répondez aux questions :**
- What style of authentication? **1** (Login form authenticator)
- The class name of the authenticator? **AppAuthenticator**
- The controller name? **SecurityController**
- Do you want to generate a `/logout` URL? **yes**

### 3.2 Configurer la redirection après login

Ouvrez `src/Security/AppAuthenticator.php`. Trouvez la méthode `onAuthenticationSuccess` :

```php
public function onAuthenticationSuccess(Request $request, TokenInterface $token, string $firewallName): ?Response
{
    if ($targetPath = $this->getTargetPath($request->getSession(), $firewallName)) {
        return new RedirectResponse($targetPath);
    }

    // Modifiez cette ligne pour rediriger vers l'accueil
    return new RedirectResponse($this->urlGenerator->generate('app_accueil'));
}
```

### 3.3 Mettre à jour la navigation

Dans `templates/base.html.twig`, modifiez la barre de navigation pour afficher des liens différents selon que l'utilisateur est connecté ou non :

```twig
<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
    <div class="container">
        <a class="navbar-brand" href="{{ path('app_accueil') }}">Symfony TP3</a>
        <div class="navbar-nav me-auto">
            <a class="nav-link" href="{{ path('app_accueil') }}">🏠 Accueil</a>
            <a class="nav-link" href="{{ path('app_articles') }}">📰 Articles</a>
        </div>
        <div class="navbar-nav">
            {% if app.user %}
                <span class="nav-link text-white me-3">👤 {{ app.user.userIdentifier }}</span>
                <a class="btn btn-outline-danger btn-sm" href="{{ path('app_logout') }}">Déconnexion</a>
            {% else %}
                <a class="nav-link" href="{{ path('app_login') }}">Connexion</a>
                <a class="btn btn-primary btn-sm ms-2" href="{{ path('app_register') }}">Inscription</a>
            {% endif %}
        </div>
    </div>
</nav>
```

#### ✏️ Question 4
> Qu'est-ce que l'objet `app` dans Twig ? D'où vient `app.user` ?

### 3.4 Créer le template de connexion

Ouvrez `templates/security/login.html.twig`. Modifiez-le pour un rendu moderne avec Bootstrap :

```twig
{% extends 'base.html.twig' %}

{% block title %}Connexion{% endblock %}

{% block body %}
    <div class="row justify-content-center">
        <div class="col-md-5">
            <form method="post">
                {% if error %}
                    <div class="alert alert-danger">{{ error.messageKey|trans(error.messageData, 'security') }}</div>
                {% endif %}

                {% if app.user %}
                    <div class="alert alert-info mb-3">
                        Vous êtes déjà connecté en tant que {{ app.user.userIdentifier }}, 
                        <a href="{{ path('app_logout') }}">Déconnexion</a>
                    </div>
                {% endif %}

                <h1 class="h3 mb-4 font-weight-normal text-center">🔐 Connexion</h1>
                
                <div class="mb-3">
                    <label for="inputEmail" class="form-label">Email</label>
                    <input type="email" value="{{ last_username }}" name="email" id="inputEmail" class="form-control" autocomplete="email" required autofocus>
                </div>
                
                <div class="mb-3">
                    <label for="inputPassword" class="form-label">Mot de passe</label>
                    <input type="password" name="password" id="inputPassword" class="form-control" autocomplete="current-password" required>
                </div>

                <input type="hidden" name="_csrf_token" value="{{ csrf_token('authenticate') }}">

                <div class="checkbox mb-3">
                    <label>
                        <input type="checkbox" name="_remember_me"> Se souvenir de moi
                    </label>
                </div>

                <button class="btn btn-lg btn-success w-100 mt-2" type="submit">Se connecter</button>
            </form>
        </div>
    </div>
{% endblock %}
```

---

## Partie 4 — Autorisation et Contrôle d'accès (40 min)

### 4.1 Protéger des routes via `security.yaml`

Retournez dans `config/packages/security.yaml`. Dans la section `access_control`, ajoutez une règle pour restreindre l'accès à la création d'articles :

```yaml
access_control:
    - { path: ^/articles/nouveau, roles: ROLE_USER }
    - { path: ^/categories, roles: ROLE_ADMIN }
```

### 4.2 Protéger via les contrôleurs (Attributs)

Une méthode plus précise consiste à utiliser l'attribut `#[IsGranted]`. Cela permet de restreindre l'accès au niveau d'une classe entière ou d'une méthode spécifique.

Modifiez `src/Controller/ArticlesController.php` :

```php
use Symfony\Component\Security\Http\Attribute\IsGranted;

// On peut restreindre l'accès à toute la création/édition
#[Route('/articles/nouveau', name: 'app_article_nouveau')]
#[IsGranted('ROLE_USER')]
public function nouveau(...) 
{
    // ...
}
```

> **💡 Explication** : Si un utilisateur non connecté ou n'ayant pas le rôle requis tente d'accéder à cette méthode, Symfony intercepte la requête et redirige vers la page de connexion (pour les anonymes) ou affiche une erreur 403 (Accès interdit).

### 4.3 Masquer des éléments dans Twig

On utilise `is_granted()` pour n'afficher les boutons que si l'utilisateur a les droits :

```twig
{% if is_granted('ROLE_USER') %}
    <a href="{{ path('app_article_nouveau') }}" class="btn btn-primary">➕ Créer un article</a>
{% endif %}
```

#### ✏️ Question 5
> Quelle est la différence entre **Authentification** et **Autorisation** ?

#### ✏️ Question 6
> Qu'est-ce que la **Hiérarchie de rôles** ? Comment configurer `security.yaml` pour que `ROLE_ADMIN` possède automatiquement les droits de `ROLE_USER` ?

---

## Partie 5 — Exercice de synthèse : Propriété des données (20 min)

### 🧩 Objectif : Lier les articles à leurs auteurs

1.  **Relation Doctrine** : Ajoutez une relation `ManyToOne` sur l'entité `Article` vers l'entité `User` (nommez le champ `auteur_user`).
2.  **Mise à jour automatique** : Dans la méthode `nouveau()` du contrôleur, affectez l'utilisateur connecté comme auteur de l'article :
    ```php
    $article->setAuteurUser($this->getUser());
    ```
3.  **Restriction d'édition** : Modifiez la méthode `modifier()` pour que **seul l'auteur de l'article** (ou un admin) puisse le modifier.
    > **Indice** : Utilisez `$this->getUser()` et comparez avec `$article->getAuteurUser()`. Si ce n'est pas le bon utilisateur, lancez une exception :
    > `throw $this->createAccessDeniedException('Vous n\'êtes pas l\'auteur !');`

---

## 📚 Ressources utiles

| Ressource | Lien |
|-----------|------|
| Documentation Sécurité | https://symfony.com/doc/current/security.html |
| Authentication vs Authorization | https://symfony.com/doc/current/security.html#authentication-vs-authorization |
| Entity User | https://symfony.com/doc/current/security.html#the-user |
| IsGranted Attribute | https://symfony.com/doc/current/security.html#securing-controllers-and-other-services |

---

## 📝 Récapitulatif des commandes

```bash
# Générer le système de sécurité (User)
php bin/console make:user

# Générer l'authentification (Login)
php bin/console make:auth

# Générer l'inscription
php bin/console make:registration-form

# Vérifier les rôles de l'utilisateur actuel (via le Profiler)
# Cliquer sur l'icône cadenas dans la barre de debug
```

---

## ✅ Critères d'évaluation

| Critère | Points |
|---------|--------|
| Entité User correctement créée et migrée | /3 |
| Système d'inscription fonctionnel avec Bootstrap | /4 |
| Connexion et déconnexion opérationnelles | /3 |
| Protection des routes (Access Control + Attributes) | /4 |
| Relation Article ↔ User et exercice de synthèse | /4 |
| Réponses aux questions | /2 |
| **Total** | **/20** |

---

> **📌 Rendu** : Fournissez le **lien vers votre dépôt GitHub**. Assurez-vous d'avoir créé une branche `feature-security-auth` et d'avoir effectué une **Pull Request** vers `main`. Incluez vos réponses aux questions dans `REPONSES.md`.
