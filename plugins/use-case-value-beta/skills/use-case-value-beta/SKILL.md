---
name: use-case-value-beta
description: Priorise et chiffre la VALEUR business des use cases d'AI/automatisation à partir de tout matériel de découverte de processus (transcripts d'ateliers, post-its, inventaires CSV de 500+ lignes, notes de réunion). Six sources d'Impact $ explicites + pain points + personnes affectées + transversalité + colonne Dépend de + colonne Impact Potentiel Estimé. Step 2bis Root Cause Analysis (5 Whys + Ishikawa) avant chiffrage. Règle d'or v1.3 adoucie avec paire d'anchors (signal heures vague + N personnes cité = inférable via défauts conservateurs documentés à la borne basse du range plausible) en plus des chiffres durs cités, formules simples, et inférences obvious sur citations FTE explicites. Refusé : multiplicateurs sortis du chapeau (% captable, probabilité, valeur moyenne sans citation) ou inférence avec un seul anchor manquant. Score Priorité Impact v1.3 amplifié pour les use cases transverses : Personnes_factor en paliers stepwise (1.0 / 1.2 / 1.5 / 1.8) ET Transversalité_factor (1.0 / 1.3 / 1.6) explicites dans la formule. Algorithme Verdict déterministe numéroté. Synthèse Top 10 use cases × 3 lignes par entrée (classement décisif sans section À chiffrer en atelier séparée). Aucune dimension d'effort technique : focus pur VALEUR, effort traité par un futur skill séparé. Sortie : dossier projet par run contenant CSV 27 colonnes et synthèse one-pager prose narrative française Québec. Triggers : prioriser, valeur use case, chiffrer impact, note de cadrage, atelier découverte, post-its, transcript, root cause, dépendances use cases, 5 whys, ishikawa, top 10.
---

# Use Case Value v1.2 — Priorisation par impact chiffré

Ce skill transforme du matériel d'atelier de découverte en une **note de cadrage business** qui répond à une seule question : **"Quel use case mérite d'être chiffré et scopé en premier sur la base de l'impact business documenté ou clairement chiffrable ?"**

## Ce que ce skill produit

Un dossier `/mnt/user-data/outputs/priorisation_impact_<projet>_<YYYY-MM-DD>/` contenant :

1. **`priorisation_use_cases.csv`** : 27 colonnes, ligne FORMULES (row 2), ligne SOURCE (row 3), puis une ligne par use case.
2. **`synthese_priorisation.md`** : synthèse exécutive one-pager en prose narrative française Québec.

## Ce que ce skill NE produit PAS

- Aucune estimation de Build, Run cost, Payback, ROI An 1, T-shirt. L'effort technique est hors scope.
- Aucune valeur inventée dans les cellules Impact $. Voir règle d'or adoucie ci-dessous.
- Aucun bundling de use cases dans la synthèse (un Top = une root cause actionnable).

## Toujours commencer par lire le framework

Avant de traiter tout input, lire dans cet ordre :

1. `references/framework.md` : 27 colonnes, rubriques 1-3, sources méthodologiques.
2. `references/calculation_rules.md` : standards de méthode (80 $/h, 100 000 $/FTE), formule Score v1.2, algorithme Verdict déterministe.
3. `references/exemples_chiffrage_impact.md` : frontière chiffre dur / inférence obvious / hypothèse, exemples colonne par colonne.

