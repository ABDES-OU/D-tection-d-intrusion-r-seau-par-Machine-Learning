# Détection d'intrusion réseau par Machine Learning

Comparaison de **Random Forest**, **XGBoost** et **SVM (noyau RBF)** sur le dataset **CSE-CIC-IDS2018** (classification multiclasse : trafic Benign + 7 types d'attaques).

Projet réalisé dans le cadre du module Machine Learning du Master **CITEch** (Cybersécurité Intelligente et Technologies Émergentes), Faculté des Sciences de Rabat, Université Mohammed V.

- **Auteurs :** OUHMAN Abdessamad, MAZZI Mouaad
- **Encadrant :** Pr. LAANAYA Hicham
- **Année universitaire :** 2025-2026

---

## Objectif

Les systèmes de détection d'intrusion (IDS) basés sur des signatures échouent face aux attaques nouvelles ou polymorphes. Ce projet évalue la capacité de trois algorithmes supervisés à distinguer le trafic normal du trafic malveillant, et à identifier le type d'attaque, à partir de caractéristiques de flux réseau.

## Dataset

[CSE-CIC-IDS2018](https://www.unb.ca/cic/datasets/ids-2018.html) (Communications Security Establishment & Canadian Institute for Cybersecurity), capturé sur un environnement simulé AWS.

Quatre journées de capture ont été utilisées :

| Fichier | Attaques présentes |
|---|---|
| `02-14-2018.csv` | FTP / SSH Brute Force |
| `02-23-2018.csv` | Brute Force Web, XSS, SQL Injection |
| `03-01-2018.csv` | Infiltration |
| `03-02-2018.csv` | Bot |

- 3,47 M de lignes brutes, 80 features par flux réseau, 8 classes
- Déséquilibre extrême : environ 2,7 M de flux Benign contre seulement 53 lignes de SQL Injection

> **Le dataset n'est pas inclus dans ce dépôt** (fichiers trop volumineux). Téléchargez les quatre fichiers ci-dessus depuis le lien officiel et placez-les dans le même dossier que le notebook.

## Méthodologie

1. **Fusion** des quatre fichiers CSV
2. **Nettoyage** : lignes d'en-tête dupliquées, valeurs infinies et manquantes, valeurs négatives impossibles, doublons ; encodage One-Hot de `Protocol`. Résultat : 3 476 850 lignes brutes, environ 1,47 M de lignes après nettoyage (78 features numériques)
3. **Split train / test** 80 / 20, stratifié
4. **Sélection des 10 meilleures features** (`SelectKBest`, ANOVA F), ajustée sur le jeu d'entraînement uniquement
5. **Standardisation** (`StandardScaler`), ajustée sur le jeu d'entraînement uniquement
6. **Entraînement et évaluation** des trois modèles

Le split est volontairement fait **avant** la sélection de features et la standardisation, afin d'éviter toute fuite de données (*data leakage*) vers le jeu de test.

## Modèles

| Modèle | Configuration |
|---|---|
| Random Forest | 100 arbres de décision |
| XGBoost | Boosting de gradient |
| SVM | Noyau RBF, entraînée sur un échantillon de 50 000 lignes (coût de calcul) |

## Résultats (multiclasse, 8 classes)

| Modèle | Accuracy | F1 (pondéré) | ROC-AUC (OvR) |
|---|---|---|---|
| Random Forest | 0,9806 | 0,9752 | 0,9881 |
| **XGBoost** | **0,9813** | 0,9747 | **0,9900** |
| SVM (RBF) | 0,9782 | 0,9700 | 0,9719 |

### Ce que les moyennes cachent

L'accuracy globale (environ 98 %) masque de vraies lacunes sur les classes rares :

- **Benign, Bot, SSH Brute Force** : recall d'environ 99-100 %
- **Infiltration** : seulement 7 à 17 % des attaques détectées selon le modèle
- **SQL Injection** : très peu d'exemples ; la SVM n'en détecte aucun
- **Brute Force Web** : la SVM échoue totalement (0 %)

En cybersécurité, un bon score global ne garantit pas une bonne détection des menaces réelles : la performance **par classe** doit toujours être examinée.

### Conclusion

- XGBoost offre le meilleur compromis performance / rapidité
- Random Forest est une alternative robuste, avec un F1 pondéré quasi identique
- La SVM passe mal à l'échelle sur ce volume de données déséquilibrées

## Pistes d'amélioration

- Rééquilibrage des classes (SMOTE, sous-échantillonnage de Benign)
- Pondération des classes (`class_weight='balanced'`)
- Optimisation des hyperparamètres (GridSearch, Optuna)
- Autres familles de modèles (réseaux de neurones, détection d'anomalies)

## Installation et exécution

```bash
# 1. Cloner le dépôt
git clone <URL_DU_DEPOT>
cd <NOM_DU_DEPOT>

# 2. Installer les dépendances
pip install pandas numpy matplotlib seaborn scikit-learn xgboost jupyter

# 3. Placer les 4 fichiers CSV du dataset dans ce dossier

# 4. Lancer le notebook
jupyter notebook OUHMAN_ABDESSAMAD_MAZZI_MOUAAD_Projet_ML.ipynb
```

## Contenu du dépôt

```
.
├── OUHMAN_ABDESSAMAD_MAZZI_MOUAAD_Projet_ML.ipynb   # Notebook complet (EDA, nettoyage, modèles, résultats)
├── Présentation_OUHMAN_ABDESSAMAD_MAZZI_MOUAAD.pptx # Présentation du projet
└── README.md
```

## Références

- Sharafaldin, I., Lashkari, A. H., Ghorbani, A. A. *Toward Generating a New Intrusion Detection Dataset and Intrusion Traffic Characterization*, ICISSP 2018.
- Canadian Institute for Cybersecurity, CSE-CIC-IDS2018 : https://www.unb.ca/cic/datasets/ids-2018.html

## Remarque

Projet à but pédagogique. Les résultats dépendent du nettoyage appliqué, de la graine aléatoire et de la machine utilisée (notamment pour les temps d'entraînement).
