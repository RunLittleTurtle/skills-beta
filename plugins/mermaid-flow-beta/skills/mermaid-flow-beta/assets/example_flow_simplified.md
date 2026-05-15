# Flow 3-S — Demande client traitée par le chatbot

> Le client soumet une demande au chatbot ; selon la complexité détectée, soit le chatbot répond directement, soit il pose des questions de précision avant de produire une réponse. Le résultat est ensuite sauvegardé dans l'historique de conversation.
>
> **Persona : Utilisateur qui consulte un chatbot pour obtenir une réponse rapide sans avoir à formuler tous les détails dès le départ.**

```mermaid
%%{init: {'flowchart': {'curve': 'basis'}}}%%
flowchart TB
    subgraph LEG["Légende"]
        direction LR
        L1["👤 Action du client"] ~~~ L2["🤖 Action de l'assistant IA"] ~~~ L3["⚙️ Action du système"] ~~~ L4["⚖️ Décision"]
    end

    subgraph FLOW[" "]
        direction LR
        A["👤 1. Soumet sa demande"]
        B["🤖 2. Analyse le contexte"]
        C{"⚖️ Type de demande ?"}
        D["🤖 3a. Génère la réponse complète"]
        E["👤 4a. Valide en un clic"]
        F["🤖 3b. Pose des questions de précision"]
        G["👤 4b. Répond aux questions"]
        H["🤖 5b. Affine la réponse"]
        I["⚙️ 6. Sauvegarde dans l'historique"]

        A --> B --> C
        C -->|"Simple"| D --> E --> I
        C -->|"Complexe"| F --> G --> H --> I
    end

    LEG ~~~ FLOW

    classDef clientAction fill:#E1BEE7,stroke:#6A1B9A,color:#000
    classDef iaAction fill:#BBDEFB,stroke:#1565C0,color:#000
    classDef systemAction fill:#CFD8DC,stroke:#455A64,color:#000
    classDef decision fill:#FFF9C4,stroke:#F57F17,color:#000

    class A,E,G clientAction
    class B,D,F,H iaAction
    class I systemAction
    class C decision
    class L1 clientAction
    class L2 iaAction
    class L3 systemAction
    class L4 decision

    style FLOW fill:none,stroke:none
```

---

## Notes sur cet exemple

- **Variante 5 — 2 chemins parallèles** : deux sous-flux distincts (`Simple` 2 étapes / `Complexe` 3 étapes) qui convergent vers le même node `I`.
- **Label générique « le chatbot »** : nulle part dans le diagramme on ne voit le nom de l'agence ou du client. Si la source disait « le chatbot Mia analyse pour Authentik », la Passe B aurait détecté l'ambiguïté et proposé ce mapping.
- **Pas de 🖥️ UI** : aucun écran n'apparaît explicitement dans le flow. La légende dynamique omet donc `L4 uiAction` et renumérote `Décision` en `L4`.
- **Numérotation `3a/4a` et `3b/4b/5b`** : marque visuellement les deux branches, avec convergence sur `6` après réunion.
- **Labels descriptifs des branches** (`Simple` / `Complexe`) plutôt que `Oui` / `Non` : signal que c'est une variante 5, pas une variante 2.
