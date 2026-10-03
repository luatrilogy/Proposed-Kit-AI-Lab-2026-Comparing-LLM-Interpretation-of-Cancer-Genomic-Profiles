# Can an AI a Tumor Profile. Does It Know What the Evidence Actually Supports?

## Research Question

Can large language models convert de-identified cancer genomic profiles into accurate, evidence-grounded molecular interpretations, and does providing curated cancer-variant evidence reduce unsupported clinical claims?

## Hypothesis

Large language models will correctly identify well-known cancer-driving alterations but will overinterpret rare or uncertain variants and sometimes assign treatment relevance unsupported by the supplied evidence. Providing retrieved evidence from a curated cancer knowledge base will improve accuracy and calibration, although omissions and disagreements between models will remain.

## Learning Objective

Learn how to distinguish a fluent, medically plausible response from a faithful and evidence-grounded molecular interpretation. Develop methods for measuring hallucination, uncertainty, reproducibility, and evidence use in AI-generated biomedical reports.

## Domain

Computational oncology / Bioinformatics / AI evaluation / Precision medicine

## Overview

Students will construct standardized, de-identified **molecular case cards** from open-access cancer data in the [NCI Genomic Data Commons](https://portal.gdc.cancer.gov/). Each card will contain a cancer type and a selected set of somatic mutations, copy-number alterations, or gene-expression abnormalities.

The same case cards will be submitted to multiple language models using a fixed prompt and structured output format. Models will be asked to:

1. Identify the most biologically important alterations.
2. Explain the pathways that may be affected.
3. Classify each finding as diagnostic, prognostic, potentially treatment-relevant, or currently uncertain.
4. State what evidence would be required before making a stronger conclusion.
5. Abstain when the supplied information is insufficient.

Model outputs will be compared with curated evidence from [CIViC](https://civicdb.org/) and with a deterministic variant-matching baseline. The project evaluates research interpretation, **not clinical diagnosis or treatment selection**.

## Rationale

Molecular cancer reports can contain many alterations, only some of which have established biological or clinical significance. Language models may help researchers and healthcare professionals organize this information, but fluent responses can conceal unsupported conclusions.

A controlled benchmark can reveal:

- Which models reliably extract molecular information
- Which types of variants models overinterpret
- Whether curated evidence improves model performance
- Where expert review remains essential

## Data

Use open-access, processed data from the NCI Genomic Data Commons rather than raw sequencing files or identifiable patient information.

A feasible pilot could use:

- **TCGA-LUAD:** lung adenocarcinoma
- **TCGA-SKCM:** skin cutaneous melanoma
- **TCGA-BRCA:** breast invasive carcinoma
- Open somatic mutation MAF files
- Selected copy-number or gene-expression results
- Limited, nonidentifying clinical context
- CIViC variant evidence accessed through its public API

Every case should receive a synthetic study identifier. No real patient names, dates, notes, or other identifying information should be sent to an external model.

### Example Molecular Case Card

```json
{
  "study_id": "CASE_014",
  "cancer_type": "lung adenocarcinoma",
  "alterations": [
    {
      "gene": "EGFR",
      "protein_change": "p.L858R",
      "type": "missense"
    },
    {
      "gene": "TP53",
      "protein_change": "p.R273H",
      "type": "missense"
    }
  ]
}
```

## Study Designs

### Starter: Can One Model Faithfully Read a Tumor Profile?

#### Variables

- **Independent variable:** Molecular case profile
- **Dependent variables:**
  - Alteration-extraction accuracy
  - Driver-prioritization precision
  - Omission rate
  - Number of alterations invented by the model
  - Appropriateness of uncertainty statements

#### Method

1. Select approximately 25 cases from one TCGA cancer cohort.
2. Create a standardized case card containing the cancer type and selected somatic alterations.
3. Ask one model to return a structured molecular interpretation.
4. Compare every claimed alteration with the original case card.
5. Compare the model's prioritization with CIViC evidence and a rule-based matching baseline.
6. Manually classify errors using a predefined scoring rubric.

#### Outcomes

Measure whether the model can accurately summarize supplied genomic information before evaluating treatment-related reasoning.

**Property tested:** Faithfulness

---

### Standard: Does Retrieved Evidence Reduce Unsupported Claims?

#### Variables

- **Independent variable:** Evidence condition
  - Case profile only
  - Case profile plus retrieved CIViC evidence
- **Dependent variables:**
  - Supported-claim precision
  - Evidence recall
  - Unsupported treatment-claim rate
  - Correct evidence-level classification
  - Appropriate abstention rate

#### Method

1. Select approximately 40 cases containing both common driver alterations and rare or uncertain variants.
2. Run every case under both evidence conditions.
3. Randomize the order of the conditions.
4. Require the model to connect every major conclusion to a supplied evidence item or explicitly label it unsupported.
5. Score responses without revealing the model or condition to the evaluator.
6. Compare performance using paired statistical tests and bootstrap confidence intervals.

#### Outcomes

Determine whether retrieval improves interpretation or merely gives the model additional language with which to produce plausible answers.

**Property tested:** Evidence grounding

---

### Ambitious: Do Different Models Interpret the Same Tumors Differently?

#### Variables

- **Independent variables:**
  - Model: GPT, Claude, Gemini, and optionally an open-weight model
  - Evidence condition: with or without CIViC retrieval
  - Cancer cohort
  - Repeated generation
- **Dependent variables:**
  - Faithfulness
  - Supported-claim precision and recall
  - Calibration
  - Abstention
  - Cross-model agreement
  - Within-model reproducibility

#### Method

1. Construct 60 case cards balanced across three cancer types.
2. Run each case through each model under both evidence conditions.
3. Repeat each run three times with fixed settings.
4. Use the same prompt, output schema, evidence packet, and scoring rubric for every model.
5. Compare models using confidence intervals and a mixed-effects model that accounts for repeated evaluations of the same cases.
6. Have a genomics or oncology mentor independently review a sample of ambiguous outputs.

#### Outcomes

Determine whether apparent differences between models are systematic, statistically credible, and clinically meaningful rather than products of random generation.

**Property tested:** Comparability and robustness

## Evaluation Framework

| Property | Scientific question | Measurement |
| --- | --- | --- |
| Faithfulness | Did the model correctly read the supplied profile? | Invented, omitted, or mischaracterized alterations |
| Evidence grounding | Are its claims supported by cancer literature? | Precision and recall against CIViC |
| Calibration | Does its certainty reflect evidence strength? | Confidence compared with CIViC evidence level |
| Reproducibility | Does it give the same interpretation repeatedly? | Agreement across repeated runs and models |

## Expected Benefit

The project could identify tasks for which language models may safely assist researchers or clinicians—such as organizing molecular findings and locating relevant evidence—as well as tasks where they remain unreliable. Its central benefit would be a reproducible benchmark for detecting unsupported or overconfident cancer-genomics interpretations before similar systems are considered for clinical use.

## Ethical and Safety Boundaries

- This project evaluates research interpretation, not medical diagnosis.
- Model outputs must not be used to recommend treatment to a real patient.
- Only public, de-identified, open-access data should be used.
- Findings should be described as molecular or therapeutic hypotheses, not prescriptions.
- Any claims about clinical usefulness should require review by qualified oncology or genomics professionals.
- External model providers should never receive protected health information.

## Known Issues

- TCGA was designed primarily for molecular characterization, not prospective treatment recommendation.
- Treatment and response information is incomplete and inconsistent across cohorts.
- Raw sequencing files and some detailed genomic files require controlled access.
- Variant names must be normalized before matching TCGA records to CIViC.
- CIViC is curated but is not a complete representation of all oncology evidence.
- LLM training data are unknown, so models may have memorized well-known variants.
- Self-reported model confidence is not equivalent to clinical probability.
- No result should be described as medical advice, a diagnosis, or a patient-specific treatment recommendation.

## Suggested Repository Structure

```text
.
├── data/
│   ├── raw/                 # Public GDC files or download manifests
│   ├── processed/           # Normalized molecular case cards
│   └── evidence/            # Retrieved CIViC evidence
├── prompts/
│   ├── system_prompt.md
│   ├── case_only_prompt.md
│   └── evidence_prompt.md
├── schemas/
│   ├── case_card.schema.json
│   └── model_output.schema.json
├── src/
│   ├── download_gdc.py
│   ├── build_case_cards.py
│   ├── retrieve_civic.py
│   ├── run_models.py
│   └── score_outputs.py
├── results/
│   ├── raw_outputs/
│   ├── scored_outputs/
│   └── figures/
├── notebooks/
│   └── exploratory_analysis.ipynb
├── tests/
├── .gitignore
├── LICENSE
└── README.md
```

## Possible Wilms Tumor Extension

Because Wilms tumor is not a TCGA cancer cohort, a later pediatric extension should use the **TARGET-WT** project in the Genomic Data Commons. This could connect the evaluation framework to prior work on WT1-associated targets while remaining separate from the initial adult-cancer pilot.

## Key Resources

- [The Cancer Genome Atlas Program](https://www.cancer.gov/ccg/research/genome-sequencing/tcga)
- [NCI Genomic Data Commons](https://portal.gdc.cancer.gov/)
- [GDC API Documentation](https://docs.gdc.cancer.gov/API/Users_Guide/Getting_Started/)
- [CIViC Cancer Variant Knowledgebase](https://civicdb.org/)
- [CIViC Documentation](https://docs.civicdb.org/)

## Disclaimer

This repository is intended solely for educational and research purposes. It does not provide medical advice, diagnosis, or treatment recommendations. Any potential clinical application would require independent validation, appropriate regulatory review, and qualified professional oversight.

## Data Access and Acquisition

### Required Pilot Data

The initial study will use open-access somatic mutation data from the **TCGA Lung Adenocarcinoma project (TCGA-LUAD)** in the NCI Genomic Data Commons.

Data will be obtained from the [GDC Data Repository](https://portal.gdc.cancer.gov/repository) using the following filters:

| Repository field | Required value |
| --- | --- |
| Project | `TCGA-LUAD` |
| Access | `Open` |
| Data Category | `Simple Nucleotide Variation` |
| Data Type | `Masked Somatic Mutation` |
| Data Format | `MAF` |
| Experimental Strategy | `WXS` |
| Workflow Type | `Aliquot Ensemble Somatic Variant Merging and Masking` |

The filtered file metadata will first be exported in TSV format. A GDC manifest will then be generated to provide a reproducible record of the files included in the download.

The project will use processed, open-access MAF files rather than raw sequencing files. No controlled-access BAM, FASTQ, protected MAF, germline, or identifiable patient data will be used.

### Case Selection

The initial dataset will contain 25 unique TCGA-LUAD cases. Cases will be selected according to the following prespecified procedure:

1. Retain only open-access masked somatic mutation files.
2. Restrict the dataset to primary-tumor samples.
3. Retain one sample per unique GDC case.
4. Exclude cases without a readable mutation file.
5. Randomly select 25 unique cases using seed `2026`.
6. Save the selected case identifiers and GDC file UUIDs in a versioned manifest.

The selection script, random seed, query filters, query date, and GDC data-release number will be preserved in the repository.

### Clinical Metadata

Limited, nonidentifying metadata will be downloaded using the GDC cohort’s **Biospecimen/Clinical** export. The pilot will retain only fields required to describe and validate the molecular profile, such as:

- Synthetic study identifier
- GDC project
- Primary diagnosis
- Tumor sample type
- Available tumor stage, if analytically necessary

Names, dates, treatment notes, geographic data, and other potentially identifying or unnecessary information will not be collected. TCGA case identifiers will be replaced with synthetic study identifiers before records are submitted to an external language model.

### Molecular Case Cards

Each selected case will be converted into a standardized JSON record:

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

Only fields needed for the experiment will be included in the model-facing case card.

### Reference Evidence

Cancer-variant evidence will be obtained from a fixed release of the [CIViC knowledge base](https://civicdb.org/releases). The release date and version will be recorded before analysis.

The study will retain accepted CIViC evidence concerning:

- Variant identity
- Cancer type
- Evidence type
- Evidence level
- Evidence direction
- Clinical significance
- Associated therapy, if present
- Supporting publication

Using a frozen CIViC release will prevent the reference standard from changing during the experiment.

### Later Cohort Expansion

After validating the pipeline on TCGA-LUAD, the study may be expanded to:

- [TCGA-BRCA](https://portal.gdc.cancer.gov/projects/TCGA-BRCA): breast invasive carcinoma
- [TCGA-SKCM](https://portal.gdc.cancer.gov/projects/TCGA-SKCM): skin cutaneous melanoma

All cohorts will be processed using the same inclusion rules, variant normalization procedure, prompt, and scoring framework.

### Optional Multimodal Extension

Gene expression and copy-number data are not required for the initial mutation-focused study. A later extension may download:

| Data modality | GDC data category | Data type | Workflow |
| --- | --- | --- | --- |
| RNA expression | `Transcriptome Profiling` | `Gene Expression Quantification` | `STAR - Counts` |
| Copy number | `Copy Number Variation` | `Gene Level Copy Number` | `ASCAT3` |

These data will be introduced only after the somatic-mutation pipeline and evaluation rubric have been validated.

### Repository Policy

Raw GDC files will not be committed directly to GitHub. The repository will instead contain:

- GDC query filters
- Download manifest
- File metadata
- Data-release number
- Case-selection script
- Random seed
- Checksums where available
- Instructions for reproducing the download

Large downloaded files, API credentials, and controlled-access tokens will be excluded through `.gitignore`.
