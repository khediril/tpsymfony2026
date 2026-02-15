# TP2 — Formulaires, Validation et Relations Doctrine avec Symfony 7.4

**Module** : Développement Web — Framework PHP  
**Durée** : 3 heures  
**Prérequis** : Avoir réalisé le TP1 (contrôleurs, routes, Twig, Doctrine basique)

---

## 🎯 Objectifs pédagogiques

À l'issue de ce TP, l'étudiant sera capable de :

1. Créer et gérer des formulaires avec le composant **Form** de Symfony
2. Valider les données saisies avec le composant **Validator**
3. Définir des **relations entre entités** avec Doctrine (OneToMany / ManyToOne)
4. Utiliser les **messages flash** pour informer l'utilisateur
5. Mettre en place les opérations CRUD complètes (Créer, Lire, Modifier, Supprimer)

---

## 📋 Sommaire

| Partie | Contenu | Durée estimée |
|--------|---------|---------------|
| 1 | Formulaires Symfony | 50 min |
| 2 | Validation des données | 30 min |
| 3 | Relations Doctrine (Catégorie ↔ Article) | 45 min |
| 4 | CRUD complet avec modification et suppression | 35 min |
| 5 | Exercice de synthèse | 20 min |

---

## ⚙️ Préparation

Reprenez le projet du TP1 (`tp1_symfony`). Vérifiez que tout fonctionne :

```bash
cd tp1_symfony
symfony server:start
```

Assurez-vous que le composant **Form** et **Validator** sont bien installés :

```bash
composer require form validator
```

### 🔀 Workflow Git : Synchroniser et créer une branche pour les formulaires

Avant de commencer, synchronisez votre branche `main` avec le dépôt distant et créez une branche dédiée :

```bash
git checkout main
git pull origin main
git checkout -b feature-forms-validation
```

> **💡 Rappel** : On ne travaille jamais directement sur `main`. Chaque fonctionnalité est développée sur une branche dédiée, puis fusionnée via une Pull Request (PR) sur GitHub.

---

## Partie 1 — Formulaires Symfony (50 min)

### 1.1 Comprendre l'architecture des formulaires

Dans Symfony, un formulaire est défini dans une classe dédiée appelée **FormType**. Cette approche sépare la logique de construction du formulaire du contrôleur.

```
Utilisateur → Formulaire HTML → FormType (validation + mapping) → Entité → Base de données
```

### 1.2 Créer un FormType pour l'entité Article

Générez le formulaire avec le MakerBundle :

```bash
php bin/console make:form ArticleType Article
```

Cette commande crée le fichier `src/Form/ArticleType.php`. Ouvrez-le et observez le code généré.

Modifiez-le pour personnaliser les champs :

```php
<?php

namespace App\Form;

use App\Entity\Article;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\CheckboxType;
use Symfony\Component\Form\Extension\Core\Type\DateTimeType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ArticleType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('titre', TextType::class, [
                'label' => 'Titre de l\'article',
                'attr' => [
                    'placeholder' => 'Saisissez le titre...',
                    'class' => 'form-control',
                ],
            ])
            ->add('contenu', TextareaType::class, [
                'label' => 'Contenu',
                'attr' => [
                    'rows' => 8,
                    'placeholder' => 'Rédigez votre article...',
                    'class' => 'form-control',
                ],
            ])
            ->add('auteur', TextType::class, [
                'label' => 'Auteur',
                'attr' => [
                    'placeholder' => 'Nom de l\'auteur',
                    'class' => 'form-control',
                ],
            ])
            ->add('dateCreation', DateTimeType::class, [
                'label' => 'Date de création',
                'widget' => 'single_text',
                'data' => new \DateTime(),
                'attr' => ['class' => 'form-control'],
            ])
            ->add('publie', CheckboxType::class, [
                'label' => 'Publier immédiatement ?',
                'required' => false,
                'attr' => ['class' => 'form-check-input'],
                'label_attr' => ['class' => 'form-check-label'],
                'row_attr' => ['class' => 'form-check mb-3'],
            ])
            ->add('enregistrer', SubmitType::class, [
                'label' => '💾 Enregistrer',
                'attr' => ['class' => 'btn btn-primary w-100'],
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Article::class,
        ]);
    }
}
```

