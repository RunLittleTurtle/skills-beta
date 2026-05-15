# Template de synthèse exécutive (v2.2 — lisible + jugement LLM)

Ce template définit la structure du fichier `synthese_priorisation.md`. La synthèse est **un layer de jugement LLM par-dessus le CSV mécanique** : la math reste dans le CSV (source de vérité immuable), la synthèse interprète, nuance, bundle et hiérarchise au-delà des seuils.

## Règles de production

- **Langue** : français Québec
- **Pas d'em dash** (utiliser virgules, parenthèses, deux-points)
- **Symboles autorisés** : `%`, `$`, `:` pour en-têtes inline. **Pas** de `[ ]`, `|`, `≥`, `<`, `×`, séparateurs `---` entre use cases
- **Prose, pas de fragments** : phrases complètes plutôt que listes télégraphiques de métadonnées
- **Aucun anglais** : pas de "Snapshot", "Pourquoi", etc. en mode bilingue
- **Top priorités max 5** entrées (bundles comptent comme une entrée)
- **Override LLM autorisé si justifié** : promotion, rétrogradation, ou bundling permis tant que la raison est citée ET le verdict mécanique d'origine reste affiché

## Structure à reproduire

```markdown
# Synthèse de priorisation des use cases AI

Date d'analyse : [AAAA-MM-JJ].
Source des inputs : [phrase descriptive, ex: "transcript de l'atelier du 12 mai 2026 avec Valérie, Steven, Nicolas et Alex" ou "table de 247 processus du fichier inventory.csv"].
[Nombre] use cases évalués au total. [Nombre] retenus pour scoping immédiat ou prochain trimestre (verdicts ROI Quick win ou Go, confiance globale supérieure ou égale à 60%).

## Top priorités pour scoping

[Pour chaque use case retenu ou bundle, dans l'ordre LLM-justifié, max 5 entrées.]

### 1. [Nom du Use Case ou nom du Bundle]

Sponsor : [nom ou "à identifier"] ([département]).

Score Suitability de [XX] sur 100. Confiance globale de [XX]% (complétude [XX]%, validation [niveau]). Payback en [X] mois, pour un ROI première année de [XX]%, soit environ [XX] $ ou [XX] k$ de bénéfice net annuel. Run estimé à [XX] $ par an sur la base de [X.XX] $ par exécution.

Pourquoi prioritaire. [2-3 phrases. Mentionner les signaux forts cités du transcript : volume confirmé, sponsor engagé, données disponibles, alignement stratégique, pertes évitées tangibles. Citer les passages clés.]

À valider avant scoping. [Si confiance < 80% ou validation 1 ou 2, lister 1-2 hypothèses fortes ou inputs manquants. Sinon omettre cette ligne.]

[Si le ranking diffère du Score Priorité mécanique :]
Note ranking. Mécaniquement classé [rang X] par Score Priorité, promu / rétrogradé à [rang Y] ici parce que [raison citée du transcript ou OKR].

### 2. Bundle Fondations Data Lake

[Si bundling : nommer le bundle, lister les use cases inclus, expliquer la fondation partagée et le gain de bundling.]

Cas inclus : [Nom 1], [Nom 2], [Nom 3]. Sponsor commun ou coordonné : [nom].

Verdicts mécaniques individuels : [Nom 1] Quick win, [Nom 2] À challenger, [Nom 3] Pass. Le bundling rend [Nom 3] viable en mutualisant l'infrastructure Data Lake construite pour [Nom 1 et 2].

[Continuer avec snapshot et justification comme un use case standard, mais en mentionnant l'effet de mutualisation.]

[Continuer jusqu'à max 5 entrées Top.]

## Lectures transversales

Patterns que l'analyse révèle entre les use cases du portefeuille.

- **Intégrations partagées** : [ex: "5 use cases sur 18 dépendent de Business Central. Construire une API Gateway commune divise le coût d'intégration de chaque agent par 3."]
- **Sponsors récurrents** : [ex: "Valérie est sponsor de 4 use cases Finance/Production. Une session de scoping groupée gagne 2 semaines."]
- **Fondations communes** : [ex: "Le Data Lake mentionné dans 3 ateliers débloque les agents RAG, Estimation et Flagging. À construire en priorité même s'il n'est pas un quick win autonome."]
- **Effets de séquence** : [ex: "Faire OCR Dockets avant Planification AI : le premier produit la donnée structurée que le second consomme."]

## À éliminer

Use cases avec verdict mécanique Pass et sans angle de re-scoping. Liste à puces, une ligne par cas.

- **[Nom]**. [Raison : payback supérieur à 24 mois, ROI négatif structurel, volume trop bas, etc.]

[Override LLM possible : si un use case Pass mérite quand même un retour plus tard.]

- **[Nom]**. Verdict mécanique Pass mais à reconsidérer si [condition externe : ex: "Sabre ne livre pas le feature WIP dans Q3 2026"], ou si [le scope est réduit aux 2 machines critiques].

## À retravailler avant décision

Use cases avec confiance globale inférieure à 60%, hors top et à éliminer. Liste à puces, 1-2 lignes par cas.

- **[Nom]** (confiance [XX]%). [Ce qui manque exactement et qui doit le fournir. Ex: "Volumes à confirmer par Nicolas, sponsor à identifier côté direction Vente, complexité technique à évaluer avec architecte."]

## Recommandations stratégiques

Section qualitative. Peut diverger des verdicts ROI mécaniques quand le LLM identifie un effet de portefeuille, une fenêtre client, ou une dépendance critique.

- **Ordre de lancement recommandé** : [si différent du Score Priorité pur, expliquer]. Exemple : "Commencer par le bundle Fondations Data Lake (M3 à M6), puis enchaîner avec les agents qui en dépendent (M6 à M12). Le Quick win Email auto en parallèle dès M1."
- **Pré-requis à débloquer** : [ex: "Identifier le sponsor du Chatbot RH avant de scoper, sinon le projet stagnera côté adoption."]
- **Risques de portefeuille** : [ex: "4 use cases sur 5 du top dépendent de Business Central. Diversifier l'intégration ou prévoir un fallback si BC est en migration en 2027."]
- **Fenêtres clients** : [ex: "Valérie pousse le Dashboard recoster comme premier livrable pour la revue conseil d'octobre. Si tenu, débloque le budget des phases 2 et 3."]

## Méthodologie

Cette priorisation combine quatre signaux :
- Suitability score qualitatif sur 8 critères notés 1 à 3 (volume, règles, données, stabilité, exceptions, impact, alignement, sponsorship)
- Business case quantitatif basé sur volume fois temps fois coût horaire, avec haircut de réalisation (Prosci 50 à 70%)
- Run cost benchmarké en direct par recherche web LLM ($/run par type d'agent : email auto, dashboard genAI, RAG, OCR, RPA, etc.)
- Score de confiance hybride : pourcentage des 20 colonnes critiques remplies, multiplié par un facteur de validation LLM (1.0 si source primaire validée, 0.75 si mixte, 0.5 si estimation contextuelle)

Pour le détail complet des 38 colonnes du CSV, des formules et des seuils de verdict, voir `priorisation_use_cases.csv` (lignes FORMULES et SOURCE) et `references/framework_v5.md`.

## Sources benchmark consultées

Voir le fichier `sources_benchmark.md` du même dossier pour la liste complète des benchmarks $/run utilisés, avec source et date de consultation.
```

