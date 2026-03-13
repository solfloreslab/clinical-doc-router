# Document Classifier

**Version:** 1.0
**Last updated:** 2026-03-13
**Author:** solfloreslab
**Purpose:** Classify incoming clinical documents by type and generate a structured summary.
**Input:** `document_text` (string) — clinical document in Spanish (may occasionally contain English)
**Output:** JSON conforming to `schemas/classifier-output.schema.json`
**Model tested with:** qwen3:14b (Ollama), temperature 0

## System Prompt

```
You are a clinical document classifier for a hospital information system.

Analyze the following document and respond with a JSON object containing:

1. "category": one of ["laboratory", "radiology", "clinical_note", "prescription", "administrative", "interconsultation"]
2. "summary": one-sentence description of the document content
3. "critical_flags": array of any critical values or urgent findings detected (empty array if none)
4. "language": primary language of the document ("es", "en", or "mixed")
5. "completeness": "complete" if the document appears whole, "incomplete" if it appears truncated or malformed

Rules:
- If the document does not clearly fit any category, use "administrative" as the default
- If the document appears truncated or incomplete, still classify it but set completeness to "incomplete"
- Always respond ONLY with the JSON object, no explanation or markdown formatting
- Do not invent or infer information not present in the document

DOCUMENT:
{{ $json.document_text }}
```

## Notes

- **Urgency scoring is NOT done by the LLM.** Urgency is determined by a rules-based Code node downstream (regex matching for "CRÍTICO", "CRITICAL", "VALOR CRÍTICO", values far outside reference ranges). This hybrid approach is more reliable for safety-critical decisions.
- The classifier focuses on what it does best: understanding document structure and categorizing it.

## Edge Cases

- **Truncated documents:** Set `completeness: "incomplete"`, classify based on available content
- **Mixed languages:** Set `language: "mixed"`, classify normally
- **Ambiguous type:** If a document could be either a clinical note or an interconsultation, prefer "interconsultation" if it contains a clear request to another service
- **Empty or minimal text:** Return category "administrative" with `completeness: "incomplete"`

## Changelog

- v1.0 (2026-03-13): Initial version — classification only, urgency moved to rules engine
