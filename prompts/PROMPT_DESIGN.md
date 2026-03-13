# Prompt Design Rationale

## Design Principles

### 1. Separation of Concerns

Each prompt handles exactly one task. The **classifier** categorizes documents; the **extractors** pull structured data; the **rewriter** transforms clinical language. This separation:

- Makes each prompt testable in isolation
- Allows independent versioning and improvement
- Reduces prompt complexity and hallucination risk

### 2. Hybrid AI + Rules for Safety-Critical Decisions

**Urgency scoring is NOT performed by the LLM.** Instead:

- The LLM classifier identifies the document category
- A deterministic rules engine (Code node) evaluates urgency based on:
  - Regex patterns: `VALOR CRÍTICO`, `CRITICAL`, `*** ... ***`
  - Numeric thresholds: values > 2x upper reference limit
  - Category defaults: administrative = low, radiology = medium

**Why:** In clinical settings, false negatives on critical values are a patient safety risk. Deterministic rules are auditable, predictable, and do not hallucinate. The LLM excels at understanding document structure — the rules engine excels at threshold evaluation.

### 3. Structured Output Enforcement

All extraction prompts produce JSON conforming to schemas defined in `schemas/`. The n8n workflow uses Structured Output Parsers that:

- Validate LLM output against the JSON schema
- Auto-retry with correction if the output is malformed
- Guarantee downstream nodes receive predictable data structures

### 4. Language Strategy

| Component | Language | Reason |
|-----------|----------|--------|
| System prompts | English | LLMs perform better with English instructions |
| Input documents | Spanish | Realistic for Spanish hospital setting |
| Output field names | English | Interoperability with FHIR-like schemas |
| Extracted text values | Original language | Preserves clinical meaning without translation risk |
| Patient-friendly output | Spanish | Target audience: Spanish-speaking patients |

### 5. Temperature Selection

| Prompt | Temperature | Reason |
|--------|-------------|--------|
| Classifier | 0 | Deterministic category assignment |
| Lab extractor | 0 | Exact value extraction, no creativity |
| Radiology extractor | 0 | Exact finding extraction |
| Notes extractor | 0 | Exact clinical data extraction |
| Patient-friendly | 0.3 | Natural language requires slight variability |

### 6. Safety Guardrails in Prompts

- **No invention rule:** "Do not invent or infer information not present in the document"
- **Dosage preservation:** "Medication dosages must be extracted EXACTLY as written"
- **No diagnosis beyond source:** Patient-friendly rewriter cannot add medical advice not in the original
- **Data quality flag:** Every extractor reports `data_quality` to signal incomplete extractions

## Versioning Strategy

Prompts are stored in versioned directories (`v1/`, `v2/`, ...) to support:

- A/B testing between prompt versions
- Rollback capability if a new version degrades quality
- Audit trail for regulatory compliance (traceability of AI behavior changes)

Each prompt file includes a changelog section documenting what changed and why.

## Future Improvements (v2 Roadmap)

- Add few-shot examples for each extractor (improves accuracy on edge cases)
- Add LOINC code suggestion in lab extractor output
- Add ICD-10 code suggestion in notes extractor output
- Implement confidence scoring based on extraction completeness
- Multi-document correlation (e.g., linking a lab report to the patient's discharge note)