Sans ce contexte, tu vas remplir le CSV de manière incorrecte (notamment violer la règle d'or ou lister des symptômes au lieu de root causes).

## Règle d'or v1.3 adoucie

**Cellules Impact $ (colonnes 13 à 18) acceptent** :

1. **Valeur citée verbatim** par un sponsor ou participant atelier
2. **Formule simple sur chiffres durs cités**, avec standards de méthode comme paramètres de conversion :
   - 80 $/h (coût horaire fully-loaded standard) appliqué à des heures par semaine citées
   - 100 000 $/an (FTE chargé fully-loaded standard) appliqué à une saturation FTE citée
3. **Inférence obvious sur signal cité explicitement** :
   - "Sophie est à temps plein sur les soumissions" → col Main d'Œuvre = 100 000 $
   - "20 h/sem de matching" → col Temps Perdu = 80 000 $
4. **NOUVEAU v1.3 — Inférence avec défaut conservateur si DEUX anchors verbaux concrets** :
   - Signal d'heures vague (`quelques heures par semaine`, `pas mal de temps`, `tout le monde y passe du temps`) **ET** nombre de personnes cité
   - Conversion avec défaut conservateur (voir tableau dans `calculation_rules.md` et `exemples_chiffrage_impact.md`)
   - Documenter dans Notes : `Hypothèses : 'quelques heures' converti en 2 h/sem par personne (défaut conservateur v1.3), 20 personnes citées.`

**Défauts conservateurs v1.3** (à la borne basse pour anti-gonflage) :
- "quelques heures par semaine" → 2 h/sem
- "pas mal de temps" → 3 h/sem
- "tout le monde y passe du temps" → 2 h/sem
- "une partie significative de la journée" → 2 h/jour
- "la moitié de la journée" → 4 h/jour

**Cellules Impact $ refusent toujours (gonflage interdit)** :
- **Signal d'heures vague SANS nombre de personnes** ou nombre de personnes sans signal d'heures (un seul anchor) → 0
- Multiplicateurs sortis du chapeau : % captable, probabilité, valeur moyenne contrat sans citation → 0
- Volumes inventés sans signal dans les inputs → 0

**Test de décision v1.3** : "Pour cette valeur de cellule, ai-je (a) une citation verbatim sponsor OU (b) une formule simple sur chiffres durs OU (c) une inférence obvious sur signal cité + standard méthode OU (d) une paire d'anchors (heures vague + nombre personnes) + défaut conservateur documenté ? Si oui → cellule. Sinon → cellule = 0, hypothèse en Notes."

**Une cellule à 0 ne signifie pas "pas d'impact"**. Elle signifie "anchors verbaux insuffisants". Toute cellule à 0 doit avoir une question structurée dans Notes (col 25) qui alimentera col 27 Impact Potentiel Estimé.

## Inputs acceptés

- Photos de post-its / whiteboards
- Transcripts (toute langue, sortie en français Québec)
- Tableaux d'inventaire de processus (jusqu'à 500+ lignes)
- Notes libres
- Mix de ces formats

Si aucun input fourni : demander à l'utilisateur. Ne jamais inventer de use cases.

## Workflow

### Step 0 — Détecter le projet et créer le dossier de sortie

Inférer le nom du projet (nom de fichier source, entité client en boucle dans transcript, ou fallback date-heure).

Créer `/mnt/user-data/outputs/priorisation_impact_<projet>_<YYYY-MM-DD>/`.

### Step 1 — Inventorier les candidats use cases bruts

Lister tous les concepts centrés (process, workflow, pain point automatisable) qui apparaissent dans les inputs. Ne pas filtrer à ce stade.

### Step 2bis — Analyse Root Cause (mini 5 Whys + Ishikawa léger)

Avant de chiffrer, identifier pour chaque candidat brut s'il est une **root cause** (cause profonde actionnable) ou un **symptôme** (effet d'un autre process).

Pour chaque candidat, appliquer un mini 5 Whys :
1. Quel pain point apparent ?
2. Pourquoi ? Quelle cause sous-jacente ?
3. Pourquoi cette cause persiste-t-elle ?
4. Quel niveau d'automatisation l'adresserait directement ?

Le **use case à enregistrer** est celui actionnable par automatisation. Les symptômes consolidés deviennent Pain Points #1, #2, #3 sur la ligne de la root cause.

**Représentation CSV** :
- Plusieurs symptômes / une automatisation possible → une seule ligne pour la root cause, symptômes en Pain Points
- Symptômes distincts mais dépendance partagée → plusieurs lignes avec col 26 Dépend de pointant vers la fondation
- Le concept partagé est lui-même un use case (Data Lake, intégration) → ligne pour la fondation + lignes consommatrices avec col 26 pointant vers la fondation

Documenter dans Notes (col 25) : `Root cause : cette ligne consolide les symptômes X, Y, Z identifiés en Step 2bis pour éviter de double-compter.`

### Step 2 — Remplir colonnes 1 à 12

Pour chaque root cause retenue : Use Case, Description, Sponsor, Département, Type Agent, Volume, Personnes Affectées, Transversalité, Pain Points #1-3 (les symptômes consolidés si applicable).

Ne jamais inventer. Cellule sans source claire = vide.

### Step 3 — Chiffrer l'Impact $ (règle d'or adoucie v1.2)

Pour chaque colonne Impact $ (13 à 18) :

1. **Parcourir les inputs** pour chiffres durs cités ou signaux convertibles via standards (heures citées, FTE saturé cité, valeur explicite citée).

