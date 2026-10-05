# Reliability of LLM-Based Structured Clinical Information Extraction

**Independent Research Project | 2026**

This project investigates the reliability of large language models (LLMs) for converting clinical dialogue into structured data.

The work began from a practical observation: an LLM output can satisfy a strict JSON schema while still being incorrect in meaning. Information may be omitted, mapped to the wrong field, assigned an incorrect status or certainty, or introduced without sufficient support from the source transcript.

The project therefore separates **structural validity** from **semantic reliability** and studies how schema design and source-evidence quality affect both.

> **Research status:** Study 1 and Study 2 are complete. A third-stage intervention study focused on evidence-aware abstention and verification is being designed.

---

## Research Questions

The project has developed through a sequence of research questions.

### Study 1 — Structural vs. Semantic Reliability

**Do schema constraints improve both structural validity and semantic reliability in LLM-generated structured information?**

### Study 2 — Evidence, Schema Design, and Semantic Failure

After Study 1 showed that structural validity could improve without corresponding semantic improvement, the follow-up question became:

**Under what evidence and schema conditions do semantically incorrect structured predictions occur?**

Study 2 examines the interaction among:

- source-evidence quality,
- target-field semantics,
- schema representation, and
- model commitment or abstention behavior.

### Planned Stage 3 — Evidence-Aware Reliability

The next research question is:

**Can an LLM be made better at recognizing when available evidence is insufficient to justify a structured prediction and decide when to commit, abstain, verify, or seek additional evidence?**

Stage 3 is currently a planned research direction and has not yet been executed.

---

# Key Findings

## Study 1

On **35 held-out ACI-BENCH clinical dialogues**:

| Condition | Schema-Valid Outputs | Schema Errors | Semantic Errors |
|---|---:|---:|---:|
| Baseline | 0 / 35 | 848 | 71 |
| Schema-Constrained | 35 / 35 | 0 | 83 |

Schema-constrained generation completely eliminated the observed structural violations, improving schema validity from **0/35 to 35/35**.

However, semantic errors did not improve. They increased from **71 baseline errors to 83 constrained errors**.

> **Finding:** Structural compliance does not guarantee semantic reliability.

---

## Study 2

A controlled follow-up experiment evaluated:

- **48 source scenarios**
- **4 schema variants per scenario**
- **192 total generations**

Source evidence was systematically varied across:

- explicit,
- ambiguous,
- absent, and
- contradictory evidence.

Schema representation was varied across:

- forced-required,
- required-nullable,
- optional, and
- semantically richer schemas.

Four structured extraction fields were examined:

- medication action,
- symptom status,
- follow-up condition, and
- symptom severity.

### Overall Semantic Results

| Metric | Result |
|---|---:|
| Total generations | 192 |
| Semantically correct | 165 |
| Semantic errors | 27 |
| Semantic accuracy | 85.94% |

### Results by Evidence Condition

| Evidence Condition | Correct | Errors | Accuracy |
|---|---:|---:|---:|
| Explicit | 48 / 48 | 0 | 100% |
| Ambiguous | 31 / 48 | 17 | 64.58% |
| Absent | 42 / 48 | 6 | 87.50% |
| Contradictory | 44 / 48 | 4 | 91.67% |

Semantic failures were strongly concentrated under **ambiguous evidence**, whereas all generations under explicit evidence were semantically correct in this controlled experiment.

### Results by Schema Variant

| Schema Variant | Correct | Errors | Accuracy |
|---|---:|---:|---:|
| Forced-required | 40 / 48 | 8 | 83.33% |
| Required-nullable | 42 / 48 | 6 | 87.50% |
| Optional | 42 / 48 | 6 | 87.50% |
| Semantically richer | 41 / 48 | 7 | 85.42% |

No schema representation was universally superior.

Importantly, simply allowing a field to be `null` or omitted did **not** consistently prevent the model from making an incorrect commitment under ambiguous evidence.

> **Finding:** Abstention capability does not necessarily produce appropriate abstention behavior.

### Results by Target Field

| Target Field | Correct | Errors | Accuracy |
|---|---:|---:|---:|
| `follow_up.condition` | 42 / 48 | 6 | 87.50% |
| `medications[].action[]<item>` | 42 / 48 | 6 | 87.50% |
| `symptoms[].severity` | 47 / 48 | 1 | 97.92% |
| `symptoms[].status` | 34 / 48 | 14 | 70.83% |

The effect of ambiguity was therefore not uniform across fields.

For example, under ambiguous evidence:

- follow-up condition: **0/12 errors**
- medication action: **6/12 errors**
- symptom severity: **1/12 error**
- symptom status: **10/12 errors**

This suggests that source ambiguity interacts with the semantics of the target field rather than producing a uniform degradation across all structured variables.

