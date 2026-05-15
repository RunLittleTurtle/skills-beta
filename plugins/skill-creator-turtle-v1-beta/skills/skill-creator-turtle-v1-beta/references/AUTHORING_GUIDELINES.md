# Guidelines pour rédiger un bon SKILL.md

À lire quand tu (skill-creator-turtle-v1-beta) guides l'utilisateur dans la rédaction d'un nouveau skill, OU quand l'utilisateur demande de "vérifier la qualité" d'un skill existant.

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
