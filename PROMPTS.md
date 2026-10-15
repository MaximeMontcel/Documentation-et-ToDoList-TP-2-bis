# Consignes données à l'agent IA — TP 2 bis

Ce fichier retrace les consignes exactes que j'ai données à l'agent IA pour ce TP, dans l'ordre.
Il complète `DOCUMENTATION.md` et `TODO_LIST.md`.

## Outil utilisé (transparence)

- **Agent IA : Claude Code** (application de bureau, modèle Claude d'Anthropic).
- Le sujet du TP prévoyait **OpenCode**. J'ai utilisé Claude Code, avec lequel j'avais terminé le site au TP 1-2. La démarche reste celle du TP : consignes précises, une tâche à la fois, vérification et validation à chaque étape.
- L'agent travaillait sur le dossier du site (`C:\wamp64\www\infoleader`, relié à mon dossier de cours).
- Certaines de mes réponses ont été choisies parmi des **réponses proposées par l'interface** (boutons), puis envoyées telles quelles. Elles sont recopiées ci-dessous exactement comme elles ont été envoyées.

---

## Partie A — Analyse du site et DOCUMENTATION.md

**Consigne 1** — j'ai collé à l'agent l'intégralité du sujet du TP 2 bis (sections 1 à 12), suivie de :

> voici le nouveau tp toujours en lien avec le meme site

Le sujet contenait la consigne d'analyse de la partie A.2, que l'agent a appliquée :

> Tu es développeur web senior. Analyse l'intégralité du site web présent dans le dossier courant et rédige un fichier DOCUMENTATION.md à la racine du projet.
>
> Règles impératives :
> - Base-toi UNIQUEMENT sur le code réellement présent. N'invente rien : si une information manque, indique-le explicitement.
> - Écris en français, au format Markdown, avec une table des matières en haut du document.
> - Utilise un ton professionnel et des phrases courtes.
>
> Plan attendu du document : 1. Présentation générale […] 10. Limites connues et risques […]
>
> Écris directement le fichier DOCUMENTATION.md.

*Résultat* : `DOCUMENTATION.md`, avec 2 sections ajoutées (« Tests » et « Points forts »). Chaque affirmation cite le fichier qui la justifie. L'agent a vérifié qu'aucun accès à une base de données n'existe dans le code, et qu'aucun secret (identifiants, adresses personnelles) n'apparaît dans le document, qui est destiné à un dépôt public.

---

## Partie C — TODO_LIST.md

**Consigne 2** (après ma relecture de DOCUMENTATION.md) :

> J'ai relu DOCUMENTATION.md, c'est bon. Génère TODO_LIST.md (Partie C).

L'agent a appliqué la consigne de la partie C du sujet : contexte BTS SIO 1re année, WAMP uniquement, aucune dépendance, paiement simulé, JSON accepté, SQLite en option, priorités P0/P1/P2, complexité, critères d'acceptation, « Définition de Terminé » et jalons.

*Résultat* : `TODO_LIST.md`, avec 31 actions dont 4 P0. La colonne « Fichiers concernés », suggérée par le sujet, a été ajoutée. Une vérification automatique a confirmé qu'aucune action ne propose de serveur distant, Docker, déploiement automatisé, paiement réel ou dépendance.

---

## Parties B et D — GitHub et Discord (faites par moi)

Création du dépôt public, dépôt des fichiers et publication du lien sur Discord : je les ai faits moi-même, sans l'agent.

**Consigne 3** :

> DOCUMENTATION.md et TODO_LIST.md sont publiés sur GitHub et le lien est sur Discord. Partie E : commence par la première tâche P0 (S1), en m'expliquant avant ce que tu vas modifier.

---

## Partie E — Évolution du site, tâche par tâche (jalon 1 : P0)

Pour chaque tâche, l'agent m'a présenté ce qu'il allait modifier et les fichiers concernés, puis **a attendu ma validation**. Après la modification, il m'a donné une procédure de test dans le navigateur.

### Tâche S1 — Bloquer l'accès web aux fichiers sensibles

**Consigne 4** :
> Validé : réalise S1 avec les 3 règles, y compris Options -Indexes.

**Consigne 5** :
> comment vérifier

**Consigne 6** :
> j'ai pas de message de contact dans messages de contact et la page sauvegarde n'est pas dispo

*Esprit critique* : mon test a révélé un vrai problème. Mon navigateur affichait une **ancienne version de `contact.html` gardée en cache**, qui n'envoyait rien. L'agent l'a prouvé avec le journal d'accès d'Apache, puis l'a corrigé dans le `.htaccess` (les pages HTML sont revérifiées à chaque visite). La page « Sauvegarde » est désactivée exprès : c'est la tâche D4 (P2).

**Consigne 7** :
> Après Ctrl+F5, le message de contact apparaît bien dans Journaux. S1 validée : passe à S2 en m'expliquant avant ce que tu vas modifier.

### Tâche S2 — Ne plus afficher les erreurs PHP

**Consigne 8** :
> la page sauvegarde n'est pas dispo

**Consigne 9** :
> OK, la Sauvegarde attendra. Réalise S2 comme décrit (config.php, ligne dans Système, test, page piège temporaire).

**Consigne 10** :
> S2 validée dans mon navigateur. Supprime test-s2.php puis passe à la tâche P0 suivante (R1), en m'expliquant avant ce que tu vas modifier.

### Tâche R1 — Informer sous le formulaire de contact (RGPD)

**Consigne 11** :
> vos données perso n'apparait pas

*Remarque* : l'agent n'avait pas encore fait la modification, car il attendait ma validation. Il l'a faite ensuite.

**Consigne 12** :
> R1 validée, je vois bien le paragraphe. Passe à la dernière tâche P0 (S3), en m'expliquant avant ce que tu vas modifier.

### Tâche S3 — Limiter les envois du formulaire de contact

**Consigne 13** :
> Validé : réalise S3 comme décrit (1 par minute, 5 par heure, journalisation, tests).

*Esprit critique* : les tests automatiques ont révélé un défaut d'ergonomie. Une personne qui renvoyait un message avec une adresse invalide voyait « patienter » au lieu de « adresse invalide ». L'ordre des contrôles a été corrigé : les erreurs de saisie s'affichent d'abord, et la limite ne s'applique qu'au moment d'enregistrer.

**Consigne 14** :
> S3 validée dans mon navigateur : le 2e envoi demande bien de patienter. Le jalon P0 est terminé.

### Ce fichier

**Consigne 15** :
> fait le prompt

---

## Vérifications faites à chaque étape

- **Par moi, dans le navigateur** (WAMP local) : chaque tâche, en suivant la procédure de test donnée par l'agent, avant de valider.
- **Par les tests automatiques** (`verifier-panel.cmd`) : relancés après chaque tâche, avec 0 échec. Un test a été ajouté pour chaque tâche (S1, S2, R1 dans `tests/check_panel.ps1` ; S3 dans `tests/check_logs.php`).
- **Mise à jour des documents** : chaque tâche terminée est cochée dans `TODO_LIST.md`, et la limite correspondante est marquée « corrigée » dans `DOCUMENTATION.md`.
