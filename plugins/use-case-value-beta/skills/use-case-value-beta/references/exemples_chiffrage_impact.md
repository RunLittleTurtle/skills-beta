# Exemples de chiffrage Impact $ — règle d'or adoucie v1.2

Ce document aide le LLM à décider, pour chaque colonne Impact $ (cols 13 à 18), ce qui peut entrer dans la cellule (chiffre dur, formule simple sur chiffres durs, ou inférence obvious sur signal cité) versus ce qui doit rester en Notes comme question à poser au sponsor.

## Principe général v1.3 (règle d'or adoucie + inférences avec défauts conservateurs)

**Va dans la cellule** :
- Valeur citée verbatim par un sponsor ou participant atelier
- Valeur tirée d'un tableau source (Excel, CSV, rapport)
- Formule simple sur chiffres durs cités appliquant un standard de méthode (heures × 80 $/h)
- **Inférence obvious sur signal cité explicitement** convertie via standard de méthode documenté :
  - Citation FTE explicite → 100 000 $/an par FTE saturé
  - Heures par semaine citées → × 50 sem × 80 $/h
- **NOUVEAU v1.3 : Inférence avec défaut conservateur si DEUX anchors verbaux concrets** :
  - Signal d'heures vague (`quelques heures par semaine`, `pas mal de temps`, `tout le monde y passe du temps`) ET nombre de personnes cité → conversion avec défaut conservateur (voir tableau ci-dessous)
  - Documenter dans Notes : `Hypothèses : 'quelques heures' converti en 2 h/sem par personne (défaut conservateur v1.3), 20 personnes citées par Steven en atelier.`

**Va en Notes (jamais dans la cellule)** :
- Signal d'heures vague SANS nombre de personnes cité (un seul anchor manquant)
- Estimation contextuelle du LLM avec multiplicateur inventé (`% captable`, `probabilité de`, `valeur moyenne contrat` sans citation)
- Extrapolation de volumes sans signal dans les inputs
- Toute valeur dont la dérivation invente un paramètre

**Test décisif v1.3** : "Pour cette valeur de cellule, est-ce que j'ai (a) une citation verbatim sponsor OU (b) une formule simple sur chiffres durs OU (c) une inférence obvious sur signal cité + standard méthode OU (d) une paire d'anchors (heures vague + nombre personnes) + défaut conservateur documenté ? Si oui → cellule. Sinon → cellule = 0, hypothèse en Notes."

## Défauts conservateurs pour signaux d'heures vague (v1.3)

Tableau de conversion à utiliser quand un signal d'heures vague est cité ET un nombre de personnes est cité :

| Signal verbal | Défaut conservateur par personne | Range plausible | Note Hypothèses obligatoire |
|---------------|----------------------------------|-----------------|------------------------------|
| "quelques heures par semaine" | 2 h/sem | 2-5 h/sem | "défaut conservateur 2 h/sem, valeur basse du range" |
| "pas mal de temps par semaine" | 3 h/sem | 3-6 h/sem | "défaut conservateur 3 h/sem" |
| "du temps notable", "ça consomme" | 2 h/sem | 2-5 h/sem | "conservateur" |
| "une partie significative de la journée" | 2 h/jour (10 h/sem) | 2-4 h/jour | "borne basse du range" |
| "la moitié de la journée" | 4 h/jour (20 h/sem) | citation plus précise | "valeur médiane du range" |
| "tout le monde y passe du temps" | 2 h/sem | conservateur | "défaut bas faute de précision" |

**Tous ces défauts sont à la borne basse** pour préserver l'anti-gonflage. La note Hypothèses dans Notes documente toujours `valeur conservative à valider en atelier si arbitrage critique`.

Si le sponsor cite un range explicite (ex : "entre 5 et 10 heures par semaine"), utiliser la borne basse (5 h/sem).

---

## Exemples par colonne

