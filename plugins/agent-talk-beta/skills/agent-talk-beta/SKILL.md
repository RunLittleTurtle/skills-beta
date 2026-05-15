---
name: agent-talk-beta
description: Permettre à deux instances Claude (même repo nested OU cross-projet) d'échanger des messages structurés via un dossier-bridge partagé — conversation back-and-forth, rapport one-shot avec question, ou handoff de contexte. Notification par pings courts `[PING] @inst-X` dans `COORDINATION.FOCUS.md` (si même repo + coordination active) ou dans un `BRIDGE.PINGS.md` dédié. Complète `/coordination` (qui gère les locks de fichiers partagés) sans le remplacer. À invoquer quand un agent veut poser une question à une autre instance, lui passer un brief, ou démarrer une discussion async sans polluer le log de coordination.
---

# agent-talk-beta — communication inter-instances Claude

Aide deux (ou plus) instances Claude à s'échanger des messages structurés via un dossier-bridge partagé. Trois types : `conv` (back-and-forth), `report` (Q&A one-shot), `handoff` (passation de contexte). Notification par ligne `[PING] @inst-X` courte, dans `COORDINATION.FOCUS.md` si same-repo, sinon dans `BRIDGE.PINGS.md` au bridge.

**Réponds en français. Sois concis. Confirme les actions en une phrase.**

## Étape 1 — Vérifications préalables

### 1a. Contexte de base
- `pwd` → mémorise `CWD`
- `git rev-parse --show-toplevel 2>/dev/null` → mémorise `GIT_ROOT` (peut être null si hors repo)

### 1b. Recherche des fichiers COORDINATION accessibles (walk-up depuis CWD)

Le skill `coordination` écrit `COORDINATION.*.md` à la racine d'un repo git. Mais en mode nested (cwd plus profond que ce git root, ou repo avec submodules), le fichier peut être dans un dossier ancestor PLUS HAUT que `GIT_ROOT`. On remonte donc l'arbre depuis `CWD` jusqu'à `$HOME` (exclu) ou `/` :

```bash
dir="$CWD"
while [ "$dir" != "/" ] && [ "$dir" != "$HOME" ] && [ -n "$dir" ]; do
  for f in "$dir"/COORDINATION.*.md; do
    [ -e "$f" ] || continue
    case "$f" in
      */COORDINATION.README.md|*/COORDINATION.example.md) ;;
      *) echo "$f" ;;
    esac
  done
  dir=$(dirname "$dir")
done
```

Mémorise la liste ordonnée (du plus proche au plus loin) comme `COORD_CANDIDATES` :
- Si vide → `COORD_ACTIVE=false`, `COORD_FOCUS_FILE=null`
- Si exactement 1 → `COORD_FOCUS_FILE = ce fichier`, `COORD_ACTIVE=true`
- Si >1 → `COORD_ACTIVE=true`, choix différé à l'Étape 4d (notification)

Pas de blocage ici — `agent-talk-beta` fonctionne aussi hors repo et sans coordination (cross-projet via path absolu désigné).

## Étape 2 — Scanner les bridges existants accessibles

Cherche `BRIDGE.README.md` dans trois zones (parallèle, ignore les erreurs) :

### 2a. Walk-up depuis CWD (host à mon niveau OU plus haut)

```bash
dir="$CWD"
while [ "$dir" != "/" ] && [ "$dir" != "$HOME" ] && [ -n "$dir" ]; do
  for f in "$dir"/.agent-talk-beta/*/BRIDGE.README.md; do
    [ -e "$f" ] || continue
    echo "$f"
  done
  dir=$(dirname "$dir")
done
```

Cela couvre :
- `<CWD>/.agent-talk-beta/*/` (je suis l'agent host)
- Tous les ancêtres jusqu'à `$HOME` exclu (l'autre agent host est plus HAUT que mon cwd)

### 2b. Walk-down depuis CWD (host plus profond que moi)

```bash
find "$CWD" -maxdepth 5 -type f -name BRIDGE.README.md -path '*/.agent-talk-beta/*' 2>/dev/null
```

