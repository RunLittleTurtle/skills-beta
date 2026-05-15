# use-case-value-beta (v1.3)

> Priorise les use cases d'AI/automatisation par **impact business chiffré documenté ou clairement chiffrable**, à partir de matériel d'atelier de découverte (transcripts, post-its, inventaires, notes). Six sources d'Impact $ explicites + colonne `Dépend de` + colonne `Impact Potentiel Estimé`. Step 2bis Root Cause Analysis (5 Whys / Ishikawa) avant chiffrage. **Règle d'or v1.3 adoucie** : les cellules acceptent les chiffres durs cités, les formules simples sur chiffres durs, les inférences obvious sur signaux cités via standards de méthode documentés (80 $/h, 100 000 $/FTE), ET **les inférences sur paire d'anchors verbaux** (signal d'heures vague + nombre de personnes cité = chiffrable via défauts conservateurs documentés à la borne basse). Refusé : multiplicateurs sortis du chapeau ou inférence avec un seul anchor manquant. **Score Priorité Impact v1.3 amplifié pour use cases transverses** : Personnes_factor en paliers stepwise (1.0 / 1.2 / 1.5 / 1.8) + Transversalité_factor (1.0 / 1.3 / 1.6) explicites. Algorithme Verdict déterministe. **Synthèse Top 10 × 3 lignes** par entrée (classement décisif sans section À chiffrer en atelier séparée). **Aucune dimension d'effort technique** : focus pur VALEUR.

## Nouveautés v1.3 (vs v1.2)

Après run réel sur Faspac, l'utilisateur a constaté que l'use case "Agent tri courriel transverse" classait en bas du tri par Score, alors qu'il a 20 personnes affectées, des sponsors activement engagés et "quelques heures par semaine" de temps perdu par personne. Diagnostic : (1) le LLM reste trop strict sur les signaux qualitatifs faibles, (2) la formule de Score n'amplifie pas assez les use cases transverses (log faible sur Personnes, Transversalité absente), (3) la synthèse Top 5 × 5-8 lignes reporte la décision avec sections "À chiffrer".

v1.3 traite ces trois points :

- **Règle d'or v1.3 — paire d'anchors verbaux** : un signal d'heures vague (`quelques heures par semaine`, `pas mal de temps`, `tout le monde y passe du temps`) combiné avec un nombre de personnes cité devient chiffrable dans la cellule via un défaut conservateur documenté. Exemple : 20 personnes citées par Steven + "quelques heures" = 20 × 2 h/sem × 50 sem × 80 $/h = 160 000 $ en Impact Temps Perdu, avec note explicite "défaut conservateur 2 h/sem pour 'quelques', valeur basse du range plausible". Si un seul anchor manque (signal d'heures sans personnes ou inverse), cellule = 0. Le gonflage reste interdit.
- **Score Priorité Impact refondu** : Personnes_factor en paliers stepwise (1.0 / 1.2 / 1.5 / 1.8 selon tranche) et Transversalité_factor explicite (1.0 / 1.3 / 1.6 selon Transversalité 1/2/3) remplacent le log faible v1.2. Un use case transverse touchant 20 personnes est amplifié par 1.5 × 1.6 = 2.4 vs 1.0 × 1.0 = 1.0 pour un use case ponctuel touchant 3 personnes. La formule v1.2 ne donnait que 1.26× d'écart.
- **Synthèse Top 10 × 3 lignes** : la section "Les dix priorités par impact" remplace "Top 5", chaque entrée tient en 3 lignes utiles (titre + sponsor/contexte + impact/verdict). La section "À chiffrer en atelier avant décision" est supprimée : les use cases "À chiffrer prioritairement" ou "secondairement" apparaissent dans le Top 10 avec leur verdict mentionné pour transparence. Classement décisif sans revalidation systématique.
- **Urgence Stratégique rubrique élargie** : les signaux d'engagement émotionnel des sponsors en atelier ("ça m'excite", "wow", "il faut absolument") sont reconnus comme indicateurs Urgence ≥ 2 au même titre qu'un OKR ou une date butoir.

