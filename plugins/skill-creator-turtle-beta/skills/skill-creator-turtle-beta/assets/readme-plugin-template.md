# <slug>

> <description complète, identique au champ `description` du frontmatter de SKILL.md et de plugin.json>

## Use case

<1-3 phrases qui expliquent CONCRÈTEMENT quand invoquer ce skill — quelle situation, quel input, quel output. Reprendre le « À invoquer quand… » de la description si présent, en l'étoffant.>

## Installation

### Pour Claude Code

```
/plugin marketplace add <github_user>/<repo_name>
/plugin install <slug>@<repo_name>
```

Mise à jour : `/plugin marketplace update <repo_name>`. Désinstallation : `/plugin uninstall <slug>@<repo_name>`.

> **Mode local** (sans GitHub) : remplace `<github_user>/<repo_name>` par le chemin absolu du repo, ex: `/plugin marketplace add /Users/foo/code/<repo_name>`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone <repo_url>
cp -R <repo_name>/plugins/<slug>/skills/<slug> <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Slash command Claude Code : `/<slug>:<slug>` (ou `/<slug>` selon ta config).

<Optionnel : 1-2 phrases sur le déroulé interactif si le skill a un workflow de questions, ou un exemple d'invocation avec arguments si pertinent.>

## Contenu

- `skills/<slug>/SKILL.md` — workflow complet (frontmatter `name` + `description` + étapes).
<-- Lignes optionnelles à ajouter si le skill a un dossier references/ ou assets/ : -->
<!-- - `skills/<slug>/references/<fichier>.md` — <quand le lire> -->
<!-- - `skills/<slug>/assets/<fichier>.md` — <à quoi ça sert> -->

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.

<!--
Substitutions à faire au moment de la génération (skill-creator-turtle Étape 7 Type A) :

- <slug>            : valeur saisie à l'Étape 3
- <description>     : valeur saisie à l'Étape 4 (description complète, telle quelle)
- <github_user>     : config.github_user (mode github)  ou  <chemin_local>  (mode local — remplacer toute la phrase install par le chemin)
- <repo_name>       : config.repo_name
- <repo_url>        : config.repo_url (mode github) ; en mode local, supprimer la sous-section "Pour les autres outils" ou adapter avec un chemin
- <author_name>     : config.author_name (utilisé seulement si l'utilisateur ajoute une mention "Auteur" — non inclus dans le template par défaut)

Mode local additionnel : remplace la commande `/plugin marketplace add <github_user>/<repo_name>` par `/plugin marketplace add <repo_dir>` (chemin local absolu) et supprime le bloc `git clone <repo_url>` (ou pointe vers une URL si l'utilisateur a un remote autre que GitHub).

Lignes "Contenu" : générer dynamiquement selon ce qui existe vraiment (dossier references/ et/ou assets/ présent ou pas dans le skill généré).
-->
