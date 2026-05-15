# Règles de calcul et valeurs par défaut (v5.2)

Formules exactes, seuils et défauts pour générer le CSV de priorisation v5 (38 colonnes). À lire après `framework_v5.md`.

## Frontière math / jugement (v2.2)

**Le CSV est la source de vérité numérique** : verdicts ROI et Confiance Globale calculés mécaniquement par les seuils ci-dessous, immutables une fois la ligne remplie.

**La synthèse exécutive est un layer de jugement LLM** par-dessus le CSV : le LLM peut promouvoir, rétrograder ou bundler des use cases dans la synthèse s'il a une raison citée. Mais ces overrides apparaissent uniquement dans `synthese_priorisation.md` ; le CSV reste fidèle aux verdicts mécaniques.

Cette séparation préserve la traçabilité : un reviewer peut toujours comparer le ranking mécanique du CSV vs le ranking synthèse, et juger si les overrides LLM sont justifiés.

## Format des cellules qualitatives

Les 9 colonnes 1-3 (cols 6-13 et 32) utilisent le format **`<chiffre> - <label>`** dans la cellule :
- Exemples : `3 - récurrent standardisé`, `2 - moyen (rework)`, `1 - faible`.
- Pour les calculs, le LLM extrait le préfixe numérique : `parseNumber("3 - récurrent standardisé") = 3`.
- Si la cellule est vide ou ne commence pas par un chiffre 1-3, elle compte comme NON remplie pour la confiance.

## Barème T-shirt (col 25 Build Base)

| T-shirt | Build Base ($) | Effort indicatif |
|---|---|---|
| XS | 10 000 | < 2 semaines |
| S | 25 000 | 3-4 semaines |
| M | 50 000 | 6-8 semaines |
| L | 90 000 | 10-12 semaines |
| XL | 150 000 | 14+ semaines |

Si l'architecte fournit un montant override (ex: "M mais plutôt 65k"), utiliser le montant override.

## Valeurs par défaut

**Taux Réalisation (col 22)** : `0.6` par défaut.
- 0.5 pour use cases à fort impact organisationnel (adoption complexe)
- 0.6 pour cas standards
- 0.7 pour automatisations à faible friction utilisateur

**Risque (col 32)** : `2 - moyen` par défaut.

**Coût Unitaire Agent (col 27)** : pas de défaut chiffré ; rempli par benchmark web (voir section dédiée). Fallback si recherche échoue : `(Build × 0.15) / Volume Qté` avec flag `BENCHMARK_FALLBACK` dans Notes.

## Formules de calcul

### Suitability Score (col 14)
```
Suitability = MOYENNE(parseNumber(cols 6, 7, 8, 9, 10, 11, 12, 13)) × 33.33
```
Donne un score sur 100 (range effective 33.33-100).

### Gain Heures Annuel (col 20)
```
Gain_H = Volume_Qte × (Temps_Actuel - Temps_Cible)
```
Si Temps_Actuel = 0 (cas pertes évitées directes uniquement), Gain_H = 0.

**Note v5** : la formule ne multiplie plus par Personnes Impactées. Volume Qté = total annuel d'occurrences, tous exécutants confondus.

### Bénéfice Brut Annuel (col 21)
```
SI Temps_Actuel = 0:
  Benefice_Brut = Pertes_Evitees_Directes
SINON:
  Benefice_Brut = Gain_H × Cout_Horaire + Pertes_Evitees_Directes (si applicable)
```

### Bénéfice Net Annuel (col 23)
```
Benefice_Net = Benefice_Brut × Taux_Realisation
```

### Build Base (col 25)
```
SI montant fourni: utiliser le montant
SINON: lookup table t-shirt (voir barème)
SI ni montant ni t-shirt: laisser vide (critique pour confiance)
```

### Run Annuel (col 28) — NOUVELLE FORMULE v5
```
Run_Annuel = Volume_Qte × Cout_Unitaire_Agent
```
Remplace l'ancienne heuristique `Build × 0.15`. Si Coût Unitaire Agent absent et benchmark indisponible, fallback documenté en Notes.

