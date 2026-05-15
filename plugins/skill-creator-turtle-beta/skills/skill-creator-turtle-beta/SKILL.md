---
name: skill-creator-turtle-beta
description: Créer un nouveau skill Claude Code OU modifier un skill existant, dans une marketplace personnelle (Type A), en standalone perso (Type B), ou pour un autre outil compatible agentskills.io (Type C). Détecte automatiquement les fichiers de référence passés en argument et les skills installés sur la machine. Préserve le slug d'origine lors d'une modification, snapshot le skill dans /tmp/ avant édition. Persiste la config dans ~/.claude/skill-creator-turtle/config.json (clés stable + beta_*). À invoquer quand l'utilisateur veut créer ou modifier un skill, peu importe son setup ou ses intentions.
---

# skill-creator-turtle-beta — Créer ou modifier un skill (version beta alignée Anthropic)

Cette version étend le skill-creator-turtle stable avec un workflow de **modification** d'un skill existant (préserve le slug, snapshot, édition guidée) et adopte les principes Anthropic : prose explicative plutôt que tables ALL-CAPS, theory of mind plutôt que directives rigides, lean instructions plutôt que sur-spécification.

**Réponds dans la langue de l'utilisateur (français par défaut si ambigu). Sois concis. Confirme chaque action en une phrase.**

---

## Principes

Ce skill ne suppose rien sur l'identité de l'utilisateur courant. Il marche pour n'importe qui avec sa propre config GitHub et ses propres chemins. Trois cibles couvertes en création (A, B, C) plus la modification (D) des skills déjà installés sur la machine.

**La description du frontmatter est le trigger primary**. C'est elle qui dit à Claude *quoi* le skill fait et *quand* l'invoquer. Si Claude ne déclenche pas un skill correctement, le réflexe est de revoir la description avant de revoir la logique. Anthropic le résume ainsi : *"When to trigger, what it does. This is the primary triggering mechanism — include both what the skill does AND specific contexts for when to use it."*

**Progressive disclosure** : ce SKILL.md vise moins de 500 lignes. Les workflows longs (détection de fichiers de référence, modification d'un skill existant) vivent dans `references/`, lus à la demande seulement quand l'étape concernée se déclenche.

**Theory of mind plutôt qu'ALL-CAPS** : Claude répond mieux à du raisonnement qu'à des directives rigides. Quand ce skill impose une garde-fou (par exemple « préserver le slug d'origine lors d'une modification »), il en explique la raison (le slug identifie le skill côté Claude Code, le renommer casse les configs des autres utilisateurs). Cette logique vit dans la section « Pourquoi ces choix » à la fin.

**Ce skill ne fait jamais** : supposer un compte GitHub (mode local disponible), supposer Claude Code (Type C couvre les autres outils), hardcoder un chemin absolu, renommer un skill lors d'une modification.

---

## Étape -1 — Détection automatique de fichiers de référence (toujours en premier)

Cette étape s'exécute avant toutes les autres, dès l'invocation du skill, pour éviter que l'utilisateur ait à écrire en texte libre des consignes du genre « analyse profondément ce fichier ».

Scanne le message initial. Si tu repères un ou plusieurs tokens qui ressemblent à un chemin de fichier et que ce(s) fichier(s) existe(nt) sur disque (peu importe l'extension), lis `${CLAUDE_SKILL_DIR}/references/REFERENCE_DETECTION.md` et suis le workflow détaillé qui s'y trouve : lecture → questions sur rôle/profondeur → analyse réelle → question dynamique sur les composantes à reproduire → brief injecté dans les Étapes 4, 6a, 6c (création) ou M.5c (modification, mode régénérer).

Si aucun path détecté, passe directement à l'Étape 0.0.

La liste des composantes proposée à l'utilisateur est dérivée de l'analyse réelle des fichiers fournis. Un fichier `.py` produit des options différentes d'un `.md` ou d'un CSV. Les détails sont dans le fichier annexe.

---

## Étape 0.0 — Que veux-tu faire ? (BRANCHEMENT PRINCIPAL)

Demande via `AskUserQuestion` :

> "Que veux-tu faire ?
>
> - **A) Créer un skill dans une marketplace personnelle** (recommandé pour skills partageables) : structure plugin marketplace complète, multi-skills, partage via `/plugin marketplace add`.
>
> - **B) Créer un skill standalone perso** (juste pour moi, ici, maintenant) : un seul fichier `~/.claude/skills/<slug>/SKILL.md`, pas de marketplace, pas de Git, utilisable immédiatement.
>
> - **C) Créer un skill pour un autre outil** (OpenCode, GitHub Copilot, Cursor, Claude Desktop, etc.) : SKILL.md au standard agentskills.io placé dans le dossier de l'outil cible.
>
> - **D) Modifier un skill existant** (le mien ou un installé) : je liste les skills détectés sur la machine, tu choisis lequel, je snapshot avant édition, on modifie ensemble. Je préserve le slug d'origine."

