# Documentation technique — Smart Fridge & Nutrition Coach

## 1. Présentation et périmètre

Smart Fridge est une application web de gestion des produits alimentaires et de suivi nutritionnel. Elle propose un inventaire de réfrigérateur, des alertes de péremption, des recettes, un journal de consommation, un planning de repas, des notifications et une interface d'administration.

**Ce que cette documentation permet :** démarrer l'application, retrouver les composants, comprendre les échanges de données, tester un parcours utilisateur et identifier les limites techniques.

**Prérequis de lecture :** notions élémentaires de Python, HTTP (GET/POST), SQL, environnement virtuel et variables d'environnement. Un lecteur débutant peut suivre les commandes de la section 4.

## 2. Architecture et interactions

```mermaid
flowchart TD
    U[Utilisateur / navigateur] --> F[Pages HTML FastAPI]
    F --> R[Routes et logique métier]
    R --> ORM[SQLAlchemy]
    ORM --> DB[(PostgreSQL Supabase ou SQLite)]
    R --> M[TheMealDB - recettes]
    R --> USDA[USDA FoodData Central - nutrition]
    R --> T[Service de traduction]
    A[API JSON /docs] --> R
```

- **Serveur :** FastAPI, démarré dans `main.py`.
- **Interface :** HTML rendu côté serveur via `app/web/layout.py` et `app/web/pages/`, styles dans `static/`.
- **API :** routes de compte dans `app/routers/` ; documentation OpenAPI accessible via `/docs` lorsque le serveur est actif.
- **Persistance :** modèles SQLAlchemy dans `app/db/models.py`, connexion et sessions dans `app/db/database.py`.
- **Services externes :** TheMealDB et USDA via `app/web/data.py` et d'autres fonctions de pages ; traductions et caches côté application.

### Arborescence utile

```text
main.py                         # Création FastAPI, routeurs, middleware, erreurs
app/
  core/                         # Sécurité, JWT, dépendances, calcul métabolique
  db/database.py                # DATABASE_URL, moteur, sessions, ajustements de schéma
  db/models.py                  # Modèles et tables SQLAlchemy
  routers/auth.py               # API inscription, connexion, compte
  routers/profile.py            # API profil
  web/layout.py                 # Rendu des pages et middleware d'authentification
  web/data.py                   # Requêtes de données externes
  web/pages/
    home.py                     # Accueil
    fridge.py                   # Frigo, recettes, consommation et restes
    nutrition.py                # Bilan nutritionnel
    planner.py                  # Planning des repas
    notifications.py            # Notifications
    alerts.py                   # Alertes
    admin.py                    # Administration
    profile.py                  # Profil utilisateur
static/                          # Ressources CSS et autres fichiers statiques
create_admin.py                 # Création d'un administrateur
migrate_sqlite_to_supabase.py   # Migration de données
 test_supabase_connection.py    # Vérification de connexion PostgreSQL
requirements-supabase.txt       # Dépendances additionnelles PostgreSQL/traduction
README.md                       # Présentation et consignes du dépôt
```

**Attention :** `app/web/routes.py` existe mais les pages sont enregistrées individuellement depuis `main.py`. Pour modifier une route effectivement active, partir des imports et `include_router` de `main.py`, pas uniquement du nom d'un fichier.

## 3. Configuration et base de données

### Paramètres

| Paramètre | Où | Utilité | Vérification |
|---|---|---|---|
| `DATABASE_URL` | `.env` à la racine | Connexion PostgreSQL ; si absent, repli SQLite `sqlite:///./smartfridge.db` | Script de connexion puis démarrage |
| `USDA_API_KEY` | Variable d'environnement | Requêtes USDA | Résultats du catalogue / nutrition |
| Paramètres JWT | `app/core/jwt.py` et configuration associée | Émission et validation de jetons | Tester connexion et route protégée |

Ne jamais publier le `.env`, un mot de passe ou une clé d'API dans Git. Ne pas copier les identifiants d'une autre personne.

### Tables identifiées dans `app/db/models.py`

| Table | Fonction |
|---|---|
| `users` | Comptes, profil et rôle administrateur |
| `fridge_items` | Produits, quantité, catégorie et date de péremption |
| `app_dates` | Date simulée par utilisateur |
| `daily_logs` | Historique et instantanés journaliers |
| `recipe_translations` | Traductions de recettes conservées |
| `nutrition_intakes` | Consommations nutritionnelles |
| `recipe_leftovers` | Restes de recettes |
| `meal_plans` | Repas planifiés |
| `recipe_nutrition_cache` | Valeurs nutritionnelles estimées en cache |
| `notifications` | Notifications et état de lecture |
| `admin_logs` | Historique d'actions administratives |

