# Synthèse du projet Mouv’Air

État du 31 octobre 2026, fondé sur les trois dépôts locaux et les vérifications décrites en fin de document. Ce relevé porte sur l’implémentation locale ; les exécutions GitHub et le déploiement en production ne sont pas vérifiés ici.

## Objectif et parcours disponible

Mouv’Air aide à choisir un créneau d’une heure pour pratiquer une activité dehors selon la météo et la qualité de l’air. L’application fournit un indicateur de conditions extérieures ; elle ne formule pas d’avis médical et ne personnalise pas ses recommandations selon la santé de l’utilisateur.

Le parcours principal est implémenté :

1. rechercher une ville, automatiquement à partir de trois caractères ou par validation du formulaire, et la distinguer par sa région et son pays ;
2. choisir entre course tranquille, course intense, vélo et marche ;
3. consulter 24 créneaux futurs d’une heure, exprimés dans le fuseau de la ville ;
4. examiner la note globale, la note météo, la note qualité de l’air et leurs détails ;
5. consulter séparément les concentrations prévues de six pollens ;
6. accéder au meilleur créneau complet pour l’activité choisie.

Les créneaux sont regroupés par date locale. En cas d’égalité, le premier dans le temps est retenu. Une meilleure option défavorable est signalée et aucune recommandation n’est produite lorsque toutes les notes globales sont incomplètes ou que les données sont trop anciennes.

## Architecture

- **Frontend** : React, TypeScript en mode strict et Vite.
- **Backend** : FastAPI, Pydantic, httpx et redis-py en mode asynchrone.
- **Fournisseur** : Open-Meteo pour la recherche de villes, la météo, la qualité de l’air et les pollens ; les prévisions de qualité de l’air proviennent de CAMS ENSEMBLE via Open-Meteo.
- **Stockage** : aucun compte ni base de données métier ; Redis partage les caches, les verrous et
  les compteurs de quota, avec un cache mémoire L1 dans chaque processus.

L’API expose `GET /health`, `GET /api/locations` et `GET /api/forecast`. La récupération des données, l’assemblage des créneaux, le cache et le calcul des scores sont séparés dans le backend. Les appels météo et qualité de l’air sont effectués en parallèle.

Le contrat d’un score sportif contient les métadonnées `sport` et `label`, puis trois sections imbriquées :

- `weather` pour la note météo, ses trois facteurs, ses explications et ses alertes ;
- `air` pour la note air, l’AQI européen, sa catégorie et les recommandations associées ;
- `overall` pour la note globale et son appréciation.

Les anciens doublons plats du score ont été retirés du backend et des types frontend.

Le frontend sépare l’assemblage de l’écran dans `src/app`, les fonctionnalités `locations` et
`forecast` dans `src/features`, puis le client HTTP et les utilitaires communs dans `src/shared`.
Les composants, hooks, appels API, types et fonctions sans état sont regroupés par fonctionnalité.
Les tests restent à côté du code vérifié ; `src/test` contient leur préparation et les fixtures.

Les feuilles CSS sont placées à côté des composants. Les couleurs, surfaces, bordures et espacements communs,
avec les variantes sombres, sont centralisés dans `src/app/tokens.css`. Les styles du cadre et de la
recherche utilisent leurs propres classes. Les guides `mouvair-front/docs/architecture.md` et
`mouvair-front/docs/configuration.md` détaillent ces conventions.

Le backend distingue le transport Open-Meteo (`open_meteo.py`), l’assemblage pur des données et des
horaires (`forecast_slots.py`), le calcul (`scoring.py`), les services et leurs caches. Les erreurs
de disponibilité partagent un contrat commun dans `errors.py` ; `main.py` les traduit en réponses
HTTP. Les journaux sont regroupés dans `diagnostics.py`.

## Présentation de l’interface

Le résumé présente la note globale, puis les notes météo et qualité de l’air. « À retenir » affiche
au maximum les deux premières explications météo ; la qualité de l’air conserve sa note et son
onglet de détail. Les alertes météo restent dans une section distincte.

