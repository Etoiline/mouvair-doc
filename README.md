# Mouv’Air — Documentation

Ce dépôt rassemble la documentation produit et technique de Mouv’Air.

## Documents

- [Étape 1 — Vision du produit](01-vision-produit.md) : validée le 6 septembre 2026.
- [Étape 2 — Utilisateurs et besoins](02-utilisateurs-et-besoins.md) : validée le 6 septembre 2026.
- [Étape 3 — Périmètre de la première version](03-perimetre-premiere-version.md) : validée le 6 septembre 2026.
- [Étape 4 — Parcours utilisateur](04-parcours-utilisateur.md) : validée le 6 septembre 2026.
- [Étape 5 — Règles du score](05-regles-du-score.md) : règles produit validées le 7 septembre 2026 ; implémentation à adapter.
- [Étape 6 — Données, horaires et actualisation](06-donnees-et-horaires.md) : règles produit validées le 7 septembre 2026 ; implémentation à adapter.
- [Étape 7 — Organisation des informations à l’écran](07-organisation-des-informations.md) : validée le 7 septembre 2026.
- [Synthèse du projet](synthese.md) : fonctionnalités, architecture, état technique et priorités.

## Organisation locale

Les trois dossiers sont placés au même niveau :

```text
mouvair/
├── mouvair-front/
├── mouvair-back/
└── mouvair-doc/
```

Le code de l’interface se trouve dans `mouvair-front`, celui de l’API dans `mouvair-back`.

## Créer une release

La version courante de la documentation est conservée dans `VERSION`. Le dépôt doit être propre et
`CHANGELOG.md` doit contenir au moins une ligne de changement non publiée sous son titre. Le script
demande s’il faut incrémenter la version majeure, mineure ou corrective, met à jour `VERSION` et le
changelog, puis crée le commit `🔖 Release x.y.z` :

```powershell
python scripts/release.py
```

Pour prévisualiser la prochaine version sans modifier le dépôt ni créer de commit :

```powershell
python scripts/release.py --dry-run
```
