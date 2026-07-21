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
    %% TODO — Compléter avec tes 3 sources retenues + 1 bonus si traité

    SRC1[📡 ?<br/>Format ?<br/>Volume ?<br/>Fréquence ?]
    SRC2[📋 ?<br/>Format ?]
    SRC3[📝 ?<br/>Format ?]

    INGEST[🔄 Ingestion<br/>à concevoir en M3-B2]
    BDD[(🗄️ BDD pivot<br/>SQLite)]
    MODEL[🧠 Modèle existant Acerox<br/>prédiction défauts qualité]

    SRC1 --> INGEST
    SRC2 --> INGEST
    SRC3 --> INGEST
    INGEST -->|normalisation + dédup| BDD
    BDD -->|consommée par| MODEL

    classDef source fill:#e1f5ff,stroke:#0277bd
    classDef tofix fill:#fff4e1,stroke:#c97a00,stroke-dasharray: 5 5
    class SRC1,SRC2,SRC3 source
    class INGEST tofix
```

## Légende

> Reformule en 5 lignes max ce que le schéma raconte (qui produit quelle
> donnée, qui consomme, contraintes critiques).

- **Producteur** : ...
- **Consommateur final** : ...
- **Contraintes critiques** (fréquence / RGPD / qualité) : ...

## Décisions associées

- Source(s) retenues en priorité : ...
- Source(s) écartées : ...
- Source bonus (rapports `.md`) traitée ? oui / non, pourquoi : ...

---

*Schéma produit par <prénom>, <date>, dans le cadre du brief M3-B1 ATOS.*
