# Résumé exécutif — SPÉ 1, état au 13 novembre 2026

## Contexte

Pour fixer le stade d'un cancer du sein, le pathologiste cherche au microscope des métastases dans les ganglions lymphatiques. C'est un travail long, et les petites métastases peuvent passer inaperçues. Les données viennent de CAMELYON17, dans sa version Camelyon17-WILDS : des patchs de 96 × 96 pixels tirés de 50 lames annotées, issues de 5 hôpitaux néerlandais.

Chaque hôpital a ses propres scanners et colorations, donc ses propres couleurs. Un modèle peut fonctionner dans les hôpitaux d'entraînement et échouer dans un hôpital qu'il n'a jamais vu.

## La demande

Construire un **système d'aide au diagnostic**, pas seulement un modèle :

1. classer les patchs (normal ou tumoral), avec une référence simple obligatoire et un CNN par transfert ;
2. remonter du patch à la lame (cellules tumorales isolées, micro- ou macro-métastase), puis au stade pN de la patiente ;
3. évaluer la fiabilité : faux négatifs, résultats par hôpital, interprétabilité, limites cliniques et réglementaires.

**Calendrier :**

| Étape | Date |
|---|---|
| Follow-up (20 % de la note) | 13 novembre |
| Dépôt des livrables : code, rapport de 15 pages max, supports, annexe IA | **26 novembre à 18h00**, aucun retard accepté |
| Soutenance de 10 min, note individuelle | 27 novembre |

## L'axe retenu

**La généralisation à un hôpital inconnu, et le coût d'un faux négatif.** Chaque choix est jugé hôpital par hôpital, et en priorité sur l'hôpital 1, que le modèle ne voit jamais à l'entraînement. Le seuil de décision privilégie le rappel, car une métastase manquée est l'erreur la plus grave. Le niveau patiente est traité honnêtement : faute de données complètes, on mesure l'effet de nos erreurs sur le stade pN au lieu de prétendre prédire ce stade.

## Nos décisions

| # | Décision | Pourquoi |
|---|---|---|
| 1 | Découpage **par patiente**, avec un ensemble `val_lames`, à la place du `id_val` officiel | `id_val` partage ses lames avec l'entraînement, ce qui crée une fuite et gonfle les scores. Aucune lame n'est dans deux ensembles, et le code le vérifie. |
| 2 | Hôpital 1 (`val_ood`) pour tous les réglages, hôpital 2 pour le test **une seule fois** | C'est la règle du sujet, et la seule mesure honnête de la généralisation. |
| 3 | Référence : histogrammes de couleur + régression logistique | C'est la référence simple obligatoire. Elle montre aussi ce que la couleur seule permet. |
| 4 | ResNet18 pré-entraîné en 96 × 96 : d'abord la dernière couche, puis le dernier bloc | Léger, rapide sur un GPU T4 gratuit, et taille native des patchs. Le passage en 224 × 224 reste à tester. |
| 5 | Trois traitements de la couleur comparés : `base`, `couleur` (variations aléatoires), `gris` | Le sujet demande au moins une technique contre l'écart entre hôpitaux. Le modèle retenu est le meilleur sur l'hôpital 1. |
| 6 | Seuil des patchs : le plus haut seuil qui garde un rappel ≥ 95 % sur l'hôpital 1 | Un faux négatif est une métastase manquée. Un faux positif est revu par le pathologiste. |
| 7 | Perte non pondérée, mais précision recalculée pour une tumeur rare (20 %, 5 %, 1 % des patchs) | Le jeu est équilibré à 50/50 par construction. Le déséquilibre clinique se traite par le seuil et par la discussion de la précision. |
| 8 | Étendue des métastases mesurée sur la grille des patchs : cases d'environ 31 µm, zones de cases voisines | La géométrie vient de l'article WILDS (annexe E.2), et tous les patchs tumoraux y sont conservés. |
| 9 | Classe de la lame selon la plus grande zone (ITC ≤ 0,2 mm, micro ≤ 2 mm, macro au-delà), puis règles pN | Ce sont les seuils officiels de CAMELYON17. |
| 10 | Lames prédites en entier uniquement pour les hôpitaux 1 et 2 | Les lames des hôpitaux 0, 3 et 4 ont servi à l'entraînement. |

## Nos résultats à ce jour

### Vérifiés, à partir des métadonnées et sans modèle

- **Données.** 50 lames, toutes positives, issues de 43 patientes : 16 ITC, 17 micro-métastases, 17 macro-métastases. Aucune patiente n'a ses 5 ganglions annotés.
- **Découpage.**

  | Ensemble | Patchs | Lames |
  |---|---|---|
  | `entr` | 21 727 | 23 |
  | `val_lames` | 5 273 | 7 |
  | `val_ood` (H1) | 4 000 | 10 |
  | `test` (H2) | 8 000 | 10 |

- **Constat 1 : les lames sont très inégales.** Elles comptent de 888 à 61 110 patchs. La plus grosse lame dépasse à elle seule tout l'hôpital 1 (34 904 patchs).
- **Constat 2 : la part de tumeur varie énormément.** Elle va de 0,1 % à 93 % selon la lame, alors que le jeu est équilibré à 50 % par construction.
- **Plafond de la méthode lame.** Avec les labels vrais, notre mesure d'étendue retrouve la classe officielle de **44 lames sur 50 (88 %)**. Les 6 écarts sont des ITC classées micro.
- **Stade pN.** Nos règles redonnent **100 %** des stades officiels des 100 patientes. Avec les classes du plafond, 5 patientes changent de stade, toutes vers un stade plus élevé (aucune sous-stadification).

### À produire par l'exécution sur Colab

Le pipeline complet a été exécuté de bout en bout sur des données factices pour vérifier le code. **Les scores réels ne sont pas encore produits.** Il faut lancer le notebook sur Colab avec un GPU T4, dont la section 9 génère `resultats_followup.md`. Restent à obtenir :

- le constat 3 : la part de patchs dont l'hôpital est reconnu par la seule couleur ;
- la ROC-AUC de la référence et des 3 variantes du CNN, par hôpital ;
- le modèle retenu, le seuil, le rappel et la spécificité ;
- la précision attendue en conditions réelles ;
- Grad-CAM ;
- la classe des lames de l'hôpital 1 et l'effet sur le pN.

## Limites à assumer

- Toutes les lames sont positives et aucune patiente n'est complète : on mesure l'effet des erreurs sur le pN, sans prédire un vrai stade.
- WILDS n'a gardé qu'une partie des patchs normaux : sur une lame entière, les faux positifs seraient plus nombreux.
- Le seuil des lames est réglé sur l'hôpital 1, donc le résultat par lame y est optimiste.
- La résolution d'environ 0,96 µm par pixel est déduite de l'article WILDS. C'est un paramètre (`UM_PAR_PX`).

## Prochaines étapes (d'ici le 26 novembre)

1. Exécuter le notebook sur Colab et compléter les notes de follow-up.
2. Tester plusieurs graines, puis une normalisation de la coloration (Macenko) et/ou une entrée en 224 × 224.
3. Affiner l'estimation de l'étendue (lissage, taille minimale de zone) et analyser les faux négatifs avec Grad-CAM.
4. Lancer le test final sur l'hôpital 2, une seule fois.
5. Rédiger le rapport, les supports de soutenance et l'annexe « Usage de l'IA ».
