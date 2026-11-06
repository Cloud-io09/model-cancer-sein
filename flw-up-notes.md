# Follow-up SPÉ 1 — vendredi 13 novembre 2026

Présentation de **10 minutes**, qui compte pour **20 % de la note finale**. Critère évalué : régularité et sérieux du travail, capacité à intégrer les retours, organisation du groupe.

Les valeurs ci-dessous sortent du notebook exécuté en entier le 6 novembre sur un Mac M2 (GPU MPS, graine 42, une seule exécution). Une exécution sur Colab donnera des chiffres légèrement différents. La section du notebook qui les produit est indiquée à chaque fois, et les captures sont dans `figures/`.

---

## 1. Ce que le sujet exige pour ce rendez-vous

| # | Attendu | Où le trouver |
|---|---|---|
| 1 | L'échantillon retenu et le découpage (par lame, par hôpital) | sections 2 et « Découpage par patiente » |
| 2 | Des exemples d'images par hôpital | section 3, grille 5 × 8 |
| 3 | Trois constats d'exploration | section 3 |
| 4 | Le score de la référence simple | section 4 |
| 5 | Un premier modèle par transfert, avec ses résultats par hôpital | sections 5 et 6 |
| 6 | Le plan pour passer du patch à la patiente | section 8 |

---

## 2. Déroulé conseillé (10 min)

| Temps | Diapo | Contenu |
|---|---|---|
| 0:00 | 1. Problème | Le pathologiste cherche des métastases dans les ganglions ; le stade pN en découle. Un faux négatif est une métastase manquée. 5 hôpitaux, des couleurs différentes. |
| 1:00 | 2. Données et découpage | Camelyon17-WILDS, sous-échantillon de 39 000 patchs. Découpage par patiente (voir §3). |
| 2:30 | 3. Images par hôpital | Grille de patchs normaux et tumoraux, histogrammes RGB. |
| 3:30 | 4. Trois constats | Voir §4. |
| 5:00 | 5. Référence simple | Histogrammes de couleur + régression logistique : scores par hôpital. |
| 6:00 | 6. Premier CNN | ResNet18 par transfert, 3 variantes de couleur : tableau ROC-AUC par hôpital, seuil retenu, matrice de confusion. |
| 7:30 | 7. Plan patch → patiente | Méthode de l'étendue, plafond de 88 %, règles pN vérifiées (voir §6). |
| 8:30 | 8. Suite et organisation | Prochaines étapes, répartition des rôles, questions à poser. |

---

## 3. Données et découpage

### Le jeu de données (vérifié sur les métadonnées)

| Hôpital | Rôle | Lames | Patientes | Patchs (WILDS complet) | Patchs par lame | Classes des lames (ITC / micro / macro) |
|---|---|---|---|---|---|---|
| H0 | entraînement | 10 | 7 | 59 436 | 1 596 – 13 637 | 3 / 3 / 4 |
| H3 | entraînement | 10 | 10 | 129 838 | 3 466 – 57 375 | 4 / 3 / 3 |
| H4 | entraînement | 10 | 9 | 146 722 | 4 914 – 61 110 | 5 / 2 / 3 |
| H1 | validation hors distribution | 10 | 8 | 34 904 | 888 – 10 404 | 2 / 5 / 3 |
| H2 | test, jamais vu | 10 | 9 | 85 054 | 3 694 – 31 878 | 2 / 4 / 4 |

Au total : 50 lames annotées, **toutes positives**, venant de 43 patientes. On compte 16 ITC, 17 micro-métastases et 17 macro-métastases. Aucune patiente n'a ses 5 ganglions annotés.

### Notre découpage

| Ensemble | Contenu | Patchs | Rôle |
|---|---|---|---|
| `entr` | H0, H3, H4, sans les patientes mises de côté | 21 727 (23 lames) | entraînement |
| `val_lames` | patientes entières mises de côté dans H0, H3 et H4 | 5 273 (7 lames) | performance « même distribution », sans fuite |
| `val_ood` | H1 | 4 000 (10 lames) | choix du modèle et des seuils |
| `test` | H2 | 8 000 (10 lames) | une seule fois, à la fin |

