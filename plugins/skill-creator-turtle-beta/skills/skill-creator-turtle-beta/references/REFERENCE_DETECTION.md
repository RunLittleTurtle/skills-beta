# Détection automatique de fichiers de référence

À lire quand l'invocation initiale du skill-creator-turtle contient un ou plusieurs chemins de fichiers existants. Ce workflow remplace les consignes en texte libre du genre "analyse profondément ce fichier" par une séquence de questions structurées, et termine par un brief de référence injecté dans les Étapes 4, 6a et 6c du workflow principal.

**Principe central** : l'utilisateur fournit un ou plusieurs fichiers de référence (gabarits de qualité, exemples bien faits, fichiers à reproduire). Le skill-creator-turtle les **lit, les analyse réellement**, et propose à l'utilisateur des composantes **DÉRIVÉES DU CONTENU RÉEL** des fichiers (pas une liste hardcodée). L'utilisateur coche celles que le skill généré doit reproduire.

---

## -1.1 Scanner le message d'invocation

Cherche dans le message initial complet de l'utilisateur (la phrase qui a déclenché ce skill) **tous les tokens qui ressemblent à un chemin de fichier** :

- Absolus : `/Users/...`, `/home/...`, `/var/...`, etc.
- Avec tilde : `~/foo/bar.md`
- Relatifs : `./docs/spec.txt`, `src/app.py`, `data/users.csv`
- Patterns glob : `docs/*.md`, `src/**/*.py`
- URLs locales : `file:///path/to/file`

**Aucun filtre par extension.** Tout token qui ressemble à un chemin est candidat.

**Validation** : pour chaque candidat, vérifie que le fichier (ou le dossier, ou le glob) **existe** sur disque (`ls <path>` ou tentative directe de `Read`). Si le path n'existe pas, signale-le à l'utilisateur en une ligne et demande s'il veut corriger ou ignorer.

Pour les patterns glob : expanse-les en liste de fichiers concrets (`ls docs/*.md`).

Si aucun path détecté ou validé → SKIP toute cette étape, retourne au workflow principal à l'Étape 0.0.

---

## -1.2 Lire chaque fichier détecté

Pour chaque fichier validé :

1. Lire avec le tool `Read`. Le tool gère automatiquement md, txt, code, json, yaml, csv, PDF, images.
2. Pour les très gros fichiers (>1000 lignes) : lire les 500 premières lignes + un échantillon en milieu/fin pour avoir une vue d'ensemble. Indiquer à l'utilisateur qu'on a tronqué.
3. Pour un fichier binaire non lisible : signaler "Fichier binaire non lisible, je le saute" et continuer avec les autres.

Affiche un récap court à l'utilisateur, **avant de poser les questions** :

> "J'ai trouvé N fichier(s) de référence :
> - `<chemin1>` (<format>, <taille>) — <brève description en 1 ligne du contenu>
> - `<chemin2>` ...
>
> Je vais te poser quelques questions pour comprendre comment les utiliser."

---

## -1.3 AskUser : rôle de chaque fichier

`AskUserQuestion` (multiSelect activé pour permettre de cumuler les rôles) :

> "Quel est le rôle de ce/ces fichier(s) dans le skill que tu veux créer ?
>
> - **Modèle d'OUTPUT** : le skill généré doit produire des fichiers qui ressemblent à ceux-ci (même structure, même style, même conventions)
> - **Modèle d'INPUT** : le skill généré doit savoir lire / accepter ce format en entrée
> - **Référence métier / règles** : le skill doit appliquer les règles, conventions, ou logique métier qu'on peut extraire de ces fichiers
> - **Exemple de cas d'usage** : juste pour comprendre le contexte, pas un template direct"

**Si plusieurs fichiers** et que tu suspectes que le rôle peut différer : demande d'abord "Tous ces fichiers ont-ils le même rôle ?" — si non, pose la question pour CHAQUE fichier individuellement.

Cette question est **universelle** : elle marche pour n'importe quel type de fichier (doc métier, code, données, mockup, etc.).

