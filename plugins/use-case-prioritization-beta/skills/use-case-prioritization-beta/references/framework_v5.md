# Framework de priorisation des use cases AI (v5)

Référence des 38 colonnes du CSV v5. Combine triage qualitatif (Suitability), business case quantitatif (ROI), output architecte minimal, Run cost benchmarké par recherche web, et **score de confiance hybride (complétude × validation LLM)**.

## Changements v4 → v5

- **Échelles 1-5 → 1-3** sur tous les critères qualitatifs. Granularité plus humaine en atelier.
- **Format cellule : chiffre + label dans la même cellule** (ex: `3 - récurrent standardisé`). Le LLM extrait le préfixe numérique pour calculer.
- **Run Annuel benchmarké** : remplace l'heuristique `Build × 0.15` par `Volume Qté × Coût Unitaire Agent`, où le Coût Unitaire est récupéré par recherche web LLM en fin de workflow.
- **8 colonnes supprimées** : Nb Intégrations Identifiées, Volume Annuel (unité), Personnes Impactées, Notes Architecte (fusionnée dans Notes), Nb Colonnes Critiques, Qualité Source, Cohérence Check, Confiance Globale (réintroduite v2.1 sous forme calculée hybride).
- **3 nouvelles colonnes** : Type Agent, Coût Unitaire Agent ($/run), **Validation LLM (v2.1)**.
- **Volume Qté = total annuel** (avant pouvait être per-person × Personnes Impactées). Simplification : un seul nombre.
- **Confiance hybride v2.1** : Verdict Confiance lit maintenant `Confiance Données × Validation_Multiplier`. La complétude seule saturait à 100% quand le LLM remplissait tous les slots avec des estimations contextuelles. La col Validation LLM (1-3) capture la fraction validée vs estimée.

## Principe directeur v5

**Séparation des rôles** :
- **Analyste fonctionnel** : remplit cols 1-23 et 32-37. Cartographie + chiffrage business avec le client.
- **Architecte dev** : remplit cols 24-25. Output minimal : T-shirt + montant.
- **LLM (toi)** : remplit cols 14, 20, 21, 23, 25, 26, 27, 28-31, 33-36 (calculs et benchmarks).

## Conventions transversales

- **(input)** : à remplir manuellement par analyste
- **(LLM)** : rempli par le LLM (calcul ou benchmark)
- **(ARCHI)** : à remplir par l'architecte
- **Format `<n> - <label>`** pour les colonnes 1-3 (cols 6-13 et 32) : le chiffre est utilisé pour le calcul, le label pour la lisibilité humaine
- **N/A** : la question ne s'applique pas, compte comme rempli pour la confiance
- **Vide** : pas encore évalué, fait baisser la confiance

---

## Bloc 1 : Identité (cols 1-5)

### 1. Use Case
**Rôle** : Nom court du processus, formulé comme verbe d'action. Source : BABOK 10.10 Business Case.

### 2. Pain Point
**Rôle** : Description du problème actuel, citation directe du client si possible. Source : BABOK 10.10, atelier SVA/cartographie.

### 3. Source Atelier
**Rôle** : D'où vient l'info (couleur post-its, transcript, document). Traçabilité. Source : convention interne.

### 4. Sponsor / Owner
**Rôle** : Personne nommée qui porte le processus. "À identifier" = drapeau rouge, compte comme NON rempli. Source : BABOK 10.18, Prosci ADKAR.

### 5. Département
**Rôle** : Unité organisationnelle. Sert au regroupement portefeuille. Source : gouvernance organisationnelle.

---

## Bloc 2 : Suitability Score qualitatif (cols 6-14)

Filtre amont rapide. Chaque critère noté 1-3 dans le format `<n> - <label>`. Score composite sur 100.

### 6. Volume Standardisation
**Valeurs** :
- `1 - sporadique/ad-hoc`
- `2 - régulier mais variable`
- `3 - récurrent standardisé`

**Rôle** : Le processus est-il assez fréquent et standardisé pour valoir l'automatisation ? Source : UiPath Process Assessment Framework.

### 7. Règles Claires
**Valeurs** :
- `1 - jugement humain dominant`
- `2 - règles partielles`
- `3 - déterministe clair`

**Rôle** : Les règles métier sont-elles documentées et déterministes ? Source : Deloitte Automation Suitability.

### 8. Disponibilité Données
**Valeurs** :
- `1 - silos/absentes`
- `2 - accessibles avec effort`
- `3 - API/structurées prêtes`

**Rôle** : Les données nécessaires existent-elles et sont-elles accessibles ? Source : Gartner Data Readiness Assessment.

### 9. Stabilité Processus
**Valeurs** :
- `1 - changements fréquents`
- `2 - stable mais évolue`
- `3 - très stable`

