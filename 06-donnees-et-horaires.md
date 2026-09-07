# Données, horaires et actualisation

Étape 6 — Règles produit validées le 7 septembre 2026 ; implémentation à adapter.

## Horaires des créneaux — Validés

- Chaque créneau correspond à une heure complète, par exemple 15 h–16 h.
- La liste commence à la prochaine heure pleine : à 14 h 35, le premier départ proposé est 15 h.
- La liste affiche 24 créneaux consécutifs, même lorsque certaines données manquent.
- Les dates et heures sont celles de la ville sélectionnée, avec son fuseau indiqué.
- Un créneau dont le départ est passé est retiré des choix et ne peut plus être recommandé.
- Les données météo, air et pollen sont alignées sur le même créneau.

Cette organisation évite de proposer une sortie déjà commencée. Au changement d’heure, chaque créneau conserve une durée réelle de 1 h, avec une distinction explicite des heures répétées.

## Actualisation — Validée

- À la sélection d’une ville : charger ses prévisions.
- Toutes les 15 minutes, lorsque la page est visible : vérifier les dernières données disponibles.
- Au retour sur la page : actualiser si la dernière récupération réussie date de plus de 15 minutes.
- Un bouton « Actualiser » permet une vérification manuelle.
- À chaque changement d’heure : retirer les départs passés et demander les données nécessaires pour conserver 24 créneaux.

Pendant l’actualisation, les résultats restent visibles. L’activité et le créneau sélectionnés sont conservés s’ils sont encore disponibles ; sinon, le premier créneau disponible est sélectionné.

En cas d’échec :

- Conserver les dernières données avec le message « Actualisation impossible — dernières données récupérées à 14 h 20 », en utilisant l’heure réelle de dernière récupération réussie.
- Ne pas modifier l’heure de dernière récupération réussie.
- Proposer de réessayer.
- Continuer à retirer les départs passés.

L’heure affichée est intitulée « Données récupérées à… ». Une récupération réussie ne signifie pas nécessairement que le fournisseur a produit de nouvelles prévisions.

## Cache et données anciennes — Validés

Règles validées le 7 septembre 2026.

- Réutiliser pendant 15 minutes les prévisions déjà récupérées pour un lieu, si elles couvrent les créneaux demandés.
- Après 15 minutes, effectuer une nouvelle récupération auprès du fournisseur.
- Le bouton « Actualiser » suit la même règle et peut restituer des données récemment mises en cache.
- Une réponse issue du cache conserve l’heure réelle de récupération auprès du fournisseur.

Si le fournisseur ne répond plus, appliquer les règles suivantes selon le temps écoulé depuis la dernière récupération réussie :

| Âge des données récupérées | Comportement |
|---|---|
| Jusqu’à 1 h | Conserver les notes avec le message d’échec d’actualisation |
| Plus de 1 h | Conserver les dernières données à titre indicatif, signaler qu’elles sont anciennes et suspendre le « meilleur créneau » |

Les départs passés restent retirés. Si les données ne couvrent plus les 24 créneaux, les heures manquantes sont affichées comme indisponibles.

Le seuil de 1 h est un choix de fonctionnement Mouv’Air, pas une durée de validité scientifique des prévisions. Servir une réponse en cache ne remet pas à zéro son âge.

## Conditions sur une sortie de 1 h — Méthode validée

Méthode validée le 7 septembre 2026.

Pour une sortie de 15 h à 16 h :

| Donnée | Valeur retenue |
|---|---|
| Température ressentie | Parmi les valeurs de 15 h et 16 h, celle donnant la sous-note la plus basse pour l’activité |
| Vent moyen | Valeur la plus élevée entre 15 h et 16 h |
| Rafales | Maximum prévu sur 15 h–16 h |
| Précipitations | Cumul prévu sur 15 h–16 h |
| Qualité de l’air | Plus mauvais AQI entre 15 h et 16 h |
| Pollen | Pour chaque pollen, concentration la plus élevée entre 15 h et 16 h |
| Orage, neige, précipitations verglaçantes | Alerte si l’événement est annoncé à l’une des deux bornes |

Selon la [documentation Open-Meteo](https://open-meteo.com/en/docs), consultée le 7 septembre 2026, les précipitations et les rafales horodatées à 16 h concernent respectivement le cumul et le maximum de l’heure précédente. La température et le vent moyen horodatés à 16 h représentent des valeurs à cet instant. Le cumul et les rafales horodatés en fin de créneau sont donc utilisés pour cette sortie.

Les bornes correspondent à deux instants séparés d’une heure réelle, y compris au changement d’heure. Cette méthode vise à ne pas masquer une dégradation en fin de sortie ; elle reste une approximation horaire et ne garantit pas la détection de tous les changements entre les bornes.

Les valeurs affichées sont celles utilisées dans le calcul, accompagnées de la mention « Conditions retenues pour ce créneau de 1 h ». Les valeurs de vent moyen et de rafales retenues sont ensuite combinées selon les coefficients validés dans les règles du score.

Si une valeur nécessaire manque, la note concernée reste incomplète. La disponibilité du pollen reste indépendante des notes.

## Données manquantes ou incohérentes — Règles validées

Règles validées le 7 septembre 2026.

- Une valeur absente ou invalide n’est jamais remplacée par zéro. Une température négative reste valide ; une pluie, un vent, un AQI ou une concentration de pollen négatifs sont invalides.
- Si une donnée nécessaire manque ou est invalide, la note concernée affiche « Indisponible ». La note n’est pas recalculée avec les seuls critères disponibles.
- La météo et l’air restent indépendants : si l’une des notes manque, l’autre reste visible, mais la note globale est indisponible.
- Pour les pollens, chaque type reste indépendant : les concentrations disponibles sont affichées.
- Une alerte météo détectée reste visible, même si la note météo ne peut pas être calculée.
- Une donnée dont l’horaire ne correspond pas au créneau n’est pas utilisée pour le compléter.
- Le créneau reste affiché, mais sans note globale il ne peut pas être proposé comme meilleur créneau.
