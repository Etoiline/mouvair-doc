# Organisation des informations à l’écran

Étape 7 — Validée le 7 septembre 2026.

## Résumé du créneau sélectionné — Validé

Présenter les informations dans cet ordre :

1. Date et horaires du créneau, par exemple lundi 7 septembre, 15 h–16 h.
2. Activité choisie, par exemple course tranquille.
3. Note globale, en premier parmi les notes, avec son appréciation.
4. Notes météo et qualité de l’air, visibles séparément juste dessous.
5. Alertes éventuelles, directement visibles.
6. Jusqu’à deux explications météo principales pour comprendre la note, selon les règles du score.

Dans « À retenir », afficher uniquement les deux premières explications météo fournies par l’API. Les informations sur la qualité de l’air sont présentées dans sa note dédiée et dans l’onglet de détail.

Les conditions détaillées et les pollens apparaissent ensuite.

Les règles de calcul, d’explication et de disponibilité restent celles des [règles du score](05-regles-du-score.md) et des [données et horaires](06-donnees-et-horaires.md).

## Conditions détaillées — Blocs validés

Présenter trois blocs sous le résumé :

- **Météo** : température ressentie en °C, pluie en mm sur l’heure, vent moyen et rafales en km/h. Afficher la mention « Conditions retenues pour ce créneau de 1 h ».
- **Qualité de l’air** : AQI européen, catégorie correspondante et recommandations pour la population générale et les personnes sensibles. Présenter l’AQI comme un indice, sans symbole %.
- **Pollens prévus** : concentration de chaque pollen en grains/m³, avec « Indisponible » si nécessaire. Rappeler que les pollens n’entrent pas dans les notes.

## Liste des créneaux et meilleur créneau — Validés

- Présenter les 24 créneaux dans l’ordre chronologique, regroupés par date.
- Pour chacun, afficher les horaires, la note globale, son appréciation et le signalement des alertes éventuelles.
- Distinguer clairement le créneau sélectionné ; un clic affiche son détail.
- Identifier le meilleur par la mention « Meilleur créneau ». Un bouton permet d’y accéder directement.
- Conserver les créneaux incomplets avec la mention « Données incomplètes », sans note globale.
- Si même le meilleur créneau est défavorable, un message le précise.
- Si la recommandation est suspendue, notamment pour des données trop anciennes, désactiver le bouton avec une explication.

## Chargement, actualisation et indisponibilité — Validés

- **Premier chargement** : afficher « Chargement des prévisions pour Lyon… ». La liste des créneaux apparaît lorsque les données sont prêtes.
- **Actualisation en arrière-plan** : conserver les résultats visibles et remplacer temporairement le libellé du bouton par « Actualisation… ».
- **Premier chargement impossible** : afficher « Impossible de charger les prévisions » avec un bouton « Réessayer ». La recherche de ville reste accessible.
- **Échec avec des données déjà disponibles** : conserver les résultats et afficher « Actualisation impossible — dernières données récupérées à 14 h 20 », avec l’heure réelle de la dernière récupération réussie.
- **Données vieilles de plus d’une heure** : afficher « Données anciennes » près des notes. Les résultats restent consultables à titre indicatif, mais le meilleur créneau n’est plus proposé.
- **Créneau incomplet** : afficher « Données incomplètes ». Chaque note disponible reste visible, la note manquante affiche « Indisponible » et la note globale est indisponible.
- **Pollens indisponibles** : afficher le message prévu uniquement dans le bloc pollen, sans masquer les notes météo et air.
- **Ville introuvable** : afficher « Aucune ville trouvée. Vérifiez l’orthographe ou précisez le pays. »

Les alertes météo déjà détectées restent visibles pendant une erreur d’actualisation et lorsqu’une note est indisponible.
