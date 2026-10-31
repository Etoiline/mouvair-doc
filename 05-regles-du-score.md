# Règles du score

Étape 5 — Règles produit validées le 7 septembre 2026 et implémentation vérifiée le 9 septembre 2026.

## Organisation validée

Pour chaque créneau, l’application présente :

- **Une note météo sur 100** : évalue les conditions météorologiques pour l’activité choisie.
- **Une note qualité de l’air sur 100** : calculée à partir des données de pollution.
- **Une note globale sur 100** : combine les deux notes et sert à comparer les créneaux.

Les facteurs détaillés et les explications permettent de comprendre les notes.

## Base de la note météo — Validée

La première version utilise trois facteurs :

- **Température ressentie** : avec des plages adaptées à chaque activité.
- **Précipitations** : selon leur intensité et l’activité.
- **Vent et rafales** : pour leur effet sur la pratique, notamment à vélo. Leur effet thermique est déjà intégré à la température ressentie.

Des situations fortement défavorables imposent un plafond sur la note météo. Pour les orages prévus, avec ou sans grêle, le plafond validé est de 10/100 ; la neige et les précipitations verglaçantes ont également leurs règles dans les sections dédiées.

La température ressentie sert de base à l’évaluation thermique. La première version n’intègre pas de calcul WBGT ni le modèle de Sports Medicine Australia. Aucune sous-note d’humidité indépendante n’est ajoutée, afin d’éviter de compter deux fois son effet thermique.

## Pondérations de la note météo — Validées

| Activité | Température ressentie | Précipitations | Vent et rafales |
|---|---:|---:|---:|
| Course tranquille | 50 % | 25 % | 25 % |
| Course intense | 60 % | 20 % | 20 % |
| Vélo | 35 % | 30 % | 35 % |
| Marche | 40 % | 30 % | 30 % |

Ces pondérations sont des choix de conception propres à Mouv’Air. Chaque facteur reçoit une sous-note sur 100, selon les courbes propres à l’activité validées ci-dessous.

La note météo avant plafonnement est la somme des sous-notes multipliées par leur poids. Les pondérations fixent l’importance de chaque facteur ; les seuils déterminent à partir de quelles conditions il devient pénalisant.

Exemple pour la course tranquille : une sous-note de température ressentie de 80, de précipitations de 100 et de vent de 60 donne `80 × 0,50 + 100 × 0,25 + 60 × 0,25 = 80/100`, avant un éventuel plafond.

La qualité de l’air est évaluée séparément et n’entre pas dans cette moyenne météo.

## Plafond progressif — Principe validé

Un facteur très défavorable ne doit pas être masqué par les bonnes sous-notes des autres facteurs. Pour limiter cette compensation, la note météo peut être soumise à un plafond qui diminue progressivement avec la dégradation du facteur concerné, notamment la température ressentie.

La note finale est la plus petite valeur entre la moyenne pondérée et le plafond applicable. Un plafond ne remonte jamais une note et ne constitue pas une pénalité fixe à soustraire.

Le principe progressif évite le saut lié au déclenchement brutal d’un plafond fixe. Les notes doivent conserver leurs valeurs intermédiaires, sans être limitées à quelques chiffres ronds.

La formule du plafond thermique est validée pour les quatre activités ci-dessous. Les plafonds des précipitations et du vent et leur combinaison sont également validés ci-dessous. Le plafond fixe à 59 précédemment discuté n’est pas retenu. La règle propre aux orages est validée dans la section dédiée ci-dessous.

## Température ressentie — Course tranquille de 1 h

Courbe, plafond thermique et règle d’arrondi validés le 7 septembre 2026. Ces valeurs sont des paramètres Mouv’Air, pas des seuils médicaux.

| Température ressentie | Sous-note température |
|---|---:|
| −10 °C ou moins | 0 |
| 0 °C | 40 |
| 5 °C | 75 |
| 10 à 18 °C | 100 |
| 23 °C | 75 |
| 28 °C | 40 |
| 35 °C ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Par exemple, 24 °C donne 68 et 26 °C donne 54.

En notant `S_temperature`, `S_precipitations` et `S_vent` les sous-notes non arrondies :

