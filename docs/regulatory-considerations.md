# Regulatory Considerations

> **This document analyzes the regulatory landscape that would apply if a system like the Clinical Document Router were deployed in a clinical setting within the European Union. This project is a demonstration only — it has not undergone any regulatory assessment.**

## 1. EU Artificial Intelligence Act (Regulation 2024/1689)

### Risk Classification

The EU AI Act classifies AI systems by risk level. A clinical document routing system that processes patient data and influences clinical workflows would likely fall under **high-risk AI** (Annex III, Section 5):

> *"AI systems intended to be used as safety components in the management and operation of [...] healthcare"*

Specifically, a system that:
- Classifies clinical documents to determine routing priority
- Detects critical laboratory values and triggers alerts
- Influences the order in which clinicians review results

...would meet the criteria for a high-risk AI system because errors (false negatives on critical values, misclassification of urgent documents) could have direct patient safety consequences.

### Requirements for High-Risk AI Systems (Title III, Chapter 2)

If deployed in production, the following would be required:

| Requirement | Article | What It Means |
|------------|---------|---------------|
| Risk management system | Art. 9 | Continuous identification and mitigation of risks throughout the system lifecycle |
| Data governance | Art. 10 | Training and validation data must be relevant, representative, and free from bias |
| Technical documentation | Art. 11 | Complete documentation of system design, development, and testing |
| Record-keeping | Art. 12 | Automatic logging of system operations for traceability |
| Transparency | Art. 13 | Users must understand the system's capabilities and limitations |
| Human oversight | Art. 14 | Humans must be able to override or halt the system |
| Accuracy, robustness, cybersecurity | Art. 15 | The system must perform consistently and be resilient to attacks |
| Conformity assessment | Art. 43 | Third-party or self-assessment before placing on the market |

### How This Demo Relates

This demonstration workflow implements several patterns that align with EU AI Act principles:

- **Human oversight:** The workflow outputs structured data for human review; it does not make autonomous clinical decisions
- **Transparency:** All prompts are documented with version history; the urgency engine uses auditable, deterministic rules
- **Record-keeping:** Every response includes metadata (model, prompt version, timestamp, processing time)
- **Hybrid approach:** Safety-critical urgency scoring uses rules (not LLM), reducing the AI risk surface

However, this demo does **NOT** satisfy the full requirements. It lacks formal risk assessment, bias evaluation, conformity assessment, and continuous monitoring.

## 2. EU Medical Device Regulation (MDR 2017/745)

### Software as a Medical Device (SaMD)

Under MDR, software qualifies as a medical device if it:
- Is intended by the manufacturer to be used for medical purposes
- Performs a function that meets the definition of a medical device

A clinical document router **could** qualify as SaMD if it:
- Makes or influences clinical decisions (e.g., prioritizing critical results)
- Is marketed as a tool for improving patient safety or clinical outcomes

### MDCG 2019-11 Guidance

The Medical Device Coordination Group guidance on software qualification states that software performing the following functions is likely a medical device:
- Analysis of patient data to aid diagnosis
- Triage or prioritization of clinical information
- Alert generation based on clinical thresholds

The critical value alert feature of this workflow would likely qualify under this guidance.

### Classification

If classified as a medical device, a clinical document router would likely be **Class IIa** under MDR Rule 11 (software intended to provide information used for diagnostic or therapeutic decisions).

### This Demo's Position

This project is explicitly **NOT** a medical device:
- It is not intended for clinical use
- It uses synthetic data only
- It has not undergone clinical validation
- It carries a clear disclaimer (see `DISCLAIMER.md`)

## 3. General Data Protection Regulation (GDPR / RGPD)

### Special Category Data

Clinical documents contain **special category personal data** under Article 9 of GDPR:
- Health data
- Patient identification
- Clinical findings and diagnoses

Processing special category data requires:
1. **Lawful basis:** Article 9(2)(h) — processing for healthcare purposes under the responsibility of a healthcare professional
2. **Data Protection Impact Assessment (DPIA):** Required under Article 35 when processing health data at scale
3. **Data minimization:** Only process the minimum data necessary
4. **Purpose limitation:** Data processed only for the stated purpose
5. **Storage limitation:** Retain processed data only as long as necessary

### On-Premise Architecture as Privacy Safeguard

This workflow is designed for fully on-premise deployment:

| Component | Location | Data Flow |
|-----------|----------|-----------|
| n8n | Hospital network | No external connections |
| Ollama (LLM) | Hospital network | Local inference only |
| Document storage | Hospital network | Documents never leave the perimeter |

**No patient data is sent to external APIs, cloud services, or third-party providers.** This architecture eliminates the primary GDPR risk of international data transfers (Chapter V).

### This Demo's Position

- All data in this repository is **entirely synthetic**
- No real PHI (Protected Health Information) has been processed
- The architecture demonstrates privacy-by-design principles

## 4. Spanish National Regulations

### Ley Orgánica 3/2018 (LOPDGDD)

Spain's implementation of GDPR adds specific provisions for health data processing, including requirements for healthcare organizations to designate a Data Protection Officer (DPO).

### Real Decreto 311/2022 (Esquema Nacional de Seguridad)

Public healthcare institutions in Spain must comply with the National Security Framework, which establishes security measures for information systems processing health data.

### Interoperability Requirements

The Spanish Ministry of Health promotes interoperability through:
- **Historia Clínica Digital del SNS (HCDSNS):** National framework for shared electronic health records
- **HL7 Spain:** National affiliate promoting FHIR adoption in Spanish healthcare

A production deployment would need to align with these national interoperability standards.

## 5. Clinical Safety Considerations

### Risk Analysis (Informative)

Even as a demonstration, it is valuable to consider the clinical risks:

| Risk | Severity | Mitigation in This Design |
|------|----------|--------------------------|
| False negative: critical value not detected | **High** | Hybrid approach: regex rules + LLM. Rules catch known patterns; LLM provides secondary detection |
| Misclassification: lab report routed as admin | Medium | LLM classifier with structured categories; fallback to `administrative` is safe (delays, not misses) |
| Incorrect extraction: wrong medication dose | **High** | Prompt explicitly requires EXACT dosage extraction; data_quality flag for incomplete documents |
| System downtime: documents not processed | Medium | Existing manual processes remain as fallback; system is non-blocking |
| LLM hallucination: invented clinical data | Medium | Extraction prompts include "do not invent" instruction; output validation against JSON schemas |

### Human-in-the-Loop Requirement

This workflow is designed as a **decision support tool**, not an autonomous decision-maker. All outputs require human review before clinical action. The alerts flag urgency for human attention — they do not trigger automatic clinical interventions.

## Summary

| Regulation | Applies If Deployed? | Status of This Demo |
|-----------|---------------------|-------------------|
| EU AI Act (high-risk) | Yes | Not assessed — demo only |
| EU MDR (SaMD, Class IIa) | Likely | Not a medical device — demo only |
| GDPR / RGPD | Yes | Synthetic data only — no real PHI |
| Spanish LOPDGDD | Yes | Not applicable — no real data processed |
| ENS (National Security) | Yes (public hospitals) | Not applicable — local demo |

---

*This regulatory analysis is provided for educational and portfolio purposes. It does not constitute legal advice. Any production deployment of AI systems in healthcare settings must involve qualified regulatory, legal, and clinical safety professionals.*
