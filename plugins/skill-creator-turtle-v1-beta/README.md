# skill-creator-turtle-v1-beta

> Meta-skill **GÉNÉRIQUE et ADAPTATIF** : crée interactivement un nouveau skill Claude Code (ou compatible [agentskills.io](https://agentskills.io)) dans **TA propre marketplace** — pas une marketplace prédéfinie. Détecte automatiquement les fichiers de référence passés en argument et pose des questions ciblées sur l'angle/profondeur d'analyse. Persiste la config dans `~/.claude/skill-creator-turtle/config.json`.

## Use case

Tu veux créer un skill Claude Code et tu ne veux pas réinventer la structure (`plugin.json` + `marketplace.json` + `SKILL.md` + README + commit + push) à la main. Le skill-creator-turtle-v1-beta gère **3 cibles** :

| Type | Cible | Mode |
|---|---|---|
| **A** | Marketplace personnelle multi-skills (recommandé pour skills partageables) | GitHub (push auto) ou local |
| **B** | Skill standalone perso, juste pour toi (`~/.claude/skills/<slug>/SKILL.md`) | Sans Git |
| **C** | Skill pour un autre outil (OpenCode, Cursor, Copilot, Claude Desktop…) | Sans Git |

**Détection automatique de fichiers de référence** : invoque-le avec un ou plusieurs paths (ex: `/skill-creator-turtle-v1-beta:skill-creator-turtle-v1-beta <path1> <path2>`) et le skill lit les fichiers, te pose des questions ciblées sur le rôle de chaque fichier (modèle d'output / d'input / règles métier / exemple) et la profondeur d'analyse, puis te propose une **liste DYNAMIQUE de composantes à reproduire** (sections, blocs, patterns, conventions) extraite de l'analyse réelle des fichiers — pas une liste hardcodée. Tu n'as PAS à écrire « analyse profondément » en texte libre.

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills-beta
/plugin install skill-creator-turtle-v1-beta@skills-beta
```

Mise à jour : `/plugin marketplace update skills-beta`. Désinstallation : `/plugin uninstall skill-creator-turtle-v1-beta@skills-beta`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills-beta.git
cp -R skills/plugins/skill-creator-turtle-v1-beta/skills/skill-creator-turtle-v1-beta <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Trois formes :

```
# Création classique (interactive complète)
/skill-creator-turtle-v1-beta:skill-creator-turtle-v1-beta

# Création avec analyse de fichier(s) de référence
/skill-creator-turtle-v1-beta:skill-creator-turtle-v1-beta /path/to/reference.md
/skill-creator-turtle-v1-beta:skill-creator-turtle-v1-beta /path/to/file1.py /path/to/file2.py
```

Le skill détecte ton mode (`github` vs `local`), ta config GitHub via `gh api user`, et te guide à travers : type de skill (A/B/C) → slug → description → mode de rédaction (interactif / brouillon collé / squelette minimal) → génération des fichiers → commit + push (sur confirmation).

## Garanties

- **Aucun chemin/nom hardcodé** : tout passe par les variables de ta config (`~/.claude/skill-creator-turtle/config.json`).
- **Aucune supposition** sur ton identité ni sur ton compte GitHub : si `gh` n'est pas installé/authentifié, le skill bascule en mode local.
- **Frontmatter SKILL.md** : `name` + `description` UNIQUEMENT (standard agentskills.io strict).
- **Progressive disclosure** : si le SKILL.md généré dépasse 500 lignes, le skill propose de scinder via `references/` + `assets/`.

## Contenu

- `skills/skill-creator-turtle-v1-beta/SKILL.md` — workflow principal (détection référence, branchement A/B/C, étapes 0 à 11).
- `skills/skill-creator-turtle-v1-beta/references/AUTHORING_GUIDELINES.md` — limites officielles, exemples bons/mauvais, anti-patterns (lu pendant la rédaction du body).
- `skills/skill-creator-turtle-v1-beta/references/REFERENCE_DETECTION.md` — workflow détaillé de l'Étape -1 (détection paths → analyse réelle des fichiers → liste DYNAMIQUE de composantes).
- `skills/skill-creator-turtle-v1-beta/assets/readme-template.md` — template du README de marketplace (utilisé au bootstrap d'un nouveau repo).
- `skills/skill-creator-turtle-v1-beta/assets/readme-plugin-template.md` — template du README généré à la racine de chaque nouveau plugin.

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