**Rôle** : Le processus change-t-il souvent ? Risque d'obsolescence de l'automatisation. Source : UiPath stabilité.

### 10. Faible Taux Exceptions
**Valeurs** :
- `1 - >20% exceptions`
- `2 - 5-20% exceptions`
- `3 - <5% exceptions`

**Rôle** : Proportion de cas hors-norme à coder spécialement. Source : Lean Six Sigma - variabilité.

### 11. Impact Erreurs Actuelles
**Valeurs** :
- `1 - faible (cosmétique)`
- `2 - moyen (rework)`
- `3 - élevé (régulatoire/financier)`

**Rôle** : Conséquence des erreurs humaines actuelles. Source : Lean - Cost of Poor Quality (Juran).

### 12. Alignement Stratégique
**Valeurs** :
- `1 - hors-roadmap`
- `2 - adjacent à un OKR`
- `3 - OKR explicite client`

**Rôle** : Le use case s'aligne-t-il avec les OKR ou la vision du client ? Source : SAFe Strategic Themes, framework OKR.

### 13. Sponsor Engagé
**Valeurs** :
- `1 - passif`
- `2 - intéressé`
- `3 - actif/présent en atelier`

**Rôle** : Niveau d'engagement du sponsor identifié à col 4. Source : Prosci ADKAR - dimension Sponsorship.

### 14. Suitability Score (0-100)
**Rôle** : Score composite des 8 critères ci-dessus. Permet de filtrer rapidement avant de chiffrer.

**Formule (LLM)** : `MOYENNE(parseNumber(cols 6-13)) × 33.33`

Seuil pratique : < 50 = passer, > 70 = chiffrer. Source : UiPath Suitability composite.

---

## Bloc 3 : Quantitatif économique (cols 15-23)

### 15. Volume Qté
**Rôle** : Nombre **total** d'occurrences du processus par an (tous exécutants confondus). Source : ingénierie industrielle (Taylor, Gilbreth).

### 16. Temps Actuel (h/occ)
**Rôle** : Heures consommées par occurrence aujourd'hui. Source : Time and Motion Study (MTM, MOST).

### 17. Temps Cible (h/occ)
**Rôle** : Heures consommées par occurrence après automatisation. Souvent 10-30% du temps actuel pour les processus AI-assistés. Source : pratique RPA / Hyperautomation.

### 18. Coût Horaire ($)
**Rôle** : Coût horaire chargé (salaire + charges + overhead) moyen. Source : Fully Loaded Cost (Horngren, Kaplan HBS).

### 19. Pertes Évitées Directes ($)
**Rôle** : Montant annuel de pertes évitées (non-conformité, scrap, fraude). Distinct du temps économisé. Source : Lean - Cost of Poor Quality (Juran).

### 20. Gain Heures Annuel
**Rôle** : Heures économisées par an.

**Formule (LLM)** : `Volume Qté × (Temps Actuel − Temps Cible)`

Si Temps Actuel = 0 (cas pertes évitées directes uniquement), Gain Heures = 0.

### 21. Bénéfice Brut Annuel ($)
**Rôle** : Valeur monétaire théorique du gain.

**Formule (LLM)** :
- Si Temps Actuel = 0 : `Pertes Évitées Directes`
- Sinon : `Gain Heures × Coût Horaire + Pertes Évitées (si applicable)`

Source : Productivité × Managerial Accounting.

### 22. Taux Réalisation
**Valeurs** : décimal 0-1, défaut **0.6**. Reflète le change management overhead (Prosci/McKinsey).

**Rôle** : Haircut sur le bénéfice brut. On ne récupère jamais 100% du temps gagné.

### 23. Bénéfice Net Annuel ($)
**Rôle** : Bénéfice réaliste après haircut. Chiffre clé utilisé pour Payback, ROI, Score Priorité.

**Formule (LLM)** : `Bénéfice Brut × Taux Réalisation`

---

## Bloc 4 : Output architecte (cols 24-25)

### 24. Taille Build (T-shirt) [ARCHI]
**Valeurs** : XS / S / M / L / XL.

**Rôle** : Estimation d'effort dev par l'architecte après son calcul mental (intégrations, complexité, données). Output principal. Source : Agile Estimation (Cohn 2005), Scrum/SAFe.

### 25. Build Base ($) [ARCHI]
**Rôle** : Montant en dollars. Par défaut lookup barème agence. Override possible par l'architecte.

**Barème agence MIA** : XS=10k$, S=25k$, M=50k$, L=90k$, XL=150k$.

---

## Bloc 5 : Type d'agent et Run cost benchmarké (cols 26-28)

### 26. Type Agent (NEW v5)
**Valeurs typiques** : `email auto`, `dashboard genAI`, `RAG simple`, `RAG complexe`, `OCR`, `RPA déterministe`, `agent conversationnel`, `document processing`, `autre`.

