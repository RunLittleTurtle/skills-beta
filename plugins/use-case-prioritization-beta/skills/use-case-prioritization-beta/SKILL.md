---
name: use-case-prioritization-beta
description: Score, prioritize, and build business cases for AI/automation use cases from any process discovery input (post-it photos, atelier transcripts, process inventories of up to 500+ rows, meeting notes, or mixed workshop materials). Combines BABOK Business Case methodology, UiPath Suitability scoring on a /3 scale with inline qualitative labels, fully-loaded cost calculations, and a Run cost benchmarked live by web search per agent type (email auto, dashboard genAI, RAG, OCR, RPA, etc.). Use this skill whenever the user has process discovery materials and needs to decide which use case to scope first, build an enveloppe budgétaire, or triage an automation pipeline. Triggers on keywords like prioriser, prioritization, use case, ROI, payback, business case, suitability, scoping, enveloppe budgétaire, quel use case en premier, automation pipeline, process triage, atelier découverte, post-its, cartographie processus, chiffrer use case. Always outputs both a working CSV (38-column v5 schema (with hybrid completeness × validation confidence) with FORMULES and SOURCE rows) and an executive summary markdown in French Quebec.
---

# Use Case Prioritization Framework (v5)

This skill turns process discovery materials into a structured priorisation artifact for Product Managers and Functional Analysts who need to decide which AI/automation use case to scope first.

## What this skill produces

Two files, always:

1. **`priorisation_use_cases.csv`** (38 columns, French Quebec headers): the working file for the analyst. Includes FORMULES row (row 2) and SOURCE row (row 3) under the header for traceability.

2. **`synthese_priorisation.md`**: executive summary for directeurs de compte and stakeholders. Top 5 priorities with 2-3 lines each, plus "À éliminer" and "À retravailler" sections.

## Cell format for qualitative columns (v5 important)

The 9 qualitative columns (cols 6-13 Suitability criteria + col 32 Risque) use the format **`<chiffre> - <label>`** in the cell, e.g. `3 - récurrent standardisé`, `2 - moyen (rework)`, `1 - faible`. The number drives the math, the label drives human readability. No separate legend column.

When computing averages or formulas, parse the integer prefix before ` - ` (space-dash-space). Empty cells or cells without a valid `1`, `2`, or `3` prefix count as NOT filled for the Confiance score.

## Always start by reading the framework

Before processing any input, read `references/framework_v5.md` in full. It explains the 37 columns, their sources (BABOK, UiPath, SAFe, DAMA-DMBOK, etc.), the calculation logic, and the qualitative value labels.

Then read `references/calculation_rules.md` for the exact formulas, default values, verdict thresholds, and the agent benchmark fallback table.

## Inputs this skill accepts

- **Photos of post-its / whiteboards** from ateliers de découverte
- **Transcripts** of meetings with stakeholders or architects (any language, but output in French Quebec)
- **Process inventory tables** (Excel/CSV with up to 500+ processes)
- **Free-form notes** from workshops
- **Mixed inputs** (combination of any of the above)

If no input is provided, ask the user what materials to work from. Do not invent use cases.

## Workflow

### Step 0: Detect project name and create output folder (v2.2)

Before producing anything, **infer a short project label** from the inputs to name the output folder. Look in order at:
1. The filename of the main input (ex: `faspac_atelier.md` → `faspac`)
2. The recurring client entity name in the transcript (ex: a company name mentioned in every paragraph)
3. The folder where inputs were found

Form the output folder path: `/mnt/user-data/outputs/priorisation_<projet>_<YYYY-MM-DD>/`.

If no clear project label can be inferred, fall back to: `/mnt/user-data/outputs/priorisation_<YYYY-MM-DD>_<HHMM>/`.

**Announce the chosen folder name to the user before writing anything**. Example: `"Je vais écrire les livrables dans priorisation_faspac_2026-05-13/. Tu peux renommer après ou me dire de changer maintenant."`

Create the folder (and parents if needed) before Step 7 writes inside it. All artifacts (`priorisation_use_cases.csv`, `synthese_priorisation.md`, `sources_benchmark.md`) go into this folder.

### Step 1: Inventory the use cases

