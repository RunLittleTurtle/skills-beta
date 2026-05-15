# Modification d'un skill existant — workflow M.1 à M.7

À lire quand l'utilisateur a choisi « Modifier un skill existant » à l'Étape 0.0 du SKILL.md principal. Décrit le sous-workflow complet pour modifier un skill en préservant son slug d'origine et en gardant un snapshot de rollback.

Principe central : on modifie toujours la **source** d'un skill (un fichier que l'utilisateur peut éditer), jamais la **copie installée en cache** (qui sera écrasée au prochain `/plugin update`). Quand la source n'est pas accessible localement, le skill propose de cloner d'abord.

---

## M.1 — Détecter tous les skills modifiables sur la machine

Scanner quatre familles d'emplacements en parallèle, en lecture seule :

### M.1.1 — Skills standalones perso

```bash
ls -d ~/.claude/skills/*/  2>/dev/null
```

Chaque dossier qui contient un `SKILL.md` est un standalone. Éditable directement.

### M.1.2 — Skills dans marketplaces locales clonées

Lis la config (`~/.claude/skill-creator-turtle/config.json`) pour récupérer les `repo_dir` connus :
- `repo_dir` (stable, ex: `~/code/skills`)
- `beta_repo_dir` (beta, ex: `~/code/skills-beta`)

Pour chaque `<repo_dir>` existant :
```bash
ls -d <repo_dir>/plugins/*/skills/*/  2>/dev/null
```

Chaque dossier qui contient un `SKILL.md` est un skill marketplace local. Éditable. Source de vérité pour le `/plugin install` correspondant.

### M.1.3 — Skills installés via /plugin install (copies read-only)

```bash
ls -d ~/.claude/plugins/cache/*/plugins/*/skills/*/  2>/dev/null
ls -d ~/.claude/plugins/marketplaces/*/plugins/*/skills/*/  2>/dev/null
```

Ces emplacements contiennent les copies installées des plugins. Read-only : ne pas éditer directement (écrasé au prochain update). Affichées en M.2 avec note « (installé, source dans repo X) » si on peut résoudre la source localement.

### M.1.4 — Extraction des métadonnées