**Rôle** : Catégorisation du type d'agent à construire. Sert de clé pour le lookup benchmark à la col 27. Source : taxonomie pratique agents AI 2025-2026.

### 27. Coût Unitaire Agent ($/run) (NEW v5)
**Rôle** : Coût d'une exécution unitaire de l'agent. Rempli par le LLM via recherche web ciblée en avant-dernière étape du workflow (voir SKILL.md).

**Méthode** : médiane des benchmarks trouvés sur sources fiables (pricing officiel OpenAI/Anthropic, rapports a16z/Menlo/Sequoia, blogs ingénierie produit récents <12 mois). Source citée dans col 37 Notes avec la date de consultation.

**Fallback** si aucun benchmark fiable trouvé : `(Build × 0.15) / Volume Qté` (heuristique TBM ancienne), flag `BENCHMARK_FALLBACK` dans Notes.

### 28. Run Annuel ($)
**Rôle** : Coût d'opération annuelle (inference LLM, infra, monitoring). Remplace l'ancienne heuristique 15%-du-build par un calcul ancré dans la réalité économique des agents AI.

**Formule (LLM)** : `Volume Qté × Coût Unitaire Agent`

---

## Bloc 6 : Coûts d'opération et ROI (cols 29-31)

### 29. Coût An 1 Total ($)
**Formule (LLM)** : `Build Base + Run Annuel`. Source : comptabilité analytique.

### 30. Payback (mois)
**Formule (LLM)** :
- Si `(Bénéfice Net − Run Annuel) ≤ 0` : `999` (jamais rentable)
- Sinon : `Build Base × 12 / (Bénéfice Net − Run Annuel)`

Source : Finance corporative - Payback Period (Fisher 1930, Brealey-Myers).

### 31. ROI An 1 (%)
**Formule (LLM)** : `(Bénéfice Net − Coût An 1) / Coût An 1 × 100`

Souvent négatif An 1 même pour bons projets. Source : ROI finance corporative.

---

## Bloc 7 : Risque et Score Priorité (cols 32-33)

### 32. Risque
**Valeurs** :
- `1 - faible` (technique connue, sponsor fort, données simples)
- `2 - moyen` (défaut si non spécifié)
- `3 - élevé` (technique inconnue, pas de sponsor, données problématiques)

**Rôle** : Risque global d'échec du projet (adoption, technique, organisationnel). Source : BABOK 10.38 Risk Analysis.

### 33. Score Priorité
**Formule (LLM)** : `Bénéfice Net / (Build Base × parseNumber(Risque))`

Plus élevé = meilleur. Source : WSJF simplifié (SAFe).

---

## Bloc 8 : Méta-confiance hybride (cols 34-36)

