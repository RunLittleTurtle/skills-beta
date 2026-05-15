# Acteurs — classification et anti-confusion noms propres

Lu en Passe A et en Passe B de l'étape 2 de SKILL.md. Deux sections : la classification des 5 acteurs (héritée du stable, affinée), et la règle anti-confusion noms propres (nouveau dans la beta).

---

## Les 5 acteurs

### 👤 Client

Verbe d'action humain effectué par la personne qui utilise le produit. Verbes-signaux : *saisit, choisit, valide, lit, refuse, clique, glisse, sélectionne, demande, soumet, ajuste*. Le client est toujours celui qui vit le besoin que le flow résout — pour les flows internes d'entreprise (ex: PO qui configure un produit), c'est cette PO qui est « le client » du flow, pas le client final de l'entreprise.

### 🤖 IA

Analyse contextuelle, génération, suggestion, classification, traduction. Verbes-signaux : *analyse, propose, génère, recommande, traduit, résume, infère, classe*. Au cœur : tout ce qui implique un LLM ou un modèle qui doit « comprendre » avant de produire.

### ⚙️ Système

Job automatique invisible, sans intelligence. Verbes-signaux : *sauvegarde, calcule, envoie, met à jour la BD, archive, supprime, déclenche, route*. Backend silencieux : l'utilisateur ne voit rien apparaître à l'écran à cause de cette étape.

### 🖥️ UI qui s'affiche

Écran, modal, fenêtre, panneau visible qui apparaît à l'utilisateur. Verbes-signaux : *s'ouvre, s'affiche, apparaît, montre, présente, défile, charge à l'écran*. Indicateur : si on retire cette étape, l'utilisateur ne voit pas une information importante.

### ⚖️ Décision

Question conditionnelle fermée. Forme : losange `{"⚖️ <Question> ?"}`. Toujours une question qui se résout par Oui/Non, ou par un label descriptif court (`Simple` / `Complexe`, `B2B` / `B2C`).

---

## Distinctions critiques

### ⚙️ Système vs 🖥️ UI

Le piège le plus fréquent. La règle : ⚙️ est silencieux (backend, invisible), 🖥️ est visible (quelque chose apparaît à l'écran). Si l'étape suivante du client dépend de voir une info à l'écran, c'est 🖥️. Si l'étape est purement technique (recalcul, sauvegarde, sync), c'est ⚙️.

Exemple : « le compte est créé » → 🖥️ si l'utilisateur voit l'écran de confirmation, ⚙️ si c'est juste une ligne en BD qui sert à autre chose plus loin.

### 🤖 IA vs ⚙️ Système

Si l'étape implique un LLM, une classification contextuelle, ou de la génération adaptative → 🤖. Si c'est un job automatique sans intelligence (regex, formule, requête fixe) → ⚙️.

Exemple : « le système trie par date » → ⚙️. « Le système propose les 3 plus pertinents » → 🤖 (la pertinence demande un jugement).

---

## Anti-confusion noms propres (nouveau dans la beta)

C'est la règle ajoutée pour corriger le cas Mia/Authentik. Pour un public peu technique (parties prenantes, directeurs de compte), les noms propres d'agences, de clients ou de produits n'apportent rien et exposent au risque de confusion.

### Le cas fondateur

Source d'un scénario : « le chatbot Mia analyse la demande pour Authentik ». Dans le contexte réel :

- **Mia** = l'agence (MIA Innovation, employeur de la PO qui rédige le scénario).
- **Authentik** = le client final qui recevra le chatbot.
- **Le chatbot** = le produit construit par l'agence Mia pour le client Authentik.

Le diagramme stable a nommé l'IA « Mia », ce qui était doublement faux : ni l'agence ni le client ne désignent le chatbot lui-même. Le label correct était simplement « le chatbot ».

### Règle

Préfère par défaut les labels génériques :

- *le chatbot, l'app, le système, l'IA, l'assistant, le produit*

Plutôt que les noms propres. Le directeur de compte qui glisse le diagramme dans son plan de projet n'a pas le contexte interne et confondra facilement les rôles.

### Quand questionner l'utilisateur

À la Passe B de l'étape 2, repère chaque nom propre dans la source. Pour chacun :

- Est-il porteur d'information pour un directeur de compte externe ?
- Apparaît-il dans plus d'un rôle dans la source (agence ET produit, client ET produit, personne ET fonction) ?

**Si oui à au moins une question, c'est une ambiguïté détectée**. Liste-la à l'étape 3 du SKILL.md avec un mapping générique proposé. Exemple :

> *« Ambiguïté détectée : « Mia » apparaît dans la source à la fois comme nom d'équipe et comme nom du chatbot. Je propose d'afficher le chatbot comme « le chatbot » dans le diagramme. Tu confirmes ou tu préfères un nom propre spécifique ? »*

**Si non aux deux questions, prends le nom tel quel** — ne questionne pas pour le simple plaisir de questionner. Exemple acceptable sans question : un flow B2B où apparaît « Salesforce » sans ambiguïté possible (c'est clairement l'outil CRM).

### Quand l'utilisateur insiste pour un nom propre

S'il dit « non, mets vraiment "Authentik" », accepte. Mais signale en une phrase : *« Note : le diagramme sera moins lisible pour un lecteur qui ne connaît pas Authentik. »* Pas d'insistance au-delà.

---

## Patterns de classification rapide

Liste mentale pour la Passe A, à utiliser quand le verbe est ambigu :

- *« L'utilisateur voit X »* → toujours 🖥️ (ce qui s'affiche).
- *« Le produit propose X »* → presque toujours 🤖 (proposition = jugement).
- *« Le système enregistre X »* → presque toujours ⚙️ (sauvegarde silencieuse).
- *« L'utilisateur configure X »* → 👤 (action humaine).
- *« Si X alors Y »* → ⚖️ (décision).
- Verbe IA fréquent : *analyse, propose, suggère, identifie, classe, génère, résume*.
- Verbe système fréquent : *sauvegarde, archive, route, met à jour, supprime, déclenche*.
- Verbe UI fréquent : *s'ouvre, s'affiche, apparaît, charge*.
