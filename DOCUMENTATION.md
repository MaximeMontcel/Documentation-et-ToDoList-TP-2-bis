# Documentation technique — IT Config

Site vitrine d'une entreprise de matériel informatique, avec un panel d'administration en PHP natif.
Ce document décrit **uniquement le code présent dans le projet**. Chaque affirmation cite le fichier qui la justifie.

> Projet réalisé en BTS SIO 1re année. Le site fonctionne **uniquement en local sous WAMP** (Windows).

## Table des matières

1. [Présentation générale](#1-présentation-générale)
2. [Stack technique](#2-stack-technique)
3. [Arborescence du projet](#3-arborescence-du-projet)
4. [Points d'entrée et routage](#4-points-dentrée-et-routage)
5. [Stockage des données](#5-stockage-des-données)
6. [Authentification, sessions et sécurité](#6-authentification-sessions-et-sécurité)
7. [Front-end](#7-front-end)
8. [Intégrations externes](#8-intégrations-externes)
9. [Configuration](#9-configuration)
10. [Tests](#10-tests)
11. [Points forts](#11-points-forts)
12. [Limites connues et risques](#12-limites-connues-et-risques)

---

## 1. Présentation générale

Le projet se compose de deux parties.

**Le site vitrine (public)** présente l'entreprise fictive « IT Config ».
- `index.html` : accueil, promotions, avantages, chiffres clés, produits phares.
- `produits.html` : catalogue en 4 catégories (ordinateurs, périphériques, écrans, réseau).
- `contact.html` : coordonnées et formulaire de contact.
- `contact.php` : reçoit le formulaire de contact et enregistre le message.

**Le panel d'administration (privé)**, dans `admin/`, réservé à un administrateur connecté.
- Connexion avec protection anti-brute-force (`admin/login.php`).
- « Mot de passe oublié » avec code à 8 chiffres envoyé par e-mail (`admin/forgot.php`).
- Tableau de bord (`admin/index.php`).
- Configuration des e-mails SMTP et test d'envoi (`admin/mails.php`).
- Changement du mot de passe (`admin/password.php`).
- Changement de l'e-mail de connexion (`admin/user.php`).
- Maintenance : diagnostic système et purge de `storage/tmp` (`admin/maintenance.php`).
- Maintenance : journal des connexions et messages de contact (`admin/logs.php`).

Fonction prévue mais **non codée** : « Sauvegarde » (export JSON). L'onglet est affiché désactivé (`views/maintenance_nav.php`).

## 2. Stack technique

| Élément | Détail | Justification |
|---|---|---|
| Serveur web | Apache 2.4 (WAMP, Windows) | ligne « Serveur web » de `app/Maintenance/SystemCheck.php` |
| Langage serveur | PHP 8 natif, sans framework, `declare(strict_types=1)` | tous les fichiers `app/` et `admin/` |
| Version PHP utilisée | 8.3 (minimum vérifié : 8.0) | `SystemCheck::environment()` |
| Front-end | HTML5, CSS3, JavaScript sans bibliothèque | `css/`, `js/`, `admin/assets/` |
| Stockage | Fichiers JSON et JSONL dans `storage/` | `app/Support/FileStore.php` |
| Base de données | **Aucune** | aucun appel PDO/mysqli dans le projet |
| Bibliothèque externe | PHPMailer 7 (3 fichiers, sans Composer) | `vendor/phpmailer/`, `scripts/install_phpmailer.ps1` |
| Extensions PHP requises | openssl, mbstring, json, session | `SystemCheck::environment()` |

Aucun gestionnaire de dépendances n'est utilisé (ni Composer, ni npm). `vendor/autoload.php` est un chargeur minimal écrit par `scripts/install_phpmailer.ps1`.

## 3. Arborescence du projet

```
infoleader/
├── index.html, produits.html, contact.html   Site vitrine (pages statiques)
├── contact.php                  Réception du formulaire de contact
├── css/style.css                Thème du site vitrine
├── js/site.js                   Script commun du site vitrine
├── images/                      Illustrations SVG des produits
├── admin/                       Contrôleurs du panel (pages accessibles)
│   ├── _guard.php               Contrôle de session (inclus par chaque page protégée)
│   ├── login.php, logout.php, forgot.php
│   ├── index.php, mails.php, password.php, user.php
│   ├── maintenance.php, logs.php
│   └── assets/                  CSS et JavaScript du panel
├── app/                         Logique PHP (inaccessible depuis le web)
│   ├── bootstrap.php            Chargement de la configuration et des classes
│   ├── config.php               Constantes de configuration
│   ├── Auth/                    Comptes, connexion, mots de passe, anti-brute-force
│   ├── Mail/                    Envoi des e-mails (Mailer, SmtpSender)
│   ├── MailConfig/              Configuration SMTP (MailSettings)
│   ├── Maintenance/             Diagnostic, purge, lecture des journaux
│   ├── Contact/                 Messages de contact (ContactStore)
│   └── Support/                 Outils communs (session, CSRF, journal, fichiers)
├── views/                       Gabarits HTML du panel (inaccessibles depuis le web)
├── storage/                     Données (inaccessibles depuis le web)
│   ├── logs/                    Journaux
│   └── tmp/                     Fichiers temporaires
├── scripts/                     Scripts en ligne de commande (inaccessibles depuis le web)
├── tests/                       Tests automatiques (inaccessibles depuis le web)
├── vendor/                      PHPMailer (inaccessible depuis le web)
├── verifier-panel.cmd           Lance tous les tests
└── .gitignore                   Exclut storage/ et vendor/ de Git
```

Les dossiers `app/`, `views/`, `storage/`, `storage/tmp/`, `scripts/`, `tests/` et `vendor/` contiennent chacun un `.htaccess` avec `Require all denied`.

## 4. Points d'entrée et routage

Il n'y a **pas de routeur central**. Chaque URL correspond à un fichier réel servi par Apache.

| URL | Fichier | Rôle |
|---|---|---|
| `/infoleader/` | `index.html` | Accueil (statique) |
| `/infoleader/produits.html` | `produits.html` | Catalogue (statique) |
| `/infoleader/contact.html` | `contact.html` | Formulaire (statique) |
| `/infoleader/contact.php` | `contact.php` | Traitement du formulaire (POST uniquement) |
| `/infoleader/admin/` | `admin/index.php` | Tableau de bord (protégé) |
| `/infoleader/admin/login.php` | `admin/login.php` | Connexion |
| `/infoleader/admin/forgot.php` | `admin/forgot.php` | Mot de passe oublié (POST uniquement) |
| `/infoleader/admin/*.php` | autres contrôleurs | Modules du panel (protégés) |

Déroulé type d'une page du panel (exemple : `admin/password.php`) :
1. `require _guard.php` : charge `app/bootstrap.php`, démarre la session, redirige vers `login.php` si personne n'est connecté.
2. En POST : vérification du jeton CSRF, puis traitement par une classe de `app/`.
3. Après un POST réussi : redirection (schéma « POST puis redirection ») pour éviter un double envoi.
4. Affichage : `views/layout_header.php`, puis la vue, puis `views/layout_footer.php`.

## 5. Stockage des données

Le site **n'utilise pas de base de données**. Toutes les données sont des fichiers dans `storage/`.
Toutes les lectures et écritures JSON passent par `app/Support/FileStore.php`.

| Fichier | Format | Contenu | Classe responsable |
|---|---|---|---|
| `storage/accounts.json` | JSON | Compte administrateur : id, email, password_hash, role, created_at, last_login | `app/Auth/AccountRepository.php` |
| `storage/rate_limits.json` | JSON | Échecs de connexion par IP et par e-mail (e-mail haché en SHA-1) | `app/Auth/RateLimiter.php` |
| `storage/password_resets.json` | JSON | Codes « mot de passe oublié » **hachés** (bcrypt), expiration, essais | `app/Auth/PasswordReset.php` |
| `storage/mail_config.json` | JSON | Configuration SMTP, y compris le mot de passe SMTP | `app/MailConfig/MailSettings.php` |
| `storage/contact_messages.jsonl` | JSONL (1 message par ligne) | Messages du formulaire de contact | `app/Contact/ContactStore.php` |
| `storage/logs/AAAA-MM-JJ.jsonl` | JSONL | Journal des événements (date, IP, type, détail) | `app/Support/Logger.php` |
| `storage/logs/mail-outbox.jsonl` | JSONL | Envois simulés (destinataire, sujet, date — jamais le contenu) | `app/Mail/Mailer.php` |
| `storage/logs/mail-debug.log` | Texte | Trace SMTP, seulement si le débogage est activé (identifiants masqués) | `app/Mail/SmtpSender.php` |
| `storage/tmp/` | Fichiers | Fichiers temporaires des écritures atomiques | `app/Support/FileStore.php` |

Fonctionnement de `FileStore` :
- `readJson()` renvoie une valeur par défaut si le fichier est absent ou illisible.
- `writeJson()` écrit d'abord dans `storage/tmp/`, puis renomme le fichier (écriture « atomique »).
- `appendLine()` ajoute une ligne avec un verrou (`LOCK_EX`), pour les journaux.

## 6. Authentification, sessions et sécurité

### Connexion — `admin/login.php`, `app/Auth/Auth.php`
- Mot de passe vérifié avec `password_verify()`, stocké avec `password_hash()` (bcrypt).
- Compte inconnu : un hachage factice est quand même vérifié, pour un temps de réponse comparable.
- Même message « Identifiants incorrects » que l'e-mail ou le mot de passe soit faux.

### Anti-brute-force — `app/Auth/RateLimiter.php`, `app/config.php`
- Délais croissants après chaque échec : 3 s, 10 s, 30 s, 60 s, puis 300 s.
- Blocage de l'IP pendant 4 h au 10e échec.
- Remise à zéro après une connexion réussie, ou après 1 h sans échec.
- Le mot de passe actuel saisi dans les modules « Mot de passe » et « Utilisateur » est soumis aux mêmes règles.

### Sessions — `app/Support/Session.php`
- Cookie `httponly`, `SameSite=Lax`, nom personnalisé.
- `session_regenerate_id(true)` à la connexion, au changement de mot de passe et au changement d'e-mail.
- Déconnexion automatique après 30 minutes d'inactivité.
- Déconnexion en POST avec jeton CSRF (`admin/logout.php`).

### CSRF — `app/Support/Csrf.php`
- Jeton aléatoire de 64 caractères en session, vérifié avec `hash_equals()` sur tous les formulaires du panel.

### Mot de passe oublié — `app/Auth/PasswordReset.php`, `admin/forgot.php`
- Code à 8 chiffres généré avec `random_int()`, stocké haché, valable 5 minutes, 5 essais maximum, à usage unique.
- Même message à l'écran, que le compte existe ou non. L'échec d'envoi de l'e-mail est seulement journalisé, pour ne pas révéler l'existence du compte.

### Politique de mot de passe — `app/Auth/PasswordPolicy.php`
- Au moins 20 caractères, une majuscule, une minuscule, un chiffre et un caractère spécial.
- Ne doit pas contenir la partie de l'e-mail située avant le @. 72 octets au maximum (limite de bcrypt).
- Contrôle JavaScript dans la page (`admin/assets/password.js`) **et** contrôle PHP, qui fait foi (`app/Auth/PasswordChange.php`).

### Protection XSS et des fichiers
- Toutes les données affichées passent par la fonction `e()` (`htmlspecialchars`) — `app/Support/helpers.php`.
- Purge de `storage/tmp` : aucun nom de fichier n'est reçu du navigateur, et `realpath()` vérifie chaque chemin (`app/Maintenance/TmpPurger.php`).

### Journal des événements — `app/Support/Logger.php`
- Événements tracés : connexion réussie ou échouée, blocage d'IP, demande de code, changement de mot de passe ou d'e-mail, envois d'e-mails, purges.
- Les mots de passe et les codes ne sont jamais journalisés.

## 7. Front-end

### Site vitrine
- Pages HTML statiques. L'en-tête et le pied de page sont **recopiés** dans chaque page.
- `css/style.css` : thème « sombre néon », couleurs regroupées dans des variables CSS (`:root`).
- `js/site.js` :
  - menu mobile (bouton « hamburger », touche Échap) ;
  - bouton « retour en haut » ;
  - apparition des cartes au défilement ;
  - mise en avant de la catégorie visible sur la page Produits.
- Accessibilité : lien « Aller au contenu », focus visible, respect de `prefers-reduced-motion`.
- Pas de bascule thème clair / sombre : le thème est sombre uniquement.
- Les produits et les prix sont écrits en dur dans le HTML.

### Panel d'administration
- `admin/assets/admin.css` : même thème sombre néon que le site.
- Scripts JavaScript :

| Script | Rôle |
|---|---|
| `flash.js` | Les messages disparaissent après 6 s. |
| `login.js` | Compte à rebours pendant le délai anti-brute-force. |
| `forgot.js` | Fenêtre « mot de passe oublié ». |
| `mail.js` | Le port suit la sécurité choisie : 465 ou 587. |
| `password.js`, `user.js` | Règles cochées au fil de la saisie. |
| `confirm.js` | Fenêtres de confirmation avant les purges. |

## 8. Intégrations externes

- **Envoi d'e-mails par SMTP** avec PHPMailer (`app/Mail/SmtpSender.php`).
  - Sécurité SSL/TLS sur le port 465, ou STARTTLS sur le port 587.
  - Délai maximal de 15 s (connexion et chaque réponse du serveur).
  - Messages d'erreur clairs en français, classés par cause (certificat, authentification, connexion…).
  - Utilisé par le « mot de passe oublié » (code envoyé à l'adresse du compte) et par le bouton « Tester l'envoi ».
- **Mode simulation** : tant que le SMTP n'est pas configuré, aucun e-mail ne part, et le code s'affiche à l'écran (`app/Mail/Mailer.php`, `MAIL_MODE`).
- La fonction PHP `mail()` est seulement décrite dans le module e-mails. Elle n'est pas utilisée, car aucun serveur de messagerie local n'existe sous WAMP.
- Aucune autre API ni service tiers. Aucun paiement.

## 9. Configuration

| Fichier | Contenu |
|---|---|
| `app/config.php` | Chemins (`STORAGE_PATH`, `LOGS_PATH`…), fuseau `Europe/Paris`, sessions (30 min), anti-brute-force, code de réinitialisation, longueur minimale du mot de passe, délai SMTP |
| `storage/mail_config.json` | Paramètres SMTP saisis dans le module « Configuration des e-mails » (absent tant que rien n'est enregistré) |
| `app/MailConfig/MailSettings.php` | Valeurs par défaut du SMTP : serveur du TP, port 465, SSL/TLS |
| `.gitignore` | Exclut `storage/`, `mail_config.json`, les journaux de débogage et `vendor/` |
| `scripts/create_admin.php` | Création du compte administrateur en ligne de commande |
| `scripts/install_phpmailer.ps1` | Téléchargement de PHPMailer dans `vendor/` |

## 10. Tests

Le projet contient des tests automatiques, lancés par `verifier-panel.cmd`.

| Test | Objet |
|---|---|
| `tests/check_panel.ps1` | Parcours complet par HTTP : connexion, CSRF, blocage, mot de passe oublié, module e-mails, fichiers protégés (403) |
| `tests/check_rate_limiter.php` | Délais et blocage anti-brute-force |
| `tests/check_password_reset.php` | Codes de réinitialisation |
| `tests/check_mail_settings.php` | Configuration SMTP, masquage des secrets, classement des erreurs |
| `tests/check_reset_send.php` | Envoi du code (simulation et SMTP) |
| `tests/check_password_change.php` | Module « Mot de passe » |
| `tests/check_email_change.php` | Module « Utilisateur » |
| `tests/check_maintenance.php` | Diagnostic et purge de `storage/tmp` |
| `tests/check_logs.php` | Journaux et messages de contact |

Chaque test sauvegarde les fichiers de `storage/` qu'il modifie et les restaure à la fin.

## 11. Points forts

- Mots de passe et codes **hachés**, jamais stockés ni journalisés en clair.
- Protection CSRF sur tous les formulaires du panel, sessions régénérées aux moments sensibles.
- Anti-brute-force côté serveur, avec des délais progressifs et un blocage temporaire de l'IP.
- Aucune information sur l'existence d'un compte (mêmes messages, temps de réponse comparable).
- Données et code sensibles hors de portée du navigateur (`.htaccess`, réponse 403 vérifiée par les tests).
- Code organisé : contrôleurs (`admin/`), logique (`app/`), affichage (`views/`).
- Logique testable en ligne de commande, avec environ 250 vérifications automatiques.
- Aucune dépendance installée par un gestionnaire de paquets. PHPMailer est la seule bibliothèque.

## 12. Limites connues et risques

### Sécurité
1. **Mot de passe SMTP en clair** dans `storage/mail_config.json`. C'est nécessaire pour l'envoyer au serveur, mais il n'est ni chiffré ni haché. Le fichier est seulement protégé par `.htaccess` et exclu de Git.
2. **Formulaire de contact sans CSRF ni limite d'envoi** (`contact.php`). Il n'y a qu'un champ piège anti-robot : un robot peut remplir `storage/contact_messages.jsonl`.
3. **Aucun en-tête de sécurité HTTP** : pas de `Content-Security-Policy`, de `X-Frame-Options` ni de `X-Content-Type-Options`. Aucun `.htaccess` à la racine du site.
4. **Messages d'erreur PHP affichés** : `display_errors = On` dans la configuration PHP de WAMP. Une erreur peut révéler des chemins de fichiers.
5. **`session.use_strict_mode = 0`** dans la configuration PHP de WAMP, et cookie de session sans l'option `secure` (normal en HTTP local, à revoir en HTTPS).
6. **La protection des dossiers dépend du `.htaccess`.** Si Apache ignorait ces fichiers (`AllowOverride None`), `storage/` serait lisible.
7. `scripts/create_admin.php` affiche le mot de passe **en clair** pendant la saisie dans la console.

### Stockage des données
8. **Accès concurrents** : `FileStore::writeJson()` est atomique, mais une requête peut lire, puis une autre écrire entre-temps. Une mise à jour peut alors en écraser une autre.
9. **Croissance sans limite** : les journaux (`storage/logs/*.jsonl`), les messages de contact et les entrées anciennes de `rate_limits.json` ne sont jamais nettoyés automatiquement.
10. **La page « Journaux » relit tous les fichiers** à chaque affichage (`LogReader::events()`). Elle n'affiche que les 200 derniers événements.

### Données personnelles (RGPD)
11. Les journaux et les messages de contact contiennent des **adresses e-mail et des adresses IP**, sans durée de conservation définie.
12. Le site ne contient **ni mentions légales, ni politique de confidentialité**, ni information sur l'usage des données du formulaire de contact.

### Qualité du code et fonctionnel
13. **En-tête et pied de page dupliqués** dans `index.html`, `produits.html`, `contact.html` et `contact.php` : chaque modification est à faire 4 fois.
14. **Contenu en dur et daté** : promotions « valables jusqu'au 30 septembre » dans `index.html`, prix et produits écrits dans le HTML, et la carte de la page Contact n'est qu'un emplacement vide.
15. **Fonction « Sauvegarde » non codée** (onglet désactivé).
16. **Un seul compte administrateur**. Le script `create_admin.php` remplace le compte existant.
17. Les fenêtres de confirmation des purges **nécessitent JavaScript** pour s'ouvrir.
18. Les liens CSS et JS portent un numéro de version (`?v=3`) à **augmenter à la main** après chaque modification.

### Tests
19. `tests/check_panel.ps1` demande le **mot de passe administrateur réel**, et change temporairement ce mot de passe pendant le test.
20. Le JavaScript (contrôles de saisie, menu, modales) **n'est pas testé automatiquement**.

### Environnement local
21. L'envoi SMTP dépend du poste : un antivirus qui analyse les e-mails peut intercepter la connexion sécurisée et provoquer une erreur de certificat.
22. Ce fichier et les autres fichiers `.md` placés à la racine sont **lisibles depuis le navigateur** (`http://localhost/infoleader/DOCUMENTATION.md`).
