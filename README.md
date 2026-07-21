# M3-B1 — Squelette repo (entretien client + identification sources)

> **Repo template GitHub.** Clique sur **« Use this template »** en haut à
> droite de cette page → **Create a new repository** → nomme-le
> `M3-B1-<client>-<prénom>` sur **ton** compte GitHub personnel (le nom
> du client te sera révélé mardi 9h).

---

## 🚀 Démarrage (4 commandes)

```bash
# 0. Clone ton repo perso fraîchement créé
git clone git@github.com:<ton-user>/M3-B1-<client>-<prenom>.git
cd M3-B1-<client>-<prenom>

# 1. Environnement virtuel
python -m venv .venv && source .venv/bin/activate     # Linux/macOS
# .venv\Scripts\activate                              # Windows

# 2. Dépendances
pip install -r requirements.txt

# 3. Vérification
jupyter notebook notebooks/M3-B1_template.ipynb       # → s'ouvre dans le navigateur
```

Si ces 4 commandes marchent, ton poste est prêt.

> ⚠️ Les 3 sources te seront fournies **mardi 9h après l'entretien
> fictif** par la formatrice (Discord). Place-les dans `data/`. Le
> `.gitignore` exclut `data/*.csv`, `data/*.json`, `data/*.log` du commit.

---

## 📁 Structure du repo

```
M3-B1-<client>-<prenom>/
├── data/                                # gitignored (sauf jeu jouet provenance)
│   ├── capteurs_iot.csv                 # fourni mardi 9h
│   ├── erp_export.json                  # fourni mardi 9h
│   ├── logs_machines.log                # fourni mardi 9h
│   ├── capteurs_site_A.csv              # versionné — jeu jouet fusion/provenance
│   └── capteurs_site_B.csv              # versionné — 2e export comparable
├── notebooks/
│   ├── M3-B1_template.ipynb             # exploration rapide 3 sources
│   └── M3-B1_fusion_provenance.ipynb    # fusion 2 exports + colonne source
├── ressources/                          # 📚 mini-cours d'appui
│   ├── README.md
│   ├── 01_Entretien_client_essentiel.md
│   ├── 02_Cartographie_sources_essentiel.md
│   ├── 03_Risques_RGPD_multisources_essentiel.md
│   ├── 04_Schema_Mermaid_flux_essentiel.md
│   ├── 05_Note_identification_essentiel.md
│   └── liens_officiels.md
├── identification_sources.md            # livrable principal (2-3 pages)
├── flux_donnees.md                      # schéma Mermaid + légende
├── notes_entretien.md                   # tes notes prises à 9h30
├── requirements.txt
├── .gitignore
└── README.md (ce fichier — à compléter)
```

---

## 📚 Mini-cours d'appui

Les **5 mini-cours pédagogiques** sont fournis dans
[`./ressources/`](./ressources/). Lecture juste-à-temps, ~15-20 min chacun :

| Tâche | Mini-cours |
|---|---|
| Préparer l'entretien client | [`01_Entretien_client_essentiel.md`](./ressources/01_Entretien_client_essentiel.md) |
| Cartographier les sources | [`02_Cartographie_sources_essentiel.md`](./ressources/02_Cartographie_sources_essentiel.md) |
| Risques RGPD multi-sources | [`03_Risques_RGPD_multisources_essentiel.md`](./ressources/03_Risques_RGPD_multisources_essentiel.md) |
| Schématiser les flux (Mermaid) | [`04_Schema_Mermaid_flux_essentiel.md`](./ressources/04_Schema_Mermaid_flux_essentiel.md) |
| Rédiger la note d'identification | [`05_Note_identification_essentiel.md`](./ressources/05_Note_identification_essentiel.md) |

Cf. [`./ressources/README.md`](./ressources/README.md) pour l'ordre détaillé.

---

## 🧭 Démarche attendue

1. **Préparation entretien** (30 min, 9h00-9h30) — 8-12 questions catégorisées
2. **Entretien fictif** (30 min, 9h30-10h00) — visio Discord, notes au fil de l'eau
3. **Analyse des 3 sources** (2 h, 10h00-12h30) — `pd.read_*` + `info` + `describe`
3 bis. **Fusion + provenance** (~30 min) — ouvre
   [`notebooks/M3-B1_fusion_provenance.ipynb`](./notebooks/M3-B1_fusion_provenance.ipynb) :
   réunis **deux exports comparables** (`capteurs_site_A/B.csv`, déjà dans
   `data/`) avec une colonne `source`, et débusque les 3 pièges (ids qui se
   chevauchent, unité de température oubliée, colonne absente). C'est le geste
   **exact** que le cas d'usage certif attend côté données — reporte une phrase
   dans `identification_sources.md`.
4. **Schéma Mermaid** (1 h, 13h30-14h30) — `flux_donnees.md`
5. **Note d'identification** (1 h 30, 14h30-16h00) — `identification_sources.md`
6. **Restitution** (30 min, 16h00-16h30) — 3 min/apprenant en plénière

→ Compétences visées : **C1 — imiter** + **C2 — adapter**.

---

## ✅ Conventions de code

- Python 3.11+
- Pas de transformation de données dans le notebook (juste exploration)
- Markdown structuré dans tous les `.md`
- Mermaid intégré dans `flux_donnees.md`

---

## 🆘 Bloqué·e ?

1. Relis le mini-cours correspondant à ta tâche (cf. [`./ressources/README.md`](./ressources/README.md))
2. Si tu n'arrives pas à parser les logs : pas grave, décris-les en
   pseudo-code dans la note. **Pas de code lourd ici** — c'est M3-B2 demain.
3. Demande sur Discord (`fil-M3-B1`).