Le graphique « Heure par heure », sous-titré « Note globale sur 100 », précède les détails. Il
regroupe les 24 créneaux par date et permet de sélectionner une heure. Une carte réunit ensuite
les onglets météo, qualité de l’air et pollens. Les mesures météo sont associées à leurs sous-notes ;
l’AQI européen reste identifié séparément de la note air.

Sur mobile, la première sélection d’une ville compacte l’en-tête et la recherche. Ce mode reste
actif pendant les recherches suivantes. Les notes météo et air sont côte à côte, le graphique peut
défiler horizontalement et la hauteur des détails suit l’onglet actif. Les six pollens occupent
deux colonnes. Les thèmes clair et sombre disposent de styles adaptés.

Les mentions de source et d’usage restent en bas de la fenêtre lorsque le contenu est court,
puis suivent le contenu dans le défilement lorsque la page s’allonge.

## Créneaux et données

L’API construit 24 créneaux consécutifs à partir de la prochaine heure pleine dans le fuseau de la ville. Les timestamps portent leur décalage UTC, ce qui permet de représenter les fuseaux décalés de 30 ou 45 minutes ainsi que les heures répétées lors du passage à l’heure d’hiver. Chaque intervalle conserve une durée réelle d’une heure.

Les séries météo, air et pollen sont associées par timestamp. Pour une sortie, le calcul examine les deux bornes horaires : il retient la température ressentie la plus pénalisante pour l’activité, le vent moyen et l’AQI les plus élevés, les précipitations et rafales décrivant l’heure du créneau, ainsi que le maximum de chaque pollen. Les codes météo des deux bornes permettent de conserver les alertes détectées.

Une valeur manquante ou invalide n’est pas remplacée par zéro. Le créneau reste présent, la note concernée devient incomplète et les autres informations disponibles restent affichées.

## Calcul des notes

Le moteur applique les règles validées dans [les règles du score](05-regles-du-score.md) :

- la note météo combine température ressentie, précipitations, vent et rafales selon des courbes et des pondérations propres à chaque activité ;
- la note air est commune aux quatre activités et suit la conversion `max(0, 100 − AQI européen)` ;
- la note globale combine météo et air avec des poids propres à l’activité ;
- les plafonds progressifs limitent la compensation d’un facteur très dégradé ;
- les orages, précipitations verglaçantes et chutes de neige appliquent leurs alertes et plafonds dédiés ;
- les calculs conservent leurs décimales et l’affichage utilise un arrondi à l’entier, avec les demis vers le haut.

Les catégories de l’AQI et les recommandations destinées à la population générale et aux personnes sensibles restent consultables dans les détails de qualité de l’air. Les pollens d’aulne, bouleau, graminées, armoise, olivier et ambroisie sont présentés en grains/m³ et n’entrent dans aucune note.

L’intensité des explications météo tient compte des appréciations météo et globale affichées.
La formulation « pénalise fortement » est réservée à un facteur qui impose un plafond lorsque ces
deux appréciations sont dégradées ; une note correcte ou favorable conserve une formulation mesurée.

## Actualisation et résilience

Le backend réutilise pendant 15 minutes une prévision complète mise en cache pour une position.
Chaque processus conserve au plus 256 positions en mémoire et Redis fournit un second niveau partagé.
Les demandes simultanées identiques sont regroupées en une seule récupération fournisseur, y
compris entre instances grâce à un verrou distribué.

Le cache partagé et ses verrous utilisent les coordonnées demandées comme identité, tout en
conservant les coordonnées renvoyées par le fournisseur dans les données. L’espace de clés `v2`
isole les anciennes entrées. Les fuseaux, dates avec décalage UTC et durées réelles d’une heure
sont contrôlés à la lecture ; une entrée invalide est écartée. Pendant un rafraîchissement partagé,
l’attente distingue la nouvelle récupération de l’ancienne prévision encore présente dans Redis.

