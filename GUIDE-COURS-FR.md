# Incident Desk — aide-mémoire pour le cours

## État vérifié sur cet ordinateur

- L’application Incident Desk répond actuellement sur `http://127.0.0.1:8000`.
- PostgreSQL local fonctionne dans Docker sur `127.0.0.1:55432`; la base s’appelle `incidentdesk`.
- Les comptes `Nazairefabsimple` et `Nazairefabadmin` existent dans cette base locale. Le premier n’a ni statut staff ni statut superutilisateur; le second est superutilisateur.
- Les migrations sont appliquées et les tests passent : 30 tests découverts, 25 réussis et 5 ignorés hors du laboratoire de sécurité optionnel.
- `makemigrations --check` et la vérification Django locale ne signalent aucun problème.
- Le dépôt n’a pas encore de remote GitHub. Aucun déploiement Vercel ni base hébergée n’a été créé.

Les liens locaux ne sont accessibles que depuis cet ordinateur tant que le serveur fonctionne :

- [Connexion administrateur Django](http://127.0.0.1:8000/admin/login/?next=/admin/)
- [Connexion à l’application](http://127.0.0.1:8000/accounts/login/)
- PostgreSQL n’est pas une page web : adresse `127.0.0.1`, port `55432`, base `incidentdesk`. La configuration complète est dans le fichier local `.env`, qui ne doit pas être publié.

Pour démarrer ou vérifier les services dans PowerShell, depuis le dossier contenant `manage.py` :

```powershell
docker compose up -d --wait db
.\.venv\Scripts\python.exe manage.py migrate
.\.venv\Scripts\python.exe manage.py runserver 127.0.0.1:8000
```

Le port 8000 répond déjà sur cet ordinateur. Ne démarrez pas une deuxième instance sur le même port si le serveur est encore en marche.

## Comptes demandés

Les deux comptes demandés sont déjà présents dans la base locale avec les rôles attendus. Le mot de passe actuel n’a pas été vérifié ni réinitialisé; si la connexion échoue, réinitialisez-le depuis Django Admin avec un mot de passe privé.

Pour refaire cette configuration dans une nouvelle base, créer l’administrateur Django dans un terminal du projet :

```powershell
.\.venv\Scripts\python.exe manage.py createsuperuser
```

Saisir le nom d’utilisateur administrateur demandé, puis le mot de passe demandé aux invites interactives. Ensuite, se connecter à l’[administration Django](http://127.0.0.1:8000/admin/login/?next=/admin/), ouvrir **Users**, puis ajouter l’utilisateur simple demandé. Laisser **Staff status** et **Superuser status** décochés pour ce compte.

Ne mettez aucun mot de passe dans le code, ce document, GitHub, une capture d’écran ou une variable publique. Le mot de passe fourni pour le cours ne doit pas être réutilisé sur un déploiement public; utilisez des mots de passe distincts et privés en production. Ce projet ne crée pas automatiquement de comptes à l’installation.

Vérification à présenter :

1. Se connecter comme utilisateur simple à l’application.
2. Créer un incident de test sans donnée personnelle ou réelle.
3. Créer un deuxième utilisateur simple et vérifier qu’il ne voit pas l’incident du premier.
4. Se déconnecter et montrer que l’accès aux pages privées requiert une connexion.

Les incidents sont filtrés par propriétaire dans les vues de l’application. Le compte superutilisateur peut administrer les données via Django Admin. Il n’existe pas encore de rôle métier « administrateur » distinct ni de gestion complète des rôles dans l’application.

## GitHub : publier le code

Liens utiles : [créer un dépôt GitHub](https://github.com/new) et [installer Git pour Windows](https://git-scm.com/download/win).

Le dossier du projet contient déjà des métadonnées Git, mais Git n’est pas disponible dans l’environnement vérifié et aucun dépôt distant n’est configuré. Après installation de Git, ouvrir PowerShell dans le dossier contenant `manage.py` :

```powershell
git status --short --branch
git remote -v
```

Créer ensuite un dépôt personnel vide sur GitHub, puis suivre [les étapes GitHub du projet](GITHUB.md). Avant chaque envoi, vérifier que `.env`, `.venv`, les bases de données et les secrets ne sont pas inclus. Le lien du dépôt sera `https://github.com/VOTRE-NOM/incident-desk` une fois créé; ce lien n’est pas encore actif.

Le workflow [tests.yml](../.github/workflows/tests.yml) exécute les tests Django sur SQLite et PostgreSQL lors des push et pull requests. C’est une base de CI, pas encore un pipeline complet de sécurité ou un blocage garanti du déploiement.

## Déploiement : liens à créer avec vos comptes

- [Vercel](https://vercel.com/dashboard) hébergera l’application après connexion au dépôt GitHub.
- [Neon](https://console.neon.tech/) est un exemple de fournisseur PostgreSQL séparé pour la base hébergée.
- La procédure et les variables requises sont dans le [guide de déploiement](DEPLOYMENT.md).

À ce stade, il n’y a **ni URL publique de l’application, ni URL de base PostgreSQL hébergée**. Ces liens ne peuvent être obtenus qu’après création des projets avec vos comptes et configuration des secrets dans Vercel. Ne publiez jamais `DATABASE_URL` : elle contient les identifiants de connexion à la base.

Pour répondre au professeur :

- **Site en cours** : `http://127.0.0.1:8000` (local uniquement).
- **Base en cours** : PostgreSQL local, `127.0.0.1:55432`, base `incidentdesk` (pas une URL web).
- **Dépôt GitHub** : pas encore publié; le remote manque.
- **Site et base en ligne** : pas encore créés; ne présentez pas d’URL fictive comme un déploiement.

## Questions probables

**Quelle technologie utilisez-vous ?** Django/Python côté serveur, PostgreSQL pour les données, HTML/CSS pour les pages. Les migrations Django gèrent le schéma.

**Comment l’application protège-t-elle les incidents ?** Il faut être connecté; les listes et les détails sont limités aux incidents de l’utilisateur. Le propriétaire est défini côté serveur. Les formulaires utilisent CSRF et les vues Django ORM.

**Que fait actuellement la CI ?** Elle vérifie les migrations et lance les tests sur deux configurations de base de données. Le déploiement et les analyses SAST/DAST ne sont pas encore des contrôles obligatoires.

**Qu’est-ce qui reste à faire ?** Publier le dépôt, configurer puis vérifier le déploiement et la base hébergée, et poursuivre les étapes de sécurité et de gestion des rôles indiquées dans le [calendrier du projet](MILESTONES.md). La page CV et les rôles métier relèvent de l’étape finale; ils ne sont pas à présenter comme déjà terminés.
