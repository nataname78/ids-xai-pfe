# Synthèse EDA — CIC-IDS2017

*Document de travail, à intégrer au rapport final.*

## 1. Vue d'ensemble du dataset

| Caractéristique | Valeur |
|-----------------|--------|
| Source | Canadian Institute for Cybersecurity (2017) |
| Période de capture | 5 jours ouvrés (lundi → vendredi) |
| Volume total | 2 830 743 flows réseau |
| Nombre de features | 78 features statistiques + 1 label |
| Format | 8 fichiers CSV (un par session de capture) |
| Taille brute | ~ 850 MB décompressés |

## 2. Distribution des classes

| Classe | Nombre | Pourcentage |
|--------|-------:|------------:|
| BENIGN | 2 273 097 | 80.30% |
| DoS Hulk | 231 073 | 8.16% |
| PortScan | 158 930 | 5.61% |
| DDoS | 128 027 | 4.52% |
| DoS GoldenEye | 10 293 | 0.36% |
| FTP-Patator | 7 938 | 0.28% |
| SSH-Patator | 5 897 | 0.21% |
| DoS slowloris | 5 796 | 0.20% |
| DoS Slowhttptest | 5 499 | 0.19% |
| Bot | 1 966 | 0.07% |
| Web Attack — Brute Force | 1 507 | 0.05% |
| Web Attack — XSS | 652 | 0.02% |
| Infiltration | 36 | 0.001% |
| Web Attack — SQL Injection | 21 | 0.0007% |
| Heartbleed | 11 | 0.0004% |

**Ratio de déséquilibre :** ~206 000:1 entre la classe majoritaire (BENIGN) et la classe la plus rare (Heartbleed).

## 3. Constat sur la qualité des données

### 3.1 Doublons

- **256 479 lignes dupliquées (9,06%)**
- Cause probable : artefacts de l'outil de capture de flows, ou recouvrement entre sessions
- Impact si non traités : sur-évaluation des performances du modèle (les mêmes données peuvent se retrouver dans train et test)

### 3.2 Valeurs manquantes

- `Flow Bytes/s` : 1 358 NaN (0,05%)

### 3.3 Valeurs infinies

- `Flow Packets/s` : 2 867 infinis
- `Flow Bytes/s` : 1 509 infinis

Cause : divisions par zéro lorsque la durée du flow est extrêmement courte (< 1 microseconde).

### 3.4 Caractères corrompus

Les classes `Web Attack` apparaissent avec un caractère de remplacement Unicode (`�`) en raison d'un problème d'encodage UTF-8 / Latin-1 du dataset original. À normaliser.

### 3.5 Valeurs négatives aberrantes

Plusieurs features de type "durée" présentent des valeurs négatives qui n'ont pas de sens physique :
- `Flow Duration` : min = -13 microsecondes
- `Flow IAT Min` : min = -14 microsecondes
- `Flow IAT Max` : min = -13 microsecondes

Cause : bugs de l'outil CICFlowMeter utilisé pour générer le dataset, documentés dans la littérature.

### 3.6 Colonne dupliquée

`Fwd Header Length` apparaît deux fois (avec et sans suffixe `.1`) — feature redondante à supprimer.

## 4. Stratégie de traitement adoptée

### 4.1 Regroupement des classes

Pour permettre un apprentissage statistiquement viable, les classes sont regroupées comme suit :

| Classes originales | Classe consolidée | Effectif après fusion |
|--------------------|-------------------|----------------------|
| BENIGN | `BENIGN` | 2 273 097 |
| DoS Hulk, DoS GoldenEye, DoS slowloris, DoS Slowhttptest | `DoS` | 252 661 |
| DDoS | `DDoS` | 128 027 |
| PortScan | `PortScan` | 158 930 |
| FTP-Patator, SSH-Patator | `Brute Force` | 13 835 |
| Web Attack Brute Force, XSS, SQL Injection | `Web Attack` | 2 180 |
| Bot | `Bot` | 1 966 |
| **Heartbleed, Infiltration** | **EXCLUS** | 47 (effectif insuffisant) |

**Justification scientifique :** les classes Heartbleed (n=11) et Infiltration (n=36) ne permettent pas un apprentissage statistique fiable et un test set représentatif. Leur inclusion biaiserait l'évaluation.

### 4.2 Nettoyage des données

- Suppression des 256 479 doublons
- Suppression des lignes contenant des NaN sur les colonnes critiques
- Suppression des lignes contenant des valeurs infinies
- Suppression des lignes avec valeurs négatives aberrantes sur les durées
- Suppression de la colonne dupliquée `Fwd Header Length.1`
- Normalisation des noms de colonnes (suppression des espaces parasites)
- Normalisation des labels (correction des caractères corrompus)

### 4.3 Volume final estimé

Après nettoyage : **environ 2,56 millions de flows**, répartis sur **7 classes équilibrées en termes de minimum d'effectifs** (toutes ≥ 1 900 échantillons).

## 5. Insights métier

### 5.1 Hiérarchie réelle des menaces

Les volumes relatifs reflètent assez fidèlement la réalité des SOC en production :
- 80% du trafic est légitime → problème de **fatigue d'alerte** chez les analystes
- Les attaques par volume (DoS, PortScan, DDoS) dominent les données → faciles à détecter par anomalie statistique
- Les attaques sophistiquées (Heartbleed, Infiltration) sont rares → besoin de méthodes capables de **généraliser sur peu d'exemples**

### 5.2 Implications pour la modélisation

1. Toute approche naïve obtiendra ~80% d'accuracy en prédisant toujours "BENIGN" — d'où la nécessité de métriques **par classe** (F1-score, precision, recall) et non globales.
2. Le taux de faux positifs (FPR) est critique : avec 2,3M de flows bénins, un FPR de 1% génère 23 000 fausses alertes — ingérable pour un SOC.
3. L'explicabilité (SHAP, LIME) prend tout son sens : permettre à l'analyste de comprendre rapidement *pourquoi* une alerte est levée pour limiter le temps de triage.

## 6. Suite des travaux

- **Notebook 02 — Preprocessing :** nettoyage + regroupement + sauvegarde en Parquet
- **Notebook 03 — Baseline ML :** Random Forest + XGBoost
- **Notebook 04 — Deep Learning :** DNN + FT-Transformer
- **Notebooks 05-07 — XAI :** SHAP, LIME, évaluation des explications
