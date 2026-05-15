# Variantes Mermaid — squelettes et règles de combinaison

Lu à l'étape 4a de SKILL.md, après que la Passe C ait identifié la variante adaptée. Chaque variante hérite du même squelette commun (légende + flow + classDef + style). Seul le contenu des subgraph `FLOW` change.

---

## Squelette commun (toujours utilisé comme base)

````markdown
# Flow <X-S> - <titre court>

> <description du contexte en 1-2 phrases>
>
> **Persona : <persona en 1 phrase>**

```mermaid
%%{init: {'flowchart': {'curve': 'basis'}}}%%
flowchart TB
    subgraph LEG["Légende"]
        direction LR
        L1["👤 Action du client"] ~~~ L2["🤖 Action de l'assistant IA"] ~~~ L3["⚙️ Action du système"] ~~~ L4["🖥️ UI qui s'affiche"] ~~~ L5["⚖️ Décision"]
    end

    subgraph FLOW[" "]
        direction LR
        <NODES>

        <LIENS>
    end

    LEG ~~~ FLOW

    classDef clientAction fill:#E1BEE7,stroke:#6A1B9A,color:#000
    classDef iaAction fill:#BBDEFB,stroke:#1565C0,color:#000
    classDef systemAction fill:#CFD8DC,stroke:#455A64,color:#000
    classDef uiAction fill:#C8E6C9,stroke:#2E7D32,color:#000
    classDef decision fill:#FFF9C4,stroke:#F57F17,color:#000

    class <IDS_CLIENT> clientAction
    class <IDS_IA> iaAction
    class <IDS_SYSTEM> systemAction
    class <IDS_UI> uiAction
    class <IDS_DECISION> decision
    class L1 clientAction
    class L2 iaAction
    class L3 systemAction
    class L4 uiAction
    class L5 decision

    style FLOW fill:none,stroke:none
```
````

La légende n'inclut que les acteurs réellement utilisés dans le flow. Si pas d'UI : retire `L4`, la `classDef uiAction`, et renumérote `L5` décision en `L4`. Idem pour les autres acteurs absents.

---

## Variante 1 — Linéaire simple

Pas de décision, pas de boucle. Le flux avance d'une étape à la suivante sans bifurcation.

```
A["👤 1. Clique 'Générer avec IA'"]
B["🖥️ 2. Fenêtre conversationnelle s'ouvre"]
C["👤 3. Saisit ses critères (dates, voyageurs)"]
D["🤖 4. Analyse les besoins et propose 3 options"]
E["👤 5. Choisit une option"]
F["⚙️ 6. Sauvegarde dans le compte"]
G["🖥️ 7. Affiche le résumé final"]

A --> B --> C --> D --> E --> F --> G
```

---

## Variante 2 — Avec décision (courte)

Un losange Oui/Non qui converge rapidement. Les deux branches sont courtes (1 étape chacune) avant convergence ou re-bouclage.

```
A["👤 1. Saisit sa demande"]
B["🤖 2. Analyse la demande"]
C{"⚖️ Demande complète ?"}
D["🤖 3. Génère la proposition"]
E["🤖 3'. Demande des précisions"]
F["👤 4. Valide ou ajuste"]
G["⚙️ 5. Sauvegarde"]

A --> B --> C
C -->|"Oui"| D
C -->|"Non"| E
E --> A
D --> F --> G
```

---

## Variante 3 — Avec boucle

Retour explicite vers une étape antérieure, typiquement depuis la branche « Non » d'une décision. **Limite stricte : 1 boucle maximum par diagramme** — plus = vulgarisation ratée.

```
A["👤 1. Soumet une version"]
B["🤖 2. Analyse et donne feedback"]
C{"⚖️ Satisfait ?"}
D["👤 3. Ajuste"]
E["⚙️ 4. Sauvegarde la version finale"]

A --> B --> C
C -->|"Non"| D
D --> B
C -->|"Oui"| E
```

---

## Variante 4 — Avec référence à un autre flow

**Quand l'invoquer en priorité** : si la source mentionne explicitement un autre flow par un en-tête distinct (`FLOW 2`, `SCÉNARIO 2`, `SÉQUENCE ALTERNATIVE`, `LOOP DE MODIFICATION`), c'est le signal le plus net pour utiliser cette variante. Le node `REF1` économise 1 à 3 nodes du budget principal tout en préservant la trace de la suite. Voir SKILL.md, sous-passe C-bis.