### Selon la réponse
- **A** → continuer à l'Étape 0.1 (config marketplace stable ou beta).
- **B** → SKIP Étapes 0.1, 1, 2, 2bis, 8, 9, 10. Aller direct à l'Étape 3 (slug). À l'Étape 7, écris uniquement `~/.claude/skills/<slug>/SKILL.md`.
- **C** → SKIP les mêmes étapes. Aller à l'Étape 0.5 (déterminer dossier cible). À l'Étape 7, écris uniquement le SKILL.md dans le dossier cible.
- **D** → lis `${CLAUDE_SKILL_DIR}/references/MODIFICATION_WORKFLOW.md` et suis les étapes M.1 à M.7. Reviens ici seulement pour le récap final (Étape 11 adapté).

---

## Étape 0.5 — Dossier cible (Type C uniquement)

Si Type C, demande via `AskUserQuestion` :

> "Quel outil cible utilises-tu ? (le skill sera placé dans son dossier skills)"

Options par défaut :
- **OpenCode** → `~/.opencode/skills/`
- **GitHub Copilot** → variable (voir [docs Copilot](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills))
- **Cursor** → variable (voir [docs Cursor](https://cursor.com/docs/context/skills))
- **Claude Desktop (macOS)** → `~/Library/Application Support/Claude/skills/`
- **Autre / chemin personnalisé** → saisie via "Other"

Vérifie que le dossier existe (sinon propose de le créer). Stocke dans `<target_dir>` pour cette session, pas dans la config persistée (chaque skill peut viser un outil différent).

---

## Étape 0 — Charger ou initialiser la configuration (Type A uniquement)

La config vit dans `~/.claude/skill-creator-turtle/config.json` et supporte deux contextes (stable et beta) :

```json
{
  "mode": "github" | "local",
  "github_user": "<login>",
  "author_name": "<nom complet>",
  "author_url": "<URL profile>",

  "repo_name": "skills",
  "repo_dir": "/Users/.../code/skills",
  "repo_url": "https://github.com/<login>/skills.git",

  "beta_repo_name": "skills-beta",
  "beta_repo_dir": "/Users/.../code/skills-beta",
  "beta_repo_url": "https://github.com/<login>/skills-beta.git"
}
```

### 0.1 — Choix du contexte (stable vs beta)

Le skill `skill-creator-turtle-beta` détecte son propre contexte via son `name:` (suffixe `-beta` → contexte beta par défaut). Mais l'utilisateur peut vouloir créer un skill stable depuis l'invocation beta (par exemple ajouter directement à la marketplace stable sans passer par beta). Demande via `AskUserQuestion` :

> "Dans quelle marketplace ajouter ce skill ?
>
> - **Beta** (`<beta_repo_name>`, recommandé pour une nouvelle création — tu pourras promouvoir vers stable plus tard) : `<beta_repo_dir>`
> - **Stable** (`<repo_name>`, partage public direct) : `<repo_dir>`"

Stocke le choix dans une variable locale `<context>` (valeurs `"beta"` ou `"stable"`). Le reste de l'Étape 0 utilise les variables correspondantes (`repo_name`/`beta_repo_name`, etc.) selon ce choix.

### 0.2 — Si la config existe et est valide

Charge `~/.claude/skill-creator-turtle/config.json`, vérifie que les clés requises pour le contexte choisi sont présentes. Si oui, affiche un résumé (« Mode : <github|local>, repo <context> : <repo_name>, dir : <repo_dir> ») et passe à l'Étape 1.

Si la config existe mais que les clés `beta_*` manquent alors que `<context> = "beta"`, va à 0.3 pour les remplir.

### 0.3 — Sinon, détecter le mode et auto-remplir

En parallèle : `command -v gh` et (si présent) `gh auth status`.

**Cas A — `gh` absent ou pas authentifié** : propose mode `local` (recommandé) ou configurer `gh` d'abord.

**Cas B — `gh` présent et authentifié** : propose mode `github` (recommandé) ou `local`.

Auto-remplir les champs via `gh api user` et `git config`. Pour les champs spécifiques au contexte choisi :

| Champ | Mode github | Mode local |
|---|---|---|
| `<context>_name` | suggérer `"skills"` ou `"skills-beta"` selon contexte | idem |
| `<context>_dir` | suggérer `$HOME/code/<context>_name` | idem |
| `<context>_url` | `https://github.com/<login>/<context>_name.git` | (omis) |

Pose chaque choix via `AskUserQuestion` (valeur détectée = première option « Recommandée »).

### 0.4 — Sauvegarder

Écris la config dans `~/.claude/skill-creator-turtle/config.json` (mkdir d'abord). Confirme : « Config sauvegardée. Tu peux la modifier à tout moment dans ce fichier. »

Tous les Étapes suivantes utilisent ces variables avec le préfixe du contexte choisi. Aucun chemin ou nom hardcodé.

---

## Étape 1 — Vérifications préalables

### Mode github
En parallèle : `gh auth status`, vérifier que `gh api user --jq .login` retourne bien `<github_user>` de la config (sinon mismatch — demande quoi faire), et `git config --global user.email` non vide.

Si quelque chose manque, dis exactement ce qui manque et propose des solutions (réparer ou basculer en mode local), puis arrête ou continue selon la réponse.

### Mode local
Vérifier `command -v git`. Sans git on ne peut pas init un repo, mais on peut quand même créer les fichiers — demande à l'utilisateur s'il veut continuer sans git. Pas besoin de `gh` ni de connexion Internet.

---

## Étape 2 — État du repo (selon mode et contexte)

Soit `<dir>` = `<context>_dir` (le repo correspondant au contexte choisi à l'Étape 0.1).

### Mode github
- Si `<dir>` n'existe pas :
  - `gh repo view <github_user>/<context>_name --json name 2>/dev/null` pour vérifier l'existence du repo distant
  - Si oui → `git clone <context>_url <dir>`
  - Si non → bootstrap (Étape 2bis)
- Si `<dir>` existe → `git -C <dir> pull --ff-only`

Si pull/clone échoue (conflits, divergence, permissions), arrête et affiche l'erreur. La raison : réparer en silence cache des problèmes structurels.

### Mode local
- Si `<dir>` n'existe pas → bootstrap (Étape 2bis local).
- Si `<dir>` existe avec `.claude-plugin/marketplace.json` → utilise tel quel.
- Si `<dir>` existe sans `marketplace.json` → demande à l'utilisateur (continuer ou choisir un autre dossier).

---

## Étape 2bis — Bootstrap d'un nouveau repo (première utilisation du contexte)

### Mode github

1. Confirmer via `AskUserQuestion` : « Créer le repo `<github_user>/<context>_name` (public) maintenant ? »
2. Si oui :
   - `mkdir -p <dir>/.claude-plugin <dir>/plugins`
   - Crée `<dir>/.claude-plugin/marketplace.json`, `README.md`, `LICENSE` (templates ci-dessous, adaptés au contexte beta si applicable — banner BETA dans le README).
   - `cd <dir> && git init -b main && git add -A && git commit -m "Initial commit: empty <context>_name marketplace"`
   - `gh repo create <github_user>/<context>_name --public --description "<description adaptée au contexte>" --source=. --push`

### Mode local

1. Confirmer : « Créer le dossier `<dir>` (marketplace locale) maintenant ? »
2. Si oui : `mkdir -p` + fichiers comme ci-dessus. Optionnel : `git init` pour versioning local.

### Templates

**`marketplace.json` (contexte beta)** : ajoute la mention « BETA — versions instables » dans la description top-level.

**`marketplace.json` (contexte stable)** : description neutre comme dans le stable existant.

**`LICENSE`** : MIT avec `Copyright (c) <année courante> <author_name>`.

**`README.md`** : lis `${CLAUDE_SKILL_DIR}/assets/readme-template.md`, substitue les placeholders. Pour le contexte beta, ajoute en tête le banner d'avertissement « Usage personnel, versions instables » (voir le README du skills-beta de ce repo comme modèle).

---

## Étape 3 — Slug du nouveau skill

`AskUserQuestion` avec texte libre :

> "Quel est le slug du nouveau skill ? Format kebab-case, ex: `pr-review`, `weekly-recap`. Si tu crées une version expérimentale d'un skill existant, suffixe `-beta` ou `-v2` (cohabite avec l'original)."

Validation : pattern `^[a-z][a-z0-9-]*$`, et `<dir>/plugins/<slug>/` ne doit pas exister. Sinon explique et redemande.

---

## Étape 4 — Description du frontmatter (trigger primary)

`AskUserQuestion` avec texte libre :

> "Décris en 1-3 phrases QUAND ce skill doit être invoqué. Commence par un verbe d'action (Coordonner..., Générer..., Analyser...). Cette description est lue par Claude pour décider d'activer le skill — c'est le trigger primary, fais-la spécifique."

Si l'Étape -1 a produit un brief de référence, propose une description **pré-rédigée** basée sur le brief (ex: « Transformer N'IMPORTE QUEL input en document structuré comme `<nom du fichier de référence>` : sections X/Y/Z, diagramme de séquence, scénario narratif. À invoquer quand... »). L'utilisateur accepte, ajuste ou réécrit.

Garde le texte final tel quel (pas de troncature, pas de reformulation auto).

---

## Étape 5 — Mode de rédaction du body

`AskUserQuestion` avec 3 options :
- **Interactif** : poser des questions sur les étapes, le skill génère tout.
- **Coller un brouillon** : l'utilisateur fournit le markdown complet.
- **Squelette minimal** : générer un template à remplir plus tard.

---

## Étape 6a — Mode interactif

Avant de commencer, lis `${CLAUDE_SKILL_DIR}/references/AUTHORING_GUIDELINES.md` pour avoir en tête les limites officielles (frontmatter, < 500 lignes, < 5k tokens), le pattern progressive disclosure, et les principes Anthropic (lean instructions, theory of mind).

Si l'Étape -1 a produit un brief de référence, utilise-le comme squelette de départ : propose les étapes correspondant aux sections / diagrammes / règles détectés, plutôt que de partir de zéro.

Rappelle à l'utilisateur le principe d'adaptabilité :

> "Ce skill sera potentiellement installé chez d'autres. Évite de hardcoder ton nom, ton login GitHub, des chemins absolus, ou des URLs spécifiques à ton compte. Préfère des détections runtime (`gh api user`, `git config`, `$HOME`). Si le skill est volontairement perso, mentionne-le explicitement dans la description."

Demande ensuite : langue, style (concis / détaillé), portée (générique ou personnel), nombre d'étapes (typiquement 3-7), puis pour chaque étape : titre + description libre + commandes shell impliquées (optionnel).

Pendant la rédaction, si tu détectes des chemins/noms hardcodés, alerte. Si le SKILL.md en construction dépasse 500 lignes, propose progressive disclosure (créer `references/` et `assets/`, déplacer les détails).

---

## Étape 6b — Mode "coller un brouillon"

`AskUserQuestion` avec texte libre :

> "Colle ici le contenu markdown complet du body (sans le frontmatter `---`)."

Utilise tel quel.

---

## Étape 6c — Mode "squelette minimal"

Par défaut :

```markdown
# <Nom du skill>

<description fournie à l'Étape 4>

**Réponds en français. Sois concis.**

## Étape 1 — TODO

À remplir.

## Étape 2 — TODO

À remplir.

## Étape 3 — TODO

À remplir.
```

Si l'Étape -1 a produit un brief, remplace les `Étape N — TODO` par des étapes pré-titrées correspondant aux composantes détectées.

---

## Étape 7 — Construire les fichiers du nouveau skill

### Type A — Marketplace personnelle

Structure dans `<dir>` :

```
<dir>/plugins/<slug>/
├── .claude-plugin/plugin.json
├── README.md
└── skills/<slug>/SKILL.md
```

**`plugin.json`** : `name`, `description` (frontmatter complet), `version: "1.0.0"`, `author` (config), `license: "MIT"`, `homepage` (URL repo en mode github, omis en mode local).

**`README.md`** : lis `${CLAUDE_SKILL_DIR}/assets/readme-plugin-template.md`, substitue les placeholders (`<slug>`, `<description>`, `<github_user>`, `<context>_name`, `<context>_url`). En mode `local`, remplace `/plugin marketplace add <github_user>/<context>_name` par `/plugin marketplace add <dir>` (chemin absolu) et adapte le bloc `git clone`. Adapte la section « Contenu » selon les dossiers réellement créés (`references/`, `assets/`).

**`SKILL.md`** : frontmatter `name: <slug>` + `description: <description>` + body construit à l'Étape 6.

### Type B — Skill standalone perso

Crée uniquement `~/.claude/skills/<slug>/SKILL.md` (mkdir d'abord). Pas de plugin.json, pas de marketplace.json. Slash command `/<slug>` disponible immédiatement.

### Type C — Skill pour un autre outil

Crée uniquement `<target_dir>/<slug>/SKILL.md` (mkdir d'abord). Standard agentskills.io. Pas de plugin.json (les autres outils ne s'en servent pas).

**Pour Types B et C : SKIP les Étapes 8, 9, 10. Va directement à l'Étape 11.**

---

## Étape 8 — Mettre à jour `marketplace.json` (Type A uniquement)

Lis `<dir>/.claude-plugin/marketplace.json`. Ajoute une entrée à la fin du tableau `plugins` :

```json
{
  "name": "<slug>",
  "source": "./plugins/<slug>",
  "description": "<description courte, 1 phrase>"
}
```

Format `source` important : utilise toujours `"./plugins/<slug>"` (chemin relatif explicite). Le format court `"<slug>"` avec `metadata.pluginRoot` est rejeté par le validator actuel.

Préserve le formatting (2 espaces, virgules correctes). Réécris le fichier complet. Après écriture, lance `claude plugin validate <dir>` et résous toute erreur avant de continuer.

---

## Étape 9 — Mettre à jour `README.md`

Lis `<dir>/README.md`. Trouve la section `## Skills disponibles` (ou équivalent) et son tableau. Ajoute une ligne :

```
| `<slug>` | <description courte 1 phrase> | <use case 1 phrase> |
```

Si la section ou le tableau n'existent pas, dis-le à l'utilisateur et demande comment procéder.

---

## Étape 10 — Commit et push ?

### Mode github
`AskUserQuestion` :
- **Oui, commit + push maintenant** : `cd <dir> && git add -A && git commit -m "Add <slug> skill" && git push`
- **Non, plus tard** : affiche les commandes manuelles.

### Mode local
- **Oui, commit local** (si `.git` présent) : `cd <dir> && git add -A && git commit -m "Add <slug> skill"`
- **Non, juste laisser les fichiers**.

Aucun push possible en mode local — par design.

---

## Étape 11 — Récap final

Affiche `✓ Skill <slug> créé.` puis le bloc adapté :

**Type A, mode github pushé** :
```
Fichiers : <dir>/plugins/<slug>/
GitHub   : https://github.com/<github_user>/<context>_name/tree/main/plugins/<slug>
Install  : /plugin marketplace add <github_user>/<context>_name  →  /plugin marketplace update <context>_name  →  /plugin install <slug>@<context>_name
Slash    : /<slug>:<slug>
```

**Type A, pas pushé** : ajoute `À faire : cd <dir> && git add -A && git commit -m "..." && git push`.

**Type A, mode local** : remplace l'URL GitHub par `Install : /plugin marketplace add <dir>` et mentionne partage par zip + `marketplace add <chemin>` chez le destinataire.

**Type B** : `Fichier : ~/.claude/skills/<slug>/SKILL.md` ; slash `/<slug>` dispo ; suppression `rm -rf ~/.claude/skills/<slug>`.

**Type C** : `Fichier : <target_dir>/<slug>/SKILL.md` ; activation selon doc outil cible (souvent auto au prochain démarrage).

**Type D (modification)** : Le récap est géré par `MODIFICATION_WORKFLOW.md` (M.7) — il inclut le path snapshot + commande rollback + diff résumé + (Type A) commande `/plugin update`.

---

## Sous-workflow MODIFY (Étape 0.0 = D)

Quand l'utilisateur a choisi « Modifier un skill existant », lis `${CLAUDE_SKILL_DIR}/references/MODIFICATION_WORKFLOW.md` et suis les étapes M.1 à M.7 :

- M.1 : détecter les skills modifiables (standalones, marketplaces locales, copies installées en cache)
- M.2 : présenter la liste, demander lequel modifier
- M.3 : snapshot dans `/tmp/<slug>-snapshot-<timestamp>/`
- M.4 : lire le skill complet (SKILL.md + references/ + assets/ + plugin.json si Type A)
- M.5 : mode de modification (interactif diff par section / coller nouveau body / régénérer à partir d'un brief de référence)
- M.6 : validation structurelle (frontmatter, slug invariant, taille < 500 lignes)
- M.7 : commit + push (Type A) ou enregistrer fichier (Type B/C) + récap avec snapshot + diff

Le slug d'origine est préservé sauf confirmation explicite contraire de l'utilisateur (règle Anthropic).

---

## Fichiers annexes (progressive disclosure)

| Fichier | Quand le lire |
|---|---|
| `${CLAUDE_SKILL_DIR}/references/REFERENCE_DETECTION.md` | Étape -1 — invocation contenant des paths de fichiers existants. Workflow scan → lecture → analyse → composantes dynamiques → brief. |
| `${CLAUDE_SKILL_DIR}/references/AUTHORING_GUIDELINES.md` | Étape 6a/6c — quand tu guides la rédaction du body, ou quand l'utilisateur veut vérifier la qualité d'un skill. Contient limites officielles, principes Anthropic, exemples bons/mauvais. |
| `${CLAUDE_SKILL_DIR}/references/MODIFICATION_WORKFLOW.md` | Étape 0.0 = D — sous-workflow M.1 à M.7 pour modifier un skill existant. |
| `${CLAUDE_SKILL_DIR}/assets/readme-template.md` | Étape 2bis — bootstrap d'un nouveau repo de marketplace. |
| `${CLAUDE_SKILL_DIR}/assets/readme-plugin-template.md` | Étape 7 Type A — README généré à la racine du plugin. |

`${CLAUDE_SKILL_DIR}` est la variable Claude Code qui pointe vers le dossier de ce skill.

---

## Pourquoi ces choix

### Adaptabilité avant tout
Aucun chemin ou nom n'est hardcodé : tout passe par la config (Étape 0). La raison : ce skill est installable chez d'autres utilisateurs qui ont leur propre login GitHub, leur propre arborescence, leur propre nom. Si tu vois un chemin absolu dans le code généré, c'est probablement une fuite — signale-le.

### Pas de symlinks
Distribution via `/plugin install`, jamais via `ln -s`. Les symlinks introduisent des bugs cross-machine subtils (paths qui marchent en dev mais cassent à l'install). Si tu as besoin de partager du code entre skills, duplique-le ou extrais-le dans un fichier de référence partagé.

### Frontmatter strict (`name` + `description` uniquement)
C'est le standard agentskills.io et le validator Anthropic refuse les champs custom (`version`, `tags`, `keywords`). Si tu veux versionner, mets-le dans `plugin.json` à la racine du plugin, pas dans le SKILL.md.

### Préserver le slug en modify
Quand l'utilisateur demande de modifier `research-helper`, le fichier mis à jour reste `research-helper` (pas `research-helper-v2`). La raison : le slug identifie le skill côté Claude Code, et le renommer casse les liens, les configs, et les scripts des autres utilisateurs qui l'ont installé. Si on veut vraiment une nouvelle identité, c'est une création (Étape 0.0 = A), pas une modification.

### Snapshot avant toute édition
M.3 copie le skill complet vers `/tmp/<slug>-snapshot-<timestamp>/`. La raison : si la modification casse quelque chose, rollback en une commande (`cp -R /tmp/<slug>-snapshot-<ts>/* <original-path>/`). `/tmp/` est purgé au reboot — si tu veux persister, copie ailleurs avant.

### Détection des skills installés multi-source
M.1 scanne quatre familles d'emplacements (standalones, marketplaces locales stable, marketplaces locales beta, copies en cache `/plugin install`). La raison : les sources éditables (les deux premières) doivent être priorisées sur les copies read-only en cache, qui seront écrasées au prochain `/plugin update`. Le skill redirige vers la source quand elle existe localement, propose un clone sinon.

### Limite < 500 lignes pour le SKILL.md
Recommandation officielle Anthropic et agentskills.io. Au-delà, le coût en tokens à chaque invocation grimpe, et la maintenabilité descend. Quand un workflow gonfle (cas de la modification dans ce skill), on l'externalise dans `references/` lu à la demande — c'est exactement le pattern de progressive disclosure.

### Arrêter sur erreur, pas réparer en silence
Si `git pull` échoue, si le JSON est invalide, si le slug est déjà pris : on s'arrête et on dit exactement ce qui s'est passé. Réparer en silence cache des problèmes structurels (mauvais compte GitHub, conflit en attente, divergence de remote) qui resurgissent plus tard plus coûteux à diagnostiquer.

### Mismatch de compte GitHub détecté
Si `gh api user --jq .login` ne correspond pas au `github_user` de la config, le skill prévient avant de continuer. La raison : pousser depuis le mauvais compte crée un commit avec une auteur incorrect et peut révéler une identité non voulue.
