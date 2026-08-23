# Note d'identification des sources — Acerox Métallurgie

> Document remis à **Sébastien Marchand** (chef de projet industrialisation
> Acerox). **2-3 pages max.** Public : décideur métier non-technique —
> langage courant, pas de jargon scikit-learn ou SQL.
> Auteur : Nawelle — Date : 21/07/2026

## 1. Contexte

> 1 paragraphe — qui est Acerox ? quel est leur existant ? quelle est la
> demande FastIA ?

Acerox est une entreprise de métallurgie. Elle dispose déjà d'un modèle d'IA réactif (qui constate les problèmes après coup) et souhaite évoluer vers un modèle prédictif, ce qui l'a amenée à solliciter FastIA. Les données capteurs sont collectées depuis 2 ans, mais un seul mois a été mis à disposition pour ce projet, sur 3 sites (dont 2 seulement font de la production, avec les mêmes machines).

## 2. Demande métier reformulée

> 1 paragraphe — ce que Sébastien veut **vraiment**, distingué de ce qu'il
> a *dit*. Reformulation en termes de décision métier à améliorer.

Ce que Sébastien a demandé : faire évoluer le modèle existant d'un mode réactif à un mode préventif, pour déclencher une intervention *avant* que le problème survienne.

Ce que je comprends qu'il cherche vraiment : réduire les non-conformités sur un site précis qui concentre 3 fois plus de problèmes que les autres — chaque pièce non conforme coûtant environ 12 000 €, l'enjeu est avant tout financier. Il n'y a pas de contrainte de rapidité : ce qui compte, c'est la fiabilité de la décision. Sébastien préfère nettement déclencher une maintenance qui s'avère inutile plutôt que de rater un vrai problème et devoir arrêter une ligne en urgence — le modèle doit donc privilégier la prudence, quitte à sur-alerter. Le succès se mesure à -20 % de non-conformité sur la ligne problématique en 3 mois, et la sortie attendue n'est pas un score technique mais une décision claire : déclencher une maintenance ou non.

## 3. Inventaire des sources

> Une ligne par source que **vous** avez identifiée (fichiers reçus +
> ce que vous découvrez en explorant les données). Le nom exact du fichier
> fait partie de l'inventaire à dresser.