---

## -1.4 AskUser : profondeur d'analyse

`AskUserQuestion` (single select) :

> "Quelle profondeur d'analyse pour extraire les patterns et règles ?
>
> - **Structurelle** (rapide) : juste les sections / fonctions / colonnes / éléments principaux du fichier
> - **Approfondie** (recommandée) : structure + intentions + style + conventions implicites
> - **Exhaustive** : chaque détail, chaque sous-pattern, chaque règle métier déductible"

Cette question est **universelle** : elle influence la finesse de l'analyse en -1.5.

---

## -1.5 ANALYSER les fichiers selon la profondeur choisie

**C'est l'étape la plus importante.** Le skill-creator-turtle analyse réellement le contenu des fichiers pour identifier les **composantes structurantes présentes**. Le résultat est utilisé pour générer dynamiquement les options de la question -1.6.

### Quoi extraire (selon le type de fichier)

Adapte automatiquement l'analyse au format détecté :

**Pour un document markdown / texte** :
- Sections et hiérarchie (h1/h2/h3)
- Types de blocs présents (mermaid, code blocks, tableaux, listes, citations, formules math, images)
- Style des diagrammes (sequenceDiagram, flowchart, classDiagram, ER, etc. — si mermaid)
- Patterns de numérotation (1.1/1.2, A/B/C, romain, etc.)
- Naming convention (titres en français vs anglais, casse, longueur)
- Style rédactionnel (longueur paragraphes, voix active/passive, ton, langue, niveau de détail)
- Structure des tableaux (colonnes récurrentes, ex Code/Nom/Type)
- Sections récurrentes typiques (ex : Contexte, Objectif, Acteur, Scénario, Annexes)

**Pour un fichier de code (.py, .ts, .js, etc.)** :
- Style des docstrings (Google, NumPy, reST, JSDoc, aucun)
- Type hints / annotations (présence, exhaustivité)
- Conventions de naming (snake_case, camelCase, PascalCase)
- Organisation des imports (groupés stdlib/third-party/local, triés alphabétiquement)
- Pattern de gestion d'erreurs (exceptions custom, codes de retour, Result types)
- Style de tests (pytest fixtures, parametrize, mock patterns)
- Architecture (classes vs fonctions, fichiers par feature ou par couche)
- Validation (Pydantic, Zod, manuelle, etc.)
- Logging (loguru, stdlib, structlog)

**Pour un fichier de données (CSV, JSON, YAML, XML)** :
- Schéma (colonnes / clés présentes, types, obligatoires vs optionnelles)
- Conventions de naming des champs
- Format des identifiants (regex, longueur, prefix)
- Langue des headers vs contenu
- Granularité (1 ligne = 1 entité ?)
- Patterns de valeurs (énumérations détectées)

**Pour un PDF / docx** :
- Sections et structure
- Présence de tableaux, graphiques, images
- Style (template, formel/informel)
- Récurrences (header/footer, numérotation)

### Si plusieurs fichiers fournis

Détecte explicitement **les patterns COMMUNS** entre les fichiers vs les patterns spécifiques à chacun. Cas d'usage typique : l'utilisateur donne 3 fichiers Python "exemples de bonnes pratiques" et veut que le skill généré reproduise les patterns présents dans **les 3** (les patterns communs sont les invariants à imposer).

Tag chaque composante détectée avec sa fréquence : "présent dans 3/3 fichiers" (commun) vs "présent dans 1/3" (spécifique).

### Profondeur

- **Structurelle** : extraire seulement les éléments visibles à la première lecture (sections, blocs, headers).
- **Approfondie** : ajouter les conventions implicites (style, naming, patterns récurrents).
- **Exhaustive** : ajouter chaque détail, chaque sous-pattern, chaque règle métier déductible. Plus lent.

---

## -1.6 AskUser : composantes à reproduire (DYNAMIQUE)

**C'est ici que la liste est générée à partir de l'analyse en -1.5, jamais une liste fixe.**