**Pourquoi ne pas garder `id_val` (le découpage officiel de WILDS) ?** Il vient des mêmes lames que `train`. Les patchs d'une même lame se ressemblent beaucoup, donc le score sur `id_val` mesure surtout la mémorisation des lames. Nous mettons de côté des patientes entières. Le notebook vérifie qu'aucune lame n'est à la fois en entraînement et en validation.

**Pourquoi tirer les patientes sous contrainte ?** La part de patchs tumoraux varie de 0,1 % à 93 % selon la lame. Un tirage purement au hasard pouvait donner une validation presque sans tumeur. On tire donc des patientes jusqu'à couvrir au moins 10 % et au plus 40 % des patchs de chaque classe de l'hôpital.

---

## 4. Trois constats d'exploration (à illustrer)

1. **La couleur trahit l'hôpital.** Les histogrammes RGB diffèrent nettement d'un hôpital à l'autre. Une régression logistique sur la seule couleur reconnaît l'hôpital de lames jamais vues avec **79,0 %** de bonnes réponses, contre 20 % au hasard (section 3).
   → Un modèle peut apprendre l'hôpital plutôt que les cellules. D'où les variantes de couleur et Grad-CAM.
2. **Les lames sont très inégales.** Une lame peut compter de 888 à 61 110 patchs. À elle seule, la plus grosse lame (H4) a plus de patchs que tout l'hôpital H1 (34 904).
   → Il faut découper par lame ou par patiente, et regarder les résultats par hôpital, pas seulement en global.
3. **Le jeu est artificiellement équilibré.** WILDS contient 50 % de patchs tumoraux dans chaque hôpital, par construction : tous les patchs tumoraux ont été gardés, les normaux sous-échantillonnés. En clinique, la grande majorité des patchs sont normaux.
   → La précision mesurée ici sera bien plus basse en conditions réelles (tableau de la section 6).

Constat de réserve : la part de tumeur d'une lame va de 0,1 % à 93 %. Les lames ITC n'ont presque aucun patch tumoral, ce sont les cas les plus difficiles.

---

## 5. Résultats au niveau patch

### Référence simple (section 4)
48 variables par patch (16 classes de couleur × 3 canaux), standardisées, puis régression logistique.

| | ROC-AUC | Rappel (seuil 0,5) |
|---|---|---|
| `val_lames` | 0,836 | 0,678 |
| `val_ood` (H1) | 0,761 | 0,716 |

### Premier CNN (sections 5 et 6)
ResNet18 pré-entraîné sur ImageNet. Entraînement en deux temps : 2 époques avec seule la dernière couche, puis 2 époques d'ajustement fin du dernier bloc. Patchs gardés en 96 × 96.

| Modèle | ROC-AUC `val_lames` H0 / H3 / H4 | ROC-AUC `val_ood` H1 |
|---|---|---|
| référence | 0,976 / 0,938 / 0,545 | 0,761 |
| `base` (retournements seuls) | 0,988 / 0,983 / 0,969 | 0,955 |
| `couleur` (+ variations de teinte et de contraste) | 0,988 / 0,982 / 0,980 | 0,961 |
| `gris` (niveaux de gris) | 0,977 / 0,974 / 0,980 | 0,951 |

- **Modèle retenu :** `couleur`. Il est choisi sur la meilleure ROC-AUC sur `val_ood`, jamais sur le test. Attention : 0,010 d'écart entre les trois variantes, sur une seule graine. On ne peut pas encore dire qu'une variante est meilleure.
- **Seuil :** 0,14. C'est le plus haut seuil qui garde un rappel d'au moins 95 % sur `val_ood`.
- **Au seuil retenu, sur `val_ood` :**
  - rappel : 0,951 (atteint par construction, puisque le seuil est réglé sur ces mêmes données) ;
  - spécificité : 0,771 ;
  - matrice de confusion : `figures/confusion_roc.png`.
