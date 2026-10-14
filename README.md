# GStock — version web

Application de gestion de stock en français : React/Vite côté interface, Django REST Framework côté serveur, PostgreSQL pour la production. Le mode SQLite local est réservé au développement et aux tests.

## Démarrage Windows

Double-cliquer sur **lanceur.cmd** pour demarrer les deux serveurs en arriere-plan et ouvrir GStock dans un nouvel onglet Firefox. Le lanceur attend la disponibilite de l'API et de l'interface, reutilise les serveurs deja actifs et signale les ports occupes par une autre application. Les journaux des serveurs demarres sont conserves dans logs/. Firefox doit etre installe ; l'installation initiale reste assuree par scripts/setup.ps1.

Environnement vérifié : Python 3.14.6, Node 26.5.0, npm 11.17.0. Depuis la racine du projet :

```powershell
./scripts/setup.ps1
```

Puis dans deux terminaux distincts :

```powershell
./scripts/start-backend.ps1
./scripts/start-frontend.ps1
```

Ouvrir **http://127.0.0.1:5175**. Le backend écoute sur 127.0.0.1:8005. Créer votre compte sur /register : l’inscription crée votre organisation et ouvre une session. Les scripts ne suppriment pas les données et ne remplacent pas un .env existant.

La configuration locale backend/.env a été générée avec une clé aléatoire, sans la publier. Pour un autre poste, setup.ps1 génère la sienne à partir de backend/.env.example. Ne jamais versionner .env, les bases, les images privées ou des mots de passe.

## Fonctions implémentées

- Comptes : inscription, connexion, déconnexion, profil, changement d’email avec mot de passe actuel, changement et réinitialisation de mot de passe.
- Organisation et rôles : administrateur, gestionnaire, vendeur, comptable, lecteur ; autorisations vérifiées côté serveur ; création d’accès par invitation et désactivation.
- Produits : liste paginée, recherche, filtres, tri, création, détails, modification, archivage, catégories, images privées validées et réencodées.
- Stock : entrées, sorties, corrections signées, historique avec utilisateur/motif/solde ; quantité protégée contre l’édition directe ; stock négatif refusé.
- Clients et fournisseurs : création, modification, recherche et archivage.
- Ventes et achats : documents multi-lignes, brouillons, validation/réception atomique, annulation complète avec mouvement inverse et historique.
- Dashboard : indicateurs réels de l’organisation, période, ventes par jour, alertes, derniers mouvements et activité.
- Rapports : valorisation aux prix d’achat actuels, meilleurs produits, exports CSV et Excel protégés contre les formules injectées.
- Site public : accueil, À propos, Fonctionnalités, Contact ; démonstration fictive clairement isolée sous /demo ; interface mobile avec tableaux défilables, chargements, erreurs et confirmations.

## Limites et configuration externe

Le **service email doit être configuré** avec votre fournisseur et vos coordonnées officielles. Le backend console affiche les emails dans le terminal local ; il ne les livre pas. Le contact reste explicitement indisponible tant que CONTACT_EMAIL n’est pas configuré. Ne pas exposer les liens de réinitialisation des logs de développement.

Une organisation par compte, une devise par organisation ; pas de retours partiels, de réceptions partielles, de gestion des paiements/taxes, de PDF, de valorisation FIFO ou de fonctionnement hors ligne. Ces limites sont indiquées dans l’interface. Desktop/licences : préparation documentée seulement.

Import de catalogue : dans **Produits → Importer des produits**, téléchargez le modèle CSV, remplacez sa ligne d’exemple, choisissez votre CSV UTF-8 ou XLSX, analysez puis confirmez. Maximum 1 000 produits / 5 Mo, première feuille Excel, sans formules. Référence et nom obligatoires ; catégories existantes de votre organisation uniquement. Les références existantes ou répétées sont refusées, aucun produit n’est remplacé. Toutes les lignes doivent être valides avant création ; le stock initial génère des mouvements traçables.

**La livraison a été vérifiée localement sur SQLite.** PostgreSQL et son test concurrent, SMTP réel, reverse proxy HTTPS, sauvegarde/restauration PostgreSQL et déploiement public ne sont pas validés sur ce poste. La configuration de production est préparée ; ce n’est pas une certification ni un déploiement commercial effectué.

## Qualité

```powershell
.\.venv\Scripts\python.exe backend/manage.py test inventory
.\.venv\Scripts\python.exe backend/manage.py check
.\.venv\Scripts\python.exe backend/manage.py makemigrations --check --dry-run
```

Dans frontend :

```powershell
npm run lint
npm test
npm run build
npm audit
```

Tests critiques : organisations, références croisées, rôles, CSRF, authentification, stock négatif, traçabilité, rollback d’une vente multi-lignes, validation/annulation répétée, images privées, exports et calculs réels. Le test concurrent requiert TEST_POSTGRES=true et une base PostgreSQL de test ; il est ignoré sous SQLite. Les tests frontend Node vérifient les erreurs API, la pagination des choix et les erreurs d’export. Le navigateur a servi aux contrôles de parcours et responsive ; ce contrôle manuel ne remplace pas une suite E2E automatisée.

## Architecture et guides

- frontend/src : pages, composants réutilisables, contextes de session, hooks de chargement et services API.
- backend/config : configuration par environnement, URLs et WSGI.
- backend/inventory : modèles, serializers, permissions, opérations de stock et API ; migrations conservées et versionnables.
- scripts : installation et lancement Windows.
- deploy/nginx.conf.example : modèle de reverse proxy ; compose.yaml : PostgreSQL local optionnel.

Consulter [l’API](docs/API.md), [installation/déploiement/sauvegardes](docs/DEPLOYMENT.md), [le bilan web](docs/WEB-DELIVERY.md), [la feuille de route](ROADMAP.md), [le journal](CHANGELOG.md) et [la préparation desktop](docs/DESKTOP.md). [L’audit initial](docs/AUDIT.md) décrit l’état antérieur, pas les fonctions actuelles.

## Données et sauvegarde initiale

backups/initial-2026-10-13.zip conserve les sources originales ; aucune donnée antérieure n’a été détruite. Une organisation explicitement nommée « Organisation de test GStock » et son article/vente d’essai ont été créés dans la base locale pour vérifier le navigateur. Ce sont des données de développement séparées de votre future organisation ; ne pas déployer dev.sqlite3 en production.

Le projet n’a pas encore d’historique Git. Initialiser Git après revue des fichiers et des exclusions, puis créer un premier commit ; les migrations et verrouillages doivent être suivis. Les secrets, sauvegardes privées et données doivent rester exclus.