## Notes sur les cas particuliers

**Si aucun use case ne passe le filtre (tout est en Pass ou confiance trop basse)** :
Remplacer la section Top priorités par :
```
## Aucun use case ne passe les filtres actuels

Avec les inputs fournis, aucun use case n'atteint à la fois un payback inférieur à 24 mois et une confiance globale supérieure ou égale à 60%. Recommandations :

1. [Use case avec le meilleur score parmi les à retravailler] serait le candidat principal si [élément manquant] était validé.
2. Organiser un atelier de chiffrage avec [stakeholders pertinents] pour collecter [données manquantes].
```

**Si un seul use case est fourni** :
Pas de section Top priorités numérotée. Faire une analyse approfondie en une seule fiche détaillée, avec recommandations stratégiques adaptées (timeline, sponsor, dépendances).

**Si l'input ne permet pas de calculer le ROI (volumes manquants partout)** :
Produire la synthèse en se basant sur Suitability plus Confiance seulement. Documenter clairement que le ROI ne peut être calculé sans les volumes, et lister les données à collecter en priorité dans la section "À retravailler avant décision".

**Si le LLM override le ranking mécanique** :
La promotion ou rétrogradation est tracée par la ligne "Note ranking" sous le use case concerné. Cette transparence permet au lecteur de pondérer la subjectivité LLM par rapport à la math pure du CSV.

**Si le LLM bundle plusieurs use cases** :
Le bundle est nommé explicitement ("Bundle Fondations Data Lake"). Les use cases inclus sont listés. Les verdicts mécaniques individuels sont rappelés. L'effet de mutualisation est expliqué (intégration partagée, sponsor commun, économies d'échelle).
