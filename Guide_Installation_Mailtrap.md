# 📧 Guide — Installer et Utiliser Mailtrap avec Symfony

**Module** : Développement Web — Framework PHP  
**Prérequis** : Avoir un projet Symfony fonctionnel avec `symfony/mailer` installé

---

## 🤔 Qu'est-ce que Mailtrap ?

**Mailtrap** est un service en ligne qui permet de **tester l'envoi d'emails** sans réellement les envoyer aux destinataires. Tous les emails envoyés depuis votre application sont **interceptés** et affichés dans une boîte de réception virtuelle.

> ⚠️ **Pourquoi en a-t-on besoin ?**  
> Pendant le développement, on ne veut pas envoyer de vrais emails aux utilisateurs.  
> Mailtrap agit comme un « piège à emails » : il capture tout ce que votre application envoie pour que vous puissiez le vérifier sans risque.

---

## 📝 Étape 1 — Créer un compte Mailtrap

1. Rendez-vous sur **[https://mailtrap.io](https://mailtrap.io)**
2. Cliquez sur **Sign Up** (inscription gratuite)
3. Vous pouvez vous inscrire avec :
   - Votre **adresse email**
   - Votre compte **Google**
   - Votre compte **GitHub**
4. **Confirmez votre adresse email** en cliquant sur le lien reçu dans votre boîte de réception

> 💡 Le plan gratuit est largement suffisant pour les besoins du TP.

---

## 📬 Étape 2 — Accéder à l'Email Sandbox (boîte de test)

Une fois connecté au tableau de bord Mailtrap :

1. Dans le menu de gauche, cliquez sur **Email Testing** → **Inboxes**
2. Vous verrez une boîte de réception par défaut appelée **My Inbox**
3. Cliquez sur **My Inbox** pour l'ouvrir

C'est ici que tous vos emails de test apparaîtront.

---

## 🔑 Étape 3 — Récupérer les identifiants SMTP

Toujours dans **My Inbox** :

1. Cliquez sur l'onglet **SMTP Settings** (ou **Show Credentials**)
2. Vous trouverez les informations suivantes :

| Paramètre     | Valeur                          |
|----------------|--------------------------------|
| **Host**       | `sandbox.smtp.mailtrap.io`      |
| **Port**       | `2525` (ou `587`)              |
| **Username**   | `votre_username` (ex: `a1b2c3d4e5f6g7`) |
| **Password**   | `votre_password` (ex: `h8i9j0k1l2m3n4`) |

> 📌 **Gardez cette page ouverte**, vous en aurez besoin pour l'étape suivante.

### 🖼️ Repérer les identifiants dans l'interface

Dans l'interface Mailtrap, cherchez la section qui ressemble à ceci :

```
Host:     sandbox.smtp.mailtrap.io
Port:     25, 465, 587 or 2525
Username: xxxxxxxxxxxxxxx
Password: xxxxxxxxxxxxxxx
```

---

## ⚙️ Étape 4 — Installer le composant Mailer dans Symfony

Si ce n'est pas déjà fait, installez le composant **Mailer** de Symfony :

```bash
composer require symfony/mailer
```

---

## 🔧 Étape 5 — Configurer Symfony avec Mailtrap

### 5.1 Modifier le fichier `.env`

Ouvrez le fichier `.env` (ou `.env.local`) à la racine de votre projet Symfony.

Cherchez la ligne `MAILER_DSN` et remplacez-la par :

```env
MAILER_DSN=smtp://VOTRE_USERNAME:VOTRE_PASSWORD@sandbox.smtp.mailtrap.io:2525
```

### 5.2 Exemple concret

Si vos identifiants Mailtrap sont :
- **Username** : `a1b2c3d4e5f6g7`
- **Password** : `h8i9j0k1l2m3n4`

Alors votre ligne sera :

```env
MAILER_DSN=smtp://a1b2c3d4e5f6g7:h8i9j0k1l2m3n4@sandbox.smtp.mailtrap.io:2525
```

> ⚠️ **Important** : N'ajoutez **pas d'espaces** autour du `=` et ne mettez **pas de guillemets** autour de la valeur.

---

## 📤 Étape 6 — Envoyer un email de test

### 6.1 Créer une action dans un contrôleur

Ajoutez cette méthode dans un contrôleur existant (ou créez-en un nouveau) :

```php
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Email;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

#[Route('/test-email', name: 'app_test_email')]
public function testEmail(MailerInterface $mailer): Response
{
    $email = (new Email())
        ->from('noreply@monsite.com')
        ->to('etudiant@exemple.com')
        ->subject('🎉 Test Email depuis Symfony !')
        ->text('Ceci est un email de test envoyé depuis Symfony avec Mailtrap.')
        ->html('<h1>Bravo !</h1><p>Votre configuration Mailtrap fonctionne correctement. 🚀</p>');

    $mailer->send($email);

    $this->addFlash('success', 'Email envoyé avec succès ! Vérifiez votre boîte Mailtrap.');

    return $this->redirectToRoute('app_article_index');
}
```

### 6.2 Tester

1. Lancez votre serveur Symfony :
   ```bash
   symfony server:start
   ```
2. Ouvrez votre navigateur et accédez à : `http://localhost:8000/test-email`
3. Retournez sur **Mailtrap** → **My Inbox**
4. ✅ Vous devriez voir votre email de test apparaître !

---

## 📧 Étape 7 — Envoyer un email HTML enrichi (avec Twig)

Pour des emails plus élaborés, utilisez les **templates Twig** :

### 7.1 Créer le template

Créez le fichier `templates/emails/notification.html.twig` :

```twig
{# templates/emails/notification.html.twig #}

<div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px;">
    <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 30px; border-radius: 10px 10px 0 0; text-align: center;">
        <h1 style="color: white; margin: 0;">{{ subject }}</h1>
    </div>
    
    <div style="background: #f9f9f9; padding: 30px; border-radius: 0 0 10px 10px;">
        <p>Bonjour,</p>
        <p>{{ message }}</p>
        
        {% if article is defined %}
        <div style="background: white; padding: 15px; border-radius: 8px; border-left: 4px solid #667eea; margin: 15px 0;">
            <strong>{{ article.titre }}</strong><br>
            <small style="color: #666;">Catégorie : {{ article.categorie.nom }}</small>
        </div>
        {% endif %}
        
        <p style="color: #999; font-size: 12px; margin-top: 30px;">
            Cet email a été envoyé automatiquement depuis notre application Symfony.
        </p>
    </div>
</div>
```

### 7.2 Envoyer avec le template

```php
use Symfony\Bridge\Twig\Mime\TemplatedEmail;

#[Route('/test-email-twig', name: 'app_test_email_twig')]
public function testEmailTwig(MailerInterface $mailer): Response
{
    $email = (new TemplatedEmail())
        ->from('noreply@monsite.com')
        ->to('etudiant@exemple.com')
        ->subject('Nouvel article publié !')
        ->htmlTemplate('emails/notification.html.twig')
        ->context([
            'subject' => 'Nouvel article publié !',
            'message' => 'Un nouveau contenu vient d\'être ajouté sur le blog.',
        ]);

    $mailer->send($email);

    $this->addFlash('success', 'Email Twig envoyé ! Vérifiez Mailtrap.');

    return $this->redirectToRoute('app_article_index');
}
```

---

## 🔍 Étape 8 — Vérifier les emails dans Mailtrap

Dans l'interface Mailtrap, pour chaque email reçu vous pouvez :

| Onglet         | Description                                      |
|----------------|--------------------------------------------------|
| **HTML**       | Voir le rendu HTML de l'email                    |
| **Text**       | Voir la version texte brut                       |
| **Raw**        | Voir le code source complet de l'email           |
| **HTML Check** | Vérifier la compatibilité avec les clients email |
| **Spam Analysis** | Vérifier le score anti-spam de votre email    |

---

## 🐛 Résolution de problèmes courants

### ❌ Erreur : `Connection could not be established with host`

**Cause** : Les identifiants SMTP sont incorrects ou le port est bloqué.

**Solution** :
1. Vérifiez que le `Username` et `Password` dans `.env` correspondent bien à ceux de Mailtrap
2. Essayez de changer le port (`587` au lieu de `2525`)
3. Vérifiez votre connexion internet

### ❌ Erreur : `Expected response code "250" but got code "550"`

**Cause** : L'adresse d'expéditeur (`from`) n'est pas acceptée.

**Solution** : Vérifiez que l'adresse `from` est bien formatée (ex: `noreply@monsite.com`)

### ❌ L'email n'apparaît pas dans Mailtrap

**Solutions** :
1. Videz le cache Symfony : `php bin/console cache:clear`
2. Vérifiez que vous utilisez bien `.env.local` et non `.env` si vous avez les deux fichiers
3. Vérifiez que le `MAILER_DSN` n'est pas écrasé dans un autre fichier `.env.*`

### ❌ Erreur : `Transport "smtp" not found`

**Solution** :
```bash
composer require symfony/mailer
```

---

## 📝 Récapitulatif

```bash
# 1. Installer le composant Mailer
composer require symfony/mailer

# 2. Configurer .env avec les identifiants Mailtrap
# MAILER_DSN=smtp://USERNAME:PASSWORD@sandbox.smtp.mailtrap.io:2525

# 3. Vider le cache après modification du .env
php bin/console cache:clear

# 4. Tester l'envoi
# Accéder à la route /test-email dans le navigateur

# 5. Vérifier dans Mailtrap → My Inbox
```


