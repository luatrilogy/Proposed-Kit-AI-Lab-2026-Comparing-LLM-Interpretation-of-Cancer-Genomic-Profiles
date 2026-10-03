# Data Declaration

This file declares every dataset used by the project **Hand an AI a Tumor Profile. Does It Know What the Evidence Actually Supports?** It must be updated with final filenames, checksums, and case counts before model data collection begins.

## Data Freeze

| Component | Fixed version |
| --- | --- |
| GDC source release | Data Release 46.0, August 10, 2026 |
| GDC project | TCGA-LUAD |
| GDC query date | October 3, 2026 |
| CIViC release | October 1, 2026 monthly release |
| Case-selection seed | `2026` |
| Planned Standard sample | 25 unique primary-tumor cases |
| Planned Starter subset | 15 stratified cases from the Standard sample |

Once the selected case manifest and checksums are entered below, no case, mutation, CIViC entry, or exclusion rule may be changed without creating a new dataset version and documenting the reason.

## Ground-Truth Form

The project uses a **human-coded gold standard**. CIViC evidence statements are manually curated from published cancer literature and reviewed by CIViC editors. Only accepted evidence items and accepted assertions from the dated release are eligible for scoring.

CIViC is not exhaustive. A claim without a match is coded `not found in the selected reference`, not `false`.

## Source 1: TCGA-LUAD Through the NCI Genomic Data Commons

### Creator and Purpose

The Cancer Genome Atlas was created by the National Cancer Institute and National Human Genome Research Institute to molecularly characterize human cancers. The Genomic Data Commons distributes harmonized TCGA data and associated metadata.

### Source Links

- TCGA program: <https://www.cancer.gov/ccg/research/genome-sequencing/tcga>
- GDC portal: <https://portal.gdc.cancer.gov/>
- TCGA-LUAD project: <https://portal.gdc.cancer.gov/projects/TCGA-LUAD>
- GDC data-access policy: <https://docs.gdc.cancer.gov/Encyclopedia/pages/Data_Access_Policy/>

### Access and Permitted Use

The project uses only GDC files labeled **open**. Open GDC data may be used for research subject to the NIH Genomic Data Sharing Policy. Users must not attempt to identify participants and must acknowledge the dataset and repository in resulting presentations or publications. No controlled-access files or authentication tokens belong in this repository.

### Repository Filters

| Field | Value |
| --- | --- |
| Project | `TCGA-LUAD` |
| Access | `Open` |
| Data Category | `Simple Nucleotide Variation` |
| Data Type | `Masked Somatic Mutation` |
| Data Format | `MAF` |
| Experimental Strategy | `WXS` |
| Workflow Type | `Aliquot Ensemble Somatic Variant Merging and Masking` |

## Source 2: CIViC

### Creator and Purpose

CIViC is an open, community-curated knowledge base for the clinical interpretation of cancer variants. Evidence entries connect molecular profiles to published diagnostic, prognostic, predictive, predisposing, oncogenic, or functional evidence.

### Source Links

- CIViC: <https://civicdb.org/>
- Dated releases: <https://civicdb.org/releases/main>
- Documentation: <https://docs.civicdb.org/>
- Data licensing: <https://civicdb.org/releases/licensing>

### License

CIViC knowledge-base content is released under the Creative Commons Public Domain Dedication, CC0 1.0 Universal. The project will cite CIViC and preserve the release date even though attribution is not legally required by CC0.

### Fixed Reference Files

Use only these accepted-only TSV files from the October 1, 2026 release:

```text
data/evidence/civic-2026-10-01/01-Oct-2026-VariantSummaries.tsv
data/evidence/civic-2026-10-01/01-Oct-2026-MolecularProfileSummaries.tsv
data/evidence/civic-2026-10-01/01-Oct-2026-AcceptedClinicalEvidenceSummaries.tsv
data/evidence/civic-2026-10-01/01-Oct-2026-AcceptedAssertionSummaries.tsv
```

Nightly files and accepted-and-submitted files are excluded because their contents either change during the experiment or include entries that have not received accepted status.

## Case Construction

### Unit

