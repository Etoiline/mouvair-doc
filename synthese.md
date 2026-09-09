# Synthèse du projet Mouv’Air

État du 9 septembre 2026, fondé sur les trois dépôts locaux et les vérifications décrites en fin de document.

## Objectif et parcours disponible

Mouv’Air aide à choisir un créneau d’une heure pour pratiquer une activité dehors selon la météo et la qualité de l’air. L’application fournit un indicateur de conditions extérieures ; elle ne formule pas d’avis médical et ne personnalise pas ses recommandations selon la santé de l’utilisateur.

Le parcours principal est implémenté :

1. rechercher une ville et la distinguer par sa région et son pays ;
2. choisir entre course tranquille, course intense, vélo et marche ;
3. consulter 24 créneaux futurs d’une heure, exprimés dans le fuseau de la ville ;
4. examiner la note globale, la note météo, la note qualité de l’air et leurs détails ;
5. consulter séparément les concentrations prévues de six pollens ;
6. accéder au meilleur créneau complet pour l’activité choisie.

Les créneaux sont regroupés par date locale. En cas d’égalité, le premier dans le temps est retenu. Une meilleure option défavorable est signalée et aucune recommandation n’est produite lorsque toutes les notes globales sont incomplètes ou que les données sont trop anciennes.

## Architecture

- **Frontend** : React, TypeScript en mode strict et Vite.
- **Backend** : FastAPI, Pydantic et httpx.
- **Fournisseur** : Open-Meteo pour la recherche de villes, la météo, la qualité de l’air et les pollens.
- **Stockage** : aucun compte ni base de données ; le cache des prévisions réside en mémoire dans l’API.

L’API expose `GET /health`, `GET /api/locations` et `GET /api/forecast`. La récupération des données, l’assemblage des créneaux, le cache et le calcul des scores sont séparés dans le backend. Les appels météo et qualité de l’air sont effectués en parallèle.

Le contrat d’un score sportif contient les métadonnées `sport` et `label`, puis trois sections imbriquées :

- `weather` pour la note météo, ses trois facteurs, ses explications et ses alertes ;
- `air` pour la note air, l’AQI européen, sa catégorie et les recommandations associées ;
- `overall` pour la note globale et son appréciation.

Les anciens doublons plats du score ont été retirés du backend et des types frontend.

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

Les catégories de l’AQI et les recommandations destinées à la population générale et aux personnes sensibles restent visibles à côté de la note air. Les pollens d’aulne, bouleau, graminées, armoise, olivier et ambroisie sont présentés en grains/m³ et n’entrent dans aucune note.

## Actualisation et résilience

Le backend réutilise pendant 15 minutes une prévision complète mise en cache pour une position. Le cache accepte au plus 256 positions, disparaît au redémarrage du serveur et regroupe les demandes simultanées identiques en une seule récupération fournisseur.

Après l’échec d’une actualisation, l’API recompose toujours la fenêtre des 24 départs futurs :

- jusqu’à une heure d’âge, les dernières notes restent utilisables avec l’état `refresh_failed` ;
- au-delà d’une heure, les données restent consultables avec l’état `stale` et la recommandation du meilleur créneau est suspendue ;
- les heures qui ne sont plus couvertes restent visibles avec des données incomplètes.

Le frontend permet l’actualisation manuelle, vérifie les données toutes les 15 minutes et au début d’un nouveau créneau, puis rattrape un contrôle manqué au retour sur un onglet visible. Les résultats, l’activité et le créneau sélectionné restent affichés pendant une actualisation lorsque cela est possible. Le premier chargement, son échec avec réessai, l’échec d’une actualisation existante et les données anciennes possèdent des états distincts.

## État d’alignement

| Domaine documenté | État dans la version locale | Vérification principale |
|---|---|---|
| Trois notes, courbes, pondérations, plafonds et alertes | Implémenté | Tests du moteur de score |
| 24 créneaux locaux d’une heure et agrégation des deux bornes | Implémenté | Tests d’assemblage, fuseaux et changements d’heure |
| AQI, recommandations et six pollens | Implémenté | Tests backend et contrat typé frontend |
| Cache, données anciennes et requêtes simultanées | Implémenté | Tests du service de prévision |
| Résumé, détails, liste par date et meilleur créneau | Implémenté | Compilation TypeScript et build de production |
| Chargement, actualisation, réessai et états incomplets | Implémenté | Compilation TypeScript et build de production |

## Vérifications effectuées

- Tests backend : 80 sur 80 réussis le 9 septembre 2026.
- Contrôle Ruff du backend : réussi.
- Compilation TypeScript et build de production du frontend : réussis.
- Contrat sérialisé du score : vérifié sans les anciens champs plats.

Le frontend ne possède pas encore de suite automatisée de tests de composants ou de parcours de bout en bout. Aucun pipeline d’intégration continue n’est présent dans les trois dépôts.

## Limites et prochaines vérifications utiles

1. Ajouter des tests frontend ciblés sur la sélection, le meilleur créneau, l’actualisation et les états d’erreur.
2. Vérifier manuellement le rendu responsive, le clavier et les annonces des technologies d’assistance sur les principaux navigateurs.
3. Mettre en place une intégration continue pour exécuter les 80 tests backend, Ruff et le build frontend à chaque modification.
4. Prévoir un cache partagé, une supervision et des limites de requêtes si l’API est déployée avec plusieurs processus ou plusieurs instances.
5. Continuer à présenter les données Open-Meteo comme des prévisions de modèle, dont la précision ne correspond pas à une mesure exacte au niveau de la rue.
