# TP1 — Introduction au Framework Symfony 7.4

**Module** : Développement Web — Framework PHP  
**Durée** : 3 heures  
**Système** : Ubuntu 24.04 LTS  
**Prérequis** : Notions de base en POO PHP

---

## 🎯 Objectifs pédagogiques

À l'issue de ce TP, l'étudiant sera capable de :

1. Installer et configurer un projet Symfony 7.4
2. Comprendre l'architecture MVC et la structure d'un projet Symfony
3. Créer des contrôleurs et définir des routes
4. Utiliser le moteur de templates **Twig** pour afficher des vues
5. Créer une entité et la persister en base de données avec **Doctrine ORM**

---

## 📋 Sommaire

| Partie | Contenu | Durée estimée |
|--------|---------|---------------|
| 0 | Préparation de l'environnement (Ubuntu 24.04) | 30 min |
| 1 | Création du projet et découverte | 20 min |
| 2 | Contrôleurs et Routage | 35 min |
| 3 | Templates Twig | 35 min |
| 4 | Introduction à Doctrine (Entités & BDD) | 45 min |
| 5 | Exercice de synthèse | 15 min |

---

## Partie 0 — Préparation de l'environnement sous Ubuntu 24.04 (30 min)

Avant de commencer à développer avec Symfony, il est nécessaire d'installer et configurer tous les outils requis sur votre machine.

### 0.1 Mise à jour du système

Ouvrez un terminal et mettez à jour votre système :

```bash
sudo apt update && sudo apt upgrade -y
```

### 0.2 Installation de PHP 8.3 et des extensions nécessaires

Ubuntu 24.04 inclut PHP 8.3 dans ses dépôts officiels :

```bash
sudo apt install -y php php-cli php-common php-mysql php-xml php-curl php-mbstring php-zip php-intl php-bcmath php-gd
```

Vérifiez l'installation :

```bash
php -v
```

Vous devriez voir une sortie similaire à :

```
PHP 8.3.x (cli) ...
```

> **💡 Explication des extensions installées** :
> | Extension | Rôle |
> |-----------|------|
> | `php-mysql` | Connexion à MySQL/MariaDB |
> | `php-xml` | Manipulation XML (requis par Symfony) |
> | `php-curl` | Requêtes HTTP |
> | `php-mbstring` | Support des chaînes multi-octets (UTF-8) |
> | `php-zip` | Compression/décompression (requis par Composer) |
> | `php-intl` | Internationalisation (traductions, formats) |
> | `php-bcmath` | Calculs de précision arbitraire |
> | `php-gd` | Manipulation d'images |

### 0.3 Installation de Composer

**Composer** est le gestionnaire de dépendances de PHP. Il est indispensable pour tout projet Symfony.

```bash
cd ~
curl -sS https://getcomposer.org/installer -o composer-setup.php
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
rm composer-setup.php
```

Vérifiez l'installation :

```bash
composer -V
```

Résultat attendu :

```
Composer version 2.x.x ...
```

### 0.4 Installation de Git

**Git** est le système de contrôle de version indispensable pour tout développeur.

```bash
sudo apt install -y git
```

Configurez votre identité Git :

```bash
git config --global user.name "Votre Nom"
git config --global user.email "votre.email@exemple.com"
```

Vérifiez :

```bash
git --version
```

### 0.5 Installation de MySQL 8

Installez le serveur MySQL :

```bash
sudo apt install -y mysql-server
```

Démarrez le service et activez-le au démarrage :

```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

Connectez-vous à MySQL en tant que root pour créer un utilisateur dédié au développement :

```bash
sudo mysql
```

Dans la console MySQL, exécutez :

```sql
CREATE USER 'dsi2.3'@'localhost' IDENTIFIED BY 'dsi2.3';
GRANT ALL PRIVILEGES ON *.* TO 'dsi2.3'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Testez la connexion avec le nouvel utilisateur :

