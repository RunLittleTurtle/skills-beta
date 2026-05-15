---
name: skill-creator-turtle-v1-beta
description: Meta-skill GÉNÉRIQUE et ADAPTATIF qui aide n'importe quel utilisateur à créer un skill Claude Code (ou compatible agentskills.io). Détecte automatiquement les fichiers de référence passés en argument (ex `/skill-creator-turtle-v1-beta <path1> <path2>`) et pose des questions ciblées sur l'angle/profondeur d'analyse et le rôle de chaque fichier — l'utilisateur n'a PAS à écrire "analyse profondément" en texte libre. Supporte 3 cibles — (A) marketplace personnelle multi-skills sur GitHub ou en local, (B) skill standalone perso direct dans ~/.claude/skills/ sans marketplace, (C) skill pour un autre outil (OpenCode, GitHub Copilot Enterprise, Cursor, Claude Desktop, etc.). Persiste la config dans ~/.claude/skill-creator-turtle/config.json. À invoquer quand l'utilisateur veut créer un skill, peu importe son setup ou ses intentions.
---

# skill-creator-turtle-v1-beta — Créer un nouveau skill packagé (générique, adaptatif)

## Philosophie & garanties

Skill **générique** : ne suppose RIEN sur l'identité de l'utilisateur courant. Fonctionne pour n'importe qui avec **ses** infos GitHub et **ses** chemins. Trois types couverts (choix à l'Étape 0.0) : **A** marketplace personnelle multi-skills (GitHub ou local), **B** skill standalone perso (`~/.claude/skills/<slug>/SKILL.md`, sans Git), **C** skill pour un autre outil (OpenCode, Cursor, Copilot, Claude Desktop…). Les skills GÉNÉRÉS sont eux-mêmes adaptatifs (pas de hardcoding du créateur).

**Le skill ne fait JAMAIS** : supposer un compte GitHub ; supposer Claude Code (Type C couvre les autres outils) ; écrire un chemin absolu hardcodé ; forcer l'install de `gh` ou la création d'un compte GitHub.

---

**Réponds dans la langue de l'utilisateur (français par défaut si ambigu). Sois concis. Confirme chaque action en une phrase.**

---

## Étape -1 — Détection automatique de fichiers de référence (toujours en premier)

**Cette étape s'exécute AVANT toutes les autres**, dès l'invocation du skill, pour éviter que l'utilisateur ait à écrire en texte libre des consignes du genre "analyse profondément ce fichier".

**Logique** :
1. Scanne le message initial de l'utilisateur. Si tu repères un ou plusieurs tokens qui ressemblent à un chemin de fichier ET que ce(s) fichier(s) existe(nt) sur disque (peu importe l'extension), lis `${CLAUDE_SKILL_DIR}/references/REFERENCE_DETECTION.md` et suis le workflow détaillé qui s'y trouve (lecture des fichiers → questions sur rôle/profondeur → analyse réelle → question dynamique sur les composantes à reproduire → brief injecté dans 4/6a/6c).
2. Si aucun path détecté → SKIP cette étape, va directement à l'Étape 0.0.

**Important** : la liste des composantes proposée à l'utilisateur est **DYNAMIQUE**, dérivée de l'analyse réelle des fichiers fournis (sections, blocs, patterns, conventions, style). Ce n'est jamais une liste hardcodée — un fichier `.py` produit des options différentes d'un `.md` ou d'un CSV. Les détails sont dans le fichier annexe.

---

## Étape 0.0 — Type de skill à créer (BRANCHEMENT PRINCIPAL)

**Avant tout**, demande à l'utilisateur via `AskUserQuestion` :

