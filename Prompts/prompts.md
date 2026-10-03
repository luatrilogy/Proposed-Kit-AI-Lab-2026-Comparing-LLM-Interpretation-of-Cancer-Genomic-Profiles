# Prompt Registry

This file contains the exact prompts used in the experiment. Every prompt has
a stable ID that should be recorded in the run log. If any wording changes,
create a new prompt ID instead of editing the version used for prior runs.

Bracketed fields such as `[STUDY_ID]` are filled programmatically from the
frozen dataset. Do not manually rewrite prompts between cases.

---

## P1 — System Prompt

**Used for:** Every model call in both experimental conditions  
**Added:** October 3, 2026

```text
You are participating in a research benchmark about evidence grounding in
cancer-genomics interpretation. Analyze only the information supplied in this
conversation. Do not browse the web or use external tools. This is not a real
patient and your response must not give medical advice, prescribe treatment,
or provide dosing. Distinguish supported conclusions from hypotheses and
abstain when the supplied information is insufficient. Return valid JSON that
matches the requested schema and do not include text outside the JSON.
```

---

## P2 — Case Interpretation Prompt

**Used for:** Both the case-only and evidence-supplied conditions  
**Added:** October 3, 2026

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

### Condition values for `[EVIDENCE_PACKET]`

For the **case-only condition**, insert exactly:

```text
No external evidence has been supplied.
```

For the **evidence-supplied condition**, insert the matched packet generated
from the frozen CIViC release. Each entry should include only these fields:

```text
CIViC evidence ID
Molecular profile
Disease
Evidence type
Evidence direction
Evidence level
Clinical significance
Therapy, when present
Evidence statement
```

---

## P3 — JSON Format Repair Prompt

**Used for:** One repair attempt when a response is not valid JSON  
**Added:** October 3, 2026

```text
Reformat your previous response as valid JSON matching the requested schema.
Do not add, remove, strengthen, or reinterpret any substantive claim. Do not
include Markdown fences or text outside the JSON object.
```

The original response must be retained. Record whether P3 was used in the run
log, and do not allow more than one repair attempt per model call.

---

## Minimum Run-Log Fields

Each call should record:

- Run ID
- Study ID
- Condition (`case_only` or `evidence_supplied`)
- Prompt IDs (`P1` and `P2`, plus `P3` if needed)
- Provider and exact model identifier
- Model snapshot or release date
- Temperature and maximum output tokens
- Date and time of the call
- Repeat number
- Raw-output filename
- Whether JSON repair was needed

