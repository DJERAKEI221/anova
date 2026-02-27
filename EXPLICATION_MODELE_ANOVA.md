# Explication détaillée du modèle ANOVA à mesures répétées

Ce document explique **détail par détail** le modèle utilisé dans le projet (chapitres 2 et 3) : structure des données, rôle de l’identifiant **id**, modèle statistique, vérifications et comparaisons post-hoc.

---

## 1. Contexte et objectif

- **Variable à expliquer** : **taille** (hauteur des plantes).
- **Facteurs** :
  - **groupe** (terreau) : Ma, Ca, An — type de terreau.
  - **variete** : Var1, Var2 — variété de plante.
  - **periode** : T1, T2, T3, T4 — moment de mesure (mesures répétées dans le temps).

On **répète les mesures de T1 à T4** pour **chaque variété** (Var1 et Var2), et ce dans **chaque groupe** (Ma, Ca, An). Donc pour chaque combinaison (groupe × variété), on a des mesures aux quatre périodes T1, T2, T3, T4. À chaque (groupe × variété) correspondent plusieurs plantes. Selon le protocole, deux cas sont possibles :

- **Cas A — Mesures répétées (même plante aux 4 périodes)** : la **même** plante est mesurée à T1, puis T2, puis T3, puis T4. Chaque plante a donc **4 lignes** (une par période). C’est le plan « mesures répétées » : le facteur *période* est **intra-sujet**. Le code et le modèle du projet (ANOVA mixte, **id**, Error(id/periode)) sont faits pour **ce cas**.
- **Cas B — Plantes différentes à chaque période** : à l’intérieur de chaque (groupe × variété), on mesure des **plantes différentes** à T1, à T2, à T3 et à T4. Chaque plante n’est alors mesurée qu’**une seule fois** (à une seule période). Le facteur *période* est alors **inter-sujets**. Il faudrait un modèle ANOVA à 3 facteurs entièrement inter-sujets (groupe × variete × periode), **sans** structure Error(id/periode).

**Important** : si votre protocole correspond au **cas B** (plantes différentes selon la période), le modèle actuel (ANOVA à mesures répétées avec Error(id/periode)) et la création d’**id** qui regroupe 4 lignes par plante **ne sont pas adaptés**. Il faut soit avoir des données au format « une ligne par plante » avec une seule période par plante, soit adapter le modèle (ANOVA 3 facteurs entre sujets). Ce document et le code des chapitres 2 et 3 décrivent le **cas A** (même plante mesurée 4 fois).

---

## 2. Structure des données (format long)

Les données sont en **format long** : une ligne = **une mesure** (une plante à une période).

| Variable   | Rôle                                      | Exemple |
|-----------|--------------------------------------------|---------|
| **groupe**  | Facteur : type de terreau                  | Ma, Ca, An |
| **variete** | Facteur : variété                          | Var1, Var2 |
| **periode** | Facteur : moment (T1, T2, T3, T4)          | T1, T2, T3, T4 |
| **taille**  | Variable dépendante (numérique)            | 31.6, 8.5, … |
| **id**      | Identifiant de l’individu qui se répète = groupe_variete_numéro (chaque plante) | Ma_Var1_1, Ma_Var1_2, Ca_Var2_1, … |

- **periode** est **une seule variable** dont les **modalités** sont T1, T2, T3, T4. On ne crée pas quatre colonnes T1, T2, T3, T4.
- **On répète les mesures de T1 à T4 pour les variétés Var1 et Var2** (dans chaque groupe). Donc pour chaque (groupe × variété), on a des mesures aux 4 périodes. Dans le **cas mesures répétées** (cas A) : la même plante est mesurée à T1, T2, T3, T4 et apparaît sur **4 lignes** (une par période) ; elle appartient à **un seul** groupe et **une seule** variété. Si on mesure des **plantes différentes** à chaque période (cas B), chaque plante n’a qu’**une** ligne (voir section 1).

---

## 3. Création des identifiants **id** dans ce cadre

**Cadre** : les mesures sont **répétées de T1 à T4 pour Var1 et Var2** dans **chaque groupe** (Ma, Ca, An). L’**individu qui se répète** est la combinaison **(groupe × variété)** : **id** = **groupe_variete_numéro** (plusieurs plantes par groupe×variété ; traitement, pas l’id).

### 3.1 À quoi sert **id** ?

- **id** identifie l’**individu qui se répète** : la combinaison **(groupe × variete)** Il faut **plusieurs plantes par (groupe × variete)** pour que le modèle ait des ddl résiduels > 0 (sinon F, p et ESM = NaN/NA).
- Sans **id**, le logiciel ne sait pas que les 4 lignes (T1, T2, T3, T4) d’une même plante appartiennent au **même individu**. Il les traiterait comme 4 individus différents.
- Avec **id**, on dit explicitement : « ces 4 mesures sont des **mesures répétées** sur le **même** sujet ». Le modèle peut alors :
  - estimer la variabilité **entre sujets** (différences entre plantes),
  - et la variabilité **à l’intérieur d’un sujet** (évolution dans le temps),
  - et utiliser le bon terme d’erreur pour tester la **période** et les interactions avec la période.