Les noms des tables sont ceux des déclarations de modèles. Pour les colonnes exactes et contraintes, consulter `app/db/models.py` ; ne pas inférer de clés étrangères non déclarées.

**Création de schéma :** `main.py` appelle `Base.metadata.create_all(bind=engine)` au démarrage. Cela crée les tables absentes mais **ne migre pas automatiquement toutes les colonnes d'une table existante**. `app/db/database.py` contient des ajustements ponctuels de colonnes (`ensure_fridge_user_id_column`, `ensure_user_profile_columns`) ; toute autre modification structurelle exige une migration vérifiée et une sauvegarde préalable.

## 4. Installer et lancer le projet (Windows PowerShell)

### Prérequis

- Python installé et accessible avec `python`.
- Accès à une base PostgreSQL Supabase **ou** utilisation volontaire du mode SQLite de repli.
- Accès Internet pour les APIs de recettes/nutrition.
- Fichiers du projet extraits dans un dossier local.

### Procédure

1. Ouvrir PowerShell **dans le dossier qui contient `main.py`**.
2. Créer l'environnement :

   ```powershell
   python -m venv venv
   .\venv\Scripts\Activate.ps1
   ```

3. Installer les dépendances. L'archive examinée contient `requirements-supabase.txt` mais **pas de `requirements.txt` à la racine** : la commande indiquée dans le README (`pip install -r requirements.txt`) ne fonctionnera pas telle quelle sur cette archive. Utiliser au minimum :

   ```powershell
   python -m pip install fastapi uvicorn sqlalchemy python-dotenv "psycopg[binary]" requests python-multipart passlib bcrypt python-jose deep-translator
   python -m pip install -r requirements-supabase.txt
   ```

   Cette liste est une **base de démarrage**, pas un verrouillage de versions garanti. Si un import manque, identifier le paquet correspondant et ajouter un manifeste de dépendances versionné.

4. Créer `.env` à la racine, avec la chaîne fournie par votre propre projet Supabase :

   ```dotenv
   DATABASE_URL=postgresql+psycopg://UTILISATEUR:MOT_DE_PASSE@HOTE:5432/postgres
   USDA_API_KEY=VOTRE_CLE_SI_UTILISEE
   ```

   Adapter les autres paramètres nécessaires après lecture de `app/core/jwt.py` et des modules concernés. Ne pas laisser de valeurs d'exemple en production.

5. Tester la connexion puis démarrer :

   ```powershell
   python test_supabase_connection.py
   uvicorn main:app --reload
   ```

6. Ouvrir `http://127.0.0.1:8000/` et `http://127.0.0.1:8000/docs`.

### Résultat attendu

- Le serveur démarre sans traceback.
- La page d'accueil s'affiche.
- `/docs` présente les routes API.
- Une inscription/connexion de test fonctionne dans une base de test.

**En cas d'échec :** `ModuleNotFoundError` → dépendance manquante ; erreur SQLAlchemy/psycopg → vérifier `DATABASE_URL`, réseau et droits ; erreur `Address already in use` → arrêter l'ancien serveur ; changements invisibles → vérifier le dossier de lancement et redémarrer Uvicorn.

## 5. Parcours techniques vérifiables

### 5.1 Authentification et profil

**Rôle :** créer un compte, se connecter et gérer ses informations.  
**Code :** `app/routers/auth.py`, `app/routers/profile.py`, `app/core/security.py`, `app/core/jwt.py`, `app/web/layout.py`.  
**Routes API :** `POST /register`, `POST /login`, `GET /me` (selon les préfixes éventuellement définis sur les routeurs), et routes de profil.  
**Test :** créer un utilisateur de test, se connecter, ouvrir le profil, vérifier qu'un utilisateur non connecté ne peut pas modifier le profil d'autrui.  
**Limite :** la présence d'un schéma Bearer dans OpenAPI ne prouve pas à elle seule que chaque page HTML applique le même contrôle ; vérifier les dépendances et middleware réels.

### 5.2 Inventaire et péremption

**Rôle :** ajouter, consulter et supprimer des produits, détecter les dates proches.  
**Code :** `app/web/pages/fridge.py`, `app/web/pages/alerts.py`, `app/db/models.py`.  
**Routes :** `GET /fridge`, `POST /fridge`, `POST /fridge/delete`, `POST /fridge/next-day`, `GET /alerts`.  
**Exemple :** ajouter un produit de test avec date de péremption proche, consulter Frigo puis Alertes ; utiliser l'avancement de date simulée et vérifier le changement d'alerte.  
**Résultat attendu :** le produit apparaît pour le bon compte et l'alerte reflète la date simulée.  
**Cas particulier :** la date simulée (`app_dates`) n'est pas nécessairement la date réelle utilisée dans tous les autres modules ; vérifier la cohérence avant de rapprocher péremption et consommation.