> **💡 Explication** :
> - Chaque `->add()` définit un champ du formulaire.
> - Le second argument spécifie le **type de champ** (TextType, TextareaType, etc.).
> - Le troisième argument est un tableau d'**options** (label, attributs HTML, etc.).
> - `data_class` lie le formulaire à l'entité `Article`.

#### ✏️ Question 1
> Quel est l'avantage de créer un FormType dans une classe séparée plutôt que de construire le formulaire directement dans le contrôleur ?

### 1.3 Utiliser le formulaire dans le contrôleur

Modifiez la méthode `nouveau()` de `ArticlesController` pour utiliser le formulaire au lieu de créer l'article en dur :

```php
use App\Form\ArticleType;
use Symfony\Component\HttpFoundation\Request;

#[Route('/articles/nouveau', name: 'app_article_nouveau')]
public function nouveau(Request $request, EntityManagerInterface $em): Response
{
    $article = new Article();
    
    // Création du formulaire
    $form = $this->createForm(ArticleType::class, $article);
    
    // Traitement de la requête
    $form->handleRequest($request);
    
    if ($form->isSubmitted() && $form->isValid()) {
        $em->persist($article);
        $em->flush();
        
        // Message flash de confirmation
        $this->addFlash('success', 'Article créé avec succès !');
        
        return $this->redirectToRoute('app_articles');
    }
    
    return $this->render('articles/nouveau.html.twig', [
        'formulaire' => $form,
    ]);
}
```

> **💡 Points clés** :
> - `handleRequest()` remplit automatiquement l'entité avec les données soumises.
> - `isSubmitted()` vérifie si le formulaire a été soumis.
> - `isValid()` vérifie que les données respectent les contraintes de validation.
> - `addFlash()` crée un message temporaire affiché une seule fois après la redirection.

### 1.4 Afficher le formulaire dans Twig

Créez le template `templates/articles/nouveau.html.twig` :

```twig
{% extends 'base.html.twig' %}

{% block title %}Nouvel article{% endblock %}

{% block body %}
    <div class="row justify-content-center">
        <div class="col-md-8">
            <h1 class="mb-4">📝 Créer un nouvel article</h1>

            {{ form_start(formulaire) }}
                <div class="mb-3">
                    {{ form_row(formulaire.titre) }}
                </div>
                <div class="mb-3">
                    {{ form_row(formulaire.contenu) }}
                </div>
                <div class="mb-3">
                    {{ form_row(formulaire.auteur) }}
                </div>
                <div class="mb-3">
                    {{ form_row(formulaire.dateCreation) }}
                </div>
                <div class="mb-3">
                    {{ form_row(formulaire.publie) }}
                </div>
                {{ form_row(formulaire.enregistrer) }}
            {{ form_end(formulaire) }}

            <a href="{{ path('app_articles') }}" class="btn btn-link mt-3 text-muted">
                ← Retour à la liste
            </a>
        </div>
    </div>
{% endblock %}
```

### 1.5 Intégration de Bootstrap 5

Nous allons utiliser **Bootstrap 5** pour mettre en forme notre application. Mettez à jour `templates/base.html.twig` pour inclure le framework et l'affichage des messages flash :

```twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Symfony TP2{% endblock %}</title>
        <!-- Bootstrap 5 CSS -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
        {% block stylesheets %}{% endblock %}
    </head>
    <body class="bg-light">
        <nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
            <div class="container">
                <a class="navbar-brand" href="{{ path('app_accueil') }}">Symfony TP2</a>
                <div class="navbar-nav">
                    <a class="nav-link" href="{{ path('app_accueil') }}">🏠 Accueil</a>
                    <a class="nav-link" href="{{ path('app_articles') }}">📰 Articles</a>
                    <a class="nav-link" href="{{ path('app_categories') }}">📂 Catégories</a>
                </div>
            </div>
        </nav>

        <div class="container bg-white p-4 shadow-sm rounded">
            {# Messages flash (Alertes Bootstrap) #}
            {% for label, messages in app.flashes %}
                {% for message in messages %}
                    <div class="alert alert-{{ label == 'danger' ? 'danger' : (label == 'success' ? 'success' : 'warning') }} alert-dismissible fade show" role="alert">
                        {{ message }}
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                    </div>
                {% endfor %}
            {% endfor %}

            {% block body %}{% endblock %}
        </div>

        <footer class="text-center py-4 text-muted mt-5">
            &copy; {{ "now"|date("Y") }} — TP Symfony 7.4
        </footer>

        <!-- Bootstrap 5 JS Bundle -->
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
        {% block javascripts %}{% endblock %}
    </body>
</html>
```

Testez le formulaire sur **http://localhost:8000/articles/nouveau**. Remplissez et soumettez : l'article doit être créé en base et un message de confirmation doit s'afficher.

#### ✏️ Question 2
> Expliquez le cycle de vie d'un formulaire Symfony : que se passe-t-il entre l'affichage du formulaire vide et l'enregistrement en base de données ?

### 1.6 Les fonctions Twig pour les formulaires

Voici un récapitulatif des fonctions Twig disponibles pour le rendu des formulaires :

| Fonction | Rôle |
|----------|------|
| `form_start(form)` | Balise `<form>` d'ouverture |
| `form_end(form)` | Balise `</form>` + champs restants |
| `form_row(form.champ)` | Label + champ + erreurs |
| `form_label(form.champ)` | Label seul |
| `form_widget(form.champ)` | Champ seul (input/textarea/select) |
| `form_errors(form.champ)` | Erreurs de validation du champ |
| `form(form)` | Rendu complet du formulaire en une ligne |

#### ✏️ Question 3
> Quelle est la différence entre `form_row()` et l'utilisation séparée de `form_label()` + `form_widget()` ? Dans quel cas préférer l'un ou l'autre ?

---

## Partie 2 — Validation des données (30 min)

> **📝 Note Git** : Nous restons sur la branche `feature-forms-validation` car les formulaires et la validation sont étroitement liés.

### 2.1 Ajouter des contraintes de validation

Symfony utilise des **attributs PHP** pour définir les contraintes de validation directement sur l'entité.

Modifiez `src/Entity/Article.php` pour ajouter des contraintes :

```php
use Symfony\Component\Validator\Constraints as Assert;

class Article
{
    // ... autres propriétés

    #[ORM\Column(length: 255)]
    #[Assert\NotBlank(message: 'Le titre ne peut pas être vide.')]
    #[Assert\Length(
        min: 5,
        max: 255,
        minMessage: 'Le titre doit contenir au moins {{ limit }} caractères.',
        maxMessage: 'Le titre ne peut pas dépasser {{ limit }} caractères.'
    )]
    private ?string $titre = null;

    #[ORM\Column(type: Types::TEXT)]
    #[Assert\NotBlank(message: 'Le contenu ne peut pas être vide.')]
    #[Assert\Length(
        min: 20,
        minMessage: 'Le contenu doit contenir au moins {{ limit }} caractères.'
    )]
    private ?string $contenu = null;

    #[ORM\Column(length: 100)]
    #[Assert\NotBlank(message: 'L\'auteur est obligatoire.')]
    #[Assert\Length(
        min: 2,
        max: 100,
        minMessage: 'Le nom de l\'auteur doit contenir au moins {{ limit }} caractères.'
    )]
    private ?string $auteur = null;

    #[ORM\Column(type: Types::DATETIME_MUTABLE)]
    #[Assert\NotNull(message: 'La date de création est obligatoire.')]
    private ?\DateTimeInterface $dateCreation = null;

    // ... reste du code
}
```

### 2.2 Tester la validation

Accédez à **http://localhost:8000/articles/nouveau** et essayez de soumettre :
- Un formulaire vide → les messages d'erreur s'affichent
- Un titre trop court (moins de 5 caractères)
- Un contenu trop court (moins de 20 caractères)

