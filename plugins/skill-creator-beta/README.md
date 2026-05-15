# skill-creator-beta

> **BETA** — Version expérimentale de [`skill-creator`](https://github.com/RunLittleTurtle/skills/tree/main/plugins/skill-creator). Aligne sur les principes Anthropic (lean instructions, theory of mind, prose explicative au lieu de tables ALL-CAPS de règles strictes) et ajoute la capacité de **modifier** un skill existant en plus de le créer.

## Use case

Mêmes cas que `skill-creator` stable :

| Type | Cible | Mode |
|---|---|---|
| **A** | Marketplace personnelle multi-skills (skills partageables) | GitHub (push auto) ou local |
| **B** | Skill standalone perso (`~/.claude/skills/<slug>/SKILL.md`) | Sans Git |
| **C** | Skill pour un autre outil (OpenCode, Cursor, Copilot, Claude Desktop…) | Sans Git |

**Plus, en exclusivité beta** : option **D — modifier** un skill existant. Le skill liste les skills détectés sur la machine (standalones, marketplaces locales clonées, copies installées via `/plugin install`), tu choisis lequel, le skill snapshot le dossier dans `/tmp/` avant édition, puis te guide à travers la modification (mode interactif, coller un nouveau body, régénérer à partir de fichiers de référence). Le slug d'origine est préservé (convention Anthropic).

**Détection automatique de fichiers de référence** : identique au stable. Invoque avec un ou plusieurs paths et le skill lit, analyse, propose des composantes dynamiques à reproduire.

## Installation

```
/plugin marketplace add RunLittleTurtle/skills-beta
/plugin install skill-creator-beta@skills-beta
```

Mise à jour : `/plugin marketplace update skills-beta`. Désinstallation : `/plugin uninstall skill-creator-beta@skills-beta`.

Cohabite avec `skill-creator` stable installé depuis `RunLittleTurtle/skills` — les deux slash commands `/skill-creator` et `/skill-creator-beta` sont utilisables en parallèle.

## Invocation

```
# Création classique ou modification (le skill demande à l'Étape 0.0)
/skill-creator-beta:skill-creator-beta

# Création avec analyse de fichier(s) de référence
/skill-creator-beta:skill-creator-beta /path/to/reference.md

# Modification avec nouveaux requirements
/skill-creator-beta:skill-creator-beta /path/to/new-requirements.md
# (à l'Étape 0.0 tu choisis D — modifier, le brief de référence sert à régénérer)
```

## Différences avec le stable

| Aspect | Stable (`skill-creator`) | Beta (`skill-creator-beta`) |
|---|---|---|
| Frontmatter strict (`name` + `description`) | OUI | OUI (inchangé) |
| 3 cibles A/B/C | OUI | OUI (inchangé) |
| Étape -1 détection auto de fichiers | OUI | OUI (inchangé) |
| Section "Règles d'exécution" | Table ALL-CAPS rigide | Prose explicative qui dit le pourquoi (theory of mind) |
| Workflow modification d'un skill existant | NON | OUI (Étape 0.0 option D, sous-workflow dans `references/MODIFICATION_WORKFLOW.md`) |
| Détection des skills installés | NON | OUI (scan standalones + marketplaces locales + cache installé) |
| Snapshot avant édition | NON | OUI (`/tmp/<slug>-snapshot-<timestamp>/`) |
| Préservation du slug en modify | NON | OUI (règle Anthropic stricte) |
| Validation structurelle post-édition | NON | OUI (frontmatter, slug invariant, taille < 500 lignes) |

## Contenu

- `skills/skill-creator-beta/SKILL.md` — workflow principal (création A/B/C + modification D, étapes -1, 0.0, 0.5, 0, 1, 2, 2bis, 3-11, sous-workflow modify).
- `skills/skill-creator-beta/references/AUTHORING_GUIDELINES.md` — limites officielles, principes Anthropic, exemples bons/mauvais (lu pendant la rédaction du body).
- `skills/skill-creator-beta/references/REFERENCE_DETECTION.md` — workflow détaillé de l'Étape -1 (identique au stable).
- `skills/skill-creator-beta/references/MODIFICATION_WORKFLOW.md` — sous-workflow M.1 à M.7 du mode modification (lu seulement quand Étape 0.0 = D).
- `skills/skill-creator-beta/assets/readme-template.md` — template README de marketplace.
- `skills/skill-creator-beta/assets/readme-plugin-template.md` — template README de plugin.

## Statut beta

Cette version est en cours de validation. Tester contre les 10 cas du plan de validation (voir le repo). Quand stable, le contenu est promu vers `skill-creator` du repo `RunLittleTurtle/skills`.

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
