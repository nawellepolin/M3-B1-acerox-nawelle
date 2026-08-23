# M3-B1 — Acerox Métallurgie — Identification des sources

> Brief M3-B1 (formation FastIA/ATOS). Client fictif : **Acerox Métallurgie**
> (~800 salariés, 3 sites). Contact : Sébastien Marchand, chef de projet
> industrialisation. Objectif du brief : identifier quelles sources de
> données enrichir en priorité pour faire évoluer leur modèle de prédiction
> de défauts d'un mode réactif à un mode préventif — **sans coder
> l'ingestion ni le modèle** (ça, c'est M3-B2).
>
> Auteur : Nawelle Polin — Date : 21/07/2026

---

## 🔎 Lire cette livraison en 3 minutes

1. **`notes_entretien.md`** — ce que Sébastien a dit pendant l'entretien
   (9h30-10h00), avec la distinction *dit* / *interprété*, et les questions
   restées sans réponse.
2. **`identification_sources.md`** — le livrable principal : demande de
   Sébastien reformulée en besoin métier, inventaire des 5 sources
   (format, volume, qualité, risques RGPD, pertinence), sources
   retenues/écartées et pourquoi, 2 risques RGPD formulés en termes
   opérationnels, 5 questions encore ouvertes.
3. **`flux_donnees.md`** — le schéma Mermaid des flux (sources → ingestion
   → BDD pivot → modèle existant Acerox) + légende + décisions associées.
4. **`notebooks/M3-B1_nawelle.ipynb`** — exploration des 3 sources fournies
   (`capteurs_iot.csv`, `erp_export.json`, `logs_machines.log`) : lecture,
   `info`/`describe`/`head`, pas de transformation. Déjà exécuté de bout en
   bout, les sorties sont visibles sans avoir besoin de relancer le kernel.
5. **`notebooks/M3-B1_fusion_provenance.ipynb`** — l'exercice de fusion :
   réunion de `capteurs_site_A.csv` et `capteurs_site_B.csv` avec une
   colonne `source`, et traitement des 3 pièges volontaires (identifiants
   de machine qui se chevauchent entre sites, unité de température
   oubliée sur un site, colonne absente d'un des deux exports). Déjà
   exécuté, sans erreur.

**Résultat en une phrase** : les sources les plus exploitables sont
`capteurs_iot.csv` et `logs_machines.log`, qui se recoupent sur une même
ligne à Roubaix (candidate au "site 3x plus problématique" cité en
entretien) ; `erp_export.json` vient en complément après pseudonymisation
de `ouvrier_id` ; les deux échantillons `capteurs_site_A/B.csv` sont
écartés comme données d'entraînement (période et formats incompatibles).
Le point encore bloquant : aucune source ne porte le label
"non-conformité", pourtant la cible chiffrée du projet.

---

## 📁 Structure du repo

```
M3-B1-acerox-nawelle/
├── data/
│   ├── capteurs_iot.csv, erp_export.json, logs_machines.log   # fournis mardi 9h, gitignorés
│   └── capteurs_site_A.csv, capteurs_site_B.csv                # versionnés — jeu jouet fusion/provenance
├── notebooks/
│   ├── M3-B1_nawelle.ipynb              # exploration des 3 sources (sans transformation)
│   └── M3-B1_fusion_provenance.ipynb    # fusion 2 exports + colonne source, 3 pièges traités
├── ressources/                          # mini-cours d'appui (fournis avec le template)
├── identification_sources.md            # livrable principal
├── flux_donnees.md                      # schéma Mermaid + légende
├── notes_entretien.md                   # notes prises pendant l'entretien
├── requirements.txt
└── README.md                            # ce fichier
```

> `data/capteurs_iot.csv`, `erp_export.json` et `logs_machines.log` sont
> gitignorés (fournis en direct par la formatrice, pas destinés à être
> publiés). Les notebooks sont livrés **déjà exécutés** : leurs sorties
> sont donc lisibles directement sur GitHub même sans ces fichiers.

---

## 🚀 Reproduire l'exploration (optionnel)

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebooks/M3-B1_nawelle.ipynb
```

Nécessite d'avoir replacé les 3 fichiers sources dans `data/` (non fournis
dans ce repo, cf. ci-dessus).

---

## ✅ Ce qui a été fait / pas fait

Fait : entretien mené et noté, 5 sources explorées sans transformation,
exercice de fusion + provenance avec les 3 pièges traités, schéma de
flux Mermaid rendu, note d'identification structurée avec risques RGPD
et questions ouvertes.

Volontairement pas fait (hors périmètre M3-B1, cf. brief) : code
d'ingestion, conception du modèle ML, AIPD juridique formelle, choix
d'un orchestrateur ETL.

---

*Livraison produite par Nawelle, 21/07/2026, dans le cadre du brief M3-B1.*