- **Au même seuil, sur `val_lames` :** rappel de 0,984 (H0), 0,970 (H3) et 0,930 (H4). H4 passe sous la cible de 95 % : un seuil réglé sur un hôpital ne se transpose pas tel quel à un autre.
- **Écart entre `val_lames` et `val_ood` :** 0,983 (moyenne simple de H0, H3 et H4) contre 0,961, soit 0,022 de ROC-AUC. C'est la mesure de l'effet du changement d'hôpital.
- **Précision attendue si seulement 1 % des patchs sont tumoraux :** 4,0 %, soit 23,9 faux positifs pour 1 vrai positif (section 6).

Message clé : le CNN doit **battre la référence sur `val_ood`**, pas seulement sur `val_lames`. Sinon il n'apporte rien hors des hôpitaux vus. C'est le cas ici : 0,961 contre 0,761.

---

## 6. Plan patch → lame → patiente (déjà prototypé)

1. **Géométrie (WILDS, annexe E.2).** Les patchs font 96 px et sont posés tous les 32 px, à environ 0,96 µm par pixel. Le label vaut pour le carré central de 32 × 32 px, soit environ **31 µm** de côté.
2. **Étendue.** On marque comme tumorales les cases où la probabilité dépasse `SEUIL_LAME`, puis on regroupe les cases voisines en zones. Le diamètre de la plus grande zone donne la classe de la lame : ITC ≤ 0,2 mm, micro ≤ 2 mm, macro au-delà.
3. **Plafond de la méthode (vérifié).** Appliquée aux étiquettes vraies des 50 lames, la méthode retrouve la classe officielle de **44 lames sur 50 (88 %)**. Les 6 écarts sont des ITC classées micro : la grille de 31 µm est grossière, et elle ne compte pas les cellules (critère « moins de 200 cellules »). Aucun modèle ne fera mieux que ce plafond.
4. **Règles pN (vérifiées).** Codées dans le notebook, elles redonnent **100 %** des stades officiels des 100 patientes. Si l'on remplace les classes des 50 lames par celles du plafond, **5 patientes changent de stade, toutes vers un stade plus élevé** (sur-stadification, sans sous-stadification).
5. **Avec le modèle.** On prédit tous les patchs des 10 lames de H1 (environ 1,5 Go), puis on produit la carte de chaleur, la classe de chaque lame et l'effet sur le pN. Résultats : **9 lames bien classées sur 10**, et **1 stade pN changé sur 8 patientes, dont 0 sous-stadification**. L'unique erreur (lame 14, patiente 024, ITC classée micro, 0,22 mm) vient de la grille : les labels vrais donnent la même erreur. Ce score est **très optimiste, voir la première limite ci-dessous**.
6. **Test final.** Le même traitement sera appliqué une seule fois aux 10 lames de H2, à la fin.

**Limites à annoncer d'emblée :**
- **le 9/10 est gonflé par la construction de WILDS (mesuré sur H1).** Les patchs normaux y sont clairsemés : chacun a en moyenne 0,07 à 0,13 voisin sur 8. Les faux positifs restent donc isolés et ne forment pas de grande zone. Au seuil de 0,14, 23 % des patchs normaux sont pourtant marqués tumoraux (11 % au seuil de 0,5), ce qui donne de 111 à 697 zones par lame, mais la plus grande zone reste presque toujours la vraie tumeur. D'où un score de lames identique (90 %) pour les seuils 0,14, 0,5, 0,7 et 0,9. Sur une lame entière, où la grille est complète, ces faux positifs se toucheraient et l'étendue serait surestimée. Ce résultat ne vaut donc pas performance clinique ;
- les lames sont toutes positives et aucune patiente n'est complète, donc on mesure l'effet de nos erreurs sur le stade pN, pas un vrai stade prédit ;
- `SEUIL_LAME` est réglé sur H1, donc le résultat par lame y est optimiste.

---

## 7. Questions probables et réponses

