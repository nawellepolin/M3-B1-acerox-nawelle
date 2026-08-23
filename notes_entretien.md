# Notes d'entretien — Sébastien Marchand (Acerox)

> Notes prises au fil de l'eau pendant l'entretien fictif 9h30-10h00.
> **5 catégories à pré-remplir** avant l'entretien. Tu complètes au fil
> de l'eau. Distinction explicite *dit* (ce que Sébastien a effectivement
> formulé) vs *interprété* (ta lecture).
>
> Conservé en livrable du brief.

**Date** : 21/07/2026 — **Durée** : 30 min — **Présents** : Sébastien Marchand (chef de projet industrialisation Acerox) + Nawelle (FastIA)

---

## 1. Besoin métier

> Quelle décision Sébastien veut-il améliorer ? Quel KPI métier ?

**Dit** : Il veut faire évoluer le modèle existant d'un mode réactif à un mode préventif, pour déclencher une intervention *avant* le problème. Objectif affiché : limiter le nombre de défauts sur un site en particulier, qui en compte 3x plus que les autres. Le modèle analyse les données de capteurs pour donner une information sur... (réponse interrompue / pas allée au bout — à reclarifier).

**Interprété** : La vraie décision à améliorer, ce n'est pas "avoir un modèle prédictif" en soi, c'est "quand déclencher une maintenance sur telle ligne". Le site 3x plus problématique semble être le vrai point de douleur derrière toute la démarche — le reste (passage réactif → préventif) n'est qu'un moyen d'y arriver. Sébastien n'a pas terminé sa phrase sur ce que le modèle doit précisément prédire à partir des capteurs — signe qu'il n'a peut-être pas lui-même une vision complètement formée de la sortie attendue au moment de cette question (voir section Critères de succès où c'est finalement précisé : une décision de maintenance oui/non).

## 2. Sources et formats

> Qu'a-t-il à disposition ? Où ? Sous quel format ?

**Dit** : Format JSON pour l'ERP, texte brut pour les données machine.

**Interprété** : Réponse partielle : Sébastien ne mentionne explicitement que 2 formats (JSON ERP, texte brut machine) alors qu'on sait par ailleurs qu'il existe aussi des données de capteurs au format CSV. Soit il les a oubliées, soit il les range mentalement dans "données machine" sans distinguer le format des logs (texte) de celui des capteurs (CSV) — à vérifier, ça montre qu'il ne maîtrise pas finement l'inventaire technique de ses propres données (cf. mini-cours : "il sur-estime ce qu'il a comme données").

## 3. Volumétrie et fréquence

> Combien de données ? À quelle cadence arrivent-elles ?

**Dit** : Une mesure par seconde pour les capteurs ; export ERP quotidien (J+1). Données collectées depuis 2 ans, mais seul 1 mois a été mis à disposition pour ce projet. 3 sites au total, dont seulement 2 qui font de la production (avec les mêmes machines).

**Interprété** : Le "1 mesure/seconde" annoncé est à vérifier une fois les fichiers en main — les volumétries déclarées à l'oral sont rarement exactes (cf. mini-cours : "on a 1 million de lignes → souvent c'est 100k"). Le fait qu'il n'y ait que 2 sites sur 3 en production laisse penser que le 3ᵉ site (probablement celui qui n'a qu'une seule ligne suivie dans les capteurs) n'est pas comparable aux deux autres — à ne pas mélanger dans l'analyse du site "3x plus problématique".

## 4. Contraintes (RGPD, sécurité, propriété)

> Quelles obligations légales ? Qui possède les données ? Quels accès
> sécurisés ?

**Dit** : Les données ERP contiennent des données sensibles (identité des opérateurs). Une pièce non conforme coûte 12 000 €. Pas de contrainte sur le temps d'entraînement ni sur la latence. Préférence pour une maintenance anticipée plutôt que de rater un décalage sur les lignes et devoir arrêter la production.

**Interprété** : Sébastien mentionne spontanément la sensibilité des données opérateurs sans qu'on ait eu à insister — bon signe, mais ça reste une réponse orale, pas une validation DPO formelle (à écrire noir sur blanc dans les points à clarifier). L'absence de contrainte de latence + la préférence pour l'anticipation traduisent un vrai arbitrage métier : il accepte des faux positifs (maintenances déclenchées "pour rien") pour éviter à tout prix un faux négatif (un vrai problème raté qui coûte 12 000 € et arrête une ligne). C'est une contrainte de modélisation implicite (favoriser le rappel) qu'il n'a jamais formulée en ces termes.

## 5. Critères de succès

> Comment Sébastien saura-t-il qu'on a réussi ?

**Dit** : -20 % de non-conformité sur la ligne problématique en 3 mois. En sortie, il attend une prédiction de la maintenance à déclencher ou non.

**Interprété** : Le critère est chiffré et borné dans le temps, ce qui est un bon signe (rare qu'un client donne un objectif aussi précis spontanément). Mais il présuppose qu'on sache déjà mesurer la "non-conformité" quelque part dans le système d'information — or aucune source évoquée jusqu'ici ne semble contenir ce label explicitement (l'ERP a un `statut` mais pas de champ "non conforme"). C'est le trou le plus important à combler avant de pouvoir même évaluer un futur modèle.

---

## Questions restées sans réponse

> Notes-le honnêtement — c'est précieux pour la note d'identification.

- Où est enregistrée la "non-conformité" (le KPI de succès) ? Dans quel système, sous quel format ?
- Quel est le site exact qui a 3x plus de problèmes que les autres ? Sébastien ne l'a pas nommé.
- Le 3ᵉ site (hors production) est-il quand même dans le périmètre du projet, ou à exclure d'office ?
- Les données de capteurs existent-elles aussi en dehors de "texte brut" et JSON — sous quel format exactement, et Sébastien en a-t-il conscience ?
- Le DPO d'Acerox a-t-il déjà été consulté sur l'usage des identifiants opérateurs dans un modèle prédictif ?
- Quelle est la fréquence réelle des mesures capteurs (le "1/seconde" annoncé est-il vérifié) ?

## Mes impressions à chaud (10 min après)

Sébastien articule bien l'enjeu financier (12k€/pièce) et le compromis rappel/précision qu'il attend du modèle, mais reste flou dès qu'on entre dans le détail technique des sources : il ne cite que 2 formats sur au moins 3, ne nomme pas le site problématique, et n'a pas de définition opérationnelle de son propre critère de succès ("non-conformité") — signe classique du client qui connaît son métier mais pas son système d'information. Ma prochaine étape n'est pas de lui redemander des précisions par écrit tout de suite, mais d'aller regarder les fichiers moi-même : plusieurs de ces zones grises (le site problématique, le format des capteurs, la fréquence réelle) se répondent probablement en 10 minutes d'exploration de données plutôt qu'en relance d'entretien.

---

*Notes d'entretien produites par Nawelle, 21/07/2026.*
