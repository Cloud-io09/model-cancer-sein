# SPÉ 1 — Détection de métastases du cancer du sein

[![Ouvrir dans Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Cloud-io09/model-cancer-sein/blob/eljily/run-notebook/SPE1_notebook_demarrage.ipynb)

Projet MBA 2 Data & BI, Epitech 2026-2027. On classe des patchs d'images de ganglions (tumeur ou non), puis on remonte à la classe de chaque lame et au stade pN de la patiente.

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
| `flw-up-notes.md` | notes et résultats pour le follow-up du 13 novembre |
| `figures/` | figures pour les slides |
| `data/` | liste des patchs, métadonnées et stades officiels (pas d'images) |
| `usage_ia.md` | annexe « Usage de l'IA » |
