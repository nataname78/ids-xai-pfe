# IDS-XAI — Détection d'intrusion réseau explicable

> Projet de fin d'études (PGE5, M2 IA & Data Science) — aivancity School for Technology, Business & Society.
> Auteur : Nathan Makila

**Question de recherche :** un modèle qui détecte très bien les attaques réseau est-il aussi un modèle auquel un analyste SOC peut faire confiance ? Et peut-on mesurer la fiabilité des explications, pas seulement celle des prédictions ?

## Résultats en un coup d'œil

| Modèle (CIC-IDS2017, 7 classes, 377 682 flux de test) | Accuracy | F1 macro |
|---|---|---|
| Random Forest | 0,996 | 0,900 |
| **XGBoost + Optuna** | **0,999** | **0,969** |
| DNN (PyTorch) | 0,962 | 0,725 |
| FT-Transformer | 0,995 | 0,793 |

Le modèle tabulaire classique bat les deux réseaux de neurones : sur ces données, le deep learning n'apporte rien de plus.

## Ce qui a été vérifié

**1. Fuite de données (test de `Destination Port`).** Cette variable était la plus importante selon SHAP, et chaque attaque de CIC-IDS2017 cible des ports précis. Je l'ai retirée et réentraîné à l'identique : le F1 macro passe de 0,972 à 0,961 (−1,1 point). La fuite via ce port existe mais ne suffit pas à expliquer la performance. Reste à tester : un découpage temporel plutôt qu'aléatoire (voir `notebooks/12_fuite_sans_dport.ipynb`).

**2. Généralisation (UNSW-NB15).** Le même pipeline tombe à 0,767 d'accuracy et 0,565 de F1 macro sur un autre jeu de données. Trois configurations testées (par défaut, hyperparamètres transférés, Optuna réoptimisé) donnent toutes un F1 macro compris entre 0,558 et 0,565 : le réglage ne résout pas l'écart, il vient des données.

**3. Qualité des explications (SHAP vs LIME).**

| Critère | SHAP | LIME |
|---|---|---|
| Stabilité (Jaccard) | 1,00 | 0,56 |
| Fidélité (AOPC) | 0,119 | 0,115 |
| Plausibilité (% de features liées à la cybersécurité) | 93,2 % | 88,1 % |

Sur les cas locaux étudiés, les deux méthodes ne retiennent que ~27 % des mêmes variables, mais s'accordent sur le sens de l'effet dans ~92 % des cas. Conclusion pratique : on ne peut pas substituer l'une à l'autre sans le dire.

## Limites connues

- Découpage aléatoire (stratifié) et non temporel : les scores de CIC-IDS2017 sont probablement optimistes.
- La classe `Bot` reste la plus difficile (F1 ≈ 0,81 pour XGBoost, 292 exemples de test).
- Les explications ne sont pas évaluées avec de vrais analystes SOC.

## Structure

```
notebooks/   exploration, modèles, XAI, tests de fuite et de validation croisée
src/         données, modèles, entraînement, évaluation, XAI
results/     métriques, figures, valeurs SHAP
dashboard/   démo Streamlit
```

## Reproduire

```bash
git clone https://github.com/nataname78/ids-xai-pfe.git
cd ids-xai-pfe
python -m venv .venv && source .venv/bin/activate   # Windows : .venv\Scripts\activate
pip install -r requirements.txt
```

Les jeux de données ne sont pas inclus : [CIC-IDS2017](https://www.unb.ca/cic/datasets/ids-2017.html) et [UNSW-NB15](https://research.unsw.edu.au/projects/unsw-nb15-dataset).

## Stack

Python · scikit-learn · XGBoost · Optuna · PyTorch · SHAP · LIME · MLflow · Streamlit

## Licence

Projet académique — usage pédagogique.