### Coût An 1 Total (col 29)
```
Cout_An1 = Build_Base + Run_Annuel
```

### Payback en mois (col 30)
```
SI (Benefice_Net - Run_Annuel) <= 0:
  Payback = 999 (jamais rentable)
SINON:
  Payback = Build_Base × 12 / (Benefice_Net - Run_Annuel)
```

### ROI An 1 en % (col 31)
```
ROI_An1 = (Benefice_Net - Cout_An1) / Cout_An1 × 100
```

### Score Priorité (col 33)
```
Score = Benefice_Net / (Build_Base × parseNumber(Risque))
```
Risque ∈ {1, 2, 3}. Plus le score est élevé, plus le use case est prioritaire.

### Confiance Données en % (col 34)
```
Conf_Donnees = Nb_Colonnes_Critiques_Remplies / 20 × 100
```

### Validation LLM (col 35, v2.1)

**Auto-évaluation par le LLM** ligne par ligne. Format cellule : `<n> - <label>`.

Rubric :
- `3 - source primaire validée` : 
  - Sponsor explicitement nommé ET présent en atelier
  - Volume Qté EXPLICITEMENT cité dans le transcript (pas inféré)
  - Temps Actuel/Cible OU Pertes Évitées EXPLICITEMENT cité avec chiffre précis
  - Pain Point cité textuellement (citation client)
- `2 - mixte source + estimation` : 
  - Sponsor nommé mais niveau d'engagement déduit du contexte
  - 1-2 chiffres clés cités dans transcript, autres extrapolés
  - Pain Point reformulé depuis description du contexte
- `1 - estimation contextuelle` :
  - Sponsor `À identifier` OU inféré du département
  - Volume/Temps/Pertes TOUS estimés depuis le contexte général
  - Pain Point reformulé sans citation directe
  - Suitability inférée plus que validée par signaux explicites

Le LLM doit **justifier le niveau choisi en col 38 Notes** par 1-2 citations du transcript ou par la nature des estimations.

### Confiance Globale (calculée pour Verdict Confiance)

**Pas une colonne visible** dans le CSV (calculée à la volée). Combine complétude × validation :

```
Validation_Multiplier:
  "3 - source primaire validée"    → 1.0
  "2 - mixte source + estimation"  → 0.75
  "1 - estimation contextuelle"    → 0.5
  (vide ou invalide)               → 0.5 (défaut conservateur)

Confiance_Globale = Confiance_Donnees × Validation_Multiplier
```

C'est sur cette `Confiance_Globale` que le Verdict Confiance (col 36) applique ses seuils.

## Benchmark web du Coût Unitaire Agent

À effectuer en **avant-dernière étape** du workflow (avant la synthèse exec), pour chaque ligne du CSV en fonction du Type Agent.

### Méthode

1. Lancer une recherche web ciblée :
   - `"[type agent] cost per run 2026 benchmark"`
   - `"[type agent] LLM inference cost per call site:a16z.com OR site:menlovc.com"`
   - `"[type agent] API pricing 2026"`
2. Sources prioritaires : pricing officiels (OpenAI, Anthropic), rapports a16z / Menlo / Sequoia, blogs ingénierie produit récents (<12 mois).
3. Prendre la **valeur médiane** des benchmarks trouvés. Remplir col 27 (Coût Unitaire Agent).
4. Citer source + date dans col 37 Notes :
   ```
   Benchmark Run: $0.04/email auto (a16z State of AI 2026, consulté 2026-05-13)
   ```
5. Si aucun benchmark fiable trouvé : fallback `(Build × 0.15) / Volume Qté`, flag `BENCHMARK_FALLBACK` en Notes.

### Table de fallback indicative (à valider chaque trimestre)

| Type Agent | $/run typique 2026 |
|---|---|
| email auto | 0.02 - 0.05 |
| dashboard genAI | 0.50 - 1.00 |
| RAG simple | 0.005 - 0.02 |
| RAG complexe | 0.05 - 0.15 |
| OCR | 0.001 - 0.01 |
| RPA déterministe | 0.0001 - 0.001 |
| agent conversationnel | 0.10 - 0.30 |
| document processing | 0.05 - 0.20 |

