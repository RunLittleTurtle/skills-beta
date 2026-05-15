# Gabarit synthèse exécutive use-case-value-beta v1.3 — Top 10 × 3 lignes

Ce gabarit produit une **note de cadrage one-pager en prose narrative française Québec** pour directeurs de compte et sponsors clients.

Format compact v1.3 : 10 use cases prioritaires, 3 lignes chacun, classement décisif sans section "À chiffrer en atelier" séparée.

## Règles de style absolues

- **Prose narrative compacte**. Phrases complètes mais denses.
- **Pas d'em dash** dans aucun texte. Remplacer par virgule, parenthèse ou deux-points.
- **Pas de crochets** comme `[XX %]`. Écrire en toutes lettres.
- **Pas de séparateurs** `---` entre les Top.
- **Pas de pipe** `|` dans le corps du texte.
- **Pas de symboles** `≥`, `<`, `×`. Écrire "supérieur ou égal à", "moins de", "fois".
- **Garder uniquement** `%` et `$`.
- Les chiffres en toutes lettres : `220 000 $` pas `220k$`.
- **Mention sobre du Verdict** dans la 3e ligne de chaque entrée.

## Structure du document v1.3

```markdown
# Priorisation par impact — [nom du projet] ([date])

[Préambule en prose, 1 paragraphe court]

## Les dix priorités par impact

### 1. [Titre du use case]
Sponsor : [Prénom Nom] ([rôle ou département]). [Contexte chiffré, une phrase].
[Impact documenté ou potentiel, une phrase]. [Verdict + recommandation, une phrase].

### 2. [...]

[...]

### 10. [...]

## Dépendances et root causes

[Section OPTIONNELLE — incluse uniquement si col 26 Dépend de a des valeurs
non vides OU si Step 2bis a consolidé des symptômes. 2-3 paragraphes prose.]

## Recommandations méthodologiques

[1 paragraphe court.]
```

**Sections supprimées en v1.3** :
- "À chiffrer en atelier avant décision" : disparue. Les use cases avec verdict "À chiffrer prioritairement" ou "secondairement" apparaissent dans le Top 10 si leur Score les y place, avec leur verdict mentionné en ligne 3 pour transparence.

## Préambule type

1 paragraphe court (4 à 6 lignes wrappées) couvrant :

- Date et nom du projet
- Sources d'inputs
- Nombre de **candidats** identifiés en Step 1 et nombre de **root causes** retenues après Step 2bis
- Rappel scope impact-only (effort à venir séparément)

Exemple :

```
Cette note synthétise l'analyse d'impact business des use cases identifiés
lors de l'atelier de découverte du 13 mai 2026 avec l'équipe Faspac.
Vingt-deux candidats use cases ont été extraits des transcripts, des
post-its de cartographie et de la note Ishikawa. Après la Step 2bis Root
Cause Analysis (consolidation des symptômes sous leurs causes profondes
actionnables), il reste douze root causes distinctes, dont dix sont
classées ci-dessous par Score Priorité Impact décroissant. L'effort
technique de réalisation fera l'objet d'une analyse complémentaire.
```

## Top entry type (3 lignes utiles, max 5 lignes wrappées)

Chaque entrée Top suit le pattern :

1. **Ligne 1 (titre)** : `### N. [Titre du use case]`
2. **Ligne 2 (sponsor + contexte)** : `Sponsor : [Prénom Nom] ([rôle]). [Contexte chiffré une phrase].`
3. **Ligne 3 (impact + verdict)** : `[Impact documenté ou potentiel, une phrase]. [Verdict + recommandation, une phrase].`

Exemple 1 — use case transverse avec impact élevé :

```
### 1. Agent tri courriel transverse
Sponsor : Steven (Direction). 20 personnes touchées avec environ 2 heures par
semaine chacune, soit 160 000 $/an documenté en temps perdu transverse.
Sponsors activement engagés en atelier, verdict Critique, scoping immédiat
recommandé comme premier quick win transverse.
```

Exemple 2 — use case avec chiffrage partiel et potentiel élevé :

