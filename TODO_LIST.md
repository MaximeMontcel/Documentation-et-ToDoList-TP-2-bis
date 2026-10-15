# Plan d'améliorations — IT Config

Ce plan découle de la section « Limites connues et risques » de [DOCUMENTATION.md](DOCUMENTATION.md) (numéros notés « L1 », « L2 »…).
Il est conçu pour un étudiant de BTS SIO 1re année, sur votre site tel qu'il est aujourd'hui.

## Table des matières

1. [Cadre à respecter](#1-cadre-à-respecter)
2. [Légende](#2-légende)
3. [Actions par thème](#3-actions-par-thème)
4. [Définition de Terminé](#4-définition-de-terminé)
5. [Séquencement par jalons](#5-séquencement-par-jalons)
6. [Petit lexique](#6-petit-lexique)

---

## 1. Cadre à respecter

- **WAMP local uniquement** : Apache + PHP sous Windows, dans `C:\wamp64\www`. Aucun serveur distant, VPS, Docker, Nginx, PHP-FPM, nom de domaine réel ni déploiement automatisé.
- **Aucune nouvelle dépendance** : ni Composer, ni npm, ni framework. Une action qui en demanderait une est marquée « sous accord du professeur ».
- **Aucun paiement réel** : le site ne vend rien en ligne. Si un jour un panier est ajouté, le paiement restera une **simulation** (aucune passerelle bancaire, aucune donnée de carte).
- **Stockage en fichiers JSON** : c'est un choix valable, ce n'est pas un défaut. Une base **SQLite** est proposée seulement en option (action D5).
- **Le site doit fonctionner après chaque action** : vous testez dans le navigateur et lancez `verifier-panel.cmd`.

## 2. Légende

| Priorité | Sens |
|---|---|
| **P0** | Critique : à faire en premier (sécurité ou obligation légale simple) |
| **P1** | Important : améliore nettement la fiabilité ou la maintenance |
| **P2** | Optionnel : confort, évolution, pour aller plus loin |

| Complexité | Repère |
|---|---|
| **simple** | Quelques lignes, 1 ou 2 fichiers, moins d'une heure |
| **moyen** | Une nouvelle fonction ou un nouveau fichier, une séance |
| **complexe** | Plusieurs fichiers et une réflexion d'architecture, plusieurs séances |

---

## 3. Actions par thème

### 3.1 Sécurité

| ID | Action | Prio. | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| S1 | **Bloquer l'accès web aux fichiers sensibles de la racine** (`.md`, `.cmd`, `.gitignore`) avec un `.htaccess` à la racine. Réf. L22. | P0 | simple | `.htaccess` (nouveau, racine) | `http://localhost/infoleader/DOCUMENTATION.md` et `.../verifier-panel.cmd` renvoient **403**. Les pages du site s'affichent toujours. |
| S2 | **Ne plus afficher les erreurs PHP à l'écran** : `display_errors` désactivé par le code, erreurs écrites dans `storage/logs/php-errors.log`. Réf. L4. | P0 | simple | `app/config.php` | Une erreur volontaire (fichier de test) n'affiche plus de chemin dans la page, mais apparaît dans `php-errors.log`. Test retiré ensuite. |
| S3 | **Limiter les envois du formulaire de contact** : 1 message par minute et 5 par heure pour une même IP. Réf. L2. | P0 | moyen | `contact.php`, `app/Contact/ContactStore.php` | Deux envois en moins d'une minute : le 2e affiche « Merci de patienter » et n'est pas enregistré. Le 1er envoi fonctionne toujours. |
| S4 | **Ajouter des en-têtes de sécurité HTTP** : `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: same-origin`. Réf. L3. | P1 | simple | `.htaccess` (racine, voir S1) | Les 3 en-têtes apparaissent dans l'onglet Réseau des outils de développement (F12), sur une page du site et du panel. |
| S5 | **Activer le mode strict des sessions** (`session.use_strict_mode`) par le code. Réf. L5. | P1 | simple | `app/Support/Session.php` | `ini_get('session.use_strict_mode')` vaut 1 dans le panel. La connexion, la déconnexion et `verifier-panel.cmd` fonctionnent. |
| S6 | **Ajouter un jeton CSRF au formulaire de contact** : `contact.html` devient `contact.php`, qui affiche le formulaire. Réf. L2. | P1 | moyen | `contact.html` → `contact.php`, `app/Support/Csrf.php`, liens des pages | Un envoi sans jeton (formulaire recopié ailleurs) est refusé. Un envoi normal fonctionne. |
| S7 | **Masquer la saisie du mot de passe** dans `scripts/create_admin.php`, avec un petit lanceur PowerShell (`Read-Host -AsSecureString`). Réf. L7. | P2 | moyen | `scripts/create_admin.php`, `scripts/create_admin.ps1` (nouveau) | Le mot de passe tapé n'apparaît plus dans la console. Le compte créé permet de se connecter. |
| S8 | **Chiffrer le mot de passe SMTP** avec `openssl_encrypt()` (extension déjà active) et une clé dans `storage/app.key`. Réf. L1. Limite : la clé reste sur le même poste. | P2 | moyen | `app/MailConfig/MailSettings.php`, `storage/app.key` (nouveau) | `mail_config.json` ne contient plus le mot de passe lisible. « Tester l'envoi » fonctionne toujours. |

### 3.2 Stockage des données

| ID | Action | Prio. | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| D1 | **Sauvegarde locale de `storage/`** : script qui copie le dossier dans `sauvegardes\AAAA-MM-JJ_HHMM\`, hors du site web. | P1 | simple | `scripts/sauvegarder.ps1` (nouveau) | Après le lancement, un dossier daté contient une copie complète de `storage/`. Le site n'est pas modifié. |
| D2 | **Éviter les écritures perdues** : verrou (`flock`) autour de « lire puis écrire » dans `FileStore`. Réf. L8. | P1 | moyen | `app/Support/FileStore.php`, classes de `app/Auth/` | Tous les tests de `verifier-panel.cmd` passent. Deux connexions échouées simultanées comptent bien 2 échecs. |
| D3 | **Nettoyer les vieilles entrées** de `rate_limits.json` (plus de 24 h sans activité) à chaque écriture. Réf. L9. | P1 | simple | `app/Auth/RateLimiter.php` | Une entrée datée artificiellement d'il y a 2 jours disparaît après un nouvel échec de connexion. |
| D4 | **Coder l'outil « Sauvegarde »** du panel : téléchargement d'un export JSON des messages de contact et du journal (sans hash, sans mot de passe SMTP). Réf. L15. | P2 | moyen | `admin/backup.php` (nouveau), `views/maintenance_nav.php` | L'onglet « Sauvegarde » est actif. Le fichier téléchargé est un JSON valide qui ne contient aucun secret. |
| D5 | **Option : passer un stockage en SQLite** (inclus dans PHP sous WAMP, extension `pdo_sqlite` déjà active). Commencer par les messages de contact seulement. **Sous accord du professeur.** | P2 | complexe | `app/Contact/ContactStore.php`, `storage/site.sqlite` (nouveau) | Les messages sont enregistrés et lus dans SQLite avec des requêtes préparées. L'affichage dans le panel est identique. `check_logs.php` est adapté et passe. |

### 3.3 Conformité / RGPD

> RGPD : règlement européen sur les données personnelles. Il impose d'informer les personnes et de limiter la durée de conservation.

| ID | Action | Prio. | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| R1 | **Informer sous le formulaire de contact** : à quoi servent les données, durée de conservation, droit de suppression. Réf. L12. | P0 | simple | `contact.html` | Un paragraphe d'information est visible juste au-dessus du bouton « Envoyer le message ». |
| R2 | **Créer une page « Mentions légales et confidentialité »**, liée depuis le pied de page de toutes les pages. Réf. L12. | P1 | simple | `mentions-legales.html` (nouveau), pied de page des 4 pages | Le lien est présent dans le pied de page de chaque page et mène à la nouvelle page. |
| R3 | **Durée de conservation automatique** : suppression des journaux de plus de 90 jours et des messages de plus de 12 mois, avec un bouton dans Maintenance. Réf. L11. | P1 | moyen | `app/Maintenance/LogReader.php`, `app/Contact/ContactStore.php`, `admin/logs.php` | Un fichier de journal daté de plus de 90 jours est supprimé par le bouton, et l'action est journalisée. Les fichiers récents restent. |

### 3.4 Qualité du code

| ID | Action | Prio. | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| Q1 | **Mettre à jour le contenu daté** : promotions « jusqu'au 30 septembre » à corriger ou à retirer. Réf. L14. | P1 | simple | `index.html` | Aucune date dépassée n'apparaît sur l'accueil. |
| Q2 | **Factoriser l'en-tête et le pied de page** : pages en `.php` avec `include` d'un en-tête et d'un pied communs. Réf. L13. | P1 | moyen | `index.html` → `index.php`, idem pour les autres pages, `partials/header.php` et `partials/footer.php` (nouveaux) | Modifier un lien du menu dans `partials/header.php` le change sur toutes les pages. Les anciennes adresses `.html` redirigent vers les nouvelles. |
| Q3 | **Version automatique du CSS et du JS** : `filemtime()` à la place de `?v=3` écrit à la main. Réf. L18. **Dépend de Q2.** | P2 | simple | `partials/header.php`, `views/layout_header.php` | Après une modification de `style.css`, la nouvelle version s'affiche sans Ctrl+F5 et sans rien changer à la main. |
| Q4 | **Fenêtres de confirmation sans JavaScript** : page de confirmation de secours si JavaScript est désactivé. Réf. L17. | P2 | moyen | `admin/maintenance.php`, `admin/logs.php`, vues associées | JavaScript désactivé, la purge reste possible après une confirmation explicite. |

### 3.5 Front-end

| ID | Action | Prio. | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| F1 | **Catalogue produits dans un fichier JSON** affiché par PHP, à la place des produits écrits en dur. Réf. L14. **Dépend de Q2.** | P2 | moyen | `storage/products.json` ou `data/products.json` (nouveau), `produits.php` | Ajouter un produit dans le JSON le fait apparaître sur la page, sans toucher au HTML. |
| F2 | **Bouton thème clair / sombre**, avec le choix mémorisé (`localStorage`). Les couleurs sont déjà des variables CSS. | P2 | moyen | `css/style.css`, `js/site.js`, en-tête des pages | Le bouton bascule le thème. Le choix est conservé après un rechargement. Les contrastes restent lisibles. |
| F3 | **Remplacer l'emplacement de carte** par une image statique ou un lien vers un plan, sans traceur tiers. | P2 | simple | `contact.html` | L'emplacement vide est remplacé, et aucune requête vers un service externe n'est faite au chargement. |

### 3.6 Tests

| ID | Action | Prio. | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| T1 | **Checklist de tests manuels du JavaScript** : menu mobile, règles cochées, modales, messages qui disparaissent. Réf. L20. | P1 | simple | `TESTS_MANUELS.md` (nouveau) | Le fichier liste chaque comportement, avec l'action à faire et le résultat attendu. Une passe complète est cochée. |
| T2 | **Tester le panel sans le mot de passe réel** : `check_panel.ps1` crée un mot de passe temporaire, puis remet le fichier de comptes à l'identique. Réf. L19. | P2 | moyen | `tests/check_panel.ps1`, `verifier-panel.cmd` | `verifier-panel.cmd` ne demande plus de mot de passe. `accounts.json` est identique avant et après. |
| T3 | **Ajouter un test pour chaque action réalisée** (S3, D2, D3, R3…). | P1 | moyen | `tests/` (nouveaux fichiers), `verifier-panel.cmd` | Chaque action terminée a au moins une vérification automatique, et `verifier-panel.cmd` affiche 0 échec. |

### 3.7 Documentation

| ID | Action | Prio. | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| X1 | **Créer `PROMPTS.md`** avec les consignes exactes données à l'agent IA, et l'outil utilisé. | P1 | simple | `PROMPTS.md` (nouveau) | Le fichier contient chaque consigne, dans l'ordre, et il est déposé sur GitHub. |
| X2 | **Créer un `README.md` d'installation sous WAMP** : copie du dossier, création du compte, installation de PHPMailer, lancement des tests. | P1 | simple | `README.md` (nouveau) | Une personne qui suit le README sur un poste WAMP neuf arrive jusqu'à la page de connexion. |
| X3 | **Mettre à jour `DOCUMENTATION.md`** après chaque action terminée (fonctionnement et section « Limites connues »). | P1 | simple | `DOCUMENTATION.md` | La limite corrigée est retirée ou marquée « corrigée », avec le nom de l'action. |

### 3.8 Exploitation locale

| ID | Action | Prio. | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| E1 | **Ajouter au diagnostic Système** la taille des journaux et le nombre de messages de contact, avec un avertissement au-delà d'un seuil. Réf. L9. | P2 | simple | `app/Maintenance/SystemCheck.php` | La page Système affiche ces deux lignes. Un seuil dépassé apparaît en « Attention ». |
| E2 | **Documenter le problème d'antivirus et de certificat SMTP**, avec sa solution, dans le README. Réf. L21. | P2 | simple | `README.md` | Le README explique le symptôme (erreur de certificat), la cause et la solution. |

---

## 4. Définition de Terminé

Une action est **terminée** seulement si **tout** ce qui suit est vrai :

1. Le **critère d'acceptation** de l'action est vérifié dans le navigateur, sur `http://localhost/infoleader/`.
2. Le site **fonctionne toujours** : pages publiques, connexion, et chaque module du panel s'ouvre.
3. `verifier-panel.cmd` affiche **0 échec**, et un test a été ajouté si l'action modifie du PHP (voir T3).
4. **Aucune nouvelle dépendance** n'a été ajoutée sans accord du professeur. Rien ne sort du poste local.
5. **Aucun secret** (mot de passe, identifiant SMTP) n'apparaît dans le code, les journaux ou les fichiers publiés.
6. Le code ajouté est **commenté en français**, et vous savez l'expliquer à l'oral.
7. `DOCUMENTATION.md` est **à jour** (X3), et l'action est **cochée** dans ce fichier.

---

## 5. Séquencement par jalons

> Ne traitez qu'**une action à la fois**, et testez-la avant de passer à la suivante.

### Jalon 1 — Corrections critiques (P0)
- [x] S1 — Bloquer l'accès web aux fichiers sensibles de la racine *(terminé : `.htaccess` racine, listes de dossiers désactivées, contrôles ajoutés à `tests/check_panel.ps1`)*
- [ ] S2 — Ne plus afficher les erreurs PHP
- [ ] R1 — Informer sous le formulaire de contact
- [ ] S3 — Limiter les envois du formulaire de contact

### Jalon 2 — Fiabilité et sécurité (P1)
- [ ] S4 — En-têtes de sécurité HTTP
- [ ] S5 — Mode strict des sessions
- [ ] D1 — Sauvegarde locale de `storage/`
- [ ] D3 — Nettoyage de `rate_limits.json`
- [ ] D2 — Verrou contre les écritures perdues
- [ ] T3 — Un test par action réalisée

### Jalon 3 — Conformité, contenu, documentation (P1)
- [ ] Q1 — Contenu daté
- [ ] R2 — Mentions légales et confidentialité
- [ ] R3 — Durée de conservation automatique
- [ ] X1 — `PROMPTS.md`
- [ ] X2 — `README.md` d'installation
- [ ] T1 — Checklist de tests manuels
- [ ] S6 — Jeton CSRF sur le formulaire de contact
- [ ] Q2 — Factorisation de l'en-tête et du pied de page

### Jalon 4 — Pour aller plus loin (P2)
- [ ] Q3, Q4, F1, F2, F3, S7, S8, D4, T2, E1, E2
- [ ] D5 — SQLite (**sous accord du professeur**)

`X3` (mise à jour de la documentation) s'applique **à la fin de chaque action**.

---

## 6. Petit lexique

| Terme | Explication courte |
|---|---|
| `.htaccess` | Fichier de réglages qu'Apache lit dans un dossier (accès, en-têtes…). |
| CSRF | Attaque qui fait envoyer un formulaire à votre insu depuis un autre site. Un jeton secret la bloque. |
| En-têtes HTTP | Informations envoyées par le serveur avec chaque page. Certaines protègent le navigateur. |
| `display_errors` | Réglage PHP qui affiche les erreurs dans la page. Pratique pour développer, risqué sinon. |
| `flock` | Verrou de fichier PHP : une seule requête écrit à la fois. |
| JSON / JSONL | Format texte de données. JSONL = un objet JSON par ligne, pratique pour les journaux. |
| Mode strict des sessions | PHP refuse un identifiant de session qu'il n'a pas créé lui-même. |
| RGPD | Règlement européen sur la protection des données personnelles. |
| SQLite | Base de données contenue dans un seul fichier. Elle est fournie avec PHP, rien à installer. |
| `localStorage` | Petite mémoire du navigateur, propre à chaque site, qui survit à la fermeture de la page. |
