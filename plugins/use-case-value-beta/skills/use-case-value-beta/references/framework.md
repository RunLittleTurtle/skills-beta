# Framework use-case-value-beta v1.2 — 27 colonnes, focus impact chiffré

Ce framework adapte quatre sources méthodologiques à un usage strictement orienté **valeur business chiffrable** :

- **BABOK 10.10 Business Case** (IIBA Business Analysis Body of Knowledge v3) : pour la décomposition de la valeur en sources distinctes.
- **SAFe WSJF (Weighted Shortest Job First)** : adapté en mode impact-only avec Cost of Delay.
- **5 Whys (Taiichi Ohno, Toyota Production System) + Ishikawa Diagram (Kaoru Ishikawa)** : pour la Step 2bis d'analyse root cause.
- **DAMA-DMBOK Data Quality Dimensions** : pour le critère Disponibilité des données (col 20).

Le skill se distingue de `use-case-prioritization-beta` v2.2 par cinq choix structurants :

1. **Aucune dimension d'effort** : focus pur valeur, l'effort est traité par un futur skill complémentaire.
2. **Six sources d'Impact $ explicites** capturant la valeur multi-dimensionnelle (temps, opportunités, erreurs, saturation, dépendances, frictions).
3. **Règle d'or adoucie sur les estimations** (v1.2) : les cellules acceptent les chiffres durs cités, les formules simples sur chiffres durs, et les **inférences obvious sur signaux cités** convertis via les standards de méthode documentés. Les paramètres de conversion inventés (% captable, probabilité, valeur moyenne sans citation) restent en Notes.
4. **Step 2bis Root Cause + colonne Dépend de** : analyse 5 Whys / Ishikawa avant chiffrage pour distinguer root causes actionnables vs symptômes.
5. **Score intègre Impact Potentiel Estimé** (v1.2) : la col 27 capture les estimations Notes avec un discount d'incertitude, pour que le tri par Score reflète aussi le potentiel à chiffrer en atelier.

---

## Liste des 27 colonnes

### Bloc 1 — Identification (cols 1 à 5)

| # | Colonne | Type | Source |
|---|---------|------|--------|
| 1 | Use Case | texte | input |
| 2 | Description | texte 1-2 phrases | input ou LLM synthétise |
| 3 | Sponsor | texte | input |
| 4 | Département principal | texte | input |
| 5 | Type Agent | texte court | LLM catégorise |

**Type Agent (col 5)** — valeurs autorisées :
- `email auto`, `OCR`, `dashboard genAI`, `RAG`, `vision`, `RPA`, `autre`

### Bloc 2 — Contexte d'usage (cols 6 à 9)

| # | Colonne | Type | Rubrique |
|---|---------|------|----------|
| 6 | Volume Annuel Qté | nombre | nombre de runs / unités traitées par an |
| 7 | Volume Annuel Unité | texte court | factures/an, soumissions/an, etc. |
| 8 | Personnes Affectées (Nb) | nombre | combien dans l'organisation sont touchées |
| 9 | Transversalité | 1-3 + label | voir rubrique |

**Transversalité (col 9)** :
- `1 - un département` / `2 - inter-départemental` / `3 - toute l'organisation`

### Bloc 3 — Pain points (cols 10 à 12)

| # | Colonne | Type | Source |
|---|---------|------|--------|
| 10 | Pain Point Principal | texte 1 phrase obligatoire | input |
| 11 | Pain Point #2 | texte 1 phrase optionnel | input |
| 12 | Pain Point #3 | texte 1 phrase optionnel | input |

**Note Step 2bis** : si plusieurs symptômes ont une root cause commune, consolidés ici sur la ligne de la root cause.

### Bloc 4 — Sources d'Impact $ annuel (cols 13 à 18)

**Règle d'or adoucie v1.2** : ces 6 colonnes acceptent :
1. Valeur citée verbatim par sponsor / participant atelier / table source
2. Formule simple sur chiffres durs cités avec standards de méthode comme paramètres de conversion (80 $/h, 100 000 $/FTE)
3. Inférence obvious sur signal cité (ex : "1 FTE saturé" → 100 000 $/an si le sponsor a nommé une quantité FTE)