```
### 2. Agent d'estimation soumission from Business Central
Sponsor : Valérie Audet-Nadon (Finance et Commercial). Cycle actuel trois
semaines vs cible 24 heures, Sophie citée à temps plein soit 100 000 $/an
documenté en saturation. Verdict À chiffrer prioritairement, potentiel
additionnel de 280 000 $ sur opportunités contrats à confirmer en atelier
de chiffrage avec Maxime.
```

Exemple 3 — use case "Forte (chiffrage à compléter)" :

```
### 3. Dashboard recoster écarts production
Sponsor : Nicolas L (Direction Opérations). 250 000 $/an cités en
non-conformité documentée, une cellule dominante chiffrée. Verdict Forte
(chiffrage à compléter), à scoper dès que le pourcentage de captation par
l'agent est confirmé en atelier.
```

## Section "Dépendances et root causes" (optionnelle)

Inclure uniquement si :
- Au moins une ligne du CSV a une valeur non vide dans col 26 Dépend de, OU
- Step 2bis a consolidé plusieurs symptômes sous une root cause

Format : 2-3 paragraphes en prose, identifiant les chaînes critiques et les fondations à débloquer.

Exemple :

```
## Dépendances et root causes

Trois des dix priorités partagent une dépendance au Data Lake, qui apparaît
dans la col 26 Dépend de des use cases Workestimity reconciliation, Agent
flagging consommation MP et Dashboard recoster écarts. Le Data Lake
n'apparaît pas dans le Top 10 par impact direct (il n'adresse pas un pain
point business mesurable isolément), mais sa construction est préalable à
la captation des plusieurs centaines de milliers de dollars d'impact des
trois use cases consommateurs. Ordre de lancement recommandé : Data Lake
en mois un, suivi des trois use cases consommateurs en parallèle.

L'analyse Step 2bis a permis de regrouper plusieurs symptômes sous des use
cases actionnables, évitant de triple-compter l'impact. La saturation
Valérie sur les soumissions, par exemple, n'apparaît pas comme un use case
distinct mais comme un symptôme dont la root cause Agent d'estimation
soumission est listée en priorité deux.
```

Si aucune dépendance ni consolidation : **omettre entièrement cette section**.

## Section "Recommandations méthodologiques"

1 paragraphe court (4 à 6 lignes wrappées) qui rappelle les limites du chiffrage actuel, identifie les use cases qui méritent un atelier de chiffrage si pertinent, et mentionne la prochaine étape (futur skill effort).

Exemple :

```
L'impact total chiffré dans le CSV applique la règle d'or v1.3 qui autorise
les inférences sur signaux verbaux concrets (heures par personne, FTE
saturé) avec défauts conservateurs documentés à la borne basse du range
plausible. Pour les use cases en priorité un à trois, un atelier de
validation rapide avec les sponsors confirmera les défauts conservateurs.
L'effort technique de réalisation (taille de build, coût agent, intégrations)
n'est pas évalué dans cette note et viendra dans une analyse complémentaire
qui se branchera sur le CSV.
```

## Quality checks avant export

- [ ] Préambule mentionne date, sources, nombre de candidats avant et après Step 2bis, nombre de use cases retenus dans le Top 10
- [ ] **Exactement 10 entrées** dans la section "Les dix priorités par impact" (ou moins si le CSV a moins de 10 root causes ; dans ce cas, mentionner explicitement le nombre)
- [ ] Chaque entrée Top : **3 lignes utiles maximum** (titre + sponsor/contexte + impact/verdict)
- [ ] Section "Dépendances et root causes" présente si col 26 a des valeurs OU si Step 2bis a consolidé des symptômes ; omise sinon
- [ ] Section "Recommandations méthodologiques" courte, 1 paragraphe
- [ ] **AUCUNE section "À chiffrer en atelier"** séparée (supprimée en v1.3)
- [ ] Aucun em dash, aucun crochet, aucun pipe dans le corps du texte
- [ ] Chiffres en toutes lettres (220 000 $)
- [ ] Document tient sur 1 page A4 imprimée (Top 10 × 3 lignes = ~40 lignes prose)
- [ ] Aucune section Méthodologie longue
