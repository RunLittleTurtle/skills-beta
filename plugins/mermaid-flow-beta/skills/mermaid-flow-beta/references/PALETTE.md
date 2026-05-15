# Palette, classDef et légende dynamique

Lu à l'étape 4a de SKILL.md, en complément des squelettes de `VARIANTS.md`. Cette palette est figée pour assurer la cohérence visuelle entre les diagrammes que le directeur de compte agrège dans son plan de projet.

---

## Couleurs exactes (ne jamais modifier sans accord explicite)

| Acteur | Emoji | classDef | Fill | Stroke | Texte |
|---|---|---|---|---|---|
| Client | 👤 | `clientAction` | `#E1BEE7` (violet pastel) | `#6A1B9A` | `#000` |
| IA | 🤖 | `iaAction` | `#BBDEFB` (bleu pastel) | `#1565C0` | `#000` |
| Système | ⚙️ | `systemAction` | `#CFD8DC` (gris pastel) | `#455A64` | `#000` |
| UI qui s'affiche | 🖥️ | `uiAction` | `#C8E6C9` (vert pastel) | `#2E7D32` | `#000` |
| Décision | ⚖️ | `decision` | `#FFF9C4` (jaune pastel) | `#F57F17` | `#000` |

Texte toujours en `color:#000`. Pas d'autre couleur que ces 5.

---

## Bloc classDef à inclure dans chaque diagramme

```
classDef clientAction fill:#E1BEE7,stroke:#6A1B9A,color:#000
classDef iaAction fill:#BBDEFB,stroke:#1565C0,color:#000
classDef systemAction fill:#CFD8DC,stroke:#455A64,color:#000
classDef uiAction fill:#C8E6C9,stroke:#2E7D32,color:#000
classDef decision fill:#FFF9C4,stroke:#F57F17,color:#000
```

Si un acteur est absent du flow, retire sa ligne `classDef` correspondante.

---

## IDs des nodes

- Nodes principaux : single letter `A`, `B`, `C`, `D`, `E`, `F`, `G` (max 7 idéal, 10 hard cap).
- Branches parallèles (variante 5) : suffixes `a` et `b` (ex: `D` reste, mais on a `3a` et `3b` dans les labels ; les IDs Mermaid restent uniques `D`, `E`, `F`, `G`, `H`).
- Références à d'autres flows : `REF1`, `REF2`, ...
- Légende : `L1`, `L2`, `L3`, `L4`, `L5`.

Forme :

- Actions = rectangles `["..."]`
- Décisions = losanges `{"..."}`

---

## Labels

- Emoji au début du label (`"👤 1. Clique 'Générer'"`).
- 7-10 mots maximum par label.
- Parenthèses pour les détails (ex: `"Saisit ses critères (dates, voyageurs)"`).
- Numérotation `"1. ..."`, `"2. ..."` : optionnelle, recommandée si plus de 5 étapes.
- Pour la variante 5 (2 chemins parallèles) : `"3a. ..."` / `"3b. ..."`, puis convergence avec numéro commun (`"6. ..."`).

---

## Liens

- Linéaire : `A --> B --> C`
- Décision courte : `C -->|"Oui"| D` / `C -->|"Non"| E`
- Décision avec labels descriptifs (variante 5) : `C -->|"Simple"| D` / `C -->|"Complexe"| F`
- Boucle : `D --> B` (retour vers une étape antérieure)
- Convergence (variante 5) : deux branches qui pointent vers le même node de résultat
- Liaison légende-flux : `LEG ~~~ FLOW` (toujours en bas du diagramme)

---

## Légende dynamique

La légende est un subgraph `LEG["Légende"]` avec `direction LR` et 5 nodes `L1` à `L5` connectés par `~~~` (pointillés invisibles). **Mais elle n'inclut que les acteurs réellement présents dans le flow**.

### Règle de construction

1. Identifie quels acteurs apparaissent dans le flux (parmi 👤, 🤖, ⚙️, 🖥️, ⚖️).
2. Inclus uniquement les nodes `L<n>` correspondants, dans l'ordre canonique : client → IA → système → UI → décision.
3. Renumérote séquentiellement (pas de trous).
4. Retire les `classDef` non utilisées.
5. Retire les lignes `class L<n> ...` non utilisées.

### Exemple : flow sans UI

Si le flow ne contient pas de 🖥️, la légende passe de 5 à 4 nodes :

```
subgraph LEG["Légende"]
    direction LR
    L1["👤 Action du client"] ~~~ L2["🤖 Action de l'assistant IA"] ~~~ L3["⚙️ Action du système"] ~~~ L4["⚖️ Décision"]
end
```

Et le bloc `class` devient :

```
class L1 clientAction
class L2 iaAction
class L3 systemAction
class L4 decision
```

Et la `classDef uiAction` est retirée.

---

## Vérifications finales (étape 4b)

Avant de produire le fichier, vérifie :

1. Chaque node est classé dans **exactement** une classe. Sinon Mermaid le rend gris par défaut, ce qui casse la lecture.
2. Labels ≤ 7-10 mots, emoji au début, parenthèses pour les détails.
3. Légende dynamique : pas de node d'acteur absent, classDef et `class L<n>` cohérents.
4. Max 10 nodes principaux hors légende (cible 7).
5. Numérotation cohérente : `"1."`, `"2."`, `"3a."`, `"3b."`, etc.
6. `flowchart TB` au niveau racine, `direction LR` dans chaque subgraph.
7. `style FLOW fill:none,stroke:none` en fin de diagramme.
8. Toujours commencer par `%%{init: {'flowchart': {'curve': 'basis'}}}%%`.
