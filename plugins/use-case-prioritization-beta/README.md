# use-case-prioritization-beta (v2.2)

> Scorer, prioriser et chiffrer des use cases d'AI/automatisation à partir de matériel de découverte de processus (photos de post-its, transcripts d'ateliers, inventaires CSV de 500+ lignes, notes de réunion). Combine BABOK Business Case, UiPath Suitability sur **échelle 1-3** avec **labels qualitatifs intégrés dans les cellules**, coûts fully-loaded, **Run cost benchmarké par recherche web** selon le type d'agent, et **score de confiance hybride** (complétude × validation LLM). Sortie : un **dossier projet par run** contenant le CSV de travail (38 colonnes v5), une synthèse exécutive lisible avec **layer de jugement LLM** (bundling, lectures transversales, recommandations stratégiques), et un fichier `sources_benchmark.md` auditable.

## Use case

Tu as fini un atelier de découverte (post-its, transcript, inventaire de processus) et tu dois décider **quel use case scoper en premier**, construire une **enveloppe budgétaire** crédible, ou faire le **triage d'une pipeline d'automatisation**. Le skill transforme le matériel brut en deux livrables prêts à partager : un CSV 37 colonnes pour l'analyste fonctionnel (avec lignes FORMULES et SOURCE pour traçabilité) et une synthèse markdown pour les directeurs de compte (Top 5 + sections « À éliminer » / « À retravailler »).

## Nouveautés v2 (vs v1) et v2.1 (vs v2)

**v2.0** :
- **Échelles 1-5 → 1-3** sur tous les critères qualitatifs (Suitability, Risque). Granularité plus humaine en atelier.
- **Format cellule `<chiffre> - <label>`** : la cellule contient à la fois le chiffre (pour la math) ET le label qualitatif (pour la lisibilité humaine). Exemple : `3 - récurrent standardisé`.
- **Run cost benchmarké LLM** : remplace l'heuristique `Build × 15%` par `Volume Qté × Coût Unitaire Agent`, où le $/run est récupéré par recherche web ciblée (a16z, Menlo, pricing officiels OpenAI/Anthropic) en avant-dernière étape. Source et date citées dans Notes.
- **8 colonnes supprimées** (Nb Intégrations, Volume Annuel unité, Personnes Impactées, Notes Architecte fusionnée dans Notes, Nb Colonnes Critiques, Qualité Source, Cohérence Check, Confiance Globale composite) ; **2 nouvelles** (Type Agent, Coût Unitaire Agent).

**v2.1** :
- **Nouvelle col 35 `Validation LLM`** (`1 - estimation contextuelle` / `2 - mixte` / `3 - source primaire validée`) : le LLM s'auto-évalue ligne par ligne sur la fraction de valeurs issues de la source vs estimées depuis le contexte. Justification obligatoire dans Notes.
- **Verdict Confiance hybride** : `Confiance Globale = Confiance Données × Validation_Multiplier` (1.0 / 0.75 / 0.5). Évite que toutes les lignes saturent à 100% quand le LLM remplit tous les slots avec des estimations.
- Total : **38 colonnes** au lieu de 37.

**v2.2** :
- **Dossier de sortie par run** : chaque invocation crée `priorisation_<projet>_<YYYY-MM-DD>/` avec inférence auto du nom de projet depuis les inputs (fallback date-heure si ambigu). Archivage propre, traçabilité par client.
- **Synthèse exécutive plus lisible** : prose au lieu de fragments télégraphiques. Suppression des crochets `[XX]`, pipes `|`, séparateurs `---`, symboles `≥`/`<`/`×`. Garde uniquement `%` et `$` (universels).
- **Layer de jugement LLM** sur la synthèse : le LLM peut **bundler** (ex: "Fondations Data Lake"), **promouvoir/rétrograder** un use case du ranking mécanique avec justification citée, et ajouter deux nouvelles sections (`Lectures transversales` pour les patterns inter-rows, `Recommandations stratégiques` pour l'ordre de lancement et les pré-requis). Le verdict mécanique du CSV reste affiché pour traçabilité.
- **`sources_benchmark.md`** auditable : compile toutes les sources web utilisées pour les Coût Unitaire Agent ($/run) avec date de consultation. Permet l'audit reviewer et la mise à jour trimestrielle.

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills-beta
/plugin install use-case-prioritization-beta@skills-beta
```

Mise à jour : `/plugin marketplace update skills-beta`. Désinstallation : `/plugin uninstall use-case-prioritization-beta@skills-beta`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills-beta.git
cp -R skills/plugins/use-case-prioritization-beta/skills/use-case-prioritization-beta <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Slash command Claude Code : `/use-case-prioritization-beta:use-case-prioritization-beta` (ou `/use-case-prioritization-beta` selon ta config).

Le skill détecte les mots-clés courants : `prioriser`, `prioritization`, `use case`, `ROI`, `payback`, `business case`, `suitability`, `scoping`, `enveloppe budgétaire`, `quel use case en premier`, `automation pipeline`, `process triage`, `atelier découverte`, `post-its`, `cartographie processus`, `chiffrer use case`.

## Contenu

- `skills/use-case-prioritization-beta/SKILL.md` — workflow complet (frontmatter `name` + `description` + étapes).
- `skills/use-case-prioritization-beta/references/framework_v5.md` — explication des 38 colonnes v5, sources méthodologiques (BABOK, UiPath, SAFe, DAMA-DMBOK).
- `skills/use-case-prioritization-beta/references/calculation_rules.md` — formules v5, valeurs par défaut, seuils de verdict, table de fallback benchmark.
- `skills/use-case-prioritization-beta/references/csv_template.csv` — gabarit CSV 37 colonnes avec lignes FORMULES, SOURCE et exemple use case fictif.
- `skills/use-case-prioritization-beta/references/exec_summary_template.md` — gabarit de synthèse exécutive en français Québec, avec layer de jugement LLM (bundling, lectures transversales, recommandations stratégiques).
- `skills/use-case-prioritization-beta/references/sources_benchmark_template.md` — gabarit du fichier auditable de sources web utilisées pour les benchmarks $/run.

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