| Question | Réponse courte |
|---|---|
| Pourquoi ne pas utiliser `id_val` ? | Il partage les lames de l'entraînement, ce qui crée une fuite. Nous découpons par patiente (`val_lames`). |
| Pourquoi régler sur H1 et pas sur H2 ? | H2 est le test : y régler quoi que ce soit fausserait la mesure finale. Il ne sert qu'une fois. |
| Pourquoi un rappel de 95 % ? | Un faux négatif est une métastase manquée, ce qui peut mener à une sous-stadification et un traitement insuffisant. Un faux positif est revu par le pathologiste. C'est un choix à discuter avec un clinicien. |
| Et la précision ? | Elle chute quand la tumeur devient rare : voir le tableau de la section 6. L'outil sert au tri et à l'aide, pas au diagnostic seul. |
| Pourquoi ne pas pondérer les classes ? | Le sous-échantillon est équilibré à 50/50. Le déséquilibre clinique est traité par le choix du seuil et la discussion de la précision. |
| Pourquoi ResNet18 en 96 × 96 ? | Modèle léger, rapide sur un GPU T4 gratuit, et taille native des patchs. Passer à 224 × 224 est prévu en comparaison (paramètre `TAILLE`). |
| Le modèle regarde-t-il la couleur ? | On le vérifie de trois façons : Grad-CAM (section 7), les résultats par hôpital, et la variante `gris` qui retire la couleur. |
| Comment passez-vous en millimètres ? | 32 px × 0,96 µm/px ≈ 31 µm par case. Source : WILDS, annexe E.2. La résolution exacte (0,23 à 0,25 µm/px avant réduction) est un paramètre. |
| Les résultats sont-ils stables ? | Pas encore mesuré : plusieurs graines sont prévues d'ici le rendu. Le choix de la variante `couleur` reste donc provisoire. |
| 9 lames sur 10, c'est un bon résultat ? | Non, pas tel quel. Il est identique du seuil 0,14 au seuil 0,9, car les patchs normaux de WILDS sont trop clairsemés pour que les faux positifs se regroupent. Il faudrait des lames entières pour mesurer la vraie performance. |
| Cadre réglementaire ? | Un tel outil est un dispositif médical, et un système à haut risque au sens de l'AI Act. Il exige validation clinique, traçabilité et supervision humaine. Le pathologiste reste décisionnaire. |

---

## 8. Suite jusqu'au rendu (jeudi 26 novembre, 18h00)

- [ ] Plusieurs graines : l'écart entre variantes est-il plus grand que l'écart entre graines ?
- [ ] Normalisation de la coloration (Macenko ou Reinhard) et/ou entrée en 224 × 224, mesurées par hôpital.
- [ ] Meilleure estimation de l'étendue : lissage des probabilités, taille minimale de zone, justification de `SEUIL_LAME`.
- [ ] Analyse Grad-CAM des faux négatifs.
- [ ] Test final sur H2, **une seule fois**.
- [ ] Rapport (15 pages max), supports de soutenance (10 min), README de reproduction, annexe « Usage de l'IA ».

### Organisation du groupe (à compléter, c'est noté)

| Membre | Rôle |
|---|---|
| `[nom]` | données et exploration |
| `[nom]` | modèles et évaluation |
| `[nom]` | patch → patiente |
| `[nom]` | rapport, éthique, réglementation |

Rappel : chaque membre doit pouvoir expliquer **tout** le projet, car la note de soutenance est individuelle.

---

## 9. Questions à poser à l'enseignant

1. Le découpage par patiente à la place de `id_val` lui convient-il ?
2. Le rappel minimal de 95 % est-il un objectif raisonnable, ou attend-il une analyse coût par coût ?
3. Pour le niveau patiente, mesurer l'effet de nos erreurs sur le pN suffit-il, ou attend-il l'option avancée (lames entières, lames négatives) ?
4. Quel canal utiliser pour le dépôt du 26 novembre ?

---

## 10. Checklist avant le rendez-vous

- [x] Notebook exécuté en entier avec les sorties enregistrées (Mac M2, 6 novembre).
- [x] Toutes les valeurs `[à remplir]` complétées.
- [x] Captures prêtes dans `figures/` : grille d'images, histogrammes RGB, matrice de confusion, Grad-CAM, cartes de chaleur. Le tableau ROC-AUC est au §5.
- [x] Test H2 **non lancé** (`LANCER_TEST = False`).
- [ ] Usage de l'IA noté au fur et à mesure (outil, tâches, ce que le groupe a vérifié lui-même).