### Col 13 — Impact $ Temps Perdu

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "20 à 30 heures par semaine de matching" (Shirley) | `100 000 $` | Formule : 25 h/sem × 50 sem × 80 $/h. Anchor heures cité. |
| "Valérie passe 6 heures par soumission, fait 5 soumissions/sem" (Valérie) | `120 000 $` | Formule : 30 h/sem × 50 × 80. Anchors heures et volume cités. |
| "Eric estime 10 h/sem sur la planif manuelle" (Eric) | `40 000 $` | Formule : 10 × 50 × 80. Anchor heures cité. |
| "L'équipe de 5 personnes y passe environ 2 h/jour chacune" (Nicolas) | `200 000 $` | Formule : 5 × 2 × 250 × 80. Anchors équipe et heures cités. |
| **NOUVEAU v1.3** "Tout le monde y passe quelques heures par semaine" (Steven) + Personnes Affectées = 20 cité | `160 000 $` | Inférence v1.3 : 20 personnes × 2 h/sem (défaut conservateur "quelques") × 50 × 80. Deux anchors verbaux concrets : signal d'heures vague + nombre de personnes cité. |
| **NOUVEAU v1.3** "L'équipe de 8 personnes y passe pas mal de temps chaque semaine" (Nicolas) | `96 000 $` | Inférence v1.3 : 8 × 3 h/sem (défaut conservateur "pas mal de temps") × 50 × 80. |
| **NOUVEAU v1.3** "Chacun des 15 estimateurs perd une partie significative de la journée là-dessus" (Maxime) | `300 000 $` | Inférence v1.3 : 15 × 2 h/jour (défaut conservateur "partie significative") × 250 × 80. |

**Va en Notes** (v1.3, plus permissif mais toujours strict si anchor manque) :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "On y passe beaucoup de temps" (sans nombre de personnes ni précision horaire) | `0` | `À chiffrer : combien de personnes y consacrent du temps et combien d'heures par semaine ?` |
| "Quelques heures par semaine" sans nombre de personnes cité | `0` | `À chiffrer : combien de personnes sont concernées ? Si 10 personnes × 2 h/sem = +80 000 $.` |
| "Beaucoup de monde" sans signal d'heures | `0` | `À chiffrer : combien d'heures par personne en moyenne ?` |

### Col 14 — Impact $ Opportunités Manquées (très restrictif)

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "On perd environ 10 contrats par an à 50 000 $ chacun" (Maxime) | `500 000 $` | Citation directe avec valeur ET volume cités. |
| "Tableau commercial montre 8 leads perdus en 2025, valeur moyenne 75 000 $" | `600 000 $` | Tableau source explicite. |

**Va en Notes** (cas typique) :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Trois semaines au lieu de 24 heures, ça bloque" (Valérie, sans chiffre contrat) | `0` | `À chiffrer : combien de contrats perdus par an ? À titre indicatif si 5 contrats × 100 000 $ × 30 % capture = +150 000 $.` |
| "Les gros clients nous trouvent trop lents" | `0` | `À chiffrer : volume et valeur des appels d'offres perdus.` |

### Col 15 — Impact $ Coût Erreurs

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "180 000 $ de non-conformité par an" (Nicolas) | `180 000 $` | Citation directe valeur annuelle. |
| "2 erreurs majeures/an à 15 000 $ chacune en marge perdue" (Maxime) | `30 000 $` | Citation directe volume × coût. |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "180 000 $ non-conformité" + LLM veut appliquer "40 % captable" | `0` | `À chiffrer : quel % de la non-conformité est captable par vision auto ? À titre indicatif si 40 % captable = +72 000 $.` |
| "On fait des erreurs occasionnellement" | `0` | `À chiffrer : taux d'erreur et coût moyen.` |

**Cas frontière critique** : le 180 000 $ cité par le sponsor est l'**impact actuel total**, pas l'impact captable de l'automatisation. Le multiplicateur "% captable" est une hypothèse LLM, donc cellule = 0 et le 40 % captable va en Notes.

### Col 16 — Impact $ Main d'Œuvre / Saturation (adoucissement v1.2 important ici)

**Va dans la cellule (NOUVEAU v1.2)** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "Sophie est à temps plein sur les soumissions" (Maxime) | `100 000 $` | **NOUVEAU v1.2** : citation FTE explicite ("temps plein" = 1 FTE saturé) → standard FTE 100 000 $/an. |
| "1 personne dédiée au matching PO" (Shirley) | `100 000 $` | **NOUVEAU v1.2** : citation FTE explicite. |
| "Il faut quelqu'un de full-time là-dessus, sinon ça plante" (Nicolas) | `100 000 $` | **NOUVEAU v1.2** : citation FTE explicite ("quelqu'un de full-time" = 1 FTE). |
| "Engagement Direction : on évite 1 FTE chargé à 120 000 $" | `120 000 $` | Citation directe avec valeur explicite (override du standard 100k). |
| "Valérie a fait 200 h sup payées à 60 $/h en 2025" (paie) | `12 000 $` | Donnée source explicite. |

