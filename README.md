<div align="center">

# 🌱 TRAVAIL PRATIQUE D'ANOVA  
**Analyse de la variance à mesures répétées appliquée à la croissance des plantes**

*Projet ENSAE – ISE 2*

[![R](https://img.shields.io/badge/R-4.3+-276DC3.svg)](https://www.r-project.org/)
[![Quarto](https://img.shields.io/badge/Quarto-Book-3C6E71.svg)](https://quarto.org/)
[![Status](https://img.shields.io/badge/Statistiques-ANOVA_à_mesures_répétées-2C3E50.svg)](#-à-propos)
[![Version](https://img.shields.io/badge/version-1.0.0-18BC9C.svg)](https://github.com/djerakei221/anova)

**▶ [Accéder au livre en ligne](https://djerakei221.github.io/anova/)**

---

</div>

## 📋 Table des Matières

- [À Propos](#-à-propos)
- [Contexte académique](#-contexte-académique)
- [Objectifs du travail pratique](#-objectifs-du-travail-pratique)
- [Structure du livre](#-structure-du-livre)
- [Jeu de données](#-jeu-de-données)
- [Méthodes statistiques](#-méthodes-statistiques)
- [Structure du projet Quarto](#-structure-du-projet-quarto)
- [Installation et rendu local](#-installation-et-rendu-local)
- [Publication GitHub Pages](#-publication-github-pages)
- [Références](#-références)

---

## 📖 À Propos

Ce dépôt contient un **livre Quarto** intitulé :

> **TRAVAIL PRATIQUE D'ANOVA – Analyse de la variance à mesures répétées appliquée à la croissance des plantes**

Le projet illustre, sur un **jeu de données réelles de croissance de plantes**, l’utilisation de :

- l’**ANOVA à mesures répétées**,  
- des **modèles mixtes de type split-plot**,  
- les **vérifications des hypothèses** (normalité, homogénéité, sphéricité),  
- les **comparaisons post-hoc** (Tukey, moyennes marginales, etc.).

L’objectif est de proposer un **support pédagogique complet**, mêlant théorie, exploration descriptive et application détaillée en R.

---

## 🎓 Contexte académique

Travail réalisé par des **Élèves Ingénieurs Statisticiens Économistes (ISE‑2)** dans le cadre de l’**École nationale de la Statistique et de l’Analyse économique Pierre NDIAYE (ENSAE)**.

Le livre est pensé comme un **complément de cours** sur :

- les **modèles linéaires**,  
- l’**ANOVA à mesures répétées**,  
- et l’**interprétation appliquée** des résultats sur un cas concret de croissance de plantes selon le **fertilisant**, la **variété** et la **période**.

---

## 🎯 Objectifs du travail pratique

| Objectif | Description |
|----------|-------------|
| **Comprendre le cadre de l’ANOVA à mesures répétées** | Présenter le contexte, les hypothèses et les principales variantes (facteur intra‑sujet, modèles mixtes). |
| **Décrire le jeu de données** | Explorer la hauteur des plantes par fertilisant, variété et période (statistiques descriptives, graphiques). |
| **Mettre en œuvre une ANOVA à mesures répétées** | Ajuster des modèles avec `afex::aov_car` et `car::Anova`, interpréter F, p-values, tailles d’effet. |
| **Vérifier les hypothèses** | Normalité des résidus, homogénéité des variances (Levene), sphéricité (Mauchly, GG, HF). |
| **Réaliser des comparaisons post-hoc** | Estimer des moyennes marginales (`emmeans`), effectuer des tests de Tukey globaux et par période. |
| **Relier théorie et pratique** | Faire le lien entre le cadre théorique (chapitre 1) et l’analyse complète du jeu de données (chapitres 2 et 3). |

---

## 📚 Structure du livre

Le livre est organisé en trois chapitres principaux :

1. **Chapitre 1 – Théorie sur l’ANOVA à mesures répétées**  
   Contexte, définition, hypothèses (normalité, homogénéité, sphéricité), test de Mauchly, corrections de Greenhouse–Geisser / Huynh–Feldt, modèle mixte (split‑plot) et lien avec les chapitres appliqués.

2. **Chapitre 2 – Analyse descriptive des variables d’étude**  
   Import du fichier `donnees/donnees.xlsx`, construction de l’identifiant sujet, tableaux de moyennes/écarts‑types, graphiques de distribution et d’interaction (fertilisant × variété × période).

3. **Chapitre 3 – Facteurs explicatifs de la variable dépendante**  
   Ajustement d’une ANOVA à mesures répétées sur la hauteur, vérification des hypothèses, tests de Levene et Mauchly, comparaisons post‑hoc avec `emmeans`, modèles type DABIRE avec `car::Anova`, et conclusion sur les combinaisons fertilisant × variété × période les plus performantes.

---

## 🌾 Jeu de données

Le jeu de données se trouve dans `donnees/donnees.xlsx` et contient notamment :

- `fertilisant` : type de terreau (Ma, Ca, An),  
- `variete` : variété de plante (Var1, Var2),  
- `periode` : temps de mesure (T1, T2, T3, T4),  
- `hauteur` : hauteur des plantes (variable dépendante),  
- `id` : identifiant technique de plante (sujet) pour les mesures répétées.

Les scripts des chapitres 2 et 3 s’occupent de :

- nettoyer les données (`hauteur` en numérique, gestion des virgules),  
- construire un identifiant sujet adapté aux modèles à mesures répétées,  
- mettre les facteurs au bon format (`factor` avec niveaux ordonnés si nécessaire).

---

## 📐 Méthodes statistiques

Les principales méthodes utilisées sont :

- **ANOVA à mesures répétées** (`afex::aov_car`)  
  - Effets fixes : fertilisant, variété, période et interactions ;  
  - Terme d’erreur : `Error(id / periode)` ;  
  - Type III des sommes de carrés ;  
  - Tests F, p‑values, tailles d’effet (η² partiel).

- **Modèles multivariés à mesures répétées** (`car::Anova`)  
  - Décomposition par période,  
  - Tests multivariés (Wilks) sur l’effet du temps et les interactions.

- **Diagnostics**  
  - Normalité des résidus : Q–Q plots, histogrammes, test de Shapiro–Wilk ;  
  - Homogénéité des variances : test de Levene (`car::leveneTest`) ;  
  - Sphéricité : test de Mauchly et corrections GG / HF.

- **Comparaisons post‑hoc** (`emmeans`)  
  - Moyennes marginales fertilisant × variété,  
  - Contrastes de Tukey globaux,  
  - Contrastes de Tukey par période (T1–T4).

---

## 🏗️ Structure du projet Quarto

```text
PROJET_ANOVA/
├── _quarto.yml                  # Configuration du livre (chapitres, formats, output-dir=docs)
├── index.qmd                    # Page d’accueil / présentation générale
├── 01-theorie-anova-mesures-repetees.qmd
├── 02-analyse-descriptive.qmd
├── 03-facteurs-explicatifs.qmd
├── donnees/
│   └── donnees.xlsx             # Jeu de données de croissance des plantes
├── images/                      # Logos, illustrations
│   ├── ansd.png
│   └── ensae.png
├── styles.css                   # Styles personnalisés pour le livre
└── docs/                        # Site généré par Quarto (GitHub Pages)
    ├── index.html
    ├── 01-theorie-anova-mesures-repetees.html
    ├── 02-analyse-descriptive.html
    ├── 03-facteurs-explicatifs.html
    └── site_libs/...
```

---

## 💻 Installation et rendu local

### Prérequis

- **R** ≥ 4.3  
- **Quarto CLI** (`https://quarto.org`)  
- (Optionnel) **RStudio** pour un confort maximal.

### Cloner le dépôt

```bash
git clone https://github.com/djerakei221/anova.git
cd anova
```

### Rendre le livre en local

```bash
quarto render
```

- Le site est généré dans le dossier `docs/`.
- Pour le prévisualiser localement :

```bash
quarto preview
```

---

## 🌐 Publication GitHub Pages

Le livre est déployé automatiquement via **GitHub Pages** :

1. Dans `_quarto.yml` :

   ```yaml
   project:
     type: book
     output-dir: docs
   ```

2. Après chaque modification :

   ```bash
   quarto render
   git add .
   git commit -m "Mise à jour du livre"
   git push
   ```

3. Sur GitHub : **Settings → Pages**  
   - Source : `Deploy from a branch`  
   - Branch : `main`  
   - Folder : `/docs`

GitHub sert alors le livre à l’adresse :  
👉 `https://djerakei221.github.io/anova/`

---

## 📖 Références

- **Quarto** : [https://quarto.org](https://quarto.org)  
- **afex** (ANOVA à mesures répétées) : [https://github.com/singmann/afex](https://github.com/singmann/afex)  
- **car** (modèles à mesures répétées multivariés) : [https://cran.r-project.org/package=car](https://cran.r-project.org/package=car)  
- **emmeans** (moyennes marginales et post-hoc) : [https://cran.r-project.org/package=emmeans](https://cran.r-project.org/package=emmeans)  

---

<div align="center">

### 📅 Dernière mise à jour : 2026

**École nationale de la Statistique et de l’Analyse économique Pierre NDIAYE (ENSAE)**  

[⬆ Retour en haut](#-table-des-matières)

</div>