| Source | Format | Volume | Fréquence | Qualité observée | Risques RGPD | Pertinence métier |
|---|---|---|---|---|---|---|
| `capteurs_iot.csv` | CSV | 51 000 relevés, 8 capteurs sur 3 sites (Lyon, Roubaix, Saint-Étienne) | Environ 1 mesure toutes les 30 min par capteur, sur 1 mois (avril 2026) — en écart avec le rythme "temps réel" annoncé en entretien, à clarifier | Quelques valeurs manquantes et doublons (~1-2 %) ; **sur la ligne 3 de Roubaix, deux problèmes distincts** : un capteur de vibration qui semble bloqué (perte de signal probable), et un capteur de température qui affiche des valeurs anormalement hautes — mais qui, une fois l'unité vérifiée, redevient cohérent avec le reste de l'usine (probable erreur d'unité, pas une vraie surchauffe) | Aucune donnée personnelle | Source la plus riche : la panne de vibration est probablement réelle et exploitable ; la "surchauffe" ne doit pas être prise pour argent comptant tant que l'unité n'est pas confirmée |
| `erp_export.json` | JSON, export quotidien (J+1) | 2 000 ordres de fabrication, 10 références produit | Quotidien | Ne couvre que Roubaix et Saint-Étienne (**Lyon absent**) ; ~5 % d'identifiants d'employé manquants | ⚠️ Contient un identifiant nominatif d'employé — donnée sensible confirmée en entretien | Donne le contexte production (quantités, statuts) mais **ne contient aucun indicateur de non-conformité**, pourtant l'objectif chiffré du projet |
| `logs_machines.log` | Texte brut, en continu | 30 000 lignes sur 1 mois | Continu | Niveaux Info/Alerte/Erreur ; nécessite un traitement pour être exploitable ; la ligne 3 de Roubaix déclenche 2x plus d'alertes que les autres lignes | Aucune donnée personnelle | Coïncide avec les anomalies capteurs sur la même ligne, mais ne prouve pas une vraie surchauffe : une partie des alertes vient probablement de la même erreur d'unité ; le taux d'alerte plus élevé s'explique plus sûrement par la vraie panne de vibration |
| `capteurs_site_A.csv` / `capteurs_site_B.csv` | CSV (échantillons) | ~20-25 lignes chacun, quelques machines | Horaire, 1 seul jour (juin 2026 — période différente du reste) | Formats différents entre les deux fichiers et du fichier principal ; mêmes identifiants de machine utilisés dans les deux fichiers mais pour des équipements manifestement différents ; l'un des deux fichiers a la même erreur d'unité de température que la ligne 3 de Roubaix | Aucune donnée personnelle | Échantillons isolés, non comparables entre eux ni avec `capteurs_iot.csv` (période et format différents) — à ne surtout pas utiliser pour tirer une conclusion sur une autre source (piège identifié, voir `M3-B1_fusion_provenance.ipynb`) |

**Pourquoi on garde une colonne de provenance** : en réunissant `capteurs_site_A.csv` et `capteurs_site_B.csv`, on a ajouté une colonne indiquant l'origine de chaque ligne avant de les empiler. C'est elle seule qui a permis de repérer que les deux fichiers utilisent les mêmes identifiants de machine pour des équipements différents, et qu'un des deux fichiers a une erreur d'unité de température — sans cette traçabilité, ces problèmes seraient passés inaperçus.

### Risques RGPD identifiés

1. **Identifiant direct** : `erp_export.json` contient `ouvrier_id`, un identifiant nominatif d'employé. C'est une donnée sensible confirmée par Sébastien en entretien. *Recommandation* : pseudonymiser `ouvrier_id` (hachage à clé, remplacé par un identifiant technique) avant tout chargement en base pivot, et ne jamais l'exposer au modèle sous sa forme brute.

2. **Réidentification indirecte par croisement** : pris isolément, ni `capteurs_iot.csv` ni `logs_machines.log` ne contiennent de donnée nominative. Mais en croisant `logs_machines.log` (horodatage + `LINE-n` + alerte) et `capteurs_iot.csv` (site + ligne + horodatage) avec l'ERP (`ouvrier_id` rattaché à un ordre de fabrication, donc à une plage horaire sur une ligne), on peut reconstituer **quel opérateur était aux commandes d'une ligne au moment précis d'une alerte ou d'une anomalie** — sans qu'aucun champ ne soit explicitement nominatif dans les logs ou les capteurs. C'est un risque de réidentification indirecte, pas une simple variable sensible à retirer. *Recommandation* : ne pas croiser logs/capteurs et ERP à la maille "ligne + horodatage précis" tant que `ouvrier_id` n'est pas pseudonymisé en amont ; si une analyse par opérateur est un jour nécessaire, l'agréger (par équipe/poste) plutôt que par individu, et la soumettre au DPO Acerox avant mise en œuvre.

## 4. Recommandations

> 3-5 puces. Quelles sources ingérer en priorité ? Lesquelles écarter et
> pourquoi ?

- Prioriser `capteurs_iot.csv` et `logs_machines.log` : ces deux sources se recoupent sur la ligne 3 de Roubaix, qui correspond très probablement au "site 3x plus problématique" cité en entretien.
- Intégrer `erp_export.json` en complément (contexte quantités/statuts), en anonymisant l'identifiant employé avant tout traitement, et en excluant Lyon des comparaisons puisqu'il n'a pas d'ERP.
- Avant toute conclusion sur la ligne 3 de Roubaix, faire vérifier par l'équipe technique si le capteur de température affiche bien en degrés Celsius : une fois l'hypothèse d'erreur d'unité testée, les valeurs redeviennent parfaitement normales. Le capteur de vibration, lui, reste un problème distinct et probablement réel, à traiter comme une vraie panne matérielle.
- Écarter `capteurs_site_A.csv` et `capteurs_site_B.csv` comme données d'entraînement : échantillons trop petits, période différente, formats incompatibles entre eux. Ne surtout pas s'en servir pour "confirmer" l'anomalie de la ligne 3 de Roubaix — leur propre anomalie de température vient du même type d'erreur, pas d'un vrai phénomène partagé.
- Avant tout entraînement, obtenir une source de vérité pour la "non-conformité" : c'est l'objectif chiffré du projet (-20 % en 3 mois) mais aucune source actuelle ne l'enregistre.
- Vu l'absence de contrainte de rapidité et la préférence exprimée pour éviter de rater un vrai problème, privilégier une approche prudente, quitte à générer des interventions superflues.
- Ne pas croiser `logs_machines.log`/`capteurs_iot.csv` avec l'ERP à la maille individuelle (ligne + horodatage précis) avant pseudonymisation de `ouvrier_id` : ce croisement permet de reconstituer indirectement quel opérateur était sur une ligne au moment d'une anomalie (voir risques RGPD, section 3).

## 5. Points à clarifier avec Sébastien

> 3-5 questions ouvertes restantes — preuve de lucidité sur ce qu'on ne
> sait pas encore.

1. Où est enregistrée la "non-conformité" citée comme objectif de succès ? Aucun fichier reçu ne contient cette information.
2. Le capteur de température de la ligne 3 de Roubaix affiche-t-il bien en degrés Celsius ? Une fois l'hypothèse d'erreur d'unité testée, la "surchauffe" disparaît et redevient normale — à confirmer avec l'équipe technique avant de conclure à un vrai risque thermique.
3. Le capteur de vibration de cette même ligne est-il un problème matériel déjà connu de la maintenance terrain ?
4. Le "site à 3x plus de problèmes" cité en entretien est-il bien celui identifié (Roubaix, ligne 3), ou fait-il référence à un autre indicateur que nos données ne permettent pas, à elles seules, de confirmer ?
5. Pourquoi l'ERP ne couvre-t-il pas Lyon, alors que les capteurs si ? Lyon fait-il partie des "2 sites qui font de la production" évoqués en entretien ?

## 6. Limites de cette note

> Ce qu'on n'a **pas** fait, et qu'il faudrait faire plus tard.

- Pas d'analyse statistique fouillée des sources (M3-B1 = identification,
  pas EDA complète)
- Pas d'AIPD juridique formelle (recommandation : escalader au DPO Acerox)
- L'hypothèse d'erreur d'unité de température est déduite par calcul, pas confirmée par l'équipe technique Acerox — à valider avant toute décision
- Pas de vérification terrain de l'hypothèse de panne du capteur de vibration

---

*Note produite par Nawelle, 21/07/2026, dans le cadre du brief M3-B1 ATOS.*
