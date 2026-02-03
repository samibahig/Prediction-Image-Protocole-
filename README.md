# Prediction-Image-Protocole-02_Fevrier_2026
A partir d'images radiologiques, on veut prédire des CSV, 
 Classification Automatique de Protocoles
Architecture du Système
┌─────────────────────────────────────────────────────────────┐
│                    PIPELINE COMPLET                         │
└─────────────────────────────────────────────────────────────┘

    [500 Images]
         ↓
    ┌────────────────┐
    │  1. OCR        │  ← Extraction de texte (Tesseract)
    │  (Tesseract)   │     - Nom d'examen
    └────────────────┘     - Info clinique
         ↓                 - Numéro de requête
    [Texte extrait]
         ↓
    ┌────────────────┐
    │  2. Mapping    │  ← Lier texte OCR ↔ Numéro requête ↔ Protocole
    └────────────────┘
         ↓
    [Données structurées: exam_name, protocol]
         ↓
    ┌────────────────┐
    │  3. ML Model   │  ← Classification (TF-IDF + RandomForest/etc.)
    │  Training      │
    └────────────────┘
         ↓
    [Modèle entraîné]
         ↓
    ┌────────────────┐
    │  4. Prediction │  ← Nouvelle image → Protocole prédit
    └────────────────┘

PROBLÈME PRINCIPAL À RÉSOUDRE
Le Problème de Mapping
Vous avez 3 sources de données séparées:

500 images (formulaires PDF/PNG)

Contiennent: nom d'examen, info clinique, numéro de carte


Fichier Excel (410 lignes)

   Requete  | protocol
   ---------|----------
   1        | C+
   2        | C+
   ...      | ...

Texte OCR (extrait des images)
Contient: nom d'examen, mais numéro de requête pas toujours clair


QUESTION CRITIQUE:
Comment savoir quelle image correspond à quel numéro de requête?
Option A: Numéros de fichiers correspondent aux numéros de requête
Image: 001.png → Requête #1 → Protocole: C+
Image: 002.png → Requête #2 → Protocole: C+












    
