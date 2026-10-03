# Hand an AI a Tumor Profile. Does It Know What the Evidence Actually Supports?

## Research Question

Does providing a large language model with curated CIViC evidence reduce the proportion of unsupported molecular and treatment-relevance claims it makes when interpreting de-identified TCGA lung adenocarcinoma profiles?

## Hypothesis

The model will usually recognize widely documented driver alterations but will overinterpret rare or uncertain variants because canonical cancer associations are more likely to be represented consistently in its training data. Providing a matched packet of accepted CIViC evidence will reduce claims unsupported by the fixed reference, increase appropriate abstention, and improve evidence-level classification, although it will not eliminate omissions or contradictions.

## Learning Objective

Learn how to distinguish a fluent, medically plausible answer from a faithful, evidence-grounded molecular interpretation, and determine whether retrieval of curated evidence measurably changes model behavior.

## Domain

Computational oncology / bioinformatics / AI evaluation / precision medicine

## Property Tested

**Evidence grounding.** In this project, evidence grounding means whether a model's substantive molecular claims are supported by the fixed CIViC reference evidence. The primary independent variable is whether the model receives a matched CIViC evidence packet in addition to the same TCGA case profile.

Faithfulness, abstention, and reproducibility will be reported as secondary measurements, not as separate primary properties.

## Removal Test

If the language model is removed, the TCGA profiles can still be normalized and matched to CIViC, but the experiment's central question—whether access to curated evidence changes an LLM's interpretations—disappears. The model's behavior is therefore the research subject rather than a replaceable tool for an otherwise complete cancer-genomics project.

## Overview