```text
moyenne_meteo = 0,50 × S_temperature + 0,25 × S_precipitations + 0,25 × S_vent
plafond_thermique = min(100, S_temperature + 20)
note_meteo_avant_arrondi = min(moyenne_meteo, plafond_thermique, plafond_precipitations, plafond_vent)
```

Ce plafond s’applique côté chaleur comme côté froid. Il limite à 20 points la compensation par les autres facteurs par rapport à la sous-note température. Les plafonds des précipitations et du vent s’appliquent également selon la règle commune ci-dessous ; les plafonds événementiels s’appliquent selon les sections dédiées.

Les décimales sont conservées pendant les calculs. La note finale affichée est arrondie à l’entier le plus proche, avec les demis arrondis vers le haut.

Exemples avec des sous-notes précipitations et vent de 100, sans autre plafond :

| Ressenti | Sous-note température | Moyenne météo | Plafond thermique | Note météo affichée |
|---|---:|---:|---:|---:|
| 18 °C | 100 | 100 | 100 | 100 |
| 21 °C | 85 | 92,5 | 100 | 93 |
| 24 °C | 68 | 84 | 88 | 84 |
| 26 °C | 54 | 77 | 74 | 74 |
| 28 °C | 40 | 70 | 60 | 60 |
| 30 °C | ≈ 28,6 | ≈ 64,3 | ≈ 48,6 | 49 |
| 35 °C | 0 | 50 | 20 | 20 |
| 0 °C | 40 | 70 | 60 | 60 |

## Température ressentie — Course intense de 1 h

Courbe et plafond thermique validés le 7 septembre 2026. Ces valeurs sont des paramètres Mouv’Air, pas des seuils médicaux.

| Température ressentie | Sous-note température |
|---|---:|
| −10 °C ou moins | 0 |
| 0 °C | 30 |
| 5 °C | 70 |
| 8 à 15 °C | 100 |
| 20 °C | 70 |
| 25 °C | 30 |
| 30 °C ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Les décimales sont conservées pendant les calculs. La note finale affichée est arrondie à l’entier le plus proche, avec les demis arrondis vers le haut.

```text
moyenne_meteo = 0,60 × S_temperature + 0,20 × S_precipitations + 0,20 × S_vent
plafond_thermique = min(100, S_temperature + 20)
note_meteo_avant_arrondi = min(moyenne_meteo, plafond_thermique, plafond_precipitations, plafond_vent)
```

Le plafond s’applique côté chaleur comme côté froid. Les plafonds des précipitations et du vent s’appliquent également selon la règle commune ci-dessous ; les plafonds événementiels s’appliquent selon les sections dédiées.

Exemples avec des sous-notes précipitations et vent de 100, sans autre plafond :

| Ressenti | Sous-note température | Moyenne météo | Plafond thermique | Note météo affichée |
|---|---:|---:|---:|---:|
| 15 °C | 100 | 100 | 100 | 100 |
| 18 °C | 82 | 89,2 | 100 | 89 |
| 22 °C | 54 | 72,4 | 74 | 72 |
| 24 °C | 38 | 62,8 | 58 | 58 |
| 27 °C | 18 | 50,8 | 38 | 38 |
| 30 °C | 0 | 40 | 20 | 20 |

## Température ressentie — Vélo de 1 h

Courbe et plafond thermique validés le 7 septembre 2026. Ces valeurs sont des paramètres Mouv’Air, pas des seuils médicaux.

| Température ressentie | Sous-note température |
|---|---:|
| −10 °C ou moins | 0 |
| 0 °C | 35 |
| 7 °C | 75 |
| 12 à 22 °C | 100 |
| 27 °C | 70 |
| 32 °C | 25 |
| 36 °C ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Les décimales sont conservées pendant les calculs. La note finale affichée est arrondie à l’entier le plus proche, avec les demis arrondis vers le haut.

```text
moyenne_meteo = 0,35 × S_temperature + 0,30 × S_precipitations + 0,35 × S_vent
plafond_thermique = min(100, S_temperature + 20)
note_meteo_avant_arrondi = min(moyenne_meteo, plafond_thermique, plafond_precipitations, plafond_vent)
```

Le plafond s’applique côté chaleur comme côté froid. Les plafonds des précipitations et du vent s’appliquent également selon la règle commune ci-dessous ; les plafonds événementiels s’appliquent selon les sections dédiées.

