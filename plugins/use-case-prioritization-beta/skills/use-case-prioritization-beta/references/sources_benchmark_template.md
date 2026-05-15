# Sources benchmark Coût Unitaire Agent (template)

Ce fichier compile toutes les sources web utilisées pour les valeurs **Coût Unitaire Agent ($/run)** dans la priorisation. Il sert d'**audit trail** : permet à un reviewer de remonter à la source, à un mainteneur de mettre à jour les benchmarks trimestriellement, et au client de comprendre d'où viennent les chiffres.

## Règles de production

- **Une section par Type Agent unique** rencontré dans le CSV
- **Pour chaque benchmark** : source (URL ou nom du rapport), date de consultation, valeur retenue
- **Médiane** : si plusieurs sources, indiquer la médiane utilisée
- **Fallback** : si le benchmark est `BENCHMARK_FALLBACK` (recherche infructueuse), le mentionner explicitement
- **Fraîcheur** : sources de moins de 12 mois (règle interne agence MIA sur la fraîcheur LLM)

## Structure à reproduire

```markdown
# Sources benchmark — [Projet] ([YYYY-MM-DD])

Date d'analyse : [AAAA-MM-JJ].
Use cases concernés : [nombre de lignes du CSV].

## email auto — $X.XX/run (médiane)

- [Nom du rapport ou de la page], [date de publication ou édition], consulté le [YYYY-MM-DD]
  - URL : [si applicable]
  - Valeur citée : $X.XX par exécution
  - Contexte : [1 phrase, ex: "agent triage et drafting Claude Sonnet avec cache prompt 90% off"]

- [Source 2]...

**Médiane retenue** : $X.XX par exécution.

## dashboard genAI — $X.XX/run (médiane)

[Même structure]

## RAG simple — $X.XX/run (médiane)

[Même structure]

## [autres types agent rencontrés]

[Même structure]

---

## Fallbacks utilisés (si applicable)

[Si une ligne du CSV a flag `BENCHMARK_FALLBACK` dans Notes, l'expliquer ici.]

- Use case [Nom] : Type Agent `autre` non standard, recherche web infructueuse. Fallback `(Build × 0.15) / Volume Qté` appliqué. Recommandation : faire estimer par l'architecte tokens en suivi.

---

## Mise à jour suggérée

Ces benchmarks devraient être revus chaque trimestre. Les prix d'inference LLM évoluent rapidement (baisses 2x à 3x par an sur les modèles de pointe). À la prochaine session de priorisation, vérifier si les valeurs ci-dessus sont encore valides ou si une nouvelle recherche web s'impose.

Sources prioritaires à monitorer :
- a16z State of AI (rapport annuel)
- Menlo Ventures AI infrastructure reports
- Pricing officiels : OpenAI, Anthropic, Google
- Blogs ingénierie produit : Ramp, Notion, Vercel, Linear (use cases AI réels avec chiffres)
```

## Notes de production

**Si un Type Agent revient dans plusieurs use cases**, ne pas dupliquer la section : une seule section par type, le LLM s'y réfère pour toutes les lignes concernées.

**Si la valeur a été surchargée par l'architecte** (override manuel parce qu'il connaît mieux le contexte tokens du projet), le mentionner clairement :

```markdown
## document processing — $0.15/run (override architecte)

Note : la recherche web suggérait $0.05 à $0.20 (Rossum + ABBYY + Claude RAG, consulté 2026-05-13). L'architecte (Maxime) a surchargé à $0.15 sur la base de son estimation tokens pour le contexte Faspac (documents PDF + GPT-4 vision pour layout complexe).
```

**Si le client a déjà des chiffres internes**, les utiliser en priorité et le noter :

```markdown
## OCR — $0.02/run (chiffre client)

Le client utilise déjà Microsoft AI Builder à $0.02 par document (facture interne 2026-Q1, partagée par Shirley). Aligné avec le benchmark web Rossum (consulté 2026-05-13).
```
