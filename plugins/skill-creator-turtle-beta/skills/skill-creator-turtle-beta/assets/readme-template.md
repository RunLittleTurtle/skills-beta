# `<repo_name>` — Marketplace de skills de `<author_name>`

Marketplace Claude Code regroupant les skills publics de `<author_name>`. Une seule commande pour les avoir tous, tu actives ceux que tu veux.

## Installation

### Pour Claude Code

```
/plugin marketplace add <github_user>/<repo_name>
```

Va dans l'onglet **Discover** et active les skills voulus (Espace pour toggle), ou installe en CLI :

```
/plugin install <nom-du-skill>@<repo_name>
```

- **Mise à jour** : `/plugin marketplace update <repo_name>`
- **Désinstallation** : `/plugin uninstall <nom>@<repo_name>`

### Pour les autres outils (Claude Desktop, OpenCode, GitHub Copilot, Cursor, etc.)

Les skills respectent le standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill voulu dans le dossier skills de ton outil :

```bash
git clone https://github.com/<github_user>/<repo_name>.git
cp -R <repo_name>/plugins/<nom-du-skill>/skills/<nom-du-skill> <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

## Skills disponibles

| Nom | Description | Use case |
|---|---|---|
| _(aucun pour le moment — ajoute-en avec `skill-creator-turtle`)_ | | |

## Licence

MIT — voir [LICENSE](./LICENSE).