Exemples avec des sous-notes précipitations et vent de 100, sans autre plafond :

| Ressenti | Sous-note température | Moyenne météo | Plafond thermique | Note météo affichée |
|---|---:|---:|---:|---:|
| 20 °C | 100 | 100 | 100 | 100 |
| 24 °C | 88 | 95,8 | 100 | 96 |
| 28 °C | 61 | 86,35 | 81 | 81 |
| 30 °C | 43 | 80,05 | 63 | 63 |
| 34 °C | 12,5 | 69,375 | 32,5 | 33 |

## Température ressentie — Marche de 1 h

Courbe et plafond thermique validés le 7 septembre 2026. Ces valeurs sont des paramètres Mouv’Air, pas des seuils médicaux.

| Température ressentie | Sous-note température |
|---|---:|
| −10 °C ou moins | 0 |
| 0 °C | 50 |
| 5 °C | 75 |
| 10 à 22 °C | 100 |
| 28 °C | 70 |
| 33 °C | 30 |
| 38 °C ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Les décimales sont conservées pendant les calculs. La note finale affichée est arrondie à l’entier le plus proche, avec les demis arrondis vers le haut.

```text
moyenne_meteo = 0,40 × S_temperature + 0,30 × S_precipitations + 0,30 × S_vent
plafond_thermique = min(100, S_temperature + 20)
note_meteo_avant_arrondi = min(moyenne_meteo, plafond_thermique, plafond_precipitations, plafond_vent)
```

Le plafond s’applique côté chaleur comme côté froid. Les plafonds des précipitations et du vent s’appliquent également selon la règle commune ci-dessous ; les plafonds événementiels s’appliquent selon les sections dédiées.

Exemples avec des sous-notes précipitations et vent de 100, sans autre plafond :

| Ressenti | Sous-note température | Moyenne météo | Plafond thermique | Note météo affichée |
|---|---:|---:|---:|---:|
| 20 °C | 100 | 100 | 100 | 100 |
| 25 °C | 85 | 94 | 100 | 94 |
| 29 °C | 62 | 84,8 | 82 | 82 |
| 31 °C | 46 | 78,4 | 66 | 66 |
| 35 °C | 18 | 67,2 | 38 | 38 |
| 0 °C | 50 | 80 | 70 | 70 |

## Précipitations — Courses tranquille et intense de 1 h

Courbe commune validée le 7 septembre 2026. Ces valeurs sont des réglages Mouv’Air.

| Quantité prévue sur 1 h | Sous-note précipitations |
|---|---:|
| 0 mm | 100 |
| 0,5 mm | 80 |
| 2 mm | 40 |
| 5 mm ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Par exemple, 1 mm donne environ 66,67 et 3 mm environ 26,67. Les décimales sont conservées pendant le calcul de la note météo.

Cette sous-note représente 25 % de la moyenne météo pour la course tranquille et 20 % pour la course intense, conformément aux pondérations validées.

La courbe porte sur la quantité prévue sur une heure. Les règles de neige et d’orage sont validées dans les sections dédiées. La règle des précipitations verglaçantes et les limites concernant le verglas au sol sont précisées dans la section dédiée. Le plafond des précipitations suit la règle commune ci-dessous.

## Précipitations — Vélo de 1 h

Courbe validée le 7 septembre 2026. Ces valeurs sont des réglages Mouv’Air.

| Quantité prévue sur 1 h | Sous-note précipitations |
|---|---:|
| 0 mm | 100 |
| 0,2 mm | 75 |
| 1 mm | 35 |
| 3 mm ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Par exemple, 0,5 mm donne 60, 1,5 mm donne 26,25 et 2 mm donne 17,5. Les décimales sont conservées pendant le calcul de la note météo.

Cette sous-note représente 30 % de la moyenne météo pour le vélo. Le plafond des précipitations suit la règle commune ci-dessous.

## Précipitations — Marche de 1 h

Courbe validée le 7 septembre 2026. Ces valeurs sont des réglages Mouv’Air.

