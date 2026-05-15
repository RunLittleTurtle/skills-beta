# agent-talk-beta

> Échanger des messages structurés (conversation back-and-forth, rapport one-shot, handoff de contexte) entre deux instances Claude — même repo à des profondeurs différentes OU projets séparés — via un dossier-bridge partagé. Notification par pings courts `[PING] @inst-X` qui ne polluent pas le log de `/coordination`.

## Use case

Tu lances Claude dans deux fenêtres iTerm avec des cwd différents. Soit elles sont dans le même repo à des profondeurs différentes (ex: agent A dans `/repo/`, agent B dans `/repo/sub/`), soit dans des projets totalement séparés (ex: agent A dans `~/code/skills/` qui crée un skill, agent B dans `~/code/projet-test/` qui le teste).

`/coordination` gère très bien les locks sur des fichiers partagés. Mais parfois un agent a besoin de **parler** à l'autre — poser une question, passer un brief, démarrer une discussion async. Mettre ça dans le log `COORDINATION.FOCUS.md` finit par overcrowd le journal.

`agent-talk-beta` crée un dossier-bridge (`<cwd>/.agent-talk-beta/<topic>/` ou `~/.claude/agent-talk-bridges/<topic>/`) avec :

- `conv-<slug>.md` pour les threads back-and-forth
- `report-<slug>.md` pour les rapports one-shot avec question
- `handoff-<slug>.md` pour les passations de contexte

Et un système de pings courts : `- [PING] feedback-slug @inst-B by inst-A — bridge:conv-feedback-slug.md — "feedback sur le slug ?"`. Le ping va dans `COORDINATION.FOCUS.md` si les deux agents partagent le même repo + coordination active, sinon dans `BRIDGE.PINGS.md` dédié au bridge.

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills-beta
/plugin install agent-talk-beta@skills-beta
```

Mise à jour : `/plugin marketplace update skills-beta`. Désinstallation : `/plugin uninstall agent-talk-beta@skills-beta`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills-beta.git
cp -R skills/plugins/agent-talk-beta/skills/agent-talk-beta <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

```
/agent-talk-beta:agent-talk-beta
```

Au premier lancement dans un cwd, le skill scanne les bridges existants accessibles :

- **Walk-up** depuis `<cwd>` jusqu'à `$HOME` : trouve les `.agent-talk-beta/<topic>/` créés par un agent au même niveau OU plus haut dans l'arbre.
- **Walk-down** depuis `<cwd>` (jusqu'à 5 niveaux) : trouve les `.agent-talk-beta/<topic>/` créés par un agent plus profond.
- **Zone neutre** : `~/.claude/agent-talk-bridges/*/` pour les bridges cross-projet.

Pareil pour les `COORDINATION.*.md` (cf. skill `coordination`) : le walk-up trouve le fichier coord même s'il est au-dessus du `git rev-parse --show-toplevel` local (cas des nested git repos / submodules).

S'il y en a, il propose de te connecter. Sinon il propose d'en créer un. Mode : `same-repo nested` (bridge dans le cwd local) ou `cross-projet` (path absolu désigné, default `~/.claude/agent-talk-bridges/<topic>/`).

## Trois types de fichiers d'échange

| Préfixe | Type | Quand l'utiliser |
|---|---|---|
| `conv-<slug>.md` | Conversation back-and-forth | Discussion async, question + suivi alterné |
| `report-<slug>.md` | Rapport one-shot avec question | Contexte + question précise, le destinataire répond une fois |
| `handoff-<slug>.md` | Passation de contexte | Brief structuré (sujet, files, décisions, next, blocages) pour qu'un autre agent continue |

Chaque message se termine par un turn marker explicite `**Tour : @<destinataire>**` (ou `**Tour : (fermé)**` pour clôturer).

## Statuts dans le log de notification

| Statut | Signification |
|---|---|
| `[PING]` | Nouveau message dispo pour le destinataire |
| `[READ-PING]` | Le destinataire a lu, ack |
| `[CLOSED]` | Fichier de message clôturé |
| `[JOIN]` | Une instance a rejoint le bridge |
| `[BRIDGE-CREATED]` | Bridge créé (en mode same-repo avec coordination) |

Détails complets du protocole dans `skills/agent-talk-beta/SKILL.md` et dans le `BRIDGE.README.md` généré à la création de chaque bridge.

## Complémentarité avec `coordination`

| Skill | Rôle |
|---|---|
| `coordination` | Locks logiques sur des fichiers partagés (même repo). Évite que 2 agents écrivent en même temps sur le même fichier de doc. |
| `agent-talk-beta` | Conversations / rapports / handoffs entre agents. Peut fonctionner sur le même repo en parallèle de `coordination`, ou en cross-projet sans coordination. |

Les deux skills s'utilisent indépendamment ou ensemble. En mode same-repo, `agent-talk-beta` peut piggyback sur `COORDINATION.FOCUS.md` pour les pings (au lieu de créer un fichier de notif séparé), si tu actives ce mode au setup.

## Contenu

- `skills/agent-talk-beta/SKILL.md` — workflow complet (détection bridges existants, création, modes de notification, identité, envoi/lecture/listing messages, règles session, templates générés).

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
