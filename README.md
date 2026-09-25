# 📊 Orange D4D — Mobilité & COVID-19 en Côte d’Ivoire

## Méthodologie de préparation et d’ajustement des données de mobilité

Ce dépôt présente une **méthodologie de préparation, d’enrichissement et d’ajustement de données de mobilité en Côte d’Ivoire**, développée à partir des données du **Orange D4D Challenge** et de plusieurs sources complémentaires.

L’objectif est de transformer des données historiques de mobilité issues de 2011–2012 en une base exploitable pour l'étude de la mobilité pendant la période **COVID-19**, notamment dans le cadre de simulations et de modèles épidémiologiques.

> **Note :** ce dépôt documente principalement la méthodologie de préparation et de construction du dataset final. Il ne contient pas l'ensemble du projet de modélisation épidémiologique ni les données brutes utilisées dans le projet original.

---

## 🎯 Objectif

Le principal problème rencontré est l'écart temporel entre les données de mobilité disponibles et la période étudiée.

Les données Orange D4D disponibles décrivent des flux de mobilité datant de **2011–2012**, alors que l'objectif du projet est d'étudier la dynamique de mobilité pendant la pandémie de **COVID-19 à partir de 2020**.

La méthodologie mise en place cherche donc à construire une estimation des flux de mobilité contemporains en combinant :

* les flux historiques Orange D4D ;
* l'évolution démographique de la Côte d'Ivoire ;
* les variations de mobilité observées pendant la COVID-19 ;
* les données réelles de cas de COVID-19.

---

# 🗂️ 1. Données de départ : Orange D4D

Les données initiales proviennent du **Orange D4D Challenge**, qui contient notamment des informations issues du réseau mobile en Côte d'Ivoire.

Le dossier initial contient plusieurs fichiers, notamment :

```text
dat/
├── Abidjan.Commune.CallNetwork.txt
├── cities_departmentID.csv
├── cities_regionID.csv
├── region_stats.txt
├── pole_pop.txt
├── pole_tower.txt
└── autres fichiers TXT
```

Les données sont réparties dans plusieurs fichiers et nécessitent une phase de consolidation et de structuration.

---

# 🔍 2. Identification d'un fichier consolidé

Un fichier Excel consolidé a également été identifié :

```text
D4D_MOBILITE_CI_FINAL.xlsx
```

Il contient notamment les feuilles suivantes :

| Feuille                 | Description                 |
| ----------------------- | --------------------------- |
| `flux_departements`     | Flux entre départements     |
| `flux_sous_prefectures` | Flux entre sous-préfectures |
| `referentiel_geo`       | Référentiel géographique    |
| `mapping_antennes`      | Correspondance des antennes |
| `stats_globales`        | Statistiques générales      |
| `top_departements`      | Principaux départements     |
| `top_corridors`         | Principaux corridors        |
| `metadonnees`           | Documentation               |

La feuille `flux_departements`, contenant environ **2 450 corridors**, constitue la base principale utilisée pour l'analyse des flux entre départements.

---

# 🧭 3. Reconstruction de la mobilité

La construction des flux repose sur plusieurs étapes.

### A. Géolocalisation des antennes

Les fichiers contenant les identifiants de département et de région sont combinés afin de construire un référentiel géographique des antennes.

Environ **1 231 antennes** sont ainsi associées à des informations géographiques.

### B. Comptage des déplacements

Les données de handover permettent d'identifier les changements d'antennes des utilisateurs.

Lorsqu'un utilisateur passe :

```text
Antenne A → Antenne B
```

et que les antennes appartiennent respectivement aux départements :

```text
Département X → Département Y
```

le déplacement est agrégé dans le corridor :

```text
X → Y
```

### C. Enrichissement géographique

Les flux sont ensuite enrichis avec :

* les coordonnées géographiques des départements ;
* les distances entre départements ;
* les flux journaliers moyens.

La distance peut notamment être calculée à partir des coordonnées géographiques selon la formule de **Haversine**.

### D. Agrégation

Des statistiques complémentaires sont ensuite produites :

* volumes de mobilité ;
* moyennes ;
* principaux départements ;
* principaux corridors.

---

# ⚠️ 4. Problème temporel

Un problème majeur apparaît lors de la comparaison des périodes :

```text
Données de mobilité : 2011–2012
                 ↓
Objectif :        2020+
```

Il existe donc un décalage d'environ **8 à 9 ans** entre les observations de mobilité et la période COVID-19 étudiée.

Une stratégie d'ajustement est donc utilisée pour produire une estimation des flux contemporains.

---

# 🌍 5. Données complémentaires

Trois sources complémentaires sont intégrées.

## Google COVID-19 Mobility Reports

Les rapports de mobilité Google permettent d'obtenir les variations de mobilité observées pendant la pandémie.

Les données sont filtrées pour conserver uniquement :

```text
Côte d'Ivoire
```

La base filtrée contient environ **11 705 observations quotidiennes**.

Les principales catégories comprennent notamment :

* commerces ;
* transports ;
* lieux de travail ;
* autres catégories de mobilité.

---

## 🦠 Our World in Data

Les données **Our World in Data** sont utilisées pour obtenir les observations COVID-19 nécessaires à la validation.

Les données filtrées pour la Côte d'Ivoire contiennent notamment :

* date ;
* nouveaux cas ;
* cas cumulés ;
* décès ;
* population.

---

## 👥 Banque mondiale

Les données de population de la **Banque mondiale** sont utilisées afin d'estimer l'évolution démographique entre la période des données D4D et la période étudiée.

L'indicateur utilisé est :

```text
SP.POP.TOTL
```

La méthodologie documentée utilise notamment :

```text
Population 2012 : 23 467 078
Population 2020 : 28 915 449
```