> "Que veux-tu faire avec ce nouveau skill ?
>
> - **A) Ajouter à ma marketplace personnelle** (recommandé pour skills partageables) : structure complète plugin marketplace, multi-skills dans un seul dépôt (`<user>/skills` ou dossier local), partage facile via `/plugin marketplace add`.
>
> - **B) Skill perso standalone** (juste pour moi, ici, maintenant) : un seul fichier `~/.claude/skills/<slug>/SKILL.md`, pas de marketplace, pas de Git, pas de partage. Utilisable immédiatement dans Claude Code.
>
> - **C) Skill pour un autre outil** (OpenCode, GitHub Copilot, Cursor, Claude Desktop, etc.) : SKILL.md au standard agentskills.io placé dans le dossier skills de l'outil cible. Tu m'indiques l'outil ou le chemin."

**Selon la réponse** :
- **Type A** → continuer à l'Étape 0.1 (config marketplace).
- **Type B** → SKIP Étapes 0.1, 1, 2, 2bis, 8, 9, 10. Aller direct à l'Étape 3 (slug). À l'Étape 7, écris UNIQUEMENT `~/.claude/skills/<slug>/SKILL.md`. Pas de plugin.json, pas de Git.
- **Type C** → SKIP les mêmes étapes. À l'Étape 7, écris UNIQUEMENT le SKILL.md dans le dossier cible (voir Étape 0.5 pour déterminer le dossier).

---

## Étape 0.5 — Dossier cible (Type C uniquement)

Si Type C, demande via `AskUserQuestion` :

> "Quel outil cible utilises-tu ? (le skill sera placé dans son dossier skills)"