| Quantité prévue sur 1 h | Sous-note précipitations |
|---|---:|
| 0 mm | 100 |
| 0,5 mm | 85 |
| 2 mm | 55 |
| 5 mm | 15 |
| 8 mm ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Par exemple, 0,8 mm donne 79, 1,3 mm donne 69 et 3 mm donne environ 41,67. Les décimales sont conservées pendant le calcul de la note météo.

Cette sous-note représente 30 % de la moyenne météo pour la marche. Le plafond des précipitations suit la règle commune ci-dessous.

## Vent et rafales — Méthode validée

Méthode validée le 7 septembre 2026. Les coefficients sont des réglages Mouv’Air.

La valeur utilisée pour calculer la sous-note vent est la plus grande valeur entre le vent moyen et une fraction des rafales, toutes les vitesses étant exprimées en km/h :

| Activité | Valeur utilisée |
|---|---|
| Course tranquille, course intense, marche | `max(vent_moyen, 0,60 × rafales)` |
| Vélo | `max(vent_moyen, 0,75 × rafales)` |

Exemple avec un vent moyen de 20 km/h et des rafales à 40 km/h : la valeur de calcul est 24 km/h pour les courses et la marche, et 30 km/h pour le vélo.

Cette valeur est un intermédiaire de calcul, pas une mesure météo. L’interface affiche les prévisions réelles séparément : « Vent : 20 km/h — Rafales : 40 km/h » dans cet exemple.

Les courbes de vent des quatre activités sont validées ci-dessous.

## Vent — Courses tranquille et intense de 1 h

Courbe commune validée le 7 septembre 2026. Ces valeurs sont des réglages Mouv’Air.

La valeur de calcul est `max(vent_moyen, 0,60 × rafales)`, en km/h.

| Valeur de calcul | Sous-note vent |
|---|---:|
| 0 à 10 km/h | 100 |
| 20 km/h | 75 |
| 30 km/h | 40 |
| 50 km/h ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Par exemple, 16 km/h donne 85, 24 km/h donne 61 et 37 km/h donne 26. Les décimales sont conservées pendant le calcul de la note météo.

Cette sous-note représente 25 % de la moyenne météo pour la course tranquille et 20 % pour la course intense. Le plafond du vent suit la règle commune ci-dessous.

## Vent — Vélo de 1 h

Courbe validée le 7 septembre 2026. Ces valeurs sont des réglages Mouv’Air.

La valeur de calcul est `max(vent_moyen, 0,75 × rafales)`, en km/h.

| Valeur de calcul | Sous-note vent |
|---|---:|
| 0 à 8 km/h | 100 |
| 15 km/h | 70 |
| 25 km/h | 30 |
| 40 km/h ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Par exemple, 12 km/h donne environ 82,86, 19 km/h donne 54 et 32 km/h donne 16. Les décimales sont conservées pendant le calcul de la note météo.

Cette sous-note représente 35 % de la moyenne météo pour le vélo. Le plafond du vent suit la règle commune ci-dessous.

## Vent — Marche de 1 h

Courbe validée le 7 septembre 2026. Ces valeurs sont des réglages Mouv’Air.

La valeur de calcul est `max(vent_moyen, 0,60 × rafales)`, en km/h.

| Valeur de calcul | Sous-note vent |
|---|---:|
| 0 à 15 km/h | 100 |
| 30 km/h | 70 |
| 45 km/h | 30 |
| 65 km/h ou plus | 0 |

Entre deux points, la sous-note évolue par interpolation linéaire. Par exemple, 21 km/h donne 88, 36 km/h donne 54 et 52 km/h donne 19,5. Les décimales sont conservées pendant le calcul de la note météo.

Cette sous-note représente 30 % de la moyenne météo pour la marche. Le plafond du vent suit la règle commune ci-dessous.

## Plafonds des trois facteurs météo — Règle commune validée

Extension aux précipitations et au vent et combinaison des plafonds validées le 7 septembre 2026, pour les quatre activités.

Chaque plafond est égal à la sous-note non arrondie du facteur concerné plus 20, limité à 100 :

```text
plafond_thermique = min(100, S_temperature + 20)
plafond_precipitations = min(100, S_precipitations + 20)
plafond_vent = min(100, S_vent + 20)
note_meteo_avant_arrondi = min(moyenne_meteo, plafond_thermique, plafond_precipitations, plafond_vent)
```

