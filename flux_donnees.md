# Schéma des flux de données — Acerox Métallurgie

> Schéma Mermaid à compléter. Doit montrer :
> - **Sources** (capteurs IoT, ERP, logs, *bonus rapports `.md`*)
> - **Ingestion** (à concevoir en M3-B2)
> - **BDD pivot** (à modéliser en M3-B2)
> - **Modèle existant** Acerox (placeholder, hors-sujet ici)
>
> Légende explicite : qui produit, qui consomme, contraintes.

## Schéma

```mermaid
flowchart LR
    SRC_IOT[📡 capteurs_iot.csv<br/>CSV, 51k lignes/mois<br/>continu<br/>Provenance : capteurs terrain<br/>Fraîcheur : avril 2026<br/>Version : schéma stable]
    SRC_ERP[📋 erp_export.json<br/>JSON, 2k ordres/mois<br/>batch quotidien J+1<br/>Provenance : ERP Acerox<br/>Fraîcheur : J+1<br/>Version : contient ouvrier_id RGPD]
    SRC_LOG[📝 logs_machines.log<br/>Texte brut, 30k lignes/mois<br/>continu<br/>Provenance : journal machines<br/>Fraîcheur : temps réel<br/>Version : parsing regex requis]

    INGEST[🔄 Ingestion<br/>à concevoir en M3-B2]
    BDD[(🗄️ BDD pivot<br/>SQLite)]
    MODEL[🧠 Modèle existant Acerox<br/>prédiction défauts qualité]

    SRC_IOT -->|dédup + vérif. unité| INGEST
    SRC_ERP -->|pseudonymisation ouvrier_id| INGEST
    SRC_LOG -.->|parsing regex - bonus| INGEST
    INGEST -->|normalisation + dédup| BDD
    BDD -->|consommée par| MODEL

    classDef source fill:#e1f5ff,stroke:#0277bd
    classDef tofix fill:#fff4e1,stroke:#c97a00,stroke-dasharray: 5 5
    class SRC_IOT,SRC_ERP,SRC_LOG source
    class INGEST tofix
```

## Légende

> Reformule en 5 lignes max ce que le schéma raconte (qui produit quelle
> donnée, qui consomme, contraintes critiques).

- **Producteur** : capteurs terrain (3 sites), ERP Acerox (Roubaix + Saint-Étienne), journal machines — tous sur avril 2026.
- **Consommateur final** : le futur modèle prédictif de maintenance, via la BDD pivot (à concevoir en M3-B2).
- **Contraintes critiques** : `ouvrier_id` (ERP) à pseudonymiser (RGPD, identifiant direct) ; croiser logs/capteurs (ligne + horodatage) avec l'ERP permet de reconstituer indirectement l'opérateur en poste au moment d'une anomalie — à ne pas faire à la maille individuelle avant pseudonymisation (RGPD, réidentification indirecte) ; unité du capteur de température de Roubaix LINE-3 à vérifier avant tout calcul ; logs à parser (regex) ; aucune source ne porte le label "non-conformité", pourtant la cible du modèle.

## Décisions associées

- Source(s) retenues en priorité : `capteurs_iot.csv` et `logs_machines.log` (anomalies exploitables), `erp_export.json` en complément (contexte production).
- Source(s) écartées : `capteurs_site_A.csv` et `capteurs_site_B.csv` — échantillons non comparables (autre période, schéma incompatible, bug d'unité) : voir `M3-B1_fusion_provenance.ipynb`.
- Source bonus (rapports `.md`) traitée ? Non — aucun rapport `.md` fourni dans ce jeu de données.

---

*Schéma produit par Nawelle, 21/07/2026, dans le cadre du brief M3-B1 ATOS.*