> **💡 Note** : Grâce à `$form->isValid()` dans le contrôleur, le formulaire n'est traité que si **toutes** les contraintes sont respectées. Les erreurs sont automatiquement affichées à côté des champs concernés.

### 2.3 Les contraintes de validation courantes

| Contrainte | Rôle | Exemple |
|------------|------|---------|
| `NotBlank` | Champ non vide | Titre obligatoire |
| `NotNull` | Valeur non nulle | Date obligatoire |
| `Length` | Longueur min/max | Titre entre 5 et 255 |
| `Email` | Format email valide | Contact |
| `Range` | Valeur numérique min/max | Prix entre 0 et 10000 |
| `Regex` | Expression régulière | Code postal |
| `Type` | Type de donnée | Nombre entier |
| `Url` | URL valide | Lien externe |
| `Choice` | Valeur parmi une liste | Statut |

#### Exercice 2.1 — Validation personnalisée

Ajoutez les contraintes suivantes sur l'entité `Article` :
- Le champ `auteur` ne doit contenir que des **lettres et espaces** (utilisez `Assert\Regex`)

```php
#[Assert\Regex(
    pattern: '/^[a-zA-ZÀ-ÿ\s\-]+$/',
    message: 'Le nom de l\'auteur ne peut contenir que des lettres, espaces et tirets.'
)]
```

Testez en saisissant un auteur avec des chiffres ou des caractères spéciaux.

#### ✏️ Question 4
> Quelle est la différence entre la validation **côté client** (HTML5, JavaScript) et la validation **côté serveur** (Symfony Validator) ? Pourquoi est-il important de toujours valider côté serveur ?

### 🔀 Workflow Git : Committer et pousser les formulaires et la validation

```bash
git add .
git commit -m "feat: Formulaires Symfony avec ArticleType, validation et messages flash"
git push origin feature-forms-validation
```

**Créez une Pull Request sur GitHub :**
1. Allez sur votre dépôt GitHub
2. Cliquez sur **"Compare & pull request"**
3. Sélectionnez `base: main` ← `compare: feature-forms-validation`
4. Titre : `feat: Formulaires et validation des données`
5. Fusionnez la PR (**"Merge pull request"** → **"Confirm merge"**)
6. (Optionnel) Supprimez la branche distante

---

## Partie 3 — Relations Doctrine (45 min)

### 🔀 Workflow Git : Synchroniser et créer une branche pour les relations

```bash
git checkout main
git pull origin main
git checkout -b feature-categories-relations
```

### 3.1 Concept des relations

En base de données relationnelle, les entités peuvent être liées entre elles. Les relations les plus courantes sont :

| Relation | Description | Exemple |
|----------|-------------|---------|
| **ManyToOne** | Plusieurs entités liées à une seule | Plusieurs articles → une catégorie |
| **OneToMany** | Une entité liée à plusieurs | Une catégorie → plusieurs articles |
| **ManyToMany** | Plusieurs à plusieurs | Articles ↔ Tags |
| **OneToOne** | Une à une | Utilisateur ↔ Profil |

Dans ce TP, nous allons créer une relation **ManyToOne / OneToMany** entre `Article` et une nouvelle entité `Categorie`.

### 3.2 Créer l'entité Categorie

```bash
php bin/console make:entity Categorie
```

Ajoutez les propriétés :

| Propriété | Type | Nullable |
|-----------|------|----------|
| `nom` | `string` (100) | non |
| `description` | `text` | oui |

### 3.3 Créer la relation

Utilisez à nouveau `make:entity` pour ajouter la relation **sur l'entité Article** :

```bash
php bin/console make:entity Article
```

Quand l'assistant vous demande un nouveau champ, ajoutez :

```
Nom du champ : categorie
Type : ManyToOne
Classe liée : Categorie
Nullable : oui (pour l'instant)
Souhaitez-vous ajouter une propriété dans Categorie (OneToMany) ? oui
Nom de la propriété dans Categorie : articles
```

