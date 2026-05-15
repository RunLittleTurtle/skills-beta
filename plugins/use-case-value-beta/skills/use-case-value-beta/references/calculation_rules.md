# Règles de calcul use-case-value-beta v1.3

Ce document détaille les formules, standards de méthode, défauts conservateurs pour signaux qualitatifs, formule de Score v1.3, et algorithme Verdict du CSV 27 colonnes.

## Règle d'or v1.3 sur les estimations

**Cellules Impact $ (colonnes 13 à 18) acceptent** :

1. **Valeur citée verbatim** par sponsor, participant atelier ou table source.

2. **Formule simple sur chiffres durs cités**, avec standards de méthode comme paramètres de conversion :
   - Coût horaire fully-loaded : 80 $/h
   - FTE chargé fully-loaded : 100 000 $/an

3. **Inférence obvious sur signal cité explicitement** + standard de méthode documenté :
   - "1 FTE saturé", "à temps plein", "dédié" → 100 000 $/an
   - "X h/sem" cité → X × 50 × 80 $/h

4. **Inférence avec défaut conservateur si DEUX anchors verbaux concrets** (NOUVEAU v1.3) :
   - Un signal d'heures même vague (`quelques heures par semaine`, `une partie significative du temps`, `pas mal de temps`) ET un nombre de personnes cité
   - Conversion via défaut conservateur documenté (voir tableau ci-dessous)
   - Documenter explicitement dans Notes : `Hypothèses : signal "quelques heures" converti en 2 h/sem par personne (défaut conservateur v1.3), 20 personnes citées par Steven en atelier.`

**Cellules Impact $ refusent toujours (gonflage interdit)** :
- Signal d'heures vague SANS nombre de personnes cité → 0
- Nombre de personnes sans aucun signal d'heures → 0
- Multiplicateurs inventés sur des chiffres durs (% captable, probabilité, valeur moyenne contrat sans citation) → 0
- Volumes inventés sans signal dans les inputs → 0

