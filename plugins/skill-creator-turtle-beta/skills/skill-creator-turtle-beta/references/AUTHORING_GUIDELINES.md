# Guidelines pour rédiger un bon SKILL.md

À lire quand tu (skill-creator-turtle) guides l'utilisateur dans la rédaction d'un nouveau skill, OU quand l'utilisateur demande de "vérifier la qualité" d'un skill existant.

## Sources officielles (2026)

- [Anthropic Claude Code docs](https://code.claude.com/docs/en/skills) — référence Claude Code
- [Anthropic Claude API docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) — vue cross-product
- [agentskills.io specification](https://agentskills.io/specification) — standard ouvert

## Limites concrètes

| Élément | Limite |
|---|---|
| `name` (frontmatter) | 1-64 caractères, kebab-case minuscule, pas de double-tiret, pas de tiret en début/fin |
| `description` (frontmatter) | 1-1024 caractères ; doit dire QUOI fait le skill ET QUAND l'invoquer |
| Combiné `description` + `when_to_use` | tronqué à 1536 caractères dans le listing skills |
| Body `SKILL.md` | **< 500 lignes** recommandé (Anthropic + agentskills.io) |
| Body `SKILL.md` (tokens) | **< 5 000 tokens** quand le skill est chargé en contexte (Anthropic) |
| `compatibility` (optionnel) | 1-500 caractères |

## Pattern "Progressive disclosure"

Quand le SKILL.md dépasse 500 lignes, **scinde** : garde le workflow principal dans `SKILL.md`, déplace les détails dans des fichiers annexes lus à la demande.

```
my-skill/
├── SKILL.md          # workflow principal (< 500 lignes)
├── references/       # docs détaillées (lues à la demande)
│   └── REFERENCE.md
├── scripts/          # exécutables (jamais chargés en contexte)
│   └── helper.py
└── assets/           # templates, schémas, exemples
    └── template.md
```

Référence depuis `SKILL.md` :

```markdown
Pour le détail des règles, voir [references/REFERENCE.md](references/REFERENCE.md).
Le template README est dans `${CLAUDE_SKILL_DIR}/assets/readme-template.md`.
```

`${CLAUDE_SKILL_DIR}` est la variable Claude Code qui pointe vers le dossier du skill (utile pour des paths absolus quand le cwd peut varier).

## Frontmatter minimal et conforme

```yaml
---
name: pr-review
description: Examiner une pull request pour bugs, sécurité et performance. Commence par fetcher le diff via gh, puis analyse en 4 axes (logique métier, sécurité, perf, lisibilité). À invoquer quand l'utilisateur demande de reviewer une PR par numéro ou veut une seconde opinion sur des changements.
---
```

Anti-patterns dans le frontmatter :
- `version`, `tags`, `keywords` non standard → pas dans agentskills.io spec
- Description vague ("aide avec X") → Claude ne saura pas quand l'invoquer
- Description sans "À invoquer quand..." → manque le déclencheur

## Style du body

- **Verbe d'action en première phrase** (ex: "Coordonner...", "Générer...", "Analyser...").
- **Pas de narration** ("Ce skill va...") → écris directement les instructions.
- **Pas de comments meta** ("Note : ce skill est cool") → garde uniquement ce qui aide Claude à exécuter.
- **Étapes numérotées** quand le workflow a un ordre. Sections markdown sinon.
- **Style explicite** en haut (langue, concision) : `**Réponds en français. Sois concis.**`
- **Exemples concrets** (BON vs MAUVAIS) quand un format précis est critique.

## Bons exemples de structure

### Skill court (< 100 lignes) — référence inline

```markdown
---
name: api-conventions
description: Conventions API REST pour ce codebase. À appliquer quand l'utilisateur écrit ou modifie un endpoint.
---

Quand tu écris un endpoint API :
- Naming RESTful (`/users/:id`, pas `/getUser`)
- Erreurs format `{error: {code, message, details}}`
- Validation systematique des inputs
```

### Skill workflow (200-500 lignes) — étapes structurées

Voir `coordination` (419 lignes) dans cette même marketplace : 7 étapes claires, tableau de référence, exemples BON/MAUVAIS. Bon modèle.

### Skill long (> 500 lignes) — progressive disclosure

Si tu dépasses 500 lignes, scinde. Le SKILL.md principal contient le workflow et référence des fichiers annexes pour les détails.

## Adaptabilité (générique vs personnel)

- **Skill générique** (cible : partage public) :
  - Pas de chemin absolu (`/Users/<toi>/...`)
  - Pas de login GitHub hardcodé
  - Pas de nom d'organisation hardcodé
  - Détection runtime : `git rev-parse --show-toplevel`, `gh api user`, `git config`, `$HOME`
  - Mention dans la description : "À invoquer quand n'importe quel utilisateur..."

- **Skill personnel** (cible : usage privé) :
  - Hardcoding OK mais **mentionner explicitement** dans la description : "Skill personnel pour le workflow de \<X\>, pas conçu pour être partagé."
  - Préférable de le mettre en Type B (standalone `~/.claude/skills/`) plutôt que dans une marketplace publique.

## Erreurs courantes

| Erreur | Conséquence | Fix |
|---|---|---|
| `description` < 50 chars, vague | Claude ne déclenche pas le skill | Ajoute des mots-clés et "À invoquer quand X..." |
| Body > 500 lignes monolithique | Coût token élevé en contexte | Progressive disclosure (references/, assets/) |
| Chemins absolus hardcodés | Inutilisable par d'autres | Variables runtime (`$HOME`, `git rev-parse`) |
| Frontmatter avec champs custom | Validateurs Claude Code rejettent | Stick au standard agentskills.io |
| Symlinks vers le skill | Bugs cross-machine | Distribution via marketplace ou cp -R |
| Pas de "Réponds en X" | Le skill peut répondre dans la mauvaise langue | Ajoute `**Réponds en français. Sois concis.**` |

---

## Principes Anthropic (skill-creator-turtle officiel)

Source : [github.com/anthropics/skills/tree/main/skills/skill-creator-turtle](https://github.com/anthropics/skills/tree/main/skills/skill-creator-turtle). Quatre principes guident la rédaction des skills officiels Anthropic, qui s'appliquent aussi à tout skill personnel.

### 1. La description est le trigger primary

> *"When to trigger, what it does. This is the primary triggering mechanism — include both what the skill does AND specific contexts for when to use it."*

Si Claude ne déclenche pas un skill au bon moment, le réflexe doit être de revoir la description avant la logique. Bonne description = un verbe d'action en tête + une phrase « À invoquer quand... » + des mots-clés spécifiques au domaine. Pas de vague (« aide avec X »).

### 2. Progressive disclosure

Métadonnées (frontmatter ~100 mots) toujours chargées en contexte. Body SKILL.md < 500 lignes, chargé seulement quand le skill se déclenche. Ressources bundlées (`references/`, `assets/`, `scripts/`) lues à la demande, jamais préchargées.

Concrètement : si une section du SKILL.md gonfle au-delà d'une utilisation occasionnelle (un workflow de cas particulier, une table de référence, un long template), externalise-la dans `references/<sujet>.md` et mets juste un pointeur dans le SKILL.md.

### 3. Lean instructions plutôt qu'ALL-CAPS

> *"Remove things that aren't pulling their weight. Explain why behind requests rather than issuing rigid ALL-CAPS directives — today's LLMs respond better to reasoning than rote rules."*

Pas de « RÈGLE STRICTE : NE JAMAIS X ». Préfère « La raison : X. Si tu vois Y, c'est probablement une fuite, signale-le. » Claude saura alors généraliser à des cas non prévus, au lieu de chercher à matcher littéralement la règle.

### 4. Theory of mind

> *"They have good theory of mind and when given a good harness can go beyond rote instructions."*

Quand tu décris un workflow, donne le contexte et l'intention, pas juste la procédure. Claude infère mieux les cas limites quand il comprend pourquoi une étape existe. Évite les sur-spécifications du genre « fais exactement X, puis Y, puis Z » quand tu peux dire « l'objectif est Q, et voici les outils dont tu disposes ».

### Lien avec les anti-patterns ci-dessus

Ces principes Anthropic complètent (pas remplacent) les anti-patterns listés. Un SKILL.md peut respecter la limite < 500 lignes (anti-pattern technique) tout en étant rempli de directives ALL-CAPS (anti-pattern stylistique). Bonne rédaction = les deux niveaux.