This project will construct standardized, de-identified molecular case cards from open-access somatic-mutation data in the [NCI Genomic Data Commons](https://portal.gdc.cancer.gov/). The Starter and Standard studies will use lung adenocarcinoma cases from TCGA-LUAD.

Each case will be presented to the same model under two conditions:

1. **Case only:** the model receives the molecular case card without external evidence.
2. **Evidence supplied:** the model receives the identical case card plus accepted CIViC evidence retrieved for its alterations.

The prompt, output schema, model settings, case information, and scoring procedure will remain constant. Only the evidence packet will differ. The project evaluates model interpretation for research purposes; it does not test clinical diagnosis or prescribe treatment.

## Rationale

Cancer profiles may contain many alterations, only some of which have established biological or clinical relevance. Language models can organize this information fluently, but fluency may conceal unsupported conclusions. A paired experiment can test whether curated evidence makes those answers measurably better grounded or merely gives the model more material with which to sound convincing.

The resulting benchmark may identify narrow tasks for which LLMs could assist researchers—such as organizing alterations and locating relevant evidence—while documenting errors that make expert review indispensable.

## Data and Ground Truth

### Ground-Truth Form

This project uses a human-coded gold standard. CIViC evidence items are manually curated from published cancer literature and reviewed by CIViC editors. Only accepted evidence items and accepted assertions from the dated October 1, 2026 release will be included.

A claim missing from CIViC will be labeled not found in the selected reference, not automatically false or hallucinated. CIViC is curated but not exhaustive.

### Input Dataset

The input profiles will come from the TCGA-LUAD project in NCI Genomic Data Commons Data Release 46.0. The study will use open-access, processed masked somatic mutation MAF files rather than raw sequencing files, protected MAF files, or germline data.

The fixed 25-case Standard dataset will be generated from unique primary-tumor cases using seed `2026`. To prevent the case cards from all being alike, the sample will contain three prespecified evidence strata:

- Eight cases with at least one exact alteration matched to accepted CIViC level A or B evidence
- Eight cases whose strongest exact match is accepted level C, D, or E evidence
- Nine cases with no exact alteration match in the accepted CIViC release

If a stratum contains fewer eligible cases than required, all eligible cases will be retained and the remaining positions will be randomly sampled from the adjacent stratum. The Starter study will use a stratified 15-case subset of the same fixed dataset.

Dataset creators, access conditions, release dates, inclusion rules, expected files, and provenance are declared in [`data/DATA.md`](data/DATA.md).

### Example Molecular Case Card

```json
{
  "study_id": "CASE_014",
  "cancer_type": "lung adenocarcinoma",
  "alterations": [
    {
      "gene": "EGFR",
      "protein_change": "p.L858R",
      "variant_classification": "Missense_Mutation"
    },
    {
      "gene": "TP53",
      "protein_change": "p.R273H",
      "variant_classification": "Missense_Mutation"
    }
  ]
}
```

Only fields necessary for the experiment will enter a model prompt. Public TCGA case identifiers will be replaced with synthetic study identifiers.

## Variables

### Primary Independent Variable

**Evidence condition:**

- Case profile only
- Identical case profile plus matched accepted CIViC evidence

No other case attribute changes between these two conditions.

### Primary Dependent Variable

**Unsupported-by-reference claim rate:**

```text
claims classified as contradicted or not found
------------------------------------------------
all substantive molecular claims in the response
```

A substantive claim associates an alteration with biological function, diagnosis, prognosis, oncogenicity, therapeutic response, resistance, or treatment relevance.

### Secondary Dependent Variables

- Supported-claim precision
- Contradicted-claim rate
- Input-invention count
- Appropriate-abstention rate
- Evidence-level classification accuracy
- Number of substantive claims per response
- Agreement across repeated runs

## Controls and Run Registration

Before the first call, the following fields must be completed in the run log. No pilot data will be collected while any field is blank.

| Setting | Registered value |
| --- | --- |
| Provider | `<PROVIDER>` |
| Exact model/API identifier | `<MODEL_ID>` |
| Model release or snapshot date | `<MODEL_SNAPSHOT_DATE>` |
| Temperature | `0` |
| Maximum output | `1,200 tokens` |
| Tool use/web browsing | Disabled |
| Conversation structure | Every call begins in a new conversation |
| Case-order seed | `2026` |
| Condition order | Counterbalanced within case |
| Repeats | Defined separately for each study size |
| Prompt | Exact template below; no unlogged changes |

If an API does not expose one of these settings, the run log will say `not exposed` rather than guessing. A model update during collection will stop the run; affected calls will be restarted under one fixed version or reported as a separate block.

## Fixed Prompt

The following prompt will be used word for word, with bracketed fields filled programmatically from the frozen dataset.

### System Message

```text
You are participating in a research benchmark about evidence grounding in
cancer-genomics interpretation. Analyze only the information supplied in this
conversation. Do not browse the web or use external tools. This is not a real
patient and your response must not give medical advice, prescribe treatment,
or provide dosing. Distinguish supported conclusions from hypotheses and
abstain when the supplied information is insufficient. Return valid JSON that
matches the requested schema and do not include text outside the JSON.
```

### User Message

```text
CANCER TYPE:
[CANCER_TYPE]

SOMATIC ALTERATIONS:
[ALTERATION_LIST]

REFERENCE EVIDENCE:
[EVIDENCE_PACKET]

For each alteration, report:
1. whether it is present in the case card;
2. its possible diagnostic, prognostic, oncogenic, functional, or predictive
   relevance;
3. whether any treatment relevance is supported, unsupported, conflicting, or
   cannot be determined;
4. the evidence level and evidence identifier when supplied;
5. a concise uncertainty statement.

If no external evidence was supplied, do not invent evidence identifiers.
If the evidence does not support a conclusion, state that explicitly.

Return this JSON structure:
{
  "study_id": "[STUDY_ID]",
  "interpretations": [
    {
      "gene": "string",
      "protein_change": "string",
      "present_in_case": true,
      "relevance_type": "diagnostic|prognostic|oncogenic|functional|predictive|uncertain",
      "claim": "string",
      "treatment_relevance": "supported|unsupported|conflicting|cannot_determine",
      "evidence_level": "A|B|C|D|E|not_supplied|not_found",
      "evidence_id": "string or null",
      "uncertainty": "string"
    }
  ],
  "overall_summary": "string",
  "additional_testing_needed": ["string"],
  "limitations": ["string"]
}
```

For the case-only condition, `[EVIDENCE_PACKET]` will contain exactly:

```text
No external evidence has been supplied.
```

For the evidence-supplied condition, it will contain a structured packet generated from the frozen CIViC release. Each entry will include the CIViC evidence ID, molecular profile, disease, evidence type, evidence direction, evidence level, significance, therapy when present, and evidence statement.

## Study Designs and Sample Sizes

### Starter: Does Curated Evidence Reduce Unsupported Claims?

```text
15 cases × 2 conditions × 2 repeats × 1 model = 60 calls
```

#### Method

1. Use the stratified 15-case subset of TCGA-LUAD.
2. Run every case under both evidence conditions.
3. Repeat every case-condition pair twice in a new conversation.
4. Counterbalance the condition order within each case.
5. Score the shuffled, de-labeled outputs using the fixed rubric below.
6. Compare the paired per-case unsupported-by-reference claim rates.

#### Outcome

Estimate whether supplying curated evidence changes grounding for one model on a feasible pilot dataset.

**Primary property:** Evidence grounding

---

### Standard: Full TCGA-LUAD Benchmark

```text
25 cases × 2 conditions × 3 repeats × 1 model = 150 calls
```

#### Method

1. Use all 25 fixed TCGA-LUAD cases.
2. Run the same paired conditions with three independent repeats.
3. Use the same model version, prompt, settings, evidence-retrieval procedure, and scoring rubric.
4. Estimate the per-case paired difference with a bootstrap 95% confidence interval.
5. Examine results by the three prespecified evidence strata.

#### Outcome

Test whether the Starter result persists across a larger and more varied sample from the same cancer cohort.

**Primary property:** Evidence grounding

---

### Ambitious: Cross-Model and Cross-Cohort Replication

```text
60 cases × 2 conditions × 3 repeats × 3 models = 1,080 calls
```

#### Method

1. Construct 20-case samples from TCGA-LUAD, TCGA-BRCA, and TCGA-SKCM using the same stratification procedure.
2. Run the paired conditions on one fixed model version from each of three providers.
3. Keep prompts, settings, evidence packets, repeats, and scoring identical wherever provider controls permit.
4. Report any provider setting that cannot be held constant.
5. Compare the evidence-condition effect across models and cohorts rather than treating raw scores as perfectly interchangeable.

#### Outcome

Determine whether the grounding effect replicates across selected models and cancer cohorts. This version remains an extension and will not begin until the Starter study is complete.

**Primary property:** Evidence grounding

## Scoring Criteria

### Unit of Analysis

The unit of analysis is a substantive claim, not an entire report. Before condition labels are revealed, each claim will be assigned to exactly one category.

| Category | Operational rule |
| --- | --- |
| Supported | The frozen CIViC release contains accepted evidence matching the molecular profile, relevant disease context, and direction of the claim. |
| Contradicted | Accepted CIViC evidence for the relevant context conflicts with the claim's direction or significance. |
| Not found | No matching accepted evidence is present in the frozen CIViC release. This does not prove the claim false. |
| Input invention | The response discusses a molecular alteration that was not present in the case card. |
| Appropriate abstention | The response explicitly states that the available information is insufficient and does not replace the missing evidence with a stronger conclusion. |

Evidence-level classification is correct only when the reported level matches the supplied CIViC entry. A treatment claim is counted as supported only when the disease, molecular profile, direction, and therapy agree with the reference entry.

### Blinding and Reliability

Outputs will be assigned random identifiers, shuffled, and stripped of model and condition labels before scoring. A second scorer, preferably a mentor with genomics or oncology experience, will independently score at least 20 randomly selected reports. Percent agreement and Cohen's kappa will be reported. Disagreements will be resolved only after the initial agreement statistics are calculated.

### Analysis

The primary statistic is the paired per-case difference in unsupported-by-reference claim rate between the case-only and evidence-supplied conditions. The report will include the mean or median difference, a bootstrap 95% confidence interval, all raw case-level values, and a sensitivity analysis that reports contradicted and not-found claims separately.

## Expected Benefit

The project may identify tasks for which language models can assist researchers, such as organizing molecular findings or tracing claims to supplied evidence, while documenting failure modes that require expert review. Its main contribution is a reproducible benchmark and scoring framework, not a clinical decision system.

## Ethical and Safety Boundaries

- This project evaluates research interpretation, not diagnosis or treatment selection.
- Model outputs will never be used to guide care for a real patient.
- Only public, de-identified, open-access data will be used.
- External model providers will not receive names, dates, public TCGA identifiers, clinical notes, geographic information, or protected health information.
- Findings will be described as benchmark results, not proof of clinical safety or effectiveness.
- Ambiguous outputs may be reviewed by a qualified mentor, but no new information will be collected from human research participants.

## Known Issues

- A model may refuse to interpret the case because it classifies the request as medical advice.
- Medical disclaimers or boilerplate may obscure the molecular claims being scored.
- A model may notice that one condition contains a curated evidence packet and explicitly comment on the manipulation.
- CIViC is curated but not exhaustive; `not found` does not mean scientifically false.
- TCGA was designed primarily for molecular characterization, not prospective treatment recommendation.
- Variant names and genome builds must be normalized before TCGA-to-CIViC matching.
- Famous cancer variants may have been represented frequently in model training data, making reasoning difficult to separate from memorization.
- The same nominal temperature can behave differently across providers, and some interfaces may not expose identical settings.
- Model refusals will remain in the dataset and be scored; they will not be deleted.
- The model may produce invalid JSON. One format-repair request using a fixed repair prompt will be allowed and logged; the original output will be retained.

## What the Result Will Support

If the evidence-supplied condition produces fewer unsupported-by-reference claims, the study will support the conclusion that providing curated CIViC evidence improved the tested model's grounding on the selected TCGA-LUAD cases. It will not establish that the model can diagnose cancer, select treatment for an individual patient, outperform oncology professionals, or generalize to cancers, variants, prompts, or model versions outside the benchmark.

## Human Participants and Independence

This project uses only pre-existing, public, de-identified data and does not recruit or collect new information from human participants. A second scorer evaluates model outputs rather than serving as a research subject.

This work stands on its own and is not being submitted as coursework, a junior paper, or a thesis. It is distinct from prior WT1 immunotherapy work: that project modeled candidate targets and tumor dynamics, whereas this project experimentally evaluates LLM evidence grounding on TCGA profiles.

## Data Access and Acquisition

### Required Pilot Data

The pilot will use open-access somatic mutation data from TCGA-LUAD. Data will be obtained through the NCI Genomic Data Commons using these filters:

| Repository field | Required value |
| --- | --- |
| Project | `TCGA-LUAD` |
| Access | `Open` |
| Data Category | `Simple Nucleotide Variation` |
| Data Type | `Masked Somatic Mutation` |
| Data Format | `MAF` |
| Experimental Strategy | `WXS` |
| Workflow Type | `Aliquot Ensemble Somatic Variant Merging and Masking` |

The repository will preserve the filtered manifest, file metadata, sample sheet, release number, query date, and selection script. Raw MAF files will remain in ignored local storage.

### Frozen CIViC Reference

The study will use these accepted-only TSV files from the October 1, 2026 monthly CIViC release:

- `01-Oct-2026-VariantSummaries.tsv`
- `01-Oct-2026-MolecularProfileSummaries.tsv`
- `01-Oct-2026-AcceptedClinicalEvidenceSummaries.tsv`
- `01-Oct-2026-AcceptedAssertionSummaries.tsv`

No nightly or accepted-and-submitted release will be mixed into the reference after the dataset is frozen.

## Repository Structure

```text
.
├── data/
│   ├── DATA.md
│   ├── manifests/
│   ├── metadata/
│   ├── evidence/
│   │   └── civic-2026-10-01/
│   └── raw/                  # Ignored; downloaded locally
├── prompts/
│   └── prompts.md
├── .gitignore
└── README.md
```

## Reproducibility Checklist

- [ ] Co-lead or mentor approval received
- [ ] Exact provider and model/API identifier registered
- [ ] Dataset files and checksums recorded in `data/DATA.md`
- [ ] Twenty-five cases fixed with seed `2026`
- [ ] Prompt files match the prompt printed above
- [ ] Tool use and browsing disabled
- [ ] Run log created with standard columns unchanged
- [ ] Starter's 60 calls completed before expanding scope
- [ ] Condition labels hidden before scoring
- [ ] Second scorer assigned before labels are revealed

## Key Resources

- [The Cancer Genome Atlas Program](https://www.cancer.gov/ccg/research/genome-sequencing/tcga)
- [NCI Genomic Data Commons](https://portal.gdc.cancer.gov/)
- [GDC API Documentation](https://docs.gdc.cancer.gov/API/Users_Guide/Getting_Started/)
- [CIViC Data Releases](https://civicdb.org/releases/main)
- [CIViC Documentation](https://docs.civicdb.org/)

## Disclaimer

This repository is intended solely for educational and research purposes. It does not provide medical advice, diagnosis, or treatment recommendations. Any potential clinical application would require independent validation, appropriate regulatory review, and qualified professional oversight.
