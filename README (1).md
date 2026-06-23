# IDS-XAI : Système de Détection d'Intrusion Explicable

> Projet de Fin d'Études — PGE5 — aivancity School for Technology, Business & Society
> Auteur : Nathan Makila

## Contexte

Ce projet vise à développer un système de détection d'intrusion réseau (IDS) basé sur l'intelligence artificielle, avec un accent particulier sur **l'explicabilité (XAI)** des décisions de détection. L'objectif est de produire un système non seulement performant, mais aussi compréhensible par les analystes SOC (Security Operations Center).

## Problématique

Les systèmes IDS modernes basés sur le machine learning souffrent de deux limites majeures en production :

1. **Boîte noire** : un analyste cyber ne fait pas confiance à une alerte sans justification claire.
2. **Faux positifs** : des taux trop élevés génèrent une fatigue d'alerte et noient les vraies menaces.

Ce projet adresse ces deux limites en combinant des modèles performants (XGBoost, FT-Transformer) avec des méthodes XAI (SHAP, LIME) et une évaluation rigoureuse de la qualité des explications.

## Contributions visées

- Comparaison rigoureuse d'un modèle ML (XGBoost) et d'un modèle DL moderne (FT-Transformer/TabNet) sur CIC-IDS2017
- Application et comparaison de SHAP et LIME pour expliquer les détections
- Évaluation de la **fiabilité** des explications (stabilité, fidélité, plausibilité)
- Validation croisée sur UNSW-NB15 (généralisation)
- Dashboard démo pour visualisation des alertes expliquées

## Stack technique

| Catégorie | Outils |
|-----------|--------|
| Langage | Python 3.11+ |
| Manipulation données | NumPy, Pandas, scikit-learn |
| Modèle ML | XGBoost + Optuna (HPO) |
| Modèle DL | PyTorch + pytorch-tabular |
| XAI | SHAP, LIME, Captum |
| Visualisation | Matplotlib, Seaborn, Plotly |
| Tracking | MLflow |
| Démo | Streamlit |
| Versioning | Git + GitHub |

## Structure du projet

```
ids-xai-pfe/
├── data/
│   ├── raw/              # Datasets bruts (gitignored)
│   ├── processed/        # Datasets prétraités
│   └── external/         # Données externes éventuelles
├── notebooks/            # Notebooks d'exploration et d'expériences
├── src/
│   ├── data/             # Chargement, preprocessing
│   ├── models/           # Définitions des modèles
│   ├── training/         # Boucles d'entraînement
│   ├── evaluation/       # Métriques et comparaisons
│   ├── xai/              # Méthodes XAI et évaluation
│   └── utils/            # Helpers
├── experiments/configs/  # Configs YAML reproductibles
├── results/              # Figures, métriques, modèles sauvegardés
├── dashboard/            # Application Streamlit
├── report/               # Rapport final
├── tests/                # Tests unitaires
└── scripts/              # Scripts shell utilitaires
```

## Datasets utilisés

- **CIC-IDS2017** (principal) — Canadian Institute for Cybersecurity
  https://www.unb.ca/cic/datasets/ids-2017.html
- **UNSW-NB15** (validation croisée)
  https://research.unsw.edu.au/projects/unsw-nb15-dataset

## Setup

```bash
# Cloner le repo
git clone https://github.com/nataname78/ids-xai-pfe.git
cd ids-xai-pfe

# Créer un environnement virtuel
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # Linux/Mac

# Installer les dépendances
pip install -r requirements.txt

# Vérifier l'installation
python -c "import torch; print('CUDA disponible:', torch.cuda.is_available())"
```

## Roadmap

- [ ] **Phase A — Fondations** (S1–S3) : setup, EDA, preprocessing
- [ ] **Phase B — Modèles ML** (S4–S6) : Random Forest, XGBoost, tuning
- [ ] **Phase C — Modèles DL** (S7–S9) : DNN, FT-Transformer
- [ ] **Phase D — XAI** (S10–S13) : SHAP, LIME, évaluation explications
- [ ] **Phase E — Livraison** (S14) : rapport final, soutenance

## Statut actuel

🟢 Projet en phase de démarrage — semaine 1

## Licence

Projet académique — usage pédagogique uniquement.
