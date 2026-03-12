# 🏥 Classification Automatique des Protocoles TDM

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10-blue?logo=python" alt="Python"/>
  <img src="https://img.shields.io/badge/CamemBERT-Fine--tuned-orange" alt="CamemBERT"/>
  <img src="https://img.shields.io/badge/F1--Score-0.996-brightgreen" alt="F1-Score"/>
  <img src="https://img.shields.io/badge/OCR-Tesseract-lightblue" alt="OCR"/>
  <img src="https://img.shields.io/badge/CRCHUM-Recherche-red" alt="CRCHUM"/>
  <a href="https://colab.research.google.com/drive/1Pb_PW6nKgQy3ZK4oVnPAleWflkYCZIxX">
    <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
  </a>
</p>

> **Projet de recherche — CRCHUM / Université de Montréal**  
> Classification automatique des protocoles TDM (C+ / C- / C- C+) à partir de formulaires de demande d'examen radiologique manuscrits — pipeline OCR + NLP + CamemBERT fine-tuné.

---

## 🎯 Contexte clinique

Les formulaires de **Demande d'Examen Radiologique** du CRCHUM contiennent deux champs clés extraits par OCR :

| Champ | Exemple |
|-------|---------|
| **EXAMEN DEMANDÉ** | `TDM thorax-abdomen`, `scan abdo-pelv` |
| **RENSEIGNEMENTS CLINIQUES** | `douleur, dyspnée`, `volvulus de l'estomac` |

À partir de ces deux champs, le modèle prédit le **protocole d'injection** :

| Protocole | Signification |
|-----------|---------------|
| **C+** | Avec injection de produit de contraste |
| **C-** | Sans injection |
| **C- C+** | Double phase (sans puis avec contraste) |

---

## 🗂️ Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                    PIPELINE COMPLET                         │
└─────────────────────────────────────────────────────────────┘

    [~500 Images PNG]          [Excel Protocoles]
    (formulaires CRCHUM)       (étiquettes C+ / C- / C- C+)
           │                            │
           ▼                            ▼
    ┌──────────────┐          ┌──────────────────────┐
    │  1. OCR      │          │  2. Nettoyage &      │
    │  Tesseract   │          │     Consolidation    │
    │  → exam_name │          │  C_ → C-             │
    │  → clinique  │          └──────────────────────┘
    └──────────────┘                    │
           └──────────┬─────────────────┘
                      ▼
           [Texte extrait + Labels — 410 requêtes]
                      │
                      ▼
           ┌──────────────────────┐
           │  3. Données          │
           │  Synthétiques        │
           │  ~15 000 exemples    │
           └──────────────────────┘
                      │
           ┌──────────┴──────────────┐
           ▼                         ▼
    ┌─────────────┐         ┌──────────────────┐
    │  Baseline   │         │  Fine-tuning     │
    │  Sentence-  │         │  CamemBERT       │
    │  BERT+LogReg│         │  10 epochs       │
    └─────────────┘         └──────────────────┘
                      ▼
           ┌──────────────────────┐
           │  Prédicteur Robuste  │
           │  v2.0                │
           └──────────────────────┘
```

---

## 📊 Résultats

| Modèle | F1-Score | Accuracy |
|--------|----------|----------|
| Baseline — Sentence-BERT + LogReg | 0.952 | 0.951 |
| **CamemBERT Fine-tuné** | **0.996** | **0.995** |
| **Gain** | **+4.4%** | **+4.4%** |

---

## 📁 Données

### Structure du fichier OCR extrait (410 images)

| Colonne | Description | Exemple |
|---------|-------------|---------|
| `image_name` | Fichier PNG source | `Diapositive21.PNG` |
| `exam_name` | Examen demandé (OCR) | `scan abdo-pelv` |
| `card_number` | Numéro de carte patient | `220124` |
| `full_text` | Texte brut OCR complet | *(texte brut)* |

La colonne `protocol` (C+ / C- / C- C+) provient d'un Excel séparé, lié via `image_name`. Les données ne sont pas distribuées (confidentialité patients).

**Qualité OCR** : 320 / 410 images avec exam_name extrait (78%). Le bruit typique (`sc4nn3r`, `CTAbdoPehien`) est normalisé par le Prédicteur Robuste v2.0.

---

## 🚀 Démarrage rapide

### Colab (recommandé)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1Pb_PW6nKgQy3ZK4oVnPAleWflkYCZIxX)

### Local

```bash
git clone https://github.com/samibahig/Prediction-Image-Protocole-.git
cd Prediction-Image-Protocole-
pip install -r requirements.txt
# Tesseract requis : sudo apt-get install tesseract-ocr tesseract-ocr-fra
jupyter notebook notebooks/Classification_Protocoles_TDM_CamemBERT.ipynb
```

---

## ⚙️ Stack technique

| Composant | Technologie |
|-----------|------------|
| OCR | Tesseract + pytesseract |
| Baseline NLP | `all-MiniLM-L6-v2` + LogisticRegression |
| Fine-tuning | `camembert-base` via HuggingFace Transformers |
| Framework | PyTorch + scikit-learn |
| Données | 410 images réelles + ~15 000 synthétiques |

### Prédicteur Robuste v2.0
- ✅ Détection hors-domaine (IRM, Radiographie, Écho, PET → rejetés)
- ✅ Normalisation bruit OCR (`sc4nn3r → scanner`)
- ✅ Seuil de confiance configurable (défaut : 0.85)

---

## 📁 Structure

```
Prediction-Image-Protocole-/
├── notebooks/
│   └── Classification_Protocoles_TDM_CamemBERT.ipynb
├── src/
│   └── nouveau_modele_imagerie_bert_2_classes.py
├── results/          ← généré à l'exécution
├── requirements.txt
└── README.md
```

---

## 🔮 Prochaines étapes

- [ ] Validation clinique avec les radiologues (CRCHUM)
- [ ] Amélioration OCR (90 images avec exam_name manquant)
- [ ] Déploiement API REST (FastAPI + Docker)
- [ ] Intégration système RIS hospitalier

---

## 👥 Auteurs

**Sami Bahig** — Data Scientist  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Sami_Bahig-blue?logo=linkedin)](https://www.linkedin.com/in/sami-bahig/)

**Oumnia** — Data Scientist, CRCHUM / Université de Montréal

---

*MIT License — Données cliniques non distribuées.*