(maxdepth 5 = `<CWD>/<a>/<b>/<c>/.agent-talk-beta/<topic>/BRIDGE.README.md` — suffisant pour la plupart des cas)

### 2c. Bridges cross-projet stockés en zone neutre

```bash
ls ~/.claude/agent-talk-bridges/*/BRIDGE.README.md 2>/dev/null
```

### 2d. Consolidation

Déduplique la liste (un même path peut apparaître via 2a et 2b si CWD = parent). Pour chaque match, lis les 20 premières lignes du `BRIDGE.README.md` pour extraire : `topic`, `mode` (`same-repo` ou `cross-projet`), `created_at`, `host_cwd`, `notif` (path du fichier de pings). Mémorise comme `EXISTING_BRIDGES`.

## Étape 3 — Question principale

`AskUserQuestion` adapté à l'état :

### Cas A — Aucun bridge existant

> *"Aucun bridge agent-talk-beta détecté. Que veux-tu faire ?"*

Options :
- **"Créer un nouveau bridge"** (Recommended)
- **"Sortir sans rien faire"** — Termine le skill, ne charge pas le protocole

### Cas B — Au moins un bridge existant

> *"<N> bridges détectés. Que veux-tu faire ?"*

Options (limite à 4 max, regroupe le reste sous "Other") :
- **"Connecter à `<topic>` (`<mode>`, host `<host_cwd>`)"** (Recommended si bridge actif récent) — une option par bridge
- **"Créer un nouveau bridge"**
- **"Sortir sans rien faire"**

## Étape 4 — Création d'un nouveau bridge

### 4a. Mode

`AskUserQuestion` :

> *"Mode du bridge ?"*

Options :
- **"Same-repo nested"** (visible seulement si `GIT_ROOT` non null) — Les deux agents dans le même git repo, à des profondeurs cwd différentes. Bridge dans le cwd local.
- **"Cross-projet"** — Repos différents ou hors repo. Bridge à un path absolu désigné.

### 4b. Topic

`AskUserQuestion` (texte libre via "Other") :

> *"Topic du bridge (slug kebab-case, ex: `skill-test-feedback`, `handoff-storefront-cleanup`) ?"*

Valide le format `^[a-z][a-z0-9-]*$`. Si invalide, redemande.

### 4c. Path du bridge

**Si same-repo nested** :
- Path par défaut = `<CWD>/.agent-talk-beta/<topic>/`
- Confirme à l'utilisateur en une phrase : *"Bridge créé dans `<CWD>/.agent-talk-beta/<topic>/`. Si ton autre instance Claude a un cwd PLUS PROFOND, elle accédera quand même au bridge (mais idéalement le bridge devrait être chez elle — tu peux migrer manuellement plus tard). Si elle a un cwd au-dessus, elle y accède aussi."*

