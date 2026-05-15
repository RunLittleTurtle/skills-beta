# skills-beta — Marketplace BETA personnelle de Samuel Audette

> **AVERTISSEMENT — Usage personnel, versions instables et expérimentales**
>
> Cette marketplace contient des skills en cours de développement, des refontes expérimentales, des versions parallèles non éprouvées, des archives, et des forks adaptés. Toute entrée ici peut casser, changer de nom, ou disparaître à tout moment. Quand un skill beta est éprouvé, il est promu vers [`RunLittleTurtle/skills`](https://github.com/RunLittleTurtle/skills) (la marketplace stable).

## Installation (en parallèle de la stable)

```
/plugin marketplace add RunLittleTurtle/skills-beta
/plugin install <slug>@skills-beta
```

Mise à jour : `/plugin marketplace update skills-beta`. Désinstallation : `/plugin uninstall <slug>@skills-beta`.

Les deux marketplaces (`skills` et `skills-beta`) cohabitent sans conflit. Les slash commands beta utilisent le suffixe `-beta` (par exemple `/agent-talk-beta`) pour cohabiter avec leurs équivalents stables, sauf `product-management` qui garde son slug d'origine (fork Anthropic, slug conservé pour ne pas casser le mécanisme interne).

## Skills disponibles en beta

| Nom | Type | Description courte |
|---|---|---|
| `agent-talk-beta` | parallèle | Messages structurés entre instances Claude (conversation, rapport, handoff) via dossier-bridge partagé. Complète `/coordination`. |
| `product-brief-v1-beta` | archive | Version 1 originale de `product-brief`. Conservée pour reproductibilité. La version active est `product-brief` (stable). |
| `product-management` | fork | Boîte à outils PM forkée du plugin Anthropic (knowledge-work-plugins) adaptée pour Claude Code CLI — slug conservé pour compatibilité interne. |
| `scenario-uc-v2-beta` | parallèle | Version parallèle de `scenario-uc` avec scénarios alternatifs format HEC, boucles LOOP/FIN LOOP, préfixe AS-IS/TO-BE. |
| `skill-creator-turtle-v1-beta` | beta active | Refonte de `skill-creator-turtle` alignée sur les principes Anthropic + workflow de modification d'un skill existant. |
| `use-case-prioritization-beta` | draft | Score + priorise + chiffre use cases (BABOK + UiPath Suitability + Run cost benchmarké web). |
| `use-case-value-beta` | parallèle | Priorise use cases par impact business chiffré (6 sources Impact $, Score amplifié transverse, Top 10 décisif). |

### Légende des types

- **parallèle** : version expérimentale qui cohabite avec une version stable du même skill
- **archive** : version historique conservée pour reproductibilité
- **fork** : adaptation d'un plugin externe (Anthropic, communauté)
- **beta active** : refonte d'un skill stable en cours de validation avant promotion
- **draft** : skill pas encore production-ready

## Workflow de promotion vers stable

Quand un skill beta est éprouvé :

1. Copier le contenu de `skills-beta/plugins/<slug>-beta/` vers `skills/plugins/<slug>/` (sans le suffixe `-beta`).
2. Renommer le dossier interne `skills/<slug>-beta/` → `skills/<slug>/`.
3. Patcher le frontmatter `name:` du SKILL.md (`<slug>-beta` → `<slug>`).
4. Patcher `plugin.json` (`name` + `homepage`).
5. Mettre à jour `marketplace.json` et `README.md` du repo stable.
6. Vérifier qu'aucune mention résiduelle de `<slug>-beta` ne traîne : `grep -r "<slug>-beta" stable/plugins/<slug>/`.
7. Commit + push stable.

Le `<slug>-beta` peut rester dans la marketplace beta comme archive ou être retiré au prochain cycle.

## Structure du repo

```
skills-beta/
├── .claude-plugin/marketplace.json    Catalogue (7 plugins)
├── plugins/
│   └── <slug>/
│       ├── .claude-plugin/plugin.json Manifest du plugin
│       ├── README.md                  Page du plugin
│       └── skills/<slug>/SKILL.md     Le skill (frontmatter + workflow)
├── CLAUDE.md, COORDINATION.README.md  Protocole multi-instance
├── README.md
└── LICENSE
```

## Licence

MIT — voir [LICENSE](./LICENSE).