One item is one synthetic molecular case card derived from one unique TCGA-LUAD primary-tumor case.

### Eligibility Rules

A case is eligible when:

1. It belongs to TCGA-LUAD.
2. It has an open-access masked somatic mutation MAF produced by the specified workflow.
3. Its associated sample is a primary tumor.
4. The MAF file is readable and can be mapped to one unique case.

When more than one eligible aliquot exists for a case, the deterministic selection script will retain one according to a prespecified sorted rule and record the discarded alternatives.

### Diversity and Stratified Sampling

The Standard dataset contains 25 cases:

- Eight with at least one exact alteration matched to accepted CIViC level A or B evidence
- Eight whose strongest exact match is accepted level C, D, or E evidence
- Nine with no exact alteration match in the accepted CIViC release

Cases will be randomly selected within strata using seed `2026`. If a stratum is too small, all eligible cases in that stratum will be retained and the remaining positions will be drawn from the adjacent stratum. The Starter subset will contain 15 cases while preserving representation from all available strata.

This procedure prevents all case cards from containing only famous, well-supported driver mutations.

### Variant Normalization

MAF variants will be normalized using, when available:

- HGNC gene symbol
- Protein change/HGVSp
- Coding change/HGVSc
- Reference genome build
- Chromosome and genomic coordinates
- Reference and alternate alleles

Matching will proceed from exact HGVS/coordinate matches to documented aliases. Every non-exact match will be logged and manually reviewed before it is used in an evidence packet.

### Model-Facing Data

The model will receive only:

- Synthetic study identifier
- Cancer type
- Gene symbol
- Protein change
- Variant classification
- Condition-specific evidence packet

The model will not receive TCGA barcodes, names, dates, locations, clinical notes, or other unnecessary metadata.

## Exclusions

- Controlled-access files
- Raw FASTQ or BAM sequencing data
- Germline mutation data
- Non-primary samples for the pilot
- Cases that cannot be mapped unambiguously to one eligible MAF
- CIViC submitted, pending, rejected, deprecated, or nightly-only entries
- Variants whose normalization remains ambiguous after review

Every exclusion and its reason will be written to `data/processed/exclusions.tsv`.

## File Inventory and Checksums

Complete this table after downloading and processing the frozen data. SHA-256 is preferred.

| Relative path | Rows/items | SHA-256 | Status |
| --- | ---: | --- | --- |
| `data/manifests/gdc_manifest.TCGA-LUAD.masked-maf.txt` | `<COUNT>` | `<SHA256>` | Pending |
| `data/metadata/gdc_file_metadata.TCGA-LUAD.masked-maf.tsv` | `<COUNT>` | `<SHA256>` | Pending |
| `data/metadata/gdc_sample_sheet.TCGA-LUAD.masked-maf.tsv` | `<COUNT>` | `<SHA256>` | Pending |
| `data/processed/selected_cases.tsv` | `25` | `<SHA256>` | Pending |
| `data/evidence/civic-2026-10-01/01-Oct-2026-VariantSummaries.tsv` | `<COUNT>` | `<SHA256>` | Pending |
| `data/evidence/civic-2026-10-01/01-Oct-2026-MolecularProfileSummaries.tsv` | `<COUNT>` | `<SHA256>` | Pending |
| `data/evidence/civic-2026-10-01/01-Oct-2026-AcceptedClinicalEvidenceSummaries.tsv` | `<COUNT>` | `<SHA256>` | Pending |
| `data/evidence/civic-2026-10-01/01-Oct-2026-AcceptedAssertionSummaries.tsv` | `<COUNT>` | `<SHA256>` | Pending |

## Versioning Rules

- The first completed dataset is version `1.0`.
- Fixing a formatting or metadata error without changing cases increments the patch version.
- Changing cases, evidence entries, inclusion rules, or normalization logic creates a new major dataset version.
- Every change must be described in a dated changelog entry below.

## Changelog

### 2026-10-03 — Draft declaration

- Fixed the intended GDC and CIViC source releases.
- Defined the 25-case sample, stratification, eligibility, exclusion, and privacy rules.
- Checksum fields remain pending until the filtered files and selected cases are finalized.