La moyenne utilise les pondérations propres à l’activité. Les plafonds ne s’additionnent pas : la plus petite valeur s’applique. La note finale est arrondie à l’entier le plus proche, avec les demis arrondis vers le haut, après tous les calculs.

Une sous-note nulle limite donc la note météo à 20/100 au maximum, quelle que soit l’activité. La moyenne peut donner une note encore plus basse. Les courbes propres aux activités déterminent quand cette situation se produit.

Exemple pour le vélo : température 100, précipitations 35 et vent 100 donnent une moyenne de 80,5. Le plafond des précipitations vaut 55 ; les deux autres valent 100. La note météo finale est donc 55/100.

Les plafonds des orages et de la neige sont définis ci-dessous. La règle des précipitations verglaçantes et les limites concernant le verglas au sol sont précisées dans la section dédiée.

## Orages — Règle validée

Règle validée le 7 septembre 2026 pour les quatre activités. La valeur du plafond est un réglage Mouv’Air.

- Déclenchement : la prévision horaire indique un orage, avec ou sans grêle.
- Effet : la note météo est plafonnée à 10/100, après prise en compte de la moyenne pondérée et des plafonds progressifs.
- Message prioritaire : « Orage prévu sur ce créneau : reportez votre sortie. »

Le plafond est fixe : le déclenchement repose sur la présence de l’événement dans la prévision horaire. Il ne remonte jamais une note. Par exemple, une note de 83 ou de 24 devient 10 ; une note de 6 reste 6.

```text
si orage_prevu :
    note_meteo_avant_arrondi = min(note_meteo_avant_arrondi, 10)
```

La règle d’arrondi validée est appliquée après tous les plafonds.

## Neige prévue — Règle validée

Règle validée le 7 septembre 2026. Ces plafonds sont des réglages Mouv’Air.

Déclenchement : la prévision horaire annonce des chutes de neige sur le créneau.

| Activité | Plafond météo |
|---|---:|
| Course tranquille | 40 |
| Course intense | 40 |
| Vélo | 20 |
| Marche | 60 |

La note météo est la plus petite valeur entre la note calculée et tous les plafonds applicables, avant l’arrondi final. Le plafond neige ne remonte jamais une note.

Message associé : « Chutes de neige prévues : adaptez votre sortie aux conditions du terrain. »

Cette règle concerne uniquement les chutes de neige prévues sur le créneau. Elle ne permet pas de déduire l’état du sol ni la présence de verglas. La règle des précipitations verglaçantes est précisée dans la section dédiée.

## Précipitations verglaçantes — Règle validée

Règle validée le 7 septembre 2026 pour les quatre activités. La valeur du plafond est un réglage Mouv’Air.

- Déclenchement : la prévision horaire annonce de la pluie ou de la bruine verglaçante.
- Effet : la note météo est plafonnée à 10/100.
- Message prioritaire : « Précipitations verglaçantes prévues : risque de surfaces glissantes, reportez votre sortie. »

La plus petite valeur entre la moyenne pondérée et tous les plafonds applicables est conservée, avant l’arrondi final. Une note déjà inférieure à 10 ne remonte pas.

Cette règle ne détecte pas tout le verglas au sol. L’application n’affiche pas « verglas présent » sur la seule base d’une température négative, ni « absence de verglas » lorsque cette alerte ne se déclenche pas.

## Niveaux de la note météo — Validés

Niveaux et libellés validés le 7 septembre 2026. Ils s’appliquent à la note météo finale affichée, après les plafonds et l’arrondi.

| Note météo | Libellé |
|---|---|
| 80 à 100 | Conditions favorables |
| 60 à 79 | Conditions correctes |
| 40 à 59 | Conditions mitigées |
| 20 à 39 | Conditions défavorables |
| 0 à 19 | Conditions très défavorables |
| Données nécessaires manquantes | Données incomplètes — sans note |

Les messages d’orage, de neige ou de précipitations verglaçantes restent visibles à côté du niveau. Le libellé décrit les conditions prévues, sans garantir la sécurité de la sortie.

Ces niveaux sont validés pour la note météo. La note globale reprend ces niveaux selon la section dédiée ; la note air conserve ses catégories propres.

## Affichage et explications météo — Règles validées