2. **Si chiffre dur direct OU formule simple sur citation OU inférence obvious sur signal cité** :
   - Inscrire la valeur dans la cellule
   - Documenter dans Notes la citation source + la formule de conversion utilisée
   - Exemple : `Sources : Sophie a dit "20 h/sem". Hypothèses : 20 × 50 × 80 $/h = 80 000 $.`
   - Exemple FTE : `Sources : Maxime a dit "Sophie est à temps plein". Hypothèses : 1 FTE × 100 000 $ standard = 100 000 $.`

3. **Si paramètre de conversion devrait être inventé** (% captable, probabilité, valeur unitaire sans citation) :
   - Laisser cellule à **0**
   - Ajouter dans Notes section "À chiffrer en atelier" :
     ```
     À chiffrer : [question]. À titre indicatif si [hypothèse réaliste], cela
     représenterait environ [valeur] en Impact $ [colonne].
     ```

4. **Calculer col 27 Impact Potentiel Estimé** à la fin de Step 3 :
   - Somme stricte des estimations indicatives chiffrées présentes dans Notes du même use case
   - Si Notes mentionne `+150 000 $ Opportunités` ET `+50 000 $ Main d'Œuvre` → col 27 = 200 000 $
   - Si aucune estimation indicative chiffrée → col 27 = 0

Consulter `references/exemples_chiffrage_impact.md` pour exemples colonne par colonne avec la règle d'or adoucie.

### Step 4 — Évaluer scores 1-3 (cols 20 à 22)

- Disponibilité Données (col 20) : DAMA-DMBOK Accessibility
- Complétude Chiffrage (col 21) : rubrique élargie v1.2 (cellule dominante chiffrée = niveau 2)
- Urgence Stratégique (col 22) : WSJF Cost of Delay

Voir `framework.md` §Bloc 5 pour rubriques précises.

### Step 5 — Calculer Impact Total, Score, Verdict (cols 19, 23, 24)

- **Col 19 Impact Total Documenté** = SUM(col13:col18)
- **Col 23 Score Priorité Impact** = `(col19 + 0.3 × col27) × Complétude × Urgence × (1 + 0.2 × log(Personnes))`, normalisé sur max
- **Col 24 Verdict Impact** = algorithme déterministe numéroté de `calculation_rules.md`

### Step 5bis — Remplir Dépend de (col 26)

Pour chaque ligne, lister les use cases prérequis identifiés en Step 2bis (texte libre, séparés par virgules, vide si aucune dépendance). Les dépendances non-use-case vont en Notes, pas en col 26.

### Step 6 — Rédiger la synthèse en prose narrative (Top 10 × 3 lignes v1.3)

Suivre `references/synthese_template.md`. Structure v1.3 :

1. **Préambule** : 1 paragraphe court (date, sources, nombre de candidats avant et après Step 2bis, scope impact-only)
2. **Top 10 use cases** : 10 entrées × 3 lignes chacune (titre + sponsor/contexte chiffré + impact/verdict/recommandation). Classement par Score Priorité Impact décroissant.
3. **Dépendances et root causes** : section optionnelle 2-3 paragraphes, incluse seulement si col 26 a des valeurs OU si Step 2bis a consolidé des symptômes
4. **Recommandations méthodologiques** : 1 paragraphe court

**Section "À chiffrer en atelier avant décision" supprimée en v1.3** : les use cases avec verdict "À chiffrer prioritairement" ou "secondairement" apparaissent dans le Top 10 si leur Score les y place, avec leur verdict mentionné en ligne 3 pour transparence. Classement décisif, pas de revalidation systématique.

Style français Québec strict : pas d'em dash, pas de crochets, pas de pipes dans le corps, chiffres en toutes lettres (`220 000 $`), garder uniquement `%` et `$`.

### Step 7 — Présenter le dossier

Utiliser `present_files` : CSV d'abord, puis synthèse.

## Format technique du CSV (RFC 4180)

Le CSV généré DOIT respecter RFC 4180 pour s'ouvrir correctement dans Numbers, Excel, LibreOffice, Google Sheets et autres viewers :