**Va en Notes** (v1.2, plus restrictif qu'on pourrait penser) :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Valérie est saturée" (sans citation FTE explicite ni heures sup chiffrées) | `0` | `À chiffrer : Valérie représente-t-elle vraiment un FTE saturé sur cette tâche ? Si oui = 100 000 $ Main d'Œuvre.` |
| "On ne peut pas grossir sans embaucher" (sans engagement chiffré) | `0` | `À chiffrer : combien de FTE évités si l'automatisation marche ?` |
| "L'expert RH est sollicité trop souvent" (sans heures sup ni FTE) | `0` | `À chiffrer : heures sup ou FTE équivalent ?` |

**Différence clé v1.0/v1.1 → v1.2** : avant, "Sophie est à temps plein" → 0 (pas de citation $ verbatim). Maintenant, "à temps plein" est un anchor FTE explicite qui se convertit avec le standard de méthode à 100 000 $. La traçabilité est préservée : la citation verbatim + le standard documenté pointent au résultat.

### Col 17 — Impact $ Dépendances Externes

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "Pénalités contractuelles Sabre 30 000 $/an documentées" | `30 000 $` | Document source explicite. |
| "Retards fournisseur causent 50 000 $ de stockage supplémentaire/an" | `50 000 $` | Citation valeur explicite. |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Sabre est lent" (sans pénalité chiffrée) | `0` | `À chiffrer : coûts directs des retards Sabre.` |
| "On dépend de fournisseurs externes" | `0` | `À chiffrer : volume et coût des blocages annuels.` |

### Col 18 — Impact $ Frictions Inter-Départementales

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "Estimation et Production échangent 5 emails par dossier, 30 min/dossier × 1500 dossiers × 70 $/h" (Maxime) | `52 500 $` | Citation volume + temps + taux. |
| "Comptabilité refait 20 % du travail de Réception, 10 h/sem chacune" (Shirley) | `40 000 $` | Citation volume et temps. |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Il y a beaucoup d'allers-retours" (sans chiffre) | `0` | `À chiffrer : combien d'échanges et temps consommé en moyenne ?` |

---

## Frontière critique : chiffre dur vs hypothèse LLM (rappel v1.2)

**Test rapide** : "Pour cette valeur de cellule, est-ce que tous les paramètres de la formule sont :
- (a) cités verbatim dans les inputs OU
- (b) un standard de méthode documenté (80 $/h, 100 000 $/FTE) ?

Si OUI les deux → cellule.
Si NON, il y a un paramètre inventé (% captable, probabilité, valeur unitaire non citée) → cellule = 0, hypothèse en Notes."

Exemples qui transforment une formule en hypothèse (cellule = 0) :
- **% captable** d'un pain actuel
- **Probabilité de capture** d'opportunités
- **Valeur moyenne** contrat sans citation
- **Multiplicateur de gain** sur un coût existant

Exemples qui restent dans la cellule (v1.2 adoucie) :
- **Heures par semaine** citées par sponsor + 80 $/h standard
- **FTE saturé** cité par sponsor + 100 000 $ standard
- **Valeur annuelle directe** citée par sponsor
- **Volume × valeur unitaire** tous deux cités par sponsor

## Standards de méthode (résumé)

**Coût horaire fully-loaded** : 80 $/h. Appliqué seulement si heures citées explicitement.

**FTE chargé fully-loaded** : 100 000 $/an. Appliqué seulement si saturation FTE citée explicitement ("temps plein", "1 FTE", "dédié", "full-time"). Cohérent avec 80 $/h × 1250 h productives/an.

Si le sponsor cite un taux ou profil spécifique différent, utiliser cette valeur et documenter la source dans Notes.

## Interaction avec Step 2bis Root Cause et col 27 Impact Potentiel

Step 2bis identifie les root causes actionnables, les symptômes se consolident en Pain Points sur la ligne de la root cause. L'Impact $ chiffré capture l'impact agrégé sans double-compter.

Toutes les estimations indicatives chiffrées présentes dans Notes sous "À chiffrer en atelier" alimentent la col 27 Impact Potentiel Estimé qui entre dans le Score avec discount 0.3. Donc les use cases haut potentiel non encore chiffré ne tombent plus à Score 0 dans le tri.
