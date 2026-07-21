# Préparer un corpus pour du RAG — Mini-cours

> Brief associé : M3-B1 (mission ⭐ optionnelle)
> Durée de lecture : ~25 min
> Pré-requis : avoir exploré les 3 sources Acerox ; Python/pandas ; avoir
> installé les deps bonus (`chromadb`, `sentence-transformers`).

## Pourquoi cette techno ?

Les rapports techniques d'Acerox (`rapports_techniques_2024.zip`) sont du **texte
non structuré** : impossible à ranger proprement dans une table SQL, et inutile
à « scorer » comme une donnée tabulaire. Pourtant ils contiennent du savoir
métier précieux (causes de défauts, fiabilité des capteurs…). On veut pouvoir
les **interroger par le sens** — « pourquoi des défauts sur le laminage ? » —
et récupérer les passages pertinents, même s'ils n'emploient pas les mots exacts
de la question.

La recherche plein-texte classique (`LIKE '%laminage%'`, `grep`) échoue dès qu'il
y a synonyme ou reformulation. La **recherche sémantique** résout ça : elle
compare le *sens*, pas les caractères. C'est la brique de récupération
(*retrieval*) d'un système RAG.

> ⚠️ **On s'arrête à la récupération.** Brancher un LLM qui *génère* une réponse
> à partir des passages (le « G » de RAG), c'est **M7-B2**. Ici tu construis et
> tu interroges l'index — rien de plus. Et pour la *prédiction de défauts*
> (tabulaire), un RAG **n'aide pas** : ce corpus sert l'**aide au diagnostic
> humain**, pas le modèle.

## Concepts clés

- **Chunk (segment)** : un morceau de texte de taille gérable découpé dans un
  document. On n'indexe pas un rapport entier d'un bloc, mais des **passages** —
  pour retrouver *le bon paragraphe*, pas *tout le fichier*.
- **Embedding (plongement)** : un **vecteur de nombres** qui représente le *sens*
  d'un texte. Deux textes de sens proche → deux vecteurs proches. C'est ce qui
  permet de chercher par sens et non par mot-clé.
- **Similarité sémantique** : la **distance** entre deux embeddings. Petite
  distance = sens proche. C'est le critère de classement des passages pertinents.
- **Base vectorielle (ici ChromaDB)** : une base qui **stocke les embeddings** et
  retrouve vite les plus proches d'une requête. Un index, mais sur le sens.
- **Modèle d'embedding** : le modèle qui transforme un texte en vecteur. Son
  choix est une **décision** (langue, domaine) — pas un détail. Les rapports
  Acerox sont **en français** → un modèle anglophone sous-performe.

> Le geste tient en 4 temps : **découper → encoder → ranger → interroger.**

## Exemple minimal qui tourne

Autonome (n'a pas besoin du zip), pour saisir l'idée en 2 minutes :

```python
# sentence-transformers==3.3.1  ·  chromadb==0.5.18
from sentence_transformers import SentenceTransformer
import chromadb

textes = [
    "Le capteur de température de la ligne 3 renvoie des valeurs aberrantes.",
    "La maintenance préventive réduit les arrêts machine.",
    "Le taux de rebuts a augmenté après le changement de fournisseur.",
]
modele = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")  # adapté au FR
emb = modele.encode(textes).tolist()

client = chromadb.Client()                      # en mémoire, pour l'exemple
col = client.create_collection("demo")
col.add(ids=["d1", "d2", "d3"], documents=textes, embeddings=emb)

q = "problème de sonde thermique sur une ligne de production"
res = col.query(query_embeddings=modele.encode([q]).tolist(), n_results=2)
for doc, dist in zip(res["documents"][0], res["distances"][0]):
    print(f"{dist:.3f}  {doc}")
```

Résultat attendu : la requête « **sonde thermique** » fait remonter en tête « **capteur
de température** » — alors qu'**aucun mot n'est commun**. C'est ça, la recherche
par le sens.

## Exercice guidé

À partir de l'exemple ci-dessus :
1. Remplace le modèle par `all-MiniLM-L6-v2` (anglophone) et **compare les
   distances** sur les mêmes textes FR. Tu verras qu'elles se **dégradent**.
2. Change la requête en quelque chose de proche d'un seul des 3 textes, et
   vérifie que le bon passage remonte en premier.

> Leçon attendue : sur un corpus français, le modèle **multilingue** resserre les
> distances → meilleure récupération. **Le modèle d'embedding se choisit selon la
> langue et le domaine.** (On creusera ça en M7.)

## Pièges fréquents

| Piège | Conséquence |
|---|---|
| Indexer le rapport **entier** sans chunker | la requête retrouve « tout le fichier », pas le passage utile |
| Mélanger des embeddings de **modèles différents** dans une collection | distances incomparables, classement faux |
| Modèle d'embedding **anglophone** sur corpus FR | similarité dégradée, passages mal classés |
| Relancer le notebook sans **vider** la collection | doublons qui s'accumulent à chaque run (non idempotent) |
| Croire que c'est « du RAG complet » | non : c'est la **préparation + récupération**. La génération = M7-B2 |

| Symptôme | Cause probable |
|---|---|
| Les distances sont **toutes > 0.9** | modèle d'embedding pas adapté à la langue du corpus |
| La requête renvoie n'importe quoi | chunks trop gros (un chunk = plusieurs sujets) ou corpus mal chargé |
| `ImportError: sentence_transformers` | deps bonus pas installées (`pip install sentence-transformers chromadb`) |
| La collection **grossit** à chaque exécution | pas de reset/`delete` avant le `add` |
| Téléchargement du modèle qui bloque | 1er run = téléchargement (~120 Mo), c'est normal ; relancer si coupé |

## Pour aller plus loin

- Doc officielle ChromaDB : <https://docs.trychroma.com/>
- sentence-transformers — *Pretrained models* : <https://www.sbert.net/docs/pretrained_models.html>
- Modèles d'embedding **français/multilingues** : `paraphrase-multilingual-MiniLM-L12-v2`,
  `dangvantuan/sentence-camembert-base` (voir la galerie modèles pré-entraînés des ressources publiques).
- La suite logique : **M7-B2** (RAG complet — récupération **+ génération** par un LLM, avec arbitrage vs ML classique).

## Vérification (checklist apprenant)

- [ ] J'ai fait tourner l'exemple minimal et vu « sonde thermique » → « capteur de température »
- [ ] Je sais expliquer en 2 min ce qu'est un **embedding** et une **distance sémantique**
- [ ] J'ai compris pourquoi on **chunke** au lieu d'indexer le fichier entier
- [ ] J'ai vu l'effet du **modèle d'embedding** (FR multilingue vs anglophone) sur les distances
- [ ] Je sais où s'arrête la **préparation/récupération** vs le **RAG complet (M7-B2)**
- [ ] J'ai écrit 3 lignes dans ma note sur **relationnel (capteurs/ERP) vs vectoriel (rapports)**
