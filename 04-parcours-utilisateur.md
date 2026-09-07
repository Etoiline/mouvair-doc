# Parcours utilisateur

Étape 4 — Validée le 6 septembre 2026.

Ce document décrit le parcours visé pour la première version, y compris les comportements restant à implémenter.

## 1. Rechercher une ville

L’utilisateur saisit un nom et lance la recherche. Il choisit ensuite parmi les résultats, accompagnés de leur région et pays pour distinguer les villes homonymes.

## 2. Choisir son activité

Les prévisions s’affichent avec « Course tranquille » sélectionnée par défaut. L’utilisateur peut passer à la course intense, au vélo ou à la marche. La durée de sortie de 1 h est indiquée.

## 3. Consulter les conditions

Le premier créneau disponible est sélectionné. L’écran présente sa note globale et son appréciation, ses notes météo et air, les conditions détaillées et les principales explications. La ville, les horaires locaux et la mention « Données récupérées à… » restent visibles.

## 4. Comparer les heures

Une liste horaire permet de parcourir les prochaines 24 heures. Sélectionner une heure affiche son détail. Changer d’activité conserve l’heure sélectionnée pour faciliter la comparaison.

## 5. Repérer le meilleur créneau

Un bouton mène au créneau le mieux noté pour l’activité choisie. En cas d’égalité, le plus proche dans le temps est retenu. Si tous les créneaux sont défavorables, un message le précise.

## 6. Actualiser ou changer de ville

Les données se renouvellent automatiquement et un bouton permet une actualisation manuelle. L’utilisateur peut rechercher une autre ville à tout moment.

## Situations particulières

- Aucun résultat : inviter à vérifier ou préciser le nom de la ville.
- Service indisponible : expliquer l’échec et proposer de réessayer.
- Données manquantes : conserver l’heure visible avec la mention « Données incomplètes », afficher les notes disponibles et « Indisponible » pour la note manquante, sans note globale ni sélection comme meilleur créneau.
- Actualisation en échec : conserver les dernières prévisions en indiquant qu’elles n’ont pas pu être actualisées.