### 5.3 Recettes, consommation et restes

**Rôle :** rechercher des recettes, afficher leurs instructions, déclarer une portion consommée et gérer le reste.  
**Code :** `app/web/pages/fridge.py`, `app/web/data.py`, modèles `NutritionIntakeDB` et `RecipeLeftoverDB`.  
**Routes :** `GET /recipes`, `GET /recipes/{meal_id}/instructions`, `POST /recipes/consume`, `POST /recipes/leftover/consume`.  
**Exemple :** sélectionner une recette, consulter ses instructions, déclarer 50 % consommés, puis vérifier le bilan nutritionnel et les restes.  
**Résultat attendu :** seule la portion déclarée est ajoutée au suivi réel ; le reste peut être retrouvé.  
**Limites :** disponibilité et traduction des recettes dépendent des services externes et du cache ; les valeurs nutritionnelles sont des estimations, pas une mesure clinique.

### 5.4 Planning des repas

**Rôle :** enregistrer et retirer des recettes prévues sur un calendrier.  
**Code :** `app/web/pages/planner.py`, `MealPlanDB`, `RecipeNutritionCacheDB`.  
**Routes identifiées :** `GET /planner`, `POST /planner/save`, `POST /planner/delete`.  
**Exemple :** choisir un jour et un repas, enregistrer une recette, recharger `/planner`, vérifier qu'elle persiste, puis la supprimer.  
**Résultat attendu :** la recette planifiée reste visible après rechargement et disparaît après suppression.  
**Limite importante :** dans **cette archive examinée**, `planner.py` ne contient **pas** de route « Préparer » ni de logique de consommation depuis le planning. La fonctionnalité « Planifier → Préparer → Manger » évoquée dans d'autres versions ne doit **pas** être présentée comme implémentée ici sans intégrer et tester les fichiers correspondants.

### 5.5 Nutrition et notifications

**Rôle :** afficher le suivi nutritionnel et les alertes persistantes.  
**Code :** `app/web/pages/nutrition.py`, `app/web/pages/notifications.py`, `app/core/metabolism.py`.  
**Routes :** `GET /nutrition`, `GET /notifications`, `POST /notifications/read`, `POST /notifications/read-all`.  
**Test :** déclarer un repas consommé, ouvrir Nutrition ; ouvrir Notifications, marquer un élément lu et actualiser.  
**Résultat attendu :** la consommation apparaît dans le bilan ; l'état lu/non lu reste cohérent après actualisation.  
**Limite :** vérifier les règles d'objectifs nutritionnels et l'origine des données avant d'interpréter les résultats.

### 5.6 Administration

**Rôle :** administrer les utilisateurs, produits et journaux.  
**Code :** `app/web/pages/admin.py`, `app/core/admin.py`, `create_admin.py`.  
**Routes :** `GET /admin`, `GET /admin/users/{user_id}`, plusieurs `POST /admin/users/{user_id}/...`.  
**Test :** créer un administrateur en environnement de test avec `python create_admin.py` ; vérifier qu'un compte ordinaire n'accède pas aux opérations administrateur.  
**Résultat attendu :** l'administrateur accède aux écrans autorisés, l'utilisateur standard est refusé.  
**Attention :** tester les permissions sur le serveur, pas seulement la visibilité des boutons.

## 6. Modifier le projet sans casser les autres modules

| Besoin | Fichiers de départ | Vérification minimale |
|---|---|---|
| Ajouter un champ produit | `app/db/models.py`, `app/web/pages/fridge.py` | Migration + création/lecture d'un produit |
| Changer une règle de péremption | `app/web/pages/alerts.py`, `app/web/pages/fridge.py` | Dates J-3, J-1, aujourd'hui, dépassée |
| Modifier une formule nutritionnelle | `app/core/metabolism.py`, `app/web/pages/nutrition.py` | Cas de test avec données connues |
| Ajouter un statut au planning | `app/db/models.py`, `app/web/pages/planner.py` | Persistance après redémarrage, migration |
| Ajouter une notification | `app/web/pages/notifications.py` et producteur concerné | Notification créée, lue, non dupliquée |
| Ajouter une route | Module `app/web/pages/` puis `main.py` | URL répond, contrôle d'accès testé |
| Modifier le style | `static/`, `app/web/layout.py` | Vérifier affichage desktop et mobile |

### Contrôles avant livraison

