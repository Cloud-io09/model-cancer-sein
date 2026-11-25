# Résultats

Généré le 25/11/2026 à 22:38 — QUICK=False, époques 2+4, TAILLE=96, variantes ['base', 'couleur', 'gris', 'macenko'], graines [42, 43, 44], calcul sur mps.

## 1. Découpage par patiente

| ens | patchs | lames | patientes | part_tumorale |
|---|---|---|---|---|
| entr | 21727 | 23 | 20 | 0.500 |
| test | 8000 | 10 | 9 | 0.500 |
| val_lames | 5273 | 7 | 6 | 0.500 |
| val_ood | 4000 | 10 | 8 | 0.500 |

## 2. Trois constats d'exploration

1. La couleur trahit l'hôpital : un histogramme de couleur reconnaît l'hôpital de lames jamais vues dans 79% des cas (hasard : 20 %). Un modèle peut donc apprendre l'hôpital au lieu des cellules.
2. Les lames sont très inégales : de 888 à 61 110 patchs par lame. La plus grosse (hôpital 4) en compte 61 110, l'hôpital 1 entier 34 904. D'où un découpage par patiente et des résultats par hôpital.
3. La part de tumeur va de 0.1% à 93% selon la lame, et WILDS est équilibré à 50 % par construction : en clinique, la précision sera bien plus basse qu'ici.

Hôpital reconnu par la seule couleur : 79.0% avant normalisation de Macenko, 58.0% après (hasard : 20 %).

## 3. Référence simple (histogrammes de couleur + régression logistique)

ROC-AUC : val_lames 0.836 · val_ood (hôpital 1) 0.761

H1 sert aux réglages ; seul H2 (section 7) est hors distribution.

## 4. CNN par transfert (ResNet18) : ROC-AUC par hôpital

| modèle | val_lames · H0 (entr.) | val_lames · H3 (entr.) | val_lames · H4 (entr.) | val_ood · H1 (valid. hors distrib.) |
|---|---|---|---|---|
| resnet18_base | 0.988 | 0.990 | 0.960 | 0.967 |
| resnet18_couleur | 0.984 | 0.978 | 0.976 | 0.951 |
| resnet18_gris | 0.974 | 0.976 | 0.978 | 0.942 |
| resnet18_macenko | 0.954 | 0.973 | 0.981 | 0.947 |
| référence | 0.976 | 0.938 | 0.545 | 0.761 |

### Stabilité sur les graines (moyenne et écart-type)

| variante | val_lames mean | val_lames std | val_ood mean | val_ood std | seuil val_ood mean | seuil val_ood std | spécificité val_ood mean | spécificité val_ood std |
|---|---|---|---|---|---|---|---|---|
| base | 0.981 | 0.003 | 0.965 | 0.002 | 0.213 | 0.040 | 0.803 | 0.019 |
| couleur | 0.981 | 0.003 | 0.960 | 0.008 | 0.083 | 0.064 | 0.750 | 0.097 |
| gris | 0.978 | 0.003 | 0.949 | 0.007 | 0.180 | 0.277 | 0.800 | 0.143 |
| macenko | 0.969 | 0.002 | 0.944 | 0.003 | 0.177 | 0.280 | 0.773 | 0.167 |

### Détail par graine

| variante | graine | val_lames | val_ood | seuil val_ood | spécificité val_ood | seuil val_lames | époque perte val_ood min |
|---|---|---|---|---|---|---|---|
| base | 42 | 0.978 | 0.967 | 0.170 | 0.824 | 0.050 | 6 |
| base | 43 | 0.984 | 0.962 | 0.220 | 0.791 | 0.170 | 5 |
| base | 44 | 0.981 | 0.965 | 0.250 | 0.793 | 0.130 | 4 |
| couleur | 42 | 0.979 | 0.951 | 0.010 | 0.638 | 0.150 | 4 |
| couleur | 43 | 0.981 | 0.963 | 0.120 | 0.800 | 0.190 | 4 |
| couleur | 44 | 0.985 | 0.965 | 0.120 | 0.811 | 0.250 | 5 |
| gris | 42 | 0.975 | 0.942 | 0.500 | 0.964 | 0.080 | 3 |
| gris | 43 | 0.981 | 0.956 | 0.020 | 0.737 | 0.240 | 6 |
| gris | 44 | 0.977 | 0.950 | 0.020 | 0.699 | 0.160 | 3 |
| macenko | 42 | 0.971 | 0.947 | 0.500 | 0.963 | 0.050 | 3 |
| macenko | 43 | 0.968 | 0.941 | 0.010 | 0.652 | 0.040 | 5 |
| macenko | 44 | 0.967 | 0.945 | 0.020 | 0.702 | 0.060 | 3 |

- Modèle retenu (meilleure ROC-AUC moyenne sur val_ood) : **base** — graine 42 : val_lames 0.978, val_ood 0.967 (écart val_lames − val_ood : +0.011)