Le géocodage commence à trois caractères. Les variantes de casse et d’espacement partagent la même
entrée ; les résultats sont conservés 24 heures et les absences de résultat 5 minutes. Un cache local
borné permet encore de répondre aux recherches déjà connues pendant une panne Redis.

Après l’échec d’une actualisation, l’API recompose toujours la fenêtre des 24 départs futurs :

- jusqu’à une heure d’âge, les dernières notes restent utilisables avec l’état `refresh_failed` ;
- au-delà d’une heure, les données restent consultables avec l’état `stale` et la recommandation du meilleur créneau est suspendue ;
- les heures qui ne sont plus couvertes restent visibles avec des données incomplètes.

Le frontend permet l’actualisation manuelle, vérifie les données toutes les 15 minutes et au début d’un nouveau créneau. Les appels automatiques sont suspendus lorsque l’onglet est masqué ; un contrôle manqué ou des données trop anciennes déclenchent une vérification au retour. Les résultats, l’activité et le créneau sélectionné restent affichés pendant une actualisation lorsque cela est possible. L’activité et l’onglet de détail sont conservés lors d’un changement de ville. Le premier chargement, son échec avec réessai, l’échec d’une actualisation existante et les données anciennes possèdent des états distincts.

## Quotas Open-Meteo et fonctionnement dégradé

Les géocodages, appels météo et appels air partagent quatre fenêtres glissantes Redis. Une marge de
10 % fixe les seuils Mouv’Air à 540 appels par minute, 4 500 par heure, 9 000 par jour et 270 000 sur
31 jours. Un géocodage réserve une unité ; une prévision réserve atomiquement deux unités avant de
lancer météo et air. Un cache hit ne consomme pas le budget.

Chaque client est également limité séparément : 120 recherches de villes par minute et 1 200 par
heure, puis 40 demandes de prévisions par minute et 240 par heure. Son adresse n’est pas stockée en
clair et un en-tête de proxy n’est accepté que si le proxy direct est explicitement déclaré fiable.
Une limite client renvoie `429`. Un quota global indisponible sans donnée de repli renvoie `503`.
Ces réponses exposent le délai `Retry-After`. Une autre indisponibilité du fournisseur sans repli
renvoie `502`. Les réponses de limitation conservent les en-têtes CORS pour les origines autorisées,
afin que le navigateur puisse lire le message et le délai de reprise.

En production, la panne de Redis interdit par défaut les nouveaux appels Open-Meteo, mais les caches
mémoire et les anciennes prévisions restent utilisables. `/health` signale alors l’état `degraded`.
Un mode local doit être activé explicitement pour le développement mono-processus et ne constitue
pas une garantie multi-instance.

## Diagnostics et exécution

L’API émet des journaux JSON avec un identifiant interne par requête. Ils indiquent les durées HTTP,
les sources des réponses (fournisseur, mémoire ou Redis), les échecs, les motifs de repli, les
limitations et les changements de disponibilité Redis. Les événements applicatifs ne recopient
pas les recherches, les coordonnées, les adresses IP ni les messages bruts d’exception.

Docker Compose permet de lancer l’API et Redis ensemble. Le démarrage de l’API attend la santé de
Redis et une sonde contrôle ensuite `/health`. Les procédures locales et le déploiement prévu sur
Railway sont détaillés dans `mouvair-back/README.md` ; le déploiement frontend sur Vercel est décrit
dans `mouvair-front/README.md`.

## Vérifications automatisées

Les deux dépôts applicatifs possèdent un workflow `.github/workflows/checks.yml`, déclenché à chaque
push et pull request, avec un lancement manuel disponible sur la branche par défaut.

| Dépôt | Contrôles configurés |
|---|---|
| Frontend | Installation `pnpm install --frozen-lockfile`, puis `pnpm check` : Vitest, TypeScript et construction Vite. Node est fixé dans `.node-version` et pnpm dans `package.json`. |
| Backend | Ruff et suite Pytest complète, avec Redis 8.2 éphémère et `TEST_REDIS_URL` pour activer les tests d’intégration. |