1. Sauvegarder la base de test avant toute modification de schéma.
2. Vérifier la syntaxe des fichiers Python modifiés : `python -m compileall -q app main.py`.
3. Démarrer l'application et ouvrir la page modifiée.
4. Tester un parcours nominal **et** un cas d'erreur (non connecté, entrée invalide, API externe indisponible).
5. Vérifier qu'aucune clé, mot de passe, `.env` ou dossier `venv` n'est ajouté au dépôt.
6. Mettre à jour la présente documentation si une route, un modèle, une configuration ou une limite change.

## 7. Diagnostic et limites connues

| Symptôme | Cause possible | Action | Résultat à vérifier |
|---|---|---|---|
| Démarrage impossible | Dépendance absente | Lire la dernière ligne du traceback, installer le paquet | Démarrage sans erreur |
| Connexion base refusée | `DATABASE_URL` invalide ou accès réseau | Exécuter le script de connexion, contrôler URL et droits | Connexion réussie |
| Table présente, colonne absente | `create_all()` ne migre pas une table existante | Sauvegarder puis appliquer une migration ciblée | Colonne visible dans PostgreSQL |
| Recettes absentes/lentes | API TheMealDB inaccessible ou lente | Vérifier réseau, logs et cache | Recettes disponibles ou erreur gérée |
| Données nutritionnelles absentes | Clé USDA absente ou API indisponible | Contrôler `USDA_API_KEY` et la réponse externe | Données affichées ou message explicite |
| Fonction planifier/préparer absente | Version de `planner.py` sans intégration | Vérifier les routes et fichiers de la version déployée | Bouton et parcours testés après intégration |
| Une modification n'apparaît pas | Ancien serveur ou mauvais dossier | Arrêter Uvicorn, relancer depuis le dossier de `main.py` | Nouveau comportement visible |

### Limites et risques à connaître

- **Sécurité CORS :** `main.py` configure `allow_origins=["*"]` ; à restreindre aux origines autorisées avant une exposition publique.
- **Gestion du schéma :** pas de dispositif général de migrations versionnées identifié dans les fichiers de démarrage inspectés.
- **Dépendances :** le `requirements.txt` annoncé dans le README manque dans cette archive ; rendre l'installation reproductible est une amélioration prioritaire.
- **Données externes :** les recettes, traductions et données USDA peuvent être incomplètes, lentes ou indisponibles.
- **Documentation OpenAPI :** le code ajoute un schéma Bearer de manière globale ; vérifier que cette représentation correspond aux protections effectives des routes.
- **Archive :** elle contient un environnement `venv/` ; celui-ci ne doit pas servir de source de vérité des dépendances ni être versionné.

## 8. Références, maintenance et responsabilités

| Référence | Emplacement | Quand la mettre à jour |
|---|---|---|
| Code de démarrage et routeurs | `main.py`, `app/routers/`, `app/web/pages/` | Ajout ou modification de route |
| Schéma des données | `app/db/models.py`, migrations éventuelles | Changement de table/colonne |
| Configuration | `app/db/database.py`, exemple `.env` sans secrets | Changement de fournisseur ou de variable |
| Installation | `README.md`, manifeste de dépendances | Mise à jour Python/paquets |
| Décisions d'architecture (ADR) | Dossier `docs/adr/` **à créer si adopté** | Décision technique structurante |
| Cette documentation | `docs/Documentation_Technique_Smart_Fridge_V2.md` **emplacement conseillé** | Chaque livraison modifiant un parcours, une route, un modèle ou une limite |

**Responsabilité proposée :** le développeur qui modifie un composant met à jour sa section et fait relire le changement par son binôme avant fusion/livraison. Ce processus est une **recommandation**, pas une règle actuellement vérifiée dans le dépôt.

**Critère de validité :** les commandes d'installation fonctionnent sur une machine propre ; les scénarios de la section 5 passent ; les limites de la section 7 sont réévaluées ; les chemins cités correspondent à la version livrée.

## 9. Autoévaluation selon les six critères demandés

| Critère | Réponse apportée par cette documentation |
|---|---|
| **Adaptée au lecteur** | Public et prérequis explicités dès l'introduction |
| **Utile** | Tâches concrètes : installer, tester, diagnostiquer, modifier |
| **Exacte** | Code et routes rattachés à une archive identifiée ; limites et absences signalées |
| **Actionnable** | Commandes, fichiers à modifier et procédures pas à pas |
| **Vérifiable** | Résultat attendu indiqué pour chaque parcours et problème courant |
| **Maintenable** | Sources de vérité, occasions de mise à jour et responsabilité proposées |

---

*Cette documentation décrit la version inspectée et distingue les comportements visibles dans le code des opérations nécessitant un test d'exécution. Elle doit être revue après chaque évolution significative du projet.*