**Refusé** : multiplicateurs sortis du chapeau (% captable, probabilité, valeur moyenne sans citation), volumes inventés, extrapolations sans anchor verbal.

| # | Colonne | Définition | Exemple de chiffrage valide |
|---|---------|------------|-----------------------------|
| 13 | Impact $ Temps Perdu | Coût annuel du temps humain absorbé par les tâches répétitives | `25 h/sem × 50 × 80 $/h = 100 000 $` (heures citées) |
| 14 | Impact $ Opportunités Manquées | Revenu retardé ou perdu chiffré par le sponsor | `10 contrats/an × 50 000 $ = 500 000 $` (citation verbatim) |
| 15 | Impact $ Coût Erreurs | Coût annuel des erreurs humaines actuelles | `180 000 $ de non-conformité annuelle citée` |
| 16 | Impact $ Main d'Œuvre / Saturation | Surcoût heures sup, embauche évitée, FTE saturé | `1 FTE saturé cité × 100 000 $ = 100 000 $` (citation FTE) |
| 17 | Impact $ Dépendances Externes | Coût des blocages externes | `Pénalités contractuelles 50 000 $/an citées` |
| 18 | Impact $ Frictions Inter-Départementales | Doublons, allers-retours chiffrés temps × salaire | `2 FTE × 20 % × 75 000 $ = 30 000 $` (chiffres cités) |