**Si cross-projet** :
- `AskUserQuestion` (texte libre via "Other") avec default proposé :

  > *"Path du bridge ? Default proposé : `~/.claude/agent-talk-bridges/<topic>/`"*

  Options :
  - **"`~/.claude/agent-talk-bridges/<topic>/`"** (Recommended)
  - **"Autre path"** (l'utilisateur tape via "Other")

- Expand `~/` et `$HOME/` si présents dans le path tapé.

### 4d. Mode de notification

**Si same-repo nested ET `COORD_ACTIVE=true`** :

Si `COORD_CANDIDATES` a plusieurs entrées (plusieurs `COORDINATION.*.md` trouvés en remontant l'arbre — rare mais possible si nested repos avec coordination à plusieurs niveaux), `AskUserQuestion` d'abord :

> *"Plusieurs fichiers COORDINATION trouvés en remontant depuis ton cwd. Lequel utiliser ?"*

Options : un par candidat, format `"<path> (depth <N> depuis CWD)"`, le plus proche en premier (Recommended). L'utilisateur tape un autre path via "Other" s'il veut.

Mémorise le choix comme `COORD_FOCUS_FILE`.

Puis `AskUserQuestion` :

> *"Où mettre les pings ? `<COORD_FOCUS_FILE>` (recommandé, moins de fichiers) ou un `BRIDGE.PINGS.md` dédié au bridge (sépare strictement locks/chat) ?"*

Options :
- **"COORDINATION.FOCUS.md"** (Recommended) — réutilise `<COORD_FOCUS_FILE>`
- **"BRIDGE.PINGS.md dédié"** — crée `<bridge-root>/BRIDGE.PINGS.md`

**Si same-repo nested ET `COORD_ACTIVE=false`** :
- Dis à l'utilisateur en une phrase : *"`/coordination` n'est pas active dans ce repo. Le bridge utilisera son propre `BRIDGE.PINGS.md`. Tu peux lancer `/coordination` plus tard si tu veux unifier les notifs."*
- Force `NOTIF_FILE = <bridge-root>/BRIDGE.PINGS.md`.

**Si cross-projet** :
- Force `NOTIF_FILE = <bridge-root>/BRIDGE.PINGS.md` (pas de coordination partagée possible).

### 4e. Bootstrap

- `mkdir -p <bridge-root>` (vérifie qu'il n'existait pas déjà avec contenu — si oui, propose Étape 5 connect)
- Écris `<bridge-root>/BRIDGE.README.md` (template ci-dessous, Étape 9)
- Si `NOTIF_FILE = <bridge-root>/BRIDGE.PINGS.md` : écris-le (template Étape 9)
- Si `NOTIF_FILE = <COORD_FOCUS_FILE>` (same-repo + COORDINATION) : append au log :
  ```
  - [BRIDGE-CREATED] <topic> by <INST_SELF> (<HH:MM>) — path=<bridge-root>, notif=here
  ```
- Si mode same-repo : propose à l'utilisateur (en une ligne, pas `AskUserQuestion`) : *"Ajouter `.agent-talk-beta/` au `.gitignore` du repo pour ne pas committer le bridge ? (Réponds dans le chat si oui.)"* — ne le fais PAS automatiquement.
- Confirme : *"Bridge `<topic>` créé en mode `<mode>` à `<bridge-root>`. Notif = `<NOTIF_FILE>`."*

Passe à Étape 6 (identité).

## Étape 5 — Connexion à un bridge existant

- Lis `<bridge-root>/BRIDGE.README.md` pour confirmer `mode`, `host_cwd`, `notif`
- Mémorise `BRIDGE_ROOT`, `BRIDGE_MODE`, `NOTIF_FILE`
- **Cas warning** : si `BRIDGE_MODE = same-repo` ET `CWD` est plus profond que `host_cwd` (string strictly longer + startswith host_cwd), signale à l'utilisateur : *"Ton cwd `<CWD>` est plus profond que le host `<host_cwd>`. Idéalement le bridge devrait être chez toi. Tu peux migrer manuellement (`mv <bridge-root> <CWD>/.agent-talk-beta/<topic>/`) puis update BRIDGE.README.md, mais c'est optionnel — le bridge fonctionne quand même."* Ne fais rien automatiquement.

Passe à Étape 6.

## Étape 6 — Identité de l'instance

**Si `BRIDGE_MODE = same-repo` ET `COORD_ACTIVE = true`** :
- Lis `<COORD_FOCUS_FILE>`. Cherche le tableau Contributeurs et la dernière entrée `[DONE] <inst-id> joined`.
- Si tu peux déduire ton `inst-id` du contexte de la session (ex: `/coordination` lancé plus tôt dans la conv) → réutilise-le. Sinon `AskUserQuestion` (texte libre via "Other") : *"Quel `inst-id` as-tu sur le focus coordination courant ?"*

**Sinon** (cross-projet OU same-repo sans coordination) :
- Lis le tableau Contributeurs de `<bridge-root>/BRIDGE.PINGS.md` (si existe, sinon ignore — tu es le premier).
- Calcule la prochaine lettre dispo : `inst-A` si vide, sinon max + 1.
- `AskUserQuestion` : *"Identité pour ce bridge : `inst-<lettre>`. Garder ou renommer ?"*. Options : "Garder inst-<lettre>" / "Renommer" (texte libre via "Other").
- Append au tableau : `| <INST_SELF> | <HH:MM> | (joined for <topic>) |`
- Append au log activité : `- [JOIN] <INST_SELF> joined <topic> (<HH:MM>)`

Mémorise `INST_SELF`.

## Étape 7 — Action principale

Scanne d'abord les pings `@<INST_SELF>` non-lus dans `<NOTIF_FILE>` (cf. Étape 7b §1-2 pour la logique de détection). Mémorise `UNREAD_PINGS_COUNT`.

`AskUserQuestion` :

> *"Que veux-tu faire sur le bridge `<topic>` ?"*

Options :
- **"Lire les `<N>` messages reçus"** (Recommended si `UNREAD_PINGS_COUNT > 0`)
- **"Envoyer un message"** (Recommended sinon)
- **"Lister les fichiers du bridge"**
- **"Charger le protocole pour le reste de la session"** — termine le skill, applique l'Étape 8

### 7a. Envoyer un message

1. `AskUserQuestion` : *"Type de message ?"*. Options :
   - **"conv"** — Conversation back-and-forth (destinataire répond par append, alterné)
   - **"report"** — Rapport one-shot avec question (destinataire répond une fois puis clôt)
   - **"handoff"** — Passation de contexte (sujet, files touchés, décisions, next, blocages)

2. `AskUserQuestion` : *"Destinataire ?"*. Lis les contributeurs connus (depuis BRIDGE.PINGS.md ou COORDINATION) et propose-les en options, plus "Other" pour taper.

3. `AskUserQuestion` (texte libre via "Other") : *"Slug du sujet (kebab-case court, ex: `feedback-naming`, `q-api-shape`) ?"*

4. Path = `<bridge-root>/<type>-<slug>.md`. Si existe déjà :
   - `AskUserQuestion` : *"`<type>-<slug>.md` existe déjà. Append un nouveau message au fichier OU choisir un autre slug ?"*
   - Si append : skippe création, va à 7a-bis (append).

5. Récupère HH:MM courant.

6. **Si type = handoff** : analyse le contexte de la session (files lus/édités, décisions, blocages) et génère ce template (demande validation interactive AVANT d'écrire si la session est riche, sinon génère direct) :

   ```markdown
   # handoff-<slug>

   > Type : handoff | Topic : <slug> | Bridge : <topic>
   > Créé par <INST_SELF> à <HH:MM>

   ## Sujet
   <1-2 phrases>

   ## Files / sources touchés
   - <path1>
   - <path2>

   ## Décisions
   - <décision 1>

   ## Next steps
   - <next 1>

   ## Blocages / questions ouvertes
   - <blocage 1>

   ---

   **Tour : @<destinataire>**
   ```

   **Pour `conv` et `report`** : demande à l'utilisateur le contenu du premier message (texte libre dans le chat, pas `AskUserQuestion`), puis structure :

   ```markdown
   # <type>-<slug>

   > Type : <type> | Topic : <slug> | Bridge : <topic>
   > Créé par <INST_SELF> à <HH:MM>

   ## <INST_SELF> (<HH:MM>)

   <contenu>

   ---

   **Tour : @<destinataire>**
   ```

7. Écris le fichier (Write).

8. Demande à l'utilisateur (texte libre, court) : *"Résumé en 1 ligne pour le ping ?"*. Si l'utilisateur ne répond pas en 1 tour, génère un résumé toi-même depuis le contenu (max 80 chars).

9. Append à `<NOTIF_FILE>` :
   ```
   - [PING] <slug> @<destinataire> by <INST_SELF> (<HH:MM>) — bridge:<type>-<slug>.md — "<résumé>"
   ```

10. Confirme : *"Message `<type>-<slug>.md` envoyé à `@<destinataire>`. Ping ajouté à `<NOTIF_FILE>`."*

### 7a-bis. Append à un fichier de conv existant

1. Lis le fichier en entier.
2. Vérifie le dernier `**Tour : @inst-X**` :
   - Si `@<INST_SELF>` → c'est ton tour, OK pour append
   - Si `@<autre>` → c'est le tour d'un autre, **demande à l'humain** : *"Le dernier turn marker est `@<autre>`, pas toi. Append quand même (force) ou laisser ?"*
   - Si `(fermé)` → la conv est close, demande à l'humain si on rouvre.
3. Récupère HH:MM. Append à la fin du fichier :
   ```markdown

   ## <INST_SELF> (<HH:MM>)

   <contenu>

   ---

   **Tour : @<destinataire>**
   ```
4. Append `[PING]` retour à `<NOTIF_FILE>` (même format que 7a §9).
5. Confirme.

### 7b. Lire les messages reçus

1. Lis `<NOTIF_FILE>`. Grep les lignes contenant `[PING]` ET `@<INST_SELF>`.
2. Pour chaque ping trouvé, cherche une ligne `[READ-PING] <slug> by <INST_SELF>` POSTÉRIEURE (plus loin dans le fichier). Si absente → ping non-lu.
3. Présente la liste des pings non-lus avec slug + path + résumé + auteur + HH:MM.
4. Pour chaque ping non-lu, `AskUserQuestion` : *"Lire `<bridge-root>/<type>-<slug>.md` maintenant ?"*. Options : **"Oui, lire"** / **"Skip pour l'instant"**.
5. Si "Oui, lire" :
   - Read le fichier
   - Affiche le contenu (ou résumé si très long, > 200 lignes)
   - Append à `<NOTIF_FILE>` : `- [READ-PING] <slug> by <INST_SELF> (<HH:MM>) — ack`
   - `AskUserQuestion` : *"Répondre maintenant ?"*. Si oui → workflow 7a-bis sur ce fichier.

### 7c. Lister les fichiers du bridge

Liste `<bridge-root>/*.md` SAUF `BRIDGE.README.md` et `BRIDGE.PINGS.md`. Pour chaque, affiche :
- Type (`conv` / `report` / `handoff`)
- Slug
- Dernier auteur (cherche dernière section `## <inst-X> (HH:MM)`)
- Dernier HH:MM
- Statut tour (cherche dernier `**Tour : @inst-X**` ou `(fermé)`)

Vue d'ensemble. Aucune action implicite après affichage.

## Étape 8 — Règles pour le reste de la session

À partir de maintenant et tant que la session dure, **TU DOIS** appliquer ce protocole :

> **Identité** : tu es `<INST_SELF>` sur bridge `<topic>` à `<bridge-root>`. Notif = `<NOTIF_FILE>`.
>
> **Pour envoyer un nouveau message** : workflow 7a (créer fichier + ping).
>
> **Pour append à un fichier de conv/report existant** : workflow 7a-bis (vérifier turn marker, append, ping retour).
>
> **Pour répondre à un ping reçu** : workflow 7b.
>
> **Si tu vois un nouveau ping `@<INST_SELF>` au cours de la session** (par ex. en relisant le notif file pour une autre raison) : signale-le à l'humain en une phrase et propose de traiter via 7b.
>
> **Pas de locks** : agent-talk-beta n'utilise pas le système `[LOCKED]` de coordination. Le turn marker `**Tour : @inst-X**` est le protocole d'exclusion. Si tu suspectes une collision réelle (deux agents écrivent en même temps), demande à l'humain d'arbitrer.
>
> **Messages courts** : le bridge sert à poser une question, demander un feedback, passer un brief. Les détails techniques restent dans les files source du repo (référencés par path). N'y duplique pas de la doc.
>
> **Clôture** : pour terminer une conv/report, change le turn marker en `**Tour : (fermé)**` et append `- [CLOSED] <slug> by <INST_SELF> (<HH:MM>) — <raison courte>` à `<NOTIF_FILE>`.

## Étape 9 — Templates générés

### Template `BRIDGE.README.md` (créé à l'Étape 4e)

Substitue `<topic>`, `<mode>`, `<host_cwd>`, `<notif>`, `<INST_SELF>`, `<HH:MM>`, `<YYYY-MM-DD>` :

```markdown
# BRIDGE.README.md — `<topic>`

> Protocole et légende du bridge agent-talk-beta.
> Pour rejoindre : lance `/agent-talk-beta` et choisis "Connecter à `<topic>`".

## Config

- **Topic** : `<topic>`
- **Mode** : `<same-repo | cross-projet>`
- **Host cwd** : `<host_cwd>`
- **Notif** : `<notif>` (où vont les `[PING]`, `[READ-PING]`, `[CLOSED]`)
- **Créé par** : `<INST_SELF>` le `<YYYY-MM-DD HH:MM>`

## À quoi sert ce bridge

Permettre à des instances Claude (même repo ou repos différents) d'échanger des messages structurés sans polluer le log de coordination. Trois types :

| Préfixe | Type | Usage |
|---|---|---|
| `conv-<slug>.md` | Conversation back-and-forth | Discussion async |
| `report-<slug>.md` | Rapport one-shot avec question | Contexte + question précise |
| `handoff-<slug>.md` | Passation de contexte | Brief structuré pour qu'un autre continue |

## Statuts dans le notif

| Statut | Signification |
|---|---|
| `[PING]` | Nouveau message dispo pour le destinataire |
| `[READ-PING]` | Le destinataire a lu, ack |
| `[CLOSED]` | Fichier clôturé |
| `[JOIN]` | Une instance a rejoint le bridge |
| `[BRIDGE-CREATED]` | Bridge créé (same-repo + coordination) |

## Format d'un ping

```
- [PING] <slug> @<destinataire> by <auteur> (<HH:MM>) — bridge:<type>-<slug>.md — "<résumé 1 ligne>"
```

## Turn marker

Chaque message se termine par `**Tour : @<destinataire>**`. Pour clôturer : `**Tour : (fermé)**`.

## Règles

- Pas de locks. Le turn marker suffit.
- Messages courts. Détails dans les files source.
- En mode same-repo, le bridge est dans `.agent-talk-beta/<topic>/` — pense à l'ajouter au `.gitignore` du repo.
- En mode cross-projet, le bridge est externe (souvent `~/.claude/agent-talk-bridges/<topic>/`).
```

### Template `BRIDGE.PINGS.md` (créé à l'Étape 4e si notif dédié)

Substitue `<topic>`, `<INST_SELF>`, `<HH:MM>` :

```markdown
# BRIDGE.PINGS.md — `<topic>`

> Log de pings et de présence pour ce bridge.
> Protocole : voir `BRIDGE.README.md`.

---

## Contributeurs

| Instance | Joined | Notes |
|---|---|---|
| <INST_SELF> | <HH:MM> | (créateur du bridge) |

**Lettres utilisées** : <lettre> → prochaine = `inst-<lettre+1>`

---

## Activité (ancien → récent, append en bas)

- [JOIN] <INST_SELF> joined <topic> (<HH:MM>)
```

## Règles générales

- **Pas de commit automatique.** L'utilisateur décide quand committer.
- **Pas de suppression de fichier sans confirmation explicite.**
- **Idempotence du setup** : si `BRIDGE.README.md` existe déjà au path cible, propose de connecter au lieu de re-bootstrap.
- **Pas de duplication d'entrée** : avant d'append `[JOIN]`, vérifie qu'une ligne `[JOIN] <INST_SELF>` n'existe pas déjà pour ce bridge — sinon skip.
- **Pas de modification du structure de COORDINATION.FOCUS.md** : agent-talk-beta n'ajoute que des entrées (`[BRIDGE-CREATED]`, `[PING]`, `[READ-PING]`, `[CLOSED]`) au log activité, jamais ne change le format ou les tableaux existants.
- **Format de slug** : kebab-case court (`feedback-naming`, `q-api-shape`).
- **Timestamps HH:MM** locaux 24h. Date complète au format `YYYY-MM-DD HH:MM` dans les configs.
- **Réponses en français.** Confirmation en une phrase.
