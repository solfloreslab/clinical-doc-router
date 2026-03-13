# Use Cases

## 1. Emergency Department — Critical Lab Value Triage

**Scenario:** The hospital laboratory processes hundreds of blood tests per shift. Results are sent electronically, but critical values (e.g., glucose > 400 mg/dL, potassium > 6.0 mEq/L) require immediate physician notification.

**Current process:** Lab technicians manually review each result, flag critical values, and call the ordering physician by phone. This process is error-prone during peak hours and adds delays.

**With Clinical Document Router:**
- Lab results are sent to the webhook as they are validated
- The classifier identifies the document as `laboratory`
- The rules-based urgency engine detects critical value patterns (urgency 5)
- The lab extractor pulls structured data including exact values and reference ranges
- An alert payload is generated with the critical values and ordering physician's name
- The alert system notifies the physician within seconds of result validation

**Impact:** Reduces critical value notification time from minutes to seconds. Deterministic rules ensure no critical value is missed.

## 2. Outpatient Clinics — Document Pre-Classification

**Scenario:** A large outpatient center receives faxes, emails, and scanned documents from external providers — referral letters, external lab results, imaging reports, insurance forms. Administrative staff manually sorts each document before filing in the EHR.

**Current process:** Staff opens each document, reads it, determines the type, and files it in the correct EHR section. During high-volume periods, documents queue up and clinical records may be incomplete during consultations.

**With Clinical Document Router:**
- Documents are digitized and sent to the webhook
- Automatic classification separates clinical from administrative documents
- Clinical documents are further categorized (lab, radiology, notes)
- Structured data is extracted and pre-populated in EHR staging areas
- Administrative documents are logged but not processed with AI (cost-efficient)

**Impact:** Reduces manual classification time. Clinical documents are available in structured form before the patient's appointment.

## 3. Hospital Administration — Document Flow Analytics

**Scenario:** Hospital management needs to understand document processing volumes, bottlenecks, and patterns to optimize staffing and resource allocation.

**With Clinical Document Router:**
- Every processed document generates metadata: category, source, timestamp, processing time
- The `metadata` field in each response feeds directly into a BI dashboard
- Patterns emerge: peak lab result times, external referral volumes, administrative document ratios

**Metrics available:**
- Documents processed per hour/day/week by category
- Average processing time by document type
- Critical value detection frequency
- Source distribution (scanner vs. email vs. portal)
- Classification confidence distribution

## 4. Digital Transformation — Replacing Analog Circuits

**Scenario:** A hospital transitioning from paper-based to digital workflows needs to demonstrate quick wins to build momentum and overcome resistance to change.

**With Clinical Document Router:**
- Deployed as a pilot in one department (e.g., laboratory)
- Staff sees immediate benefit: faster document filing, automatic critical alerts
- Success metrics build the case for expanding to other departments
- The workflow is modular — new document types can be added without redesigning the system

**Change management alignment:**
- Non-invasive: the workflow sits between existing systems, does not replace them
- Reversible: staff can continue manual processes in parallel during the pilot
- Measurable: built-in metrics demonstrate ROI from day one

## 5. Patient Communication — Plain Language Summaries

**Scenario:** Patients receive clinical documents (discharge notes, lab results) through the patient portal but struggle to understand medical terminology.

**With Clinical Document Router:**
- Clinical notes pass through the patient-friendly rewriter
- Medical abbreviations are expanded, technical terms are explained in plain language
- The output is structured: "What we found", "What it means", "What to do", "When to worry"
- Medication instructions are preserved exactly (safety-critical)

**Impact:** Improves patient comprehension, reduces unnecessary calls to the clinic for clarification, and supports shared decision-making.