For each input source:
- A use case is a **centered concept** (process, task, workflow) with associated pain points, volumes, stakeholders, and impact indicators
- In post-it diagrams, use cases are typically the central blue/dominant notes with arrows pointing in (causes) and out (pain points or consequences)
- In transcripts, use cases are the topics being discussed for automation
- In tables, each row is typically a use case (verify column meaning first)

Create one row per use case. Do not duplicate. If two sources describe the same process, merge their information into one row.

### Step 2: Extract what is stated, leave the rest empty

For each use case, fill columns based on what the input **explicitly states or strongly implies**. Never fabricate quantitative data. Empty cells correctly lower the Confiance Données score and signal what needs to be validated.

**Qualitative scores (cols 6-13 Suitability and col 32 Risque)** can be estimated from context. If the input mentions "errors every day" and "20 people involved", you can reasonably estimate Impact Erreurs Actuelles = `2 - moyen (rework)` and Volume Standardisation = `3 - récurrent standardisé`. **Always write the cell as `<chiffre> - <label>`**, never just the chiffre. Document the reasoning in the Notes column.

**Quantitative inputs (volumes, hours, costs)** must come from the input. If the transcript says "50k emails/an", use 50000. If the input has no time data, leave Temps Actuel empty.

### Step 3: Categorize the agent type (col 26)

For each use case, pick the most fitting **Type Agent** value:
- `email auto` (email triage, drafting, response)
- `dashboard genAI` (custom dashboards, report generation)
- `RAG simple` (single doc Q&A)
- `RAG complexe` (multi-doc, multi-source retrieval with reasoning)
- `OCR` (document scan to data)
- `RPA déterministe` (rule-based screen automation, no LLM)
- `agent conversationnel` (chatbot, virtual assistant)
- `document processing` (extraction, classification, summarization)
- `autre` (if none fits, describe in Notes)

This drives the benchmark search in Step 5.

### Step 4: Apply the formulas

Once inputs are extracted and the agent type is categorized, calculate the derived columns using formulas from `references/calculation_rules.md`:

- Suitability Score (col 14) — `MOYENNE(parseNumber(cols 6-13)) × 33.33`
- Gain Heures Annuel (col 20) — `Volume Qté × (T_actuel − T_cible)`
- Bénéfice Brut Annuel (col 21)
- Bénéfice Net Annuel (col 23)
- Build Base via T-shirt lookup (col 25)
- Coût An 1 Total (col 29) — *requires Run Annuel from Step 5*
- Payback (col 30), ROI An 1 (col 31) — *also depend on Step 5*
- Score Priorité (col 33)
- Confiance Données (col 34) — counting filled critical columns
- Validation LLM (col 35) — *self-assessment, see Step 6*
- Verdict Confiance (col 36), Verdict ROI (col 37)

Cols 28-31 depend on the benchmark from Step 5 — compute those after Step 5 completes.

### Step 5: Benchmark the agent run cost (NEW v5, web search)

For each row, on the basis of col 26 Type Agent, look up the **Coût Unitaire Agent ($/run)** via web search.

1. Search queries to run :
   - `"[type agent] cost per run 2026 benchmark"`
   - `"[type agent] LLM inference cost per call site:a16z.com OR site:menlovc.com"`
   - `"[type agent] API pricing 2026"`
2. Prioritize sources: official pricing pages (OpenAI, Anthropic), recent reports from a16z / Menlo / Sequoia, engineering blogs from product companies (Ramp, Notion, Vercel, etc.) dated within the last 12 months.
3. Take the **median** of credible benchmarks found. Fill col 27 Coût Unitaire Agent.
4. **Cite source and date in col 37 Notes**, e.g.: `Benchmark Run: $0.04/email auto (a16z State of AI 2026, consulté 2026-05-13)`.
5. If no credible benchmark is found, use fallback `(Build × 0.15) / Volume Qté` and flag `BENCHMARK_FALLBACK` in Notes. A fallback table per agent type lives in `calculation_rules.md`.

Compute col 28 `Run Annuel = Volume Qté × Coût Unitaire Agent`, then finish the dependent columns (29 Coût An 1, 30 Payback, 31 ROI, 37 Verdict ROI).

### Step 6: Self-assess Validation LLM (NEW v2.1)

For each row, fill **col 35 Validation LLM** with `<n> - <label>` based on the **fraction of values that came from the source vs LLM contextual estimation** :