Ces valeurs sont indicatives — toujours préférer un benchmark web frais à jour.

## Liste des 20 colonnes critiques (pour confiance col 34)

1. Pain Point (col 2)
2. Sponsor / Owner (col 4) — "À identifier" compte comme NON rempli
3. Département (col 5)
4. Volume Standardisation (col 6)
5. Règles Claires (col 7)
6. Disponibilité Données (col 8)
7. Stabilité Processus (col 9)
8. Faible Taux Exceptions (col 10)
9. Impact Erreurs Actuelles (col 11)
10. Alignement Stratégique (col 12)
11. Sponsor Engagé (col 13)
12. Volume Qté (col 15)
13. Temps Actuel (col 16)
14. Temps Cible (col 17)
15. Coût Horaire OU Pertes Évitées Directes (cols 18 ou 19, une des deux suffit)
16. Taux Réalisation (col 22)
17. Taille Build (col 24)
18. Type Agent (col 26)
19. Coût Unitaire Agent (col 27)
20. Risque (col 32)

**Règles de comptage** :
- Valeur numérique = compté comme rempli
- Cellule au format `<n> - <label>` avec n ∈ {1,2,3} = compté comme rempli
- `N/A` (string) = compté comme rempli (la question a été considérée)
- Vide = NON compté
- `À identifier` / `À valider` pour Sponsor = NON compté
- `0` = compté comme rempli (zéro est une valeur valide pour Pertes Évitées)

## Seuils de verdicts

### Verdict Confiance (col 36) — basé sur Confiance Globale hybride
| Confiance Globale | Verdict |
|---|---|
| ≥ 80% | Fiable |
| 60-79% | À valider 1-2 points |
| 40-59% | Hypothèses fortes |
| < 40% | Atelier de validation requis |

Où `Confiance_Globale = Confiance_Donnees × Validation_Multiplier` (voir section ci-dessus).

**Exemples** :
- Complétude 100% + Validation `3 - validée` → 100% → `Fiable`
- Complétude 100% + Validation `2 - mixte` → 75% → `À valider 1-2 points`
- Complétude 100% + Validation `1 - estimation` → 50% → `Hypothèses fortes`
- Complétude 55% + Validation `1 - estimation` → 27.5% → `Atelier de validation requis`

### Verdict ROI (col 37) — basé sur col 30 Payback
| Payback (mois) | Verdict |
|---|---|
| < 6 | Quick win |
| 6-12 | Go |
| 12-18 | À challenger |
| 18-24 | Stratégique seulement |
| > 24 (ou 999) | Pass |

## Conventions de remplissage

### Quand le Pain Point n'est pas explicite
Reformuler le problème central en une phrase. Exemples :
- Post-it "30h/semaine" + "erreurs scrap" → "30h/sem erreurs production, scrap milliers de $"
- Transcript "on perd beaucoup d'argent en non-conformité" + chiffre 160k$ → "160k$ pertes non-conformité par an"

### Quand le Sponsor est nommé
Si le client cite une personne ("Robert au VP ventes s'occupe de ça"), mettre "Robert VPventes". Si seulement un département est mentionné, "À identifier" dans Sponsor (compte NON rempli).

### Quand les volumes sont approximatifs
Si le client dit "environ 12k-15k par année", utiliser la médiane (13 500) et noter "Volume entre 12k et 15k confirmé" dans Notes.

### Quand le Temps Cible n'est pas spécifié
Heuristique : viser 20% du Temps Actuel pour les processus AI-assistés. Documenter "Temps cible estimé à 20% (à valider)" dans Notes.

### Quand l'input mentionne des montants en $
- Lié à des pertes/erreurs → Pertes Évitées Directes ($) (col 19)
- Lié à un coût de personne → Coût Horaire ($) (col 18)
- Lié à un build → Build Base ($) (col 25)
