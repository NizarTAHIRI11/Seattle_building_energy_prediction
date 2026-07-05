# Seattle Building Energy Prediction

> Prédiction de la consommation énergétique de bâtiments non résidentiels à Seattle, à partir de leurs caractéristiques structurelles — modèle exposé via une API BentoML, conteneurisée et déployée sur Google Cloud Run.
> *Predicting energy consumption of non-residential buildings in Seattle from structural features — model served via a BentoML API, containerized and deployed on Google Cloud Run.*

![Python](https://img.shields.io/badge/Python-3.x-blue)
![scikit--learn](https://img.shields.io/badge/scikit--learn-RandomForest-orange)
![BentoML](https://img.shields.io/badge/BentoML-API-informational)
![Docker](https://img.shields.io/badge/Docker-container-2496ED)
![Cloud Run](https://img.shields.io/badge/Google%20Cloud-Run-4285F4)

## Sommaire
- [Contexte](#contexte)
- [Objectifs](#objectifs)
- [Données](#données)
- [Méthodologie](#méthodologie)
- [API](#api--déploiement-du-modèle)
- [Docker](#docker)
- [Déploiement Cloud](#déploiement-cloud)
- [Structure du projet](#structure-du-projet)
- [Installation locale](#installation-locale)
- [Stack technique](#stack-technique)

## Contexte
Dans le cadre de la stratégie de la ville de Seattle visant la **neutralité carbone d'ici 2050**, ce projet analyse et modélise la consommation énergétique des bâtiments non résidentiels, à partir de relevés réalisés en **2016**.

L'objectif est de construire un modèle capable de **prédire la consommation d'énergie** d'un bâtiment à partir de ses caractéristiques structurelles.

## Objectifs
- Analyse exploratoire (EDA) du dataset
- Feature engineering
- Comparaison de plusieurs modèles de machine learning
- Optimisation du modèle sélectionné (GridSearchCV)
- Identification des variables les plus influentes
- Déploiement du modèle via une **API BentoML**
- Conteneurisation avec **Docker**
- Déploiement sur **Google Cloud Run**

## Données
- **Source** : Ville de Seattle — *2016 Building Energy Benchmarking*
- **Fichier** : `data/2016_Building_Energy_Benchmarking.csv`
- **Contenu** : caractéristiques des bâtiments (surface, usage, année de construction...), consommation énergétique, émissions de CO₂

## Méthodologie

### 1. Analyse exploratoire & nettoyage
- Suppression des colonnes trop incomplètes (>50% de valeurs manquantes)
- Récupération du `ZipCode` manquant par correspondance de coordonnées GPS
- Imputation par médiane sur `ENERGYSTARScore`
- Suppression des lignes sans `NumberofBuildings` ou `TotalGHGEmissions`

### 2. Feature engineering
Nouvelles variables créées :
- `BuildingAge` — âge du bâtiment (2016 − année de construction)
- `AreaPerFloor` — surface par étage
- `NbUses` — nombre de types d'usage du bâtiment
- `HasElectricity`, `HasGas` — indicateurs binaires de source d'énergie
- Encodage one-hot des variables catégorielles

### 3. Modélisation
Modèles comparés :
- Régression linéaire
- Random Forest
- SVR

Métriques d'évaluation : **R², MAE, RMSE**

### 4. Optimisation
`GridSearchCV` sur le Random Forest (`n_estimators`, `max_depth`, `min_samples_split`), validation croisée à 5 folds.

### 5. Interprétation
Analyse des **feature importances** pour identifier les 15 variables les plus influentes.

## API – Déploiement du modèle
API développée avec **BentoML**, exposant un endpoint de prédiction.

**Endpoint**
```
POST /predict
```

**Exemple de requête**
```json
{
  "ENERGYSTARScore": 75,
  "PropertyGFABuilding(s)": 120000,
  "LargestPropertyUseTypeGFA": 90000,
  "ZipCode": 98101,
  "NumberofFloors": 12,
  "YearBuilt": 1998,
  "PrimaryPropertyType": "Hotel",
  "LargestPropertyUseType": "Hotel"
}
```

**Réponse**
```json
{
  "prediction": 123.45
}
```

La validation des entrées est assurée par un schéma **Pydantic** (`service.py`), avec bornes réalistes (ex. `ZipCode` entre 98000 et 98300, `YearBuilt` ≤ 2016).

## Docker

**Build**
```bash
docker build -t api_energy .
```

**Run**
```bash
docker run -p 3000:3000 api_energy
```

## Déploiement Cloud
API déployée sur **Google Cloud Run** :
🔗 https://api-energy-610310238285.europe-west1.run.app/


## Structure du projet

```
seattle-building-energy-prediction/
│   .gitignore
│   Dockerfile
│   README.md
│   requirements.txt
│   test_api.http
│
├───data
│       2016_Building_Energy_Benchmarking.csv
│
├───notebook
│       template_modelistation_supervisee.ipynb
│
└───src
        best_model.py
        service.py
```

## Installation locale

```bash
git clone https://github.com/NizarTAHIRI11/seattle-building-energy-prediction.git
cd seattle-building-energy-prediction
pip install -r requirements.txt
python src/best_model.py
bentoml serve src/service:svc
```

## Stack technique
`Python` · `Pandas` · `NumPy` · `Scikit-learn` · `BentoML` · `Pydantic` · `Docker` · `Google Cloud Run`

## Remarque
Ce projet a une visée pédagogique et démontre :
- la construction d'un modèle de machine learning de bout en bout
- son exposition via une API
- son déploiement en production