CSV reste à 27 colonnes. Workflow inchangé. Algorithme Verdict déterministe inchangé.

## Nouveautés v1.2 (vs v1.1)

Après premier usage réel sur Faspac (18 use cases, atelier 13 mai 2026), un rapport diagnostic a identifié 7 limites structurelles. v1.2 traite les 4 items Critique/Important + adoucit la règle d'or. Les 3 items Moyens sont reportés à v1.3.

- **Règle d'or adoucie** : les cellules Impact $ acceptent désormais les **inférences obvious sur signaux cités explicitement**, via standards de méthode documentés. Exemple : si le sponsor dit "Sophie est à temps plein sur les soumissions", la col Main d'Œuvre = 100 000 $ (FTE chargé standard). Avant, cette citation laissait la cellule à 0. Le gonflage reste interdit : multiplicateurs sortis du chapeau (% captable, probabilité, valeur moyenne non citée) refusés en cellule.
- **FTE chargé fully-loaded standard** = 100 000 $/an, ajouté au coût horaire standard 80 $/h comme paramètre de conversion. Appliqué uniquement quand un sponsor cite explicitement une saturation FTE.
- **Nouvelle col 27 Impact Potentiel Estimé** : le LLM somme les estimations indicatives chiffrées présentes dans Notes (section "À chiffrer en atelier"). N'entre pas dans Impact Total Documenté (qui reste pur chiffres durs) mais entre dans le Score Priorité Impact avec discount 0.3. Conséquence : les use cases haut potentiel non encore chiffré (Score 0 en v1.1) apparaissent correctement dans le tri.
- **Score Priorité Impact refondu** : `(Impact Total Documenté + 0.3 × Impact Potentiel Estimé) × Complétude × Urgence × (1 + 0.2 × log(Personnes))`. Le facteur 0.3 reflète l'incertitude des estimations Notes vs chiffres durs.
- **Algorithme Verdict déterministe** : if-elif-else numéroté remplace la grille texte interprétable. Inclut nouvelle ligne "Forte (chiffrage à compléter)" pour combler le trou Impact > 100k$ + Complétude = 1.
- **Rubrique Complétude élargie** : niveau 2 reconnaît désormais "1 à 3 cellules chiffrées dont la cellule dominante" (avant : 2 à 3 cellules chiffrées). Une seule cellule à 250 000 $ qui capture l'essentiel mérite niveau 2.
- **Quality check anti-prudence-excessive** : si Verdict mécanique = "À chiffrer prioritairement", le LLM ne doit pas downgrader en "secondairement" par prudence.
- CSV passe de 26 à 27 colonnes. Workflow inchangé sauf Step 3 mis à jour pour règle d'or adoucie et calcul col 27.

## Nouveautés v1.1 (rappel)

- Step 2bis Root Cause Analysis (5 Whys + Ishikawa) avant chiffrage : distingue root cause actionnable vs symptôme.
- Col 26 Dépend de : modélise les dépendances inter-use-cases.
- Section optionnelle "Dépendances et root causes" dans la synthèse.

## Use case

Tu as fini un atelier de découverte (transcripts, post-its, inventaires) et tu dois décider quel use case mérite d'être chiffré et scopé en premier sur la base de l'impact business **réellement documenté** chez ton client. Le skill transforme le matériel brut en deux livrables prêts à partager : un CSV de travail 25 colonnes (avec FORMULES et SOURCE pour traçabilité) et une synthèse exécutive en prose narrative pour les directeurs de compte.

## Pourquoi ce skill et pas `use-case-prioritization-beta` v2.2 ?

`use-case-prioritization-beta` v2.2 calcule un ROI complet (impact + effort + payback + verdicts hybrides) mais a deux limites observées en usage réel terrain :

