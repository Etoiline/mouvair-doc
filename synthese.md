# Synthèse du projet Mouv’Air

Analyse du 6 septembre 2026, fondée sur le code local et les vérifications disponibles.

## Objectif et fonctionnalités

Mouv’Air aide à choisir un créneau pour pratiquer une activité dehors selon la météo et la qualité de l’air.

Dans le prototype analysé, l’utilisateur recherche une ville, choisit une activité et consulte les scores horaires, le meilleur créneau disponible, quatre facteurs et les raisons qui dégradent les conditions. La version cible documentée présente séparément une note météo, une note qualité de l’air et une note globale, ainsi que les informations sur les pollens.

Quatre profils sont implémentés : course tranquille, course intense, vélo et marche, tous présentés pour une durée de 1 h.

## Architecture

- Frontend : React, TypeScript en mode strict et Vite.
- Backend : FastAPI, Pydantic et httpx.
- Sources : Open-Meteo pour la recherche de villes, la météo et la qualité de l’air.
- Stockage : aucune base de données ni gestion de comptes dans la version examinée.

L’API expose `GET /health`, `GET /api/locations` et `GET /api/forecast`.

La récupération des données, les modèles et les règles de calcul sont séparés. Les appels météo et qualité de l’air sont effectués en parallèle.

## Calcul du score

Le moteur actuel combine quatre facteurs : qualité de l’air, température ressentie, précipitations et vent. Chaque activité possède ses pondérations et ses courbes de tolérance.

Des plafonds limitent la note dans certaines conditions défavorables, notamment en cas d’orage ou de qualité de l’air dégradée. Le score est accompagné de sous-notes et d’explications.

Les règles produit cibles sont désormais documentées : trois notes distinctes, pondérations par activité, courbes, plafonds, alertes, qualité de l’air, pollens et traitement d’une sortie réelle de 1 h. Le code analysé n’applique pas encore l’ensemble de cette spécification et utilise une seule entrée horaire. L’application présente le résultat comme un indicateur de conditions extérieures et non comme un avis médical.

## Points forts

- Parcours principal implémenté avec une architecture simple et lisible.
- Règles métier centralisées et ajustables indépendamment de l’interface.
- Validation des coordonnées et des réponses de l’API par des modèles typés.
- Délais d’attente et traitement des erreurs HTTP du fournisseur côté backend.
- Styles prévus pour les petits écrans.

## Limites et priorités

1. **Fiabiliser la fenêtre horaire.** Le backend supprime les heures dont une donnée manque puis conserve jusqu’à 24 entrées. Les heures peuvent être discontinues ou couvrir plus de 24 heures. Le statut « incomplet » est défini mais jamais produit.
2. **Actualiser les prévisions.** Une page laissée ouverte conserve ses données et peut recommander un créneau passé.
3. **Adapter le calcul aux règles validées.** Séparer les trois notes, appliquer les nouvelles pondérations et évaluer chaque créneau sur une sortie réelle de 1 h.
4. **Renforcer la gestion des réponses externes.** Des champs absents ou des séries incohérentes peuvent provoquer des erreurs internes. Aucun cache ni mécanisme de limitation des requêtes n’est présent.
5. **Étendre la validation.** Ajouter des tests des endpoints, de l’assemblage des prévisions et du parcours utilisateur. Aucun pipeline CI n’a été détecté lors de l’analyse.

## Vérifications effectuées

- Compilation TypeScript et build frontend : réussis.
- Tests backend : 6 sur 6 réussis.
- Contrôle Ruff : réussi.

Le rendu dans un navigateur et les appels réels à Open-Meteo n’ont pas été vérifiés. Aucun dépôt Git n’a été détecté dans les dossiers du frontend et du backend au moment de l’analyse.

## Évaluation

Le projet constitue une base de MVP cohérente. Les prochaines évolutions devraient porter d’abord sur la fiabilité des horaires, les données manquantes et l’actualisation, puis sur les tests d’intégration et la documentation du score.
