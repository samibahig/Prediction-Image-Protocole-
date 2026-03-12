# 🏥 Classification Automatique des Protocoles TDM

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10-blue?logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/🤗 CamemBERT-Fine--tuned-orange"/>
  <img src="https://img.shields.io/badge/F1--Score-99.6%25-brightgreen"/>
  <img src="https://img.shields.io/badge/OCR-Tesseract-lightblue"/>
  <img src="https://img.shields.io/badge/CRCHUM-Recherche-darkred"/>
  <a href="https://colab.research.google.com/drive/1Pb_PW6nKgQy3ZK4oVnPAleWflkYCZIxX">
    <img src="https://colab.research.google.com/assets/colab-badge.svg"/>
  </a>
</p>

<p align="center">
  <b>Projet de recherche — CRCHUM / Université de Montréal</b><br/>
  Pipeline complet OCR → NLP → CamemBERT pour la classification automatique<br/>
  des protocoles d'injection de scanner TDM à partir de formulaires radiologiques manuscrits.
</p>

---

## 📋 Table des matières

- [🎯 Contexte clinique](#-contexte-clinique)
- [🗂️ Pipeline](#-pipeline)
- [📊 Résultats](#-résultats)
- [📁 Données](#-données)
- [🚀 Démarrage rapide](#-démarrage-rapide)
- [⚙️ Stack technique](#-stack-technique)
- [📂 Structure du repo](#-structure-du-repo)
- [🔮 Prochaines étapes](#-prochaines-étapes)
- [👥 Auteurs](#-auteurs)

---

## 🎯 Contexte clinique

En radiologie hospitalière, chaque examen TDM (tomodensitométrie / scanner) nécessite de déterminer le **protocole d'injection de produit de contraste**. Ce choix est fait par les technologues en lisant la requête manuscrite du médecin.

Ce projet **automatise cette décision** à partir des formulaires de Demande d'Examen Radiologique du CRCHUM :

```
┌─────────────────────────────────────────────────────┐
│  DEMANDE D'EXAMEN RADIOLOGIQUE (CRCHUM)             │
│                                                     │
│  EXAMEN DEMANDÉ :  TDM thorax-abdomen               │ ← extrait par OCR
│                                                     │
│  RENSEIGNEMENTS CLINIQUES :  douleur, dyspnée       │ ← extrait par OCR
│                                                     │
│  → Protocole prédit automatiquement :  C+  ✅        │
└─────────────────────────────────────────────────────┘
```

### Les 3 protocoles à prédire

| Protocole | Signification | Fréquence |
|-----------|---------------|-----------|
| **C+** | Avec injection de produit de contraste | ~60% |
| **C-** | Sans injection | ~25% |
| **C- C+** | Double phase (sans puis avec) | ~15% |

> 📂 Voir [`sample_data/example_ocr_output.csv`](sample_data/example_ocr_output.csv) pour un exemple anonymisé de la structure des données.

---

## 🗂️ Pipeline

```
  410 Images PNG                   Excel Protocoles
  (formulaires CRCHUM)          (étiquettes C+ / C- / C- C+)
         │                                │
         ▼                                ▼
  ┌─────────────────┐        ┌────────────────────────┐
  │  ÉTAPE 1 — OCR  │        │  ÉTAPE 2 — Nettoyage   │
  │  Tesseract      │        │  • C_ → C-             │
  │  ─────────────  │        │  • 3 classes finales   │
  │  exam_name      │        └────────────────────────┘
  │  rens. clinique │
  └─────────────────┘
         │                                │
         └──────────────┬─────────────────┘
                        ▼
           [Dataset structuré — 410 requêtes réelles]
                        │
                        ▼
           ┌────────────────────────┐
           │  ÉTAPE 3 — Augmentation│
           │  ~15 000 exemples      │
           │  synthétiques générés  │
           │  (5 000 / protocole)   │
           └────────────────────────┘
                        │
           ┌────────────┴────────────────┐
           ▼                             ▼
  ┌──────────────────┐       ┌───────────────────────┐
  │  ÉTAPE 4         │       │  ÉTAPE 5              │
  │  Baseline        │       │  Fine-tuning          │
  │  Sentence-BERT   │       │  CamemBERT            │
  │  + LogReg        │       │  camembert-base       │
  │  F1 = 0.952      │       │  10 epochs, lr=2e-5   │
  └──────────────────┘       │  F1 = 0.996 🏆        │
                             └───────────────────────┘
                        │
                        ▼
           ┌────────────────────────┐
           │  ÉTAPE 6              │
           │  Prédicteur Robuste   │
           │  v2.0                 │
           │  • Hors-domaine       │
           │  • Bruit OCR corrigé  │
           │  • Seuil confiance    │
           └────────────────────────┘
```

> 📓 Notebook complet : [`notebooks/Classification_Protocoles_TDM_CamemBERT.ipynb`](notebooks/Classification_Protocoles_TDM_CamemBERT.ipynb)  
> 🐍 Script source : [`src/nouveau_modele_imagerie_bert_2_classes.py`](src/nouveau_modele_imagerie_bert_2_classes.py)

---

## 📊 Résultats

| Modèle | F1-Score | Accuracy | Delta |
|--------|:--------:|:--------:|:-----:|
| Baseline — Sentence-BERT + LogReg | 0.952 | 0.951 | — |
| **CamemBERT Fine-tuné (3 classes)** | **0.996** | **0.995** | **+4.4%** |

### Détail par classe (CamemBERT)

| Classe | Précision | Rappel | F1 |
|--------|:---------:|:------:|:--:|
| C+ | 0.998 | 0.997 | 0.997 |
| C- | 0.995 | 0.996 | 0.995 |
| C- C+ | 0.996 | 0.994 | 0.995 |

> 📈 Graphiques de comparaison générés dans [`results/`](results/) après exécution du notebook.

---

## 📁 Données

> ⚠️ **Confidentialité** : Les images et données patients du CRCHUM ne sont **pas distribuées** dans ce repo (Loi 25, QC). Seul un exemple anonymisé est fourni.

### Format du fichier OCR extrait

Le pipeline génère un CSV structuré à partir des images (voir [`sample_data/example_ocr_output.csv`](sample_data/example_ocr_output.csv)) :

| Colonne | Description | Exemple |
|---------|-------------|---------|
| `image_name` | Fichier PNG source | `Diapositive21.PNG` |
| `exam_name` | Examen demandé (extrait OCR) | `TDM thorax-abdomen` |
| `card_number` | Numéro de carte patient | `220124` |
| `full_text` | Texte brut OCR complet | *(texte brut)* |
| `protocol` | **Label cible** (depuis Excel) | `C+` |

### Statistiques du dataset

| Métrique | Valeur |
|----------|--------|
| Images réelles | 410 |
| exam_name extrait (OCR) | 320 / 410 (78%) |
| Exemples synthétiques | ~15 000 |
| Dataset final (mixte) | ~11 000 uniques |
| Classes | 3 (C+, C-, C- C+) |

### Bruit OCR typique

Le Prédicteur Robuste v2.0 normalise automatiquement les erreurs OCR fréquentes :

| OCR brut | Texte corrigé |
|----------|--------------|
| `seanabdopely` | `scan abdo-pelv` |
| `CTAbdoPehien` | `CT abdo-pelvien` |
| `sc4nn3r thor4x` | `scanner thorax` |
| `TDMthors` | `TDM thorax` |

---

## 🚀 Démarrage rapide

### Option 1 — Google Colab (recommandé, GPU gratuit)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1Pb_PW6nKgQy3ZK4oVnPAleWflkYCZIxX)

1. Ouvrir le notebook Colab
2. Connecter Google Drive (`drive.mount('/content/drive')`)
3. Adapter les chemins dans la **Cellule 3 — Configuration**
4. Exécuter toutes les cellules

### Option 2 — Local

```bash
# 1. Cloner le repo
git clone https://github.com/samibahig/Prediction-Image-Protocole-.git
cd Prediction-Image-Protocole-

# 2. Installer les dépendances Python
pip install -r requirements.txt

# 3. Installer Tesseract (OCR)
# Ubuntu/Debian:
sudo apt-get install tesseract-ocr tesseract-ocr-fra
# macOS:
brew install tesseract tesseract-lang

# 4. Configurer les chemins (Cellule 3 du notebook)
# PROTOCOLS_FILE = 'chemin/vers/Protocoles_requetes.xlsx'
# IMAGE_DIR = 'chemin/vers/images_png/'
# OUTPUT_DIR = 'chemin/vers/resultats/'

# 5. Lancer le notebook
jupyter notebook notebooks/Classification_Protocoles_TDM_CamemBERT.ipynb
```

> 📦 Voir [`requirements.txt`](requirements.txt) pour la liste complète des dépendances.

---

## ⚙️ Stack technique

| Composant | Technologie | Version |
|-----------|-------------|---------|
| OCR | Tesseract + pytesseract | ≥ 4.1 |
| Embeddings baseline | `all-MiniLM-L6-v2` (Sentence-BERT) | — |
| Classifier baseline | LogisticRegression (scikit-learn) | ≥ 1.3 |
| Fine-tuning | `camembert-base` (HuggingFace) | — |
| Deep Learning | PyTorch | ≥ 2.0 |
| NLP | 🤗 Transformers | ≥ 4.30 |
| Data | pandas, numpy | — |
| Viz | matplotlib, seaborn | — |

### Prédicteur Robuste v2.0 — Fonctionnalités

```python
predictor = RobustProtocolPredictor(model, tokenizer, label_encoder)

# Exemple d'utilisation
result = predictor.predict("TDM thorax avec injection")
# → {'protocol': 'C+', 'confidence': 0.98, 'normalized_text': 'tdm thorax avec injection'}

result = predictor.predict("IRM genou gauche")
# → {'protocol': None, 'reason': 'HORS_DOMAINE: IRM détecté', 'confidence': 0.0}
```

- ✅ Rejet des examens hors-domaine (IRM, Radiographie, Échographie, PET)
- ✅ Normalisation du bruit OCR avant classification
- ✅ Seuil de confiance configurable (défaut : 0.85)
- ✅ Explication des décisions de rejet

---

## 📂 Structure du repo

```
Prediction-Image-Protocole-/
│
├── 📓 notebooks/
│   └── Classification_Protocoles_TDM_CamemBERT.ipynb   ← Pipeline complet
│
├── 🐍 src/
│   └── nouveau_modele_imagerie_bert_2_classes.py        ← Script Python source
│
├── 📊 results/          ← Généré à l'exécution (non versionné)
│   ├── confusion_matrix_bert_ocr_3class.png
│   ├── comparaison_models_ocr_3class_final.png
│   ├── metrics_baseline.csv
│   ├── metrics_bert_ocr_3class.csv
│   └── bert_ocr_3class_final/                           ← Modèle sauvegardé
│       ├── config.json
│       ├── tokenizer.json
│       └── pytorch_model.bin
│
├── 📄 sample_data/
│   └── example_ocr_output.csv                          ← Exemple anonymisé
│
├── 📋 requirements.txt
├── 🚫 .gitignore
└── 📖 README.md
```

---

## 🔮 Prochaines étapes

- [ ] **Validation clinique** avec les radiologues du CRCHUM
- [ ] **Améliorer OCR** : 90 images avec exam_name manquant (22%)
- [ ] **Collecte données** : augmenter le dataset réel
- [ ] **API REST** : déploiement FastAPI + Docker
- [ ] **Intégration RIS** : connecter au système d'information radiologique hospitalier
- [ ] **Monitoring** : drift detection en production

---

## 👥 Auteurs

**Sami Bahig** — Data Scientist  
Formation MILA · Expérience CRCHUM  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Sami_Bahig-0077B5?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/sami-bahig/)
[![GitHub](https://img.shields.io/badge/GitHub-samibahig-181717?logo=github&logoColor=white)](https://github.com/samibahig)

**Oumnia** — Data Scientist  
CRCHUM / Université de Montréal

---

## 🔗 Projets connexes

| Projet | Description | Lien |
|--------|-------------|------|
| **FAERS 2025** | Pharmacovigilance FDA — Signaux de sécurité médicamenteux | [![GitHub](https://img.shields.io/badge/GitHub-faers--2025-181717?logo=github)](https://github.com/samibahig/faers-2025-pharmacovigilance) |

---

<p align="center">
  <sub>MIT License · Données cliniques non distribuées (Loi 25, Québec) · CRCHUM / Université de Montréal</sub>
</p>