**Principe v1.3** : si on a au moins **deux anchors verbaux concrets** (signal d'heures même vague + nombre de personnes), le LLM peut convertir avec un défaut conservateur documenté. Si un seul anchor manque ou un paramètre doit être inventé, cellule = 0 et question en Notes.

## Défauts conservateurs pour signaux qualitatifs (v1.3)

Quand un signal d'heures vague est cité ET un nombre de personnes est cité :

| Signal verbal | Défaut conservateur par personne | Note Hypothèses |
|---------------|----------------------------------|-----------------|
| "quelques heures par semaine" | 2 h/sem | Range plausible 2-5 h/sem, défaut basse pour anti-gonflage |
| "pas mal de temps par semaine" | 3 h/sem | Range plausible 3-6 h/sem |
| "du temps notable", "ça consomme" | 2 h/sem | Conservateur |
| "une partie significative de la journée" | 2 h/jour soit 10 h/sem | Range plausible 2-4 h/jour |
| "la moitié de la journée" | 4 h/jour soit 20 h/sem | Citation plus précise, valeur médiane |
| "tout le monde y passe du temps" | 2 h/sem | Conservateur si pas de précision |

**Tous ces défauts sont à la borne basse du range plausible** pour préserver l'anti-gonflage. La note documente toujours `valeur conservative à valider en atelier si arbitrage critique`.

Si le sponsor cite un range explicite ("entre 5 et 10 heures par semaine"), utiliser la borne basse du range cité.

## Standards de méthode — paramètres de conversion conditionnels

Important : ces standards s'appliquent uniquement quand un signal verbal correspondant est cité. Ce ne sont pas des valeurs appliquées automatiquement.

### Coût horaire fully-loaded standard de conversion

`80 $/h` — profil mixte employé québécois, charges sociales et overhead inclus.

**Quand l'appliquer** : si un sponsor cite des heures explicites OU un signal d'heures vague + nombre de personnes (v1.3).

**Comment l'appliquer** :
```
Impact $ Temps Perdu = heures par semaine × 50 semaines × 80 $/h
```

Si le sponsor cite un taux différent, utiliser ce taux et documenter dans Notes.

### FTE chargé fully-loaded standard de conversion

`100 000 $/an` — profil mixte employé québécois, productivité annuelle effective.

**Quand l'appliquer** : uniquement si un sponsor cite explicitement une saturation FTE ("1 personne à temps plein", "1 FTE saturé", "dédié").

**Comment l'appliquer** :
```
Impact $ Main d'Œuvre Saturation = nombre de FTE saturés × 100 000 $
```

**Citations valides** : "Sophie est à temps plein", "1 personne dédiée", "il faut quelqu'un de full-time".

**Citations INVALIDES** (cellule = 0, va en Notes) : "ça mobilise du monde", "c'est lourd", "ça pèse" — pas de quantité FTE nommée.

## Formules pour les colonnes calculées

### Col 19 — Impact Total Documenté ($/an)

```
Impact Total Documenté = SUM(col13 ; col14 ; col15 ; col16 ; col17 ; col18)
```

Reflète uniquement les chiffres durs et inférences admissibles selon la règle d'or v1.3.

### Col 27 — Impact Potentiel Estimé ($/an)

Le LLM somme les estimations indicatives présentes dans Notes (col 25) sous "À chiffrer en atelier". Chaque "À titre indicatif, si [hypothèse], cela représenterait environ [valeur]" contribue sa valeur indicative.

N'entre PAS dans Impact Total Documenté (col 19). Entre dans Score Priorité Impact avec discount 0.3.

### Col 23 — Score Priorité Impact (formule v1.3)

```
Score brut = (Impact Total Documenté + 0.3 × Impact Potentiel Estimé)
           × Complétude Chiffrage
           × Urgence Stratégique
           × Personnes_factor
           × Transversalité_factor

Personnes_factor (paliers stepwise) :
  Personnes Affectées < 5         → 1.0
  5 ≤ Personnes Affectées < 15    → 1.2
  15 ≤ Personnes Affectées < 50   → 1.5
  Personnes Affectées ≥ 50        → 1.8

Transversalité_factor :
  Transversalité = 1 (un département)        → 1.0
  Transversalité = 2 (inter-départemental)   → 1.3
  Transversalité = 3 (toute l'organisation)  → 1.6

Score Priorité Impact (normalisé) =
  Score brut / MAX(Score brut sur toutes les lignes du CSV)
```

**Changement v1.2 → v1.3** :
- Personnes Affectées passe de log faible `(1 + 0.2 × log(N))` à paliers stepwise (1.0 → 1.8)
- Transversalité ajoutée comme multiplicateur explicite (1.0 → 1.6)

**Effet** : un use case transverse touchant 20 personnes (1.5 × 1.6 = 2.4) est correctement amplifié vs un use case ponctuel touchant 3 personnes (1.0 × 1.0 = 1.0), facteur 2.4×. La formule v1.2 ne donnait que 1.26× d'écart.

### Col 24 — Verdict Impact (algorithme déterministe, inchangé v1.2)

Exécuter l'algorithme strictement séquentiellement, premier match l'emporte :

```
1. SI Impact Total Documenté > 200 000 $
       ET Urgence Stratégique ≥ 2
       ET Complétude Chiffrage ≥ 2
   ALORS Verdict = "Critique"

2. SINON SI Impact Total Documenté > 100 000 $
            ET Complétude Chiffrage ≥ 2
   ALORS Verdict = "Forte"

3. SINON SI Impact Total Documenté > 100 000 $
            ET Complétude Chiffrage = 1
            ET au moins une cellule des col 13 à 18 > 0
   ALORS Verdict = "Forte (chiffrage à compléter)"

4. SINON SI Impact Total Documenté ≤ 100 000 $
            ET Impact Potentiel Estimé > 100 000 $
            ET Urgence Stratégique ≥ 2
            ET sponsor actif détecté dans Notes
   ALORS Verdict = "À chiffrer prioritairement"

5. SINON SI Impact Total Documenté ≤ 100 000 $
            ET Impact Potentiel Estimé > 0
   ALORS Verdict = "À chiffrer secondairement"

6. SINON SI Urgence Stratégique = 3
            ET Impact Total Documenté ≤ 100 000 $
   ALORS Verdict = "Stratégique long-terme"

7. SINON
   ALORS Verdict = "Faible impact"
```

**Détection "sponsor actif"** : sponsor cité avec verbe d'action ("je pousse", "j'ai besoin", "il faut", "ça m'excite", "wow"), OKR explicite, ou date butoir.

**Anti-prudence-excessive** : si l'algorithme demande Verdict = "À chiffrer prioritairement", le LLM doit l'écrire tel quel. Ne pas downgrader en "secondairement" par prudence.

Note : avec la règle d'or v1.3 plus permissive et le Score amplifié, beaucoup de use cases qui passaient en "À chiffrer prioritairement" en v1.2 vont désormais avoir Impact Total Documenté > 100 000 $ et passer en "Forte" ou "Critique". La synthèse Top 10 affiche tous les verdicts avec transparence.

## Lignes FORMULES et SOURCE (rows 2 et 3 du CSV)

Le gabarit `csv_template.csv` contient :

- **Row 1** : header avec 27 noms de colonnes
- **Row 2 (FORMULES)** : formules ou "input direct" / "manuel selon rubrique" / "LLM dérive de Notes"
- **Row 3 (SOURCE)** : référence méthodologique
- **Row 4+** : data

## Col 26 — Dépend de (texte libre, n'entre PAS dans le calcul)

Inchangé depuis v1.1. Signal qualitatif pour la synthèse et la planification.

## Vérifications de cohérence avant `present_files`

- Si col 19 > 0 : au moins une cellule des col 13 à 18 doit être > 0 ET tracée à une citation source ou une formule simple ou une inférence v1.3 avec défaut conservateur documenté dans Notes.
- Si col 13 à 18 = 0 : Notes doit contenir une question "À chiffrer" correspondante avec estimation indicative.
- Col 27 = somme stricte des estimations indicatives chiffrées dans Notes du même use case.
- Si Verdict = "Critique" : col 19 > 200 000 $ ET cellules > 0 toutes tracées dans Notes.
- Si Verdict = "À chiffrer prioritairement" : col 27 > 100 000 $ ET sponsor actif documenté.
- Si col 27 > 100 000 $ ET sponsor actif ET Urgence ≥ 2 mais Verdict = "À chiffrer secondairement" : c'est une erreur de prudence excessive, repasser à "prioritairement".
- Aucune cellule Impact $ ne contient une valeur sans anchor verbal cité + standard de méthode documenté OU sans paire d'anchors (heures vague + personnes) avec défaut conservateur documenté.
- **Step 2bis Root Cause appliqué** : chaque ligne est une root cause actionnable.
- **Col 26 Dépend de** : si dépendance inter-use-cases, listée avec nom exact d'un autre Use Case.
- **Format CSV RFC 4180** : Notes entre guillemets doubles, sauts de ligne entre sections (pas de pipes), guillemets internes doublés.
