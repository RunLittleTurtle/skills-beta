# Product Management (adapté Claude Code)

Boîte à outils Product Management pour Claude Code CLI. **Fork adapté** du plugin officiel d'Anthropic [`product-management`](https://github.com/anthropics/knowledge-work-plugins/tree/main/product-management), qui est initialement conçu pour Claude Cowork (app desktop). Cette version fonctionne dans un repo de code ou un dossier local de PRD/notes — pas besoin de Cowork.

## Ce qui a changé vs. l'original

| Aspect | Source Cowork | Cette version Claude Code |
|---|---|---|
| Environnement cible | App desktop avec UI dédiée | CLI dans le CWD courant |
| MCP servers | `.mcp.json` avec 16 connecteurs HTTP préconfigurés | Aucun `.mcp.json` — opt-in via ta config Claude Code globale |
| Sources de contexte | Slack, Linear, Notion, Figma, Amplitude, etc. via MCP | Fichiers markdown locaux (Obsidian, dossiers de PRD), paste manuel, et MCP optionnels que tu as configurés |
| Langue interface | EN | FR (descriptions + README), EN (corps des skills — préservé pour qualité) |
| Auteur | Anthropic | Samuel Audette (adaptation) |

Le **contenu pédagogique des skills** (méthodologies, frameworks, templates) reste **identique au plugin officiel** — c'est de la matière de PM solide qui ne dépend pas de l'environnement.

## Installation

```
/plugin marketplace add RunLittleTurtle/skills
/plugin marketplace update skills
/plugin install product-management@skills
```

Puis `/reload-plugins` si besoin.

## Commandes & Skills

### Commande explicite
| Commande | Action |
|---|---|
| `/product-management:brainstorm <topic>` | Brainstormer un sujet produit avec un partenaire de réflexion incisif |

### Skills (auto-déclenchés)
Tu n'as pas besoin de taper une slash command — Claude détecte le contexte et invoque le bon skill quand tu décris ton besoin. Tu peux aussi forcer un skill en mentionnant son nom.

| Skill | Quand l'invoquer |
|---|---|
| `write-spec` | Tourner un problème ou une idée vague en PRD structuré (problème, goals/non-goals, user stories, requirements P0/P1/P2, métriques de succès) |
| `roadmap-update` | Ajouter/retirer/reprioriser un item, déplacer une timeline, construire un roadmap Now/Next/Later ou OKR-aligned |
| `stakeholder-update` | Générer un weekly/monthly/launch update tailored par audience (exec, eng, partners, customers) |
| `synthesize-research` | Synthétiser interviews, surveys, tickets support en thèmes + personas + opportunités priorisées |
| `competitive-brief` | Analyser un compétiteur ou un area feature : matrice de comparaison, positionnement, gaps, implications stratégiques |
| `metrics-review` | Review weekly/monthly/quarterly de métriques produit : scorecard, trends, anomalies, actions recommandées |
| `sprint-planning` | Scoper un sprint : capacité, P0/P1/stretch, dépendances, risques, definition of done |
| `product-brainstorming` | Mode "thinking partner" : exploration de problèmes, idéation, test d'assumptions, exploration stratégique avec frameworks PM (HMW, JTBD, OST, First Principles, OODA, SCAMPER) |

## Stack & workflows

Conçu pour ton stack :

- **Obsidian** : note où sont les fichiers (par ex. `~/Notes/PRDs/`) et invoque le skill — Claude lira les fichiers que TU lui pointes. Le plugin ne scanne pas ton vault sans permission explicite (consigne mémoire respectée).
- **Coda** : si ton MCP Coda est connecté, les skills l'utiliseront pour pull/push des PRD. Sinon, paste le contenu.
- **MIA-Innovation** : aucun assumption sur la structure d'équipe. Adapte les templates (statut G/Y/R, formats d'update) à ta réalité MIA.

## MCP connectors (optionnels)

Le plugin source venait avec un `.mcp.json` listant Slack/Linear/Asana/Notion/Figma/Amplitude/etc. **Cette version retire ce fichier** parce qu'il forçait l'OAuth de tous ces services au moment du `/plugin install`. À la place :

- Configure les MCP que TU utilises dans ta config Claude Code globale (`~/.claude/settings.json` ou `~/.claude/mcp.json`)
- Les skills détectent à l'exécution quels MCP sont disponibles (via les placeholders `~~category` documentés dans [CONNECTORS.md](CONNECTORS.md))
- Si rien n'est connecté, les skills tombent en mode "ask user / paste content" — tout fonctionne quand même

## Crédits & licence

- Plugin source : [anthropics/knowledge-work-plugins/product-management](https://github.com/anthropics/knowledge-work-plugins/tree/main/product-management) (MIT, Anthropic)
- Adaptation Claude Code : Samuel Audette, MIT

## Voir aussi

- [CONNECTORS.md](CONNECTORS.md) — convention `~~category` pour les outils tool-agnostic
- [Plugin source officiel](https://github.com/anthropics/knowledge-work-plugins) — si tu préfères installer la version Cowork originale via `/plugin marketplace add anthropics/knowledge-work-plugins`