Pour chaque SKILL.md trouvé, extrais via Read :
- `name:` du frontmatter
- Première ligne de `description:` (tronquée à ~100 chars pour l'affichage)
- Chemin absolu du dossier du skill (parent du SKILL.md)
- Famille parmi : `standalone`, `marketplace-local-stable`, `marketplace-local-beta`, `installed-cache`, `bundled-marketplace`

Pour les skills en cache/bundled, tente de résoudre la source : si le nom de la marketplace dans le path matche un `repo_name` connu de la config, propose la source correspondante (`<repo_dir>/plugins/<slug>/`).

---

## M.2 — Présenter la liste et demander lequel modifier

`AskUserQuestion` avec la liste consolidée, regroupée par famille. Si plus de ~10 entrées, demande d'abord de filtrer par famille ou par mot-clé pour réduire la liste.

Format d'affichage :

```
<slug> (<famille>) — <description courte>
Path : <chemin>
```

### Si l'utilisateur choisit un skill installed-cache ou bundled

Vérifie si la source existe localement :
- **Source localement présente** → propose : « Le skill `<slug>` est installé en read-only. Sa source est dans `<repo_dir>/plugins/<slug>/`. Je modifie la source ; tu feras `/plugin update <slug>@<repo_name>` ensuite pour rafraîchir. OK ? »
- **Source absente** → propose deux options :
  1. Cloner d'abord le repo source et éditer la source (recommandé).
  2. Faire une copie standalone dans `~/.claude/skills/<slug>-fork/` et l'éditer (déconseillé : crée un fork divergent).

Ne propose jamais d'éditer directement la copie en cache.

---

## M.3 — Snapshot avant édition

Crée un backup horodaté du dossier complet du skill (pas juste le SKILL.md) :

```bash
SNAPSHOT_DIR="/tmp/<slug>-snapshot-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$SNAPSHOT_DIR"
cp -R "<skill-dir>" "$SNAPSHOT_DIR/"
```

Stocke `$SNAPSHOT_DIR` en variable locale pour le récap final (M.7).

Confirme à l'utilisateur : « Snapshot : `<SNAPSHOT_DIR>` (sera supprimé au reboot — copie ailleurs si tu veux le garder). Rollback : `cp -R <SNAPSHOT_DIR>/<slug>/* <original-path>/` ».

---

## M.4 — Lire le skill complet

Lis tout le contenu du skill choisi :
1. Le SKILL.md actuel (capture frontmatter + body en entier).
2. Liste et lis les fichiers dans `references/` et `assets/` du dossier du skill (s'ils existent).
3. Si Type A (marketplace) : lis aussi `<dir>/plugins/<slug>/.claude-plugin/plugin.json` et trouve la ligne du skill dans `<dir>/.claude-plugin/marketplace.json`.

Présente un récap court à l'utilisateur :

> "Skill `<slug>` (famille <X>). Frontmatter actuel — name: `<slug>`, description : `<début de description...>`. Body : <N> lignes. References/ : <liste ou aucun>. Assets/ : <liste ou aucun>. Plugin.json : version `<version>`."

---

## M.5 — Mode de modification

`AskUserQuestion` avec 3 options :

- **M.5a — Interactif (recommandé)** : tu me dis quoi changer en texte libre ou par sections, je propose un diff par section que tu valides.
- **M.5b — Coller** : tu colles le nouveau body (ou frontmatter) complet, je remplace.
- **M.5c — Régénérer à partir de fichiers de référence** : tu m'as fourni des fichiers de référence (Étape -1 a produit un brief), je régénère un nouveau body qui remplace l'ancien — utile pour une refonte majeure.

### M.5a — Mode interactif

Demande : « Qu'est-ce que tu veux modifier ? » (texte libre).

Réponses typiques :
- « Ajoute une étape entre l'Étape 2 et l'Étape 3 pour valider X. »
- « Réécris la description du frontmatter pour mentionner Y. »
- « Remplace la table de la section Z par de la prose explicative qui dit le pourquoi. »

Pour chaque demande :
1. Cite la section actuelle (lignes ou titre).
2. Propose la version modifiée (diff lisible avant/après).
3. Attends confirmation explicite (`OK` / `ajuste` / `annule`).
4. Applique seulement après OK.

Recommencer jusqu'à ce que l'utilisateur dise « c'est tout ».

### M.5b — Mode coller

Demande : « Colle ici le nouveau contenu. Précise si c'est le body complet (sans frontmatter) ou le SKILL.md complet (avec frontmatter). »

Vérifie :
- Si frontmatter inclus : le `name:` correspond toujours au slug d'origine (sinon préviens — pas de renommage en silence).
- Si seulement le body : préserve le frontmatter existant inchangé.

### M.5c — Mode régénérer (avec brief de référence)

Si l'Étape -1 a produit un brief (voir `REFERENCE_DETECTION.md` -1.7), utilise-le pour régénérer un nouveau body :
1. Récupère la liste des composantes cochées par l'utilisateur en -1.6.
2. Génère un nouveau body structuré : étapes correspondant aux composantes, instructions de style, format conforme aux fichiers de référence.
3. Le frontmatter `name:` reste inchangé. La `description:` peut être proposée à mise à jour si le brief le suggère (demande confirmation).

---

## M.6 — Validation structurelle

Avant d'écrire les changements, vérifie :

1. **Frontmatter complet** : `name:` et `description:` présents, valides YAML. Si manquant ou cassé, refuse et explique.
2. **Slug invariant** : le `name:` du frontmatter modifié est identique au `name:` original. Si différent, préviens explicitement (« Tu as changé le name de `<ancien>` à `<nouveau>`. Confirme-tu ? Cela peut casser les configs des utilisateurs qui ont installé ce skill. »).
3. **Taille du SKILL.md** : compte les lignes (`wc -l`). Si > 500, propose progressive disclosure (créer un fichier dans `references/`, déplacer une section longue). Pas bloquant, juste un avertissement.
4. **YAML frontmatter unique** : un seul bloc `---ouvrant---fermant` en début de fichier. Pas de double frontmatter.
5. **Description longueur** : entre 50 et 1024 caractères. Si trop courte (< 50), Claude risque de mal déclencher le skill (rappelle le principe « trigger primary »).

Si une validation échoue de façon non bloquante, demande à l'utilisateur si on continue ou si on corrige.

---

## M.7 — Commit, push, récap

### Action selon le Type d'origine du skill modifié

| Type | Action |
|---|---|
| **A — marketplace local stable/beta** | `git -C <repo_dir> add -A && git -C <repo_dir> commit -m "Update <slug>: <résumé en 1 ligne fourni par l'utilisateur>"`. Demande si on push : si oui, `git -C <repo_dir> push`. |
| **B — standalone** | Pas de commit. Le fichier est mis à jour en place dans `~/.claude/skills/<slug>/SKILL.md`. |
| **C — autre outil** | Pas de commit Git habituellement. Le fichier est mis à jour en place dans `<target_dir>/<slug>/`. |

### Récap final

Affiche :

```
✓ Skill <slug> modifié.

Snapshot         : <SNAPSHOT_DIR>
Rollback         : cp -R <SNAPSHOT_DIR>/<slug>/* <original-path>/
                   (le snapshot sera supprimé au prochain reboot)

Diff résumé      : <N> lignes ajoutées, <M> supprimées dans SKILL.md
                   <K> autres fichiers modifiés (références, plugin.json, marketplace.json…)

[Type A pushé]
GitHub           : https://github.com/<github_user>/<context>_name/tree/main/plugins/<slug>
Pour rafraîchir  : /plugin marketplace update <context>_name  →  /plugin update <slug>@<context>_name

[Type A pas pushé]
À faire          : cd <repo_dir> && git push

[Type B]
Slash command    : /<slug> (déjà actif, recharge si besoin)

[Type C]
Activation       : selon doc outil cible (souvent auto au prochain démarrage)
```

---

## Cas particuliers

### L'utilisateur veut renommer le slug malgré l'avertissement
Acceptable si l'utilisateur confirme explicitement après l'avertissement M.6. Dans ce cas, le workflow devient hybride :
1. Le snapshot est conservé sous l'ancien nom.
2. Le dossier du skill est renommé (`mv <ancien-path> <nouveau-path>`).
3. Le `name:` du frontmatter est mis à jour.
4. Pour Type A : `plugin.json` (`name`), `marketplace.json` (entrée `name`), `README.md` du repo sont mis à jour.
5. Préviens que les utilisateurs qui ont l'ancien slug installé devront `/plugin uninstall <ancien>@<repo>` puis `/plugin install <nouveau>@<repo>`.

### Le skill cible n'a pas de `references/` ni `assets/`
M.4 saute la lecture de ces dossiers, M.5a propose seulement de modifier SKILL.md, M.6 valide juste sur le SKILL.md.

### Le skill cible dépasse 500 lignes
M.6 émet l'avertissement, propose progressive disclosure. Si l'utilisateur veut ajouter encore plus de contenu, recommande fortement de scinder en `references/<sujet>.md` avant de continuer.

### Plusieurs skills installés portent le même slug (collision multi-marketplace)
Possible si l'utilisateur a installé `<slug>@stable` et `<slug>@beta` simultanément. M.1 doit lister les deux. M.2 doit les distinguer par leur path complet pour que le choix soit non ambigu.