Vérifiez les fichiers générés :

**Dans `src/Entity/Article.php`** — un nouveau champ apparaît :
```php
#[ORM\ManyToOne(inversedBy: 'articles')]
private ?Categorie $categorie = null;
```

**Dans `src/Entity/Categorie.php`** — une collection est ajoutée :
```php
#[ORM\OneToMany(targetEntity: Article::class, mappedBy: 'categorie')]
private Collection $articles;
```

### 3.4 Migrer la base de données

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

#### ✏️ Question 5
> Ouvrez le fichier de migration généré. Quelle modification SQL a été ajoutée pour créer la relation ? Qu'est-ce qu'une **clé étrangère** ?

### 3.5 Créer le CRUD pour les catégories

#### Exercice 3.1 — Contrôleur CategorieController

Créez le contrôleur :

```bash
php bin/console make:controller CategorieController
```

Modifiez-le pour implémenter la liste et la création de catégories :

```php
<?php

namespace App\Controller;

use App\Entity\Categorie;
use App\Repository\CategorieRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class CategorieController extends AbstractController
{
    #[Route('/categories', name: 'app_categories')]
    public function index(CategorieRepository $categorieRepository): Response
    {
        $categories = $categorieRepository->findAll();

        return $this->render('categorie/index.html.twig', [
            'categories' => $categories,
        ]);
    }

    #[Route('/categories/nouvelle', name: 'app_categorie_nouvelle')]
    public function nouvelle(Request $request, EntityManagerInterface $em): Response
    {
        $categorie = new Categorie();

        $form = $this->createFormBuilder($categorie)
            ->add('nom', TextType::class, [
                'label' => 'Nom de la catégorie',
                'attr' => [
                    'placeholder' => 'Ex: Technologie, Sport...',
                    'class' => 'form-control',
                ],
            ])
            ->add('description', TextareaType::class, [
                'label' => 'Description',
                'required' => false,
                'attr' => [
                    'rows' => 4,
                    'class' => 'form-control',
                ],
            ])
            ->add('enregistrer', SubmitType::class, [
                'label' => '💾 Créer la catégorie',
                'attr' => ['class' => 'btn btn-primary w-100 mt-3'],
            ])
            ->getForm();

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $em->persist($categorie);
            $em->flush();

            $this->addFlash('success', 'Catégorie "' . $categorie->getNom() . '" créée !');
            return $this->redirectToRoute('app_categories');
        }

        return $this->render('categorie/nouvelle.html.twig', [
            'formulaire' => $form,
        ]);
    }
}
```

> **💡 Note** : Ici, nous utilisons `createFormBuilder()` directement dans le contrôleur au lieu d'un FormType séparé. C'est acceptable pour les formulaires simples, mais **pour les entités complexes, privilégiez un FormType dédié** comme nous l'avons fait pour `Article`.

#### ✏️ Question 6
> Quels sont les avantages et inconvénients de `createFormBuilder()` par rapport à un FormType séparé (`make:form`) ?

#### Exercice 3.2 — Templates des catégories

Créez `templates/categorie/index.html.twig` :

```twig
{% extends 'base.html.twig' %}

{% block title %}Catégories{% endblock %}

{% block body %}
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h1>📂 Catégories</h1>
        <a href="{{ path('app_categorie_nouvelle') }}" class="btn btn-primary">
            ➕ Nouvelle catégorie
        </a>
    </div>

    {% if categories is empty %}
        <div class="alert alert-info text-center py-5">
            <p class="mb-0">Aucune catégorie pour le moment.</p>
        </div>
    {% else %}
        <div class="row g-4">
            {% for categorie in categories %}
                <div class="col-md-4">
                    <div class="card h-100 shadow-sm border-0 border-start border-primary border-4">
                        <div class="card-body">
                            <h3 class="card-title h5 mb-3">{{ categorie.nom }}</h3>
                            {% if categorie.description %}
                                <p class="card-text text-muted small">{{ categorie.description }}</p>
                            {% endif %}
                        </div>
                        <div class="card-footer bg-transparent border-0 pt-0">
                            <span class="badge bg-secondary rounded-pill">
                                📰 {{ categorie.articles|length }} article(s)
                            </span>
                        </div>
                    </div>
                </div>
            {% endfor %}
        </div>
    {% endif %}
{% endblock %}
```