- `3 - source primaire validée` : sponsor explicitly named AND present at workshop, Volume Qté + Temps OR Pertes Évitées explicitly cited in transcript, Pain Point cited verbatim.
- `2 - mixte source + estimation` : sponsor named but engagement inferred, 1-2 key numbers from transcript with others extrapolated, Pain Point reformulated from context.
- `1 - estimation contextuelle` : sponsor "À identifier" or inferred from department, Volume/Temps/Pertes all estimated, Pain Point reformulated, Suitability inferred more than validated.

**Always justify the chosen level in col 38 Notes**, with 1-2 transcript quotes or a clear note about what was estimated vs. validated. Example: `Validation 2: volume 50k cité par Marie DAF (transcript), mais coût horaire 60$ et taux 0.6 sont défauts agence non confirmés.`

The Verdict Confiance (col 36) reads both Confiance Données (col 34) and Validation LLM (col 35) via the multiplier rule in `calculation_rules.md`:
- Validation `3` → multiplier 1.0
- Validation `2` → multiplier 0.75
- Validation `1` → multiplier 0.5

So Confiance Globale = Confiance Données × Multiplier, and the verdict threshold applies to that.

### Step 7: Build the CSV

Use `references/csv_template.csv` as the starting structure. It contains:
- Row 1: Header (38 columns)
- Row 2: FORMULES (do not modify)
- Row 3: SOURCE (do not modify)
- Row 4: example use case (replace or remove)

Append data rows (one per use case). Save to **the project folder created in Step 0**: `<output_folder>/priorisation_use_cases.csv`.

### Step 8: Generate the executive summary (with LLM judgment layer, v2.2)

Use `references/exec_summary_template.md` for the structure. The synthesis is a **LLM judgment layer on top of the mechanical CSV**. The CSV is the immutable source of numerical truth; the synthesis interprets, nuances, bundles, and prioritizes beyond mechanical thresholds.

**Key rules** :
- French Quebec, no em dashes (use commas, parentheses, or colons instead)
- **Allowed symbols** : `%`, `$`, `:` for inline headers. **No brackets** `[ ]`, **no pipes** `|`, **no** `≥`, `<`, `×`, **no** `---` separators between use cases
- Prose, not telegraphic fragments — full sentences instead of inline metadata pipes
- Maximum 5 entries in "Top priorités" (bundles count as one entry)
- 2-3 lines per use case in the justification
- Cite benchmark sources used in the analysis (transparence)

**LLM judgment scope (override allowed if justified)** :
- **Bundling** : group 2-3 use cases that share a foundation (Data Lake, Business Central integration, common sponsor) into a single Top entry. Name the bundle explicitly.
- **Promotion** : raise a use case above its mechanical rank if there's a strategic reason (explicit client OKR, dependency that unblocks others, client window). Cite the reason. Show original mechanical rank.
- **Rétrogradation** : drop a mechanical Top 5 if you spot a risk the score didn't capture (sponsor lukewarm under high completeness, external dependency, late-breaking blocker). Cite the reason.
- **Lectures transversales** : section that flags cross-row patterns (shared integrations, common sponsors, sequencing effects, portfolio risks).
- **Recommandations stratégiques** : qualitative recommendations on launch order, prerequisites, portfolio risks, client windows. Can diverge from pure ranking.

**Guardrails (anti-hallucination)** :
- Any ranking override must be **written and justified** with a specific quote or signal from the source material
- The **mechanical verdict stays visible** (e.g., "verdict mécanique : Pass, mais à re-scoper si Sabre ne livre pas dans Q3 2026")
- Bundles must be **named** ("Bundle Fondations Data Lake : 3 use cases combinés...")
- **Recommandations stratégiques** are in a separate section from Verdict ROI to preserve traceability

Save to **the project folder**: `<output_folder>/synthese_priorisation.md`.

### Step 9: Compile sources_benchmark.md (v2.2)

Compile all the web search benchmarks used for col 27 (Coût Unitaire Agent) into a separate file for auditability. Use `references/sources_benchmark_template.md` as the structure. One section per Type Agent encountered in the CSV, with source URLs and consultation date.

Save to **the project folder**: `<output_folder>/sources_benchmark.md`.

### Step 10: Present the folder and its files

