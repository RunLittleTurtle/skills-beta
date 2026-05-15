# skills-beta — Marketplace BETA personnelle de Samuel Audette

> **AVERTISSEMENT — Usage personnel, versions instables**
>
> Cette marketplace contient des skills en cours de développement, des refontes expérimentales, et des versions parallèles non éprouvées de skills publiés sur [`RunLittleTurtle/skills`](https://github.com/RunLittleTurtle/skills). Toute entrée ici peut casser, changer de nom, ou disparaître à tout moment. Quand une version beta est éprouvée, elle est promue vers `skills` et retirée d'ici (ou laissée comme archive).

## Installation (en parallèle de la stable)

```
/plugin marketplace add RunLittleTurtle/skills-beta
/plugin install skill-creator-beta@skills-beta
```

Les deux marketplaces cohabitent sans conflit grâce au suffixe `-beta` dans les slugs (`skill-creator` côté stable, `skill-creator-beta` ici). Les slash commands `/skill-creator` et `/skill-creator-beta` sont distincts et utilisables en parallèle pour comparer.

## Skills actuellement en beta

| Nom | Stable correspondant | Statut |
|---|---|---|
| `skill-creator-beta` | `skill-creator` | refonte alignée principes Anthropic + ajout workflow modification |

## Workflow de promotion vers stable

Quand un skill beta est éprouvé :

1. Copier le contenu de `skills-beta/plugins/<slug>-beta/` vers `skills/plugins/<slug>/` (remplaçant l'ancien).
2. Renommer le dossier interne `skills/<slug>-beta/` → `skills/<slug>/`.
3. Patcher le frontmatter `name:` du SKILL.md (`<slug>-beta` → `<slug>`).
4. Patcher `plugin.json` (name + homepage).
5. Mettre à jour `marketplace.json` et `README.md` du repo stable.
6. Vérifier qu'aucune mention résiduelle de `<slug>-beta` ne traîne : `grep -r "<slug>-beta" stable/plugins/<slug>/`.
7. Commit + push stable.

Le skill `<slug>-beta` peut rester dans la marketplace beta comme archive ou être retiré au prochain cycle.

## Licence

MIT — voir [LICENSE](./LICENSE).