Les workflows utilisent des actions fixées par SHA, des permissions de lecture, un délai maximal
de dix minutes et l’annulation de l’exécution précédente sur la même référence. Leurs contrôles
ne déploient pas l’application. Le détail de l’installation backend reste dans son README et son
workflow, où l’outillage est en cours de mise à jour dans la copie locale examinée.

## État d’alignement

| Domaine documenté | État dans la version locale | Vérification principale |
|---|---|---|
| Trois notes, courbes, pondérations, plafonds et alertes | Implémenté | Tests du moteur de score |
| 24 créneaux locaux d’une heure et agrégation des deux bornes | Implémenté | Tests d’assemblage, fuseaux et changements d’heure |
| AQI, recommandations et six pollens | Implémenté | Tests backend et contrat typé frontend |
| Caches partagés, données anciennes et requêtes simultanées | Implémenté | Tests des services de prévision et de géocodage |
| Quotas Open-Meteo et limites par client | Implémenté | Tests atomiques, concurrence et contrats HTTP |
| Résumé météo, trois onglets, graphique par date et meilleur créneau | Implémenté | Tests de parcours frontend et build de production |
| Chargement, recherche automatique, actualisation, réessai et états incomplets | Implémenté | Tests des hooks et des parcours frontend |
| Organisation par fonctionnalité, styles isolés et variables visuelles communes | Implémenté | Inspection des sources, tests et build frontend |
| Présentation mobile, thèmes clair et sombre et pied de page | Implémenté | Contrôles visuels ponctuels desktop et mobile |
| Contrat HTTP, CORS des limites et validation du cache partagé | Implémenté | Tests d’API, de cache et d’intégration Redis |
| Journaux JSON et contexte concurrent des requêtes | Implémenté | Tests des diagnostics backend |
| Intégration continue frontend et backend avec Redis | Configurée localement | Workflows présents ; résultats distants à confirmer sur GitHub |

## Vérifications effectuées

Les contrôles ci-dessous ont été effectués localement le 31 octobre 2026 :

- Backend : 228 tests réussis, dont 13 contre le Redis local, avec `TEST_REDIS_URL` sur la base de test 15. Ce total inclut les six tests du script de release présents dans les modifications locales d’outillage.
- Ruff : `ruff check .` réussi.
- Frontend : `pnpm check` réussi, soit 17 tests dans trois fichiers, puis vérification TypeScript et construction de production.
- Installation frontend : `pnpm install --frozen-lockfile` réussie ; syntaxe de son workflow validée avec actionlint.
- Contrat sérialisé du score : absence des anciens champs plats couverte par les tests backend.
- Docker Compose local : services API et Redis observés avec l’état `healthy`.

Les parcours et hooks frontend sont testés dans jsdom avec des API simulées. Des contrôles visuels
ponctuels ont couvert le desktop et le mobile dans les deux thèmes ; les tests automatisés dans un
navigateur réel restent à ajouter.

## Limites et prochaines vérifications utiles

1. Confirmer sur GitHub les résultats des workflows des deux dépôts après publication des branches concernées.
2. Compléter les contrôles du rendu responsive, du clavier et des annonces des technologies d’assistance sur les principaux navigateurs.
3. Ajouter quelques tests dans un navigateur réel pour les interactions, le défilement et les changements de taille de fenêtre.
4. Vérifier le déploiement complet Vercel–Railway, les origines CORS, les proxies de confiance et le comportement lors d’une panne Redis.
5. Exploiter les journaux existants avec une collecte, des tableaux de suivi et des alertes de production ; leur configuration n’a pas été vérifiée ici.
6. Maintenir l’alignement entre les modèles Pydantic et les types TypeScript, encore définis séparément.
7. Continuer à présenter les données Open-Meteo comme des prévisions de modèle, dont la précision ne correspond pas à une mesure exacte au niveau de la rue.