---

# Main Interpretation

Together, Study 1 and Study 2 suggest that reliable structured LLM generation cannot be reduced to schema compliance alone.

The results are most consistent with semantic reliability being shaped by the interaction among:

**source evidence × target-field semantics × schema representation**

Three findings are particularly important:

1. **Perfect structural validity can coexist with semantic error.**

2. **Ambiguous evidence is a major failure condition in the controlled Study 2 scenarios.**

3. **Providing an abstention mechanism is different from ensuring that a model recognizes when it should abstain.**

Richer schemas also helped selectively rather than universally. For example, the semantically richer representation produced **12/12 correct outputs for follow-up conditions**, while simpler schemas performed better for symptom severity.

These results motivate the next stage of the project: moving from diagnosing semantic failures toward designing mechanisms that help models recognize insufficient evidence before committing to a prediction.

---

# Study 1 — Dataset and Experimental Design

Study 1 uses the publicly available [ACI-BENCH](https://github.com/wyim/aci-bench) clinical dialogue dataset.

The experiments use the Task B test data containing doctor-patient transcripts, reference clinical notes, and case identifiers.

| Split | Cases | Purpose |
|---|---:|---|
| Development / Pilot | 5 | Schema and evaluation-protocol development |
| Held-out Evaluation | 35 | Frozen final evaluation |
| **Total** | **40** | |

The original ACI-BENCH dataset is not redistributed in this repository.

The held-out experiment used the same frozen:

- model family,
- prompt conditions,
- JSON Schema,
- evaluation definitions, and
- structural validation procedure.

**Model:** `gemini-3.5-flash`

---

# Study 1 Generation Conditions

## Baseline Generation

The model receives the source clinical transcript and instructions describing the desired structured output.

No JSON Schema is enforced during generation.

## Schema-Constrained Generation

The same source transcript is provided, but generation is constrained using a frozen JSON Schema defining:

- required fields,
- object structures,
- data types,
- enumerations, and
- allowed properties.

The goal is to isolate the effect of schema enforcement from the semantic correctness of the resulting extraction.

---

# Structured Clinical Schema

The Study 1 schema represents major clinical-information categories including:

- patient information,
- encounter information,
- medical history,
- symptoms,
- medications,
- physical examination,
- diagnostic tests,
- assessment,
- plan, and
- follow-up.

It also distinguishes semantic states such as:

- present vs. denied vs. resolved symptoms,
- current vs. recommended medications,
- reviewed vs. ordered tests,
- confirmed vs. suspected assessments, and
- medication actions including start, continue, increase, decrease, refill, stop, and recommend.

Structural validation uses **JSON Schema Draft 2020-12**.

---

# Evaluation Framework

## Structural Evaluation

Generated outputs are evaluated for:

- JSON validity,
- JSON Schema validity,
- required-field violations,
- additional-property violations,
- type violations, and
- enumeration violations.

## Semantic Evaluation

Outputs are evaluated against the source evidence using a frozen error taxonomy.

| Error Type | Description |
|---|---|
| **Partial extraction** | Correct core information is captured, but an important supported detail is missing |
| **Field mis-mapping** | Supported information is assigned to the wrong structured field |
| **Status / certainty error** | Information is extracted but assigned an incorrect status or certainty |
| **Omission** | Supported information that should be represented is missing |
| **Unsupported inference** | A factual or action claim is introduced without sufficient support from the source |

The transcript is treated as the primary evidence source for Study 1 semantic adjudication.

---

# Study 1 Error-Category Analysis

Full category-level annotations were available for a **14-case held-out subset**.

Across these cases:

| Error Category | Baseline | Schema-Constrained |
|---|---:|---:|
| Field mis-mapping | 2 | 1 |
| Omission | 26 | 15 |
| Partial extraction | 7 | 8 |
| Status / certainty error | 2 | 5 |
| Unsupported inference | 2 | 11 |
| **Total** | **39** | **40** |

These category-level counts should not be interpreted as covering all 35 held-out cases.

The subset illustrates why aggregate semantic-error counts alone are insufficient: schema constraints may reduce some forms of error while increasing others.

---

# Study 2 — Controlled Mechanism Analysis

Study 2 was designed after Study 1 to investigate possible conditions associated with semantic failure.

Unlike Study 1, which uses naturally occurring clinical dialogues, Study 2 uses **controlled source scenarios** that systematically vary the evidence available for a target structured prediction.

The experiment contains:

- 48 source scenarios,
- 4 schema variants,
- 192 total generations,
- 4 target fields,
- 4 evidence conditions.

All 192 generations were completed and adjudicated.

## Evidence Conditions

- **Explicit:** the source clearly supports the target value.
- **Ambiguous:** the source allows multiple plausible interpretations.
- **Absent:** the source does not contain evidence supporting a target value.
- **Contradictory:** the source contains conflicting evidence.

## Schema Conditions

- **Forced-required**
- **Required-nullable**
- **Optional**
- **Semantically richer**

The purpose is not simply to compare schema accuracy, but to examine how schema representation interacts with the evidence available to the model.

---

# Study 2 Error Distribution

Across 27 semantic errors:

| Error Category | Count |
|---|---:|
| Status / certainty error | 14 |
| Field mis-mapping | 5 |
| Unsupported inference | 4 |
| Omission | 3 |
| Partial extraction | 1 |

The concentration of status/certainty errors is also consistent with the lower observed reliability of `symptoms[].status` relative to the other tested fields.

---

# Reproducibility

The project uses several safeguards to preserve experimental integrity:

- frozen evaluation protocols,
- frozen schemas,
- frozen semantic-error definitions,
- fixed generation configurations,
- matched experimental conditions,
- SHA-256 hashes of experimental artifacts,
- checkpointed generation outputs,
- preserved gold annotations, and
- separation between Study 1 and Study 2 artifacts.

Study 2 generation completed successfully for all **192/192 experimental slots** with valid generated responses.

Completed frozen outputs are not regenerated during analysis.

---

# Large Experimental Artifacts

The complete Study 2 generation package is too large to store directly in the main Git repository.

The repository therefore contains the components needed to understand and audit the experiment, including:

- experimental design,
- scenario specifications,
- analysis code,
- aggregate results,
- tables,
- figures,
- adjudication methodology,
- artifact manifests, and
- SHA-256 hashes identifying frozen outputs.

Large raw artifacts may be distributed separately where appropriate.

Their omission from the Git repository does not affect the reported aggregate analyses.

---

# Limitations

The results should be interpreted within the scope of the experimental design.

Important limitations include:

- Study 1 evaluates a single frozen model family.
- Study 2 uses controlled synthetic scenarios rather than estimating real-world clinical error prevalence.
- Study 2 contains three independently authored scenarios per field × evidence condition.
- Comparisons across evidence conditions are descriptive because the evidence conditions are represented by different source scenarios rather than perfectly paired transformations.
- Small numbers of discordant paired observations limit statistical conclusions about individual schema variants.
- The experiments evaluate structured information extraction reliability, not clinical diagnosis or medical decision-making.
- The findings should therefore be interpreted as evidence about model behavior under the tested conditions rather than universal causal claims.

---

# Planned Stage 3 — Evidence-Aware Abstention and Verification

Study 1 established that schema enforcement can solve structural validity without solving semantic reliability.

Study 2 then showed that semantic failure is strongly associated with evidence ambiguity in the controlled scenarios and that allowing abstention does not guarantee appropriate abstention behavior.

Stage 3 will investigate whether an explicit evidence-aware mechanism can improve this decision process.

The central question is:

> **Can an LLM recognize when the source evidence is insufficient to justify a structured prediction and appropriately choose to commit, abstain, verify, or request additional evidence?**

Potential mechanisms under consideration include:

- explicit evidence-support assessment before prediction,
- confidence or support-state representation,
- evidence-linked verification,
- targeted self-checking before commitment,
- retrieval or clarification triggers, and
- revision of an initial prediction when evidence does not sufficiently support it.

The Stage 3 protocol has **not yet been frozen**, and no Stage 3 results are currently reported.

---

# Research Direction

The broader goal of this work is to move beyond treating valid structured output as equivalent to reliable reasoning.

The current research direction is toward **evidence-aware LLM and agentic systems** that can:

- evaluate whether available evidence supports a conclusion,
- distinguish uncertainty from absence,
- avoid unsupported commitment,
- seek additional information when necessary,
- verify intermediate conclusions, and
- revise decisions when new evidence becomes available.

---

# Repository Structure

```text
llm-reliability-structured-extraction/
│
├── README.md
│
├── paper/
│   └── manuscript.pdf
│
├── study1/
│   ├── notebooks/
│   ├── schema/
│   ├── analysis/
│   └── results/
│
├── study2/
│   ├── protocol/
│   ├── scenarios/
│   ├── analysis/
│   ├── tables/
│   └── figures/
│
├── reproducibility/
│   ├── artifact_manifest.json
│   └── artifact_hashes.txt
│
└── docs/
    ├── methodology.md
    └── semantic_adjudication_protocol.md
```

---

## Project Status

**Study 1:** Complete  
**Study 2:** Complete  
**Stage 3:** Experimental design / planning  
**Manuscript:** In preparation

This repository represents independent research and should not be interpreted as a peer-reviewed publication unless explicitly stated otherwise.