```bash
mysql -u dsi2.3 -p
# Entrez le mot de passe : dsi2.3
```

> **⚠️ Attention** : En environnement de production, utilisez toujours un mot de passe fort et des privilèges restreints. Ces identifiants sont uniquement pour le développement local.

### 0.6 Installation du Symfony CLI

Le **Symfony CLI** est un outil en ligne de commande qui facilite le développement avec Symfony (serveur local, création de projets, vérification de sécurité…).

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash
sudo apt install -y symfony-cli
```

Vérifiez l'installation :

```bash
symfony version
```

### 0.7 Vérification globale de l'environnement

Utilisez la commande intégrée de Symfony CLI pour vérifier que votre environnement est prêt :

```bash
symfony check:requirements
```

Cette commande vérifie que PHP et toutes les extensions requises sont correctement installées. Vous devriez voir des ✅ pour toutes les vérifications obligatoires.

#### ✏️ Question 1
> Lancez la commande `symfony check:requirements`. Listez les éventuels avertissements (*warnings*) affichés et expliquez leur impact.

### 📋 Résumé de l'environnement

| Outil | Commande de vérification | Version attendue |
|-------|-------------------------|------------------|
| PHP | `php -v` | 8.3.x |
| Composer | `composer -V` | 2.x |
| Git | `git --version` | 2.x |
| MySQL | `mysql --version` | 8.x |
| Symfony CLI | `symfony version` | 5.x |

---

## Partie 1 — Création du projet et découverte (20 min)

### 1.1 Création du projet Symfony

Créez un nouveau projet Symfony avec le **Symfony CLI** :

```bash
symfony new tp1_symfony --webapp
cd tp1_symfony
```

> **💡 Explication** :
> - `symfony new` utilise le Symfony CLI pour créer un nouveau projet.
> - L'option `--webapp` installe automatiquement tous les composants nécessaires pour une application web complète (Twig, Doctrine, Form, Security, Mailer…). Sans cette option, seul le squelette minimal (*skeleton*) est installé.

Initialisez un dépôt Git (le CLI le fait souvent automatiquement, mais vérifiez) :

```bash
git init
git add .
git commit -m "Initial commit - Projet Symfony 7.4"
```

### 1.2 Créer le dépôt distant sur GitHub

Conformément au workflow Git collaboratif, nous allons héberger le projet sur GitHub.

1. Connectez-vous à [github.com](https://github.com) et créez un nouveau dépôt (**New repository**) :
   - **Nom** : `tp1-symfony`
   - **Visibilité** : Public
   - **NE cochez PAS** "Add a README file" (le projet existe déjà localement)

2. Liez votre dépôt local au dépôt distant et poussez le code :

```bash
git remote add origin https://github.com/VOTRE_USERNAME/tp1-symfony.git
git branch -M main
git push -u origin main
```

> **💡 Explication** :
> - `git remote add origin ...` : lie votre dépôt local au dépôt GitHub (nommé `origin`)
> - `git branch -M main` : renomme la branche principale en `main`
> - `git push -u origin main` : pousse le code et établit le suivi entre les branches locale et distante

3. Actualisez la page GitHub : vos fichiers doivent apparaître.

> **⚠️ Règle importante** : À partir de maintenant, **ne poussez jamais directement sur `main`**. Chaque fonctionnalité sera développée sur une **branche dédiée**, puis fusionnée via une **Pull Request** (PR) sur GitHub.

### 1.3 Lancement du serveur de développement

Lancez le serveur local intégré au Symfony CLI :

```bash
symfony server:start -d
```

> **📝 Note** : Le serveur démarre par défaut sur **https://127.0.0.1:8000** (avec support HTTPS automatique). Vous pouvez aussi utiliser l'option `-d` pour le lancer en arrière-plan : `symfony server:start -d`.

Ouvrez votre navigateur sur **https://127.0.0.1:8000**. Vous devriez voir la page d'accueil de Symfony.

### 1.4 Exploration de la structure du projet

Prenez quelques minutes pour explorer l'arborescence du projet :

```
tp1_symfony/
├── config/          # Fichiers de configuration (routes, services, packages)
├── public/          # Point d'entrée web (index.php, assets publics)
├── src/
│   ├── Controller/  # Contrôleurs de l'application
│   ├── Entity/      # Entités Doctrine (modèles)
│   └── Repository/  # Repositories Doctrine
├── templates/       # Templates Twig
├── var/             # Cache et logs
├── vendor/          # Dépendances (géré par Composer)
├── .env             # Variables d'environnement
└── composer.json    # Dépendances du projet
```

#### ✏️ Question 2
> Quel est le rôle du dossier `public/` ? Pourquoi le fichier `index.php` se trouve-t-il dans ce dossier ?

#### ✏️ Question 3
> Ouvrez le fichier `.env` à la racine du projet. Quelle variable d'environnement contrôle le mode de l'application (développement/production) ?

---

## Partie 2 — Contrôleurs et Routage (35 min)

### 🔀 Workflow Git : Créer une branche pour les contrôleurs

Avant de commencer cette partie, créez une branche dédiée :

```bash
git checkout main
git pull origin main
git checkout -b feature-controllers
```

> **💡 Bonne pratique** : Toujours partir d'un `main` à jour avant de créer une nouvelle branche.

### 2.1 Création de votre premier contrôleur

Utilisez le **MakerBundle** pour générer un contrôleur :

```bash
php bin/console make:controller AccueilController
```

Cette commande crée deux fichiers :
- `src/Controller/AccueilController.php`
- `templates/accueil/index.html.twig`

### 2.2 Analyse du contrôleur généré

Ouvrez le fichier `src/Controller/AccueilController.php` :

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class AccueilController extends AbstractController
{
    #[Route('/accueil', name: 'app_accueil')]
    public function index(): Response
    {
        return $this->render('accueil/index.html.twig', [
            'controller_name' => 'AccueilController',
        ]);
    }
}
```