En résumé : **id** = une valeur par **plante** ; indispensable pour que l’ANOVA à mesures répétées soit correcte (plan « sujet × période »).

### 3.2 Quand crée-t-on **id** ?

- Si le fichier contient **déjà** une colonne **id**, on la garde.
- Si **id** est **absent**, on le **crée** en numérotant les plantes dans chaque (groupe, variete) puis **id** = `groupe_variete_numéro`.

### 3.3 Comment **id** est-il créé (dans ce cadre) ?

**Dans ce cadre** : les mesures sont répétées **T1 à T4 pour Var1 et Var2** dans chaque groupe ; chaque plante a **4 lignes** (une par période). Pour créer un **id** unique par plante, on numérote les plantes **à l’intérieur de chaque (groupe, variete)** puis **id** = `groupe_variete_numéro`. Les données sont en format long : plusieurs lignes par plante (une par période). Il faut regrouper les lignes qui correspondent à la **même** plante.

**Idée** : les lignes d’une même plante ont le même **groupe** et la même **variete**, et une ligne pour chaque **periode** (T1, T2, T3, T4). Donc, pour un couple (groupe, variete), les lignes sont ordonnées (ex. par période). Le 1er T1, le 1er T2, le 1er T3, le 1er T4 correspondent à la **plante n°1** ; le 2e T1, 2e T2, 2e T3, 2e T4 à la **plante n°2**, etc.

**Code utilisé :**

**Code** : ordonner par (groupe, variete, periode), puis dans chaque (groupe, variete) : `within_subj = (row_number() - 1) %/% n_periodes + 1`, puis `id = paste0(groupe, "_", variete, "_", within_subj)`. On concatène **groupe** et **variete** ; tous les enregistrements d’un même (groupe, variete) ont le **même id**. Résultat : 6 identifiants (Ma_Var1, Ma_Var2, Ca_Var1, Ca_Var2, An_Var1, An_Var2).

**Exemple concret (après création de id) :**

| groupe | variete | periode | taille | id      |
|--------|---------|---------|--------|---------|
| Ma     | Var1    | T1      | 31.6   | Ma_Var1_1 |
| Ma     | Var1    | T2      | 33.2   | Ma_Var1_1 |
| Ma     | Var1    | T3      | 35.1   | Ma_Var1_1 |
| Ma     | Var1    | T4      | 37.0   | Ma_Var1_1 |
| Ma     | Var1    | T1      | 30.1   | Ma_Var1_2 |
| …      | …       | …       | …      | …       |

**Résumé** : **id** = une valeur par **plante** (groupe_variete_numéro). Les mesures répétées T1–T4 sont reliées par **id** à la même plante. **id** est créé seulement si la colonne **id** est absente du fichier.

---

## 4. Le modèle ANOVA (formule et sens)

### 4.1 Formule utilisée (R / afex)

En termes de modèle linéaire à effets fixes + structure d’erreur, on a en substance :

- **taille ~ groupe × variete + Error(id / periode)**

Interprétation :

- **groupe × variete** : effets **fixes** (inter-sujets) — effet du groupe, effet de la variété, interaction groupe×variete. Ils expliquent les différences **entre plantes** (entre les combinaisons terreau×variété).
- **Error(id / periode)** : la variabilité est décomposée en :
  - **id** : variabilité **entre sujets** (entre plantes) — chaque plante a son propre niveau moyen.
  - **id:periode** (ou effet « période dans id ») : variabilité **intra-sujet** (évolution dans le temps, spécifique à chaque plante). C’est sur ce terme qu’on teste **periode** et les interactions **groupe×periode**, **variete×periode**, **groupe×variete×periode**.

### 4.2 Facteurs « inter-sujets » et « intra-sujet »

- **Inter-sujets (between)** : **groupe** et **variete**. Une plante est dans **une seule** combinaison (ex. Ma–Var1). On compare les **moyennes entre** ces combinaisons.
- **Intra-sujet (within)** : **periode**. **Chaque** plante est mesurée à T1, T2, T3 et T4. On compare l’**évolution dans le temps** (effet période et interactions avec période).

Le **id** permet justement de séparer la variabilité **entre** sujets (id) et **à l’intérieur** de chaque sujet (id:periode).

### 4.3 Type III (sommes de carrés)

- **type = 3** dans **afex** : on utilise les **sommes de carrés de type III**.
- Chaque effet (groupe, variete, periode, interactions) est testé **en tenant compte des autres** effets du modèle. Les résultats ne dépendent pas de l’ordre des termes. C’est adapté quand on s’intéresse à **tous** les effets (principaux et interactions) dans un même modèle.

### 4.4 Correction Greenhouse-Geisser et η² partiel

