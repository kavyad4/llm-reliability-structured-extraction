# Reliability of LLM-Based Structured Clinical Information Extraction

**Independent Research Project | 2026**

This project investigates the reliability of large language models (LLMs) for
converting clinical dialogue into structured JSON.

The central motivation is that an LLM output can be syntactically valid while
still being unreliable: information may be omitted, mapped to the wrong field,
assigned an incorrect status, or introduced without sufficient support from
the source transcript.

This study evaluates whether explicit schema constraints improve both
**structural validity** and **semantic reliability** in structured LLM
generation.

> **Status:** Ongoing research. The development/pilot phase is complete and the
> frozen held-out evaluation is currently in progress.

---

## Research Question

**How effectively do schema constraints improve the reliability of
LLM-generated structured clinical information?**

The study examines two different dimensions of reliability:

1. **Structural reliability**
   - Is the output valid JSON?
   - Does it satisfy the required JSON Schema?
   - Are required fields present?
   - Are field types and allowed values correct?

2. **Semantic reliability**
   - Did the model extract the supported information?
   - Was the information mapped to the correct field?
   - Were important details omitted?
   - Were status or certainty labels correct?
   - Did the model introduce unsupported information?

The larger goal is to distinguish **structural correctness** from
**semantic correctness**.

---

## Why This Matters

Structured generation is increasingly used to transform unstructured text into
machine-readable data.

In clinical and other high-stakes workflows, simply producing valid JSON is not
enough.

For example, a model may correctly identify that a medication appears in a
conversation but incorrectly label the medication action as `continue`,
`stop`, or `start`.

Such an output may be perfectly schema-valid while still being semantically
incorrect.

This project studies these failure modes separately rather than treating
successful JSON generation as equivalent to reliable extraction.

---

## Dataset

This study uses the publicly available
[ACI-BENCH](https://github.com/wyim/aci-bench) clinical dialogue dataset.

The experiments use the Task B test set,
`clinicalnlp_taskB_test1.json`, containing doctor-patient transcripts,
reference clinical notes, and case identifiers.

The notebook downloads the dataset directly from the official ACI-BENCH
repository to preserve reproducibility.

| Split | Cases | Purpose |
|---|---:|---|
| Development / Pilot | 5 | Schema and evaluation development |
| Held-out Evaluation | 35 | Frozen evaluation |
| **Total** | **40** | |

The original dataset is not redistributed in this repository.

## Experimental Design

Two generation conditions are currently compared.

### A. Baseline Generation

The model receives the clinical transcript and a lightweight description of
the desired JSON structure.

No JSON Schema is enforced during generation.

### B. Schema-Constrained Generation

The same transcript is provided to the model, but generation is constrained by
a frozen JSON Schema.

The schema specifies required fields, object structures, types, enumerations,
and allowed properties.

The held-out experiment uses:

**Model:** `gemini-3.5-flash`

The same frozen model, schema, prompt templates, and evaluation rules are used
throughout the held-out experiment.

---

## Structured Clinical Schema

The project defines a structured schema covering major clinical information
categories:

- Patient
- Encounter
- Medical history
- Symptoms
- Medications
- Physical examination
- Diagnostic tests
- Assessment
- Plan
- Follow-up

The schema includes explicit distinctions such as:

- present vs. denied vs. resolved symptoms,
- current vs. recommended medications,
- reviewed vs. ordered diagnostic tests,
- confirmed vs. suspected assessments, and
- medication actions such as start, continue, increase, decrease, refill,
  stop, and recommend.

The schema is validated using JSON Schema Draft 2020-12.

---

## Evaluation Framework

### Structural Evaluation

Each generated output is evaluated for:

- JSON validity
- JSON Schema validity
- required-field violations
- additional-property violations
- type violations
- enumeration violations

### Semantic Evaluation

Model outputs are manually reviewed against the source transcript using five
frozen error categories:

| Error Type | Description |
|---|---|
| **Partial extraction** | Correct core information is captured, but an important supported detail is missing |
| **Mapping error** | Supported information is assigned to the wrong structured field |
| **Status / certainty error** | Information is extracted but assigned an incorrect clinical status or certainty |
| **Omission** | Supported information that should be represented is missing |
| **Unsupported inference** | The model introduces a factual or action claim not supported by the transcript |

The transcript is treated as the primary evidence source during semantic
adjudication.

---

## Development / Pilot Results

Five cases were used for development and evaluation-protocol refinement.

### Structural Results

| Condition | Schema-Valid Outputs | Total Schema Errors |
|---|---:|---:|
| Baseline | 0 / 5 | 182 |
| Schema-Constrained | 5 / 5 | 0 |

### Semantic Results

| Condition | Observed Semantic Errors |
|---|---:|
| Baseline | 41 |
| Schema-Constrained | 16 |

This corresponds to an observed reduction from **41 to 16 semantic errors**
across the five development cases.

**Important:** These are development/pilot results and are not reported as
held-out performance.

---

## Preliminary Held-Out Results

The held-out evaluation contains **35 cases** and is currently in progress.

For the first **3 completed paired cases**:

### Structural Reliability

| Condition | Schema-Valid Outputs | Total Schema Errors |
|---|---:|---:|
| Baseline | 0 / 3 | 68 |
| Schema-Constrained | 3 / 3 | 0 |

### Semantic Reliability

| Condition | Semantic Errors |
|---|---:|
| Baseline | 15 |
| Schema-Constrained | 6 |

This represents an **interim observed reduction of 60%**, but the sample is
currently too small for a final conclusion.

A particularly interesting preliminary pattern is visible across error types:

| Error Category | Baseline | Schema-Constrained |
|---|---:|---:|
| Partial extraction | 4 | 2 |
| Mapping error | 1 | 0 |
| Status / certainty error | 0 | 0 |
| Omission | 10 | 1 |
| Unsupported inference | 0 | 3 |

The preliminary results suggest an important trade-off:

> **Schema constraints can eliminate structural violations and substantially
> reduce omissions, while still allowing — and in some cases introducing —
> unsupported semantic commitments.**

This observation is preliminary and will be reassessed after the full held-out
evaluation is complete.

---

## Reproducibility

The experimental workflow includes several safeguards intended to make the
evaluation reproducible:

- frozen schema before held-out evaluation,
- frozen prompt templates,
- frozen semantic error definitions,
- fixed held-out model,
- identical cases across generation conditions,
- SHA-256 hashes of raw model outputs,
- structural validation using the same schema for both conditions, and
- checkpointing of completed generation results.

Completed outputs are not regenerated once frozen.

---

## Current Research Status

### Completed

- Public dataset preparation
- Structured clinical schema development
- Gold annotation development
- Semantic error taxonomy
- Five-case development/pilot experiment
- Baseline generation condition
- Schema-constrained generation condition
- Structural validation pipeline
- Frozen held-out evaluation protocol

### In Progress

- 35-case held-out generation
- Manual semantic adjudication
- Error-category analysis
- Final statistical comparison

The results in this repository should therefore be interpreted as
**work in progress rather than final study conclusions**.

---

## Repository Structure

```text
llm-reliability-structured-extraction/
│
├── README.md
│
├── notebooks/
│   └── LLM_Reliability_Experiments.ipynb
│
├── schema/
│   └── clinical_schema.json
│
├── results/
│   └── heldout_results_summary.json
│
└── docs/
    └── research_overview.pdf