Options proposées (avec dossier par défaut) :
- **OpenCode** → `~/.opencode/skills/`
- **GitHub Copilot** → variable selon config (demande à l'utilisateur ou voir [docs Copilot](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills))
- **Cursor** → variable (voir [docs Cursor](https://cursor.com/docs/context/skills))
- **Claude Desktop (macOS)** → `~/Library/Application Support/Claude/skills/`
- **Autre / chemin personnalisé** → l'utilisateur saisit le chemin via "Other"

Vérifie que le dossier existe (sinon, demande de le créer ou de corriger). Stocke dans une variable locale `<target_dir>` pour cette session — pas dans la config persistée (chaque skill peut viser un outil différent).

---

## Étape 0 — Charger ou initialiser la configuration utilisateur (Type A uniquement)

Le skill stocke la config dans `~/.claude/skill-creator-turtle/config.json`. Format :

```json
{
  "mode": "github" | "local",
  "github_user": "<login GitHub, omis en mode local>",
  "author_name": "<nom complet>",
  "author_url": "<URL profile GitHub si mode github, sinon omis>",
  "repo_name": "<nom du repo/dossier, ex: skills>",
  "repo_dir": "<chemin local absolu, ex: /Users/foo/code/skills>",
  "repo_url": "<URL git si mode github, omis en mode local>"
}
```

**Logique** :

### 0.1 — Si `~/.claude/skill-creator-turtle/config.json` existe et est valide
Charge-le, affiche un résumé ("Mode: <github|local>, repo: <Y>, dir: <Z>") et passe à l'Étape 1.

### 0.2 — Sinon, détecter le mode disponible

En parallèle :
- `command -v gh` (existe ?)
- Si oui : `gh auth status 2>&1` (authentifié ?)

**Cas A — `gh` absent OU pas authentifié** :
Demande via `AskUserQuestion` :
> "Tu n'as pas (ou pas configuré) `gh` CLI. Tu veux :
> - **Mode local** (recommandé pour toi) : créer tes skills sans GitHub, marketplace locale uniquement
> - **Configurer GitHub d'abord** : je m'arrête, tu lances `gh auth login` puis tu me ré-invoques
> - **Autre** : précise"

Si choix "local" → mettre `mode: "local"` dans la config.

**Cas B — `gh` présent et authentifié** :
Demande via `AskUserQuestion` :
> "Mode pour ta marketplace de skills :
> - **GitHub** (recommandé) : repo public/privé sur GitHub, push automatique, partageable par lien
> - **Local** : fichiers locaux uniquement, pas de push, marketplace via chemin local"

### 0.3 — Auto-détecter les autres champs

| Champ | Mode `github` | Mode `local` |
|---|---|---|
| `github_user` | `gh api user --jq .login` | (omis) |
| `author_name` | `gh api user --jq .name` (fallback `git config user.name`) | `git config user.name` (sinon demander) |
| `author_url` | `https://github.com/<github_user>` | (omis) |
| `repo_name` | suggérer `"skills"` | suggérer `"skills"` |
| `repo_dir` | suggérer `$HOME/code/<repo_name>` | suggérer `$HOME/code/<repo_name>` |
| `repo_url` | `https://github.com/<github_user>/<repo_name>.git` | (omis) |

Pose chaque choix via `AskUserQuestion` (valeur détectée = première option "Recommandée"). Permet override par "Other".

### 0.4 — Sauvegarder
Écris la config dans `~/.claude/skill-creator-turtle/config.json` (`mkdir -p ~/.claude/skill-creator-turtle` d'abord). Confirme : "Config sauvegardée (mode: <mode>). Tu peux la modifier à tout moment dans ce fichier."

**Tous les Étapes suivantes utilisent ces variables. JAMAIS de chemin ou nom hardcodé.**

---

## Étape 1 — Vérifications préalables (selon mode)

### Si `mode: "github"`
Lance en parallèle :
- `gh auth status` — l'utilisateur doit être authentifié
- Vérifier que `gh api user --jq .login` retourne bien `<github_user>` de la config (sinon, mismatch — demander quoi faire : update config ? changer de compte ?)
- `git config --global user.email` — doit retourner une valeur

Si quelque chose manque, dis exactement ce qui manque et propose des solutions (réparer, ou basculer en mode local), puis arrête ou continue selon la réponse.

### Si `mode: "local"`
Lance :
- `command -v git` — git doit être installé (sinon, dis-le et arrête : sans git on ne peut pas init un repo, mais on peut quand même créer les fichiers — demande à l'utilisateur s'il veut continuer SANS git)
- `git config --global user.name` (informatif, optionnel en mode local)

Pas besoin de `gh` ni de connexion Internet en mode local.

---

## Étape 2 — État du repo skills personnel (selon mode)

### Si `mode: "github"`
Vérifie `<repo_dir>` :

- **Cas A : `<repo_dir>` n'existe pas** :
   - Vérifie si le repo GitHub existe : `gh repo view <github_user>/<repo_name> --json name 2>/dev/null`
   - **Cas A.1 : repo GitHub existe** → `git clone <repo_url> <repo_dir>`
   - **Cas A.2 : repo GitHub n'existe pas** → bootstrap un nouveau repo (Étape 2bis)

- **Cas B : `<repo_dir>` existe** → `git -C <repo_dir> pull --ff-only`

Si le pull/clone échoue (conflits, divergence, permissions), dis exactement l'erreur et arrête.

### Si `mode: "local"`
Vérifie `<repo_dir>` :

- **Cas A : `<repo_dir>` n'existe pas** → bootstrap (Étape 2bis local).
- **Cas B : `<repo_dir>` existe avec `.claude-plugin/marketplace.json`** → utilise tel quel, pas besoin de pull.
- **Cas C : `<repo_dir>` existe mais pas de `marketplace.json`** → demande à l'utilisateur : "Le dossier existe mais ne ressemble pas à une marketplace skills. Continuer en l'utilisant quand même (création de la structure) ou choisir un autre dossier ?"

---

## Étape 2bis — Bootstrap d'un nouveau repo skills (première utilisation)

### Si `mode: "github"`

1. Confirme via `AskUserQuestion` : "Créer le repo `<github_user>/<repo_name>` (public) maintenant ?"
2. Si oui :
   - `mkdir -p <repo_dir>/.claude-plugin <repo_dir>/plugins`
   - Crée `<repo_dir>/.claude-plugin/marketplace.json`, `README.md`, `LICENSE` (templates ci-dessous)
   - `cd <repo_dir> && git init -b main && git add -A && git commit -m "Initial commit: empty <repo_name> marketplace"`
   - `gh repo create <github_user>/<repo_name> --public --description "Marketplace personnelle de skills Claude Code" --source=. --push`
3. Si non, arrête le workflow.

### Si `mode: "local"`

1. Confirme via `AskUserQuestion` : "Créer le dossier `<repo_dir>` (marketplace locale) maintenant ?"
2. Si oui :
   - `mkdir -p <repo_dir>/.claude-plugin <repo_dir>/plugins`
   - Crée `<repo_dir>/.claude-plugin/marketplace.json`, `README.md`, `LICENSE` (templates adaptés ci-dessous)
   - Optionnel (demande à l'utilisateur) : `cd <repo_dir> && git init -b main && git add -A && git commit -m "Initial commit"` (utile pour versioning local même sans GitHub)
3. Si non, arrête.

### Templates pour le bootstrap

**`marketplace.json` (mode github)** :
```json
{
  "name": "<repo_name>",
  "description": "Marketplace personnelle de skills Claude Code de <author_name>",
  "owner": {
    "name": "<author_name>",
    "url": "<author_url>"
  },
  "plugins": []
}
```

**`marketplace.json` (mode local)** : identique mais SANS `owner.url`.

**`LICENSE`** : MIT classique avec `Copyright (c) <année courante> <author_name>`.

**`README.md`** : lis `${CLAUDE_SKILL_DIR}/assets/readme-template.md`, substitue les placeholders (`<repo_name>`, `<author_name>`, `<github_user>`) par les valeurs de la config, écris le résultat dans `<repo_dir>/README.md`. En mode local, retire les sections `/plugin marketplace add <github_user>/<repo_name>` (remplace par `/plugin marketplace add <repo_dir>` chemin local).

---

## Étape 3 — Demander le slug du nouveau skill

`AskUserQuestion` avec une question texte libre (option "Other") :

> "Quel est le slug du nouveau skill ? Format kebab-case, ex: `pr-review`, `weekly-recap`, `daily-standup`."

**Validation** :
- Pattern `^[a-z][a-z0-9-]*$` (lowercase, chiffres et tirets, commence par une lettre)
- `<repo_dir>/plugins/<slug>/` ne doit pas exister

Si invalide ou déjà existant, explique pourquoi et redemande.

---

## Étape 4 — Demander la description (frontmatter)

`AskUserQuestion` avec texte libre :

> "Décris en 1-3 phrases QUAND ce skill doit être invoqué. Commence par un verbe d'action (Coordonner..., Générer..., Analyser...). Cette description sera lue par le LLM pour décider d'activer le skill."

**Si l'Étape -1 a produit un brief de référence**, propose une description **pré-rédigée** basée sur le brief (ex: "Transformer N'IMPORTE QUEL input (texte, fichier, image, idée) en document structuré comme `<nom du fichier de référence>` : sections X/Y/Z, diagramme de séquence, scénario narratif. À invoquer quand..."). L'utilisateur peut accepter telle quelle, ajuster, ou réécrire.

Garde le texte final tel quel (pas de troncature, pas de reformulation auto).

---

## Étape 5 — Mode de rédaction du body

`AskUserQuestion` avec 3 options :
- **Interactif** : poser des questions sur les étapes, je génère tout
- **Coller un brouillon** : l'utilisateur fournit le markdown complet
- **Squelette minimal** : générer un template à remplir plus tard

---

## Étape 6a — Mode interactif

**Avant de commencer**, lis `${CLAUDE_SKILL_DIR}/references/AUTHORING_GUIDELINES.md` pour avoir en tête : limites officielles (frontmatter, longueur < 500 lignes, < 5k tokens), pattern progressive disclosure, anti-patterns courants. Réfère-toi à ce fichier quand l'utilisateur demande "est-ce un bon skill ?".

**Si l'Étape -1 a produit un brief de référence**, utilise-le comme **squelette de départ** : propose à l'utilisateur les étapes correspondant aux sections / diagrammes / règles détectés, plutôt que de partir de zéro. Exemple : si le brief mentionne "sections: contexte, scénario, séquence diagrammée, matrice d'attributs", propose 4-5 étapes correspondantes et demande confirmation/ajustement avant de les détailler.

Rappelle à l'utilisateur le **principe d'adaptabilité** :

> "Ce skill que tu crées sera potentiellement installé chez d'autres. Évite de hardcoder ton nom, ton login GitHub, des chemins absolus, ou des URLs spécifiques à ton compte. Préfère des détections runtime (`gh api user`, `git config`, `$HOME`). Si le skill est volontairement perso (workflow lié à ton équipe), mentionne-le explicitement dans la description."

Puis demande successivement :

1. **Langue** de réponse (français / anglais / autre)
2. **Style** (concis / détaillé)
3. **Portée** : générique (partageable) ou personnel (mention explicite dans la description)
4. **Nombre d'étapes** (typiquement 3-7, mais sans limite stricte — adapter au workflow)
5. **Pour chaque étape** : titre + description libre (aussi détaillée que nécessaire) + commandes shell impliquées (optionnel)

Pendant la rédaction, si tu détectes des chemins/noms hardcodés, alerte : "Tu mentionnes `<X>` qui semble spécifique à ton setup. Veux-tu le rendre dynamique ?".

**Surveillance de la longueur** : si le SKILL.md en cours de construction dépasse 500 lignes, propose à l'utilisateur de scinder via le pattern progressive disclosure (créer `references/` et `assets/` dans le dossier du skill, déplacer les détails). Ne refuse pas, juste informe et propose.

Construis le body progressivement au format markdown :

```markdown
# <Nom lisible du skill>

<Phrase d'accroche reprenant la description>.

**<Instructions de style : langue + concision>**

## Étape 1 — <Titre>

<Description>

[Si commandes : bloc de code shell]

## Étape 2 — <Titre>

...
```

---

## Étape 6b — Mode "coller un brouillon"

`AskUserQuestion` avec texte libre :

> "Colle ici le contenu markdown complet du body de ton skill (sans le frontmatter `---`, juste le contenu après)."

Utilise tel quel.

---

## Étape 6c — Mode "squelette minimal"

**Cas par défaut (sans brief de référence)** :

```markdown
# <Nom du skill>

<description telle que fournie à l'étape 4>

**Réponds en français. Sois concis.**

## Étape 1 — TODO

À remplir.

## Étape 2 — TODO

À remplir.

## Étape 3 — TODO

À remplir.
```

**Si l'Étape -1 a produit un brief de référence** : remplace les `Étape N — TODO` génériques par des étapes pré-titrées correspondant aux sections / diagrammes / règles détectés (ex: `## Étape 1 — Lire et structurer l'input`, `## Étape 2 — Générer le diagramme de séquence`, `## Étape 3 — Rédiger le scénario narratif`...). Le contenu de chaque étape reste "À remplir" mais le titre guide l'utilisateur.

---

## Étape 7 — Construire les fichiers du nouveau skill (selon Type)

### Type A — Marketplace personnelle

Crée la structure dans `<repo_dir>` :

```
<repo_dir>/plugins/<slug>/
├── .claude-plugin/plugin.json
├── README.md                     ← visible sur GitHub à la racine du plugin
└── skills/<slug>/SKILL.md
```

**`<repo_dir>/plugins/<slug>/.claude-plugin/plugin.json`** :
```json
{
  "name": "<slug>",
  "description": "<description complète frontmatter>",
  "version": "1.0.0",
  "author": { "name": "<author_name>", "url": "<author_url>" },
  "license": "MIT",
  "homepage": "<https://github.com/<github_user>/<repo_name>> ou omis en mode local"
}
```

**`<repo_dir>/plugins/<slug>/README.md`** : lis `${CLAUDE_SKILL_DIR}/assets/readme-plugin-template.md`, substitue les placeholders (`<slug>`, `<description>`, `<github_user>`, `<repo_name>`, `<repo_url>`) par les valeurs de la config + des saisies (Étapes 3/4). **En mode `local`** : remplace la commande `/plugin marketplace add <github_user>/<repo_name>` par `/plugin marketplace add <repo_dir>` (chemin local absolu) et adapte/supprime le bloc `git clone` selon le remote disponible. Adapte la section « Contenu » en fonction des dossiers réellement créés dans le skill (`references/`, `assets/` présents ou pas).

**`<repo_dir>/plugins/<slug>/skills/<slug>/SKILL.md`** :
```markdown
---
name: <slug>
description: <description>
---

<body construit à l'étape 6>
```

### Type B — Skill standalone perso

Crée UNIQUEMENT :
```
~/.claude/skills/<slug>/SKILL.md
```
Pas de plugin.json, pas de marketplace.json. Le SKILL.md a juste le frontmatter `name` + `description` + body. Disponible immédiatement dans Claude Code (slash command `/<slug>`).

`mkdir -p ~/.claude/skills/<slug>` avant d'écrire.

### Type C — Skill pour un autre outil

Crée UNIQUEMENT :
```
<target_dir>/<slug>/SKILL.md
```

(`<target_dir>` vient de l'Étape 0.5). Pas de plugin.json (les autres outils ne s'en servent pas). Le SKILL.md respecte le standard agentskills.io.

`mkdir -p <target_dir>/<slug>` avant d'écrire.

---

**Pour Types B et C : SKIP les Étapes 8, 9, 10. Va directement à l'Étape 11 (récap final adapté).**

---

## Étape 8 — Mettre à jour `marketplace.json` (Type A uniquement)

Lis `<repo_dir>/.claude-plugin/marketplace.json` (JSON valide). Ajoute une nouvelle entrée à la fin du tableau `plugins` :

```json
{
  "name": "<slug>",
  "source": "./plugins/<slug>",
  "description": "<description courte, 1 phrase>"
}
```

**Format `source` important** : utilise toujours `"./plugins/<slug>"` (chemin relatif explicite). Le format court `"<slug>"` avec `metadata.pluginRoot` est rejeté par le validator actuel.

Préserve le formatting (2 espaces d'indentation, virgules correctes entre entrées). Réécris le fichier complet.

**Validation** : après écriture, lance `claude plugin validate <repo_dir>` et résous toute erreur avant de continuer.

---

## Étape 9 — Mettre à jour `README.md`

Lis `<repo_dir>/README.md`. Trouve la section `## Skills disponibles` et son tableau. Ajoute une nouvelle ligne :

```
| `<slug>` | <description courte 1 phrase> | <use case 1 phrase> |
```

Si la section ou le tableau n'existent pas (par ex. README édité manuellement), dis-le à l'utilisateur et demande comment procéder (créer la section, ignorer, autre).

---

## Étape 10 — Commit (et push si applicable) ?

### Si `mode: "github"`
`AskUserQuestion` :
- **Oui, commit + push maintenant** :
  ```bash
  cd <repo_dir> && git add -A && git commit -m "Add <slug> skill" && git push
  ```
- **Non, plus tard** : affiche les commandes manuelles.

### Si `mode: "local"`
`AskUserQuestion` :
- **Oui, commit local** (si le repo a un `.git`) :
  ```bash
  cd <repo_dir> && git add -A && git commit -m "Add <slug> skill"
  ```
- **Non, juste laisser les fichiers** : pas de commit, l'utilisateur gère git lui-même.

Aucun push possible en mode local — c'est par design, pas un bug.

---

## Étape 11 — Récap final

Affiche `✓ Skill <slug> créé.` puis le bloc adapté au type :

**Type A (marketplace, mode `github` pushé)** :
```
Fichiers : <repo_dir>/plugins/<slug>/
GitHub   : https://github.com/<github_user>/<repo_name>/tree/main/plugins/<slug>
Install  : /plugin marketplace add <github_user>/<repo_name>  →  /plugin marketplace update <repo_name>  →  /plugin install <slug>@<repo_name>
Slash    : /<slug>:<slug>
```

**Type A — variantes** : si pas pushé, ajoute `À faire : cd <repo_dir> && git add -A && git commit -m "Add <slug> skill" && git push`. En mode `local`, remplace l'URL GitHub par `Install : /plugin marketplace add <repo_dir>` (chemin local, pas owner/repo) et mentionne le partage par zip + `/plugin marketplace add <chemin>` chez le destinataire.

**Type B (standalone perso)** : `Fichier : ~/.claude/skills/<slug>/SKILL.md` ; slash `/<slug>` dispo immédiatement ; édition directe du fichier, suppression `rm -rf ~/.claude/skills/<slug>`.

**Type C (autre outil)** : `Fichier : <target_dir>/<slug>/SKILL.md` (format standard agentskills.io) ; activation selon la doc de l'outil (souvent auto au prochain démarrage) ; partage = envoie le dossier ou le SKILL.md.

---

## Fichiers annexes (progressive disclosure)

Ce skill suit le pattern **progressive disclosure** recommandé par agentskills.io. Le SKILL.md contient le workflow ; les détails et templates vivent dans des fichiers annexes lus à la demande :

| Fichier | Quand le lire |
|---|---|
| `${CLAUDE_SKILL_DIR}/assets/readme-template.md` | Phase de bootstrap (Étape 2bis) — template du README généré pour le repo de marketplace |
| `${CLAUDE_SKILL_DIR}/assets/readme-plugin-template.md` | Étape 7 Type A — template du `README.md` généré à la racine de chaque nouveau plugin (visible sur GitHub quand on navigue dans `plugins/<slug>/`) |
| `${CLAUDE_SKILL_DIR}/references/AUTHORING_GUIDELINES.md` | Quand tu guides l'utilisateur dans la rédaction du body (Étape 6a/c), ou quand l'utilisateur veut vérifier la qualité d'un skill existant. Contient les limites officielles (frontmatter, longueur), exemples bons/mauvais, anti-patterns. |
| `${CLAUDE_SKILL_DIR}/references/REFERENCE_DETECTION.md` | Étape -1 — quand l'invocation contient des chemins de fichiers existants. Workflow détaillé : scan → lecture → questions sur rôle/profondeur → **analyse réelle des fichiers** → liste DYNAMIQUE de composantes à reproduire → brief injecté dans 4/6a/6c. |

`${CLAUDE_SKILL_DIR}` est la variable Claude Code qui pointe vers le dossier de ce skill.

---

## Règles d'exécution (à respecter strictement)

| Règle | Pourquoi |
|---|---|
| Pas de chemin/nom hardcodé — tout passe par les variables de config (Étape 0) | Le skill doit marcher chez n'importe qui |
| Pas de symlink. Distribution via `/plugin install`, jamais via `ln -s` | Bugs cross-machine |
| Pas de `gh repo create` sauf bootstrap (Étape 2bis) | Tout va dans le mono-repo |
| Ne touche QUE aux fichiers du nouveau skill + le `marketplace.json` global + le `README.md` global du repo (Type A) | Pas de side-effects sur les autres skills |
| Frontmatter SKILL.md = `name` + `description` UNIQUEMENT | Standard agentskills.io ; pas de `version`/`tags` non standard |
| SKILL.md généré : viser < 500 lignes ; si plus, scinder via `references/` + `assets/` | Recommandation officielle Anthropic + agentskills.io |
| Si erreur (pull conflict, JSON invalide, slug en doublon, push refusé) → arrête et dis exactement ce qui s'est passé | Ne pas réparer en silence |
| Si `gh api user --jq .login` ≠ config GitHub user → prévenir avant de continuer | Détection changement de compte |