> **💡 Explication** :
> - **`#[Route('/accueil', name: 'app_accueil')]`** : cet attribut PHP associe l'URL `/accueil` à cette méthode. Le paramètre `name` donne un identifiant unique à la route, utilisable ensuite pour générer des liens (ex : `path('app_accueil')` dans Twig).
> - **`$this->render('accueil/index.html.twig', [...])`** : cette méthode fait **deux choses** :
>   1. Elle charge le fichier template Twig situé dans `templates/accueil/index.html.twig`
>   2. Elle lui transmet un tableau de **variables** accessibles dans le template (ici, la variable `controller_name` sera utilisable dans Twig via `{{ controller_name }}`)
>
>   Le résultat est un objet `Response` contenant le HTML généré, que Symfony renvoie au navigateur.
>
> ```
> Contrôleur                          Template Twig                    Navigateur
> ───────────                         ──────────────                   ──────────
> render('accueil/index.html.twig',   Reçoit les variables,            Reçoit le
>   ['controller_name' => '...'])  →  génère le HTML             →     HTML final
> ```

Testez en accédant à **http://localhost:8000/accueil**.

### 2.3 Exercice : Créer des routes supplémentaires

#### Exercice 2.1 — Route avec paramètre

Ajoutez une nouvelle méthode dans `AccueilController` qui affiche un message de bienvenue personnalisé :

```php
#[Route('/bonjour/{prenom}', name: 'app_bonjour')]
public function bonjour(string $prenom): Response
{
    return new Response("<h1>Bonjour $prenom ! Bienvenue sur Symfony 7.4</h1>");
}
```

Testez avec : **http://localhost:8000/bonjour/Marie**

#### Exercice 2.2 — Route avec valeur par défaut et contrainte

Créez une méthode qui affiche le profil d'un utilisateur avec un identifiant numérique :