Mécanisme à deux étages : (1) **complétude** des slots (combien de cellules critiques sont remplies), et (2) **validation LLM** (la fraction de ces valeurs qui vient de la source vs de l'estimation contextuelle). Le verdict final combine les deux.

### 34. Confiance Données (%)
**Rôle** : Pourcentage de complétude des 20 colonnes critiques (voir liste plus bas).

**Formule (LLM)** : `Nb_Colonnes_Critiques_Remplies / 20 × 100`

Source : Data Completeness Ratio (DAMA-DMBOK).

### 35. Validation LLM (NEW v2.1)
**Valeurs** :
- `1 - estimation contextuelle` : LLM a inféré la majorité des chiffres depuis le contexte général (sponsor "À identifier" ou inféré, volume/temps/pertes tous estimés, pain point reformulé)
- `2 - mixte source + estimation` : certains chiffres explicites dans le transcript + d'autres inférés (sponsor nommé mais engagement déduit, 1-2 chiffres clés cités, autres extrapolés)
- `3 - source primaire validée` : sponsor explicitement nommé ET présent atelier, Volume Qté + Temps OU Pertes Évitées explicitement cités dans le transcript, Pain Point cité textuellement

**Rôle** : Capture la **fraction validée** des données vs estimée par le LLM. Sert de **multiplicateur de fiabilité** sur la complétude (col 34). Sans ce signal, la complétude seule peut faussement saturer à 100% quand le LLM remplit tous les slots avec des estimations contextuelles.

**Méthode** : le LLM s'auto-évalue ligne par ligne, en se basant sur la traçabilité du transcript et du matériel d'atelier. Documenter dans Notes les sources clés ayant justifié le niveau choisi.

Source : Validation Source vs Estimation - pratique data triangulation appliquée à l'extraction LLM.

### 36. Verdict Confiance
**Formule** : `Confiance Globale = Confiance Données × Validation_Multiplier`

| Validation LLM | Multiplicateur |
|---|---|
| `3 - source primaire validée` | 1.0 |
| `2 - mixte source + estimation` | 0.75 |
| `1 - estimation contextuelle` | 0.5 |

**Seuils** sur Confiance Globale :
| Confiance Globale | Verdict |
|---|---|
| ≥ 80% | Fiable |
| 60-79% | À valider 1-2 points |
| 40-59% | Hypothèses fortes |
| < 40% | Atelier de validation requis |

**Exemple** : Confiance Données 100% (tous slots remplis) + Validation `2 - mixte` → Confiance Globale 75% → Verdict `À valider 1-2 points`. Plus honnête qu'un `Fiable` qui ignore la part estimation.

---

## Bloc 9 : Décision (cols 37-38)

### 37. Verdict ROI
**Seuils** sur Payback (col 30) :
| Payback (mois) | Verdict |
|---|---|
| < 6 | Quick win |
| 6-12 | Go |
| 12-18 | À challenger |
| 18-24 | Stratégique seulement |
| > 24 (ou 999) | Pass |

Source : Decision Rules - pratique cabinet conseil et benchmarks RPA.

### 38. Notes
**Rôle** : Commentaires libres analyste + notes architecte + **source/date du benchmark Coût Unitaire Agent** + **justification du niveau Validation LLM** (citations clés du transcript ou nature des estimations). Hypothèses à valider, drapeaux rouges, prochaines étapes.

Format pour le benchmark : `Benchmark Run: $X.XX/<type agent> (<source>, consulté YYYY-MM-DD)`.

Source : BABOK 10.10 - section Assumptions and Risks.

---

## Liste des 20 colonnes critiques pour la confiance

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

Build Base et Run Annuel ne sont pas critiques (auto-calculés depuis T-shirt et benchmark).

---

## Workflow d'utilisation

1. **Atelier client (analyste)** — Cols 1-13 : Identité + 8 critères Suitability dans le format `<n> - <label>`.
2. **Post-atelier (analyste)** — Si Suitability < 50, marquer en Pass. Sinon, remplir cols 15-22 (quantitatif économique).
3. **Architecte** — Lire la fiche, livrer cols 24-25 (T-shirt, Build Base optionnel).
4. **Analyste** — Catégoriser col 26 (Type Agent) et remplir col 32 (Risque).
5. **LLM (skill)** — Benchmark web pour col 27 (Coût Unitaire Agent), puis recalcule cols 14, 20-21, 23, 28-31, 33-34.
6. **LLM auto-évaluation (v2.1)** — Remplir col 35 (Validation LLM) en pesant la fraction des chiffres validés depuis transcript vs estimés depuis contexte. Justifier dans Notes. Verdict Confiance (col 36) en découle automatiquement.
7. **Décision** — Trier par Score Priorité, filtrer par Verdict ROI ∈ {Quick win, Go, À challenger}, valider Verdict Confiance ≥ "À valider 1-2 points".

---

## Changelog v4 → v5

**Échelles** : 1-5 → 1-3 sur les 9 critères qualitatifs (cols 6-13 et 32). Format cellule : `<n> - <label>` au lieu de chiffre seul.

**Supprimées** (8 cols) :
- Nb Intégrations Identifiées (ancienne col 15) — info imprécise en atelier, non utilisée dans formules
- Volume Annuel (unité) (ancienne col 16) — convention "annuel" implicite
- Personnes Impactées (ancienne col 20) — sociologique, Volume Qté capte déjà le total
- Notes Architecte (ancienne col 29) — fusionnée dans Notes
- Nb Colonnes Critiques Remplies (ancienne col 36) — redondant avec Confiance Données
- Qualité Source (ancienne col 37) — méta imprécise, remplacée v2.1 par Validation LLM
- Cohérence Check (ancienne col 38) — méta imprécise
- Confiance Globale (ancienne col 40) — composite supprimé v2.0, réintroduit v2.1 comme calculé via Validation LLM × Confiance Données

**Ajoutées** (3 cols) :
- Type Agent (col 26) — catégorisation pour benchmark
- Coût Unitaire Agent ($/run) (col 27) — benchmarké par recherche web LLM
- Validation LLM (col 35, v2.1) — fraction source vs estimation, multiplicateur de confiance

**Formules modifiées** :
- Suitability : `MOYENNE × 20` (8 critères 1-5) → `MOYENNE × 33.33` (8 critères 1-3)
- Gain Heures : `Volume × ΔT × Personnes` → `Volume × ΔT` (Volume = total)
- Run Annuel : `Build × 0.15` → `Volume Qté × Coût Unitaire Agent`
- Confiance Données : `Nb_Rempli / 20 × 100` (dénominateur inchangé, mais composition des critiques renouvelée)
- Verdict Confiance (v2.1) : `seuils sur Confiance Données × Validation_Multiplier` (hybride complétude + qualité estimation)