soit un facteur d'ajustement démographique d'environ :

```text
1,2322
```

---

# 🧹 6. Nettoyage des données

## Google Mobility

Le fichier original étant volumineux, seules les données nécessaires à la Côte d'Ivoire sont conservées.

Le processus comprend :

1. Filtrage du pays ;
2. Sélection des variables utiles ;
3. Renommage des colonnes ;
4. Agrégation mensuelle.

Les variations quotidiennes sont ensuite transformées en facteurs multiplicatifs mensuels.

Par exemple :

```text
Variation moyenne = -33 %

Facteur = 1 + (-33 / 100)

Facteur = 0,67
```

Ainsi, une variation de mobilité de -33 % correspond à un facteur de **0,67**.

---

# 🔄 7. Ajustement des flux historiques

La méthodologie repose sur un **double ajustement**.

## Ajustement démographique

Les flux historiques sont d'abord ajustés selon l'évolution de la population :

```text
Flux 2020 de base =
Flux 2012 × Facteur démographique
```

avec :

```text
Facteur démographique = 1,2322
```

### Exemple

Pour un flux historique de :

```text
35 872 déplacements/jour
```

on obtient :

```text
35 872 × 1,2322
≈ 44 203 déplacements/jour
```

---

## Ajustement lié à la mobilité COVID-19

Le flux ajusté démographiquement est ensuite multiplié par le facteur de mobilité correspondant à la période étudiée :

```text
Flux final =
Flux 2012 × Facteur démographique × Facteur mobilité
```

### Exemple : avril 2020

Avec un facteur de mobilité de :

```text
0,67
```

on obtient :

```text
44 203 × 0,67
≈ 29 616 déplacements/jour
```

Cette approche permet ainsi de représenter les variations temporelles de mobilité associées à la pandémie.

---

# 📦 8. Dataset final

La méthodologie aboutit à un fichier consolidé :

```text
DONNEES_FINALES_HACKATHON.xlsx
```

Il est organisé autour de cinq ensembles de données.

| Feuille                 | Contenu           | Utilisation              |
| ----------------------- | ----------------- | ------------------------ |
| `flux_ajuste_2020`      | Flux ajustés      | Base de mobilité         |
| `google_mobility`       | Mobilité COVID    | Variations temporelles   |
| `covid_ci`              | Données COVID     | Validation               |
| `facteurs_ajustement`   | Facteurs mensuels | Ajustement des flux      |
| `population_officielle` | Population        | Ajustement démographique |

---

# 📊 9. Résultats de la préparation

La méthodologie permet d'obtenir une base structurée comprenant :

* environ **50 départements** ;
* environ **2 450 corridors de mobilité** ;
* des flux ajustés à partir de la croissance démographique ;
* des facteurs de mobilité mensuels ;
* des données COVID-19 pour la validation ;
* des données démographiques officielles.

Le flux journalier total ajusté pour la période de référence est estimé à environ :

```text
1,97 million de déplacements/jour
```

---

# 🔬 10. Utilisation pour la modélisation épidémiologique

Le dataset final peut être utilisé comme entrée pour des modèles de propagation épidémique prenant en compte la mobilité entre territoires.

Un exemple de workflow est :

```text
Flux de mobilité
       │
       ▼
Ajustement démographique
       │
       ▼
Ajustement COVID-19
       │
       ▼
Matrice de mobilité
       │
       ▼
Modèle épidémiologique
       │
       ▼
Simulation de propagation
       │
       ▼
Comparaison avec les cas réels
```

La méthodologie peut notamment servir de base à un modèle **SEIR multi-compartiments** intégrant les déplacements entre départements.

---

# ⚠️ 11. Limites et hypothèses

Cette méthodologie repose sur plusieurs hypothèses importantes.

### 1. Stabilité de la structure spatiale

La structure relative des flux observée en 2011–2012 est supposée rester suffisamment représentative pour servir de base aux estimations ultérieures.

### 2. Ajustement démographique uniforme

Le facteur démographique est appliqué uniformément aux flux.

Cela suppose que l'évolution de la population se traduit proportionnellement par une évolution des flux.

### 3. Mobilité agrégée

Les données Google Mobility sont disponibles à un niveau agrégé et ne décrivent pas directement les flux entre chaque département.

### 4. Données COVID nationales

Les données COVID utilisées pour la validation ne fournissent pas nécessairement le même niveau de granularité géographique que les flux de mobilité.

### 5. Flux estimés

Les flux obtenus pour la période COVID-19 sont des **estimations construites à partir de données historiques et de facteurs d'ajustement**. Ils ne doivent pas être interprétés comme des observations directes des déplacements de 2020.

---

# 📚 12. Sources

Les principales sources utilisées sont :

1. **Orange D4D Challenge** — données historiques de mobilité en Côte d'Ivoire
2. **Google COVID-19 Community Mobility Reports** — variations de mobilité
3. **Our World in Data** — données COVID-19
4. **Banque mondiale** — données démographiques

---

# 🔭 Perspectives

Les prochaines étapes possibles comprennent :

* Construction d'une matrice de mobilité départementale dynamique ;
* Intégration des flux dans un modèle SEIR ;
* Calibration du modèle sur les données COVID-19 observées ;
* Analyse de scénarios de restriction de mobilité ;
* Comparaison de différentes stratégies d'ajustement ;
* Amélioration de la granularité spatiale des données ;
* Intégration de données de mobilité plus récentes lorsque disponibles.

---

## 👤 Projet

**Orange D4D — COVID-19 Mobility Analysis in Côte d'Ivoire**

Ce dépôt documente la méthodologie de préparation et d'ajustement des données de mobilité utilisée comme base pour une analyse de la propagation du COVID-19 en Côte d'Ivoire.