Créez `templates/categorie/nouvelle.html.twig` :

```twig
{% extends 'base.html.twig' %}

{% block title %}Nouvelle catégorie{% endblock %}

{% block body %}
    <div class="row justify-content-center">
        <div class="col-md-8">
            <h1 class="mb-4">➕ Créer une catégorie</h1>

            {{ form_start(formulaire) }}
                <div class="mb-3">{{ form_row(formulaire.nom) }}</div>
                <div class="mb-3">{{ form_row(formulaire.description) }}</div>
                {{ form_row(formulaire.enregistrer) }}
            {{ form_end(formulaire) }}

            <a href="{{ path('app_categories') }}" class="btn btn-link mt-3 text-muted">
                ← Retour aux catégories
            </a>
        </div>
    </div>
{% endblock %}
```

### 3.6 Intégrer la catégorie dans le formulaire Article

Modifiez `src/Form/ArticleType.php` pour ajouter un sélecteur de catégorie :

```php
use App\Entity\Categorie;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;

// Dans buildForm(), ajoutez ce champ :
->add('categorie', EntityType::class, [
    'class' => Categorie::class,
    'choice_label' => 'nom',
    'label' => 'Catégorie',
    'placeholder' => '-- Choisir une catégorie --',
    'required' => false,
    'attr' => ['class' => 'form-control'],
])
```

> **💡 Explication** : `EntityType` génère un `<select>` rempli automatiquement avec les entités `Categorie` de la base de données. Le `choice_label` définit quel champ afficher comme texte des options.

Testez : créez d'abord quelques catégories, puis créez un article en lui attribuant une catégorie.

### 3.7 Mettre à jour la navigation

Dans `templates/base.html.twig`, ajoutez le lien vers les catégories dans la balise `<nav>` :

```twig
<nav>
    <a href="{{ path('app_accueil') }}">🏠 Accueil</a>
    <a href="{{ path('app_articles') }}">📰 Articles</a>
    <a href="{{ path('app_categories') }}">📂 Catégories</a>
</nav>
```

### 🔀 Workflow Git : Committer et pousser les relations et catégories

```bash
git add .
git commit -m "feat: Entité Categorie, relation ManyToOne, CRUD catégories et EntityType"
git push origin feature-categories-relations
```

**Créez une Pull Request sur GitHub :**
1. Titre : `feat: Relations Doctrine et gestion des catégories`
2. `base: main` ← `compare: feature-categories-relations`
3. Fusionnez la PR et supprimez la branche

---

## Partie 4 — CRUD complet : Modification et Suppression (35 min)

### 🔀 Workflow Git : Synchroniser et créer une branche pour le CRUD

```bash
git checkout main
git pull origin main
git checkout -b feature-crud-articles
```

### 4.1 Modifier un article

Ajoutez la méthode de modification dans `ArticlesController` :

```php
#[Route('/articles/{id}/modifier', name: 'app_article_modifier', requirements: ['id' => '\d+'])]
public function modifier(Article $article, Request $request, EntityManagerInterface $em): Response
{
    $form = $this->createForm(ArticleType::class, $article);
    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
        $em->flush(); // Pas besoin de persist() car l'entité est déjà gérée par Doctrine

        $this->addFlash('success', 'Article modifié avec succès !');
        return $this->redirectToRoute('app_article_detail', ['id' => $article->getId()]);
    }

    return $this->render('articles/modifier.html.twig', [
        'formulaire' => $form,
        'article' => $article,
    ]);
}
```

> **💡 Point important** : Lors de la modification, l'entité est déjà **gérée** (managed) par Doctrine. Il n'est donc pas nécessaire d'appeler `persist()`. Un simple `flush()` suffit pour enregistrer les changements.

