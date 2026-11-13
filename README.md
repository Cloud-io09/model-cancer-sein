# SPÉ 1 — Détection de métastases du cancer du sein

[![Ouvrir dans Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Cloud-io09/model-cancer-sein/blob/eljily/run-notebook/SPE1_notebook_demarrage.ipynb)

Projet MBA 2 Data & BI, Epitech 2026-2027. On classe des patchs d'images de ganglions (tumeur ou non), puis on remonte à la classe de chaque lame et au stade pN de la patiente.

## Résultats

ROC-AUC au niveau des patchs. H0, H3 et H4 : hôpitaux de l'entraînement, sur des patientes mises de côté. H1 : hôpital jamais vu à l'entraînement.

| Modèle | H0 | H3 | H4 | H1 (jamais vu) |
|---|---|---|---|---|
| Référence (couleurs + régression logistique) | 0,976 | 0,938 | 0,545 | 0,761 |
| ResNet18 `base` | 0,988 | 0,983 | 0,969 | 0,955 |
| ResNet18 `couleur` (retenu) | 0,988 | 0,982 | 0,980 | **0,961** |
| ResNet18 `gris` | 0,977 | 0,974 | 0,980 | 0,951 |

- Seuil de 0,14, choisi sur H1 pour détecter au moins 95 % des tumeurs : rappel 0,951, spécificité 0,771.
- Du patch à la patiente : nos règles pN redonnent 100 % des stades officiels. Sur H1, 9 lames sur 10 sont bien classées, mais ce score est optimiste : dans WILDS, les patchs normaux sont clairsemés, donc les faux positifs restent isolés et ne forment pas de grande zone.
- Test final sur l'hôpital 2 : pas encore lancé, il ne sera utilisé qu'une fois.

## Relancer le notebook sur Colab

1. Cliquer sur le bouton « Open in Colab » ci-dessus.
2. Exécution → Modifier le type d'exécution → **GPU T4**.
3. Exécution → **Tout exécuter**. La première fois, le notebook télécharge les données (environ 10 Go, 15 à 20 minutes).

Le test final sur l'hôpital 2 est désactivé (`LANCER_TEST = False`) : il ne sert qu'une fois, à la fin.

## Relancer en local

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt jupyter
.venv/bin/jupyter notebook SPE1_notebook_demarrage.ipynb
```

Le notebook utilise le GPU s'il y en a un (CUDA, ou MPS sur Mac).

## Contenu

| Fichier | Rôle |
|---|---|
| `SPE1_notebook_demarrage.ipynb` | tout le pipeline, avec les résultats affichés |
| `figures/` | figures pour les slides |
| `data/` | liste des patchs, métadonnées et stades officiels (pas d'images) |
| `usage_ia.md` | annexe « Usage de l'IA » |
| `01_Sujet_…pdf`, `02_Regles_…pdf`, `LISEZMOI.txt` | sujet, règles et consignes d'origine du pack élèves (le dossier `donnees/` qu'il cite correspond ici à `data/`) |