1. **Le modèle de Bénéfice Net est trop étroit** : il repose principalement sur Heures × Coût horaire, ce qui sous-estime massivement les use cases dont la valeur vient des opportunités commerciales, des erreurs absorbées en marge, ou de la saturation de ressources critiques. Un use case "Soumissions clients" peut classer rang 9 mécanique alors qu'il vaut 200 000 $ dans la réalité.

2. **Le CSV à 38 colonnes et la synthèse à 5 sections deviennent illisibles** pour un directeur de compte non technique. Trop de chiffres par phrase, jargon de bundling LLM, perte du fil narratif.

`use-case-value-beta` adopte une approche différente : **séparation de la lecture impact et de la lecture effort**. Ce skill se concentre uniquement sur l'impact, avec 6 sources de valeur explicites et une règle d'or stricte sur l'absence d'estimations LLM dans les cellules. L'effort sera traité par un futur skill séparé qui pourra se brancher sur le CSV de sortie pour fermer la lecture ROI.

Le plugin v2.2 `use-case-prioritization-beta` reste disponible en parallèle ; les deux skills coexistent dans la marketplace.

## Règle d'or — chiffres durs uniquement dans les cellules

Pour éviter que les estimations LLM viennent gonfler les vraies données :

- **Cellules Impact $ (cols 13-18)** : uniquement des valeurs citées par un sponsor / participant atelier, ou dérivées par formule simple sur ces chiffres durs (ex : "20-30 heures par semaine" × 50 sem × 80 $/h fully-loaded). Sinon : 0 ou vide.
- **Colonne Notes (col 25)** : les estimations indicatives, sous forme structurée :
  ```
  À chiffrer en atelier :
  - Combien de contrats perdus par an à cause du délai 3 semaines ? À titre
    indicatif, si 5 contrats par an × 50 000 $ × 40 % probabilité capture,
    +100 000 $ en Impact Opportunités.
  ```
  Ce sont des questions à poser au sponsor, pas des données à utiliser pour le verdict.

Conséquence : un use case "vraiment important mais mal chiffré" classera initialement avec un Verdict "À chiffrer prioritairement". C'est honnête et ça aiguille le travail vers la bonne prochaine étape (atelier de validation avec le sponsor).

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills-beta
/plugin install use-case-value-beta@skills-beta
```

Mise à jour : `/plugin marketplace update skills-beta`. Désinstallation : `/plugin uninstall use-case-value-beta@skills-beta`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills-beta.git
cp -R skills/plugins/use-case-value-beta/skills/use-case-value-beta <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Slash command Claude Code : `/use-case-value-beta:use-case-value-beta` (ou `/use-case-value-beta` selon ta config).

Triggers détectés : `prioriser par impact`, `valeur use case`, `quel use case en premier`, `chiffrer la valeur`, `note de cadrage`, `atelier découverte`, `post-its`, `transcript atelier`, `inventaire processus`, `impact business`, `pertes évitées`, `opportunités manquées`.

## Contenu

- `skills/use-case-value-beta/SKILL.md` — workflow Step 0-7.
- `skills/use-case-value-beta/references/framework.md` — explication des 25 colonnes, rubriques 1-3, sources méthodologiques (BABOK Business Case, WSJF, DAMA-DMBOK Data Availability).
- `skills/use-case-value-beta/references/calculation_rules.md` — formules, défauts (coût horaire 80 $/h), grille de verdict.
- `skills/use-case-value-beta/references/csv_template.csv` — gabarit 25 colonnes avec FORMULES + SOURCE + exemple fictif Soumissions Acme.
- `skills/use-case-value-beta/references/synthese_template.md` — gabarit synthèse one-pager prose narrative.
- `skills/use-case-value-beta/references/exemples_chiffrage_impact.md` — aide-mémoire pour distinguer chiffre dur vs hypothèse, exemples typiques.

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
