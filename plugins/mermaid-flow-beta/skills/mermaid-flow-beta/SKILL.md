---
name: mermaid-flow-beta
description: Version expérimentale du skill mermaid-flow. Transformer un flow (texte, transcript, scénario, fichier markdown, mermaid existant ou image) en un flowchart Mermaid SIMPLIFIÉ pour parties prenantes et directeurs de compte (max 10 étapes, idéalement 7). Analyse en 3 passes (narrative, ambiguïtés noms propres, valeur vs bruit) avant génération. Préfère labels génériques (le chatbot, l'app) plutôt que noms propres de produits/équipes/clients — signale les ambiguïtés détectées. 5 variantes disponibles : linéaire, décision, boucle, référence externe, 2 chemins parallèles. Produit un fichier .md au format strict avec légende dynamique, palette pastel light-mode, emojis acteurs (👤 client, 🤖 IA, ⚙️ système, 🖥️ UI, ⚖️ décision). À invoquer quand l'utilisateur veut vulgariser un processus métier en diagramme accessible pour un plan de projet.
---

# mermaid-flow-beta — vulgarisation de processus pour plans de projet

Aide l'utilisateur à transformer un flow (texte, transcript, scénario, fichier markdown, mermaid existant ou image) en un diagramme Mermaid simplifié et lisible pour des **parties prenantes et directeurs de compte**, qui l'intégreront dans un plan de projet. Format strict aligné sur la référence : 7 étapes idéales, légende dynamique, palette pastel light-mode, emojis acteurs.

**Réponds en français. Sois concis. Confirme les actions en une phrase.**

---

## Étape 1 — Récupérer l'input

### 1a. Identifier le type d'input

Si l'utilisateur n'a rien fourni d'explicite, demande-lui dans le chat (pas via AskUserQuestion) :

> *« Tu veux convertir quoi ? Du texte que tu colles ici, un fichier markdown local, un mermaid existant à simplifier, ou une image/screenshot ? »*

### 1b. Ingestion par type

**Texte/transcript/scénario dans le chat** : le contenu est déjà dans le contexte. Ne lis rien de plus, passe à l'étape 2.

**Fichier markdown local** : demande le path absolu en texte libre. Utilise `Read` sur le fichier complet. Si le fichier contient déjà un bloc ```` ```mermaid ```` , traite-le comme une variante mermaid existant ci-dessous mais lis aussi le contexte texte autour (titres, paragraphes) pour enrichir l'analyse.

**Mermaid existant à simplifier** : peut être inline dans le chat ou dans un fichier. Si fichier, `Read`. Localise le bloc Mermaid, parse les nodes et les liens. Identifie les opportunités de simplification :

- Nodes du même acteur consécutifs → candidats à fusion
- Nodes très techniques (`API call XYZ`) → reformuler en humain
- Plus de 10 nodes principaux → réduction obligatoire
- Pas d'emojis acteurs → ajouter selon classification (Passe A)
- Pas de légende → ajouter le squelette LEG

Présente la réduction proposée dans le plan d'analyse de l'étape 3.

**Image/screenshot** : demande le path absolu. Utilise `Read` (multimodal). Identifie les éléments lisibles. Si l'image est ambiguë, demande à l'utilisateur de décrire le flow en texte. Ne pas inventer.

---

## Étape 2 — Analyse profonde en 3 passes (interne)

Cette étape se fait **dans ta tête**, pas en sortie utilisateur. C'est ici que la beta investit le temps d'analyse qui manquait dans le stable : trois passes successives sur la source avant de produire quoi que ce soit. Ne saute aucune passe — chacune répond à une question différente.

### Passe A — Lecture narrative et acteurs

Lis intégralement la source. Réponds dans ta tête :

- **Persona cible** : pour qui ce flow existe-t-il ? Une phrase humaine courte qui parle d'un besoin réel. Bon : *« Voyageur qui veut planifier un road-trip sans se prendre la tête. »* Mauvais : *« Utilisateur final du module de réservation. »* Si pas explicite, déduis du contexte : qui prend les décisions ? C'est probablement la persona.
- **Distinction** : qu'est-ce qui rend ce flow unique parmi les flows similaires ? Quel moment-clé du processus métier il capture ?
- **Étapes brutes et acteurs** : énumère toutes les étapes (pas de filtre). Pour chacune, classe l'acteur parmi 👤 client, 🤖 IA, ⚙️ système, 🖥️ UI qui s'affiche, ⚖️ décision. Pour les distinctions fines (⚙️ vs 🖥️, IA vs système) et la liste complète des verbes-signaux, lis `references/ACTORS_RULES.md`.
- **Décisions et boucles** : repère les mots-clés conditionnels (`si`, `selon`, `dans le cas où`, `ou`, `sinon`) et de boucle (`réessaie`, `recommence`, `retour à l'étape`). Une boucle = lien retour vers une étape antérieure. Limite : maximum 1 boucle par diagramme.

### Passe B — Détection des ambiguïtés de noms propres

C'est la passe critique ajoutée dans la beta. Pour un public peu technique, les noms propres d'agences, de clients ou de produits n'apportent rien et exposent au risque de confusion. Cas réel : « chatbot Mia » dans une source où « Mia » désigne l'agence (MIA Innovation, l'employeur) et « Authentik » le client final — le chatbot lui-même n'est ni l'un ni l'autre. Le label correct dans le diagramme est « le chatbot ».

Pour chaque nom propre repéré dans la source, demande-toi :

- Apparaît-il dans plus d'un rôle dans le texte ? (agence ET produit, client ET produit, personne ET fonction)
- Est-il vraiment porteur d'information pour un directeur de compte, ou est-ce du jargon interne ?

**Si tu détectes une ambiguïté** (même nom dans plusieurs rôles), garde-la pour l'étape 3 et propose un mapping générique par défaut. **Sinon, prends les noms tels quels** — ne questionne pas pour le simple plaisir de questionner.

La règle complète et les exemples sont dans `references/ACTORS_RULES.md` (section « Anti-confusion noms propres »).

### Passe C — Valeur vs bruit, et réduction à 7 étapes (cap 10)

Le public cible est un directeur de compte qui glisse ce diagramme dans un plan de projet. Il a besoin de comprendre **ce que vit le client** et **où l'IA/le système intervient** — pas les détails techniques de validation, de logging, de cache, ou les sous-étapes internes invisibles.

Pour chaque étape brute de la Passe A, demande-toi :

- Cette étape fait-elle avancer le persona vers son objectif ?
- Sa disparition rendrait-elle le diagramme incompréhensible ?

Garde uniquement les étapes qui font avancer. Le reste est regroupé sous un concept-parent ou supprimé.

Puis applique la règle de réduction :

- Si > 10 étapes brutes : fusionne les étapes consécutives du **même acteur** en une seule.
- Si toujours > 10 : élève le niveau d'abstraction (regroupe sous un concept-parent).
- Si < 5 étapes brutes : n'invente pas d'étapes. Garde tel quel, ajoute du contexte en parenthèses pour enrichir les labels.
- **Cible** : 7 étapes principales (hors décisions losanges). **Hard cap** : 10 nodes principaux.

### Choix de variante (issue des 3 passes)

À ce stade, tu sais quelle variante du catalogue conviendra le mieux. Les 5 variantes :

1. **Linéaire** : pas de décision, pas de boucle.
2. **Avec décision** : un losange Oui/Non, branches courtes (1-2 étapes chacune).
3. **Avec boucle** : retour depuis la branche Non vers une étape antérieure.
4. **Avec référence à un autre flow** : node `REF1` qui pointe vers un autre diagramme.
5. **2 chemins parallèles** *(nouveau dans la beta)* : une décision majeure qui mène à **2 sous-flows distincts de 2-3 étapes** avant convergence. À choisir quand l'analyse révèle deux possibilités métier réelles (ex : B2B vs B2C, paiement comptant vs financement, demande simple vs complexe). Cap : 2 branches max — si 3 émergent, fusionne ou documente l'écart à l'utilisateur.

Pour les squelettes Mermaid complets de chaque variante, lis `references/VARIANTS.md`.

---

## Étape 3 — Proposer le plan et valider

D'abord, présente ton plan **en texte dans le chat** :

> *« Plan d'analyse pour `<nom du flow>` :*
>
> *- **Persona** : <1 phrase>*
> *- **Distinction** : <1 phrase>*
> *- **Ambiguïtés détectées** (si Passe B en a trouvé) : <ex: « Mia » apparaît à la fois comme agence et chatbot dans la source — propose de l'afficher comme « le chatbot »>*
> *- **N étapes identifiées** :*
>   *1. 👤 <étape 1>*
>   *2. 🖥️ <étape 2>*
>   *... (jusqu'à 7-10)*
> *- **Bifurcations** : <description ou « aucune »>*
> *- **Boucles** : <description ou « aucune »>*
> *- **Variante** : <linéaire | décision | boucle | référence | 2 chemins parallèles>*
> *- **Couleurs** : palette pastel light-mode standard »*

Si la Passe B n'a rien détecté, n'inclus pas la ligne « Ambiguïtés détectées » — pas de friction inutile.

Puis utilise `AskUserQuestion` :

- **Question** : *« OK pour générer le diagramme avec ce plan ? »*
- **Options** :
  - *« Oui, génère »* (Recommended)
  - *« Ajuster »* — l'utilisateur précise via Other ce qu'il veut changer (persona, étapes, mapping de noms propres, variante)
  - *« Annuler »*

**Si Ajuster** : prends le feedback, retravaille en interne, repropose un nouveau plan. Boucle jusqu'à validation explicite.

**Si Annuler** : termine avec *« Génération annulée. »*

---

## Étape 4 — Générer le fichier markdown

### 4a. Construire la sortie

1. Charge le squelette commun et la variante choisie depuis `references/VARIANTS.md`.
2. Charge la palette de couleurs depuis `references/PALETTE.md` (`classDef`, IDs, règles légende dynamique).
3. Remplis les nodes (IDs `A`, `B`, `C`, ... single letter ; `REF1`, `REF2` pour références) avec les étapes retenues à la Passe C.
4. Applique les labels génériques retenus à la Passe B (ex: « le chatbot » plutôt qu'un nom propre ambigu).
5. Construis les liens selon la structure (linéaire, branches Oui/Non, boucle, ref, 2 chemins).
6. Classe chaque node dans exactement une classe (`clientAction`, `iaAction`, `systemAction`, `uiAction`, `decision`).
7. Construis la **légende dynamique** : ne mets que les acteurs réellement présents (si pas d'UI dans ton flow, retire `L4` et `classDef uiAction`, renumérote).
8. Remplis le titre, le blockquote contextuel, la persona en en-tête du fichier.

### 4b. Vérifications finales

- Chaque node assigné à exactement 1 classe.
- Labels ≤ 7-10 mots, emoji au début, parenthèses pour les détails.
- Légende ne contient que les acteurs présents.
- Max 10 nodes principaux (hors légende), cible 7.
- Numérotation `"1. ..."` cohérente si > 5 étapes.

Un exemple complet d'output (variante 5 — 2 chemins parallèles) est dans `assets/example_flow_legende.md`.

---

## Étape 5 — Sauvegarder et confirmer

### 5a. Déterminer l'emplacement

- Si l'input était un **fichier markdown** : propose de sauvegarder dans le **même dossier** que le fichier source, avec le suffixe `_legende.md` (ex: `flow_5.md` → `flow_5_legende.md`). Confirme via `AskUserQuestion`.
- Si l'input était **texte/transcript/mermaid/image** dans le chat : demande le path absolu via `AskUserQuestion` (pas de défaut hardcodé — le poste de travail de l'utilisateur peut ne pas avoir de `~/Desktop`).

### 5b. Écrire et confirmer

1. `Write` du fichier markdown final au path validé.
2. Confirme en une phrase : *« Flowchart sauvegardé dans `<path>`. <N> étapes, variante <linéaire|décision|boucle|ref|2 chemins>. »*
3. **Une seule fois par session**, suggère à l'utilisateur de prévisualiser avec un viewer Mermaid (mermaid.live, extension VS Code).

---

## Règles de comportement

- Ne génère pas le fichier sans validation explicite de l'utilisateur (étape 3 est obligatoire).
- N'invente pas d'étapes ni d'acteurs si la source est ambiguë — demande à l'utilisateur.
- Si tu modifies la palette de couleurs, tu casses la cohérence visuelle entre diagrammes que le directeur de compte agrège dans son plan de projet. La palette est traitée comme un acquis ; seul un utilisateur insistant peut demander un écart, et tu signales alors la perte de cohérence.
- Ne dépasse pas 10 nodes principaux hors légende. Si la source l'exige, propose de découper en plusieurs flows reliés par référence (variante 4).
- Ne commit jamais automatiquement le fichier généré.
- Si l'utilisateur veut s'écarter du format strict (ex: « ajoute des couleurs vives »), préviens que ça casse la cohérence avec les autres flows existants, mais accepte si insistance.

---

## Pourquoi ces choix

**Labels génériques par défaut** — Le cas Mia/Authentik est l'incident fondateur de cette beta. Dans la source, « Mia » désignait l'agence et « Authentik » le client final ; le chatbot lui-même n'avait pas de nom propre clair. Le diagramme stable a nommé l'IA « Mia », ce qui était doublement faux (ni l'agence ni le client). Pour un directeur de compte qui ne connaît pas l'écosystème, un label générique (« le chatbot », « l'app », « le système ») élimine ce risque sans rien coûter en clarté.

**Trois passes d'analyse explicites** — La version stable mélangeait analyse et structuration. La beta sépare : Passe A pour comprendre, Passe B pour désambiguïser les noms, Passe C pour réduire à ce qui apporte de la valeur au directeur de compte. Le séparation produit des diagrammes plus pertinents et permet à l'étape 3 de signaler explicitement chaque décision prise.

**Variante 5 — 2 chemins parallèles** — La variante « décision » du stable se résout en 1 étape par branche puis converge. Ce n'est pas représentatif des bifurcations métier réelles (B2B/B2C, simple/complexe), qui ont chacune leur propre sous-séquence. La variante 5 corrige ce manque en autorisant 2-3 étapes par branche. Cap à 2 branches max : au-delà, la vulgarisation est ratée et il vaut mieux découper en deux flows séparés.

**Palette pastel fixe** — Quand un directeur de compte agrège plusieurs flows dans un plan de projet, la cohérence visuelle est plus importante que la créativité par flow. La palette est figée pour cette raison.

**Max 10 nodes, cible 7** — Au-delà, le cerveau d'un non-technique décroche. La règle des 7 ± 2 (Miller) guide la cible.

**Validation interactive obligatoire** — Sans étape 3, le skill produit régulièrement des diagrammes hors sujet (mauvaise persona, mauvais découpage, mauvais nom propre). Le coût d'un AskUserQuestion est bien inférieur au coût de devoir re-générer.

**Progressive disclosure des références** — Templates Mermaid, palette et règles d'acteurs vivent dans `references/` plutôt qu'inline. Ils sont lus uniquement quand l'étape concernée se déclenche. Bénéfice : SKILL.md reste sous les 250 lignes, le coût en tokens à l'invocation est réduit, et chaque fichier de référence peut évoluer indépendamment.

---

## Fichiers annexes

| Fichier | Quand le lire |
|---|---|
| `references/ACTORS_RULES.md` | Passe A — classification des 5 acteurs, distinctions fines (⚙️ vs 🖥️, IA vs système). Passe B — règle anti-confusion noms propres, exemples. |
| `references/VARIANTS.md` | Étape 4a — squelettes Mermaid des 5 variantes (linéaire, décision, boucle, ref, 2 chemins parallèles) et règles de combinaison. |
| `references/PALETTE.md` | Étape 4a — couleurs exactes, `classDef`, format légende dynamique, vérifications finales. |
| `assets/example_flow_legende.md` | Étape 4 — exemple complet d'output (variante 5 avec 2 chemins parallèles). |