Use the `present_files` tool to share the three outputs (in order):
1. `priorisation_use_cases.csv` (primary working artifact)
2. `synthese_priorisation.md` (executive summary)
3. `sources_benchmark.md` (audit trail)

Mention the folder name in the message so the user can locate everything together.

## Critical principles

**Never fabricate data**. Empty cells lower confidence, which is correct behavior. A CSV with 60% confidence is honest and useful. A CSV with 95% confidence built on invented numbers is worse than useless because it misleads decision-making.

**Always write qualitative cells in the format `<n> - <label>`**. Just `3` is ambiguous; `3 - récurrent standardisé` carries the meaning for any human reading the CSV later.

**Use "N/A" sparingly**. N/A means "the question genuinely doesn't apply to this use case", not "I don't know". N/A counts as a filled cell for confidence. Empty counts as not filled.

**Don't override verdicts**. Verdicts are calculated from thresholds. If the math says Pass, don't write Go because you have a hunch. Hunches go in the Notes column.

**Always cite the benchmark source**. The Run Annuel is now anchored in real data instead of a heuristic — that traceability is what makes it credible. Without source + date, a reviewer can't validate.

**The architect output is just T-shirt size**. If no architect input is available, leave Taille Build empty (it is a critical column, so confidence will drop). Note in the Notes column that architect input is needed.

**Output language is always French Quebec**. Even if the input is in English, translate. Never use em dashes in any output text (replace with comma, parenthesis, or colon).

## Edge cases

- **No quantitative data at all**: Build the CSV with qualitative columns filled. Confiance Données will be low (30-50%). The synthèse explicitly recommends a chiffrage workshop. This is the correct outcome, not a failure.

- **Single process**: Generate a 1-row CSV. The synthèse focuses on that single use case with deeper analysis.

- **500+ processes table**: Process all rows. The CSV handles it. The synthèse still highlights top 5. If processing is too long, batch by department or by Suitability Score and warn the user.

- **Conflicting information between sources**: Document the conflict in Notes column. Use the most credible source for the cell value (transcript validated > note d'atelier > photo de post-it).

- **Benchmark search returns nothing usable**: Use the fallback table in `calculation_rules.md`, mark `BENCHMARK_FALLBACK` in Notes, and recommend in the synthèse that the architect provide a real estimate before committing.

- **Architect input arrives later**: User can update the CSV themselves and the dependent columns recalculate. The skill handles initial extraction.

## Quality checks before presenting

Before calling present_files, verify:
- [ ] Output folder created and announced to user (v2.2)
- [ ] All three files (`priorisation_use_cases.csv`, `synthese_priorisation.md`, `sources_benchmark.md`) saved inside the same project folder
- [ ] CSV has FORMULES row and SOURCE row preserved (38 columns)
- [ ] Each data row has at least Use Case and Pain Point filled
- [ ] Qualitative cells (cols 6-13, 32, 35) are in the `<n> - <label>` format, not just a number
- [ ] Verdict ROI matches Payback threshold (see calculation_rules.md)
- [ ] **Verdict Confiance reflects the hybrid `Conf_Données × Validation_Multiplier`**, not raw completeness
- [ ] Every row has Validation LLM (col 35) filled with justification in Notes
- [ ] Every row has Type Agent (col 26) and Coût Unitaire Agent (col 27) filled, with benchmark source cited in Notes
- [ ] **Sanity check**: if all rows show Confiance Données = 100%, double-check Validation LLM — too many `3 - source primaire validée` likely means the LLM is overstating validation
- [ ] **Synthèse readability (v2.2)** : no `[XX]` brackets, no `|` pipes, no `---` separators between use cases, no `≥`/`<`/`×` symbols in prose
- [ ] **Synthèse judgment (v2.2)** : if any ranking was overridden, it's justified with a cited reason AND the mechanical rank stays visible
- [ ] **Synthèse bundles (v2.2)** : if bundled, the bundle name is explicit and member use cases are listed
- [ ] **Lectures transversales** and **Recommandations stratégiques** sections present in the synthèse
- [ ] No em dashes anywhere in either file
- [ ] Notes column flags major hypotheses, missing data points, benchmark sources, AND the Validation LLM justification
- [ ] `sources_benchmark.md` lists at least one source per unique Type Agent in the CSV
