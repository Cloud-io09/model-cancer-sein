# Détection de métastases du cancer du sein en histopathologie

Projet SPÉ 1 — MBA 2 Data & BI, Epitech, 2026-2027.

Ce projet classe des patchs d'images de ganglions lymphatiques (normal ou tumoral) avec un CNN, puis remonte du patch à la lame (ITC, micro- ou macro-métastase) et au stade pN de la patiente. Il s'appuie sur Camelyon17-WILDS, qui réunit 5 hôpitaux. Le modèle s'entraîne sur 3 d'entre eux et il est évalué sur 2 hôpitaux jamais vus.

## Relancer le projet

### Sur Google Colab (recommandé)

1. Ouvrir [le notebook dans Colab](https://colab.research.google.com/github/Cloud-io09/model-cancer-sein/blob/main/SPE1_notebook_demarrage.ipynb).
2. Choisir le GPU : *Exécution → Modifier le type d'exécution → GPU T4*.
3. Lancer *Exécution → Tout exécuter*.

Le notebook clone ce dépôt pour récupérer `data/`. Il reconstruit ensuite le sous-échantillon depuis Hugging Face, soit environ 10 Go téléchargés et 10 à 20 minutes. Copiez ensuite `camelyon17_sous_echantillon.parquet` dans `MyDrive/SPE1/` et mettez `SOURCE = "drive"` pour les sessions suivantes.

### En local

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt jupyter
jupyter notebook SPE1_notebook_demarrage.ipynb
```

Un GPU est fortement conseillé.

## Contenu du dépôt

| Chemin | Contenu |
|---|---|
| `SPE1_notebook_demarrage.ipynb` | Pipeline complet : données, exploration, modèles, évaluation, Grad-CAM, patch → lame → pN, synthèse, test final |
| `data/ids_sous_echantillon.csv` | Les 39 000 patchs du sous-échantillon |
| `data/metadata_complete.zip` | Hôpital, lame, patiente, coordonnées et label des 455 954 patchs (sans images) |
| `data/stages.csv` | Classe de chaque lame et stade pN des 100 patientes (CAMELYON17, CC0) |
| `docs/` | Sujet, règles communes, notes de follow-up, résumé exécutif |

Les images ne sont pas versionnées, car elles sont trop volumineuses. Le notebook les télécharge.

## Paramètres principaux (section 0 du notebook)

| Paramètre | Rôle |
|---|---|
| `QUICK` | Essai rapide sur quelques centaines de patchs |
| `VARIANTES` | Traitements de la couleur comparés : `base`, `couleur`, `gris` |
| `CIBLE_RAPPEL` | Rappel minimal visé sur la validation pour fixer le seuil (0,95 par défaut) |
| `SEUIL_LAME` | Seuil des patchs pour délimiter les zones tumorales d'une lame |
| `LAMES_H1` | Prédire tous les patchs des lames de l'hôpital 1 (environ 1,5 Go) |
| `LANCER_TEST` | Test final sur l'hôpital 2, **à n'exécuter qu'une fois** |

## Sorties

La section 9 écrit `resultats_followup.md`, qui contient tous les chiffres clés, ainsi que les figures dans `figures/`. Sur Colab, elle télécharge aussi l'archive `followup.zip`.

## Sources

- CAMELYON17 : https://camelyon17.grand-challenge.org (licence CC0) ; Bándi et al., IEEE TMI, 2019.
- Camelyon17-WILDS : Koh et al., *WILDS: A Benchmark of in-the-Wild Distribution Shifts*, ICML 2021. L'annexe E.2 décrit la géométrie des patchs.
- Miroir Hugging Face : `wltjr1007/Camelyon17-WILDS`.

L'usage d'outils d'IA générative est déclaré dans l'annexe « Usage de l'IA » du rapport.