#### ✏️ Question 7
> Pourquoi ne faut-il pas appeler `persist()` lors de la modification d'une entité existante ? Quel concept Doctrine explique ce comportement ?

Créez `templates/articles/modifier.html.twig` :

```twig
{% extends 'base.html.twig' %}

{% block title %}Modifier : {{ article.titre }}{% endblock %}

{% block body %}
    <div class="row justify-content-center">
        <div class="col-md-8">
            <h1 class="mb-4">✏️ Modifier l'article</h1>
            <p class="text-muted mb-4 pb-2 border-bottom">Modification de « {{ article.titre }} »</p>

            {{ form_start(formulaire) }}
                <div class="mb-3">{{ form_row(formulaire.titre) }}</div>
                <div class="mb-3">{{ form_row(formulaire.contenu) }}</div>
                <div class="mb-3">{{ form_row(formulaire.auteur) }}</div>
                <div class="mb-3">{{ form_row(formulaire.categorie) }}</div>
                <div class="mb-3">{{ form_row(formulaire.dateCreation) }}</div>
                <div class="mb-3">{{ form_row(formulaire.publie) }}</div>
                {{ form_row(formulaire.enregistrer) }}
            {{ form_end(formulaire) }}

            <a href="{{ path('app_article_detail', {id: article.id}) }}" class="btn btn-link mt-3 text-muted">
                ← Annuler
            </a>
        </div>
    </div>
{% endblock %}
```

### 4.2 Supprimer un article

Ajoutez la méthode de suppression dans `ArticlesController` :

```php
#[Route('/articles/{id}/supprimer', name: 'app_article_supprimer', requirements: ['id' => '\d+'], methods: ['POST'])]
public function supprimer(Article $article, Request $request, EntityManagerInterface $em): Response
{
    // Vérification du token CSRF pour la sécurité
    if ($this->isCsrfTokenValid('supprimer_' . $article->getId(), $request->request->get('_token'))) {
        $em->remove($article);
        $em->flush();

        $this->addFlash('success', 'Article supprimé avec succès.');
    } else {
        $this->addFlash('danger', 'Token CSRF invalide. Suppression annulée.');
    }

    return $this->redirectToRoute('app_articles');
}
```

> **⚠️ Sécurité** : La suppression utilise :
> - La méthode **POST** (jamais GET pour une action destructive)
> - Un **token CSRF** pour protéger contre les attaques Cross-Site Request Forgery

#### ✏️ Question 8
> Qu'est-ce qu'une attaque CSRF ? Pourquoi est-il dangereux d'utiliser une simple requête GET pour supprimer une ressource ?

### 4.3 Ajouter les boutons d'action

