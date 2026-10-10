# EventCheck — Version 0.5 — base de données embarquée

Application Angular + API Java Spring Boot + H2 embarquée et persistante. PostgreSQL reste facultatif via un profil séparé. Cette livraison conserve la préparation et l’import CSV/Excel et ajoute un prototype de caméra/OCR local, champs modifiables et recherche assistée, sans confirmation automatique. La démonstration contient aussi dix invités fictifs, trois agents et un superviseur.

La version complète (avec le JAR prêt à lancer) est disponible dans [les versions GitHub](https://github.com/htinou/eventcheck/releases). Les sources seules sont dans `EventCheck_sources_0.5.0.zip`.

Pour passer de 0.4 à 0.5 : arrêter l’application, sauvegarder le dossier utilisateur EventCheck, puis lancer la nouvelle version. Le dossier de données et les accès existants sont conservés ; le schéma de base ne change pas.

## Nouveautés de la version 0.5

- Ajout et modification d’un invité en brouillon, sans remplacer les autres participants.
- Modèle CSV téléchargeable depuis « Les invités », également fourni dans `docs/modele_invites.csv` : remplacer les deux exemples fictifs.
- OCR : libellés sans deux-points et sur des lignes séparées, noms MRZ TD1, progression, choix de photo et erreurs explicites. Sur téléphone via HTTP, utiliser une photo ; la caméra en direct exige HTTPS. HEIC dépend du navigateur ; JPEG ou PNG sont recommandés.

## Ce qui fonctionne

- Création et modification des événements en brouillon, date, fuseau et fin de conservation prévue.
- Affectation des comptes existants, rôles agent et superviseur.
- Import CSV UTF-8/XLSX : aperçu, choix des colonnes, erreurs bloquantes, confirmation des homonymes, remplacement atomique.
- Ouverture explicite : informations, équipe et liste ensuite figées ; révisions protégeant des modifications concurrentes.
- Prototype caméra/OCR local (aucun prestataire, aucune photo envoyée), nom et prénom corrigibles avant recherche.
- Connexion individuelle et déconnexion avec session serveur et protection CSRF.
- Choix d’un événement affecté et de l’entrée active sur le téléphone.
- Recherche insensible aux accents et à la casse, homonymes distincts.
- Fiche invité, confirmation humaine, heure serveur, entrée et agent enregistrés.
- Une seule arrivée par invité : contrainte SQL et traitement des tentatives concurrentes.
- Reprise d’une confirmation avec la même clé si son résultat est inconnu.
- Dashboard et listes partagés, rafraîchissement toutes les secondes sur écran actif.
- Mise en pause des confirmations si la connexion est perdue.
- Clôture par le superviseur, sans clôture automatique à 100 %.
- Interface responsive ; les vues de contrôle, supervision et confirmation sont illustrées dans `docs/screenshots/`.

Le scan est un prototype à qualifier sur de vrais téléphones, pas une lecture universelle de pièces d’identité. Aucune photo n’est envoyée ni stockée. L’export et la purge planifiée restent à développer. Voir `docs/OCR_PROTOTYPE.md` pour les limites et la recette. Pas de mode hors connexion, d’annulation de présence ni de réouverture de l’accueil. Pas encore de service worker/PWA installable : cette livraison est une application web responsive.

## Préparer un événement

Avec `supervision`, choisir « Créer un événement », indiquer son nom et sa date (Europe/Paris), puis préparer les informations et l’équipe. Les trois entrées sont créées automatiquement. Importer le fichier et vérifier les colonnes du nom, du prénom et de la référence facultative. Après une modification des colonnes, cliquer « Vérifier les colonnes choisies » pour recalculer l’aperçu. Les références répétées et les champs invalides bloquent l’import ; les homonymes restent des personnes distinctes et demandent une confirmation explicite.

Les boutons « Ajouter un invité + » et « Modifier » permettent de gérer un participant en brouillon. Nom et prénom sont obligatoires, la référence facultative est unique ; les homonymes demandent une confirmation. Après ouverture, ces actions sont figées.

L’import remplace toute la liste, uniquement en brouillon, après revalidation côté serveur. Formats : CSV UTF-8, virgule ou point-virgule, avec en-tête ; XLSX, première feuille, valeurs sans formules. Limites : 100 invités, 2 Mo, 30 colonnes ; nom, prénom et référence limités à 100 caractères. Les archives XLSX sont aussi limitées à 16 Mo décompressés et 256 entrées. Aucun aperçu ne modifie la base.

Enregistrer les informations et l’équipe, importer les invités, puis ouvrir l’accueil. L’ouverture fige la préparation. Si une autre session a modifié le brouillon, la demande échoue avec `REVISION_CONFLICT` : revenir aux événements et rouvrir la préparation avant de reprendre. Les comptes sont préprovisionnés ; cette étape ne crée pas de comptes. La date de fin de conservation est enregistrée mais le traitement automatique de suppression reste à développer.

Exemple : `docs/exemple_invites.csv`.

## Démarrage recommandé : sans base à installer

Prérequis unique : **Java 17 ou ultérieur**. Ni PostgreSQL, ni Docker, ni Python, ni Node ne sont nécessaires pour utiliser le programme livré.

1. Décompresser l’archive.
2. Windows : lancer `Demarrer_EventCheck.bat`. macOS : lancer `Demarrer_EventCheck.command` (ou `sh Demarrer_EventCheck.command` dans Terminal). Linux : `sh Demarrer_EventCheck.sh`.
3. Ouvrir **http://localhost:8080**.
4. Utiliser `supervision`, `accueil.a`, `accueil.b` ou `accueil.c`. Le mot de passe initial est la valeur `demo.password` du fichier `EventCheck/acces-local.properties` dans votre dossier utilisateur. Son emplacement est affiché au démarrage.

Alternative universelle, depuis le dossier décompressé :

```bash
java -jar release/eventcheck.jar
```

Le premier démarrage crée automatiquement une base H2, son schéma, les comptes locaux et les invités fictifs. La base **sur disque**, `EventCheck/eventcheck.mv.db`, conserve les événements, listes, affectations et présences lors des redémarrages. Les mots de passe générés sont conservés dans `acces-local.properties` et ne sont pas inclus dans l’archive. Les lanceurs utilisent explicitement le mode embarqué et ignorent les anciens paramètres PostgreSQL.

Pour déplacer ou sauvegarder les données : arrêter EventCheck proprement (Ctrl+C), puis copier **tout le dossier utilisateur EventCheck**, avec la base et le fichier d’accès. Ne pas copier la base pendant que le programme écrit dedans. Pour utiliser un autre dossier, définir `EVENTCHECK_DATA_DIR` avant le démarrage. Ne pas supprimer ou remplacer le fichier d’accès en espérant réinitialiser les comptes ; une base existante conserve ses identifiants.

Une seule instance d’EventCheck ouvre cette base. Plusieurs agents travaillent dans des navigateurs reliés à cette même instance, pas avec une base locale par téléphone. Le mode local écoute seulement sur `127.0.0.1`. Pour trois téléphones, prévoir un serveur accessible sur le réseau et un accès HTTPS ; définir `BIND_ADDRESS=0.0.0.0` pour l’écoute réseau et `COOKIE_SECURE=true` derrière HTTPS. La caméra sur téléphone requiert HTTPS. L’application ne fournit pas de certificat TLS ni de déploiement réseau automatique.

Les données PostgreSQL d’une ancienne version ne sont pas converties automatiquement en H2. Le mode embarqué crée sa propre base ; le profil PostgreSQL permet toujours de retrouver une ancienne base, sans la modifier par une migration H2.

## Démarrage facultatif avec PostgreSQL et Docker

Prérequis : Docker avec Docker Compose et Python 3. Les ports locaux 8080, 5432 et 5433 doivent être libres.

Depuis ce dossier :

```bash
python3 scripts/init_demo.py
docker compose --env-file .env.local up --build
```

Ouvrir **http://localhost:8080**. La première construction télécharge les dépendances Java/Angular. PostgreSQL attend d’être prêt avant le démarrage de l’application. La migration SQL est exécutée par Flyway.

Comptes disponibles : `accueil.a`, `accueil.b`, `accueil.c`, `supervision`. Le mot de passe commun de démonstration est la valeur `DEMO_PASSWORD` générée dans `.env.local`. Ce fichier n’est pas dans l’archive et est ignoré par Git. Les mots de passe sont hachés en base. Ouvrir trois profils de navigateur ou trois fenêtres privées indépendantes pour simuler les trois agents ; des onglets dans le même profil partagent la même session.

Le profil `demo` crée les données uniquement si l’événement fictif n’existe pas. Les arrivées restent conservées après redémarrage ; une clôture reste effective. Ne pas remplacer les mots de passe de `.env.local` après initialisation en pensant modifier automatiquement les comptes déjà présents. Pour recommencer un test, utiliser une nouvelle base de démonstration. Ne pas supprimer un volume contenant des données réelles.

L’exposition réseau est limitée à cette machine dans `compose.yml`. Pour un pilote sur trois téléphones, prévoir un hébergement HTTPS accessible aux appareils, mettre `COOKIE_SECURE=true`, et gérer l’accès et les comptes avant de saisir des données réelles. Le stockage serveur des sessions convient à une seule instance API. Le conteneur fourni est un démarrage de démonstration locale, pas un déploiement de production complet.

## Démarrage sans construire l’image de l’application

L’archive contient également `release/eventcheck.jar`, qui embarque l’interface compilée. Prérequis : Java 17 ou ultérieur, PostgreSQL 17 et Python 3. Pour démarrer uniquement la base avec Docker :

```bash
python3 scripts/init_demo.py
docker compose --env-file .env.local up -d --wait db
SPRING_PROFILES_ACTIVE=postgres,demo python3 scripts/run_jar.py
```

Ouvrir http://localhost:8080. `run_jar.py` lit les secrets locaux sans les afficher, définit le profil de démonstration et lance le JAR. Avec une base existante dédiée, définir `DB_URL`, `DB_USER` et `DB_PASSWORD` dans l’environnement ; le script respecte ces valeurs. Ne pas pointer la démonstration vers une base contenant d’autres applications.

## Développement depuis les sources

Prérequis pour construire les sources : Java 17, Maven 3.9.11 ou ultérieur, Node.js 24 et npm. PostgreSQL 17 est facultatif. Les dépendances Angular sont verrouillées dans `frontend/package-lock.json` ; les dépendances Java sont gérées par Spring Boot 3.5.16.

```bash
cd frontend
npm ci
npm run build
```

```bash
cd backend
mvn -DskipTests package
```

Pour servir l’interface avec Angular, démarrer `npm start` dans `frontend/` et utiliser http://localhost:4200. Le proxy envoie `/api/**` vers l’API sur 8080 ; aucun CORS permissif n’est nécessaire.

Pour lancer l’API seule, définir les variables `DB_URL`, `DB_USER`, `DB_PASSWORD`, `DEMO_PASSWORD`, `SPRING_PROFILES_ACTIVE=demo` et `COOKIE_SECURE=false` en développement HTTP local, puis exécuter `mvn spring-boot:run` dans `backend/`. Pour construire un JAR avec l’interface depuis les sources, utiliser `python3 scripts/build_release.py` à la racine. Le Dockerfile effectue aussi cette compilation complète.

## Tests de backend sur PostgreSQL natif

Prérequis : Maven et la base dédiée `eventcheck_test`. Les tests manipulent uniquement leurs fixtures identifiées, mais doivent toujours être exécutés dans une base de test isolée.

```bash
python3 scripts/init_demo.py
docker compose --env-file .env.local --profile test up -d --wait testdb
TEST_PROFILE=postgres python3 scripts/test_backend.py
```

Le script active `PG_NATIVE=true`. Les 16 tests couvrent les droits, le CSRF, les sessions, l’unicité SQL, la reprise idempotente, l’isolation entre événements, les homonymes, la clôture et trois scénarios spécifiques à PostgreSQL natif :

1. Trois validations concurrentes pour un invité, répétées 100 fois.
2. Trois reprises simultanées avec la même clé.
3. Clôture qui attend une validation engagée, puis refuse la suivante.

Les trois tests de concurrence sont volontairement ignorés si `PG_NATIVE=true` n’est pas défini. Un résultat avec trois tests ignorés n’est donc pas une validation des verrous natifs. Le workflow `.github/workflows/verify.yml` fournit PostgreSQL 17 et active ces tests ; il doit être exécuté après mise en dépôt.

## Tests de navigateur

Sur une démonstration démarrée et une base de données fictive uniquement :

```bash
cd frontend
npm ci
npx playwright install chromium
npm run e2e
```

Définir `DEMO_PASSWORD` dans l’environnement, avec la même valeur que dans `.env.local`. Par défaut, les tests utilisent http://127.0.0.1:4200 ; définir `E2E_BASE_URL=http://127.0.0.1:8080` pour le JAR ou Docker. Six scénarios sont exécutés en viewport ordinateur et mobile Chromium : recherche/homonymes, droits de supervision, coupure réseau, partage d’une arrivée entre deux sessions déconnexion et réponse perdue après enregistrement. Les scénarios de partage et de reprise enregistrent les arrivées d’Aarav Singh, Lucas Moreau, Camille Roy et Thomas Bernard dans la base de démonstration.

Le viewport mobile Chromium n’est pas un test Safari iPhone. Les essais iPhone Safari et Android Chrome sur appareils réels restent nécessaires avant un pilote.

## API livrée

| Route | Usage |
| --- | --- |
| GET `/api/v1/auth/csrf` | Jeton CSRF à envoyer sous `X-CSRF-TOKEN` pour les mutations |
| POST `/api/v1/auth/login` | Corps `{login,password}` ; session renouvelée, puis récupérer un nouveau jeton |
| POST `/api/v1/auth/logout` | Invalidation de session |
| GET `/api/v1/me` | Compte connecté |
| GET `/api/v1/events` | Événements affectés au compte |
| GET `/api/v1/events/{id}/snapshot` | Liste, arrivées, compteurs, rôle et entrées |
| POST `/api/v1/events/{id}/checkins` | Corps `{guestId,entranceId}`, en-tête `Idempotency-Key` UUID |
| POST `/api/v1/events/{id}/close` | Clôture réservée au superviseur |

La validation renvoie 201 pour une arrivée créée, 200 pour une reprise reconnue et 409 pour un invité déjà enregistré par une autre tentative. L’agent et l’heure viennent du serveur. Les réponses d’erreur ne contiennent ni document ni trace technique.

## Structure

```text
backend/       API, sécurité, migration Flyway et tests d’intégration
frontend/      Interface Angular, verrou npm et tests Playwright
scripts/       Configuration locale, construction et tests
docs/          Cahier des charges, état de livraison et captures
release/       JAR prêt à lancer, interface embarquée
compose.yml    PostgreSQL et démonstration locale
Dockerfile     Construction complète Angular + Java
```

## Limites de cette livraison

Lire `docs/VALIDATION.md` pour les vérifications réellement exécutées. La recette native de concurrence, la mesure des objectifs de performance et les essais sur téléphones réels ne doivent pas être considérés comme acquis simplement parce que le code des tests est fourni. La suppression planifiée, la durée de conservation effectivement appliquée, les sauvegardes et les procédures d’exploitation restent à compléter avant données réelles.

## Tests navigateur et limitation de connexion

La suite comprend davantage de connexions que le quota anti-bruteforce de 20 tentatives par IP sur dix minutes. Exécuter séparément `npm run e2e -- --project=desktop` puis `npm run e2e -- --project=mobile`, en redémarrant l’API entre les projets (pas la base). Le workflow CI fait ce redémarrage. Ne pas relever le quota de production pour faire passer les tests. Le corpus de 30 images est exécuté une seule fois sur le projet desktop ; la lecture, la correction et l’arrêt de caméra sont aussi testés dans le viewport mobile.

Tests embarqués, sans serveur de base : `cd backend` puis `mvn test`. Les profils et URL de test sont distincts de la base utilisateur.
