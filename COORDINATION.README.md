# COORDINATION.README.md

> Protocole et légende du système de coordination multi-instance Claude pour ce repo.
> Pour activer la coordination dans une session, utilise la commande `/coordination` au début de session.

## À quoi sert ce système

Coordonner plusieurs instances Claude (et toi, l'humain) qui travaillent en parallèle sur ce repo, pour éviter qu'elles écrasent mutuellement leurs édits sur les mêmes fichiers.

**Lecture, recherche, planification : LIBRE**, pas besoin de lock.
**Edit/Write sur un fichier partagé : OBLIGATOIRE** d'écrire une entrée `[LOCKED]` dans le log avant.

## Architecture multi-focus

Un fichier de log par **focus** (sujet/projet) :
- `COORDINATION.SPRINT.md` — instances qui travaillent sur les sprints
- `COORDINATION.TASK.md` — instances qui travaillent sur les tâches/user stories
- `COORDINATION.PRD.md` — instances qui travaillent sur les PRD
- ... (autres focus créés à la volée selon besoin)

Une instance ne lit/n'écrit QUE dans le fichier correspondant à son focus. Les instances sur des focus différents ne se voient pas — par convention, elles ne touchent pas aux mêmes fichiers de doc.

## Statuts d'une entrée

| Statut | Signification | Action requise par les autres |
|---|---|---|
| `[INTENT]` | Planifié, pas commencé | Aucune (autres peuvent continuer) |
| `[LOCKED]` | En train d'écrire MAINTENANT | **Personne d'autre n'écrit sur ce fichier/section** |
| `[WAITING]` | Veut écrire mais bloqué par un `[LOCKED]` actif | Attend que le lock se libère |
| `[DONE]` | Terminé | Aucune (servira de contexte aux suivants) |
| `[CANCELLED]` | Abandonné | Aucune |

**Quand le statut change**, on **modifie la ligne sur place**, on ne la déplace PAS. La ligne reste à sa position chronologique.

## Format d'une entrée (Claude structuré)

```
- [STATUS] <slug> by <inst-id> (<timing>) — <fichier> §<section> — "<intent ou résultat>"
```

- **`<slug>`** : court, kebab-case (ex: `bilan-9`, `meetings-section`)
- **`<inst-id>`** : `inst-A`, `inst-B`, … ou `human` pour l'humain
- **`<timing>`** :
  - `[LOCKED]` : `claimed HH:MM, TTL 30min`
  - `[DONE]` : `HH:MM → HH:MM` (début → fin)
  - `[INTENT]` / `[WAITING]` : `added HH:MM`
- **`<section>`** : `§Bilan`, `§3.2`, `L12` (ligne 12), ou `tout` si fichier entier
- **`"<…>"`** : intention courte si en cours, résultat court si fini

## Format simplifié (humain)

L'humain peut écrire plus rudimentaire. Tant que la ligne commence par `[STATUS]`, c'est OK :

```
- [DONE] human: ajouté date rétro sprint 9 ligne 12 (14:42)
```

## Règles d'usage

1. **Lecture/recherche/planification** : LIBRE, pas besoin de claim.
2. **Edit/Write** : ajoute une ligne `[LOCKED]` AVANT, change-la en `[DONE]` APRÈS.
3. **Avant de claim** : lis le log, vérifie qu'aucun `[LOCKED]` ou `[WAITING]` non périmé ne couvre ta zone. Ignore les locks dont `claimed + TTL < maintenant` (= périmés).
4. **Sur collision** (un autre lock couvre ta zone) : ajoute une ligne `[WAITING]` et **demande à l'humain** quoi faire (attendre, switcher, abandonner).
5. **Quand un `[LOCKED]` que tu attendais devient `[DONE]`** : RELIS le fichier modifié, valide que ton plan tient toujours, et **demande à l'humain "OK go ?"** avant de claim.
6. **Toi (humain)** : tu peux écrire ici aussi, owner = `human`. Si tu vois un lock zombie (TTL dépassé), tu peux le passer en `[CANCELLED]` à la main.

## Quoi grep pour quoi

| Tu veux savoir… | Cherche… |
|---|---|
| Qui écrit MAINTENANT | `[LOCKED]` |
| Qui veut écrire mais bloqué | `[WAITING]` |
| Ce qui est planifié mais pas commencé | `[INTENT]` |
| Ce qui a été terminé récemment | `[DONE]` |
| Toute l'activité sur un fichier précis | `<nom-du-fichier>` |
| Toute l'activité d'une instance | `inst-A` (ou autre owner) |