Modifiez `templates/articles/detail.html.twig` pour ajouter les boutons modifier et supprimer :

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
        {% if article.categorie %}
            <span class="badge bg-info text-dark ms-2">📂 {{ article.categorie.nom }}</span>
        {% endif %}
    </p>

    <div class="lead mb-5 border-bottom pb-4">
        {{ article.contenu }}
    </div>

    {# Boutons d'action #}
    <div class="d-flex gap-2 align-items-center">
        <a href="{{ path('app_article_modifier', {id: article.id}) }}" class="btn btn-primary">
            ✏️ Modifier
        </a>
        
        <form method="post" action="{{ path('app_article_supprimer', {id: article.id}) }}"
              onsubmit="return confirm('Êtes-vous sûr de vouloir supprimer cet article ?');" class="d-inline">
            <input type="hidden" name="_token" value="{{ csrf_token('supprimer_' ~ article.id) }}">
            <button type="submit" class="btn btn-danger">🗑️ Supprimer</button>
        </form>

        <a href="{{ path('app_articles') }}" class="btn btn-outline-secondary">
            ← Retour à la liste
        </a>
    </div>
{% endblock %}
```

Ajoutez également un bouton modifier dans la liste des articles (`templates/articles/index.html.twig`), dans chaque ligne du tableau :

```twig
<td>
    <a href="{{ path('app_article_detail', {id: article.id}) }}" class="btn btn-outline-primary btn-sm">
        👁️ Voir
    </a>
</td>
```

### 4.4 Tester le CRUD complet

Vérifiez que toutes les opérations fonctionnent :
1. **Create** : Créer un article avec le formulaire → `/articles/nouveau`
2. **Read** : Lister et afficher les articles → `/articles` et `/articles/{id}`
3. **Update** : Modifier un article → `/articles/{id}/modifier`
4. **Delete** : Supprimer un article avec confirmation → bouton supprimer

### 🔀 Workflow Git : Committer et pousser le CRUD complet

```bash
git add .
git commit -m "feat: CRUD complet articles avec modification, suppression et protection CSRF"
git push origin feature-crud-articles
```

**Créez une Pull Request sur GitHub :**
1. Titre : `feat: CRUD complet avec modification et suppression`
2. `base: main` ← `compare: feature-crud-articles`
3. Fusionnez la PR, puis synchronisez en local :

```bash
git checkout main
git pull origin main
```

---

## Partie 5 — Exercice de synthèse (20 min)

### 🧩 Mini-projet : CRUD complet pour les catégories

En vous basant sur tout ce que vous avez appris, complétez le CRUD des catégories **de manière autonome** :

1. **Créer un `CategorieType`** avec `make:form` pour le formulaire des catégories

2. **Ajouter des contraintes de validation** sur l'entité `Categorie` :
   - Le nom est obligatoire, entre 2 et 100 caractères
   - La description, si renseignée, doit faire au moins 10 caractères

3. **Ajouter les routes suivantes dans `CategorieController`** :
   - `/categories/{id}` → Affiche la catégorie et **la liste de ses articles**
   - `/categories/{id}/modifier` → Formulaire de modification
   - `/categories/{id}/supprimer` → Suppression avec protection CSRF

4. **Créer les templates Twig correspondants**

5. **Workflow Git** : Développez tout sur une branche `feature-crud-categories`, puis créez une PR pour fusionner dans `main`

#### 🏆 Bonus
- Dans la page de détail d'une catégorie, affichez un compteur d'articles et un lien direct pour créer un article dans cette catégorie
- Empêchez la suppression d'une catégorie qui contient encore des articles (affichez un message d'erreur)
- Ajoutez un filtre par catégorie sur la page de liste des articles (via un paramètre GET `?categorie=ID`)

---

## 📚 Ressources utiles

| Ressource | Lien |
|-----------|------|
| Documentation Forms | https://symfony.com/doc/current/forms.html |
| Types de champs | https://symfony.com/doc/current/reference/forms/types.html |
| Validation | https://symfony.com/doc/current/validation.html |
| Contraintes disponibles | https://symfony.com/doc/current/reference/constraints.html |
| Relations Doctrine | https://symfony.com/doc/current/doctrine/associations.html |
| Protection CSRF | https://symfony.com/doc/current/security/csrf.html |

---

## 📝 Récapitulatif des commandes

```bash
# Créer un formulaire
php bin/console make:form NomType NomEntite

# Créer/modifier une entité (ajouter des champs ou relations)
php bin/console make:entity NomEntite

# Générer et appliquer une migration
php bin/console make:migration
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

---

## ✅ Critères d'évaluation

| Critère | Points |
|---------|--------|
| Formulaire ArticleType fonctionnel avec les bons types de champs | /3 |
| Validation correcte sur l'entité Article (contraintes + messages) | /3 |
| Relation ManyToOne Categorie ↔ Article fonctionnelle | /3 |
| CRUD Articles complet (Créer, Lire, Modifier, Supprimer) | /4 |
| Messages flash et protection CSRF | /2 |
| Exercice de synthèse (CRUD Catégories) | /3 |
| Réponses aux questions | /2 |
| **Total** | **/20** |

---

> **📌 Rendu** : À la fin de la séance, fournissez le **lien vers votre dépôt GitHub** sur la plateforme de dépôt prévue. Assurez-vous que toutes vos branches ont été fusionnées dans `main`. Incluez un fichier `REPONSES.md` contenant vos réponses aux questions.