- Seuil pour un rappel ≥ 95% sur val_ood : **0.17** → rappel 0.950, spécificité 0.825

- Matrice de confusion val_ood (VN, FP, FN, VP) : 1649, 351, 100, 1900

- Précision attendue si 1 % des patchs étaient tumoraux : 5.2%

- Sur-apprentissage : époque où la perte val_ood est minimale (première graine) — base 6/6, couleur 4/6, gris 3/6, macenko 3/6

### Probabilités sur val_lames et val_ood (modèle retenu)

| index | proba moyenne des patchs tumoraux | part des tumoraux sous 0,5 | seuil pour 95 % de rappel | ROC-AUC |
|---|---|---|---|---|
| val_lames | 0.863 | 0.134 | 0.050 | 0.978 |
| val_ood | 0.909 | 0.085 | 0.170 | 0.967 |

### Grad-CAM sur l'hôpital 1 (40 patchs par catégorie, moyenne et écart-type)

Part sur les noyaux : 0,25 = aucune préférence (hors cartes nulles).

| catégorie | Grad-CAM nulle | part Grad-CAM sur noyaux | corrélation Grad-CAM / hématoxyline | hématoxyline moyenne | luminosité moyenne | p |
|---|---|---|---|---|---|---|
| VP | 0.000 | 0.242 | -0.044 | 0.557 | 0.546 | 0.966 |
| FN | 0.350 | 0.269 | 0.060 | 0.521 | 0.629 | 0.061 |
| FP | 0.000 | 0.264 | 0.077 | 0.509 | 0.617 | 0.602 |
| VN | 0.925 | 0.226 | 0.020 | 0.847 | 0.517 | 0.008 |

### Fuite de id_val

- 84% des patchs id_val de WILDS recouvrent en partie un patch d'entraînement

- Même modèle, ROC-AUC : id_val (lames vues) 0.996 · val_lames (patientes nouvelles) 0.980

## 5. Patch → lame → patiente

- Plafond de la méthode (labels vrais, 50 lames) : 88% des lames bien classées ; 5 stades pN changés, dont 0 sous-stadifications

- Règles pN : 100% de concordance avec le stade officiel des 100 patientes

- Modèle, lames de l'hôpital 1 (seuil 0.5) : 9 / 10 bien classées ; 1 stades pN changés, dont 0 sous-stadifications

| officielle | negative | itc | micro | macro |
|---|---|---|---|---|
| negative | 0 | 0 | 0 | 0 |
| itc | 0 | 1 | 1 | 0 |
| micro | 0 | 0 | 5 | 0 |
| macro | 0 | 0 | 0 | 3 |

## Figures

- figures/01_patchs_par_hopital.png
- figures/02_couleurs_par_hopital.png
- figures/03_patchs_par_lame.png
- figures/04_confusion_et_roc.png
- figures/05_gradcam_H1_FN.png
- figures/05_gradcam_H1_FP.png
- figures/05_gradcam_val_lames.png
- figures/05_gradcam_val_ood.png
- figures/06_cartes_lames_H1.png
- figures/07_courbes_perte.png
- figures/08_macenko.png
- figures/09_probas_par_ensemble.png

## 7. Test final : H2

Modèle base (graine 42), seuil des patchs 0.17, seuil des lames 0.5.

### Patchs, modèle retenu

| hôpital | n | ROC-AUC | rappel | précision | accuracy |
|---|---|---|---|---|---|
| H2 (test, jamais vu) | 8000 | 0.865 | 0.318 | 0.966 | 0.653 |

- Matrice de confusion (VN, FP, FN, VP) : 3955, 45, 2729, 1271 ; spécificité 0.989

- Seuil pour 95 % de rappel sur H2 (non utilisé) : 0.50

  Note ajoutée après l'exécution : 0.50 est la valeur par défaut de `seuil_rappel` quand **aucun** seuil entre 0,01 et 0,99 n'atteint 95 % de rappel. Sur H2, même le seuil le plus bas ne détecte pas 95 % des tumeurs.

### Patchs, référence

| hôpital | n | ROC-AUC | rappel | précision | accuracy |
|---|---|---|---|---|---|
| H2 (test, jamais vu) | 8000 | 0.398 | 0.288 | 0.357 | 0.384 |

### Lames de l'hôpital 2 : 5 / 10 bien classées (plafond de la méthode sur H2 : 9 / 10)

| officielle | negative | itc | micro | macro |
|---|---|---|---|---|
| negative | 0 | 0 | 0 | 0 |
| itc | 0 | 2 | 0 | 0 |
| micro | 0 | 3 | 1 | 0 |
| macro | 0 | 1 | 1 | 2 |

- Stades pN changés : 2, dont 2 sous-stadifications

| pid | officiel | recalculé | erreur |
|---|---|---|---|
| patient_042 | pN1 | pN0(i+) | sous-stadification (traitement insuffisant) |
| patient_045 | pN2 | pN1 | sous-stadification (traitement insuffisant) |