- **Délimiteur** : virgule (`,`)
- **Encodage** : UTF-8
- **Tout champ contenant une virgule, un retour à la ligne, ou un guillemet doit être entouré de guillemets doubles** (`"..."`)
- **Les guillemets internes doivent être doublés** (`""` pour un guillemet littéral à l'intérieur d'un champ quoté)
- **La colonne Notes (col 25)** contient typiquement des virgules et des retours à la ligne, elle doit donc **TOUJOURS être entourée de guillemets doubles**
- **Section dans Notes** : utiliser des sauts de ligne pour séparer les sections (Sources, Hypothèses, Root cause, À chiffrer en atelier, Autres notes), **JAMAIS de pipes `|`** (les pipes confusent les auto-détecteurs de délimiteur des viewers)
- **Sponsor avec virgule** (ex : "Maxime Lavoie, Directeur Commercial") : champ quoté
- **Cellules avec apostrophes ou parenthèses** : généralement OK sans quote, mais quoter si combiné avec virgule

Exemple correct pour une cellule Notes :
```csv
"Sources : Sophie a cité 20 à 25 h/sem (atelier 7 mai 2026). Maxime a dit ""temps plein"".
Hypothèses : 80 $/h appliqué aux 20 h/sem = 80 000 $.
À chiffrer en atelier : combien de contrats perdus par an ?"
```

## Principes critiques

**Step 2bis n'est pas optionnel.** Sans cette analyse, on liste des symptômes et on triple-compte l'impact.

**Inférence obvious autorisée v1.2, gonflage toujours interdit.** Si le sponsor cite "1 FTE saturé", convertir avec le standard 100 000 $/an est légitime. Si le sponsor dit "ça mobilise du monde" sans nommer de quantité, cellule = 0.

**Ne pas surcharger le Verdict par prudence excessive.** L'algorithme est déterministe. Si l'étape 4 demande "À chiffrer prioritairement" parce que col 27 > 100 000 $ et sponsor actif détecté, écrire "À chiffrer prioritairement". Ne pas downgrader en "secondairement".

**Sortie en français Québec.** Aucun em dash.

## Cas particuliers

- **Pas de chiffres durs ni d'inférence obvious possible** : cols 13-18 à 0, col 27 capture le potentiel. Complétude = 1. Synthèse recommande atelier chiffrage.
- **Un seul use case** : CSV 1 ligne, synthèse approfondie.
- **500+ use cases** : Step 2bis va réduire à 50-100 root causes. CSV gère, synthèse garde Top 5.
- **Information conflictuelle** : documenter dans Notes, source la plus crédible l'emporte.

## Quality checks avant `present_files`

- [ ] Dossier `priorisation_impact_<projet>_<YYYY-MM-DD>/` créé
- [ ] CSV a 27 colonnes, lignes FORMULES (row 2) et SOURCE (row 3) préservées
- [ ] **Step 2bis Root Cause appliqué** : chaque ligne est une root cause actionnable
- [ ] Chaque ligne a au moins Use Case et Sponsor remplis
- [ ] **Aucune cellule Impact $ ne contient une valeur sans anchor** : toutes valeurs > 0 sont (a) une citation verbatim sponsor OU (b) une formule simple sur citation OU (c) une inférence obvious sur signal cité + standard de méthode documenté. Tracé dans Notes.
- [ ] Les cellules à 0 ont une question structurée correspondante dans Notes
- [ ] **Col 27 Impact Potentiel Estimé** = somme stricte des estimations chiffrées présentes dans Notes
- [ ] **Col 26 Dépend de** remplie si dépendances inter-use-cases
- [ ] Verdict matche l'algorithme déterministe (cf. calculation_rules.md)
- [ ] **Anti-prudence-excessive** : si col 27 > 100 000 $ ET sponsor actif ET Urgence ≥ 2, Verdict doit être "À chiffrer prioritairement", pas "secondairement"
- [ ] **CSV format RFC 4180** : colonne Notes entre guillemets doubles, sauts de ligne entre sections (PAS de pipes `|`), guillemets internes doublés (`""`)
- [ ] Aucun em dash dans CSV ni synthèse
- [ ] Synthèse a préambule + **Top 10** + Dépendances et root causes (si pertinent) + Recommandations méthodologiques
- [ ] **Aucune section "À chiffrer en atelier" séparée** dans la synthèse (supprimée v1.3)
- [ ] Chaque entrée Top : **3 lignes utiles maximum** (titre + sponsor/contexte + impact/verdict)
- [ ] Synthèse en prose narrative complète
- [ ] Chiffres en toutes lettres
- [ ] Score Priorité Impact appliqué avec **Personnes_factor paliers stepwise** ET **Transversalité_factor** (formule v1.3)
- [ ] Inférences v1.3 sur signaux d'heures vague + N personnes : défaut conservateur documenté dans Notes (`'quelques' converti en 2 h/sem par personne, valeur basse du range plausible`)