**Standards de méthode** (voir `calculation_rules.md` pour les conditions d'application) :
- Coût horaire fully-loaded : 80 $/h (appliqué seulement si heures citées par sponsor)
- FTE chargé fully-loaded : 100 000 $/an (appliqué seulement si saturation FTE citée par sponsor)

### Bloc 5 — Score et confiance (cols 19 à 22)

| # | Colonne | Type | Logique |
|---|---------|------|---------|
| 19 | Impact Total Documenté ($/an) | calculé | SUM(col13 à col18). Pur chiffres durs et inférences obvious admissibles. |
| 20 | Disponibilité Données | 1-3 + label | voir rubrique |
| 21 | Complétude Chiffrage | 1-3 + label | voir rubrique v1.2 élargie |
| 22 | Urgence Stratégique | 1-3 + label | voir rubrique |

**Disponibilité Données (col 20)** — DAMA-DMBOK Data Availability :
- `3 - structurées dans système accessible`
- `2 - mixtes partiellement accessibles`
- `1 - dispersées non structurées`

**Complétude Chiffrage (col 21) — rubrique élargie v1.2** :
- `3 - toutes sources pertinentes chiffrées` : 4 à 6 cellules sur 6 ont une valeur non-zéro
- `2 - sources principales chiffrées` : 1 à 3 cellules chiffrées **dont la cellule dominante capturée** (celle qui capte l'essentiel de l'impact pour ce use case)
- `1 - chiffres marginaux` : 0 cellule chiffrée OU 1 cellule sur une dimension secondaire seulement

L'élargissement v1.2 reconnaît qu'un use case avec une seule cellule chiffrée à 250 000 $ qui capture la dimension dominante (ex : Coût Erreurs sur un use case de qualité produit) mérite niveau 2, pas 1.

**Urgence Stratégique (col 22) — rubrique élargie v1.3** : WSJF Cost of Delay + signaux d'engagement sponsor :
- `3 - critique court terme` : OKR explicite, fenêtre client ou concurrence, date butoir mentionnée, OU sponsor activement engagé en atelier avec émotion d'adhésion ("ça m'excite", "wow", "il faut absolument", plusieurs interventions soutenues)
- `2 - important moyen terme` : sponsor identifié, mention récurrente, soutien quand interrogé, pas de date butoir
- `1 - nice-to-have` : mention isolée, pas de sponsor actif

L'élargissement v1.3 reconnaît que les signaux d'engagement émotionnel des sponsors en atelier (Faspac : "ça me ferait tout le monde s'être comme excité" de Valérie) sont des indicateurs d'urgence stratégique au même titre qu'un OKR ou une date butoir.

### Bloc 6 — Verdict (cols 23 à 25)

| # | Colonne | Type | Logique |
|---|---------|------|---------|
| 23 | Score Priorité Impact | calculé | Voir formule v1.3 dans calculation_rules.md (intègre col 27 avec discount 0.3 + Personnes_factor paliers + Transversalité_factor) |
| 24 | Verdict Impact | calculé via algorithme déterministe v1.2 | Voir algorithme numéroté dans calculation_rules.md |
| 25 | Notes | texte structuré | Sources citées, hypothèses, root cause, À chiffrer en atelier, autres notes |

**Notes (col 25)** — structure suggérée, **utiliser des sauts de ligne entre les sections (pas de pipes `|`)** et entourer la cellule complète de guillemets doubles dans le CSV (RFC 4180) :

```
Sources : [citations atelier verbatim, fichier source, date]
Hypothèses : [coût horaire ou FTE standard utilisé, formules]
Root cause : [si applicable]
À chiffrer en atelier :
- [Question]. À titre indicatif, si [hypothèse], cela représenterait [valeur].
Autres notes : [...]
```

Les estimations chiffrées de la section "À chiffrer en atelier" alimentent la col 27 Impact Potentiel Estimé.

**Important format CSV** : la cellule Notes contient des virgules, des retours à la ligne et parfois des guillemets. Elle doit donc être entourée de guillemets doubles dans le fichier CSV (`"..."`) et les guillemets internes doivent être doublés (`""`), conformément à RFC 4180. Ne JAMAIS utiliser de pipes `|` comme séparateurs internes, ils confusent les auto-détecteurs de délimiteur des viewers CSV (Numbers, LibreOffice, Excel).

### Bloc 7 — Dépendances (col 26)

| # | Colonne | Type | Source |
|---|---------|------|--------|
| 26 | Dépend de | texte libre | Use cases prérequis identifiés en Step 2bis |

**Dépend de (col 26)** :
- Liste les use cases que celui-ci nécessite en amont
- Format texte libre, plusieurs prérequis séparés par virgules
- Vide si aucune dépendance
- N'entre pas dans le calcul du Score ni du Verdict, signal qualitatif pour la synthèse

### Bloc 8 — Potentiel à chiffrer (col 27, NOUVEAU v1.2)

| # | Colonne | Type | Source |
|---|---------|------|--------|
| 27 | Impact Potentiel Estimé ($/an) | calculé par LLM | Somme des estimations indicatives chiffrées présentes dans Notes (col 25) sous "À chiffrer en atelier" |

**Impact Potentiel Estimé (col 27)** :
- Le LLM somme les valeurs indicatives chiffrées des items "À chiffrer en atelier" dans Notes du même use case
- Si Notes contient `À titre indicatif si 5 contrats × 100 000 $ × 30 % = +150 000 $` ET `Si 0.5 FTE évité ≈ +50 000 $` → col 27 = 200 000 $
- Si aucune estimation indicative chiffrée dans Notes → col 27 = 0
- **N'entre PAS dans Impact Total Documenté (col 19)** qui reste pur chiffres durs
- **Entre dans le Score Priorité Impact (col 23) avec discount 0.3** pour éviter que les use cases haut potentiel non chiffré apparaissent à Score 0 dans le tri

---

## Sources méthodologiques détaillées

- **BABOK v3 §10.10 Business Case** : justifie la décomposition de la valeur en sources distinctes.
- **SAFe WSJF** : Cost of Delay adapté. Job Size retiré, déféré à un futur skill. SAFe Enabler informe la col 26.
- **5 Whys de Taiichi Ohno + Ishikawa Diagram de Kaoru Ishikawa** : informent la Step 2bis Root Cause Analysis.
- **DAMA-DMBOK §3 Data Quality Dimensions** : Availability/Accessibility pour col 20.
- **MIA Innovation pratique terrain** : la règle d'or adoucie v1.2 vient de l'observation Faspac : la version stricte v1.0/v1.1 produisait des Scores à 0 pour les use cases à fort potentiel non encore chiffré, masquant leur priorité réelle dans le tri. L'adoucissement (inférence obvious sur signal cité + col 27 Impact Potentiel avec discount) préserve l'anti-gonflage tout en rendant le Score utile pour le tri opérationnel.