Règles validées le 7 septembre 2026.

- Les sous-notes sont affichées à l’entier, selon le même arrondi que la note finale : entier le plus proche, demis vers le haut. Les calculs conservent les décimales.
- Les valeurs météo restent visibles : température ressentie, quantité de précipitations sur l’heure, vent moyen et rafales séparés.
- Jusqu’à deux explications principales identifient ce qui dégrade la note.
- Toutes les alertes événementielles restent visibles, même si le total dépasse deux messages, dans cet ordre : orage, précipitations verglaçantes, neige.

Sélection des explications principales à partir des valeurs non arrondies :

1. Si un plafond progressif réduit la moyenne, expliquer en premier le facteur qui impose le plafond le plus bas.
2. Compléter avec le facteur qui retire le plus de points à la moyenne, compte tenu de son poids, sans répéter le premier facteur. La contribution à la dégradation est `poids × (100 − sous-note)`.

Le mot « fortement » est réservé au premier facteur qui impose un plafond, lorsque les appréciations météo et globale sont toutes deux sous le niveau « Correct ». Si l’une est correcte, favorable ou indisponible, conserver une formulation sans cet intensif. Cette règle de rédaction ne modifie ni les notes, ni les plafonds, ni les alertes.

Exemple pour le vélo : température 100, précipitations 35 et vent 100 donnent une note météo de 55/100. Avec une note air de 85/100, la globale vaut 64/100 (« Correct ») ; le message est « Les précipitations prévues pénalisent ce créneau pour le vélo. » Avec une note air de 40/100, la globale vaut 51/100 (« Mitigé ») ; « pénalisent fortement » reste adapté.

Si les trois sous-notes valent 100 et qu’aucune alerte n’est présente, afficher : « Conditions météo favorables pour cette activité. »

## Qualité de l’air — Principe validé

Principe validé le 7 septembre 2026.

La note qualité de l’air est identique pour les quatre activités au même endroit et à la même heure. Elle décrit la qualité de l’air du lieu. Son influence sur la note globale est adaptée à l’activité par les pondérations définies dans la section dédiée à la note globale.

Changer de sport ne modifie donc pas la note air, mais peut modifier son importance dans la note globale.

Affichage validé :

- Une note air sur 100, où une note élevée signifie une meilleure qualité de l’air.
- L’indice de pollution utilisé, clairement identifié et distinct de la note Mouv’Air.
- Une explication lorsque l’air pénalise le créneau.
- La mention « Données incomplètes » si les données nécessaires manquent.

L’indice européen AQI fourni par Open-Meteo est validé comme référence. Les catégories et recommandations sont conservées selon la section dédiée ci-dessous. La conversion en note est validée dans la section dédiée.

## Indice air et information pollen — Bases validées

Décisions validées le 7 septembre 2026.

La note air utilise l’indice européen AQI fourni par Open-Meteo. L’affichage distingue « Note air Mouv’Air » et « Indice européen AQI ». Les données sont des prévisions issues des modèles CAMS, pas une mesure au niveau exact de la rue.

Le pollen est inclus dans la première version sous forme d’information séparée : types de pollens et concentrations prévues. Il ne modifie ni la note air, ni la note météo, ni la note globale. Aucune personnalisation selon les allergies n’est prévue à ce stade.