```php
#[Route('/profil/{id}', name: 'app_profil', requirements: ['id' => '\d+'], defaults: ['id' => 1])]
public function profil(int $id): Response
{
    return new Response("<h1>Profil de l'utilisateur n°$id</h1>");
}
```

#### ✏️ Question 4
> Que se passe-t-il si vous accédez à `/profil/abc` ? Pourquoi ?

#### Exercice 2.3 — Lister les routes

Utilisez la console pour lister toutes les routes de votre application :

```bash
php bin/console debug:router
```

#### ✏️ Question 5
> Combien de routes sont actuellement définies dans votre application ? Identifiez celles que vous avez créées et celles fournies par Symfony (debug, profiler…).

### 🔀 Workflow Git : Committer et pousser les contrôleurs

Maintenant que vos contrôleurs et routes fonctionnent, sauvegardez votre travail :

```bash
git add .
git commit -m "feat: Ajout des contrôleurs AccueilController avec routes et paramètres"
git push origin feature-controllers
```

**Créez une Pull Request sur GitHub :**
1. Allez sur votre dépôt GitHub
2. Cliquez sur **"Compare & pull request"** (ou créez une PR depuis l'onglet "Pull requests")
3. Sélectionnez `base: main` ← `compare: feature-controllers`
4. Titre : `feat: Ajout des contrôleurs et du routage`
5. Cliquez sur **"Create pull request"**, puis **"Merge pull request"** → **"Confirm merge"**
6. (Optionnel) Supprimez la branche distante en cliquant sur **"Delete branch"**

---

## Partie 3 — Templates Twig (35 min)

### 🔀 Workflow Git : Synchroniser et créer une branche pour Twig

```bash
git checkout main
git pull origin main
git checkout -b feature-twig-templates
```

### 3.1 Introduction à Twig

Twig est le moteur de templates par défaut de Symfony. Il permet de séparer la logique métier (PHP) de la présentation (HTML).

**Syntaxe de base :**

| Syntaxe | Usage |
|---------|-------|
| `{{ variable }}` | Afficher une variable |
| `{% instruction %}` | Exécuter une instruction (if, for, block…) |
| `{# commentaire #}` | Commentaire (non affiché) |

### 3.2 Le template de base

Ouvrez le fichier `templates/base.html.twig` :

```twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %}
        {% block javascripts %}{% endblock %}
    </body>
</html>
```

### 3.3 Comprendre l'héritage de templates

L'héritage de templates est un concept fondamental de Twig. Il fonctionne comme l'**héritage en POO** : un template **parent** (ou « layout ») définit la structure commune de toutes les pages, et chaque template **enfant** ne redéfinit que les parties qui changent.

**Analogie** : imaginez un plan de maison (le template parent) qui définit les murs, le toit et les emplacements des pièces. Ensuite, chaque pièce (template enfant) est décorée différemment, mais la structure reste la même.

#### Comment ça fonctionne ?

Le template parent `base.html.twig` définit des **blocs** — des zones remplaçables identifiées par un nom :

```
┌─────────────────────────────────────────────────────────┐
│  base.html.twig (template PARENT)                       │
│                                                         │
│  <!DOCTYPE html>                                        │
│  <html>                                                 │
│    <head>                                               │
│      <title>┌─────────────────────┐</title>             │
│             │ {% block title %}   │ ← bloc remplaçable  │
│             │   "Welcome!"       │ ← valeur par défaut  │
│             │ {% endblock %}      │                      │
│             └─────────────────────┘                      │
│    </head>                                               │
│    <body>                                                │
│      ┌────────────────────────────┐                      │
│      │ {% block body %}           │ ← bloc remplaçable   │
│      │   (vide par défaut)        │                      │
│      │ {% endblock %}             │                      │
│      └────────────────────────────┘                      │
│    </body>                                               │
│  </html>                                                 │
└─────────────────────────────────────────────────────────┘
```

Un template enfant **hérite** du parent avec `{% extends %}` et **remplace** uniquement les blocs souhaités :

```
┌─────────────────────────────────────────────────────────┐
│  accueil/index.html.twig (template ENFANT)              │
│                                                         │
│  {% extends 'base.html.twig' %}   ← hérite du parent    │
│                                                         │
│  {% block title %}Accueil{% endblock %}                  │
│       ↑ remplace "Welcome!" par "Accueil"               │
│                                                         │
│  {% block body %}                                        │
│    <h1>Bienvenue !</h1>            ← remplace le bloc    │
│    <p>Contenu de la page...</p>      body (qui était     │
│  {% endblock %}                      vide par défaut)    │
└─────────────────────────────────────────────────────────┘
```

**Résultat final** généré par Twig (fusion parent + enfant) :

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Accueil</title>          <!-- bloc title remplacé -->
  </head>
  <body>
    <h1>Bienvenue !</h1>            <!-- bloc body remplacé -->
    <p>Contenu de la page...</p>
  </body>
</html>
```

> **💡 Points clés à retenir** :
> - `{% extends 'base.html.twig' %}` **doit être la première ligne** du template enfant
> - Le template enfant ne peut contenir du HTML **que dans des blocs** `{% block %}...{% endblock %}`
> - Si un bloc n'est **pas redéfini** par l'enfant, la **valeur par défaut** du parent est utilisée
> - Ce mécanisme évite la **duplication de code** : la navigation, le footer, les CSS et JS sont écrits une seule fois dans le parent

### 3.4 Exercice : Créer un layout avec navigation

#### Exercice 3.1 — Personnaliser le template de base

Modifiez `templates/base.html.twig` pour y ajouter une barre de navigation et un style CSS minimal :

```twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Mon Application Symfony{% endblock %}</title>
        <!-- Bootstrap 5 CSS -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
        {% block stylesheets %}{% endblock %}
    </head>
    <body class="bg-light">
        <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
            <div class="container">
                <a class="navbar-brand" href="{{ path('app_accueil') }}">Symfony TP1</a>
                <div class="navbar-nav">
                    <a class="nav-link" href="{{ path('app_accueil') }}">🏠 Accueil</a>
                    <a class="nav-link" href="{{ path('app_articles') }}">📰 Articles</a>
                </div>
            </div>
        </nav>

        <div class="container my-5 shadow-sm p-4 bg-white rounded">
            {% block body %}{% endblock %}
        </div>

        <footer class="text-center py-4 text-muted">
            &copy; {{ "now"|date("Y") }} — TP Symfony 7.4
        </footer>

        <!-- Bootstrap 5 JS Bundle -->
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
        {% block javascripts %}{% endblock %}
    </body>
</html>
```

> **💡 Note** : La fonction `path()` génère l'URL à partir du **nom** de la route.

#### Exercice 3.2 — Créer la page d'accueil

Modifiez `templates/accueil/index.html.twig` :

```twig
{% extends 'base.html.twig' %}

{% block title %}Accueil{% endblock %}

{% block body %}
    <h1>👋 Bienvenue sur notre application Symfony !</h1>
    <p>Ceci est votre première application construite avec le framework <strong>Symfony 7.4</strong>.</p>
    <p>Date actuelle : {{ "now"|date("d/m/Y H:i") }}</p>
{% endblock %}
```

#### Exercice 3.3 — Créer une page avec une boucle et des conditions

Créez un nouveau contrôleur pour les articles :

```bash
php bin/console make:controller ArticlesController
```

Modifiez `src/Controller/ArticlesController.php` :

```php
#[Route('/articles', name: 'app_articles')]
public function index(): Response
{
    $articles = [
        ['titre' => 'Introduction à Symfony',    'auteur' => 'Alice',  'publie' => true],
        ['titre' => 'Les bases de Twig',          'auteur' => 'Bob',    'publie' => true],
        ['titre' => 'Doctrine ORM en pratique',   'auteur' => 'Claire', 'publie' => false],
        ['titre' => 'Sécurité avec Symfony',      'auteur' => 'David',  'publie' => true],
        ['titre' => 'API Platform (brouillon)',   'auteur' => 'Eve',    'publie' => false],
    ];

    return $this->render('articles/index.html.twig', [
        'articles' => $articles,
    ]);
}
```

Modifiez `templates/articles/index.html.twig` :

```twig
{% extends 'base.html.twig' %}

{% block title %}Liste des articles{% endblock %}

{% block body %}
    <h1 class="mb-4">📰 Liste des articles</h1>

    {% if articles is empty %}
        <div class="alert alert-info">Aucun article disponible.</div>
    {% else %}
        <table class="table table-striped table-hover mt-4">
            <thead class="table-dark">
                <tr>
                    <th>#</th>
                    <th>Titre</th>
                    <th>Auteur</th>
                    <th>Statut</th>
                </tr>
            </thead>
            <tbody>
                {% for article in articles %}
                    <tr>
                        <td>{{ loop.index }}</td>
                        <td>{{ article.titre }}</td>
                        <td>{{ article.auteur }}</td>
                        <td>
                            {% if article.publie %}
                                <span class="badge bg-success">✅ Publié</span>
                            {% else %}
                                <span class="badge bg-warning text-dark">📝 Brouillon</span>
                            {% endif %}
                        </td>
                    </tr>
                {% endfor %}
            </tbody>
        </table>
        <p class="mt-3 text-muted">
            Total : {{ articles|length }} article(s)
        </p>
    {% endif %}
{% endblock %}
```

#### ✏️ Question 6
> Expliquez le rôle de `loop.index` dans la boucle `{% for %}`. Quelle est la différence avec `loop.index0` ?

### 🔀 Workflow Git : Committer et pousser les templates

```bash
git add .
git commit -m "feat: Mise en place des templates Twig avec layout, navigation et liste d'articles"
git push origin feature-twig-templates
```

**Créez une Pull Request sur GitHub :**
1. Titre : `feat: Templates Twig avec héritage et boucles`
2. `base: main` ← `compare: feature-twig-templates`
3. Fusionnez la PR et supprimez la branche

---

## Partie 4 — Introduction à Doctrine ORM (45 min)

### 🔀 Workflow Git : Synchroniser et créer une branche pour Doctrine

```bash
git checkout main
git pull origin main
git checkout -b feature-doctrine-articles
```

### 4.1 Configuration de la base de données

Ouvrez le fichier `.env` et configurez la connexion à la base de données avec l'utilisateur créé en Partie 0 :

```dotenv
DATABASE_URL="mysql://dsi2.3:dsi2.3@127.0.0.1:3306/tp1_symfony?serverVersion=8.0"
```

> **📝 Note** : Les identifiants `dsi2.3:dsi2.3` correspondent à l'utilisateur MySQL créé lors de la préparation de l'environnement (Partie 0).

Créez la base de données :

```bash
php bin/console doctrine:database:create
```

### 4.2 Création d'une entité

Utilisez le MakerBundle pour créer une entité `Article` :

```bash
php bin/console make:entity Article
```

Ajoutez les propriétés suivantes lorsque l'assistant vous le demande :

| Propriété | Type | Nullable |
|-----------|------|----------|
| `titre` | `string` (255) | non |
| `contenu` | `text` | non |
| `auteur` | `string` (100) | non |
| `dateCreation` | `datetime` | non |
| `publie` | `boolean` | non |

Vérifiez le fichier généré dans `src/Entity/Article.php`.

### 4.3 Migration de la base de données

Générez et exécutez la migration :

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

#### ✏️ Question 7
> Ouvrez le fichier de migration généré dans `migrations/`. Quelle requête SQL y est contenue ? Que fait-elle ?

### 4.4 Exercice : CRUD basique avec Doctrine

#### Exercice 4.1 — Insérer des données

Créez une route pour ajouter un article en base de données. Modifiez `ArticlesController` :

```php
use App\Entity\Article;
use Doctrine\ORM\EntityManagerInterface;

// ... dans la classe ArticlesController

#[Route('/articles/nouveau', name: 'app_article_nouveau')]
public function nouveau(EntityManagerInterface $em): Response
{
    $article = new Article();
    $article->setTitre('Mon premier article');
    $article->setContenu('Ceci est le contenu de mon premier article créé avec Doctrine.');
    $article->setAuteur('Étudiant');
    $article->setDateCreation(new \DateTime());
    $article->setPublie(true);

    $em->persist($article);
    $em->flush();

    return new Response("Article créé avec l'id : " . $article->getId());
}
```

Testez en accédant à **http://localhost:8000/articles/nouveau**.

> **⚠️ Important** : Placez cette route **avant** la route `/articles` dans votre contrôleur, ou assurez-vous que l'ordre des routes ne crée pas de conflit.

#### Exercice 4.2 — Lire les données depuis la BDD

Modifiez la méthode `index()` de `ArticlesController` pour récupérer les articles depuis la base de données :

```php
use App\Repository\ArticleRepository;

#[Route('/articles', name: 'app_articles')]
public function index(ArticleRepository $articleRepository): Response
{
    $articles = $articleRepository->findAll();

    return $this->render('articles/index.html.twig', [
        'articles' => $articles,
    ]);
}
```

Mettez à jour le template `templates/articles/index.html.twig` pour utiliser les objets `Article` :

```twig
{% extends 'base.html.twig' %}

{% block title %}Liste des articles{% endblock %}

{% block body %}
    <h1 class="mb-4">📰 Articles (depuis la BDD)</h1>

    {% if articles is empty %}
        <div class="alert alert-info">
            Aucun article en base de données. 
            <a href="{{ path('app_article_nouveau') }}" class="alert-link">Créer un article</a>
        </div>
    {% else %}
        <div class="table-responsive">
            <table class="table table-bordered table-hover mt-4">
                <thead class="table-dark text-center">
                    <tr>
                        <th>ID</th>
                        <th>Titre</th>
                        <th>Auteur</th>
                        <th>Date</th>
                        <th>Statut</th>
                    </tr>
                </thead>
                <tbody>
                    {% for article in articles %}
                        <tr>
                            <td class="text-center">{{ article.id }}</td>
                            <td>{{ article.titre }}</td>
                            <td>{{ article.auteur }}</td>
                            <td>{{ article.dateCreation|date('d/m/Y') }}</td>
                            <td class="text-center">
                                {% if article.publie %}
                                    <span class="badge bg-success">✅ Publié</span>
                                {% else %}
                                    <span class="badge bg-warning text-dark">📝 Brouillon</span>
                                {% endif %}
                            </td>
                        </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
        <p class="mt-3 text-muted">
            {{ articles|length }} article(s) en base de données
        </p>
    {% endif %}
{% endblock %}
```

#### Exercice 4.3 — Afficher un article par son ID

Ajoutez une méthode dans `ArticlesController` :

```php
#[Route('/articles/{id}', name: 'app_article_detail', requirements: ['id' => '\d+'])]
public function detail(Article $article): Response
{
    return $this->render('articles/detail.html.twig', [
        'article' => $article,
    ]);
}
```

Créez le template `templates/articles/detail.html.twig` :

```twig
{% extends 'base.html.twig' %}

{% block title %}{{ article.titre }}{% endblock %}

{% block body %}
    <h1 class="display-4">{{ article.titre }}</h1>
    <p class="text-muted mb-4 pb-2 border-bottom">
        Par <strong class="text-dark">{{ article.auteur }}</strong> 
        — {{ article.dateCreation|date('d/m/Y à H:i') }}
        {% if article.publie %}
            <span class="badge bg-success ms-2">✅ Publié</span>
        {% else %}
            <span class="badge bg-warning text-dark ms-2">📝 Brouillon</span>
        {% endif %}
    </p>
    
    <div class="lead mb-5">
        {{ article.contenu }}
    </div>

    <a href="{{ path('app_articles') }}" class="btn btn-outline-secondary">
        ← Retour à la liste
    </a>
{% endblock %}
```

> **💡 Note** : Symfony utilise le **ParamConverter** pour convertir automatiquement le paramètre `{id}` de l'URL en objet `Article`. Si l'article n'existe pas, une page 404 est automatiquement retournée.

#### ✏️ Question 8
> Qu'est-ce que le **ParamConverter** ? Quel avantage apporte-t-il par rapport à une recherche manuelle via le Repository ?

### 🔀 Workflow Git : Committer et pousser Doctrine

```bash
git add .
git commit -m "feat: Entité Article avec Doctrine, migration et CRUD basique"
git push origin feature-doctrine-articles
```

**Créez une Pull Request sur GitHub :**
1. Titre : `feat: Intégration Doctrine ORM avec entité Article`
2. `base: main` ← `compare: feature-doctrine-articles`
3. Fusionnez la PR, puis synchronisez en local :

```bash
git checkout main
git pull origin main
```

---

## Partie 5 — Exercice de synthèse (15 min)

### 🧩 Mini-projet : Gestion de tâches (To-Do List)

En vous basant sur tout ce que vous avez appris, réalisez les étapes suivantes **de manière autonome** :

1. **Créer une entité `Tache`** avec les propriétés :
   - `titre` (string, 255)
   - `description` (text, nullable)
   - `terminee` (boolean, défaut `false`)
   - `dateCreation` (datetime)

2. **Générer et exécuter la migration**

3. **Créer un `TacheController`** avec les routes suivantes :
   - `/taches` → liste toutes les tâches
   - `/taches/ajouter` → crée une tâche en dur et la persiste
   - `/taches/{id}` → affiche le détail d'une tâche

4. **Créer les templates Twig** correspondants, en héritant de `base.html.twig`

5. **Ajouter un lien** dans la barre de navigation vers la page des tâches

6. **Workflow Git** : Développez tout sur une branche `feature-taches`, puis créez une PR pour fusionner dans `main`

#### 🏆 Bonus
- Ajoutez une route `/taches/{id}/terminer` qui passe une tâche à `terminee = true`
- Affichez les tâches terminées avec un style barré (`text-decoration: line-through`)
- Triez les tâches : non terminées en premier

---

## 📚 Ressources utiles

| Ressource | Lien |
|-----------|------|
| Documentation Symfony | https://symfony.com/doc/current/index.html |
| Documentation Twig | https://twig.symfony.com/doc/ |
| Documentation Doctrine | https://www.doctrine-project.org/projects/orm/en/current/index.html |
| Symfony CLI | https://symfony.com/download |
| SymfonyCasts (tutoriels) | https://symfonycasts.com/ |

---

## 📝 Récapitulatif des commandes utiles

```bash
# Créer un projet (avec Symfony CLI)
symfony new mon_projet --webapp

# Lancer le serveur
symfony server:start

# Vérifier l'environnement
symfony check:requirements

# Générer un contrôleur
php bin/console make:controller NomController

# Générer une entité
php bin/console make:entity NomEntite

# Créer la base de données
php bin/console doctrine:database:create

# Générer une migration
php bin/console make:migration

# Exécuter les migrations
php bin/console doctrine:migrations:migrate

# Lister les routes
php bin/console debug:router

# Vider le cache
php bin/console cache:clear

# --- Workflow Git ---
# Synchroniser avant de commencer une nouvelle feature
git checkout main
git pull origin main

# Créer une branche de fonctionnalité
git checkout -b feature-nom-feature

# Committer avec des messages conventionnels
git add .
git commit -m "feat: Description de la fonctionnalité"

# Pousser la branche vers GitHub
git push origin feature-nom-feature

# Après fusion de la PR, revenir sur main
git checkout main
git pull origin main
```


