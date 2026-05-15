# scenario-uc-v1-beta

> **[archivé v1]** Version 1 originale du skill `scenario-uc`, conservée pour reproductibilité. La version active (V2 avec mode AS-IS / TO-BE, séquences alternatives HEC, boucles LOOP/FIN LOOP et validation interactive renforcée) est [`scenario-uc`](https://github.com/RunLittleTurtle/skills/tree/main/plugins/scenario-uc) dans la marketplace stable.

> Transforme n'importe quel input (texte, brouillon md, PDF, image, URL Drive, idée verbale) en un fichier markdown de scénario use-case au format Authentik PRD : description, intent, séquence numérotée, diagramme de séquence Mermaid. Analyse profonde + validation interactive avant génération. Sortie en français.

## Use case

Formaliser un cas d'utilisation produit (description avec acteurs typés + intent + séquence numérotée + `sequenceDiagram`) à partir de n'importe quelle source — texte brut, brouillon, PDF, screenshot de whiteboard, URL Google Drive, conversation. Pensé pour le Projet 4 Authentik et tout PRD du même style.

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills-beta
/plugin install scenario-uc-v1-beta@skills-beta
```

Mise à jour : `/plugin marketplace update skills-beta`. Désinstallation : `/plugin uninstall scenario-uc-v1-beta@skills-beta`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills-beta.git
cp -R skills-beta/plugins/scenario-uc-v1-beta/skills/scenario-uc-v1-beta <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Slash command Claude Code : `/scenario-uc-v1-beta:scenario-uc-v1-beta` (ou `/scenario-uc-v1-beta` selon ta config).

Le skill demande l'input puis valide interactivement le plan d'analyse avant de générer le fichier.

## Contenu

- `skills/scenario-uc-v1-beta/SKILL.md` — workflow complet (frontmatter `name` + `description` + étapes).

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