Selon la [documentation Open-Meteo](https://open-meteo.com/en/docs/air-quality-api), consultée le 7 septembre 2026 :

- L’indice européen retient le plus élevé des sous-indices de cinq polluants : PM₂,₅, PM₁₀, ozone, dioxyde d’azote et dioxyde de soufre. Le pollen n’y entre pas.
- Les prévisions de pollen couvrent l’aulne, le bouleau, les graminées, l’armoise, l’olivier et l’ambroisie. Elles sont disponibles en Europe pendant la saison pollinique, avec jusqu’à quatre jours de prévisions.

L’affichage du pollen et de ses données manquantes est validé dans la section dédiée. Aucun niveau qualitatif pollen n’est prévu en première version.

## Catégories et recommandations air — Principe validé

Principe validé le 7 septembre 2026 : conserver les catégories de qualité de l’air et leurs recommandations à côté de la note Mouv’Air. La note seule ne remplace pas ces informations.

Plages publiées par [Open-Meteo](https://open-meteo.com/en/docs/air-quality-api), consultées le 7 septembre 2026 :

| Indice européen fourni par Open-Meteo | Qualité de l’air |
|---|---|
| 0–20 | Bonne |
| 20–40 | Acceptable |
| 40–60 | Modérée |
| 60–80 | Mauvaise |
| 80–100 | Très mauvaise |
| Au-dessus de 100 | Extrêmement mauvaise |

Ces plages reprennent la présentation du fournisseur. L’affectation exacte aux frontières suit la convention Mouv’Air validée en fin de document. L’indice peut dépasser 100 ; la documentation ne donne pas de borne supérieure à la dernière catégorie. Ce n’est pas l’échelle américaine.

Résumé des [recommandations de l’Agence européenne pour l’environnement](https://airindex.eea.europa.eu/AQI/?webgl=0), consultées le 7 septembre 2026 :

| Qualité | Population générale | Personnes sensibles |
|---|---|---|
| Bonne / acceptable | Activités extérieures habituelles | Activités extérieures habituelles |
| Modérée | Activités habituelles | Envisager de réduire les efforts intenses si des symptômes apparaissent |
| Mauvaise | Envisager de réduire les efforts intenses en cas de symptômes | Envisager de réduire l’activité physique, surtout dehors et en cas de symptômes |
| Très mauvaise | Envisager de réduire les efforts intenses en cas de symptômes | Réduire l’activité physique, surtout dehors et en cas de symptômes |
| Extrêmement mauvaise | Réduire l’activité physique extérieure | Éviter l’activité physique extérieure |

Les personnes sensibles comprennent notamment les adultes et enfants ayant des problèmes respiratoires et les adultes ayant une maladie cardiaque. Les symptômes cités comprennent toux, irritation des yeux ou mal de gorge. Ces indications sont générales ; aucune déduction du profil de santé de l’utilisateur n’est prévue.

La conversion sur 100 est validée dans la section suivante.

## Conversion de l’indice air — Validée

Formule validée le 7 septembre 2026 : pour un indice européen AQI valide et non négatif, `note_air = max(0, 100 − AQI)`.

La note air est identique pour les quatre activités. Les décimales sont conservées pour les calculs. L’affichage utilise le même arrondi que la météo : entier le plus proche, demis vers le haut.

| AQI européen | Note air affichée |
|---|---:|
| 17 | 83 |
| 36 | 64 |
| 58 | 42 |
| 83 | 17 |
| 112 | 0 |

L’indice original reste affiché même au-delà de 100, lorsque la note air vaut zéro. La catégorie et les recommandations sont déterminées à partir de cet indice original, sans utiliser les libellés météo.

Cette conversion est un choix Mouv’Air. La note n’est ni un pourcentage de pollution ni un pourcentage de sécurité.

## Note globale — Pondérations validées

Pondérations validées le 7 septembre 2026. Ces valeurs sont des réglages Mouv’Air.

| Activité | Note météo | Note air |
|---|---:|---:|
| Course tranquille | 65 % | 35 % |
| Course intense | 60 % | 40 % |
| Vélo | 70 % | 30 % |
| Marche | 70 % | 30 % |

La moyenne globale est la somme des notes météo et air multipliées par leurs poids. La note air reste identique entre les activités ; son poids change. La note météo est déjà adaptée à l’activité.

Les calculs utilisent les valeurs non arrondies, avec la note météo après ses plafonds. L’arrondi intervient pour l’affichage : entier le plus proche, demis vers le haut.

Exemple pour une note météo de 85 et une note air de 43, avant plafonds globaux :

| Activité | Moyenne globale non arrondie | Moyenne globale affichée |
|---|---:|---:|
| Course tranquille | 70,3 | 70 |
| Course intense | 68,2 | 68 |
| Vélo | 72,4 | 72 |
| Marche | 72,4 | 72 |

Les plafonds globaux validés ci-dessous limitent la compensation entre météo et air, y compris lors des événements météorologiques défavorables.

## Note globale — Plafonds validés

Règles validées le 7 septembre 2026 pour les quatre activités. Ces plafonds sont des réglages Mouv’Air.

- Plafond météo : note météo non arrondie + 20, limité à 100.
- Plafond air : note air non arrondie + 20, limité à 100.
- La note globale retient la plus petite valeur entre la moyenne pondérée et ces deux plafonds.
- Les plafonds événementiels s’appliquent aussi directement à la note globale : 10 pour l’orage, 10 pour les précipitations verglaçantes, et pour la neige 40 pour les deux courses, 20 pour le vélo et 60 pour la marche.

```text
plafond_global_meteo = min(100, note_meteo_non_arrondie + 20)
plafond_global_air = min(100, note_air_non_arrondie + 20)
note_globale_non_arrondie = min(moyenne_globale, plafond_global_meteo, plafond_global_air)
pour chaque plafond_evenementiel applicable :
    note_globale_non_arrondie = min(note_globale_non_arrondie, plafond_evenementiel)
```

La note météo utilisée a déjà reçu ses plafonds. Les plafonds ne s’additionnent pas et ne remontent jamais une note. L’arrondi à l’entier le plus proche, demis vers le haut, intervient uniquement pour l’affichage final.

Exemples pour la course tranquille, sans autre événement que celui indiqué :

| Météo | Air | Moyenne globale | Note globale affichée après plafonds |
|---|---:|---:|---:|
| 85 | 83 | 84,3 | 84 |
| 85 | 43 | 70,3 | 63 |
| 90 | 12 | 62,7 | 32 |
| 25 | 90 | 47,75 | 45 |
| 10 — orage | 95 | 39,75 | 10 |

Les catégories et recommandations air restent affichées séparément.

## Note globale — Niveaux et données manquantes validés

Règles validées le 7 septembre 2026. Les niveaux s’appliquent après tous les plafonds et l’arrondi final.

| Note globale affichée | Libellé |
|---|---|
| 80 à 100 | Conditions favorables |
| 60 à 79 | Conditions correctes |
| 40 à 59 | Conditions mitigées |
| 20 à 39 | Conditions défavorables |
| 0 à 19 | Conditions très défavorables |

Si la note météo ou la note air manque, aucune note globale n’est calculée ; afficher « Données incomplètes ». La note disponible reste visible séparément. L’absence de données pollen ne bloque aucune note.

Les alertes météo et les recommandations air restent visibles même lorsqu’un libellé global est affiché.

## Pollen — Affichage validé

Règles validées le 7 septembre 2026 pour la première version.

Un encart « Pollens prévus » est associé à l’heure sélectionnée :

- Afficher chaque type de pollen disponible et sa concentration en grains/m³.
- Conserver les valeurs égales à zéro : 0 signifie une concentration prévue nulle. Une donnée manquante affiche « Indisponible ».
- Si aucune donnée n’est disponible, afficher « Prévisions de pollen indisponibles pour ce lieu ou ce créneau ».
- Ne pas afficher de niveaux « faible / modéré / élevé » dans la première version ; leurs seuils nécessiteraient une référence adaptée à chaque pollen.
- Afficher le rappel : « Information pollen présentée séparément ; elle n’entre pas dans les notes Mouv’Air. »

Exemple fictif :

| Pollen | Concentration prévue |
|---|---:|
| Bouleau | 18 grains/m³ |
| Graminées | 42 grains/m³ |
| Ambroisie | Indisponible |

## Frontières des catégories air — Convention validée

Convention validée le 7 septembre 2026 : chaque borne supérieure appartient à la catégorie qui se termine à cette valeur. La catégorie est déterminée à partir de l’indice européen original non arrondi.

| AQI européen non arrondi | Catégorie |
|---|---|
| De 0 à 20 inclus | Bonne |
| Plus de 20 à 40 inclus | Acceptable |
| Plus de 40 à 60 inclus | Modérée |
| Plus de 60 à 80 inclus | Mauvaise |
| Plus de 80 à 100 inclus | Très mauvaise |
| Plus de 100 | Extrêmement mauvaise |

Exemple : 40 reste « Acceptable » ; 40,1 devient « Modérée ». La catégorie ne dépend pas de l’arrondi de la note air Mouv’Air.

L’ancienne proposition de pondérations mélangeant directement les quatre facteurs météo et air n’est pas retenue. Le moteur actuel applique l’organisation à trois notes décrite dans ce document.