- **Sphéricité (Mauchly)** : pour le facteur **periode**, l’ANOVA à mesures répétées suppose une condition sur les variances des différences entre périodes (hypothèse de sphéricité). Si elle n’est pas vérifiée, les tests sont trop « optimistes ».
- **Greenhouse-Geisser (GG)** : on corrige les degrés de liberté (et donc la p-value) pour le facteur **periode** et les interactions avec **periode** lorsque la sphéricité n’est pas tenable. **afex** applique cette correction dans le tableau ANOVA.
- **η² partiel (pes)** : pour chaque effet, on affiche la **taille d’effet** (part de variance expliquée par cet effet, en tenant compte des autres). Cela permet de voir quels effets sont non seulement significatifs mais aussi substantiels.

---

## 5. Vérifications des conditions de validité

### 5.1 Normalité des résidus

- On utilise les **résidus** du modèle linéaire sous-jacent (ajusté par **afex**).
- **Graphiques** : Q-Q plot et histogramme des résidus.
- **Objectif** : vérifier visuellement que la distribution des résidus n’est pas trop éloignée d’une loi normale. Si c’est raisonnable, les tests F et les intervalles de confiance restent utilisables.

### 5.2 Homogénéité des variances (Levene)

- **Sur quoi** : sur la **taille moyenne par plante** (une valeur par **id**), on calcule la moyenne des 4 tailles (T1–T4) pour chaque plante.
- **Entre quoi** : entre les **groupes** définis par **groupe × variete** (ex. Ma–Var1, Ma–Var2, Ca–Var1, …).
- **Hypothèse nulle** : les variances de ces moyennes sont **égales** entre les groupes.
- **Interprétation** : un *p* > 0,05 va dans le sens de l’homogénéité ; on considère alors que la condition est acceptable pour l’ANOVA inter-sujets (groupe, variete, groupe×variete).

### 5.3 Sphéricité (Mauchly)

- **Sur quoi** : sur le facteur **periode** (et les contrastes entre T1, T2, T3, T4).
- **Hypothèse** : certaines égalités de variances des différences entre périodes (condition de sphéricité).
- **Si non respectée** : **afex** utilise la correction **Greenhouse-Geisser** (et éventuellement Huynh-Feldt) pour les tests concernant **periode** et les interactions avec **periode**. Le détail (statistique Mauchly, ε GG, etc.) s’obtient avec `summary(modele)`.

---

## 6. Comparaisons post-hoc (emmeans, Tukey)

Une fois le modèle ajusté, on veut savoir **quelles conditions** diffèrent significativement.

### 6.1 Moyennes marginales (groupe × variete)

- Pour chaque combinaison **(groupe, variete)**, on estime la **moyenne de taille** (en moyenne sur toutes les périodes et tous les sujets de cette combinaison).
- On affiche aussi l’**erreur-type (ESM)** et l’**intervalle de confiance 95 %**.
- **Utilité** : voir le « niveau moyen » de chaque condition (terreau × variété), sans détailler par période.

### 6.2 Comparaisons deux à deux : groupe × variete (Tukey)

- On compare **toutes les paires** de combinaisons (ex. Ma–Var1 vs Ca–Var2).
- Pour chaque paire : **différence** des moyennes, **erreur-type**, **statistique t**, **p-value**, et un indicateur de significativité (*, **, ***).
- **Ajustement Tukey** : on contrôle le risque d’erreur globale quand on fait plusieurs comparaisons. On voit ainsi **quelles** combinaisons groupe×variete diffèrent significativement.

### 6.3 Comparaisons deux à deux par période (Tukey)

- Même principe (Tukey sur groupe × variete), mais **séparément pour chaque période** (T1, T2, T3, T4).
- **Utilité** : savoir **à quelles périodes** les différences entre (groupe, variete) sont significatives. Par exemple, « en T1 il n’y a pas de différence, mais en T4 Ma–Var1 est significativement plus élevé que Ca–Var2 ».

---

## 7. Résumé en une phrase

Le **id** permet d’identifier chaque plante pour que le modèle traite correctement les **mesures répétées** (période) ; l’ANOVA mixte teste les effets **groupe**, **variete**, **periode** et leurs interactions, avec correction GG si besoin et vérifications (normalité, Levene, sphéricité) ; les tableaux de moyennes et les Tukey permettent d’interpréter **quelles conditions** (terreau × variété, et à quelles périodes) donnent les tailles les plus élevées ou les plus faibles.

---

## 8. Fichiers et code concernés

- **Chapitre 2** (`02-analyse-descriptive.qmd`) : import, création de **id** si absent, descriptifs et graphiques.
- **Chapitre 3** (`03-facteurs-explicatifs.qmd`) : même import et **id**, puis modèle **aov_ez**, tableau ANOVA, vérifications (normalité, Levene, Mauchly), moyennes marginales et comparaisons Tukey (groupe×variete et par période).

Ce fichier **EXPLICATION_MODELE_ANOVA.md** sert de référence pour comprendre **pourquoi** on crée **id**, **comment** il est construit, et **comment** le modèle et les sorties sont interprétés.