`AskUserQuestion` (multiSelect activé) avec une liste construite dynamiquement à partir des composantes détectées :

> "Voici les composantes que j'ai détectées dans tes fichiers de référence. Coche celles que le skill généré doit reproduire dans ses outputs :"

Formule chaque option en termes **concrets** et compréhensibles, en citant le fichier source quand c'est utile :

**Exemples pour un .md de doc métier (cas du scenario_1)** :
- "Sections Contexte / Acteur / Objectif / Scénario / Séquence / Attributs"
- "Diagramme mermaid de type sequenceDiagram"
- "Tableau récapitulatif avec colonnes Code / Nom / Type"
- "Numérotation hiérarchique 1.1 / 1.2 / 1.3"
- "Style narratif au présent, voix active, en français"

**Exemples pour des .py de bonnes pratiques** :
- "Docstrings Google-style sur toutes les fonctions publiques"
- "Type hints exhaustifs (paramètres + retours)"
- "Imports groupés stdlib / third-party / local, triés"
- "Gestion d'erreurs avec exception custom AuthError (commune aux 3/3 fichiers)"
- "Tests pytest avec fixtures parametrize"

**Exemples pour un CSV** :
- "Colonnes obligatoires : id, nom, catégorie"
- "Codes au format regex `^[A-Z]{2}\\d{4}$`"
- "Headers en français, valeurs en anglais"

**Limites pratiques** : `AskUserQuestion` accepte jusqu'à ~10 options. Si tu détectes plus de 10 composantes, regroupe les moins importantes ou propose les 10 plus saillantes en mentionnant qu'on peut ajouter les autres en mode "Other".

---

## -1.7 Synthétiser le brief de référence

Construis en mémoire un brief structuré qui sera **injecté automatiquement** dans les Étapes 4, 6a et 6c du workflow principal :

```
RÉFÉRENCES
- Fichier 1 : <chemin> — rôle: <X>, profondeur: <Y>
- Fichier 2 : ...

PATTERNS COMMUNS DÉTECTÉS (si plusieurs fichiers)
- <pattern 1> (3/3)
- <pattern 2> (3/3)

PATTERNS SPÉCIFIQUES (info)
- <pattern 3> (1/3, fichier <X>)

COMPOSANTES COCHÉES PAR L'UTILISATEUR (à reproduire)
- <composante 1>
- <composante 2>
- ...

CONSIGNE GÉNÉRÉE POUR LE SKILL
"Le skill <slug> doit prendre en entrée <type d'input à inférer>, et produire en sortie un fichier qui contient : <composante 1>, <composante 2>, ... Le style doit reproduire <style détecté>."
```

Cette consigne est ensuite utilisée :

- **Étape 4 (description)** : pré-remplir une suggestion de description basée sur la consigne.
- **Étape 6a (mode interactif)** : utiliser les composantes cochées comme squelette d'étapes proposées (1 étape par grosse composante : "Étape 1 — Lire l'input", "Étape 2 — Générer le diagramme", etc.).
- **Étape 6c (squelette minimal)** : remplacer les `Étape N — TODO` génériques par des étapes pré-titrées correspondant aux composantes.

---

## Cas particuliers

### L'utilisateur donne un dossier au lieu d'un fichier

Si un path détecté est un dossier : demande via `AskUserQuestion` :
> "`<dossier>` est un dossier. Tu veux :
> - Analyser tous les fichiers du dossier (ou un sous-ensemble par extension)
> - Choisir un fichier précis dedans
> - Ignorer ce path"

### Aucune composante intéressante détectée

Si l'analyse en -1.5 ne retourne rien d'exploitable (fichier vide, format non reconnu, contenu trop bref) : préviens l'utilisateur et passe directement à l'Étape 0.0 sans poser -1.6.

### L'utilisateur veut compléter avec des composantes non détectées

À la question -1.6, l'option "Other" est toujours disponible. Texte libre pour ajouter des composantes que l'analyse n'a pas captées.
