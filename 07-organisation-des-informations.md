# Organisation des informations à l’écran

Étape 7 — En cours. Résumé, blocs détaillés et liste des créneaux validés le 7 septembre 2026.

## Résumé du créneau sélectionné — Validé

Présenter les informations dans cet ordre :

1. Date et horaires du créneau, par exemple lundi 7 septembre, 15 h–16 h.
2. Activité choisie, par exemple course tranquille.
3. Note globale, en premier parmi les notes, avec son appréciation.
4. Notes météo et qualité de l’air, visibles séparément juste dessous.
5. Alertes éventuelles, directement visibles.
6. Deux explications principales pour comprendre la note, selon les règles du score.

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

## À définir et valider

- Présentation des états de chargement, d’indisponibilité et d’actualisation.