Node `REF1["⚙️ Voir flow X — <titre>"]` qui pointe vers un autre diagramme. Utile quand le processus complet est découpé en plusieurs flows reliés. Classe `REF1` en `systemAction` (gris) — pas de classDef séparée nécessaire.

```
A["👤 1. Lance la modification"]
REF1["⚙️ Voir flow 2-S — Création initiale"]
B["🤖 2. Récupère le contexte"]
C["👤 3. Sélectionne ce qui change"]
D["⚙️ 4. Met à jour"]

A --> REF1 --> B --> C --> D
```

---

## Variante 5 — 2 chemins parallèles (nouveau dans la beta)

Une décision majeure qui mène à **2 sous-flows distincts de 2-3 étapes chacun** avant convergence. À choisir quand l'analyse révèle deux possibilités métier réelles : B2B vs B2C, paiement comptant vs financement, demande simple vs complexe, parcours guidé vs parcours autonome.

```
A["👤 1. Soumet sa demande"]
B["🤖 2. Analyse le contexte"]
C{"⚖️ Type de demande ?"}
D["🤖 3a. Génère la proposition complète"]
E["👤 4a. Valide en un clic"]
F["🤖 3b. Pose des questions de précision"]
G["👤 4b. Répond aux questions"]
H["🤖 5b. Affine la proposition"]
I["⚙️ 6. Sauvegarde le résultat"]

A --> B --> C
C -->|"Simple"| D --> E --> I
C -->|"Complexe"| F --> G --> H --> I
```

**Règles de la variante 5** :

- **2 branches max**. Si l'analyse en révèle 3 ou plus, fusionne 2 d'entre elles ou signale-le à l'utilisateur en étape 3 pour qu'il choisisse les 2 plus représentatives.
- **2-3 étapes par branche**. Une branche d'1 étape relève de la variante 2 (décision courte). Une branche de 4+ étapes signale qu'il faut découper en 2 flows reliés (variante 4).
- **Convergence obligatoire** vers un node de résultat commun (`I` dans l'exemple). Sinon ce sont deux flows distincts, pas une variante 5.
- **Labels de branche descriptifs** (`Simple` / `Complexe`, `B2B` / `B2C`), pas `Oui` / `Non` — sinon c'est une variante 2.
- **Numérotation** : `3a` / `4a` pour la branche 1, `3b` / `4b` / `5b` pour la branche 2, puis numéro commun (`6`) après convergence.

---

## Règles de combinaison

Une seule **complexité structurelle dominante** par diagramme (la variante choisie : boucle, ou 2 chemins, ou ref externe). Cette complexité dominante est ce que le lecteur retient à première lecture.

**Mais une décision secondaire courte est autorisée** en complément, à condition de respecter ces contraintes :

- **1 décision secondaire max** en plus de la dominante.
- **Branches courtes** : 1 étape par branche (typiquement 2 boutons terminaux, 2 actions finales du client). Au-delà, ce n'est plus secondaire — c'est la variante 5.
- **Position préférée** : en début ou en fin de flow, pas au milieu (sinon ça vole la vedette à la variante dominante).
- **La décision dominante reste un losange explicite** — ne fusionne pas une décision en un node UI ou IA juste pour éviter une combinaison. Si la source montre vraiment 2 boutons, garde un losange.

Exemples acceptés :

- Variante 1 (linéaire) + losange terminal « ⚖️ Valide ou rejette ? » avec 2 actions client courtes en sortie.
- Variante 3 (boucle) + losange terminal « ⚖️ Sauvegarder ou exporter ? » avant la fin.
- Variante 5 (2 chemins) + losange terminal commun après convergence.

Exemples refusés (vraies combinaisons à découper) :

- 2 chemins parallèles + boucle au milieu d'une branche → variante 5 dominante, supprime la boucle ou découpe en 2 flows reliés par référence (variante 4).
- Décision majeure au milieu + boucle qui repart de plus loin → variante 3 dominante, déplace la décision en amont, ou découpe.
- 2 décisions majeures dans un même flow → fusionne-les en une seule décision majeure en amont, ou découpe.

La raison : un diagramme pour un directeur de compte porte une complexité principale (ce qui rend ce flow unique) et tolère un détail terminal de plus (les 2 options finales typiques). Au-delà, c'est le signal qu'il faut découper en flows reliés par référence — et c'est précisément l'usage du dossier proposé en étape 5a du SKILL.md.
